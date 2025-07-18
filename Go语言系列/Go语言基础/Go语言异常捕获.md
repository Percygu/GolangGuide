---
tags:
  - Go
  - golang
  - go基础语法
  - 异常捕获
---

# Go语言异常捕获

## recover异常捕获
异常其实就是指程序运行过程中发生了`panic`，那么我们为了不让程序报错退出，可以在程序中加入`recover`机制，将异常捕获，打印出异常，这样也方便我们定位错误。而捕获的方式我们之前在讲`defer`的时候也提到过，一般是用`recover`和`defer`搭配使用来捕获异常。
下面请看个具体例子：

```go
func main() {         
    defer func() {                 
        if error:=recover();error!=nil{   
            fmt.Println("出现了panic,使用reover获取信息:",error)   
        }         
    }()         
    fmt.Println("11111111111")        
    panic("出现panic")         
    fmt.Println("22222222222") 
 }
```
运行结果：
```
11111111111
出现了panic,使用reover获取信息: 出现panic
```
**注意**，这里有了`recover`之后，程序不会在`panic`出中断，再执行完`panic`之后，会接下来执行`defer recover`函数，但是当前函数`panic`后面的代码不会被执行，但是调用该函数的代码会接着执行。
如果我们在`main`函数中未加入`defer func(){...}`，当我们的程序运行到底8行时就会`panic`掉，而通常在我们的业务程序中对于程序`panic`是不可容忍的，我们需要程序健壮的运行，而不是是不是因为一些`panic`挂掉又被拉起，所以当发生`panic`的时候我们要让程序能够继续运行，并且获取到发生`panic`的具体错误，这就可以用上述方法。

## panic传递
当一个函数发生了`panic`之后，若在当前函数中没有`recover`，会一直向外层传递直到主函数，如果迟迟没有`recover`的话，那么程序将终止。如果在过程中遇到了最近的`recover`，则将被捕获。
看下面例子：

```go
package main

import "fmt"

func testPanic1(){
   fmt.Println("testPanic1上半部分")
   testPanic2()
   fmt.Println("testPanic1下半部分")
}

func testPanic2(){
   defer func() {
      recover()
   }()
   fmt.Println("testPanic2上半部分")
   testPanic3()
   fmt.Println("testPanic2下半部分")
}

func testPanic3(){
   fmt.Println("testPanic3上半部分")
   panic("在testPanic3出现了panic")
   fmt.Println("testPanic3下半部分")
}

func main() {
   fmt.Println("程序开始")
   testPanic1()
   fmt.Println("程序结束")
}
```
运行结果：
```
程序开始
testPanic1上半部分
testPanic2上半部分
testPanic3上半部分
testPanic1下半部分
程序结束    
``` 
解析：
调用链：`main-->testPanic1-->testPanic2-->testPanic3`，但是在`testPanic3`中发现了一个`panic`，由于`testPanic3`没有`recover`，向上找，在`testPanic2`中找到了`recover`，`panic`被捕获了，程序接着运行，由于`testPanic3`发生了`panic`，所以不再继续运行，函数跳出返回到`testPanic2`，`testPanic2`中捕获到了`panic`，也不会再继续执行，跳出函数`testPanic2`，到了`testPanic1`接着运行。

所以`recover`和`panic`可以总结为以下两点：
1. `recover()`只能恢复当前函数级或以当前函数为首的调用链中的函数中的`panic()`，恢复后调用当前函数结束，但是调用此函数的函数继续执行
2. 函数发生了`panic`之后会一直向上传递，如果直至`main`函数都没有`recover()`，程序将终止，如果是碰见了`recover()`，将被`recover`捕获。
