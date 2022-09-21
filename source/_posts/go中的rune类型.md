---
title: go中的rune类型
categories:
  - go
tags:
  - go
date: 2021-05-01 20:09:03
updated: 2021-05-01 20:09:03

---

在 java 等语言中，有char类型，而在go语言中，使用的是 rune 类型。

char 类型在 java中是16位的，因为用的是unicode编码。

而 rune 类型在 go中是32位的，因为go用的是utf-8编码。

可以在这个网站查看码点：[UTF-8 encoding table and Unicode characters](https://utf8-chartable.de/unicode-utf8-table.pl?start=19840&names=-&utf8=dec)
<!--more-->
## 什么是rune

源码中，rune是这样定义的：

rune 是int32的别名，在所有方面都等同于int32

```go
// rune is an alias for int32 and is equivalent to int32 in all ways. It is
// used, by convention, to distinguish character values from integer values.
type rune = int32
```

我们做个测试：

```go
package main
import (
	"fmt"
)
func main() {
	fmt.Println(len("z中国")) // 7
	fmt.Println(len("zhong")) // 5
}
```

因为 ”中“ 和 ”国“ 两个字的unicode码 ，都在utf-8 的第三个区段 `U+ 0800 ~ U+  FFFF` ，所以都占三个字节，长度就是 7。

而且由于编码的原因，我们不知道字符串应该按照几个字节处理，如果处理不当，就会出现乱码：

```go
package main

import (
	"fmt"
)

func main() {
	var str = "a中国"
	fmt.Println(str[:3]) // a��
}
```

所以，在 go 语言中，使用 range 来遍历字符串，会自动按照 rune 类型去处理，很方便

或者你可以使用 `[]rune(str)` 将字符串转为 rune 数组类型，然后使用下标遍历。。

```go
package main

import (
	"fmt"
)

func main() {
	var str = "a中国"
	for _, s := range str {
		fmt.Println(s)
	}
    fmt.Println([]rune(str))
}
// output:
97
20013
22269
[97 20013 22269]
```



## []rune 和 []byte的区别

看下源码对 byte 的定义：

```go
// byte is an alias for uint8 and is equivalent to uint8 in all ways. It is
// used, by convention, to distinguish byte values from 8-bit unsigned
// integer values.
type byte = uint8
```
和 rune 差不多，只不过是 uint8 类型的别名

使用 []byte遍历一下字符串，看看有什么不同：
```go
package main

import (
	"fmt"
)

func main() {
	var str = "a中国"
	
	for _, s := range []byte(str) {
		fmt.Println(s)
	} 
	fmt.Println([]byte(str))
	
}
// output:
97
228
184
173
229
155
189
[97 228 184 173 229 155 189]
```

去文章开头放的链接里查一下 “中” 和 “国” 这两个字的码点如下：

| Unicode<br/>code point (hex) | Unicode<br/>code point (octal) | character | UTF-8<br/>(dec.) |
| -------------------------------- | ---------------------------------------------- | ------------- | ------------------------ |
| U+4E2D                           | 20013                                          | 中            | 228 184 173              |
| U+56FD                           | 22269                                          | 国            | 229 155 189              |

这样一看，是不是全部都对上了。。



