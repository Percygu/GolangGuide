---
tags:
  - Go
  - golang
  - go基础语法
  - 接口
---

# Go语言接口

## 什么是接口
Go语言中的接口很简单，就是一组方法的声明。当某一种类型实现了所有这些声明的方法，那么就称这种类型为该接口的一种实现。

## 接口定义
Go语言中同样用关键字``interfac`e`来定义，接口的定义格式如下：
```go
type 接口名 interface {
    方法名1(参数列表1) (返回值列表1)
    方法名2(参数列表2) (返回值列表2)
}
```
- 接口名：表示接口的名称
- 方法名：表示接口的方法名称
- 参数列表：表示方法的参数列表
- 返回值列表：表示方法的返回值列表
下面看一个具体的interface的例子
```go
package main

import "fmt"

type Phone interface {
    Call()
    SendMessage()
}

type Apple struct{
    PhoneName string
}

func (a Apple) Call() {
    fmt.Printf("%s有打电话功能\n", a.PhoneName)
}

func (a Apple) SendMessage() {
    fmt.Printf("%s有发短信功能\n", a.PhoneName)
}


type HuaWei struct{
    PhoneName string
}

func (h HuaWei) Call() {
    fmt.Printf("%s有打电话功能\n", h.PhoneName)
}

func (h HuaWei) SendMessage() {
    fmt.Printf("%s有发短信功能\n", h.PhoneName)
}


func main() {
    a := Apple{"apple"}
    b := HuaWei{"huawei"}
    a.Call()
    a.SendMessage()
    b.Call()
    b.SendMessage()
    
    var phone Phone      // 声明一个接口类型phone
    phone = new(Apple)   // 注意这种创建方式，new函数参数是接口的实现
    phone.(*Apple).PhoneName = "Apple"  // 这里使用断言给phone的成员赋值，后面会讲到接口的断言
    phone.Call()
    phone.SendMessage()
}
```
运行结果：
```
apple有打电话功能
apple有发短信功能
huawei有打电话功能
huawei有发短信功能
Apple有打电话功能
Apple有发短信功能
```
上述声明了一个`Phone`的接口，有两个方法`Call()`和`Sendmessage()`，然后定义了两个结构`Apple`和`HuaWei`，这两个结构都实现了`Phone`接口定义的所有方法，但是实现的方法不同，所以`Apple`和`HuaWei`是`Phone`接口的两种实现。
注意：上述例子的`phone`变量的定义，首先声明`phone`为一个接口，接着用`new`方法为这个`phone`定义，注意，这里`new`的参数必须是`phone`的一种实现，假设这里`Apple`结构没有实现`SendMessage`方法，那么`Apple`结构就不是`Phone`接口的一个实现，上述代码的45行用`new`定义`phone`的时候就会报错。

## 实现多个接口
有上述接口的定义我们知道，类型可以实现接口，那么一种类型可以实现多个接口吗，答案是：**可以的**  。
请看下面例子：
```go
 package main

import "fmt"

type MyWriter interface {
   MyWriter(s string)
}

type MyRead interface {
   MyReader()
}

type MyWriteReader struct {
}

func (r MyWriteReader) MyWriter(s string) {
   fmt.Printf("call MyWriteReader MyWriter %s\n", s)
}

func (r MyWriteReader) MyReader() {
   fmt.Printf("call MyWriteReader MyReader\n")
}

func main() {
   var myRead MyRead
   myRead = new(MyWriteReader)
   myRead.MyReader()

   var myWriter MyWriter
   myWriter = MyWriteReader{}
   myWriter.MyWriter("hello")
}
```
运行结果：
```
call MyWriteReader MyReader
call MyWriteReader MyWriter hello
``` 
上述例子我们定义了一个`MyWriter`接口，该接口有一个方法`MyWriter`，`MyWriter` 方法接受一个 string 类型的 参数。接着，我们定义了一个 `MyReader` 接口，该接口有一个方法 `MyReader`。然后定义了一个`ReadWriter`结构，实现了接口 `MyWriter` 和接口 `MyReader`中的所有方法，所以`ReadWriter`分别实现了两个不同的接口`MyWriter`和`MyReader`。

# 空接口
没有任何方法声明的接口称之为空接口`interface{}`。所有的类型都实现了空接口，因此空接口可以存储任意类型的数值。
Golang很多库的源代码都会以空接口作为参数，表示接受任意类型的参数，比如`fmt` 包下的 `Print` 系列函数。
```go
func Println(a ...interface{}) (n int, err error)
func Print(a ...interface{}) (n int, err error)
```
看下面例子：
```go
package main

import (
    "fmt"
)

func main() {
    var any interface{}
    any = 10
    fmt.Println(any)

    any = "golang"
    fmt.Println(any)

    any = true
    fmt.Println(any)
}
```
运行结果：
```
10
golang
true
``` 
上述例子首先声明了一个空接口`any`，首先用来存储整形变量10，然后又用来存储字符串·golang，最后用来存储布尔型变量true，所以空接口可以存储任意类型的数值。`fmt.Println`函数的参数是空接口类型`interface{}`，所以能将结果正确打印出来。

