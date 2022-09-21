---
title: JavaScript中的this
categories:
  - javascript
tags:
  - js
  - this
date: 2020-05-24 01:11:21
updated: 2020-05-24 01:11:21

---

## 默认绑定规则

this的指向在函数定义的时候是无法确定的，只有在函数执行的时候才能确定。

因为浏览器和`node.js`的全局对象有一些区别。

在浏览器中:

```javascript
function A(){
	console.log(this.a); // 1
}
var a = 1;
A()
```

但是在`node.js`中:

```javascript
function A(){
	console.log(this.a); // undefined
	console.log(this.global.b); // 2
}
var a = 1;
b = 2;
A()
```

从上面的代码中可以看出：

- 浏览器的`this`是`window`对象，`node.js`的`this`包含的东西更加多，其中有`this.global`就是`node.js`中的全局对象。
- `node.js`的全局变量必须使用无修饰符`b=2`或者`global.b=2`的方式来声明。

补充：

- 严格模式下`use strict`,浏览器的**函数内部**`this`指向为`undefined`,而`node.js`的`this`为空对象`{}`。
- 严格模式下浏览器全局变量只可以使用`this.a= 1 window.a=1`不能使用`a=1`,`node.js`也只能使用`global.a=1`
- 更多严格模式，或者浏览器环境的`js`和`node.js`环境的`js`后面再详细说明吧。

考虑到以上因素，以下全部以浏览器环境为准。。

## 隐式绑定
<!--more-->
大部分情况下，`this`会指向那个调用它的对象

```javascript
function A(){
	var a = 11;
  var b = 22;
  console.log(this.a) // undefined
  console.log(this.b) // 2
}
var b = 2;
A();
```

上面的` A()`可以理解为`window.A()`,所以`this`指向了`window`，但是window下面没有`a`这个属性，所以是undefined.

```javascript
function A() {
  var a = 11;
  var b = 22;
  console.log(this.a) // 123
  console.log(this.b) // 234
}

var obj1 = {
  a: 123,
  b: 234,
  fn: A
}

var obj2 = {
  a: 666,
  b: 999,
  obj: obj1
}

obj1.fn() 

obj2.obj.fn()
```

上面的两种调用方式的结果都是一样的，`this.a // 123  this.b // 234`。

为什么呢，因为如果存在多次调用，对象属性引用链只有一层或者说最后一层在**调用**位置中起作用。就是说不管调用多少层，`this`只会指向第一层。

## 隐式丢失

继续：

```javascript
function A() {
  var a = 11;
  var b = 22;
  console.log(this.a) // 1
  console.log(this.b) // 2
}

var obj1 = {
  a: 123,
  b: 234,
  fn: A
}

var obj2 = {
  a: 666,
  b: 999,
  obj: obj1
}
this.a=1;
this.b=2;
var fn1 = obj1.fn
var fn2 = obj2.obj.fn
fn1()
fn2()
```

上面的`fn1`和`fn2`的输出结果都是一样的`this.a  // 1 this.b // 2`

神奇？为什么和上一步的结论不一样呢，为什么这次的多层调用最终`this`指向了全局？？？（当然也取决于是否严格模式）

**原来这里的`fn1`和`fn2`都只是拿到了`function A`的一个引用！！！**不信你输出一下他们的值。

所以说这并没有和上面的结论冲突！！核心在**调用**，调用了才会确定`this`的指向。

## 显式绑定

`call`、`apply`、`bind`、和`new`绑定等

