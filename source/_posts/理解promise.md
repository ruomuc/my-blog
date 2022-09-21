---
title: 理解promise
categories:
  - javascript
tags:
  - js
  - promise
date: 2018-07-30 12:42:59
updated: 2018-08-26 22:28:01
cover: https://proxy.qnoss.seeln.com/images/wp4202354-aconcagua-wallpapers.jpg
---
## promise的基本理解
- es6原生支持Promise对象
- Promise 是一个容器，里面保存着某个未来才会结束的事件
- Promise 对象不受外界影响，它代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）、rejected（已失败）。只有异步操作的结果能是状态发生改变。并且状态一旦改变就不能不能在变化。状态的改变只有两种可能：从pending——>fulfilled/rejected。
	
## 基本用法
```
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```
###### Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。
###### resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。
	
    	
        
##### promise.then
```
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```
####### then有两个参数，第一个参数是成功时resolved调用，第二个参数是失败时rejected调用，第二个参数可选，他们的参数都是promise对象传出的值。

##### 一个promise 实现 ajax 的操作
```js
const getJSON = function(url) {
  const promise = new Promise(function(resolve, reject){
    const handler = function() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
    const client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```
##### 如果一个promise 实例的resolve返回是另一个promise 实例，那他的状态不取决与自己 而取决于返回的promise实例的状态
```js
const p1 = new Promise(function (resolve, reject) {
  // ...
});

const p2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})
```
```js
const p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

const p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail
```
<!--more-->
## Promise.prototype
###### 实例具有then方法，也就是说，then方法是定义在原型对象
```js
Promise {constructor: ƒ, then: ƒ, catch: ƒ, finally: ƒ, Symbol(Symbol.toStringTag): "Promise"}
catch:ƒ catch()
constructor:ƒ Promise()
all:ƒ all()
arguments:(...)
caller:(...)
length:1
name:"all"
__proto__:ƒ ()
[[Scopes]]:Scopes[0]
arguments:(...)
caller:(...)
length:1
name:"Promise"
prototype:Promise {constructor: ƒ, then: ƒ, catch: ƒ, finally: ƒ, Symbol(Symbol.toStringTag): "Promise"}
race:ƒ race()
reject:ƒ reject()
resolve:ƒ resolve()
Symbol(Symbol.species):(...)
get Symbol(Symbol.species):ƒ [Symbol.species]()
__proto__:ƒ ()
[[Scopes]]:Scopes[0]
finally:ƒ finally()
then:ƒ then()
Symbol(Symbol.toStringTag):"Promise"
__proto__:Object
```
- promise.prototype.then(arg1,arg2):
then方法的第一个参数是resolved状态的回调函数，第二个参数（可选）是rejected状态的回调函数。
- Promise.prototype.catch:
######Promise.prototype.catch方法是.then(null, rejection)的别名，用于指定发生错误时的回调函数。
Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。
```js
getJSON('/post/1.json').then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
```
- 一般来说，不要在then方法里面定义 Reject 状态的回调函数（即then的第二个参数），总是使用catch方法。
- Promise.prototype.finally()：
 finally方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。该方法是 ES2018 引入标准的。finally方法的回调函数不接受任何参数，这意味着没有办法知道前面的promise状态是什么。
```js
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
//
server.listen(port)
  .then(function () {
    // ...
  })
  .finally(server.stop);
 ```
------------2018年8月26日22:28:01 ----------------------------------------------------

更新了，看到一个超级大佬写的关于promise，是在看的是一知半解，不敢搬，就贴个链接吧：
[We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)。 我是用google翻译勉强看完的，纯英文的，我英文不好qaq。。