# 类型断言
在介绍断言之前，先看一个例子
```go
package main

func main() {
    var a int = 1
    var i interface{} = a
    var b int = i
}
```
运行结果：  
```
./prog.go:6:14: cannot use i (variable of type interface{}) as type int in variable declaration:
        need type assertion
```
上述代码中，声明了一个整形变量`a`，然后声明了一个空接口`i`，并赋值为`a`，接着声明了一个整形变量`b`，并赋值为`i`，但是编译器会报错，不能将`interface{}`类型的变量`i`赋值给整型变量`b`。
所以在写代码的时候我们需要注意，可以将任意类型的变量赋值给空接口`interface{}`类型，但是反过来不行。
那为了让这个操作能够完成，我们需要怎么做呢？就是**断言**。类型断言（Type Assertion）接口操作，用来检查接口变量的值是否实现了某个接口或者是否是某个具体的类型
断言的一般格式为：  
```go   
value, ok := x.(T)
```
- T 是具体某个类型，类型断言会检查 x 的动态类型是否等于具体类型 T。如果检查成功，类型断言返回的结果是 x 的值，其类型是 T。
- x 是接口类型，类型断言会检查 x 的动态类型是否满足 T。如果检查成功，返回值是一个类型为 T 的接口值。
- 无论 T 是什么类型，如果 x 是 nil 接口值，类型断言都会失败。
请看下面例子：
```go
package main

import (
    "fmt"
)

func main() {
    var x interface{}
    x = 8
    val, ok := x.(int)
    fmt.Printf("val is %d, ok is %t\n", val, ok)
}
```
运行结果：
```
val is 8, ok is true
```
上述代码中，首先声明了一个空接口`x`，并赋值为8，然后使用类型断言将`x`断言为`int`类型，并返回断言后的值和断言是否成功。**注意：如果在断言的过程中，没有bool这个值的判断，如果断言成功程序正常运行，假设interface{}存储的值跟要断言的类型不一致，则程序会报panic**
看下面例子：
```go
package main

import (
    "fmt"
)

func main() {
    var x interface{}
    x = "golang"
    val := x.(int)
    fmt.Println(val)
}           
```
运行结果：
```
panic: interface conversion: interface {} is string, not int
```

## 接口作函数参数
接口做函数参数，在函数定义的时候，形参为接口类型，在函数调用的时候，实参为该接口的具体实现
具体看下面看例子：
```go
package main

import "fmt"

type Reader interface {
   Read() int
}

type MyReader struct {
   a, b int
}

func (m *MyReader) Read() int {
   return m.a + m.b
}

func DoJob(r Reader) {
   fmt.Printf("myReader is %d\n", r.Read())
}

func main() {
   myReader := &MyReader{2, 5}
   DoJob(myReader)
}
```
运行结果：
```
myReader is 7
```
上述代码中，首先定义了一个`Reader`接口，然后定义了一个`MyReader`结构，并实现了`Reader`接口的`Read`方法，接着定义了一个`DoJob`函数，该函数的参数为`Reader`接口类型，在函数调用的时候，实参为`MyReader`结构，所以`DoJob`函数可以调用`MyReader`结构实现的`Read`方法。

若函数的形参为空接口，则实参可以为任意类型，因为空接口没有定义任何方法，任意类型都是空接口的一种实现。
```go
package main

import "fmt"


func DoJob(value interface{}) {
   fmt.Printf("value is %v\n", value)
}

func main() {
   val := 10
   DoJob(val)
}
```
运行结果：
```
value is 10
```

## 接口嵌套
接口嵌套就是一个接口中包含了其他接口，如果要实现外部接口，则需要实现内部嵌套的接口对应的所有方法。
看具体例子：
```go
package main

import "fmt"

type A interface {
    run1()
}

type B interface {
    run2()
}

// 定义嵌套接口C
type C interface {
    A
    B
    run3()
}

type Runner struct {}
    
// 实现嵌套接口A的方法
func (r Runner ) run1() {
    fmt.Println("run1!!!!")
}

// 实现嵌套接口B的方法
func (r Runner ) run2() {
    fmt.Println("run2!!!!")
}

func (r Runner ) run3() {
    fmt.Println("run3!!!!")
}

func main() {
   var runner C
   runner = new(Runner)  // runner实现了C接口的所有方法
   runner.run1()
   runner.run2()
   runner.run3()
}
```
运行结果：  
```
run1!!!!
run2!!!!
run3!!!!
```








## 接口的嵌套

