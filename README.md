# public_tools

统一发布的二进制工具集合。

private 仓库的release 会 copy一份到当前仓库，public的release需要到public仓库获取.

## 工具列表

| 工具 | 源码仓库 | 功能 | 命令 |
|------|----------|------|------|
| wechat | [wechat-decrypt-rs](https://github.com/forechoandlook/wechat-decrypt-rs)  private | 微信数据库解密 + CLI 查询工具 [文档](./wechat.md) | `npx skills add forechoandlook/public_tools --skill wechat`|
| gssh | [gssh](https://github.com/forechoandlook/gssh)  private | 命令行远程连接工具 [文档](./gssh.md) | `npx skills add forechoandlook/public_tools --skill gssh` |
| opencli_plus | [opencli](https://github.com/forechoandlook/opencli)  public | web/本地信息获取、执行的命令行工具 [文档](./opencli_plus.md) | `npx skills add forechoandlook/public_tools --skill opencli_plus` |
