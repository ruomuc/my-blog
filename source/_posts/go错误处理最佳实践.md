---
title: go错误处理最佳实践
categories:
  - go
tags:
  - go
  - 错误处理
date: 2021-07-24 14:22:27
updated: 2021-07-24 14:22:27

---

用 go 写过业务代码之后，就会发现，go的错误处理很让人头痛。。（事实上被吐槽的确实很多~）

然后刚好在极客时间薅了一节体验课。整理一下。以供参考。

注：对于初学者而言，最起码你要自己写一个curd的demo，不然看完之后不会有什么感觉。

<!--more-->

## Sentinel Error

预定义的特定错误，我们叫为 sentinel error。

这个名字来源于计算机编程中使用一个特定值来表示不可能进行进一步处理的做法。所以对于 Go，我们使用特定的值来表示错误。

常用的比如 io 标准库的 EOF 错误表示读文件读到了结尾，但是它并不属于错误，

所以我们可能会这么写：

```go
if err != nil && err != io.EOF {
    return nil, err
}
// dosomething
```

包括在使用 gorm 的时候：

```go
if err != nil && err != gorm.ErrRecordNotFound {
    return nil, err
}
// dosomething
```

这种写法有什么缺点呢：
- 无法携带上下文，携带信息少
- Sentinel Error 成为了你的 API 的公共部分，必须暴露给外部。
- Sentinel Error 在调用者和被调用者之间创建了依赖关系（强耦合），如果需要改动，必须要兼容。

**所以尽可能的避免使用 Sentinel Error，尽管标准库中有使用它们。**

<br>

## Error types
Error types 可以携带更多的上下文

自定义一个 Error类型，然后实现 Error ()方法：

```go
type MyError struct {
	Msg  string
	File string
	Line int
}

func (me *MyError) Error() string {
	return fmt.Sprintf("%s:%d: %s", me.File, me.Line, me.Msg)
}

func test() error {
	return &MyError{"Something Wrong", "server.go", 32}
}

func main() {
	err := test()
	switch err := err.(type) {
	case nil:
	// doSomething
	case *MyError:
		fmt.Println("error occurred on line:", err.Line)
	default:
		// doSomething
	}
}
// output: 
error occurred on line: 32
```

os 标准库中的实现：

```go
type PathError struct {
    Op   string
    Path string
    Err  error
}
func (e *PathError) Error() string
```

<br>

缺点：

- Error types 需要把自定义Error类型（MyError）暴露出去，比Sentinel Error更加脆弱。

**虽然 Error types 携带了更多的附加信息，但是并没有解决根本问题。不推荐使用。。**

<br>

## Opaque errors

我们将这种风格称为不透明错误处理。

不再依赖类型，而依赖方法。

net 标准库中的实现：

```go
type Error interface {
    error
    Timeout() bool   // Is the error a timeout?
    Temporary() bool // Is the error temporary?
}

type InvalidAddrError string

func (e InvalidAddrError) Error() string   { return string(e) }
func (e InvalidAddrError) Timeout() bool   { return false }
func (e InvalidAddrError) Temporary() bool { return false }
```

可以这样判断错误类型：

```go
if e, ok := err.(net.InvalidAddrError); ok {
    return e.Temporary()
}
```

无论内部的错误类型如何改变，只要 Temporary 方法还在，就是兼容的。

**推荐使用这种方式**

<br>

## Wrap errors

**日志记录与错误无关，且对调试没有帮助的信息应被视为噪音。**

The error has been logged (错误日志要被记录)

The application is back to 100% integrity (应用程序处理错误，)

```go
// 不应该出现这种处理
if err != nil {
    // 仅仅打印日志，没有处理错误
    fmt.Println(err)
}
```

The current error is not reported any longer (之后不再报告当前错误)

不应该每一层都打印记录错误日志，一种是对错误降级记录后不再向上返回，一种是不记录向上返回错误，由最上层统一记录日志。

**主要有 Wrap、WithMessage、Cause 三个方法**：

