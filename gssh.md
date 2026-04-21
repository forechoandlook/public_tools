# gssh - SSH Session Manager for Agents

gssh 是一个供 Agent 使用的 SSH Session 管理工具。通过 Go 语言实现，支持多个 SSH 会话管理、命令执行、端口转发和自动重连。

**设计原则：完全非交互、纯脚本化。所有密码和参数通过命令行传递，不支持交互式操作。**

## 特性

- 📋 多个 SSH Session 并发管理与 Session 持久化
- 🚀 快速文件上传与执行（支持多种模式）
- 🔗 SSH Config 支持（host alias）
- 💻 命令执行与结果透传（stdout/stderr/exit code）
- 🔀 TCP 端口转发（本地/远程）
- 🔄 断线自动重连
- 🔐 支持密码认证和 SSH 密钥认证
- 🏃 Jump Host（堡垒机）支持
- 📁 SFTP 文件传输支持（上传/下载）
- 🔧 工作流系统（Workflow）- 多步骤自动化
- 📦 .env 文件支持
- ✨ 自动更新功能

## 安装

### 使用 npm（推荐）

```bash
npm install -g @zzzwy/gssh
```

### 使用 install.sh

```bash
./install.sh
```

### 手动安装

```bash
# 克隆项目
git clone https://github.com/forechoandlook/gssh.git
cd gssh

# 构建
go build -o bin/gssh ./cmd/gssh
go build -o bin/gssh-daemon ./cmd/daemon

# 复制到系统路径（Linux/macOS）
sudo cp bin/gssh /usr/local/bin/
sudo cp bin/gssh-daemon /usr/local/bin/
```

## 启动服务

```bash
# 启动 gssh daemon（自动启动或手动启动）
gssh-daemon &

# 或使用后台方式
nohup gssh-daemon > /dev/null 2>&1 &
```

## 使用方法

### 快速开始

#### 1. 使用 SSH Config（推荐）

```bash
# 添加到 ~/.ssh/config
Host myserver
    HostName example.com
    User admin
    Port 2222
    IdentityFile ~/.ssh/id_rsa

# 直接使用 host alias
gssh connect myserver
gssh run deploy.sh
gssh exec "systemctl status nginx"
```

#### 2. 直接连接

```bash
# 默认密钥认证（自动查找 ~/.ssh/id_ed25519, id_rsa 等默认密钥）
gssh connect -u admin -h 192.168.1.1

# 密码认证
gssh connect -u admin -h 192.168.1.1 -p 22 -P "password"

# 密钥认证（显式指定密钥路径）
gssh connect -u admin -h example.com -i ~/.ssh/id_ed25519

# 指定 session ID
gssh connect -s mysession -u admin -h example.com -i ~/.ssh/id_rsa
```

### 执行命令

```bash
# 使用默认 session
gssh exec "ls -la"

# 指定 session
gssh exec -s <session_id> "pwd"

# 带超时命令（秒）
gssh exec -t 10 "sleep 5"

# sudo 命令
gssh exec --sudo --sudo-password "password" "systemctl restart nginx"

# sudo 以其他用户运行
gssh exec --sudo --sudo-password "password" --sudo-user "www-data" "whoami"

# sudo login shell
gssh exec --sudo --sudo-password "password" --sudo-login "whoami"

# su 命令（适用于禁用了 sudo 但有 su 的机器）
gssh exec --su --su-password "password" "whoami"
gssh exec --su --su-password "password" --su-user "www-data" "whoami"
gssh exec --su --su-password "password" --su-login "whoami"

# 后台运行
gssh exec "nohup python3 app.py > /dev/null 2>&1 &"
```

### 快速脚本执行 (gssh run)

#### 模式 1：上传并执行文件

```bash
# 上传并执行脚本
gssh run deploy.sh

# 带参数
gssh run deploy.sh production api-server

# 指定远程目录
gssh run -d /opt/scripts deploy.sh

# 指定 session
gssh run -s <session_id> deploy.sh
```

#### 模式 2：直接执行命令（-c flag）

```bash
# 一行式命令
gssh run -c "free -h"

# 带管道和重定向
gssh run -c "ps aux | grep python | wc -l"

# Python 脚本
gssh run -c "python3 -c 'import json; print(json.dumps({\"status\": \"ok\"}))'"

# 指定 session
gssh run -s <session_id> -c "date && whoami"
```

#### 模式 3：上传多个文件后执行命令（--exec flag）

