---
tags:
  - Go
  - golang
  - go进阶
  - sync
  - 并发安全
---

# sync包
在前面学习`channel`的时候，我们说到在Go语言并发编程中，倡导使用通信共享内存，不要使用共享内存通信，`goroutine`之间尽量通过`channel`来协作。而在其他的传统语言中，都是通过共享内存加上锁机制来保证并发安全的，同样Go语言也提供了对共享内存并发安全机制的支持，这些功能都存在于`sync`包下。

## sync.WaitGroup
在前面很多`goroutine`的示例中，我们都是通过`time.Sleep()`方法让主`goroutine`等待一段时间以便子`gortoutine`能够执行完打印结果，显然这不是一个很好的办法，因为我们不知道所有的子`gortoutine`要多久才能执行完，不能确切的知道需要等待多久。那要怎么处理呢？

### 使用channel实现等待
看下面例子：
```go
package main

import (
   "fmt"
)

func main() {
   ch := make(chan struct{}, 10)
   for i := 0; i < 10; i++ {
      go func(i int) {
         fmt.Printf("num:%d\n",i)
         ch <- struct{}{}
      }(i)
   }

   for i := 0; i < 10; i++ {
      <-ch
   }

   fmt.Println("end")
}
```
运行结果：
```
num:0
num:2
num:1
num:4
num:6
num:7
num:5
num:8
num:9
num:3
end
```
我们在每个`goroutine`中，向管道里发送一条数据，这样我们在程序最后，通过`for`循环将管道里的数据全部取出，直到数据全部取出完毕才能继续后面的逻辑，这样就可以实现等待各个`goroutine`执行完。
但是，这样使用`channel`显得并不优雅，其次，我们得知道具体循环的次数，来创建管道的大小，假设次数非常的多，则需要申请同样数量大小的管道出来，对内存也是不小的开销。

### 使用WaitGroup实现等待
这里我们可以用`sync`包下的`WaitGroup`来实现，Go语言中可以使用`sync.WaitGroup`来实现并发任务的同步以及协程任务等待。
`sync.WaitGroup`是一个对象，里面维护者一个计数器，并且通过三个方法来配合使用：
- (wg * WaitGroup) Add(delta int)  计数器加delta
- (wg *WaitGroup) Done()           计数器减1
- (wg *WaitGroup) Wait()           会阻塞代码的运行，直至计数器减为0
先看示例：
```go
package main

import (
   "fmt"
   "sync"
)

var wg sync.WaitGroup

func myGoroutine() {
   defer wg.Done()
   fmt.Println("myGoroutine!")
}

func main() {
   wg.Add(10)
   for i := 0; i < 10; i++ {
      go myGoroutine()
   }
   wg.Wait()
   fmt.Println("end!!!")
}
```

运行结果：
```
myGoroutine!
myGoroutine!
myGoroutine!
myGoroutine!
myGoroutine!
myGoroutine!
myGoroutine!
myGoroutine!
myGoroutine!
myGoroutine!
end!!!
```
程序首先把`wg`的计数设置为10，每个`for`循环运行完毕都把计数器减1，`main`函数中执行到`wg.Wait()`会一直阻塞，直到`wg`的计数器为零。最后打印了10个`myGoroutine!`，是所有子`goroutine`任务结束后主`goroutine`才退出。
**注意：`sync.WaitGroup`对象的计数器不能为负数，否则会panic，在使用的过程中，我们需要保证`add()`的参数值，以及执行完`Done()`之后计数器大于等于零**

