---
title: go的math/rand标准库
categories:
  - go
tags:
  - go
date: 2021-06-22 23:15:45
updated: 2021-06-22 23:15:45

---

go语言中 math/rand 包来提供随机数相关功能。

## 基本用法

为了方便测试，我们新建一个 time_test.go 文件
<!--more-->

执行以下代码：

```go
func TestTime(t *testing.T) {
	for i := 0; i < 5; i++ {
		fmt.Println(rand.Int())
	}
}
// output:
5577006791947779410
8674665223082153551
6129484611666145821
```

我们发现，我们无论执行多少次，输出的结果总是一样的。。

查找标准库文档，我们发现我们可以修改 seed 来获取不一样的结果

```go
// func Seed(seed int64) { globalRand.Seed(seed) }
func TestTime(t *testing.T) {
	rand.Seed(2)
	for i := 0; i < 3; i++ {
		fmt.Println(rand.Int())
	}
}
// output:
1543039099823358511
2444694468985893231
474893212811123542
```

我们通过 rand.Seed() 来设置一个 seed，seed只要是一个 int64 类型的参数就可以。(ps: 默认使用的seed 是 1)

虽然我们多次执行，输出任然没有变化，但是至少已经和上一段程序不一样了。

我们可以尝试多种不同的值。

这时候我会感到很疑惑？

1. 为什么程序每次执行输出的随机数都是一样的？
   - 这个再下文的伪随机会说到
2. 既然修改 seed ，可以得到不同的随机数，那为什么不每次调用 `rand.Int()` 前都调用一次 `rand.Seed(x)`呢？
   - 个人认为没有意义，还有可能会引起性能问题，哪怕是这么做了，也不能改变伪随机的事实

## 伪随机

通过前文发现，产生随机数的核心方法好像在于  rand.Seed() 方法。

我们查看 rand.Seed() 源码，核心差不多就是这些：

生成：

```go
const (
	rngLen   = 607
	rngTap   = 273
)

// Seed uses the provided seed value to initialize the generator to a deterministic state.
func (rng *rngSource) Seed(seed int64) {
	rng.tap = 0
	rng.feed = rngLen - rngTap

	seed = seed % int32max
	if seed < 0 {
		seed += int32max
	}
	if seed == 0 {
		seed = 89482311
	}

	x := int32(seed)
	for i := -20; i < rngLen; i++ {
		x = seedrand(x)
		if i >= 0 {
			var u int64
			u = int64(x) << 40
			x = seedrand(x)
			u ^= int64(x) << 20
			x = seedrand(x)
			u ^= int64(x)
			u ^= rngCooked[i]
			rng.vec[i] = u
		}
	}
}
```

获取：

```go
// Uint64 returns a non-negative pseudo-random 64-bit integer as an uint64.
func (rng *rngSource) Uint64() uint64 {
	rng.tap--
	if rng.tap < 0 {
		rng.tap += rngLen
	}

	rng.feed--
	if rng.feed < 0 {
		rng.feed += rngLen
	}

	x := rng.vec[rng.feed] + rng.vec[rng.tap]
	rng.vec[rng.feed] = x
	return uint64(x)
}
```



看起来有些复杂。（~~完全看不懂 哈哈~~）

只用知道这里使用的是专业的数学方法，目的是给 的 607 个槽设置对应的值。 

这也导致了相同的 seed, 最终设置到 rng.vec 里面的值是相同的, 通过 Intn 取出的也是相同的值。

## 并发安全

查看源码，我们默认使用的 rand 对象是这样创建的：

```go
type lockedSource struct {
	lk  sync.Mutex
	src *rngSource
}

/*
 * Top-level convenience functions
 */
var globalRand = New(&lockedSource{src: NewSource(1).(*rngSource)})

```

创建出来的对象，我断点出来是这样的：

<img src="https://image.seeln.com/images/WX20210623-001621%402x.png" style="zoom:50%;" />

以 rand.Int() 为例，差不多是如下调用顺序：
```go
// Int returns a non-negative pseudo-random int.
func (r *Rand) Int() int {
	u := uint(r.Int63())
	return int(u << 1 >> 1) // clear sign bit if int == int32
}
```

```go
// Int63 returns a non-negative pseudo-random 63-bit integer as an int64.
func (r *Rand) Int63() int64 { return r.src.Int63() }
```

```go
func (r *lockedSource) Int63() (n int64) {
	r.lk.Lock()
	n = r.src.Int63()
	r.lk.Unlock()
	return
}
```

```go
// Int63 returns a non-negative pseudo-random 63-bit integer as an int64.
func (rng *rngSource) Int63() int64 {
	return int64(rng.Uint64() & rngMask)
}
```

```go
// Uint64 returns a non-negative pseudo-random 64-bit integer as an uint64.
func (rng *rngSource) Uint64() uint64 {
	rng.tap--
	if rng.tap < 0 {
		rng.tap += rngLen
	}

	rng.feed--
	if rng.feed < 0 {
		rng.feed += rngLen
	}

	x := rng.vec[rng.feed] + rng.vec[rng.tap]
	rng.vec[rng.feed] = x
	return uint64(x)
}
```

其中，lockedSource 的所有方法，都会有一个全局共享锁，所以可以保证并发安全。

当然我们也可以创建一个自定义的 rand 对象：

```go
func TestTime(t *testing.T) {
	r := rand.New(rand.NewSource(1))
	for i := 0; i < 3; i++ {
		fmt.Println(r.Int())
	}
}
```

当然，这样在并发场景下，会出现问题。



参考链接：

https://golang.org/pkg/math/rand/

https://gocn.vip/topics/11689



