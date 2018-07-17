---
title: Golang 新开发者要注意的陷阱和常见错误【初级一】
date: 2018-07-11 00:36:34
updated: 2018-07-11 00:56:34
categories: 他山之石
tags: [100days,golang,技巧,常见错误]
photos: 
- http://pbcq37bxu.bkt.clouddn.com/the-aerolite-hole.jpg
---


原文（EN）：[50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)

原文（CN）：[鸟窝大神博客](http://colobu.com/2015/09/07/gotchas-and-common-mistakes-in-go-golang/)

本文只涉及其中的初级相关内容，后续深入学习实践后再行归纳总结中级和高级内容。

---

### 开（左）大括号不能放在单独的一行
在大多数其他使用大括号的语言中，你需要选择放置它们的位置。Go 的方式不同，你可以为此感谢下自动分号的注入（没有预读）。是的，Go 中也是有分号的：-）

错误示例：
```
package main

import "fmt"

func main()
{// error, can't have the opening brace on a separate line
	fmt.Println("Hello world!")

}
```
编译错误：
> ./basic.go:43:1: syntax error: unexpected semicolon or newline before {

有效示例：
```
package main

import "fmt"

func main() {

	fmt.Println("Hello world!")

}
```
### 未使用的变量
如果你有未使用的变量，代码将编译失败。当然也有例外。在函数内一定要使用声明的变量，但未使用的全局变量是没问题的。
如果你给未使用的变量分配了一个新的值，代码还是会编译失败。你需要在某个地方使用这个变量，才能让编译器愉快的编译。

错误示例：
```
package main

var gvar int //not an error

func main() {  
    var one int   //error, unused variable
    two := 2      //error, unused variable
    var three int //error, even though it's assigned 3 on the next line
    three = 3     
}
```
编译错误：
> /tmp/sandbox473116179/main.go:6: one declared and not used
/tmp/sandbox473116179/main.go:7: two declared and not used
/tmp/sandbox473116179/main.go:8: three declared and not used

有效示例：
```
package main

import "fmt"

func main() {  
    var one int
    _ = one
    two := 2 
    fmt.Println(two)
    var three int 
    three = 3
    one = three
    var four int
    four = four
}
```
另一个选择是注释掉或者移除未使用的变量 ：-）

### 未使用的Imports
如果你引入一个包，而没有使用其中的任何函数、接口、结构体或者变量的话，代码将会编译失败。
你可以使用 goimports 来增加引入或者移除未使用的引用：
```
go get golang.org/x/tools/cmd/goimports
```
如果你真的需要引入的包，你可以添加一个下划线标记符，_，来作为这个包的名字，从而避免编译失败。下滑线标记符用于引入，但不使用。

错误示例：
```
package main

import (  
    "fmt"
    "log"
    "time"
)
func main() {  
}
```
编译错误：
> /tmp/sandbox627475386/main.go:4: imported and not used: "fmt"
/tmp/sandbox627475386/main.go:5: imported and not used: "log"
/tmp/sandbox627475386/main.go:6: imported and not used: "time"

有效示例：
```
package main

import (  
    _ "fmt"
    "log"
    "time"
)
var _ = log.Println
func main() {  
    _ = time.Now
}
```
另一个选择是移除或者注释掉未使用的imports ：-）

### 简式的变量声明仅可以在函数内部使用
错误示例：
```
package main

myvar := 1 //error

func main() {  
}
```
编译错误：
> /tmp/sandbox265716165/main.go:3: non-declaration statement outside function body

有效示例：
```
package main

var myvar = 1

func main() {  
}
```
### 使用简式声明重复声明变量
你不能在一个单独的声明中重复声明一个变量，但在多变量声明中这是允许的，其中至少要有一个新的声明变量。
重复变量需要在相同的代码块内，否则你将得到一个隐藏变量。
错误示例：
```
package main

func main() {  
    one := 0
    one := 1 //error
}
```
编译错误：
> /tmp/sandbox706333626/main.go:5: no new variables on left side of :=

有效示例：
```
package main

func main() {  
    one := 0
    one, two := 1,2
    one,two = two,one
}
```
