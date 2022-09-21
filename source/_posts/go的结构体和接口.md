---
title: go的结构体和接口
categories:
  - go
tags:
  - go
date: 2021-02-07 15:24:36
updated: 2021-02-07 15:24:36
cover: https://proxy.qnoss.seeln.com/images/wp4221820-asteroid-belt-wallpapers.jpg
---

刚开始学go的时候，第一遍文档看完，没有搞懂结构体和接口的意思，现在大概明白了，记录一下。

## 函数和方法

在了解结构体和接口之前

在我写`JavaScript`的时候，大部分不区分这两个叫法，但是在`Java`中，定义在类中的函数，习惯称之为"方法"。

我在网上找到这么个说法：

- 函数是独立的功能，与对象无关，需要显示的传递数据
- 方法与对象和类相关，依赖对象而调用，可以直接处理对象上的数据，也就是隐式传递数据
- c语言中只有函数

emm...  和我理解的差不多。。

在`golang`中可以这么理解：

- 函数属于一个包，分为公有函数（首字母大写，其他包可以用），私有函数（首字母小写，只能当前包内部使用）。
- 方法属于一个结构体。

## 结构体

**结构体的声明和使用：**

```go
//...
type Person struct{
	age int
	name string
}

//...省略main函数
p := &Person{age:22,name:"ruomu"} // 等价于 p := &Person{22,"ruomu"}
fmt.Printf("age= %d, name= %s",p.age,p.name) // age= 22, name= ruomu
```
<!--more-->
**结构体的嵌套：**

```go
//...
type Address struct{
    country string
    city string
}

type Person struct{
   	age int
	name string
    add Address
}

//...省略main函数
p := &Person{age:22,name:"ruomu",add:Address{country:"china",city:"beijing"}} 
p.add.city // beijing
```

**结构体的方法：**

``` go
//...
type Person struct{
   	age int
	name string
}

func (p *Person) Hello(){
    fmt.Println("hello ",p.name)
}

//...省略main函数
p := Person{age:22,name:"ruomu"}
p.Hello() // hello ruomu
```

我们仔细一点会发现上面的代码中，接收者`p *Person`是一个指针类型，但是我们调用者`p := Person{age:22,name:"ruomu"}` 却是一个值类型。

事实上：

- 方法的接收者既可以是一个值类型，也可以是一个指针类型。
- 如果使用一个值类型变量调用指针类型接收者的方法，Go 语言编译器会自动帮我们取指针调用，以满足指针接收者的要求。
- 同样的原理，如果使用一个指针类型变量调用值类型接收者的方法，Go 语言编译器会自动帮我们解引用调用，以满足值类型接收者的要求。

所以，接收者定义为值类型还是指针类型，取决你想不想修改原始对象的值，不用考虑调用方是不是指针类型。


**方法可以赋值给一个变量，调用时第一个参数必须是该方法的接收者: **

```go
//...
type Person struct{
   	age int
	name string
}

func (p Person) Hello(){
    fmt.Println("hello ",p.name)
}

//...省略main函数
h := Person.Hello
h(p)
```

这里Hello方法的接收者写成指针类型就会报错，试了几种写法都不行。。。

## 接口

以上面的Person结构体为例

**接口的定义和实现：**

```go
//...
type WalkRun interface {
	Walk()
	Run()
}

type  Person struct {
	age int
	name string
}

func (p *Person)Walk(){
	fmt.Print("walk",p.name)
}

func(p *Person)Run(){
	fmt.Printf("%s run",p.name)
}

func  main()  {
	p := &Person{name:"zm",age:1}
	WalkRun.Run(p) // zm run 
    WalkRun.Walk(p) // walk zm
}
```

和java一样，如果一个接口有多个方法，那么需要实现接口的每个方法才算是实现了这个接口。（抽象类除外）

这里有一点需要注意，和结构体的方法不同，实现了接口的方法：

- 接收者是指针类型时，调用必须传入指针类型。
- 接收者是值类型时，调用者随意。（指针类型会被转为值类型，和前文描述的行为相同）

## 继承和组合

go语言中，没有继承的概念，所以go语言利用的是组合来达到代码复用的目的。

以go语言的io标准包的接口为例：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
//ReadWriter是Reader和Writer的组合
type ReadWriter interface {
    Reader
    Writer
}
```

当然结构体也可以组合：

```go
//...
type Address struct{
    country string
    city string
}

type Person struct{
   	age int
	name string
    Address
}

//...省略main函数
p := &Person{age:22,name:"ruomu",Address:Address{country:"china",city:"beijing"}}
p.city // beijing
```


**写在最后：**

接口组合并不是继承，因为它虽然可以复用"被继承"的接口的方法，但是却无法重写该方法。

感觉就是个语法糖，由于我oop接触的不多，反而不受java那套影响，更容易接受一些。。



参考链接：

- [https://www.cnblogs.com/wancy86/p/7271850.html](https://www.cnblogs.com/wancy86/p/7271850.html)

- [https://kaiwu.lagou.com/course/courseInfo.htm?courseId=536#/detail/pc?id=5231](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=536#/detail/pc?id=5231)