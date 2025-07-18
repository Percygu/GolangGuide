---
tags:
  - Go
  - golang
  - go基础语法
  - 循环
---

# Go语言循环
Go语言的循环不像其他语言一样有多种，比如c++有`for`，`while`，`do-while`。在go语言中循环就只有`for`一种，所以用起来也是十分的方便。

## for循环的形式
Go语言的`for`循环有 3 种形式，只有其中的一种使用分号。

1. 第一种：类似于c语言的for循环
```go
for init; condition; post {

}
```

2. 第二种：类似于c语言的while
```go
for condition {

}
```

3. 第三种：类似于c语言的 for(;;) 
```go
for {

}
```
- init： 一般为赋值表达式，给控制变量赋初值；
- condition： 关系表达式或逻辑表达式，循环控制条件；
- post： 一般为赋值表达式，给控制变量增量或减量。
```go
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```

## for range
for 循环的`range`格式可以对`slice`、`map`、数组、字符串等进行迭代循环。格式如下：
```go
for key, value := range oldMap {
    newMap[key] = value
}
```
以上代码中的`key`和`value`是可以省略的，如果省略`key`，则返回索引，如果省略`value`，则返回元素值。
如果只想读取 key，格式如下：
```go
for key := range oldMap
```
如果只想读取 value，格式如下：
```go
for _, value := range oldMap
```
代码展示：
```go
package main

import "fmt"

func main() {
   for i := 0; i < 5; i++ {
      fmt.Printf("current i %d\n", i)
   }
   j := 0
   for {
      if j == 5 {
         break
      }
      fmt.Printf("current j %d\n", j)
      j++
   }
   var strAry = []string{"aa", "bb", "cc", "dd", "ee"} //是的，不指定初始个数也ok
   //切片初始化
   var sliceAry = make([]string, 0)
   sliceAry = strAry[1:3]
   for i, str := range sliceAry {
      fmt.Printf("slice i %d, str %s\n", i, str)
   }
   //字典初始化
   var dic = map[string]int{
      "apple":      1,
      "watermelon": 2,
   }
   for k, v := range dic {
      fmt.Printf("key %s, value %d\n", k, v)
   }
}
```
输出如下：
```
current i 0
current i 1
current i 2
current i 3
current i 4
current j 0
current j 1
current j 2
current j 3
current j 4
slice i 0, str bb
slice i 1, str cc
key apple, value 1
key watermelon, value 2
```

## for range的坑
这块可能需要点`golang`基础，也可以把后面的先看了再来看这块内容。
通过上面例子不难发现`for range`对于数组，map的遍历非常方便，但是`for range`也有不好的坑，其中不乏一些熟悉`golang`的开发人员，往往也会掉到坑里，下面就给大家来捋一捋`golang for range`中那些容易遇到的坑，这也是面试`golang`基础时通常会被面试官问到的

### for range取不到所有元素的地址
> **PS：如果你用的是Go 1.22版本以及之后的话，那是能取到地址的，而且要注意，这个地址是临时变量的地址，不是原元素的地址**
```go
package main

import "fmt"

func main() {
    arr := [2]int{1, 2}
    res := []*int{}
    for _, v := range arr {
       res = append(res, &v)
    }
    // expect: 1 2 预期的结果
    // but 
    // result: 2 2 实际的结果
    fmt.Println(*res[0], *res[1])
}
```
上述代码通过定义一个数组`arr`，数组元素为`1`，`2`。然后试图通过取到数组的这两个元素的地址放到切片`res`中，最后通过取地址操作符`*res[0]`和`*res[1]`打印出切片中的元素，希望结果输出`1`和`2`，但结果恰恰不是我们所预期的那样。
代码实际输出：
```
2 2
```
实际输出的是两个2，那么问题在哪里呢？
<font color="#dd0000">我们可以在每次for range 循环打印出v的地址，会发现v是不变的，那么我们每次将v的地址加入 到res中，res中最终所有的元素都是一个地址，这一个地址最终指向的是v最后遍历得到的值——也就是2
</font>

```go
package main

import "fmt"

func main() {
    arr := [2]int{1, 2}
    res := []*int{}
    for _, v := range arr {
       // v 每次都是同一个变量
       fmt.Println(&v)
       res = append(res, &v)
    }
    // expect: 1 2
    // but
    // result: 2 2
    fmt.Println(*res[0], *res[1])
}
```
代码输出：
```
0xc000018050
0xc000018050
2 2
```

那如果我们想要得到预期的结果`1`和`2`，应该怎么改呢？两种方式：
1. 第一种：使用局部变量`v1`拷贝`v`
```go
for _, v := range arr {
    //局部变量v替换了v，也可用别的局部变量名
    v1 := v 
    res = append(res, &v1)
}
```
2. 第二种：直接使用索引获取原来的元素
```go
for k := range arr {
    res = append(res, &arr[k])
}
```

### 循环是否会停止？
```go
v := []int{1, 2, 3}
for i := range v {
    v = append(v, i)
}
```
在循环遍历的同时往遍历的切片追加元素，循环会停止吗？
答案是：**会**。
在Go语言中，`for i := range v`语句会在循环开始前对切片`v`的长度进行一次评估，并将这个长度用于控制循环的迭代次数。之后，如果在循环体内修改了切片`v`的长度（比如通过`append`函数），这个修改并不会影响已经确定的循环迭代次数。
上述例子可以看作是下面这个代码：
```go
v := []int{1, 2, 3}
length := len(v)
for i := 0; i < length; i++ {
    v = append(v, i)
}
```

