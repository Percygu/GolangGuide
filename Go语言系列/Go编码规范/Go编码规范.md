---
tags:
  - Go
  - golang
  - go编码规范
  - 程序格式化
---

# Go编码规范

## 1.代码规范

代码范旨在为日常Go项目开发提供一个代码的规范指导，方便形成一个良好的代码，提高代码的可读性，规范性和统一性。

## 2. 代码风格

### 2.1 【必须】格式化

* 代码都必须用 `gofmt` 格式化。

### 2.2 【推荐】换行

* 建议一行代码不要超过`120列`，超过的情况，使用合理的换行方法换行。

* 例外场景：

  * 函数签名（当然，这可能意味着你需要重新考虑是否传递了过多参数）

  ```go
  // 长的函数签名可以超过列数限制
  func (i *webImpl) GenerateAgentInstallLink(ctx context.Context, req *pb.GenerateAgentInstallLinkRequest) (*pb.GenerateAgentInstallLinkResponse, error) {    
      ...
  }

  // 不要在函数签名中为了满足推荐列数换行！
  func (i *webImpl) GenerateAgentInstallLink(ctx context.Context,
      req *pb.GenerateAgentInstallLinkRequest) (*pb.GenerateAgentInstallLinkResponse, error) {   
      ... // gofmt 会使签名与函数内语句对齐，导致代码可读性降低
  }
  ```

  * 长字符串文字（如果存在换行符 `\n`，应考虑使用原始字符串字面量 `` `raw string literal` ``）

  ```go
  // 长的字符串可以超过列数限制
  pubkey := "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDPqE4yo8w7lezTpCkTt40whHoubL+0Qhsq1L3hF/lDW8zx1NpFTR9fC1HOqmUXj7tqPVg2xOAH+..."

  // 不要为了满足推荐列数换行并拼接字符串！
  pubkey := "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDPqE4yo8w7lezTpCkTt40whHoubL+0Qhsq1L3hF/lDW8zx1NpFTR9fC1HOqmUXj7tqPVg2xOAH+"+
  "zi2SqaZVeeXmsF5GAGFJcUylujr78Wf6od8//SApYx8RCSkRhGo8cTsxADlBCoTttJvk6Ocmy+uqEFXulsI0j+nh2x352eCExlDSqr0Me0J0LIGq/u9eqwhNN5k"+
  ... // 按列换行不会使代码更加可读，但会使字符串的搜索和修改更加困难！

  // 长的多行文本可以考虑使用原始字符串字面量
  tmpl := `some
  long
  tedious
  template`
  ```

  import 模块语句

  * 工具生成代码

  * struct tag

### 2.3 【必须】括号和空格

* 遵循 `gofmt` 的逻辑。

* 运算符和操作数之间要留空格。

* 作为输入参数或者数组下标时，运算符和运算数之间不需要空格，紧凑展示。

### 2.4 【必须】import 规范

* 使用 `goimports` 自动格式化引入的包名，import 规范原则上以 `goimports` 规则为准。

* `goimports` 会自动把依赖包按首字母排序，并对包进行分组管理，通过空行隔开，默认分为本地包（标准库、内部包）、第三方包。

* 标准包永远位于最上面的第一组。

* 内部包是指不能被外部 import 的包，如 GoPath 模式下的包名或者非域名开头的当前项目的 GoModules 包名。

* 带域名的包名都属于第三方包，如 github.com/xxx/xxx，不用区分是否是当前项目内部的包。

* `goimports` 默认最少分成本地包和第三方包两大类，这两类包必须分开不能放在一起。本地包或者第三方包内部可以继续按实际情况细分不同子类。

* 不要使用相对路径引入包：

```go
// 不要采用这种方式
import (    
    "../net"
)
```

* 应该使用完整的路径引入包：

```plaintext
import (    
    "xxxx.com/proj/net"
)
```

* 包名和 git 路径名不一致时，或者多个相同包名冲突时，使用别名代替，别名命名规范和包命名规范保持一致：

