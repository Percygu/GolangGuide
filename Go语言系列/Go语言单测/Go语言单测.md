---
tags:
  - Go
  - golang
  - go单测
  - 单元测试
---

# Go单元测试


## 1. 什么是单元测试

单元测试是软件开发中的一种测试方法，旨在验证代码中最小可测试单元（如函数、方法、类）的行为是否符合预期，它是开发流程的重要组成部分。单元测试的目标是发现代码中的缺陷和错误，并确保代码的正确性和稳定性。

Go语言中自带有一个轻量级的测试框架testing和自带的go test命令来实现单元测试和性能测试。在包目录内，以\_test.go为后缀名的源文件都是go test的一部分，而不是go build的构建部分。

## 2. Go单元测试命名规范

1. 在 Go 中，测试文件的命名规则非常重要。测试文件**必须以 `_test.go` 结尾**，否则 Go 的测试框架在执行时将不会识别这些文件。

举个例子，如果你的主程序文件名是 `hello.go`，那么对应的测试文件应命名为 `hello_test.go`。

注意：

* 测试方法名以`Test`开头，参数要用`testing` ，例如`func TestXxx(t *testing.T)`

* 在测试的时候通过`go test`命令进行测试

## 3. Go语言的测试框架

Go语言有以下几种常见的测试框架：

| 测试框架         | 推荐指数  |
| ------------ | ----- |
| Go原生testing包 | ★★★☆☆ |
| GoConvey     | ★★★★★ |
| testify      | ★★★☆☆ |

从测试用例编写的复杂度来看：testify比GoConvey简单；GoConvey比Go自带的testing包简单。然而在测试框架的选择上，我们更推荐使用GoConvey，主要原因有：

* GoConvey与其他Stub/Mock框架的兼容性比Testify更好

* Testify虽然自带Mock功能，但需要手动编写Mock类；而GoMock可以一键自动生成这些重复的代码

接下来也会重点讲一下Go原生testing包的单测写法和GoConvey的主要用法

## 4. Go自带的testing包

`testing`包为Go语言的package提供了自动化测试支持。通过`go test`命令，可以自动执行如下形式的任何函数：

```go
func TestXxx(*testing.T)
```

注意：`Xxx`可以是任何字母数字字符串，但第一个字母不能是小写字母。在这些测试函数中，可以使用`Error`、`Fail`等方法来指示测试失败。

要创建一个新的测试套件，需要创建一个名称以`_test.go`结尾的文件，该文件包含上述`TestXxx`函数。将该文件放在与被测试文件相同的包中。该文件会在正常的程序包构建中被排除，但在运行`go test`命令时会被包含。更多详情可执行`go help test`和`go help testflag`查看。

> 创建一个新的文件夹hello1，作为项目的根目录。在项目中根目录创建一个文件夹gotest，在gotest目录中创建一个example.go用于编写被测试代码，然后同样在gotest目录中创建一个example\_test.go，用于编写测试代码。后续的所有测试用例都是在此基础上进行

### 4.1 基础示例

被测试代码：

```go
package gotest

func Factorial(n int) int {
    if n <= 0 {
       return 1
    }
    return n * Factorial(n-1)
}
```

测试代码：

```go
func TestFactorial(t *testing.T) {
    var (
        input    = 5
        expected = 120
    )
    actual := Factorial(input)
    if actual != expected {
        t.Errorf("Factorial(%d) = %d; expected %d", input, actual, expected)
    }
}
```

在gotest目录下执行`go test .`，输出：

```go
➜  gotest go test .
ok      hello1/gotest   0.332s
```

注意：➜  gotest表示是在gotest执行的后面的命令。上述当输入为`inut`时，结果实际结果`actual`和预期结果`expected`相等，表示测试通过。如果我们将`Factorial`函数修改为错误的实现：

```go
func Factorial(n int) int {
    if n <= 0 {
        return 1
    }
    return n * Factorial(n-2) // 错误的递归调用
}
```

再执行`go test .`，将输出：

```go
➜  gotest go test .
--- FAIL: TestFactorial (0.00s)
    example_test.go:12: Factorial(5) = 15; expected 120
FAIL
FAIL    hello1/gotest   0.334s
FAIL
```

### 4.2 Table-Driven测试

Table-Driven方式可以在同一个测试函数中测试多个用例，将`TestFactorial`函数改为最初正确的形式，再次测试

```go
func TestFactorial(t *testing.T) {
    var factorialTests = []struct {
        input    int // 输入值
        expected int // 预期结果
    }{
        {0, 1},
        {1, 1},
        {2, 2},
        {3, 6},
        {4, 24},
        {5, 120},
        {6, 720},
    }

    for _, tt := range factorialTests {
        actual := Factorial(tt.input)
        if actual != tt.expected {
            t.Errorf("Factorial(%d) = %d; expected %d", tt.input, actual, tt.expected)
        }
    }
}
```

程序输出

```go
➜  gotest go test .
ok      hello1/gotest   0.345s
```

