---
draft: false
title: '配置 Ingress 网关'
weight: 2
---

参考链接：[通过 Ingress 访问您的应用 | Kuboard](https://www.kuboard.cn/learning/k8s-intermediate/service/ingress.html)

- 创建 Ingress 类

  📌 位置：集群管理 → 网络 → IngressClass
- 设置 K8s 集群外部负载均衡器

  使用负载均衡器转发端口，此处先使用之前配置好的 Frp 内网穿透将局域网的 Ingress 端口映射到云服务器，然后由云服务器的 Nginx 反向代理此端口

  - Frp 配置

    > 此处的本地端口为 Ingress 创建时随机生成
    >

    |代理名称|代理类型|加密|本地 IP|本地端口|远程端口|
    | ---------------| ----------| ------| --------------| ----------| ----------|
    |k8s-control-1|tcp|✅|192.168.0.31|30519|8010|
  - 云服务器 Nginx 配置

    域名：`k8s-local.tsanfer.com`

    代理地址：`http://localhost:8010`

    > 顺便还可以在 Nginx 面板里设置下 HTTPS 需要的 SSL 证书
    >
- 创建示例工作负载

  📌 位置：名称空间 → 常用操作 → 创建工作负载

  - 基本信息

    工作负载类型: `部署（Deployment）`

    工作负载名称: `nginx-for-test-ingress`

  - 容器信息

    添加工作容器

    名称: `nginx-for-test-ingress`

    镜像: `docker.1ms.run/library/nginx:1.29.2-alpine-slim`

    镜像拉取策略: `本地不存在时拉取镜像（IfNotPresent）`
  - 服务

    服务类型：`ClusterIP`

    端口：`80`（抽象后的服务端口）

    目标端口：`80`（Pod上的端口）

    > 两个端口可以填一样的，原因在于每个服务都有自己的 IP
    >
  - 应用路由

    IngressClass：`my-ingress-controller`（从IngressClass列表选择）

    路由规则-域名：`k8s-local.tsanfer.com`

    路由规则-协议：`HTTP`（不打开 HTTPS）

    路由规则-路径映射：`前缀匹配`​ `/`​ `nginx-for-test-ingress`​ `80:80`

  之后保存即可

  > 如果镜像拉不到，可以手动删除 Pod 多拉几次
  >

- 效果

  在浏览器中访问对应域名即可

  ![image](assets/image-20260212234414-s4r269f.png)
