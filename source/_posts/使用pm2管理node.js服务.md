---
title: 使用pm2管理node.js服务
categories:
  - node.js
tags:
  - node.js
  - pm2
date: 2019-01-10 21:09:12
updated: 2019-01-10 21:09:12
cover: https://proxy.qnoss.seeln.com/images/wp4202395-aconcagua-wallpapers.jpg
---
今天研究了一下使用pm2来管理node服务器，这样不仅可以看到每个服务占用的内存和cpu，还可以很方便的启动和重启，也可以使用`--watch`，自动重启，而且官方说`pm2 reload`来重启服务器是‘无缝’重启的。。

[官方快速入门文档传送门](http://pm2.keymetrics.io/docs/usage/quick-start/)

简单的使用步骤：
1.下载
` npm install pm2 -g`

2.启动服务
` pm2 start < app name>`

3.重启服务
` pm2 restart < app name>`
` pm2 reload < app name>`

4.停止服务
`pm2 stop < app name>`

5.查看所有服务状态
`pm2 list`

6.删除服务，和stop不同的是，删除后不会存在list列表中。
`pm2 delete < id>`

7.删除所有服务
`pm2 delete all`

8.如果遇到稀奇古怪的问题可以使用
`pm2 kill`
慎用，会杀死所有服务，守护进程也会杀掉好像，和`delete`的区别不清楚。

9.高大上的监控界面，目前只知道用来看日志。
`pm2 monit`

其他用法还不清楚。
然后就是配置文件：
```json
{
    "apps": {
        "name": "wuwu",                             // 项目名          
        "script": "./bin/www",                      // 执行文件
        "cwd": "./",                                // 根目录
        "args": "",                                 // 传递给脚本的参数
        "interpreter": "",                          // 指定的脚本解释器
        "interpreter_args": "",                     // 传递给解释器的参数
        "watch": true,                              // 是否监听文件变动然后重启
        "ignore_watch": [                           // 不用监听的文件
            "node_modules",
            "logs"
        ],
        "exec_mode": "cluster_mode",                // 应用启动模式，支持fork和cluster模式
        "instances": 4,                             // 应用启动实例个数，仅在cluster模式有效 默认为fork；或者 max
        "max_memory_restart": 8,                    // 最大内存限制数，超出自动重启
        "error_file": "./logs/app-err.log",         // 错误日志文件
        "out_file": "./logs/app-out.log",           // 正常日志文件
        "merge_logs": true,                         // 设置追加日志而不是新建日志
        "log_date_format": "YYYY-MM-DD HH:mm:ss",   // 指定日志文件的时间格式
        "min_uptime": "60s",                        // 应用运行少于时间被认为是异常启动
        "max_restarts": 30,                         // 最大异常重启次数，即小于min_uptime运行时间重启次数；
        "autorestart": true,                        // 默认为true, 发生异常的情况下自动重启
        "cron_restart": "",                         // crontab时间格式重启应用，目前只支持cluster模式;
        "restart_delay": "60s"                      // 异常重启情况下，延时重启时间
        "env": {
           "NODE_ENV": "production",                // 环境参数，当前指定为生产环境 process.env.NODE_ENV
           "REMOTE_ADDR": "爱上大声地"               // process.env.REMOTE_ADDR
        },
        "env_dev": {
            "NODE_ENV": "development",              // 环境参数，当前指定为开发环境 pm2 start app.js --env_dev
            "REMOTE_ADDR": ""
        },
        "env_test": {                               // 环境参数，当前指定为测试环境 pm2 start app.js --env_test
            "NODE_ENV": "test",
            "REMOTE_ADDR": ""
        }
    }
}
```

ps:我觉得最强大的是，pm2内置了 `cluster` 模块，可以指定启动进程个数，然后就是一堆服务监听一个端口，实现了集群.emm..
但是好像也有不好的地方。