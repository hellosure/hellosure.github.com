---
layout: post
title: Java多线程总结之由synchronized说开去
category: Java
tags: [Java,多线程]
---

迁移自[Java多线程总结之由synchronized说开去](http://hellosure.iteye.com/blog/1121157)

+ synchronized与wait()/notify()
+ JMM与synchronized
+ ThreadLocal与synchronized
+ ReentrantLock与synchronized

## synchronized与wait()/notify()：
 
最重要一条： **synchronized是针对对象的隐式锁使用的，注意是对象！ **

举个小例子，该例子没有任何业务含义，只是为了说明synchronized的基本用法：

{% highlight java %}
Class MyClass(){ 
  	synchronized void myFunction(){  
    		//do something  
  	}  
}  
  
public static void main(){  
  	MyClass myClass = new MyClass();  
  	myClass.myFunction();  
}  
{% endhighlight %}

好了，就这么简单。 

> myFunction()方法是个同步方法，隐式锁是谁的？答：是该方法所在类的对象。 
> 看看怎么使用的：myClass.myFunction();很清楚了吧，隐式锁是myClass的。 
> 说的再明白一点，**线程想要执行myClass.myFunction();就要先获得myClass的锁。**

下面总结一下： 

1. synchronized关键字的作用域有二种： 

> 1）是某个对象实例内，synchronized  aMethod(){}可以防止多个线程同时访问这个对象的synchronized方法
>（如果一个对象有多个synchronized方法，只要一个线程访问了其中的一个synchronized方法，其它线程不能同时访问这个对象中任 > 何一个synchronized方法）。这时，不同的对象实例的synchronized方法是不相干扰的。也就是说，其它线程照样可以同时访问相同> 类的另一个对象实例中的synchronized方法； 

> 2）是某个类的范围，synchronized static aStaticMethod{}防止多个线程同时访问这个类中的synchronized static方法。
> 它可以对类的所有对象实例起作用。（注：这个可以认为是对Class对象起作用） 

2. 除了方法前用synchronized关键字，synchronized关键字还可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。用法是: synchronized(this){...}，它的作用域是this，即是当前对象。当然这个括号里可以是任何对象，synchronized对方法和块的含义和用法并不本质不同； 

3. synchronized关键字是不能继承的，也就是说，基类的方法synchronized f(){} 在继承类中并不自动是synchronized f(){}，而是变成了f(){}。继承类需要你显式的指定它的某个方法为synchronized方法； 

synchronized可能造成死锁，比如：
{% highlight java %}
class DeadLockSample{    
    public final Object lock1 = new Object();    
    public final Object lock2 = new Object();    
    
    public void methodOne(){    
       synchronized(lock1){    
          ...    
          synchronized(lock2){...}    
       }    
    }    
    
    public void methodTwo(){    
       synchronized(lock2){    
      ...    
          synchronized(lock1){...}    
       }    
    }    
}  
{% endhighlight %}

> 假设场景：线程A调用methodOne()，获得lock1的隐式锁后，在获得lock2的隐式锁之前线程B进入运行，调用methodTwo()，抢先获得> 了lock2的隐式锁，此时线程A等着线程B交出lock2，线程B等着lock1进入方法块，死锁就这样被创造出来了。 

下面举一个有业务含义的例子帮助理解，并展示一下synchronized与wait()、notifyAll()的使用。 
> 这里先介绍一下这两个方法： 
> **wait()/notify()：调用任意对象的 wait() >方法导致线程阻塞，并且该对象上的锁被释放。而调用任意对象的notify()方法则导致因调用该对象的 wait() >方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。** 

好了，再来看看synchronized与这两个方法之间的关系： 
>1. 有synchronized的地方不一定有wait,notify 
>
>2. 有wait,notify的地方必有synchronized.这是因为wait和notify不是属于线程类，而是每一个对象都具有的方法（事实上，这两个>方法是Object类里的），而且，这两个方法都和对象锁有关，有锁的地方，必有synchronized。
>
>慢着，让我们思考一下Java这个设计是否合理？前面说了，锁是针对对象的，wait()/notify()的操作是与对象锁相关的，那么把wait>()/notify()设计在Object中也就是合情合理的了。 
>
>恩，再想一下，为什么有wait,notify的地方必有synchronized？ 
>synchronized方法中由当前线程占有锁。另一方面，调用wait()notify()方法的对象上的锁必须为当前线程所拥有。因此，wait()/no>tify()方法调用必须放置在synchronized方法中，**synchronized方法的上锁对象就是调用wait()notify()方法的对象**。若不满足这>一条件，则程序虽然仍能编译，但在运行时会出现IllegalMonitorStateException 异常。 

好了，以上准备知识充足了，现在说例子：银行转账，同一时刻只有一个人可以转账。 
那么我们自然想到在Bank类中有一个同步的转账方法： 
{% highlight java %}
public Class Bank(){  
  float account[ACCOUNT_NUM];  
  ...  
  public synchronized void transfer(from, to, amount){  
    //转账  
  }  
}  
{% endhighlight %}

现在有一个问题，如果一个人获得了使用银行的锁，但是余额不足怎么办？ 
好，那我们进行改进： 
{% highlight java %}
public Class Bank(){  
  float account[ACCOUNT_NUM];  
  ...  
  public synchronized void transfer(int from, int to, float amount){  
    while(account[from]){  
      wait();  
    }  
    account[from] -= amount;  
    account[to] += amount;  
    notifyAll();  
  }  
}  
{% endhighlight %}

> 这样就满足需求了。 
>**可见，用对象锁来管理试图进入synchronized方法的线程， **
>**另外，由条件判断来管理已经进入同步方法中的线程即当前线程** 

> 这里还补充两点： 
>1. 调用wait()方法前的判断最好用while，而不用if；因为while可以实现被唤醒后线程再次作条件判断；而if则只能判断一次 
>2. 用notifyAll()优先于notify()。 
>另外注意一点： 
>**能调用wait()/notify()的只有当前线程，前提是必须获得了对象锁，就是说必须要进入到synchronized方法中。 **

## JMM与synchronized：
补充一点JMM的相关知识，对理解线程同步很有好处。 

> JVM中（留神：马上讲到的这两个存储区只在JVM内部与物理存储区无关）存在一个主内存（Main >Memory），Java中所有的变量存储在主内存中，所有实例和实例的字段都在此区域，对于所有的线程是共享的（相当于黑板，其他人都>可以看到的）。
>
>每个线程都有自己的工作内存（Working >Memory），工作内存中保存的是主存中变量的拷贝，（相当于自己笔记本，只能自己看到），工作内存由缓存和堆栈组成，其中缓存保>存的是主存中的变量的copy，堆栈保存的是线程局部变量。
>
>线程对所有变量的操作都是在工作内存中进行的，线程之间无法直接互相访问工作内存，变量的值得变化的传递需要主存来完成。在J>MM中通过并发线程修改的变量值，必须通过线程变量同步到主存后，其他线程才能访问到。 

看看这个图是不是更形象
![jmm](http://photo2.bababian.com/usr832855/upload1/20090614/sBtYKP1GU1okcV3oHvBCcWyO9WOwCsqaK9YtCbHK70RnhSeacQjf2Mg==.jpg "JMM")

> 好啦，下面来看线程对某个变量的操作步骤： 
>1. 从主内存中复制数据到工作内存 
>2. 执行代码，对数据进行各种操作和计算 
>3. 把操作后的变量值重新写回主内存中 

> 现在举个例子，设想两个棋手要通过两个终端显示器(Working Memory)对奕，而观众要通过服务器大屏幕(Main Memory >)观看他们的比赛过程。这两个棋手相当于是同步中的线程，观众相当于其它线程。棋手是无法直接操作服务器的大屏幕的，他只能看>到自己的终端显示器，只能先从服务器上将当前结果先复制到自己的终端上（步骤1），然后在自己的终端上操作（步骤2），将操作的>结果记录在终端上，然后在某一时刻同步到服务器上（步骤3）。他所能看到的结果就是从服务器上复制到自己的终端上的内容，而要>想把自己操作后的结果让其他人看到必须同步到服务器上才行。至于什么时候同步，那要看终端和服务器的通信机制。 

> 回到这三个步骤，这个顺序是我们希望的，但是，JVM并不保证第1步和第3步会严格按照上述次序立即执行。因为根据java语言规范的> 规定，线程的工作内存和主存间的数据交换是松耦合的，什么时候需要刷新工作内存或者什么时候更新主存的内容，可以由具体的虚> 拟机实现自行决定。由于JVM可以对特征代码进行调优，也就改变了某些运行步骤的次序的颠倒，那么每次线程调用变量时是直接取> 自己的工作存储器中的值还是先从主存储器复制再取是没有保证的，任何一种情况都可能发生。同样的，线程改变变量的值之后，是> 否马上写回到主存储器上也是不可保证的，也许马上写，也许过一段时间再写。那么，在多线程的应用场景下就会出现问题了，多个> 线程同时访问同一个代码块，很有可能某个线程已经改变了某变量的值，当然现在的改变仅仅是局限于工作内存中的改变，此时JVM并> 不能保证将改变后的值立马写到主内存中去，也就意味着有可能其他线程不能立马得到改变后的值，依然在旧的变量上进行各种操作> 和运算，最终导致不可预料的结果。 

> 这可如何是好呢？还好有synchronized和volatile： 
> 1. 多个线程共有的字段应该用synchronized或volatile来保护. 
> 2. synchronized负责线程间的互斥.即同一时候只有一个线程可以执行synchronized中的代码. 
> **synchronized还有另外一个方面的作用**：在线程进入synchronized块之前，会把工作存内存中的所有内容映射到主内存上，然后> 把工作内存清空再从主存储器上拷贝最新的值。而在线程退出synchronized块时，同样会把工作内存中的值映射到主内存，不过此时> 并不会清空工作内存。这样一来就可以强制其按照上面的顺序运行，以保证线程在执行完代码块后，工作内存中的值和主内存中的值> 是一致的，保证了数据的一致性！ 
> 3. volatile负责线程中的变量与主存储区同步.但不负责每个线程之间的同步. 
volatile的含义是：**线程在试图读取一个volatile变量时，会从主内存区中读取最新的值。现在很清楚了吧。**

-EOF-
