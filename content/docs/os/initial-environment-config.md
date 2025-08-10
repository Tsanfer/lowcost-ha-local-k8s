---
draft: false
title: '初始化环境配置'
weight: 3
---

{{< callout emoji="🛠" >}}
  此页内容待完善
{{< /callout >}}


## PVE 服务器配置

> 每个 PVE 节点都要进行此节的相关配置

### 删除订阅弹窗

```bash
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service

```

重开浏览器（或使用浏览器无痕无痕模式）即可验证效果

### 替换 apt 软件源

参考链接：[Proxmox - USTC Mirror Help](https://mirrors.ustc.edu.cn/help/proxmox.html)

```bash
# 屏蔽订阅企业源
mv /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.list.bak

# 若是通过一键安装脚本安装的 PVE，则无需进行此处的配置
# 修改基础系统（Debian）的软件源
sed -i 's|^deb http://ftp.debian.org|deb https://mirrors.ustc.edu.cn|g' /etc/apt/sources.list
sed -i 's|^deb http://security.debian.org|deb https://mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
# 修改 Proxmox 的软件源（PVE 8）
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
# 修改 Ceph 的软件源
if [ -f /etc/apt/sources.list.d/ceph.list ]; then
  CEPH_CODENAME=`ceph -v | grep ceph | awk '{print $(NF-1)}'`
  source /etc/os-release
  echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-$CEPH_CODENAME $VERSION_CODENAME no-subscription" > /etc/apt/sources.list.d/ceph.list
fi

```

- 更新系统软件源列表

```bash
# 更新系统软件源列表
apt -y update

```

- 升级软件包（包含升级操作系统）

```bash
# 升级软件包（包含升级操作系统）
apt -y upgrade
# 升级操作系统
# apt dist-upgrade

```

安装 sudo 命令

```bash
apt -y install sudo

```

### 替换 CT Templates 软件源

```bash
# 替换 CT Templates 软件源
sed -i.bak 's|http://download.proxmox.com|https://mirrors.ustc.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
# 生效替换
systemctl restart pvedaemon
# /usr/share/perl5/PVE/APLInfo.pm 文件属于 pve-manager 软件包，该软件包升级后，需要重新替换 URL。

```

### 添加系统监控项

参考链接：[GitHub - a904055262/PVE-manager-status](https://github.com/a904055262/PVE-manager-status)

- 修改信息栏

```bash
GITHUB_MIRROR="hub.gitmirror.com/https://raw.githubusercontent.com"
wget -q https://${GITHUB_MIRROR}/a904055262/PVE-manager-status/refs/heads/main/showtempcpufreq.sh && \
  chmod +x showtempcpufreq.sh && \
  ./showtempcpufreq.sh

```

### 修改存储类型为目录存储

由于LVM-thin是通过逻辑卷管理虚拟机的磁盘，缺少文件系统这一层，实际的IO效率比采用文件系统低很多

‍

---

## 数据库服务器配置

### 设置免密 sudo 权限

配置免密码 sudo[ ]()权限：

```bash
# 使用以下命令添加免密 sudo 规则
echo '%user_name ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/user_name
# echo '%tsanfer ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/tsanfer

```

重新登陆当前用户 shell

```bash
su - user_name
# su - tsanfer

```

验证免密 sudo 权限：

```bash
sudo ls -la

```

## 使用脚本初始化服务器

在此推荐作者本人编写的服务器初始化脚本：[Tsanfer/Setup_server: 个人 Ubuntu 服务器自动初始化脚本](https://github.com/Tsanfer/Setup_server)

```bash
bash -c "$(curl -fsSL https://ghfast.top/https://raw.githubusercontent.com/Tsanfer/Setup_server/main/Setup.sh)"
# bash -c "$(wget https://ghfast.top/https://raw.githubusercontent.com/Tsanfer/Setup_server/main/Setup.sh -O -)"

```

使用此脚本可完成以下操作：

- APT 软件更新、默认软件安装（可选择是否更换系统软件源）

  > PVE 系统请勿通过此脚本更换软件源
  >
- 配置 swap 内存
- 服务器测试
- 配置终端
- 自选软件安装/卸载
- 安装和更新 Docker
- 安装/删除 docker 容器
- 清理 APT 空间

![2025-7-10_04-54-28_brpv97otqh.png](https://cdn.tsanfer.com/image/2025-7-10_04-54-28_brpv97otqh.png "脚本选择界面（服务器初始化脚本）")

![2025-7-10_04-53-29_ek2gar9qpp.png](https://cdn.tsanfer.com/image/2025-7-10_04-53-29_ek2gar9qpp.png "调用 neofetch 查看系统信息")

‍

‍
