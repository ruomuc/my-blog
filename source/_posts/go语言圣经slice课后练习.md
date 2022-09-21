---
title: go程序设计语言-slice课后练习
categories:
  - go
tags:
  - go
date: 2021-05-17 21:30:47
updated: 2021-05-17 21:30:47
cover: https://proxy.qnoss.seeln.com/images/wp4221866-asteroid-belt-wallpapers.jpg
---

本来想写一片关于 slice 切片的文章，但是又复习了一下《go程序设计语言》，并且网上类似的东西数不胜数。。

于是想着把课后练习题做了吧，(ps：当时看第一遍的时候不太明白所以没做)。

本文代码可以直接在 playground 中运行。

<!--more-->

## 练习 4.3: 
> 重写reverse函数，使用数组指针代替slice。

<br>

```go
package main

import (
	"fmt"
)

func main() {
	nums:=[10]int{1,2,3,4,5,6,7,8,9,10}
	Reverse(&nums)
	fmt.Println(nums) // [10 9 8 7 6 5 4 3 2 1]
}

func Reverse(nums *[10]int) {
	for i, j := 0, len(nums)-1; i < j; i,j = i+1,j-1 {
		nums[i], nums[j] = nums[j], nums[i]
	}
}
```

Tips: 在 golang 中，建立了 `arrPtr := &arr` 这种类似地址关系后，* 允许不写。

所以等同于：

```go
package main

import (
	"fmt"
)

func main() {
	nums:=[10]int{1,2,3,4,5,6,7,8,9,10}
	Reverse(&nums)
	fmt.Println(nums) // [10 9 8 7 6 5 4 3 2 1]
}

func Reverse(numPtr *[10]int) {
	for i, j := 0, len(*numPtr)-1; i < j; i,j = i+1,j-1 {
		(*numPtr)[i], (*numPtr)[j] = (*numPtr)[j], (*numPtr)[i]
	}
}
```

为什么要用 () 吧 *numPtr 括起来呢。

因为在 **Golang** 中 * 寻址运算符和 [] 中括号运算符的优先级是不同的！

- [] 中括号是**初等运算符**
- 寻址运算符是**单目运算符**

初等运算符的优先级是大于单目运算符的，因此先参与计算的是 arrPtr[0]，arrPtr[0] 其实就是数组的第一个元素，就是数字 1。
数字 1 必然是 int 类型，而不是一个地址，因此针对数字 1 使用 * 寻址运算符自然也就发生了错误。

## 练习 4.4: 

> 编写一个rotate函数，通过一次循环完成镟转。

<br>

说让通过一次循环翻转，没说要原地翻转。

```go
package main

import (
	"fmt"
)

func main() {
	nums := []int{1,2,3,4,5,6,7,8,9,10}
	fmt.Println(Rotate(nums, 2)) // [3 4 5 6 7 8 9 10 1 2]
}


func Rotate(nums []int, r int) []int{
	n := len(nums)
  // 声明一个切片,用于存储旋转后的结果
	ans := make([]int, n, n)
	for i := 0; i < n; i++ {
		index := i+r
		if index >= n {
			index = index - n
		}
		ans[i] = nums[index] // 这个是向左旋转
		// ans[index] = nums[i] // 这个是向右旋转
	}
	return ans
}
```

## 练习 4.5: 

> 写一个函数在原地完成消除[]string中相邻重复的字符串的操作。

<br>

关键是原地，不开辟额外空间。

```go
package main

import (
	"fmt"
)

func main() {
	strs := []string{"a", "a","a", "b", "b", "c"}
	fmt.Println(Clear(strs)) // [a b c]
}

func Clear(strs []string) []string{
	for i, j := 0, 1; j < len(strs)-1; i, j = i+1 ,j+1 {
		if strs[i] == strs[j] {
			copy(strs[j:],strs[j+1:])
			strs = strs[:len(strs)-1]
			i--
			j--
		}
	}
	return strs
}
```

## 练习 4.6: 

> 编写一个函数，原地将一个UTF-8编码的[]byte类型的slice中相邻的空格(参考 unicode.IsSpace)替换成一个空格返回

<br>

空格的 unicode 码是 32。 

和上一题没有区别。只是多了一个 unicode.IsSpace() 的判断。

```go
package main

import (
	"fmt"
	"unicode"
)

func main() {
	strs := []byte{228, 184, 173, 32, 32, 33, 33, 229, 155, 189 } // 中<空格><空格>!!国
	fmt.Println(string(clear(strs)))                                                    // 中<空格>!!国
}

func clear(strList []byte) []byte {
	for i, j := 0, 1; i < len(strList)-1; i, j = i+1, j+1 {
		if strList[i] == strList[j] && unicode.IsSpace(rune(strList[i])) {
			copy(strList[j:], strList[j+1:])
			strList = strList[:len(strList)-1]
			i--
			j--
		}
	}
	return strList
}

```

## 练习 4.7: 

> 修改reverse函数用于原地反转UTF-8编码的[]byte。是否可以不用分配额外的内存?


```go
package main

import (
	"fmt"
)

func main() {
	strs := "asadqweq"
	fmt.Println(string(reverse([]byte(strs))))
}

func reverse(strs []byte) []byte {
	for i, j := 0, len(strs)-1; i < j; i, j = i+1, j-1 {
		strs[i], strs[j] = strs[j], strs[i]
	}
	return strs
}

```

---
2022年2月3日17:59:16：

可以在这里查看 The Go Programming Language 其它的课后习题答案代码：

https://github.com/ruomuc/go-programming-language