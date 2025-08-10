---
draft: false
title: '安装配置虚拟化平台 PVE'
weight: 2
---

{{< callout emoji="🛠" >}}
  此页内容待完善
{{< /callout >}}

参考资源：

[GitHub - sqlsec/PVE: 国光的 PVE 生产力环境搭建教程](https://github.com/sqlsec/PVE)

[GitHub - oneclickvirt/pve: 用于 Proxmox VE 的多架构一键服务器部署脚本，支持批量或单独开设虚拟机，内置内外网端口转发和NAT，兼容 ARM64 与 AMD64 架构](https://github.com/oneclickvirt/pve)

---

## 安装 PVE

### 方式一：命令行脚本安装

参考教程：

[一键虚拟化项目 - 库苏恩](https://virt.spiritlhl.net/)

> 推荐在纯净的 Linux 发行版中运行脚本安装 PVE

此种安装方式无需外接显示器与键盘，可远程安装，在机器主板未集成 IPMI 等带外管理模块时，使用此方式安装更为便捷

- 安装 PVE 脚本依赖

进入刚重装好的 debian 系统，安装依赖

```bash
apt -y update && apt -y install wget curl

```

- 检测环境

```bash
# bash <(wget -qO- --no-check-certificate https://raw.githubusercontent.com/oneclickvirt/pve/main/scripts/check_kernal.sh)
bash <(wget -qO- --no-check-certificate https://cdn.spiritlhl.net/https://raw.githubusercontent.com/oneclickvirt/pve/main/scripts/check_kernal.sh)

```

- 安装 PVE

```bash
# curl -L https://raw.githubusercontent.com/oneclickvirt/pve/main/scripts/install_pve.sh -o install_pve.sh && chmod +x install_pve.sh && bash install_pve.sh
curl -L https://cdn.spiritlhl.net/https://raw.githubusercontent.com/oneclickvirt/pve/main/scripts/install_pve.sh -o install_pve.sh && \
  chmod +x install_pve.sh && \
  bash install_pve.sh
```

脚本运行完毕后，执行 reboot 重启系统，然后再次使用 SSH 登录，等待至少 20 秒后，若系统未自动重启，则再次执行此脚本

```bash
bash install_pve.sh
```

期间可设置本机的主机名，只能为英文和数字，比如：`PVE2`​

后续可手动更改主机名：

```bash
sudo hostnamectl set-hostname your-new-hostname
```

PVE 安装完成后进入本机的 8006 端口就可在网页端使用 PVE，比如：`https://192.168.0.3:8006`​

### 方式二：官方系统镜像手动安装

> 此方法选择 ext4 或 xfs 做文件系统时，会默认创建 LVM 逻辑卷管理，可能导致相应硬盘的性能下降

PVE 官方镜像地址：[ISO - Proxmox Virtual Environment](https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso)

下载官方镜像，并制作 U 盘启动盘，插入待安装的主机，进入主机 BIOS。选择 U 盘引导启动，之后可根据系统提示安装设置 PVE。

![2025-6-21_18-34-32_9756fro1gm.jpg](https://cdn.tsanfer.com/image/2025-6-21_18-34-32_9756fro1gm.jpg "设置 PVE 网络接口
")​

PVE 安装完成后进入本机的 8006 端口就可在网页端使用 PVE，比如：`https://192.168.0.41:8006`​

## PVE 创建集群

参考链接：

[第五章 集群管理 — Promxox VE 中文文档 7.1 文档](https://pve-doc-cn.readthedocs.io/zh-cn/latest/chapter_pvecm/index.html)

[电脑死机服务继续？集群原来用起来这么有安全感！——PVE集群初体验_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1VF4m1A7L2/)

> PVE 的集群为多主架构，各节点功能相同，没有主次之分

### 创建集群

选择一个节点，通过网页创建集群

数据中心 → 集群 → `创建集群`​

### 新增集群节点

> 为避免虚拟机ID冲突，Proxmox VE规定新节点加入集群前不能配置有任何虚拟机。如果节点上已有虚拟机，可以首先使用vzdump将所有虚拟机备份，然后删除节点上的虚拟机，待加入集群后再用新的虚拟机ID恢复原有虚拟机。

- 复制已创建的机器认证信息

  数据中心 → 集群 → `加入信息`​

- 进入其他节点网页加入集群

  数据中心 → 集群 → `加入集群`​

  粘贴信息并输入密码

![2025-6-21_18-16-29.png](https://cdn.tsanfer.com/image/2025-6-21_18-16-29.png "PVE 集群
")

### 创建存储

- PVE 可用的存储类型

  |名称|PVE名称|级别|支持共享|支持快照|是否稳定|
  | :---------------| :-------------| ------| ----------| ----------| ----------|
  |ZFS（本地）|zfspool|块|否|是|是|
  |目录|dir|文件|否|否1|是|
  |BTRFS|btrfs|文件|否|是|实验性|
  |Proxmox Backup|pbs|均是|是|n/a|是|
  |NFS|nfs|文件|是|否1|是|
  |CIFS|cifs|文件|是|否1|否|
  |GlusterFS|glusterfs|文件|是|否1|是|
  |CephFS|cephfs|文件|是|是|是|
  |LVM|lvm|块|否2|否|是|
  |LVM-thin|lvmthin|块|否|是|是|
  |iSCSI/kernel|iscsi|块|是|否|是|
  |iSCSI/libiscsi|iscsidir ect|块|是|否|是|
  |Ceph/RBD|rbd|块|是|是|是|
  |ZFS over iSCSi|zsf|块|是|是|是|

### 创建 LVM-thin 存储

> LVM是在逻辑卷创建时就按设置的卷容量大小预先分配所需空间。LVM-thin存储池是在向卷内写入数据时按实际写入数据量大小分配所需空间。LVM-thin所用的存储空间分配方式允许创建容量远大于物理存储空间的存储卷，因此也称为“薄模式”。

LVM-thin存储池不能被多个节点同时共享使用，只能用于节点本地存储

## 部署超融合 Ceph 集群

参考链接：[第八章 部署超融合Ceph集群 — Promxox VE 中文文档 7.1 文档](https://pve-doc-cn.readthedocs.io/zh-cn/latest/chapter_pveceph/index.html)

> 不要使用硬RAID
>
> Ceph需要直接控制磁盘硬件设备。硬件RAID控制器并非为Ceph所设计，其写操作管理和缓存算法可能干扰Ceph对磁盘的正常操作，从而把事情复杂化，并导致性能降低

### 初始化Ceph安装和配置

- 安装方式一：命令行手动安装

  ```bash
  # 安装所有必要组件
  apt -y install \
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
- 安装方式二： 通过 Web 面板安装

  > 此方式在安装部分组件时无法进行国内加速
  >

  打开集群节点中的 Web 面板，使用向导进行安装，数据中心 → Ceph → 安装 Ceph

  Ceph 的版本选择刚才命令提示的版本，存储库选择 `无订阅`​，以使用本地镜像源加速下载

在所有 ceph 节点中重复以上安装操作

- 配置 Ceph

  不管安装方式是命令行还是 Web 面板，安装完成后都能打开 Web 面板对 Ceph 进行配置

  此处需要配置 PVE 节点的 IP，条件允许的情况下应将公网 IP 和集群 IP 分离，以提高网络稳定性，此处为简化配置，选择同一 IP

## 创建高可用 Ceph 分布式存储集群

‍

### 

​​

‍
