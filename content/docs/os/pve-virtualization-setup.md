---
draft: false
title: '安装配置虚拟化平台 Proxmox VE'
weight: 3
---

{{< callout emoji="🛠" >}}
  此页内容待完善
{{< /callout >}}

参考资源：

[GitHub - sqlsec/PVE: 国光的 PVE 生产力环境搭建教程](https://github.com/sqlsec/PVE)

[GitHub - oneclickvirt/pve: 用于 Proxmox VE 的多架构一键服务器部署脚本，支持批量或单独开设虚拟机，内置内外网端口转发和NAT，兼容 ARM64 与 AMD64 架构](https://github.com/oneclickvirt/pve)

---

## PVE 创建集群

参考链接：

[第五章 集群管理 — Promxox VE 中文文档 7.1 文档](https://pve-doc-cn.readthedocs.io/zh-cn/latest/chapter_pvecm/index.html)

[电脑死机服务继续？集群原来用起来这么有安全感！——PVE集群初体验_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1VF4m1A7L2/)

> PVE 的集群为多主架构，各节点功能相同，没有主次之分

- 创建集群

选择一个节点，通过网页创建集群

📌 位置：数据中心 → 集群 → `创建集群`​

- 新增集群节点

  > 为避免虚拟机 ID 冲突，Proxmox VE 规定新节点加入集群前不能配置有任何虚拟机。如果节点上已有虚拟机，可以首先使用 vzdump 将所有虚拟机备份，然后删除节点上的虚拟机，待加入集群后再用新的虚拟机 ID 恢复原有虚拟机。
  >

  - 复制已创建的机器认证信息

  📌 位置：数据中心 → 集群 → `加入信息`​

  - 进入其他节点网页加入集群

  📌 位置：数据中心 → 集群 → `加入集群`​

  粘贴信息并输入密码

![2025-9-1_18-32-39_fti1pkh12d.png](https://cdn.tsanfer.com/image/2025-9-1_18-32-39_fti1pkh12d.png "Proxmox VE 集群")

- （附加）删除异常节点

当特定节点出现异常，需要删除该节点时，可运行

```bash
pvecm delnode pve-3
```

## 部署超融合 Ceph 集群

> 创建高可用 Ceph 分布式存储集群

参考链接：

[第八章 部署超融合Ceph集群 — Promxox VE 中文文档 7.1 文档](https://pve-doc-cn.readthedocs.io/zh-cn/latest/chapter_pveceph/index.html)

[Proxmox VE + Ceph 超融合项目实战（第七部分：Ceph分布式存储） - Varden - 博客园](https://www.cnblogs.com/varden/p/15209401.html)

> 不要使用硬 RAID
>
> Ceph 需要直接控制磁盘硬件设备。硬件 RAID 控制器并非为 Ceph 所设计，其写操作管理和缓存算法可能干扰 Ceph 对磁盘的正常操作，从而把事情复杂化，并导致性能降低

### Ceph安装和初始化配置

- 安装方式一： 通过 Web 面板安装

  > 此方式在安装部分组件时无法进行国内加速
  >

  📌 位置：节点 → Ceph → `安装 Ceph`​

  打开集群节点中的 Web 面板，使用向导进行安装

  Ceph 的版本选择刚才命令提示的版本，存储库选择 `无订阅`，以使用本地镜像源加速下载。
- 安装方式二：命令行手动安装

  > 此方式适用于网络不佳，需要加速安装 Ceph 时
  >

  ```bash
  # 安装所有必要组件
  sudo apt -y install \
    ceph-base \
    ceph-common \
    ceph-mgr \
    ceph-mon \
    ceph-osd \
    python3-cephfs \
    python3-rados \
    python3-rbd \
    librados2-perl
  ```

在所有 ceph 节点中重复以上安装操作

- 初始化配置 Ceph

  不管安装方式是命令行还是 Web 面板，安装完成后都能打开 Web 面板对 Ceph 进行配置

- 配置 Ceph 监视器

  📌 位置：节点 → Ceph → 监视器

  每个节点都 `创建` `监视器` 与 `管理员`，数量按需选择，这里每个节点各添加一个监视器与管理员

- 配置 Ceph OSD

  📌 位置：节点 → Ceph → OSD

  配置 Ceph OSD 对象存储守护进程，它负责管理存储的数据对象，执行数据的复制、恢复、回收等任务，确保数据的可靠性和高可用性。

  将每个节点都选择未使用的硬盘 `创建 Ceph OSD`，其的 DB 和 WAL 盘暂时保持默认，设备类型按情况选择：`NVMe` 或 `SSD`。

  ![2025-9-9_20-14-47.png](https://cdn.tsanfer.com/image/2025-9-9_20-14-47.png "配置 Ceph OSD")​​
- 配置 Ceph 资源池

  📌 位置：节点 → Ceph → 资源池

  ​`创建资源池`，成功后每个集群节点都会多出一个共享的块存储，用于存放虚拟磁盘。此处命名此存储为：`SSD-Storage`。

  ![2025-9-1_18-48-08_t2fdtan53e.png](https://cdn.tsanfer.com/image/2025-9-1_18-48-08_t2fdtan53e.png "配置 Ceph 资源池")
- 配置 CephFS 文件系统

  上面 Ceph 资源池创建的是块设备，这里创建 CephFS 作为文件系统用于存储系统镜像和一般文件

  📌 位置：节点 → Ceph → CephFS

  为每个节点都 `创建` 元数据服务器。

  接着 `创建 CephFS`，成功后每个节点会多一个共享存储。此处命名此 CephFS 为：`cephfs`​

  ![2025-9-1_19-16-01_6ew5xnr8gt.png](https://cdn.tsanfer.com/image/2025-9-1_19-16-01_6ew5xnr8gt.png "配置 CephFS 文件系统")

![2025-9-3_17-01-25_4bh2fnns9h.png](https://cdn.tsanfer.com/image/2025-9-3_17-01-25_4bh2fnns9h.png "Ceph 配置完成")

- （附加）卸载 Ceph

如果不小心损坏了 Ceph 系统的话，比较彻底的解决方法就是重装它，参考卸载 Ceph 命令如下（若未卸载干净，可多执行两遍）：

```bash
systemctl stop ceph-mon.target
systemctl stop ceph-mgr.target
systemctl stop ceph-mds.target
systemctl stop ceph-osd.target
rm -rf /etc/systemd/system/ceph*
killall -9 ceph-mon ceph-mgr ceph-mds
rm -rf /var/lib/ceph/mon/  /var/lib/ceph/mgr/  /var/lib/ceph/mds/
pveceph purge
apt -y purge ceph-mon ceph-osd ceph-mgr ceph-mds
apt -y purge ceph-base ceph-mgr-modules-core
rm -rf /etc/ceph/* /etc/pve/ceph.conf /etc/pve/priv/ceph.*
apt -y autoremove
```

- （附加）删除 Ceph 监视器

```bash
ceph mon remove pve-3
```

编辑 Ceph 配置文件 `/etc/pve/ceph.conf`，删除相应节点的配置信息

## 创建虚拟机

参考链接：[pve_configuration_notes/04.PVE创建模板虚拟机.md at main · CallMeR/pve_configuration_notes · GitHub](https://github.com/CallMeR/pve_configuration_notes/blob/main/04.PVE%E5%88%9B%E5%BB%BA%E6%A8%A1%E6%9D%BF%E8%99%9A%E6%8B%9F%E6%9C%BA.md)

> 虚拟机镜像：`debian-13-genericcloud-amd64.qcow2`​

将虚拟机制作成模板，可以在以后创建新的虚拟机时，快速地从该模板进行克隆，从而大大缩短系统安装和配置的时间。

### 创建虚拟机

云环境镜像链接：

Debian：[Index of /images/cloud/trixie/latest](https://cloud.debian.org/images/cloud/trixie/latest/)

Alpine：[downloads | Alpine Linux](https://www.alpinelinux.org/downloads/)

因为云环境版本的 Debian 镜像内去掉了大量的硬件驱动，所以体积更小，并且还优化了虚拟机环境的适配，因此选择 `genericcloud` 版本的镜像来创建虚拟机。如果使用 Alpine 系统的话，选择 `virtual` 版本的系统镜像。

过程如下：

1. 点击 Proxmox VE 管理后台，点击右上角的 `创建虚拟机`，进入虚拟机创建流程

2. 常规：可以自定义虚拟机名称
3. 操作系统：`不使用任何介质`​
4. 系统：勾选 `Qemu 代理`​
5. 磁盘：由于使用 Debian 云镜像制作虚拟机模板，此处 `删除所有磁盘`​
6. CPU：`核心数` 酌情设置，类别选择 `host`，主板有多个 CPU 的可以勾选启用 NUMA
7. 内存：容量酌情设置，因为不需要超开所以 `取消勾选 Ballooning 设备`。如果内存资源确实紧张，可以考虑在必要时临时启用此功能，以更灵活地调整内存使用。
8. 网络：`关闭防火墙`，将 `Multiqueue` 设置为与 `核心数相同`，防止网络I/O操作成为瓶颈。

![2025-9-2_00-39-46_vgh69ffcwn.png](https://cdn.tsanfer.com/image/2025-9-2_00-39-46_vgh69ffcwn.png "创建虚拟机完成")

### 加载系统镜像

把官方的 Cloud Image 云环境系统镜像添加到虚拟机中，步骤如下：

1. 📌 位置：虚拟机 → 硬件

    卸载 CD/DVD 驱动器
2. 导入镜像文件

    把镜像文件 `上传` 到服务器，并检验其完整性，比如将 `debian-13-genericcloud-amd64.qcow2` 镜像上传到 `/root/OS-images`，然后执行

    ```bash
    # 检验镜像文件哈希值
    sha512sum /root/OS-images/debian-13-genericcloud-amd64.qcow2
    ```

    🖥️ 示例输出：

    ```bash
    d76122c87c940d1ab9334f4307c98c01dc42f0b49a20cddf278d59b92d34ab63d05ac1f40dffda3d2d32e381f097706eee6ccbf79a596bfb2cbb3d83c635ae35  /tmp/Debian/debian-13-genericcloud-amd64.qcow2
    ```

    将此结果与官方仓库中的 `SHA512SUMS` 文件记录值比对，如果一致则镜像正确

    将镜像文件 `导入` 虚拟机，替换其中的 `虚拟机ID` 和 `存储ID` 为实际值，此处设置硬盘格式为 `qcow2`，以支持硬盘快照

    ```bash
    # 将 qcow2 镜像导入虚拟机中
    qm importdisk <VM_ID> /root/OS-images/debian-13-genericcloud-amd64.qcow2 <local_stroage> --format qcow2

    :<<EOF 比如
    qm importdisk 100 /root/OS-images/debian-13-genericcloud-amd64.qcow2 SSD-Storage --format qcow2
    EOF
    ```

    🖥️ 示例输出：

    ```bash
    importing disk '/root/OS-images/debian-13-genericcloud-amd64.qcow2' to VM 100 ...
    format 'qcow2' is not supported by the target storage - using 'raw' instead
    transferred 0.0 B of 3.0 GiB (0.00%)
    transferred 30.7 MiB of 3.0 GiB (1.00%)
    ...
    transferred 3.0 GiB of 3.0 GiB (99.25%)
    transferred 3.0 GiB of 3.0 GiB (100.00%)
    unused0: successfully imported disk 'SSD-Storage:vm-100-disk-0'
    ```

    之后就能看见硬件中多了一个 `磁盘` 设备

    ![2025-9-3_23-35-22_ulowhleg7a.png](https://cdn.tsanfer.com/image/2025-9-3_23-35-22_ulowhleg7a.png "导入系统镜像完成")​​

### 配置硬件参数

📌 位置：虚拟机 → 硬件

1. ​`编辑` 硬盘

    勾选 `丢弃`：有助于回收不再使用存储空间，提高存储效率。

    勾选 `SSD仿真`：让虚拟机将虚拟磁盘视为 SSD 存储设备，从而利用 SSD 的高性能特性。

    勾选 `IO Thread`：启用 IO 线程可以提高虚拟机的磁盘 I/O 性能。
2. 由于导入的硬盘容量较小，所以需要在 `磁盘操作` 中 `调整大小`，酌情增加磁盘空间。
3. 添加 `CloudInit 设备`，初始化系统和管理配置云环境镜像

    总线/设备：`SCSI`，编号 `1` （与先前的硬盘编号不同）

    存储：为之前创建的 Ceph 资源池，比如：`SSD-Storage`​
4. 添加 `串口` 设备，允许通过 Xterm.js 连接虚拟机，以实现共享剪切板等功能
5. 硬件设备添加完成后，如下图所示：

    ![2025-9-11_17-15-25.png](https://cdn.tsanfer.com/image/2025-9-11_17-15-25.png)

### 配置虚拟机 Cloudinit 初始化参数

📌 位置：虚拟机 → Cloud-Init

1. 设置用户名和密码
2. IP 配置：

    这里选择 `DHCP` 通过路由器分配 IP，通过 IP-MAC 绑定统一管理 IP

    如需设置静态 IP 可设置 `IP 地址` 与 `网关`​

    IPv6 选择 SLAAC 自我分配地址

Cloudinit 要在虚拟机关机情况下修改才会生效

### 配置虚拟机选项

根据需求配置选项，类似于设置主板 BIOS，以下为示例

📌 位置：虚拟机 → 选项

1. 开机自启动：勾选 `开机自启动`​
2. 启动/关机顺序：设置虚拟机之间的 `启动顺序`，关机顺序与开机顺序相反，1 为最先启动
3. 引导顺序：`启用` 导入的镜像设备，并将其排在 `首位`​

### 设置备注信息

📌 位置：虚拟机 → 概要

按需设置，此处示例：

```bash
### 服务器信息

- 系统： Debian 13

- 用途： Debian ( 模板 )

- 自启： 是

- 用户： tsanfer

- IPv4： 192.168.0.100/24
 
- IPv6： SLAAC
```

然后打开虚拟机即可，记得路由器开启 DHCP，这样才能分配 IP 给虚拟机

![2025-9-8_01-48-14.png](https://cdn.tsanfer.com/image/2025-9-8_01-48-14.png "虚拟机创建成功")

## 制作虚拟机模板

进入虚拟机，对其进行一些配置后生成虚拟机模板

### 配置 SSH

参考本教程前面的章节，配置 SSH，在网页中的虚拟机终端中键入

```bash
sudo vim /etc/ssh/sshd_config
```

配置选项值为

```bash
PasswordAuthentication yes
PermitEmptyPasswords no
```

修改完成保持后，重启 SSH

```bash
sudo systemctl restart ssh.service
```

之后就可通过 SSH 客户端连接虚拟机

### （可选）添加免密 sudo 权限

> 通过 Cloudinit 初始化的用户，默认开启免密 sudo 权限

参考本教程前面的章节

配置免密码 sudo[ ]()权限：

```bash
# 使用以下命令添加免密 sudo 规则
echo '%<USERNAME> ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/<USERNAME>

:<<EOF 比如
echo '%tsanfer ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/tsanfer
EOF
```

重新登陆当前用户 shell

```bash
su - <USERNAME>

:<<EOF 比如
su - tsanfer
EOF
```

验证免密 sudo 权限：

```bash
sudo ls -la
```

### 配置软件仓库镜像源

apt 换源

```bash
sudo sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' \
  /etc/apt/sources.list.d/debian.sources
```

### 配置 ZSH

参考本教程前面的章节，使用笔者本人的 Debian 初始化脚本，初始化 ZSH：

```bash
if command -v curl >/dev/null 2>&1; then
    bash -c "$(curl -fsSL https://ghfast.top/https://raw.githubusercontent.com/Tsanfer/Setup_server/main/Setup.sh)"
elif command -v wget >/dev/null 2>&1; then
    bash -c "$(wget https://ghfast.top/https://raw.githubusercontent.com/Tsanfer/Setup_server/main/Setup.sh -O -)"
else
    echo "请先安装 curl 或 wget" >&2
fi
```

### 安装相关常用软件

```bash
# 安装系统软件
# 安装网络工具
# 写入磁盘

sudo apt -y install qemu-guest-agent guestfs-tools htop btop tmux logrotate cron neovim zsh git && \
sudo apt -y install nftables sshguard lsof && \
sudo sync
```

### 调整系统时区

```bash
# 调整系统时区
# 检查系统时间
sudo timedatectl set-timezone Asia/Shanghai && date -R
```

🖥️ 示例输出：

```bash
Tue, 02 Sep 2025 11:20:29 +0800
```

### 生成虚拟机模板

关闭虚拟机，右键点击虚拟机，选择 `转换成模板`​

![2025-9-9_22-25-07.png](https://cdn.tsanfer.com/image/2025-9-9_22-25-07.png "虚拟机模板生成成功")​​​​

在后续使用模板时，需要在路由器中配置 IP-MAC 映射

## 创建 LXC 容器

参考链接：[无特权容器vs特权容器——PVE下创建LXC容器基础流程_网络存储_什么值得买](https://post.smzdm.com/p/azoq5m9r/)

> LXC 容器镜像：`debian-13-standard`​

### 下载 CT 容器模板

📌 位置：文件存储 → CT 模板 → 模板

此处 Promox 官方提供了十几种 Linux 操作系统容器模板，选择合适的模板下载即可，比如 `debian-13-standard`​​

![2025-9-9_22-16-35.png](https://cdn.tsanfer.com/image/2025-9-9_22-16-35.png "CT 容器模板下载成功")​​​​

### 创建 LXC 容器

点击主页右上角 `创建 CT` 进入创建页面

- 常规：选中 `无特权容器` 和 `嵌套`，按需设置主机名、密码，或者也可使用密钥登陆

- 模板：存储选择下载好模板的存储、模板选择下载好的系统

- 磁盘：按需设置磁盘大小

- CPU：核心数按需设置

- 内存：内存和 Swap 虚拟内存按需设置

- 网络：按实际网络设置，比如，设置 IPv4 为 `DHCP`，IPv6 为 `SLAAC` 自我分配地址，`关闭防火墙`​

- DNS：按需设置，此处可保持空白默认

然后确认保存，等待其完成后可以看见多了一个容器，先不要开机，接下来还要对其进行设置

![2025-9-9_22-23-33.png](https://cdn.tsanfer.com/image/2025-9-9_22-23-33.png "创建 LXC 容器成功")​​

### 配置 LXC 容器

📌 位置：容器

- 网络：可复制生成的容器 MAC 地址，在路由器中进行 IP-MAC 绑定

- DNS：可按需设置主机名，比如：`Debian-13-LXC-template`​

- 选项：

  - 开机自启动：`是`​
  - 控制台模式：`shell`​
  - 功能：`嵌套` 表示可在容器中创建容器

Proxmox VE 默认将宿主机上的普通用户映射为容器内的 root 用户，限制 root 用户的权限，这样的做法相比使用特权容器来说更加安全。

配置完成后即可运行 LXC 容器，默认使用 root 用户登陆。

![2025-9-9_22-28-35.png](https://cdn.tsanfer.com/image/2025-9-9_22-28-35.png "运行 LXC 容器")​​

## 制作 LXC 容器模板

此步骤与制作虚拟机模板，以及服务器初始化类似，这里简略解释如何设置。

### 初始化 LXC 容器

apt 换源

```bash
sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' \
  /etc/apt/sources.list.d/debian.sources
```

安装 sudo

```bash
apt -y install sudo
```

使用笔者本人的 Debian 初始化脚本，初始化 LXC 容器的一部分设置。用这个脚本的主要目的是 `安装常用软件`、`配置终端`，还能顺便调用融合怪脚本 `测试服务器`。

```bash
if command -v curl >/dev/null 2>&1; then
    bash -c "$(curl -fsSL https://ghfast.top/https://raw.githubusercontent.com/Tsanfer/Setup_server/main/Setup.sh)"
elif command -v wget >/dev/null 2>&1; then
    bash -c "$(wget https://ghfast.top/https://raw.githubusercontent.com/Tsanfer/Setup_server/main/Setup.sh -O -)"
else
    echo "请先安装 curl 或 wget" >&2
fi
```

- 设置 root 的 ssh 密码登陆

可根据实际情况设置

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

- 调整系统时区

```bash
# 调整系统时区
# 检查系统时间
sudo timedatectl set-timezone Asia/Shanghai && date -R
```

### 生成 LXC 容器模板

之后即可关闭容器创建模板，步骤类似创建虚拟机模板。

关闭容器，右键点击容器，选择 `转换成模板`​

![2025-9-9_22-37-48.png](https://cdn.tsanfer.com/image/2025-9-9_22-37-48.png "创建 LXC 容器模板完成")​​

在后续使用模板时，需要在路由器中配置 IP-MAC 映射

## 备份与快照

参考链接：[讲解PVE虚拟机的:备份,快照,复制,克隆 笔记250703讲解PVE虚拟机的:备份,快照,复制,克隆 PVE虚拟机备份 - 掘金](https://juejin.cn/post/7522399266100346889)

### 备份

📌 位置：数据中心 → 备份 → 添加

- 常规：按需设置备份对象、备份计划和存储等配置，比如对 `Debian-13-LXC-template` 计划每日 `02:30` 备份、存储为 `cephfs`​
- 保留：按需设置，比如保留上次 `3`、保留每天 `6`、保留每周 `3`、保留每月 `11`、保留每年 `4`​

可点击 `现在运行` 测试备份

![2025-9-3_16-57-58_94n5jj10o6.png](https://cdn.tsanfer.com/image/2025-9-3_16-57-58_94n5jj10o6.png "备份成功")

### 快照

📌 位置：容器/虚拟机 → 快照 → 做快照

![2025-9-3_23-54-33_uwkzkhqq9i.png](https://cdn.tsanfer.com/image/2025-9-3_23-54-33_uwkzkhqq9i.png "快照制作完成")​​

## 高可用设置

📌 位置：数据中心 → HA → 添加

选择需要高可用的 KVM 虚拟机与 LXC 容器，按需设置最大重启与最大重定位等参数。设置完 HA 高可用过后节点如果断开与集群的连接，一段时间后该节点会自动重启

![2025-9-10_16-39-56.png](https://cdn.tsanfer.com/image/2025-9-10_16-39-56.png "设置 HA 高可用完成")​​

## 创建只读用户

用于展示 Proxmox VE 后台

📌 位置：数据中心 → 权限

- 用户：`添加`​

  - 领域：`Proxmox VE authentication server`​
  - 设置用户名和密码
- 权限：`添加用户`​

  - 路径：`/`​
  - 用户：`viewer@pve`​
  - 角色：`PVEAuditor`​

‍
