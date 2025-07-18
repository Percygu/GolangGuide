# Go语言结构体

有时候内置的基本类型并不能满足我们的业务需求，我们需要一些复合结构。比如我们想要描述一个学生，1个学生既有学号，年龄，性别，分数等这些属性，而单一的数据类型往往只能描述其中一个属性，我们想要描述这个学生，就需要把这些属性都要描述出来，这个时候就需要用到结构体了。

## 结构体定义
像很多其他的高级语言一样，go语言也支持结构体来定义复合数据类型
定义方式如下：
```go
type Student struct {
    ID int
    Name string
    Age int
    Score int
}   
```
上述方法定义了一个`Student`类型的结构体，`Student`包含四个属性，分别是`string`类型的`Name`和`int`类型的`ID`，`Age`和`Score`。

## 结构体初始化
### 键值对初始化
在初始化的时候以属性：值的方式完成，如果有的属性不写，则为默认值
```go
package main

import "fmt"

type Student struct {
    ID int
    Name string
    Age int
    Score int
}


func main() {
    st := Student{
        ID : 100,
        Name : "zhangsan",
        Age : 18,
        Score : 98,
    }
    fmt.Printf("学生st: %v\n", st)
}
```
运行结果：
```
学生st: {100 zhangsan 18 98}
```

### 值列表初始化
在初始化的时候直接按照属性顺序以属性值来初始化，看下面例子
```go
package main

import "fmt"

type Student struct {
    ID int
    Name string
    Age int
    Score int
}


func main() {
    st := Student{
       101,
       "lisi",
       20,
       97,
    }
    fmt.Printf("学生st: %v\n", st)
}
```
运行结果：
```
学生st: {101 lisi 20 97}
```
**注意：以值列表的方式初始化结构体，值列表的个数必须等于结构体属性个数，且要按顺序，否则会报错**

## 结构体成员访问   
使用点号 `.` 操作符来访问结构体的成员，`.` 前可以是结构体变量或者结构体指针
```go
package main

import "fmt"

type Student struct {
    ID int
    Name string
    Age int
    Score int
}


func main() {
    st1 := Student{
        ID : 100,
        Name : "zhangsan",
        Age : 18,
        Score : 98,
    }
    fmt.Printf("学生1的姓名是: %s\n", st1.Name)
    
    st2 := &Student{
        ID : 101,
        Name : "lisi",
        Age : 20,
        Score : 97,
    }
    fmt.Printf("学生2的分数是: %d\n", st2.Score)
}   
```
运行结果：
```
学生1的姓名是: zhangsan
学生2的分数是: 97
```

