---
title: Koa入门(一)
categories:
  - node.js
tags:
  - koa
date: 2018-08-26 22:33:25
updated: 2018-08-26 22:33:25

---
## koa的安装
安装
```
npm init 
npm install koa 
```
## 第一个hello world
```js
var Koa = require('koa');
var app = new Koa();

app.use(ctx => {
    console.log(ctx.request,ctx.response);
    ctx.body = 'hello world!';
})

app.listen(3000);
console.log('server is listen on 3000');

```
控制台结果:
```
λ node koatest2.js
server is listen on 3000
```
浏览器输入127.0.0.1:3000：
```
hello world!
```
ps: `var Koa = require('koa');` 如果`K`是大写就是koa2，小写就是koa，详细差异自己百度啦。

## Koa中间件的级联
洋葱模型：
<!--more-->
![](//image.seeln.com/images/yangcongmoxing.png)

比如下面这段代码：
```js
var Koa = require('koa');
var app = new Koa();

//X-response-time 
app.use(async (ctx, next) => {
	//（1）
    var start = new Date;
    await next();
    //（5）
    var ms = new Date - start;
    ctx.set('X-Response-Time', 10 + 'ms');
});

//logger 
app.use(async (ctx, next) => {
	//（2）
    var start = new Date;
    await next();
    //（4）
    var ms = new Date - start;
    console.log('%s %s - %s', ctx.method, ctx.url, ms);
});

//response
app.use(async (ctx) => {
	//（3）
    ctx.body = 'hello world';
});

app.listen(3000);
console.log('server is listen on 3000');

```
控制台输出：
```
λ node koatest2.js
server is listen on 3000
GET / - 2
GET /favicon.ico - 1
```
执行顺序如图中：(1)->(2)->(3)->(4)->(5)。

## 关于async和await
不说其他的异步写法，单说这个async的写法，我的个人理解是:
比如有这样一段代码：
```js

//通过id获取信息
function getInfoById(id,callback){
	...
	...
	callback(info);//一系列操作返回用户信息
}
//通过信息获取姓名
function getUserNameByInfo(info,callback){
	...
	...
	callback(name); //一系列操作之后返回了一个name
}

//这是如果我有一个id要获取name就要这么做
function getUserNameById(id){
	getInfoById(id,function(data){
		if(data){ 
			var info = data; //data => info
			getUserNameByInfo(info,function(data){
				if(data){
					var name = data; // data=>name
					console.log(name); //用户姓名
				}
			});
		}
	});
}

```
上面这种就是最常规的js异步回调函数，因为下一个函数的参数依赖于上一个函数的执行结果，如果不写成回调，就会报错，说值为undefined等。
然后es7引入新语法，注意是es7规范。node v8以上才支持，低版本v7有其他方式支持，但是现在的新版node.js是天然支持的，很好用！！！
比如上面那段代码：
```js
//通过id获取信息
function getInfoById(id,callback){
	...
	...
	callback(info);//一系列操作返回用户信息
}
//通过信息获取姓名
function getUserNameByInfo(info,callback){
	...
	...
	callback(name); //一系列操作之后返回了一个name
}

//这是如果我有一个id要获取name就要这么做
aync function getUserNameById(id){ 
	let info = await getInfoById(id);
	let name = await getUserNameByInfo(info);
	console.log(name);
}
```
简直惊呆了有木有！谁他喵的还写回调地狱哦。![回调地狱了解一下](//image.seeln.com/images/huidiaodiyu.jpg)

上面那种写法要注意：
- async 卸载function前面代表这是一个异步函数
- await只有在异步函数里面才可以使用，不然会报错的哦
- 暂时不知道了，想到了再补充...


PS：其实koa我用别人的demo都搭了一个[主页](http://www.ruomu.cc)了，只不过不太好看，并且是前后端一起写的，很乱。。有时间了换一个，然后再系统的看一下这个框架，学新不学旧嘛。。况且express的话我做项目也有接触到。基数大但是总是要被迭代掉的吧。