## sync.Once
在我们写项目的时候，程序中有很多的逻辑只需要执行一次，最典型的就是项目工程里配置文件的加载，我们只需要加载一次即可，让配置保存在内存中，下次使用的时候直接使用内存中的配置数据即可。这里就要用到`sync.Once`。
`sync.Once`可以在代码的任意位置初始化和调用，并且线程安全。`sync.Once`最大的作用就是延迟初始化，对于一个`sync.Once`变量我们并不会在程序启动的时候初始化，而是在第一次用的它的时候才会初始化，并且只初始化这一次，初始化之后驻留在内存里，这就非常适合我们之前提到的配置文件加载场景，设想一下，如果是在程序刚开始就加载配置，若迟迟未被使用，则既浪费了内存，又延长了程序加载时间，而`sync.Once`就刚好解决了这个问题。
使用示例：
```go
// 声明配置结构体Config
type Config struct{}

var instance *Config
var once sync.Once     // 声明一个sync.Once变量

// 获取配置结构体
func InitConfig() *Config {
   once.Do(func(){
      instance = &Config{}
   })
   return instance
}
```
只有在第一次调用InitConfig()获取Config 指针的时候才会执行once.Do(func(){instance = &Config{} })语句，执行完之后instance就驻留在内存中，后面再次执行InitConfig()的时候，就直接返回内存中的instance。

### sync.Once与init()的区别
有时候我们使用init()方法进行初始化，init()方法是在其所在的package首次加载时执行的，而sync.Once可以在代码的任意位置初始化和调用，是在第一次用的它的时候才会初始化。

## sync.Lock
说到并发编程，就不得不谈一个老生常谈的问题，那就是资源竞争，也就是我们这节要讲的并发安全。因为一旦开启了多个`goroutine`去处理问题，那么这些`goroutine`就有可能在同一时间操作同一个系统资源，比如同一个变量，同一份文件等等，这里我们如果不加控制的话，可能会出现并发安全问题，在Go语言中，有两种方式来控制并发安全，锁和原子操作
举个例子，看下面代码
```go
package main

import (
    "fmt"
    "sync"
)

var (
    num int
    wg  = sync.WaitGroup{}
)

func add() {
    defer wg.Done()
    num += 1
}

func main() {
    var n = 10 * 10 * 10 * 10
    wg.Add(n)

    for i := 0; i < n; i++ {
       // 启动n个goroutine去累加num
       go add()
    }

    // 等待所有goroutine执行完毕
    wg.Wait()

    // 不出意外的话，num应该等于n，但是，但是，但是实际上不一致！
    fmt.Println(num == n)
}
```
运行结果：
```
false
```
我们用`n`（这里是10000，可以自行修改，尽量数字大一点）个`goroutine`去给`num`做累加，最后并`num`并不等于`n`，这就是并发问题，同一时间有多个`goroutine`都在对`num`做`+1`操作，但是后一个并不是在前一次执行完的基础之上运行的，可能两次运行`num`的初始相同，这样前一个`num+1`的结果就被后一个覆盖了，看起来好像只做了一个加法。为了避免类似的并发安全问题，我们一般会采用下面两种方式处理，在go语言中并发相关的都在`sync`包下面。

### 锁
#### 互斥锁Mutex
互斥锁是一种最常用的控制并发安全的方式，它在同一时间只允许一个goroutine对共享资源进行访问。
互斥锁的声明方式如下：
```go
var lock sync.Mutex
```
互斥锁有两个方法
```go
func (m *Mutex) Lock()     // 加锁
func (m *Mutex) Unlock()   // 解锁
```
一个互斥锁只能同时被一个`goroutine`锁定，其它`goroutine`将阻塞直到互斥锁被解锁才能加锁成功。`sync.Mutex`在使用的时候要注意：**对一个未锁定的互斥锁解锁将会产生运行时错误**。
对上面的例子稍作修改，加上互斥锁：
```go
package main

import (
    "fmt"
    "sync"
)

var (
    num int
    wg  = sync.WaitGroup{}
    // 我们用锁来保证num的并发安全
    mu = sync.Mutex{}
)

func add() {
    mu.Lock()
    defer wg.Done()
    num += 1
    mu.Unlock()
}

func main() {
    var n = 10 * 10 * 10 * 10
    wg.Add(n)

    for i := 0; i < n; i++ {
       // 启动n个goroutine去累加num
       go add()
    }

    // 等待所有goroutine执行完毕
    wg.Wait()

    fmt.Println(num == n)
}
```
运行结果：
```
true
```
我们可以自行修改n的值，在我们能开启足够多的`goroutine`的情况下，他结果一定会是`true`。
本例使用了前面介绍的`sync.WaitGroup`来等待所有协程执行结束。并且在`add`函数里使用了互斥锁来保证`num += 1`操作的并发安全，但是注意不要忘了用`mu.Unlock`来进行解锁，否则其他`goroutine`将一直等待加锁造成阻塞。


