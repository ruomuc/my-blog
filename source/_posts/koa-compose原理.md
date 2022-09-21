---
title: koa-compose原理
categories:
  - node.js
tags:
  - node.js
  - koa
date: 2020-10-01 20:32:53
updated: 2020-10-01 20:32:53
cover: https://proxy.qnoss.seeln.com/images/TNoSBtf-black-hole-wallpaper.jpg
---

`koa`是一个很优秀的轻量级web框架，一般我自己写个小东西都会用它。

它的中间件执行顺序就和一个洋葱一样，所以被称为洋葱模型，但是一直不知道它的原理是什么，有时间就看了一下。

<br>

## 中间件的注册

`koa`中注册很简单，使用`app.use(xxx)`就可以注册一个中间件。

源码如下：

```js
use(fn) {
  if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
  if (isGeneratorFunction(fn)) {
    deprecate('Support for generators will be removed in v3. ' +
              'See the documentation for examples of how to convert old middleware ' +
              'https://github.com/koajs/koa/blob/master/docs/migration.md');
    fn = convert(fn);
  }
  debug('use %s', fn._name || fn.name || '-');
  this.middleware.push(fn);
  return this;
}
```

1. 判断是不是一个`'function'`，如果不是报错`middleware must be a function!`。
2. 判断这个函数是不是一个生成器函数即`Generator`函数，如果不是，将其转为`Generator`函数。
3. 将中间件函数放入一个全局的`middleware`数组中。

还没完，还需要把这个`middleware`数组，使用`koa-compose`这个方法来加工一下，才可以实现洋葱模型。
<!--more-->
```js
const compose = require('koa-compose');

listen(...args) {
  debug('listen');
  const server = http.createServer(this.callback());
  return server.listen(...args);
}
callback() {
  const fn = compose(this.middleware);

  if (!this.listenerCount('error')) this.on('error', this.onerror);

  const handleRequest = (req, res) => {
    const ctx = this.createContext(req, res);
    return this.handleRequest(ctx, fn);
  };

  return handleRequest;
}
```

1. 在调用`app.listen`来监听端口时，有一个`callback`函数会被执行。
2. 在`callback`函数中的第一行，使用了`compose(this.middleware)`来处理我们注册的中间件函数。

<br>

## koa-compose源码分析

源码很简洁：

```js
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */

  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

1. 首先判断`middleware`是不是一个数组，不是数组抛出错误`Middleware stack must be an array!`。
2. 逐个判断`middleware`里的元素是不是一个函数，不是的话，抛出错误`Middleware must be composed of functions!`。
3. 返回一个函数
   1. 使用`index`巧妙的避免了`next`执行两次的问题。
   2. `if (i === middleware.length) fn = next`,`if (!fn) return Promise.resolve()`这两行应该是递归终止条件。

<br>

##### 洋葱模型原理

`koa`的中间件执行顺序大家都很清楚，但是一直没搞懂这段代码为什么会像那样数组，直到看到这样一个例子。。

原谅我，递归的代码脑子稍微没转过来就搞不清楚最终的执行结果。

```js
function test1() {
  console.log(1)
  test2();
  console.log(5)
  return Promise.resolve();
}
function test2() {
  console.log(2)
  test3();
  console.log(4)
}

function test3() {
  console.log(3)
  return;
}
test1();
```

上面的代码依次输出，`1 2 3 4 5`

再结合`compose`的源码，看一下，瞬间就明白了一大半。

其实就是把下一个中间件函数，当做`next`参数传入当前中间件函数，并且递归执行。



参考链接：

https://www.cnblogs.com/tugenhua0707/p/10204009.html

