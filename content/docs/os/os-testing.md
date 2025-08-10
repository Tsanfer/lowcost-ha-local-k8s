---
draft: false
title: '（待补全）操作系统测试'
weight: 4
---

{{< callout emoji="🚧" >}}
  此页内容待补全
{{< /callout >}}

### 临时挂载硬盘

> PVE 服务器可在网页面板中挂载硬盘到目录

为了测试所有硬盘的性能，可临时挂载硬盘用于性能测试

- 挂载普通硬盘

查看硬盘信息

```bash
sudo fdisk -l

# 返回结果示例：
# Disk /dev/nvme0n1: 953.87 GiB, 1024209543168 bytes, 2000409264 sectors
# Disk model: EAGET SSD 1TB                           
# Units: sectors of 1 * 512 = 512 bytes
# Sector size (logical/physical): 512 bytes / 512 bytes
# I/O size (minimum/optimal): 512 bytes / 512 bytes

```

格式化硬盘（比如设备路径为 `/dev/nvme0n1`​）

```bash
sudo mkfs -F -t ext4 /dev/nvme0n1
# sudo mkfs -F -t ext4 /dev/sda

```

创建挂载点（比如挂载路径为 `/mnt/test`​）

```bash
sudo mkdir -p /mnt/test

```

挂载硬盘

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

测试完成后，卸载硬盘并删除挂载点

```bash
sudo umount /dev/nvme0n1
# sudo umount /dev/sda
# sudo umount /dev/mapper/test-test

rm -r /mnt/test

```
