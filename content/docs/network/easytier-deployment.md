---
draft: false
title: '去中心化异地组网工具 EasyTier 部署'
weight: 2
---

EasyTier 官网：[EasyTier - 简单、安全、去中心化的异地组网方案](https://easytier.cn/)

EasyTier 为 P2P 组网方式，不区分服务端和客户端

每个局域网中只需一台设备部署 EasyTier 作为代理网关，就可使整个局域网中的所有设备都具有访问异地局域网的能力，无需每台机器都安装代理软件

## 云端服节点部署

在云端部署 EasyTier 节点，是为了让任意两个位于各自 NAT 后面的局域网设备，都能随时、自动、稳定地相互连接。

其作用之一是：帮助完成打洞的第一步。云节点有固定公网 IP 和端口，可充当中介让所有节点交换各自的外网地址和端口，帮助打洞

另外一个作用是：充当中继节点转发流量。当节点间打洞失败，云节点立刻自动降级为中继节点，流量先走云再转发，保证链路可达。

- 创建目录及配置文件

创建项目文件夹

```bash
mkdir -p ~/easytier
cd ~/easytier

```

容器编排文件：`~/easytier/docker-compose.yml`​

```yaml
services:
  easytier:
    image: easytier/easytier:v2.3.1
    hostname: easytier
    container_name: easytier
    restart: unless-stopped
    # network_mode: host
    ports:
      - 11010:11010 # tcp
      - 11010:11010/udp # udp
      - 11011:11011/udp # wg
      - 11011:11011 # ws
      - 11012:11012 # wss
    #   - 11013:11013 # ipv6 tcp
    #   - 11013:11013/udp # ipv6 udp
    cap_add:
      - NET_ADMIN
      - NET_RAW
    environment:
      - TZ=Asia/Shanghai
    devices:
      - /dev/net/tun:/dev/net/tun
    volumes:
      - ${EASYTIER_PATH}:/root
    #   - /etc/machine-id:/etc/machine-id:ro # 映射宿主机机器码
    # command: -d --network-name ${NET_NAME} --network-secret ${NET_PASS} --enable-kcp-proxy --use-smoltcp
    # command: --hostname easytier-relay -d --network-name ${NET_NAME} --network-secret ${NET_PASS}
    command: |
      --hostname=easytier-relay
      --network-name=${NET_NAME}
      --network-secret=${NET_PASS}
      --dhcp
      --relay-network-whitelist
    # 不转发其他虚拟网的网络包，而是只帮助他们建立 P2P 链接
    # --relay-network-whitelist --relay-all-peer-rpc
    # --config-server k8s-local
    # --proxy-forward-by-system
    # -l 'tcp://[::]:11013' -l 'udp://[::]:11013'

#   easytier_WEB:
#     image: lmq8267/easytier:v2.3.1
#     restart: always
#     volumes:
#     - ${EASYTIER_WEB_PATH}/:/opt/easytier-web
#     ports:
#     - '22020:22020/udp' # --config-server-port
#     - '11211:11211/tcp' # --api-server-port
#     - '11210:11210/tcp' # web-http
#     environment:
#     - WEB=1
#     - TZ=Asia/Shanghai
#     - LANG=zh_CN
#     container_name: Easytier-web
#     command: '--console-log-level info -d /opt/easytier-web/et.db -c 22020 -a 11211 -l 11210'

```

容器编排环境变量文件：`~/easytier/.env`​

```bash
EASYTIER_PATH=/root/easytier
NET_NAME=net-name
NET_PASS=password

```

- 启动容器

```bash
# cd ~/easytier

docker compose up -d

```

容器开始运行日志如下：

![2025-7-28_23-21-55_f77kgmkgmn.png](https://cdn.tsanfer.com/image/2025-7-28_23-21-55_f77kgmkgmn.png "EasyTier 云端节点开始运行")

- 重启容器

```bash
# cd ~/easytier

docker compose restart

:<<COMMENT
cd ~/easytier && \
  docker compose rm -sf && \
  docker compose up
COMMENT

```

- 如想删除容器编排，可执行如下命令

```bash
# cd ~/easytier

docker compose rm -sf

```

## 连接网络

### Docker Compose 方式

- 创建项目文件夹

```bash
mkdir -p ~/easytier
cd ~/easytier

:<<EOF
mkdir -p /opt/1panel/docker/compose/easytier
cd /opt/1panel/docker/compose/easytier
EOF

```

容器编排文件位置：`~/easytier/docker-compose.yml`​

```yaml
services:
  easytier:
    image: easytier/easytier:v2.3.1
    hostname: easytier
    container_name: easytier
    restart: unless-stopped
    network_mode: host
    # ports:
    #   - 11010:11010 # tcp
    #   - 11010:11010/udp # udp
    #   - 11011:11011/udp # wg
    #   - 11011:11011 # ws
    #   - 11012:11012 # wss
    #   # - 11013:11013 # web
    #   - 11014:11014 # ipv6 tcp
    #   - 11014:11014/udp # ipv6 udp
    cap_add:
      - NET_ADMIN
      - NET_RAW
    environment:
      - TZ=Asia/Shanghai
    devices:
      - /dev/net/tun:/dev/net/tun
    volumes:
      - ${EASYTIER_PATH}:/root
      - /etc/machine-id:/etc/machine-id:ro # 映射宿主机机器码
    command:
    # --config-file ${CONFIG_FILE}
    - --network-name=${NET_NAME}
    - --network-secret=${NET_PASS}
    - --dhcp
    - --listeners=
        tcp://0.0.0.0:11010,
        udp://0.0.0.0:11010,
        wg://0.0.0.0:11011,
        ws://0.0.0.0:11011,
        wss://0.0.0.0:11012,
        tcp://[::]:11014,
        udp://[::]:11014
    - --peers=tcp://${PEER}:11010
    - --proxy-networks=${PROXY_NETWORKS}
    # - --config-server=udp://${EASYTIER_WEB_SERVER}:22020/temp
    - --accept-dns=true
    # - --bind-device=true
    - --multi-thread=true
    # - --enable-kcp-proxy=true
    - --use-smoltcp=true # 开启以让 docker 容器外局域网访问 easytier
    # 不转发其他虚拟网的网络包，而是只帮助他们建立 P2P 链接
    # - --relay-network-whitelist=true
    - --relay-all-peer-rpc=true
    # --config-server ${NET_NAME}
    # --proxy-forward-by-system
    # -l 'tcp://[::]:11013' -l 'udp://[::]:11013'

```

容器编排环境变量文件：`~/easytier/.env`​

```bash
EASYTIER_PATH=/root/easytier
NET_NAME=net-name
NET_PASS=password
PEER=peer.easytier.com
PROXY_NETWORKS=192.168.0.0/24
```

- 启动容器

```bash
# cd ~/easytier

docker compose up -d

```

应用运行日志如下：

![2025-7-28_23-28-17_ynu9q4e0tr.png](https://cdn.tsanfer.com/image/2025-7-28_23-28-17_ynu9q4e0tr.png "EasyTier 节点加入网络")

- 重启容器

```bash
# cd ~/easytier

docker compose restart

:<<COMMENT
cd ~/easytier && \
  docker compose rm -sf && \
  docker compose up
COMMENT

```

若局域网其他设备无法连接 EasyTier，则检查局域网 EasyTier 网关的防火墙是否放行。

若还不行，则试着配置 EasyTier 网关的路由转发规则：

```bash
# 安装iptables-persistent，它会在系统启动时，加载 iptables 规则
sudo apt -y update && sudo apt -y install iptables-persistent

# 把 FORWARD 设为 ACCEPT
sudo iptables --policy FORWARD ACCEPT

# 保存当前规则（IPv4）
sudo netfilter-persistent save
# sudo sh -c 'iptables-save > /etc/iptables/rules.v4'
# 保存当前规则（IPv6）
# sudo sh -c 'iptables-save > /etc/iptables/rules.v6'

# 开机自动加载（安装包已默认启用）
sudo systemctl enable netfilter-persistent

# docker 可能会覆盖 iptables 的设置，需要让本规则在 docker 启动完毕后生效
# 确保目录存在
mkdir -p /etc/systemd/system/netfilter-persistent.service.d
# 写入 override 片段
cat >/etc/systemd/system/netfilter-persistent.service.d/after-docker.conf <<'EOF'
[Unit]
After=docker.service
Wants=docker.service
EOF

# 使服务的修改立即生效
sudo systemctl daemon-reload

# 可在重启后检查结果
# sudo iptables -L -v | grep "FORWARD"

# Docker 容器内部修改 iptables
# docker exec -it easytier sh -c "apk add iptables"
# docker exec -it easytier sh -c "iptables --policy FORWARD ACCEPT"

```

如想删除容器编排，可执行如下命令

```bash
# cd ~/easytier

docker compose rm -sf

```

### OpenWrt 方式

- 查看 OpenWrt 的 CPU 指令集架构

通过官方网站在线查找：

[[OpenWrt Wiki] 硬件表格: 软件包下载](https://openwrt.org/zh/toh/views/toh_packagedownload)

或者通过命令查看：

```bash
# 查看系统版本
cat /etc/openwrt_release

# 示例输出
# DISTRIB_ID='OpenWrt'
# DISTRIB_RELEASE='24.10.0'
# DISTRIB_REVISION='r28427-6df0e3d02a'
# DISTRIB_TARGET='ramips/mt7621'
# DISTRIB_ARCH='mipsel_24kc' # 此行即为系统架构信息
# DISTRIB_DESCRIPTION='OpenWrt 24.10.0 r28427-6df0e3d02a'
# DISTRIB_TAINTS=''

# 查看是否支持 fpu
grep "fpu" /proc/cpuinfo
# 
```

- 运行方式一：通过 LuCI 图形化界面运行 EasyTier

  - 下载对应版本的 EasyTier 的安装包：

  参考链接：[EasyTier/luci-app-easytier: OpenWrt里的EasyTier安装包（IPK和APK）](https://github.com/EasyTier/luci-app-easytier)

  可在相应 Github 仓库的 Actions 中下载编译好的文件，比如：`EasyTier-mipsel_24kc-openwrt-22.03`​，或者还可以 fork 仓库到自己的账号下，修改相关配置后，手动执行 github action 构建自定义的安装包

  解压下载的压缩包，上传其中的文件，比如：

  EasyTier 安装包：`easytier_2.4.0_mipsel_24kc.ipk`​

  LuCI 图形界面插件安装包：`luci-app-easytier_2.4.0_all.ipk`​

  LuCI 图形界面插件中文语言包：`luci-i18n-easytier-zh-cn_git-25.207.01220-8d8cdd3_all`​

  - 安装依赖及 EasyTier：

  ```bash
  EASYTIER_VERSION="2.4.0_mipsel_24kc"
  EASYTIER_LUCI_VERSION="2.4.0"
  EASYTIER_LUCI_ZH_CN_VERSION="git-25.207.01220-8d8cdd3"

  opkg update
  # 安装依赖
  opkg install kmod-tun
  # 将 easytier 上传到 openwrt
  opkg install ./easytier_${EASYTIER_VERSION}.ipk
  opkg install ./luci-app-easytier_${EASYTIER_LUCI_VERSION}_all.ipk
  opkg install ./luci-i18n-easytier-zh-cn_${EASYTIER_LUCI_ZH_CN_VERSION}_all.ipk

  #卸载
  #EASYTIER_VERSION="2.3.1"
  #opkg remove luci-app-easytier_${EASYTIER_VERSION}_all.ipk

  #更新版本需要先卸载再安装新的ipk然后去管理界面关闭插件 修改参数后重新点击应用并保存
  #安装后openwrt管理界面里不显示easytier 请注销登录或关闭窗口重新打开  
  ```

  重新登陆 LuCI 面板，然后可在面板中配置 EasyTier

  > 注意 OpenWrt 版的 EasyTier 的 RPC 端口为15888
  >

  可以通过官方的配置文件生成工具，生成配置文件：[EasyTier Dashboard](https://easytier.cn/web/index.html#/config_generator "EasyTier Dashboard")

  以下为参考配置（启动方式为配置文件）：

  ```yaml
  instance_name = "openwrt" # 设备名称
  instance_id = "f9f05421-7c6b-47d6-a67d-4d58d49f51d8" # 设备 ID
  dhcp = true
  listeners = [
      "tcp://0.0.0.0:11010",
      "udp://0.0.0.0:11010",
      "wg://0.0.0.0:11011",
      "ws://0.0.0.0:11011",
      "wss://0.0.0.0:11012",
      "tcp://[::]:11014",
      "udp://[::]:11014",
  ]

  rpc_portal = "0.0.0.0:15888" # OpenWrt 环境差异配置

  [network_identity]
  network_name = "net-name" # EasyTier 网络名
  network_secret = "password" # EasyTier 网络密码

  [[peer]]
  uri = "tcp://peer.example.com:11010" # 对等节点地址

  [[proxy_network]]
  cidr = "192.168.0.0/24" # 子网代理范围

  [flags]
  accept_dns = true
  enable_magic_dns = true
  enable_kcp_proxy = true
  bind-device = true
  multi-thread = true

  ```

  运行成功后，可在 EasyTier 配置页面的连接信息中查看对等节点信息

  ![2025-7-28_23-14-14_k6qddcizka.png](https://cdn.tsanfer.com/image/2025-7-28_23-14-14_k6qddcizka.png "OpenWrt 运行 EasyTier 异地组网工具")

- 运行方式二（仅参考）：直接通过二进制文件运行

下载 EasyTier 可执行文件

```bash
cd

opkg install kmod-tun

EASYTIER_VERSION="2.3.1"
CPU_ARCH="mipsel"
# GITHUB_RELEASE_ENDPOINT="https://github.com"
GITHUB_RELEASE_ENDPOINT="https://get-github.hexj.org/download"

curl -L --progress-bar \
  ${GITHUB_RELEASE_ENDPOINT}/EasyTier/EasyTier/releases/download/v${EASYTIER_VERSION}/easytier-linux-${CPU_ARCH}-v${EASYTIER_VERSION}.zip \
  -o ~/easytier.zip
unzip ~/easytier.zip
cd easytier-linux-${CPU_ARCH}

```

命令行运行

```bash
easytier-core \
  --network-name net-name \
  --network-secret password \
  --dhcp \
  --listeners=\
tcp://0.0.0.0:11010,\
udp://0.0.0.0:11010,\
wg://0.0.0.0:11011,\
ws://0.0.0.0:11011,\
wss://0.0.0.0:11012,\
tcp://[::]:11014,\
udp://[::]:11014\
  --peers=tcp://peer.example.com:11010 \
  --proxy-networks=192.168.0.0/24 \
  --accept-dns=true \
  --bind-device=true \
  --multi-thread=true \
  --enable-kcp-proxy=true
  # --config-server=udp://peer.example.com:22020/temp \
  # --use-smoltcp=true
  # --proxy-forward-by-system=true
  # enable_magic_dns=true
```

## 静态路由配置

EasyTier 通过子网代理连通本地局域网与异地局域网

若 EasyTier 部署于路由器等默认路由网关设备中，则无需配置静态路由（端口转发）

若 EasyTier 部署于局域网默认网关之外的设备，则需要在默认网关中配置静态路由（端口转发），以确保局域网中所有设备访问异地局域网的流量流向 EasyTier 代理网关。以下为示例配置，其中 `192.168.5.2`​ 为 EasyTier 代理网关的局域网 IP 地址，`192.168.0.0`​ 为异地局域网网段：

- 名称：easytier-lab  
  接口：LAN  
  静态路由地址：192.168.5.2  
  子网掩码：255.255.255.0  
  网关：192.168.0.0

配置界面如图所示：

![2025-7-28_23-45-11_g3snjzxtdb.png](https://cdn.tsanfer.com/image/2025-7-28_23-45-11_g3snjzxtdb.png "路由器管理界面配置静态路由")

## 测试连通性

异地组网完成后，可通过测试异地设备间的延迟来判断网络间的连通性。这里选取未安装 EasyTier 代理网关的设备进行测试。

先使用 192.168.0.0 网段的设备 ping 192.168.5.0 网段的设备

![2025-7-31_15-31-47_e6e3xvqtt8.png](https://cdn.tsanfer.com/image/2025-7-31_15-31-47_e6e3xvqtt8.png "192.168.0.0 网段设备 ping 192.168.5.0 网段设备")​​​

然后再使用 192.168.5.0 网段的设备 ping 192.168.0.0 网段的设备

![2025-7-31_15-34-10_o5nv458o0h.png](https://cdn.tsanfer.com/image/2025-7-31_15-34-10_o5nv458o0h.png "192.168.5.0 网段设备 ping 192.168.0.0 网段设备")​​​

若两端可互相 ping 通，即表明异地组网成功。这里测得数毫秒的端到端延迟，为打洞成功后的链路延迟。上述所有用于连通性测试的设备均未安装 EasyTier。

‍
