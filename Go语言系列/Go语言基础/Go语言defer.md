---
tags:
  - Go
  - golang
  - go基础语法
  - defer
---

# Go语言defer

## defer关键字
`defer`顾名思义，延迟。它是go语言中的一个关键字，主要用在函数或方法前面，作用是用于函数和方法的延迟调用，在语法上，`defer`与普通的函数调用没有什么区别。
在使用上非常简单，只需要弄清楚以下几点即可：
1. 延迟的函数的什么时候被调用？
  1. 函数`return`的时候
  2. 发生`panic`的时候
2. 延迟调用的语法规则
  1. `defer`关键字后面表达式必须是函数或者方法调用
  2. 延迟内容不能被括号括起来

  ## defer执行顺序
上一小节说到`defer`关键字后面的函数调用会在函数`return`或者发生`panic`的时候执行，这个在单个`defer`的时候很好理解，但当一个函数中有多个`defer`的时候，他们的顺序是怎么样的呢？
`defer`语句的执行顺序是先进后出LIFO。下面请看具体例子：
  ```go
  package main

import "fmt"

func defer1() {
        fmt.Println("defer1")
}

func defer2() {
        fmt.Println("defer2")
}

func defer3() {
        fmt.Println("defer3")
}

func main() {
        defer defer1()
        defer defer2()
        defer defer3()
}
```
运行结果：
```
defer3
defer2
defer1
```
可以看到执行顺序跟栈是一样的，先调用，后执行

## defer的使用场景
通过前面的小节我们知道了`defer`关键字主要是用于延迟调用，那么什么场景下需要我们用到延迟调用了，有过Go语言基础的同学在一些代码中经常看到`defer`关键字。`defer`关键字一般用在以下两个场景中

### 资源的释放
通过`defer`延迟调用机制，我们可以简洁优雅处理资源回收问题，从而避免在复杂的代码逻辑情况下，遗漏相关的资源回收问题，用的比较多的就是类似网络连接，数据库连接，以及文件句柄的资源的释放。
看看下面一个复制文件的函数:
```go
func CopyFile(dstFile, srcFile string) (wr int64, err error) { 
    src, err := os.Open(srcFile)     
    if err != nil {         
        return     
    }      
    dst, err := os.Create(dstFile)     
    if err != nil {         
        return    
    }      
    wr, err = io.Copy(dst, src)     
    dst.Close()     
    src.Close()     
    return 
}
```
仔细看这段代码，其实是有问题的，比如当地6行执行失败，程序直接返回了，但我们并没有关闭前面打开的文件资源src，这样就造成了资源的浪费。
那么用`defer`关键字，我们可以怎么做呢？
```go
func CopyFile(dstFile, srcFile string) (wr int64, err error) {
    src, err := os.Open(srcFile)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstFile)
    if err != nil {
        return
    }
    defer dst.Close()
    
    wr, err = io.Copy(dst, src)  
    return wr, err
}
```
只要我们正确打开了某个资源，比如`src`和`dst`，没返回`err`的情况下，都可以用`defer`延迟调用来关闭资源，注意，这是`go`语言中非常常见的一种资源关闭方式。

### 配合recover一起处理panic
`defer`另一个常用的地方就是在处理程序`panic`的时候，关于程序的异常捕获我们将在下一个小节讲到，这里大家可以先了解一下，`go`语言中用`panic`来抛出异常，用`recover`来捕获异常，所以当我们的程序出现异常的时候，我们需要知道是发生了什么异常的时候，就可以用`defer recover`来捕获异常

```go
package main

import "fmt"

func main() {
   defer func() {
      if r := recover(); r != nil {
         fmt.Println(r)
      }
   }()
   a := 1
   b := 0
   fmt.Println("result:", a/b)
}
```
运行结果：
```
result: runtime error: integer divide by zero
```
可以看到，程序并没有输出`result`，这是因为我们尝试对一个除数为0的数做除法，这是不允许的，所以程序回panic，但我们用`defer`在程序发生`panic`的时候捕获了这个异常，打印出异常信息：`runtime error: integer divide by zero`。

