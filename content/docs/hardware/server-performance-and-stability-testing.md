---
draft: false
title: '服务器性能及稳定性测试'
weight: 6
---

{{< callout emoji="🛠" >}}
  此页内容待完善
{{< /callout >}}

## CPU 测试

### 方式一：CPU-Z 基准测试

> 此方法适用于 WinPE 环境

在 WinPE 中使用 CPU-Z 进行 CPU 基准测试

![2025-8-7_00-31-29_05en83z4q2.png](https://cdn.tsanfer.com/image/2025-8-7_00-31-29_05en83z4q2.png "超融合集群节点1 -  CPU-Z 测试")

CPU-Z 基准测试结果如下（附有测试结果查看链接）：

|设备|CPU 名称|CPU 频率|主板类型|单核得分|多核得分|测试结果查看链接|
| -----------------| -----------------------| -------------| -----------| ----------| ----------| ------------------|
|超融合集群节点1|Intel Xeon E5-2695 v4|2594.83 MHz|Intel X99|386|7236|[https://valid.x86.fr/btxlpp](https://valid.x86.fr/btxlpp)|

‍

### 方式二：Geekbench 6 基准测试

参考链接：[Geekbench 6 - Cross-Platform Benchmark](https://www.geekbench.com/)

> 此方法可用于 Linux 环境

‍

将下载的压缩文件上传到服务器，解压后即可直接运行，比如上传压缩文件到当前用户家目录后，可进行如下操作进行基准测试

```bash
GEEKBENCH_VERSION="6.4.0"

# 切换到当前用户家目录
cd

# 创建目标文件夹并解压文件
mkdir -p Geekbench && \
  tar -xzvf Geekbench-${GEEKBENCH_VERSION}-Linux.tar.gz -C Geekbench --strip-components=1

# 递归赋予可执行权限
chmod -R +x Geekbench/

# 运行 Geekbench
./Geekbench/geekbench6

```

Geekbench 6 基准测试结果如下（附有测试结果查看链接）：

|设备|单核得分|多核得分|CPU 名称|CPU 频率|主板类型|测试结果查看链接|
| -----------------| ----------| ----------| -----------------------| ----------| --------------| ------------------|
|超融合集群节点1|1113|8566|Intel Xeon E5-2695 v4|3.30 GHz|Intel X99|[https://browser.geekbench.com/v6/cpu/12786975](https://browser.geekbench.com/v6/cpu/12786975)|
|超融合集群节点2|155|319|Intel Celeron J1900|2.00 GHz|Centerm D610|[https://browser.geekbench.com/v6/cpu/12787312](https://browser.geekbench.com/v6/cpu/12787312)|
|超融合集群节点3|154|489|Intel Celeron J1900|2.00 GHz|Centerm D610|[https://browser.geekbench.com/v6/cpu/12787313](https://browser.geekbench.com/v6/cpu/12787313)|
|主数据库设备|154|485|Intel Celeron J1900|2.00 GHz|Centerm D610|[https://browser.geekbench.com/v6/cpu/12787315](https://browser.geekbench.com/v6/cpu/12787315)|
|从数据库设备1|155|471|Intel Celeron J1900|2.00 GHz|Centerm D610|[https://browser.geekbench.com/v6/cpu/12787319](https://browser.geekbench.com/v6/cpu/12787319)|
|从数据库设备2|155|320|Intel Celeron J1800|2.42 GHz|Centerm C93|[https://browser.geekbench.com/v6/cpu/12787353](https://browser.geekbench.com/v6/cpu/12787353)|

## 内存测试

> 在 WinPE 中进行内存测试

### 内存性能测试

使用 AIDA64 Cache & Memory Benchmark 测试 CPU 缓存与内存的性能，并查看内存通道数。

![2025-8-6_21-03-21_z352l4dmaj.png](https://cdn.tsanfer.com/image/2025-8-6_21-03-21_z352l4dmaj.png "超融合集群节点1 - CPU 缓存与内存测试")

内存性能测试结果如下：

|设备|读速度（GiB/s）|写速度（GiB/s）|复制速度（GiB/s）|延迟（ns）|内存代数|内存频率|内存通道数|
| -----------------| -----------------| -----------------| -------------------| ------------| ----------| ----------| ------------|
|超融合集群节点1|61.1|55.03|53.93|88.3|DDR4|2133 MHz|四通道|
|||||||||
|||||||||
|||||||||
|||||||||
|||||||||

### 内存稳定性测试

使用 TestMem5 进行内存稳定性测试，此处简单测试内存稳定性，仅运行一轮测试

![2025-8-7_00-32-54_bvigyyv917.png](https://cdn.tsanfer.com/image/2025-8-7_00-32-54_bvigyyv917.png "超融合集群节点1 - TestMem5 测试")

## 硬盘测试

参考链接：

[NAND Flash存储器与SSD简介 - SSD Fans](http://www.ssdfans.com/?p=2495)

[FTL之垃圾回收、写放大和OP - SSD Fans](http://www.ssdfans.com/?p=8163)

[产品技术 - 创见资讯](https://cn.transcend-info.com/Embedded/Technologies/)

[专业技术-建兴储存科技 SSSTC Technology Corporation](https://www.ssstc.com/cn/technology/)

因为所有服务器均为全闪配置，硬盘测试统一按固态硬盘的标准执行。

固态硬盘的存储芯片大多采用 NAND Flash，其最小物理存储单元是浮栅 MOSFET 晶体管。这种晶体管通过在栅极施加电压，利用量子隧穿效应将电子吸引到浮栅中，并根据存储的电子数量人为划分出几种不同的状态。为了确保浮栅中的电子数量可控，在吸引电子之前，需要先向衬底施加高电压，将浮栅中的电子吸出，以消除其他电子的干扰。因此，每次向固态硬盘写入数据前，都需要先进行擦除操作。正是由于这种“先擦除后写入”的特性，才衍生出了诸如垃圾回收、TRIM 和磨损均衡等多种机制，以避免因长时间擦除而导致的等待延迟。

鉴于固态硬盘工作模式的复杂性，并且不同的使用场景对固态硬盘的性能要求也不一致，所以最终需要根据实际情况对硬盘进行综合评估。

### 查看固态硬盘 S.M.A.R.T.

固态硬盘的 S.M.A.R.T. 数据保存在主控芯片内，可以使用工厂级量产工具或刷固件的方式重置，从而呈现虚假的统计结果。故 S.M.A.R.T. 仅作初步判断固态硬盘情况的参考。

查看固态硬盘 S.M.A.R.T. 信息的方法有很多，这里以 CrystalDiskInfo 为例进行演示。可以根据软件给出的“健康状态”来初步判断硬盘状态，当 S.M.A.R.T. 中没有定义剩余寿命时则可查看“总计写入”，结合厂商给定的 TBW 总写入字节初步判断硬盘状态。

除了“健康状态”与“总计写入”之外，还应根据 S.M.A.R.T. 中的其他参数判断硬盘状态，下面给出一些较为重要的参数：

|ID|属性名称|含义|
| ----| ----------------------------| --------------------------------|
|05|重新分配扇区数|已重映射扇区的坏块数量|
|C4|扇区物理位置重分配事件计数|已发生屏蔽坏块事件的次数|
|C5|有待处置扇区数|状态存疑，需保持关注的扇区数量|
|C7|CRC 错误计数|线材或接口异常|

### 安全擦除固态硬盘

参考链接：

[如何安全地擦除/抹掉固态硬盘数据 - Lemonade](https://limonene.top/2024-01-10/%E5%A6%82%E4%BD%95%E5%AE%89%E5%85%A8%E5%9C%B0%E6%93%A6%E9%99%A4-%E6%8A%B9%E6%8E%89%E5%9B%BA%E6%80%81%E7%A1%AC%E7%9B%98%E6%95%B0%E6%8D%AE/)

[Secure Erase 和 Sanitize 的区别是什么 | Western Digital](https://support-cn.wd.com/app/answers/detailweb/a_id/43969/~/secure-erase%E5%92%8Csanitize%E7%9A%84%E5%8C%BA%E5%88%AB%E6%98%AF%E4%BB%80%E4%B9%88)

Secure Erase 与 Sanitize 是固态硬盘的两种擦除方式：Secure Erase 仅清除映射表，相当于删除文件索引；Sanitize 不仅删除映射表，还会物理擦除所有已写入的块，彻底清除盘上全部数据，无法恢复。这两种硬盘擦除方式通常都能在一分钟内完成，比动辄数小时的全盘写入快得多，这里实测 1 TB 固态硬盘能在 10 秒内完成全盘擦除。

- 安全擦除 NVMe 硬盘：

安装 nvme-cli 命令行工具

```bash
sudo apt -y update && sudo apt -y install nvme-cli

```

查看 NVMe 设备：

```bash
sudo nvme list

```

示例输出

```bash
Node                  Generic               SN                   Model                                    Namespace Usage                      Format           FW Rev  
--------------------- --------------------- -------------------- ---------------------------------------- --------- -------------------------- ---------------- --------
/dev/nvme0n1          /dev/ng0n1            002115101B16         SSSTC CA6-8D1024                         1           1.02  TB /   1.02  TB    512   B +  0 B   ERA2101 
```

此处待测试的硬盘设备地址为：`/dev/nvme0n1`​

查看 NVMe 硬盘是否支持安全擦除（Secure Erase）或深度擦除（Sanitize）：

```bash
sudo nvme id-ctrl -H /dev/nvme0n1 | grep "fna" -A 5
sudo nvme id-ctrl -H /dev/nvme0n1 | grep "sanicap" -A 5

```

示例输出

```bash

fna       : 0
  [3:3] : 0     Format NVM Broadcast NSID (FFFFFFFFh) Supported
  [2:2] : 0     Crypto Erase Not Supported as part of Secure Erase
  [1:1] : 0     Crypto Erase Applies to Single Namespace(s)
  [0:0] : 0     Format Applies to Single Namespace(s)

sanicap   : 0xa0000002
  [31:30] : 0x2 Media is additionally modified after sanitize operation completes successfully
  [29:29] : 0x1 No-Deallocate After Sanitize bit in Sanitize command Not Supported
    [2:2] : 0   Overwrite Sanitize Operation Not Supported
    [1:1] : 0x1 Block Erase Sanitize Operation Supported
    [0:0] : 0   Crypto Erase Sanitize Operation Not Supported

```

这里显示这块 NVMe 固态支持 Block Erase 方式的 Sanitize 深度擦除

采用 Sanitize 完成全盘擦除：

```bash
sudo nvme sanitize --sanact=2 /dev/nvme0n1 
```

查看擦除进度：

```bash
sudo nvme sanitize-log -H /dev/nvme0n1

```

示例输出

```bash
Sanitize Progress                      (SPROG) :  23297 (35.548401%)
Sanitize Status                        (SSTAT) :  0x2
        [2:0]   Sanitize in Progress.
        [7:3]   Number of completed passes if most recent operation was overwrite:      0
          [8]   Global Data Erased cleared: a NS LB in the NVM subsystem has been written to or a PMR in the NVM subsystem has been enabled
Sanitize Command Dword 10 Information (SCDW10) :  0x2
Estimated Time For Overwrite                   :  4294967295 (No time period reported)
Estimated Time For Block Erase                 :  4294967295 (No time period reported)
Estimated Time For Crypto Erase                :  4294967295 (No time period reported)
Estimated Time For Overwrite (No-Deallocate)   :  4294967295 (No time period reported)
Estimated Time For Block Erase (No-Deallocate) :  4294967295 (No time period reported)
Estimated Time For Crypto Erase (No-Deallocate):  4294967295 (No time period reported)

```

等待擦除完成，此处实测等待时间在 10 秒钟以内。当显示 Most Recent Sanitize Command Completed Successfully 时，即擦除完成。

- 安全擦除 SATA 硬盘：

安装 hdparm 命令行工具

```bash
sudo apt -y update && sudo apt -y install hdparm

```

查看当前的 SATA 硬盘设备：

```bash
sudo fdisk -l | grep "Disk /dev/sd" -A 5

```

结果示例

```bash
Disk /dev/sda：55.9 GiB，60022480896 字节，117231408 个扇区
磁盘型号：ShineDisk M667 6
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节

--
Disk /dev/sdb：30 GiB，32212254720 字节，62914560 个扇区
磁盘型号：Flash Disk      
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
```

此处待测试的硬盘设备地址为：`/dev/sda`​

查看 SATA 硬盘是否支持深度擦除（Sanitize）：

```bash
sudo hdparm -I /dev/sda | grep "SANITIZE feature set" -A 5

```

示例输出

```bash
	   *	SANITIZE feature set
	   *	BLOCK_ERASE_EXT command
	   *	DOWNLOAD MICROCODE DMA command
	   *	WRITE BUFFER DMA command
	   *	READ BUFFER DMA command
	   *	Data Set Management TRIM supported (limit 8 blocks)
```

- SATA 固态硬盘支持 Sanitize 深度擦除

这里显示支持 Block Erase 模式的 Sanitize 深度擦除

全盘深度擦除：

```bash
hdparm --yes-i-know-what-i-am-doing --sanitize-block-erase /dev/sda
```

查看擦除进度：

```bash
hdparm --sanitize-status /dev/sda
```

示例输出

```bash
/dev/sda:
Issuing SANITIZE_STATUS command
Sanitize status:
    State:    SD2 Sanitize operation In Process
    Progress: 0x41c2 (25%)

```

等待擦除完成，这个过程通常不会超过一分钟。当显示 Last Sanitize Operation Completed Without Error 时，即擦除完成。

- SATA 固态硬盘不支持 Sanitize 深度擦除

如果 SATA 硬盘不支持 Sanitize 深度擦除，则使用安全擦除：

参考链接：[Advanced: Erasing SATA Drives by using the Linux hdparm Utility - GROK Knowledge Base](https://grok.lsu.edu/article.aspx?articleid=16716)

查看是否支持安全擦除：

```bash
hdparm -I /dev/sda | grep "Security:" -A 10

```

示例输出

```bash
Security: 
        Master password revision code = 65534
                supported
        not     enabled
        not     locked
                frozen
        not     expired: security count
                supported: enhanced erase
        2min for SECURITY ERASE UNIT. 2min for ENHANCED SECURITY ERASE UNIT.
Checksum: correct
```

此处硬盘还被冻结着，显示 `frozen`​，需要解冻。

睡眠电脑等待几秒钟后开机，让硬盘解冻：

```bash
echo -n mem > /sys/power/state

```

设置安全擦除密码：

```bash
hdparm --user-master username --security-set-pass password /dev/sda
hdparm -I /dev/sda | grep "Security:" -A 10

```

示例输出，显示 `enabled`​ 和 `Security level high`​

```bash
Security: 
        Master password revision code = 65534
                supported
                enabled
        not     locked
        not     frozen
        not     expired: security count
                supported: enhanced erase
        Security level high
        2min for SECURITY ERASE UNIT. 2min for ENHANCED SECURITY ERASE UNIT.
```

执行安全擦除命令：

```bash
# 支持增强擦除
hdparm --user-master username --security-erase-enhanced password /dev/sda
hdparm -I /dev/sda | grep "Security:" -A 10
# 不支持增强擦除
# hdparm --user-master username --security-erase password /dev/sda
```

示例输出，恢复正常

```bash
Security: 
        Master password revision code = 65534
                supported
        not     enabled
        not     locked
        not     frozen
        not     expired: security count
                supported: enhanced erase
        6min for SECURITY ERASE UNIT. 6min for ENHANCED SECURITY ERASE UNIT.
```

- （可选）验证擦除效果

查看硬盘中的数据，结果应全为零

```bash
dd if=/dev/sda bs=4K iflag=direct status=progress | hexdump
# 只看头部一部分
# dd if=/dev/sda bs=4K count=10K iflag=direct status=progress | hexdump

```

示例输出

```bash
0000000 0000 0000 0000 0000 0000 0000 0000 0000
*
59970899968 bytes (60 GB, 56 GiB) copied, 398 s, 151 MB/s
14653926+0 records in
14653926+0 records out
df99e6000
60022480896 bytes (60 GB, 56 GiB) copied, 398.343 s, 151 MB/s

```

### 硬盘性能测试

- 性能测试方式一：ezFIO 固态硬盘稳定态测试

> 在 Linux Live CD 中进行 ezFIO 测试

参考链接：

[ezFIO – Powerful, Simple NVMe SSD Benchmark Tool - NVM Express](https://nvmexpress.org/ezfio-powerful-simple-nvme-ssd-benchmark-tool/)

[earlephilhower/ezfio: Simple NVME/SAS/SATA SSD test framework for Linux and Windows](https://github.com/earlephilhower/ezfio)

[ezfio/ezFIO User Guide.pdf at master · earlephilhower/ezfio](https://github.com/earlephilhower/ezfio/blob/master/ezFIO%20User%20Guide.pdf)

这里使用 ezFIO 测试固态硬盘性能，其是 NVM Express 推荐过的企业级 SSD 综合测试工具，可测试固态硬盘在稳定态下的真实性能表现。ezFIO 的测试流程分为顺序和随机两阶段，共包含 11 个关键步骤：

**Sequential（顺序阶段）：**

1. **第 1 次顺序填充：**   用 **128KB 块大小** 顺序写满整个设备的可寻址空间。
2. **第 2 次顺序填充：**   重复第 1 步，确保 **预留空间（Over-provisioning）**    也被填满。
3. **顺序读测试（按块大小）：**   测试不同块大小的 **顺序读性能**（此时设备已顺序预处理）。
4. **随机读测试（按块大小）：**   测试不同块大小的 **随机读性能**（避免因 4K 随机写预处理导致大块读性能被低估）。
5. **顺序写测试（512B 块，按队列深度）：**   测试 **512B 块大小** 的顺序写性能，并观察 **队列深度** 的影响。
6. **顺序写测试（按块大小）：**   测试 **512B 到 128KB** 不同块大小的顺序写性能。

**Random（随机阶段）：**

7. **第 1 次随机填充：**   用 **4KB 块大小** 随机写满整个设备容量，确保每个 4KB 块都被覆盖。
8. **第 2 次随机填充：**   重复第 1 步，确保 **预留空间** 也被随机填充。
9. **4K 随机读测试（按队列深度）：**   测试 **4KB 随机读性能** 随 **队列深度** 的变化。
10. **4K 混合读写测试（按队列深度）：**   测试 **4KB 块大小的混合读写（如 70% 读 / 30% 写）**    随队列深度的性能。
11. **长期稳定性测试：**   运行 **20 分钟的 4K 随机混合读写（如 70% 读 / 30% 写）**    测试，观察 **IOPS 稳定性**。
12. **4K 随机写测试：**   测试 **纯 4KB 随机写性能**（IOPS 和延迟）。
13. **持续随机写测试（按块大小）：**   测试 **512B 到 128KB** 不同块大小的 **持续随机写性能**。

‍

以下是 ezFIO 稳定态测试的过程：

1. 先确认待测硬盘不是系统盘，避免操作系统读写干扰测试结果。此处使用 Live CD 系统进行测试，从U盘引导启动。
2. 然后对硬盘进行安全擦除操作
3. 使用 ezFIO 对硬盘进行测试，过程如下：

    - 安装 ezFIO

    Windows：

    先安装 fio，然后再将 ezfio 的 github 仓库压缩包下载到本地，解压即可

    Debian：

    ```bash
    apt -y install fio sdparm git && \
      git clone https://ghfast.top/https://github.com/earlephilhower/ezfio

    ```

    - 运行测试

    Windows：

    打开 `ezfio.bat`​ 脚本文件，会出现 GUI 界面，选择对应硬盘测试即可

    Linux：

    ```bash
    # 单硬盘
    # ./ezfio/ezfio.py --drive /dev/nvme0n1 --utilization 1
    # ./ezfio/ezfio.py --drive /dev/nvme0n1 --yes

    # 防止断连后会话消失
    sudo apt update && sudo apt install tmux
    tmux new -s ezfio

    # 防止终端关闭而导致测试停止
    DEV=/dev/nvme0n1
    # DEV=/dev/sda
    # LOG="ezfio_$(basename "$DEV")_$(date +%Y%m%d_%H%M%S).log"

    ./ezfio/ezfio.py --drive "$DEV" --yes

    # 多硬盘
    # ./ezfio/ezfio.py --drive /dev/sda,/dev/nvme0n1 --yes

    # 如果断连，可重新挂回会话
    # tmux attach -t ezfio     
    ```

‍

下面是 ezFIO 的测试结果：

‍

<!-- <iframe sandbox="allow-forms allow-presentation allow-same-origin allow-scripts allow-modals allow-popups" src="https://www.kdocs.cn/l/ce2slOMI0Zhp" data-src="" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="height: 861px; width: 1432px;"></iframe> -->

‍<iframe sandbox="allow-forms allow-presentation allow-same-origin allow-scripts allow-modals allow-popups" src="https://www.kdocs.cn/l/ce2slOMI0Zhp" data-src="" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="height: 600px; width: 800px;"></iframe>

```bash
# pve-1
-rw-r--r-- 1 root root 194K Aug  5 10:04 ezfio_results_60GB_36cores_2100MHz_sda_debian_2025-08-05_01-14-07.ods
-rw-r--r-- 1 root root 192K Aug  5 17:56 ezfio_results_1024GB_36cores_2100MHz_nvme0n1_debian_2025-08-05_10-16-17.ods

# pve-2
-rw-r--r-- 1 root root 213K Aug  5 14:53 ezfio_results_64GB_4cores_1990MHz_sdb_debian_2025-08-05_03-01-05.ods

# pve-3
-rw-r--r-- 1 root root 211K Aug  5 14:54 ezfio_results_64GB_4cores_1990MHz_sdb_debian_2025-08-05_03-01-06.ods

# pgsql-primary
-rw-r--r-- 1 root root 189K Aug  5 12:10 ezfio_results_128GB_4cores_1990MHz_sda_debian_2025-08-04_17-56-53.ods

# pgsql-replica-1
-rw-r--r-- 1 root root 173K Aug  3 19:54 ezfio_results_128GB_4cores_1990MHz_sda_debian_2025-08-03_16-42-27.ods

# pgsql-replica-2
-rw-r--r-- 1 root root 192K Aug  5 16:24 ezfio_results_128GB_2cores_2410MHz_sda_debian_2025-08-03_21-35-40.ods
```

‍

- 性能测试方式二：CrystalDiskMark 快速测试

> 在 WinPE 中进行 CrystalDiskMark 测试

硬盘根据使用场景的不同有各种各样的读写方式，如果仅想简单快速的查看硬盘的性能，可用使用消费级固态硬盘的通用测试工具 CrystalDiskMark 8 的读写方式进行测试。默认情况下，此软件会用指定的队列深度和线程数，对一个事先生成的测试文件做顺序/随机、读/写 IO 操作，根据硬盘在一定时间内的读写总数据量推算 MB/s 与 IOPS。重复此过程五次，取最优值输出。其的默认读写方式为以下四种：

1. **SEQ1M Q8T1**：顺序 1 MiB 读写，队列深度 8，线程数 1。模拟大文件同时读写这类高吞吐量场景。
2. **SEQ1M Q1T1**：顺序 1 MiB 读写，队列深度 1，线程数 1。模拟大文件复制、视频播放、顺序加载场景。
3. **RND4K Q32T1**：随机 4 KiB 读写，队列深度 32，线程数 1。模拟系统盘多任务并发场景，例如读写大量小文件、数据库、游戏加载。
4. **RND4K Q1T1**：随机 4 KiB 读写，队列深度 1，线程数 1。模拟单线程随机读写场景，例如操作系统启动、应用冷启动

‍

以下是 CrystalDiskMark 性能测试的过程：

1. 先确认待测硬盘不是系统盘，避免操作系统读写干扰测试结果。
2. 把整块硬盘划成一个分区，然后对整个分区发送一次 TRIM 指令，告知固态硬盘主控芯片，整个硬盘都已不再使用可以全部擦除。使其完成全盘 GC 垃圾回收过程，这一过程大多不超过一分钟的时间。这一过程可以在 WinPE 中使用 DiskGenius 软件完成。

    ![2025-8-6_15-18-31_1yh1nes8mn.png](https://cdn.tsanfer.com/image/2025-8-6_15-18-31_1yh1nes8mn.png)
3. 使用 CrystalDiskMark 8 对硬盘进行空盘测速。

‍

下面是 CrystalDiskMark 8 的测试结果：

‍

‍

### 硬盘坏块扫描

参考链接：

[固态硬盘扫描坏道的检测意义以及修复方法 - 知乎](https://zhuanlan.zhihu.com/p/685319584)

[【深入浅出SSD】探究固态硬盘SSD卡顿、掉盘、掉速现象 - 知乎](https://zhuanlan.zhihu.com/p/586715739)

[为什么固态会掉盘？著名的30分钟修复是什么原理？这么做对吗？ - 知乎](https://zhuanlan.zhihu.com/p/57617932)

> 在 WinPE 中进行硬盘坏块扫描

在实际使用中，大多数固态硬盘出现故障的原因并非存储颗粒过度擦除的问题，而是主控芯片的固件错误、主控芯片虚焊或物理损伤等其他问题。这些因素往往会导致掉盘或性能下降。

而针对坏块问题，固态硬盘具备 OP 预留空间和 FTL 逻辑映射表的坏块替换功能，这让硬盘的主控芯片能够自动识别并屏蔽坏块，因此用户无需手动进行坏块屏蔽。

然而，S.M.A.R.T. 仅被动记录故障信息，本身无法主动检测坏块。只有当用户恰好读取到损坏区域时，主控芯片才会察觉并更新数据，这导致S.M.A.R.T.信息存在滞后性。因此，扫描固态硬盘的坏块可以提前发现那些尚未被 S.M.A.R.T. 表记录的潜在问题区域，从而帮助用户及时采取措施，避免数据丢失或性能下降。

需要指出的是，重要数据备份不应仅仅依赖于存储部件本身的可靠性，而应通过增加冗余备份来提高数据的安全性。具体而言，应遵循数据备份的“3-2-1”原则：至少保持三份数据副本，使用两种不同类型的存储介质进行备份，并确保其中至少一份备份存储在异地。

‍

以下是硬盘的坏道扫描结果：

![2025-6-21_18-27-15_9zvqqazdci.jpg](https://cdn.tsanfer.com/image/2025-6-21_18-27-15_9zvqqazdci.jpg)

![2025-6-21_18-27-16_.54jfv70am.jpg](https://cdn.tsanfer.com/image/2025-6-21_18-27-16_.54jfv70am.jpg)

![2025-6-21_18-27-15_9zvqqazdci.jpg](https://cdn.tsanfer.com/image/2025-6-21_18-27-15_9zvqqazdci.jpg)

![2025-6-21_18-27-14_n4tyz15uui.jpg](https://cdn.tsanfer.com/image/2025-6-21_18-27-14_n4tyz15uui.jpg)

![2025-6-21_18-27-13_c07ge7df29.jpg](https://cdn.tsanfer.com/image/2025-6-21_18-27-13_c07ge7df29.jpg)

![2025-6-21_18-27-12_6nxgwu71da.jpg](https://cdn.tsanfer.com/image/2025-6-21_18-27-12_6nxgwu71da.jpg)

![2025-6-21_18-27-11_xidv5duneo.jpg](https://cdn.tsanfer.com/image/2025-6-21_18-27-11_xidv5duneo.jpg)

由于对固态硬盘进行了全盘读取，可能会检测出之前未识别的坏块。在这种情况下，相应的 S.M.A.R.T.信息也会随之更新。因此，这里应再次查看 S.M.A.R.T. 信息，以判断硬盘是否存在异常。

- （可选）测试固态硬盘 SLC 缓外速度

如果使用场景需要大量向固态硬盘中写入数据，可能会导致硬盘的 SLC 缓存被用尽。在这种情况下，就需要测试固态硬盘的缓外速度。具体方法是向硬盘中写入一个大文件，观察速度何时开始下降。速度下降时的写入空间大小即为 SLC 缓存的容量，而下降后的写入速度则为 SLC 缓外速度。可以使用 HD Tune 的文件基准功能来完成这项测试。

## 功耗与散热测试

此处通过调节 CPU 占用率来模拟设备在不同负载下的运行状态。

### 方式一：AIDA64 系统稳定性测试

> 在 Windows PE 中进行 AIDA64 系统稳定性测试

> 此方式适用于在组装硬件的过程测试单个设备，便于随时调试相关硬件

在 WinPE 系统中仅做待机与满载测试，目前未找到在此系统中进行全核心三分之一负载测试的方法。

进入 WinPE 系统后不做任何操作保持开机状态，即可视为进行待机测试

进入 WinPE 系统后可使用 AIDA64 等压力测试工具进行满载测试

### 方式二：stress 稳定性测试

> 在 Linux 中进行 stress 稳定性测试

> 此方式适用于在已安装完成的 Linux 系统中使用，并可进行三分之一负载测试

- 安装负载测试软件

```bash
# 临时缓存密码，免密 sudo 15分钟
# sudo -s cd
sudo apt -y update && sudo apt -y install stress

```

- 查看系统占用

```bash
# 安装 htop
sudo apt -y install htop

```

查看系统 CPU 负载

```bash
# 运行系统监控
htop

```

- （PVE 服务器可用）终端中显示传感器及 CPU 功耗（防止系统过热）

参考链接：[GitHub - KoolCore/Proxmox_VE_Status: 开源 PVE 网页后台添加处理器温度、频率，NVMe 和 SSD 信息的脚本工具，以及一些自用脚本](https://github.com/KoolCore/Proxmox_VE_Status)

性能测试时，处理器温度，负载，功耗，网卡温度等性能监控工具

```bash
GITHUB_MIRROR="hub.gitmirror.com/https://raw.githubusercontent.com"
wget -q https://${GITHUB_MIRROR}/KoolCore/Proxmox_VE_Status/refs/heads/main/sensors_logs.sh && \
  chmod +x sensors_logs.sh && \
  ./sensors_logs.sh

```

> 在 PVE 中的另一种查看系统占用的方式是，在 PVE 网页中添加显示项，相关脚本：[GitHub - a904055262/PVE-manager-status](https://github.com/a904055262/PVE-manager-status)

- 使用 stress 模拟三分之一负载测试

设置参数：

```bash
# 设置期望的系统总CPU使用百分比 (0-100)
PERCENT=33

# 自动计算多核系统的真实配额
# 公式：CPUQuota = (百分比 × CPU核心数)%
CPUQUOTA=$(( PERCENT * $(nproc) ))

# 临时缓存密码，免密 sudo 15分钟
# sudo -s cd
sudo systemd-run --scope -p CPUQuota="${CPUQUOTA}%" stress --cpu $(nproc) &

```

- 使用 stress 进行满载测试

```bash
# 后台运行压力测试
# 测试所有核心
stress --cpu $(nproc) &

# sysbench
# sysbench cpu --threads=36 --cpu-max-prime=20000 --time=60 run

```

- 停止测试

```bash
# 临时缓存密码，免密 sudo 15分钟
# sudo -s cd
sudo pkill -f stress
sudo pkill -f stress
sudo pkill -f stress

```

### 方式三：stress-ng 稳定性测试

- 安装 stress-ng

```bash
sudo apt -y update && sudo apt -y install stress-ng
```

- 日常负载模拟测试

模拟中等数据库或虚拟化节点日常负载：

所有逻辑核同时跑 50 % 利用率，20 % 内存持续读写，4 线程混合随机 4K + 顺序 64K 读写

```bash
sudo stress-ng \
  --cpu 0 --cpu-load 50 \
  --vm 4 --vm-bytes 20% `# 占 25 GB 左右，模拟缓存/缓冲` \
  --iomix 4 --hdd 4 --hdd-opts direct `# 4K/64K 混合，绕过缓存` \
  --timeout 1800s \
  &

```

- 满载测试

```bash
sudo stress-ng \
  --cpu 0 --cpu-method all `# 所有逻辑核 100 %` \
  --vm 0 --vm-bytes 95% `# 锁定 95 % 物理内存` \
  --iomix 0 `# 磁盘 100 % 随机/顺序混合 IO` \
  --hdd 0 --hdd-opts direct `# 绕过缓存，直接落盘` \
  --timeout 1800s `# 30 min；0 表示永久` \
  &
```

- 停止测试

```bash
sudo pkill -f stress-ng
sudo pkill -f stress-ng
sudo pkill -f stress-ng

```

### 实测主服务器功耗与温度

> 测试时的环境温度约为 26 度，未特别指出时主板均未放入机箱

满载 CPU 功耗 80W 左右，全核睿频为 2.4GHz 左右。

- 一般电源与下压式散热器功耗

主服务器最开始使用的是一般电源供电，下压式散热器散热，此时实测待机功耗约为 80W，满载功耗约为 300W，满载温度超过 90 度

![2025-7-9_17-44-50_9it67eb3of.jpg](https://cdn.tsanfer.com/image/2025-7-9_17-44-50_9it67eb3of.jpg "主服务器功耗_待机_下压式散热器_一般电源")

![2025-7-9_17-46-53_oa6ncrchar.jpg](https://cdn.tsanfer.com/image/2025-7-9_17-46-53_oa6ncrchar.jpg "主服务器功耗_满载_下压散热器_一般电源")

![2025-7-9_18-12-19_se33d2dm0a.png](https://cdn.tsanfer.com/image/2025-7-9_18-12-19_se33d2dm0a.png "主服务器满载功耗-下压式散热器-一般电源-未放入机箱")

- 金牌电源与下压式散热器功耗

为了改善电源效率，改用金牌电源供电，此时实测待机功耗约为 70W，满载功耗约为 210W，满载温度不超过 90 度。与一般电源相比，待机功耗下降了约 12%，满载功耗下降了约 30%。

![2025-7-9_17-44-17_xsnsktfpog.jpg](https://cdn.tsanfer.com/image/2025-7-9_17-44-17_xsnsktfpog.jpg "主服务器功耗_待机_下压式散热器_金牌电源")

![2025-7-9_17-45-33_a521xfmeuo.jpg](https://cdn.tsanfer.com/image/2025-7-9_17-45-33_a521xfmeuo.jpg "主服务器功耗_满载_下压式散热器_金牌电源")

![2025-7-9_18-07-29_5x9vtl3nb5.png](https://cdn.tsanfer.com/image/2025-7-9_18-07-29_5x9vtl3nb5.png "主服务器满载功耗-下压式散热器-金牌电源-未放入机箱")

- 金牌电源与塔式散热器功耗

由于散热效果较差，改用塔式散热器散热，此时满载温度维持在 60 度以下。与下压式散热器相比，满载温度下降了约 30 度。然后将主板放入机箱，后续使用时也均会保持主板在机箱内，这时的实测待机功耗约为 70W，三分之一负载功耗约为 100W，满载功耗约为 200W，满载温度维持在 70 度以下。这里所说的三分之一负载指的是全核心的负载均为三分之一，而非三分之一的核心满载其余三分之二的核心待机。

![2025-7-9_17-38-40_76gwl05c6l.jpg](https://cdn.tsanfer.com/image/2025-7-9_17-38-40_76gwl05c6l.jpg "主服务器功耗_待机_塔式散热器_金牌电源")

![2025-7-9_18-25-47_winzx4ls8w.jpg](https://cdn.tsanfer.com/image/2025-7-9_18-25-47_winzx4ls8w.jpg "主服务器功耗_三分之一负载_塔式散热器_金牌电源")

![2025-7-9_17-42-31_20w65tdlys.jpg](https://cdn.tsanfer.com/image/2025-7-9_17-42-31_20w65tdlys.jpg "主服务器功耗_满载_塔式散热器_金牌电源")

![2025-7-9_18-45-06_8npt4d0e4h.png](https://cdn.tsanfer.com/image/2025-7-9_18-45-06_8npt4d0e4h.png "主服务器-三分之一负载-每个线程占用情况")

![2025-7-9_18-03-55_3knqz3ccgh.png](https://cdn.tsanfer.com/image/2025-7-9_18-03-55_3knqz3ccgh.png "主服务器满载功耗-塔式散热器-金牌电源-未放入机箱")

![2025-7-9_17-53-05_yudif2ymrl.png](https://cdn.tsanfer.com/image/2025-7-9_17-53-05_yudif2ymrl.png "主服务器满载功耗-塔式散热器-金牌电源-放入机箱后")

### 实测系统总功耗

> 此处测试的设备为：所有服务器以及路由器与交换机

- 待机功耗

实测总系统待机功耗约为 100W

![2025-7-9_19-03-00_xgglz44woa.jpg](https://cdn.tsanfer.com/image/2025-7-9_19-03-00_xgglz44woa.jpg "总系统功耗_待机")

- 三分之一负载功耗

当所有服务器均为三分之一负载时，实测总系统功耗约为 140W

![2025-7-9_18-28-17_bvf0ysr55j.jpg](https://cdn.tsanfer.com/image/2025-7-9_18-28-17_bvf0ysr55j.jpg "总系统功耗_三分之一负载")

![2025-7-9_00-18-42_q71wxa7vv5.png](https://cdn.tsanfer.com/image/2025-7-9_00-18-42_q71wxa7vv5.png "三分之一负载时的系统监控")

- 满载功耗

实测总系统满载功耗约为 220W

![2025-7-9_19-03-45_pe2e8yfkdt.jpg](https://cdn.tsanfer.com/image/2025-7-9_19-03-45_pe2e8yfkdt.jpg)
