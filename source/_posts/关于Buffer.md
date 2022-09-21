---
title: 关于Buffer
categories:
  - node.js
tags:
  - buffer
  - node.js
date: 2018-08-15 00:35:48
updated: 2018-08-15 00:35:48

---
## Buffer结构
#### 模块结构
1.一个很像Array的对象。

2.Buffer是典型的js和c\++结合的模块，性能部分由c++实现，非性能相关由js实现。

3.Buffer所占用的内存不是V8分配的，属于堆外内存。并且在nodejs启动时加载，并放在global上，无需require()就可以直接使用

#### Buffer对象
Buffer对象类似于数组，元素为16进制的两位数，即十进制的0~255。
```
var str = "深入浅出node.js";
var buf = new Buffer(str,'utf-8');
console.log(buf);
//=> <Buffer e6 b7 b1 e5 85 a5 e6 b5 85 e5 87 ba 6e 6f 64 65 2e 6a 73>
```
中文字utf-8下站三个元素，字母和半角标点符号站一个元素。
Buffer可以通过length来获取长度
```
var buf = new Buffer(100);
console.log(buf.length);
// => 100
```
以上代码初始化了一个长度为100字节的Buffer对象。
```
console.log(buf[10]);
// => 0
```
以上代码可以打印出下标为10的元素值，但是buf对象是初始化并没有赋值，所以结果为0。这里和朴大书里说的有些出入，朴大会输出一个0~255的随机值，但是我试验过一直都是0。所以我选择相信自己。。。

赋值
```
buf[10] = 100; // => 100
buf[10] = -100; // +256直到得到一个0~255的数  => 156
buf[10] = 300; // -256直到得到一个0~255的数 => 44
buf[10] = 3.14 // 去掉小数，再遵循以上规则 => 3
console.log(buf[10])
```

#### Buffer内存分配
Buffer对象的内存分配不是在V8的堆内存中，是在node的C++层面实现申请的。为此采用的是，在C++层面申请内存，在JavaScript中分配内存的策略。

