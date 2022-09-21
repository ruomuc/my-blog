---
title: linux部署网站
categories:
  - 计算机网络与操作系统
tags:
  - linux
date: 2018-08-13 23:31:05
updated: 2018-08-13 23:31:05

---
## 安装nodejs
这个是真的难安，装了卸，卸了装。。。orz

1.cd /usr/local/src #定位到这个目录，下载的文件会在这个目录,版本号随着下载包的改变，最简单就是官网的linux安装包下载路径复制过来。不要问我为什么是user/local/src路径，因为别的路径环境变量配置不出来，搞了一下午。
```
wget https://nodejs.org/dist/v6.9.4/node-v6.9.4-linux-x64.tar.gz
```
2.解压文件,版本号同理，就是刚下的安装包。
```
tar xvf node-v6.9.4-linux-x64.tar.gz
```
3.移动,别问我为什么，实验出真理。
```
mv node-v6.9.4-linux-x64 /usr/local/node
```
4.重点：NODE环境配置
```
vi /etc/profile

#在最下面加入
#node config 
export NODE_HOME=/usr/local/node
export PATH=$PATH:$NODE_HOME/bin  
export NODE_PATH=$NODE_HOME/lib/node_modules 
```
如果你不想用vi，那就用xftp也可以。
5.保存
```
:wq

source /etc/profile
```
6.验证安装
```
node -v 

npm -v
```

## 运行项目
1.安装pm2管理工具
以下操作都是在linux环境下哦
```
npm install -g pm2
```

2.cd项目根目录，即app.js或index.js文件同级目录,先运行一下。
```
node app.js/index.js

或

npm start 

或

node ./bin/www
```
具体哪一个自己本地做完项目心里会有数的，看你package.json里的设置。

3.如果这个时候服务器启动成功
```
http://ip:port
```
4.如果访问不了请检车你的端口是否对外开放。
centos6
```
iptables -I INPUT -p tcp --dport 端口号 -j ACCEPT #开启
service iptables save # 保存
service iptables status  #查看
```
centos7
```
firewall-cmd --zone=public --add-port=3000/tcp --permanent #开启
firewall-cmd --reload #重启
firewall-cmd --list-all #查看
```
你可以去找个端口扫描网站扫描一下。
5.上面都成功了
```
pm2 start ./bin/www
```
这是我的项目启动命令，具体看配置，和你用啥脚手架生成的。
pm2详细操作命令请百度。

6.附上个人[主页](http://www.ruomu.cc)，目前是没有啥子交互，一个主页来跳转，毕竟域名太贵了。


## 还有个域名解析端口的问题
域名访问的好像默认是80端口，我的网站用的是3000端口，所以要个域名后面带带上：port，是在是太丑了，于是我只好改成了80，但是80端口只有一个，一个服务器不能就放一个网站吧，看了一下有什么url转发，反向代理什么的，下次在研究吧。。

<br>
<hr>

-------------------------------------2018年11月9日 20:35:44---------------------------------------------------------------

<hr>

更新一下mysql的安装，上面那个域名和端口的问题已经在另一篇博文解决了，见[nginx+node.js实现简单的URL分发](https://ruomuc.gitee.io/blog/2018/09/03/nginx+node-js%E5%AE%9E%E7%8E%B0%E7%AE%80%E5%8D%95%E7%9A%84URL%E5%88%86%E5%8F%91/)。
<!--more-->
## linux安装mysql
1.找到mysql官网的mysql yum 存储库。
`mysql80-community-release-el7-1.noarch.rpm`,就是这样一个东西。
2.连接上服务器，输入
`wget -i -c http://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm`
3.然后可以使用yum安装
`yum -y install mysql80-community-release-el7-1.noarch.rpm`
4.安装mysql 服务器
`yum -y install mysql-community-server`

##mysql数据库配置
1.启动mysql
`systemctl start  mysqld.service`
2.查看mysql启动状态
`systemctl status mysqld.service`
如果看到一个绿色的active(running)就说明mysql启动成功。
3.找到mysql.log文件
`cd /var/log/mysql.log`
打开找到下面这一行：
`A temporary password is generated for root@localhost: xxxxx`,
xxxxx就是密码。
4.登录
`mysql -uroot -p`

如果上述安装失败，你可以直接安，如果你有云源的话:
```
#yum install mysql
#yum install mysql-server
#yum install mysql-devel
```

可能是我这个服务器有毒，
结合下面的资料：https://www.linuxidc.com/Linux/2018-08/153468.htm
https://blog.csdn.net/m0_37949684/article/details/77962377
才安好，折腾几个小时，orz。

然后还有个navicat无法连接远程数据库。
https://blog.csdn.net/langyu1021/article/details/78900689

因为查的资料太多了，有点复杂。学会用百度google安装这种东西慢慢踩坑，而且每次遇到的问题还不一样 。=.=！，中间rm -rf 搞错了删了一个系统文件夹，赶紧恢复快照重装，脑壳疼。

反正最后大功告成，吓得我赶紧保存个快照。

最后又遇到一个服务器会莫名其妙关闭我刚刚打开的端口号：
永久开启端口号：
`firewall-cmd --permanent --zone=public --add-port=2888/tcp`
`firewall-cmd --reload  #重新载入服务`








