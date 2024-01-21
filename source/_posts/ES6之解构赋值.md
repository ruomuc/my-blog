---
title: ES6之解构赋值
categories:
  - JavaScript
tags:
  - JavaScript
  - es6
date: 2020-06-04 22:34:42
updated: 2020-06-04 22:34:42

---
ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。
## 数组的解构赋值
#### 基本用法
左右两边**模式相同**，即可赋值:
```javascript
let [a,b] = [0,1];
console.log(a,b) // 0 1

let [a,b,[c]] = [0,1,[3]]
console.log(a,b,c) // 0 1 3
```
左右两边**模式不完全相同**，可以成功赋值部分:
```javascript
let [a,b,c] = [0,1];
console.log(a,b,c) // 0 1 undefined

let [a,b] = [0,1,2];
console.log(a,b) // 0 1

let [a,...b,c] = [0,1,2,3];
console.log(a,b,c) // 0 [ 1, 2, 3 ]
```
事实上，只要某种数据结构具有 **Iterator 接口**(后面会说到)，都可以采用数组形式的解构赋值：
```javascript
function* fibs() {
  let a = 0;
  let b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}
let [first, second, third, fourth, fifth, sixth] = fibs();
console.log(first, second, third, fourth, fifth, sixth) // 0 1 1 2 3 5
```
#### 默认值
首先，`ES6`允许解构赋值指定默认值，但是必须当值严格相等于`undefined`才会生效：
```javascript
let [a = 1] = [];
console.log(a); // 1

let [b = 1] = [null];
console.log(b); // null

let [c = 1] = [undefined];
console.log(c); // 1
```
<!--more-->
结构复制可以引用其它变量，但必须声明过：

```javascript
let [x = y, y = 1] = [undefined, 1];
console.log(x, y); // ReferenceError: y is not defined

let [x = 1 , y = x] = [10];
console.log(x,y); // 10 10
```
默认值可以是一个函数：
```javascript
function f() {
  return 10;
}
let [x = f()] = [];
console.log(x); // 10
```
<br>
## 对象的解构赋值
#### 基本用法
可以看到，对象解构赋值，对于顺序没有要求，如果能找到与变量名称相同的`key`值，就赋值，找不到就是`undefined`
```javascript
let { a, b, c } = { a: 1, b: 2, c: 3 }
console.log(a, b, c); // 1 2 3

let { c, b, a } = { a: 1, b: 2, c: 3 }
console.log(a, b, c); // 1 2 3

let { a, c, b } = { a: 1 }
console.log(a, b, c); // 1 undefined undefined
```
如果想要赋值的变量名和对象的`key`值不相同：
```javascript
let { a: obj1, b: obj2, c: obj3 } = { a: 1, b: 2, c: 3 }
console.log(obj1, obj2, obj3); // 1 2 3
```
对象也可以嵌套赋值,下面的代码中`p`和`y`都是模式，`x`和`obj2`才是变量：
```javascript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};
let { p: [x, { y: obj2 }] } = obj;
console.log(x, obj2);
```
#### 默认值
对象的解构赋值也可以设置默认值，生效条件是，对象的属性严格相等与`undefined`：
```javascript
let { a, b = 12 } = { a: 1 };
console.log(a, b); // 1 12 

let { a: obj1, b: obj2 = 12 } = { a: 11, b: undefined };
console.log(obj1, obj2); // 11 12 

let { a: obj1, b: obj2 = obj1 } = { a: 12 };
console.log(obj1, obj2); // 12 12
```
<br>
## 字符串解构赋值
字符串被转换成了一个类似数组的对象：
```javascript
const [a, b, c, d, e] = 'hello';
console.log(a, b, c, d, e); // h e l l o

const { 0: a, 1: b, 2: c, 3: d, 4: e, length: len } = 'hello';
console.log(a, b, c, d, e, len); // h e l l o 5
```
可以看到，字符串即可以作为数组也可以作为对象解构赋值，但是当做对象时，还可以获取到`length`属性。
<br>
## 数值和布尔值的解构赋值
规则是，只要等号右边的值不是对象或数组，就先将其转为对象：
```javascript
let { __proto__: s } = Object(123);
console.log(s === Number.prototype); // true

let { __proto__: s } = Object(true);
console.log(s === Boolean.prototype) // true
```
看过我之前写的那篇原型链的文章，应该能看懂上面的代码。
<br>
## 函数参数的解构赋值
函数的参数也可以使用解构赋值,规则和上面的相同。
```javascript
function a([x, y = 1], { z, c }) {
  console.log(x, y, z, c)
}
a([1, 2], { z: 3, c: 4 })
```
<br>
## 用途
这部分内容，没什么好说的，拷贝忍者上线~
#### 交换变量值
```javascript
let x = 1;
let y = 2;

[x, y] = [y, x];
```
#### **从函数返回多个值**

```javascript
// 返回一个数组

function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

#### **函数参数的定义**

```javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

#### **提取 JSON 数据**

```javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

#### **函数参数的默认值**

```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more configs
} = {}) {
  // ... do stuff
};
```

#### **遍历 Map 结构**

任何部署了 Iterator 接口的对象，都可以用`for...of`循环遍历。Map 结构原生支持 Iterator 接口，配合变量的解构赋值，获取键名和键值就非常方便。

```javascript
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
```

#### **输入模块的指定方法**

```javascript
const { SourceMapConsumer, SourceNode } = require("source-map");
```