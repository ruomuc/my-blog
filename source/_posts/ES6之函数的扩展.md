---
title: ES6之函数的扩展
categories:
  - JavaScript
tags:
  - JavaScript
  - es6
date: 2020-07-11 13:51:59
updated: 2020-07-11 13:51:59

---

## 函数参数的默认值

### 基本用法

es6之前函数的参数不能使用默认值，只能采取这种写法：

```js
function a(x){
  x=x||1; // x的默认值为1
  console.log(x);
}
a(); // 1
```

但是es6可以为函数的参数设置默认值：

```js
function a(x = 1){
  console.log(x);
}
a(); // 1
```

**不能有重复参数**：

非严格模式下，重复参数不会报错：

```js
function a(x, x, y) {
  console.log(x, x, y)
}
a(1, 2, 3); // 2 2 3
```

但是严格模式或者使用参数默认值时，不能使用这种写法：

```js
'use strict'
function a(x, x, y) {
  console.log(x, x, y)
}
a(1, 2, 3); // Duplicate parameter name not allowed in this context

// 或者

function a(x, x, y = 1) {
  console.log(x, x, y)
}
a(1, 2, 3); // Duplicate parameter name not allowed in this context
```

**参数变量是默认声明的**
<!--more-->
不能再次使用`let`或者`const`声明和参数相同的变量：

```js
function foo(x = 5) {
  let x = 1; // error
  const x = 2; // error
}
foo()
```

**函数参数默认值时惰性求值**

下面代码中的`p`的默认值是`x+1`，每次调用都会重新计算`x+1`的值：

```js
let x = 99;
function foo(p = x + 1) {
  console.log(p);
}
foo() // 100
x++;
foo() // 101
```

### 与解构赋值默认值结合使用

**基本用法**

```javascript
function foo({x, y = 5}) {
  console.log(x, y);
}
foo(1); // 1 5
```

下面两种写法的结果是相同的，但是执行过程确不同：

- 写法一：默认值为`{}`，结构赋值的默认值为`x=0 y=0`
- 写法二：默认值为`{x:0,y:0}`，结构赋值之后`x=0 y=0`

```js
// 写法一
function m1({ x = 0, y = 0 } = {}) {
  return [x, y];
}

// 写法二
function m2({ x, y } = { x: 0, y: 0 }) {
  return [x, y];
}

console.log(m1())
console.log(m2())
```

### 参数默认值的位置

通常情况下，定义了默认值的参数应该是函数的尾参数，因为前面的参数无法省略，触发你你显示的传入`undefined`

```js
function a(a=1,b){
  console.log(a,b)
}
a(10) // 10 undefined
a(undefined,2) // 1 2 
```

### 函数参数的length属性

指定了默认值以后，函数的`length`属性，将返回没有指定默认值的参数个数。也就是说，指定了默认值后，`length`属性将失真。

并且函数的`length`属性会忽略第一个有默认值的参数后面所有的参数。

```javascript
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
(function (a=0, b, c = 5) {}).length // 0
```

### 作用域

一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域（context）。

```javascript
var x = 1;
function f(x, y = x) {
  console.log(y);
}
f(2) // 2
```

调用函数`f`时，参数形成一个单独的作用域。在这个作用域里面，默认值变量`x`指向第一个参数`x`，而不是全局变量`x`，所以输出是`2`。
<br>

## rest 参数

### 基本用法

ES6 引入 rest 参数（形式为`...变量名`），用于获取函数的多余参数，这样就不需要使用`arguments`对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。

```javascript
function add(...values) {
  let sum = 0;
  for (var val of values) {
    sum += val;
  }
  return sum;
}
add(2, 5, 3) // 10
```

### rest和arguments的区别

**arguments是一个类数组对象，而rest参数是一个真正的数组。**

- arguments想要调用数组的方法要这样`Array.prototype.slice.call(arguments).sort()`
- 它们都有length属性，返回参数数量
- 函数的length属性不包括rest参数

