---
title: node.js面试题整理(二)
categories:
  - node.js
tags:
  - 面试题
date: 2018-07-03 13:07:57
updated: 2018-07-03 13:07:57
cover: https://proxy.qnoss.seeln.com/images/mJMhjjY-black-hole-wallpaper.jpg
---

## 模块机制
###### 其实官方文档已经说得很清楚了, 每个 node 进程只有一个 VM 的上下文, 不会跟浏览器相差多少, 模块机制在文档中也描述的非常清楚了:
```js
function require(...) {
  var module = { exports: {} };
  ((module, exports) => {
    // Your module code here. In this example, define a function.
    function some_func() {};
    exports = some_func;
    // At this point, exports is no longer a shortcut to module.exports, and
    // this module will still export an empty default object.
    module.exports = some_func;
    // At this point, the module will now export some_func, instead of the
    // default object.
  })(module, module.exports);
  return module.exports;
}
```
##### <font color = red>如果 a.js require 了 b.js, 那么在 b 中定义全局变量 t = 111 能否在 a 中直接打印出来?</font>
###### 每个 .js 能独立一个环境只是因为 node 帮你在外层包了一圈自执行, 所以你使用 t = 111 定义全局变量在其他地方当然能拿到. 情况如下:
```js
// b.js
(function (exports, require, module, __filename, __dirname) {
  t = 111;
})();

// a.js
(function (exports, require, module, __filename, __dirname) {
  // ...
  console.log(t); // 111
})();
```
  
  
</br>
##### <font color = red>a.js 和 b.js 两个文件互相 require 是否会死循环? 双方是否能导出变量? 如何从设计上避免这种问题?</font>
###### CommonJS模块的重要特性是加载时执行，即脚本代码在require的时候，就会全部执行。CommonJS的做法是，一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。
```js
//a.js
exports.done = false;
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 执行完毕');
```
###### 上面代码之中，a.js脚本先输出一个done变量，然后加载另一个脚本文件b.js。注意，此时a.js代码就停在这里，等待b.js执行完毕，再往下执行。再看b.js的代码。
```js
//b.js
exports.done = false;
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 执行完毕');
```
###### 上面代码之中，b.js执行到第二行，就会去加载a.js，这时，就发生了"循环加载"。系统会去a.js模块对应对象的exports属性取值，可是因为a.js还没有执行完，从exports属性只能取回已经执行的部分，而不是最后的值。a.js已经执行的部分，只有一行。
```js
exports.done = false;
```
###### 因此，对于b.js来说，它从a.js只输入一个变量done，值为false。然后，b.js接着往下执行，等到全部执行完毕，再把执行权交还给a.js。于是，a.js接着往下执行，直到执行完毕。我们写一个脚本main.js，验证这个过程。
```js
var a = require('./a.js');
var b = require('./b.js');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);
```
执行main.js,运行结果如下:
```js
$ node main.js

在 b.js 之中，a.done = false
b.js 执行完毕
在 a.js 之中，b.done = true
a.js 执行完毕
在 main.js 之中, a.done=true, b.done=true
```
###### 上面的代码证明了两件事。一是，在b.js之中，a.js没有执行完毕，只执行了第一行。二是，main.js执行到第二行时，不会再次执行b.js，而是输出缓存的b.js的执行结果，即它的第四行。
```js
exports.done = true;
```
> <font color = blue>ES6模块的运行机制与CommonJS不一样，它遇到模块加载命令import时，不会去执行模块，而是只生成一个引用。等到真的需要用到时，再到模块里面去取值。因此，ES6模块是动态引用，不存在缓存值的问题，而且模块里面的变量，绑定其所在的模块</font>

