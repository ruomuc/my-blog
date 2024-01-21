---
title: ES6之字符串的扩展
categories:
  - JavaScript
tags:
  - JavaScript
  - es6
date: 2020-06-09 22:15:37
updated: 2020-06-09 22:15:37

---
本文整理一下`ES6`中字符串的扩展和新增的方法。

可以先去看一下这篇文章[彻底弄懂Unicode编码](https://liyucang-git.github.io/2019/06/17/彻底弄懂Unicode编码/)，最好用笔和纸撸两遍，下文转换过程将省略。

<br>

## 字符的 Unicode 表示法

在`JavaScript`中，可以使用`\uxxxx`表示`unicode`编码，`xxxx`表示`unicode`编码的码点，但是当码点超过`\u0000 ~ \uFFFF`这个区间，会被截断。
比如中文`𠮷`的码点是`0x20BB7`，但是在`JavaScript`中输出的是：

```javascript
let a = '\u20BB7';
console.log(a); // ₻7
```
事实上`JavaScript`只能处理`UCS-2` 编码。
至于什么是`UCS-2`，两者的关系简单说，**就是 UTF-16 取代了 UCS-2，或者说 UCS-2 整合进了 UTF-16**。所以，现在只有 UTF-16，没有 UCS-2。
所以我们把这个码点转化为`utf-16`双字节表示：

```javascript
let a = '\uD842\uDFB7';
console.log(a); // 𠮷
```
所以`ES6` 对这一点做出了改进，只要将码点放入大括号，就能正确解读该字符。
```javascript
let a = '\u{20BB7}';
console.log(a); // 𠮷
```
<br>

## 字符串的遍历器接口

ES6 为字符串添加了遍历器接口，使得字符串可以被`for...of`循环遍历。

```javascript
for (let c of 'hello') {
  console.log(c);
}
// h
// e
// l
// l
// o
```

关键是，`for...of`可以正确的识别码点，而普通`for`循环不可以：

```javascript
let str = String.fromCodePoint(0x20bb7);
for (let i = 0; i < str.length; i++) {
  console.log(str[i]);
}
// �
// �

for(let c of str){
  console.log(c);
}
// 𠮷
```
<br>

## 模板字符串
`ES6`中可以使用反引号(**`**)来定义字符串，并且可以在其中嵌入变量，函数等。

<!--more-->

#### 基本用法
```javascript
let str1 = `hello world!`;
console.log('单行字符串', str1); // 单行字符串 hello world!

let str2 = `hello
world!`
console.log('多行字符串', str2); 
// 多行字符串 hello
// world!

let name = 'world'
let str3 = `hello ${name}!`
console.log('带变量的模板字符串', str3); // 带变量的模板字符串 hello world!

function sayHello(name) {
  return `hello ${name}!`;
}
let str4 = `${sayHello('world')}`
console.log('带函数的模板字符串', str4); // 带函数的模板字符串 hello world!
```
#### 标签模板
模板字符串可以紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串。这被称为“标签模板”功能（tagged template）。

**标签模板的解析规则**

- 模板字符串中，非变量`${}`部分会被变量分割后，放进一个数组中，当做函数的第一个参数。
- 变量部分，从第二个参数开始，依次传入函数中。
- 下面的代码中，第二个参数采取了`rest`的写法，正常应该是有多少个变量，后面就跟多少个参数的。

```javascript
let age = 18;
let name = 'zm';
let weight = '65';

function userInfo(normalArr, ...data) {
  console.log(normalArr, data) // [ '用户的基本信息为:年龄=', '姓名=', '体重', 'kg' ] [ 18, 'zm', '65' ]
}

userInfo`用户的基本信息为:年龄=${age}姓名=${name}体重${weight}kg`
```

**raw属性**

- 标签模板第一个参数有`raw`属性
- 区别是第一个参数数组不会对特殊字符进行转义。换行符`\n`会换行。
- `raw`属性会对特殊字符转义。换行符`\n`会被转义为`\\n`，输出为`\n`字符串。

```javascript
function rawStr(stringArr) {
  console.log(stringArr[0], stringArr)
  console.log(stringArr.raw[0], stringArr.raw)
}

rawStr`第一行\n第二行`
// 第一行
// 第二行 [ '第一行\n第二行' ]
// 第一行\n第二行 [ '第一行\\n第二行' ]
```
<br>
## 字符串的新增方法
#### String.fromCodePoint()
`ES5` 提供`String.fromCharCode()`方法不能识别大于`0XFFFF`的码点，所以`ES6`补充了这个方法：
```javascript
console.log(String.fromCharCode(0x20bb7)); // ஷ
console.log(String.fromCodePoint(0x20bb7)); // 𠮷
```
#### String.raw()
会返回一个斜杠都被转义的字符串：
```javascript
let str = String.raw`第一行\n第二行`
console.log(`第一行\n第二行`); 
// 第一行
// 第二行
console.log(str); // 第一行\n第二行
```
#### codePointAt()
```javascript
let str = '𠮷a';
console.log(str.length)
console.log(str.charAt(0), str.charAt(1), str.charAt(2))
console.log(str.charCodeAt(0).toString(16), str.charCodeAt(1).toString(16), str.charCodeAt(2).toString(16))
console.log(str.codePointAt(0).toString(16), str.codePointAt(1).toString(16), str.codePointAt(2).toString(16))
// 3
// � � a
// d842 dfb7 61
// 20bb7 dfb7 61
```
从上面的代码可以看到，`JavaScript`中各种处理字符的方式，`𠮷`这个字符是有四个字节存储的，以`utf-16`表示应该为`0xD842 0xDFB7`,所以长度为3。
虽然`ES6`提供了`codePointAt()`来识别四个字节存储的字符，但是他的下标为2处，并不是字母`a`，所以还是不太对。
可以使用`for...of`：
```javascript
let s = '𠮷a';
for (let ch of s) {
  console.log(ch.codePointAt(0).toString(16));
}
// 20bb7
// 61
```
那``codePointAt`有什么用呢？
它可以用来判断一个字符是由两个字节还是四个字节组成：
```javascript
let s1 = '𠮷';
let s2 = 'a';

function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}

console.log(is32Bit(s1)); // true
console.log(is32Bit(s2)); // false
```
#### includes(), startsWith(), endsWith()
- **includes()**：返回布尔值，表示是否找到了参数字符串。第二个参数表示从第几个下标位置开始，包括该位置的值。
- **startsWith()**：返回布尔值，表示参数字符串是否在原字符串的头部。第二个参数表示从第几个下标位置开始，包括该位置的值。
- **endsWith()**：返回布尔值，表示参数字符串是否在原字符串的尾部。第二个参数表示从第几个下标位置开始，不包括该位置的值。
- 以上方法的第二个参数都可以不传，默认为0。
```javascript
let s = 'Hello world!';
console.log(s.startsWith('world', 6)) // true  
console.log(s.endsWith('Hello', 5)) // true   
console.log(s.includes('Hello', 6))// false   
console.log(s.includes('world', 6))// true 
```
#### repeat()
`repeat`方法返回一个新字符串，表示将原字符串重复`n`次。
使用比较简单，参数表示重复几次：
- 小数会被取整
- 字符串会转为数字
- `NAN`等共同与0
- `Infinity`和负数会报错
```javascript
console.log('hello'.repeat(2)); // hellohello
console.log('hello'.repeat(2.5)) // hellohello
console.log('hello'.repeat(NaN)) // ''
```
#### padStart()，padEnd()
<font color = rgba(154,69,11)>emm...这个是`ES2017(ES8)`的功能。</font>
也比较简单：
- 第一个参数表示，补全后的最大长度
- 第二参数表示，用来补全的字符串
```javascript
'x'.padStart(5, 'ab') // 'ababx'
'x'.padStart(4, 'ab') // 'abax'

'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba'

'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"
'123456'.padStart(10, '0') // "0000123456"
```
#### trimStart()，trimEnd()
- `trimStart()`只消除头部的空格
- `trimEnd()`只消除尾部的空格
```javascript
const s = '  abc  ';

s.trim() // "abc"
s.trimStart() // "abc  "
s.trimEnd() // "  abc"
```
#### matchAll()
这个我在node 10.x 没有跑通，然后去看了下这是`ES2020`才提出的，node 12.x 才支持。
我也不知道ES6指南为什么有这么多后面的新特性，等正则那一章再简单介绍一下吧。

ps: 我觉得这些新特性，模板字符串我平时用的最多的，但是之前确实不知道标签模板这个功能。