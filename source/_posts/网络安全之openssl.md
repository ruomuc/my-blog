---
title: 网络安全之openssl
categories:
  - node.js
tags:
  - 安全
date: 2018-11-06 19:04:11
updated: 2018-11-06 19:04:11

---
客户端和服务端的数据传输如果不加密就会被别人截取而泄露数据。对于开发者而言，加密和解密最好是放在传输层，而应用层的数据处理仍然是透明的。
node在网络安全提供了，tls、https、crypto三个模块。tls类似net、https是http安全版。crypto提供了SHA1、MD5等加密算法。

## TLS/SSL 结构
node底层采用的是openssl实现TSL/SSL的，所以可以使用openssl生成公钥和私钥。

1.下载安装openssl
我是在http://slproweb.com/products/Win32OpenSSL.html 下载windows安装包。
然后配置环境变量。
然后随便在一个地方打开cmd命令提示符，输入openssl就可以进入生成秘钥操作。
2.生成私钥。
```
//生成服务器私钥
genrsa -out server.key 1024
//生成客户端私钥
genrsa -out client.key 1024
```
上面生成了两个1024位长的RSA私钥文件。当前操作目录下会有server.key 和 client.key
3.生成公钥
```
rsa -in server.key -pubout -out server.pem

rsa -in client.key -pubout -out client.pem
```
公私钥还是会被窃听，窃听者通过扮演假的服务器和客户端，来伪造响应和请求。
所以就有了数字证书。
## 数字证书
数字证书中包含了服务器的名称和主机名、服务器的公钥、签名颁发机构的名称、来自签名颁发机构的签名。在连接建立之前，会通过证书中的签名确认收到的公钥是来自目标服务器的，从而产生信任关系。

为了保证数据安全，我们引入一个第三方CA。CA的作用是为站点颁发证书，这个证书中具有CA通过自己的公钥和私钥实现的签名。

为了得到签名证书，服务器需要通过自己的私钥生成CSR文件，CA机构将通过这个文件颁发属于该服务器的签名证书，只要通过CA机构就能验证证书是否合法。

1.生成扮演CA角色需要的文件
```
genrsa -out ca.key 1024 //生成了一个ca.key文件

req -new -key ca.key -out ca.csr //输入一堆信息确认后生成一歌ca.csr文件

x509 -req -in ca.csr -signkey ca.key -out ca.crt //生成一个ca.crt文件
```

2.服务器向CA机构申请签名证书。
```
req -new -key server.key -out server.csr
```
这一步报错了
```
problem creating object tsa_policy1=1.2.3.4.1
10964:error:08064066:object identifier routines:OBJ_create:oid exists:crypto\objects\obj_dat.c:698:
error in req
```
顺便找到一个linux下的解决方案：https://www.howtoforge.com/tutorial/how-to-install-openssl-from-source-on-linux/

有人说是openssl版本导致的，所以换个版本从新下载安装。1.0.2
重装之后，ojbk。
会在当前目录生成一个`server.csr`文件。

然后现在可以向自己的CA机构申请签名了，生成签名需要CA的证书和私钥，最终颁发一个带有CA签名的证书。
```
x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt
```
生成server.crt文件。

客户端在发起请求回去获取服务器端的证书，并通过CA证书验证服务器证书的真伪。

## TSL服务
他喵的，怎么验证都失败，心态崩了，感觉是csr生成时信息没填好。
Demo github 地址：https://github.com/ruomuc/test_demos/tree/master/openssl_demo

--------------------2018年11月10日 11:32:02--------------------------
ps：https的demo也是一样的问题，我觉得就是生成csr的时候域名每填好。。。orz
就先了解一下吧。