</br>
##### <font color = red>比较 AMD, CMD, CommonJS 三者的区别</font>
###### 1.CMD和AMD之间的差异
- commonjs的目标是制定一个js模块化的标准，它的目标制定一个可以同时在客服端和服务端运行的模块。这些模块拥有自己独立的作用域，也可以向顶层曝露出自己的api也就是module.exports。在ES6中common被制定为标准。但是在ES6还未被浏览器完美支持的情况下，commonjs规范之能在服务端发挥它的作用。比如在nodejs和webpack等中。而我们服务端的异步加载模块主要有2种加载方案，CMD和AMD，这两种规范的典型是seajs和rjs(requirejs)。这两种方案虽然都是加载模块的解决方案，但是还是有一些的差别。
- cmd是commonjs另外的一种模块加载方案，这个规范本身偏向于commonjs的规范。他以一个文件就是一个模块和ES6中标准的commonjs规范类似。
- AMD是为了弥补commonjs规范在浏览器中目前无法支持ES6的一种解决方案。异步模块定义规范（AMD）制定了定义模块的规则，这样模块和模块的依赖可以被异步加载。这和浏览器的异步加载模块的环境刚好适应（浏览器同步加载模块会导致性能、可用性、调试和跨域访问等问题）。requirejs是AMD规范的实践者，RequireJS 是一个JavaScript模块加载器。它非常适合在浏览器中使用，但它也可以用在其他脚本环境, 就像 Rhino and Node. 使用RequireJS加载模块化脚本将提高代码的加载速度和质量。

</br>
###### 2.require的实现原理
见[require的实现原理](http://ruomuc.gitee.io/blog/2018/07/03/require%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/)。
<!--more-->
##### <font color = red>热更新</font>
###### 从面试官的角度看, 热更新 是很多程序常见的问题. 对客户端而言, 热更新意味着不用换包, 当然也包含着 md5 校验/差异更新等复杂问题; 对服务端而言, 热更新意味着服务不用重启, 这样可用性较高同时也优雅和有逼格. 问的过程中可以一定程度的暴露应聘程序员的水平.

###### 从 PHP 转 node 的同学可能会有些想法, 比如 PHP 的代码直接刷上去就好了, 并没有所谓的重启. 而 node 重启看起来动作还挺大. 当然这里面的区别, 主要是与同时有 PHP 与 node 开发经验的同学可以讨论, 也是很好的切入点.

###### 在 Node.js 中做热更新代码, 牵扯到的知识点可能主要是 require 会有一个 cache, 有这个 cache 在, 即使你更新了 .js 文件, 在代码中再次 require 还是会拿到之前的编译好缓存在 v8 内存 (code space) 中的的旧代码. 但是如果只是单纯的清除掉 require 中的 cache, 再次 require 确实能拿到新的代码, 但是这时候很容易碰到各地维持旧的引用依旧跑的旧的代码的问题. 如果还要继续推行这种热更新代码的话, 可能要推翻当前的架构, 从头开始从新设计一下目前的框架.

###### 不过热更新 json 之类的配置文件的话, 还是可以简单的实现的, 更新 require 的 cache 可以实现, 不会有持有旧引用的问题, 可以参见我 2 年前写着玩的例子, 但是如果旧的引用一直被持有很容易出现内存泄漏, 而要热更新配置的话, 为什么不存数据库? 或者用 zookeeper 之类的服务? 通过更新文件还要再发布一次, 但是存数据库直接写个接口配个界面多爽你说是不是?

###### 所以这个问题其实本身其实是值得商榷的, 可能是典型的 X-Y Problem, 不过聊起来确实是可以暴露水平.

##### <font color = red>上下文</font>
###### 如果你已经了解 ①② 那么你也应该了解, 对于 Node.js 而言, 正常情况下只有一个上下文, 甚至于内置的很多方面例如 require 的实现只是在启动的时候运行了内置的函数.
###### 每个单独的 .js 文件并不意味着单独的上下文, 在某个 .js 文件中污染了全局的作用域一样能影响到其他的地方.
###### 而目前的 Node.js 将 VM 的接口暴露了出来, 可以让你自己创建一个新的 js 上下文, 这一点上跟前端 js 还是区别挺大的. 在执行外部代码的时候, 通过创建新的上下文沙盒 (sandbox) 可以避免上下文被污染:
<font color = red> ps: 所以说上下文简单的理解就是语文上的意思，当前作用域，比如变量的作用域，函数的作用域，进程运行的环境等。</font>

##### <font color = red>包管理</font>

为什么我装了全局, 但是提示我 not found
npm yarn
锁版本:为了使部署服务器时的版本和开发时的版本一致。
lerna：一个用户管理多个包模块的工具。
left-pad事件
greenkeeper 等


















