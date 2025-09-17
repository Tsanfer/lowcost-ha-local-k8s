---
draft: false
title: '虚拟化平台 Proxmox VE 测试'
weight: 4
---

{{< callout emoji="🚧" >}}
  此页内容待补全
{{< /callout >}}

## 性能测试

对比物理机、虚拟机、容器间的性能差距。

### 挂载硬盘

为了测试所有硬盘的性能，可临时挂载硬盘用于性能测试

- 方式一：图形化方式

📌 位置：节点 → 磁盘

硬盘挂载过程可在 Proxmox VE 网页后台中以图形化方式进行。

- 方式二：命令行方式

  - 挂载普通硬盘

  查看硬盘信息

  ```bash
  sudo fdisk -l

  ```

  🖥️ 示例输出：

  ```bash
  ...

  Disk /dev/nvme0n1: 953.87 GiB, 1024209543168 bytes, 2000409264 sectors
  Disk model: SSSTC CA6-8D1024                        
  Units: sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes	

  ...
  ```

  格式化硬盘（比如设备路径为 `/dev/nvme0n1`）

  ```bash
  sudo mkfs -F -t ext4 /dev/nvme0n1
  # sudo mkfs -F -t ext4 /dev/sda

  ```

  创建挂载点（比如挂载路径为 `/mnt/test`）

  ```bash
  sudo mkdir -p /mnt/test

  ```

  临时挂载硬盘

  ```bash
  sudo mount -t ext4 /dev/nvme0n1 /mnt/test
  # sudo mount -t ext4 /dev/sda /mnt/test

  ```

  - 挂载 LVM-thin 存储

  在 PVE 网页中可创建 LVM-thin 存储到对应硬盘，创建完成后执行

  ```bash
  # 格式化分区
  sudo mkfs -t ext4 /dev/mapper/test-test
  # 创建临时挂载点
  sudo mkdir -p /mnt/test
  # 临时挂载分区到指定目录
  sudo mount /dev/mapper/test-test /mnt/test

  ```

  - 验证是否挂载成功

  ```bash
  df -h

  ```

  - 卸载硬盘并删除挂载点

  卸载硬盘并删除挂载点，可在宿主机测试完成后执行

  ```bash
  sudo umount /dev/nvme0n1
  # sudo umount /dev/sda
  # sudo umount /dev/mapper/test-test

  rm -r /mnt/test

  ```

### 安装性能测试脚本

