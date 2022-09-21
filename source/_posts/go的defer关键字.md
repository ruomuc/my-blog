---
title: go的defer关键字
categories:
  - go
tags:
  - go
date: 2021-05-30 19:55:29
updated: 2021-05-30 19:55:29
cover: https://proxy.qnoss.seeln.com/images/wp4221867-asteroid-belt-wallpapers.jpg
---

go 语言中的 defer 关键字常用语关闭文件描述符、数据库连接或解锁资源。（这是 node.js 没有的）

本文主要研究一下，defer 如何使用以及它的执行机制。
<!--more-->
## 基本用法
假设有如下方法，用于处理一个文件：

```go
func DealFile () error {
  file, err := os.Open("test.txt")
  if err != nil {
    return err
  }
  // 省略业务操作
  // ...
  file.Close()
  return nil
}
```

这段程序有一个问题。。如果 os.Open() 返回了一个错误，那么将永远不会执行 file.Close()，当打开的文件数过多时，就会触发"too many open files“的系统错误，从而让整个系统陷入崩溃。

那么如何避免呢，可以使用 defer 关键字，在 os.Open() 之后马上调用 file.Close()：

```go
func DealFile () error {
  file, err := os.Open("test.txt")
  defer file.Close()
  if err != nil {
    return err
  }
  // 省略业务操作
  // ...
 
  return nil
}
```



## 执行规则

defer 的使用看起来很简单，但是如果没有真正理解 defer 的执行规则，会遇到很多奇怪的问题。

### defer 的执行顺序

多个 defer 调用，如何执行：

```go
func main() {
  i := 1
  defer fmt.Println(i)
  i++
  defer fmt.Println(i)
  i++
  defer fmt.Println(i)
  i++
  fmt.Println(i)
}
// output:
4
3
2
1
```

很简单可以看出，多个 defer 的执行顺序是**后进先出**。


### defer 的参数解析

看三个代码片段：

```go
func main() {
  i := 1
  defer fmt.Println(i)
  i++
  fmt.Println(i)
}
// output:
2
1
```

```go
func main() {
  i := 1
  defer func (){fmt.Println(i)}() 
  i++
  fmt.Println(i)
}
// output:
2
2
```

```go
func main() {
  i := 1
  defer func (i int){fmt.Println(i)}(i) 
  i++
  fmt.Println(i)
}
// output:
2
1
```

1. defer 关键字的参数是实时调用并解析计算的。
2. 使用不传参的匿名函数可以解决这一点。

### defer链式调用

还有一种还少见的情况，链式调用：

```go
type T struct{}

func (t T) f(n int) T {
  fmt.Println(n)
  return t
}

func main() {
  var t T
  defer t.f(1).f(2)
  fmt.Println(3)
}
// output:
1
3
2
```

这段代码是我在某大佬博客里整理的面试题里看到的，是这样解释的：

defer 延迟调用时，需要保存函数指针和参数，因此链式调用的情况下，除了最后一个函数/方法外的函数/方法都会在调用时直接执行。也就是说 t.f(1) 直接执行，然后执行 fmt.Print(3)，最后函数返回时再执行 .f(2)，因此输出是 132。







