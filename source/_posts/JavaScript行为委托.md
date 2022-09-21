---
title: JavaScript行为委托
categories:
  - javascript
tags:
  - js
  - 原型链
date: 2021-03-21 16:04:52
updated: 2021-03-21 16:04:52

---

在 java 、c# 等语言中，类是一种必须使用的”设计模式“，但是在 JavaScript 等语言中，它是可选的，并且在JavaScript中实现类总是会有各种缺陷。

还有要注意的一点是，JavaScript中没有类，只有对象，没有继承，只有委托（prototype）。

包括 es6 的 class 语法在内的各种语法糖，也不会改变这个事实。

**首先要了解两个技巧**

1.  [使用 call 方法调用父构造函数可以实现继承](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call#%E4%BD%BF%E7%94%A8_call_%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8%E7%88%B6%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)
2.  [用 `Object.create`实现类式继承](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create#用_object.create实现类式继承)

**"类"风格的代码**

基于 原型链的 ”继承” 不是真正的继承（拷贝副本），只是”引用“而已。
<!--more-->

```js
function Foo(who){
    this.me = who
}
Foo.prototype.identify = function(){
    return "I am " + this.me
}

Foo.prototype.speak = function(){
  console.log("Foo name is , " + this.identify() + ".")
}

function Bar(who){
    // ”继承“ me属性
    Foo.call(this,who)
}
// “继承” identify 和 speak 方法
Bar.prototype = Object.create(Foo.prototype)

// 重写 speak 方法
Bar.prototype.speak = function(){
    console.log("Hello, " + this.identify() + ".")
}

var b1 = new Bar("b1")
var b2 = new Bar("b2")
var f1 = new  Foo("f1")

b1.speak() // Hello, I am b1.
b2.speak() // Hello, I am b2.
f1.speak() // Foo name is , I am f1. 实现了“多态”

```

但是如果父类修改了它的方法

```js
// ...
Foo.prototype.identify = function (){
  return "Foo modified identify  " + this.me
}
b1.speak() // Hello, Foo modified identify  b1.
```

**委托风格**

委托意味着某些对象找不到属性或者方式时，会把这个请求委托给另一个对象。

1.  没有父子级的关系，无需继承。
2. b1 把 speak 需要的一些行为放在 Foo 中，两个对象协同完成 speak 这一动作。

```js
Foo = {
  init: function (who) {
    this.me = who
  },
  identify: function () {
    return 'I am ' + this.me
  }
}

Bar = Object.create(Foo)
Bar.speak = function () {
  console.log('Hello, ' + this.identify() + '.')
}

var b1 = Object.create(Bar)
b1.init('b1')
var b2 = Object.create(Bar)
b2.init('b2')
b1.speak() // Hello, I am b1.
b2.speak() // Hello, I am b2.
```

类的代码组织方式类似一棵树，是垂直的， 而委托的代码组织方式是并排的。



ps: 因为我工作使用的第一门语言就是 JavaScript ，中间也没有怎么用过OOP，所以对于 js 的这一套是很好接受的。