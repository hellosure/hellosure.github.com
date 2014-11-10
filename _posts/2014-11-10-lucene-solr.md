---
layout: post
title: 说说Lucene和Solr
category: Lucene
tags: [Lucene,Solr]
---

### Lucene和Solr有什么区别？

首先Solr是基于Lucene做的，Lucene是一套信息检索工具包，但并不包含搜索引擎系统，它包含了索引结构、读写索引工具、相关性工具、排序等功能，因此在使用Lucene时你仍需要关注搜索引擎系统，例如数据获取、解析、分词等方面的东西。
而Solr的目标是打造一款企业级的搜索引擎系统，因此它更接近于我们认识到的搜索引擎系统，它是一个搜索引擎服务，通过各种API可以让你的应用使用搜索服务，而不需要将搜索逻辑耦合在应用中。而且Solr可以根据配置文件定义数据解析的方式，更像是一个搜索框架，它也支持主从、热换库等操作。还添加了飘红、facet等搜索引擎常见功能的支持。
因而，Lucene使用上更加灵活，但是你需要自己处理搜素引擎系统架构，以及其他附加附加功能的实现。而Solr帮你做了更多，但是是一个处于高层的框架，Lucene很多新特性不能及时向上透传，所以有时候可能发现需要一个功能，Lucene是支持的，但是Solr上已经看不到相关接口。

----

Lucene更像是一个SDK。 有完整的API族以及对应的实现。你可以利用这些在自己的应用里实现高级查询（基于倒排索引技术的），Lucene对单机或者桌面应用很实用很方便。
但是Lucene，需要开发者自己维护索引文件，在多机环境中备份同步索引文件很是麻烦。于是，就有了Solr。 

而Solr是一个有HTTP接口的基于Lucene的查询服务器，封装了很多Lucene细节，自己的应用可以直接利用诸如 .../solr?q=abc 这样的HTTP GET/POST请求去查询，维护修改索引。

给个比方就是，Lucene是给你一堆包，让你自己从底层构建一个数据库。而Solr是一个实现好的数据库程序，安装后就可以直接用了。

----

Many people new to Lucene and Solr will ask the obvious question: Should I use Lucene or Solr?
The answer is simple: if you're asking yourself this question, in 99% of situations, what you want to use is Solr.
A simple way to conceptualize the relationship between Solr and Lucene is that of a car and its engine. You can't drive an engine, but you can drive a car. Similarly, Lucene is a programmatic library which you can't use as-is, whereas Solr is a complete application which you can use out-of-box.

### Solr简介

solr需要运行在一个servlet 容器里，例如tomcat5.5。solr在lucene的上层提供了一个基于HTTP/XML的Web Services，我们的应用需要通过这个服务与solr进行交互。

[http://www.ibm.com/developerworks/cn/java/j-solr1/index.html]

[http://www.ibm.com/developerworks/cn/java/j-solr2/index.html]

-EOF-
