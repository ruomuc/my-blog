---
title: koa源码之delegates
categories:
  - node.js
tags:
  - node.js
  - koa
date: 2020-08-04 19:49:35
updated: 2020-08-04 19:49:35
cover: https://proxy.qnoss.seeln.com/images/VTJgx3C-black-hole-wallpaper.jpg
---

最近看koa的源码，看到了了一个`TJ`大神写的包[delegates](https://www.npmjs.com/package/delegates)，实现了[委托模式/代理模式](https://www.runoob.com/design-pattern/proxy-pattern.html)。

当前包的版本为`1.0.0`，代码就100多行，周下载量一千两百多万。。果然是巨佬。

<br>

## 基本结构

目录结构很简单，主要文件只有一个`index.js`和`test`文件夹下的单元测试。

`index.js`文件的内容：

- **Delegator**：构造方法
- **Delegator.auto**：自动委托所有可以委托的属性
- **Delegator.prototype.method**：委托一个方法
- **Delegator.prototype.getter**：委托属性的**getter**方法
- **Delegator.prototype.setter**：委托属性的**setter**方法
- **Delegator.prototype.access**: 同时委托属性的**getter**和**setter**方法
- **Delegator.prototype.fluent**：委托属性，实现jQuery风格，不传参数获取属性值，传参修改属性值。

ps: 在koa源码中，使用的也是`delegates`的`1.0.0`的包，但是`index.js`中没有找到`auto`这个方法。

<br>
<!--more-->

## 举例和实现

#### 构造方法

```js
function Delegator(proto, target) {
  if (!(this instanceof Delegator)) return new Delegator(proto, target);
  this.proto = proto;
  this.target = target;
  this.methods = [];
  this.getters = [];
  this.setters = [];
  this.fluents = [];
}
```

这是一个构造函数，`if (!(this instanceof Delegator)) return new Delegator(proto, target);`

这一行的目的是，可以使用`Delegator(proto,target)`来实例化一个对象。

#### **method**

源码没什么难度，核心代码：

```js
proto[name] = function () {
  return this[target][name].apply(this[target], arguments);
};
```

举个列子：

```js
const delegates = require('delegates')
var obj = {}
obj.request = {
  foo: (foo) => {
    return foo;
  }
}
delegates(obj, 'request').method('foo')
console.log(obj.foo('aaa')); // aaa
console.log(obj.request.foo('bbb')); // bbb
```

#### **getter**

举个例子：

```js
const delegates = require('delegates');

var obj = {}
obj.request = {
  foo: (foo) => {
    return foo;
  },
  phone: 'iphone11',
  get getter1() {
    return 'hello';
  }
}
Object.defineProperty(obj.request, 'getter', {
  get: function () {
    return this.phone;
  },
  enumerable: true
})
delegates(obj, 'request').getter('getter');
delegates(obj, 'request').getter('getter1');
console.log(obj.getter); // 'iphone11'
console.log(obj.getter1); // 'hello'
```

从上面的代码例子中可以知道，`get getter(){}`还是`Object.defineProperty`效果是一样的。

核心代码：

```js
proto.__defineGetter__(name, function () {
  return this[target][name];
});
```

[**__defineGetter__**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/__defineGetter__)方法可以将一个函数绑定在当前对象的指定属性上，当那个属性的值被读取时，你所绑定的函数就会被调用。

但是在`MDN web docs`文档中说它是一个非标准的方法，未来可能会停止支持。。。

所以这样改一下也没什么问题吧：

```js
Object.defineProperty(proto, name, {
  get: function () {
    return this[target][name]
  }
})
```

#### setter

举个例子：

```js
const delegates = require('delegates');

var obj = {}
obj.request = {
  foo: (foo) => {
    return foo;
  },
  phone: 'iphone11'
}
Object.defineProperty(obj.request, 'setter', {
  set: function (value) {
    this.phone = value;
  },
  enumerable: true
})
delegates(obj, 'request').setter('setter');
obj.setter = 'iphone12';
console.log(obj.request.phone); // iphone12
```

核心代码：

```js
proto.__defineSetter__(name, function (val) {
  return this[target][name] = val;
});
```

同理也可以改为：

```js
Object.defineProperty(proto, name, {
  get: function (val) {
    return this[target][name] = val;
  }
})
```

#### access

核心代码：

```js
return this.getter(name).setter(name);
```

没什么好说的，就是同时代理了`getter`和`setter`

#### fluent

核心代码也比较简单：

```js
proto[name] = function (val) {
  if ('undefined' != typeof val) {
    this[target][name] = val;
    return this;
  } else {
    return this[target][name];
  }
};
```

根据代码可以看出来，fluent代理的属性，传参时为获取值，不传参时为设置值。

#### auto

不过神奇的是，我直接下载的源码有这个方法并且有对应的测试用例，但是npm下载的包中确没有`auto`这个方法。

1. 新建一个`Delegator`对象，获取被委托对象的所有属性。
2. 循环遍历这些属性。
   1. 获取这个属性的详细描述(`Object.getOwnPropertyDescriptor`)。
   2. 如果有`get`方法，委托`getter`。
   3. 如果有`set`方法，委托`setter`。
   4. 如果有`value`属性：
      1. 如果`value`是`Function`类型，委托`method`。
      2. 如果不是委托`getter`。
      3. 如果这个属性是可写的`writable = true`，委托`setter`。

```js
Delegator.auto = function (proto, targetProto, targetProp) {
  var delegator = Delegator(proto, targetProp);
  var properties = Object.getOwnPropertyNames(targetProto);
  for (var i = 0; i < properties.length; i++) {
    var property = properties[i];
    var descriptor = Object.getOwnPropertyDescriptor(targetProto, property);
    if (descriptor.get) {
      delegator.getter(property);
    }
    if (descriptor.set) {
      delegator.setter(property);
    }
    if (descriptor.hasOwnProperty('value')) { // could be undefined but writable
      var value = descriptor.value;
      if (value instanceof Function) {
        delegator.method(property);
      } else {
        delegator.getter(property);
      }
      if (descriptor.writable) {
        delegator.setter(property);
      }
    }
  }
};
```

