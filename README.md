# public_tools

统一发布的二进制工具集合。

## 工具列表

| 工具 | 源码仓库 | 功能 | 版本 |
|------|----------|------|------|
| wechat | [wechat-decrypt-rs](https://github.com/forechoandlook/wechat-decrypt-rs) | 微信数据库解密 + CLI 查询工具 | v0.0.1 |

## 下载

前往 [Releases](https://github.com/forechoandlook/public_tools/releases) 下载。

**推荐安装方式：**

```bash
curl -sL https://raw.githubusercontent.com/forechoandlook/public_tools/main/install.sh | bash
```

安装脚本会自动检测平台（macOS ARM64、x86_64、Linux x86_64）、检查版本并安装到 `~/.local/bin`。

**已有二进制？直接更新：**

```bash
./wechat update
```

## wechat

微信 macOS 数据库解密与查询工具，支持：

- 会话列表、消息历史、联系人搜索
- 群聊管理、朋友圈、收藏、公众号文章
- Daemon 模式实时推送新消息
- WAL 增量更新，不重复解密
- `wechat update` 自更新

### 使用方法

```bash
./wechat sessions --limit 20
./wechat history "联系人名" --limit 50
./wechat update

# 启动 Daemon 实时监控
./wechat --log-level debug
./wechat --help
```

### 自更新

```bash
./wechat update
```

会自动从 `wechat-latest` release 下载最新版本并替换二进制。

详细文档见 [wechat-decrypt-rs](https://github.com/forechoandlook/wechat-decrypt-rs)。
