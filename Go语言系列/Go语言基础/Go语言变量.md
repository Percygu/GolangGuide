---
tags:
  - Go
  - golang
  - go基础语法
  - 变量
---

# Go语言变量

Go 语言变量名由字母、数字、下划线组成，其中首个字符不能为数字。
声明变量的一般形式是使用 var 关键字：
```go
var identifier type
```
也可以同时声明多个变量
```go
var identifier1, identifier2 type
```
```go
package main
import "fmt"
func main() {
    var a string = "Hello"
    fmt.Println(a)

    var b, c int = 1, 2
    fmt.Println(b, c)
}
```
以上实例输出结果为：
```shell
Hello
1 2
```

## 变量声明

### 第一种，指定变量类型，如果没有初始化，则变量默认为零值
零值就是变量没有做初始化时系统默认设置的值。
```go
package main
import "fmt"
func main() {
    var a string = "Hello" // 声明一个变量并初始化
    fmt.Println(a)

    var b int    // 没有初始化就为零值
    fmt.Println(b)

    var c bool   // bool 零值为 false
    fmt.Println(c)
}
```
以上实例执行结果为：
```shell
Hello
0
false
```
- 数值类型（包括complex64/128）为 0
- 布尔类型为 false
- 字符串为 ""（空字符串）
- 以下几种类型为 nil：
```go
package main

import "fmt"

var a *int
var b []int
var c map[string]int
var d chan int
var e func(string) int
var f error

func main() {
    fmt.Println("Hello World!")
    fmt.Println(a == nil)
    fmt.Println(b == nil)
    fmt.Println(c == nil)
    fmt.Println(d == nil)
    fmt.Println(e == nil)
    fmt.Println(f == nil)
}
```
输出结果是
```shell
Hello World!
true
true
true
true
true
true
```

```go
package main

import "fmt"

func main() {
    var i int
    var f float64
    var b bool
    var s string
    fmt.Printf("%v %v %v %q\n", i, f, b, s)
}
```
输出结果是
```shell
0 0 false ""
```

### 第二种，根据值自行判定变量类型
```go
var v_name = value
```
```go
package main
import "fmt"
func main() {
    var d = true
    fmt.Println(d)
}
```
输出结果是：
```shell
true
```

### 第三种，:= 声明变量
```go
v_name := value
```
**intVal := 1** 等价于：
```go
var intVal int = 1
```
可以将 `var intVal int = 1"` 简写为 `ntVal := 1`。如果变量已经使用 `var` 声明过了，再使用 `:=` 声明变量，就产生编译错误
```go
var intVal int 
intVal :=1 // 这时候会产生编译错误，因为 intVal 已经声明，不需要重新声明
```
直接使用下面的语句即可：
```go
intVal := 1 // 此时不会产生编译错误，因为有声明新的变量，因为 := 是一个声明语句
```

```go
package main
import "fmt"
func main() {
    f := "Hello" // var f string = "Hello"
    fmt.Println(f)
}
```
输出结果是：
```shell
Hello
```

### 多变量声明
```go
//类型相同多个变量, 非全局变量
var vname1, vname2, vname3 type
vname1, vname2, vname3 = v1, v2, v3

var vname1, vname2, vname3 = v1, v2, v3 // 和 python 很像,不需要显示声明类型，自动推断

vname1, vname2, vname3 := v1, v2, v3 // 出现在 := 左侧的变量不应该是已经被声明过的，否则会导致编译错误


// 这种因式分解关键字的写法一般用于声明全局变量
var (
    vname1 v_type1
    vname2 v_type2
)
```
```go
package main

var x, y int
var ( // 这种因式分解关键字的写法一般用于声明全局变量
    a int
    b bool
)

var c, d int = 1, 2
var e, f = 123, "hello"

//这种不带声明格式的只能在函数体中出现
//g, h := 123, "hello"

func main() {
    g, h := 123, "hello"

    println(x, y, a, b, c, d, e, f, g, h)
}
```
以上实例执行结果为：
```shell
0 0 0 false 1 2 123 hello 123 hello
```
### 注意事项
我们知道可以在变量的初始化时省略变量的类型而由系统自动推断，声明语句写上 var 关键字其实是显得有些多余了，因此我们可以将它们简写为`a := 50` 或 `b := false`。`a` 和 `b` 的类型（int 和 bool）将由编译器自动推断。

这是使用变量的首选形式，但是它只能被用在函数体内，而不可以用于全局变量的声明与赋值。使用操作符 `:=` 可以高效地创建一个新的变量，称之为初始化声明。

如果在相同的代码块中，我们不可以再次对于相同名称的变量使用初始化声明，例如：`a := 20` 就是不被允许的，编译器会提示错误 `no new variables on left side of :=`，但是 `a = 20` 是可以的，因为这是给相同的变量赋予一个新的值。

如果你在定义变量 `a` 之前使用它，则会得到编译错误 `undefined: a`。 如果你声明了一个局部变量却没有在相同的代码块中使用它，同样会得到编译错误，例如下面这个例子当中的变量 `a`：
```go
package main

import "fmt"

func main() {
   var a string = "abc"
   fmt.Println("hello, world")
}
```
尝试编译这段代码将得到错误：**`a declared but not used`**，此外，单纯地给 `a` 赋值也是不够的，这个值必须被使用，所以使用
```go
fmt.Println("hello, world", a)
```
会移除错误。但是全局变量是允许声明但不使用的。 同一类型的多个变量可以声明在同一行，如：
```go
var a, b, c int
```
多变量可以在同一行进行赋值，如：
```go
var a, b int
var c string
a, b, c = 5, 7, "abc"
```
上面这行假设了变量 `a`，`b` 和 `c` 都已经被声明，否则的话应该这样使用：
```go
a, b, c := 5, 7, "abc"
```
右边的这些值以相同的顺序赋值给左边的变量，所以 `a` 的值是 `5`， `b` 的值是 `7`，`c` 的值是 `abc`。 这被称为并行或同时赋值。

如果你想要交换两个变量的值，则可以简单地使用 `a, b = b, a`，两个变量的类型必须是相同。

空白标识符`_`也被用于抛弃值，如值`5`在`_, b = 5, 7`中被抛弃。`_`实际上是一个只写变量，你不能得到它的值。这样做是因为Go语言中你必须使用所有被声明的变量，但有时你并不需要使用从一个函数得到的所有返回值。

并行赋值也被用于当一个函数返回多个返回值时，比如这里的`val`和错误`err`是通过调用`Func1`函数同时得到：`val, err = Func1(var1)`。

## 变量的生命周期
全局变量生命周期是程序存活时间，在不发生内存逃逸的情况下，局部变量是函数存活时间
```go
package main

import "fmt"

var globalStr string
var globalInt int

func main() {
   var  localStr string
   var  localInt int
   localStr = "first local"
   localInt = 2021
   globalInt = 1024
   globalStr = "first global"
   fmt.Printf("globalStr is %s\n", globalStr)   //globalStr is first global
   fmt.Printf("globalStr is %d\n", globalInt)   //globalStr is 1024
   fmt.Printf("localInt is %s\n", localStr)     //localInt is first local
   fmt.Printf("localInt int is %d\n", localInt) //localInt int is 2021
}
```











