---
draft: false
title: '使用 Hugo 和 Github Pages 生成并部署静态站点'
weight: 1
---

{{< callout emoji="🛠" >}}
  此页内容待完善
{{< /callout >}}


## 生成 Hugo 及 Hextra 配置文件

参考链接：

[Hextra](https://imfing.github.io/hextra/zh-cn/)

[Hugo Documentation](https://gohugo.io/documentation/)

此处使用 Hugo 的 Hextra 主题来构建项目

- 依赖

  需要先安装以下软件：

  - [Hugo（扩展版）](https://gohugo.io/installation/)
  - [Git](https://git-scm.com/)
  - [Go](https://go.dev/)

- 初始化 Hugo 站点

```bash
hugo new site lowcost-ha-local-k8s --format=yaml
```

- 使用 Hugo 模块初始化 Hextra 主题

```bash

# 初始化 Hugo 模块
cd lowcost-ha-local-k8s
# GITHUB_MIRROR="ghfast.top/https://github.com"
GITHUB_MIRROR="github.com"
hugo mod init ${GITHUB_MIRROR}/Tsanfer/lowcost-ha-local-k8s
# 整理并下载 Hugo 模块所需依赖
hugo mod tidy
hugo mod graph

# 添加 Hextra 主题
hugo mod get ${GITHUB_MIRROR}/imfing/hextra
```

- 在配置文件中启用 Hextra 主题

文件位置：`./hugo.yaml`​

```yaml {linenos=table,filename="./hugo.yaml"}
module:
  imports:
    - path: github.com/imfing/hextra
#    - path: ghfast.top/https://github.com/imfing/hextra
```

- 创建站点页面

```bash
hugo new content/_index.md
hugo new content/docs/_index.md
```

- 本地浏览站点

```bash
hugo server --buildDrafts --disableFastRender
```

用浏览器访问运行服务的机器的 1313 端口即可，比如 `http://localhost:1313/`​

- （可选）更新 Hugo 模块与主题

如果需要更新 Hugo 模块或者主题，则可执行：

```bash
# 更新所有 Hugo 模块
# hugo mod get -u

# 更新 Hextra 主题
# GITHUB_MIRROR="ghfast.top/https://github.com"
GITHUB_MIRROR="github.com"

hugo mod get -u ${GITHUB_MIRROR}/imfing/hextra
```

## 自动构建部署到 Github Pages

参考链接：[部署站点 – Hextra](https://imfing.github.io/hextra/zh-cn/docs/guide/deploy-site/)

更改图片缓存路径：

文件位置：`./hugo.yaml`​

```yaml {linenos=table,filename="./hugo.yaml"}
caches:
  images:
    dir: :cacheDir/images
```

创建 Github Actions 文件：

```bash
mkdir -p .github/workflows
touch .github/workflows/hugo.yaml
```

根据 Hextra 教程配置 CI/CD 流程：

文件位置：`.github/workflows/pages.yaml`​

```yaml {linenos=table,hl_lines=[60,61],filename=".github/workflows/pages.yaml"}
# 用于构建和部署 Hugo 站点到 GitHub Pages 的示例工作流
name: 部署 Hugo 站点到 Pages

on:
  # 在推送到默认分支时运行
  push:
    branches: ["main"]

  # 允许您从 Actions 选项卡手动运行此工作流
  workflow_dispatch:

# 设置 GITHUB_TOKEN 的权限以允许部署到 GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# 只允许一个并发部署，跳过在运行中和最新排队之间的运行。
# 但是，不要取消正在运行的运行，因为我们希望这些生产部署能够完成。
concurrency:
  group: "pages"
  cancel-in-progress: false

# 默认使用 bash
defaults:
  run:
    shell: bash

jobs:
  # 构建任务
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.138.0
    steps:
      - name: 检出
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 获取所有历史记录以支持 .GitInfo 和 .Lastmod
          submodules: recursive
      - name: 设置 Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.22'
      - name: 设置 Pages
        id: pages
        uses: actions/configure-pages@v4
      - name: 设置 Hugo
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: 使用 Hugo 构建
        env:
          # 为了最大程度地兼容 Hugo 模块
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
        # --baseURL "https://www.example.com/"
      - name: 上传工件
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # 部署任务
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: 部署到 GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

提交 Git 更新并推送到 Github：

```bash
git add -A
git commit -m "Create hugo.yaml"
git push
```

将 Github 仓库设置中的 Pages → Build and deployment → Source 设置为 GitHub Actions

- 添加自定义域名加速访问

如果需要自定义域名访问，则需要 Github Actions 配置文件：

文件位置：`.github/workflows/pages.yaml`​

```yaml {linenos=table,hl_lines=[4],filename=".github/workflows/pages.yaml"}
        run: |
          hugo \
            --gc --minify \
            --baseURL "https://www.example.com/"
```

将 `--baseURL` 参数改为自定义的域名

在 Github 仓库设置的 Pages → Custom domain 中填入自定义域名

登录 DNS 服务商后台，添加一条 CNAME 记录：将域名指向 GitHub Pages 的地址（如 username.github.io）

