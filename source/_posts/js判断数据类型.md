---
title: js判断数据类型
categories:
  - javascript
tags:
  - js
date: 2021-03-03 21:48:38
updated: 2021-03-03 21:48:38

---

之前对js的类型判断用到的比较少，只知道typeof 和 instanceof。。

MDN上面其实介绍的挺详细的。。这里随便记一下

## [typeof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof)

typeof 是一个**一元**操作符，返回被操作数类型的字符串。

typeof 可以在你基本可以确定数据是哪一类数据，需要稍加区分的时候使用。

```js
typeof 1 // "number"
typeof '1' // "string"
typeof true // "boolean"
typeof Symbol("a") // "symbol"
typeof undefined // "undefined"
typeof null // "object"
typeof function(){} // "function"
typeof new Object() // "object"
typeof [1,2,3] // "object"
typeof new Date() // "object"
typeof /\w/ // "object"
```

可以看到，null、数组和对象等。类型都被判断为了"object"。

并且`typeof null=== "object" `是一个很蛋疼的bug。

在《你不知道的JavaScript》一书中译者说道：原理是这样的，不同的对象在底层都表示为二进制，在JavaScript中二进制前三位都为0的话，就会被判断为 object 类型，null的二进制就是全0，自然前三位也是0，所以执行 typeof 时会返回 "object"。

## [Object.prototype.toString()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)

每个类在内部都有一个 [[Class]] 属性。

Object.prototype.toString 的原理是当调用的时候, 就取值内部的 [[Class]] 属性值, 然后拼接成 '[object ' + [[Class]] + ']' 这样的字符串并返回。

对于使用 typeof 无法判断的类型，可以使用 `Object.prototype.toString()`来检测，该方法会获取对象的 [[Class]] 值。
<!--more-->
因为大部分对象的 toString 方法都被重写过，所以要使用 `Object.prototype.toString.call /Object.prototype.toString.apply `的方式调用。

```js
var toString = Object.prototype.toString;


toString.call(1); // "[object Number]"
toString.call('1'); // "[object String]"
toString.call(true); // "[object Boolean]"
toString.call(new Date()); // [object Date]
toString.call(new String()); // [object String]
toString.call(Math); // [object Math]
toString.call(undefined); // [object Undefined]
toString.call(null); // [object Null]
```

## [instanceof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)

语法：`obj instanceof constructor`

`instanceof` 运算符用来检测 `constructor.prototype `是否存在于参数 `object` 的原型链上。

看个例子：

```js
function AA(){}
var a = new AA()
var b = new Object()
a instanceof AA // true
a instanceof Object // true
b instanceof AA // false
b instanceof Object // true
```

看不懂 `b instanceof AA === false `  和 `a instanceof Object === true` 的，需要去了解一下原型链。

我之前写过一篇关于原型链的：[JavaScript原型链](http://ruomuc.gitee.io/blog/2020/05/21/javascript%E5%8E%9F%E5%9E%8B%E9%93%BE/)

## isNaN & Number.isNaN

先看 isNaN

```js
isNaN(NaN) // true

isNaN("string") // true
isNaN(undefined) // true

```

明显不合理，NaN是一个特殊的值，isNaN 把 "string" 和 undefined 等都判断成了 "NaN"。

所以 ES6 为了弥补这个bug，(并没有修复isNaN)。增加了 Number.isNaN。

```js
Number.isNaN(NaN) // true

Number.isNaN("string") // false
Number.isNaN(undefined) // false
```

## 总结

- 判断比较简单的，基本可以确定的数据类型，使用 typeof
- 判断 typeof 无法判断的（大部分"object"类型），使用 `Object.prototype.toString`
- 判断原型链的方法的 某个"构造函数"的 prototype 是否在 对象的原型链上，使用 instanceof
- 判断是不是 ”NaN“值，推荐使用 `Number.isNaN()`



ps: 关于原型链的文章，后面可能还会写。