#### 读写锁RWMutex
读写锁就是将读操作和写操作分开，可以分别对读和写进行加锁，一般用在大量读操作、少量写操作的情况。
读写锁的声明方式如下：
```go
var rw sync.RWMutex
```
读写锁有两个方法：
```go
func (rw *RWMutex) Lock()     // 对写锁加锁
func (rw *RWMutex) Unlock()   // 对写锁解锁

func (rw *RWMutex) RLock()    // 对读锁加锁
func (rw *RWMutex) RUnlock()  // 对读锁解锁
```
读写锁的使用遵循以下几个法则：
1. 同时只能有一个 goroutine 能够获得写锁定。
2. 同时可以有任意多个 gorouinte 获得读锁定。
3. 同时只能存在写锁定或读锁定（读和写互斥）。
通俗理解就是可以多个`goroutine`同时读，但是只有一个`goroutine`能写，共享资源要么在被一个或多个`goroutine`读取，要么在被一个`goroutine`写入， 读写不能同时进行。 
读写锁示例：
```go
package main

import (
   "fmt"
   "sync"
   "time"
)

var cnt = 0

func main() {
   var mr sync.RWMutex
   for i := 1; i <= 3; i++ {
      go write(&mr, i)
   }
   for i := 1; i <= 3; i++ {
      go read(&mr, i)
   }

   time.Sleep(time.Second)
   fmt.Println("final count:", cnt)
}

func read(mr *sync.RWMutex, i int) {
   fmt.Printf("goroutine%d reader start\n", i)
   mr.RLock()
   fmt.Printf("goroutine%d reading count:%d\n", i, cnt)
   time.Sleep(time.Millisecond)
   mr.RUnlock()

   fmt.Printf("goroutine%d reader over\n", i)
}

func write(mr *sync.RWMutex, i int) {
   fmt.Printf("goroutine%d writer start\n", i)
   mr.Lock()
   cnt++
   fmt.Printf("goroutine%d writing count:%d\n", i, cnt)
   time.Sleep(time.Millisecond)
   mr.Unlock()

   fmt.Printf("goroutine%d writer over\n", i)
}
```
运行结果：
```
goroutine3 reader start
goroutine3 reading count:0
goroutine1 writer start
goroutine2 writer start
goroutine1 reader start
goroutine2 reader start
goroutine3 writer start
goroutine3 reader over
goroutine1 writing count:1
goroutine1 writer over
goroutine1 reading count:1
goroutine2 reading count:1
goroutine2 reader over
goroutine2 writing count:2
goroutine1 reader over
goroutine2 writer over    
goroutine3 writing count:3
goroutine3 writer over
final count: 3
```
简单分析：首先`goroutine3`开始加了读锁，开始读取，读到count的值为0，然后`goroutine1`尝试写入，`goroutine2`尝试写入，但是都会阻塞，因为`goroutine3`加了读锁，不能再加写锁，在第8行`goroutine3` 读取完毕之后，`goroutine1`争抢到了锁，加了写锁，写完释放写锁之后，`goroutine1`和`goroutine2`同时加了读锁，读到count的值为1。可以看到读写锁是互斥的，写写锁是互斥的，读读锁可以一起加。


