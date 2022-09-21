---
title: 初识redo log和binlog
categories:
  - 数据库
tags:
  - mysql
date: 2021-08-13 23:21:21
updated: 2021-08-13 23:21:21
cover: https://proxy.qnoss.seeln.com/images/mysql%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg
---

整理下 mysql 的 redo log 和 binlog 相关知识。

这个是照着画的一个 mysql 逻辑架构图，方便后面对照着理解。

<!--more-->

**需要注意的是：mysql8.0 干掉了查询缓存这个模块。**（因为查询缓存失效的很频繁，本来就很鸡肋，默认都是关闭的）

<img src="https://proxy.qnoss.seeln.com/images/mysql%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg" style="zoom: 50%;" />

##### 什么是物理日志 和 逻辑日志？

我的理解是这样的，**物理日志记录的是值得变更结果，逻辑日志记录的是值得变更过程。**

比如把 a 值初始为1，我们给 a 加二再减一

- 物理日志为：`a:1->a:3` 和 `a:3->a:2`
- 逻辑日志为:  `a=a+2 `和 `a=a-1`。ps: 在mysql中可以理解为一条条sql语句

##### 什么是 redo log？

redo log 是**物理日志**，并且是 **InnoDB 特有的**。

 mysql 的 WAL （Write-Ahead Logging）技术就是通过它实现的，WAL 的意思就是先写日志，再写磁盘。

- 这里需要注意一下，所谓的 “先写日志”，其实也是写磁盘，只不过是顺序写，非常快。
- ”再写磁盘“ 的意思是，向 mysql 真正存储业务数据的地方写数据（也在磁盘上，但是在写之前还要查找、索引B+树节点变动等），很慢。

> 举个例子，做生意的小商家记账的时候，不可能每一笔账的马上记到总账本里面，因为总账本很厚，找到对应的地方并记录一笔账目是很费时间的，一般都会在一张纸上或者小黑板上先记着，然后等人少的时候，或者晚上关门的时候，再计入总账本；我们的 redo log 就对应这个例子里的小黑板。

redo log 保证了数据库即使发生异常重启（这种能力称为 crash-safe），之前的记录都不会丢失。

> 还是用上面那个小商家举例，假如商家突然有事关门了，重新营业的时候，只需要根据总账本和小黑板的账目，恢复到之前的状态，不会丢失任何一笔账。

##### redo log的记录模式

InnoDB 的 redo log 大小是固定的，我们可以使用： `show variables like '%innodb_log_file%';`  命令查看。

<img src="https://proxy.qnoss.seeln.com/images/redolog%E9%BB%98%E8%AE%A4%E9%85%8D%E7%BD%AE.PNG" style="zoom:150%;" />

默认是两个文件，一个文件 48 M。

<img src="https://proxy.qnoss.seeln.com/images/redolog%E8%AF%BB%E5%86%99%E6%96%B9%E5%BC%8F.jpg" style="zoom: 80%;" />

- write pos 是当前记录的位置，一边写一边后移。
- checkpoint 是当前要擦除的位置，也是往后推移并且循环的。
- 中间绿色的就是空白位置。

##### 如何设置redo log的大小？

当一个日志文件写满后，innodb会自动切换到另一个日志文件，而且会触发数据库的checkpoint，这回导致 innodb 缓存脏页的小批量刷新，会明显降低innodb的性能。

- 如果innodb_log_file_size过小，会导致innodb频繁地checkpoint
- 如果innodb_log_file_size过大，会导致在崩溃恢复InnoDB时，会导致恢复时间变长。 ps: 其实我倒觉得这不是什么大问题

所以说，要根据自己的业务量设置合适的 innodb_log_file_size。网上有很多设置方法，这里就不展开了。

##### 什么是 binlog?

前面说了，redo log 是InnoDB 特有的，所以是引擎层的日志模块，而 binglog 是 Server 层的。

binlog 是一个逻辑日志。

为什么会有两份日志呢？

- 最开始 MySQL 里并没有 InnoDB 引擎。MySQL 自带的引擎是 MyISAM

- MyISAM 没有 crash-safe 的能力，binlog 日志只能用于归档

##### binlog 和 redo log 的不同点

- redo log 是 InnoDB 引擎特有的，binlog 是 mysql 的 Server 层实现的

- redo log 是物理日志，binlog 是逻辑日志
- redo log 是循环写，binlog 是追加写

##### 执行流程和两阶段提交

假设有表 tableA，有一条记录 id =2 ，c = 1，我们执行下面这条语句：

```sql
update tableA set c = c+1 where id = 2;
```

<img src="https://proxy.qnoss.seeln.com/images/%E4%B8%A4%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A4.png" style="zoom:80%;" />



我们将 redo log 的写入拆成了两个步骤：

- 第一步写 redo log 并且是 prepare 状态
- 写完 binlog 后，将redo log 由 prepare 变为 commit 状态

这样做的目的是为了保持数据一致性，使 redo log 和 binlog 的数据一致。

如果是一段提交，我们分析一下两种场景：

1. 先写 redo log，再写 binlog，并且写完redo log 后数据库 crash 了。我们恢复数据时：
   - redo log 中 c 的值是 2
   - binlog 没有日志
   - 虽然我们可以通过 redo log 恢复为 c = 2这一正确数据，但是 binlog 中没有这条记录，使用binlog 恢复数据就和 crash 前不一致。
2. 先写 binlog，再写 redo log，并且写完 binlog 后数据库 crash：
   - redo log 没有日志
   - binlog 日志 “将 c 由 1 改为 2”
   - binlog 中存在记录“ 把 c 由 1 改为 2”，但是 redo log 中 c 的值仍然是 1

两阶段提交如何恢复呢？

1. 写redo log，处于 prepare 阶段，还没写 binlog，数据库 crash：
   - redo log 中 c =2，但状态是 prepare，并未提交
   - binlog 没有日志
   - 数据库恢复时，使用 binlog 恢复 c 值为 1，使用 redo log 会忽略掉未提交的日志，c 值也是 1
2. 写 redo log，处于 prepare 阶段，写binlog，还没来得及提交 redo log，数据库 crash：
   - redo log 中 c=2, 但状态是 prepare，并未提交
   - binlog 日志 “将 c 由 1 改为 2”
   - 数据库恢复时，使用 binlog 恢复 c 值 为2，使用redo log，虽然状态为 prepare，但 prepare和binlog日志都是完整的，会自动commit，满足数据一致性。



ps: 这里其实每个细节展开都有好多知识，这里只是先有个概念。



参考链接：

- 《MySQL实战45讲》





