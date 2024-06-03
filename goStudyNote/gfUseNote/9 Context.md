[toc]

官方文档: [Context: 业务流程共享变量 - GoFrame (ZH)-Latest - GoFrame官网 - 类似PHP-Laravel, Java-SpringBoot的Go企业级开发框架](https://goframe.org/pages/viewpage.action?pageId=3672552)

# Context



## 结构定义

结构体中需要包含业务中传递的信息，这里包含了会话（Session）、用户信息（User）以及一个键值对映射（Data）。

新建`model/context.go`文件：

```
├── model
│   |   ├── do
│   │   ├── entity
│   │   └── context.go
```

```go
package model

import (
	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/net/ghttp"
)

const (
	// 上下文变量存储键名，前后端系统共享
	ContextKey = "ContextKey"
)

// 请求上下文结构
type Context struct {
	Session *ghttp.Session // 当前Session管理对象
	User    *ContextUser   // 上下文用户信息
	Data    g.Map          // 自定KV变量，业务模块根据需要设置，不固定
}

// 请求上下文中的用户信息
type ContextUser struct {
	Id       uint   // 用户ID
	Passport string // 用户账号
	Nickname string // 用户名称
	Avatar   string // 用户头像
}
```

## 逻辑封装

封装上下文的相关操作。这包括初始化上下文、获取上下文和设置用户信息等功能。

新建`logic/context/context.go`文件

```
├── logic
│   │   ├── context
│   │   │   └── context.go
│   │   └── logic.go
```

```go
package context

import (
	"context"
	"demo/internal/model"
	"demo/internal/service"

	"github.com/gogf/gf/v2/net/ghttp"
)

func init() {
	service.RegisterContext(New())
}

type sContext struct{}

func New() *sContext {
	return &sContext{}
}

// 初始化上下文对象指针到上下文对象中，以便后续的请求流程中可以修改。
func (s *sContext) Init(r *ghttp.Request, customCtx *model.Context) {
	r.SetCtxVar(model.ContextKey, customCtx)
}

// 获得上下文变量，如果没有设置，那么返回nil
func (s *sContext) Get(ctx context.Context) *model.Context {
	value := ctx.Value(model.ContextKey)
	if value == nil {
		return nil
	}
	if localCtx, ok := value.(*model.Context); ok {
		return localCtx
	}
	return nil
}

// 将上下文信息设置到上下文请求中，注意是完整覆盖
func (s *sContext) SetUser(ctx context.Context, ctxUser *model.ContextUser) {
	s.Get(ctx).User = ctxUser
}
```

## 上下文变量注入

上下文变量的注入通过定义一个中间件来实现。这个中间件在请求的最开始执行，用于初始化上下文，将请求中携带的参数置入结构体中，以便后续的处理函数可以便捷调用。

新建`logic/middleware/middleware.go`文件

```
├── logic
│   │   ├── context
│   │   ├── middleware
│   │   │   └── middleware.go
│   │   └── logic.go
```

```go
package middleware

import (
	"demo/internal/model"
	"demo/internal/service"

	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/net/ghttp"
)

type sMiddleware struct{}

func init() {
	service.RegisterMiddleware(New())
}

func New() *sMiddleware {
	return &sMiddleware{}
}

func (*sMiddleware) Ctx(r *ghttp.Request) {
	// 初始化，务必最开始执行
	customCtx := &model.Context{
		Session: r.Session,
		Data:    make(g.Map),
	}
	service.Context().Init(r, customCtx)

	// TODO 获取用户信息
	// ...

	// 将自定义的上下文对象传递到模板变量中使用
	r.Assigns(g.Map{
		"Context": customCtx,
	})
	// 执行下一步请求逻辑
	r.Middleware.Next()
}
```

## 上下文变量使用

可以通过以下方式获取上下文：

```go
service.Context().Get(ctx)
```

可以通过 Data 字段存储和获取自定义的键值对数据：

```go
// 设置自定义键值对
service.Context.Get(ctx).Data[key] = value
// 获取自定义键值对
service.Context.Get(ctx).Data[key]
```

