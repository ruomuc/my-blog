---
title: 关于redis的基本操作
categories:
  - 数据库
tags:
  - redis
date: 2019-03-26 20:06:13
updated: 2019-4-13 23:37:28

---
前面还有个mysql必知必会和es6没写完。flag先放哪里，闲下来了再搞。
看了一个《redis入门指南》来深入了解下redis，作者好像是ioredis模块的作者。在cnode论坛上看到的巨佬。

###### redis默认支持16个数据库，可以使用SELECT 命令来更换数据库
`redis> SELECT 1`
当然这并不意味这redis可以和mysql一样，每个不同的项目都应该有一个独立的redis实例
使用`FLUSHALL`可以清空一个redis实例下所有的数据库
###### 匹配符合规则的键名
- `?` 匹配一个字符
- `*` 匹配任意个字符
- `[]` 匹配括号间的任意字符
- `\x` 匹配字符x，斜杠是转义符

`redis> keys *`
看起来和正则差不多，但是只支持以上几个

###### 判断一个键是否存在
`redis> EXISTS bar`
存在返回1 不存在返回0

###### 删除键
`redis> DEL bar`
可以删除多个键，返回值是删除键的个数
###### 获取键的类型
`redis> TYPE bar`
返回：`string`,`list`等

## 字符串类型 string
一般字符串允许存储的最大容量是512MB

###### 赋值和取值
`SET key value`
`GET key`
exp:
`redis> SET bar hello`

`redis> GET bar`

###### 增加或减少指定的整数
`redis> SET bar 1`
这是bar是1
`redis> INCR bar`
bar加1是2
`redis> INCRBY bar 3`
bar加3是5
`redis> DECR bar`
bar减1是4
`redis> DECRBY bar 3`
bar减3是1

###### 增加指定浮点数
`redis> INCRBYFLOAT bar 2.7`
bar加2.7是3.7

###### 向尾部追加值
`APPEND key value`

`redis> SET bar hello`
`redis> APPEND bar world!`
`redis> GET bar`
bar的值是 helloworld!

###### 获取键值长度
`STRLEN key`
`redis> SET bar 你好`
`redis> STRLEN bar`
中文在utf-8长度为3，所以结果是6
如果键不存在返回0

###### 同时获取、设置多个键值
`MGET key[key...]`
`MSET key value [key value...]`

MGET / MSET 和 GET/SET 类似。
<!--more-->

## 哈希 hash
###### 取值和赋值
`HSET key field value`
`HGET key field`
`HMSET key field value [field value...]`
`HMGET key field [field...]`
`HGETALL key`

HSET 命令不区分插入和更新操作，当执行的是插入操作时，即字段之前不存在 ，返回1。当执行的是更新操作时，即之前字段已经存在，返回0。当然如果键不存在，还会创建它。

###### 判断字段是否存在
`HEXISTS key field`
当字段或者键不存在时，返回0。字段存在时，返回1

###### 当字段不存在时赋值
`HSETNX key field value`
HSETNX 和 HSET 类似，只不过当字段存在时不做任何操作，字段不存在时才去创建它。

###### 增加数字
和上面的字符串类型类似，这里是增加整数字段
`HINCRBY person score 60`
如果person这个键不存在，那么这个命令会自动建立该键并默认score字段在执行命令前为0。

###### 删除字段
`HDEL key field[fields...]`
删除一个或多个字段。返回值是被删除的字段个数
`redis> HDEL car price`

###### 只获取字段名或字段值
`HKEYS key`
`HVALS key`
只获取字段名或者字段值
`redis> HKEYS car`
`'name'`
`'model'`
`redis> HVALS car`
`'BWM'`
`'C200'`

###### 获取字段数量
`HLEN key`
`2`

## 列表类型 list
列表类型可以存储一个有序的字符串列表，emm。和数组有点像，常用的操作就是向列表两端添加元素，或者获取列表的某一个片段。

###### 向列表两端添加元素
`LPUSH key value [value...]`
向左边添加一个或者多个元素
`RPUSH key value [value...]`
向右边添加一个或者多个元素
返回值是增加后的列表长度。

###### 从列表两端添加元素
`LPOP key`
从左边弹出一个元素
`RPOP key`
从右边弹出一个元素
返回值都是弹出的元素的值。

###### 获取列表中的元素个数
`LLEN key`
当key不存在时，返回0
如果列表中只有一个元素，pop之后会删除这个键。ps：我试了一下是这样的。
时间复杂度为O(1),因为读取的是现成的值

