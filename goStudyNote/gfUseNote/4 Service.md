# Server层

涉及文件夹： `internal `下的 `service`与`logic` 文件夹

## Service 文件夹

- 创建 `example.go` 文件

```go
package service

import ...

// 1. 定义接口

type IExample interface {
  Function(...) 
}

// 2.定义接口变量

var localExample IExample

// 3.定义获取接口实例函数

func Example() IExample {

  if local_ == nil {
     panic("Example interface unrealized")
  }
    
  return localExample // 返回接口变量
}
// 4.定义接口实现注册方法

func RegisterExample(i IExample) {
  localExample = i
}
```



## Logic 文件夹

- 在 `Logic` 文件夹下创建 `logic.go`  文件

```go
package logic

import(
	_ "example_object/internal/logic/example" //初始化接口实现包
)
```

- 在 `main.go` 初始化 `logic`
`import _ "union_demo/internal/logic`

- 创建 `example` 文件夹，并在文件夹下创建 `example.go`
```go
package example

import ...

// 注册接口
func init() {
	service.RegisterExample(&sExample{})
}

type sExample struct{}

// 具体实现
func (* sExample)Function(...){
	......
}

```