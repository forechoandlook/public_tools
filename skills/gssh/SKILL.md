---
name: use-gssh
description: "Manage SSH sessions with gssh CLI for agents"
allowed-tools: Read, Write, Glob, Grep, Bash
version: 2.0.0
---
完全非交互、纯脚本化。所有密码和参数通过命令行传递，不支持交互式操作。

## 安装更新
方式 1: npm
`npm install -g @zzzwy/gssh`

方式 2: 直接安装脚本（不需要 npm）
`curl -sSL https://github.com/forechoandlook/public_tools/releases/download/gssh-latest/install.sh | bash`

gssh update 会自动从 GitHub 检查新版本

**版本信息说明**
- 版本格式：`tag+commit_short_hash`
- `gssh --version` 显示完整版本号
- `gssh update` 会比对本地和远程版本，相同则提示已是最新，不同则自动更新

## 启动

```bash
# 放后台启动
gssh-daemon &

# 推荐使用pm2 管理
pm2 start gssh-daemon --name gssh
```

## 连接认证

```bash
# 默认密钥认证（自动查找 ~/.ssh/id_ed25519, id_rsa 等默认密钥）
gssh connect -u user -h host

# 使用密码
gssh connect -u user -h host -p port -P "password"

# SSH 密钥（显式指定）
gssh connect -u user -h host -i ~/.ssh/id_ed25519

# 使用 SSH config 别名（自动读取配置）
gssh connect my_host_alias
```

**默认密钥搜索顺序**: `~/.ssh/id_ed25519` → `~/.ssh/id_rsa` → `~/.ssh/id_ecdsa` → `~/.ssh/id_ecdsa_sk` → `~/.ssh/id_ed25519_sk` → `~/.ssh/id_dsa`

## 环境变量和密码管理

```bash
# 创建 .env 文件存储密码
cat > .env << EOF
GSSH_SUDO_PASSWORD=mypassword
EOF

# 自动加载 .env（优先级：--env > .env > ~/.gssh/.env）
gssh -env .env exec --sudo "command"

# 或默认加载当前目录的 .env
gssh exec --sudo "command"
```

## 会话管理

```bash
# 建立连接
gssh connect -u user -h host
gssh use <session_id> --save prod    # 保存为别名

# 后续直接用别名
gssh use prod
gssh exec "ls -la"                   # 自动用 prod 会话

# 列表和管理
gssh use --list                      # 显示所有别名
gssh use --forget prod               # 删除别名
```

## 核心命令

| 命令            | 说明             |
| --------------- | ---------------- |
| `connect`       | 建立 SSH 连接    |
| `exec`          | 执行命令         |
| `run`           | 上传并执行文件   |
| `scp` / `sync`  | 文件传输与同步   |
| `sftp`          | SFTP 操作        |
| `forward`       | 端口转发         |
| `forwards`      | 列出所有端口转发 |
| `forward-close` | 关闭端口转发     |
| `workflow`      | 工作流系统       |
| `list`          | 列出 session     |
| `use`           | 管理会话别名     |
| `feedback`      | 本地反馈记录与 GitHub 上报 |

## 执行命令

```bash
# 普通命令
gssh exec "ls -la"

# 指定 session（直接用 ID 或别名）
gssh exec -s prod "pwd"              # 用别名
gssh exec -s <session_id> "pwd"      # 用 ID

# 带超时命令（秒）
gssh exec -t 10 "ls -la"

# 后台运行（推荐格式）
gssh exec "nohup python3 app.py > /dev/null 2>&1 &"

# sudo 命令（推荐从 .env 读取密码）
gssh -env .env exec --sudo "systemctl restart nginx"

# sudo 以其他用户运行
gssh -env .env exec --sudo --sudo-user "www-data" "whoami"

# sudo login shell
gssh -env .env exec --sudo --sudo-login "whoami"
```

## 文件上传和执行

**注意：`gssh run` 命令必须同时指定文件和 `--exec` 命令。如果只需执行命令，请使用 `gssh exec`。**

```bash
# 上传单个文件，然后执行命令
gssh run deploy.sh --exec "./deploy.sh"

# 上传多个文件，然后执行命令
gssh run code.tar.gz go.mod go.sum -d /app --exec "tar xzf code.tar.gz && go test ./..."

# 上传后自动清理
gssh run package.tar.gz install.sh -d /opt --exec "tar xzf package.tar.gz && ./install.sh" --cleanup
```

## 工作流系统（多步自动化）

