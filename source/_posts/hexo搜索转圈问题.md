---
title: hexo搜索转圈问题
categories:
  - others
tags:
  - 其它
date: 2020-05-22 17:10:19
updated: 2020-05-22 17:10:19
cover: https://proxy.qnoss.seeln.com/images/wp4221822-asteroid-belt-wallpapers.jpg
---

今天hexo的搜索功能一直转菊花，对于我这种强迫症来说，博客没有搜索也太难受了吧。

google了一下发现很多人也遇到了这个问题。

说一下我的解决步骤吧。



##### 原因

我打开控制台，点击搜索，获取`search.xml`文件时，并没有报错。

所以我的博客的原因是因为`xml`文件有特殊字符。

于是我先打开vscode设置，把`editor.renderControlCharacters `设置为true，用正则的方式去搜索如下内容

```
;/[\u0000]|[\u0001]|[\u0002]|[\u0003]|[\u0004]|[\u0005]|[\u0006]|[\u0007]|[\u0008]|[\u000b]|[\u000c]|[\u000d]|[\u000e]|[\u000f]|[\u0010]|[\u0011]|[\u0012]|[\u0013]|[\u0014]|[\u0015]|[\u0016]|[\u0017]|[\u0018]|[\u0019]|[\u001a]|[\u001b]|[\u001c]|[\u001d]|[\u001e]|[\u001f]|[\u001c]|[\u007f]/gm
```

but，我搜不到，于是我打开控制台，找到请求`search.xml`，点击`response`，复制内容到编辑器中，推荐用`sublime`。然后在搜索上面那串正则，搜到之后在原文修改完。

`hexo clean , hexo g ,hexo d`重新部署。

然后用无痕模式试一下，或者强制刷新`ctrl+R`再清除`search.xml`的缓存。

大功告成！



参考链接：[解决Hexo博客搜索异常](https://www.itfanr.cc/2017/11/24/resolve-hexo-blog-search-exception/)

