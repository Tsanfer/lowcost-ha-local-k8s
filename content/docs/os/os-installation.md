---
draft: false
title: '安装操作系统'
weight: 1
---

{{< callout emoji="🛠" >}}
  此页内容待完善
{{< /callout >}}


## 操作系统选择

### PVE 服务器

参考链接：[FAQ - Proxmox VE](https://pve.proxmox.com/wiki/FAQ#faq-support-table)

PVE 官方推荐最新的 Proxmox VE 8 安装于 Debian 12 发行版

### 数据库服务器

参考链接：[OS兼容性 | PIGSTY](https://pigsty.cc/docs/reference/compatibility/)

Pigsty 官方推荐安装其系统于 EL9，Debian 12，Ubuntu 22.04 发行版

## 方式一：通过脚本重装操作系统

参考链接：[bin456789/reinstall: 一键 DD/重装脚本 (One-click reinstall OS on VPS)](https://github.com/bin456789/reinstall)

> 此方式适合已安装操作系统的情况

更新软件列表

```bash
apt -y update 

```

下载运行系统重装脚本，此处可自行设置发行版本和用户密码

```bash
curl -O https://cnb.cool/bin456789/reinstall/-/git/raw/main/reinstall.sh || wget -O reinstall.sh $_
bash reinstall.sh debian 12 --password 123456 --ssh-port 22 --web-port 58080
```

重启

```bash
reboot

```

系统重装过程可通过链接 SSH 或主机的 web 端口查看，比如：`http://192.168.0.3:58080`​

```bash
# 重新连接 SSH 后查看安装进度
tail -fn+1 /reinstall.log

```

安装完成后，系统会自动重启

## 方式二：通过系统镜像手动安装操作系统

> 此方式适合未安装操作系统的或无网络连接的情况

> Debian 官方镜像链接：[下载 Debian](https://www.debian.org/distrib/)

此处以安装 Debian 为例，其他发行版的镜像安装方式也大同小异。

先将官方镜像文件放入U盘启动盘中，然后在服务器开机时选择从对应镜像启动，之后根据提示安装即可。

询问安装软件时，选择 SSH 服务器与工具标准系统工具，以方便后续连接服务器操作

- 设置 root 的 ssh 密码登陆

先通过刚刚创建的自定义用户登陆 ssh，并切换为 root

```bash
su

```

然后在 root 状态下配置 SSH 服务

```bash
# 设置允许 Root 用户登录
cat /etc/ssh/sshd_config | grep -Eq "^[# ]?PermitRootLogin " ; [ $? -eq 0 ] && \
sed -i 's/^[# ]\?PermitRootLogin.*/PermitRootLogin yes/g' /etc/ssh/sshd_config || \
echo -e "\nPermitRootLogin yes" >> /etc/ssh/sshd_config

# 设置密码认证
cat /etc/ssh/sshd_config | grep -Eq "^[# ]?PasswordAuthentication " ; [ $? -eq 0 ] && \
sed -i 's/^[# ]\?PasswordAuthentication.*/PasswordAuthentication yes/g' /etc/ssh/sshd_config || \
echo -e "\nPasswordAuthentication yes" >> /etc/ssh/sshd_config

# 启动/重启 SSH 服务
ps -ef | grep -q ssh ; [ $? -eq 0 ] && \
systemctl restart sshd || systemctl enable --now sshdsu
systemctl enable ssh.service || service sshd restart

```

之后即可退出当前用户和终端，通过 ssh 以 root 身份访问服务器

- 更换软件源

更换 Debian 的镜像中的默认软件源，防止系统使用 cdrom 路径的软件源

```bash
if command -v curl >/dev/null 2>&1; then
    bash <(curl -sSL https://linuxmirrors.cn/main.sh)
elif command -v wget >/dev/null 2>&1; then
    wget -qO- https://linuxmirrors.cn/main.sh | bash
else
    echo "请先安装 curl 或 wget" >&2
    exit 1
fi
```

验证效果

```bash
apt -y update

```

‍
