---
title: js对象的属性描述符
categories:
  - javascript
tags:
  - js
  - 属性描述符
date: 2021-02-21 15:24:48
updated: 2021-02-21 15:24:48
cover: https://proxy.qnoss.seeln.com/images/QTVoVUT-black-hole-wallpaper.jpg
---

## 属性描述符

JavaScript提供了一系列方法来修改和获取对象中属性的行为。

思考下面的代码：

```js
var myobj = {
    a:2
}
Object.getOwnPropertyDescriptor(myobj,'a')
//{
//    configurable: true
//    enumerable: true
//    value: 2
//    writable: true
//}
```

这就是**myobj**中属性"a"的描述符：

- `value:2` 代表a的值是2
- `enumerable: true`  代表属性a是可枚举的
- `configurable: true`  代表属性a是可配置的，也就是a的属性描述符是可以修改的，可以使用`defineProperty`来修改属性描述符。
- `writable: true`  代表属性a的值是可写的，也就是值是可以修改的

特殊说明：
<!--more-->
```js
var myobj = {
  a: 2
}

Object.defineProperty(myobj, 'a', {
  value: 2,
  writable: true,
  configurable: false,
  enumerable: true
})

myobj.a = 3
console.log(myobj.a)
Object.defineProperty(myobj, 'a', {
  value: 2,
  writable: false
})
myobj.a = 3
console.log(myobj.a)
console.log(Object.getOwnPropertyDescriptor(myobj,"a"))
delete myobj.a
console.log(myobj.a)

//3
//2
//{ value: 2, writable: false, enumerable: true, configurable: false }
//2
```

观察上面这段代码可以知道：

- 即便属性 `configurable:false` ，我们还是可以把 writable 状态由 true 改为 false
- 如果属性的`configurable: false` ，那么会禁止删除这个属性

### getter 和 setter

除了以上属性描述符，还有两个比较特殊的

观察以下代码：

```js
var obj = {
  get a () {
    return 2
  }
}
Object.defineProperty(obj, 'b', {
  enumerable: true,
  get: function () {
    return this.a * 2
  }
})
console.log(obj.a) // 2
console.log(obj.b) // 4
obj.a = 111
obj.b = 222
console.log(obj.a) // 2
console.log(obj.b) // 4
```

甚至可以这样：

```js
var obj = {
  a: 2
}

Object.defineProperty(obj, 'c', {
  enumerable: true,
  get: function () {
    this.a = this.a * 2
    return this.a
  }
})

console.log(obj.c) // 4
console.log(obj.c) // 8
console.log(obj.c) // 16
```

- 属性没有 value 值，也可以通过 get 描述符定义的函数来获取值。
- 如果只定义了get 而没有set 的话，那么无法修改属性值，所以get和set通常是同时出现的。
- 如果你同时定义了get/set 描述符，那么就不能出现value描述符，否则会报错。
- 使用set/get，可以定义很灵活且个性化的对象。。



## 不变性

在JavaScript中你有很多方法来使属性或者对象不可改变，但是所有方法创建的都是浅不变性，它们只会影响目标对象的直接属性。

如果目标对象引用了其他对象，其他对象的内容不受影响，仍然是可变的。

#### 对象常量

```js
var myobj = {
  a: 2
}
Object.defineProperty(myobj, 'a', {
  value: 2,
  writable: false,
  configurable: false,
  enumerable: true
})
```

使用 `writable: false`和`configurable: false`就可以创建一个不可修改和删除的常量属性。

#### 禁止扩展

```js
var myobj = {
  a:2
}
Object.preventExtensions(myobj)
myobj.b = 3
console.log(myobj.b)
```

使用`Object.preventExtensions(...)`可以禁止向一个对象添加新属性，并保留已有属性。

- 严格模式下添加新属性会报错
- 非严格模式下回静默失败

#### 密封

```js
var myobj = {
  a:2
}
Object.seal(myobj)
```

`Object.seal()` 等于下面两个的组合：

- `Object.preventExtensions()`
- `configurable: false`

密封之后，不仅不能添加新属性，也不可以重新配置和删除现有属性。但是可以修改现有属性的值。

#### 冻结

```js
var myobj = {
  a:2
}
Object.freeze(myobj)
```

`Object.freeze()` 等于下面三个的组合：

- `Object.preventExtensions()`
- `configurable: false`
- `writable: false`

冻结就是在密封的基础上，禁止修改现有属性的值。

