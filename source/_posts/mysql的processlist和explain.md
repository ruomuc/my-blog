---
title: mysql的processlist和explain
categories:
  - 数据库
tags:
  - mysql
date: 2019-11-17 13:24:56
updated: 2019-11-17 13:24:56

---

## show processlist

一般用到 `show processlist` 或 `show full processlist` 都是为了查看当前 mysql 是否有压力，都在跑什么语句，当前语句耗时多久了，有没有什么慢 SQL 正在执行之类的。

###### 每一列代表的意思

- `id`:链接mysql 服务器线程的唯一标识，可以通过kill来终止此线程的链接。
- `user`:当前线程链接数据库的用户
- `host`:显示这个语句是从哪个ip 的哪个端口上发出的。可用来追踪出问题语句的用户
- `db`:线程链接的数据库，如果没有则为null
- `command`:显示当前连接的执行的命令，一般就是休眠或空闲（sleep），查询（query），连接（connect）
- `time`：线程处在当前状态的时间，单位是秒
- `state`:显示使用当前连接的sql语句的状态，很重要的列，后续会有所有的状态的描述，请注意，state只是语句执行中的某一个状态，一个 sql语句，已查询为例，可能需要经过copying to tmp table，Sorting result，Sending data等状态才可以完成
- `info`:线程执行的sql语句，如果没有语句执行则为null。这个语句可以使客户端发来的执行语句也可以是内部执行的语句

```mysql
-- 查询非 Sleep 状态的链接，按消耗时间倒序展示，自己加条件过滤
select id, db, user, host, command, time, state, info
from information_schema.processlist
where command != 'Sleep'
order by time desc 
```
###### kill
```mysql
-- 查询执行时间超过2分钟的线程，然后拼接成 kill 语句
select concat('kill ', id, ';')
from information_schema.processlist
where command != 'Sleep'
and time > 2*60
order by time desc 
```

###### 主要的状态有

- `checking table`:　正在检查数据表（这是自动的）。
- `Closing tables`:　正在将表中修改的数据刷新到磁盘中，同时正在关闭已经用完的表。这是一个很快的操作，如果不是这样的话，就应该确认磁盘空间是否已经满了或者磁盘是否正处于重负中。
- `copyng to tmp table on disk`:　由于临时结果集大于tmp_table_s`ize，正在将临时表从内存存储转为磁盘存储以此节省内存。
- `creating tmp table`:　正在创建临时表以存放部分查询结果。
- `locked`:　被其他查询锁住了。
- `sending data`:　正在处理SELECT查询的记录，同时正在把结果发送给客户端。
- `sleeping `:　正在等待客户端发送新请求.。



## explain 优化sql

###### 语法
- `explain select`
- `EXPLAIN EXTENDED SELECT ....`  将执行计划“反编译”成SELECT语句，运行`SHOW WARNINGS` 可得到被MySQL优化器优化后的查询语句 
- `EXPLAIN PARTITIONS SELECT ....` 用于分区表的EXPLAIN

<!--more-->
###### 通过explain可以知道mysql是如何处理语句，分析出查询或是表结构的性能瓶颈。通过expalin可以得到：

-  表的读取顺序
- 表的读取操作的操作类型
- 哪些索引可以使用
- 哪些索引被实际使用
- 表之间的引用
- 每张表有多少行被优化器查询`

###### explain显示字段

- `id`:语句的执行顺序标识
- `select_type`:使用的查询类型，主要有以下几种查询类型
  - `simple`:简单类型,语句中没有子查询或union
  - `Primary`:查询中若包含任何复杂的子部分，最外层查询则被标记为：PRIMARY，不是主键
  - `union`:UNION之后的第二个SELECT，则被标记为UNION，第一个select 为primary 
  - `dependent subquery`:子查询中内层中第一个select语句
  - `dependent union`:子查询中union后的select，依赖于外部的结果集。
  - `SUBQUERY`
  - `devived`:派生表的查询语句
  - `uncacheable subquery `:结果集无法缓存的子查询
  - `union result `:union中合并的结果
- `table` :显示这一步所访问的数据库中表的名称
- `type`:这列很重要,显示了连接使用了哪种类别,有无使用索引。从最好到最差的连接类型为：
  - `system`:system为const一个特例，即表中只有一条记录。
  - `const`:const是在where条件以常量作为查询条件，表中最多有一条记录匹配。由于是常量，所以实际上只需要读一次。
  - `eq_ref`:最多只会有一条匹配结果，一般是通过主键或是唯一索引来访问。一般会出现在连接查询的语句中。
  - `ref`:非唯一性索引扫描，返回匹配某个单独值的所有行。常见于使用**非唯一索引即唯一索引的非唯一前缀**进行的查找
  - `range`:索引范围扫描，常见于between、<、>、in等的查询
  - `index`:全索引树被扫描Full Index Scan
  - `all`:全表扫描，效果是最不理想的。
- `possible_keys`:查询可以利用的索引，如果没有任何索引可以使用，就会显示成null，这项对内容的优化时索引的调整非常重要。
- `key`:从**possible_keys**中所选择使用的索引
- `key_len`:key_len列显示mysql决定使用的键长度，如果键是null，则长度为null。使用的索引长度，一般越短越好。
- `ref`:表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值
- `rows`:通过系统收集到的统计信息，估计出来的结果集记录条数
- `extra`:查询中每一步实现的额外细节信息。。



其实就是一些能google到的东西，之前用的不多所以没在意，现在整理一下。

