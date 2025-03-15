## 变量与数据类型

### 变量定义

> 简单使用

```go
package main

import "fmt"

func main() {
    var name = "lisi"
    
    fmt.Println(name)
}
```

> 变量的定义

Go语言是静态类型语言，就是所有的类型我们都需要明确的去定义。我们这里先不管其他类型，先了解 string，我们用它来表示字符串！在Go语言中，我们声明一个变量一般是使用var关键字：

```go
var name type
```

+ `var`：是声明变量的关键字，是固定的写法，大家记住即可，你要声明一个变量，就需要一个var.
+ `name`：就是我们变量的名字，你可以按照自己的需求给它定一个名字。用来后续使用！
+ `type`：就是用来代表变量的类型。

Go语言和许多编程语言不同，它在声明变量时将==变量的类型放在变量的名称之后==。这样做的好处就是可以避免像C语言中那样含糊不清的声明形式。

```go
var a, b *int
```

变量的命名规则遵循骆驼命名法，即首个单词小写，每个新单词的首字母大写.

> 批量定义变量

```go
var (
    addr string
    phone string
    age int
)
```

当一个变量被声明之后，如果没有显示的给他赋值，系统自动赋予它该类型的零值：

+ 整型和浮点型变量的默认值为 0 和 0.0
+ 字符串变量的默认值为空字符串
+ 布尔型变量默认为 false
+ 切片、函数、指针变量的默认为 nil

### 变量初始化

> 变量的初始化标准格式

```
var 变量名 类型 = 值（表达式）
```

比如：

```go
var name string = "hemou"
var age int = 18
```

> 短变量声明与初始化

```go
package main

import "fmt"

func main() {
    // 类型自动推导
    name := "hemou"
    age := 18
    
    fmt.Println(name, age) // hemou 18
    fmt.Println("%T,%T", name, age) // string,int
}
```

这是G0语言的推导声明写法，编译器会自动根据右值类型推断出左值的对应类型。

它可以自动的推导出一些类型，但是使用也是有限制的；

+ 定义变量，同时显式初始化。
+ 不能提供数据类型
+ 只能用在**函数内部**。不能随便到处定义(关于函数，我们后面会讲解，听一下就好这里)

因为简洁和灵活的特点，简短变量声明被广泛用于大部分的**局部变量的声明和初始化**。

**注意**：由于使用了 `:=`，而不是赋值的 `=`，因此推导声明写法的**左值变量必须是没有定义过的变量**。若定义过，将会发生编译错误。

```go
func main() {
	var name string
	
	name := "hemou"
}
```

这时会编译报错

### 变量内存地址

```go
func main() {
	var num int
    num = 100
    
    fmt.Printf("num: %d, 内存地址：%p", num, &num)
}
```

### 匿名变量

匿名变量的特点是一个下画线 `_`，`_` 本身就是一个特殊的标识符，被称为空白标识符。它可以像其他标识符那样用于变量的声明或赋值（任何类型都可以赋值给它），但任何赋给这个标识符的值都将被抛弃，因此这些值不能在后续的代码中使用，也不可以使用这个标识符作为变量对其它变量进行赋值或运算。使用匿名变量时，只需要在变量声明的地方使用下画线替换即可。例如：

```go
package main

import "fmt"

func test() (int, int) {
	return 10, 20
}

func main() {
	a, _ := test()
	_, b := test()

	fmt.Print(a, b)
}
```

在编码过程中，可能会遇到没有名称的变量、类型或方法。虽然这不是必须的，但有时候这样做可以极大地增强代码的灵活性，这些变量被统称为匿名变量。

匿名变量不占用内存空间，不会分配内存。匿名变量与匿名变量之间也不会因为多次声明而无法用。

## 运算符

1. 算术运算符：+，-，*，/，%，++，--
2. 赋值运算符：=，+=，~=，*=，/=，%=
3. 关系运算符：==，!=，>，<，>=，<=

4. 逻辑运算符：&&，||，!

5. 位运算符：&，|，^

6. 其它运算符：&，*

### 变量交换

```go
var (
    a int = 1
    b int = 2
)

a, b = b, a
```

## 流程控制

### for 循环

```go
func main() {
    // 普通循环
	for i := 0; i < 10; i++ {
		fmt.Println(i)
	}
    // 类似 while 写法
    i := 0
    for i < 10 {
        fmt.Println(i)
        i++
    }
    // 死循环
    for {
        
    }
    for ;; {
        
    }
    // for range
    str := "Hello, World!"
	for idx, val := range str {
		fmt.Printf("idx: %d, val: %c\n", idx, val)
	}
    // 只需要索引
    for i := range str {
        fmt.Println(i)
    }

    // 只需要值
    for _, val := range str {
        fmt.Printf("%c\n", val)
    }
    // 遍历数组/切片
    arr := []int{1, 2, 3}
    for i, v := range arr {
        fmt.Printf("索引：%d, 值：%d\n", i, v)
    }

    // 遍历 map
    m := map[string]int{"a": 1, "b": 2}
    for k, v := range m {
        fmt.Printf("键：%s, 值：%d\n", k, v)
    }
}
```

## 函数

### defer 关键字

函数中，程序员经常需要创建资源，为了在函数执行完毕后，及时的释放资源，Go的设计者提供defer关键字

```go

func add(a int, b int) int {
    // 在Golang中，程序遇到defer关键字，不会立即执行 defer 后的语句
    // 而是将 defer 后的语句压入一个栈中，然后继续执行函数后面的语句
	defer fmt.Println("a =", a)
	defer fmt.Println("b =", b)
	sum := a + b
	fmt.Println("sum =", sum)
	return sum
}

func main() {
	fmt.Println(add(1, 2))
}
// output
sum = 3
b = 2
a = 1
3
```

需要注意的是，defer 会将后面的代码语句压入栈中，也会将相关的值同时拷贝入栈中，不会随着函数后面的变化而变化。

```go
func add(a int, b int) int {
	defer fmt.Println("a =", a)
	defer fmt.Println("b =", b)

	a += 20
	b += 30

	sum := a + b
	fmt.Println("sum =", sum)
	return sum
}

func main() {
	fmt.Println(add(1, 2))
}
// output
sum = 53
b = 2
a = 1
53
```

