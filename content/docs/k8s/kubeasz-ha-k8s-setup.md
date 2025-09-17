---
draft: false
title: '使用 kubeasz 安装高可用 K8s 集群'
weight: 1
---

参考链接：[GitHub - easzlab/kubeasz: 使用Ansible脚本安装K8S集群，介绍组件交互原理，方便直接，不受国内网络环境影响](https://github.com/easzlab/kubeasz)

> 可在以控制节点1作为部署节点，进行以下操作

> 以下操作均需在 root 用户下执行，目标主机也要设置 root 用户的 SSH 登陆

如果 Debian 系统中未安装 iptables，则运行命令安装

```bash
apt -y install iptables
```

## 设置 SSH 免密登陆

生成公私钥对：

```bash
ssh-keygen -t rsa
```

复制公钥到受控主机：

```bash
ssh-copy-id -o StrictHostKeyChecking=no root@192.168.0.31
ssh-copy-id -o StrictHostKeyChecking=no root@192.168.0.32
ssh-copy-id -o StrictHostKeyChecking=no root@192.168.0.33
ssh-copy-id -o StrictHostKeyChecking=no root@192.168.0.34
ssh-copy-id -o StrictHostKeyChecking=no root@192.168.0.35
ssh-copy-id -o StrictHostKeyChecking=no root@192.168.0.36
```

## 部署节点安装 Docker

> 只有部署节点需要安装 Docker

```bash
bash <(curl -sSL https://linuxmirrors.cn/docker.sh)
```

## 在部署节点编排 K8s 安装

### 下载源码、二进制、离线镜像

```bash
# 下载
KUBEASZ_VERSION="3.6.7"
GITHUB="gh-proxy.com/github.com"

wget https://${GITHUB}/easzlab/kubeasz/releases/download/${KUBEASZ_VERSION}/ezdown

chmod +x ./ezdown
```

修改脚本中的 Docker 二进制下载地址，防止下载失败

```bash
sed -i '/DOCKER_URL/s/mirrors.tuna.tsinghua.edu.cn/mirrors.aliyun.com/' ezdown
```

安装

```bash
## 安装
# 国内环境
sudo ./ezdown -D
# 海外环境
# sudo ./ezdown -D -m standard
```

脚本运行成功后，所有文件（kubeasz代码、二进制、离线镜像）都放在 `/etc/kubeasz` 目录下

### 创建集群配置实例

```bash
# 容器化运行kubeasz
./ezdown -S

# 创建新集群 k8s-local
docker exec -it kubeasz ezctl new k8s-local
```

然后根据提示配置 `/etc/kubeasz/clusters/k8s-local/hosts` 和 `/etc/kubeasz/clusters/k8s-local/config.yml`。依据先前的节点规划修改 hosts 文件中的主机信息，以确保各节点配置准确无误；而集群层面的主要配置选项以及其他集群组件的设置，则可在 config.yml 文件中进行调整。以下为示例配置：

文件位置：`/etc/kubeasz/clusters/k8s-local/hosts`​

```ini
# 'etcd' cluster should have odd member(s) (1,3,5,...)
[etcd]
192.168.0.31
192.168.0.32
192.168.0.33
192.168.0.34
192.168.0.35
192.168.0.36

# master node(s), set unique 'k8s_nodename' for each node
# CAUTION: 'k8s_nodename' must consist of lower case alphanumeric characters, '-' or '.',
# and must start and end with an alphanumeric character
[kube_master]
192.168.0.31 k8s_nodename='k8s-control-1'
192.168.0.32 k8s_nodename='k8s-control-2'
192.168.0.33 k8s_nodename='k8s-control-3'

# work node(s), set unique 'k8s_nodename' for each node
# CAUTION: 'k8s_nodename' must consist of lower case alphanumeric characters, '-' or '.',
# and must start and end with an alphanumeric character
[kube_node]
192.168.0.34 k8s_nodename='k8s-worker-1'
192.168.0.35 k8s_nodename='k8s-worker-2'
192.168.0.36 k8s_nodename='k8s-worker-3'

...

```

文件位置：`/etc/kubeasz/clusters/k8s-local/config.yml`​

```yaml
...

# kubeconfig 配置参数
CLUSTER_NAME: "k8s-local"
CONTEXT_NAME: "context-{{ CLUSTER_NAME }}"

...
```

### 分步骤执行集群安装脚本

- 添加安装命令别名

用 `dk` 别名替换 `docker exec -it kubeasz` 命令

```bash
# 检测当前使用的 Shell 类型并设置对应的配置文件
case "$SHELL" in
    */bash)
        config_file="$HOME/.bashrc"
        ;;
    */zsh)
        config_file="$HOME/.zshrc"
        ;;
    */fish)
        config_file="$HOME/.config/fish/config.fish"
        ;;
    *)
        echo "Unsupported shell: $SHELL"
        exit 1
        ;;
esac

# 检查配置文件是否存在，如果不存在则创建
if [ ! -f "$config_file" ]; then
    echo "Config file $config_file does not exist. Creating it..."
    mkdir -p "$(dirname "$config_file")"
    touch "$config_file"
fi

# 检查别名是否已经存在
if grep -qF -- 'alias dk="docker exec -it kubeasz"' "$config_file"; then
    echo "Alias already exists in $config_file."
else
    # 添加别名到配置文件
    echo 'alias dk="docker exec -it kubeasz"' >> "$config_file"
    echo "Alias added to $config_file."
fi

# 重新加载配置文件
source "$config_file"

# 提示用户别名已生效
echo "Alias dk is now available."
```

- 分部安装 K8s 集群

查看帮助

```bash
dk ezctl help setup
```

01：环境准备

```bash
dk ezctl setup k8s-local prepare
# dk ezctl setup k8s-local 01
```

02：安装 etcd 集群

```bash
dk ezctl setup k8s-local etcd
# dk ezctl setup k8s-local 02
```

03：安装容器运行时

```bash
dk ezctl setup k8s-local container-runtime
# dk ezctl setup k8s-local 03
```

04：安装控制节点

```bash
dk ezctl setup k8s-local kube-master
# dk ezctl setup k8s-local 04
```

05：安装工作节点

```bash
dk ezctl setup k8s-local kube-node
# dk ezctl setup k8s-local 05
```

06：安装网络插件

```bash
dk ezctl setup k8s-local network
# dk ezctl setup k8s-local 06
```

07：安装其他常用插件

```bash
dk ezctl setup k8s-local cluster-addon
# dk ezctl setup k8s-local 07
```

熟悉之后可以使用 `dk ezctl setup k8s-local all` 一键安装以上所有部件，笔者实测一键安装所有组件的总耗时为 10 ~ 15 分钟左右。

### 验证安装效果

安装完成后，重启终端，此时可在安装节点中执行 kubectl 命令

- 查看所有节点

  ```bash
  kubectl get nodes -A
  ```

  🖥️ 示例输出：

  ```plaintext
  NAME            STATUS                     ROLES    AGE   VERSION
  k8s-control-1   Ready,SchedulingDisabled   master   15h   v1.33.1
  k8s-control-2   Ready,SchedulingDisabled   master   15h   v1.33.1
  k8s-control-3   Ready,SchedulingDisabled   master   15h   v1.33.1
  k8s-worker-1    Ready                      node     15h   v1.33.1
  k8s-worker-2    Ready                      node     15h   v1.33.1
  k8s-worker-3    Ready                      node     15h   v1.33.1
  ```

- 查看所有命名空间

  ```bash
  kubectl get namespaces -A
  ```

  🖥️ 示例输出：

  ```plaintext
  NAME              STATUS   AGE
  default           Active   15h
  kube-node-lease   Active   15h
  kube-public       Active   15h
  kube-system       Active   15h
  ```

- 查看集群所有 Pod

  ```bash
  kubectl get pods -A
  ```

  🖥️ 示例输出：

  ```plaintext
  NAMESPACE     NAME                                       READY   STATUS    RESTARTS      AGE
  kube-system   calico-kube-controllers-647ddc7bfd-w452z   1/1     Running   0             15h
  kube-system   calico-node-4c6rd                          1/1     Running   0             15h
  kube-system   calico-node-8xqvg                          1/1     Running   0             15h
  kube-system   calico-node-bgmtd                          1/1     Running   1 (15h ago)   15h
  kube-system   calico-node-dns9n                          1/1     Running   0             15h
  kube-system   calico-node-sxllh                          1/1     Running   0             15h
  kube-system   calico-node-x9fwp                          1/1     Running   0             15h
  kube-system   coredns-5c4d969fb-nq74r                    1/1     Running   0             15h
  kube-system   metrics-server-74f6d6fdd5-hggs4            1/1     Running   0             15h
  kube-system   node-local-dns-8lvnr                       1/1     Running   0             15h
  kube-system   node-local-dns-97tc8                       1/1     Running   0             15h
  kube-system   node-local-dns-pjcdz                       1/1     Running   0             15h
  kube-system   node-local-dns-ptvph                       1/1     Running   0             15h
  kube-system   node-local-dns-vdncg                       1/1     Running   0             15h
  kube-system   node-local-dns-zgcxx                       1/1     Running   0             15h
  ```

- 查看所有服务

  ```bash
  kubectl get services -A
  ```

  🖥️ 示例输出：

  ```plaintext
  NAMESPACE     NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
  default       kubernetes          ClusterIP   10.68.0.1       <none>        443/TCP                  16h
  kube-system   kube-dns            ClusterIP   10.68.0.2       <none>        53/UDP,53/TCP,9153/TCP   16h
  kube-system   kube-dns-upstream   ClusterIP   10.68.40.13     <none>        53/UDP,53/TCP            16h
  kube-system   metrics-server      ClusterIP   10.68.215.155   <none>        443/TCP                  16h
  kube-system   node-local-dns      ClusterIP   None            <none>        9253/TCP                 16h
  ```

- 查看所有部署（无状态）

  ```bash
  kubectl get deployments -A
  ```

  🖥️  示例输出：

  ```plaintext
  kube-system   calico-kube-controllers   1/1     1            1           16h
  kube-system   coredns                   1/1     1            1           16h
  kube-system   metrics-server            1/1     1            1           16h
  ```

- 查看所有有状态集

  ```bash
  kubectl get statefulsets -A
  ```

- 查看所有守护进程集

  ```bash
  kubectl get daemonsets -A
  ```

  🖥️ 示例输出：

  ```plaintext
  NAMESPACE     NAME             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
  kube-system   calico-node      6         6         6       6            6           kubernetes.io/os=linux   16h
  kube-system   node-local-dns   6         6         6       6            6           <none>                   16h
  ```

- 查看所有工作

  ```bash
  kubectl get jobs -A
  ```

- 查看所有定时任务

  ```bash
  kubectl get cronjobs -A
  ```

- 查看所有入口

  ```bash
  kubectl get ingress -A
  ```

- 查看所有资源类型

  ```bash
  kubectl api-resources
  ```

  🖥️ 示例输出：

  ```plaintext
  NAME                                SHORTNAMES   APIVERSION                        NAMESPACED   KIND
  bindings                                         v1                                true         Binding
  componentstatuses                   cs           v1                                false        ComponentStatus

  ...

  storageclasses                      sc           storage.k8s.io/v1                 false        StorageClass
  volumeattachments                                storage.k8s.io/v1                 false        VolumeAttachment
  ```

- 查看所有资源

  ```bash
  kubectl get all -A
  ```

  🖥️ 示例输出：

  ```plaintext
  NAMESPACE     NAME                                           READY   STATUS    RESTARTS      AGE
  kube-system   pod/calico-kube-controllers-647ddc7bfd-w452z   1/1     Running   0             15h
  kube-system   pod/calico-node-4c6rd                          1/1     Running   0             15h
  kube-system   pod/calico-node-8xqvg                          1/1     Running   0             15h
  kube-system   pod/calico-node-bgmtd                          1/1     Running   1 (15h ago)   15h
  kube-system   pod/calico-node-dns9n                          1/1     Running   0             15h
  kube-system   pod/calico-node-sxllh                          1/1     Running   0             15h
  kube-system   pod/calico-node-x9fwp                          1/1     Running   0             15h
  kube-system   pod/coredns-5c4d969fb-nq74r                    1/1     Running   0             15h
  kube-system   pod/metrics-server-74f6d6fdd5-hggs4            1/1     Running   0             15h
  kube-system   pod/node-local-dns-8lvnr                       1/1     Running   0             15h
  kube-system   pod/node-local-dns-97tc8                       1/1     Running   0             15h
  kube-system   pod/node-local-dns-pjcdz                       1/1     Running   0             15h
  kube-system   pod/node-local-dns-ptvph                       1/1     Running   0             15h
  kube-system   pod/node-local-dns-vdncg                       1/1     Running   0             15h
  kube-system   pod/node-local-dns-zgcxx                       1/1     Running   0             15h

  NAMESPACE     NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
  default       service/kubernetes          ClusterIP   10.68.0.1       <none>        443/TCP                  15h
  kube-system   service/kube-dns            ClusterIP   10.68.0.2       <none>        53/UDP,53/TCP,9153/TCP   15h
  kube-system   service/kube-dns-upstream   ClusterIP   10.68.40.13     <none>        53/UDP,53/TCP            15h
  kube-system   service/metrics-server      ClusterIP   10.68.215.155   <none>        443/TCP                  15h
  kube-system   service/node-local-dns      ClusterIP   None            <none>        9253/TCP                 15h

  NAMESPACE     NAME                            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
  kube-system   daemonset.apps/calico-node      6         6         6       6            6           kubernetes.io/os=linux   15h
  kube-system   daemonset.apps/node-local-dns   6         6         6       6            6           <none>                   15h

  NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
  kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           15h
  kube-system   deployment.apps/coredns                   1/1     1            1           15h
  kube-system   deployment.apps/metrics-server            1/1     1            1           15h

  NAMESPACE     NAME                                                 DESIRED   CURRENT   READY   AGE
  kube-system   replicaset.apps/calico-kube-controllers-647ddc7bfd   1         1         1       15h
  kube-system   replicaset.apps/coredns-5c4d969fb                    1         1         1       15h
  kube-system   replicaset.apps/metrics-server-74f6d6fdd5            1         1         1       15h
  ```

- 查看某个 pod 的描述（需指定命名空间）

```bash
kubectl describe pods -n kube-system coredns-5c4d969fb-nq74r
```

🖥️  示例输出：

```plaintext
Name:                 coredns-5c4d969fb-nq74r
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Service Account:      coredns
Node:                 k8s-worker-1/192.168.0.34
Start Time:           Sat, 13 Sep 2025 21:23:21 +0800
Labels:               k8s-app=kube-dns
                      pod-template-hash=5c4d969fb
Annotations:          <none>
Status:               Running
SeccompProfile:       RuntimeDefault
IP:                   172.20.230.1
IPs:
  IP:           172.20.230.1
Controlled By:  ReplicaSet/coredns-5c4d969fb
Containers:
  coredns:
    Container ID:  containerd://4a01448307275dd39539c4b8c10057183eb6aab3681df36685b588ad2911c0b2
    Image:         easzlab.io.local:5000/easzlab/coredns:1.12.1
    Image ID:      easzlab.io.local:5000/easzlab/coredns@sha256:4f7a57135719628cf2070c5e3cbde64b013e90d4c560c5ecbf14004181f91998
    Ports:         53/UDP, 53/TCP, 9153/TCP
    Host Ports:    0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile
    State:          Running
      Started:      Sat, 13 Sep 2025 21:23:38 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      memory:  500Mi
    Requests:
      cpu:        100m
      memory:     70Mi
    Liveness:     http-get http://:8080/health delay=60s timeout=5s period=10s #success=1 #failure=5
    Readiness:    http-get http://:8181/ready delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /etc/coredns from config-volume (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-696jn (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      coredns
    Optional:  false
  kube-api-access-696jn:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 CriticalAddonsOnly op=Exists
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```

- 查看 kubeconfig 集群访问文件

```bash
kubectl config view --raw
```

🖥️ 示例输出：

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz
    server: https://192.168.0.31:6443
  name: k8s-local
contexts:
- context:
    cluster: k8s-local
    user: admin
  name: context-k8s-local
current-context: context-k8s-local
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate-data: bcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcd
    client-key-data: cdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklm
```