关于`call`、`apply`、`bind`详情可以看[[译] JavaScript 中至关重要的 Apply, Call 和 Bind](https://hijiangtao.github.io/2017/05/07/Full-Usage-of-Apply-Call-and-Bind-in-JavaScript/)，这篇文章我看了几遍，也只懂了个大概，他们三个好像区别不是很大，所以我后面会写一篇简单捋一下三者的区别。

`this`指向很好理解，绑定了某个对象，`this`就指向绑定的对象



### new绑定

[`new`运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
官网说使用**`new` 运算符**创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。**`new`** 关键字会进行如下的操作：

1. 创建一个空的简单JavaScript对象（即`{}`）；**` var a = {}`**
2. 链接该对象（即设置该对象的构造函数）到另一个对象 ； **`a.__proto__ = 构造函数.prototype`**
3. 将步骤1新创建的对象作为`this`的上下文 ；**`this 指向 a对象`**
4. 如果该函数没有返回**对象**，则返回`this`。 **`如果返回的null，还是返回this `**

```javascript
function A() {
  this.a = 1;
}
var obj = new A();
console.log(obj.a)
```

这段代码比较好理解，没什么好说的。

## 优先级

显示绑定大于隐式绑定是毋庸置疑的。

主要看看，`new`绑定和`apply call bind`谁的优先级高？

```javascript
function A(a) {
  this.a = a
}

var obj1 = {a:123}
var foo = A.bind(obj1)
foo(1)
console.log(obj1)
var newFoo = new foo(12)
console.log(obj1.a)
console.log(newFoo.a)
```

可以看到，`new`操作修改了`bind`的绑定。

所以最终结果是：<font color = "red" >new绑定 > 显示绑定（call、apply） > 隐式绑定 > 默认绑定</font>

### 如何判断this绑定对象

如果要判断一个运行中函数的this绑定，就需要找到这个函数的直接调用位置。找到后可以按照顺序应用下面这四条规则来判断this的绑定对象。

1. 由new调用？绑定到新创建对象。
2. 由call或者apply（或者bind）调用？绑定到指定对象。
3. 由上下文对象调用？绑定到那个上下文对象。
4. 默认：在严格模式下绑定到undefined，否则绑定到全局对象

## 箭头函数

说完了` this`,但是发现好像漏掉了什么，在es6中引入的箭头函数，也和`this`的指向有一定的关系。

这里只探讨一下箭头函数的`this`，对于箭头函数的其他特性，有机会在写。

**箭头函数默认绑定外层的this对象**

```javascript
var obj = {
  a: function () {
    console.log(this)
  },
  b: () => console.log(this)
}
obj.a(); // obj对象,因为方法a由Obj对象调用
obj.b(); // window对象，严格模式会输出undefined


```
上面的代码没什么好分析的，在看一段：
```javascript
var obj1 = {
  a: {
    fna: function () {
      console.log(this) // obj1.a对象
    }
  },
  b: {
    fnb: () => console.log(this) // window对象
  },
  c: function fnc() {
    return () => console.log(this) // obj1 对象
  },
  d: () => {
    return () => console.log(this)
  },
  e: function a() {
    console.log(this); // obj.e 对象
    var f1 = function () { console.log(this) } // window对象
    f1()
  }
  f: function ff() {
    console.log(this) // obj1对象
    function fff() {
      console.log(this) // window对象
      var ffff = () => console.log(this); // window对象
      ffff()
    }
    fff()
  }
}
obj1.a.fna(); 
obj1.b.fnb(); 
obj1.c()(); 
obj1.d()(); 
obj1.e(); 
obj1.f()
```

- 第一个：调用`fna()`方法的为`obj1.a`这个对象
- 第二个：箭头函数没有被非箭头函数包含，无论嵌套多少层，都是中指向最外层的this，即window
- 第三个：箭头函数被非箭头函数包含，`this`指向调用非箭头函数的对象
- 第四个：同第二个，无论嵌套多少箭头函数或者对象，` this`始终指向最外层
- 第五个：非箭头函数嵌套，里面的函数没有被，所以`this`指向 全局对象，箭头函数的出现就是为了解决这一类问题
- 第六个：虽然箭头函数被非箭头函数包含，但是包含它的非箭头函数没有被一个对象调用，所以他们的`this`指向的都是全局

下面看下`bind apply call `能否修改箭头函数的指向

```javascript
var obj4 = {
  a: () => {
    console.log(this)
  },
  b: function () {
    var f1 = () => console.log(this)
    f1()
  }
}
obj4.a() // window
obj4.b() // obj4
var obj5 = {}
obj4.a.call(obj5) // window
obj4.b.call(obj5) // obj5
```

- ` call`无法修改箭头函数的指向，由于`this`在箭头函数中已经按照词法作用域绑定了，无法修改。
- 但是`call`可以修改包含箭头函数的非箭头函数的`this`指向，箭头函数内的`this`也同时被修改。
- 由此证明，箭头函数的`this` 是绑定在包含它的那个非箭头函数上面。



最后留一个问题：

```javascript
var obj4 = {
  a: () => {
    console.log(this)
  },
  b: function () {
    var f1 = () => console.log(this)
    f1()
  }
}

var bar = obj4.b
bar()
```

上面的代码会输出什么呢？

虽然是参考的别人的文章，但是我是从头到尾结合很多篇文章捋了一遍的，例子的代码也全部是手敲执行的。。。

参考链接：

https://juejin.im/post/5c049e6de51d45471745eb98

https://www.cnblogs.com/pssp/p/5216085.html

https://www.liaoxuefeng.com/wiki/1022910821149312/1031549578462080