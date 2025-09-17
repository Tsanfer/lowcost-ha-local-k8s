---
draft: false
title: '部署 K8s 管理面板 Kuboard'
weight: 2
---

参考链接：

[Kuboard_Kubernetes教程_K8S安装_管理界面](https://kuboard.cn/)

[Kubernetes的边界 | Kuboard](https://kuboard.cn/learning/k8s-bg/what-is-k8s.html#kubernetes%E7%9A%84%E8%BE%B9%E7%95%8C)

> Kubernetes 不是什么：
>
> - 不部署源码、不编译或构建应用程序。持续集成、分发、部署（CI/CD）的工作流极大程度上取决于组织的文化、偏好以及技术要求。Kubernetes可以作为部署平台参与到 CI/CD 流程，但是不涉及镜像构建和分发的过程
>
>   译者注：可选的有 `Jenkins `/ `Gitlab Runner` / `docker registry` / `harbor`等
>
> ---
>
> - 不提供应用程序级别的服务，包括：中间件（例如，消息总线）、数据处理框架（例如，Spark）、数据库（例如，mysql）、缓存（例如，Redis），或者分布式存储（例如，Ceph）。此类组件可以在 Kubernetes 上运行，或者可以被运行在 Kubernetes 上的应用程序访问
>   不限定日志、监控、报警的解决方案。Kubernetes 提供一些样例展示如何与日志、监控、报警等组件集成，同时提供收集、导出监控度量（metrics）的一套机制。您可以根据自己的需要选择日志、监控、报警组件
>
>   译者注：可选的有 `ELK`/ `Prometheus`/ `Graphana`/ `Pinpoint`/ `Skywalking` / `Metrics Server` 等
>
> ---
>
> - 不提供或者限定配置语言（例如，jsonnet）。Kubernetes提供一组声明式的 API，您可以按照自己的方式定义部署信息。
>
>   译者注：可选的有 `helm`/`kustomize`/`kubectl`/`kubernetes dashboard`/`kuboard`/`octant`/`k9s`等
>
> ---
>
> - 不提供或限定任何机器的配置、维护、管理或自愈的系统。
>
>   译者注：在这个级别上，可选的组件有 `puppet`、`ansible`、`salt stack` 等

参考链接：

[安装 Kubernetes 多集群管理工具 - Kuboard v4 | Kuboard](https://kuboard.cn/v4/install/)

## 快速安装 Kuboard V4

​`~/kuboard-quick/docker-compose.yml`​

```yaml
configs:
  create_db_sql:
    content: |
      CREATE DATABASE kuboard DEFAULT CHARACTER SET = 'utf8mb4' DEFAULT COLLATE = 'utf8mb4_unicode_ci';
      create user 'kuboard'@'%' identified by 'kuboardpwd';
      grant all privileges on kuboard.* to 'kuboard'@'%';
      FLUSH PRIVILEGES;

services:
  db:
    image: swr.cn-east-2.myhuaweicloud.com/kuboard/mariadb:11.3.2-jammy
    # image: mariadb:11.3.2-jammy
    # swr.cn-east-2.myhuaweicloud.com/kuboard/mariadb:11.3.2-jammy 与 mariadb:11.3.2-jammy 镜像完全一致
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD: kuboardpwd
      MYSQL_ROOT_PASSWORD: kuboardpwd
      TZ: Asia/Shanghai
    volumes:
      - ./kuboard-mariadb-data:/var/lib/mysql:Z
    configs:
      - source: create_db_sql
        target: /docker-entrypoint-initdb.d/create_db.sql
        mode: 0777
    networks:
      kuboard_v4_dev:
        aliases:
          - db
  kuboard:
    image: swr.cn-east-2.myhuaweicloud.com/kuboard/kuboard:v4
    # image: eipwork/kuboard:v4
    restart: unless-stopped
    environment:
      - DB_DRIVER=org.mariadb.jdbc.Driver
      - DB_URL=jdbc:mariadb://db:3306/kuboard?serverTimezone=Asia/Shanghai
      - DB_USERNAME=kuboard
      - DB_PASSWORD=kuboardpwd
    ports:
      - "8000:80"
    depends_on:
      - db
    networks:
      kuboard_v4_dev:
        aliases:
          - kuboard

networks:
  kuboard_v4_dev:
    driver: bridge

```

部署容器

```bash
cd ~/kuboard-quick
sudo docker compose up -d
```

‍

## 通过外置数据库安装 Kuboard V4（未成功）

### 准备数据库

> 主数据库设备中执行以下命令

添加用户和数据库信息到 Pigsty 配置文件

​`~/pigsty/pigsty.yml`​

```yaml
all:
  children:
    pg-meta:
      vars:
        pg_users:
          - { name: kuboard, password: <PASSWORD>, pgbouncer: true, roles: [ dbrole_admin ], comment: pigsty admin user }
        pg_databases:
          - { name: kuboard, owner: kuboard, comment: kuboard database, schemas: [ kuboard ] }
```

使用 Pigsty 的 Ansible 剧本创建用户和数据库

```bash
cd ~/pigsty
./pgsql-user.yml -l pg-meta -e username=kuboard
./pgsql-db.yml -l pg-meta -e dbname=kuboard
```

### 启动 Kuboard

> 在部署节点中执行以下命令

​`~/kuboard/docker-compose.yml`​

```yaml
services:
  kuboard:
    image: swr.cn-east-2.myhuaweicloud.com/kuboard/kuboard:v4.0.0.0-rc.06
    # image: eipwork/kuboard:v4.0.0.0-rc.06
    container_name: kuboard
    restart: unless-stopped
    ports:
      - "80:80/tcp"
    environment:
      TZ: "Asia/Shanghai"
      DB_DRIVER: "org.postgresql.Driver"
      DB_URL: "jdbc:postgresql://<DB_IP>:<PORT>/kuboard?currentSchema=kuboard&characterEncoding=UTF8"
      DB_USERNAME: "kuboard"
      DB_PASSWORD: "<PASSWORD>"

```

此处的端口由于 Pigsty 部署了 HAProxy 和连接池，因此可以通过 `5433` 端口从 HAProxy → 连接池 → 数据库进行读写操作。

部署容器

```bash
cd ~/kuboard
sudo docker compose up -d
```

## 打开 Kuboard 界面

浏览器进入对应网址即可，比如：`http://192.168.0.31:8000`​

默认用户名：`admin`，默认密码：`Kuboard123`​

![2025-9-17_18-15-39.png](https://cdn.tsanfer.com/image/2025-9-17_18-15-39.png "Kuboard 面板")

