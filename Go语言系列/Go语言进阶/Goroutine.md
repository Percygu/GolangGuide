---
tags:
  - Go
  - golang
  - go进阶语法
  - Goroutine
---

# Goroutine
`goroutine`是Go语言对于协程的支持，可以把它理解为Go语言的协程。这是一个Go语言并发编程的终极杀器，它让我们的并发编程变得简单。

Go语言的并发只会用到`goroutine`，并不需要我们去考虑用多进程或者是多线程。有过C++或者Java经验的同学可能知道，线程本身是有一定大小的，一般OS线程栈大小为2MB，且线程在创建和上下文切换的时候是需要消耗资源的，会带来性能损耗，所以在我们用到多线程技术的时候，我们往往会通过池化技术，即创建线程池来管理一定数量的线程。
在Go语言中，一个`goroutine`栈在其生命周期开始时占用空间很小（一般2KB），并且栈大小可以按需增大和缩小，`goroutine`的栈大小限制可以达到1GB，但是一般不会用到这么大。所以在Go语言中一次创建成千上万，甚至十万左右的`goroutine`理论上也是可以的。
在Go语言中，我们用多`goroutine`来完成并发，在某个任务需要并发执行的时候，只需要把这个任务包装成一个函数，开启一个`goroutine`去执行这个函数就可以了。并不需要我们来维护一个类似于线程池的东西，也不需要我们去关心协程是怎么切换和调度的，因为这些都已经有Go语言内置的调度器帮我们做了，并且效率还非常高。

## Goroutine使用
`goroutine`使用起来非常方便，通常我们会将需要并发的任务封装成一个函数，然后再该函数前加上`go`关键字就行了，这样就开启了一个`goroutine`。
```go
func()
go func()  // 会并发执行这个函数
```

## 主协程
和其它语言一样，Go程序的入口也是`main`函数。在程序开始执行的时候，Go程序会为`main`函数创建一个默认的`goroutine`，我们称之为主协程，我们后来人为的创建的一些`goroutine`，都是在这个主`goroutine`的基础上进行的。
下面请看个例子：
```go
package main

import "fmt"

func myGroutine() {
        fmt.Println("myGroutine")
}

func main() {
        go myGroutine()
        fmt.Println("end!!!")
}
```
运行结果：
```
end!!!
myGroutine
```
很奇怪，明明是多协程任务，为什么只打印了主协程里的"end！！！"，而没有打印我们开启的协程里的输出"myGroutine"，按理不是应该都打印出来吗？
这是因为：当`main`函数返回的时候该`goroutine`就结束了，当主协程退出的时候，其他剩余的`goroutine`不管是否运行完，都会跟着结束。所以，这里主协程打印完"end！！！"之后就退出了，`myGroutine`协程可能还没运行到`fmt.Println("myGroutine")`语句也跟着退出了。
接下来我们让主`goroutine`执行完`fmt.Println("end!!!")`之后不立刻退出，而是等待2s，看一下运行结果：
```go
package main

import (
 "fmt"
 "time"
 )

func myGroutine() {
        fmt.Println("myGroutine")
}

func main() {
        go myGroutine()
        fmt.Println("end!!!")
        time.Sleep(2*time.Second)
}
```
运行结果：
```
end!!!
myGroutine
```
此时打印出了我们想要的结果，这里我们通过让主协程睡眠2s来等待子协程执行完了之后再退出，后面我们会学习到更好的方法，这里就不再过多阐述。   

## 多协程调用
在Go语言中，我们可以通过`go`关键字来开启多个协程，每个协程可以并发执行，互不干扰。
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func myGoroutine(name string, wg *sync.WaitGroup) {
    defer wg.Done()

    for i := 0; i < 5; i++ {
       fmt.Printf("myGroutine %s\n", name)
       time.Sleep(10 * time.Millisecond)
    }
}

func main() {
    var wg sync.WaitGroup
    wg.Add(2)

    go myGoroutine("goroutine1", &wg)
    go myGoroutine("goroutine2", &wg)

    wg.Wait()
}
```
运行结果：
```
myGroutine goroutine1
myGroutine goroutine2
```
从结果中可以看到，两个协程并发执行，互不干扰。注意在上述例子中，我们使用了`sync.WaitGroup`来等待所有协程执行完毕之后再退出。关于`sync.WaitGroup`的详细介绍，可以参考[sync.WaitGroup](https://pkg.go.dev/sync#WaitGroup)。后续也会在`sync`章节详细介绍。




