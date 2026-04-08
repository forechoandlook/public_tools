# wechat

本地微信数据解密与查询工具。从微信进程内存提取密钥，解密本地数据库，提供 CLI 查询接口。

支持 macOS（ARM64 / x86_64）。

## 安装

```bash
curl -fsSL https://github.com/forechoandlook/public_tools/releases/download/wechat-latest/install.sh | bash
```

安装到 `~/.local/bin/wechat`。更新：

```bash
wechat update
```

## 快速开始

首次使用需要扫描密钥（微信必须处于运行状态）：

```bash
sudo wechat find-keys
```

之后直接查询（无需 sudo，无需 daemon）：

```bash
wechat sessions --limit 20
wechat history "张三" --limit 50
wechat search "关键词"
```

## 命令

### 会话 / 消息

```bash
wechat sessions --limit 20
wechat sessions --limit 20 --offset 20          # 分页
wechat history "张三" --limit 50
wechat history "张三" --start-time "2026-01-01"
wechat search "关键词"
wechat search "关键词" --chat "张三"             # 限定聊天
wechat new-messages                              # 最近新消息
wechat subscribe "张三"                          # 实时订阅
```

### 联系人

```bash
wechat contacts                                  # 全部
wechat contacts --filter friends                 # 仅好友（local_type=3/5/6）
wechat contacts --filter public                  # 仅公众号（gh_xxx）
wechat contacts --filter chatrooms               # 仅群聊
wechat contacts --limit 100 --offset 100         # 分页
wechat chatrooms "关键词"                         # 搜索群聊
wechat group-members "群名" --limit 50           # 群成员（按发言数排序）
```

### 收藏 / 朋友圈 / 公众号

```bash
wechat favorites --keyword "关键词"
wechat moments --keyword "关键词"
wechat public-accounts                           # 订阅的公众号列表
wechat public-account-articles "gh_xxx"          # 某公众号全部文章
```

### 统计分析

```bash
wechat chat-activity                             # 全局活跃度
wechat chat-activity --chat "张三" --interval daily
```

## 输出格式

默认输出 **CSV**（适合 agent / 脚本消费）：

```bash
wechat sessions --limit 20
# display_name,is_group,summary,time_str,username,...
```

切换为 JSON：

```bash
wechat --format json sessions
WECHAT_FORMAT=json wechat sessions
```

只输出指定字段：

```bash
wechat --fields "display_name,summary,time_str" sessions --limit 20
WECHAT_FIELDS="username,display_name" wechat contacts --filter friends
```

## 时间过滤

所有查询命令均支持 `--start-time` / `--end-time`（格式 `YYYY-MM-DD` 或 `YYYY-MM-DD HH:MM`）：

```bash
wechat history "张三" --start-time "2026-01-01" --end-time "2026-04-01"
wechat sessions --start-time "2026-03-01"
wechat public-account-articles "机器之心" --start-time "2026-01-01"
```

## 反馈

```bash
wechat feedback "描述问题"           # 记录到 ~/.wechat/feedback.jsonl
wechat feedback "描述问题" --open    # 打开浏览器预填 GitHub issue
wechat feedback --upload             # 批量查看并上传所有本地记录
```

## 运行模式

| 模式 | 触发条件 | 说明 |
|------|---------|------|
| daemon | `wechat`（无参数）| 长驻进程，监听 WAL 变化，支持实时推送 |
| client | 有子命令且 daemon 运行中 | 连接 daemon 查询，响应快 |
| direct | daemon 未运行 / `--direct` | 自动 fallback，首次解密后复用缓存（< 100ms）|

缓存刷新：

```bash
WECHAT_REFRESH=1 wechat sessions     # 强制重新解密，获取最新数据
```

## Daemon 模式（长驻）

```bash
# 启动
wechat --log-level info

# 用 pm2 管理
pm2 start wechat --name wechatd -- --daemon-host 0.0.0.0 --daemon-port 18521 --log-level info

# 远程连接
wechat --host 192.168.1.100 --port 18521 sessions
```

## 多账号

```bash
wechat --db-dir ~/path/to/wxid_xxx/db_storage sessions
```

或在 `~/.wechat/config.json` 中指定 `db_dir`。

## 注意事项

**首次运行前**手动创建配置目录，避免 sudo 创建 root-owned 目录导致后续写入失败：

```bash
mkdir -p ~/.wechat && chmod 700 ~/.wechat
```

## 架构

```
wechat（单一二进制）
├── daemon 模式：TCP server（默认 :18521）+ WAL 轮询
├── client 模式：TCP client，连接 daemon
└── direct 模式：直接加载密钥 + 解密数据库 + 查询

crates/
├── protocol/          # Request / Response / Push 协议定义
├── wechat-core/       # SQLCipher 解密 + DBCache
├── wechat-scanner/    # 进程内存扫描密钥
├── wechat-daemon/     # 主程序：daemon + CLI
└── wechat-client/     # TCP 客户端库
```
