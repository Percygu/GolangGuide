---
tags:
  - Go
  - golang
  - 代码面试题
---

# 代码面试题

## 1.开启100个协程，顺序打印1-1000，且保证协程号1的，打印尾数为1的数字

```go
//  同时开启100个协程(分别为1号协程 2号协程 ... 100号协程，
//  1号协程只打印尾数为1的数字，2号协程只打印尾数为2的数，
//   以此类推)，请顺序打印1-1000整数以及对应的协程号；

func main() {
    s := make(chan struct{})
    //通过map的key来保证协程的顺序
    m := make(map[int]chan int, 100)
    //填充map,初始化channel
    for i := 1; i <= 100; i++ {
        m[i] = make(chan int)
    }
    //开启100个协程，死循环打印
    //go func() {  这个协程不加也可以的
        for i := 1; i <= 100; i++ {
            go func(id int) {
                for {
                    num := <-m[id]
                    fmt.Println(num)
                    s <- struct{}{}
                }
            }(i)
        }
    //}()
    //循环1-1000，并把值传递给匹配的map
    //然后通过s限制循序打印
    for i := 1; i <= 1000; i++ {
        id := i % 100
        if id == 0 {
            id = 100
        }
        m[id] <- i
        //通过s这个来控制打印顺序。每次遍历一次i
        //都通过s阻塞协程的打印，最后打印完毕
        <-s
    }

    time.Sleep(10 * time.Second)
}
```

## 2.三个goroutinue交替打印abc 10次

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    // 定义3个channel
    ch1 := make(chan struct{})
    ch2 := make(chan struct{})
    ch3 := make(chan struct{})
    var wg sync.WaitGroup
    wg.Add(3)
    // 打印a
    go func() {
       defer wg.Done()
       for i := 0; i < 10; i++ {
          <-ch1
          fmt.Println("a")
          ch2 <- struct{}{}
       }
       // 第10次的时候，打印c的goroutine写入了ch1
       // 为了防止阻塞，要消费以下ch1
       <-ch1
    }()
    // 打印b
    go func() {
       defer wg.Done()
       for i := 0; i < 10; i++ {
          <-ch2
          fmt.Println("b")
          ch3 <- struct{}{}
       }
    }()
    // 打印c
    go func() {
       defer wg.Done()
       for i := 0; i < 10; i++ {
          <-ch3
          fmt.Println("c")
          ch1 <- struct{}{}
       }
    }()
    // 启动
    ch1 <- struct{}{}
    wg.Wait()
    close(ch1)
    close(ch2)
    close(ch3)
    fmt.Println("end")
}
```

## 3.用不超过10个goroutine不重复的打印slice中的100个元素

```go
package main

import (
    "fmt"
    "sync"
)

// 用不超过10个goroutine不重复的打印slice中的100个元素
// 容量为10的有缓冲channel实现
// 每次启动10个，累计启动100个goroutine,且无序打印
func main() {
    var wg sync.WaitGroup
    // 创建切片
    ss := make([]int, 100)
    for i := 0; i < 100; i++ {
       ss[i] = i
    }
    ch := make(chan struct{}, 10)
    for i := 0; i < 100; i++ {
       wg.Add(1)
       ch <- struct{}{}
       // 写10个就阻塞了，此时goroutine中打印
       go func(idx int) {
          defer wg.Done()
          fmt.Printf("val: %d \n", ss[idx])
          // 打印结束，从缓冲channel中删除一个
          <-ch
       }(i)

    }
    wg.Wait()
    // 关闭channel
    close(ch)
    fmt.Println("end")
}

// 用不超过10个goroutine不重复的打印slice中的100个元素
// 创建10个无缓冲channel和10个goroutine
// 固定10个goroutine,且顺序打印
func test9() {
    var wg sync.WaitGroup
    // 创建切片
    ss := make([]int, 100)
    for i := 0; i < 100; i++ {
       ss[i] = i
    }
    // 创建channel和goroutine
    hashMap := make(map[int]chan int)
    sort := make(chan struct{})
    for i := 0; i < 10; i++ {
       hashMap[i] = make(chan int)
       wg.Add(1)
       go func(idx int) {
          defer wg.Done()
          for val := range hashMap[idx] {
             fmt.Printf("go id: %d, val: %d \n", idx, val)
             sort <- struct{}{}
          }
       }(i)
    }
    // 循环切片，对10取模，找到对应channel的key，写入值
    for _, v := range ss {
       id := v % 10
       hashMap[id] <- v
       // 有序
       <-sort
    }
    // 循环结束关闭channel,删除map的key
    for k, _ := range hashMap {
       close(hashMap[k])
       delete(hashMap, k)
    }
    wg.Wait()
    close(sort)
    fmt.Println("end")
}
```

## 4.两个协程交替打印奇偶数

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    //golang交替打印奇偶数
    //交替打印，可以通过channel来实现
    chan1 := make(chan struct{})
    //偶数
    go func() {
       for i := 0; i < 10; i++ {
          chan1 <- struct{}{}
          if i%2 == 0 {
             fmt.Println("打印偶数:", i)
          }
       }
    }()
    //奇数
    go func() {
       for i := 0; i < 10; i++ {
          <-chan1
          if i%2 == 1 {
             fmt.Println("打印奇数数:", i)
          }
       }
    }()
    //阻塞
    select {
    case <-time.After(time.Second * 10):
    }
}
```

