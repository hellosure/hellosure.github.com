---
layout: post
title: 当面试官时问过的题目
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

`synchrionized`:

1. 静态：类锁
2. 方法：对象锁
3. 块：任意对象锁

#### 阻塞与锁 [多线程]

`sleep()` 阻塞(CPU不可分配) -> 到时可执行(CPU可分配)
但是注意，sleep并不释放锁！其实我理解sleep和锁应该没什么关系吧，因为不需要在`synchrionized`中也能执行`sleep()`
这里需要理解的就是，如果在`synchrionized`中进行`sleep()`，那么当前线程阻塞在原地，但是其他线程没有机会来抢占锁，因为当前线程并没有让出执行`synchrionized`的资格。这点是与`wait()`的区别之处。

`yield()` 放弃CPU，CPU再重新调度一次

`join()` 等待这个线程运行结束，再向下执行

`wait()` 当前线程释放锁，阻塞在原地，其他线程有机会抢占锁
必须在`synchrionized`中才能获得锁，没有获得何来释放，因此`wait()`肯定需要在`synchrionized`中。
当前线程释放了锁，就意味着它失去了在`synchrionized`中继续执行的资格。
而其他想进入`synchrionized`的线程就可以去抢占这个锁了。

`notifyAll()` 由于Object而`wait()`的线程全部被唤醒，(因为`wait()`和`notify()`的所对象是一致的，就是调用它们的Objct)，那么这些线程就有了去抢占锁的资格，如果某线程抢占到了锁，那么就可以继续执行`synchrionized`了。

另外说一点，`wait()`与`notify()`的配合有可能产生死锁，这一点Java不保证。

>  小结：这个题目主要需要理解的是**阻塞和锁**是完全两码事，是完全两个东西。阻塞指的是当前线程不继续向下执行了，`sleep()`和`wait()`造成的结果都是阻塞，但是`sleep()`不释放锁，等待一定的时间之后确保能够执行，而`wait()`释放锁然后进入wait线程队列，需要别的线程来唤醒它才能进入notify线程队列从而再次去抢占锁的所有权。

#### 构造器 [Java基础]

    class A {
      int a = 1;
      A(){
        a = 2;
      }
    }

new A();之后，a为2.

#### clone [Java基础]

Object.clone 是浅拷贝

实现深拷贝的方法：
+ Cloneable重写clone()
+ 序列化之后再反序列化：ObjectInputStream/ObjectOutputStream

#### 重载 [Java基础]

String中有这样两个重载方法：

    valueOf(Object obj);    //1
    valueOf(char[] data);   //2

当这样调用时：

    Object obj = null;
    String.valueOf(obj);

实际调用的是方法2，因为2比1更精确匹配一些。

#### Object [Java基础]

`hashcode()`和`equals()`的关系。

#### 继承 [Java基础]

    Father(){
      getName();
      getFatherAge();
    }
      
    Son(){
      getName();
      getSonAge();
    }

当这样调用时：

    Father f = new Son();
    f.getName();            //调用的是Son的方法，这是方法覆盖
    ((Father)f).getName();  //注意这个调用的也是Son的方法，同样是方法覆盖，和上一句没有什么区别
    f.getFatherAge();       //可以调用父类的非私有方法

另外在Son中通过`super.getName()`是可以调用到父类的被覆盖方法的。

#### 集合 [多线程]


    