Go自带testing包的更多用法可以参考[Go标准库文档](https://golang.org/pkg/testing/)。

## 5. GoConvey

GoConvey适用于编写单元测试用例，并且可以兼容到testing框架中。可以通过`go test`命令或使用`goconvey`命令访问`localhost:8080`的Web测试界面来查看测试结果。GoConvey的基本用法如下：

```go
Convey("测试描述", t, func() {
    So(...)
})
```

GoConvey通常使用`So`函数进行断言，断言方式可以传入一个函数，或者使用内置的`ShouldBeNil`、`ShouldEqual`、`ShouldNotBeNil`等函数。

### 5.1 基本示例

被测试代码：

```go
package gotest

func SlicesEqual(a, b []int) bool {
    if len(a) != len(b) {
       return false
    }

    if (a == nil) != (b == nil) {
       return false
    }

    for i, v := range a {
       if v != b[i] {
          return false
       }
    }
    return true
}
```

测试代码：

```go
package gotest

import (
    . "github.com/smartystreets/goconvey/convey"
    "testing"
)

func TestSlicesEqual(t *testing.T) {
    Convey("测试切片相等性函数", t, func() {
       a := []int{1, 2, 3, 4}
       b := []int{1, 2, 3, 4}
       So(SlicesEqual(a, b), ShouldBeTrue) // a和b相等，这个判定应该为true，如果确实相等，则单测绘PASS，否侧不通过
    })
}
```

这次我们不再使用`go test .`命令，而用`go test -v`命令来查看一下具体的单测执行详情

```go
➜  gotest go test -v
=== RUN   TestSlicesEqual

  测试切片相等性函数 ✔


1 total assertion

--- PASS: TestSlicesEqual (0.00s)
PASS
ok      hello1/gotest   0.231s

```

总共执行了一个断言，测试结果跟我们预测的结果相同，a和b相等，这个判定应该为true，如果确实相等，则单测会PASS，否侧不通过。测试结果为PASS，表示通过。执行耗时为0.231s

### 5.2 嵌套测试

测试代码：

```go
package gotest

import (
    . "github.com/smartystreets/goconvey/convey"
    "testing"
)

func TestSlicesEqual(t *testing.T) {
    Convey("测试切片相等性函数", t, func() {
       Convey("当两个非空切片内容相同时", func() {
          a := []int{1, 2, 3, 4}
          b := []int{1, 2, 3, 4}
          So(SlicesEqual(a, b), ShouldBeTrue)
       })

       Convey("当两个都是nil切片时", func() {
          So(SlicesEqual(nil, nil), ShouldBeTrue)
       })

       Convey("当两个切片长度不同时", func() {
          a := []int{1, 2, 3}
          b := []int{1, 2, 3, 4}
          So(SlicesEqual(a, b), ShouldBeFalse) 
       })
    })
}
```

测试结果：

```go
➜  gotest go test -v
=== RUN   TestSlicesEqual

  测试切片相等性函数 
    当两个非空切片内容相同时 ✔
    当两个都是nil切片时 ✔
    当两个切片长度不同时 ✔


3 total assertions

--- PASS: TestSlicesEqual (0.00s)
PASS
ok      hello1/gotest   0.235s

```

内层的Convey不需要再传入`t *testing.T`参数，这个例子测试了三种情况，当两个非空切片内容相同时，当两个都是nil切片时当两个切片长度不同时的预期情况和真实的代码测试情况，三种情况都是测试通过的

GoConvey的更多用法可以参考[官方文档](https://github.com/smartystreets/goconvey)。

## 6. Stub/Mock框架

在单元测试中，我们往往需要**隔离外部依赖**（如数据库、网络、文件系统、第三方服务等），这时就会用到 **Stub** 和 **Mock** 框架。它们帮助我们**模拟依赖组件的行为**，让测试只聚焦于目标函数的逻辑本身。

### 6.1 Stub 是什么？

**Stub（桩）** 是一种最基础的替代品，它通常是你手动实现的函数或对象，用来返回**固定的值或行为**。

假设你有一个函数 `GetUserName(id)`，会从数据库中查询用户姓名。但在测试中你不想真的连数据库：

```go
func GetUserNameFromDB(id int) string {
    return "RealNameFromDB" // 真实实现，测试中不想调用
}

func GetUserName(id int, dbFunc func(int) string) string {
    return dbFunc(id)
}

```

测试中则可以写一个 Stub：

```go
func StubGetUserName(id int) string {
    return "StubUser"
}

func TestGetUserName(t *testing.T) {
    name := GetUserName(1, StubGetUserName)
    if name != "StubUser" {
        t.Fail()
    }
}

```

**Stub的特点是简单、手动、只模拟"结果"。**

### 6.2 Mock 是什么？

**Mock（模拟）** 是一种更高级的替代品，通常配合框架使用（如：GoMock、Testify）。除了返回值，它还可以**验证调用过程**，比如：

* 被调用了几次？

* 参数是否正确？

* 调用顺序对不对？

比如同样，假设我们有一个函数 `GetUserName(id)`，它依赖一个数据库查询函数 `GetUser(id)`，我们想在测试中 mock 这个函数的行为。

```go

package main

import (
    "testing"

    . "github.com/smartystreets/goconvey/convey"
    "github.com/stretchr/testify/mock"
)

type DBMock struct {
    mock.Mock
}

func (m *DBMock) GetUser(id int) string {
    args := m.Called(id)
    return args.String(0)
}

func GetUserName(id int, getUser func(int) string) string {
    return getUser(id)
}

func TestGetUserName(t *testing.T) {
    Convey("给定一个用户ID，应该返回对应的用户名", t, func() {
        db := new(DBMock)
        db.On("GetUser", 1).Return("MockUser")

        result := GetUserName(1, db.GetUser)

        So(result, ShouldEqual, "MockUser")

        // 验证 mock 调用是否正确
        db.AssertExpectations(t)
    })
}

```

Golang有以下Stub/Mock框架：

* GoStub

* GoMock

* Monkey

一般来说，GoConvey可以和GoStub、GoMock、Monkey中的一个或多个搭配使用。

### 6.3 GoStub

GoStub框架有多种使用场景：

* 基本场景：为全局变量打桩

* 基本场景：为函数打桩

* 基本场景：为过程打桩

* 复合场景：由多个基本场景组合而成

#### 6.3.1 为全局变量打桩

假设在被测函数中使用了一个全局整型变量count，当前测试用例需要将count的值固定为150：

```go

stubs := Stub(&count, 150)
defer stubs.Reset()
```

stubs是GoStub框架函数接口Stub返回的对象，该对象有Reset方法可以将全局变量恢复为原值。

#### 6.3.2 为函数打桩

设我们的代码中有以下函数定义：

```go
func Execute(cmd string, args ...string) (string, error) {
    // 实际实现...
}
```

我们可以对Execute函数打桩，代码如下：

```go
stubs := StubFunc(&Execute, "command-output", nil)
defer stubs.Reset()
```

#### 6.3.3 为过程打桩

当函数没有返回值时，我们通常称之为过程。例如，一个资源清理函数：

```go
func CleanupResources() {
    // 清理资源的代码...
}
```

我们对CleanupResources过程的打桩代码为：

```go
stubs := StubFunc(&CleanupResources)
defer stubs.Reset()
```

GoStub的更多用法可以参考[官方文档](https://github.com/prashantv/gostub)。

### 6.4 GoMock

GoMock是由Go官方开发维护的测试框架，提供了基于接口的Mock功能，能够与Go内置的testing包良好集成。GoMock包含两个主要部分：GoMock库和mockgen工具，其中GoMock库管理桩对象的生命周期，mockgen工具用于生成接口对应的Mock类源文件。

#### 6.4.1 定义接口

```go
package db

type DataStore interface {
    Create(key string, value []byte) error
    Retrieve(key string) ([]byte, error)
    Update(key string, value []byte) error
    Delete(key string) error
}
```

#### 6.4.2 生成Mock类文件

mockgen工具有两种操作模式：源文件模式和反射模式。

1. 源文件模式通过包含接口定义的文件生成Mock类：

```go
mockgen -source=datastore.go [其他选项]
```

* 反射模式通过构建程序并使用反射理解接口生成Mock类：

```go
mockgen database/sql/driver Conn,Driver
```

生成的mock\_datastore.go文件内容大致如下：

```go
// 自动生成的代码 - 请勿手动修改!
// Source: db (interfaces: DataStore)

package mock_db

import (
    gomock "github.com/golang/mock/gomock"
)

// MockDataStore 是DataStore接口的模拟实现
type MockDataStore struct {
    ctrl     *gomock.Controller
    recorder *MockDataStoreMockRecorder
}

// MockDataStoreMockRecorder 是MockDataStore的记录器
type MockDataStoreMockRecorder struct {
    mock *MockDataStore
}

// NewMockDataStore 创建一个新的模拟实例
func NewMockDataStore(ctrl *gomock.Controller) *MockDataStore {
    mock := &MockDataStore{ctrl: ctrl}
    mock.recorder = &MockDataStoreMockRecorder{mock}
    return mock
}

// EXPECT 返回一个对象，允许调用者指示预期的用法
func (_m *MockDataStore) EXPECT() *MockDataStoreMockRecorder {
    return _m.recorder
}

// Create 模拟基础方法
func (_m *MockDataStore) Create(_param0 string, _param1 []byte) error {
    ret := _m.ctrl.Call(_m, "Create", _param0, _param1)
    ret0, _ := ret[0].(error)
    return ret0
}
// ... 其他方法实现
```

#### 6.4.3 使用Mock对象进行测试

1. 导入相关包

```go
import (
    "testing"
    . "github.com/golang/mock/gomock"
    "myapp/mock/db"
    // 其他导入...
)
```

* 创建Mock控制器

Mock控制器通过NewController接口生成，是Mock生态系统的顶层控制，它定义了Mock对象的作用域和生命周期，以及期望行为。

```go
ctrl := NewController(t)
defer ctrl.Finish()
```

创建Mock对象时需要注入控制器：

```go
ctrl := NewController(t)
defer ctrl.Finish()
mockDB := mock_db.NewMockDataStore(ctrl)
mockAPI := mock_api.NewMockHttpClient(ctrl)
```

* 定义Mock对象行为

假设有这样一个场景：首先尝试获取数据失败，然后创建数据成功，再次获取就能成功。这个场景的Mock行为设置如下：

```sql
mockDB.EXPECT().Retrieve(Any()).Return(nil, errors.New("不存在"))
mockDB.EXPECT().Create(Any(), Any()).Return(nil)
mockDB.EXPECT().Retrieve(Any()).Return(dataBytes, nil)
```

其中dataBytes是测试数据的序列化结果：

```go
data := MyData{Field1: "value", Field2: 123}
dataBytes, _ := json.Marshal(data)
```

批量操作可以使用Times指定次数：

```go
mockDB.EXPECT().Create(Any(), Any()).Return(nil).Times(5)
```

多次获取不同数据时，需要设置多个行为：

```go
mockDB.EXPECT().Retrieve(Any()).Return(dataBytes1, nil)
mockDB.EXPECT().Retrieve(Any()).Return(dataBytes2, nil)
mockDB.EXPECT().Retrieve(Any()).Return(dataBytes3, nil)
```

GoMock的更多用法可以参考[官方文档](https://github.com/golang/mock)。

### 6.5 Monkey

前面我们已经了解到：

* 全局变量可通过GoStub框架打桩

* 过程可通过GoStub框架打桩

* 函数可通过GoStub框架打桩

* 接口可通过GoMock框架打桩

但还有两个问题较难解决：

1. 方法（成员函数）无法通过GoStub框架打桩，特别是当代码的OO设计较多时

2. 通过GoStub框架打桩时，对产品代码有侵入性

Monkey是Go的一个猴子补丁（monkeypatching）框架，通过在运行时重写可执行文件，将待打桩函数或方法的实现重定向到桩实现。原理类似于热补丁技术。但需要注意的是，Monkey不是线程安全的，不应用于并发测试。

Monkey框架的使用场景：

* 基本场景：为函数打桩

* 基本场景：为过程打桩

* 基本场景：为方法打桩

* 复合场景：由多个基本场景组合而成

* 特殊场景：桩中桩的案例

#### 6.5.1 为函数打桩

假设Execute是一个执行命令的函数：

```go
func Execute(cmd string, args ...string) (string, error) {
    cmdPath, err := exec.LookPath(cmd)
    if err != nil {
        log.Printf("exec.LookPath err: %v, cmd: %s", err, cmd)
        return "", errors.New("command not found")
    }

    output, err := exec.Command(cmdPath, args...).CombinedOutput()
    if err != nil {
        log.Printf("exec.Command.CombinedOutput err: %v, cmd: %s", err, cmd)
        return "", errors.New("command execution failed")
    }
    
    log.Printf("CMD[%s]ARGS[%v]OUT[%s]", cmdPath, args, string(output))
    return string(output), nil
}
```

使用Monkey打桩的代码：

```go
import (
    "testing"
    . "github.com/smartystreets/goconvey/convey"
    . "github.com/bouk/monkey"
    "myapp/utils"
)

func TestExecute(t *testing.T) {
    Convey("测试命令执行", t, func() {
        Convey("成功执行", func() {
            expectedOutput := "command output"
            guard := Patch(
                utils.Execute, 
                func(_ string, _ ...string) (string, error) {
                    return expectedOutput, nil
                })
            defer guard.Unpatch()
            
            output, err := utils.Execute("any", "any")
            So(output, ShouldEqual, expectedOutput)
            So(err, ShouldBeNil)
        })
    })
}
```

`Patch`是`Monkey`提供的函数打桩API：

1. 第一个参数是目标函数

2. 第二个参数是桩函数，通常使用匿名函数或闭包

3. 返回值是PatchGuard对象指针，用于在测试结束时移除补丁

#### 6.5.2 为过程打桩

对于没有返回值的函数（过程），打桩代码如下：

```go
guard := Patch(CleanupResources, func() {
    // 空实现或测试所需的行为
})
defer guard.Unpatch()
```

#### 6.5.3 为方法打桩

假设在分布式系统中，需要模拟从配置中心获取配置的行为：

```go
type ConfigCenter struct {
    // 字段...
}

func (c *ConfigCenter) GetConfig(key string) (string, error) {
    // 实际实现...
    return "", nil
}
```

使用Monkey对方法打桩：

```go
var cc *ConfigCenter
guard := PatchInstanceMethod(
    reflect.TypeOf(cc), 
    "GetConfig", 
    func(_ *ConfigCenter, _ string) (string, error) {
        return "{\"feature\":\"enabled\",\"timeout\":30}", nil
    })
defer guard.Unpatch()
```

PatchInstanceMethod API是Monkey提供的方法打桩API：

* 首先定义目标类的指针变量x

* 第一个参数是reflect.TypeOf(x)

* 第二个参数是方法名的字符串

* 第三个参数是替换方法

* 返回值是PatchGuard对象指针，用于移除补丁

Monkey的更多用法可以参考[官方文档](https://github.com/bouk/monkey)。

## 7. Mock场景最佳实践

### 7.1 实例函数Mock：Monkey。

Monkey框架可用于对依赖函数进行替换，完成针对当前模块的单元测试。

有如下例子，\`helper\`包是实际功能实现，\`mock\_helper\`包是用于mock的替代实现。

helper.go:

```go
package helper

import "fmt"

func FormatSum(a, b int) string {
    return fmt.Sprintf("a:%v+b:%v", a, b)
}

type Calculator struct {
}

func (*Calculator) FormatResult(a, b int) string {
    return fmt.Sprintf("a:%v+b:%v", a, b)
}
```

mock\_helper.go:

```go
package mock_helper

import (
    "fmt"
    "myapp/helper"
)

func FormatSum(a, b int) string {
    return fmt.Sprintf("a:%v+b:%v=%v", a, b, a+b)
}

// 对应helper包中的FormatResult
func FormatResult(_ *helper.Calculator, a, b int) string {
    return fmt.Sprintf("a:%v+b:%v=%v", a, b, a+b)
}
```

测试代码：

```go
func TestFormatting() {
    // 替换函数
    monkey.Patch(helper.FormatSum, mock_helper.FormatSum)
    result := helper.FormatSum(1, 2)
    fmt.Println(result)
    
    monkey.UnpatchAll() // 解除所有替换
    result = helper.FormatSum(1, 2)
    fmt.Println(result)
}

func TestMethodFormatting() {
    calc := &helper.Calculator{}
    // 参数1: 获取实例的反射类型, 参数2: 被替换的方法名, 参数3: 替换方法
    monkey.PatchInstanceMethod(reflect.TypeOf(calc), "FormatResult", mock_helper.FormatResult)
    
    result := calc.FormatResult(1, 2)
    fmt.Println(result)
    
    monkey.UnpatchAll() // 解除所有替换
    result = calc.FormatResult(1, 2)
    fmt.Println(result)
}
```

### 7.2 未实现函数Mock：GoMock

假设场景：`Company`（公司）和`Person`（人）之间的关系：

1. 公司可以举行会议

2. 公司内部的人实现了`Speaker`接口，拥有`SayHello`方法

若所有类都已实现，测试代码如下：

```go
func TestCompany_Meeting(t *testing.T) {
    // 直接创建一个Person对象
    speaker := NewPerson("小张", "工程师")
    company := NewCompany(speaker)
    t.Log(company.Meeting("张三", "实习生"))
}
```

但如果`Person`类尚未实现，可以通过GoMock模拟一个符合`Speaker`接口的对象

定义`Speaker.go`接口：

```go
package domain

type Speaker interface {
    SayHello(name, role string) (response string)
}
```

用`mockgen`命令生成Mock对象：

```go
mockgen -source=Speaker.go -destination=mock_speaker.go -package=mock_domain
```

测试代码：

```go
func TestCompany_Meeting(t *testing.T) {
    // 创建Mock控制器
    ctrl := gomock.NewController(t)
    // 创建Mock对象
    speaker := mock_domain.NewMockSpeaker(ctrl)

    // 设置期望行为
    speaker.EXPECT().SayHello(gomock.Eq("张三"), gomock.Eq("实习生")).Return(
        "你好，张三(角色：实习生)，欢迎加入公司会议。我是会议主持人。")

    // 将Mock对象传入测试对象
    company := NewCompany(speaker)

    // 执行测试
    t.Log(company.Meeting("张三", "实习生"))
}
```

### 7.3 系统内置函数Mock：Monkey

使用Monkey可以mock系统内置函数，例如json.Unmarshal：

```go
monkey.Patch(json.Unmarshal, mockUnmarshal)

func mockUnmarshal(b []byte, v interface{}) error {
    // 强制设置为指定值，无视输入
    *(v.(*models.LoginMessage)) = models.LoginMessage{
        UserID:   1,
        Username: "admin",
        Password: "admin",
    }
    return nil
}
```

取消替换：

```go
monkey.Unpatch(json.Unmarshal)    // 解除单个Patch
monkey.UnpatchAll()               // 解除所有Patch
```

### 7.4 数据库行为Mock

使用sqlmock库模拟数据库操作：

```go
func TestDatabaseQuery(t *testing.T) {
    db, mock, err := sqlmock.New(sqlmock.QueryMatcherOption(sqlmock.QueryMatcherEqual))
    if err != nil {
        t.Fatalf("创建sqlmock失败: %v", err)
    }
    defer db.Close()
    
    // 模拟查询结果
    rows := sqlmock.NewRows([]string{"id", "username"}).
        AddRow(1, "user1").
        AddRow(2, "user2")
    
    // 设置期望的SQL查询
    mock.ExpectQuery("SELECT id, username FROM users").WillReturnRows(rows)
    
    // 执行查询
    result, err := db.Query("SELECT id, username FROM users")
    if err != nil {
        t.Fatalf("查询执行失败: %v", err)
    }
    defer result.Close()
    
    // 处理结果
    var users []struct {
        ID       int
        Username string
    }
    
    for result.Next() {
        var id int
        var username string
        result.Scan(&id, &username)
        users = append(users, struct {
            ID       int
            Username string
        }{id, username})
        t.Logf("查询结果: ID=%d, 用户名=%s", id, username)
    }
    
    if result.Err() != nil {
        t.Fatalf("结果处理错误: %v", result.Err())
    }
    
    // 验证所有期望都已满足
    if err := mock.ExpectationsWereMet(); err != nil {
        t.Errorf("有未满足的期望: %v", err)
    }
}
```

### 7.5 服务器行为Mock

使用net/http/httptest模拟HTTP服务器：

```go
func TestHTTPRequest(t *testing.T) {
    // 创建处理器
    handler := func(w http.ResponseWriter, r *http.Request) {
        io.WriteString(w, `{"status": "success", "data": {"message": "Hello World"}}`)
    }

    // 创建请求和响应记录器
    req := httptest.NewRequest("GET", "/api/hello", nil)
    w := httptest.NewRecorder()
    
    // 处理请求
    handler(w, req)

    // 获取响应
    resp := w.Result()
    body, _ := ioutil.ReadAll(resp.Body)
    
    // 验证结果
    t.Logf("状态码: %d", resp.StatusCode)
    t.Logf("内容类型: %s", resp.Header.Get("Content-Type"))
    t.Logf("响应体: %s", string(body))
    
    // 可以进一步解析JSON并断言
    var result struct {
        Status string `json:"status"`
        Data   struct {
            Message string `json:"message"`
        } `json:"data"`
    }
    
    json.Unmarshal(body, &result)
    assert.Equal(t, "success", result.Status)
    assert.Equal(t, "Hello World", result.Data.Message)
}
```

对于涉及方法的情况，需要使用`PatchInstanceMethod`：

```go
func TestHTTPClient(t *testing.T) {
    var client *http.Client
    
    // 替换http.Client的Do方法
    monkey.PatchInstanceMethod(reflect.TypeOf(client), "Do", func(_ *http.Client, _ *http.Request) (*http.Response, error) {
        // 创建模拟响应
        resp := &http.Response{
            StatusCode: 200,
            Body:       ioutil.NopCloser(bytes.NewBufferString(`{"result": "mocked response"}`)),
            Header:     make(http.Header),
        }
        resp.Header.Set("Content-Type", "application/json")
        return resp, nil
    })
    defer monkey.UnpatchAll()
    
    // 测试使用http.Client的函数
    result, err := FetchData("https://api.example.com/data")
    assert.NoError(t, err)
    assert.Equal(t, "mocked response", result.Value)
}
```

## 8. 实战案例：消息通讯系统

### 8.1  项目概览

假设该项目是一个具有用户登录、查看在线用户、私聊、群聊等功能的命令行通讯系统。项目分为Client和Server两个子模块，都采用Model-Controller(Processor)-View(Main)的架构进行功能划分。另外还有一个Common模块存放通用工具类和数据结构。

```go
├─Client
│  ├─main
│  ├─model
│  ├─processor
│  └─utils
├─Common
└─Server
    ├─main
    ├─model
    ├─processor
    └─utils
```

测试目标：为核心功能模块编写单元测试，确保各模块功能的正确性、完整性和健壮性，并在代码变更后能快速验证。

单元测试应包括：

* 模块接口测试：验证参数传递、处理和返回值

* 模块数据结构测试：确保局部数据在处理过程中的完整性和正确性

* 异常处理测试：验证各种异常情况下的错误处理是否合理

接口测试应全面考察参数合法性、必要性、参数间的冗余性，以及指针引用的正确性等。数据结构测试应关注临时存储在模块内的数据结构的正确性，因为局部数据结构往往是错误的根源。

异常处理测试应关注几种常见问题：

1. 错误信息提示不足

2. 异常未被处理

3. 错误信息与实际不符

4. 错误信息未能准确定位问题

在本案例中，假设Model层向服务层提供的接口较少，只有`WritePkg`和`ReadPkg`两个核心函数，服务层基于这些基础函数封装具体业务逻辑。由于涉及网络连接，需要编写桩函数进行测试。服务层涉及多个网络连接调用和数据库操作，同样需要Mock。

鉴于需要编写Mock和桩函数，我们使用`GoStub`和`Monkey`包来简化测试，只需要编写替代接口和Mock函数，就能在测试过程中替换系统函数或依赖模块。

### 8.2 Model层与数据库测试

由于是单元测试，我们需要创建Mock数据库实例，测试CRUD操作的SQL语句执行：

```go
const (
    sqlSelect = "SELECT id, username FROM users"
    sqlDelete = "DELETE FROM users WHERE id > 100 AND id < 200"
    sqlUpdate = "UPDATE users SET status = 'active' WHERE id = 1"
    sqlInsert = "INSERT INTO users (id, username) VALUES (101, 'newuser')"
)

func TestUserRepository(t *testing.T) {
    // 创建sqlmock数据库连接
    db, mock, err := sqlmock.New(sqlmock.QueryMatcherOption(sqlmock.QueryMatcherEqual))
    if err != nil {
        t.Fatalf("创建sqlmock失败: %v", err)
    }
    defer db.Close()
    
    // 模拟查询结果
    rows1 := sqlmock.NewRows([]string{"id", "username"}).
        AddRow(1, "admin").
        AddRow(2, "user")
    rows2 := sqlmock.NewRows([]string{"id", "username"}).
        AddRow(101, "temp1").
        AddRow(102, "temp2")
    rows3 := sqlmock.NewRows([]string{"id", "username"}).
        AddRow(1, "admin")
    rows4 := sqlmock.NewRows([]string{"id", "username"}).
        AddRow(101, "newuser")

    // 设置SQL执行预期
    mock.ExpectQuery(sqlSelect).WillReturnRows(rows1)
    mock.ExpectQuery(sqlDelete).WillReturnRows(rows2)
    mock.ExpectQuery(sqlUpdate).WillReturnRows(rows3)
    mock.ExpectQuery(sqlInsert).WillReturnRows(rows4)

    // 测试用例
    var tests = []struct{
        querySql string
        expected interface{}
    }{
        {sqlSelect, nil},
        {sqlDelete, nil},
        {sqlUpdate, nil},
        {sqlInsert, nil},
    }

    for _, test := range tests {
        // 执行查询
        res, err := db.Query(test.querySql)
        assert.Equal(t, err, test.expected) // 验证无错误
        
        // 处理结果
        var users []struct {
            ID       int
            Username string
        }
        
        for res.Next() {
            var id int
            var username string
            res.Scan(&id, &username)
            users = append(users, struct {
                ID       int
                Username string
            }{id, username})
            t.Logf("查询结果: ID=%d, 用户名=%s", id, username)
        }
        
        assert.Equal(t, res.Err(), test.expected) // 验证结果处理无错误
    }
}
```

### 8.3 私聊功能测试

私聊功能涉及JSON编码和发送消息的底层操作（`WritePkg`函数），我们使用Monkey进行Mock：

```go
func TestMessageSender_SendPrivateMessage(t *testing.T) {
    var conn net.Conn
    transfer := &utils.Transfer{
        Conn: conn,
    }
    
    // Mock WritePkg方法
    monkey.PatchInstanceMethod(reflect.TypeOf(transfer), "WritePkg", func(_ *utils.Transfer, _ []byte) error {
        return nil
    })
    
    convey.Convey("测试发送私聊消息", t, func() {
        msg := &models.PrivateMessage{
            From:    "user1",
            To:      "user2",
            Content: "你好！",
        }
        
        sender := &MessageSender{Transfer: transfer}
        err := sender.SendPrivateMessage(msg)
        
        convey.So(err, convey.ShouldBeNil)
    })
    
    monkey.UnpatchAll()
}
```

### 8.4  登录功能测试

登录功能涉及服务器连接和数据处理，我们可以使用多种Mock技术结合测试：

```go
func TestMessageSender_SendPrivateMessage(t *testing.T) {
    var conn net.Conn
    transfer := &utils.Transfer{
        Conn: conn,
    }
    
    // Mock WritePkg方法
    monkey.PatchInstanceMethod(reflect.TypeOf(transfer), "WritePkg", func(_ *utils.Transfer, _ []byte) error {
        return nil
    })
    
    convey.Convey("测试发送私聊消息", t, func() {
        msg := &models.PrivateMessage{
            From:    "user1",
            To:      "user2",
            Content: "你好！",
        }
        
        sender := &MessageSender{Transfer: transfer}
        err := sender.SendPrivateMessage(msg)
        
        convey.So(err, convey.ShouldBeNil)
    })
    
    monkey.UnpatchAll()
}
```

### 4. 登录功能测试

登录功能涉及服务器连接和数据处理，我们可以使用多种Mock技术结合测试：

```go
func mockJsonUnmarshal(b []byte, v interface{}) error {
    // 强制设置登录消息对象的值
    *(v.(*models.LoginMessage)) = models.LoginMessage{
        UserID:   1,
        Username: "admin",
        Password: "password123",
    }
    return nil
}

func mockJsonMarshal(v interface{}) ([]byte, error) {
    // 简化的JSON序列化，返回固定内容
    return []byte(`{"status":"success"}`), nil
}

func TestUserProcessor_Login(t *testing.T) {
    // 创建测试消息
    message := &models.Message{
        Type: models.LoginMessageType,
        Data: "mock_login_data",
    }
    
    userProcessor := &UserProcessor{
        Conn: nil,
    }
    
    // Mock系统函数
    monkey.Patch(json.Unmarshal, mockJsonUnmarshal)
    monkey.Patch(json.Marshal, mockJsonMarshal)
    
    // Mock用户数据访问对象
    var userDao *model.UserDao
    monkey.PatchInstanceMethod(reflect.TypeOf(userDao), "Login", func(_ *model.UserDao, _ int, _ string) (*models.User, error) {
        return &models.User{
            UserID:   1,
            Username: "admin",
            Password: "password123",
        }, nil
    })
    
    // Mock传输层
    var transfer *utils.Transfer
    monkey.PatchInstanceMethod(reflect.TypeOf(transfer), "WritePkg", func(_ *utils.Transfer, _ []byte) error {
        return nil
    })
    
    // 执行测试
    convey.Convey("测试用户登录处理", t, func() {
        err := userProcessor.HandleLogin(message)
        convey.So(err, convey.ShouldBeNil)
    })
    
    // 清理Mock
    monkey.UnpatchAll()
}
```

## 8.5 工具类测试

测试网络传输工具类：

```go
func mockNetRead(conn net.Conn, _ []byte) (int, error) {
    // 模拟读取4字节数据
    return 4, nil
}

func mockJsonMarshal(v interface{}) ([]byte, error) {
    return []byte{1, 2, 3, 4}, nil
}

func mockJsonUnmarshal(data []byte, v interface{}) error {
    return nil
}

func TestTransfer_ReadPackage(t *testing.T) {
    // Mock网络读取
    monkey.Patch(net.Conn.Read, mockNetRead)
    monkey.Patch(json.Marshal, mockJsonMarshal)
    monkey.Patch(json.Unmarshal, mockJsonUnmarshal)
    
    // 创建测试服务器
    listener, _ := net.Listen("tcp", "localhost:9999")
    defer listener.Close()
    
    // 创建客户端连接
    go net.Dial("tcp", "localhost:9999")
    
    // 接受连接
    var conn net.Conn
    for {
        conn, _ = listener.Accept()
        if conn != nil {
            break
        }
    }
    
    // 创建测试对象
    transfer := &Transfer{
        Conn: conn,
        Buf:  [8096]byte{1, 2, 3, 4},
    }
    
    // 执行测试
    convey.Convey("测试数据包读取", t, func() {
        message, err := transfer.ReadPackage()
        convey.So(err, convey.ShouldBeNil)
        convey.So(message, convey.ShouldNotBeNil)
    })
    
    // 清理Mock
    monkey.UnpatchAll()
}

func TestTransfer_WritePackage(t *testing.T) {
    // Mock JSON操作
    monkey.Patch(json.Marshal, mockJsonMarshal)
    monkey.Patch(json.Unmarshal, mockJsonUnmarshal)
    
    // 创建测试对象
    transfer := &Transfer{
        Conn: nil,
        Buf:  [8096]byte{},
    }
    
    // 执行测试
    convey.Convey("测试数据包写入", t, func() {
        err := transfer.WritePackage([]byte{1, 2})
        convey.So(err, convey.ShouldBeNil)
    })
    
    // 清理Mock
    monkey.UnpatchAll()
}
```

在编写单元测试的时候，推荐使用第三方包来完成，虽然原生包能满足基本需求，但不提供断言语法，导致要写大量重复的错误检查代码，因此引入convey和assert包简化判断逻辑，可以使代码更简洁易读

更多测试实践案例可参考：

* [go-sqlmock](https://github.com/DATA-DOG/go-sqlmock)

* [GoMock实践指南](https://github.com/golang/mock)

## 9. 基准测试

除了前面提到的单元测试，测试代码单元的正确性之外，Go语言还提供了基准测试框架，可以测试一段程序的性能、CPU消耗，可以对代码做性能分析，测试方法与单元测试类似。

基准测试规则：

* 基准测试以Benchmark为前缀

* 需要一个\*testing.B类型的参数b

* 基准测试必须要执行b.N次

常见的基准测试函数写法：

```go
func BenchmarkTest(b *testing.B) {
        ...
}
```

执行基准测试时，需要添加`-bench`参数

```go
go test -bench="."
```

下面通过一个模拟负载均衡的例子，来看下基准测试：

在gotest包下准备一个`Abs` 函数作为被测试的代码位于example.go，代码如下：

```go
package gotest

import "math"

func Abs(x float64) float64 {
        return math.Abs(x)
}
```

然后在example\_test.go文件中为 `Abs` 函数编写的基准测试，代码如下：

```go
package gotest

func BenchmarkAbs(b *testing.B) {
        for i := 0; i < b.N; i++ {
                Abs(-1)
        }
}
```

注意基准测试的时候参数不再是 `*testing.T`，而是 `*testing.B`，在测试函数中，我们循环了 `b.N` 次调用 `Abs(-1)`，`b.N` 的值是一个动态值，我们无需操心，`testing` 框架会为其分配合理的值，以使测试函数运行足够多的次数，可以准确的计时。

默认情况下，执行 `go test` 命令时不会自动运行基准测试，需要显式指定 `-bench` 参数

```go
➜  gotest go test -bench="."
...
3 total assertions

goos: darwin
goarch: arm64
pkg: hello1/gotest
BenchmarkAbs-12         1000000000               0.2954 ns/op
PASS
ok      hello1/gotest   1.513s

```

`-bench` 的参数接收一个正则表达式，`.` 匹配所有基准测试。重点看一下执行结果的这一行

```go
BenchmarkAbs-12         1000000000               0.2954 ns/op
```

`BenchmarkAbs-`12 中，`BenchmarkAbs` 是测试函数名，12 是 `GOMAXPROCS` 的值，即参与执行的 CPU 核心数。`1000000000` 表示测试执行了这么多次。`0.5096 ns/op` 表示每次循环平均消耗的纳秒数。

如果想查看基准测试的内存占用情况，可以通过 `-benchmem` 参数指定：

```bash
➜  gotest go test -bench="BenchmarkAbs$" -benchmem 
...
3 total assertions

goos: darwin
goarch: arm64
pkg: hello1/gotest
BenchmarkAbs-12         1000000000               0.2932 ns/op          0 B/op          0 allocs/op
PASS
ok      hello1/gotest   0.683s

```

可以发现，加上`-benchmem` 参数后，`BenchmarkAbs-8` 这行打印了更多输出内容：

```go
BenchmarkAbs-12         1000000000               0.2932 ns/op          0 B/op 
```

`0 B/op` 表示每次执行测试代码分配了多少字节内存。`0 allocs/op` 表示每次执行测试代码分配了多少次内存。

此外，在执行 `go test` 命令时，我们可以使用 `-benchtime=Ns` 参数指定基准测试函数执行时间为 `N` 秒：

```go
➜  gotest go test -bench="BenchmarkAbs$" -benchtime=0.1s
...
3 total assertions

goos: darwin
goarch: arm64
pkg: hello1/gotest
BenchmarkAbs-12         385589265                0.3081 ns/op
PASS
ok      hello1/gotest   0.686s
```

`-benchtime` 参数值为 `time.Duration` 类型支持的时间格式。此外，`-benchtime` 参数还有一个特殊语法 `-benchtime=Nx` 参数，可以指定基准测试函数执行次数为 `N` 次：

```go
➜  gotest go test -bench="BenchmarkAbs$" -benchtime=10x
...
3 total assertions

goos: darwin
goarch: arm64
pkg: hello1/gotest
BenchmarkAbs-12               10                25.00 ns/op
PASS
ok      hello1/gotest   0.466s
```

有时在进行基准测试时，目标函数可能依赖一些预处理步骤，比如数据准备，这些数据准备的时间不应被计入函数本身的性能统计。这时候，我们可以调用 `(*testing.B).ResetTimer` 来重新开始计时，从而确保测试只衡量核心逻辑的执行时间。

```go
func BenchmarkAbsResetTimer(b *testing.B) {
        time.Sleep(100 * time.Millisecond) // 模拟数据准备阶段的耗时
        b.ResetTimer()
        for i := 0; i < b.N; i++ {
                Abs(-1)
        }
}
```

这样，在调用 `b.ResetTimer()` 之前进行的耗时操作将不会被纳入最终的基准测试时间统计中。

另外，还有一种更灵活的做法是：先调用 `b.StopTimer()` 来暂停计时，等准备工作完成后，再通过 `b.StartTimer()` 恢复计时，这样也能避免将准备过程的耗时计算在内。

```go
func BenchmarkAbsStopTimerStartTimer(b *testing.B) {
        b.StopTimer()
        time.Sleep(100 * time.Millisecond) // 模拟数据准备阶段的耗时
        b.StartTimer()
        for i := 0; i < b.N; i++ {
                Abs(-1)
        }
}
```

默认情况下，基准测试中的 `for` 循环是串行方式执行的。如果想要对被测代码进行并发性能测试，可以将其封装在 `(*testing.B).RunParallel` 方法中，实现并行调用

```go
func BenchmarkAbsParallel(b *testing.B) {
        b.RunParallel(func(pb *testing.PB) {
                for pb.Next() {
                        Abs(-1)
                }
        })
}
```

还可以使用 `(*testing.B).SetParallelism` 控制并发协程数：

```go
func BenchmarkAbsParallel(b *testing.B) {
        b.SetParallelism(2) // 设置并发 Goroutines 数量为 2 * GOMAXPROCS
        b.RunParallel(func(pb *testing.PB) {
                for pb.Next() {
                        Abs(-1)
                }
        })
}
```

可以通过 `-cpu` 参数为 `go test` 指定 GOMAXPROCS 的值，用来控制使用的 CPU 核心数量。如果想了解更多 `go test` 支持的参数选项，可以执行命令 `go help testflag` 来获取完整的帮助信息。

## 10. 小结

单元测试（Unit Test，简称 UT）是高质量软件项目中不可缺少的一个组成部分。它的核心目标是对程序中**最小的功能单位**进行验证，通常是一个函数或者方法，确保其行为符合预期。Go语言对单元测试提供了很好的支持，其自身就带有一个轻量级的测试框架testing，可以用自带的go test命令来实现单元测试和性能测试。同时也有非常多好用的第三方测试包，比如GoConvey，testify等，可以更加简洁的写测试用例。写好Go程序的单测，不仅仅可以确保代码的完整性和正确性，也是一个Gopher基本功的重要体现

