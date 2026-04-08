# public_tools

![CI](https://github.com/forechoandlook/public_tools/actions/workflows/ci.yml/badge.svg)

统一发布的二进制工具集合。

## 工具列表

| 工具 | 源码仓库 | 功能 | 版本 |
|------|----------|------|------|
| wechat | [wechat-decrypt-rs](https://github.com/forechoandlook/wechat-decrypt-rs) | 微信数据库解密 + CLI 查询工具 | v0.0.1 |

## 下载

前往 [Releases](https://github.com/forechoandlook/public_tools/releases) 下载对应平台的二进制文件。

## wechat

微信 macOS 数据库解密与查询工具，支持：

- 会话列表、消息历史、联系人搜索
- 群聊管理、朋友圈、收藏、公众号文章
- Daemon 模式实时推送新消息
- WAL 增量更新，不重复解密

### 使用方法

```bash
# 直接查询（自动解密）
./wechat sessions --limit 20
./wechat history "联系人名" --limit 50

# 启动 Daemon 实时监控
./wechat --log-level debug

# 更多命令
./wechat --help
```

详细文档见 [wechat-decrypt-rs](https://github.com/forechoandlook/wechat-decrypt-rs)。