Node采用[slab](https://www.ibm.com/developerworks/cn/linux/l-linux-slab-allocator/index.html)分配机制，简而言之，slab就是一块申请好的固定大小的内存区域，并具有三种状态：
- full：完全分配状态。
- partial：部分分配状态。
- empty：没有被分配状态。

当我们`new Buffer(size)`时：
Node以**8KB**为界限来区别Buffer是大对象还是小对象`Buffer.poolSize = 8 * 1024`。
这个**8KB**的值也就是每个slab的大小值，在JavaScript层面中，以它作为单位单元进行内存分配。

ps:1kb = 1024 byte = 8 bit = 1111 1111 = 255

申请一个新的slab单元：
```js
var pool;
function allocPool(){
	pool = new SlowBuffer(Buffer.poolSize);
	pool.use = 0;
}
//处于empty状态
```
1.如果指定Buffer大小少于8KB：
```js
new Buffer(1024);
//检查pool对象，是否被创建。如果没有则创建一个新的slab单元指向它：
if(!pool || pool.length - pool.used < this.length){
	allocPool();
}
  
```

同时：
```js
this.parent = pool; //当前Buffer对象的parent属性指向pool
this.offset = pool.used; //记录从这个slab的哪个地方开始使用
pool.used += this.length; //记录自身被使用了多少字节
if（pool.used & 7){
pool.used = (pool.used + 8 ) & ~7;
}
```
这种分配方式可能导致1KB的Buffer占用了8KB的内存（1个slab）;

2.分配大小超过8KB的Buffer对象。
将会直接分配一个SlowBuffer对象作为slab单元，这个slab单元会被这个Buffer独占。这时slab的大小应该就是Buffer的大小吧。
```
this.parent = new SlowBuffer(this.length);
this.offset = 0;
```
这个SlowBuffer类是C\++中定义的，虽然引用buffer模块可以访问，但不推荐直接操作。
上面提到的Buffer对象都是JavaScript层面的，能被V8标记回收，但是其内部的parent属性指向的SlowBuffer对象却来自于Node自身C\++中的定义，是C++层面上的Buffer对象，所以内存不在V8的堆中。
<!--more-->
#### Buffer转换
BUffer目前支持转换的字符串编码类型:
- ASCII
- UTF-8
- UTF-16LE/UCS-2
- Base64
- Binary
- Hex

1.字符串转Buffer
encoding不传值时，默认按UTF-8编码进行转码和存储。
```
new Buffer(str,[encoding]);
```
buf.write可以指定写入起始位置，长度和编码，所以一个Buffer中可以有多种编码类型，这样反转回字符串的时候需要谨慎处理。
```
buf.write(strig,[offset],[length],encoding);
```
还可以toString();
```
buf.toString([encoding,start,end]);
```
2.有些Buffer不支持的编码类型
判断编码是否支持转换
```
Buffer.isEncoding(encoding);
```
对不支持的编码类型可以借助模块完成转换。
iconv和iconv-lite
```js
var iconv = require('iconv-lite');
//Buffer转字符串
var str = iconv.decode(buf,'win1251');
//字符串转Buffer
var buf = iconv.encode(str,'win1251');
```
具体可以去查阅 https://www.npmjs.com/package/iconv-lite
			https://www.npmjs.com/package/iconv
			
#### Buffer的拼接
buffer通常是以一段一段的方式传输在内存中。
```js
var fs = require('fs');

var rs = fs.createReadStream('test.md');
var data = '';
rs.on('data',function(chunk){
	data+=chunk;
});
rs.on('end',function(){
	console.log(data);
});
```
上面的代码在国外没有宽字节编码时没有问题，但是遇到宽字节会导致乱码。
问题来自于这句代码：

```js
data += chunk;
等价于
data = data.toString() += chunk.toString();
```

复现问题：
```js
var rs = fs.createReadStream('test.md',{highWaterMark:11});
//test.md 是李白的静夜思。。。。
var data = '';
rs.on('data',function(chunk){
	data+=chunk;
});
rs.on('end',function(){
	console.log(data);
});
// => 床前明��光，疑���地上霜。举头��明月，���头思故乡。
```
为什么乱码呢。。。。
让我们看下这首诗的原本的Buffer
```
<Buffer e5 ba 8a e5 89 8d e6 98 8e e6 9c 88 e5 85 89 ef bc 8c e7 96 91 e6 98 af e5 9c b0 e4 b8 8a e9 9c 9c e3 80 82 e4 b8 be e5 a4 b4 e6 9c 9b e6 98 8e e6 9c ... >
```
我们限定了Buffer对象的长度为11，所以`e5 ba 8a | e5 89 8d |e6 98 8e | e6 9c`,床前明三个字之后就匹配不到了，剩下两个字节就以乱码的形式显示了。

解决方案一:
setEncoding();
```js
var rs = fs.createReadStream('test.md',{highWaterMark:11});
rs.setEncoding('utf-8');
//test.md 是李白的静夜思。。。。
var data = '';
rs.on('data',function(chunk){
	data+=chunk;
});
rs.on('end',function(){
	console.log(data);
});
// => 床前明月光，疑是地上霜。举头望明月，低头思故乡。
```
这种方式的data收到的不再是Buffer对象，所以问题解决了，但是字节字符的问题依然存在，并且setEncoding，但是只支持 UTF-16LE/UCS-2 、 utf-8和base64三种编码。

解决方法二：
正确的Buffer拼接：
正确的拼接方式时，使用一个数组来存储接收到的所有buffer对象，并记录所有片段的总长度，然后调用Buffer.concat()来生成一个合并的Buffer对象。
```js
var chunks = [];
var size = 0;
res.on('data',function(chunk){
	chunks.push(chunk);
	size += chunk.length;
});

res.on('end',function(){
	var buf = Buffer.concat(chunks,size);
	var str = iconv.decode(buf,'utf-8');
	console.log(str);
})
```
本问题复现代码github地址：https://github.com/ruomuc/practice/tree/master/other/buffer

Buffer.concat实现原理：
```
Buffer.concat() = function(list, length) {
	if (!Array.isArray(list)) {
		throw new Error('Usage: Buffer.concat(list,[length])');
	}
	if (list.length === 0) {
		return new Buffer(0);
	} else if (list.length === 1) {
		return list[0];
	}
	if (typeof length !== 'number') {
		length = 0;
		for (var i = 0; i < list.length; i++) {
			var buf = list[i];
			length += buf.length;
		}
	}

	var buffer = new Buffer(length);
	var pos = 0;
	for (var i = 0; i < list.length; i++) {
		var buf = list[i];
		buf.copy(buffer, pos);
		pos += buf.length;
	}
	return buffer;
}
```