```go
// 合理用法：包名和 git 路径名不一致，使用别名
import (    
    opentracing "github.com/opentracing/opentracing-go"
)

// 合理用法：多个相同包名冲突，使用别名
 import (    
     "fmt"    "os"    "runtime/trace"    
     nettrace "golang.net/x/trace"
 )
 
 // 不合理用法：包名和路径名一致，也不存在多包名冲突，并且原包名符合规范，则不应该使用别名
 import (    
     "fmt"    
     "os" 
        
     nettrace "golang.net/x/trace"
 )
```

* 【可选】第三方包的包名不符合规范可使用别名修正：

```plaintext
import (    
    qzone "git.code.oa.com/trpcprotocol/qqconnect/share_to_qzone"
)
```

* 【可选】匿名包的引用建议使用一个新的分组引入，并在匿名包上写上注释说明。

完整示例如下：

```go
import (    
    // standard package & inner package    
    "encoding/json"    
    "myproject/models"    
    "myproject/controller"    
    "strings"    
    
    // third-party package    
    "git.obc.im/obc/utils"    
    "git.obc.im/dep/beego"    
    "git.obc.im/dep/mysql"    
    opentracing "github.com/opentracing/opentracing-go"    
    
    // anonymous import package    
    // import filesystem storage driver    
    _ "git.code.oa.com/org/repo/pkg/storage/filesystem"
)
```

* 例外包

  * embed 当使用embed内嵌外部数据时不需要注释

### 2.5 【必须】错误处理

阅读 <https://go.dev/doc/effective_go#errors> 学习 go 错误处理机制的设计。

#### 2.5.1 【必须】error 处理

* `error` 作为函数的值返回，必须对 `error` 进行处理, 或将返回值赋值给明确忽略。对于 `defer xx.Close()`可以不用显式处理。

* `error` 作为函数的值返回且有多个返回值的时候，`error` 必须是最后一个参数。

```go
// 不要采用这种方式
func do() (error, int) {

}

// 要采用下面的方式
func do() (int, error) {

}
```

* 错误描述不需要标点结尾。

* 采用独立的错误流进行处理。

```go
// 不要采用这种方式
if err != nil {    
    // error handling
} else {    
    // normal code
}
    
// 而要采用下面的方式
if err != nil {    
    // error handling    
    return // or continue, etc.
 }
 // normal code
```

* 如果返回值需要初始化，则采用下面的方式：

```go
x, err := f()
if err != nil {    
    // error handling    
    return 
    // or continue, etc.
}
// use x
```

* 错误返回的判断独立处理，不与其他变量组合逻辑判断。

```go
// 不要采用这种方式：
x, y, err := f()
if err != nil || y == nil {    
    return err   // 当y与err都为空时，函数的调用者会出现错误的调用逻辑
}

// 应当使用如下方式：
x, y, err := f()
if err != nil {    
    return err
}

if y == nil {    
    return errors.New("some error")
}
```

* 【推荐】对于不需要格式化的错误，生成方式为：`errors.New("xxxx")`。

* 【推荐】建议go1.13 以上，error 生成方式为：`fmt.Errorf("module xxx: %w", err)`。

#### 2.5.2 【必须】panic 处理

* 不要使用 `panic` 进行一般的错误处理，使用 `error` 和多返回值。

