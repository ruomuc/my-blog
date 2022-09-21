---
title: node的多进程
categories:
  - node.js
tags:
  - node.js
date: 2018-12-31 16:30:15
updated: 2018-12-31 16:30:15

---
现行的软件架构主要有两种：多线程单进程(如：memcached、redis、mongodb等)和单线程多进程(nginx、node)。
多线程单进程的主要特点：

- 快：线程比进程轻量，它的切换开销要少很多。进程相当于函数间切换，每个函数拥有自己的变量；线程相当于一个函数内的子函数切换，它们拥有相同的全局变量。
- 灵活： 程序逻辑和控制方式简单，但是锁和全局变量同步比较麻烦。
- 稳定性不高： 由于只有一个进程，其内部任何线程出现问题都有可能造成进程挂掉，造成不可用。
- 性能天花板：线程和主程序受限2G地址空间；当线程到一定数量后，即使增加cpu也不能提升性能。

单线程多进程的主要特点：

- 高性能：没有频繁创建和切换线程的开销，可以在高并发的情况下保持低内存占用；可以根据CPU的数量增加进程数。
- 线程安全：没有必要对变量进行加锁解锁的操作
- 异步非阻塞：通过异步I/O可以让cpu在I/O等待的时间内去执行其他操作，实现程序运行的非阻塞
- 性能天花板：进程间的调度开销大、控制复杂；如果需要跨进程通信，传输数据不能太大。

虽然实际上node.js也不完全是单线程，只有js代码是单线程的而已，I/O等操作都丢到了一个C实现的叫Libuv的库里，和v8一样也是node的核心。

## 多进程架构
面对单进程对多核CPU利用不足的问题，就是启动多个进程即可。
node提供了child_process模块，通过child_process.fork()函数来进行进程的复制。
例如：
将以下代码保存为worker.js
```js
	var http = require('http');
	http.createServer(function(req,res){
		res.write(200,{'Content-Type':text/plain});
		res.end('Hello World\n');
	}).listen(Math.round((1+Math.random())*1000),'127.0.0.1');
	
```
将以下代码保存为master.js
```js
var fork = require('child_process').fork;
var cpus = require('os').cpus();
for(var i =0 ; i < cpus.length;++i){
	fork('./worker.js');
}
```
然后node master.js 。然后以上的代码会根据当前机器的cpu数，复制出对应的node进程数量。如果在linu环境下，可以通过`ps -ef|grep master.js`来查看。

以下就是著名的Master-Worker 模式，（主从模式）
![](https://proxy.qnoss.seeln.com/images/child_process.png)

fork()出来的这个进程拥有独立的v8实例。它需要至少30毫秒启动时间和至少10MB内存。fork()的代价是昂贵的，而且多进程并不能解决并发问题，只是为了充分利用CPU资源而已。node的大并发问题是通过事件驱动来解决的。

## child_process模块创建子进程
node提供了四种方法来创建子进程:
- spawn() 启动一个子进程来执行命令
- exec() 启动一个子进程来执行命令，与spawn() 不同的是其接口不同，它有一个回调函数来获知子进程的状况。
- execFile() 启动一个子进程来执行可执行文件。
- fork() 与spawn()类似，但是它创建子进程只需要指定要执行的JavaScript文件模块。

```js
var cp = require('child_process);

cp.spawn('node',['worker.js']);

cp.exec('node worker.js',function(err,stdout,stderr){

});

cp.execFile('worker.js',function(err,stdout,stderr){

});

cp.fork('./worker.js');
```
ps:如果用execFile()的话，文件头部要加上`#!/usr/bin/env node`
<!--more-->
## 进程间的通信
-------------------2019年1月1日14:25:45----------------------------------------
>19年的第一天更新，祝我技术越来越好。

主进程和子进程进行通信是十分容易的，类似于websocket的通信模式，发送(send)和监听(.on(''))。

先看下js前端的WebWorker API,为了是UI渲染和JS执行不互相阻塞。
```js
var worker = new Worker('worker.js') //worker.js是一个需要执行的js文件
worker.onmessage = function(event){
 document.getElementById('result').textContent = event.data;
};

```
worker.js如下：
```js
var n = 1;
search: while(true){
	for(var i = 0 ; i < Math.sqrt(n);i++){
		if(n%i==0){
			continue search;
		}
		postMessage(n);
	}
}
```
如上所示，使用postMessage来发送数据，使用onmessage来接受数据。

而node中:
parent.js
```js
	var cp = require('child_process');
	var n = cp.fork(__dirname + '/sub.js');
	
	n.on('message',function(m){
		console.log('parent get message:',m);
	})
	
	n.send({hello:'world'});
```
sub.js
```js
	process.on('message',function(m){
		console.log('child get message',m);
	})
	
	process.send({foo:'bar'});
```
至于进程之间的通信原理，也不写那么多了，总结下来就是:
> 父进程在创建子进程之前，会先创建IPC通道并监听子进程，然后再创建子进程，通过环境变量(NODE_CHILD_FD)告诉子进程这个IPC通道的文件描述符。子进程启动时，通过描述符去连接这个已存在的IPC通道，完成父子进程之间的连接。

## 句柄传递
什么是句柄？
句柄是一种可以用来表示资源的引用，它的内部包含了指向对象的文件描述符。 比如可以用来表示一个socket对象、一个UDP套接字，一个管道等。