```go
func Wrap(err error, message string) error {
	if err == nil {
		return nil
	}
	err = &withMessage{
		cause: err,
		msg:   message,
	}
	return &withStack{
		err,
		callers(),
	}
}

func WithMessage(err error, message string) error {
	if err == nil {
		return nil
	}
	return &withMessage{
		cause: err,
		msg:   message,
	}
}

func Cause(err error) error {
	type causer interface {
		Cause() error
	}

	for err != nil {
		cause, ok := err.(causer)
		if !ok {
			break
		}
		err = cause.Cause()
	}
	return err
}
```

- Wrap，给原始错误添加信息，并携带堆栈信息
- WithMessage，给错误添加信息，但不携带堆栈信息
- Cause，对应 Wrap 操作，获取原始错误

**使用 Wrap errors原则：**

- 调用第三方库/标准库时，产生错误的地方，应该使用 Wrap 包装。
- 调用包内的函数时，不应该使用 Wrap（因为上一条已经包装过了），否则堆栈信息会重复。
- 在程序顶部 或者 goroutine 顶部，适用 `%+v` 记录堆栈详细信息。
- 使用 errors.Cause 获取 root error，再和 sentinel error 进行比较判定。

### 总结

1. 只有业务代码会使用 pkg/errors 这个库，高度重用的代码（底层库标准库等）不应该使用 Wrap 包装错误。
2. 我们应该在产生错误的地方 Wrap，然后在顶层打日志。
3. 错误一旦被处理，就不应该再往上抛。

<br>

## Tips

视频里大佬分享的一些技巧

### case1

```go
// 方式1
f, err := os.Open(path)
if err != nil {
    // handler error
}
// doSomething

// 方式2
f, err := os.Open(path)
if err == nil {
    // doSomething
}
// handler error
```

建议使用方式1，方式2会导致业务代码缩进，我们应该优先处理错误，让业务逻辑成一条直线。

### case2

方式1：

```go
func CountLines(r io.Reader) (int, error) {
	var (
		br    = bufio.NewReader(r)
		lines int
		err   error
	)
	for {
		_, err = br.ReadString('\n')
		lines++
		if err != nil {
			break
		}
	}
	if err != io.EOF {
		return 0, err
	}
	return lines, nil
}
```

方式2：

```go
func CountLines(r io.Reader) (int, error) {
	var (
		br    = bufio.NewScanner(r)
		lines int
	)

	for br.Scan() {
		lines++
	}
	return lines, br.Err()
}
```

方式2 是对 方式1 代码的优化。

### case3

方式1：

```go
type Header struct {
	Key, value string
}

type status struct {
	Code   int
	Reason string
}

func WriteResponse(w io.Writer, st status, headers []Header, body io.Reader) error {
	_, err := fmt.Fprintf(w, "HTTP/1.1 %d %s\r\n", st.Code, st.Reason)
	if err != nil {
		return err
	}

	for _, h := range headers {
		_, err := fmt.Fprintf(w, "%s: %s\r\n", h.Key, h.value)
		if err != nil {
			return err
		}
	}

	if _, err = fmt.Fprintf(w, "\r\n"); err != nil {
		return err
	}

	_, err = io.Copy(w, body)
	return err
}
```

方式2：

```go
type errWrite struct {
	io.Writer
	err error
}

func (ew *errWrite) Write(buf []byte) (int, error) {
	if ew.err != nil {
		return 0, ew.err
	}
	var n int
	n, ew.err = ew.Writer.Write(buf)
	return n, nil
}

func WriteResponse(w io.Writer, st status, headers []Header, body io.Reader) error {
	ew := &errWrite{Writer: w}
	fmt.Fprintf(ew, "HTTP/1.1 %d %s\r\n", st.Code, st.Reason)

	for _, h := range headers {
		fmt.Fprintf(ew, "%s: %s\r\n", h.Key, h.value)
	}
	fmt.Fprintf(ew, "\r\n")
	return ew.err
}
```

方式2自定义 errWrite 对象，并实现 io.Writer 接口，把 err 信息保存起来，最后再 return。

并且在 Write 内部使用 `if ew.err != nil`，来减少前文中的大量 `if err != nil`判断。 



## 总结

编写底层库的时候，可以使用 Opaque errors 这种风格

编写业务代码的时候，使用 Wrap errors 来处理错误







