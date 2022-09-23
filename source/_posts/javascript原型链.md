---
title: javascript原型链
categories:
  - javascript
tags:
  - javascript
  - 原型链
date: 2020-05-21 18:33:13
updated: 2020-05-21 18:33:13

---

阮一峰老师的说原型链开始是为了解决继承的，因为JavaScript没有子类父类的概念，只有原型对象。

看着别人的讲解，自己跟着画了张图加深理解，对比着图比较容易理解。

<img src="https://image.seeln.com/images/prototype.png" style="zoom:25%;" />

#### 什么是`prototype`、`__proto__`和`constructor`?

- `__proto__`:事实上就是原型链指针

- `prototype`:是原型对象

- `constructor`: 每一个原型对象都包含一个指向`构造函数`的指针

- 大多说情况下，`__proto__`可以理解为构造器原型，举个例子。

  - ```
    function A(){}
    
    var a = new A();
    
    a.__proto__ === A.prototype;  // 这里A是a的构造函数
    ```

#### JavaScript的构造函数？

​	在java中，构造函数可以理解为，实例化类时用来初始化类的函数，但是JavaScript中，构造函数是指**使用`new`来调用的函数，和函数无关，只和调用方式有关。**

#### `prototype`和`__proto__`的区别?

- `prototype`只有函数才有这个属性
- `__proto__`每个对象都有这个属性

<!--more-->
#### `__proto__`属性指向谁?

`__proto__`指向取决于对象创建时的实现方式

##### 字面量方式

`var a = {};` 等价于`var a = new Object();`

其实 `a`就是的构造函数就是`function Object(){}`

根据图可知`a.__proto__ === Object.prototype`


##### 构造器方式
```javascript
var A = function(){}
var a = new A()
```
这里栗子上面说过，没什么特别的。
其实这样看，字面量方式和构造器方式有点类似。

##### `Object.create`方式
```javascript
var a1 = {};
var a2 = Object.create(a1);
```

#### 最后

看着这段代码和我画的图，看懂了基本就懂了。。。

```javascript
var A = function(){};
var a = new A();
console.log(a.__proto__); //A {}（即构造器function A 的原型对象）
console.log(a.__proto__.__proto__); //Object {}（即构造器function Object 的原型对象）
console.log(a.__proto__.__proto__.__proto__); //null

console.log(A.__proto__.__proto__.__proto__); //null

console.log(a.__proto__===A.prototype); // true
console.log(a.__proto__.__proto===A.__proto__.__proto__); // true
```

参考链接:

- [三张图搞懂JavaScript的原型对象与原型链](https://www.cnblogs.com/shuiyi/p/5305435.html)
- [js中__proto__和prototype的区别和关系？](https://www.zhihu.com/question/34183746)
- [Javascript继承机制的设计思想](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html)