```bash
# 上传文件并执行命令
gssh run code.tar.gz -d /tmp --exec "tar xzf code.tar.gz && go test ./..."

# 上传多个文件
gssh run app.tar.gz go.mod go.sum -d /home/builder --exec "tar xzf app.tar.gz && go test ./..."

# 带自动清理
gssh run deploy.sh config.yml -d /opt --exec "chmod +x deploy.sh && ./deploy.sh" --cleanup

# 指定 session 和超时
gssh run -s <session_id> -t 30 code.tar.gz -d /tmp --exec "tar xzf code.tar.gz && make test"
```

### 端口转发

```bash
# 本地端口转发：本地 8080 -> 远程 80
gssh forward -l 8080 -r 80

# 远程端口转发：远程 9000 -> 本地 3000
gssh forward -R -l 9000 -r 3000

# 指定 session
gssh forward -s <session_id> -l 8080 -r 3000

# 列出所有端口转发
gssh forwards

# 关闭特定端口转发
gssh forward-close <forward_id>
```

### 文件传输（SCP/SYNC）

```bash
# 上传单个文件
gssh scp -put /path/to/local/file.txt /path/to/remote/file.txt

# 上传整个目录
gssh sync -put /path/to/local/dir /path/to/remote/dir

# 下载单个文件
gssh scp -get /path/to/remote/file.txt /path/to/local/file.txt

# 下载整个目录
gssh sync -get /path/to/remote/dir /path/to/local/dir

# 指定 session
gssh scp -s <session_id> -put /local/file /remote/file
```

### SFTP 操作

```bash
# 列出远程目录
gssh sftp -c ls -p /path/to/remote/dir

# 创建远程目录
gssh sftp -c mkdir -p /path/to/remote/newdir

# 删除远程文件
gssh sftp -c rm -p /path/to/remote/file.txt

# 指定 session
gssh sftp -s <session_id> -c ls -p /remote/path
```

### Session 管理

```bash
# 列出所有 session
gssh list

# 设置默认 session
gssh use <session_id>

# 断开 session
gssh disconnect -s <session_id>

# 重连 session
gssh reconnect -s <session_id>

# 创建 session 别名（从 SSH config）
gssh connect myserver -s prod-session
```

### 工作流系统（Workflow）

创建 YAML 工作流文件用于多步骤自动化：

```yaml
# deploy.yaml
name: Deploy Application
steps:
  - name: Upload code
    command: |
      gssh scp -put app.tar.gz /tmp/app.tar.gz
  
  - name: Extract and build
    command: |
      gssh exec "cd /tmp && tar xzf app.tar.gz && cd app && ./build.sh"
  
  - name: Run tests
    command: |
      gssh exec "cd /tmp/app && npm test"
  
  - name: Deploy
    command: |
      gssh exec "cd /opt && tar xzf /tmp/app.tar.gz && systemctl restart app"
```

执行工作流：

```bash
# 运行工作流
gssh workflow -f deploy.yaml

# 指定 session
gssh workflow -s <session_id> -f deploy.yaml
```

### Jump Host（堡垒机）支持

```bash
# 通过 jump host 连接
gssh connect -u user -h target.internal \
  -j jumphost.example.com \
  -ju jumpuser \
  -ji ~/.ssh/jump_key \
  -i ~/.ssh/target_key

# 或使用 SSH config
gssh connect target-behind-jumphost
```

SSH Config 示例：
```
Host jumphost
    HostName jump.example.com
    User jumpuser
    IdentityFile ~/.ssh/jump_key

Host target-behind-jumphost
    HostName target.internal
    User targetuser
    IdentityFile ~/.ssh/target_key
    ProxyJump jumphost
```

### 反馈与问题报告

记录遇到的问题到本地 JSONL 文件，并快速上报到 GitHub：

```bash
# 记录反馈（保存到 ~/.gssh/feedback.jsonl）
gssh feedback "daemon 重连后端口转发失效"
gssh feedback --type bug "exec sudo 命令有时挂起"
gssh feedback --type feature "希望支持 SSH 代理"

# 保存反馈并打开 GitHub Issue 页面
gssh feedback --type bug "daemon crash" --open

# 仅打开最新反馈对应的 GitHub Issue 页面
gssh feedback --open
```

反馈内容包含：版本、OS、描述、活跃 session 数等上下文信息。

### 其他命令

```bash
# 显示版本
gssh -v
# 或
gssh --version

# 显示帮助
gssh -h
# 或
gssh --help

# 关闭 daemon
gssh shutdown

# 重启 daemon
gssh restart

# 检查并自动更新
gssh update
```

