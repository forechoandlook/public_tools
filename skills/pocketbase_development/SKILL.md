---
name: pocketbase-pro
description: PocketBase 工程化专家（DSL + 最佳实践合体版）。支持 schema DSL 编写、JSON 生成、diff/导入、collection 设计、安全规则、性能优化、软删除规范、常见陷阱规避。适用于中大型项目、SaaS、工具型产品。
---
# PocketBase Pro Skill

一次性给出**安全、可维护、高性能、可重复迁移**的PocketBase项目方案。

## collection 定义 的 DSL 核心语法

### Collection 基本结构

```ts
{
  id: 'pbc_xxx',                 // 必须稳定，勿改
  name: 'posts',
  type: 'base' | 'auth',
  rules: { list?, view?, create?, update?, delete?, manage? },
  fields: Field[],
  indexes?: Index[],
  extra?: object                 // auth 时常用
}
```

### 字段工厂（fields.ts）

```ts
f.text(name, { required?, min?, max?, pattern?, unique? })
f.bool(name, { default? })
f.date(name, { required? })
f.autodate(name, { onCreate?, onUpdate? })
f.number(name, { min?, max?, decimals? })
f.select(name, values: string[], { maxSelect?, required? })
f.relation(name, collectionId, { maxSelect?, required?, cascadeDelete? })
f.file(name, { maxSize?, thumbs?, mimeTypes?, multiple? })
f.json(name)
f.email / f.url / f.password
f.authFields()           // auth 集合默认字段
f.authExtra(overrides?)  // auth 额外配置
```

### 规则工厂（rules.ts）

如果没有既有描述 使用 PocketBase Rule 字符串构造器

```ts
r.admin // 会翻译为：@request.auth.role = "admin"
r.user // @request.auth.id != ""
r.verified // @request.auth.verified = true
r.isMe // id = @request.auth.id
r.isOwner(field = 'user')
r.notDeleted // isDeleted = false
r.published               // 常用于 status = "published"
r.and(...exprs)
r.or(...exprs)
```

### 索引推荐格式

```ts
{ fields: ['user', 'status'], unique?: boolean, name?: string }
```

## 软删除 & 安全基线（默认强制）

几乎所有业务 collection 都应该包含：

```ts
fields: [
  f.bool('isDeleted',   { default: false }),
  f.date('deletedAt'),
  f.relation('deletedBy', '_pb_users_auth_', { maxSelect: 1 }),
  // 业务字段...
]
```

规则基线（强烈推荐）：

```ts
rules: {
  list:  r.and(r.notDeleted, /* 业务条件 */),
  view:  r.and(r.notDeleted, /* 业务条件 */),
  delete: r.admin,           // 业务禁止物理删除
}
```

## ### demo/template

```
import { f } from '/Users/zzwy/pbutils/fields';
import { r } from '/Users/zzwy/pbutils/rules';

const USERS_ID = '_pb_users_auth_';
export const collections = [
    // --- 1. USERS (System Auth ) ---
    {
        id: USERS_ID,
        name: 'users',
        type: 'auth',
        rules: {
            list: r.or(r.admin, r.and(r.isMe, r.notDeleted)),
            view: r.or(r.admin, r.and(r.isMe, r.notDeleted)),
            create: `role = "free" || role = ""`,
            update: r.and(r.isMe, r.notDeleted, `(@request.query.role:isset = false || @request.query.role = @request.auth.role)`),
            delete: r.admin
        },
        fields: [
            ...f.authFields(),
            f.select('role', ['admin', 'vip', 'free'], { required: false }),
            f.date('vipSince'),
            f.date('vipExpiresAt'),
            f.bool('autoRenew'),
            f.bool('notificationsEnabled'),
            f.bool('isDeleted'),
            f.date('deletedAt'),
            f.relation('deletedBy', USERS_ID, { maxSelect: 1 }),
            f.autodate('createdAt', { onCreate: true }),
            f.autodate('updatedAt', { onUpdate: true, onCreate: true })
        ],
        extra: f.authExtra() // 必写 但是也可以覆盖
    },

    // --- 1.1 USER PROFILES ---
    {
        id: 'pbc_2395140099',
        name: 'user_profiles',
        type: 'base',
        rules: {
            list: r.or(r.notDeleted, r.admin),
            view: r.or(r.notDeleted, r.admin),
            create: r.and(r.user, r.isOwner('user')),  // 移除 verified 要求，允许注册时创建
            update: r.and(r.user, r.verified, r.isOwner('user')),  // 保留 verified 要求
            delete: r.admin
        },
        fields: [
            f.relation('user', USERS_ID, { maxSelect: 1, required: true }),
            f.text('displayName'),
            f.text('nickname'),
            f.url('avatar'),
            f.text('bio'),
            f.text('badge'),
            f.bool('isDeleted'),
            f.date('deletedAt'),
            f.relation('deletedBy', USERS_ID, { maxSelect: 1 }),
            f.autodate('createdAt', { onCreate: true }),
            f.autodate('updatedAt', { onUpdate: true, onCreate: true })
        ],
        indexes: [
            { fields: ['user'], unique: true, name: 'idx_user_profiles_user' }
        ]
    },
]
```

