---
title: 深入浅出node.js之网络编程
categories:
  - node.js
tags:
  - node.js
date: 2018-09-08 22:25:34
updated: 2018-09-08 22:25:34
cover: https://proxy.qnoss.seeln.com/images/wp4202391-aconcagua-wallpapers.jpg
---
好久没更博客了，最近没看书了，买了辆公路车沉迷骑行无法自拔 -.-

## node.js网络模块
 node.js提供提供了 `net 、 dgram 、 http 、https`这四个模块，分别用于处理`TCP 、 UDP 、 HTTP 、 HTTPS`适用于服务器端和客户端。

## 构建TCP服务
 TCP全名为传输控制协议，在[OSI](https://baike.baidu.com/item/%E5%BC%80%E6%94%BE%E7%B3%BB%E7%BB%9F%E4%BA%92%E8%BF%9E%E5%8F%82%E8%80%83%E6%A8%A1%E5%9E%8B?fromtitle=OSI%E4%B8%83%E5%B1%82%E6%A8%A1%E5%9E%8B&fromid=9763441)模型中属于传输层协议,许多应用层协议如[HTTP](https://baike.baidu.com/item/http)、[SMTP](https://baike.baidu.com/item/SMTP)、[IMAP](https://baike.baidu.com/item/imap)等都是基于TCP构建的。

TCP是面向连接的协议，其显著特征是在传输前需要[三次握手](https://baike.baidu.com/item/%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B/5111559)形成会话。

只有在会话形成之后，服务器端和客户端之间才能互相发送数据。在创建会话的过程中，服务器端和客户端分别提供一个套接字，这两个套接字共同形成一个连接。服务器端和客户端通过套接字实现两者之间的连接操作。

了解原理之后，通过node.js来创建一个TCP服务器端来接受网络请求。代码如下：
```js
var net = require('net');

var server = net.createServer(function	(socket){
	socket.on('data',function(data){
		socket.write('hello');
	});

	socket.on('end',function(data){
		console.log("连接断开");
	});
	socket.write("welcome to read '深入浅出node.js' : \n");
});

server.listen(3000,function(){
	console.log('server bound');
});

```

对于net.createServer创建的服务器而言，他是一个EventEmitter实例，他的自定义事件有如下几种：
- listening:在调用server.listen()绑定端口或者Domain Socket 后触发。
- connection:每个客户端套接字连接到服务器端时触发。
- close:当服务器关闭时触发。
- error：当服务器发生异常时触发。

##构建UDP服务
UDP又称用户数据包协议，与TCP一样同属于网络传输层。但是UDP不是面向连接的，所以它的资源消耗低，处理快速且灵活，所以常常应用在那种偶尔丢一两个数据包也不会产生重大影响的场景，比如音频，视频等。 DNS就是基于UDP实现的。

创建UDP套接字
```js
var dgram = require('dgram');
var socket = dgram.createSocket('udp4');
```
创建UDP服务端
```js
var dgram = require('dgram');
var server = dgram.createSocket('udp4');

server.on("message",function(msg,rinfo){
	console.log("server got:"+msg+"from"+rinfo.address+":"+rinfo.port);
});

server.on("listening",function(){
	var address = server.address();
	console.log("server listening" + address.address + "：“+address.port);
});

server.bind(41234);
```
该套接字将接受所有网卡上41234端口上的消息。在绑定完成后，将触发listening事件。
<!-more-->
TCP和UDP都是网络传输层协议，如果要构建高效的网络应用，就应该从传输层着手，但是对于一般的应用场景，无需从传输层协议着手，使用node.js提供的HTTP、HTTPS等模块绰绰有余。

简单的几行代码就可以构建一个HTTP服务器:
```js
var http = require('http');
http.createServer(function(req,res){
res.writeHead(200,{'Content-Type':'text/plain'});
res.end('hello world\n');

}).listen(1337,'127.0.0.1');
console.log('server running at http://127.0.0.1:1337/');
```
虽然几行代码就可以完成一个HTTP服务器，但是它的并发量和QPS都是不容小觑的。

ps:骑自行车摔了一跤，胳膊有点疼，就少打了点字。反正说是博客其实就是自己的笔记。自己看看就行了。我这么菜的 O(∩_∩)O