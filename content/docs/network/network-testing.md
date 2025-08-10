---
draft: false
title: '（待补全）网络测试'
weight: 3
---


{{< callout emoji="🚧" >}}
  此页内容待补全
{{< /callout >}}

### 使用 iperf3 进行网络测速

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
sudo apt install iperf3

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
