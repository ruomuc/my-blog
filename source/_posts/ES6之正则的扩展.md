---
title: ES6之正则的扩展
categories:
  - JavaScript
tags:
  - JavaScript
  - es6
date: 2020-06-30 20:24:54
updated: 2020-06-30 20:24:54

---
## RegExp构造函数

在ES5中我们可以这样声明一个正则表达式：

```javascript
var regex = new RegExp('xyz', 'i');
// 等价于
var regex = /xyz/i;

var regex = new RegExp(/xyz/i);
// 等价于
var regex = /xyz/i;
```

但是这样会报错，因为第一个参数不是字符串，而是一个正则表达式：

```javascript
var regex = new RegExp(/xyz/, 'i');
```

但是在ES6中支持这种行为，并且如果存在第二个参数，会忽略原有的修饰符：

```javascript
var reg = new RegExp(/abc/ig, 'i');
console.log(reg); // /abc/i
```
<br>

## [字符串的正则方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/@@split#)

####  match

match 方法会返回一个数组，它包括整个匹配结果，和通过捕获组匹配到的结果，如果没有匹配到则返回null

这个方法在 [`String.prototype.match()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/match) 的内部调用。例如，下面的两个方法返回相同结果。

```javascript
'abc'.match(/a/);
/a/[Symbol.match]('abc');
```

#### replace

返回用替换器替换相应匹配项后的新字符串。

```js
'abc'.replace(/a/, 'A');
/a/[Symbol.replace]('abc', 'A');
```
<!--more-->
#### search

如果成功的话，`[@@search]()` 返回该正则模式的第一个匹配项的在字符串中的位置索引。否则将返回-1。

```js
'abc'.search(/a/);
/a/[Symbol.search]('abc');
```

#### split

切割字符串

返回包含其子字符串的数组。

```js
'a-b-c'.split(/-/); // ["a", "b", "c"]
/-/[Symbol.split]('a-b-c');
```
<br>

## 修饰符

### u修饰符

ES6 对正则表达式添加了`u`修饰符，含义为“Unicode 模式”，用来正确处理大于`\uFFFF`的 Unicode 字符。

```javascript
/^\uD83D/u.test('\uD83D\uDC2A') // false
/^\uD83D/.test('\uD83D\uDC2A') // true
```

ES5 不支持四个字节的 UTF-16 编码，会将其识别为两个字符，导致第二行代码结果为`true`。

##### RegExp.prototype.unicode

查看正则对象是否设置了`u`修饰符。

```javascript
const r1 = /hello/;
const r2 = /hello/u;

r1.unicode // false
r2.unicode // true
```

### i修饰符

表示不区分大小写

并且和`u`修饰符一起用，可以识别非规范的字符

```js
/[a-z]/i.test('\u212A') // false
/[a-z]/iu.test('\u212A') // true
```

### y修饰符

`y`修饰符也叫做粘连修饰符

```js
var s = 'aaa_aa_a';
var r1 = /a+/g;
var r2 = /a+/y;

r1.exec(s) // ["aaa"]
r2.exec(s) // ["aaa"]

r1.exec(s) // ["aa"]
r2.exec(s) // null
```

使用`y`修饰符每次匹配，都是从剩余字符串的头部开始。所以上面代码的第二次匹配的头部是`_`，故匹配不到。

`y`修饰符的设计本意，就是让头部匹配的标志`^`在全局匹配中都有效。

##### RegExp.prototype.sticky

与`y`修饰符相匹配，ES6 的正则实例对象多了`sticky`属性，表示是否设置了`y`修饰符。

```javascript
var r = /hello\d/y;
r.sticky // true
```

##### RegExp.prototype.flags

```javascript
// ES5 的 source 属性
// 返回正则表达式的正文
/abc/ig.source
// "abc"

// ES6 的 flags 属性
// 返回正则表达式的修饰符
/abc/ig.flags
// 'gi'
```

### s修饰符

ES2018 [引入](https://github.com/tc39/proposal-regexp-dotall-flag)`s`修饰符，使得`.`可以匹配任意单个字符。

正则表达式中，点（`.`）是一个特殊字符，代表任意的单个字符，但是有两个例外。一个是四个字节的 UTF-16 字符，这个可以用`u`修饰符解决；另一个是行终止符（line terminator character）。

```js
/foo.bar/s.test('foo\nbar') // true
```
<br>

## 后行断言

ES2018引入后行断言

##### 什么是先行断言

“先行断言”指的是，`x`只有在`y`前面才匹配，必须写成`/x(?=y)/`。比如，只匹配百分号之前的数字，要写成`/\d+(?=%)/`。“先行否定断言”指的是，`x`只有不在`y`前面才匹配，必须写成`/x(?!y)/`。比如，只匹配不在百分号之前的数字，要写成`/\d+(?!%)/`。

```javascript
/\d+(?=%)/.exec('100% of US presidents have been male')  // ["100"]
/\d+(?!%)/.exec('that’s all 44 of them')                 // ["44"]
```
上面两个字符串，如果互换正则表达式，就不会得到相同结果。另外，还可以看到，“先行断言”括号之中的部分（(?=%)），是不计入返回结果的。
##### 什么是后行断言

“后行断言”正好与“先行断言”相反，`x`只有在`y`后面才匹配，必须写成`/(?<=y)x/`。比如，只匹配美元符号之后的数字，要写成`/(?<=\$)\d+/`。“后行否定断言”则与“先行否定断言”相反，`x`只有不在`y`后面才匹配，必须写成`/(?<!y)x/`。比如，只匹配不在美元符号后面的数字，要写成`/(?<!\$)\d+/`。

```javascript
/(?<=\$)\d+/.exec('Benjamin Franklin is on the $100 bill')  // ["100"]
/(?<!\$)\d+/.exec('it’s is worth about €90')                // ["90"]
```

上面的例子中，“后行断言”的括号之中的部分（`(?<=\$)`），也是不计入返回结果。
<br>

## 具体名匹配

ES2018 引入了[具名组匹配](https://github.com/tc39/proposal-regexp-named-groups)（Named Capture Groups），允许为每一个组匹配指定一个名字，既便于阅读代码，又便于引用。

不使用具体组匹配的写法：

```javascript
const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/;

const matchObj = RE_DATE.exec('1999-12-31');
const year = matchObj[1]; // 1999
const month = matchObj[2]; // 12
const day = matchObj[3]; // 31
```

matchObj的值是一个类数组对象：

```js
[ '1999-12-31',
  '1999',
  '12',
  '31',
  index: 0,
  input: '1999-12-31',
  groups: undefined ]
```

使用具体组匹配的写法：

```javascript
const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;

const matchObj = RE_DATE.exec('1999-12-31');
const year = matchObj.groups.year; // 1999
const month = matchObj.groups.month; // 12
const day = matchObj.groups.day; // 31
```

matchObj的值是一个类数组对象，但是groups属性有值：

```js
[ '1999-12-31',
  '1999',
  '12',
  '31',
  index: 0,
  input: '1999-12-31',
  groups: [Object: null prototype] { year: '1999', month: '12', day: '31' } ]
```

##### 解构赋值和替换

有了具名组匹配以后，可以使用解构赋值直接从匹配结果上为变量赋值。

```javascript
let {groups: {one, two}} = /^(?<one>.*):(?<two>.*)$/u.exec('foo:bar');
one  // foo
two  // bar
```

##### 引用

如果要在正则表达式内部引用某个“具名组匹配”，可以使用`\k<组名>`的写法。

```javascript
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>$/;
RE_TWICE.test('abc!abc') // true
RE_TWICE.test('abc!ab') // false

// 还可以这样写

const RE_TWICE = /^(?<word>[a-z]+)!(\1)!\2$/;
console.log(RE_TWICE.test('abc!abc!abc')) // true
console.log(RE_TWICE.test('abc!abc!ab')) // false
```

数字引用（`\1`）依然有效。

`\1`是反向引用的意思，表示获取第一个`()`匹配的引用。

```javascript
const RE_TWICE = /^(?<word>[a-z]+)!\1$/;
RE_TWICE.test('abc!abc') // true
RE_TWICE.test('abc!ab') // false
```

当然这两种引用语法还可以同时使用：

```javascript
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>!\1$/;
RE_TWICE.test('abc!abc!abc') // true
RE_TWICE.test('abc!abc!ab') // false
```

ps: 正则的用法妙不可言，平时用的比较少，就记一下这些常用的吧。