#### 死锁
提到锁，就有一个绕不开的话题：死锁。死锁就是一种状态，当两个或以上的`goroutine`在执行过程中，因争夺共享资源处在互相等待的状态，如果没有外部干涉将会一直处于这种阻塞状态，我们称这时的系统发生了死锁。思索场景一般有以下两种
1. **Lock/Unlock不成对**。这类情况最常见的场景就是对锁进行拷贝使用
```go
package main    

import (
    "fmt"
    "sync"
)

func main() {
    var mu sync.Mutex
    mu.Lock()
    defer mu.Unlock()
    copyMutex(mu)
}

func copyMutex(mu sync.Mutex) {
    mu.Lock()
    defer mu.Unlock()
    fmt.Println("ok")
}
```
运行结果：
```
fatal error: all goroutines are asleep - deadlock!      
                                                        
goroutine 1 [semacquire]:                               
sync.runtime_SemacquireMutex(0xc0000160ac, 0x0, 0x1)    
        D:/Program Files/Go/src/runtime/sema.go:71 +0x4e
sync.(*Mutex).lockSlow(0xc0000160a8)                    
        D:/Program Files/Go/src/sync/mutex.go:138 +0x10f
sync.(*Mutex).Lock(...)                                 
        D:/Program Files/Go/src/sync/mutex.go:81        
main.copyTest(0xc0000160a8)  
```
会报死锁，为什么呢？有的同学可能会注意到，这里`mu sync.Mutex`当作参数传入到函数`copyMutex`，锁进行了拷贝，不是原来的锁变量了，那么一把新的锁，在执行`mu.Lock()`的时候应该没问题。这就是要注意的地方，如果将带有锁结构的变量赋值给其他变量，锁的状态会复制。所以多锁复制后的新的锁拥有了原来的锁状态，那么在`copyMutex`函数内执行`mu.Lock()`的时候会一直阻塞，因为外层的`main`函数已经`Lock()`了一次，但是并没有机会`Unlock()`，导致内层函数会一直等待`Lock()`，而外层函数一直等待`Unlock()`，这样就造成了死锁
所以在使用锁的时候，我们应当尽量避免锁拷贝，并且保证Lock()和Unlock()成对出现，没有成对出现容易会出现死锁的情况，或者是Unlock 一个未加锁的Mutex而导致 panic。尽量养成如下使用习惯
```go
mu.Lock()
defer mu.Unlock()
```

2. **循环等待**
另一个容易造成死锁的场景就是循环等待，A等B，B等C，C等A，循环等待
```go
package main

import (
   "sync"
   "time"
)

func main() {
   var mu1, mu2 sync.Mutex
   var wg sync.WaitGroup

   wg.Add(2)
   go func() {
      defer wg.Done()
      mu1.Lock()
      defer mu1.Unlock()
      time.Sleep(1 * time.Second)

      mu2.Lock()
      defer mu2.Unlock()
   }()

   go func() {
      defer wg.Done()
      mu2.Lock()
      defer mu2.Unlock()
      time.Sleep(1 * time.Second)
      mu1.Lock()
      defer mu1.Unlock()
   }()
   wg.Wait()
}
```
运行结果：
```
fatal error: all goroutines are asleep - deadlock! 
```
死锁了，代码很简单，两个`goroutine`，一个`goroutine`先锁`mu1`，再锁`mu2`，另一个`goroutine`先锁`mu2`，再锁`mu1`，但是在它们进行第二次枷锁操作的时候，彼此等待对方释放锁，这样就造成了循环等待，一直阻塞，形成死锁。

