---
draft: false
title: '网络测试'
weight: 4
---

## 本地集群网络测试

> 本地集群 IP 位置为：广西桂林移动

### 本地带宽测试

此处使用 iperf3 进行带宽测速

- 在 OpenWrt 中创建服务端

安装 iperf3

```bash
opkg update
opkg install iperf3
```

创建服务端

```bash
iperf3 -s
```

- 在 Debian 中创建客户端

安装 iperf3

```bash
sudo apt -y install iperf3
```

运行客户端，进行测试

```bash
# 双线程测速
iperf3 -c <server_ip> -P 2
```

- 测速结果示例

```bash
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   629 MBytes   527 Mbits/sec    0             sender
[  5]   0.00-10.03  sec   626 MBytes   524 Mbits/sec                  receiver
[  7]   0.00-10.00  sec   454 MBytes   381 Mbits/sec    0             sender
[  7]   0.00-10.03  sec   453 MBytes   379 Mbits/sec                  receiver
[SUM]   0.00-10.00  sec  1.06 GBytes   908 Mbits/sec    0             sender
[SUM]   0.00-10.03  sec  1.05 GBytes   902 Mbits/sec                  receiver
```

## 中转云服务器网络测试

### 网络连通性测试

此通过连续 Ping 来测试连通性，连续 Ping 测试的作用是观测网络连通性与质量，记录丢包和延迟变化，从而一眼看出线路是否时通时断、延迟是否忽高忽低。

- （已弃用）湖北襄阳电信节点

湖北襄阳节点的非电信 IP 连续 Ping 存在丢包现象。本地连续 Ping 也有较多的丢包，实测本地丢包率约为 21%。网络丢包现象明显，故此节点无法作为中转节点。