## 命令行选项

### 全局选项

| 选项          | 说明                                     |
| ------------- | ---------------------------------------- |
| `-socket path` 或 `-addr` | TCP 地址（默认：127.0.0.1:7272）  |
| `-env file`   | 加载 .env 文件                           |
| `-v`          | 显示版本                                 |
| `--version`   | 显示版本                                 |
| `-h`          | 显示帮助                                 |
| `--help`      | 显示帮助                                 |

### Connect 命令

| 选项          | 说明                                     |
| ------------- | ---------------------------------------- |
| `-s session_id` | Session ID（可选，用于保存会话）        |
| `-u user`     | SSH 用户名                               |
| `-h host`     | 主机地址或 SSH config 中的 host alias   |
| `-p port`     | SSH 端口（默认：22）                     |
| `-P password` | SSH 密码（未指定时自动使用默认密钥认证） |
| `-i key_path` | SSH 私钥路径（未指定时自动查找默认密钥） |
| `-j jump`     | Jump host（堡垒机）                      |
| `-ju user`    | Jump host 用户名                         |
| `-jp port`    | Jump host 端口                           |
| `-jP password`| Jump host 密码                           |
| `-ji key`     | Jump host 密钥路径                       |

### Exec 命令

| 选项          | 说明                                     |
| ------------- | ---------------------------------------- |
| `-s session_id` | Session ID                              |
| `-t timeout`  | 命令超时时间（秒，0=无限制）             |
| `--sudo`      | 启用 sudo 模式                           |
| `--sudo-password` | sudo 密码                          |
| `--sudo-user` | 以指定用户运行 sudo                      |
| `--sudo-login`| sudo login shell（-i）                   |
| `--su`        | 启用 su 模式（适用于禁用了 sudo 的机器） |
| `--su-password` | su 密码                              |
| `--su-user`   | 切换到指定用户（默认：root）             |
| `--su-login`  | su login shell（su -）                   |

### Run 命令

| 选项          | 说明                                     |
| ------------- | ---------------------------------------- |
| `-s session_id` | Session ID                              |
| `-d remote_dir` | 远程目录（默认：/tmp）                  |
| `-c command`  | 直接执行命令（不上传文件）                |
| `--exec cmd`  | 上传文件后执行命令                       |
| `--cleanup`   | 执行后清理上传的文件                     |
| `-t timeout`  | 命令超时时间（秒）                       |

### Forward 命令

| 选项          | 说明                                     |
| ------------- | ---------------------------------------- |
| `-s session_id` | Session ID                              |
| `-l port`     | 本地端口                                 |
| `-r port`     | 远程端口                                 |
| `-R`          | 远程端口转发模式                         |

### SCP/SYNC 命令

| 选项          | 说明                                     |
| ------------- | ---------------------------------------- |
| `-s session_id` | Session ID                              |
| `-put`        | 上传模式（本地 -> 远程）                 |
| `-get`        | 下载模式（远程 -> 本地）                 |

### SFTP 命令

| 选项          | 说明                                     |
| ------------- | ---------------------------------------- |
| `-s session_id` | Session ID                              |
| `-c command`  | SFTP 命令（ls/mkdir/rm）                 |
| `-p path`     | SFTP 路径                                |

### Workflow 命令

| 选项          | 说明                                     |
| ------------- | ---------------------------------------- |
| `-s session_id` | Session ID                              |
| `-f file`     | 工作流 YAML 文件路径                     |

### Feedback 命令

| 选项          | 说明                                     |
| ------------- | ---------------------------------------- |
| `--type`      | 反馈类型：bug、feature、general（默认：general） |
| `--open`      | 在浏览器打开 GitHub Issue 页面           |

## 限制与注意事项

### Sudo / Su 限制

- **密码不能包含单引号**：`--sudo-password` / `--su-password` 中包含单引号会破坏 shell 语法
- **交互式命令不支持**：`sudo vim`、`sudo less` 等需要 TTY 的命令不支持
- **sudo 缓存**：sudo 有默认 15 分钟的密码缓存，期间相同 session 内的后续 sudo 命令无需重复输入密码
- **su PAM 限制**：某些严格 PAM 配置的系统（如 Ubuntu 22.04+）可能要求 su 必须在 TTY 中运行，此时会报 `su: must be run from a terminal`，需改用 sudo 或调整 PAM 配置

### Exec 命令限制

