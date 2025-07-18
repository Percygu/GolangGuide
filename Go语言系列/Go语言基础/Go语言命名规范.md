---
tags:
  - Go
  - golang
  - go基础语法
  - go命名规范
---

# Go语言命名规范

在Go语言中，任何标识符，包括变量，常量，函数名，方法名，接口名，以及自定义类型等，都应该遵循以下规则

## 1. 命名要区分大小写
Go语言是一种区分大小写的编程语言。其命名规则涵盖了变量、常量、全局函数、结构体、接口、方法等元素。在 Go 的语法规定中，对于需要暴露给外部使用的标识符，必须以大写字母开头，而那些不需要外部访问的则应使用小写字母开头。

1. 当一个名称（如常量、变量、类型、函数名或结构体字段等）以大写字母开头时，例如：Analysize，它将成为外部包代码可以访问的对象（前提是外部包需要先导入此包），这种方式被称为“导出”（类似于面向对象语言中的 public）。
2. 如果名称以小写字母开头，则该标识符仅对当前包内可见并可使用，无法被包外访问（类似于面向对象语言中的 private）。

## 1. 包名称
包名必须全部为小写单词，无下划线，也不要混合大小写，越短越好，尽量不要与标准库重名。并且包名最好和目录保持一致，这样可读性强
```go
package domain
package service
package modle
```

## 2. 文件名
文件名同样遵循简短有意义的原则，文件名必须为小写单词，允许加下划线‘_’组合方式，但是头尾不能为下划线
```go
order_service.go
```
虽然go文件命名允许出现下划线，但是为了代码的整洁性，以及避免与一些系统规定的特定后缀冲突，还是建议少使用下划线，能不用尽量不用

比如以 `_test`为后缀的go编译器会认为是测试文件，不会编译到工程里面。除了`_test`为后缀的测试文件外，还有不少系统的文件，我们在命名的时候也应当尽量避免
```go
_386.go、_amd64.go、_arm.go、_arm64.go、_android.go、_darwin.go、_dragonfly.go、_freebsd.go、_linux.go、_nacl.go、_netbsd.go、_openbsd.go、_plan9.go、_solaris.go、_windows.go、_android_386.go、_android_amd64.go、_android_arm.go、_android_arm64.go、_darwin_386.go、_darwin_amd64.go、_darwin_arm.go、_darwin_arm64.go、_dragonfly_amd64.go、_freebsd_386.go、_freebsd_amd64.go、_freebsd_arm.go、_linux_386.go、_linux_amd64.go、_linux_arm.go、_linux_arm64.go、_linux_mips64.go、_linux_mips64le.go、_linux_ppc64.go、_linux_ppc64le.go、_linux_s390x.go、_nacl_386.go、_nacl_amd64p32.go、_nacl_arm.go、_netbsd_386.go、_netbsd_amd64.go、_netbsd_arm.go、_openbsd_386.go、_openbsd_amd64.go、_openbsd_arm.go、_plan9_386.go、_plan9_amd64.go、_plan9_arm.go、_solaris_amd64.go、_windows_386.go
_windows_amd64.go
```

## 3. 常量名
1.
常量&枚举名规范起见，采用大小写混排的驼峰模式（Golang官方要求），不要出现下划线。比如
```go
const (
	    TypeBooks    = iota // 0
	    TypePhone           // 1
	    TypeCoin           // 2
	)
```

2. 常量的定义应根据功能对类型进行分类，而不是将所有类型归为一组。此外，建议将公共常量放在私有常量之前。
```go
const (
	    TypePage = "page"
	
	    // The rest are node types; home page, sections etc.
	    TypeHome         = "home"
	    TypeSection      = "section"
	    TypeTaxonomy     = "taxonomy"
	    TypeTaxonomyTerm = "taxonomyTerm"
	
	    // Temporary state.
	    TypeUnknown = "unknown"
	
	    // The following are (currently) temporary nodes,
	    // i.e. nodes we create just to render in isolation.
	    TypeRSS       = "RSS"
	    TypeSitemap   = "sitemap"
	    TypeRoboTypeXT = "roboTypeXT"
	    Type404       = "404"
	)
```

3. 如果常量类型是枚举类型，需要先创建相应类型。示例如下：
```go
	type TypeCompareType int
	
	const (
	    TypeEq TypeCompareType = iota
	    TypeNe
	    TypeGt
	    TypeGe
	    TypeLt
	    TypeLe
	)
```

## 4. 变量名
变量名称通常遵循驼峰命名法，首字母根据访问控制原则决定是大写还是小写。但对于特定名词，需遵循以下规则：
1. 如果变量是私有的，且特定名词位于名称的首位，则应使用小写字母（例如 appService）。
2. 如果变量类型为 bool，则名称应以 Has、Is、Can 或 Allow 开头。

```go
var isExist bool
var hasConflict bool
var canManage bool
var allowGitHook bool
```

## 5. 结构体名
结构体命名同样应使用驼峰命名法，变量名的首字母根据访问控制规则决定是大写还是小写。
对于 struct 的声明和初始化，建议采用多行格式，示例如下：

```go
type ServiceConfig struct {
    Port string `json:"port"`
    Address string `json:"address"`
}
config := ServiceConfig{"8080", "111.222.333.444"}
```

## 6. 接口名
接口的命名规则几乎和结构体相同，只是需要多注意一点，通常只包含单个函数的接口名以 “er” 作为后缀，例如 Reader , Writer

```go
type Reader interface {
        Read(p []byte) (n int, err error)
}
```

## 7. 函数名
1. 函数名必须为大小写混排的驼峰模式，注意，函数名开头字母大写表示该函数是可导出的，可以在包外的其他地方被调用，如果函数名开头是小写的，则只能在包内被调用。另外还需要注意一点的是：函数名不应该与标准库中的函数名冲突
```go
func DoJob     // 暴露给包外部函数

func doJob    // 包内部函数
```

2. 函数名要求精简准确，并采用用动词或动词短，比如
```go
func save(){}
func delete(){}
func getUser(){}
```

