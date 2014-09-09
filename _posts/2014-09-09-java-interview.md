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

`hashtable`和`synchronizedMap`是同步，线程安全。
但是有并发性问题：同步在单个锁，一次只有一个线程可以访问集合。
`concurrentHashMap`解决了这个并发问题：
1. 读支持完全并发，value是volatile
2. 支持一定程度的并发写，分段segment相当一个`hashtable`，相当于有分段写锁。

#### 队列 [多线程]

`BlockingQueue`是阻塞队列，使用可重入锁`ReentrantLock`
`ConcurrentLinkedQueue`是非阻塞队列，无锁，使用非阻塞算法

#### 池 [Java基础]

设计一个线程池、连接池

#### 动态代理 [Java基础]

1. JDK动态代理
2. CGLIB

#### 反射 [Java基础]

动态语言特性，**运行期**

#### 异常与错误 [Java基础]

1. Error，退出
2. Exception，受检查异常、运行时异常

#### final对象 [Java基础]

引用不可变，但是内存可变

#### HashSet [Java基础]

`HashSet`是由`HashMap`实现的

#### 默认运行模式 [设计模式]

interface - abstract class - class

#### NIO [并发]

非阻塞IO，channel - buffer

#### HTTP [web]

HTTP长短连接
HTTP head信息

#### ORM [J2EE]

为什么要用ORM框架
延迟加载：提升速度，节省内存

#### 数据库大数据量 [数据库优化]

拆分表，索引

#### SQL例子 [SQL]

表为student(id,class,score)，班级平均分小于60的班级：

    select class from student group by class having avg(score)<60;

#### cookie和session [HTTP]

HTTP无状态
1. cookie，客户端
2. session，用于在客户端和服务端保持状态，在客户端需要cookie的配合

#### 内部类 [Java基础]

非静态的内部类有外部类的引用，包括私有变量。
在外部类之外创建内部类，需要先创建外部对象。

#### 同步 [JMM]

`volatile` 从主存读取最新值
`ThreadLocal` 数据存于各线程栈
`ReentrantLock`的同步比`synchronized`慢，另外前者用与同步激烈时，后者用于同步不激烈时。

#### 序列化 [Java基础]

`transient` 字段不序列化

#### 泛型 [Java基础]

类型擦除，用Object代替。
在编译器就会将泛型擦除，字节码中不包括泛型。

#### 排序 [算法]

1. 稳定：冒泡、插入、归并
2. 不稳定：选择、快速、堆

#### 算法设计题

一个词典，每条是字符串，该词典条目可能有非常多。
现在要求设计一种算法，要求将每个字符串与整型一一映射，满足这样两个要求：
1. 从字符串到整型，以及从整型到字符串，都可以 O(1) 找到对方
2. 插入方便，空间复杂度低。

思考过程：

    O(1)可以联想到哈希算法。
    整型是个很好的信息，可以联想到内存地址。
    那么设计hashcode算法，将字符串可以映射到hash值，直接作为内存地址。
    这样知道字符串可以求哈希值找到整型，知道整型可以从内存中找到字符串，都是O(1)。
    从插入方便，可以想到链表。

-EOF-
