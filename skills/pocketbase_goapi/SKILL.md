---
name: "pocketbase"
description: "PocketBase Go 扩展参考和开发模式.当用户使用 PocketBase 钩子、路由或数据库操作时调用."
---

# PocketBase Go API 参考

## 核心结构

```go
import (
    "github.com/pocketbase/pocketbase"
    "github.com/pocketbase/pocketbase/core"
)

func main() {
    app := pocketbase.New()

    app.OnServe().BindFunc(func(se *core.ServeEvent) error {
        se.Router.GET("/api/hello", func(e *core.RequestEvent) error {
            return e.JSON(200, map[string]string{"message": "hello"})
        })
        return se.Next()
    })

    app.Start()
}
```

## 主要事件钩子

```go
app.OnServe()              // 服务启动时
app.OnRecordCreate()       // 记录创建
app.OnRecordUpdate()       // 记录更新
app.OnRecordDelete()       // 记录删除
app.OnRecordAuth()         // 用户认证
app.OnRecordUpdateRequest("collection")  // API 更新请求
app.OnRecordAuthRequest("collection")    // API 认证请求
```

## RequestEvent 核心方法

```go
e.Auth              // 当前认证的用户记录
e.Request()         // HTTP 请求
e.Response()        // HTTP 响应

// 响应
e.JSON(code, data)
e.String(code, text)
e.NoContent(code)
e.Redirect(code, url)

// 错误
e.BadRequest(message, data)
e.Unauthorized(message, data)
e.Forbidden(message, data)
e.NotFound(message, data)

// 中间件链
e.Next()
```

## 路由中间件

```go
se.Router.GET("/api/path", handler).
    Bind(middleware1).
    Bind(middleware2)

func middleware(e *core.RequestEvent) error {
    if err := e.Next(); err != nil {
        return err
    }
    return nil
}
```

## 数据库操作

```go
// 查询
record, _ := app.FindRecordById("users", "id123")
records, _ := app.FindRecordsByFilter("users", "type = 'admin'", "-created", 10)

// 创建
collection, _ := app.FindCollectionByNameOrId("users")
record := core.NewRecord(collection)
record.Set("name", "John")
app.Save(record)

// 更新
record.Set("name", "Jane")
app.Save(record)

// 删除
app.Delete(record)
```

## Hook 示例

```go
func RegisterUserHooks(app core.App) {
    // 防止非管理员篡改角色
    app.OnRecordUpdateRequest("users").BindFunc(func(e *core.RecordRequestEvent) error {
        if e.Auth != nil && e.Auth.GetString("role") == "admin" {
            return e.Next()
        }

        oldRole := e.Record.Original().GetString("role")
        newRole := e.Record.GetString("role")

        if oldRole != "" && oldRole != newRole {
            e.Record.Set("role", oldRole)
        }
        return e.Next()
    })

    // 自动恢复软删除用户
    app.OnRecordAuthRequest("users").BindFunc(func(e *core.RecordAuthRequestEvent) error {
        if e.Record != nil && e.Record.GetBool("isDeleted") {
            e.Record.Set("isDeleted", false)
            e.Record.Set("deletedAt", nil)
            e.Record.Set("deletedBy", "")
            return app.Save(e.Record)
        }
        return e.Next()
    })
}
```

## 项目结构示例

```go
func main() {
    app := pocketbase.New()

    // 迁移命令
    migratecmd.MustRegister(app, app.RootCmd, migratecmd.Config{
        Automigrate: true,
    })

    // 数据库钩子
    hooks.RegisterUserHooks(app)
    hooks.RegisterArticleHooks(app)

    // 自动任务
    hooks.RegisterCronJobs(app)

    // 自定义路由
    app.OnServe().BindFunc(func(se *core.ServeEvent) error {
        handlers.RegisterAvatarRoutes(app, se)
        return se.Next()
    })

    jsvm.MustRegister(app, jsvm.Config{})
    app.Start()
}
```

## API 变化（旧版对比）

| 旧版 | 新版 |
|------|------|
| `OnBeforeServe()` | `OnServe()` |
| `Add()` | `BindFunc()` |
| `echo.Context` | `core.RequestEvent` |
| `apis.*` 函数 | `RequestEvent` 方法 |
| 中间件 | `.Bind()` 链式调用 |
| - | `e.Auth` 直接获取认证用户 |

## 参考
- [PocketBase 官方文档](https://pocketbase.io/docs/go-overview/)