参考链接：[GitHub - oneclickvirt/ecs: VPS 融合怪服务器测评项目 GO 版本。尽量成为最全能的服务器测评项目，使用 Go 实现，无需任何环境依赖。](https://github.com/oneclickvirt/ecs)

此处使用融合怪脚本进行性能测试。

- 下载脚本安装环境

```bash
# 下载脚本
curl -L https://cdn.spiritlhl.net/https://raw.githubusercontent.com/oneclickvirt/ecs/master/goecs.sh -o goecs.sh && \
  chmod +x goecs.sh
# 更新包管理器（可选择）并安装环境
# 此处的环境安装可能会失败，可多次尝试此安装命令
export noninteractive=true && sudo ./goecs.sh env
```

- 安装脚本

```bash
# 安装 goecs
sudo ./goecs.sh install
```

- 运行测试脚本

```bash
# 使用 -diskmc=true 开启多硬盘测试
sudo goecs -diskmc=true
# sudo goecs -diskp /root
```

这里主要是测试服务器的硬件性能，所以选择的测试方式为： `硬件单项(系统信息+CPU+内存+dd磁盘测试+fio磁盘测试)`​

> 如果该脚本将 speedtest 的软件仓库添加到了 apt 的软件源中，可手动删除该软件仓库：
>
> ```bash
> sudo rm /etc/apt/sources.list.d/ookla_speedtest-cli.list
>
> ```

### 性能测试结果

> 在超融合集群节点1中进行相关测试

为了测试多种对象的性能，可能需要重置硬盘、清除数据，可视情况合理安排部署测试流程，以避免重要数据丢失。

> 此处的虚拟机和容器的硬件配置为：CPU 4 核、内存 8 GB、硬盘 20GB

‍

以下为性能测试在线查看链接：

- 宿主机测试结果（系统盘 LVM，数据盘目录）：[spiritlhl](https://paste.spiritlhl.net/#/show/FVrxf.txt)

- KVM 虚拟机测试结果（VirtIO 半虚拟化、NVMe 存储）：[spiritlhl](https://paste.spiritlhl.net/#/show/o9mIp.txt)

- LXC容器测试结果（NVMe 存储）：[spiritlhl](https://paste.spiritlhl.net/#/show/oMCxs.txt)

- Ceph 超融合集群测试结果（KVM 虚拟机）（NVMe 与 SATA 节点混合）：[spiritlhl](https://paste.spiritlhl.net/#/show/ps7GE.txt)

- Ceph 超融合集群测试结果（LXC 容器）（NVMe 与 SATA 节点混合）：[spiritlhl](https://paste.spiritlhl.net/#/show/mzibj.txt)

​

下面是 KVM 虚拟机的具体测试结果，供参考：

```plaintext
-------------------------------------VPS融合怪测试-------------------------------------
版本：v0.1.86
测评频道: https://t.me/vps_reviews
Go项目地址：https://github.com/oneclickvirt/ecs
Shell项目地址：https://github.com/spiritLHLS/ecs
--------------------------------------系统基础信息--------------------------------------
 CPU 型号            : Intel(R) Xeon(R) CPU E5-2695 v4 @ 2.10GHz
 CPU 数量            : 4 Virtual CPU(s)
 CPU 缓存            : L1: 256 KB / L2: 16 MB / L3: 16 MB
 AES-NI              : ✔️ Enabled
 VM-x/AMD-V/Hyper-V  : ✔️ Enabled
 内存                : 395.62 MB / 7.76 GB
 气球驱动            : ❌ Undetected
 内核页合并          : ❌ Undetected
 虚拟内存 Swap       : [ no swap partition or swap file detected ]
 硬盘空间            : 1.01 GB / 19.49 GB [5.2%%] /dev/sda1 - /
 启动盘路径          : /dev/sda1
 系统                : debian 13.0 [x86_64] 
 内核                : 6.12.41+deb13-cloud-amd64
 系统在线时间        : 0 days, 00 hours, 07 minutes
 时区                : CST
 负载                : 0.21 / 0.22 / 0.10
 虚拟化架构          : KVM
 NAT类型             : Symmetric
 TCP加速方式         : cubic
 IPV4 ASN            : AS9808 China Mobile Communications Corporation
 IPV4 Location       : Nanning / Guangxi / CN
--------------------------------CPU测试-通过sysbench测试--------------------------------
1 线程测试(单核)得分:   1038.54
4 线程测试(多核)得分:   3610.90
--------------------------------内存测试-通过sysbench测试---------------------------------
单线程顺序写速度: 16384.88 MB/s(17.18K IOPS, 5s)
单线程顺序读速度: 21144.05 MB/s(22.17K IOPS, 5s)
-----------------------------------硬盘测试-通过dd测试------------------------------------
测试路径          块大小                直接写入(IOPS)                     直接读取(IOPS)                 
/dev/sda1          100MB-4K Block     92.2 MB/s(22.51K IOPS, 1.14s)         121 MB/s(29.44K IOPS, 0.87s)      
/dev/sda1          1GB-1M Block       3.5 GB/s(3.36K IOPS, 0.30s)           1.3 GB/s(1.27K IOPS, 0.79s)       
-----------------------------------硬盘测试-通过fio测试-----------------------------------
测试路径         块大小       读测试(IOPS)            写测试(IOPS)            总和(IOPS)            
/                 4k        200.19 MB/s(50.0k)      200.72 MB/s(50.2k)      400.91 MB/s(100.2k)    
/                 64k       1.73 GB/s(27.0k)        1.74 GB/s(27.2k)        3.47 GB/s(54.2k)       
/                 512k      1.84 GB/s(3588)         1.93 GB/s(3778)         3.77 GB/s(7366)        
/                 1m        1.83 GB/s(1788)         1.95 GB/s(1907)         3.79 GB/s(3695)        
----------------------------------------------------------------------------------
花费          : 1 分 1 秒
时间          : Thu Sep 4 21:15:10 CST 2025
----------------------------------------------------------------------------------
```

此处通过 dd 与 fio 测试硬盘，得出不同结果的原因在于：

dd 是单线程、顺序的工作负载，它一次提交一个 I/O 请求，队列深度为 1，只有当前请求完成后才进行下一个请求。所以 dd 一般测出的是固态硬盘在低队列深度下的性能。

fio 支持异步 I/O，请求提交后无需等待完成即可提交下一个请求，允许高的队列深度，并且 fio 还支持多个线程同时工作。通过高的队列深度和多线程，测出的是固态硬盘在高并发负载下的性能。

‍

然后，再测试虚拟机与宿主机间的网络带宽，实测双向带宽均能达到 20Gbps 以上，能满足大部分需求，可见其的虚拟化网络性能优秀。

![2025-9-4_21-32-26_hqnsrso56f.png](https://cdn.tsanfer.com/image/2025-9-4_21-32-26_hqnsrso56f.png "测试 KVM 虚拟机与宿主机间的网络带宽")

根据实际的测试结果，我们可以看到 QEMU/KVM 虚拟机和 LXC 容器在性能上的表现都相当出色，两者的性能与物理机没有明显的差距。虽然，使用 Ceph 分布式存储时，硬盘测速较慢，但这主要是由于网络带宽的限制，而与虚拟化技术本身关系不大。笔者对此结果感到惊讶，虚拟机性能竟能与物理机相当，这可能与在 Windows 平台上使用虚拟机的体验有关。KVM 虚拟机的这种高性能的表现，还要部分归功于其采用了 VirtIO 半虚拟化技术，这种技术特别适用于 SCSI 控制器，能够显著提升设备的操作速度。

此外， QEMU 和 KVM 的结合使用提供了一种高效的虚拟化解决方案。QEMU 作为用户空间中的代理，负责管理虚拟机的 I/O 操作，包括磁盘、网络和图形显示等。而 KVM 则在内核空间中工作，负责将虚拟机的 CPU 指令直接映射到物理 CPU 上执行，从而实现接近原生硬件性能的虚拟化环境。

虽说虚拟化或多或少还是会有一部分性能开销，但对于大多数应用场景来说，这种开销是可以接受的，虚拟化带来的灵活性和资源管理优势远远超过了这些微小的性能损失。

## 备份还原测试

- 备份耗时

KVM 虚拟机：实测备份一个实际占用 `4GB` 空间的硬盘，耗时在 `25 秒` 左右。

LXC 容器：实测备份一个占用 `880MB` 空间的容器，耗时在 `50 秒` 左右。其中对系统文件进行压缩处理的耗时较长。

![2025-9-9_22-46-56.png](https://cdn.tsanfer.com/image/2025-9-9_22-46-56.png "备份 KVM 虚拟机")

![2025-9-9_22-47-53.png](https://cdn.tsanfer.com/image/2025-9-9_22-47-53.png "备份 LXC 容器")

- 备份文件大小

KVM 虚拟机：实测备份一个实际占用约 `4GB` 空间的硬盘，会生产 `1GB` 左右的备份文件。

LXC 容器：实测备份一个占用约 `880MB` 空间的容器，会生产 `300MB` 左右的备份文件。

此处可以看出 KVM 虚拟机和 LXC 容器生成的备份文件在压缩率上相差不大

![2025-9-9_23-11-13.png](https://cdn.tsanfer.com/image/2025-9-9_23-11-13.png "KVM 虚拟机备份文件")

![2025-9-9_23-11-43.png](https://cdn.tsanfer.com/image/2025-9-9_23-11-43.png "LXC 容器备份文件")

## 高可用测试

[How to install tcpping on Linux · GitHub](https://gist.github.com/cnDelbert/5fb06ccf10c19dbce3a7)

[PVE 高可用介绍及测试 - Varden - 博客园](https://www.cnblogs.com/varden/p/15211238.html)

[High Availability - Proxmox VE Administration Guide](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#chapter_ha_manager)

使用 tcpping 命令来监控虚拟机在的连通性。

### 热迁移测试

> Proxmox VE 不支持 LXC 容器的热迁移

首先进行主动的热迁移测试。由于应用了连接跟踪技术，在整个热迁移过程中，虚拟机始终保持连通状态，几乎没有出现延迟增加或连接中断的情况。实测热迁移一个实际占用约 `4GB` 空间的硬盘的 KVM 虚拟机，耗时约 `35秒`。

![2025-9-10_00-34-37.png](https://cdn.tsanfer.com/image/2025-9-10_00-34-37.png "KVM 虚拟机热迁移过程")

![2025-9-10_00-35-30.png](https://cdn.tsanfer.com/image/2025-9-10_00-35-30.png "通过 tcpping 监控热迁移时的网络连通性")

若在热迁移时发现迁移过程卡死，Ceph 系统提示 `slow ops`，则可通过在不同节点上 `重启 Ceph OSD` 来排查是出现问题的节点，若重启之后卡顿恢复则是此节点造成的故障。要想解决这个问题，就需要进一步排查该节点的硬盘情况。

![2025-9-11_23-17-32.png](https://cdn.tsanfer.com/image/2025-9-11_23-17-32.png "硬盘性能过低导致 Ceph 卡顿")

![2025-9-12_12-00-56.png](https://cdn.tsanfer.com/image/2025-9-12_12-00-56.png "由于硬盘卡顿，导致 Ceph mon 停止运行")

### 故障转移测试

在这里，为了模拟可能发生的意外情况，笔者采取了直接 `断开物理电源` 的方式来进行实验。这种方法可以测试系统在突然断电情况下的稳定性和数据保护能力。

此处测试故障转移一个实际占用约 `4GB` 硬盘空间的 KVM 虚拟机，耗时约 `3分钟`。

![2025-9-10_22-13-47.png](https://cdn.tsanfer.com/image/2025-9-10_22-13-47.png "故障转移成功")

![2025-9-10_22-14-17.png](https://cdn.tsanfer.com/image/2025-9-10_22-14-17.png "故障转移耗时")​​

### （附加）检测主机离线时间脚本

用脚本检测主机的离线时间

​`~/monitor_ping.sh`​

```bash
#!/bin/bash

# 提示用户输入主机地址，如果命令行中未提供
if [ -z "$1" ]; then
    read -rp "请输入要监控的主机地址: " HOST
else
    HOST="$1"
fi

INTERVAL=1  # 检查间隔时间（秒）

START_TIME=""

echo "开始监控主机: $HOST"
echo "监控开始时间：$(date +"%Y-%m-%d %H:%M:%S")"

while true; do
    if ping -c 1 "$HOST" > /dev/null 2>&1; then
        if [ ! -z "$START_TIME" ]; then
            END_TIME=$(date +%s%3N)
            DURATION=$((END_TIME - START_TIME))
            SEC=$((DURATION / 1000))
            MIN=$((SEC / 60))
            SEC=$((SEC % 60))
            echo "主机恢复时间：$(date +"%Y-%m-%d %H:%M:%S")"
            echo "掉线持续时间：$MIN 分 $SEC 秒"
            echo "------------------------"
            unset START_TIME
        fi
    else
        if [ -z "$START_TIME" ]; then
            START_TIME=$(date +%s%3N)
            echo "主机掉线时间：$(date +"%Y-%m-%d %H:%M:%S")"
        fi
    fi
    sleep $INTERVAL
done
```

赋予脚本执行权限

```bash
chmod +x ~/monitor_ping.sh
```

运行脚本

```bash
~/monitor_ping.sh
```