---
title: ES6之let和const
categories:
  - JavaScript
tags:
  - JavaScript
  - es6
date: 2020-05-28 21:14:19
updated: 2020-05-28 21:14:19

---
我前面写过一篇整理es6的文章[关于ES6]([https://ruomuc.gitee.io/blog/2019/02/23/%E5%85%B3%E4%BA%8EES6/](https://ruomuc.gitee.io/blog/2019/02/23/关于ES6/))。但是好像太监了。。。因为太长了。

然后不幸的是前两天面试被问到了`es6`的数组扩展方法，刚好我没看到。。这是一个悲伤的故事，所以我决定看阮一峰老师的[ECMAScript 6 入门](https://es6.ruanyifeng.com/)。这次一章一章的整理。尽量做到完全弄懂。
<br>

## let命令
没错，又是`let`，我已经写了不知道多少次了。。。没事多看几次加深理解嘛。。
#### 基本用法
```javascript
{
	let a = 1;
	var b = 2;
}
console.log(a); // ReferenceError: a is not defined
console.log(b); // 2
```
####  经典问题
正常的for循环：
```JavaScript
for(var i = 0; i < 6 ; i ++){
  console.log(i); // 循环打出 0-5
}
console.log(i); // 6
```
正常循环中使用异步调用：
```javascript
for(var i = 0; i < 6 ; i ++){
  setTimeout(()=>{
    console.log(i); // 一秒后，瞬间输出6个6
  },1000)
}
console.log(i); // 立刻输出一个 6
```
稍微变型一下：
```javascript
for(var i = 0; i < 6 ; i ++){
  setTimeout(()=>{
    console.log(i); // 每隔1s，输出一个6，一共输出6次
  },i*1000)

console.log(i); // 立刻输出一个 6
```
其实我们的本意是让它一秒后输出12345或者每过1s输出一个对应值。
通过上面的几个列子可以得出什么结论？

- 使用var声明的循环变量，立刻调用，会调用到正常的值，比如第一个列子，或者第三个列子的`setTimeout`的第二个参数。
- 如果不立刻使用，使用的值是循环结束后的最终值。
<br>
在没有`let`的`es6`之前的版本，我们是怎样解决这个问题的呢？
```javascript
for (var i = 0; i < 6; i++) {
  (function (i) {
    setTimeout(() => {
      console.log(i); // 一秒后，瞬间输出 0,1,2,3,4,5
    }, 1000)
  })(i)
}
console.log(i); // 立刻输出一个 6
```
上面这个方法，使用了一个匿名自执行的函数，将`i`当做参数传递进去，`i`就是一个匿名函数内的变量了，所以外部`i`的修改，不会影响内部变量。。。等下。好像发现了什么，这不是闭包的思想吗？？?
说道闭包，，我之前好像也有文章写过闭包，不过我不用去翻都知道很烂。。
关于闭包，我看了一些博客和知乎的一些回答，发现每个地方对闭包的解释都不大相同，感觉是一个很抽象的概念。
我肯定会重新写一篇文章来专门研究闭包的。。
<br>
我们用`let`写一下试试：
```javascript
for(let i = 0; i < 6 ; i ++){
  setTimeout(()=>{
    console.log(i); // 一秒后，瞬间输出0,1,2,3,4,5
  },1000)
}
console.log(i); // 立刻输出一个 6
```
可以看到，`let`这时的作用相当于前文中的匿名自执行函数，原理都是每一次循环，`i`值都在一个新的作用域里面。
<font color = red>当然`let i =0`这个声明只会执行一次，JavaScript引擎会在每一次循环结束，会记住循环变量的值，然后再一个新的块里重新声明一个循环变量并把该值赋给它。详见这篇文章[怎么理解for循环中let声明的迭代变量每次是新的变量](https://segmentfault.com/q/1010000007541743)</color>
<br>

#### 不存在变量提升
同一作用域中如果在`var`声明变量之前使用了它，不会报错，会输出`undefined`：
```javascript
function fun(){
	console.log(a); // undefined
	var a = 1;
	console.log(a); // 1
}
fun();
```
但是如果是let声明的变量会报错：
```javascript
function fun(){
	console.log(a); // ReferenceError
	let a = 1;
	console.log(a); // 1
}
fun();
```
#### 暂时性死区
暂时性死区很好理解，看起来好像和变量提升一个原理。
只要记住，使用let和const声明的变量，任何在未声明之前的操作都是不允许的。
<!--more-->

```javascript
var a = 10;
function fun(){
	var b = a;
	console.log(b); // undefined
	var a = 1;
	console.log(a); // 1	
}
fun();
```
但是使用let的话：
```javascript
var a = 10;
function fun(){
	let b = a;
	console.log(b); // ReferenceError
	let a = 1;
	console.log(a); // 1	
}
fun();
```
`ES6` 规定暂时性死区和`let`、`const`语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量，从而导致意料之外的行为。这样的错误在` ES5` 是很常见的，现在有了这种规定，避免此类错误就很容易了。
<br>
#### 不允许重复声明

很容易理解，就是`let`声明变量时，当前作用域中不允许有相同的变量,下面这几种都会报错：

```javascript
{
	let a = 1;
	var a =1;
}
{
	var a = 1;
	let a = 1;
}
{
	let a = 1;
	let a = 2;
}
```

而且我发现，作用域内声明的全局变量不算是全局，这个规则无论是在那个作用域还是用什么声明`var let const`，都是一样的

所以建议声明全局变量使用`window`或者`global`：

```javascript
// a.js
{
  a = 1; // ReferenceError 被变量提升规则处理
  let a = 22;
  console.log(a) 
}
// b.js
{
  let a = 1;
  a = 22;
  console.log(a) // 22
}
// c.js
a = 233;
console.log(a) // 233
var a = 222;
{
  let a = 1;
  console.log(a) // 1
}
console.log(global.a) // undefined
// d.js
function func(){
  a = 233; // 这个变量不会变为全局变量
  var a = 123; // 如果去掉这一行，都会输出233，a会变为全局变量。。
  console.log(a)
}
console.log(a) // ReferenceError: a is not defined
```

这个我也是才发现。。 **-.-!**



## 块级作用域

`ES5` 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。
内层变量覆盖了外层变量：

```javascript
var tmp = new Date();
function f() {
  console.log(tmp); // undefined  。这里本意是想使用外层变量，但是由于后面又定义该变量，所以被变量提升了。。
  if (false) {
    var tmp = 'hello world';
  }
}

f();
```
还有就是前文中，for循环中`var`定义的循环变量在循环结束后，变量不会消失，会泄露造成变量污染。
<br>
#### 块级作用域中的函数声明
详细的看这里[块级作用域中的函数声明](https://es6.ruanyifeng.com/#docs/let#%E5%9D%97%E7%BA%A7%E4%BD%9C%E7%94%A8%E5%9F%9F%E4%B8%8E%E5%87%BD%E6%95%B0%E5%A3%B0%E6%98%8E)，这里比较绕。。

- `ES5` 规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域声明。
- `ES6` 规定，块级作用域之中，函数声明语句的行为类似于`let`，在块级作用域之外不可引用。

但是：

- `ES5`浏览器没有遵守这个规定，为了兼容以前的旧代码，还是支持在块级作用域之中声明函数
- `ES6` 在[附录 B](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-block-level-function-declarations-web-legacy-compatibility-semantics)里面规定，浏览器的实现可以不遵守上面的规定，有自己的[行为方式](http://stackoverflow.com/questions/31419897/what-are-the-precise-semantics-of-block-level-functions-in-es6)。
  - 允许在块级作用域内声明函数。
  - 函数声明类似于`var`，即会提升到全局作用域或函数作用域的头部。
  - 同时，函数声明还会提升到所在的块级作用域的头部。

事实上，`node.js`貌似也没有遵守这些个规定。  

看代码:

```javascript
// ES5 环境
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());

// 上面的写法等于下面的写法:

function f() { console.log('I am outside!'); }

(function () {
  function f() { console.log('I am inside!'); }
  if (false) {
  }
  f(); // I am inside!
}());
```

```javascript
// ES6 环境
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());

// 上面的写法等于下面的写法:

function f() { console.log('I am outside!'); }
(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f(); // undefined
}());
```



## const命令

#### 基本用法

`const`声明的变量不得改变值，这意味着，`const`一旦声明变量，就必须立即初始化，不能留到以后赋值。

```javascript
const a = 1;
a = 2; // SyntaxError: Identifier 'a' has already been declared

const b; // SyntaxError: Missing initializer in const declaration
```

`const`命令同样**也不能变量提升**，并且**存在暂时性死区**，**只在块级作用域内有效**，**不可重复声明**。

#### const不可变属性原理

说白了`const`命令冻结的是栈中的数据，基本类型冻结的就是它本身的值，而引用类型冻结的是它的地址，对象中的值不会被冻结。

```javascript
var b = {
  a:1,
  b:2
};
const a = b;

console.log(a); // { a: 1, b: 2 }
b['a'] = 120;
console.log(a); // { a: 120, b: 2 }
```

如果想要冻结一个对象就要使用`Object.freeze()`

```javascript
var b = {
  a:1,
  b:2
};
const a = b;
Object.freeze(a)
Object.freeze(b)
console.log(a); // { a: 1, b: 2 }
console.log(b); // { a: 1, b: 2 }
b['a'] = 120;
console.log(a); // { a: 1, b: 2 }
console.log(b); // { a: 1, b: 2 }
```
ES5 只有两种声明变量的方法：`var`命令和`function`命令。ES6 除了添加`let`和`const`命令，后面章节还会提到，另外两种声明变量的方法：`import`命令和`class`命令。所以，ES6 一共有 6 种声明变量的方法。
