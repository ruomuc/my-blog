---
title: promise中的reject和catch处理上的区别
categories:
  - node.js
tags:
  - promise
date: 2018-07-17 16:19:35
updated: 2018-07-17 16:19:35

---

## reject 和 catch 的处理上
```js
auto.getData().then(function (results) {
     res.send(results);
}, next);
auto.getData().then(function (results) {
     res.send(results);
}).catch(next);
```
###### 第一种写法，next函数只会处理getData中的reject时的异常情况。
![](https://dn-cnode.qbox.me/FobQa2vcQxJQo_1kbvF3K71YXyDK)
<br/>
###### 第二种写法，catch会捕捉到在它之前的promise链上的代码抛出的异常，不仅仅是getData，还有then()里面的异常。
![](https://dn-cnode.qbox.me/Fm56oEjn1Sjfn7lV1wx-dOFvwdTX)

</br>
---

好吧看到了就整理搬运过来了，免得到时候找不到了 orz