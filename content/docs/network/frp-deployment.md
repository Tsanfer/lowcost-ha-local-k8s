---
draft: false
title: '内网穿透工具 frp 部署'
weight: 3
---

参考链接：

[【frp安装使用教程】在openwrt上实现内网穿透，并配置外网IP映射(Debian/Ubuntu) - 刘刻舟的博客](https://blog.lkz.pub/archives/1699866480203)

可以参考笔者的文章：

[使用 Frp 和 Docker 通过远程桌面和 SSH 来远程控制 Windows（反向代理） | Tsanfer&apos;s Blog](https://tsanfer.com/views/Tool/Frp_Docker_SSH_RDP.html)

---

frp 采用 C/S 模式，将服务端部署在具有公网 IP 的机器上，客户端部署在内网或防火墙内的机器上，通过访问暴露在服务器上的端口，反向代理到处于内网的服务。 在此基础上，frp 支持 TCP, UDP, HTTP, HTTPS 等多种协议，提供了加密、压缩，身份认证，代理限速，负载均衡等众多能力。

云服务器部署服务端 frps，本地 OpenWrt 路由器部署客户端 frpc

## 云端部署 frps 服务端

Frps 的 docker-compose 文件

​`./docker-compose.yml`​

```yaml
services:
  frps:
    image: snowdreamtech/frps:0.64.0
    container_name: frps
    restart: always
    network_mode: host
    volumes:
      - ./data/frps.toml:/etc/frp/frps.toml
      - ./data/ssl:/etc/frp/ssl
```

​`./data/frps.toml`​

```toml
bindAddr = "0.0.0.0"
bindPort = 7000

auth.method = "token"
auth.token = "token123456"

webServer.addr = "0.0.0.0"
webServer.port = 7500
webServer.user = "admin"
webServer.password = "admin123"
```

## 本地 OpenWrt 部署 frpc 客户端

在 OpenWrt 面板中安装 `luci-i18n-frpc-zh-cn`​

设置：

- 服务器地址：`proxy.tsanfer.com`​
- 服务器端口：`7000`​
- 令牌：`token123456`​
- TCP 多路复用：✅
- TLS：✅

- 代理设置示例：

  | 代理名称               | 代理类型 | 加密 | 本地 IP      | 本地端口 | 远程端口 |
  | ------------------------ | ---------- | ------ | -------------- | ---------- | ---------- |
  | Kuboard                | tcp      | ✅   | 192.168.0.31 | 8000     | 8000     |
  | Pigsty_Grafana         | tcp      | ✅   | g.pigsty     | 3000     | 8003     |
  | Proxmox\_VE\_web | tcp      | ✅   | 192.168.0.2  | 8006     | 8006     |

## 设置反向代理

最后在中转服务器中设置反向代理即可，例如笔者将 Proxmox VE 穿透到云服务器的 8006 端口，入口设置为 `pve.panel.tsanfer.com` 域名。而将穿透后的 Kuboard 的 8000 端口，入口设置为 `kuboard.panel.tsanfer.com` 域名。

![2025-9-17_19-47-59.png](https://cdn.tsanfer.com/image/2025-9-17_19-47-59.png "从公网访问内网 Kuboard 面板")