## 常见场景推荐模板（直接改用）

### 1. 个人私有数据（笔记、待办）

```ts
{
  name: 'notes',
  type: 'base',
  rules: {
    list:  r.isOwner('user'),
    view:  r.isOwner('user'),
    create: r.user,
    update: r.isOwner('user'),
    delete: r.admin
  },
  fields: [
    f.text('title', { required: true }),
    f.text('content'),
    f.relation('user', '_pb_users_auth_', { required: true, maxSelect: 1 }),
    f.bool('isDeleted', { default: false }),
    f.date('deletedAt'),
    f.relation('deletedBy', '_pb_users_auth_', { maxSelect: 1 }),
  ],
  indexes: [
    { fields: ['user', 'isDeleted'], name: 'idx_notes_user_deleted' }
  ]
}
```

### 2. 团队/项目内可见（任务、文档）

```ts
list:  r.and(r.notDeleted, 'project.users ~ @request.auth.id'),
view:  r.and(r.notDeleted, 'project.users ~ @request.auth.id'),
update: r.and(r.notDeleted, 'project.users ~ @request.auth.id'),
```

### 3. 公开 + 拥有者可管理（文章、商品）

```ts
list:  r.and(r.notDeleted, 'status = "published" || user = @request.auth.id'),
view:  r.and(r.notDeleted, 'status = "published" || user = @request.auth.id'),
update: r.isOwner('user'),
```

### 4. Auth collection 推荐（生产级）

```ts
{
  name: 'users',
  type: 'auth',
  extra: f.authExtra({
    minPasswordLength: 10,
    requireEmailVerification: true
  }),
  rules: {
    list:   r.admin,
    view:   r.or(r.admin, r.isMe),
    create: r.admin,           // 关闭开放注册
    update: r.or(r.admin, r.isMe),
    delete: r.admin
  }
}
```

## 性能 & 索引高频推荐

必须考虑索引的字段组合：

- user + isDeleted
- user + status + createdAt / updatedAt
- project / organization + createdAt
- 常用 relation 字段
- status / type / category + 时间字段

## 常见翻车点 & 规避

- 公开读，如果部分字段公共读还需要权限，则会失效，应该走分表的操作，将公共读的部分放到另外一张表中，通过关联查询
- relation 用 name 而非 pbc_ id → 改名后全崩
- list/view 忘记 r.notDeleted → 已删记录永久可见
- deleteRule 设为 r.user 或 r.isOwner → 误删风险极高
- 文件字段无 maxSize / mime 限制 → 磁盘秒满
- 规则写得太复杂 → SQL 爆炸，响应 >5s
- 没 diff 就直接 import → 生产误删字段/规则
- admin token 做业务逻辑 → 泄露 = 全站沦陷

## 已有功能(已经全局安装)

1. `pb-schema-check --schema ./schema_def.ts` 仅检查本地 schema 是否合法（不连接远端）
2. `pb-schema-diff --schema ./schema_def.ts --url https://prod --email ... --password ...` 对比当前定义的schema和运行的schema的区别
3. `pb-schema-import ./schema_def.ts --import --url ... --email ... --password ...` 强制推送当前定义的schema到远端

- [ ] 业务表都有 isDeleted + deletedAt
- [ ] list/view 默认包含 r.notDeleted
- [ ] deleteRule 基本为 r.admin
- [ ] 所有 relation 使用 collection id
- [ ] 高频查询组合有索引
- [ ] 文件字段有限制（大小、类型）
- [ ] 生产关闭开放注册（或加严格防护）
- [ ] 有 pb_data 定时备份策略