## sync.Map
Go语言内置的Map并不是并发安全的，在多个`goroutine`同时操作map的时候，会有并发问
具体看下面例子
```go
package main

import (
   "fmt"
   "strconv"
   "sync"
)

var m = make(map[string]int)

func getVal(key string) int {
   return m[key]
}

func setVal(key string, value int) {
   m[key] = value
}

func main() {
   wg := sync.WaitGroup{}
   wg.Add(10)
   for i := 0; i < 10; i++ {
      go func(num int) {
         defer wg.Done()
         key := strconv.Itoa(num)
         setVal(key, num)
         fmt.Printf("key=:%v,val:=%v\n", key, getVal(key))
      }(i)
   }
   wg.Wait()
}
```
运行结果：
```
fatal error: concurrent map writes
```
程序报错了，说明`map`不能同时被多个`goroutine`读写。要解决`map`的并发写问题一种方式使用我们前面学到的对`map`加锁，这样就可以了
```go
package main

import (
   "fmt"
   "strconv"
   "sync"
)

var m = make(map[string]int)
var mu sync.Mutex

func getVal(key string) int {
   return m[key]
}

func setVal(key string, value int) {
   m[key] = value
}

func main() {
   wg := sync.WaitGroup{}

   wg.Add(10)
   for i := 0; i < 10; i++ {
      go func(num int) {
         defer func() {
            wg.Done()
            mu.Unlock()
         }()
         key := strconv.Itoa(num)
         mu.Lock()
         setVal(key, num)
         fmt.Printf("key=:%v,val:=%v\n", key, getVal(key))
      }(i)
   }
   wg.Wait()
}
```
运行结果：
```
key=:9,val:=9
key=:4,val:=4
key=:0,val:=0
key=:1,val:=1
key=:2,val:=2
key=:3,val:=3
key=:6,val:=6
key=:7,val:=7
key=:5,val:=5
key=:8,val:=8
```
另外一种方式是使用`sync`包中提供的一个开箱即用的并发安全版`map`–`sync.Map`,在 Go 1.9 引入。`sync.Map`不用初始化就可以使用，同时`sync.Map`内置了诸如`Store`、`Load`、`LoadOrStore`、`Delete`、`Range`等操作方法。
具体使用方法看示例：
```go
package main

import (
   "fmt"
   "sync"
)

func main() {
   var m sync.Map
   // 1. 写入
   m.Store("name", "zhangsan")
   m.Store("age", 18)

   // 2. 读取
   age, _ := m.Load("age")
   fmt.Println(age.(int))

   // 3. 遍历
   m.Range(func(key, value interface{}) bool {
      fmt.Printf("key is:%v, val is:%v\n", key, value)
      return true
   })

   // 4. 删除
   m.Delete("age")
   age, ok := m.Load("age")
   fmt.Println(age, ok)

   // 5. 读取或写入
   m.LoadOrStore("name", "zhangsan")
   name, _ := m.Load("name")
   fmt.Println(name)
}
```
运行结果：
```
18
key is:name, val is:zhangsan
key is:age, val is:18       
<nil> false                 
zhangsan  
```
1. 通过store方法写入两个键值对
2. 读取key为age的值，读出来age为18
3. 通过range方法遍历map的key和value
4. 删除key为age的键值对，删除完之后，再次读取age，age为空，ok为false表示map里没有这个key
5. LoadOrStore尝试读取key为name的值，读取不到就写入键值对name-zhangsan，能读取到就返回原来map里的name对应的值
**注意：`sync.Map` 没有提供获取 map 数量的方法，需要我们在对 `sync.Map`进行遍历时自行计算，`sync.Map` 为了保证并发安全有一些性能损失，因此在非并发情况下，使用 `map` 相比使用 `sync.Map` 会有更好的性能**

## sync/Atomic
除了前面介绍的锁`mutex`以外，还有一种解决并发安全的策略，就是原子操作。所谓原子操作就是这一系列的操作在`cpu`上执行是一个不可分割的整体，显然要么全部执行，要么全部不执行，不会受到其他操作的影响，也就不会存在并发问题。

### atomic和mutex的区别
1. 使用方式：通常`mutex`用于保护一段执行逻辑，而`atomic`主要是对变量进行操作
2. 底层实现：`mutex`由操作系统调度器实现，而`atomic`操作有底层硬件指令支持，保证在`cpu`上执行不中断。所以`atomic`的性能也能随`cpu`的个数增加线性提升

