---
title: 流式查询mysql
categories:
  - 数据库
tags:
  - mysql
  - stream
date: 2020-07-13 21:30:54
updated: 2020-07-13 21:30:54

---
简单记录一下，使用node的流读取mysql数据，解决分页扫表慢的问题。

#### 场景

有一个计算任务，需要逐条扫描一张大表里的符合条件的数据，并且做一些简单的计算插入一张小表中去，类似`select * from tableA where condition > 1`。假设符合条件的数据有1000w条，使用分页，每页1000条，逐页查询并计算完插入小表。

#### 瓶颈
mysql分页的特性。。就是越往后越慢，晚上计算任务又多，很容易跑不完或者挂掉。
哪怕优化了分页，效果还是不如意。

#### 解决方案

我们用的 knex.js，不能算是orm，只能算是一个sql builder吧。

使用knex的stream创建一个流，然后使用node的流事件`.on('data')`监听这个流，每查询一条数据都会触发流事件。

这里只是提供了一个思路，上代码：
<!--more-->
```js
'use strict';
const logger = require('./filterLogger')
const _ = require('lodash')
const knex = require("./db")

/**
 * 流式处理数据
 * @param {String} sql 数据源sql
 * @param {Function} dealFunction 处理数据的方法
 * @param {Number} pageSize 一次处理数据大小
 */
function streamReadToWriteDB(sql, dealFunction, pageSize) {
  let n = 0;
  let tempData = [];
  return new Promise(function (resolve, reject) {
    knex.raw(sql)
      .stream(function (stream) {
        stream.on('data', data => {
          tempData.push(data)
          n++;
          if (tempData.length >= pageSize) {
            stream.pause()
            dealFunction(tempData)
              .then(() => {
                tempData = []
                stream.resume()
              })
          }
        })
        stream.on('end', () => {
          if (!_.isEmpty(tempData)) {
            logger.trace(`任务剩余${tempData.length}条未处理，即将处理！`)
            dealFunction(tempData)
              .then(results => {
                tempData = []
                logger.trace(`任务执行完毕!一共${n}片数据,每片${pageSize}条`)
                resolve(results)
              })
          } else {
            logger.trace(`任务执行完毕!一共${n}片数据,每片${pageSize}条`)

            resolve()
          }
        })
      })
      .then(() => {
        return Promise.resolve(true)
      })
      .catch(err => {
        logger.warn('流处理出错', err)
        reject()
      })
  })
}

module.exports = {
  streamReadToWriteDB
}
```

代码中的pagesize是嫌弃一次处理一条数据太浪费了，多缓存几个一起处理。

这中写法效果非常好，千万数据量速度快了几十倍。。

当然和es的scroll写法一样，流不支持跳页。