## 5.用单个channel实现0,1的交替打印

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    msg := make(chan struct{})
    go func() {
       for {
          <-msg
          fmt.Println("0")
          msg <- struct{}{}
       }
    }()
    go func() {
       for {
          <-msg
          fmt.Println("1")
          msg <- struct{}{}
       }
    }()
    msg <- struct{}{}
    time.Sleep(3 * time.Minute)

}
```

## 6.sync.Cond实现多生产者多消费者

```go
package main

import (
    "context"
    "fmt"
    "math/rand"
    "sync"
    "time"
)

func main() {
    var wg sync.WaitGroup
    var cond sync.Cond
    cond.L = new(sync.Mutex)
    msgCh := make(chan int, 5)
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    rand.Seed(time.Now().UnixNano())

    // 生产者
    producer := func(ctx context.Context, out chan<- int, idx int) {
       defer wg.Done()
       for {
          select {
          case <-ctx.Done():
             // 每次生产者退出，都唤醒一个消费者处理，防止最后有消费者线程死锁
             // 生产者比消费者多，所以cond.Signal()就可以。不然的话建议Broadcast()
             cond.Broadcast()
             fmt.Println("producer finished")
             return
          default:
             cond.L.Lock()
             for len(msgCh) == 5 {
                cond.Wait()
             }
             num := rand.Intn(500)
             out <- num
             fmt.Printf("producer: %d, msg: %d\n", idx, num)
             cond.Signal()
             cond.L.Unlock()
          }
       }
    }

    // 消费者
    consumer := func(ctx context.Context, in <-chan int, idx int) {
       defer wg.Done()
       for {
          select {
          case <-ctx.Done():
             // 消费者可以选择继续消费直到channel为空
             for len(msgCh) > 0 {
                select {
                case num := <-in:
                   fmt.Printf("consumer %d, msg: %d\n", idx, num)
                default:
                   // 如果channel已经空了，跳出循环
                   break
                }
             }
             fmt.Println("consumer finished")
             return
          default:
             cond.L.Lock()
             for len(msgCh) == 0 {
                cond.Wait()
             }
             num := <-in
             fmt.Printf("consumer %d, msg: %d\n", idx, num)
             cond.Signal()
             cond.L.Unlock()
          }
       }
    }

    // 启动生产者和消费者
    for i := 0; i < 5; i++ {
       wg.Add(1)
       go producer(ctx, msgCh, i+1)
    }
    for i := 0; i < 3; i++ {
       wg.Add(1)
       go consumer(ctx, msgCh, i+1)
    }

    // 模拟程序运行一段时间
    wg.Wait()
    close(msgCh)
    fmt.Println("all finished")
}
```

## 7.使用go实现1000个并发控制并设置执行超时时间1秒

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

func main() {
    // 创建 1000 个协程，并且进行打印
    // 总共超时时间 1s，1s 没执行完就超时，使用 ctx 进行控制

    // 定义任务 channel
    tasks := make(chan int, 1000)
    // 定义 ctx
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()
    var wg sync.WaitGroup

    // 启动 1000 个协程
    for i := 0; i < 1000; i++ {
       wg.Add(1)
       tasks <- i
       go func(id int) {
          defer wg.Done()
          select {
          case <-ctx.Done():
             return
          default:
             fmt.Printf("goroutine id: %d\n", id)
          }
       }(i)
    }

    <-ctx.Done()
    fmt.Println("exec done")
    close(tasks)
    wg.Wait()
    fmt.Println("finish")
}
```

## 8.使用两个Goroutine，向标准输出中按顺序按顺序交替打出字母与数字，输出是a1b2c3

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    // 定义两个channel，一个打印数字，一个打印字母
    numCh := make(chan struct{})
    strCh := make(chan struct{})
    var wg sync.WaitGroup
    wg.Add(2)
    // 打印字符
    go func() {
       defer wg.Done()
       for i := 'a'; i <= 'z'; i++ {
          fmt.Println(string(i))
          // 通知打印数字
          numCh <- struct{}{}
          // 阻塞等待打印字母
          <-strCh
       }
    }()
    // 打印字母
    go func() {
       defer wg.Done()
       for i := 1; i <= 26; i++ {
          <-numCh
          fmt.Println(i)
          // 通知打印字母
          strCh <- struct{}{}
       }
    }()
    wg.Wait()
    fmt.Println("finished")
}
```

## 9.编写一个程序限制10个goroutine执行，每执行完一个goroutine就放一个新的goroutine进来

```go
package main

import (
    "fmt"
    "sync"
)

// 编写一个程序限制10个goroutine执行，每执行完一个goroutine就放一个新的goroutine进来
func main() {
    var wg sync.WaitGroup
    ch := make(chan struct{}, 10)
    for i := 0; i < 20; i++ {
       wg.Add(1)
       ch <- struct{}{}
       go func(id int) {
          defer wg.Done()
          fmt.Println("id: %d", id)
          <-ch
       }(i)
    }
    wg.Wait()

}
```