* 可使用 panic 对不变量（[invariant](https://en.wikipedia.org/wiki/Invariant_%28mathematics%29#Invariants_in_computer_science)）进行断言。

```plaintext
// 不要对用户输入进行断言，使用错误返回！
v, err := strconv.Atoi(userInputFromKeyboard)
if err != nil {   
    panic(fmt.Errorf("invalid user input: %v", err))
}

// 可以对不变量进行断言// readText 读取 n 中的文本内容。Node 必须为 TextNode 或 CommentNode。
func readText(n Node) string {    
    switch n := n.(type) {    
    case *TextNode:        
        return n.Text    
    case *CommentNode:        
        return n.Comment    
    default:        
        panic(fmt.Errorf("unexpected node type: %T", n))    
    }
}
```

* 在 `func init()` 及其调用的函数中，当初始化失败影响程序运行时，可以 `panic`，例如：读取本地文件系统中的配置文件。

* 在全局变量初始时调用的函数中，当初始化失败影响程序运行时，可以 `panic`，例如：`regexp.MustCompile`，`template.Must`。

* 导出的方法一般不允许 `panic`。因特殊情况必须 `panic` 的方法应使用 `MustXXX` 的方式进行命名，并在文档中加以说明。

* 不建议使用 `log.Fatal` 进行断言：log 库必须初始化才可以使用；log 库可能无法正确显示堆栈信息；log 库在 `panic` 时可能无法有效的保留错误信息。

#### 2.5.3 【必须】recover 处理

* 必须在 `defer` 中使用。

* 在业务逻辑中一般不需要使用 `recover`。

* `recover` 用于捕获具有明确类型的 `panic`，禁止滥用 `recover` 捕获全部类型的异常。

```go
// FatalError 为包内定义的错误类型
type FatalError string

func (e FatalError) Error() string {
    return string(e) 
}

func main() {    
    defer func() {        
        e := recover() // 注意，recover 返回的是 interface{}，不要假设其是 error，因此不要将返回值命名为 err。        
        if e != nil {            
            err, ok := e.(FatalError)            
            if !ok {                
                // 继续抛出不认识的异常                
                panic(e)            
            }            
            // 响应抛出的错误        
         }    
     }()    
     panic(FatalError("错误信息"))
}
```

### 2.6 【必须】单元测试

* 单元测试文件名命名规范为 `example_test.go`。

* 测试用例的函数名称必须以 `Test` 开头，例如 `TestExample`。

* 如果存在 `func Foo`，单测函数可以带下划线，为 `func Test_Foo`。如果存在 `func (b *Bar) Foo`，单测函数可以为 `func TestBar_Foo`。下划线不能出现在前面描述情况以外的位置。

* 单测文件行数限制是普通文件的2倍，即`1600行`。单测函数行数限制也是普通函数的2倍，即为`160行`。圈复杂度、列数限制、 import 分组等其他规范细节和普通文件保持一致。

* 由于单测文件内的函数都是不对外的，所有可导出函数可以没有注释，但是结构体定义时尽量不要导出。

* 每个重要的可导出函数都要首先编写测试用例，测试用例和正规代码一起提交方便进行回归测试。

## 3. 注释

* 在编码阶段同步写好变量、函数、包注释，注释可以通过 `godoc` 导出生成文档。

* 程序中每一个被导出的（大写的）名字，都应该有一个文档注释。注意，非导出类型的方法（`func (*privateType) Method()`）可以没有文档注释。

* 所有注释掉的代码在提交 code review 前都应该被删除，除非添加注释讲解为什么不删除， 并且标明后续处理建议（比如删除计划）。

### 3.1 【必须】包注释

* 每个包都应该有一个包注释。

* 包如果有多个 go 文件，只需要出现在一个 go 文件中（一般是和包同名的文件）即可，格式为："// Package 包名 包信息描述"。

```go
// Package math provides basic constants and mathematical functions.
package math

// 或者
/*Package template implements data-driven templates for generating textual
output such as HTML.
....
*/
package template
```

### 3.2 【必须】结构体注释

* 每个需要导出的自定义结构体或者接口都必须有注释说明。

* 注释对结构进行简要介绍，放在结构体定义的前一行。

* 格式为："// 结构体名 结构体信息描述"。

* 结构体内的可导出成员变量名，如果是个生僻词，或者意义不明确的词，就必须要给出注释，放在成员变量的前一行或同一行的末尾。

```go
// User 用户结构定义了用户基础信息
type User struct {    
    Name  string    
    Email string    
    // Demographic 族群    
    Demographic string
}
```

## 3.3 【必须】方法注释

* 每个需要导出的函数或者方法（结构体或者接口下的函数称为方法）都必须有注释。注意，如果方法的接收器为不可导出类型，可以不注释，但需要质疑该方法可导出的必要性。

* 注释描述函数或方法功能、调用方等信息。

* 格式为："// 函数名 函数信息描述"。

```go
// NewtAttrModel 是属性数据层操作类的工厂方法
func NewAttrModel(ctx *common.Context) *AttrModel {    
    // TODO
}
```

* 例外方法：

  * Write Read 用于常见IO

  * ServeHTTP 用于HTTP服务

  * String 用于打印

  * Unwrap Error 用于错误处理

  * Len Less Swap 用于排序

## 3.4 【必须】变量和常量注释

* 每个需要导出的常量和变量都必须有注释说明。

* 该注释对常量或变量进行简要介绍，放在常量或者变量定义的前一行。

* 大块常量或变量定义时，可在前面注释一个总的说明，然后每一行常量的末尾详细注释该常量的定义。

* 格式为："// 变量名 变量信息描述"，斜线后面紧跟一个空格。

```go
// FlagConfigFile 配置文件的命令行参数名
const FlagConfigFile = "--config"

// 命令行参数
const (    
    FlagConfigFile1 = "--config" // 配置文件的命令行参数名1    
    FlagConfigFile2 = "--config" // 配置文件的命令行参数名2    
    FlagConfigFile3 = "--config" // 配置文件的命令行参数名3    
    FlagConfigFile4 = "--config" // 配置文件的命令行参数名4
)
    
// FullName 返回指定用户名的完整名称
var FullName = func(username string) string {    
    return fmt.Sprintf("fake-%s", username)
}
```

## 3.5 【必须】类型注释

* 每个需要导出的类型定义（type definition）和类型别名（type aliases）都必须有注释说明。

* 该注释对类型进行简要介绍，放在定义的前一行。

* 格式为："// 类型名 类型信息描述"。

```go
// StorageClass 存储类型
type StorageClass string

// FakeTime 标准库时间的类型别名
type FakeTime = time.Time
```

# 4. 命名规范

命名是代码规范中很重要的一部分，统一的命名规范有利于提高代码的可读性，好的命名仅仅通过命名就可以获取到足够多的信息。

## 4.1 【推荐】包命名

* 保持 `package` 的名字和目录一致。

* 尽量采取有意义、简短的包名，尽量不要和标准库冲突。

* 包名应该为小写单词，不要使用下划线或者混合大小写，使用多级目录来划分层级。

* 包名可谨慎地使用缩写。当缩写是程序员广泛熟知的词时，可以使用缩写。例如：

  * strconv (string conversion)

  * syscall (system call)

  * fmt (formatted I/O)

* 如果缩写有歧义或不清晰，不用缩写。

* 项目名可以通过中划线来连接多个单词。

* 简单明了的包命名，如：`time`、`list`、`http`。

* 不要使用无意义的包名，如：`util`、`common`、`misc`、`global`。`package`名字应该追求清晰且越来越收敛，符合‘单一职责’原则。而不是像`common`一样，什么都能往里面放，越来越膨胀，让依赖关系变得复杂，不利于阅读、复用、重构。注意，`xx/util/encryption`这样的包名是允许的。

## 4.2 【必须】文件命名

* 采用有意义，简短的文件名。

* 文件名应该采用小写，并且使用下划线分割各个单词。

## 4.3 【必须】结构体命名

* 采用驼峰命名方式，首字母根据访问控制采用大写或者小写。

* 结构体名应该是名词或名词短语，如 `Customer`、`WikiPage`、`Account`、`AddressParser`，它不应是动词。

* 避免使用 `Data`、`Info` 这类意义太宽泛的结构体名。

* 结构体的声明和初始化格式采用多行，例如：

```go
// User 多行声明
type User struct {    
    Name  string    
    Email string
}

// 多行初始化
u := User{    
    Name:  "john",    
    Email: "john@example.com",
}
```

## 4.4 【推荐】接口命名

* 命名规则基本保持和结构体命名规则一致。

* 单个函数的接口名以 `er` 作为后缀，例如 `Reader`，`Writer`。

```go
// Reader 字节数组读取接口
type Reader interface {    
    // Read 读取整个给定的字节数据并返回读取的长度    
    Read(p []byte) (n int, err error)
}
```

* 两个函数的接口名综合两个函数名。

* 三个以上函数的接口名，类似于结构体名。

```go
// Car 小汽车结构申明
type Car interface {    
    // Start ...    
    Start([]byte)    
    // Stop ...    
    Stop() error    
    // Recover ...    
    Recover()
}
```

## 4.5 【必须】变量命名

* 变量名必须遵循驼峰式，首字母根据访问控制决定使用大写或小写。

* 特有名词时，需要遵循以下规则：

  * 如果变量为私有，且特有名词为首个单词，则使用小写，如 `apiClient`；

  * 其他情况都应该使用该名词原有的写法，如 `APIClient`、`repoID`、`UserID`；

  * 错误示例：`UrlArray`，应该写成 `urlArray` 或者 `URLArray`；

  * 详细的专有名词列表可参考[这里](https://github.com/golang/lint/blob/738671d3881b9731cc63024d5d88cf28db875626/lint.go#L770)。

* 私有全局变量和局部变量规范一致，均以小写字母开头。

* 代码生成工具自动生成的代码可排除此规则（如 xxx.pb.go 里面的 Id）。

* 变量名更倾向于选择短命名。特别是对于局部变量。 `c`比`lineCount`要好，`i`比`sliceIndex`要好。基本原则是：变量的使用和声明的位置越远，变量名就需要具备越强的描述性。

## 4.6 【必须】常量命名

* 常量均需遵循驼峰式。

```go
// AppVersion 应用程序版本号定义
const AppVersion = "1.0.0"
```

* 如果是枚举类型的常量，需要先创建相应类型：

```go
// Scheme 传输协议
type Scheme stringconst (    
    // HTTP 表示HTTP明文传输协议    
    HTTP Scheme = "http"    
    // HTTPS 表示HTTPS加密传输协议    
    HTTPS Scheme = "https"
)
```

* 私有全局常量和局部变量规范一致，均以小写字母开头。

```go
const appVersion = "1.0.0"
```

## 4.7 【必须】函数命名

* 函数名必须遵循驼峰式，首字母根据访问控制决定使用大写或小写。

* 代码生成工具自动生成的代码可排除此规则（如协议生成文件 xxx.pb.go , gotests 自动生成文件 xxx\_test.go 里面的下划线）。

# 5. 控制结构

## 5.1 【推荐】if

* `if` 接受初始化语句，约定如下方式建立局部变量：

```go
if err := file.Chmod(0664); err != nil {    
    return err
}
```

* `if` 对两个值进行判断时，约定如下顺序：变量在左，常量在右：

```go
// 不要采用这种方式
if nil != err {    
    // error handling
}

// 不要采用这种方式
if 0 == errorCode {    
    // do something
}

// 而要采用下面的方式
if err != nil {    
    // error handling
}

// 而要采用下面的方式
if errorCode == 0 {    
    // do something
}
```

* `if` 对于bool类型的变量，应直接进行真假判断：

```go
var allowUserLogin bool

// 不要采用这种方式
if allowUserLogin == true {    
    // do something
}

// 不要采用这种方式
if allowUserLogin == false {    
    // do something
}

// 而要采用下面的方式
if allowUserLogin {    
    // do something
}

// 而要采用下面的方式
if !allowUserLogin {    
    // do something
}
```

## 5.2 【推荐】for

* 采用短声明建立局部变量：

```go
sum := 0
for i := 0; i < 10; i++ {    
    sum += 1
}
```

## 5.3 【必须】range

* 如果只需要第一项（key），就丢弃第二个：

```go
for key := range m {    
    if key.expired() {        
        delete(m, key)    
    }
}
```

* 如果只需要第二项，则把第一项置为下划线：

```go
sum := 0
for _, value := range array {    
    sum += value
}
```

## 5.4 【必须】switch

* 要求必须有 `default`：

```go
switch os := runtime.GOOS; os {    
    case "darwin":        
        fmt.Println("OS X.")    
    case "linux":        
        fmt.Println("Linux.")    
    default:        
        // freebsd, openbsd,        
        // plan9, windows...        
        fmt.Printf("%s.\n", os)
}
```

## 5.5 【推荐】return

* 尽早 `return`，一旦有错误发生，马上返回：

```go
f, err := os.Open(name)
if err != nil {    
    return err
}
defer f.Close()

d, err := f.Stat()
if err != nil {    
    return err
}

codeUsing(f, d)
```

## 5.6 【必须】goto

* 业务代码禁止使用 `goto`，其他框架或底层源码推荐尽量不用。

# 6. 函数

## 6.1 【推荐】函数参数

* 函数返回相同类型的两个或三个参数，或者如果从上下文中不清楚结果的含义，使用命名返回，其它情况不建议使用命名返回。

```go
// Parent1 ...
func (n *Node) Parent1() *Node

// Parent2 ...
func (n *Node) Parent2() (*Node, error)

// Location ...
func (f *Foo) Location() (lat, long float64, err error)
```

* 传入变量和返回变量以小写字母开头。

* 参数数量均不能超过`5个`。

* 尽量用值传递，非指针传递。

* 传入参数是 `map`，`slice`，`chan`，`interface` 不要传递指针。

## 6.2 【必须】defer

* 当存在资源管理时，应紧跟 `defer` 函数进行资源的释放。

* 判断是否有错误发生之后，再 `defer` 释放资源。

```go
resp, err := http.Get(url)
if err != nil {    
    return err
}
// 如果操作成功，再defer Close()
defer resp.Body.Close()
```

* 禁止在循环中使用 `defer`(因为这样的defer得filterSomething函数结束才能统一调用)，举例如下：

```go
// 不要这样使用
func filterSomething(values []string) {    
    for _, v := range values {        
        fields, err := db.Query(v) // 示例，实际不要这么查询，防止sql注入        
        if err != nil {            
            // xxx        
        }        
        defer fields.Close()       
        // 继续使用fields    
    }
}
    
// 应当使用如下的方式：在每个闭包return之前处理闭包中的defer
func filterSomething(values []string) {    
    for _, v := range values {        
        func() {            
            fields, err := db.Query(v) // 示例，实际不要这么查询，防止sql注入            
            if err != nil {            
                ...            
            }            
            defer fields.Close()            
            // 继续使用fields        
        }()    
     }
}
```

## 6.3 【推荐】方法的接收器

* 【推荐】推荐以类名第一个英文首字母的小写作为接收器的命名。

* 【推荐】接收器的命名在函数超过`20行`的时候不要用单字符。

* 【必须】命名不能采用 `me`，`this`，`self` 这类易混淆名称。

## 6.4 【推荐】代码行数

* 【必须】文件长度不能超过`800行`。

* 【推荐】函数长度不能超过`80行`（函数长度为函数签名左括号下一行开始到右括号上一行结束部分的行数，包括代码行，注释行，空行）。

## 6.5 【必须】嵌套

* 嵌套深度不能超过`4层`：

```go
// AddArea 添加成功或出错
func (s *BookingService) AddArea(areas ...string) error {    
    s.Lock()    
    defer s.Unlock()    
    for _, area := range areas {        
        for _, has := range s.areas {            
            if area == has {                
                return srverr.ErrAreaConflict            
            }        
        }        
        s.areas = append(s.areas, area)        
        s.areaOrders[area] = new(order.AreaOrder)    
    }    
    return nil
}
```

```go
// 建议调整为这样：

// AddArea 添加成功或出错
func (s *BookingService) AddArea(areas ...string) error {    
    s.Lock()    
    defer s.Unlock()    
    for _, area := range areas {        
        if s.HasArea(area) {            
            return srverr.ErrAreaConflict        
        }        
        s.areas = append(s.areas, area)        
        s.areaOrders[area] = new(order.AreaOrder)    
    }    
    return nil
}

// HasArea ...func (s *BookingService) 
HasArea(area string) bool {    
    for _, has := range s.areas {        
        if area == has {            
            return true        
        }    
    }    
    return false
}
```

## 6.6 【推荐】变量声明

* 变量声明尽量放在变量第一次使用前面，就近原则。

## 6.7 【必须】魔法数字

魔数应使用常量或变量做替代。

魔数常常具有以下特征：

* 缺乏解释或命名的表示相同含义的独特数值，对读者来说难以理解，影响可读性

* 在程序中出现多次，当数值改变时，可能要改不只一个地方

例如：

```go
total := 1.05 * price

// 应改写为
const TaxRate = 0.05
total := (1.0 + TaxRate) * price
```

正确的使用命名量替代魔数可以改善可读性，但是也存在代价：

* 如果命名量与使用处距离很远，则会破坏代码的局部性，使代码的理解更加困难

```go
// 不要这样做！
const (    
    ...    
    appNameFormatErrorMsgFormat = "the app name is incorrect: %q"    
    userNameFormatErrorMsgFormat = "the user name is invalid: %q"    
    ...
)

... 省略 100 行 ...
func ParseAppName(v string) error {    
    ...    
    return fmt.Errorf(appNameFormatErrorMsgFormat, v) // 开发者无法直接知道需要传入几个参数，也无法通过错误信息直接找到产生错误的位置
}
```

* 它可能使代码更加冗长

* 它可能使代码运行更慢（从编译时计算变成运行时计算）

因此，正确识别魔数非常重要。一般而言，只要在上下文中能让人一眼明白其含义，并且基本没有需要改变的可能，就不会被认为是魔术数字。常见的例子包括：

```go
d = b*b - 4*a*c // 一元二次方程判别式公式中的 4
if x%2 == 0 {} // 中的 2
for i := 0; i < max; i += 1 {} // 中的 0 和 1
os.Exit(1) // 表示程序错误的 1（当然，如果需要区分多种不同的错误，则应该为不同的退出值进行命名）
scn.Buffer(nil, 10<<20) // 中的 10<<20
strings.IndexOf(s, ":") == -1 // 中的 ":" 和 -1
```

# 7. 依赖管理

## 7.1 【必须】go1.11 以上必须使用 `go modules` 模式：

```go
go mod init git.woa.com/group/myrepo
```

## 7.2 【推荐】代码提交

* 建议所有不对外开源的工程的 `module name` 使用 `git.woa.com/group/repo` ，方便他人直接引用。

* 建议使用 `go modules` 作为依赖管理的项目不提交 `vendor` 目录。

* 建议使用 `go modules` 管理依赖的项目， `go.sum` 文件必须提交，不要添加到 .gitignore 规则中。

# 8. 应用服务

## 8.1 【推荐】应用服务接口建议有 `README.md`

* 其中建议包括服务基本描述、使用方法、部署时的限制与要求、基础环境依赖（例如最低 go 版本、最低外部通用包版本）等。

## 8.2 【必须】应用服务必须要有接口测试。

## 附：常用工具

go 语言本身在代码规范性这方面也做了很多努力，很多限制都是强制语法要求，例如左大括号不换行，引用的包或者定义的变量不使用会报错，此外 go 还是提供了很多好用的工具帮助我们进行代码的规范。

* `gofmt` ，大部分的格式问题可以通过 `gofmt` 解决， `gofmt` 自动格式化代码，保证所有的 go 代码与官方推荐的格式保持一致，于是所有格式有关问题，都以 `gofmt` 的结果为准。

* `goimports` ，此工具在 `gofmt` 的基础上增加了自动删除和引入包。

* `go vet` ，`vet` 工具可以帮我们静态分析我们的源码存在的各种问题，例如多余的代码，提前 `return` 的逻辑， `struct` 的 `tag` 是否符合标准等。编译前先执行代码静态分析。

* `golint` ，类似 `javascript` 中的 `jslint` 的工具，主要功能就是检测代码中不规范的地方。
