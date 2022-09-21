---
title: golang中的排序
categories:
  - go
tags:
  - go
  - 排序算法
date: 2020-11-06 21:55:41
updated: 2020-11-06 22:55:41

---

今天力扣每日一题是[根据数字二进制下 1 的数目排序](https://leetcode-cn.com/problems/sort-integers-by-the-number-of-1-bits/)，题目本身是简单，但是我用`golang`实现的时候遇到了排序的问题。

## javascript的实现

```js
var sortByBits = function (arr) {
  let countBit = function (num) {
    let count = 0
    while (num != 0) {
      if (num % 2 === 1) {
        count++
      }
      num = Math.floor(num / 2)
    }
    return count
  }
  return arr.sort((a, b) => {
    return countBit(a) - countBit(b) || a - b
  })
}
```

很简单，js的[sort](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)方法的参数是一个可以自定义的函数。

## golang的sort

golang有一个[sort包](https://studygolang.com/pkgdoc)，提供了多种类型的排序方式，今天我这里只看`func Sort(data Interface)`

它必须实现`sort.Interface`的三个方法:
<!--more-->

```go
type Interface interface {
    // Len方法返回集合中的元素个数
    Len() int
    // 如果 i 索引的数据小于 j 索引的数据，返回 true，且不会调用下面的 Swap()，即数据升序排序。
    Less(i, j int) bool
    // Swap方法交换索引i和j的两个元素
    Swap(i, j int)
}
```

根据官方文档的案例，一般情况下我们不需要修改`Len`和`Swap`方法，想要自定义排序规则达到`JavaScript`的`sort`方法哪种效果，需要修改`Less`方法。

官网有的东西我就不搬了，直接上代码看我怎么解决`bitcount`那道题的：

```go
// ...
// Bits is
type Bits []int
// Len is
func (b Bits) Len() int { return len(b) } // 基本不会变化
// Swap is
func (b Bits) Swap(i, j int) { b[i], b[j] = b[j], b[i] } // 基本不会变化
// Less is
func (b Bits) Less(i, j int) bool {
	if bitCount(b[i]) < bitCount(b[j]) {
		return true // 如果b[i]的二进制1的数量小于b[j]的二进制1的数量，i应该在j前面，返回true
	} else if bitCount(b[i]) == bitCount(b[j]) {
		return b[i] < b[j] // 如果二进制1的数量相等，i还是应该在j前面，返回true
	} else {
		return false // 否则返回false，调用Swap交换位置
	}
}

func bitCount(num int) int {
	count := 0
	for num != 0 {
		if num%2 == 1 {
			count++
		}
		num = int(math.Floor(float64(num / 2)))
	}
	return count
}

func sortByBits(arr []int) []int {
	sort.Sort(Bits(arr))
	return arr
}
```

如果你需要数据需要降序而不是升序，你应该使用`func Reverse(data Interface) Interface`，而不是修改`Less`方法：

```go
func sortByBits(arr []int) []int {
	sort.Sort(sort.Reverse(Bits(arr)))
	return arr
}
```

