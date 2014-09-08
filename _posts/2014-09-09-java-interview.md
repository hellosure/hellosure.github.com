---
layout: post
title: 当面试官时问的题目
category: Java
tags: [Java,MySQL,J2EE,linux,面试]
---

### 大纲

|| *类型* || *内容* ||
|| **Java** || OO ||
||  || 反射 ||
||  || 泛型 ||
||  || collection ||
||  || 多线程 ||
||  || JDBC ||
||  || JDK ||
||  || JVM ||
||  || GC ||
||  || 网络 ||
||  || 算法数据结构 ||
||  || 单测 ||
||  || 异常 ||
||  || 引用 ||
||  || concurrent ||
|| **MySQL** || SQL ||
||  || 事务 ||
||  || 索引 ||
||  || 存储过程 ||
||  || 优化 ||
||  || MongoDB ||
|| **J2EE** || JSP/servlet/filter ||
||  || struts/interceptor ||
||  || spring/ioc/aop/annotation ||
||  || hibernate/hql/lazy-load ||
||  || SSH分层 ||
||  || 模块划分 ||
||  || 设计模式 ||
||  || tomcat ||
||  || 服务化/RPC/REST ||
|| **Linux** || shell ||
||  || awk ||

### 题目举例

#### 创建线程 [多线程]

方法一：

    MyThread extends Thread {
      run();
    }
    new MyThread();
    start();
    
方法二：

    MyThread implements Runnable {
      run();
    }
    new Thread(new MyThread());
    start();
    
#### 同步 [多线程]

synchrionized:

1. 静态：类锁
2. 方法：对象锁
3. 块：任意对象锁

#### 阻塞 [多线程]


