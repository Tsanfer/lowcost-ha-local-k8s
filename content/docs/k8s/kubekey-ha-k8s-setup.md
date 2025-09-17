---
draft: false
title: '（附加）使用 KubeKey 部署高可用 K8s 集群'
weight: 4
---

{{< callout emoji="🔄" >}}
  此页内容待更新</br>
  计划用 Kuboard 代替 KubeSphere
{{< /callout >}}

通过 KubeKey 安装高可用 k8s 集群

- 全 Linux 系统
- 防火墙放行
- 可提前手动安装依赖（集群所有服务器）

  ```bash
  # 更新 apt 仓库
  sudo apt-get update -y
  # 安装 KubeSphere 依赖
  # sudo apt install socat conntrack
  sudo apt-get install -y openssl socat conntrack ipset ipvsadm ebtables chrony

  # # 关闭所有 swap 内存
  # swapoff -a
  # 删除原有 swap 文件
  # awk '/swap/ {print $1}' /etc/fstab | xargs rm 
  # 删除原有 swap 在 /etc/fstab 中的配置信息
  # sed -i '/swap/d' /etc/fstab
  ```

---

### 高可用 k8s 集群架构

![image.png](https://kubesphere.io/images/blogs/en/k8s-ha-practices/internalLoadBalancer.png)

控制节点一般为奇数，原因在于：leader 选举，要求可用节点数 > 总结点数/2

---

### master 服务器（lab-server-1）

初始化集群依赖环境：`./kk init os -y -f config-sample.yaml -y`

删除集群：`./kk delete cluster -f config-sample.yaml -y`

```bash
# KubeKey 最好只进行一次性使用，每次创建集群时，都重置一遍 KubeKey

# 下载 KubeKey

# while true; do
#   read -rp "是否启用 KubeSphere 国内安装加速? [Y/n] " input
#   case $input in
#   [yY]) # 使用国内加速安装
#     export KKZONE=cn
# 		curl -sfL https://get-kk.kubesphere.io | sh -    
# 		break
#     ;;

#   [nN]) # 使用正常安装
#     curl -sfL https://get-kk.kubesphere.io | sh -
#     break
#     ;;

#   *) echo "错误选项：$REPLY" ;;
#   esac
# done

if ping -c 3 google.com; then
		echo "##### Google 可以正常访问 #####"
else
		echo "##### Google 无法访问，启用国内加速 #####"
    ### 国内加速
    export KKZONE=cn
fi

curl -sfL https://get-kk.kubesphere.io | VERSION=v3.1.9 sh -

# 为 kk 添加可执行权限
chmod +x kk

# 显示支持的 k8s 版本
#./kk version --show-supported-k8s

# 主结点生成配置文件
#./kk create config --with-kubernetes
#./kk create config --with-kubernetes v1.24.7-k3s

./kk create config --with-kubernetes v1.33.0

# 编辑配置文件
# vim config-sample.yaml

while true; do
	echo "请配置 KubeKey 集群部署文件"
  read -rp "是否部署集群? [Y/n] " input
  case $input in
  [yY])
		# 生成集群
		export KKZONE=cn
		
		./kk delete cluster -f config-sample.yaml -y

		# Init operating system. This command will install openssl, socat, conntrack, ipset, ipvsadm, ebtables and chrony on all the nodes.
		# apt-get update -y
		# apt-get install -y openssl socat conntrack ipset ipvsadm ebtables chrony
		./kk init os -f config-sample.yaml
		
		# 若使用 --with-local-storage 指定 k8s 集群存储方式为本机存储，
		# 则每个节点都会独立存储容器镜像，可能会出现同一镜像拉取多次的情况
		./kk create cluster -f config-sample.yaml --with-local-storage -y
		# ./kk create cluster -f config-sample.yaml --with-local-storage --with-kubernetes v1.31.2 --container-manager containerd -y
    break
    
    # /etc/containerd/config.toml 文件不要禁止 cri 插件，正常情况下为：
    # disabled_plugins = []
    ;;

  [nN]) 
    break
    ;;

  *) echo "错误选项：$REPLY" ;;
  esac
done

```

- 方式二，通过 kubeadm 安装 k8s（本地）

使用本地负载均衡实现高可用（HAProxy）

`config-sample.yaml`

```yaml

apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: master1, address: 192.168.0.2, internalAddress: 192.168.0.2, user: root, password: "xxxx"}
  - {name: master2, address: 192.168.0.3, internalAddress: 192.168.0.3, user: root, password: "xxxx"}
  - {name: master3, address: 192.168.0.4, internalAddress: 192.168.0.4, user: root, password: "xxx"}
  - {name: worker1, address: 192.168.0.5, internalAddress: 192.168.0.5, user: root, password: "xxxx"}
  - {name: worker2, address: 192.168.0.6, internalAddress: 192.168.0.6, user: root, password: "xxxx"}
  - {name: worker3, address: 192.168.0.7, internalAddress: 192.168.0.7, user: root, password: "xxxx"}
  roleGroups:
    etcd:
    - master1
    - master2
    - master3
    control-plane: 
    - master1
    - master2
    - master3
    worker:
    - worker1
    - worker2
    - worker3
  controlPlaneEndpoint:
    ## Internal loadbalancer for apiservers 
    internalLoadbalancer: haproxy

    domain: lb.kubesphere.local
    address: ""
    port: 6443
  kubernetes:
    version: v1.33.0
    clusterName: cluster.local
    autoRenewCerts: true
    containerManager: containerd
  etcd:
    type: kubekey
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    # 10.233.01/000000.0
    kubeServiceCIDR: 10.233.0.0/18
    # 10.233.00/000000.0
    ## multus support. https://github.com/k8snetworkplumbingwg/multus-cni
    multusCNI:
      enabled: false
  registry:
    privateRegistry: ""
    namespaceOverride: ""
    registryMirrors: [
      "https://docker.1panel.live",
    #   "https://hub1.nat.tf",
    #   "https://dockerproxy.net"
    ]
    insecureRegistries: []
  addons: []

```

`config-sample.yaml` （云集群）

```yaml
apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: master, address: 107.175.83.132, internalAddress: 107.175.83.132, user: root, password: "xxxxxx"}
  - {name: worker1, address: 104.168.43.39, internalAddress: 104.168.43.39, user: root, password: "xxxxx"}
  - {name: worker2, address: 148.135.67.22, internalAddress: 148.135.67.22, user: root, password: "xxxx"}
  - {name: worker3, address: 23.94.223.197, internalAddress: 23.94.223.197, user: root, password: "xxxxx"}
  roleGroups:
    etcd:
    - master
    control-plane: 
    - master
    worker:
    - worker1
    - worker2
    - worker3
  controlPlaneEndpoint:
    ## Internal loadbalancer for apiservers 
    # internalLoadbalancer: haproxy

    domain: lb.kubesphere.local
    address: ""
    port: 6443
  kubernetes:
    version: v1.33.0
    clusterName: cluster.local
    autoRenewCerts: true
    containerManager: containerd
  etcd:
    type: kubekey
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
    ## multus support. https://github.com/k8snetworkplumbingwg/multus-cni
    multusCNI:
      enabled: false
  registry:
    privateRegistry: ""
    namespaceOverride: ""
    registryMirrors: []
    insecureRegistries: []
  addons: []

```

### 启用 kubectl 自动补全

```bash
# 将 completion 脚本添加到你的 ~/.zshhrc 文件
echo 'source <(kubectl completion zsh)' >>~/.zshrc

source ~/.zshrc

```

---