#### `atomic`提供的方法：
```go
func AddT(addr *T, delta T)(new T)
func StoreT(addr *T, val T)
func LoadT(addr *T) (val T)
func SwapT(addr *T, new T) (old T)
func CompareAndSwapT(addr *T, old, new T) (swapped bool)
T的类型是int32、int64、uint32、uint64和uintptr中的任意一种
```
这里就不一一演示各个方法了，以`AddT`方法为例简单看一个例子
```go
package main

import (
   "fmt"
   "sync"
   "sync/atomic"
)

func main() {

   var sum int32 =  0
   var wg sync.WaitGroup
   for i := 0; i < 100; i++ {
      wg.Add(1)
      go func() {
         defer wg.Done()
         atomic.AddInt32(&sum, 1)
      }()
   }
   wg.Wait()
   fmt.Printf("sum is %d\n",sum)
}
```
100个goroutine，每个goroutine都对sum+1，最后结果为100。

### atomic.value
上面展示的`AddT`，`StoreT`等方法都是针对的基本数据类型做的操作，假设想对多个变量进行同步保护，即假设想对一个`struct`这样的复合类型用原子操作，也是支持的吗？也可以做支持，go语言里的`atomic.value`支持任意一种接口类型进行原子操作，且提供了`Load`、`Store`、`Swap`和`CompareAndSwap`四种方法：
- `Load`：func (v *Value) Load() (val any)，从value读出数据
- `Store`：func (v *Value) Store(val any)，向value写入数据
- `Swap`：func (v *Value) Swap(new any) (old any)，用new交换value中存储的数据，返回value原来存储的旧数据
- `CompareAndSwap`：func (v *Value) CompareAndSwap(old, new any) (swapped bool)，比较value中存储的数据和old是否相同，相同的话，将value中的数据替换为new
代码示例
```go
package main

import (
    "fmt"
    "sync/atomic"
)

type Student struct {
    Name string
    Age  int
}

func main() {
    st1 := Student{
       Name: "zhangsan",
       Age:  18,
    }
    st2 := Student{
       Name: "lisi",
       Age:  19,
    }
    st3 := Student{
       Name: "wangwu",
       Age:  20,
    }
    var v atomic.Value
    v.Store(st1)
    fmt.Println(v.Load().(Student))

    old := v.Swap(st2)
    fmt.Printf("after swap: v=%v\n", v.Load().(Student))
    fmt.Printf("after swap: old=%v\n", old)

    swapped := v.CompareAndSwap(st1, st3)   // v中存储的和st1不相同，交换失败
    fmt.Println("compare st1 and v\n", swapped, v)

    swapped = v.CompareAndSwap(st2, st3)   // v中存储的和st2相同，交换成功，v中变为st3
    fmt.Println("compare st2 and v\n", swapped, v)
}
```
运行结果
```
{zhangsan 18}
after swap: v={lisi 19}
after swap: old={zhangsan 18}
compare st1 and v
 false {{lisi 19}}
compare st2 and v
 true {{wangwu 20}}
```

## sync.pool
`sync.Pool`是在`sync`包下的一个内存池组件，用来实现对象的复用，避免重复创建相同的对象，造成频繁的内存分配和gc，以达到提升程序性能的目的。虽然池子中的对象可以被复用，但是是`sync.Pool`并不会永久保存这个对象，池子中的对象会在一定时间后被gc回收，这个时间是随机的。所以，用`sync.Pool`来持久化存储对象是不可取的。
另外，`sync.Pool`本身是并发安全的，支持多个`goroutine`并发的往`sync.Pool`存取数据

### sync.pool使用方法
关于`sync.Pool`的使用，一般是通过三个方法来完成的

| 方法 | 说明 |
| --- | --- |
| New() | sync.Pool的构造函数，用于指定sync.Pool中缓存的数据类型，当调用Get方法从对象池中获取对象的时候，对象池中如果没有，会调用New方法创建一个新的对象 |
| Get() | 从对象池取对象 |
| Put() | 往对象池放对象，下次Get的时候可以复用 |