###### 获取列表片段
`LRANGE key start stop`
redis的起始索引是0
`LRANGE number 0 2`
返回前三个元素

同时支持负索引
`LRANGE number -2 -1`
返回倒数第二个元素和最后一个元素

###### 删除列表中指定的值
`LREM key count value`
- 删除前count个值为value的元素
- 当count > 0 时，从左开始删除
- 当count < 0 时，从右边开始删除
- 当count = 0 时，删除所有值为value的元素

返回实际删除元素的个数	

###### 获得或设置指定索引的元素值
`LINDEX key index`
获取指定索引的元素，索引从0开始
`LSET key index value`
将索引为 index 的 值设置为 value

###### 只保留列表指定片段
`LTRIM key start end`
LTRIM 可以删除指定索引范围之外的所有元素

###### 向列表中插入元素
`LINSERT key BEFORE/AFTER value1 value2`
先查找元素value1，然后再根据 第二个参数是 before/after 来决定将value2插入到它的前面还是后面
###### 将元素从一个列表转到另一个列表R
`POPLPUSH source destination`
- RPOPLPUSH是个很有意思的命令，从名字就可以看出它的功能：先执行RPOP命令再
执行LPUSH 命令。
- RPOPLPUSH命令会先从source列表类型键的右边弹出一个元素，然后将
其加入到destination列表类型键的左边，并返回这个元素的值，整个过程是原子的。

## 集合类型 set
在集合中每个元素都是不同的，且没有顺序。
###### 集合的增加和删除
`SADD key member [member...]`
`SREM key member [member...]`
向集合增加或者删除一个或多个元素
返回值是成功添加/删除的元素个数

###### 获得集合中的所有元素
`SMEMBERS key`
返回集合里的所有元素

###### 判断元素是否在集合中
`SISMEMBER key member`
判断元素是否在集合中
元素存在返回1 不存在返回0

###### 集合间运算
`SDIFF key [key...]`
求差集：A-B的意思就是属于A但不属于B的元素。 {1,2,3} - {2,3,4} = {1}
- `SDIFF setA setB` 计算setA与setB的差集。
- 可以传多个集合 `SDIFF setA setB setC`。从左往右依次计算，先计算setA和setB的差集，在计算他们的结果与setC的差集。
`SINTER key [key...]`
求交集：A∩B的意思就是即属于A有属于B的元素。{1,2,3} ∩ {2,3,4} = {2,3}
- `SINTER setA setB` 计算setA和setB的交集
- 多个集合运算规则同差集
`SUNION key [key...]`
求并集： A∪B的意思就是属于A或属于B的元素，但是不会重复。{1,2} ∪ {2,3} = {1,2,3}
- `SUNION setA setB` 计算setA和setB的并集
- 多个集合计算规则同差集和交集

###### 获得集合中元素个数
`SCARD key`
获取集合中的元素个数
`SCARD letter`
返回集合中的元素个数

###### 进行集合运算并将结果存储到另一个键
`SDIFFSTORE destination key [key...]`
`SINTERSTORE destination key [key...]`
`SUNIONSTORE destination key [key...]`
这组命令和上面那组的区别就是，上面会直接返回运算结果，而这组命令会将返回的结果存储到destination 这个键里面。

###### 随机获得集合中的元素
`SRANDMEMBER letter count`
还可以传递count参数来一次随机获得多个元素，根据count的正负不同，具体表现也不
同。
- （1）当count为正数时，SRANDMEMBER会随机从集合里获得count个不重复的元素。
如果count的值大于集合中的元素个数，则SRANDMEMBER会返回集合中的全部元素。
- （2）当count为负数时，SRANDMEMBER会随机从集合里获得|count|个的元素，这些元
素有可能相同。
假设现在
```md
redis＞SRANDMEMBER letters 2
1) "a"
2) "c"
redis＞SRANDMEMBER letters 2
1) "a"
2) "b"
redis＞SRANDMEMBER letters 100
1) "b"
2) "a"
3) "c"
4) "d"
redis＞SRANDMEMBER letters -2
1) "b"
2) "b"
redis＞SRANDMEMBER letters -10
1) "b"
2) "b"
3) "c"
4) "c"
5) "b"
6) "a"
7) "b"
8) "d"
9) "b"
10) "b"
```

###### 从集合中弹出一个元素
`SPOP key`
LPOP命令，作用是从列表左边弹出一个元素，但因为这里的集合是无序的，所以是随机弹出一个元素。

## 有序集合类型 zset



未完待续。。。
















