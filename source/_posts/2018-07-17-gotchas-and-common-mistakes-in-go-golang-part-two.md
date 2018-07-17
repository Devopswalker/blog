---
title: Golang 新开发者要注意的陷阱和常见错误【初级二】
date: 2018-07-17 23:02:28
updated: 2018-07-17 00:21:34
categories: 他山之石
tags: [100days,golang,技巧,常见错误]
photos: 
- http://pbcq37bxu.bkt.clouddn.com/the-aerolite-hole.jpg
---

原文（EN）：[50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)

原文（CN）：[鸟窝大神博客](http://colobu.com/2015/09/07/gotchas-and-common-mistakes-in-go-golang/)

本文只涉及其中的初级内容，后续深入学习实践后再行归纳总结中高级内容。

---

### 偶然的变量隐藏Accidental Variable Shadowing

短式变量声明的语法如此的方便（尤其对于那些使用过动态语言的开发者而言），很容易让人把它当成一个正常的分配操作。如果你在一个新的代码块中犯了这个错误，将不会出现编译错误，但你的应用将不会做你所期望的事情。

错误示例：
```
package main
import "fmt"
func main() {  
    x := 1
    fmt.Println(x)     //prints 1
    {
        fmt.Println(x) //prints 1
        x := 2
        fmt.Println(x) //prints 2
    }
    fmt.Println(x)     //prints 1 (bad if you need 2)
}
```
即使对于经验丰富的Go开发者而言，这也是一个非常常见的陷阱。这个坑很容易挖，但又很难发现。

你可以使用 vet命令来发现一些这样的问题。 默认情况下， vet不会执行这样的检查，你需要设置-shadow参数：
```
go tool vet -shadow your_file.go。
```
### 不使用显式类型，无法使用“nil”来初始化变量
**nil** 标志符用于表示 interface、函数、maps、slices 和 channels 的“零值”。如果你不指定变量的类型，编译器将无法编译你的代码，因为它猜不出具体的类型。
```
package main

func main() {  
    var x = nil //error
    _ = x
}
```

编译错误：
> ./HttpDemo.go:20:6: use of untyped nil

有效示例：
```
package main

func main() {  
    var x interface{} = nil
    _ = x
}
```
### 使用“nil” Slices and Maps

在一个 **nil** 的 slice 中添加元素是没问题的，但对一个 map 做同样的事将会生成一个运行时的 panic。

有效示例：
```
package main

func main() {  
    var s []int
    s = append(s,1)
}

```
错误示例：
```
package main

func main() {  
    var m map[string]int
    m["one"] = 1 //error
}
```

### Map的容量

你可以在map创建时指定它的容量，但你无法在 map 上使用 cap() 函数。

错误示例：

```
package main

func main() {  
    m := make(map[string]int,99)
    cap(m) //error
}
```
编译错误：

> ./HttpDemo.go:5: invalid argument m (type map[string]int) for cap

### 字符串不会为 nil

这对于经常使用 **nil** 分配字符串变量的开发者而言是个需要注意的地方。

错误示例：
```
package main

func main() {  
    var x string = nil //error
    if x == nil { //error
        x = "default"
    }
}
```
编译错误：

> ./HttpDemo.go:7: cannot use nil as type string in assignment ./HttpDemo.go:7:9: invalid operation: x == nil (mismatched types string and nil)

有效示例：

```
package main

func main() {  
    var x string //defaults to "" (zero value)
    if x == "" {
        x = "default"
    }
}
```

### Array函数的参数

如果你是一个 C 或则 C++ 开发者，那么数组对你而言就是指针。当你向函数中传递数组时，函数会参照相同的内存区域，这样它们就可以修改原始的数据。Go 中的数组是数值，因此当你向函数中传递数组时，函数会得到原始数组数据的一份复制。如果你打算更新数组的数据，这将会是个问题。

```
package main

import "fmt"

func main() {  
    x := [3]int{1,2,3}
    func(arr [3]int) {
        arr[0] = 7
        fmt.Println(arr) //prints [7 2 3]
    }(x)
    fmt.Println(x) //prints [1 2 3] (not ok if you need [7 2 3])
}
```
如果你需要更新原始数组的数据，你可以使用数组指针类型。

```
package main

import "fmt"

func main() {  
    x := [3]int{1,2,3}
    func(arr *[3]int) {
        (*arr)[0] = 7
        fmt.Println(arr) //prints &[7 2 3]
    }(&x)
    fmt.Println(x) //prints [7 2 3]
}
```
另一个选择是使用 slice。即使你的函数得到了 slice 变量的一份拷贝，它依旧会参照原始的数据。

```
package main

import "fmt"

func main() {  
    x := []int{1,2,3}
    func(arr []int) {
        arr[0] = 7
        fmt.Println(arr) //prints [7 2 3]
    }(x)
    fmt.Println(x) //prints [7 2 3]
}
```
### 在 Slice 和 Array 使用“range”语句时的出现的不希望得到的值

如果你在其他的语言中使用“for-in”或者“foreach”语句时会发生这种情况。Go 中的“range”语法不太一样。它会得到两个值：第一个值是元素的索引，而另一个值是元素的数据。
糟糕示例：
```
package main

import "fmt"

func main() {  
    x := []string{"a","b","c"}
    for v := range x {
        fmt.Println(v) //prints 0, 1, 2
    }
}
```
建议示例：
```
package main

import "fmt"

func main() {  
    x := []string{"a","b","c"}
    for _, v := range x {
        fmt.Println(v) //prints a, b, c
    }
}
```

### Slices 和 Arrays 是一维的

看起来 Go 好像支持多维的 Array 和 Slice，但不是这样的。尽管可以创建数组的数组或者切片的切片。对于依赖于动态多维数组的数值计算应用而言，Go 在性能和复杂度上还相距甚远。

你可以使用纯一维数组、“独立”切片的切片，“共享数据”切片的切片来构建动态的多维数组。

如果你使用纯一维的数组，你需要处理索引、边界检查、当数组需要变大时的内存重新分配。

使用“独立” slice 来创建一个动态的多维数组需要两步。首先，你需要创建一个外部的 slice。然后，你需要分配每个内部的 slice。内部的 slice 相互之间独立。你可以增加减少它们，而不会影响其他内部的 slice。
```
package main

func main() {  
    x := 2
    y := 4
    table := make([][]int,x)
    for i:= range table {
        table[i] = make([]int,y)
    }
}
```

使用“共享数据” slice 的 slice 来创建一个动态的多维数组需要三步。首先，你需要创建一个用于存放原始数据的数据“容器”。然后，你再创建外部的 slice。最后，通过重新切片原始数据 slice 来初始化各个内部的 slice。

```
package main

import "fmt"

func main() {  
    h, w := 2, 4
    raw := make([]int,h*w)
    for i := range raw {
        raw[i] = i
    }
    fmt.Println(raw,&raw[4])
    //prints: [0 1 2 3 4 5 6 7] <ptr_addr_x>
    table := make([][]int,h)
    for i:= range table {
        table[i] = raw[i*w:i*w + w]
    }
    fmt.Println(table,&table[1][0])
    //prints: [[0 1 2 3] [4 5 6 7]] <ptr_addr_x>
}
```
关于多维 array 和 slice 已经有了专门申请，但现在看起来这是个低优先级的特性。

### 访问不存在的 Map Keys

这对于那些希望得到“**nil**”标示符的开发者而言是个技巧（和其他语言中做的一样）。如果对应的数据类型的“零值”是“**nil**”，那返回的值将会是“**nil**”，但对于其他的数据类型是不一样的。检测对应的“零值”可以用于确定 map 中的记录是否存在，但这并不总是可信（比如，如果在二值的map中“零值”是 false，这时你要怎么做）。检测给定 map 中的记录是否存在的最可信的方法是，通过 map 的访问操作，检查第二个返回的值。

糟糕示例：

```
package main

import "fmt"

func main() {  
    x := map[string]string{"one":"a","two":"","three":"c"}
    if v := x["two"]; v == "" { //incorrect
        fmt.Println("no entry")
    }
}
```
建议示例：
```
package main

import "fmt"

func main() {  
    x := map[string]string{"one":"a","two":"","three":"c"}
    if _,ok := x["two"]; !ok {
        fmt.Println("no entry")
    }
}
```

### Strings 无法修改

尝试使用索引操作来更新字符串变量中的单个字符将会失败。string 是只读的 byte slice（和一些额外的属性）。如果你确实需要更新一个字符串，那么使用 byte slice，并在需要时把它转换为 string 类型。

错误示例：
```
package main

import "fmt"

func main() {  
    x := "text"
    x[0] = 'T'
    fmt.Println(x)
}
```
编译错误：
> ./HttpDemo.go:29:5: cannot assign to x[0]

有效示例：
```
package main

import "fmt"

func main() {  
    x := "text"
    xbytes := []byte(x)
    xbytes[0] = 'T'
    fmt.Println(string(xbytes)) //prints Text
}
```
需要注意的是：这并不是在文字 string 中更新字符的正确方式，因为给定的字符可能会存储在多个 byte 中。如果你确实需要更新一个文字 string，先把它转换为一个 rune slice。即使使用 rune slice，单个字符也可能会占据多个 rune，比如当你的字符有特定的重音符号时就是这种情况。这种复杂又模糊的“字符”本质是 Go 字符串使用 byte 序列表示的原因。

### String和Byte Slice之间的转换

当你把一个字符串转换为一个 byte slice（或者反之）时，你就得到了一个原始数据的完整拷贝。这和其他语言中 cast 操作不同，也和新的 slice 变量指向原始 byte slice 使用的相同数组时的重新 slice 操作不同。

Go 在[]byte 到 string 和 string 到[]byte 的转换中确实使用了一些优化来避免额外的分配（在 todo 列表中有更多的优化）。

第一个优化避免了当[]byte keys 用于在 map[string] 集合中查询时的额外分配:m[string(key)]。

第二个优化避免了字符串转换为[]byte 后在 for range 语句中的额外分配：for i,v := range []byte(str) {...}。

### String和索引操作

字符串上的索引操作返回一个 byte 值，而不是一个字符（和其他语言中的做法一样）。

```
package main

import "fmt"

func main() {  
    x := "text"
    fmt.Println(x[0]) //print 116
    fmt.Printf("%T",x[0]) //prints uint8
}
```
如果你需要访问特定的字符串“字符”（unicode编码的points/runes），使用for range。官方的“unicode/utf8”包和实验中的 utf8string 包（golang.org/x/exp/utf8string）也可以用。utf8string 包中包含了一个很方便的 At() 方法。把字符串转换为 rune 的切片也是一个选项。

### 字符串不总是 UTF8 文本

字符串的值不需要是 UTF8 的文本。它们可以包含任意的字节。只有在 string literal 使用时，字符串才会是 UTF8。即使之后它们可以使用转义序列来包含其他的数据。

为了知道字符串是否是 UTF8，你可以使用“unicode/utf8”包中的 ValidString() 函数。

```
package main

import (  
    "fmt"
    "unicode/utf8"
)

func main() {  
    data1 := "ABC"
    fmt.Println(utf8.ValidString(data1)) //prints: true
    data2 := "A\xfeC"
    fmt.Println(utf8.ValidString(data2)) //prints: false
}
```
### 字符串的长度

让我们假设你是 Python 开发者，你有下面这段代码：
```
data = u'♥'  
print(len(data)) #prints: 1
```
当把它转换为 Go 代码时，你可能会大吃一惊。

```
package main

import "fmt"

func main() {  
    data := "♥"
    fmt.Println(len(data)) //prints: 3
}
```
内建的 len() 函数返回 byte 的数量，而不是像 Python 中计算好的 unicode 字符串中字符的数量。

要在 Go 中得到相同的结果，可以使用“unicode/utf8”包中的 RuneCountInString() 函数。

```
package main

import (  
    "fmt"
    "unicode/utf8"
)
func main() {  
    data := "♥"
    fmt.Println(utf8.RuneCountInString(data)) //prints: 1
}
```
理论上说 RuneCountInString() 函数并不返回字符的数量，因为单个字符可能占用多个 rune。

```
package main
import (  
    "fmt"
    "unicode/utf8"
)
func main() {  
    data := "é"
    fmt.Println(len(data))                    //prints: 3
    fmt.Println(utf8.RuneCountInString(data)) //prints: 2
}
```
### 在多行的 Slice、Array 和 Map 语句中遗漏逗号

错误示例：
```
package main
func main() {  
    x := []int{
    1,
    2 //error
    }
    _ = x
}
```
编译错误：

> ./HttpDemo.go:17:12: syntax error: unexpected newline, expecting comma or }

有效示例：
```
package main

func main() {  
    x := []int{
    1,
    2,
    }
    x = x
    y := []int{3,4,} //no error
    y = y
}
```
当你把声明折叠到单行时，如果你没加末尾的逗号，你将不会得到编译错误。