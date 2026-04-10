---
name: wechat
description: wechat信息获取.当用户询问如何安装、启动、查询微信相关数据时触发.
---

## 检查、安装和更新
是否安装: `wechat --version`
安装: `curl -fsSL https://github.com/forechoandlook/public_tools/releases/download/wechat-latest/install.sh | bash`
安装后默认路径：`~/.local/bin/wechat`.
更新：`wechat update` 或重新执行安装脚本.

## 输出格式

默认输出 **CSV**（适合 agent 消费，节省 token）。需要 JSON 时：

```bash
wechat --format json sessions
# 或通过环境变量
WECHAT_FORMAT=json wechat sessions
```

只输出指定字段（`--fields`，逗号分隔，保持顺序）：

```bash
wechat --fields "display_name,summary,time_str" sessions --limit 20
# 通过环境变量
WECHAT_FIELDS="username,display_name" wechat contacts --filter friends
```

## 常用指令

```bash
# 会话列表
wechat sessions --limit 20
wechat sessions --limit 20 --offset 20        # 分页

# 聊天记录
wechat history "xx" --limit 50

# 全局搜索
wechat search "关键词"

# 联系人
wechat contacts                               # 所有联系人（含公众号、群聊、系统账号）
wechat contacts --filter friends              # 仅好友（local_type=3/5/6）
wechat contacts --filter public               # 仅公众号（gh_xxx）
wechat contacts --filter chatrooms            # 仅群聊
wechat contacts --filter contacts             # 非群聊（好友+公众号）
wechat contacts --limit 100 --offset 100      # 分页

# 群聊
wechat chatrooms                              # 所有群聊
wechat chatrooms "关键词"                      # 搜索群聊

# 收藏 / 朋友圈
wechat favorites --keyword "关键词"
wechat moments

# 公众号
wechat public-accounts                        # 返回所有公众号
wechat public-account-articles "gh_xxx"       # 某公众号所有文章链接

# 群成员分析
wechat group-members "群聊名称"
wechat group-members "群聊名称" --offset 50   # 分页（按消息数排序）

# 聊天活跃度
wechat chat-activity
wechat chat-activity --interval daily

# 时间过滤（所有命令均支持）
wechat history "xx" --start-time "2026-01-01" --end-time "2026-04-01"

# 连接远程 daemon（--host/--port 是客户端连接地址）
wechat --host 192.168.1.100 --port 18521 sessions

# 启动 daemon 时自定义绑定地址（--daemon-host/--daemon-port）
# 默认 --daemon-host 0.0.0.0（监听所有网卡），--daemon-port 18521
pm2 start wechat --name wechatd -- --daemon-host 0.0.0.0 --daemon-port 3333 --log-level info
```

## 反馈

```bash
wechat feedback "描述问题"          # 记录到本地 ~/.wechat/feedback.jsonl
wechat feedback "描述问题" --open   # 记录并打开浏览器预填 GitHub issue
wechat feedback --upload            # 展示所有本地记录并批量上传（打开浏览器）
```

---

## 模式说明

| 模式 | 触发条件 | 特点 |
|------|---------|------|
| daemon | `wechat` | 长驻，监听 WAL 变化，推送新消息 |
| client | 有子命令，daemon 运行中 | 连接 daemon 查询 |
| direct | daemon 未运行 / `--direct` | 自动 fallback，默认复用缓存（快） |

Direct 模式缓存语义：首次运行解密并写入 `_mtimes.json`，后续直接复用（< 100ms）。需要最新数据时：`WECHAT_REFRESH=1 wechat sessions`

## 联系人 local_type 说明

| local_type | 含义 |
|---|---|
| 0 | 公众号订阅 / 仅出现在群里的人 / 系统账号 |
| 1 | 微信官方服务账号 |
| 2 | 群聊 |
| 3 | 真实好友 |
| 5/6 | 企业微信联系人（@openim）|
