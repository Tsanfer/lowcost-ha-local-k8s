---
draft: false
title: '硬件设备'
weight: 2
---

> 若高可用与备份需求不强，可选择双路 CPU 主板，将整个系统的大部分节点集成入一台设备中。

## 超融合集群节点1设备

### CPU

英特尔至强处理器 E5-2695 v4，18核36线程，14纳米工艺，主频 2.10GHz，单核睿频 3.30GHz，全核睿频 2.4GHz，TDP 120W，实测 CPU 满载功耗 80W 左右，LGA 2011-3 封装可用于 X99 平台主板。

![2025-7-22_23-24-27_kps58i0hf9.jpg](https://cdn.tsanfer.com/image/2025-7-22_23-24-27_kps58i0hf9.jpg "英特尔至强处理器 E5-2695 v4")

### 内存

四根 32GB DDR4 2133MHz RECC 内存，一共 128GB 内存

![2025-7-23_15-10-15_sxiq6ow0ck.jpg](https://cdn.tsanfer.com/image/2025-7-23_15-10-15_sxiq6ow0ck.jpg "四根 32GB DDR4 2133MHz RECC 内存")

### 硬盘

全闪存储，系统盘 64GB SATA 固态硬盘，数据盘 1TB NVMe 固态硬盘。若此处使用全新出厂的固态硬盘，则需核对硬盘外包装、硬盘标签以及软件 S.M.A.R.T. 记录的 S/N 序列号是否一致，以确保硬盘为全新硬盘。

![2025-7-22_23-27-56_lbry5l3z0y.jpg](https://cdn.tsanfer.com/image/2025-7-22_23-27-56_lbry5l3z0y.jpg "64GB SATA 固态硬盘")

![2025-7-23_14-43-20_s13rusmzfk.jpg](https://cdn.tsanfer.com/image/2025-7-23_14-43-20_s13rusmzfk.jpg "1TB NVMe 固态硬盘")

### 主板

英特尔 X99/C612 芯片组主板，LGA 2011-3 CPU插槽，M-ATX 尺寸，四个 DDR4 DIMM 内存插槽，支持四通道内存，四倍八相供电，PCIe 3.0 x16/x4/x1 插槽各一个，两个使用 PCIe 3.0 x4 总线的 M.2 插槽，六个 SATA 3.0 插槽。

参考链接：

[【PCEVA】彻底讲透主板供电——原理篇_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Qt411h7Jy)

[【PCEVA】彻底讲透主板供电——实战篇_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Qt411b7Ea)

[【硬核科普】第18期：简单聊聊主板供电相数怎么看 - 知乎](https://zhuanlan.zhihu.com/p/157891361)

[关于主板供电瓦数的疑问 - 电脑讨论(新) - Chiphell - 分享与交流用户体验](https://www.chiphell.com/thread-2648944-1-1.html)

[请问主板供电怎么看支持多少瓦 - 电脑讨论(新) - Chiphell - 分享与交流用户体验](https://www.chiphell.com/thread-2639197-1-1.html)

鉴于主板 CPU 供电问题的复杂性，目前未有公认的简单有效评估主板 CPU 供电性能的方法。绝大多数情况需要通过实际测试才能得出主板的 CPU 供电能力，并且还需要结合具体的使用场景，此处暂且以经验方法估算主板大概支持的 CPU 功耗。

评估主板的 CPU 供电能力，可以通过实际为 CPU 供电的 PWM 芯片输出的总相数估算。每相输出提供 30W 的供电能力，后端电路如有倍相，则乘以1.5的倍率。以此处的四倍八相供电为例，PWM 芯片有四相输出则有 120W 的供电能力，然后又通过倍相使其变为了八相供电，所以还要乘以1.5的倍率，最后估算此主板的CPU 供电能力在 180W 左右。实测能支持 TDP 120W 的 CPU 满载运行于 80W 左右的功耗。

这里还要指出的是，CPU 的实际运行功耗除了取决于主板的供电能力之外，还要考虑 CPU 的工作频率、电压、温度、指令集以及操作系统和电源管理策略等因素。

![2025-7-22_23-19-45_h8ynz8y4xi.jpg](https://cdn.tsanfer.com/image/2025-7-22_23-19-45_h8ynz8y4xi.jpg "英特尔 X99 主板")

![2025-7-23_00-20-09_xnh6wguj3g.jpg](https://cdn.tsanfer.com/image/2025-7-23_00-20-09_xnh6wguj3g.jpg "英特尔 X99 主板 - 四通道内存")

![2025-7-23_00-19-23_6y93k5h2xb.jpg](https://cdn.tsanfer.com/image/2025-7-23_00-19-23_6y93k5h2xb.jpg "英特尔 X99 主板 - 四倍八相供电")

![2025-7-23_14-45-05_7syi9jnvrh.jpg](https://cdn.tsanfer.com/image/2025-7-23_14-45-05_7syi9jnvrh.jpg "英特尔 X99 主板 - PCIe 3.0 x16/x4/x1 插槽")

![2025-7-23_15-40-19_1357ky3p7u.jpg](https://cdn.tsanfer.com/image/2025-7-23_15-40-19_1357ky3p7u.jpg "英特尔 X99 主板 - 两个使用 PCIe 3.0 x4 总线的 M.2 插槽")

![2025-7-23_00-32-43_isryqawr0u.jpg](https://cdn.tsanfer.com/image/2025-7-23_00-32-43_isryqawr0u.jpg "英特尔 X99 主板 - 六个 SATA 3.0 插槽")

### 电源

全模组 80PLUS 金牌电源，额定功率 600W。供电接口有一个主板供电、一个 CPU 供电、两个 PCIe 供电、三个 SATA 供电

![2025-7-22_23-11-56_ge4mxs9t6v.jpg](https://cdn.tsanfer.com/image/2025-7-22_23-11-56_ge4mxs9t6v.jpg "额定600W 80 PLUS 金牌电源")

![2025-7-26_16-07-28_vgvnh9m6jw.jpg](https://cdn.tsanfer.com/image/2025-7-26_16-07-28_vgvnh9m6jw.jpg "模组电源供电输出")

> 尝试使用一般电源，但电源转换效率不佳，弃用
>
> ![2025-7-22_23-11-30_jcior0pc93.jpg](https://cdn.tsanfer.com/image/2025-7-22_23-11-30_jcior0pc93.jpg "一般电源")

### 散热风扇

塔式散热器，双塔，六铜管，三风扇

![2025-7-9_19-14-07_8s4vjrf6xj.jpg](https://cdn.tsanfer.com/image/2025-7-9_19-14-07_8s4vjrf6xj.jpg "塔式散热器")

![IMG_20250701_213135](https://cdn.tsanfer.com/image/2025-8-10_21-37-00_pt8qzp107o.png "塔式散热器-底部")

> 尝试使用下压式散热器，但散热效果不佳，弃用
>
> ![2025-7-22_22-58-29_4cstrbc8li.jpg](https://cdn.tsanfer.com/image/2025-7-22_22-58-29_4cstrbc8li.jpg "下压式散热器")

### 显卡

G100系列显卡，仅用于显示输出，后续在 BIOS 中开启 Fast Boot 后可拆除此显卡

![2025-7-22_23-33-17_qfnec7g9rd.jpg](https://cdn.tsanfer.com/image/2025-7-22_23-33-17_qfnec7g9rd.jpg "G100系列显卡")

## 超融合集群节点2、3设备

### 准系统

CPU 为英特尔赛扬处理器 J1900，2核4线程，主频 2.00 GHz，睿频 2.42 GHz，TDP10W

![2025-7-22_23-46-16_w5r8g82ika.jpg](https://cdn.tsanfer.com/image/2025-7-22_23-46-16_w5r8g82ika.jpg "瘦客户端准系统")

### 内存

8GB DDR3L 1333MHz 内存

![2025-7-22_23-51-42_avj7hy4njv.jpg](https://cdn.tsanfer.com/image/2025-7-22_23-51-42_avj7hy4njv.jpg "8GB DDR3L 1333MHz 内存")

### 硬盘

全闪存储，系统盘 64GB mSATA 固态硬盘，数据盘 1TB SATA 固态硬盘

![IMG_20250628_213829](assets/IMG_20250628_213829-20250722234006-9ojs3qg.jpg "64GB mSATA 固态硬盘")

![2025-7-23_14-47-00_xpx6mi3bp2.jpg](https://cdn.tsanfer.com/image/2025-7-23_14-47-00_xpx6mi3bp2.jpg "1TB SATA 固态硬盘")

## 数据库设备

### 准系统

与超融合集群节点2、3规格一致

### 内存

与超融合集群节点2、3规格一致

### 硬盘

全闪存储，128GB mSATA 固态硬盘

![2025-7-23_00-29-26_q1drr7p9z9.jpg](https://cdn.tsanfer.com/image/2025-7-23_00-29-26_q1drr7p9z9.jpg "128GB mSATA 固态硬盘")

## 网络设备

### 路由器

双频千兆路由器，一个 WAN 口，两个 LAN 口，一个 USB，128MB Flash 闪存，256MB DDR3 内存

![2025-7-23_00-21-39_pkx0kts6a4.jpg](https://cdn.tsanfer.com/image/2025-7-23_00-21-39_pkx0kts6a4.jpg "千兆路由器")

### 交换机

8口全千兆以太网交换机

![IMG_20250628_184911](https://cdn.tsanfer.com/image/2025-8-10_21-39-17_r930mw35z3.jpg "千兆交换机")

## 其他设备

### 瘦客户端电源

12V 10A 直流电源，使用电源分配线为超融合集群节点2、3和数据库设备供电

![2025-7-22_23-36-39_ock2a75omk.jpg](https://cdn.tsanfer.com/image/2025-7-22_23-36-39_ock2a75omk.jpg "12V 10A 直流电源")

‍
