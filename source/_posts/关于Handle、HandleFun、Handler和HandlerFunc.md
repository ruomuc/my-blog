---
title: 关于Handle、HandleFunc、Handler和HandlerFunc
categories:
  - go
tags:
  - go
  - http
date: 2021-06-10 12:48:20
updated: 2021-06-10 12:48:20
cover: https://proxy.qnoss.seeln.com/images/wp4202327-aconcagua-wallpapers.jpg
---

在go中，可以很简单实现一个http服务器。

但是在使用过程中，遇到了一些容易使人迷惑的类型、接口或方法名。

于是决定深入了解并记录一下它们的区别。

ps: 本文的前置知识是，需要对go中的面向对象和接口有一定的了解。



<!--more-->

我们可以这样创建一个http server

```go
package main

import (
	"fmt"
	"net/http"
)

type Hello struct {
	name string
}

func (h Hello) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "hello %s", h.name)
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "hello world")
}

func main() {
	http.HandleFunc("/", indexHandler)
	http.Handle("/hello", &Hello{"ruomu"})
	http.ListenAndServe(":8000", nil)
}
```

这时如果你访问 `localhost:8080/` 会看到 `hello world`

如果访问`localhost:8080/hello`会看到`hello ruomu`



## Handler

我们翻看源码`net/http/server.go`可以看到：

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

所以所有实现了 ServeHTTP 方法的结构体创建出来的对象，都可以看做是一个Handler类型的对象。

## Handle

源码：

```go
func Handle(pattern string, handler Handler) {
  DefaultServeMux.Handle(pattern, handler) 
}
```

可以看到，第一个参数不用说了，是绑定的路由，第二个参数是一个 Handler类型的对象。

再回头看前文的代码：

```go
...
type Hello struct {
	name string
}
func (h Hello) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "hello %s", h.name)
}
...
http.Handle("/hello", &Hello{"ruomu"})
```

`ServeHTTP` 方法其实对应的就是对应路由被访问时，对应的执行处理方法，至于具体什么时候被执行，如果绑定这些细节这里先不说了。

是不是知道为什么，访问 `localhost:8080/hello` 会出现 `hello ruomu`了。

## HandleFunc

源码：

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```

我们发现和 Handle 的区别：

第二个参数不一样，Handle 的第二个参数是一个Handler 类型的对象，而HandleFunc的第二个参数，直接就是一个 `func(ResponseWriter, *Request)`类型的函数 。

里面执行的语句也有点不大相同，分别是 `  DefaultServeMux.Handle(pattern, handler) ` 和 `	DefaultServeMux.HandleFunc(pattern, handler)`。它们也只是调用了不同的方法，并且第二个参数不同。。。感觉像是在套娃？ 这里先不展开说了，我们只需要知道它们是正正的注册路由的对象就行了。



试想一下，如果我们需要绑定1000个路由和其对应的处理方法，那我们是不是要写1000个struct？

```go
type r1 struct {}
func (route r1) ServeHTTP(w http.ResponseWriter, r *http.Request) {...}
type r2 struct {}
func (route r2) ServeHTTP(w http.ResponseWriter, r *http.Request) {...}
.
.
.
type r1000 struct {}
func (route r1000) ServeHTTP(w http.ResponseWriter, r *http.Request) {...}

func main() {
	http.Handle("/r1", &r1{})
  http.Handle("/r2", &r2{})
  ...
	http.Handle("/r100", &r1000{})
	http.ListenAndServe(":8000", nil)
}
```

这样的代码是难以阅读并且难以维护的，我们根本就不需要这么多结构体，我们最终关心的其实时处理方法 ServeHTTP。

看下前文的代码：

```go
...
func indexHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "hello world")
}
...
http.HandleFunc("/", indexHandler)
...
```

是不是已经知道，如何改造绑定1000个路由的代码了。。

## HandlerFunc

源码：

```go
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

这个写法很有意思啊。。

其实目的也是解决绑定1000个路由需要定义1000个struct的问题。

使用  HandlerFunc 进行强制类型转换，因为HandlerFunc也实现了 ServeHTTP方法，所以可以当做一个Handler类型的对象作为http.Handle的第二个参数。

然后Handler的ServeHTTP方法中使用 `f(w,r)`，啊这。。秒啊。。

```go
...
func indexHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "hello world")
}
...
http.Handle("/", http.HandlerFunc(indexHandler))
...
```

现在是不是疑惑，HandleFunc 和 HandlerFunc有区别？为什么要两种写法呢。。

其实我们继续看HandleFunc的源码就知道了：

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
...
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	if handler == nil {
		panic("http: nil handler")
	}
	mux.Handle(pattern, HandlerFunc(handler))
}
```

可以看到，HandleFunc 最后还是使用了 `Handle(pattern, HandlerFunc(handler))` 来处理的。。

## DefaultServeMux

这里顺便说一下，本文一开始的代码中，为什么`	http.ListenAndServe(":8000", nil) `第二个参数要传一个 nil 呢。

关于ServeMux到时候另写一片文章，这里直接不细说。

http.ListenAndServe 的第二个参数其实是一个ServeMux对象，如果不传的话，就会使用默认的 DefaultServeMux。

这篇文章代码里使用的所有绑定路由的方法，都是绑定在 DefaultServeMux上的，如果我们使用 NewServeMux创建一个自定义对象，并且当做 http.ListenAndServe 的第二个参数传入的话，它们就会失效。

`net/http/server.go:2835`

```go
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
		handler = DefaultServeMux
	}
	if req.RequestURI == "*" && req.Method == "OPTIONS" {
		handler = globalOptionsHandler{}
	}
	handler.ServeHTTP(rw, req)
}
```

## 总结

- Handler 是一个接口，只有一个 ServeHTTP方法。
- Handle 绑定路由第二个参数需要传一个Handler类型的对象。
- HandlerFunc 写法比较有意思，他也实现了 ServeHTTP方法，可以强转一个函数为Handler类型的对象，并当做Handle 的第二个参数。
-  `HandleFunc(pattern, handle)` 写法等于 `Handle(pattern, HandlerFunc(handle)) `



ps: 如果看不懂我写得东西很正常，本来就是记录不是什么教程，写得很随意。这里建议直接看参考链接里的文章。多看多练。



参考链接：

- https://perennialsky.medium.com/understand-handle-handler-and-handlefunc-in-go-e2c3c9ecef03
- https://cizixs.com/2016/08/17/golang-http-server-side/
- https://learnku.com/articles/37867

