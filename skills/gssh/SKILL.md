---
name: use-gssh
description: "Manage SSH sessions with gssh CLI for agents"
allowed-tools: Read, Write, Glob, Grep, Bash
version: 2.0.0
---
完全非交互、纯脚本化。所有密码和参数通过命令行或环境变量传递。

## 安装 / 更新

```bash
npm install -g @zzzwy/gssh
# 或
curl -sSL https://github.com/forechoandlook/public_tools/releases/download/gssh-latest/install.sh | bash
gssh update   # 自动检查并更新
```

## 启动 daemon

```bash
gssh-daemon &          # 直接后台
pm2 start gssh-daemon --name gssh  # 推荐
```

## 连接

```bash
gssh connect my_alias                      # SSH config 别名（推荐）
gssh connect -u user -h host -P "pass"     # 密码
gssh connect -u user -h host -i ~/.ssh/id_ed25519  # 密钥
gssh use <session_id> --save prod          # 保存为别名
gssh use prod                              # 切换到别名
```

## 环境变量（.env 文件）

```bash
# .env 文件示例（gssh 自动加载当前目录的 .env）
GSSH_SUDO_PASSWORD=mypassword   # sudo 密码，--sudo 时自动读取
GSSH_SU_PASSWORD=mypassword     # su 密码，--su 时自动读取
```

优先级：`--env file` > 当前目录 `.env` > `~/.gssh/.env`

## 执行命令

```bash
gssh exec "ls -la" # 多行可能出现解析问题，建议用单行或脚本文件
gssh exec -s prod "pwd"           # 指定会话别名
gssh exec -t 10 "sleep 5"        # 超时（秒）
gssh exec "nohup app > /dev/null 2>&1 &"  # 后台运行

# sudo（从 .env 自动读取 GSSH_SUDO_PASSWORD）
gssh exec --sudo "systemctl restart nginx"
gssh exec --sudo --sudo-user "www-data" "whoami"
gssh exec --sudo --sudo-login "whoami"   # login shell

# su（适用于禁用了 sudo 的机器，从 .env 自动读取 GSSH_SU_PASSWORD）
gssh exec --su "whoami"
gssh exec --su --su-user "www-data" "whoami"
gssh exec --su --su-login "whoami"       # su - login shell
```

## 文件上传与执行

```bash
# 上传并执行
gssh run deploy.sh --exec "./deploy.sh"
gssh run code.tar.gz go.mod -d /app --exec "tar xzf code.tar.gz && go test ./..."
gssh run pkg.tar.gz -d /opt --exec "tar xzf pkg.tar.gz && ./install.sh" --cleanup
```

## 文件传输

```bash
# 单文件上传/下载
gssh scp -put local.txt /remote/path/
gssh scp -get /remote/file.txt ./

# 目录同步（增量，size+mtime 比对）
gssh sync ./local_dir /remote/path/
gssh sync ./local_dir /remote/path/ --ignore .gsshignore   # 带忽略规则
```

`.gsshignore` 语法（gitignore 子集）：
```
# 注释
*.pyc
__pycache__/        # 仅匹配目录
.git/
config/secret.yaml  # 锚定路径
```
source 目录下存在 `.gsshignore` 时自动加载，也可用 `--ignore <file>` 指定。

## 端口转发

```bash
gssh forward -l 8080 -r 80        # 本地转发
gssh forward -R -l 9000 -r 3000   # 远程转发
gssh forwards                      # 列出
gssh forward-close <id>            # 关闭
```

## 会话管理

```bash
gssh list
gssh use --list
gssh use --forget prod
gssh disconnect
gssh reconnect -s <id>
```

## 工作流

```yaml
# deploy.yaml
name: deploy
steps:
  - name: Upload
    exec: "run code.tar.gz -d /app --exec 'tar xzf code.tar.gz'"
    timeout: 60
  - name: Build
    exec: "cd /app && go build"
  - name: Restart
    exec: "systemctl restart myapp"
```

```bash
gssh workflow run deploy -s prod
gssh workflow logs deploy
```

## 限制

- **密码不能含单引号**：`--sudo-password` / `--su-password` 含 `'` 会破坏 shell 语法，用 .env 文件替代
- **su PAM 限制**：部分系统（Ubuntu 22.04+）要求 su 必须在 TTY 中运行，报 `su: must be run from a terminal` 时改用 sudo
- **不支持交互式命令**：vim、top、less 等需要 TTY 的命令不可用
- **超时**：`-t` 触发后远端进程被 SIGKILL，连接立即可用
