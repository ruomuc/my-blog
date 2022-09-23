---
title: go的接口和类型
categories:
  - go
tags:
  - go
date: 2022-02-27 14:01:33
updated: 2022-02-27 14:01:33

---

二刷 the go programming language，接口这块第一遍没太懂，第二遍大概懂了。



## 接口

go语言的接口的独特之处在于，它是隐式实现的。

对于一个具体的类型，你不需要声明它实现了哪些接口，只需要提供接口所必须的方法即可。

```go
// io.go
type Writer interface {
	Write(p []byte) (n int, err error)
}

type File struct {
	*file // os specific
}

// file.go
func (f *File) Write(b []byte) (n int, err error) {
    //... 此处省略
}
```

就像标准库的  file 类型，提供了 Write 方法后，就会隐式的实现 Writer 接口。



## 类型

对于传统的面向对象语言，实现接口的目的是可以使用不同的类型调用接口值的方法，即**多态**。

go 语言中的类型可以分为 具体类型 和 接口类型：

- 具体类型，比如 string、slice 等。
- 接口类型：接口类型是一种 抽象类型，它并没有暴露所含数据的布局和结构，它所提供的仅仅是一些方法而已。如果你拿到一个接口类型，你不能知道它是什么，但你可以知道它能做什么（提供的方法）。
- 从具体类型出发，提取其共性而得出的每一种分组方式，都可以表示为一种接口类型。



## 接口值

一个接口的接口值分为两个部分：

- 具体类型，又称为 接口的动态类型。
- 具体类型的值，又称为 接口的动态值。

![](https://image.seeln.com/images/go%E7%9A%84%E6%8E%A5%E5%8F%A3%E4%B8%8E%E7%B1%BB%E5%9E%8B.png)

```go
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = nil
```

分析代码片段：

| 行数 | 接口类型  | 接口值 | 动态类型      | 动态值                                    | 备注                                                         |
| ---- | --------- | ------ | ------------- | ----------------------------------------- | ------------------------------------------------------------ |
| 1    | io.Writer | nil    | nil           | nil                                       | 一个接口值是否为 nil，取决于它的动态类型和动态值是否都是 nil |
| 2    | io.Writer |        | *os.File      | 一个指向代表进程标准输出的 os.File 的指针 |                                                              |
| 3    | io.Writer |        | *bytes.Buffer | 一个指向新分配缓冲区的指针                |                                                              |
| 4    | io.Writer | nil    | nil           | nil                                       |                                                              |



### 含有空指针的非空接口

```go
const debug = false

func main() {
	var buf *bytes.Buffer
	fmt.Println(reflect.ValueOf(buf))
	if debug {
		buf = new(bytes.Buffer)
	}

	f(buf)
	if debug {
		// do something...
	}
}

func f(out io.Writer) {
	// ...其他代码...
	fmt.Printf("%T, %v\n", out, reflect.ValueOf(out))
	if out != nil {
		out.Write([]byte("done!\n"))
	}
}
```

上述代码在 debug=false 时会报错。

因为如果 debug=false，f 函数调用时，out参数是一个**含有空指针的非空接口**（  一个接口值是否为 nil，取决于它的动态类型和动态值是否都是 nil，这里out的动态类型不是nil，而动态值是nil）。

此时 out != nil 的结果为 true，但当调用 out.Write 时，会发现 动态值为 nil， 所以会报错。 

如果把 `var buf *bytes.Buffer` 改为 `var buf io.Writer` 就可以了。



## 类型断言

类型断言就是一个作用在接口值上的操作，写作`x.(T)`，其中 x 是一个接口类型，T 是一个类型（称为断言类型）。

1. 当 T 是一个具体类型时，类型断言就是从来从它的操作数中把**具体类型的值**提取出来的操作。如果检查失败，程序崩溃。

```go
var w io.Writer // w 接口类型为 io.Writer
w = os.Stdout // w 具体类型为 *os.File, 具体值 os.Stdout 副本（一个指针）
f := w.(*os.File) // f == os.Stdout
c := w.(*bytes.Buffer) // 程序崩溃
```

2. 当 T 是一个接口类型时，那么类型断言检查 x 的动态类型是否满足 T。无论是否成功，只有**结果的接口类型**变为 T，其他都没变。

```go
var w io.Writer
w = os.Stdout
rw := w.(io.ReadWriter) // 成功，*os.File有 Read 和 Write 方法， rw 的接口类型为 io.ReadWriter

w = new(ByteCounter)
rw := w.(io.ReadWriter) // 崩溃：*ByteCounter 没有 Read 方法
```

3. 如果操作数是一个空接口，类型断言总会失败。