```js
function a(...values) {
  console.log(arguments)
  console.log(values)
}

a(1, 2, 3, 4, 5)
console.log(a.length); // 0
```

## 严格模式

从 ES5 开始，函数内部可以设定为严格模式。

但是在`ES6`中<font color = "red">只要参数使用了默认值、解构赋值、或者扩展运算符，就不能显式指定严格模式。</font>

[具体原因详见]([https://es6.ruanyifeng.com/#docs/function#%E4%B8%A5%E6%A0%BC%E6%A8%A1%E5%BC%8F](https://es6.ruanyifeng.com/#docs/function#严格模式))
<br>

## name属性

函数的`name`属性，返回该函数的函数名。这个属性早就被浏览器广泛支持，但是直到 ES6，才将其写入了标准。

```javascript
function foo() {}
foo.name // "foo"
```

ES6 对这个属性的行为做出了一些修改。如果将一个匿名函数赋值给一个变量，ES5 的`name`属性，会返回空字符串，而 ES6 的`name`属性会返回实际的函数名。

```javascript
var f = function () {};
// ES5
f.name // ""
// ES6
f.name // "f"
```

`Function`构造函数返回的函数实例，`name`属性的值为`anonymous`。

```javascript
(new Function).name // "anonymous"
```

`bind`返回的函数，`name`属性值会加上`bound`前缀。

```javascript
function foo() {};
foo.bind({}).name // "bound foo"
(function(){}).bind({}).name // "bound "
console.log((new Function().bind()).name) // bound anonymous
```

<br>

## 箭头函数

箭头函数的使用方法就不说了。。。

### 使用注意点

关于箭头函数的this的绑定问题，可以看我这篇博文[JavaScript中的this](https://ruomuc.gitee.io/blog/2020/05/24/JavaScript中的this/)

1. 函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象。
2. 不可以当作构造函数，也就是说，不可以使用`new`命令，否则会抛出一个错误。
3. 不可以使用`arguments`对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。
4. 不可以使用`yield`命令，因此箭头函数不能用作 Generator 函数。

### 不适用场合

**第一个场合是定义对象的方法，且该方法内部包括`this`。**

```javascript
const cat = {
  lives: 9,
  jumps: () => {
    this.lives--;
  }
}
```

这里的箭头函数指向全局，而不是指向cat。

**第二个场合是需要动态`this`的时候，也不应使用箭头函数。**

```javascript
var button = document.getElementById('press');
button.addEventListener('click', () => {
  this.classList.toggle('on');
});
```

这里的this指向的也是全局而不是button对象。
<br>

## 其它改动
**函数参数的尾逗号**

ES2017 [允许](https://github.com/jeffmo/es-trailing-function-commas)函数的最后一个参数有尾逗号（trailing comma）。这样的规定也使得，函数参数与数组和对象的尾逗号规则，保持一致了。

**Function.prototype.toString()**

[ES2019](https://github.com/tc39/Function-prototype-toString-revision) 对函数实例的`toString()`方法做出了修改，以前会省略注释和空格，修改后的`toString()`方法，明确要求返回一模一样的原始代码。

```js
function /* foo comment */ foo () {}

//以前
console.log(foo.toString()); // // function foo() {}
// ES2019
console.log(foo.toString()); // function /* foo comment */ foo () {}
```

**catch 命令的参数省略**

JavaScript 语言的`try...catch`结构，以前明确要求`catch`命令后面必须跟参数，接受`try`代码块抛出的错误对象。

很多时候，`catch`代码块可能用不到这个参数。但是，为了保证语法正确，还是必须写。[ES2019](https://github.com/tc39/proposal-optional-catch-binding) 做出了改变，允许`catch`语句省略参数。

```js
try {
  // ...
} catch (err) {
  // 处理错误
}

// ES2019
try {
  // ...
} catch {
  // ...
}
```
<br>
ps:  如何在typescript中使用rest参数？定一个any[] 吗？这样岂不是违反了使用ts的初衷，可读性和可维护性变差了？