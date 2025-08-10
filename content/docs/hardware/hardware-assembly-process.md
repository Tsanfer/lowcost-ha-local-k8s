---
draft: false
title: '硬件搭建过程'
weight: 3
---

## 超融合集群节点1搭建过程

参考链接：[合集·三维动画装机教程 - 硬件茶谈](https://space.bilibili.com/14871346/lists/351804)

> 安装之前先洗手，或佩戴防静电手环，以免身体所带静电损坏元件

> 安装时主板下方应铺垫一层泡沫等绝缘缓冲材料，以防止背板针脚间短路

> 安装各个部件的大致顺序为：先安装高度低的、容易被其他设备遮挡的部件，后安装高度高的、容易遮挡其他设备的部件

### CPU 安装

1. 拉起 CPU 插槽旁杠杆，取下插槽上的 CPU 保护盖
2. 将 CPU 对准方向放入 CPU 插槽中，按下杠杆

    ![2025-7-23_16-34-07_pxs0mcibkw.jpg](https://cdn.tsanfer.com/image/2025-7-23_16-34-07_pxs0mcibkw.jpg "CPU 对准方向放入 CPU 插槽中并按下杠杆")

### 内存条安装

参考链接：[【硬核科普】为什么装机内存条推荐安装到第24槽而不是13槽？为什么ITX主板超频内存更容易？_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV19A411a7sC/)

1. 打开内存插槽两边的卡扣，按压内存条两端，将四根 DDR4 内存条插入 DDR4 内存插槽使卡扣回弹即可，内存条代数需与内存插槽一致。

    此款主板内存插槽与内存通道数量相等，故插满内存条即可。若内存通道数小于插槽数，则可考虑将内存条插入不同内存通道所对应的插槽中，且所插入插槽的位置应在远离 CPU 一侧

![2025-7-23_16-39-00_51gdyyylu8.jpg](https://cdn.tsanfer.com/image/2025-7-23_16-39-00_51gdyyylu8.jpg "内存条安装")

### M.2 硬盘安装

1. 由于 NVMe 固态硬盘发热量大，高负载时可能出现掉速、卡顿的现象，为了降低其温度，此处为 NVMe 固态硬盘安装散热片

![2025-8-6_16-58-28_lnok2jc4nd.jpg](https://cdn.tsanfer.com/image/2025-8-6_16-58-28_lnok2jc4nd.jpg "NMVe 硬盘安装散热片")

2. 将硬盘插入主板上的 M.2 插槽，拧上固定螺丝，注意螺丝不要拧太紧防止滑丝，此处主板的 M.2 插槽与硬盘的 M.2 接口外形均为 M-key

![2025-8-6_17-04-38_kfbw37fdys.jpg](https://cdn.tsanfer.com/image/2025-8-6_17-04-38_kfbw37fdys.jpg "NMVE 硬盘插入 M.2 插槽")

### CPU 散热器安装

1. 安装 CPU 散热器扣具

![2025-7-23_16-36-35_v01h0kr5fc.jpg](https://cdn.tsanfer.com/image/2025-7-23_16-36-35_v01h0kr5fc.jpg "安装 CPU 散热器扣具")

2. 在 CPU 表面涂抹散热硅脂

![2025-8-10_21-43-04_csf8nv81lb.jpg](https://cdn.tsanfer.com/image/2025-8-10_21-43-04_csf8nv81lb.jpg "在 CPU 表面涂抹散热硅脂")

1. 确定散热器安装方向，出风口对准主板 I/O 挡板方向，使热风从机箱后置排风口排出。电脑散热器一般正面为进风面，有马达型号和参数的一面为排风面

![2025-7-23_16-49-25_vnqqxdh5db.jpg](https://cdn.tsanfer.com/image/2025-7-23_16-49-25_vnqqxdh5db.jpg "塔式散热器 - 出风口标识")

![2025-7-23_16-50-27_9egdg32beh.jpg](https://cdn.tsanfer.com/image/2025-7-23_16-50-27_9egdg32beh.jpg "塔式散热器 - 风扇方向标识")

4. 安装塔式 CPU 散热器，先撕下散热器底部保护膜，再将散热器底部一侧卡扣卡入 CPU 扣具，最后按压散热器另一侧弹片将另一侧卡扣卡入扣具

![2025-7-23_23-31-53_pu1287bkim.jpg](https://cdn.tsanfer.com/image/2025-7-23_23-31-53_pu1287bkim.jpg "将散热器底部一侧卡扣卡入 CPU 扣具")​​​

![2025-7-23_23-33-37_8di65d463k.jpg](https://cdn.tsanfer.com/image/2025-7-23_23-33-37_8di65d463k.jpg "按压散热器另一侧弹片将另一侧卡扣卡入扣具")

5. 将 CPU 散热器电源线插入 CPU_FAN 供电针脚​​

![2025-8-10_21-44-58_l5ua3g8tml.jpg](https://cdn.tsanfer.com/image/2025-8-10_21-44-58_l5ua3g8tml.jpg "插入 CPU_FAN 供电针脚")

> 如有需要也可同时在机箱上安装散热风扇，风扇固定完成后，将其供电线插入主板上的 SYS_FAN 针脚或连接电源上的 4-pin 供电线即可，风道方向为机箱前面板进风后面板出风，底部进风顶部出风

### 主板安装

1. 安装 I/O 挡板，挡板光滑一侧朝向机箱外部，从机箱内按压挡板使其固定

![2025-7-24_21-37-15_.g8leki9u9.jpg](https://cdn.tsanfer.com/image/2025-7-24_21-37-15_.g8leki9u9.jpg "固定 I/O 挡板")

2. 根据主板尺寸安装在对应位置安装螺丝柱，其作用是固定主板使主板悬空，防止主板背板与机箱接触短路。此处使用的主板尺寸为 M-ATX，所以需在六个对应的位置安装螺丝柱

![2025-7-25_21-08-38_k13a8nqcz4.jpg](https://cdn.tsanfer.com/image/2025-7-25_21-08-38_k13a8nqcz4.jpg "安装主板固定螺丝柱")​​​

3. 将主板对齐 I/O 面板和固定螺丝柱放入机箱，拧上固定螺丝

![2025-7-25_21-10-51_rb2prse2pq.jpg](https://cdn.tsanfer.com/image/2025-7-25_21-10-51_rb2prse2pq.jpg "将主板固定在机箱中")​​​

### SATA 硬盘安装

1. 将 SATA 硬盘固定于机箱中的硬盘位即可，某些情况下可能需要将硬盘安装于硬盘托架中

![2025-7-25_21-25-09_haq3khphxp.jpg](https://cdn.tsanfer.com/image/2025-7-25_21-25-09_haq3khphxp.jpg "在机箱中固定 SATA 硬盘")

### 电源安装

> 模组电源一般与配套的线缆搭配使用，不同型号的线缆不能通用，原因是不同型号的电源接口定义不同，没有形成行业标准

1. 连接供电线缆，模组电源线缆连接需要区分方向，不可拆分端接电源，可拆分端接主板。此处连接 24-pin 电源线缆、8-pin CPU线缆、6-pin 硬盘供电线缆。如果是非模组电源，可跳过此步

![2025-7-25_00-05-58_4bk32jqcgo.jpg](https://cdn.tsanfer.com/image/2025-7-25_00-05-58_4bk32jqcgo.jpg "模组电源连接线缆")

2. 将电源放入机箱对应位置，拧上固定螺丝即可

![2025-7-25_21-36-55_uhqwf8n0hk.jpg](https://cdn.tsanfer.com/image/2025-7-25_21-36-55_uhqwf8n0hk.jpg "在机箱中固定电源")

### 线缆连接

1. 连接主板供电线缆，将电源上的 24-pin 主板供电线插入主板上的 ATXPWR 主板供电插槽

![2025-7-24_19-18-56_al4xxn44yt.jpg](https://cdn.tsanfer.com/image/2025-7-24_19-18-56_al4xxn44yt.jpg "连接主板供电线")

2. 连接 CPU 供电线缆，将电源上的 8-pin CPU供电线插入主板上的 ATX_CPU 供电插槽

![2025-7-24_19-20-27_0qrfn5z1lb.jpg](https://cdn.tsanfer.com/image/2025-7-24_19-20-27_0qrfn5z1lb.jpg "连接 CPU 供电线")

3. 连接 SATA 硬盘数据线缆，用 SATA 数据线连接主板上的 SATA 插槽与硬盘上的 SATA 数据接口

![2025-7-24_17-54-49_r5gq9zkuz7.jpg](https://cdn.tsanfer.com/image/2025-7-24_17-54-49_r5gq9zkuz7.jpg "使用 SATA 数据线连接硬盘与主板")

4. 连接 SATA 硬盘供电线缆，将电源上的 SATA 供电线连接在硬盘电源接口上

![2025-7-24_17-55-04_yyl9mgjics.jpg](https://cdn.tsanfer.com/image/2025-7-24_17-55-04_yyl9mgjics.jpg "使用 SATA 电源线给硬盘供电")

5. 连接前面板音频接口线缆，将线缆插入主板 F_AUDIO 对应针脚

![2025-7-24_18-58-38_y7oofj0l5g.jpg](https://cdn.tsanfer.com/image/2025-7-24_18-58-38_y7oofj0l5g.jpg "连接机箱前面板音频线缆")

6. 连接机箱前面板 USB 接口线缆，将线缆插入主板 F_USB 对应针脚

![2025-7-24_18-57-48_vzjgtgepit.jpg](https://cdn.tsanfer.com/image/2025-7-24_18-57-48_vzjgtgepit.jpg "连接机箱前面板 USB 线缆")

7. 连接机箱前面板开关控制与提示灯线缆，将线缆插入主板 F_PANEL 对应针脚，具体接线方法如下图所示，开机及重启跳线可不分正负极

![](https://bkimg.cdn.bcebos.com/pic/eac4b74543a9822685e8b5d68082b9014a90eb12 "前面板跳线定义（图片来源：百度百科）")​​​

![2025-7-25_22-21-25_50myp5absk.jpg](https://cdn.tsanfer.com/image/2025-7-25_22-21-25_50myp5absk.jpg "机箱前面板开关控制及提示灯线缆")

![2025-7-24_18-59-45_98httt1n5l.jpg](https://cdn.tsanfer.com/image/2025-7-24_18-59-45_98httt1n5l.jpg "连接机箱前面板开关控制及信号灯线缆")

### 显卡安装

1. 取下 PCIe x16 插槽旁边的两块机箱挡板，一块是 PCIe 设备固定横条挡板，另一块是防尘挡板

![2025-7-25_22-46-01_ejy3a0fo3s.jpg](https://cdn.tsanfer.com/image/2025-7-25_22-46-01_ejy3a0fo3s.jpg "取下机箱上的 PCIe 挡板")

2. 打开 PCIe x16 插槽的卡扣，将显卡从插槽中间的正上方插入内并使卡扣回弹，若使用高性能显卡则还需连接电源上的 PCIe 供电线

> 此处应注意显卡的高度是否与机箱的高度匹配，全高显卡只支持全塔机箱、ATX机箱等大尺寸机箱，不支持2U机箱、Mini-ITX机箱等小尺寸机箱。而半高显卡则可以通过更换挡板以适应不同尺寸的机箱。

![2025-7-25_22-52-10_mjs455x7n4.jpg](https://cdn.tsanfer.com/image/2025-7-25_22-52-10_mjs455x7n4.jpg "将显卡插入 PCIe x16 插槽")

3. 固定显卡，先装上 PCIe 设备固定横条挡板，然后用螺丝将显卡固定于 PCIe 固定横条处

![2025-7-25_23-10-34_582yf2ed2q.jpg](https://cdn.tsanfer.com/image/2025-7-25_23-10-34_582yf2ed2q.jpg "固定显卡")

### 安装收尾

此时机箱内部全部已安装完毕，把显示器接到显卡，插上电源线，打开电源开关并按下机箱电源键即可开机。若开机异常，则复查各部件是否安装正确。如果主板上有故障提示灯，则可根据提示排查故障。在调试无误后合上机箱侧板，机器即可正常使用

![2025-7-26_15-10-50_unttp30p6i.jpg](https://cdn.tsanfer.com/image/2025-7-26_15-10-50_unttp30p6i.jpg "超融合集群节点1 - 搭建完成")-

## 超融合集群节点2、3搭建过程

> 超融合集群节点2与超融合集群节点3硬件配置基本相同，搭建步骤一致。

由于此处的瘦客户端设备设备集成了大部分的部件，故主要是完成内存和硬盘的安装。

### 内存安装

1. 将 DDR3L 内存条插入 DDR3L 内存插槽即可

![2025-7-26_15-24-43_v2xo0telaf.jpg](https://cdn.tsanfer.com/image/2025-7-26_15-24-43_v2xo0telaf.jpg "瘦客户端 - 安装内存")

### 硬盘安装

参考链接：[升腾D610加装2.5寸机械硬盘并安装FNOS_硬盘_什么值得买](https://post.smzdm.com/p/akld75o4/)

此处需要安装两块固态硬盘，一块 mSATA 硬盘作系统盘，一块 SATA 硬盘作数据盘

1. 安装系统盘，将 mSATA 固态硬盘插入 mSATA 插槽即可。

![2025-7-26_15-29-46_fuswshzctp.jpg](https://cdn.tsanfer.com/image/2025-7-26_15-29-46_fuswshzctp.jpg "瘦客户端 - 安装 mSATA 硬盘")

2. 这里还需要安装另一块 SATA 硬盘，虽然该瘦客户端预留了 SATA 数据插槽，却没有 SATA 供电口，因此需要自行取电。由于 2.5 英寸 SATA 固态硬盘供电电压仅需 5V，我们就在主板上寻找电压差恒定为 5V 的一对针脚。用万用表测量数据口旁那四根针脚，确认其为两组 5V 直流输出，可使用转接线让其为硬盘供电。

![2025-8-3_16-46-09_ds3ovew0uo.jpg](https://cdn.tsanfer.com/image/2025-8-3_16-46-09_ds3ovew0uo.jpg "SATA 供电针脚地线测量")

![2025-8-3_16-46-56_sa81nykhae.jpg](https://cdn.tsanfer.com/image/2025-8-3_16-46-56_sa81nykhae.jpg "SATA 供电针脚 5V 测量")​​​

3. 根据 SATA 电源连接线定义，取电端中间的线缆电压为 5V，其相邻的线缆均为地线。因此可仅使用中间的 5V 线缆以及与其相邻的 GND 线缆来为硬盘供电，其余三根线缆无需连接。

![2025-7-26_15-53-33_mfyd7fyvfk.png](https://cdn.tsanfer.com/image/2025-7-26_15-53-33_mfyd7fyvfk.png "SATA 电源接口定义（图片来源：Scott Mueller《Upgrading and Repairing PCs》第22版）")

![2025-7-26_17-04-46_669hrfigft.jpg](https://cdn.tsanfer.com/image/2025-7-26_17-04-46_669hrfigft.jpg "排线针脚转 2.5寸 SATA 供电")

4. 将 SATA 电源线取电端插入主板对应供电针脚，注意区分正负极以免损坏硬盘。

![2025-7-26_17-07-44_mr8rxt6qzg.jpg](https://cdn.tsanfer.com/image/2025-7-26_17-07-44_mr8rxt6qzg.jpg "将 SATA 电源转接线插入对应针脚")

5. 将 SATA 硬盘固定于机箱内，并将 SATA 供电线与数据线连接到固态硬盘。

![2025-7-26_17-12-31_9ugzv3bw54.jpg](https://cdn.tsanfer.com/image/2025-7-26_17-12-31_9ugzv3bw54.jpg "固定硬盘并连接硬盘电源与数据线")

### 安装收尾

此时机箱内部已安装完毕，把显示器接到主板视频输出，插上电源线，按下机箱电源键即可开机。若开机异常，则复查各部件是否安装正确。着重检查电脑能否识别加装的 SATA 硬盘，此处可进入电脑的 BIOS 查看硬盘信息。在调试无误后合上机箱侧板，机器即可正常使用。

![2025-7-26_17-34-40_zgqsc8caxg.jpg](https://cdn.tsanfer.com/image/2025-7-26_17-34-40_zgqsc8caxg.jpg "进入 BIOS 查看加装 SATA 硬盘信息")

![2025-7-28_18-23-33_ljkq0epcji.jpg](https://cdn.tsanfer.com/image/2025-7-28_18-23-33_ljkq0epcji.jpg "超融合集群节点2 - 搭建完成")

![2025-7-26_17-21-00_ga3at97sbl.jpg](https://cdn.tsanfer.com/image/2025-7-26_17-21-00_ga3at97sbl.jpg "超融合集群节点3 - 搭建完成")

## 数据库设备搭建过程

数据库设备共有三台，一台主数据库，两台从数据库，硬件规格基本相同，安装步骤基本一致。它们与超融合集群节点2、3的差别仅在硬盘。数据库设备没有外接第二块硬盘，而超融合集群节点2、3外接了一块硬盘。此处仅给出各数据库设备搭建完成后的效果图。

![2025-7-26_19-15-33_ak9s8ln2ab.jpg](https://cdn.tsanfer.com/image/2025-7-26_19-15-33_ak9s8ln2ab.jpg "主数据库设备 - 搭建完成")

![2025-7-28_18-49-08_llcze3qn5n.jpg](https://cdn.tsanfer.com/image/2025-7-28_18-49-08_llcze3qn5n.jpg "从数据库设备1 - 搭建完成")

![2025-8-10_21-47-30_i8xano8yfa.jpg](https://cdn.tsanfer.com/image/2025-8-10_21-47-30_i8xano8yfa.jpg "从数据库设备2 - 搭建完成 - 正面")

![2025-8-10_21-47-46_s5b8ua0txk.jpg](https://cdn.tsanfer.com/image/2025-8-10_21-47-46_s5b8ua0txk.jpg "从数据库设备2 - 搭建完成 - 背面")

## 所有服务器设备搭建完成效果

![2025-7-28_19-38-31_4a1hm9sdkc.jpg](https://cdn.tsanfer.com/image/2025-7-28_19-38-31_4a1hm9sdkc.jpg "所有服务器设备 - 搭建完成 - 正面")​​​

![2025-7-28_19-25-26_qdffzuexbn.jpg](https://cdn.tsanfer.com/image/2025-7-28_19-25-26_qdffzuexbn.jpg "所有服务器设备 - 搭建完成 - 背面")

## 连接所有设备

- 摆放设备

![2025-8-6_11-52-40_x8ykn9zefg.jpg](https://cdn.tsanfer.com/image/2025-8-6_11-52-40_x8ykn9zefg.jpg "摆放所有设备")

- 连接电源

![2025-8-6_11-52-39_y7crgyizne.jpg](https://cdn.tsanfer.com/image/2025-8-6_11-52-39_y7crgyizne.jpg "连接所有设备电源")

- 连接网线

![2025-8-6_11-52-41_3l5a2ew7zn.jpg](https://cdn.tsanfer.com/image/2025-8-6_11-52-41_3l5a2ew7zn.jpg "连接所有设备网线")

## 所有设备搭建完成效果

![2025-7-23_16-07-40_fz2bio8xgo.jpg](https://cdn.tsanfer.com/image/2025-7-23_16-07-40_fz2bio8xgo.jpg "所有设备搭建完成")

‍
