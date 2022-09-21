---
title: linux常用命令记录
categories:
  - 计算机网络与操作系统
tags:
  - linux
date: 2018-08-15 23:24:00
updated: 2018-08-15 23:24:00
cover: https://proxy.qnoss.seeln.com/images/Z7pDgOx-black-hole-wallpaper.jpg
---
##### 平常常用的linux命令，持续更新。。主要是centos6/7，有差异。

查看系统版本
```
cat /etc/redhat-release
cat /proc/version
cat /etc/issue
```

查看64位还是32位
```
getconf LONG_BIT
file /bin/ls
```

查看相关进程状态(node)
```
ps -ef | grep node
```
查询端口
```
netstat -tulpn
```
根据端口号得到其占用的进程的详细信息
```
netstat -tlnp|grep 80
```
一次性的清除占用80端口的程序
```
lsof -i :80|grep -v "PID"|awk '{print "kill -9",$2}'|sh
```
手工终止进程的运行
```
kill -9 PID
```
开启端口
```
firewall-cmd --zone=public --add-port=80/tcp --permanent
```
防火墙设置 centos 7
```
systemctl start firewalld 开启防火墙
systemctl stop firewalld 关闭防火墙
firewall-cmd --reload #重启firewall
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```




------------------------------------分割线 2018/8/23 22:39 --------------------------------------------------------------



## svn安装
###### 安装svn
```
yum -y install subversion
```
###### 下载
```
svn checkout(co) address
```
###### 解决SVN:E210007无法协商认证机制
```
sudo yum install cyrus-sasl cyrus-sasl-plain cyrus-sasl-ldap
```
###### 继续下载
```
svn checkout(co) address
```
## svn ignore
###### 要忽略具有结尾.o的所有文件，请使用：
```
svn propset svn:ignore "*.o" .
```
###### 如果你想忽略文件夹tmp
```
svn propset svn:ignore tmp .
```
###### 如果你想忽略tmp，obj，bin dirs和所有带* .o * .lib，*。la扩展名的文件。保存此文件
```
tmp
obj
bin
*.o
*.lib
*.la
```
###### 并将其命名为svnignore.txt，以下命令将完成这项工作！
<!--more-->
```
svn propset svn:ignore -F svnignore.txt .
```
###### 通过命令行忽略多个文件/目录
```
svn propedit svn:ignore .
```
这将显示要忽略的文件或目录列表。
###### 查找不受版本控制的文件
```
svn status | grep ^\? | awk '{print $2}'
```

## centos7 firewall
###### 开放端口
```
firewall-cmd --zone=public --add-port=80/tcp --permanent
```
###### 重启防火墙：
```
systemctl restart firewalld.service 
```
###### 关闭防火墙：
```
systemctl stop firewalld.service 
```
###### 查看监听(Listen)的端口
```
netstat -lntp
```
###### 检查端口被哪个进程占用
```
netstat -lnp|grep 8080
```
###### 查看node相关的进程
```
ps -ef | grep node
```
## 服务systemctl
###### 查看服务状态
```
systemctl status *
```
###### 启动服务
```
systemctl start *
```
###### 重启服务
```
systemctl restart *
```
###### 停止服务
```
systemctl stop *
```
###### 重启也会生效
```
systemctl enable *
```



