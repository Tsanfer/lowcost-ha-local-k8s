---
draft: false
title: '部署 Harbor 容器镜像仓库'
weight: 2
---

参考链接：[Harbor - Harbor 镜像仓库](https://goharbor.cn/)

## 下载 Harbor 安装包

Harbor 可通过在线与离线两种方式安装，此处为离线安装，安装包上传至服务器

1. 下载离线安装包

   参考链接：[Releases · goharbor/harbor](https://github.com/goharbor/harbor/releases)

   文件位置：`/root/harbor-offline-installer-v2.15.0.tgz`
2. 解压安装包

   ```bash
   HARBOR_VERSION="v2.15.0"
   tar -xzf harbor-offline-installer-${HARBOR_VERSION}.tgz
   ```
3. 设置配置文件

   更改配置文件名为 `harbor.yml`，然后按需修改配置文件，此处为部分示例

   文件位置：`/root/harbor/harbor.yml`

   ```yaml
   hostname: 192.168.0.62

   http:
     port: 80

   https:
     port: 443
     certificate: /root/harbor/cert/192.168.0.62.crt
     private_key: /root/harbor/cert/192.168.0.62.key

   # The initial password of Harbor admin
   # It only works in first time to install harbor
   # Remember Change the admin password from UI after launching Harbor.
   harbor_admin_password: Harbor12345

   database:
     # The password for the user('postgres' by default) of Harbor DB. Change this before any production use.
     password: root123

   # The default data volume
   data_volume: /data

   #This attribute is for migrator to detect the version of the .cfg file, DO NOT MODIFY!
   _version: 2.15.0

   # Global proxy
   # Config http proxy for components, e.g. http://my.proxy.com:3128
   # Components doesn't need to connect to each others via http proxy.
   # Remove component from `components` array if want disable proxy
   # for it. If you want use proxy for replication, MUST enable proxy
   # for core and jobservice, and set `http_proxy` and `https_proxy`.
   # Add domain to the `no_proxy` field, when you want disable proxy
   # for some special registry.
   proxy:
     http_proxy:
     https_proxy:
     no_proxy:
     components:
       - core
       - jobservice

   ```

## 配置 HTTPS 证书

> 配置证书以便使用 HTTPS 访问 Harbor 仓库

### 创建证书文件夹

```bash
mkdir -p /root/harbor/cert
cd /root/harbor/cert
```

### 生成 CA 证书

1. 生成 CA 证书私钥

   ```sh
   openssl genrsa -out ca.key 4096
   ```

2. 生成 CA 证书

   ```sh
   openssl req -x509 -new -nodes -sha512 -days 3650 \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=192.168.0.62" \
    -key ca.key \
    -out ca.crt
   ```

生成的 CA 证书文件为：`ca.key`

### 生成自签服务器证书

1. 生成服务器私钥

   ```sh
   openssl genrsa -out 192.168.0.62.key 4096
   ```

2. 生成证书签名请求

   ```sh
   openssl req -sha512 -new \
       -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=192.168.0.62" \
       -key 192.168.0.62.key \
       -out 192.168.0.62.csr
   ```

3. 配置为 SAN 和 x509 v3 标准

   ```sh
   cat > v3.ext <<-EOF
   authorityKeyIdentifier=keyid,issuer
   basicConstraints=CA:FALSE
   keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
   extendedKeyUsage = serverAuth
   subjectAltName = @alt_names

   [alt_names]
   IP.1 = 192.168.0.62
   # 如果还有其他 IP 或域名，可以继续加：
   # IP.2 = 10.0.0.100
   # DNS.1 = harbor.example.com
   EOF
   ```

4. 使用 v3.ext 文件生成证书。

   ```sh
   openssl x509 -req -sha512 -days 3650 \
       -extfile v3.ext \
       -CA ca.crt -CAkey ca.key -CAcreateserial \
       -in 192.168.0.62.csr \
       -out 192.168.0.62.crt
   ```

生成的自签证书文件是：`192.168.0.62.crt`​，生成的服务器私钥是：`192.168.0.62.key`

### 向 Docker 和 Harbor 中添加证书

需要将 CA 证书文件 `ca.key`​，自签证书 `192.168.0.62.crt`​，服务器私钥 `192.168.0.62.key`，提供给 Docker 和 Harbor

1. 将服务器证书和密钥复制到 Harbor 主机上的证书文件夹中。

   ```bash
   mkdir -p /data/cert
   cp 192.168.0.62.crt /data/cert/
   cp 192.168.0.62.key /data/cert/
   ```

2. 将 `192.168.0.62.crt`​ 转换为 `192.168.0.62.cert`​，供 Docker 使用  
   Docker 将 `.crt`​ 文件解释为 CA 证书，将 `.cert` 文件解释为客户端证书

   ```sh
   openssl x509 -inform PEM -in 192.168.0.62.crt -out 192.168.0.62.cert
   ```

3. 将服务器证书、密钥和 CA 文件复制到 Harbor 主机上的 Docker 证书文件夹中

   ```sh
   mkdir -p /etc/docker/certs.d/192.168.0.62/
   cp 192.168.0.62.cert /etc/docker/certs.d/192.168.0.62/
   cp 192.168.0.62.key /etc/docker/certs.d/192.168.0.62/
   cp ca.crt /etc/docker/certs.d/192.168.0.62/
   ```

4. 重启 Docker Engine。

   ```sh
   systemctl restart docker
   ```

## 安装 Harbor

执行安装命令

```bash
sudo /root/harbor/install.sh
```

稍等片刻后即可通过浏览器进入 harbor，笔者此处的地址为：`https://192.168.0.62`

> 若提示“核心服务不可用”，可等待一会后重新尝试登陆，可能是某些组件未加载完成

🖥 效果演示：

Harbor 管理界面如下

![2026-1-9_00-19-19.png](https://cdn.tsanfer.com/image/2026-1-9_00-19-19.png)

## 设置客户端信任自签名证书

> 所有从 Harbor 拉取镜像的容器运行时，都要设置并登陆验证，包括所有 K8s 节点

### 客户端信任自签名证书

- Docker

  复制自签名证书到 Docker 证书目录

  ```bash
  HARBOR_IP="192.168.0.62"
  HARBOR_CERT_REMOTE_DIR="/root/harbor/cert"  # Harbor 服务器上证书的路径
  HARBOR_KEY_REMOTE_FILE="${HARBOR_CERT_REMOTE_DIR}/${HARBOR_IP}.key"
  HARBOR_CERT_REMOTE_FILE="${HARBOR_CERT_REMOTE_DIR}/${HARBOR_IP}.cert"
  HARBOR_CA_REMOTE_FILE="${HARBOR_CERT_REMOTE_DIR}/ca.crt"

  DOCKER_CERT_DIR="/etc/docker/certs.d/${HARBOR_IP}"
  DOCKER_KEY_FILE="${DOCKER_CERT_DIR}/${HARBOR_IP}.key"
  DOCKER_CERT_FILE="${DOCKER_CERT_DIR}/${HARBOR_IP}.cert"
  DOCKER_CA_PATH="${DOCKER_CERT_DIR}/ca.crt"

  (
  sudo mkdir -p "$DOCKER_CERT_DIR"
  sudo scp "root@${HARBOR_IP}:${HARBOR_KEY_REMOTE_FILE}" \
       "${DOCKER_KEY_FILE}"
  sudo scp "root@${HARBOR_IP}:${HARBOR_CERT_REMOTE_FILE}" \
       "${DOCKER_CERT_FILE}"
  sudo scp "root@${HARBOR_IP}:${HARBOR_CA_REMOTE_FILE}" \
       "${DOCKER_CA_PATH}"
  )
  ```

  在系统级别信任证书（Debian 系统）

  ```bash
  HARBOR_IP="192.168.0.62"
  DOCKER_CERT_DIR="/etc/docker/certs.d/${HARBOR_IP}"
  DOCKER_CERT_FILE="${DOCKER_CERT_DIR}/${HARBOR_IP}.cert"

  sudo cp ${DOCKER_CERT_FILE} /usr/local/share/ca-certificates/${HARBOR_IP}.crt
  sudo update-ca-certificates
  ```

  重启 Docker 服务

  ```bash
  sudo systemctl restart docker
  ```

- Containerd

  复制 harbor 证书

  ```bash
  HARBOR_IP="192.168.0.62"
  HARBOR_CERT_REMOTE_DIR="/root/harbor/cert"  # Harbor 服务器上证书的路径
  HARBOR_CA_REMOTE_FILE="${HARBOR_CERT_REMOTE_DIR}/ca.crt"

  CONTAINERD_CERT_DIR="/etc/containerd/certs.d/${HARBOR_IP}"
  CONTAINERD_CA_FILE="${CONTAINERD_CERT_DIR}/ca.crt"

  (
  sudo mkdir -p "$CONTAINERD_CERT_DIR"
  sudo scp "root@${HARBOR_IP}:${HARBOR_CA_REMOTE_FILE}" \
       "${CONTAINERD_CA_FILE}"
  )
  ```

  ​`/etc/containerd/config.toml`

  ```toml
      [plugins.'io.containerd.cri.v1.images'.registry]
        config_path = '/etc/containerd/certs.d'
  ```

  创建目录，目录名就是 Harbor 地址，在该目录下创建 hosts.toml 文件

  ```bash
  HARBOR_IP="192.168.0.62"
  CONTAINERD_CERT_DIR="/etc/containerd/certs.d/${HARBOR_IP}"
  CONTAINERD_CA_FILE="${CONTAINERD_CERT_DIR}/ca.crt"

  (
  sudo mkdir -p "$CONTAINERD_CERT_DIR"
  sudo tee ${CONTAINERD_CERT_DIR}/hosts.toml > /dev/null <<EOF
  server = "https://${HARBOR_IP}"

  [host."https://${HARBOR_IP}"]
    capabilities = ["pull", "resolve", "push"]
    skip_verify = false
    ca = "${CONTAINERD_CA_FILE}"  # 指向你的证书路径
  EOF
  )
  ```

  重启 containerd

  ```bash
  sudo systemctl restart containerd
  ```

### 验证效果

> 每个使用 Harbor 仓库的服务器都要登陆

- Docker 登陆 Harbor 仓库

  ```bash
  sudo docker login 192.168.0.62 -u admin
  ```

## 配置代理缓存

配置代理缓存，将 Docker Hub 上的公共镜像缓存到私有 Harbor 仓库中，这样局域网内的其他机器就可以从内网 Harbor 快速拉取镜像

### 创建目标仓库

📌 位置：系统管理 → 仓库管理 → 新建目标

|提供者|目标名|目标URL|
| -----------------| --------------------------| ------------------------|
|Docker Registry|docker_hub_mirror_target|https://docker.1ms.run|

目标名任意即可，目标URL会自动设置

设置完成后可先 `测试连接`，无误后保存

### 创建代理缓存项目

📌 位置：项目 → 新建项目

|项目名称|镜像代理|
| -------------------| --------------------------|
|docker-hub-mirror|docker_hub_mirror_target|

目标名任意即可

## 测试效果

在 Harbor 容器宿主机，或局域网其他机器中拉取 Harbor 仓库中的镜像

### 方式一：Docker 客户端

```bash
sudo docker pull 192.168.0.62/docker-hub-mirror/library/nginx:1.29.6-alpine-slim
```

### 方式二：Kuboard 面板

> 此处在 default 名称空间中进行相关操作

- 添加镜像仓库密码

  📌 位置：名称空间 → 配置中心 → 密文

  |名称|类型|docker server|docker username|docker password|
  | ---------------| -----------------| ----------------------| -----------------| -----------------|
  |harbor-secret|Docker 仓库密码|https://192.168.0.62|admin|Harbor12345|

  此密文的名称可任意设置

- 创建示例工作负载

  📌 位置：名称空间 → 常用操作 → 创建工作负载

  - 基本信息

  |工作负载类型|工作负载名称|
  | --------------------| -------------------|
  |部署（Deployment）|nginx-from-harbor|

  - 容器信息

  添加工作容器

  |名称|镜像|ImagePullSecret|镜像拉取策略|
  | -------------------| -----------------------------------------------------------------| -----------------| --------------------------------------|
  |nginx-from-harbor|192.168.0.62/docker-hub-mirror/library/nginx:1.29.6-alpine-slim|harbor-secret|本地不存在时拉取镜像（IfNotPresent）|

  之后保存即可
- 效果

  ![2026-1-11_18-51-43.png](https://cdn.tsanfer.com/image/2026-1-11_18-51-43.png)
