# 开始

## 环境配置

+ `GOROOT`：go 的路径，目录结构通常如下
  + `bin/`：包含可执行的 Go 工具，例如 `go` 和 `gofmt`。
    + `go`：Go 的核心命令行工具，用于编译、运行和管理 Go 包。
    + `gofmt`：Go 代码格式化工具。

  + `pkg/`：存放编译后的包文件和标准库的中间文件。
    + `tool/`：存储与 Go 编译器和工具相关的辅助文件。
    + `<platform>`：包含特定平台的编译库文件，例如 `linux_amd64` 或 `windows_amd64`。

  + `src/`：包含 Go 的标准库源码。
    + 特点：按包名组织的目录结构，例如 `src/fmt` 对应 `fmt` 包，`src/net/http` 对应 `net/http` 包。

  + `lib/`：存储运行时所需的库文件或静态文件。
  + `doc/`：包含与 Go 相关的文档和示例。

+ `GOPATH`：之后生成的exe、pkg等文件所存放的路径

## HelloWorld

```go
// 文件位于：xxx/GoLang/main.go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

然后前往 `xxx/GoLang`目录运行  `go run main.go`

其中：

+ `package main`：在Go语言里，命名为main的包具有特殊的含义。Go语言的编译程序会试图把这种名字的包编译为二进制可执行文件。所有用Go语言编译的可执行程序都必须有一个名叫main的包。一个可执行程序有且仅有一个main包。
+ `import "fmt"`：该包包含格式化文本的功能，包括打印到控制台。

## 语法注意事项

1. 基本语法规则
   + 语句结尾不需要分号
   + 左大括号不能单独一行
   + 源文件以"go"为扩展名。
   + 程序的执行入口是main0函数。
   + 严格区分大小写。

```go
// 1. 语句结尾不需要分号
func main() {
    name := "张三"
    fmt.Println(name)    // 正确
    fmt.Println(name);   // 不推荐
}

// 2. 左大括号不能单独一行
func main() {    // 正确
    
}

func main()      // 错误
{
    
}
```

2. 变量声明

变量如果声明后未使用，编译器会报错

```go
// 1. 变量声明方式
var name string = "张三"    // 完整声明
var name = "张三"          // 类型推导
name := "张三"            // 简短声明（只能在函数内使用）

// 2. 未使用的变量会报错
func main() {
    a := 1      // 如果声明后未使用，编译器会报错
    fmt.Println(a)
}

// 3. 多变量声明
var (
    name = "张三"
    age  = 25
    city string
)
```

## 注释

单行注释：

```go
// 单行注释
```

多行注释：

```go
/*
多行注释
*/
```

## 编码风格

+ 变量名推荐使用驼峰命名法

+ 首字母大写的标识符会被导出（公开）

+ 首字母小写的标识符是私有的

## 参考文档

+ [Documentation - The Go Programming Language](https://go.dev/doc/)

+ [Golang中文社区](https://studygolang.com/)