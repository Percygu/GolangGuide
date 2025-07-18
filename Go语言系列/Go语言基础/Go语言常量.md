---
tags:
  - Go
  - golang
  - go基础语法
  - 常量
---

# Go语言常量

## 常量的定义

Go语言常量的定义跟其他语言类似，是一个简单值的标识符，在程序运行时，不可以被修改。常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型。常量的定义方式：

```go
const identifier [type] = value
```
其中类型说明符[type]可以省略，因为编译器可以根据变量的值来推断其类型。

```go
const b string = "abc"   // 显式常量定义
const b = "abc"          // 隐式常量定义 
```
多个相同类型的声明可以简写为：`const c_name1, c_name2 = value1, value2`
```go
const a, b = "abc", "def"
```

## 常量用作枚举

Go语言不像C++Java一样有专门的枚举类型，Go语言的枚举一般用常量表示。
```go
const (
    Unknown = 0
    Success = 1
    Fail = 2
)
```
数字 0、1 和 2 分别代表未知成功和失败
常量可以用len(), cap(), unsafe.Sizeof()函数计算表达式的值。常量表达式中，必须是编译期可以确定的值，否则编译不过
```go
package main

import (
    "fmt"
    "unsafe"
)

const (
    a = "abc"
    b = len(a)
    c = unsafe.Sizeof(a)
)

func main() {
    fmt.Printf("a = %s, b = %d, c = %d\n", a, b, c)
}
```
以上实例运行结果为：
```
abc 3 16
```
**思考题：为什么字符串"abc"的 `unsafe.Sizeof()` 是16呢?**
实际上字符串类型的`unsafe.Sizeof()`一直是16，这又是为什么呢？因为字符串类型对应一个结构体，该结构体有两个域，第一个域是指向该字符串的指针，第二个域是字符串的长度，每个域占8个字节，但是并不包含指针指向的字符串的内容，这也就是为什么`sizeof`始终返回的是16。
字符串可以理解成此结构体
```c
typedef struct {
    char* buffer;    // 8字节
    size_tlen;       // 8字节
} string;
```

## iota
`iota`是一个特殊常量，可以认为是一个可以被编译器修改的常量。`iota`在`const`关键字出现时将被重置为`0`(const 内部的第一行之前)，`const`中每新增一行常量声明将使`iota`计数一次(`iota`可理解为`const`语句块中的行索引)
看例子：

```go
const (
    a = iota
    b = iota
    c = iota
)
```
第一个`iota`等于`0`，每当`iota`在新的一行被使用时，它的值都会自动加1；所以`a=0, b=1, c=2`   可以简写为如下形式：
```go
const (
    a = iota
    b
    c
)
```
具体使用：
```go
package main

import "fmt"

func main() {
    const (
        a = iota      // 0
        b             // 1
        c             // 2
        d = "ha"      // 独立值，iota = 3
        e             // "ha"，iota = 4
        f = 100       // iota = 5
        g             // 100，iota = 6
        h = iota      // 7，恢复计数
        i             // 8
    )
    fmt.Println(a, b, c, d, e, f, g, h, i)
}
```
以上实例运行结果为：
```
0 1 2 ha ha 100 100 7 8
```
关于iota的一个有趣的例子：
```go
package main

import "fmt"
const (
    i=1<<iota    // iota = 0, i = 1 << 0 = 1
    j=3<<iota    // iota = 1, j = 3 << 1 = 6
    k            // iota = 2, k = 3 << 2 = 12
    l            // iota = 3, l = 3 << 3 = 24
)

func main() {
    fmt.Println("i=", i)
    fmt.Println("j=", j)
    fmt.Println("k=", k)
    fmt.Println("l=", l)
}
```
以上实例运行结果为：
```
i= 1
j= 6
k= 12
l= 24
```
<<表示左移，学过计算机我们都知道，左移以为其实就是乘以2，左移n位其实就是乘以2的n次方。所以：
- i=1：左移 0 位，不变仍为 1，即1*2^0=1*1=1
- j=3：左移 1 位，变为二进制 110，即 3*2^1=3*2=6
- k=3：左移 2 位，变为二进制 1100，即3*2^ 2=3*4=12
- l=3：左移 3 位，变为二进制 11000，即 3*2^ 3=3*8=24