```bash
# 创建工作流
gssh workflow create deploy

# 编辑工作流（YAML 格式）
gssh workflow edit deploy

# 执行工作流
gssh workflow run deploy -s prod

# 查看日志
gssh workflow logs deploy              # 查看最新日志
gssh workflow logs deploy deploy_2024-01-15_14-30-45.log  # 查看特定日志

# 管理工作流
gssh workflow list
gssh workflow show deploy
gssh workflow delete deploy
```

**工作流文件示例：**
```yaml
name: deploy
env: .env.prod              # 默认环境文件
steps:
  - name: "Upload and Extract"
    exec: "run code.tar.gz -d /app --exec 'tar xzf code.tar.gz'"
    timeout: 60
  
  - name: "Build"
    exec: "cd /app && go build"
  
  - name: "Test"
    exec: "cd /app && go test ./..."
  
  - name: "Deploy"
    exec: "systemctl restart myapp"
```

## 文件传输

```bash
# 上传本地文件/文件夹 -> 远程
gssh scp -put local.txt /remote/path/
gssh sync -put local_dir /remote/path/

# 下载远程文件/文件夹 -> 本地
gssh scp -get /remote/file.txt ./
gssh sync -get /remote/dir ./

# SFTP 目录操作
gssh sftp -c ls -p /home/user
gssh sftp -c mkdir -p /home/user/newdir
gssh sftp -c rm -p /home/user/file.txt
```

## 端口转发

```bash
# 本地转发: localhost:8080 -> remote:80
gssh forward -l 8080 -r 80

# 远程转发: remote:9000 -> localhost:3000
gssh forward -R -l 9000 -r 3000

# 列出所有端口转发
gssh forwards

# 关闭端口转发
gssh forward-close <forward_id>
```

## Session 管理

```bash
gssh list                    # 列出所有 session
gssh use <session-id>        # 切换默认 session
gssh disconnect             # 断开
gssh reconnect -s <id>     # 重连
```

## 反馈与问题报告

在本地记录遇到的问题，并快速上报到 GitHub：

```bash
# 记录反馈到 ~/.gssh/feedback.jsonl
gssh feedback "daemon 重连后端口转发失效"
gssh feedback --type bug "exec sudo 命令有时挂起"
gssh feedback --type feature "希望支持 SSH 代理"

# 保存反馈同时打开 GitHub Issue 页面（预填内容）
gssh feedback --type bug "daemon crash when reconnect" --open

# 仅打开最新反馈对应的 GitHub Issue
gssh feedback --open
```

**反馈记录包含：**
- `id`: 唯一标识（fb_YYYYMMDD_xxxxx）
- `type`: bug、feature 或 general
- `description`: 用户描述
- `version`: gssh 版本信息
- `os` / `arch`: 操作系统和架构
- `session_info`: 活跃 session 数（daemon 可用时）
- `timestamp`: RFC3339 时间戳

**查看本地反馈：**
```bash
cat ~/.gssh/feedback.jsonl | jq
```

**GitHub 上报：**
`--open` 打开浏览器到 `forechoandlook/public_tools` GitHub Issue 页面，自动预填版本、OS、描述等信息。

## 限制与注意事项

### sudo 限制

- **密码不能包含单引号**：`--sudo-password` 中包含单引号会破坏 shell 语法
- **交互式命令不支持**：`sudo vim`、`sudo less`、`sudo -i`（无密码）等需要 TTY 的命令不支持
- **sudo 缓存**：sudo 有默认 15 分钟的密码缓存，相同 session 内后续 sudo 命令无需重复输入密码

### exec 命令限制

- **不支持交互式命令**：vim、less、top、nano 等需要终端的交互式程序无法使用
- **密码暴露**：密码通过命令行传递，在 `ps aux` 等命令中可能可见，请使用 .env 文件管理密码而非命令行
- **管道命令需加引号**：含 `|`、`&&`、`||`、`>`、`<` 等特殊字符的命令需整体加引号

### 多行命令处理

**⚠️ 为避免多行命令解析错误，推荐使用单行命令。** 如必须使用多行，改用 shell 脚本.

## 可靠性

- **超时机制**：`-t` 超时触发后，远端进程会被 SIGKILL 终止，当前连接立即可用于下一条命令，不需要重启 daemon
- **自动重连**：连接断开后每 5 秒自动检测并重连，死连接（含网络分区场景）5 秒内可感知
- **大命令支持**：exec 命令长度无 4096 字节限制，传输文件路径、长脚本均可正常工作