### 使用迭代变量时的闭包问题
#### 3.3.1 问题
在`for range`循环中，如果在闭包中使用迭代变量，可能会遇到意想不到的结果。因为闭包会捕获迭代变量的引用，而不是它的值。
```go
package main

import (
    "fmt"
)

func main() {
    var funcs []func()

    for i := 0; i < 3; i++ {
        funcs = append(funcs, func() {
            fmt.Println(i)
        })
    }

    for _, f := range funcs {
        f()
    }
}
```
代码输出：
```
3
3
3
```
#### 解决方法
使用局部变量保存当前迭代变量的值。
```go
package main

import (
    "fmt"
)

func main() {
    var funcs []func()

    for i := 0; i < 3; i++ {
        i := i // 创建新的局部变量 i
        funcs = append(funcs, func() {
            fmt.Println(i)
        })
    }

    for _, f := range funcs {
        f()
    }
}
```
代码输出：
```
0
1   
2
```

### 修改切片中的元素
#### 问题
`for range`会创建每个元素的副本，而不是直接操作原始切片中的元素。因此，修改迭代变量不会影响原始切片。
```go
package main

import (
    "fmt"
)

func main() {
    slice := []int{1, 2, 3}
    
    for _, v := range slice {
        v *= 10
    }
    
    fmt.Println(slice) // 输出: [1 2 3]
}
```
代码输出：
```
[1 2 3]
```
#### 解决方法
使用索引访问并修改原始切片中的元素
```go
package main

import (
    "fmt"
)

func main() {
    slice := []int{1, 2, 3}
    
    for i := range slice {
        slice[i] *= 10
    }
    
    fmt.Println(slice) // 输出: [10 20 30]
}
```
代码输出：
```
[10 20 30]
```

### 遍历字典时的顺序
#### 问题
在 Go 中，使用`for range`遍历字典时，遍历顺序是随机的。每次运行程序时，顺序可能不同。
```go
package main

import (
    "fmt"
)

func main() {
    dic := map[string]int{"a": 1, "b": 2, "c": 3}

    for k, v := range dic {
        fmt.Printf("key: %s, value: %d\n", k, v)
    }
}
```
代码输出：
```
key: a, value: 1
key: c, value: 3
key: b, value: 2
```
每次输出的顺序可能不同

#### 解决方法
使用`for`循环遍历字典，并使用`sort.Strings`函数对键进行排序，再遍历
```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    dic := map[string]int{"a": 1, "b": 2, "c": 3}
    keys := make([]string, 0, len(dic))

    for k := range dic {
        keys = append(keys, k)
    }

    sort.Strings(keys)

    for _, k := range keys {
        fmt.Printf("key: %s, value: %d\n", k, dic[k])
    }
}
```
代码输出：
``` 
key: a, value: 1
key: b, value: 2
key: c, value: 3
```
**对比学习**
>    C++：
        •        std::map：有序容器，遍历顺序是键的升序。（内部使用红黑树实现）
        •        std::unordered_map：无序容器，遍历顺序不可预测。（内部使用哈希表实现） 
    Python：
        •        Python 3.7 及更高版本：字典遍历顺序是插入顺序。
        •        Python 3.6 及更早版本：字典遍历顺序可能是插入顺序，但并不保证。
>
 
 ### 字符串遍历
 #### 问题
 `for range`遍历字符串时，每次迭代会返回`Unicode`代码点`（rune）`，而不是字节。如果字符串包含多字节字符，这一点尤其重要。
```go
package main

import (
    "fmt"
)

func main() {
    str := "hello 世界"
    
    for i, r := range str {
        fmt.Printf("index: %d, rune: %c\n", i, r)
    }
}
```
代码输出：
```
index: 0, rune: h
index: 1, rune: e
index: 2, rune: l
index: 3, rune: l
index: 4, rune:  
index: 5, rune: 世
index: 6, rune: 界
```

#### 解决方法
理解`for range`返回的是 `Unicode` 代码点，而不是字节。如果需要按字节遍历，可以使用常规的`for`循环。
```go
package main

import (
    "fmt"
)

func main() {
    str := "hello 世界"
    
    for i := 0; i < len(str); i++ {
        fmt.Printf("index: %d, byte: %x\n", i, str[i])
    }
}
```
代码输出：
```
index: 0, byte: 68
index: 1, byte: 65
index: 2, byte: 6c
index: 3, byte: 6c
index: 4, byte: 6f
index: 5, byte: 20
index: 6, byte: e4
index: 7, byte: b8
index: 8, byte: 96
index: 9, byte: e7
index: 10, byte: 95
index: 11, byte: 8c
```
### 总结
1. Go 1.22之前通过`for _, v := range arr`遍历切片取不到所有变量的地址，而是同一个临时变量的地址
2. 闭包中的迭代变量：迭代变量在闭包中被捕获，导致所有闭包共享同一个变量。
3. 切片元素修改： `for range` 会创建元素的副本，直接修改迭代变量不会影响原切片。
4. 字典遍历顺序：遍历字典时，顺序是随机的，每次运行可能不同。
5. 字符串遍历：`for range` 遍历字符串时返回的是 `Unicode` 代码点（`rune`），而不是字节，可能导致多字节字符处理复杂。
6. 删除切片元素： 在 `for range` 中删除切片元素可能导致意外行为或漏掉某些元素。
7. 遍历修改映射（字典）：在`for range`循环中修改字典（如添加或删除键值对）可能导致未定义行为或错误。





