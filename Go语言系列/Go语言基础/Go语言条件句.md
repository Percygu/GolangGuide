---
tags:
  - Go
  - golang
  - go基础语法
  - 条件语句
  - 流程控制
---

# Go语言条件语句

条件语句是编程语言中最基本也是最常用的语句之一，它让程序可以根据不同的条件执行不同的代码块。Go语言提供了灵活而强大的条件语句支持。

## if 语句

### 基本语法
Go语言中的if语句不需要使用括号将条件包含起来，但是程序体必须使用大括号。基本语法如下：

```go
if condition {
    // code to execute if condition is true
}
```

例如：
```go
package main

import "fmt"

func main() {
    age := 18
    if age >= 18 {
        fmt.Println("你已经成年了")
    }
}
```
运行结果：
```
你已经成年了
```

### if-else 语句
当条件不满足时，可以使用else执行另一段代码：

```go
package main

import "fmt"

func main() {
    score := 85
    if score >= 90 {
        fmt.Println("优秀")
    } else {
        fmt.Println("继续努力")
    }
}
```
运行结果：
```
继续努力
```

### if-else if-else 语句
当需要判断多个条件时，可以使用if-else if-else结构：

```go
package main

import "fmt"

func main() {
    score := 85
    if score >= 90 {
        fmt.Println("优秀")
    } else if score >= 80 {
        fmt.Println("良好")
    } else if score >= 60 {
        fmt.Println("及格")
    } else {
        fmt.Println("不及格")
    }
}
```
运行结果：
```
良好
```

### if 的特殊写法
Go语言支持在if条件判断之前执行一个简单的语句：

```go
package main

import "fmt"

func main() {
    if num := 9; num < 10 {
        fmt.Println(num, "小于10")
    } else {
        fmt.Println(num, "大于等于10")
    }
    // 注意：num变量的作用域仅在if-else语句块内
}
```
运行结果：
```
9 小于10
```

## switch 语句

### 基本用法
switch语句是多条件判断的另一种选择，相比于if-else更加清晰和简洁。Go语言的switch比传统的C/C++语言更加灵活：
- 不需要写break语句，匹配到case后自动中断
- 可以使用任意类型作为条件值，不限于常量或整数
- case语句可以使用表达式

基本语法示例：
```go
package main

import "fmt"

func main() {
    day := 3
    switch day {
    case 1:
        fmt.Println("星期一")
    case 2:
        fmt.Println("星期二")
    case 3:
        fmt.Println("星期三")
    case 4:
        fmt.Println("星期四")
    case 5:
        fmt.Println("星期五")
    default:  // 可选的默认分支，当所有case都不匹配时执行
        fmt.Println("周末")
    }
}
```
运行结果：
```
星期三
```

### switch的特殊用法

1. 一个case可以有多个值，多个值之间使用逗号分隔：
```go
switch day {
case 1, 2, 3, 4, 5: // 可以匹配多个值，只需要满足其中一个即可
    fmt.Println("工作日")
case 6, 7:
    fmt.Println("周末")
}
```

2. 省略switch后的表达式，这种形式更接近于if-else结构：
```go
score := 85
switch {  // 省略条件表达式，默认为true
case score >= 90:
    fmt.Println("优秀")
case score >= 80:
    fmt.Println("良好")
case score >= 60:
    fmt.Println("及格")
default:
    fmt.Println("不及格")
}
```

3. fallthrough关键字：
Go语言中的switch默认带有break效果，但如果需要继续执行下一个case，可以使用fallthrough：
```go
switch {
case score >= 60:
    fmt.Println("及格")
    fallthrough  // 继续执行下一个case，不管下一个case的条件是否满足
case score >= 0:
    fmt.Println("分数有效")
}
```
运行结果：
```
及格
分数有效
```
注意：fallthrough必须是case中的最后一条语句，并且会强制执行下一个case的代码块，而不判断条件。

4. switch可以用于类型判断：
```go
var x interface{} = 25.0

switch v := x.(type) {
case int:
    fmt.Printf("x是整数，值为%d\n", v)
case float64:
    fmt.Printf("x是浮点数，值为%.2f\n", v)
case string:
    fmt.Printf("x是字符串，值为%s\n", v)
default:
    fmt.Printf("x的类型未知\n")
}
```
运行结果：
```
x是浮点数，值为25.00
```

### switch注意事项
1. case的值必须是唯一的，不能重复
2. 每个case默认带有break，不用显式写出
3. case后的表达式必须与switch的表达式类型一致
4. 使用fallthrough时要特别小心，确保逻辑正确
5. default分支可选，但建议添加以处理未考虑的情况

## 实际应用示例

### 1. 错误处理
```go
if err := doSomething(); err != nil {
    // 处理错误
    fmt.Println("发生错误:", err)
    return
}
```

### 2. 类型判断
```go
var i interface{} = "Hello"

switch v := i.(type) {
case string:
    fmt.Printf("字符串: %s\n", v)
case int:
    fmt.Printf("整数: %d\n", v)
case bool:
    fmt.Printf("布尔值: %v\n", v)
default:
    fmt.Printf("未知类型\n")
}
```

### 3. 状态机
```go
type State int

const (
    Idle State = iota
    Running
    Paused
    Stopped
)

func handleState(state State) {
    switch state {
    case Idle:
        fmt.Println("系统空闲中")
    case Running:
        fmt.Println("系统运行中")
    case Paused:
        fmt.Println("系统已暂停")
    case Stopped:
        fmt.Println("系统已停止")
    }
}
```

## 使用技巧和注意事项

1. 代码风格
   - if和else后的`{`必须和if或else在同一行
   - 优先使用正向逻辑，避免多重否定
   - 优先处理错误情况，让主逻辑更清晰

2. 性能考虑
   - 多条件判断时，将最可能满足的条件放在前面
   - 对于多个条件的判断，考虑使用switch替代if-else链

3. 常见陷阱
   - 注意if语句中初始化变量的作用域
   - 不要在switch的case中漏写break（Go自动break）
   - 使用fallthrough时要特别小心，确保逻辑正确

## 小结

条件语句是程序流程控制的基础，Go语言通过简洁的语法和一些特殊的设计，让条件语句的使用更加灵活和高效。掌握好条件语句的各种用法，对于编写清晰、可维护的代码至关重要。

记住：选择合适的条件语句形式，让代码逻辑更清晰，可读性更好。过于复杂的条件判断应考虑重构，可能需要重新设计数据结构或使用其他模式。

