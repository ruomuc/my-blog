---
title: 关于event-loop
categories:
  - node.js
tags:
  - node.js
date: 2021-04-19 23:50:53
updated: 2021-04-19 23:50:53
cover: https://proxy.qnoss.seeln.com/images/axfU4p2-black-hole-wallpaper.jpg
---

首先，js 不同宿主环境对 event loop 的实现是有区别的，这里说的都是 node.js 中的 event loop

关于 node.js 的 event loop ，之前写过两篇文章，最近回头看的时候，看不下去给删了

其实我的理解主要来源于：

- [node官方文档](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
- [cnode论坛一位大佬的文章](https://cnodejs.org/topic/5a9108d78d6e16e56bb80882)
- 由于cnode论坛挂过一次，并且部署在境外，如果上面的连接访问不了，可以[访问我复制的这个](https://github.com/ruomuc/test_demos/blob/master/blog/%E5%85%B3%E4%BA%8Eevent-loop/%E4%B8%8D%E8%A6%81%E6%B7%B7%E6%B7%86nodejs%E5%92%8C%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%AD%E7%9A%84event%20loop.md)

之前不过是照虎画猫，写的太拉胯了，这里就不在分析和探究原理了。。

这里主要是用代码来验证一下 event loop 的执行顺序，以便回顾知识的时候更加直观。

下面的代码都在[github仓库](https://github.com/ruomuc/test_demos/tree/master/blog/%E5%85%B3%E4%BA%8Eevent-loop)
<!--more-->

---

> setTimeout 和 setInterval 的执行顺序？

根据代码输出，可以知道，timers阶段的 setTimeout 和 setInterval 执行顺序是一个 FIFO 的队列。

```js
const time = 1000

setTimeout(() => {
  console.log('timeout1')
}, time)
const interval1 = setInterval(() => {
  console.log('interval1')
  clearInterval(interval1)
}, time)

setTimeout(() => {
  console.log('timeout2')
}, time)
const interval2 = setInterval(() => {
  console.log('interval2')
  clearInterval(interval2)
}, time)

// timeout1
// interval1
// timeout2
// interval2
```
---
> setTimeout 和 setImmediate ？

分析代码和输出：

- 运行多次发现，`immediate callback` 都在 `timeout callback` 先后顺序是随机的
- node.js里面`setTimeout(fn, 0)`会被强制改为`setTimeout(fn, 1)`
  - When `delay` is larger than `2147483647` or less than `1`, the `delay` will be set to `1`. Non-integer delays are truncated to an integer.
  -  [官方文档说明](https://nodejs.org/api/timers.html#timers_settimeout_callback_delay_args)
- 所以说它们的执行顺序，取决于同步代码的耗时，如果执行到timers时，刚好过去1ms，那么就会先输出 `timeout callback` ，反之

```js
setImmediate(function () {
  console.log('immediate callback')
})
setTimeout(function () {
  console.log('timeout callback')
}, 0)

// 输出
// immediate callback
// timeout callback
```
---
> poll 的 I/O callback 和 check 阶段的 setImmediate

分析代码和输出：

`open empty callback` 始终第一个输出，`open file callback` 和 `immediate callback` 先后顺序会变化

执行顺序取决于 fs.open(...) 的耗时。

```js
const fs = require('fs')

setImmediate(() => {
  console.log('immediate callback')
})
fs.open('', 1, function () {
  console.log('open empty callback')
})

fs.open('./不要混淆nodejs和浏览器中的event loop', 1, function () {
  console.log('open file callback')
})
```

输出：

```js
open empty callback
immediate callback
open file callback
// 或
open empty callback
open file callback
immediate callback
```

---

> close callbacks 阶段和 check 阶段

这个执行顺序没什么好说的，和文档描述的一致。

```js
const fs = require('fs')

const readStream = fs.createReadStream(
  __dirname + '/不要混淆nodejs和浏览器中的event loop.md'
)
readStream.close()
readStream.on('close', function () {
  console.log('close callback')
})

setImmediate(function () {
  console.log('immediate callback')
})

// immediate callback
// close callback
```

---

> process.nextTick()

上面大佬的文章中从源码层面分析了 process.nextTick() 的执行时机， 即每个阶段结束都会执行。

可以看到输出中，每个 process.nextTick() 都是在该阶段结束立即执行。

```js
const fs = require('fs')

setTimeout(() => {
  console.log('timeout callback')
  process.nextTick(function () {
    console.log('timeout tick')
  })
}, 0)

setImmediate(function () {
  console.log('immediate callback')
  process.nextTick(function () {
    console.log('immediate tick')
  })
})

const readStream = fs.createReadStream(
  __dirname + '/不要混淆nodejs和浏览器中的event loop.md'
)
readStream.close()
readStream.on('close', function () {
  console.log('close callback')
  process.nextTick(function(){
    console.log('close callback tick')
  })
})

fs.open('', 1, function () {
  console.log('open empty callback')
  process.nextTick(function () {
    console.log('open empty tick')
  })
})

```

输出：

```js
timeout callback
timeout tick
open empty callback
open empty tick
immediate callback
immediate tick
close callback
close callback tick
```

---

> microtasks 和 process.nextTick()

分析一下：

- nextTick 和 microtasks 都是一个 FIFO 的队列
- microtasks 会在执行完 nextTick 之后执行

```js
const fs = require('fs')

process.nextTick(function () {
  console.log('nextTick')
  new Promise(function (resolve) {
    resolve()
  }).then(function () {
    console.log('microtasks2')
  })
})

new Promise(function (resolve) {
  resolve()
}).then(function () {
  console.log('microtasks')
})

process.nextTick(function () {
  console.log('nextTick2')
  new Promise(function (resolve) {
    resolve()
  }).then(function () {
    console.log('microtasks3')
  })
})
fs.open('', 1, function () {
  console.log('====我是分界线====')
})
```

输出：

```js
nextTick
nextTick2
microtasks
microtasks2
microtasks3
====我是分界线====
```

---

> 制造死循环

下面两段代码都可以制造死循环。。

```js
const fs = require('fs')
function nextTick () {
  process.nextTick(function () {
    nextTick()
  })
}
nextTick()
fs.open('', 1, function () {
  console.log('====永远都不会轮到我====')
})
```

和

```js
const fs = require('fs')
function runMicrotask(){
  new Promise(function (resolve) {
    resolve()
  }).then(function () {
    runMicrotask()
  })
}
runMicrotask()

fs.open('', 1, function () {
  console.log('====永远都不会轮到我====')
})
```

