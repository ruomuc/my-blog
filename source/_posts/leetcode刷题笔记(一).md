---
title: leetcode刷题笔记(一)
categories:
  - 数据结构与算法
tags:
  - 算法刷题笔记
date: 2021-04-18 11:26:27
updated: 2021-04-18 11:26:32
cover: https://proxy.qnoss.seeln.com/images/X3PIEBb-black-hole-wallpaper.jpg
---

准备写一个系列，不是题解（~~力扣里的题解有很多，我写的太烂了~~），只是把平时刷题遇到的一些相似题拿出来记一下，这样更容易记忆和理解。

这次主要有四道题：

- [删除有序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/) - 简单
- [删除有序数组中的重复项 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array-ii/) - 中等
- [删除字符串中的所有相邻重复项](https://leetcode-cn.com/problems/remove-all-adjacent-duplicates-in-string/) - 简单
- [删除字符串中的所有相邻重复项 II](https://leetcode-cn.com/problems/remove-all-adjacent-duplicates-in-string-ii/) - 中等

<a herf="https://github.com/ruomuc/test_demos/tree/master/blog/leetcode%E5%88%B7%E9%A2%98%E7%AC%94%E8%AE%B0(%E4%B8%80)">本文代码github仓库地址</a>
<!--more-->

---

**删除有序数组中的重复项**

由题意可知：

- 输入为一个**有序**数组，并且已经按**升序排列**
- 新数组每个元素**只出现一次**
- 原地修改数组并且使用 **O(1) 额外空间**完成
- 返回值为新数组的长度
- 无需考虑数组中超过新数组长度的元素

双指针解法：

```go
//...
func removeDuplicates(nums []int) int {
	n := len(nums)
	if n == 0 {
		return 0
	}
	pre := 1
	for cur := 1; cur < n; cur++ {
		if nums[cur] != nums[pre-1] {
			nums[pre] = nums[cur]
			pre++
		}
	}
	return pre
}
```



**删除有序数组中的重复项II**

和上一题的区别就是，新数组允许每个元素**最多出现两次**

```go
//...
func removeDuplicates(nums []int) int {
	n := len(nums)
	if n <= 2 {
		return n
	}

	pre := 2
	for cur := 2; cur < n; cur++ {
		if nums[pre-2] != nums[cur] {
			nums[pre] = nums[cur]
			pre++
		}
	}
	return pre
}
```

我们发现，删除有序数组中的重复项两题解法其实是相同的。。

---

**删除字符串中的所有相邻重复项**

分析题意，我们需要删除一个字符串中，相邻并且相同的项，直到没有重复的相邻项为止。

我们可以使用栈来解题：

```go
//...
func removeDuplicates(s string) string {
	stack := make([]byte, 0)
	for i := range s {
		if len(stack) > 0 && stack[len(stack)-1] == s[i] {
			stack = stack[:len(stack)-1]
		} else {
			stack = append(stack, s[i])
		}
	}
	return string(stack)
}
```

**删除字符串中的所有相邻重复项 II**

和上题的区别是，新字符串允许有指定 k 个相邻且相等的字母。

是不是可前面的数组两题一个套路？

使用两个栈，一个记录字符，一个记录栈顶字符出现的次数。（~~不过实际写起来遇到很多坑~~）

```go
//...
func removeDuplicates(s string, k int) string {
	// 用两个栈，一个记录字符，一个记录字符出现的次数
	stack := make([]byte, 0)
	count := make([]int, 0)

	for i := range s {
		if i == 0 || len(stack) == 0 || s[i] != stack[len(stack)-1] {
			count = append(count, 1)
			stack = append(stack, s[i])
		} else {
			countTop := count[len(count)-1]
			if countTop+1 == k {
				stack = stack[:len(stack)-countTop]
				count = count[:len(count)-1]
			} else {
				count[len(count)-1] = countTop + 1
				stack = append(stack, s[i])
			}
		}
	}
	return string(stack)
}

```
