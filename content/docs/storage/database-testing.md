---
draft: false
title: '（待补全）数据库测试'
weight: 2
---


{{< callout emoji="🚧" >}}
  此页内容待补全
{{< /callout >}}

### 性能测试

使用 Sysbench 对 PosgreSQL 数据库进行性能测试

- RHEL 编译系统安装 Sysbench

```bash
GITHUB_COM="gh-proxy.com/github.com"
git clone https://${GITHUB_COM}/akopytov/sysbench.git
cd sysbench

sudo dnf -y install autoconf

./autogen.sh
# Ad --with-pgsql to build with PostgreSQL support
./configure --with-pgsql
make -j
make install
```