下面通过例子看一下`sync.Pool`的使用方式
```go
package main

import (
    "fmt"
    "sync"
)

type Student struct {
    Name string
    Age  int
}

func main() {
    pool := sync.Pool{
       New: func() interface{} {
          return &Student{
             Name: "zhangsan",
             Age:  18,
          }
       },
    }

    st := pool.Get().(*Student)
    println(st.Name, st.Age)
    fmt.Printf("addr is %p\n", st)

    pool.Put(st)

    st1 := pool.Get().(*Student)
    println(st1.Name, st1.Age)
    fmt.Printf("addr1 is %p\n", st1)
}
```
程序输出
```
zhangsan 18
addr is 0x140000a0018
zhangsan 18
addr1 is 0x140000a0018
```
在程序中，首先初始化一个`sync.Pool`对象，初始化里面的`New`方法，用于创建对象，这里是返回一个`Student`类型的指针。第一次调用`pool.Get().(*Student)`的时候，由于池子内没有对象，所以会通过`New`方法创建一个，注意`pool.Get()`返回的是一个`interface{}`，所以我们需要断言成`*Student`类型，在我们使用完，打印出`Name`和`Age`之后，再调用`Put`方法，将这个对象放回到池子内，后面我们紧接着又调用`pool.Get()`取对象，可以看到两次去除的对象地址是同一个，说明是同一个对象，表明`sync.Pool`有缓存对象的功能。
>注意
>我们在第一次`pool.Get()`取出`*Student`对象打印完地址之后，put进池子的时候没有进行一个Reset的过程，这里是因为我们取出`*Student`对象之后，仅仅是读取里面的字段，并没有修改操作，假设我们有修改操作，那么这里就需要在`pool.Put(st)`之前执行`Reset`，将对象的值复原，如果不这样做，那么下一次`pool.Get()`取出的`*Student`对象就不是我们希望复用的初始对象

假设我们对`*Student`做修改
```go
package main

import (
    "fmt"
    "sync"
)

type Student struct {
    Name string
    Age  int
}

func main() {
    pool := sync.Pool{
       New: func() interface{} {
          return &Student{
             Name: "zhangsan",
             Age:  18,
          }
       },
    }

    st := pool.Get().(*Student)
    println(st.Name, st.Age)
    fmt.Printf("addr is %p\n", st)

    // 修改
    st.Name = "lisi"
    st.Age = 20

    // 回收
    pool.Put(st)

    st1 := pool.Get().(*Student)
    println(st1.Name, st1.Age)
    fmt.Printf("addr1 is %p\n", st1)
}
```
程序输出
```
zhangsan 18
addr is 0x1400000c030
lisi 20
addr1 is 0x1400000c030
```
可以看到，我们第二次取出的对象虽然和第一次是同一个，地址形同，但是对象的字段值却发生了变化，不是我们初始化的对象了，我们想要一只重复使用一个相同的对象的话，显然这里有问题。所以，我们需要在`pool.Put(st)`回收对象之前，进行对象的`Reset`操作，将对象值复原，同时在每次我们`pool.Get()`取出完对象使用完毕之后，也不要忘了调用`pool.Put`方法把对象再次放入对象池，以便对象能够复用。

### sync.pool使用场景
1. `sync.pool`主要是通过对象复用来降低gc带来的性能损耗，所以在高并发场景下，由于每个`goroutine`都可能过于频繁的创建一些大对象，造成gc压力很大。所以在高并发业务场景下出现 GC 问题时，可以使用 `sync.Pool` 减少 GC 负担
2. `sync.pool`不适合存储带状态的对象，比如socket 连接、数据库连接等，因为里面的对象随时可能会被gc回收释放掉
3. 不适合需要控制缓存对象个数的场景，因为`Pool` 池里面的对象个数是随机变化的，因为池子里的对象是会被gc的，且释放时机是随机的

