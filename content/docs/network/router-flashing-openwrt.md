---
draft: false
title: '路由器刷 OpenWrt'
weight: 1
---

此处使用的路由器设备为：小米路由器3G，故此刷机过程适用于小米路由器，其他品牌路由器刷机也可参考此过程。

参考链接：

[小米路由器R3G刷（openwrt/原厂）保姆级教程_路由器_什么值得买](https://post.smzdm.com/p/a6plozqe/)

[【路由刷机教程】适用于小米路由器刷机工具](https://web.vip.miui.com/page/info/mio/mio/detail?postId=19134127)

---

## 刷入开发版路由器系统

下载链接：[MiWiFi – 下载](https://www1.miwifi.com/miwifi_download.html)

在小米路由器官网下载页面的 `ROM` 选项卡中下载对应路由器型号的固件，此处下载 `小米路由器3G 开发版` 固件，其文件名为：`miwifi_r3g_firmware_12f97_2.25.124.bin` 。

通过浏览器打开路由器后台，默认地址为：`192.168.31.1`​

📌 位置：常用设置 → 系统状态 → 手动升级

选择下载好的开发版固件上传

升级过程大约需要几分钟，路由器指示灯重新变蓝后，可以通过 192.168.31.1 进入管理后台。此时在管理网页中会提示系统 ROM 版本为：MiWiFi 开发版

![2025-7-28_19-40-47_7qauasnt1p.jpg](https://cdn.tsanfer.com/image/2025-7-28_19-40-47_7qauasnt1p.jpg "路由器正常运行时蓝灯常亮")

## 解锁 SSH

确保路由器已联网后，用手机连上它的 WiFi。接着去手机应用商店搜索并安装 `小米WiFi` App，打开 App 按提示绑定路由器

MiWiFi 开启 SSH 工具链接：[MiWiFi SSH](https://d.miwifi.com/rom/ssh)

进入 MiWiFi 开启 SSH 工具网站，查看绑定的路由器的 root 密码。在此网页中下载工具包，其文件名为： `miwifi_ssh.bin` 。

然后安装 SSH 解锁，具体步骤参考官方教程：

> 1. 请将下载的工具包bin文件复制到U盘（FAT/FAT32格式）的根目录下，保证文件名为miwifi_ssh.bin；
> 2. 断开小米路由器的电源，将U盘插入USB接口；
> 3. 按住reset按钮之后重新接入电源，等待十几秒种的时间后，指示灯变为黄色闪烁状态即可松开reset键；
> 4. 等待3-5秒后安装完成之后，小米路由器会自动重启，之后您就可以尽情折腾啦 ：）

![2025-7-28_19-49-49_29kfyczxqf.jpg](https://cdn.tsanfer.com/image/2025-7-28_19-49-49_29kfyczxqf.jpg "使用U盘解锁 SSH")

路由器解锁 SSH 成功之后蓝灯常亮，可使用 SSH 连接 `192.168.31.1`，用户为 `root`，密码为小米开启SSH工具官方上的 SSH 密码

此处若提示交换密钥算法没找到，则选择密钥交换算法为 `diffie-hellman-group1-sha1`​

或者使用命令行登陆：

```powershell
ssh -omacs=hmac-sha1 -oHostKeyAlgorithms=+ssh-dss -oKexAlgorithms=+diffie-hellman-group1-sha1 root@192.168.31.1
```

登陆成功效果如下

![2025-7-28_19-47-39_ukfenk4jqt.png](https://cdn.tsanfer.com/image/2025-7-28_19-47-39_ukfenk4jqt.png "通过 SSH 进入路由器命令行环境")​​

## 备份路由器数据

这里将路由器数据备份至U盘中

先找到U盘挂载的位置，在命令行中执行：

```bash
df -h

```

🖥️ 示例输出：

```bash
Filesystem                Size      Used Available Use% Mounted on
rootfs                   18.8M     18.8M         0 100% /
none                    122.8M         0    122.8M   0% /dev
tmpfs                   123.6M      4.2M    119.4M   3% /tmp
/dev/mtdblock13          18.8M     18.8M         0 100% /
tmpfs                   123.6M      4.2M    119.4M   3% /tmp
tmpfs                   123.6M      4.2M    119.4M   3% /extdisks
ubi1_0                   43.5M     11.8M     29.5M  29% /data
ubi1_0                   43.5M     11.8M     29.5M  29% /userdisk
/dev/mtdblock13          18.8M     18.8M         0 100% /userdisk/data
ubi1_0                   43.5M     11.8M     29.5M  29% /etc
/dev/sda1                30.0G    275.1M     29.7G   1% /extdisks/sda1
```

这里得知U盘挂载在 `/extdisks/sda1` 位置

在U盘根目录下创建存放备份文件的文件夹，在路由器中执行：

```bash
mkdir -p /extdisks/sda1/bak

```

查看原厂固件 ROM 分区布局，在路由器中执行：

```bash
cat /proc/mtd

```

🖥️ 示例输出：

```plaintext
dev:    size   erasesize  name
mtd0: 07f80000 00020000 "ALL"
mtd1: 00080000 00020000 "Bootloader"
mtd2: 00040000 00020000 "Config"
mtd3: 00040000 00020000 "Bdata"
mtd4: 00040000 00020000 "Factory"
mtd5: 00040000 00020000 "crash"
mtd6: 00040000 00020000 "crash_syslog"
mtd7: 00040000 00020000 "reserved0"
mtd8: 00400000 00020000 "kernel0"
mtd9: 00400000 00020000 "kernel1"
mtd10: 02000000 00020000 "rootfs0"
mtd11: 02000000 00020000 "rootfs1"
mtd12: 03580000 00020000 "overlay"
mtd13: 012a6000 0001f000 "ubi_rootfs"
mtd14: 030ec000 0001f000 "data"
```

备份路由器数据，在路由器中执行：

```powershell
dd if=/dev/mtd0 of=/extdisks/sda1/bak/ALL.bin
dd if=/dev/mtd1 of=/extdisks/sda1/bak/Bootloader.bin
dd if=/dev/mtd2 of=/extdisks/sda1/bak/Config.bin
dd if=/dev/mtd3 of=/extdisks/sda1/bak/Bdata.bin
dd if=/dev/mtd4 of=/extdisks/sda1/bak/Factory.bin
dd if=/dev/mtd5 of=/extdisks/sda1/bak/crash.bin
dd if=/dev/mtd6 of=/extdisks/sda1/bak/crash_syslog.bin
dd if=/dev/mtd7 of=/extdisks/sda1/bak/reserved0.bin
dd if=/dev/mtd8 of=/extdisks/sda1/bak/kernel0.bin
dd if=/dev/mtd9 of=/extdisks/sda1/bak/kernel1.bin
dd if=/dev/mtd10 of=/extdisks/sda1/bak/rootfs0.bin
dd if=/dev/mtd11 of=/extdisks/sda1/bak/rootfs1.bin
dd if=/dev/mtd12 of=/extdisks/sda1/bak/overlay.bin
dd if=/dev/mtd13 of=/extdisks/sda1/bak/ubi_rootfs.bin
# 此设备可能被占用导致无法备份，如有备份需要，可在 Bootloader 中进行备份
dd if=/dev/mtd14 of=/extdisks/sda1/bak/data.bin
```

若需要还原数据，可执行：

```bash
mtd write -r /extdisks/sda1/bak/ALL.bin ALL
mtd write -r /extdisks/sda1/bak/Bootloader.bin Bootloader
mtd write -r /extdisks/sda1/bak/Config.bin Config
mtd write -r /extdisks/sda1/bak/Bdata.bin Bdata
mtd write -r /extdisks/sda1/bak/Factory.bin Factory
mtd write -r /extdisks/sda1/bak/crash.bin crash
mtd write -r /extdisks/sda1/bak/crash_syslog.bin crash_syslog
mtd write -r /extdisks/sda1/bak/reserved0.bin reserved0
mtd write -r /extdisks/sda1/bak/kernel0.bin kernel0
mtd write -r /extdisks/sda1/bak/kernel1.bin kernel1
mtd write -r /extdisks/sda1/bak/rootfs0.bin rootfs0
mtd write -r /extdisks/sda1/bak/rootfs1.bin rootfs1
mtd write -r /extdisks/sda1/bak/overlay.bin overlay
mtd write -r /extdisks/sda1/bak/ubi_rootfs.bin ubi_rootfs
mtd write -r /extdisks/sda1/bak/data.bin data
```

## 刷入 Breed 覆盖原有 Bootloader

在 Breed 官网路由器对应的 Bootloader：[Breed 下载页](https://breed.hackpascal.net/)，其文件名为 `breed-mt7621-xiaomi-r3g.bin` 。

将 Breed 复制到U盘，比如U盘根目录，从U盘中刷入 Breed，覆盖路由器自带的 Bootloader：

```powershell
mtd write -r /extdisks/sda1/breed-mt7621-xiaomi-r3g.bin Bootloader
```

Bootloader 刷入完成后，路由器断电长按 Reset 键，然后通电，等待橙色灯闪烁，松开 Reset 键即可进入 Breed。此时路由器蓝灯闪烁，用以太网线连接路由器与电脑，浏览器中输入 `192.168.1.1` 即可进入 Breed Web 恢复控制台。

![2025-7-28_19-52-33_2t5wumxk98.jpg](https://cdn.tsanfer.com/image/2025-7-28_19-52-33_2t5wumxk98.jpg "路由器进入 Bootloader 的刷机模式")

![2025-7-28_19-47-56_ucijlfmw3s.png](https://cdn.tsanfer.com/image/2025-7-28_19-47-56_ucijlfmw3s.png "Breed Web 恢复控制台")​​

## 刷入 OpenWrt 固件

- 下载 OpenWrt 固件

从 OpenWrt 官网固件下载页下载 OpenWrt 固件：[OpenWrt Firmware Selector](https://firmware-selector.openwrt.org/)

可在网页中自定义预装软件与首次开机脚本

OpenWrt 常用预装软件示例：

```bash
base-files ca-bundle dnsmasq dropbear firewall4
fstools kmod-crypto-hw-eip93 kmod-gpio-button-hotplug
kmod-leds-gpio kmod-nft-offload libc libgcc
libustream-mbedtls logd mtd netifd nftables odhcp6c
odhcpd-ipv6only opkg ppp ppp-mod-pppoe procd-ujail
uboot-envtools uci uclient-fetch urandom-seed urngd
wpad-basic-mbedtls kmod-mt7603 kmod-mt76x2 kmod-usb3
kmod-usb-ledtrig-usbport luci

kmod-tun vsftpd openssh-sftp-server bash curl wget
unzip lscpu openssl-util ca-certificates
luci-i18n-base-zh-cn
```

首次开机脚本示例：  

```bash {linenos=table,hl_lines=[3,4,6,7]}
# Uncomment lines to apply:
#
wlan_name="OpenWrt"
wlan_password="password"
#
root_password="password"
lan_ip_address="192.168.0.1"
#
# pppoe_username=""
# pppoe_password=""

# log potential errors
exec >/tmp/setup.log 2>&1

if [ -n "$root_password" ]; then
  (echo "$root_password"; sleep 1; echo "$root_password") | passwd > /dev/null
fi

# Configure LAN
# More options: https://openwrt.org/docs/guide-user/base-system/basic-networking
if [ -n "$lan_ip_address" ]; then
  uci set network.lan.ipaddr="$lan_ip_address"
  uci commit network
fi

# Configure WLAN
# More options: https://openwrt.org/docs/guide-user/network/wifi/basic#wi-fi_interfaces
if [ -n "$wlan_name" -a -n "$wlan_password" -a ${#wlan_password} -ge 8 ]; then
  uci set wireless.@wifi-device[0].disabled='0'
  uci set wireless.@wifi-iface[0].disabled='0'
  uci set wireless.@wifi-iface[0].encryption='psk2'
  uci set wireless.@wifi-iface[0].ssid="$wlan_name"
  uci set wireless.@wifi-iface[0].key="$wlan_password"
  uci commit wireless
fi

# Configure PPPoE
# More options: https://openwrt.org/docs/guide-user/network/wan/wan_interface_protocols#protocol_pppoe_ppp_over_ethernet
if [ -n "$pppoe_username" -a "$pppoe_password" ]; then
  uci set network.wan.proto=pppoe
  uci set network.wan.username="$pppoe_username"
  uci set network.wan.password="$pppoe_password"
  uci commit network
fi

echo "All done!"

```

然后在网页中请求生成自定义的固件，系统生成固件完成后下载 `KERNEL1` 和 `rootfs0`​

- 刷入 OpenWrt 系统固件

> 如需使用 Breed 引导进入小米路由器原厂系统的话，需要设置从指定的分区引导，打开“环境变量编辑”，添加环境变量，名称为 `xiaomi.r3g.bootfw` ，值为 `2` 。

进入“固件更新”，选择“闪存布局”为 `小米 R3G OpenWrt`，上传 kernel1 和 rootfs0 文件，勾选“自动重启”，然后上传文件，等待路由器自动重启

![2025-7-28_19-48-17_l7pta3u38y.png](https://cdn.tsanfer.com/image/2025-7-28_19-48-17_l7pta3u38y.png "选择 OpenWrt 固件刷入")

更新过程中路由器黄灯闪烁，等待几分钟分钟蓝色常亮后即可进入 OpenWrt。OpenWrt 管理面板默认 IP 地址为 192.168.1.1，此处由于设置了首次开机脚本，将 IP 设置为了 192.168.0.1。于是在浏览器中输入 `192.168.0.1` 并验证用户名和密码后即可进入 OpenWrt 管理面板。

![2025-7-28_19-43-52_bdjpu28qr9.jpg](https://cdn.tsanfer.com/image/2025-7-28_19-43-52_bdjpu28qr9.jpg "路由器烧录程序时橙灯闪烁")

![2025-7-28_19-46-05_ow86194km6.png](https://cdn.tsanfer.com/image/2025-7-28_19-46-05_ow86194km6.png "OpenWrt LuCI 管理面板")​​

## OpenWrt 配置

> 使用 OpenWrt 的路由器时，最好网口与网线速率匹配，混用可能导致无法通过 DHCP 获得 IP 地址

### 配置局域网网段

配置项位置：网络 → 接口

在 lan 接口的常规设置中编辑 IPv4 地址，即可配置局域网网段，如果未在首次开机脚本设置，此时可设置为：`192.168.0.1`​

### 配置 DNS

📌 配置项位置：网络 → 接口

在 lan 接口的高级设置中编辑自定义的 DNS 服务器，此处设置为：`223.5.5.5` 和 `114.114.114.114`​

### 配置 WiFi

📌 配置项位置：网络 → 无线

MediaTek MT7603E 802.11b/g/n 对应 2.4GHz WiFi

MediaTek MT76x2E 802.11ac/n 对应 5GHz WiFi

编辑对应网络设备设置，ESSID 即是 WiFi 名称此处设置为 `OpenWrt`，选择加密方式为 `WPA2-PSK/WPA3-SAEMixedMode（强安全性）`，密钥即是 WiFi 密码

信道可改为自动模式避免干扰

### 配置 IP-MAC 绑定

📌 配置项位置：网络 → DHCP/DNS → 静态地址分配

将服务器分配到的 IP 地址与物理设备的 MAC 地址绑定，下表为局域网服务器 IP 地址分配情况。

|主机名|MAC 地址|IP 地址|类型|物理摆放位置|
| ---------------| -----------------| ------------| ---------------| ------------|
|pgsql-replica-2|28:75:D8:84:BB:75|192.168.0.53|从数据库2|最上层|
|pgsql-replica-1|54:F6:C5:F4:A7:48|192.168.0.52|从数据库1|第五层|
|pgsql-primary|54:F6:C5:17:97:E5|192.168.0.51|主数据库|第四层|
|pve-3|54:F6:C5:FA:B5:E7|192.168.0.4|超融合集群节点3|第三层|
|pve-2|54:F6:C5:FA:61:FB|192.168.0.3|超融合集群节点2|第二层|
|pve-1|AA:1C:04:19:B9:A0|192.168.0.2|超融合集群节点1|最下层|

### 配置域名重定向到 IP

📌 配置项位置：网络 → DHCP/DNS → 常规 → 地址

将域名重定向到 IP，其作用与局域网 DNS 类似，此处给出相关示例：

​`/router.lan/openwrt.lan/192.168.0.1`​

​`/pve-1.lan/192.168.0.2`​

​`/pve-2.lan/192.168.0.3`​

​`/pve-3.lan/192.168.0.4`​

​`/h.pigsty/g.pigsty/p.pigsty/a.pigsty/sss.pigsty/pgsql-primary.lan/192.168.0.51`​

​`/pgsql-replica-1.lan/192.168.0.52`​

​`/pgsql-replica-2.lan/192.168.0.53`​

### 配置 NTP 时间同步服务器

📌 配置项位置：系统 → 系统 → 时间同步

勾选 `作为 NTP 服务器提供服务`，将在 OpenWrt 中启动 NTP 服务端

上游 NTP 服务器设置示例：`ntp.aliyun.com`、`time1.cloud.tencent.com`、`ntp.tuna.tsinghua.edu.cn`、`time1.google.com`​

### 配置系统时区

📌 配置项位置：系统 → 系统 → 常规设置

此处配置时区为：`Aisa/Shanghai`​

### 配置 OPKG 软件仓库镜像源

参考链接：[openwrt | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/openwrt/)

进入 OpenWrt SSH 终端，执行自动替换命令

```bash
sed -i 's_https\?://downloads.openwrt.org_https://mirrors.tuna.tsinghua.edu.cn/openwrt_' /etc/opkg/distfeeds.conf

```

更新软件列表验证效果

```bash
opkg update

```

## **OpenWrt** 断网自动重连

> 此处的重连脚本中，使用了 HTTP 报文来进行上网登陆认证，可自行修改相关代码，以适应需求

### 安装依赖

安装 bash

```powershell
opkg update && opkg install bash

```

### 创建保密信息文件

使用独立文件保存保密信息，避免硬编码密码

文件位置：`/etc/network-check.conf`​

```conf {linenos=table,filename="/etc/network-check.conf"}
AUTH_URL="http://172.16.2.2/drcom/login"
DDDDD="your_actual_username"
UPASS="your_actual_password"
ISP_CODE="2"
TARGET_IP="172.16.2.2"
ALT_TARGET="223.5.5.5"
MAX_FAILS="3"
AUTH_CHECK_INTERVAL="5"
```

修改文件权限为仅 root 可读写

```bash
chmod 600 /etc/network-check.conf

```

### 创建断网重连脚本

文件位置：`/root/network-check.sh`​

```bash {linenos=table,hl_lines=[25,26,56,57,58,59,60],filename="/root/network-check.sh"}
#!/bin/bash

# ===== 使用环境变量 =====
: ${AUTH_URL:?"必须设置环境变量AUTH_URL"}
: ${DDDDD:?"必须设置环境变量DDDDD"}
: ${UPASS:?"必须设置环境变量UPASS"}
: ${ISP_CODE:?"必须设置环境变量ISP_CODE"}
: ${TARGET_IP:?"必须设置环境变量TARGET_IP"}
: ${ALT_TARGET:?"必须设置环境变量ALT_TARGET"}
: ${MAX_FAILS:?"必须设置环境变量MAX_FAILS"}
: ${AUTH_CHECK_INTERVAL:?"必须设置环境变量AUTH_CHECK_INTERVAL"}

# ===== 日志函数 =====
log() {
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    local message="$1"
    # 显示到当前终端
    echo "[$timestamp] ${message}"
    # 记录到系统日志
    logger -t "network-check" "${message}"
}

# ===== 执行认证 =====
do_auth() {
    log "尝试认证: 账号: ${DDDDD}"
    # R7=0 为无感登录模式
    curl -s -m 5 "${AUTH_URL}?callback=dr1006&DDDDD=${DDDDD}&upass=${UPASS}&0MKKey=123456&R3=${ISP_CODE}" > /dev/null
}

# ===== 网络检测函数 (兼容各版本) =====
check_network() {
    # 尝试检测认证服务器
    ping -c 1 -w 2 $TARGET_IP > /dev/null 2>&1 && return 0    # 使用 -w (新版本)
    ping -c 1 -W 2 $TARGET_IP > /dev/null 2>&1 && return 0    # 使用 -W (较旧版本)
    
    # 尝试备选地址
    ping -c 1 -w 2 $ALT_TARGET > /dev/null 2>&1 && return 0
    ping -c 1 -W 2 $ALT_TARGET > /dev/null 2>&1 && return 0
    
    return 1
}

# ===== 主程序入口 =====
log "===== 网络检测服务启动 ====="
log "主目标 IP: $TARGET_IP"
log "备选目标 IP: $ALT_TARGET"
log "用户账号: $DDDDD"
log "服务商: ${ISP_CODE} (0-校园网 1-电信 2-移动 3-联通)"
log "开始监控网络状态..."

fail_count=0
auth_counter=0

while true; do
    # 1. 认证页面检测 (每5秒检测一次)
    if [ $auth_counter -ge $AUTH_CHECK_INTERVAL ]; then
        if curl -s -m 3 "http://172.16.2.2/" | grep -q "Dr.COMWebLoginID_0.htm"; then
            log "检测到认证页面，触发认证..."
            do_auth
        fi
        auth_counter=0
    fi
    
    # 2. 网络连通性检测
    if check_network; then
        # 网络正常
        if [ $fail_count -gt 0 ]; then
            log "✅ 网络恢复连通"
            fail_count=0
        fi
    else
        # 网络连接失败
        fail_count=$((fail_count + 1))
        log "❌ 网络检测失败 (累计: ${fail_count}/${MAX_FAILS})"
        
        # 首先尝试认证
        log "尝试自动认证修复..."
        do_auth
        
        # 达到最大失败次数，重启网络
        if [ $fail_count -ge $MAX_FAILS ]; then
            log "⚠️ 达到最大失败次数，尝试重启网络服务..."
            /etc/init.d/network restart
            log "🔄 网络服务已重启，等待恢复..."
            
            # 等待网络恢复
            sleep 15
            fail_count=0
        fi
    fi
    
    # 更新计数器和等待
    auth_counter=$((auth_counter + 1))
    sleep 1
done

```

添加执行权限

```bash
chmod +x /root/network-check.sh

```

### 创建 procd 守护脚本

文件位置：`/etc/init.d/network-check`​

```bash {linenos=table,hl_lines=[8],filename="/etc/init.d/network-check"}
#!/bin/sh /etc/rc.common

# 服务初始化脚本
START=90
STOP=01
USE_PROCD=1

CONFIG="/etc/network-check.conf"

start_service() {
    # 加载配置文件
    if [ ! -f "$CONFIG" ]; then
        echo "错误: 配置文件 $CONFIG 不存在" >&2
        exit 1
    fi
    . $CONFIG || exit 1
    
    procd_open_instance
    
    # 设置环境变量
    procd_set_param env \
        AUTH_URL="$AUTH_URL" \
        DDDDD="$DDDDD" \
        UPASS="$UPASS" \
        ISP_CODE="$ISP_CODE" \
        TARGET_IP="$TARGET_IP" \
        ALT_TARGET="$ALT_TARGET" \
        MAX_FAILS="$MAX_FAILS" \
        AUTH_CHECK_INTERVAL="$AUTH_CHECK_INTERVAL"
    
    procd_set_param command /root/network-check.sh
    procd_set_param pidfile /var/run/network-check.pid
    procd_set_param stdout 1
    procd_set_param stderr 1
    procd_set_param respawn 300 5 5
    procd_close_instance
}

stop_service() {
    # 当需要特殊停止逻辑时可在此添加
    # 默认行为已能正常停止服务
    :
}

```

启用守护脚本

```bash
# 1. 设置可执行权限
chmod +x /etc/init.d/network-check

# 2. 启用服务（开机自启）
# /etc/init.d/network-check enable
service network-check enable

# 启动
service network-check start
# 停止
# service network-check stop
# 重启
# service network-check restart
# 查看状态
service network-check status
```

查看日志

```bash
logread -e network-check && logread -f network-check
# logread | grep network-check
# tail -fn+1 /data/shell/log/network-check.log

# 清除当前日志
# /etc/init.d/log restart
```

## OpenWrt WiFi 断连自动重启 WiFi

为什么 OpenWrt 会 WiFi 断连：[路由器刷Openwrt后不定时wifi无线设备掉线解决方法，尝试有效-OPENWRT专版-恩山无线论坛 - Powered by Discuz!](https://www.right.com.cn/forum/thread-8396260-1-1.html)

### 自动检测 WiFi 状态脚本

此脚本在没有WiFi客户端连接路由器时，自动重启路由器的 WiFi 设备模块

文件位置：`/root/wifi-check.sh`​

```bash {linenos=table,hl_lines=[53,58],filename="/root/wifi-check.sh"}
#!/bin/sh
export PATH="/usr/sbin:/usr/bin:/sbin:/bin"

# ===== 日志函数 =====
log() {
    local t=$(date '+%F %T')
    echo "[$t] $1"
    logger -t wifi-check "$1"
}

WIFI_INTERFACE_2G=""; WIFI_INTERFACE_5G=""

for iface in $(iw dev | awk '/Interface/ {print $2}'); do
    ch=$(iwinfo "$iface" info 2>/dev/null | \
         sed -n 's/.*Channel: *\([0-9]\+\).*/\1/p')
    [ -z "$ch" ] && continue
    case $ch in
        [0-9]|[0-9][0-9]) ;;   # 确保是纯数字
        *) continue ;;
    esac
    if [ "$ch" -le 14 ]; then
        WIFI_INTERFACE_2G=$iface
    else
        WIFI_INTERFACE_5G=$iface
    fi
done

log "检测到的接口：2.4G=$WIFI_INTERFACE_2G  5G=$WIFI_INTERFACE_5G"

# 如果两个接口都没找到，直接退出
[ -z "$WIFI_INTERFACE_2G" ] && [ -z "$WIFI_INTERFACE_5G" ] && {
    log "无法识别无线接口，脚本退出"
    exit 0
}

# ===== 主逻辑（与原先基本相同，只是接口名改为变量） =====
count_2g=0
count_5g=0
[ -n "$WIFI_INTERFACE_2G" ] && \
    count_2g=$(iw dev "$WIFI_INTERFACE_2G" station dump | grep -c Station)
[ -n "$WIFI_INTERFACE_5G" ] && \
    count_5g=$(iw dev "$WIFI_INTERFACE_5G" station dump | grep -c Station)

if [ "$count_2g" -eq 0 ] && [ "$count_5g" -eq 0 ]; then
    log "无客户端连接，准备重启无线"

    iwinfo > /tmp/wifi_abnormal.log
    log "异常状态：$(cat /tmp/wifi_abnormal.log)"

    wifi down
    sleep 2

    for m in mt7603e mt76x2e mt76x2_common mt76x02_lib mt76 mac80211; do
        lsmod | grep -q "^$m" && { log "卸载 $m"; rmmod $m; }
    done
    sleep 3

    for m in mac80211 mt76 mt76x02_lib mt76x2_common mt76x2e mt7603e; do
        log "加载 $m"; modprobe $m
    done
    sleep 2

    wifi up
    # 重启 WiFi 模块后，等待客户端重连
    sleep 30

    # 重新检测
    count_2g=0
    count_5g=0
    [ -n "$WIFI_INTERFACE_2G" ] && \
        count_2g=$(iw dev "$WIFI_INTERFACE_2G" station dump | grep -c Station)
    [ -n "$WIFI_INTERFACE_5G" ] && \
        count_5g=$(iw dev "$WIFI_INTERFACE_5G" station dump | grep -c Station)

    if [ "$count_2g" -eq 0 ] && [ "$count_5g" -eq 0 ]; then
        log "警告：重启后仍无客户端连接"
    else
        log "成功：已有客户端连接"
    fi
else
    log "当前有客户端连接，无需操作"
fi

exit 0

```

测试运行：

```bash
chmod +x /root/wifi-check.sh
/root/wifi-check.sh

```

查看运行日志：

```bash
logread -e wifi-check && logread -f wifi-check

# 清除当前日志
# /etc/init.d/log restart
```

### 添加定时任务

```bash
# 周期性运行脚本，检查 WiFi 状态
echo "*/5 * * * * /root/wifi-check.sh" >> /etc/crontabs/root
# crontab -e

# 重启定时任务
/etc/init.d/cron restart

```

查看脚本运行日志

```bash
logread -e wifi-check && logread -f wifi-check

# 清除当前日志
# /etc/init.d/log restart
```

## 还原原厂整机固件包

如需重置路由器为原厂状态，可以通过刷入原厂整机固件包实现。

断电长按 Reset 键，然后通电，等待橙色灯闪烁松开 Reset 键，路由器进入 Bootloader 启动模式，此时路由器蓝灯闪烁。

用以太网线连接电脑与路由器，确保电脑的本机 IP 在 192.168.1.0 网段内，网关为 192.168.1.1。在浏览器中输入 `192.168.1.1` 即可进入 Breed Web 恢复控制台。

进入“固件更新”，选择“闪存布局”为 `小米 R3G 原厂`。还原原厂整机固件包需要四个原厂文件： Bootloader、系统固件、EEPROM、Bdata，此前已备份或下载相关文件，此处给出其的对应关系：

​`Bootloader.bin` 为 Bootloader 文件

系统固件文件可在小米路由器官网下载

​`Factory.bin` 为 EEPROM 文件

​`Bdata.bin` 为 Bdata 文件

![2025-7-28_19-58-12_k4yujl2y04.png](https://cdn.tsanfer.com/image/2025-7-28_19-58-12_k4yujl2y04.png "选择原厂整机固件包刷入")

选择对应的四个文件，勾选“自动重启”，然后上传文件，等待路由器自动重启后，即可还原原厂整机固件包

‍
