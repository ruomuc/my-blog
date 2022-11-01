---
title: 深入理解nodejs内存分配
categories:
  - node.js
tags:
  - 内存
date: 2021-12-25 11:26:22
updated: 2021-12-25 11:26:22


---

因为最近遇到很多关于内存的问题，所以决定再次探究回顾一下 node.js 相关的内存知识。

再次删除了一些之前的关于node.js内存的博文，因为在现在看来那些写的太垃圾了。

## 内存的生命周期

- 内存分配：由 JavaScript 内部为我们分配内存。
- 内存使用：由代码来读取内存的数据，或向内存写入数据（比如赋值语句）。
- 内存释放：由 JavaScript 引擎来进行，内存释放后，可以被 JavaScript 重新分配。

![](https://blog-1301153828.cos.ap-shanghai.myqcloud.com/node.js内存的生命周期.png)

## 栈（Stack）和堆（Heap）
### 堆栈和堆的概念

在数据结构中：

> 栈的结构比较好理解，一种线性结构，限制：只允许一端出入。所以栈的特点是**LIFO(Last In First Out)后进先出**。
>
> 堆在数据结构中是一种特殊的完全二叉树，所以是一种树状结构。

在计算机内存分配中：

> **堆（heap）和栈（stack）是两种内在的管理形式**。
>
> 它们的主要区别是stack按次序排放，大小明确；heap结构则不固定，是一种可动态分配和释放的内存。单从这一点看，stack的寻址速度要比heap快，heap的灵活性则比较高。一般来说，每个线程分配一个stack，每个进程分配一个heap。

### 栈内存分配

栈是 JavaScript 用来存储静态数据的数据结构。

静态数据就是引擎在编译时就知道大小的数据。

在程序运行前分配内存的过程称为**静态内存分配**。

![](https://blog-1301153828.cos.ap-shanghai.myqcloud.com/nodejs栈内存.png)
<!--more-->
原始值在进行相互赋值时，内存是如何分配的呢？

可以发现，变量指向的还是一个内存地址，由于栈内存中存储的原始值是不可变的，所以对 nextAge 进行 +1 操作时，会重新分配一个地址存储新的值，并且把 nextAge 指向新的地址。

```js
let age = 25
let nextAge = age
nextAge = nextAge + 1
```

![](https://blog-1301153828.cos.ap-shanghai.myqcloud.com/nodejs栈内存分配2.png)

### 堆内存分配

ps : 堆内存 和 数据结构里的堆，是没有任何关系的。。只是 Heap 这个单词背沿用了下来。参见：[堆内存和数据结构堆之间的关系是什么？ - 詹姆斯.通的回答 - 知乎]( https://www.zhihu.com/question/276016774/answer/385381844)

---

堆是 JavaScript 用来存储 对象和函数等数据类型的空间。

JavaScript 在编译期间无法知道它们所需内存大小，所以只有在运行时进行内存分配。

在程序运行时分配内存的过程称为**动态内存分配**。

![](https://blog-1301153828.cos.ap-shanghai.myqcloud.com/nodejs堆内存分配.png)

并且如果修改了 newCat 的 name 字段。 cat 的 name 字段也会改变：

```js
const cat = {
  id: 1,
  name: 'mimi',
  age: 5
}

function getName(target) {
  return target.name
}

const newCat = cat
newCat.name = 'miaomiao'
console.log(getName(cat)) // miaomiao
```

### 堆栈溢出

JavaScript 中使用没有终止条件的递归调用，会产生堆栈溢出的报错。

```js
function recurse () {
  recurse()
}

recurse()

// RangeError: Maximum call stack size exceeded
//     at recurse (/Users/zhangming/ownspace/local-test/test9/index.js:17:13)
//     at recurse (/Users/zhangming/ownspace/local-test/test9/index.js:18:3)
//     at recurse (/Users/zhangming/ownspace/local-test/test9/index.js:18:3)
//     at recurse (/Users/zhangming/ownspace/local-test/test9/index.js:18:3)
//     at recurse (/Users/zhangming/ownspace/local-test/test9/index.js:18:3)
//     at recurse (/Users/zhangming/ownspace/local-test/test9/index.js:18:3)
//     at recurse (/Users/zhangming/ownspace/local-test/test9/index.js:18:3)
//     at recurse (/Users/zhangming/ownspace/local-test/test9/index.js:18:3)
//     at recurse (/Users/zhangming/ownspace/local-test/test9/index.js:18:3)
//     at recurse (/Users/zhangming/ownspace/local-test/test9/index.js:18:3)
```

但是这里的堆栈，指的是 调用堆栈（call stack），不一定和上文的堆栈使用同一内存空间。

### OOM（out of memory）

这个报错可以说时很常见了。

因为本地默认的堆内存比较大，要使用`node --max-old-space-size=10 index.js`命令可以更快得到 oom 错误信息。

```js
const { memoryUsage } = require('process')


function oom () {
  const obj = {}
  for (let i = 0; i < 10000000000; i++) {
    obj[i] = { [i * i]: i * i}
  }
}

oom()
```

日志太多了，截取一部分：

```
<--- Last few GCs --->

[11306:0x1048f3000]     1098 ms: Mark-sweep 4.9 (8.3) -> 3.9 (8.3) MB, 3.4 / 0.0 ms  (average mu = 0.932, current mu = 0.932) allocation failure scavenge might not succeed
[11306:0x1048f3000]     1147 ms: Mark-sweep 5.0 (8.3) -> 4.0 (8.6) MB, 2.3 / 0.0 ms  (average mu = 0.943, current mu = 0.954) allocation failure scavenge might not succeed


<--- JS stacktrace --->

FATAL ERROR: MarkCompactCollector: young object promotion failed Allocation failed - JavaScript heap out of memory
 1: 0x10130d6e5 node::Abort() (.cold.1) [/Users/zhangming/.nvm/versions/node/v14.17.0/bin/node]
 2: 0x1000b1c49 node::Abort() [/Users/zhangming/.nvm/versions/node/v14.17.0/bin/node]
 3: 0x1000b1daf node::OnFatalError(char const*, char const*) 
```

## 大对象内存占用

### 如何查看对象的内存占用

- [object-sizeof](https://www.npmjs.com/package/object-sizeof)
- chrome 的内存快照

object-sizeof 是根据 [ECMAScript标准](https://262.ecma-international.org/5.1/#sec-4.3.16)，对对象大小的一个估算，一般会比 chrome 的内存快照大一些，因为引擎在最终存储时可能会做一些优化。

- String 2字节
- Boolean 4字节
- Number 8字节

举例：

```js
const sizeof = require('object-sizeof')

const cat = {
  id: 1,
  name: 'mimi',
  age: 5,
  isMale: true
}

console.log(sizeof(cat)) // 58
```



### JSON和对象

#### 区别

- JSON（JavaScript Object Notation）虽然全程含有 JavaScript，但不是只有 JavaScript 可以使用，JSON 是 JavaScript 的一个子集。
- JSON 是一种轻量级的资料交换格式。 类似的还有 protobuf 等。
- Object 是 JavaScript 的一种 [数据类型](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures) 。它用于存储各种键值集合和更复杂的实体。
- 在 JavaScript 中 json 数据可以很简单的转化为 object ，当然其它语言使用对应的解析器也可以做到。

#### 互相转换

JavaScript 内置了 `JSON.parse()` 和 `JSON.stringify()` 来进行 JSON 和 Object 的相互转换。

json方法的实现：

```js
global.JSON.parse.toString()
//  function parse() { [native code] }

global.JSON.stringify.toString()
// function stringify() { [native code] }
```

关于 native code：

![](https://blog-1301153828.cos.ap-shanghai.myqcloud.com/WX20211226-125734.png)

json源码的位置：deps/v8/src/json-parser.cc

### 内存优化

上面可以看到，一个json对象，转换成JavaScript对象后，占用内存会几倍的增大。

所以开发中会遇到 JSON.parse 一个大对象后，内存容易OOM。

这里有两个简单的节省内存的方法：

第一种，减少key的数量

```js
[
  {a: 1, b: 2 },
  {a: 3, b: 4}
]

// 转换为:
[
  [a, b],
  [1, 2],
  [3, 4]
]
```

第二种，大json使用分隔符分隔：

```js
[
  {a: 1, b: 2},
  {a: 3, b: 4}
]

// 转化为一个文件
{a: 1, b: 2}\n{a:3, b:4}
```

因为解析小的json速度会更快：

```js
const a = '[{"a":1,"b":2},{"a":3,"b":4}]'
console.time('big')
JSON.parse(a)
console.timeEnd('big') // 0.315ms

const a1 = '{"a":1,"b":2}'
const a2 = '{"a":3,"b":4}'
console.time('small')
JSON.parse(a1)
JSON.parse(a2)
console.timeEnd('small') // 0.01ms
```



## 垃圾回收

垃圾回收前面已经讲过很多了，参见：

[V8的垃圾回收机制和内存限制](https://www.seeln.com/2018/07/31/V8%E7%9A%84%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6%E5%92%8C%E5%86%85%E5%AD%98%E9%99%90%E5%88%B6/#V8%E7%9A%84%E5%86%85%E5%AD%98%E9%99%90%E5%88%B6)

[golang的gc](https://www.seeln.com/2021/05/15/golang%E7%9A%84gc/)

