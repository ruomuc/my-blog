---
title: 吐槽一下javascript
categories:
  - javascript
tags:
  - javascript
  - es6
date: 2020-08-23 16:27:22
updated: 2020-08-23 16:27:22

---

最近在熟悉新公司的一些基础服务和核心库的代码，说真的想吐槽一下`JavaScript`了，风格太灵活多变了，一样的操作可以用`n`多种方式或者语法实现，读代码太痛苦了。。

这不是上次面腾讯被问到`defineProperty`这种定义属性的方式和普通的`.` 操作有什么不同，刚好想起来我就搜了一下，给我惊到了。。

<br>

## JavaScript对象属性定义

[JavaScript文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

[如何理解Object.defineProperty()?](https://juejin.im/post/6844904145191698446)

基本的语法和操作没什么好说的。

除了`defineProperty`之外，还有`defineProperties`、`getOwnPropertyDescriptor`、`getOwnPropertyDescriptors`等配套的方法。

<br>

### `defineProperty`和`.`操作的区别

原来区别还真的挺大的。。


- 通过点操作符定义的属性，`writable`,`configurable`,`enumerable`值都为`true`，`value`为赋入的值

- 通过`Object.defineProperty`只指定`value`的属性，`writable`,`configurable`,`enumerable`值都为`false`

<br>

### console.log(a+a+a) //"abc"问题

说实话这个题我是第一次遇到。。只能说一句`JavaScript`博大精深，太秀了，一般这么写业务代码会被打死吧。。

下面的代码，在浏览器控制台中可以直接使用，但在`node.js`环境中，要使用`console.log(this.a + this.a + this.a)`，或者吧`this`全部换为`global`。

[代码出处](https://juejin.im/post/6844904145191698446#heading-48)

<br>
<!--more-->
## JavaScript的遍历方式

<br>

### for...in

`for...in`	其实是对象的遍历方式，**不建议用来遍历数组，因为无法保证顺序**

**遍历对象**

- 环遍历对象本身的所有**可枚举属性**
- 遍历对象**从其构造函数原型中继承**的属性
- 遍历顺序与Object.keys()函数取到的列表一致

**遍历数组**

- 不保证顺序
- 会忽略空元素
- 会遍历出来自定义属性

```js
// for...in遍历数组的缺陷
var arr = [11, 22]
arr.name = 12313
for (const key in arr) {
  console.log(key) // 0 1 name
}
for (const iterator of Object.keys(arr)) {
  console.log(iterator) // 0 1 name
}
```


### for...of

`for...of`是`es6`提供的一种通用的遍历方式，它可以遍历所有**可迭代对象**

**遍历数组**

- 有序的
- 不会忽略空值

**和for...in的区别**

- 推荐在遍历对象时使用`for...in`，遍历数组使用`for...of`。

- `for...in`遍历出来的是`key`，`for...of`遍历出来的是`value`
- `for...of`不能遍历普通对象

``` js
// for...of修复了这个缺陷
var arr = [11, 22]
arr.name = 12313
for (const iterator of arr) {
  console.log(iterator) // 11 22
}
```

<br>

### forEach

`forEach`是数组的一个高阶函数

**forEach和for...of遍历数组的区别**

- forEach中不能使用`break`、`return`、`continue`等语句。

<br>

### map

`map()` 方法创建一个新数组，其结果是该数组中的每个元素是调用一次提供的函数后的返回值。

和`lodash`的`map`功能一样呐好像。

**map() 方法参数与forEach完全相同，二者区别仅仅在于map会将回调函数的返回值收集起来产生一个新数组。**

<br>

### filter

`filter() `参数与`forEach`完全一致，不过它的`callback`函数应该返回一个真值或假值。

<br>

### find/findIndex

`find() `方法返回数组中使`callback`返回值为`Truthy`的第一个元素的值，没有则返回`undefined`。

`findIndex()`方法与`find`方法很类似,只不过`findIndex`返回使`callback`返回值为`Truthy`的第一个元素的索引，没有符合元素则返回`-1`。

<br>

### every/some/

两个函数接收参数都与以上函数相同，返回都是布尔值。

`every`用于判断是否数组中每一项都使得`callback`返回值为`Truthy`。

`some`用于判断是否至少存在一项使得`callback`元素返回值为`Truthy`。

<br>

### reduce/reduceRight 累加器

`reduce`和`reduceRight`的区别只是执行顺序相反。

`reduce`功能很强大，用法也可以很灵活多变。

<br>

#### 基本语法

[MDN web doc](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)

<br>


#### 扁平化数组

```js
function flatten(arr) {
  return arr.reduce((prev, item) => {
    return prev.concat(Array.isArray(item) ? flatten(item) : item)
  }, [])
}

const arr = [1, 2, [3, 4, 5], [2, [1, 98, 21]]]
console.log(flatten(arr)) // [ 1, 2, 3, 4, 5, 2, 1, 98, 21 ]

// reduceRight
function flatten(arr) {
  return arr.reduceRight((prev, item) => {
    return prev.concat(Array.isArray(item) ? flatten(item) : item)
  }, [])
}

const arr = [1, 2, [3, 4, 5], [2, [1, 98, 21]]]
console.log(flatten(arr)) // [ 21, 98, 1, 2, 5, 4, 3, 2, 1 ]
```

#### list to tree

```js

let list = [
  { id: 1, parentId: '' },
  { id: 2, parentId: '' },
  { id: 3, parentId: '1' },
  { id: 4, parentId: '2' },
  { id: 5, parentId: '3' }
]

function listToTree (list) {
  let nodeInfo = list.reduce((data, node) => {
    node.children = []
    data[node.id] = node
    return data
  }, {})
  let result = []
  for (const v of list) {
    if (!v.parentId) {
      result.push(nodeInfo[v.id])
      continue
    }
    nodeInfo[v.parentId].children.push(nodeInfo[v.id])
  }

  return result
}
console.log(JSON.stringify(listToTree(list)))

//[{"id":1,"parentId":"","children":[{"id":3,"parentId":"1","children":[{"id":5,"parentId":"3","children":[]}]}]},{"id":2,"parentId":"","children":[{"id":4,"parentId":"2","children":[]}]}]
```



<br>

## [Proxy](https://es6.ruanyifeng.com/#docs/proxy)和[Reflect](https://es6.ruanyifeng.com/#docs/reflect)

它们都是`es6`的新语法。

[JS 中的 Reflect 和 Proxy](https://juejin.im/post/6844903790739456013)

<br>

## [空值合并运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator)和[可选链式操作符]([https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/%E5%8F%AF%E9%80%89%E9%93%BE](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/可选链))

因为我在看代码的时候遇到了，查了一下，居然是`node14.0.0`才支持的语法。。

#### 可选链式操作符（?.）

其实看起来就是个快捷操作的语法，帮你判断了一下空值：

```js
var obj = {}
console.log(obj.a.a) // 报错
console.log(obj.a?.a) // undefined
            
// 等同于

var obj = {}
console.log(obj.a && obj.a.a) // undefined
console.log(obj.a ? obj.a.a : 0) // 0
```



#### 空值合并运算符

**空值合并操作符**（**`??`**）是一个逻辑操作符，当左侧的操作数为 [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null) 或者 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined) 时，返回其右侧操作数，否则返回左侧操作数。	

与[逻辑或操作符（`||`）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Logical_OR_2)不同，逻辑或操作符会在左侧操作数为[假值](https://developer.mozilla.org/zh-CN/docs/Glossary/Falsy)时返回右侧操作数。也就是说，如果使用 `||` 来为某些变量设置默认值，可能会遇到意料之外的行为。比如为假值（例如，`''` 或 `0`）时。

说白了：

- `??` 只会当左侧的值为`null`或是`undefined`时，才会返回其右侧结果
- `||` 当左侧为**假值**`('',0)等`时，返回右侧结果



彩蛋：在浏览器输出`(![]+{})[-~!+[]^-~[]]+([]+{})[-~!![]]` (~手动狗头)

end~

ps:  真的是不得不吐槽一下`JavaScript`。


