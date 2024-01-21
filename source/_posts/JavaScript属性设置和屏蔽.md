---
title: JavaScript属性设置和屏蔽
categories:
  - JavaScript
tags:
  - JavaScript
  - 原型链
date: 2021-03-28 10:23:44
updated: 2021-03-28 10:23:44

---

本篇文章需要对 JavaScript 的**原型链**和**属性描述符**有一定的了解。

本篇文章主要分析一下，在 JavaScript 给对象设置属性或是修改对象的属性值背后的过程。

首先看这样一段代码，熟悉一下原型链：
<!--more-->

```js
var obj = {}
obj.a = 1
console.log(obj.a) // 1

var obj2 = {}
console.log(obj2.__proto__ === Object.prototype) // true
Object.prototype.a = 233
console.log(obj2.a) // 233
```

我们分析一下：

1. obj1 没什么好说的，给当前对象上设置一个属性 a，然后读取该值。
2. obj2 虽然没有属性 a，但是  js 会在 obj2 的原型链上查找 a 属性，然后输出  233。



**属性存在于原型链上而不存在于要赋值对象的三种情况**

**第一种情况：**

<font color ="brow">如果在**[[prototype]]**链上存在同名属性，并且没有被标记为只读 **writable:false**，创建屏蔽属性。</font>

我们思考一下，如果有一个  obj3 原型链上有一个属性 foo ，然后仍然我们给 obj3 设置一个同名属性 foo，会发生什么？
实践一下：

```js
var obj3 = {}
Object.prototype.foo = 'i am prototype foo'
obj3.foo = 'i am obj3 foo'
console.log(obj3.foo) // i am obj3 foo
console.log(Object.prototype.foo) // i am prototype foo
```

这样的输出是符合预期的，我们给 obj3 添加了一个 foo 属性，而不是修改 Object.prototype.foo 的值。

这种行为称之为**属性屏蔽**，obj3 的 foo 属性屏蔽了原型链上层的 foo 属性。

**第二种情况：**

<font color ="brow">如果在**[[prototype]]**链上存在同名属性，并且被被标记为只读 **writable:false** 。</font>

- <font color ="brow">非严格模式下：无法创建屏蔽属性，也无法修改链上的属性。</font>
- <font color ="brow">严格模式下，报错。</font>

非严格模式：

```js
var obj3 = {}
Object.defineProperty(Object.prototype, 'foo', {
  writable: false,
  value: 333
})
obj3.foo = 'i am obj3 foo' // 该语句被忽略
console.log(obj3.foo) // 333
console.log(Object.prototype.foo) // 333
```

严格模式：

```js
'use strict'
var obj3 = {}
Object.defineProperty(Object.prototype, 'foo', {
  writable: false,
  value: 333
})
obj3.foo = 'i am obj3 foo' // TypeError: Cannot assign to read only property 'foo' of object '#<Object>'
console.log(obj3.foo)
console.log(Object.prototype.foo)
```

**第三种情况：**

<font color ="brow">如果在**[[prototype]]**链上存在同名属性，并且它是一个**[[setter]]**，那么就会调用它。</font>

```js
var obj3 = {}
Object.defineProperty(Object.prototype, 'foo', {
  set: function () {
    console.log('i am a setter!')
  }
})
obj3.foo = 'i am obj3 foo' // i am a setter!
console.log(obj3.foo) // undefined
console.log(Object.prototype.foo) // undefined
```

可以到执行 obj3.foo 时，输出了 `i am a setter! `，并且也没有发生**属性屏蔽**。

可以看到，js 中设置一个属性并没有那个简单，除了第一种情况，剩下两种情况的行为都比较 "奇怪"，需要注意。

那么我们如何让 **属性屏蔽** 总是发生？

**使用Object.defineProperty来屏蔽属性**

```js
// 第二种情况
var obj = {}
Object.defineProperty(Object.prototype, 'foo1', {
  writable: false,
  value: 'Object.prototype.foo1'
})
Object.defineProperty(obj, 'foo1', {
  value: 'obj.foo1'
})
console.log(obj.foo1) // obj.foo1
console.log(Object.prototype.foo1) // Object.prototype.foo1

// 第三种情况
var obj2 = {}
Object.defineProperty(Object.prototype, 'foo2', {
  set: function () {
    console.log('i am foo2 setter')
  },
  get: function () {
    return 'get foo2 success!'
  }
})
Object.defineProperty(obj2, 'foo2', {
  value: 'obj.foo2'
})
console.log(obj2.foo2) // obj.foo2
console.log(Object.prototype.foo2) // get foo2 success!
```

可以看到，第二种和第三种情况都成功屏蔽了原型链上的同名属性！



参考链接：

- 《你不知道的JavaScript上卷》p144
- [js对象的属性描述符](http://ruomuc.gitee.io/blog/2021/02/21/js%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%B1%9E%E6%80%A7%E6%8F%8F%E8%BF%B0%E7%AC%A6/)
- [javascript原型链](http://ruomuc.gitee.io/blog/2020/05/21/javascript%E5%8E%9F%E5%9E%8B%E9%93%BE/)

