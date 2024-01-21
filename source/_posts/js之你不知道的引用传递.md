---
title: js之你不知道的引用传递
categories:
  - JavaScript
tags:
  - JavaScript
date: 2019-05-22 22:49:03
updated: 2019-05-22 22:49:03

---

之前有一篇文章[深拷贝和浅拷贝]([https://ruomuc.gitee.io/blog/2018/06/26/%E6%B7%B1%E6%8B%B7%E8%B4%9D%E5%92%8C%E6%B5%85%E6%8B%B7%E8%B4%9D/](https://ruomuc.gitee.io/blog/2018/06/26/深拷贝和浅拷贝/))说明了js中传值和传引用，或者叫值传递和引用传递。

可能是我闲的蛋疼吧。

假设有这样一段代码

```js
var a = 1;
var obj1 = {item:true}
var obj2 = {item:false}

function changeValue(a,obj1,obj2){
	a = 123;
  obj1 = {item:false}
  obj2.item = true;
}
changeValue(a,obj1,obj2)
console.log(a)
console.log(obj1)
console.log(obj2)
```

执行结果是

```
123
{item:true}
{item:true}
```

关于传值、传地址、传引用。我是在一片[c++的博文](<https://blog.csdn.net/zx3517288/article/details/53363798>)找到的，不知道准不准确。

按文中说法，如果是传引用的话，没有实参的拷贝，对形参的修改必然反映到实参上，而这里没有。而传地址，只有对形参指向的对象的修改，才会影响实参，对形参本身的修改(这里的obj2修改的不是自己，而是自己指向的对象)，并不会影响实参(这里的obj1指向了另一个对象，也就是更新了变量保存的地址)。

所以我觉得这里因该是传地址。

顺便提一下，我在看java的时候，所有地方都说java只有值传递！！！

传地址是不是也是一种传值呢，java的非基本数据类型，栈上的变量存的也是对象在堆中的地址。

再贴一条我在stackoverflow上看到的讨论帖[Is JavaScript a pass-by-reference or pass-by-value language?](https://stackoverflow.com/questions/518000/is-javascript-a-pass-by-reference-or-pass-by-value-language)，我觉得上面解释是符合这段代码的结果的。

欢迎来怼。。。