## defer与return
前面第一小节我们介绍过`defer`函数的执行是在`return`的时候，那么在具体一点，在`return`的时候，`defer`具体做了什么？又会带来什么结果？这是一个非常值得探讨的问题，也是面试官在面试中经常会问的问题，往往通过这个问题就可一看出一个面试者对`go`语言掌握的扎不扎实。
1. 例子1：
```go
package main

import "fmt"


func deferRun() {
  var num = 1
  defer fmt.Printf("num is %d", num)
  
  num = 2
  return
}

func main(){
    deferRun()
}
```
运行结果：  
```
num is 1
```
为什么？
延迟函数 `defer fmt.Printf("num is %d", num)` 的参数`num`在`defer`语句出现的时候就已经确定，`num=1`，所以不管后面怎么修改 `a` 的值，最终调用`defer`函数传递给`defer`函数的参数已经固定是`1`了，不会再变化。

2. 例子2：
```go
package main

import "fmt"

func main() {
 deferRun()
}

func deferRun() {
 var arr = [4]int{1, 2, 3, 4}
 defer printArr(&arr)
 
 arr[0] = 100
 return
}

func printArr(arr *[4]int) {
 for i := range arr {
  fmt.Println(arr[i])
 }
}   
```
运行结果：
```
100
2
3       
4
```
为什么？
通过前一个地址，我们知道在defer出现的时候，参数已经确定，但是这里传递的是地址，地址没变，但是地址对应的内容被修改了，所以输出会被修改。

3. 例子3：
```go
package main

import "fmt"

func main() {
   res := deferRun()
   fmt.Println(res)
}

func deferRun() (res int) {
  num := 1
  
  defer func() {
    res++
  }()
  
  return num
}
```
运行结果：
```
2
```
为什么？    
这是一个非常经典的例子，要想准确的的只程序的执行结果，需要我们对函数return的执行有一个细致的了解。其实函数的return并非一个原子操作，return的过程可以被分解为以下三步：
1. 设置返回值
2. 执行defer语句
3. 将结果返回
所以，在本例中，第一步是将`result`的值设置为`num`，此时还未执行`defer`，`num`的值是`1`，所以`result`被设置为`1`，然后再执行`defer`语句将`result+1`，最终将`result`返回，所以会打印出`2`。

4. 例子4：
```go
package main

import "fmt"

func main() {
    res := deferRun()
    fmt.Println(res)
}

func deferRun() int {
  var num int
  defer func() {
    num++
  }()
  
  return 1
}
```
运行结果：
```
1
```
为什么？    
本例和前面的区别返回值是匿名的，但是我们可以同样运用上面的思路，自己创建一个返回值，这里假设为`res`，运用前面的思路分析，第一步将`res`设置为1，第二步执行`defer`将`num+1`，第三步将`res`返回，所以最终结果是`1`。

5. 例子5：
```go
package main

import "fmt"

func main() {
    res := deferRun()
    fmt.Println(res)
}

func deferRun() int {
  num := 1
  defer func() {
    num++
  }()
  
  return num
}
```
运行结果：
```
1
```
同样的思路不难分析：自己创建一个返回值，这里假设为`res`，第一步将`res`设置为`num`，所以`res`的值为`1`，第二步执行`defer`将`num+1`，此时`num`为`2`，但是`res`为`1`，第三步将`res`返回，所以最终结果是`1`。

6. 例子6：
```go
package main

import "fmt"

func main() {
    res := deferRun()
    fmt.Println(res)
}

func deferRun() (res int) {
  num := 1
  defer func() {
    num++
  }()
  
  return num
}  
```
运行结果：
```
1
```
不难分析运行结果还是1，同样的三步分析法，因为`defer`改变的是`num`的值，而不是改变的`res`的值，所以结果不会变，不过`defer`函数里变为`res++`，那么结果就是`2`了。
所以，当我们碰到defer与return确定最终的返回值，可以总结为以下两点：
1. defer 定义的延迟函数的参数在`defer`语句出时就已经确定下来了
2. `return`不是原子级操作，执行过程是: 设置返回值—>执行`defer`语句—>将结果返回



