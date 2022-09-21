---
title: 异步编程之async、step。。
categories:
  - node.js
tags:
  - node.js
date: 2018-07-30 21:36:36
updated: 2018-07-30 21:36:36

---
异步的解决方案，主要了解一下async、step加上一个比较奇葩的wind应该足够了。实在不行还可以写恶魔金字塔嘛。
## async
安装引用
```js
var async = require('async');
```
1.series()

```js
async.series([
	function(callback){
		fs.readFile('file1.txt','utf-8',callback);
	},
	function(callback){
		fs.readFile('file2.txt','utf-8',callback);
	},
],function(err,results){
	//results => [file1.txt,file2.txt]
});
```
这段代码等价于
```js
	fs.readFile('file1.txt','utf-8',function(err,content){
		if(err){
			return callback(err);
		}
		fs.readFile('file2.txt','utf-8',function(err,data){
			if(err){
				return callback(err);	
			}
			callback(null,[content,data]);
		});
	});
```
每个callback()执行时会将结果保存起来，然后执行下一个调用，直到结束所有调用，最终的回调函数执行时，队列里异步调用的结果会以数组的形式传入。
异常处理规则为，一旦发生异常，就结束所有调用，并将异常传递给最终回调函数的第一个参数。
	
	
<hr>

2.异步的并行执行 async.parallel()
```js
	async.parallel([
		function(callback){
			fs.readFile('file1.txt','utf-8',callback);
		},
		function(callback){
			fs.readFile('file2.txt','utf-8',callback);
		}
	],function(err,results){
		//results => [file1.txt,file2.txt]
	})
```
parallel中，一旦产生异常，就会将异常作为第一个参数传入给最终的回调参数，只有所有异步调用都正常完成时，才会将结果以数组的方式传入。
	
	
<hr>

3.async.waterfall()
series()内的调用时不存在依赖关系的，当前一个的结果是后一个的输入时。要使用waterfall()
```js
	async.waterfall([
		function(callback){
			fs.readFile('file1.txt','utf-8',function(err,content){
				callback(err,content);
			});
		},
		function(arg1,callback){
		    //arg1 => file2.txt
			fs.readFile(arg1,'utf-8',function(err,content){
				callback(err,content);
			});
		},
		function(arg1,callback){
			//arg1 => file3.txt
			fs.readFile(arg1,'utf-8',function(err,content){
				callback(err,content);
			});
		}
	],function(err,result){
		//result => file4.txt
	});
```
	
	
<hr>
	
	
4.async.auto()
自动依赖处理这里就不说了。。。

## Step
它比async更加轻量，只有一个step接口

1.step(task1,task2,task3)
```js
Step(
	funtion readFile1(){
		fs.readFile('file1.txt','utf-8',this);
	},
	function readFile2(err,content){
		fs.readFile('file2.txt','utf-8',this);
	},
	function done(err,content){
		consolt.log(content);
	}
)
```
Step用到了this关键字，事实上，它是内部的一个next()方法，将异步调用的结果传递给下一个任务作为参数，并调用执行。当然传进来的参数可以不用，这样就是不依赖执行。

<hr>

2.Step的并行：this.parallel()
```js
Step(
	funtion readFile1(){
		fs.readFile('file1.txt','utf-8',this.parallel());
		fs.readFile('file2.txt','utf-8',this.parallel());
	},
	function done(err,content1,content2){
		//content1	=> file1.txt
		//content2 => file2.txt
	},
)
```
如果异步方法结果传回的是多个参数，step将只会取前两个参数。好吧分组什么的就不收了，感觉还是promise和async好点。

## wind
比较奇葩，不说了。





















