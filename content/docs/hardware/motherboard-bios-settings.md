---
draft: false
title: '主板 BIOS 设置'
weight: 4
---

参考教程：

[BIOS 设置 - 国光的 PVE 环境搭建教程](https://pve.sqlsec.com/2/1/)

[华南 x99 双路主板，实现 PVE 虚拟化和显卡直通 - Hu Zou](https://onela.cn/posts/zh/2024-05-19-%E5%8D%8E%E5%8D%97-x99-%E5%8F%8C%E8%B7%AF%E4%B8%BB%E6%9D%BF%E5%AE%9E%E7%8E%B0-pve-%E8%99%9A%E6%8B%9F%E5%8C%96%E5%92%8C%E6%98%BE%E5%8D%A1%E7%9B%B4%E9%80%9A/)

根据实际需要进行 BIOS 设置，这里以超融合集群节点1的 BIOS 设置为例：

开机后根据主板型号按对应的键盘按键进入 BIOS，此处为 `Delete` 键。

![2025-8-3_18-49-32_o5xo1zn608.png](https://cdn.tsanfer.com/image/2025-8-3_18-49-32_o5xo1zn608.png "超融合集群节点1 BIOS 主页")

BIOS 配置参考华南主板 BIOS 相关设置

> 如需还原 BIOS 设置，可在断开电源的情况下，拆掉主板上的 BIOS 供电电池还原

- 设置硬盘启动方式：

  Advanced - CSM Configuration - `Storage` 设置为 UEFI

  ![2025-8-3_19-04-11_zuzenut60l.png](https://cdn.tsanfer.com/image/2025-8-3_19-04-11_zuzenut60l.png "设置硬盘启动方式")

  如果是老的机器则还应将 Boot option filter 设置为 legacy only 以防止无法进入 UEFI 格式的U盘启动盘
- （可选）开启 Above 4G Decoding（vGPU 方案需要开启这个选项）：

  Advanced - PCI Subsystem settings - `Above 4G Decoding`（部分老显卡可能不支持此功能）
- （可选）开启 SR-IOV 网卡虚拟化技术 （高效先进的虚拟机网卡技术）：

  Advenced - PCI Subsustem Settings - `SR-IOV Support`​
- 开启 Intel VMX 虚拟化技术（Intel VT-x，允许虚拟机运行）：

  IntelRCSetup - Processor Configuration - `VMX`​

  ![2025-8-6_14-06-22_jzs3dmc9pj.png](https://cdn.tsanfer.com/image/2025-8-6_14-06-22_jzs3dmc9pj.png "开启 Intel VT-x 虚拟化")​​
- （可选）开启 x2APIC（中断控制器，提升多核效率）：

  IntelRCSetup - Processor Configuration - `X2APIC`（同时关闭 X2APIC\_OPT\_OUT Flag）
- （可选）开启 Numa （多路 CPU 建议开启，提高多路 CPU 运行效率，合理分配负载）：

  IntelRCSetup - Common RefCode Configuration - `Numa`​
- （可选）开启 VT-d （PCIe 件直通必须） :

  IntelRCSetup - IIO Configuration - `Intel VT for Directed i/o (VT-d)`​
- 开启来电启动：

  IntelRCSetup - PCH Configuration - PCH Devices - `PCH state after G3` 设置为 On

  ![2025-8-3_19-04-36_zh1nm0tgth.png](https://cdn.tsanfer.com/image/2025-8-3_19-04-36_zh1nm0tgth.png "开启来电自启")
- （可选）开启 SATA 硬盘热插拔：

  InterRCSetup - PCH Configuration - PCH SATA Configuration - SATA Port - `Hot Plug`​
- （可选）开启 Fast Boot 跳过设备自检（开启后，可不插显卡进入系统）：

  Boot - `Fast Boot`​
- 调整引导设备顺序：

  Boot - Boot Option Priorities

​

如果 BIOS 的启动设备类型有 NC（Network Controller），其为从板载网卡进行网络启动，如果不需要应该关闭，避免从网卡引导系统

‍
