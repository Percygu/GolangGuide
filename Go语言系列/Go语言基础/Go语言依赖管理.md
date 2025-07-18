---
tags:
  - Go
  - golang
  - go基础语法
  - modules
  - go modules
  - 依赖管理
---

# Go语言依赖管理

在工程代码中，每种语言基本上都有自己的依赖管理工具，比如python的`pip`、node.js的`npm`，java的`maven`，rust的`cargo`，Go语言也有提供自己的依赖库管理工具。Go语言从v1.5开始开始引入vendor模式，如果项目目录下有vendor目录，那么go工具链会优先使用vendor内的包进行编译、测试等。在go1.11之后，Go语言主要使用`Go modules`对代码依赖进行管理。

## Go Modules要点

### GO111MODULE环境变量
这个环境变量是Go Modules的开关，主要有以下参数：
- auto：只在项目包含了`go.mod`文件时启动go modules，在Go1.13版本中是默认值
- on：无脑启动Go Modules，推荐设置，Go1.14版本以后的默认值
- off：禁用Go Modules，一般没有使用`go modules`的工程使用；


### GOPROXY
该环境变量用于设置Go模块代理，Go后续在拉取模块版本时能够脱离传统的VCS方式从镜像站点快速拉取，`GOPROXY`的值要以英文逗号分割，默认值是`https://proxy.golang.org,direct`，但是该地址在国内无法访问，所以可以使用`goproxy.cn`来代替(七牛云配置)，设置命令：
```bash
go env -w GOPROXY=https://goproxy.cn,direct
```
也可以使用其他配置，例如阿里配置：
```bash
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/
```
该环境变量也可以关闭，可以设置为"off"，禁止Go在后续操作中使用任何Go module proxy；
上面的配置中我们用逗号分割后面的值是`direct`，它是什么意思呢？
`direct`为特殊指示符，因为我们指定了镜像地址，默认是从镜像站点拉取，但是有些库可能不存在镜像站点中，`direct`可以指示Go回源到模块版本的源地址去抓取，比如`github`，当`go module proxy`返回`404、410`这类错误时，其会自动尝试列表中的下一个，遇见`direct`时回源地址抓取；


### GOSUMDB
`GOSUMDB`（go checksum database）是Go官方为了go modules安全考虑，设定的module校验数据库，你在本地对依赖进行变动（更新/添加）操作时，Go 会自动去这个服务器进行数据校验，保证你下的这个代码库和世界上其他人下的代码库是一样的，保证Go在拉取模块版本时拉取到的模块版本数据未经篡改

GOSUMDB的值自定义格式如下：
- 格式 1：<SUMDB_NAME>+<PUBLIC_KEY>。
- 格式 2：<SUMDB_NAME>+<PUBLIC_KEY> <SUMDB_URL>。

`GOSUMDB`的默认值是`sum.golang.org`，默认值与自定义值的格式不一样，默认值在国内是无法访问，这个值我们一般不用动，因为我们一般已经设置好了`GOPROXY`，`goproxy.cn`支持代理`sum.golang.org`;
所以，环境变量`GOSUMDB`可以用来配置你使用哪个校验服务器和公钥来做依赖包的校验
但是在使用的时候需要注意，如果你的代码仓库或者模块是私有的，那么它的校验值不应该出现在互联网的公有数据库里面，但是我们本地编译的时候默认所有的依赖下载都会去尝试做校验，这样不仅会校验失败，更会泄漏一些私有仓库的路径等信息，我们可以使用`GONOSUMDB`这个环境变量来设置不做校验的代码仓库， 它可以设置多个匹配路径，用逗号相隔.

例如：
```bash
go env -w GONOSUMDB=*.example.com,test.xyz/com
```
这样的话，像 `git.example.com`, `test.xyz/com` 这些公司和自己的私有仓库就都不会做校验了。


### GONOPROXY/GONOSUMDB/GOPRIVATE
这三个环境变量放在一起说，一般在项目中不经常使用，这三个环境变量主要用于私有模块的拉取，在GOPROXY、GOSUMDB中无法访问到模块的场景中，例如拉取git上的私有仓库；
GONOPROXY、GONOSUMDB的默认值是GOPRIVATE的值，所以我们一般直接使用GOPRIVATE即可，其值也是可以设置多个，以英文逗号进行分割；例如：
```bash
go env -w GOPRIVATE="github.com/asong2020/go-localcache,git.xxxx.com"
```
也可以使用通配符的方式进行设置，对域名设置通配符号，这样子域名就都不经过`Go module proxy`和`Go checksum database`


