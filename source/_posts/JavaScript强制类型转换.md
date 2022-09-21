---
title: JavaScript强制类型转换
categories:
  - javascript
tags:
  - js
date: 2021-04-09 22:33:10
updated: 2021-04-09 22:33:10

---

类型转换和强制类型转换？

- 类型转换发生在静态类型语言的编译阶段。
- 强制类型转换发生在动态类型语言的运行时。

并且大家所说的隐式和显式，也是取决于不同开发者的认知，在JavaScript中统称为**强制类型转换**。
<!--more-->

举个显式和隐式转换的栗子：

```js
var a = 22
var b = a + ''
var c = +b

console.log(typeof a,a) // number 22
console.log(typeof b,b) // string 22
console.log(typeof c,c) // number 22
```

如果说 `a+ ''`是隐式转换的话，那么 `+b` 呢？ 事实上在 JavaScript 开源社区，一元操作符  `+ `被认为是显式强制类型转换，所以显式和隐式取决于开发者的经验。
## 转为字符串

字符串的基本转化规则就是加上 `""`。

```js
console.log(null+'') // "null"
console.log(undefined +'') // "undefined"
console.log(true+'') // "true"
console.log(false+'') // "false"
console.log(Math.pow(2,100)+'') // "1.2676506002282294e+30"
```

对于对象：

1. 如果对象没有 valueOf 和 toString 方法，比如Object.create(null) 创建的对象，那么就会报错。
2. 如果对象没有 toString() 方法，或者 toString() 方法返回的值不是一个原始值，那么调用 valueOf() 方法，并将返回值强制类型转换为字符串。
3. 如果对象有 toString() 方法，并且返回值是一个原始值，那么调用 toString() 方法，并将返回值强制类型转换为字符串。

```js
var a = Object.create(null)
console.log(String(a)) // Uncaught TypeError: Cannot convert object to primitive value

a.valueOf = function(){ return 123 }
console.log(String(a)) // 123

a.toString = function (){ return {} }
console.log(String(a)) // 123

a.toString = function (){ return 456 }
console.log(String(a)) // 456
```

[JSON.stringify](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) 并不是严格意义上的强制类型转换，但是也涉及到了转换为字符串的规则。

1. 对于简单值，JSON 字符串化和 toString() 效果基本相同。
2. 对于循环引用的对象会报错。
3. 对于 undefined、function、symbol 会自动忽略，如果在数组中会返回null。
4. 如果对象定义了 toJSON() 方法，会先调用该方法，并使用该方法的返回值进行序列化。

## 转为数字

基本规则：

```js
console.log(true) // 1
console.log(false) // 0
console.log(undefined) // NaN
console.log(null) // 0
```

对于对象：

1. 如果对象没有 valueOf 和 toString 方法，比如Object.create(null) 创建的对象，那么就会报错。
2. 如果对象没有 valueOf() 方法，或者 valueOf() 方法返回的值不是一个原始值，那么调用 toString() 方法，并将返回值强制类型转换为字符串。
3. 如果对象有 valueOf() 方法，并且返回值是一个原始值，那么调用 valueOf() 方法，并将返回值强制类型转换为字符串。

```js
var a = Object.create(null)
console.log(Number(a)) // Uncaught TypeError: Cannot convert object to primitive value

a.toString = function(){ return 123 }
console.log(Number(a)) // 123

a.valueOf = function (){ return {} }
console.log(Number(a)) // 123

a.valueOf = function (){ return 456 }
console.log(Number(a)) // 456
```

## 转为布尔值

JavaScript 中的假值：

- undefined
- null
- false
- +0、-0、NaN
- ""

除了以上假值外的都是真值，转换为布尔值后都是 true。

## Number() 和 parseInt()

- Number() 是强制类型转换， 而 parseInt() 是解析数字字符串
- 解析允许字符串中存在非数字字符串，从左向右解析，遇到非数字字符串就停止。
- 并且 parseInt() 的参数只能是字符串，遇到非字符串会将其转为字符串在进行解析。

```js
var a = '123c4m'
console.log(Number(a)) // NaN
console.log(parseInt(a)) // 123

var b = {}
console.log(parseInt(b)) // NaN
b.toString = function(){return '1234abc'}
console.log(parseInt(b)) // 1234
```

## a+'' 和 String(a)

这两种转换字符串的方式有什么区别呢。其实大部分情况下是没有区别的，只有 a 是一个对象的时候有些坑。

前文对于对象转数字和字符串的过程说的都十分清楚了，下面直接看代码。

**a + '' 执行过程：**

1. 会先 a 转为数字。
2. 然后将数字转为字符串
3. 然后拼接字符串

```js
var a = Object.create(null)

a.toString = function(){ return 123 }
console.log(a + '') // "123"

a.valueOf = function (){ return 456 }
console.log(a + '') // "456"
```

**String(a) 执行过程**

直接调用 a 的 toString() 方法。

```js
var a = Object.create(null)

a.valueOf = function (){ return 456 }
console.log(String(a)) // "456"

a.toString = function(){ return 123 }
console.log(String(a)) // "123"
```

