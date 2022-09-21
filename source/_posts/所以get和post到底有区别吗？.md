---
title: 所以get和post到底有区别吗？
categories:
  - node.js
tags:
  - http
date: 2018-11-09 18:41:59
updated: 2018-11-09 18:41:59
cover: https://proxy.qnoss.seeln.com/images/wp4221759-asteroid-belt-wallpapers.jpg
---

## 如题

http://www.cnblogs.com/logsharing/p/8448446.html

说白了就是语义区别，和大家的约定，你可以不按套路来鸭。


ps:顺便整理几个问题

## POST 和 PUT 有什么区别?

POST 是新建 (create) 资源, 非幂等, 同一个请求如果重复 POST 会新建多个资源. PUT 是 Update/Replace, 幂等, 同一个 PUT 请求重复操作会得到同样的结果.

ps: 表面来看 post类似于mysql的insert，put类似于update呗？

## cookie 与 session 的区别? 服务端如何清除 cookie?

主要区别在于, session 存在服务端, cookie 存在客户端. session 比 cookie 更安全. 而且 cookie 不一定一直能用 (可能被浏览器关掉). 服务端可以通过设置 cookie 的值为空并设置一个及时的 expires 来清除存在客户端上的 cookie.


## 什么是跨域请求? 如何允许跨域?

出于安全考虑, 默认情况下使用 XMLHttpRequest 和 Fetch 发起 HTTP 请求必须遵守同源策略, 即只能向相同 host 请求 (host = hostname : port) 注[1]. 向不同 host 的请求被称作跨域请求 (cross-origin HTTP request)。

比如express的解决方法：

```js

var express = require('express');

var app = express()

//设置跨域访问

// all方法表示:所有请求都必须通过该中间件，参数中的“*”表示对所有路径有效

app.all('*', function (req, res, next) { // 回调函数的三参数:request对象、response对象、next回调函数

    res.header("Access-Control-Allow-Origin", "*");

    res.header("Access-Control-Allow-Headers", "X-Requested-With");

    res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");

    res.header("X-Powered-By", ' 3.2.1');

    res.header("Content-Type", "application/json;charset=utf-8");

    next(); // 将request对象再传给下一个中间件

});

```

## socket hang up

在 Node.js 中当你要 response 一个请求的时候, 发现该这个 socket 已经被 "挂断", 就会就会报 socket hang up 错误.

ps:一次http请求只能res.send()一次。