### 全局缓存
`go mod download`会将依赖缓存到本地，缓存的目录是`GOPATH/pkg/mod/cache`、`GOPATH/pkg/sum`，这些缓存依赖可以被多个项目使用，未来可能会迁移到`$GOCACHE`下面；
可以使用`go clean -modcache`清理所有已缓存的模块版本数据；


## Go Modules命令
我们可以使用`go help mod`查看可以使用的命令：
```bash
go help mod
Go mod provides access to operations on modules.

Note that support for modules is built into all the go commands,
not just 'go mod'. For example, day-to-day adding, removing, upgrading,
and downgrading of dependencies should be done using 'go get'.
See 'go help modules' for an overview of module functionality.

Usage:

        go mod <command> [arguments]

The commands are:

        download    download modules to local cache
        edit        edit go.mod from tools or scripts
        graph       print module requirement graph
        init        initialize new module in current directory
        tidy        add missing and remove unused modules
        vendor      make vendored copy of dependencies
        verify      verify dependencies have expected content
        why         explain why packages or modules are needed

Use "go help mod <command>" for more information about a command.
```
命令 | 作用
--- | ---
go mod init | 生成go.mod文件
go mod download | 下载go.mod文件中指明的所有依赖放到全局缓存
go mod tidy | 整理现有的依赖，添加缺失或移除不使用的modules
go mod graph | 查看现有的依赖结构
go mod edit | 编辑go.mod文件
go mod vendor | 导出项目所有的依赖到vendor目录
go mod verify | 校验一个模块是否被篡改过
go mod why | 解释为什么需要依赖某个模块


### go.mod文件
`go.mod`是启用Go modules的项目所必须且最重要的文件，其描述了当前项目的元信息，每个`go.mod`文件开头符合包含如下信息：

