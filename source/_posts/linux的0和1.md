---
title: linux的0和1
categories:
  - 计算机网络与操作系统
tags:
  - linux
date: 2019-10-26 16:35:05
updated: 2019-10-26 16:35:05

---

很久没写博客了，因为之前写的东西够消化很久了。而且现在也不想搬运文档之类的东西，写就写点自己的东西吧。。。

虽然前面的博客搬运工居多，但也是一个知识的整理的过程吧。技术越来越好之后，尽量写一些自己体会过的东西。

而且现在又到笔记用的多，博客没什么灵感。

## linux的真与假

虽然linux会一些简单的命令，但是shell脚本确实不怎么会写，所以无聊想了解一些shell的语法，然后发现在linux中，0表示真，1表示假。

真的是这样吗？amazing！

##### 看这样一个语句

```shell
if [ 1 -eq 1 ]; echo $? // 0
then echo "true"
else echo "false"
	fi
// true
if [ 1 -eq 11 ]; echo $? // 1
then echo "true"
else echo "false"
	fi
// false
```

##### 在看这样一个语句

```shell
if test 1 -eq 1; echo $? // 0
then echo "true"
else echo "false"
	fi
	// true
	
if test 1 -eq 11; echo $? // 1
then echo "true"
else echo "false"
	fi
	//false
```

- `[ ]` 和 `test` 的作用从上面的代码来看，都是做逻辑运算。。（ps：更多作用和细节下次再说吧！）
- 因为在shell中，每个表达式执行完退出的时候，都会返回一个退出状态码，上面的`echo $? `的作用是:`前一个命令的退出状态，可用于获取函数返回值`
- 这样看来，当返回值为0时，进入了`真`的逻辑块。

##### 那这样呢？

```shell
if [ 0 ]; echo $? //0
	then echo "true"
else echo "false"
fi
// true

if [ 1 ]; echo $? //0
	then echo "true"
else echo "false"
fi
// true

```

- 现在知道上面的语句为什么都输出了`true`了吗？



### 总结

- 当 `0`和`1`作为退出码的时候确实表示真假，真实因为shell中是没有`while(1)`这种用法的。
- 所以可以理解为 0 为 真  1 为假，但是原理还是要知道的。



Ps: 做web开发的时候，前后端约定 `{code:0,message:'success'}`，这时状态码0也是代表成功的意思！