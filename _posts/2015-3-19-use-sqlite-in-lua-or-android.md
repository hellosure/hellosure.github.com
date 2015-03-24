---
layout: post
title: SQLite的使用
category: SQLite
tags: [SQLite,Lua,Android]
---

### sqlite简介

SQLite 数据库中所有的信息（比如表、视图、触发器等）都包含在一个文件夹内，方便管理和维护。

SQLite 数据库通过数据库级上的独占性和共享锁来实现独立事务处理。这意味着多个进程可以在同一时间从同一数据库读取数据，但只能有一个可以写入数据。

### 在lua中使用sqlite

jni/Application.mk中CC_USE_SQLITE := 0改为1，就可以把sqlite打包到so文件中。

官方文档说明
<http://cn.cocos2d-x.org/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/native/v3/sqlite/zh.md>

### 在android中使用sqlite

<http://www.cnblogs.com/Excellent/archive/2011/11/19/2254888.html>

-EOF-
