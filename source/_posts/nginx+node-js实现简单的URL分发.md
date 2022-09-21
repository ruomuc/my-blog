---
title: nginx+node.js实现简单的URL分发
categories:
  - node.js
tags:
  - linux
  - node.js
  - nginx
date: 2018-09-03 21:54:04
updated: 2018-09-03 21:54:04
cover: https://proxy.qnoss.seeln.com/images/ifBUets-black-hole-wallpaper.jpg
---
前段时间做了一个主页部署到了vps上面，当时遇到一个问题没有解决：关于URL分发的问题。刚好我重装了一个vps，之前那个太贵了，装了一个低配一点的，然后我的主站就要重新部署。

之前是node监听的是一个非80端口，但是，阿里云默认解析到80端口，为了好看不再域名后面加`:`和端口号，我就只能把主站的监听端口改成了80，但是80只有一个，后面部署其他项目怎么办呢。。

于是我这次准备一次解决完：
- 第一个解决方法，好像去域名解析哪里设置什么显/隐性URL解析可以解决，因为阿里云要求解析域名和被指向域名都需要备案才行，不是实名认证，是备案哦。。别的服务商我不清楚，于是pass掉了。。
- 第二个方法：使用nginx配置来进行URL的分发。说实话搞完之后发现挺简单的，虽然nginx的具体配置意思不懂，但是先实现目的就行啦。。

## 安装nginx
安装nginx
```
yum install nginx
```
如果你不能通过上述命令安装，那么更换你的云源，或者参考[linux部署网站](https://ruomuc.gitee.io/blog/2018/08/13/linux%E9%83%A8%E7%BD%B2%E7%BD%91%E7%AB%99/)一文中的node.js安装方式(我们不做伸手党！)

启动nginx
```
systemctl start nginx
```
启动之后访问你解析后的域名或者服务器ip。会有一个nginx欢迎页。。那么恭喜你安装成功。

接下来就要开始配置nginx.conf
```
find / -name nginx.conf
```
由于我也不太确定会在哪里，就搜一下喽。。
使用vi 或者 xftp 修改配置文件。
打开nginx.conf文件，找到这样一行
```
include /etc/nginx/conf.d/*.conf;
```
当然这个是我的目录，你在这个下面新建一个`*.conf`文件，`*`代表任意的意思，没有目录就新建一个目录，或者你修改这个路径，反正能加载到一个`*.conf`文件即可。
然后再新建的`*.conf`文件里面配置：
```
server {
    listen 80;
    server_name home.ruomu.cc;
    location / {
        proxy_pass http://66.42.74.169:81;
        proxy_set_header Host $host:80;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Via "nginx";
    }
}

```

这是我的配置，listen是监听的端口号，server_name，是你的转发的域名，下面proxy_pass：是你的主机ip：端口号，这个端口号就是你服务监听的端口号，比如我的主站就监听在81端口。

如果你想访问主域名或者访问80端口时不显示那个nginx欢迎页，我就乱搞了一下。
```
server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        # root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
            proxy_pass http://66.42.74.169:81;
            proxy_set_header Host $host:80;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Via "nginx";
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```
如上所示，我在nginx.conf这个配置文件里面吧root那一行注释掉，因为欢迎页好像就是这个html文件，然后下面的照着上面的配置。就成功了。
现在你访问主域名和80端口都会指向81，包括你新建的配置文件里所有的域名都可以分发到不同的端口，爽上天有木有！

修改好了之后重启一下
```
systemctl restart nginx
```
然后关于二级域名应该很简单，阿里云域名解析教程里面很详细。
目前我解析了一个主域名和一个二级域名
www.ruomu.cc  和  home.ruomu.cc

ps:我自己看自己博客的时候发现了错别字，我现在已经引入了评论系统，如果有人看到的话，可以留言的其实 -.-,虽然感觉这个博客就我一个人再看 =.=




---------------------------2018-9-4 10:04:40------------------------------------------
<!--more-->
更新一下

搜了一下nginx重定向，找到一段有用的代码如下所示：
```
 server {
  listen 80 default_server;
  server_name  www.lansgg.com lansgg.com;
  access_log  logs/lansgg.access.log main;
  error_log  logs/lansgg.error.log;
  root   /opt/nginx/nginx/html/lansgg;
  index index.html;
  if ($http_host = www.lansgg.com){
  rewrite (.*) http://www.Aries.com;
  }
  }
```
改完之后:
```
server {
    listen 80;
    server_name blog.ruomu.cc;
    # access_log  logs/lansgg.access.log main;
    # error_log  logs/lansgg.error.log;
    # root   /opt/nginx/nginx/html/lansgg;
    # index index.html;
    if ($http_host = blog.ruomu.cc){
        # rewrite (.*) https://ruomuc.gitee.io/blog/;
        # rewrite ^/(.*)$ https://ruomuc.gitee.io/blog/ permanent;
        # rewrite ^/(.*)$ https://ruomuc.gitee.io/blog/ break;
        rewrite ^/(.*)$ https://ruomuc.gitee.io/blog/  last;
    }
}
```
注释掉的都是我感觉没用的。。。其实无所谓，无非是打打日志，默认页面之类的。
关键是下面的那个rewrite，配置完之后，访问blog.ruomu.cc就可以跳转到我的博客页面了，码云的pages服务绑定域名是要收费的。。

不过我这样的坏处是，跳转之后显示的是博客的URL而不是你输入的，我试着设置了重定向的flag: permanent 、break 、 last 都不可以。。不知道是码云机制还是怎么回事。
然后我看到一个回答，我觉得应该是这个原因：
>像如
rewrite ^/abc$ http://www.ppp.com:8080/aaa last;
的这种跳转规则，作如下的解释：
如果rewrite指令的第二个参数（replacement）以http或者以https开头，则nginx内部会将该跳转作为临时重定向去处理，表现到http的响应就是会以302响应状态作为响应。
以302,301等的重定向肯定会修改地址栏的url。这个是没办法改变的。
如果不想改变地址栏的url，那可以考虑使用内部跳转：rewrite "/xxx" /abc last;的这种跳转形式。
但是这种重定向只能对站内的url进行重写。

他说URL不变只能对内战进行重写。。当人了人家码云99RMB/Y 的羊毛说薅就薅？ 对吧，反正基本目的已经实现了，再也不用记那个gitee.io什么的URL了。