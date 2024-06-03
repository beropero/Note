# Controller层

涉及文件夹 ：`internal` 下的 `controller` 文件夹和 `api` 文件夹

## api 文件夹

在 `api` 文件夹下创建 ` hi/v1`文件夹，并在 `v1` 文件夹下创建`v1`文件夹，`v_`指的是该api的版本号

目录结构如下：

```
├── api
│   ├── hi
│   │   ├── v1
│   │   │   ├── hi.go
```

在hi.go中添加需要的请求结构体和响应结构体，命名与规范路由的写法一致，请求以Req结尾，响应以Res结尾。注：首字母需要大写，不然其他包访问不了会爆红。

```go
package v1

import "github.com/gogf/gf/v2/frame/g"

type HiReq struct {
	g.Meta `path:"/hi" method:"get"`
}

type HiRes struct {
	Msg string
}
```

## controller文件夹

在 `api` 中写入请求结构体与响应结构体后，可以用 `GoFrame` 框架的代码生成工具 `gen` 快速生成 `controller` 文件。

在终端中输入指令

```
gf gen ctrl
```



![image-20240328194640660](E:\workplace\Markdown\goStudyNote\gfUseNote\3 Controller.assets\image-20240328194640660.png)

可以看到生成了四个文件。

进入 `hi_v1_hi.go` 文件, 在方法中添加具体实现的代码

```go
package hi

import (
	"context"

	"demo/api/hi/v1"
)

func (c *ControllerV1) Hi(ctx context.Context, req *v1.HiReq) (res *v1.HiRes, err error) {
	res = &v1.HiRes{
		"hi",
	}
	return
}
```

自动生成的 `hi_new.go` 文件含有获取接口实现的函数，可以在 `cmd.go` 中进行路由注册
```
// =================================================================================
// This is auto-generated by GoFrame CLI tool only once. Fill this file as you wish.
// =================================================================================

package hi

import (
	"demo/api/hi"
)

type ControllerV1 struct{}

func NewV1() hi.IHiV1 {
	return &ControllerV1{}
}
```

示例:

```go
Func: func(ctx context.Context, parser *gcmd.Parser) (err error) {
			s := g.Server()
			s.Use(ghttp.MiddlewareHandlerResponse)
			s.Group("/", func(group *ghttp.RouterGroup) {
				group.Bind(
					hi.NewV1(),
				)
			})
			s.Run()
			return nil
		},
```

运行后可以访问[127.0.0.1:8000/hi](http://127.0.0.1:8000/hi)看到响应结果：

![image-20240328200420234](E:\workplace\Markdown\goStudyNote\gfUseNote\3 Controller.assets\image-20240328200420234.png)