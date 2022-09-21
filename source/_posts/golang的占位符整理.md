---
title: golang的“占位符”整理
categories:
  - go
tags:
  - go
date: 2021-04-18 14:50:13
updated: 2021-04-18 14:50:13

---

在 golang 中，格式化输出的占位符有很多种。这里整理一下方便用的时候查阅

<!--more-->

## 通用占位符

| 占位符 | 说明                                           |
| ------ | ---------------------------------------------- |
| %v     | 值的默认格式表示                               |
| %+v    | 类似 %v，但是输出结构体时，会添加字段名（key） |
| %#v    | 值得go语法表示                                 |
| %T     | 打印值得类型                                   |
| %%     | 百分号                                         |

测试代码：
```go
package main

import "fmt"

func main() {
	fmt.Printf("%v\n", 123)
	fmt.Printf("%v\n", "hello")
	fmt.Printf("%v\n", false)

	obj := struct {
		name string
	}{"测试一下"}
	fmt.Printf("%v\n", obj)
	fmt.Printf("%+v\n", obj)
	fmt.Printf("%#v\n", obj)
	fmt.Printf("%T\n", obj)
	fmt.Printf("%d%%\n",99)
}
// output:
123
hello
false
{测试一下}
{name:测试一下}
struct { name string }{name:"测试一下"}
struct { name string }
99%
```

## 整形

| 占位符 | 说明                                       |
| ------ | ------------------------------------------ |
| %b     | 表示为二进制                               |
| %c     | 该值对应的unicode码                        |
| %d     | 表示为十进制                               |
| %o     | 表示为八进制                               |
| %x     | 表示为十六进制，使用a-f                    |
| %X     | 表示为十六进制，使用A-F                    |
| %U     | 表示为Unicode格式：U+xxx                   |
| %q     | 单引号围绕的字符字面值，由Go语法安全地转义 |

测试代码：

```go
package main

import "fmt"

func main() {
	num := 99
	fmt.Printf("%b\n",num)
	fmt.Printf("%c\n",num)
	fmt.Printf("%d\n",num)
	fmt.Printf("%o\n",num)
	fmt.Printf("%x\n",num)
	fmt.Printf("%X\n",num)
	fmt.Printf("%U\n",num)
	fmt.Printf("%q\n",num)
}
// output:
1100011
c
99
143
63
63
U+0063
'c'
```

## 浮点数与复数

| 占位符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| %b     | 无小数部分的，指数为二的幂的科学计数法，与 strconv.FormatFloat 的 'b' 转换格式一致。例如 -123456p-78 |
| %e     | 科学计数法，例如 -1234.456e+78                               |
| %E     | 科学计数法，例如 -1234.456E+78                               |
| %f     | 有小数点而无指数，例如 123.456                               |
| %g     | 根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的0）输出        |
| %G     | 根据情况选择 %E 或 %f 以产生更紧凑的（无末尾的0）输出        |

测试代码：

```go
package main

import "fmt"

func main() {
	f := 3.1415926
	fmt.Printf("%b\n", f)
	fmt.Printf("%e\n", f)
	fmt.Printf("%E\n", f)
	fmt.Printf("%f\n", f)
	fmt.Printf("%g\n", f)
	fmt.Printf("%G\n", f)
}
// output:
7074237631354954p-51
3.141593e+00
3.141593E+00
3.141593
3.1415926
3.1415926
```
### 宽度标识符

| 占位符 | 说明                                           |
| ------ | ---------------------------------------------- |
| %f     | 和上面的一样（有小数点而无指数，例如 123.456） |
| %9f    | 宽度9，默认精度                                |
| %.2f   | 默认宽度，精度2                                |
| %9.2f  | 宽度9，精度2                                   |
| %9.f   | 宽度9，精度0                                   |

这里的默认宽度和默认精度，我猜都是根据科学记数法不要指数部分。

测试代码：

```go
package main

import "fmt"

func main() {
	f := 3.1415926
	fmt.Printf("%e\n", f)
	fmt.Printf("%f\n", f)
	fmt.Printf("%9f\n", f)
	fmt.Printf("%.2f\n", f)
	fmt.Printf("%9.2f\n", f)
	fmt.Printf("%9.f\n", f)
}
// output:
3.141593e+00
3.141593
 3.141593
3.14
     3.14
        3
```


## 字符串与[]byte

| 占位符 | 说明                                                     |
| ------ | -------------------------------------------------------- |
| %s     | 直接输出字符串或[]byte                                   |
| %q     | 该值对应的双引号括起来的go语法字符串字面值，采用安全转义 |
| %x     | 每个字节用两个字符十六进制表示 a-f                       |
| %X     | 每个字节用两个字符十六进制表示 A-F                       |

测试代码：

```go
package main

import "fmt"

func main() {
	s := "中国"
	b := []byte(s)
	fmt.Println(b)
	fmt.Printf("%s，%s\n", s, b)
	fmt.Printf("%q，%q\n", s, b)
	fmt.Printf("%x，%x\n", s, b)
	fmt.Printf("%X，%X\n", s, b)
}
// output:
[228 184 173 229 155 189]
中国，中国
"中国"，"中国"
e4b8ade59bbd，e4b8ade59bbd
E4B8ADE59BBD，E4B8ADE59BBD
```

## 指针

| 占位符 | 说明                    |
| ------ | ----------------------- |
| %p     | 十六进制表示，前缀 0x   |
| %#p    | 十六进制表示，无前缀 0x |

测试代码：

```go
package main

import "fmt"

func main() {
	a := 12
	ptr := &a
	fmt.Printf("%p\n", ptr)
	fmt.Printf("%#p\n", ptr)
}
// output:
0xc00000a0f8
c00000a0f8
```

## 其他

| 占位符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| +      | 总打印数值的正负号；对于%q（%+q）保证只输出ASCII编码的字符。 |
| -      | 在右侧而非左侧填充空格（左对齐该区域）                       |
| #      | 备用格式：为八进制添加前导 0（%#o）                          |
| ' '    | (空格)为数值中省略的正负号留出空白（% d）；以十六进制（% x, % X）打印字符串或切片时，在字节之间用空格隔开 |
| 0      | 填充前导的0而非空格；对于数字，这会将填充移到正负号之后      |

这些占位符，是结合上面的去使用的（%+ %#），比较灵活，这里就不一一列举了：

```go
package main

import "fmt"

func main() {
	fmt.Printf("%+q\n", "中国")
	fmt.Printf("%-9.2f\n", 3.1415926)
	fmt.Printf("%#o\n", 17)
	fmt.Printf("% X\n", "中国")
	fmt.Printf("%09.2f\n", 3.1415926)
}
// output:
"\u4e2d\u56fd"
3.14     
021
E4 B8 AD E5 9B BD
000003.14
```

最后有个疑问：

```
+	always print a sign for numeric values;
	guarantee ASCII-only output for %q (%+q)
```

官方文档是这么描述 + 占位符的。

我打印“中国”，输出的是 ”\u4e2d\u56fd“，为什么官方说的是，保证输出ASCII码呢（guarantee ASCII-only output）？

我理解的ASCII 吗不就是 128 个，扩展了也就 256个。