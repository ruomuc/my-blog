---
title: WeekMap和WeekSet
categories:
  - JavaScript
tags:
  - JavaScript
  - gc
date: 2021-04-10 22:46:07
updated: 2021-04-10 22:46:07

---

在es6中，不仅引入了 Map 和 Set， 还有 WeekMap和WeekSet，不过我没怎么用过。

首先WeekMap 和 WeekSet 的键只能是对象；其次垃圾回收不会考虑它们对对象的引用，本文主要分析它们对于 gc 的区别。

测试代码地址:[https://github.com/ruomuc/test_demos/tree/master/blog/WeekMap%E5%92%8CWeekSet](https://github.com/ruomuc/test_demos/tree/master/blog/WeekMap%E5%92%8CWeekSet)
<!--more-->

首先分析一下Map，看这样一段代码：

```js
const { printHeap } = require('./util')
const map = new Map()

function testMap () {
  for (let i = 0; i < 1000000; i++) {
    const obj = { [`key_${i}`]: i }
    map.set(obj, i)
  }
}

printHeap('tesMap begin--')
testMap()
printHeap('tesMap end--')
global.gc()
printHeap('testMap clear--')
```

使用`node --expose-gc map.js` 启动，该参数表示允许我们手动调用 gc，一般只在测试时候使用。。不推荐在生产代码使用。

分析日志：

```js
=============tesMap begin--===========================
     rss: 18M
     heapTotal: 4M
     heapUsed: 2M
     external:0M
     arrayBuffers:0M
=============tesMap end--===========================
     rss: 351M
     heapTotal: 327M
     heapUsed: 298M
     external:0M
     arrayBuffers:0M
=============testMap clear--===========================
     rss: 349M
     heapTotal: 306M
     heapUsed: 252M
     external:0M
     arrayBuffers:0M
```

可以看到，尽管 testMap 方法调用完成，其中的 obj 临时对象都失去引用，但是内存并没有降下来。

这是因为 map 中的键对生成的对象引用，gc 无法释放这些对象。

如果我们在调用 gc 前，手动清空 map：

```js
...//
printHeap('tesMap end--')
map.clear()
global.gc()
printHeap('testMap clear--')
```

日志如下：

```js
=============tesMap begin--===========================
     rss: 18M
     heapTotal: 4M
     heapUsed: 2M
     external:0M
     arrayBuffers:0M
=============tesMap end--===========================
     rss: 352M
     heapTotal: 328M
     heapUsed: 297M
     external:0M
     arrayBuffers:0M
=============testMap clear--===========================
     rss: 349M
     heapTotal: 53M
     heapUsed: 18M
     external:0M
     arrayBuffers:0M
```

可以到，内存被大量释放了。

---



然后我们看一下 WeekMap：

```js
const { printHeap } = require('./util')
const wkMap = new WeakMap()

function testWeekMap () {
  for (let i = 0; i < 1000000; i++) {
    const obj = { [`key_${i}`]: null }
    wkMap.set(obj, i)
  }
}

printHeap('tesWeekMap begin--')
testWeekMap()
printHeap('tesWeekMap end--')
global.gc()
printHeap('tesWeekMap clear--')
```

日志如下：

```js
=============tesWeekMap begin--===========================
     rss: 19M
     heapTotal: 4M
     heapUsed: 2M
     external:0M
     arrayBuffers:0M
=============tesWeekMap end--===========================
     rss: 106M
     heapTotal: 83M
     heapUsed: 64M
     external:0M
     arrayBuffers:0M
=============tesWeekMap clear--===========================
     rss: 102M
     heapTotal: 46M
     heapUsed: 10M
     external:0M
     arrayBuffers:0M
```

可以看到，内存确实也下降了，大家可以把 global.gc() 去掉跑一变做个对比。



emm... 但是我有两个问题没搞明白，如果有大佬看到了，并且知道的话，请留言区指教一下。。

> 1. 同样是 100w个对象，weekMap 内存占用就比较少？

我的猜测是 WeekMap 支持的方法没有 Map多，比如不支持迭代，所以实现比较简洁？

> 2. 为什么，对象失去引用 并且 gc 了之后， 内存占用没有恢复到一开始的状态？

想了一下，那几个变量应该用不了好几M 的内存，不知道还有些什么东西没释放掉呢。。

