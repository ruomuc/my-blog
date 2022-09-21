---
title: 异步控制之 Bagpipe
categories:
  - node.js
tags:
  - node.js
date: 2018-07-30 23:34:18
updated: 2018-07-30 23:34:18

---
## Bagpipe
朴大的bagpipe感觉是比async的解决方案好用一些。
基本思路：
- 通过队列来控制并发量。
- 如果当前队列活跃（调用发起但未执行回调）的异步调用量小于限定值，从队列中取出执行。
- 如果活跃调用达到限定值，调用暂时存放在队列中。
- 每个异步调用结束时，从队列中取出新的异步调用执行。
```js
	var Bagpipei = require('bagpipe');
	//设置最大并发数为 10
	var bagpipe = new Bagpipe(10,{
		refuse:true; //拒绝模式
		timeout:3000; //超时控制 超时时间为3000ms
	});
	for(var i = 0 ; i < 100; ++i){
		bagpipe.push(async,function(){
			
		});
	}
````
- 拒绝模式下：如果等待的调用队列也满了之后，新来的调用就直接返回给它一个队列太忙的拒绝异常。
- 超时模式下：异步操作没有在规定时间内完成，先返回给用户一个超时异常。

<hr>

当队列长度大于1时，Bagpipe对象将触发其full事件，该事件传递队列长度值。该值有助于评估业务绩效。

```js
bagpipe.on('full',function(length){
	console.log('Button system cannot deal on time, queue jam, current queue length is:’+ length);
})

```
## async
不多说。。。