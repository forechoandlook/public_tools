# gssh - SSH Session Manager for Agents

gssh 是一个供 Agent 使用的 SSH Session 管理工具。通过 Go 语言实现，支持多个 SSH 会话管理、命令执行、端口转发和自动重连。

**设计原则：完全非交互、纯脚本化。所有密码和参数通过命令行传递，不支持交互式操作。**

## 特性

- 多个 SSH Session 并发管理
- 命令执行与结果透传（stdout/stderr/exit code）
- TCP 端口转发（本地/远程）
- 断线自动重连
- 支持密码认证和 SSH 密钥认证
- 被连接机器零配置（普通 SSH 即可）
- SFTP 文件传输支持（上传/下载）

## 安装

### 使用 npm（推荐）

```bash
npm install -g @zzzwy/gssh
```

### 手动安装

```bash
# 克隆项目
git clone https://github.com/forechoandlook/gssh.git
cd gssh

# 构建
go build -o bin/gssh ./cmd/gssh
go build -o bin/gssh-daemon ./cmd/daemon

# 复制到系统路径
cp bin/gssh /usr/local/bin/gssh
cp bin/gssh-daemon /usr/local/bin/gssh-daemon
```

## 启动服务

```bash
# 启动 gssh daemon
gssh-daemon &
```

## 使用方法

### 连接 SSH（密码认证）

```bash
gssh connect -u admin1 -h 192.168.1.1 -p port -P "your-password"
```

### 连接 SSH（密钥认证）

```bash
gssh connect -u admin1 -h example.com -i ~/.ssh/id_ed25519
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
gssh exec --sudo --sudo-password "your-sudo-password" "systemctl restart nginx"

# sudo 以其他用户运行
gssh exec --sudo --sudo-password "your-sudo-password" --sudo-user "www-data" "whoami"

# sudo login shell
gssh exec --sudo --sudo-password "your-sudo-password" --sudo-login "whoami"

# 后台运行（推荐格式）
gssh exec "nohup python3 app.py > /dev/null 2>&1 &"
```

### 端口转发

```bash
# 本地端口转发：本地 8080 -> 远程 80
gssh forward -l 8080 -r 80

# 远程端口转发：远程 9000 -> 本地 3000
gssh forward -R -l 9000 -r 3000

# 列出所有端口转发
gssh forwards

# 关闭端口转发
gssh forward-close <forward_id>
```

### 文件传输与同步（SFTP/SCP/SYNC）

```bash
# 上传文件或文件夹（本地 -> 远程）
gssh scp -put /path/to/local/file.txt /path/to/remote/file.txt
gssh sync -put /path/to/local/dir /path/to/remote/dir

# 下载文件或文件夹（远程 -> 本地）
gssh scp -get /path/to/remote/file.txt /path/to/local/file.txt
gssh sync -get /path/to/remote/dir /path/to/local/dir

# 列出远程目录
gssh sftp -c ls -p /path/to/remote/dir

# 创建远程目录
gssh sftp -c mkdir -p /path/to/remote/newdir

# 删除远程文件
gssh sftp -c rm -p /path/to/remote/file.txt
```

### Session 管理

```bash
# 列出所有 session
gssh list

# 切换默认 session
gssh use <session_id>

# 断开 session
gssh disconnect -s <session_id>

# 重连 session
gssh reconnect -s <session_id>
```

## 命令行选项

| 选项              | 说明                                     |
| ----------------- | ---------------------------------------- |
| `-socket path`    | Unix socket 路径（默认：/tmp/gssh.sock） |
| `-s session_id`   | Session ID                               |
| `-u user`         | 用户名                                   |
| `-h host`         | 主机地址                                 |
| `-p port`         | 端口（默认：22）                         |
| `-P password`     | SSH 密码                                 |
| `-i key_path`     | SSH 密钥路径                             |
| `-l local`        | 本地端口                                 |
| `-r remote`       | 远程端口                                 |
| `-R`              | 远程端口转发                             |
| `-put`            | 上传模式（本地 -> 远程）                 |
| `-get`            | 下载模式（远程 -> 本地）                 |
| `-c command`      | SFTP 命令（ls/mkdir/rm）                 |
| `-p path`         | SFTP 路径（仅 sftp 子命令）              |
| `-t timeout`      | 命令超时时间（秒）                       |
| `--sudo`          | 启用 sudo 模式                           |
| `--sudo-password` | sudo 密码                                |
| `--sudo-user`     | 以指定用户运行 sudo                      |
| `--sudo-login`    | sudo login shell（-i）                   |