- **不支持交互式命令**：vim、less、top、nano 等需要终端的交互式程序无法使用
- **密码暴露**：密码通过命令行传递，在 `ps aux` 等命令中可能可见，请在安全的系统中使用
- **管道和重定向**：含 `|`、`&&`、`||`、`>`、`<` 等特殊字符的命令需整体加引号，gssh 会自动通过 shell 执行

### Run 命令限制

- **一行式脚本**：`-c` 模式中的命令在远程 shell 中执行，支持管道和重定向
- **文件执行**：上传的文件需要有执行权限或包含正确的 shebang

### SSH Config 支持范围

- ✅ 支持：HostName, User, Port, IdentityFile
- ✅ 支持：通配符模式（如 `Host prod*`）
- ⚠️ 部分支持：ProxyJump（Jump Host）
- ❌ 不支持：某些高级选项（StrictHostKeyChecking 等）

### SFTP 命令注意

- `-p` 标志在 `connect` 命令中表示端口，在 `sftp` 命令中表示路径，两者作用域不同不会冲突

## 实战例子

### 例子 1：部署应用

```bash
# 创建持久化 session
gssh connect -s prod prod-server

# 上传部署脚本
gssh run -s prod deploy.sh production

# 验证状态
gssh run -s prod -c "systemctl status myapp"

# 查看日志
gssh run -s prod -c "tail -50 /var/log/app.log"
```

### 例子 2：批量执行命令

```bash
# 使用 SSH config
gssh connect production-server

# 执行多个检查
gssh exec "free -h"
gssh exec "df -h /"
gssh exec "ps aux | grep myapp"
gssh exec "curl -s http://localhost:8080/health | python3 -m json.tool"
```

### 例子 3：上传代码并测试

```bash
# 打包本地代码
tar czf app.tar.gz src/ go.mod go.sum

# 上传并测试
gssh run app.tar.gz go.mod go.sum -d /tmp/test --exec "tar xzf app.tar.gz && go test ./..." --cleanup
```

### 例子 4：通过堡垒机连接

```bash
# ~/.ssh/config
Host jumphost
    HostName jump.example.com
    User ops
    IdentityFile ~/.ssh/jump_key

Host internal-server
    HostName 10.0.0.5
    User admin
    IdentityFile ~/.ssh/internal_key
    ProxyJump jumphost

# 连接
gssh connect internal-server
gssh exec "whoami"
```

## 开发

### 运行测试

```bash
go test ./...
```

### 构建项目

```bash
# 构建 CLI
go build -o bin/gssh ./cmd/gssh

# 构建 Daemon
go build -o bin/gssh-daemon ./cmd/daemon
```

### 代码结构

- `cmd/daemon/` - Daemon 主程序（RPC 服务）
- `cmd/gssh/` - CLI 主程序（命令行入口）
- `internal/client/` - SSH 客户端封装
- `internal/session/` - Session 管理器和持久化
- `internal/portforward/` - 端口转发实现
- `internal/protocol/` - RPC 协议定义
- `internal/config/` - 配置加载（SSH config, .env 等）
- `pkg/rpc/` - RPC 请求处理器
- `homebrew/` - Homebrew 安装文件

## 反馈与贡献

遇到问题或有改进建议？可以通过 `gssh feedback` 命令快速上报到 GitHub：

```bash
# 本地记录反馈（自动添加版本、OS 等上下文）
gssh feedback --type bug "你遇到的问题"

# 查看本地反馈记录
cat ~/.gssh/feedback.jsonl | jq

# 上报到 GitHub（打开浏览器）
gssh feedback --open
```

所有反馈会保存到 `~/.gssh/feedback.jsonl`，可随时查看和管理。

## 常见问题

**Q: Daemon 没有启动？**  
A: 检查 daemon 是否运行：`ps aux | grep gssh-daemon`。手动启动：`gssh-daemon &`

**Q: 如何指定非标准 daemon 地址？**  
A: 使用 `-addr` 或 `-socket` 标志：`gssh -addr 127.0.0.1:8888 exec "whoami"`

**Q: 支持 Windows 吗？**  
A: 目前仅支持 macOS 和 Linux。Windows 支持在规划中。

**Q: 如何在 CI/CD 中使用？**  
A: 推荐使用 SSH 密钥认证和 session ID，例如：
```bash
gssh connect -s ci-session -u deploy -h prod.example.com -i ~/.ssh/ci_key
gssh run -s ci-session deploy.sh
```

## 许可证

MIT
