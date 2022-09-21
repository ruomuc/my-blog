---
title: 单元测试和Mocha
categories:
  - node.js
tags:
  - node.js
  - 单元测试
date: 2018-09-12 20:29:31
updated: 2018-09-12 20:29:31

---
## 单元测试的必要性
比如创建 add.js：
```js
//add.js
function add(a,b)
{
	return a+b;
}
```
上面这个代码有啥好测得？
```js
> add = function(a, b){return a + b}
[Function: add]
> add(4)
NaN
```
NaN?为什么呢。因为不够严谨，只传一个参数就报错了。
所以正确写法应该是这样的：
```js
//add2.js
function add(a, b)
{
    if (typeof a === "number" && typeof b === "number")
    {
        return a + b;
    }
    else
    {
        return undefined;
    }

}
```
人在写代码的时候会有思维漏洞，但是在写测试的时候往往会考虑各种情况。这就是所谓的TDD。

## Mocha测试框架
<!--more-->
Mocha框架+assert断言库
编写test.js
```js
var add = require('./add.js');
var assert = require("assert");

// 当2个参数均为整数时
it("should return 3", function()
{
    var sum = add(1, 2);
    assert.equal(sum, 3);
});

// 当第2个参数为String时
it("should return undefined", function()
{
    var sum = add(1, "2");
    assert.equal(sum, undefined);
});

// 当只有1个参数时
it("should return undefined", function()
{
    var sum = add(1);
    assert.equal(sum, undefined);
});
```
测试代码中使用了Node.js自带的断言库Assert的assert.equal函数，用于判定add函数返回的结果是否正确。assert.equal成功时不会发生什么，而失败时会抛出一个AssertionErro.
使用mocha执行test.js
```
mocha test.js
```
下面为输出，表示测试案例全部通过
```
✓ should return 3
✓ should return undefined
✓ should return undefined

3 passing
```
而当我们使用test1.js测试add.js时，则后面2个测试案例失败:
```
✓ should return 3
  1) should return undefined
  2) should return undefined

  1 passing (14ms)
  2 failing

  1)  should return undefined:
     AssertionError: '12' == undefined
      at Context.<anonymous> (test/test1.js:18:12)

  2)  should return undefined:
     AssertionError: NaN == undefined
      at Context.<anonymous> (test/test1.js:25:12)

```

Node.js自带的断言库Assert提供的函数有限，在实际工作中，Should等第三方断言库则更加强大和实用。
ps:关于更多的断言库，其实也没必要说了，都是搬运下来的。网上有很多。原理基本都一样。

另外说一个后端常用的测试HTTP的模块 SuperTest。
```js
var request = require("supertest");
var server = require("../server.js");
var assert = require("assert");


it("should return hello fundebug", function(done)
{
    request(server)
        .get("/")
        .expect(200)
        .expect(function(res)
        {
            assert.equal(res.text, "Hello Fundebug");
        })
        .end(done);
});
```
>SuperTest封装了发送HTTP请求的接口，并且提供了简单的expect断言来判定接口返回结果。对于POST接口，使用SuperTest的优势将更加明显，因为使用Node.js的http模块发送POST请求是很麻烦的。


原文链接：https://blog.fundebug.com/2017/03/20/nodejs-unit-test/
PS:看到就搬一点，记得更清楚一点。


