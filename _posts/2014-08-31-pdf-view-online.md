---
layout: post
title: PDF在线阅读器的使用
category: PDF
tags: [PDF]
---

Blog中的PDF链接，最好是可以提供在线阅读器来直接打开。这样一是可以不用把所有文件都上传到attach，二是比下载文件的体验要好。

Google docs viewer不能使用，原因你懂的。于是要找其他方法。

#### 解决过程

1) 在[Google](https://wen.lu)中搜关键字：pdf viewer online

2) 试了一下，这个[在线提供PDF阅读器的网站](http://view.samurajdata.se/)能使用。

3) 将PDF的上传地址输入进去，然后就能生成在线PDF阅读URL。
这里试了一下直接上传本地PDF无法生成URL。

-----------------------------------------------------

2014-09-09更新

发现上面的方法有时间限制，过了一段时间之后，上传的pdf会被从该网站的cache中清除，则链接不再有效。

于是又找了个方法来实现：

1) [pdfobject](http://www.pdfobject.com/)，min.js放入assets/js中

2) 在网站根路径里添加`android-ui.html`

3）将_post文章中的在线阅读地址链接到[这里](http://hellosure.github.io/android-ui.html)

PS：不过这样也有一个问题就是，需要浏览器的支持。

-EOF-