**module**：用于定义当前项目的模块路径（突破$GOPATH路径）
**go**：当前项目Go版本，目前只是标识作用
**require**：用于设置一个特定的模块版本
**exclude**：用于从使用中排除一个特定的模块版本
**replace**：用于将一个模块版本替换为另外一个模块版本，例如chromedp使用[golang.org/x/image](https://golang.org/x/image)这个package一般直连是获取不了的，但是它有一个[github.com/golang/image](https://github.com/golang/image)的镜像，所以我们要用replace来用镜像替换它
**retract**：用来声明该第三方模块的某些发行版本不能被其他模块使用，在Go1.16引入

示例如下：
```go
module rotatebot

go 1.17

require (
   github.com/gin-gonic/gin v1.8.1
   github.com/sirupsen/logrus v1.9.0
   gorm.io/driver/sqlite v1.4.3
   gorm.io/gorm v1.24.2
)

require (
   github.com/gin-contrib/sse v0.1.0 // indirect
   github.com/go-playground/locales v0.14.0 // indirect
)

exclude (
   github.com/json-iterator/go v1.1.12 
)

replace (
   github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd ==> github.com/modern-go/concurrent v0.0.0-20190606011245-iyfhlkiuhydg
)

retract v0.2.0
```
假设我们有上述`go.mod`文件，接下来我们分模块详细介绍一下各个部分


### module 
`go.mod`文件的第一行是`module`， 表示工程里的依赖的基路径，例如上面的项目：
```go
module rotatebot
```
工程里的`import`的路径都是以`rotatebot`开头的字符串


### go version
`go.mod`文件的第二行是`go version`，其是用来指定你的代码所需要的最低版本：
```go
go 1.17
```


### require
`require`用来指定该项目所需要的各个依赖库以及他们的版本，从上面的例子中我们看到版本部分有不同的写法，还有注释，接下来我们来解释一下这部分；


#### indirect注释
```go
github.com/gin-contrib/sse v0.1.0 // indirect
github.com/go-playground/locales v0.14.0 // indirect
```
以下场景才会添加`indirect`注释：
- 当前项目依赖包A，A又依赖包B，但是A的 `go.mod`文件中缺失B，所以在当前项目`go.mod`中补充B并添加`indirect`注释
- 当前项目依赖包A，A又依赖包B，但是依赖包A没有`go.mod`文件，所以在当前项目`go.mod`中补充B并添加`indirect`注释
- 当前项目依赖包A，A又依赖包B，当依赖包A降级不再依赖B时，这个时候就会标记indirect注释，可以执行go mod tidy移除该依赖；

Go1.17版本对此做了优化，`indirect`的module将被放在单独`require`块的，这样看起来更加清晰明了。

#### incompatible标记
`incompatible`标记其实是一个module标签归规范约束，Go module 的版本选择机制规定，Module 的版本号需要遵循 v\<major\>.\<minor\>.\<patch\> 的格式，此外，如果major版本号大于`1`时，其版本号还需要体现在Module名字中。
比如[Module github.com/RainbowMango/m](https://github.com/RainbowMango/m)，如果其版本号增长到 v2.x.x 时，其 Module 名字也需要相应的改变为：[github.com/RainbowMango/m/v2](https://github.com/RainbowMango/m/v2)。即，如果major版本号大于`1`时，需要在Module名字中体现版本。
那么如果 Module 的 major 版本号虽然变成了 v2.x.x，但 Module 名字仍保持原样会怎么样呢？ 其他项目是否还可以引用呢？
假设[github.com/gin-contrib/sse](https://github.com/gin-contrib/sse)的当前版本为v3.5.0，按照Go module 的版本选择机制，其 Module 名字需要相应的改变为： [github.com/gin-contrib/sse/v3](https://github.com/gin-contrib/sse/releases/tag/v3.5.0)，但是如果module名没改，还是[github.com/gin-contrib/sse](https://github.com/gin-contrib/sse)，则在被形目引用的时候，就会在后面加上incompatible标记，变成
```go
require (
        github.com/gin-contrib/sse v3.6.0+incompatible
)
```
除了增加 +incompatible（不兼容）标识外，在其使用上没有区别


#### 版本号
go module拉取依赖包本质也是go get行为，go get主要提供了以下命令：

命令 | 作用
--- | ---
go get | 拉取依赖，会进行指定性拉取（更新），并不会更新所依赖的其它模块
go get -u | 更新现有的依赖，会强制更新它所依赖的其它全部模块，不包括自身
go get -u -t ./... | 更新所有直接依赖和间接依赖的模块版本，包括单元测试中用到的

go get拉取依赖包取决于依赖包是否有发布的tags：

1. 拉取的依赖包没有发布tags
    - 默认取主分支最近一次的commit的commit hash，生成一个伪版本号
2. 拉取的依赖包有发布tags
    - 如果只有单个模块，那么就取主版本号最大的那个tag
    - 如果有多个模块，则推算相应的模块路径，取主版本号最大的那个tag

没有发布的tags：
```go
github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd
```
v0.0.0：根据commit的base version生成的：
- 如果没有base version，那么就是vx.0.0的形式
- 如果base version是一个预发版本，那么就是vx.y.z-pre.0的形式
- 如果base version是一个正式发布的版本，那么它就patch号加1，就是vx.y.(z+1)-0的形式

20190718012654：是这次提交的时间，格式是yyyyMMddhhmmss
fb15b899a751：是这个版本的commit id，通过这个可以确定这个库的特定的版本

有发布的tags：
```go
github.com/gin-contrib/sse v0.1.0
```


### replace
因为某些未知原因，并不是所有的包都能直接用`go get`获取到，或者说是我们想要在官方的依赖库中集成一些我们自己的功能，这时我们就需要使用`go modules`的`replace`功能了
`replace`顾名思义，就是用新的`package`去替换另一个`package`，他们可以是不同的`package`，也可以是同一个`package`的不同版本。看一下基本的语法：
```bash
go mod edit -replace=old[@v]=new[@v]
```
`old`是要被替换的package，`new`就是用于替换的package。

replace的使用步骤：
1. 首先`go get new-package`（如果你知道package的版本tag，那么这一步其实可以省略，如果想使用最新的版本而不想确认版本号，则需要这一步）
2. 然后查看`go.mod`，手动复制`new-package`的版本号（如果你知道版本号，则跳过）
3. `go mod edit -replace=old[@v]=new[@v]`
4. 接着`go mod tidy`或者`go build`或者使用其他的`go tools`，他们会去获取`new-package`然后替换掉`old-package`
5. 最后，在你的代码里直接使用`old-package`的名字，golang会自动识别出`replace`，然后实际你的程序将会使用`new-package`，替换成功


### exclude
这个特性是在Go1.16版本中引入，用来声明该第三方模块的某些发行版本不能被其他模块使用；
使用场景：发生严重问题或者无意发布某些版本后，模块的维护者可以撤回该版本，支持撤回单个或多个版本；
这种场景以前的解决办法：
维护者删除有问题版本的`tag`，重新打一个新版本的`tag`；
使用者发现有问题的版本`tag`丢失，手动介入升级，并且不明真因；
引入`retract`后，维护者可以使用`retract`在`go.mod`中添加有问题的版本：
```go
// 严重bug...
retract (
  v0.1.0
  v0.2.0
)
```
重新发布新版本后，在引用该依赖库的使用执行`go list`可以看到 版本和"严重bug..."的提醒。该特性的主要目的是将问题更直观的反馈到开发者的手中；


### go.sum文件
Go 在做依赖管理时会创建两个文件，`go.mod` 和 `go.sum`，`go.mod` 的重要性不言而喻，这个文件几乎提供了依赖版本的全部信息。而 `go.sum` 则是记录了所有依赖的 module 的校验信息，以防下载的依赖被恶意篡改，主要用于安全校验。这个文件我们一般不需要编辑，更新以来的时候会自动更新。
每行的格式如下：
```
<module> <version> <hash>
<module> <version>/go.mod <hash>
```
比如：
```
github.com/spf13/cast v1.4.1 h1:s0hze+J0196ZfEMTs80N7UlFt0BDuQ7Q+JDnHiMWKdA=
github.com/spf13/cast v1.4.1/go.mod h1:Qx5cxh0v+4UWYiBimWS+eyWzqEqokIECu5etghLkUJE=
```
其中 `module` 是依赖的路径，`version` 是依赖的版本号。如果 `version` 后面跟 `/go.mod` 表示对哈希值是 `module` 的 `go.mod` 文件；否则，哈希值是 `module` 的 `.zip` 文件。
`hash` 是以 `h1:` 开头的字符串，表示生成 checksum 的算法是第一版的HASH算法（SHA256）。如果将来在SHA-256中发现漏洞，将添加对另一种算法的支持，可能会命名为`h2`。

## Go Modules使用

使用 `go modules` 的一个前置条件是Go语言版本大于等于Go1.11；然后我们要检查环境变量 `GO111MODULE` 是否开启，执行 `go env` 查看：
```bash
go env | grep GO111MODULE
GO111MODULE="on"
```
如果GO111MODULE=off，可以执行一下命令打开
```bash
go env -w GO111MODULE=on
```
接下来就可以使用GO MODULE管理项目工程了，先创建一个项目目录
```bash
mkdir -p go_tour/main
cd go_tour
```
执行：
```bash
go mod init go_tour 
```
运行结果 ：
```
go: creating new go.mod: module go_tour
```
会在go_tour目录下生成一个go.mod文件。在main包下建立main.go文件
```go
package main

import "github.com/tidwall/gjson"

const json = `{"name":{"hello":"golang","key1":"value1"},"id":12345}`

func main() {
    value := gjson.Get(json, "name.key1")
    println(value.String())
}
```

然后在 `go_tour` 目录下执行 `go mod tidy` 命令，可以看到：
```
go: finding module for package github.com/tidwall/gjson
go: downloading github.com/tidwall/gjson v1.14.4
go: found github.com/tidwall/gjson in github.com/tidwall/gjson v1.14.4
```
可以看到此时的go.mod文件内容：
```go
module go_tour

go 1.17

require github.com/tidwall/gjson v1.14.4

require (
   github.com/tidwall/match v1.1.1 // indirect
   github.com/tidwall/pretty v1.2.0 // indirect
)
```
依赖已经安装好，并且可以使用了，在 `go.mod` 下还可以看到一个 `go.sum` 文件对依赖包的校验，`go.sum` 我们一般不用管
```
github.com/tidwall/gjson v1.14.4 h1:uo0p8EbA09J7RQaflQ1aBRffTR7xedD2bcIVSYxLnkM=
github.com/tidwall/gjson v1.14.4/go.mod h1:/wbyibRr2FHMks5tjHJ5F8dMZh3AcwJEMf5vlfC0lxk=
github.com/tidwall/match v1.1.1 h1:+Ho715JplO36QYgwN9PGYNhgZvoUSc9X2c80KVTi+GA=
github.com/tidwall/match v1.1.1/go.mod h1:eRSPERbgtNPcGhD8UCthc6PmLEQXEWd3PRB5JTxsfmM=
github.com/tidwall/pretty v1.2.0 h1:RWIZEg2iJ8/g6fDDYzMpobmaoGh5OLl4AXtGUGPcqCs=
github.com/tidwall/pretty v1.2.0/go.mod h1:ITEVvHYasfjBbM0u2Pg8T2nJnzm8xPwvNhhsoaGGjNU=
``` 
此时执行 `go run main.go` 就可以看到执行结果了
```
value1
``` 
以上就是用`go module`管理依赖库的简单用法。