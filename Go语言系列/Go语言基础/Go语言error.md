---
tags:
  - Go
  - golang
  - go基础语法
  - error
---

# Go语言error
在`Go`语言开发中，我们会多次用到`error`这个东西，尤其是在函数的返回值上，由于`go`的函数支持多返回值，所以一般会用`error`作为其中一个返回值，代表该函数执行过程中或者逻辑有出错，那究竟`error`是个什么东西呢，下面我们就来一探究竟。

## error是什么
`error`其实是Go语言的一个接口类型，就是一个普通的接口，并且也不会携带任何的堆栈信息。
接口的定义如下：
```go
type error interface {
    Error() string
}
```
通常我们会使用`errors.New()`或者`fmt.Errorf()`来返回一个`error`对象，但是需要注意，通过这两种方式返回的`error`对象都是不可以进行比较的，因为`errors.New()`返回的其实是一个地址，不能用来做等值判断，如果要是先等值判断，需要自己实现，而`fmt.Error()`的内部其实也是用到了`errors.New()`。

1. `errors.New()`函数实现：
```go
func New(text string) error {
    return &errorString{text}
}
``` 
2. `fmt.Errorf()`实现：
```go
func Errorf(format string, a ...interface{}) error {
   p := newPrinter()
   p.wrapErrs = true
   p.doPrintf(format, a)
   s := string(p.buf)
   var err error
   if p.wrappedErr == nil {
      err = errors.New(s)
   } else {
      err = &wrapError{s, p.wrappedErr}
   }
   p.free()
   return err
}
```
代码展示：
```go
package main

import （
    "fmt"
    "errors"
）
// 定义一个正数自加的函数，当传入的整数小于等于0的时候报错
func getPositiveSelfAdd(num int) (int,error) {
    if num <=0 {
        return -1,fmt.Errorf("num is not a positive number")
    }
    return num+1,nil
}

func main() {
   num1,err1 := getPositiveSelfAdd(1)
   fmt.Printf("nums is %d, err is %v\n",num1 ,err1)
   
   num2,err2 := getPositiveSelfAdd(-2)
   fmt.Printf("nums is %d, err is %v\n",num2 ,err2)
   
   err3 := errors.New("hello")
   err4 := errors.New("hello")
   fmt.Println(err3==err4)
   
   fmt.Println(err3.Error()==err4.Error())
}
```
运行结果：
```
nums is 2, err is <nil>
nums is -1, err is num is not a positive number
false
false
```
通过样例可以看到，通过上述方法创建出的两个`error`对象是不能直接比较的，即便是`error`信息一样，也会返回`false`，如果我们想要比较，可以通过`Error()`方法拿到其中的`error`字符串信息，比较字符串。

## 自定义error对象
Go语言内置的`error`创建方法非常简单，也易上手，但是有时候并不能满足我们的业务要求，因为他只返回了错误信息，类似于`error`的`message`，但是很多时候我们业务上还需要错误码，即`error code`。所以，很多时候我们可以自定义`error`对象。
```go
type MyError struct {
    code int
    msg string
}
```
对象定义完之后，接下来只需要实现`error`接口的`Error`方法即可，这样，我们就自定义了一个同时带有错误码和错误信息的error对象。
下面请看具体例子：
```go
package main

import "fmt"

type MyError struct {
    code int
    msg string
}

func (m MyError) Error() string {
   return fmt.Sprintf("code:%d,msg:%v", m.code, m.msg)
}

func NewError(code int, msg string) error {
   return MyError{
      code: code,
      msg:  msg,
   }
}

func Code(err error) int {
   if e, ok := err.(MyError); ok {
      return e.code
   }
   return -1
}

func Msg(err error) string {
   if e, ok := err.(MyError); ok {
      return e.msg
   }
   return ""
}

func main() {
    err := NewError(100, "test MyError")
    fmt.Printf("code is %d, msg is %s", Code(err), Msg(err))
}
```
运行结果：
```
code is 100, msg is test MyError
``` 








