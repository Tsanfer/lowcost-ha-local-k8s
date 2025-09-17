---
draft: false
title: '（待补全）部署 Halo 建站工具'
weight: 2
---


{{< callout emoji="🚧" >}}
  此页内容待补全
{{< /callout >}}


### 自制应用部署

参考 docker-compose:

```yaml {linenos=table,hl_lines=[14,15,16],filename="./docker-compose.yml"}
services:
  halo:
    image: registry.fit2cloud.com/halo/halo:2.20
    restart: on-failure:3
    network_mode: "host"
    volumes:
      - ./halo2:/root/.halo2
    environment:
      # JVM 参数，默认为 -Xmx256m -Xms256m，可以根据实际情况做调整，置空表示不添加 JVM 参数
      - JVM_OPTS=-Xmx256m -Xms256m
    command:
      # 修改为自己已有的 MySQL 配置
      # - --spring.r2dbc.url=r2dbc:pool:postgresql://postgresql.halo-project.svc.cluster.local:3306/halo
      - --spring.r2dbc.url=r2dbc:pool:postgresql://xxxx.com/halo
      - --spring.r2dbc.username=halo.xxxxx
      - --spring.r2dbc.password=xxxxx
      - --spring.sql.init.platform=postgresql
      # 外部访问地址，请根据实际需要修改
      - --halo.external-url=http://localhost:8090/
      # 端口号 默认8090
      - --server.port=8090

```

### k8s 方式部（未成功）

helm 自定义值文件：

`~/halo-values.yaml`

```yaml {linenos=table,filename="./halo-values.yaml"}
# halo-values.yaml
global:
  # 国内镜像加速
  # imageRegistry: "m.daocloud.io/docker.io"
  security: 
    allowInsecureImages: true
mysql:
  enabled: false
postgresql:
  enabled: false
externalDatabase:
  platform: postgresql
  host: "xxxxx.com"
  port: "5432"
  user: "xxxx"
  password: "xxxx"
  database: "halo"

```

```bash
# 添加 Halo 项目的 Helm Charts 仓库
helm repo add halo https://halo-sigs.github.io/charts/
# 从 chart 仓库中更新本地可用chart的信息 
helm repo update

helm install halo halo/halo 
# 使用外部数据库
helm install halo halo/halo -f ~/halo-values.yaml

```

‍
