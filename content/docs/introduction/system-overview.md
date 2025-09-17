---
draft: false
title: '（待补全）系统简介'
weight: 1
---

{{< callout emoji="🚧" >}}
  此页内容待补全
{{< /callout >}}

<div align="center">
  <img src="https://cdn.tsanfer.com/image/2025-8-9_16-35-49_vmk57h411s.svg" alt="低成本高可用 K8s 私有云落地指南：从硬件规划到系统上线">
</div>

<div style="display:flex;justify-content:center;gap:8px;" class="img-div">

![]()

![Static Badge](https://img.shields.io/badge/%F0%9F%8F%B7%20%E4%BD%8E%E6%88%90%E6%9C%AC-brightgreen?style=for-the-badge)

![Static Badge](https://img.shields.io/badge/%F0%9F%94%84%20%E9%AB%98%E5%8F%AF%E7%94%A8-blue?style=for-the-badge)

![Static Badge](https://img.shields.io/badge/%F0%9F%A7%A9%20%E5%8F%AF%E6%89%A9%E5%B1%95-yellow?style=for-the-badge)

![Static Badge](https://img.shields.io/badge/%F0%9F%8F%9B%20%E8%87%AA%E4%B8%BB%E5%8F%AF%E6%8E%A7-red?style=for-the-badge)

![Static Badge](https://img.shields.io/badge/%F0%9F%9B%A1%EF%B8%8F%20%E5%AE%89%E5%85%A8%E6%80%A7-grey?style=for-the-badge)

![Static Badge](https://img.shields.io/badge/%E2%99%BB%EF%B8%8F%20%E7%BB%BF%E8%89%B2%E5%8F%AF%E6%8C%81%E7%BB%AD-green?style=for-the-badge)

</div>

<div style="display:flex;justify-content:center;gap:8px;" class="img-div">

![]()

![Static Badge](https://img.shields.io/badge/OpenWrt-%2300B5E2?style=flat-square&logo=OpenWrt&logoColor=white)

![Static Badge](https://img.shields.io/badge/Proxmox-%23E57000?style=flat-square&logo=proxmox&logoColor=white)

![Static Badge](https://img.shields.io/badge/Ceph-%23EF5C55?style=flat-square&logo=Ceph&logoColor=white)

![Static Badge](https://img.shields.io/badge/EasyTier-%236A9CFC?style=flat-square&logoColor=white)

![Static Badge](https://img.shields.io/badge/Kubernetes-%23326CE5?style=flat-square&logo=Kubernetes&logoColor=white)

![Static Badge](https://img.shields.io/badge/PostgreSQL-%234169E1?style=flat-square&logo=PostgreSQL&logoColor=white)

![Static Badge](https://img.shields.io/badge/Hugo-%23FF4088?style=flat-square&logo=Hugo&logoColor=white)
</div>

<div style="display:flex;justify-content:center;gap:8px;" class="img-div">

![]()

![Static Badge](https://img.shields.io/badge/%F0%9F%9B%A0%EF%B8%8F%20%E8%87%AA%E5%BB%BA%E6%9C%8D%E5%8A%A1%E5%99%A8-grey?style=flat-square)

![Static Badge](https://img.shields.io/badge/%F0%9F%93%8C%20%E7%A1%AC%E4%BB%B6%E5%86%8D%E5%88%A9%E7%94%A8-grey?style=flat-square)

![Static Badge](https://img.shields.io/badge/%F0%9F%94%8D%20%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%B5%8B%E8%AF%95-grey?style=flat-square&logoColor=white)

![Static Badge](https://img.shields.io/badge/%F0%9F%94%97%20%E5%BC%82%E5%9C%B0%E7%BB%84%E7%BD%91-blue?style=flat-square)

![Static Badge](https://img.shields.io/badge/%F0%9F%94%97%20%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F-blue?style=flat-square)

![Static Badge](https://img.shields.io/badge/%F0%9F%9B%A1%EF%B8%8F%20%E4%B8%BB%E4%BB%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E7%89%A9%E7%90%86%E9%9A%94%E7%A6%BB-grey?style=flat-square)
</div>

<!-- <div style="display:flex;justify-content:center;gap:8px;" align="center">
  <img src="https://img.shields.io/badge/%F0%9F%8F%B7%20%E4%BD%8E%E6%88%90%E6%9C%AC-brightgreen?style=for-the-badge" alt="Static Badge">
  <img src="https://img.shields.io/badge/%F0%9F%94%84%20%E9%AB%98%E5%8F%AF%E7%94%A8-blue?style=for-the-badge" alt="Static Badge">
  <img src="https://img.shields.io/badge/%F0%9F%A7%A9%20%E5%8F%AF%E6%89%A9%E5%B1%95-yellow?style=for-the-badge" alt="Static Badge">
  <img src="https://img.shields.io/badge/%F0%9F%8F%9B%20%E8%87%AA%E4%B8%BB%E5%8F%AF%E6%8E%A7-red?style=for-the-badge" alt="Static Badge">
  <img src="https://img.shields.io/badge/%F0%9F%9B%A1%EF%B8%8F%20%E5%AE%89%E5%85%A8%E6%80%A7-grey?style=for-the-badge" alt="Static Badge">
  <img src="https://img.shields.io/badge/%E2%99%BB%EF%B8%8F%20%E7%BB%BF%E8%89%B2%E5%8F%AF%E6%8C%81%E7%BB%AD-green?style=for-the-badge" alt="Static Badge">
</div>

<div style="display:flex;justify-content:center;gap:8px;" align="center">
  <img src="https://img.shields.io/badge/OpenWrt-%2300B5E2?style=flat-square&logo=OpenWrt&logoColor=white" alt="Static Badge">
  <img src="https://img.shields.io/badge/Proxmox-%23E57000?style=flat-square&logo=proxmox&logoColor=white" alt="Static Badge">
  <img src="https://img.shields.io/badge/Ceph-%23EF5C55?style=flat-square&logo=Ceph&logoColor=white" alt="Static Badge">
  <img src="https://img.shields.io/badge/EasyTier-%236A9CFC?style=flat-square&logoColor=white" alt="Static Badge">
  <img src="https://img.shields.io/badge/Kubernetes-%23326CE5?style=flat-square&logo=Kubernetes&logoColor=white" alt="Static Badge">
  <img src="https://img.shields.io/badge/PostgreSQL-%234169E1?style=flat-square&logo=PostgreSQL&logoColor=white" alt="Static Badge">
  <img src="https://img.shields.io/badge/Hugo-%23FF4088?style=flat-square&logo=Hugo&logoColor=white" alt="Static Badge">
</div>

<div style="display:flex;justify-content:center;gap:8px;" align="center">
  <img src="https://img.shields.io/badge/%F0%9F%9B%A0%EF%B8%8F%20%E8%87%AA%E5%BB%BA%E6%9C%8D%E5%8A%A1%E5%99%A8-grey?style=flat-square" alt="Static Badge">
  <img src="https://img.shields.io/badge/%F0%9F%93%8C%20%E7%A1%AC%E4%BB%B6%E5%86%8D%E5%88%A9%E7%94%A8-grey?style=flat-square" alt="Static Badge">
  <img src="https://img.shields.io/badge/%F0%9F%94%8D%20%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%B5%8B%E8%AF%95-grey?style=flat-square&logoColor=white" alt="Static Badge">
  <img src="https://img.shields.io/badge/%F0%9F%94%97%20%E5%BC%82%E5%9C%B0%E7%BB%84%E7%BD%91-blue?style=flat-square" alt="Static Badge">
  <img src="https://img.shields.io/badge/%F0%9F%94%97%20%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F-blue?style=flat-square" alt="Static Badge">
  <img src="https://img.shields.io/badge/%F0%9F%9B%A1%EF%B8%8F%20%E4%B8%BB%E4%BB%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E7%89%A9%E7%90%86%E9%9A%94%E7%A6%BB-grey?style=flat-square" alt="Static Badge">
</div> -->

---

本教程旨在探索如何以较低的成本搭建一个可扩展、高可用且自主可控的本地 Kubernetes 集群，并详细记录从零开始的软硬件搭建和部署过程。内容涵盖硬件选型与搭建、服务器性能及稳定性、路由器刷 OpenWrt 实现异地组网，以及部署 PVE 超融合 Ceph 集群、搭建高可用主从 PostgreSQL 数据库，最终部署高可用 Kubernetes 集群，并通过内网穿透技术让其他用户可以直接通过公网访问本地集群。目前，本教程仍在建设中，后续软件部分尚未完成。未来计划加入 CI/CD、可观测性、基础网络安全等 Kubernetes 集群配套功能。

## 在线查看

|链接|展示|备注|
| ---------------------| ------------------------------------------------------------------------------------| --------------------------------------------------------------|
|[📖 指南文档](https://docs.lowcost-k8s-guide.tsanfer.com/)|![2025-9-17_20-36-44.png](https://cdn.tsanfer.com/image/2025-9-17_20-36-44.png)||
|[🖥️ 设备实物](https://docs.lowcost-k8s-guide.tsanfer.com/docs/hardware/hardware-assembly-process/#%E6%89%80%E6%9C%89%E8%AE%BE%E5%A4%87%E6%90%AD%E5%BB%BA%E5%AE%8C%E6%88%90%E6%95%88%E6%9E%9C)|![](https://cdn.tsanfer.com/image/2025-7-23_16-07-40_fz2bio8xgo.jpg)||
|[🖥️ Proxmox VE 虚拟化后台](https://pve.panel.tsanfer.com)|![2025-9-17_20-56-16.png](https://cdn.tsanfer.com/image/2025-9-17_20-56-16.png)|用户名：`viewer`​<br />密码：`viewer123`​<br />领域：Proxmox VE authentication server|
|[🎛️ Kuboard 集群管理面板](https://kuboard.panel.tsanfer.com/)|![2025-9-17_20-55-33.png](https://cdn.tsanfer.com/image/2025-9-17_20-55-33.png)|用户名：`viewer`​<br />密码：`Viewer123`​|
|[🗺️ Pigsty 高可用数据库监控](https://database.k8s-local.monitor.tsanfer.com/)|![2025-9-17_20-54-38.png](https://cdn.tsanfer.com/image/2025-9-17_20-54-38.png)||
|[🔗 Frp 内网穿透监控](https://frp.k8s-local.monitor.tsanfer.com/static/)|![2025-9-17_20-53-51.png](https://cdn.tsanfer.com/image/2025-9-17_20-53-51.png)|用户名：`admin`​<br />密码：`admin123`​|
|[📊 服务监控](https://k8s-local.monitor.tsanfer.com/)|![2025-9-17_20-57-52.png](https://cdn.tsanfer.com/image/2025-9-17_20-57-52.png)||

## 系统整体架构图

![2025-8-31_06-22-16_hjnvhn5mnm.png](https://cdn.tsanfer.com/image/2025-8-31_06-22-16_hjnvhn5mnm.png "系统整体架构图")

## 功能特性

### 🏷 低成本

- 本地物理机
- 低功耗
- 硬件通用，部件损坏后更换方便
- 低门槛搭建最小集群，可随需求逐步扩展
- 内网穿透（节约流量成本）
- OSS与CDN（节约流量成本）

### 🔄 高可用

- Kubernetes 集群高可用
- 主从 PostgreSQL 数据库高可用
- 分布式 Ceph 存储高可用

### 🧩 可扩展

- 硬件可扩展

  - 硬件资源可增删
  - 异地资源可增删（异地组网）
  - 支持硬件架构的灵活调整，以适应不同复杂度的需求
- 软件可扩展

  - 虚拟机配置可伸缩
  - 虚拟机数量可增删
  - 功能节点可增删
  - 支持软件架构的灵活调整，以适应不同复杂度的需求

### 🏛 自主可控

- 机器本地部署可控
- 硬件通用，替换方便
- 软件开源，减小风险
- 不绑定软硬件服务提供商

### 🛡️ 安全性

- 每个数据库节点独立部署，确保物理隔离
- 数据库高可用
- 数据库时间点恢复
- PVE 虚拟机快照

- 任意节点或磁盘故障零数据丢失

### ♻️ 绿色可持续

- 低功耗、低噪音
- 硬件通用，故障易更换维修
- 旧有硬件循环利用，以求物尽其用
- 小巧灵活，移动方便


> 置顶图片中左侧图标来自 [freepik](https://www.freepik.com/icon/database_13551626)，右侧图标来自 [Kubernetes](https://github.com/kubernetes/kubernetes/blob/master/logo/logo.svg)

{{< callout emoji="🛠" >}}
  此页内容待完善
{{< /callout >}}

> 《垃圾佬也要组集群！从零开始的低成本 K8s 集群搭建教程 —— 关于我用洋垃圾组集群这档事》

> 《垃圾佬也要组集群！低成本高可用 K8s 教程｜1.1 系统简介》