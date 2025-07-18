---
tags:
  - Go
  - golang
  - go基础语法
  - 方法
---

# Go语言方法

## 方法的定义
方法是一类特殊的函数，方法是绑定在某种类型的变量上的函数,它有一定的约束范围。而变量的类型不仅仅局限于结构体类型，可以是任意类型，比如`int`、`bool`等的别名类型。
有过c++或者java基础的同学可以这样理解，一个`struct`加上绑定在这个类型上的方法就等价于一个类。
方法的定义格式如下

```go
func (变量 类型) 方法名(参数列表)(返回值列表){
    
}   
```

- 变量：表示方法接收者，即方法所属的对象
- 类型：表示变量的类型
- 方法名：表示方法的名称
- 参数列表：表示方法的参数
- 返回值列表：表示方法的返回值

对比前面我们学习过的函数，方法跟函数的唯一区别就是在`func`关键字和函数名之间加了一个接收者类型和接收者，同样，这个接收者的类型可以是普通变量或者指针变量
方法的调用跟结构体属性的调用方式相同，都是通过`.`操作符来完成
下面给`Student`结构定义一个方法
```go
package main

import "fmt"

type Student struct {
    ID int
    Name string
    Age int
    Score int
}

func (st *Student) GetName() string {
    return st.Name
}

func main() {
    st := &Student{
        ID : 100,
        Name : "zhangsan",
        Age : 18,
        Score : 98,
    }
    fmt.Printf("学生st的姓名是: %s\n", st.GetName())    // 调用Student的方法GetName
}
```
运行结果：
```
学生st的姓名是: zhangsan
```

## 方法的调用
定义在指针类型或者值类型上的方法可以通过指针变量或者值变量来调用，可能读起来有点绕，下面看例子，会更清晰
```go
package main

import "fmt"

type Student struct {
    ID int
    Name string
    Age int
    Score int
}

func (st *Student) SetScore(score int) {
    st.Score = score
}

func (st Student) GetScore() int{
    return st.Score
}

func main() {
    st := &Student{
        ID : 100,
        Name : "zhangsan",
        Age : 18,
        Score : 98,
    }
    fmt.Printf("设置前，学生st的分数是: %d\n", st.GetScore())    // 通过指针调用定义在值类型的方法GetScore
    st.SetScore(100)                                       // 通过指针调用定义在指针类型上的方法
    fmt.Printf("设置后，学生st的分数是: %d\n", st.GetScore())    // 通过指针调用定义在值类型的方法GetScore
}   
```
运行结果：
```
设置前，学生st的分数是: 98
设置后，学生st的分数是: 100
```
## 继承
go语言不像c++或者java一样有显示的继承关系，因为go语言没有类的概念，所以自然就不存在父类，子类一说，自然就没有继承的关系。那么，go语言是如何实现功能的呢？
答案是：**组合**，go语言中不论是属性，还是方法的继承都是通过组合来实现的。
请看下面例子
```go
type People struct {
    Name string
    Age int
}

type Student struct {
    ID int
    Score int
    People    // 将People作为Student的一个属性，注意不要加类型，这就是隐式继承
}
```
将原来`Studen`t的定义改为`Student`和`People`两个结构来定义，通过这种组合的方式，其实`Student`就拥有了`People`的`Name`属性和`Age`属性
代码展示
```go
package main

import "fmt"

type People struct {
    Name string
    Age int
}

func (p *People) GetName() string{
    return p.Name
}

type Student struct {
    ID int
    Score int
    People    // 将People作为Student的一个属性，注意不要加类型，这就是隐式继承
}

func (st *Student) SetScore(score int) {
    st.Score = score
}

func (st Student) GetScore() int{
    return st.Score
}

func main() {
    st := &Student{
        ID : 100,
        Score: 98,
        People : People {
            Name : "zhangsan",
            Age : 18,
        },
    }
    fmt.Printf("学生st的分数是: %d\n", st.GetScore())    // 通过指针调用定义在值类型的方法GetScore                                         // 通过指针调用定义在指针类型上的方法
    fmt.Printf("学生st的姓名是: %s\n", st.GetName())
}
```
运行结果：
```
学生st的分数是: 98
学生st的姓名是: zhangsan
```
这个例子中，`Student`通过内嵌`People`获得了`People`的`Name`和`Age`属性以及`GetName`方法。