冻结是你可以应用在对象上的级别最高的不可变性，会禁止对对象本身和其任意属性的修改。但是冻结对象引用的其他对象是不生效的。



## 存在性

#### in操作符

```js
var myobj = {
  a: 2
}
myobj.__proto__.b = 3
console.log("a" in myobj) // true
console.log("b" in myobj) // true
console.log("c" in myobj) // false
```

in 操作符会检查属性是否在对象及其[[Prototype]]原型链中。

#### hasOwnProperty

```js
var myobj = {
  a: 2
}
myobj.__proto__.b = 3
console.log(myobj.hasOwnProperty("a")) // true
console.log(myobj.hasOwnProperty("b")) // false
```

与 in 操作符不同，hasOwnProperty 只会检查属性是否在当前对象中。

#### 枚举属性

```js
var myobj = {
  a: 2,
  b: 3
}
myobj.__proto__.c= 4
Object.defineProperty(myobj, 'a', {
  value: 2,
  writable: true,
  configurable: false,
  enumerable: false
})

console.log("a" in myobj) // true
console.log(myobj.hasOwnProperty("a")) // true
console.log(myobj.propertyIsEnumerable("a")) // false
console.log(myobj.propertyIsEnumerable("b")) // true
console.log(Object.keys(myobj)) // [ 'b' ]
console.log(Object.getOwnPropertyNames(myobj)) // [ 'a', 'b' ]
```

#### 遍历

##### for...in

for...in 遍历对象的可枚举属性，包括原型链上的。

```js
var myobj = {
  a: 1,
  b: 2,
  c: 3
}
myobj.__proto__.d = 4
Object.defineProperty(myobj, 'c', {
  enumerable: false
})
for (let key in myobj) {
  console.log(key, '=', myobj[key])
}
```

如果不想遍历原型链上的属性：

```js
...
for (let key in myobj) {
  if (myobj.hasOwnProperty(key)) {
    console.log(key, '=', myobj[key])
  }
}
```

##### Object.keys()

使用Object.keys() 获取对象的所有可枚举属性（不包含原型链），然后使用普通的数据遍历获取key的值。

```js

var myobj = {
  a: 1,
  b: 2,
  c: 3
}
myobj.__proto__.d = 4
Object.defineProperty(myobj, 'c', {
  enumerable: false
})
for (const key of Object.keys(myobj)) {
  console.log(key, '=', myobj[key])
}
```

##### for of

```js
var obj = {}
for (const value of myobj) {}
```

这样是会报错的 `myobj is not iterable`，因为obj对象没有迭代器对象。

但是像其他的，例如 map set 和 数组，都会有迭代器对象：

```js
var arr = []
var map = new Map()
var set = new Set()
console.log(arr[Symbol.iterator]()) // Object [Array Iterator] {}
console.log(map[Symbol.iterator]()) // [Map Entries] {  }
console.log(set[Symbol.iterator]()) // [Set Iterator] {  }
```

那我我们可以手动添加一个迭代器对象：

```js
var obj = { a: 2, b: 3 }
Object.defineProperty(obj, Symbol.iterator, {
  enumerable: false,
  writable: false,
  configurable: true,
  value: function () {
    var that = this
    var idx = 0
    var ks = Object.keys(that)
    return {
      next: function () {
        return {
          value: that[ks[idx++]],
          done: idx > ks.length
        }
      }
    }
  }
})
for (const iterator of obj) {
  console.log(iterator) // 2 3
}
```



最后给个对比表格：

|                        | 仅可枚举的 | 仅对象自身（不包含原型链） | 说明               |
| ---------------------- | ---------- | -------------------------- | ------------------ |
| in                     | 否         | 否                         | 返回值为布尔值     |
| hasOwnProperty()       | 否         | 是                         |                    |
| Object.keys()          | 是         | 是                         |                    |
| getOwnPropertyNames()  | 否         | 是                         |                    |
| propertyIsEnumerable() |            | 是                         | 判断属性是否可枚举 |
| for...in               | 是         | 否                         | 忽略symbol属性     |

遍历原型链上所有 enumerable 为 true 或 false 的属性：

```js
var myobj = {
  a: 1,
  b: 2,
  c: 3
}
myobj.__proto__.d = 4
Object.defineProperty(myobj, 'c', {
  enumerable: false
})
```



参考链接：

- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Enumerability_and_ownership_of_properties