![2025-8-29_20-51-07_n3gj0gteha.png](https://cdn.tsanfer.com/image/2025-8-29_20-51-07_n3gj0gteha.png "湖北襄阳电信节点 - 多地连续 Ping 测试")

![2025-8-29_21-24-51_mida2dmav5.png](https://cdn.tsanfer.com/image/2025-8-29_21-24-51_mida2dmav5.png "湖北襄阳电信节点 - 本地连续 Ping 测试")

- 浙江宁波电信节点

此节点连续 Ping 测试性能优秀，非海外 IP 的丢包和延迟都较低。本地实测丢包率约为 0%，往返时延平均约为 44ms，时延抖动平均约为 0.47ms。网络时延性能良好，可考虑作为中转节点。

![2025-8-29_20-58-04_ryjq9a0pgc.png](https://cdn.tsanfer.com/image/2025-8-29_20-58-04_ryjq9a0pgc.png "浙江宁波电信节点 - 多地连续 Ping 测试")

![2025-8-30_00-47-06_q8vwidri4v.png](https://cdn.tsanfer.com/image/2025-8-30_00-47-06_q8vwidri4v.png "浙江宁波电信节点 - 本地连续 Ping 测试")

### 带宽测试

参考链接：[Speedtest CLI：适用于命令行的互联网速度测试](https://www.speedtest.net/zh-Hans/apps/cli)

带宽测试结果在线查看链接：

[湖北襄阳节点带宽测试 - Speedtest by Ookla - The Global Broadband Speed Test](https://www.speedtest.net/result/c/9e25629b-c5df-4306-960f-47e32ea99f09)

[浙江宁波节点带宽测试 - Speedtest by Ookla - The Global Broadband Speed Test](https://www.speedtest.net/result/c/4d18be2e-6e12-40b6-bb71-5901f8e2b37c)

‍

使用 Ookla 的命令行程序进行带宽测试

- 安装带宽测试程序

```bash
## If migrating from prior bintray install instructions please first...
# sudo rm /etc/apt/sources.list.d/speedtest.list
# sudo apt-get update
# sudo apt-get remove speedtest
## Other non-official binaries will conflict with Speedtest CLI
# Example how to remove using apt-get
# sudo apt-get remove speedtest-cli
sudo apt-get install curl && \
curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.deb.sh | sudo bash && \
sudo apt-get install speedtest
```

- 进行带宽测试

```bash
speedtest
```

- 带宽测试结果

  - （已弃用）湖北襄阳电信节点

  实测下载带宽约为 100Mbps，上传带宽约为 50Mbps，平均时延约为 15ms，时延抖动约为 0.03ms。带宽测速性能优秀，可惜前面章节测得非电信线路存在明显丢包现象。

  ![2025-8-29_23-10-10_oydssj0i0f.png](https://cdn.tsanfer.com/image/2025-8-29_23-10-10_oydssj0i0f.png "湖北襄阳电信节点 - 带宽测试")

  - 浙江宁波电信节点

  实测下载带宽约为 200Mbps，上传带宽约为 200Mbps，平均时延约为 5ms，时延抖动约为 0.07ms。带宽测速性能优秀，可考虑作为中转节点。

  ![2025-8-29_23-14-53_uvfsfemvsi.png](https://cdn.tsanfer.com/image/2025-8-29_23-14-53_uvfsfemvsi.png "浙江宁波电信节点 - 带宽测试")

### 综合测试

参考链接：[GitHub - oneclickvirt/ecs: VPS 融合怪服务器测评项目 GO 版本。尽量成为最全能的服务器测评项目，使用 Go 实现，无需任何环境依赖。](https://github.com/oneclickvirt/ecs)

综合测试结果在线查看链接：

[湖北襄阳电信节点综合测试 - spiritlhl](https://paste.spiritlhl.net/#/show/RjxXJ.txt)

[浙江宁波电信节点综合测试 - spiritlhl](https://paste.spiritlhl.net/#/show/LOCvT.txt)

中转服务器重在网络质量，我们在此对云服务器做综合测试，核心目的为检查是否存在严重超售问题，从而把网络问题看得更透。关于更多超售相关问题可以参考本教程的相关章节。

此处使用融合怪脚本进行综合测试。

- 安装 VPS 融合怪脚本

下载脚本安装环境

```bash
# 下载脚本
curl -L https://cdn.spiritlhl.net/https://raw.githubusercontent.com/oneclickvirt/ecs/master/goecs.sh -o goecs.sh && \
  chmod +x goecs.sh
# 更新包管理器（可选择）并安装环境
# 此处的环境安装可能会失败，可多次尝试此安装命令
export noninteractive=true && sudo ./goecs.sh env
```

安装脚本

```bash
# 安装 goecs
sudo ./goecs.sh install
```

运行测试脚本

```bash
# 使用 -diskmc=true 开启多硬盘测试
sudo goecs -diskmc=true
```

这里主要是综合测试云服务器，所以选择的测试方式为： `4. 精简网络版(系统信息+CPU+内存+磁盘+回程+路由+测速节点5个)`​

> 如果该脚本将 speedtest 的软件仓库添加到了 apt 的软件源中，可手动删除该软件仓库：
>
> ```bash
> sudo rm /etc/apt/sources.list.d/ookla_speedtest-cli.list
>
> ```

- 综合测试结果

  这里给出部分测试项结果参考：

  CPU单核满血跑分：Ryzen 9 7950X 约 6500，5950X 约 5700；常见 Intel E5 系列落在 800 ~ 1000；若低于 500，则可视为性能明显不足。

  内存速度：DDR3 约为 10 ~ 20GB/s；DDR4 约为 20 ~ 40GB/s；DDR5 约为 50GB/s。如果内存速度低于 10GB/s，那么证明内存性能不佳，极大概率存在超售超卖问题。

  硬盘测试的默认队列深度为 64，线程数为 2，混合随机读写各占 50%。主要检查 4K 的读和写性能，4K 随机读写 ≥200 MB/s 是 NVMe，50 ~ 100 MB/s 算普通 SSD，10 ~ 40 MB/s 就是机械硬盘，低于 10 MB/s 基本可以判定为严重超售或限速盘。

  - （已弃用）湖北襄阳电信节点

  CPU得分1000分处于正常 Intel E5 水平；内存读写 15 ~ 20GB/s 处于正常 DDR3 内存水平；硬盘 4K 随机读写都为 200MB/s 以上处于正常 NVMe 固态硬盘水平，硬盘的 1M 随机读写总合速度为 5GB/s，更能体现出此硬盘良好的性能表现。可惜前面的章节测得非电信线路存在明显丢包现象。

  ```plaintext
  -------------------------------------VPS融合怪测试-------------------------------------
  版本：v0.1.83
  测评频道: https://t.me/vps_reviews
  Go项目地址：https://github.com/oneclickvirt/ecs
  Shell项目地址：https://github.com/spiritLHLS/ecs
  --------------------------------------系统基础信息--------------------------------------
   CPU 型号            : Intel(R) Xeon(R) Platinum 8259CL CPU @ 2.50GHz
   CPU 数量            : 2 Virtual CPU(s)
   CPU 缓存            : L1: 128 KB / L2: 8 MB / L3: 16 MB
   AES-NI              : ✔️ Enabled
   VM-x/AMD-V/Hyper-V  : ✔️ Enabled
   内存                : 566.99 MB / 1.92 GB
   气球驱动            : ✔️ Enabled
   内核页合并          : ❌ Undetected
   虚拟内存 Swap       : 149.25 MB / 4.00 GB
   硬盘空间            : 10.29 GB / 29.42 GB [35.0%%] /dev/sda1 - /
   启动盘路径          : /dev/sda1
   系统                : debian 12.11 [x86_64] 
   内核                : 6.1.0-10-amd64
   系统在线时间        : 13 days, 01 hours, 24 minutes
   时区                : CST
   负载                : 0.00 / 0.00 / 0.00
   虚拟化架构          : KVM
   NAT类型             : Full Cone
   TCP加速方式         : bbr
   IPV4 ASN            : AS151185 China Telecom
   IPV4 Location       : Wuhan / Hubei / China
   IPV4 Active IPs     : 106/256 (subnet /24) 933/16384 (prefix /18)
  --------------------------------CPU测试-通过sysbench测试--------------------------------
  1 线程测试(单核)得分:   1009.93
  2 线程测试(多核)得分:   1998.20
  --------------------------------内存测试-通过sysbench测试---------------------------------
  单线程顺序写速度: 15503.48 MB/s(16.26K IOPS, 5s)
  单线程顺序读速度: 20185.28 MB/s(21.17K IOPS, 5s)
  -----------------------------------硬盘测试-通过fio测试-----------------------------------
  测试路径         块大小       读测试(IOPS)            写测试(IOPS)            总和(IOPS)            
  /root             4k        220.09 MB/s(55.0k)      220.67 MB/s(55.2k)      440.76 MB/s(110.2k)    
  /root             64k       1.91 GB/s(29.9k)        1.92 GB/s(30.0k)        3.83 GB/s(59.9k)       
  /root             512k      2.43 GB/s(4750)         2.56 GB/s(5002)         4.99 GB/s(9752)        
  /root             1m        2.48 GB/s(2425)         2.65 GB/s(2587)         5.13 GB/s(5012)  
  ```

  - 浙江宁波电信节点

  CPU得分1000分处于正常 Intel E5 水平；内存读写 17 ~ 21GB/s 处于正常 DDR3 或低频单通道 DDR4 内存水平；硬盘 4K 随机读写都为 200MB/s 以上处于正常 NVMe 固态硬盘水平，硬盘的 1M 随机读写总合速度为 5GB/s。性能良好，可考虑作为中转节点。

  ```plaintext
  -------------------------------------VPS融合怪测试-------------------------------------
  版本：v0.1.85
  测评频道: https://t.me/vps_reviews
  Go项目地址：https://github.com/oneclickvirt/ecs
  Shell项目地址：https://github.com/spiritLHLS/ecs
  --------------------------------------系统基础信息--------------------------------------
   CPU 型号            : Intel(R) Xeon(R) Platinum 8272CL CPU @ 2.60GHz
   CPU 数量            : 2 Virtual CPU(s)
   CPU 缓存            : L1: 128 KB / L2: 8 MB / L3: 16 MB
   AES-NI              : ✔️ Enabled
   VM-x/AMD-V/Hyper-V  : ✔️ Enabled
   内存                : 1.14 GB / 3.83 GB
   气球驱动            : ✔️ Enabled
   内核页合并          : ❌ Undetected
   虚拟内存 Swap       : 0.51 MB / 8.00 GB
   硬盘空间            : 14.92 GB / 29.42 GB [50.7%%] /dev/sda1 - /
   启动盘路径          : /dev/sda1
   系统                : debian 13.0 [x86_64] 
   内核                : 6.12.38+deb13-amd64
   系统在线时间        : 1 days, 09 hours, 26 minutes
   时区                : CST
   负载                : 0.00 / 0.00 / 0.00
   虚拟化架构          : KVM
   NAT类型             : Full Cone
   TCP加速方式         : bbr
   IPV4 ASN            : AS136188 Ningbo Zhuo Zhi Innovation Network Technology Co., Ltd
   IPV4 Location       : Ningbo / Zhejiang / CN
   IPV4 Active IPs     : 212/256 (subnet /24) 7786/16384 (prefix /18)
  --------------------------------CPU测试-通过sysbench测试--------------------------------
  1 线程测试(单核)得分:   1081.20
  2 线程测试(多核)得分:   2145.45
  --------------------------------内存测试-通过sysbench测试---------------------------------
  单线程顺序写速度: 17691.70 MB/s(18.55K IOPS, 5s)
  单线程顺序读速度: 21815.29 MB/s(22.87K IOPS, 5s)
  -----------------------------------硬盘测试-通过fio测试-----------------------------------
  测试路径         块大小       读测试(IOPS)            写测试(IOPS)            总和(IOPS)            
  /root             4k        223.19 MB/s(55.8k)      223.78 MB/s(55.9k)      446.96 MB/s(111.7k)    
  /root             64k       1.11 GB/s(17.4k)        1.12 GB/s(17.5k)        2.23 GB/s(34.9k)       
  /root             512k      1.16 GB/s(2270)         1.22 GB/s(2391)         2.39 GB/s(4661)        
  /root             1m        1.23 GB/s(1199)         1.31 GB/s(1279)         2.54 GB/s(2478)        
  ```

综合连续 Ping 测试、带宽测试与服务器综合测试，此处弃用有较高丢包率的湖北襄阳电信节点，而使用浙江宁波电信节点作为中转服务器。

## 异地组网测试

### 异地网络连通性测试

可通过测试异地设备间的延迟来判断网络间的连通性。这里选取未安装 EasyTier 代理网关的设备进行测试。使用各自局域网中的非网关设备相互进行连续 Ping 测试，实测丢包率约为 0%，往返时延平均约为 90ms，时延抖动平均约为 8ms。网络时延性能良好。

![2025-8-30_02-38-15_aga6zwrulw.png](https://cdn.tsanfer.com/image/2025-8-30_02-38-15_aga6zwrulw.png "本地网段设备连续 ping 异地网段设备")​​

![2025-8-30_02-38-53_alric0x0fr.png](https://cdn.tsanfer.com/image/2025-8-30_02-38-53_alric0x0fr.png "异地网段设备连续 ping 本地网段设备")​​

### 中转带宽测试

中转带宽测试通过 iperf3 进行，软件安装步骤参考本节的“本地集群网络测试”部分，`iperf3 -c <IP>` 表示上传数据到指定 IP 的 iperf3 服务端，下面给出测试结果。由于笔者的本地与异地网络环境受限于运营商与校园网的 `上传带宽限制`，所以带宽较小。实测本地到异地的平均带宽约为 19Mbps，异地到本地的平均带宽约为 10Mbps。此种带宽白天可实时同步日志、代码仓库和监控指标，夜里错峰跑压缩后的增量备份，还可多开十来路的远程桌面。足以把异地服务器网络变成低成本但可靠的灾备中心。

![2025-8-30_02-31-29_rhwqku5khf.png](https://cdn.tsanfer.com/image/2025-8-30_02-31-29_rhwqku5khf.png "测试本地到异地的带宽（受限于运营商上传带宽）")

![2025-8-30_02-22-19_ksvmi62t59.png](https://cdn.tsanfer.com/image/2025-8-30_02-22-19_ksvmi62t59.png "测试异地到本地的带宽（受限于运营商上传带宽）")

如果能够进一步提升带宽，就有可能消除备份窗口，实现实时热备份和数据库同步。为了满足多样化的需求，目前计划进一步提升中转带宽。考虑到目前的情况，笔者认为最可行的方法是增加两个局域网的上行带宽。具体来说，可以将两地的网络接入方式升级为提供更高上行带宽的 `商用宽带`。如果预算允许，还可以考虑通过 `公网IP宽带` 接入互联网。

据了解，上行 100Mbps 不带公网 IP 的商用宽带年资费大约在 5000 元左右，而提供上下对等 100Mbps 带宽的公网 IP 宽带年资费则在 25000 元左右。需要注意的是，各地的宽带资费可能会有所不同，这里提供的数据仅供参考。

如果预算有限，无法承担更高的宽带费用，我们可以考虑优化网络架构，以减少服务器上行的大带宽业务需求。例如，由于笔者租用的国内 200Mbps 带宽的云服务年费仅为 500 元左右，且每月流量 1TB，可以考虑将部分大带宽服务迁移到这些大带宽服务器上进行处理。此外，还可以将一些静态资源存储在对象存储中，并配合 CDN 加速，以此来进一步减少带宽需求。

‍

这里给出云服务器到局域网，以及云服务器间的中转带宽测试结果，实测都能跑到接近接入带宽的速度。

![2025-9-6_16-51-10_w1wv302k3k.png](https://cdn.tsanfer.com/image/2025-9-6_16-51-10_w1wv302k3k.png "测试云服务器到本地的中转带宽")​​

![2025-9-6_16-51-30_76ztsbfyvm.png](https://cdn.tsanfer.com/image/2025-9-6_16-51-30_76ztsbfyvm.png "测试云服务器之间的中转带宽")

## 内网穿透测试

### 延迟测试

为了评估内网服务通过内网穿透后的性能影响，笔者进行了实际测量。测试方法是直接通过浏览器访问对应网页，并使用浏览器开发者工具中的“网络”标签页来查看网页的加载时间。对比内网直接访问与从公网访问穿透后的网页加载时间。

在测试中，选择了两个具体的内网服务：Proxmox VE 管理后台和 Kuboard 面板。对于 Proxmox VE 管理后台，内网直接访问的加载时间约为 1 秒，而通过公网穿透后访问的加载时间约为 2.5 秒。对于 Kuboard 面板，内网直接访问的加载时间约为 0.4 秒，而通过公网穿透后访问的加载时间约为 1.75 秒。

总体来看，通过`内网穿透`间接访问内网服务，其加载时间大约会`总共增加 1.5 秒`。本地集群到中转服务器的往返延迟约为 0.45 秒，访问网页设备到中转服务器的往返延迟也约为 0.45 秒。在此之外 `Frp` 的造成的延迟大约在 `0.5 秒` 左右。

值得注意的是，当前的内网穿透仅对 TCP 协议进行了穿透。如果对 HTTP 协议进行穿透，由于 HTTP 协议支持并发请求资源，可能会进一步降低加载时间。

![2025-9-17_19-41-34.png](https://cdn.tsanfer.com/image/2025-9-17_19-41-34.png "内网直接访问 PVE 后台的加载速度")

![2025-9-17_19-46-33.png](https://cdn.tsanfer.com/image/2025-9-17_19-46-33.png "从公网访问穿透后的 PVE 后台的加载速度")

![2025-9-17_19-56-35.png](https://cdn.tsanfer.com/image/2025-9-17_19-56-35.png "内网直接访问 Kuboard 面板的加载速度")

![2025-9-17_20-00-08.png](https://cdn.tsanfer.com/image/2025-9-17_20-00-08.png "从公网访问穿透后的 Kuboard 后台的加载速度")
