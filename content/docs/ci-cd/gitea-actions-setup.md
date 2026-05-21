---
draft: false
title: '配置 CI/CD 系统 Gitea Actions'
weight: 2
---

### 安装 Gitea Act Runner

> 在 CI/CD 节点安装 Gitea Act Runer 容器

- 获取注册令牌：

  📌 位置：管理后台 → 工作流 → 运行器 → 创建新运行器

  复制 Registration Token
- Runner 标签：`ubuntu-latest:docker://docker.io/gitea/runner-images:ubuntu-latest,ubuntu-24.04:docker://docker.io/gitea/runner-images:ubuntu-24.04,ubuntu-22.04:docker://docker.io/gitea/runner-images:ubuntu-22.04`

文件位置：`/root/gitea/act_runner/docker-compose.yml`

```yaml
services:
  runner:
    image: gitea/act_runner:0.3.0
    ports:
      - 8088:8088
    environment:
      CONFIG_FILE: /config.yaml
      GITEA_INSTANCE_URL: "${INSTANCE_URL}"
      GITEA_RUNNER_REGISTRATION_TOKEN: "${REGISTRATION_TOKEN}"
      GITEA_RUNNER_NAME: "${RUNNER_NAME}"
      GITEA_RUNNER_LABELS: "${RUNNER_LABELS}"
    volumes:
      - ./config.yaml:/config.yaml
      - ./data:/data
      - /var/run/docker.sock:/var/run/docker.sock
      # 👇 新增这一行：把宿主机的 docker 配置映射进去，继承镜像仓库登陆状态
      - ${HOME}/.docker:/root/.docker
```

文件位置：`/root/gitea/act_runner/.env`

```ini
INSTANCE_URL=http://你的Gitea地址:3000
REGISTRATION_TOKEN=你从Gitea后台获取的Token
RUNNER_NAME=
RUNNER_LABELS=ubuntu-latest:docker://docker.io/gitea/runner-images:ubuntu-latest,ubuntu-24.04:docker://docker.io/gitea/runner-images:ubuntu-24.04,ubuntu-22.04:docker://docker.io/gitea/runner-images:ubuntu-22.04
```

运行 Act Runner

> 可在宿主机中先登陆加速镜像仓库

```bash
RUNNER_VERSION="0.3.0"
REGISTRY_DOMAIN="docker.xuanyuan.run"
GITHUB_MIRROR_URL="http://192.168.0.62:3000"

(
cd /root/gitea/act_runner
# 运行一次生成命令，把配置输出到当前目录的 config.yaml
docker run --entrypoint="" --rm -it ${REGISTRY_DOMAIN}/gitea/act_runner:${RUNNER_VERSION} act_runner generate-config > config.yaml
# 设置运行镜像源
sed -i "s|docker.gitea.com|${REGISTRY_DOMAIN}/gitea|g" config.yaml
sed -i "s|github_mirror: ''|github_mirror: '${GITHUB_MIRROR_URL}'|g" config.yaml
sed -i 's/^  host: ""/  host: "192.168.0.62"/' config.yaml
sed -i 's/^  port: 0/  port: 8088/' config.yaml
sudo docker compose up -d && sudo docker compose logs -f
)
```

‍

![1775485939255.png](https://cdn.tsanfer.com/image/2026-04-06_22-32-19_282.png)