## 限制与注意事项

### sudo 限制

- **密码不能包含单引号**：`--sudo-password` 中包含单引号会破坏 shell 语法
- **交互式命令不支持**：`sudo vim`、`sudo less`、`sudo -i`（无密码）等需要 TTY 的命令不支持
- **sudo 缓存**：sudo 有默认 15 分钟的密码缓存，期间相同 session 内的后续 sudo 命令无需重复输入密码

### exec 命令限制

- **不支持交互式命令**：vim、less、top、nano 等需要终端的交互式程序无法使用
- **密码暴露**：密码通过命令行传递，在 `ps aux` 等命令中可能可见，请确保系统安全
- **管道命令需加引号**：含 `|`、`&&`、`||`、`>`、`<`、换行等特殊字符的命令需整体加引号，gssh 会自动通过 shell 执行

### SFTP 限制

- `-p` 标志在 connect 命令中表示端口，在 sftp 命令中表示路径，两者作用域不同不会冲突

## 开发

### 运行测试

```bash
go test ./...
```

### 代码结构

- `cmd/daemon/` - daemon 主程序
- `cmd/gssh/` - CLI 主程序
- `internal/client/` - SSH 客户端封装
- `internal/session/` - Session 管理器
- `internal/portforward/` - 端口转发
- `internal/protocol/` - 协议类型定义
- `pkg/rpc/` - RPC 处理器
- `homebrew/` - Homebrew 安装文件

## Test Report

A comprehensive suite of tests has been implemented to verify core functionalities and handle edge cases robustly. The test suite covers various packages including `internal/client`, `internal/portforward`, `internal/session`, and `pkg/rpc`.

### Coverage Overview

```text
?       gssh/cmd/daemon    [no test files]
?       gssh/cmd/gssh      [no test files]
ok      gssh/internal/client         coverage: 17.5% of statements
ok      gssh/internal/portforward    coverage: 44.6% of statements
ok      gssh/internal/protocol       coverage: [no statements]
ok      gssh/internal/session        coverage: 56.4% of statements
ok      gssh/pkg/rpc                 coverage: 89.6% of statements

total:  (statements)                 34.2%
```

### Key Areas Tested

1. **RPC Handler (`pkg/rpc`)**
   - **Coverage: 89.6%**
   - Validates JSON parsing and incorrect RPC parameters.
   - Tests routing for all supported commands (`connect`, `disconnect`, `exec`, `scp`, `forward`, etc.).
   - Validates proper handling of HTTP requests and Unix socket streams.

2. **Session Manager (`internal/session`)**
   - **Coverage: 56.4%**
   - Covers concurrent session creation, deduplication, and caching.
   - Handles edge cases for missing session IDs across all file transfer (SCP/SFTP) and execution subcommands.
   - Includes timeout assertions, `needsShell` heuristics validation, and missing local file uploads (`TestManagerSCP_MissingLocalFile`).

3. **Port Forwarding (`internal/portforward`)**
   - **Coverage: 44.6%**
   - Tests local and remote forwarding initialization and teardown logic.
   - Verifies the thread-safety of forwarder restarts to prevent duplicate goroutines (`TestForwarderRestartNoDuplicateGoroutine`).
   - Handles incorrect forwarding types correctly.

4. **SSH Client (`internal/client`)**
   - **Coverage: 17.5%**
   - Validates path traversals checks (`checkPathTraversal`) with both absolute and relative boundaries.
   - Tests user authentication methods (Password & Keyboard Interactive).
   - Validates missing file errors gracefully fallback during upload checks (`TestSFTPClient_UploadMissingLocalFile`).

### Running the Tests

To run the complete test suite with coverage, execute:

```bash
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out
```
