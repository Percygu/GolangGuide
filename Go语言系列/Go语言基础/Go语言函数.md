---
tags:
  - Go
  - golang
  - go基础语法
  - 函数
---

# Go语言函数

同其他语言一样，Go语言也有函数的概念，主要是为了代码复用，函数是基本的代码块，用于执行一个任务。
Go 语言最少有`main()`函数，同时`main()`函数也是整个程序执行的入口。

## 函数定义
Go语言函数定义格式如下：
```go
func function_name([parameter list]) [return_types] {
   function_body
}
```
- func：函数关键字，任何一个函数都有func关键字开始声明。
- function_name：函数名称，参数列表和返回值类型构成了函数签名。
- parameter list：参数列表，参数列表指定的是参数类型、顺序、及参数个数。参数是可选的，也就是说函数也可以不包含参数。
- return_types：返回类型，函数返回一列值。`return_types`是该列值的数据类型。有些功能不需要返回值，这种情况下return_types不是必须的。
- 函数体：函数定义的代码集合。
- 注意：Go语言函数区别C++和和Java语言的地方是可以有多个返回值，多个返回值用小括号括起来，中间用逗号分隔。

代码展示：
```go   
package main

import "fmt"

func swap(x, y string) (string, string) {
   return y, x
}

func main() {
   a, b := swap("Google", "Hello")
   fmt.Println(a, b)
}
```
运行结果：
``` 
Hello Google
```

## 函数调用
当创建函数时，你定义了函数需要做什么，通过调用该函数来执行指定任务。
调用函数，向函数传递参数，并返回值，例如：
```go
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var a int = 100
   var b int = 200
   var ret int

   /* 调用函数并返回最大值 */
   ret = max(a, b)

   fmt.Printf( "最大值是 : %d\n", ret )
}

/* 函数定义：返回两个数的最大值 */
func max(num1, num2 int) int {
   /* 定义局部变量 */
   var result int

   if num1 > num2 {
      result = num1
   } else {
      result = num2
   }
   return result
}
```
运行结果：
```
最大值是 : 200
```

## 参数传递
在Go语言中函数的参数传递都是值传递，不存在引用传递(区别于c++)
**注意：使用值传递的时候，虽然会改变形参的值，但并不会改变函数外变量即实参的值**
代码展示：
```go
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var a int = 100

   fmt.Printf("自增前 a 的值为 : %d\n", a )

   add(a)

   fmt.Printf("自增后 a 的值 : %d\n", a )
}

/* 函数定义：自增1 */
func add(a int)  {
   a++;
   fmt.Printf("add里a的值：%d\n", a)
}
```
运行结果：
```
自增前 a 的值为 : 100
add里a的值：101
自增后 a 的值 : 100
```

## 函数用法

### 函数变量
一切皆变量，Go语言里什么都可以当作变量来使用，当然函数也不例外，函数可以作为函数变量。Go语言可以很灵活的创建函数，并作为另外一个函数的实参。以下实例中我们在定义的函数中初始化一个变量，该函数仅仅是为了使用内置函数
代码展示：
```go
package main

import (
   "fmt"
   "math"
)

func main(){
   /* 声明函数变量 */
   getSquareRoot := func(x float64) float64 {
      return math.Sqrt(x)  // 求一个数的平方根
   }

   /* 使用函数变量调用函数 */
   fmt.Println(getSquareRoot(9))
}
```
运行结果：
```
3
```

再举一个用函数变量作回调函数的例子：

```go
package main 

import "fmt" 

// 声明一个函数类型 
type fc func(int) int

func main() { 
   CallBack(1, callBack)//执行函数---CallBack 
} 

func CallBack(x int, f fc) {  //定义了一个函数 testCallBack
    f(x)  //由于传进来的是callBack函数，该函数执行需要传入一个int类型参数，因此传入x 
} 

func callBack(x int) int { 
   fmt.Printf("我是回调，x：%d\n", x) 
   return x 
}
```
运行结果：
```
我是回调，x：1
```

### 函数闭包
所谓闭包，就是匿名函数，Go语言支持匿名函数调用。匿名函数就是一个"内联"语句或表达式。匿名函数的优越性在于可以直接使用函数内的变量，不必申明，这样有时候可以使代码更简单，增强代码的可读性。
代码展示：
```go
package main

import "fmt"

func getNumber() func() int {
   i:=0
   return func() int {
      i+=1
     return i  
   }
}

func main(){
   /* nextNumber 为一个函数，函数中 i 为 0 */
   nextNumber := getNumber()  

   /* 调用 nextNumber 函数，i 变量自增 1 并返回 */
   fmt.Println(nextNumber())
   fmt.Println(nextNumber())
   fmt.Println(nextNumber())
   
   /* 创建新的函数 nextNumber1，并查看结果 */
   nextNumber1 := getNumber()  
   fmt.Println(nextNumber1())
   fmt.Println(nextNumber1())
}
```
运行结果：
```
1
2
3
1
2
```
> **注意：多次调用同一个函数`nextNumber`，其中的i变量是同一个，初始值为0，每调用一次，i自增1**

代码解释：

首先，`getNumer`这个函数会返回一个闭包函数，我们先理解闭包是啥，题库里面我也写过的，闭包就是等于 匿名函数+捕获的变量，我们可以看到第7行-第10行，闭包函数里面捕获了i变量，因为i不是在闭包函数里面定义的，
第15行代码，通过`getNumber`获取了一个闭包函数——`nextNumber`，18，19，20行分别执行一次`nextNumber`函数，每次i就会自增1，并且因为每次调用`i`都是固定的，结果会累计，所以会输出1，2，3。23行重新获取了一个闭包函数，`i`是独立于上面闭包函数的，所以重新输出1，2，3



