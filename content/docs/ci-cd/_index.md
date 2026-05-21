---
draft: false
title: 'CI/CD 部署'
weight: 7
---

{{< callout emoji="🛠" >}}
  此页内容待完善
{{< /callout >}}

> CI（Continuous Integration，持续集成）：频繁合并，自动测试
>
> CD（Continuous Deployment，持续部署）：自动构建，自动部署

> CI/CD 单独使用一个虚拟机部署，操作系统为 Debian 13

容器镜像仓库：Harbor

代码托管：Gitea

CI/CD：Gitea Actions

介绍以下内容，搭建 CI/CD：

{{< cards >}}
  {{< card icon="" title="部署自托管 Gitea 代码仓库" link="./gitea-setup" >}}
  {{< card icon="" title="配置 CI/CD 系统 Gitea Actions" link="./gitea-actions-setup" >}}
{{< /cards >}}
