---
draft: false
title: '部署哪吒监控'
weight: 1
---

面板地址：[系统监控](https://k8s-local.monitor.tsanfer.com/)

## 安装服务端 Dashboard

```bash
curl -L https://gitee.com/naibahq/scripts/raw/main/install.sh -o nezha.sh && \
	chmod +x nezha.sh && \
	sudo CN=true ./nezha.sh

# 请输入暴露端口: (默认 8008)8008
# 请指定安装命令中预设的 nezha-agent 连接地址 （例如 example.com:443）k8s-local.monitor.tsanfer.com
# 是否希望通过 TLS 连接 Agent？（影响安装命令）[y/N]N
```

默认地址：域名:站点访问端口，比如：`http://k8s-local.monitor.tsanfer.com:8008`​

## 安装客户端 Agent

### Linux 安装 Agent

> 个人一键脚本
>
> - 代码
>
>   ```bash {linenos=table,hl_lines=[7,15,27]}
>   apt-get update -y 
>   apt-get install -y unzip 
>
>   # hub.gitmirror.com/https://raw.githubusercontent.com \
>   curl -L https://gh-proxy.com/raw.githubusercontent.com/nezhahq/scripts/main/agent/install.sh -o agent.sh && \
>     chmod +x agent.sh && \
>     env NZ_SERVER=k8s-local.monitor.tsanfer.com NZ_TLS=false NZ_CLIENT_SECRET=<CLIENT_SECRET> CN=true \
>     ./agent.sh
>
>   # 1. 删除非uuid行
>   sed -i '/^uuid:/!d' /opt/nezha/agent/config.yml
>
>   # 2. 在uuid行（第1行）前插入配置块
>   sed -i '1i \
>   client_secret: <CLIENT_SECRET> \
>   debug: false \
>   disable_auto_update: false \
>   disable_command_execute: false \
>   disable_force_update: false \
>   disable_nat: false \
>   disable_send_query: false \
>   gpu: false \
>   insecure_tls: false \
>   ip_report_period: 30 \
>   report_delay: 1 \
>   self_update_period: 0 \
>   server: k8s-local.monitor.tsanfer.com:8008 \
>   skip_connection_count: false \
>   skip_procs_count: false \
>   temperature: true \
>   tls: false \
>   use_gitee_to_upgrade: true \
>   use_ipv6_country_code: false' /opt/nezha/agent/config.yml
>
>   # 删除多余配置文件，仅保留默认配置文件
>   find /opt/nezha/agent -maxdepth 1 -type f -name "*.yml" ! -name "config.yml" -delete
>   /opt/nezha/agent/nezha-agent service restart
>
>   # 可能需要重启主机，才能删除多余的配置
>   # reboot
>
>   # 查看 uuid
>   grep uuid /opt/nezha/agent/config.yml
>
>   ```

安装依赖

```bash
apt-get update -y 
apt-get install -y unzip 

```

在 `服务器` 页面中，点击 `安装命令` 并选择对应操作系统，安装命令将自动复制到你的剪贴板

```bash {linenos=table,hl_lines=[4]}
curl -L https://raw.githubusercontent.com/nezhahq/scripts/main/agent/install.sh \
  -o agent.sh \
  && chmod +x agent.sh \
  && env NZ_SERVER=k8s-local.monitor.tsanfer.com:8008 NZ_TLS=false NZ_CLIENT_SECRET=<CLIENT_SECRET> CN=true \
  ./agent.sh

```

更改配置文件

```bash {linenos=table,hl_lines=[6,18]}
# 1. 删除非uuid行
sed -i '/^uuid:/!d' /opt/nezha/agent/config.yml

# 2. 在uuid行（第1行）前插入配置块
sed -i '1i \
client_secret: <CLIENT_SECRET> \
debug: false \
disable_auto_update: false \
disable_command_execute: false \
disable_force_update: false \
disable_nat: false \
disable_send_query: false \
gpu: false \
insecure_tls: false \
ip_report_period: 30 \
report_delay: 1 \
self_update_period: 0 \
server: k8s-local.monitor.tsanfer.com:8008 \
skip_connection_count: false \
skip_procs_count: false \
temperature: true \
tls: false \
use_gitee_to_upgrade: true \
use_ipv6_country_code: false' /opt/nezha/agent/config.yml

```

重启客户端

```bash
# 删除多余配置文件，仅保留默认配置文件
find /opt/nezha/agent -maxdepth 1 -type f -name "*.yml" ! -name "config.yml" -delete
/opt/nezha/agent/nezha-agent service restart

# 可能需要重启主机，才能删除多余的配置
# reboot

```

查看客户端 uuid，以此来设置面板中对应的服务器名称

```bash
grep uuid /opt/nezha/agent/config.yml

```

### OpenWrt 安装 Agent

> 小米路由器 R3G 的 CPU 为 mt7621，此 CPU 的类型为 MIPS1004Kc

- 个人一键脚本：

```bash
# 加速地址
github_release_endpoint="get-github.hexj.org/download"
# 系统架构
sys_arch="linux_mipsle"
# 哪吒监控客户端版本
nezha_agnet_version="1.10.0"
# 哪吒监控客户端安装位置
nezha_agent_install_path="/etc/nezha"

# 更新 opkg 和依赖
opkg update && opkg install wget unzip lscpu
echo
echo "系统架构："
uname -m
echo
echo "CPU 大小端："
lscpu | grep -i byte
echo

# 下载适配的依赖
wget -O nezha-agent.zip https://${github_release_endpoint}/nezhahq/agent/releases/download/v${nezha_agnet_version}/nezha-agent_${sys_arch}.zip

# 解压文件
mkdir -p ${nezha_agent_install_path}
unzip nezha-agent.zip -d ${nezha_agent_install_path}

# <<EOF 必须紧跟在命令后面，表示命令的输入内容将从下一行开始，直到遇到 EOF 为止。
cat <<EOF >${nezha_agent_install_path}/config.yml
client_secret: <CLIENT_SECRET>
debug: false
disable_auto_update: false
disable_command_execute: false
disable_force_update: false
disable_nat: false
disable_send_query: false
gpu: false
insecure_tls: false
ip_report_period: 60
report_delay: 4
self_update_period: 90
server: k8s-local.monitor.tsanfer.com:8008
skip_connection_count: false
skip_procs_count: false
temperature: true
tls: false
use_gitee_to_upgrade: true
use_ipv6_country_code: false
uuid: $(cat /proc/sys/kernel/random/uuid)
EOF

# 授予 agnet 权限并执行
#chmod +x ${nezha_agent_install_path}/nezha-agent
#${nezha_agent_install_path}/nezha-agent -c ${nezha_agent_install_path}/config.yml

# 创建服务脚本
cat <<EOF >/etc/init.d/nezha-service
#!/bin/sh /etc/rc.common

START=99
USE_PROCD=1

start_service() {
    procd_open_instance
    procd_set_param command ${nezha_agent_install_path}/nezha-agent -c ${nezha_agent_install_path}/config.yml
    procd_set_param respawn
    procd_close_instance
}

stop_service() {
    killall nezha-agent
}

restart() {
    stop
    sleep 2
    start
}
EOF

# 赋予脚本执行权限
chmod +x /etc/init.d/nezha-service
# 启用服务
/etc/init.d/nezha-service enable
/etc/init.d/nezha-service start
# 重启服务
# /etc/init.d/nezha-service restart
echo
echo "验证启动状态："
ps | grep nezha-agent
echo
echo "查看 uuid："
grep uuid ${nezha_agent_install_path}/config.yml
echo

```

### Windows 安装 Agent

> Windows 版本为 10 和 11
> PowerShell 版本为 7.5

个人一键脚本

```powershell {linenos=table,hl_lines=[9,11,22,34]}
# 下载脚本
$github_raw_endpoint = "gh-proxy.com/raw.githubusercontent.com"
$url = "https://${github_raw_endpoint}/nezhahq/scripts/main/agent/install.ps1"
$outputPath = "${env:USERPROFILE}\Downloads\nezha_agent_install.ps1"
Invoke-WebRequest -Uri $url -OutFile $outputPath -UseBasicParsing

# 赋予脚本执行权限并执行
Set-ExecutionPolicy Bypass -Scope Process -Force
$env:NZ_SERVER = "aliecs.tsanfer.com:8008"
$env:NZ_TLS = "false"
$env:NZ_CLIENT_SECRET = "<CLIENT_SECRET>"
& $outputPath

# 删除非 uuid 行
$configPath = "C:\nezha\config.yml"
$content = Get-Content -Path $configPath  # 不使用 -Raw 参数
$filteredContent = $content | Where-Object { $_ -match "^\s*uuid:" }  # 匹配包含 uuid: 的行
Set-Content -Path $configPath -Value $filteredContent

# 在 uuid 行前插入配置块
$configBlock = @'
client_secret: <CLIENT_SECRET>
debug: false
disable_auto_update: false
disable_command_execute: false
disable_force_update: false
disable_nat: false
disable_send_query: false
gpu: true
insecure_tls: false
ip_report_period: 30
report_delay: 1
self_update_period: 0
server: k8s-local.monitor.tsanfer.com:8008
skip_connection_count: false
skip_procs_count: false
temperature: true
tls: false
use_gitee_to_upgrade: true
use_ipv6_country_code: false
'@

$configContent = Get-Content -Path $configPath -Raw
$newContent = $configBlock + "`n" + $configContent
Set-Content -Path $configPath -Value $newContent

# 删除多余的配置文件，仅保留默认配置文件
$nezhaAgentPath = "C:\nezha"
Get-ChildItem -Path $nezhaAgentPath -Filter *.yml | Where-Object { $_.Name -ne "config.yml" } | Remove-Item -Force

# 重启服务
& "C:\nezha\nezha-agent.exe" service restart

# 查看 uuid
Get-Content -Path $configPath | Select-String -Pattern "uuid:"

```

### Agent 设置文件

通常位置：`/opt/nezha/agent/config.yml`

```yaml {linenos=table,hl_lines=[1,13,20],filename="/opt/nezha/agent/config.yml"}
client_secret: xxxx
debug: false
disable_auto_update: false
disable_command_execute: false
disable_force_update: false
disable_nat: false
disable_send_query: false
gpu: false
insecure_tls: false
ip_report_period: 30
report_delay: 1
self_update_period: 0
server: k8s-local.monitor.tsanfer.com:8008
skip_connection_count: false
skip_procs_count: false
temperature: true
tls: false
use_gitee_to_upgrade: true
use_ipv6_country_code: false
uuid: your_uuid

```

---

## 卸载

### 卸载 dashboard

```bash
curl -L https://gitee.com/naibahq/scripts/raw/main/install.sh -o nezha.sh && \
	chmod +x nezha.sh && \
	sudo CN=true ./nezha.sh

```

### 卸载 agent

卸载 Agent 包括停止服务、卸载服务，以及删除相关文件。以下是 Ubuntu 系统的卸载步骤：

1. 停止并卸载服务：

    ```bash
    cd /opt/nezha/agent/./nezha-agent service uninstall

    ```
2. 删除 Agent 文件夹：

    ```bash
    rm -rf /opt/nezha/agent/

    ```

如果安装了多个服务并想要全部卸载，可以使用 Agent 安装脚本的卸载功能：

```bash
./agent.sh uninstall

```
