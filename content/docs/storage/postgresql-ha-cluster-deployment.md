---
draft: false
title: '高可用 PostgreSQL 数据库集群部署'
weight: 1
---

此处的数据库结构为：一主二从

参考教程：

[Pigsty 中文文档 v3.4 | PIGSTY](https://pigsty.cc/docs/)

---

## 准备工作

### 设置免密 sudo 权限

> 每个数据库节点都要执行以下操作

安装 pigsty 的每个节点都需要有同名的专用管理员用户，且拥有免密码 sudo 权限，用户名如：`tsanfer`，用户名不能为 `dba`​

> Pigsty 安装过程中会默认创建管理员用户 `dba`，其是不同于根用户（`root`）与数据库超级用户 (`postgres`) 的专用管理员用户

配置免密码 sudo 权限：

```bash
# 使用以下命令添加免密 sudo 规则
echo '%<USERNAME> ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/<USERNAME>

:<<EOF 比如
echo '%tsanfer ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/tsanfer
EOF
```

重新登陆当前用户 shell

```bash
su - <USERNAME>
# su - tsanfer
```

验证免密 sudo 权限：

```bash
sudo ls -la
```

### 批量设置 SSH 免密登陆

> 仅在主数据库节点执行以下操作

生成的公钥默认位于：`~/.ssh/id_rsa.pub`，私钥默认位于：`~/.ssh/id_rsa`，如果不存在则可生成新的公私钥对。

生成公私钥对：

```bash
ssh-keygen -t rsa
```

复制公钥到受控主机：

```bash
ssh-copy-id -o StrictHostKeyChecking=no 192.168.0.51
ssh-copy-id -o StrictHostKeyChecking=no 192.168.0.52
ssh-copy-id -o StrictHostKeyChecking=no 192.168.0.53
```

## 安装高可用 Pigsty 集群

> 仅在主数据库节点执行以下操作

### 安装 pigsty 命令行工具

```bash
# 中国大陆
curl https://repo.pigsty.cc/pig | bash  
# 国际区域
# curl https://repo.pigsty.io/pig | bash

```

### 配置 pigsty

```bash
# 默认安装嵌入的最新 Pigsty 版本
pig sty init

# 执行 Bootstrap，安装 Ansible
pig sty boot

# 执行 Configure，生成3节点高可用配置文件
pig sty conf -c ha/trio --ip=192.168.0.51
```

配置过程生成的配置文件默认位于：`~/pigsty/pigsty.yml`，在安装前进行检查与修改定制，更改配置文件中的 IP 地址和数据库用户密码，比如以下所示

​`~/pigsty/pigsty.yml`​

```yaml {linenos=table,hl_lines=[26,27,28,34,35,36,43,48,49,50,54,60,73],filename="~/pigsty/pigsty.yml"}
---
#==============================================================#
# File      :   trio.yml
# Desc      :   Pigsty 3-node security enhance template
# Ctime     :   2020-05-22
# Mtime     :   2025-01-23
# Docs      :   https://doc.pgsty.com/config
# License   :   AGPLv3 @ https://doc.pgsty.com/about/license
# Copyright :   2018-2025  Ruohang Feng / Vonng (rh@vonng.com)
#==============================================================#

# 3 infra node, 3 etcd node, 3 pgsql node, and 1 minio node

all:

  #==============================================================#
  # Clusters, Nodes, and Modules
  #==============================================================#
  children:

    #----------------------------------#
    # infra: monitor, alert, repo, etc..
    #----------------------------------#
    infra: # infra cluster for proxy, monitor, alert, etc
      hosts: # 1 for common usage, 3 nodes for production
        192.168.0.51: { infra_seq: 1 } # identity required
        192.168.0.52: { infra_seq: 2, repo_enabled: false }
        192.168.0.53: { infra_seq: 3, repo_enabled: false }
      vars:
        patroni_watchdog_mode: off # do not fencing infra

    etcd: # dcs service for postgres/patroni ha consensus
      hosts: # 1 node for testing, 3 or 5 for production
        192.168.0.51: { etcd_seq: 1 }  # etcd_seq required
        192.168.0.52: { etcd_seq: 2 }  # assign from 1 ~ n
        192.168.0.53: { etcd_seq: 3 }  # odd number please
      vars: # cluster level parameter override roles/etcd
        etcd_cluster: etcd  # mark etcd cluster name etcd
        etcd_safeguard: false # safeguard against purging
        etcd_clean: true # purge etcd during init process

    minio: # minio cluster, s3 compatible object storage
      hosts: { 192.168.0.51: { minio_seq: 1 } }
      vars: { minio_cluster: minio }

    pg-meta: # 3 instance postgres cluster `pg-meta`
      hosts:
        192.168.0.51: { pg_seq: 1, pg_role: primary }
        192.168.0.52: { pg_seq: 2, pg_role: replica }
        192.168.0.53: { pg_seq: 3, pg_role: replica , pg_offline_query: true }
      vars:
        pg_cluster: pg-meta
        pg_users:
          - { name: dbuser_meta , password: <PASSWORD>,pgbouncer: true ,roles: [ dbrole_admin ]    ,comment: pigsty admin user }
          - { name: dbuser_view , password: DBUser.View ,pgbouncer: true ,roles: [ dbrole_readonly ] ,comment: read-only viewer for meta database }
        pg_databases:
          - { name: meta ,baseline: cmdb.sql ,comment: pigsty meta database ,schemas: [ pigsty ] ,extensions: [ { name: vector } ] }
        pg_vip_enabled: true
        pg_vip_address: <VIP_IP>/24
        pg_vip_interface: eth1


  #==============================================================#
  # Global Parameters
  #==============================================================#
  vars:

    #----------------------------------#
    # Meta Data
    #----------------------------------#
    version: v3.6.1                   # pigsty version string
    admin_ip: 192.168.0.51             # admin node ip address
    region: china                     # upstream mirror region: default|china|europe
    node_tune: oltp                   # node tuning specs: oltp,olap,tiny,crit
    pg_conf: oltp.yml                 # pgsql tuning specs: {oltp,olap,tiny,crit}.yml
    docker_registry_mirrors: ["https://docker.1panel.live","https://docker.1ms.run","https://docker.xuanyuan.me","https://registry-1.docker.io"]
    proxy_env:                        # global proxy env when downloading packages
      no_proxy: "localhost,127.0.0.1,10.0.0.0/8,192.168.0.0/16,*.pigsty,*.aliyun.com,mirrors.*,*.myqcloud.com,*.tsinghua.edu.cn"
      # http_proxy:  # set your proxy here: e.g http://user:pass@proxy.xxx.com
      # https_proxy: # set your proxy here: e.g http://user:pass@proxy.xxx.com
      # all_proxy:   # set your proxy here: e.g http://user:pass@proxy.xxx.com
    infra_portal:                     # domain names and upstream servers
      home         : { domain: h.pigsty }
      grafana      : { domain: g.pigsty ,endpoint: "${admin_ip}:3000" , websocket: true }
      prometheus   : { domain: p.pigsty ,endpoint: "${admin_ip}:9090" }
      alertmanager : { domain: a.pigsty ,endpoint: "${admin_ip}:9093" }
      blackbox     : { endpoint: "${admin_ip}:9115" }
      loki         : { endpoint: "${admin_ip}:3100" }
      #minio        : { domain: m.pigsty ,endpoint: "${admin_ip}:9001" ,scheme: https ,websocket: true }

    #----------------------------------#
    # Repo, Node, Packages
    #----------------------------------#
    repo_remove: true                 # remove existing repo on admin node during repo bootstrap
    node_repo_remove: true            # remove existing node repo for node managed by pigsty
    repo_extra_packages: [ pg17-main ] #,pg17-core ,pg17-time ,pg17-gis ,pg17-rag ,pg17-fts ,pg17-olap ,pg17-feat ,pg17-lang ,pg17-type ,pg17-util ,pg17-func ,pg17-admin ,pg17-stat ,pg17-sec ,pg17-fdw ,pg17-sim ,pg17-etl]
    pg_version: 17                    # default postgres version
    pg_locale: C.UTF-8                  # overwrite default C local
    pg_lc_collate: C.UTF-8              # overwrite default C lc_collate
    pg_lc_ctype: C.UTF-8                # overwrite default C lc_ctype

    #pg_extensions: [pg17-time ,pg17-gis ,pg17-rag ,pg17-fts ,pg17-feat ,pg17-lang ,pg17-type ,pg17-util ,pg17-func ,pg17-admin ,pg17-stat ,pg17-sec ,pg17-fdw ,pg17-sim ,pg17-etl ] #,pg17-olap]
...
```

### 安装 Pigsty

> 集群所有节点都不能有 `dba` 用户，会产生用户冲突，进而导致安装失败

```bash
# 执行 install.yml 剧本完成部署
pig sty install
```

安装过程耗时较长，实测需要约 40 分钟的时间。

## 查看效果

配置主机名到 IP 地址的映射

- 方式一：局域网 DNS 服务器配置映射

  此处以 OpenWrt 为例，通过配置 LuCI 面板中的，network → DHCP/DNS → general → address

  ```plaintext
  /h.pigsty/g.pigsty/p.pigsty/a.pigsty/sss.pigsty/<PRIMARY_IP>
  ```

  保存并应用即可生效
- 方式二：本地 HOSTS 文件配置映射

  配置本地 hosts，映射主机名到主数据库节点

  ```plaintext
  <PRIMARY_IP> h.pigsty g.pigsty p.pigsty a.pigsty sss.pigsty
  ```

之后便可在本地通过对应主机名访问服务：

|组件|端口|域名|说明|
| --------------| --------| ------| -----------------------------|
|Nginx|80/443|​`h.pigsty`​|Web 服务总入口，本地 YUM 源|
|AlertManager|9093|​`a.pigsty`​|告警聚合/屏蔽页面|
|Grafana|3000|​`g.pigsty`​|Grafana 监控面板|
|Prometheus|9090|​`p.pigsty`​|Prometheus 管理界面|
|Minio|9001|​`sss.pigsty`​|Minio 面板|

Grafana 的默认用户名：`admin`，密码：`pigsty`​

Minio 的默认用户名：`minioadmin`，密码：`minioadmin`​

‍
