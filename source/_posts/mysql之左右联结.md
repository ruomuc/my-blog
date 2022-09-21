---
title: mysql之左右联结
categories:
  - 数据库
tags:
  - mysql
date: 2019-04-14 20:17:48
updated: 2019-04-14 20:17:48
cover: https://proxy.qnoss.seeln.com/images/foCyHXt-black-hole-wallpaper.jpg
---

之前对联结表的看理解好像不是很明确，于是又看了一些资料。

再次之前假装先建立两张表:

- tableA:三个字段(id,title,price)，五条记录
- tableB:三个字断(id, title,price),三条记录
- 两张表之间有两条记录的price相同

## 内联结

`INNER JOIN`

`select * from tableA INNER JOIN tableB ON a.title = b.title where tableA.price > 100`

上面语句的意思是，获取tableA和tableB中 title 字段相等的记录。

执行过程大概如下：

- 因为上面假设的时候，他们是有两条记录满足price字段相等，所以这里会返回两条结果。
- 然后再在这两条结果里面筛选price>100的结果

## 左联结

`LEFT JOIN`

`select * from tableA LEFT JOIN tableB ON a.title = b.title where tableA.price > 100`

执行过程如下：

- 因为tableA是有五条记录的，所以联结的时候先是会返回五条记录，以左边的表为准。如果右边的表的记录少于左边，那么填充NULL
- 然后在根据where子句的条件筛选出最终的结果



## 右联结

`RIGHT JOIN`

`select * from tableA RIGHT JOIN tableB ON a.title = b.title where tableA.price > 100`

执行过程如下：

- 因为tableB是有3条记录的，所以联结的时候先是会返回3条记录，以右边的表为准。如果右边的表的记录少于左边，那么填充NULL
- 然后在根据where子句的条件筛选出最终的结果



其实左右联结可以通过两张表调换顺序实现。

总结一下：

- 内联结两张表都受影响，返回满足on条件的记录
- 左联结只有右表受到影响，左表在没有where子句筛选时，返回所有记录。
- 右联结和左联结相反，只有左表受到影响，返回右表的所有记录。