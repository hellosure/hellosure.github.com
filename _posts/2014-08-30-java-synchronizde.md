---
layout: post
title: Java多线程之由synchronized说开去
category: Java
tags: [Java,多线程,同步,synchronized,JMM,ThreadLocal,ReentrantLock]
---

迁移自[Java多线程总结之由synchronized说开去](http://hellosure.iteye.com/blog/1121157)

### 题纲：

+ synchronized与wait/notify
+ JMM与synchronized
+ ThreadLocal与synchronized
+ ReentrantLock与synchronized

### synchronized与wait/notify：

#### synchronized

synchronized是针对对象的隐式锁使用的，注意是对象！ 

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

线程想要执行myClass.myFunction(); 就要先获得myClass对象的锁。     

      
##### synchronized作用域： 

1) 某个对象实例内，synchronized  aMethod(){}可以防止多个线程同时访问这个对象的synchronized方法（如果一个对象有多个synchronized方法，只要一个线程访问了其中的一个synchronized方法，其它线程不能同时访问这个对象中任何一个synchronized方法）。这时，不同的对象实例的synchronized方法是不相干扰的。也就是说，其它线程照样可以同时访问相同类的另一个对象实例中的synchronized方法； 

2) 某个类的范围，synchronized static aStaticMethod{}防止多个线程同时访问这个类中的synchronized static方法。它可以对类的所有对象实例起作用。（注：这个可以认为是对Class对象起作用） 

除了方法前用synchronized关键字，synchronized关键字还可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。用法是: synchronized(this){...}，它的作用域是this，即是当前对象。当然这个括号里可以是任何对象，synchronized对方法和块的含义和用法无本质不同； 

synchronized关键字是不能继承的，也就是说，基类的方法synchronized f(){} 在继承类中并不自动是synchronized f(){}，而是变成了f(){}。继承类需要你显式的指定它的某个方法为synchronized方法；       

         
##### synchronized可能造成死锁

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

> 假设场景：线程A调用methodOne()，获得lock1的隐式锁后，在获得lock2的隐式锁之前线程B进入运行，调用methodTwo()，抢先获得了lock2的隐式锁，此时线程A等着线程B交出lock2，线程B等着lock1进入方法块，死锁就这样被创造出来了。      
     
     
#### wait/notify

**wait/notify：调用任意对象的wait方法导致线程阻塞，并且该对象上的锁被释放。**

**而调用任意对象的notify方法则导致因调用该对象的wait方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。**       
        
           
#### synchronized与wait/notify关系

有synchronized的地方不一定有wait,notify 

有wait,notify的地方必有synchronized。
这是因为wait和notify不是属于线程类，而是每一个对象都具有的方法（事实上，这两个方法是Object类里的），而且这两个方法都和对象锁有关，有锁的地方，必有synchronized。

慢着，让我们思考一下Java这个设计是否合理？前面说了，锁是针对对象的，wait/notify的操作是与对象锁相关的，那么把wait/notify设计在Object中也就是合情合理的了。 

恩，再想一下，为什么有wait,notify的地方必有synchronized？ 
synchronized方法中由当前线程占有锁。另一方面，调用wait/notify方法的对象上的锁必须为当前线程所拥有。因此，wait/notify方法调用必须放置在synchronized方法中，**synchronized方法的上锁对象就是调用wait()notify()方法的对象**。若不满足这一条件，则程序虽然仍能编译，但在运行时会出现IllegalMonitorStateException 异常。       
         
         
##### 举了栗子-银行转账

同一时刻只有一个人可以转账，那么我们自然想到在Bank类中有一个同步的转账方法： 

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

这样就满足需求了。 
**可见，用对象锁来管理试图进入synchronized方法的线程， **
**另外，由条件判断来管理已经进入同步方法中的线程即当前线程** 

这里还补充两点： 
1) 调用wait()方法前的判断最好用while，而不用if；因为while可以实现被唤醒后线程再次作条件判断；而if则只能判断一次 
2) 用notifyAll()优先于notify()。 

另外注意一点： 
**能调用wait()/notify()的只有当前线程，前提是必须获得了对象锁，就是说必须要进入到synchronized方法中。 **
        
### JMM与synchronized：

#### JMM

JVM中（留神：马上讲到的这两个存储区只在JVM内部与物理存储区无关）存在一个主内存（Main Memory），Java中所有的变量存储在主内存中，所有实例和实例的字段都在此区域，对于所有的线程是共享的（相当于黑板，其他人都可以看到的）。

每个线程都有自己的工作内存（Working Memory），工作内存中保存的是主存中变量的拷贝，工作内存由缓存和堆栈组成，其中缓存保存的是主存中的变量的copy，堆栈保存的是线程局部变量。

线程对所有变量的操作都是在工作内存中进行的，线程之间无法直接互相访问工作内存，变量的值得变化的传递需要主存来完成。在JMM中通过并发线程修改的变量值，必须通过线程变量同步到主存后，其他线程才能访问到。 

看看这个图是不是更形象：

![jmm](http://photo2.bababian.com/usr832855/upload1/20090614/sBtYKP1GU1okcV3oHvBCcWyO9WOwCsqaK9YtCbHK70RnhSeacQjf2Mg==.jpg "JMM")

线程对某个变量的操作步骤： 
1. 从主内存中复制数据到工作内存 
2. 执行代码，对数据进行各种操作和计算 
3. 把操作后的变量值重新写回主内存中 

> 现在举个例子，设想两个棋手要通过两个终端显示器(Working Memory)对奕，而观众要通过服务器大屏幕(Main Memory )观看他们的比赛过程。这两个棋手相当于是同步中的线程，观众相当于其它线程。棋手是无法直接操作服务器的大屏幕的，他只能看到自己的终端显示器，只能先从服务器上将当前结果先复制到自己的终端上（步骤1），然后在自己的终端上操作（步骤2），将操作的结果记录在终端上，然后在某一时刻同步到服务器上（步骤3）。他所能看到的结果就是从服务器上复制到自己的终端上的内容，而要想把自己操作后的结果让其他人看到必须同步到服务器上才行。至于什么时候同步，那要看终端和服务器的通信机制。 

回到这三个步骤，这个顺序是我们希望的，但是，JVM并不保证第1步和第3步会严格按照上述次序立即执行。因为根据java语言规范的 规定，线程的工作内存和主存间的数据交换是松耦合的，什么时候需要刷新工作内存或者什么时候更新主存的内容，可以由具体的虚拟机实现自行决定。由于JVM可以对特征代码进行调优，也就改变了某些运行步骤的次序的颠倒，那么每次线程调用变量时是直接取 自己的工作存储器中的值还是先从主存储器复制再取是没有保证的，任何一种情况都可能发生。同样的，线程改变变量的值之后，是 否马上写回到主存储器上也是不可保证的，也许马上写，也许过一段时间再写。那么，在多线程的应用场景下就会出现问题了，多个 线程同时访问同一个代码块，很有可能某个线程已经改变了某变量的值，当然现在的改变仅仅是局限于工作内存中的改变，此时JVM并 不能保证将改变后的值立马写到主内存中去，也就意味着有可能其他线程不能立马得到改变后的值，依然在旧的变量上进行各种操作 和运算，最终导致不可预料的结果。            
         
#### synchronized和volatile

这可如何是好呢？还好有synchronized和volatile： 
1) 多个线程共有的字段应该用synchronized或volatile来保护. 

2) synchronized负责线程间的互斥.即同一时候只有一个线程可以执行synchronized中的代码. 
**synchronized还有另外一个方面的作用**：在线程进入synchronized块之前，会把工作存内存中的所有内容映射到主内存上，然后> 把工作内存清空再从主存储器上拷贝最新的值。而在线程退出synchronized块时，同样会把工作内存中的值映射到主内存，不过此时> 并不会清空工作内存。这样一来就可以强制其按照上面的顺序运行，以保证线程在执行完代码块后，工作内存中的值和主内存中的值> 是一致的，保证了数据的一致性！ 

3) volatile负责线程中的变量与主存储区同步.但不负责每个线程之间的同步. 
volatile的含义是：**线程在试图读取一个volatile变量时，会从主内存区中读取最新的值。现在很清楚了吧。**
          
### ThreadLocal与synchronized

在JDK的API文档中ThreadLocal的定义第一句道出：This class provides thread-local variables.
好，这个类提供了线程本地的变量。只看这一句，让我们结合到上面JMM的知识我们来分析一下理一下头绪： 
我们已经知道了synchronized的含义是同步，也就是针对的是主存中的变量，只不过多线程执行时为了实现同步就需要每个线程在操作这个变量时要完成那三个步骤（对，就是主存与线程工作内存之间完成交互的那三步）。

我们很自然想到： 
使用目的：需要有某些变量在多个线程中共享，有共享才会需要同步。 
执行效率：直观上感觉一下，同步的执行效率肯定不高，事实上也的确是这样，为什么？看看那三步多麻烦。 

好，现在再来看看ThreadLocal的定义，我们能想到什么？ 
首先让我们思考一个问题，并不是所有多线程程序都需要共享啊，这个时候还用同步那一套岂不是很多余？让每个线程维护自己的变量不就OK了，反正又不需要共享。对，ThreadLocal就是干这个事的。另一方面，那不用多说，性能上肯定优越喽。 

小结一下：
对比synchronized和ThreadLocal首先要清楚，两者的使用目的不同，关键点就在是否需要共享变量。就是说，**ThreadLocal根本不是同步**。
再说啰嗦一点：ThreadLocal和Synchonized都用于解决多线程并发访问。但是ThreadLocal与synchronized有本质的区别，synchronized是利用锁的机制，使变量或代码块在某一时该只能被一个线程访问。而ThreadLocal为每一个线程都提供了变量的副本，使得每个线程在某一时间访问到的并不是同一个对象，这样就隔离了多个线程对数据的数据共享。Synchronized用于线程间的数据共享，而ThreadLocal则用于线程间的数据隔离。两者处于不同的问题域。 

举个栗子：

{% highlight java %}
public class ThreadLocalDemo implements Runnable {    
   private final static  ThreadLocal studentLocal = new ThreadLocal();  //ThreadLocal对象在这  
       
   public static void main(String[] agrs) {    
       TreadLocalDemo td = new TreadLocalDemo();    
         Thread t1 = new Thread(td,"a");    
         Thread t2 = new Thread(td,"b");    
        t1.start();    
        t2.start();     
      }    
       
    public void run() {    
         accessStudent();    
    }    
    
    public  void  accessStudent() {    
        String currentThreadName = Thread.currentThread().getName();    
        System.out.println(currentThreadName+" is running!");    
        Random random = new Random();    
        int age = random.nextInt(100);    
        System.out.println("thread "+currentThreadName +" set age to:"+age);    
        Student student = getStudent();  //每个线程都独立维护一个Student变量  
        student.setAge(age);    
        System.out.println("thread "+currentThreadName+" first read age is:"+student.getAge());    
        try {    
        Thread.sleep(5000);    
        }    
        catch(InterruptedException ex) {    
            ex.printStackTrace();    
        }    
        System.out.println("thread "+currentThreadName+" second read age is:"+student.getAge());    
            
    }    
        
    protected Student getStudent() {    
        Student student = (Student)studentLocal.get();  //从ThreadLocal对象中取  
        if(student == null) {    
            student = new Student();    
            studentLocal.set(student);  //如果没有就创建一个  
        }    
        return student;    
    }    
        
    protected void setStudent(Student student) {    
        studentLocal.set(student);  //放入ThreadLocal对象中  
    }    
}    
{% endhighlight %}

ThreadLocal通过一个Map来为每个线程都持有一个变量副本，用ThreadLocal对象以键值对的方式来维护这些线程独立变量 。

### ReentrantLock与synchronized

既然说到了synchronized，顺便说说ReentrantLock吧。 
ReentrantLock不熟悉？没事，concurrent包里的ArrayBlockingQueue知道吧，去看看源码，发现了吧，里面全是ReentrantLock。 
好，言归正传，ReentrantLock是何方神圣？先这么说吧，你可以认为ReentrantLock是具有和synchronized类似功能的性能功能加强版同步锁。            
          
#### synchronized缺点： 

1） 只有一个condition与锁相关联，这个condition是什么？就是synchronized对针对的对象锁。 
2） 多线程竞争一个锁时，其余未得到锁的线程只能不停的尝试获得锁，而不能中断。这种情况对于大量的竞争线程会造成性能的下降等后果。         
               
#### ReentrantLock

针对synchronized的一系列缺点，JDK5提供了ReentrantLock，目的是为同步机制进行改善。

下面来看看它是怎么改善上面这两个缺点的： 
1） 一个ReentrantLock可以有多个Condition实例。 
举个例子，还是刚才说的ArrayBlockingQueue类，看看源码（节选）： 

{% highlight java %}
public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {    
    ...    
    private final ReentrantLock lock;    
    private final Condition notEmpty;    
    private final Condition notFull;    
    ...    
    public ArrayBlockingQueue(int capacity, boolean fair) {    
        ...  
        lock = new ReentrantLock(fair);    
        notEmpty = lock.newCondition();    
        notFull =  lock.newCondition();  //为该ReentrantLock设置了两个Condition   
    }    
   
    public E take() throws InterruptedException {    
        final ReentrantLock lock = this.lock;    
        lock.lockInterruptibly();    
        try {    
            try {    
                while (count == 0)    
                    notEmpty.await();  //针对notEmpty这个condition，如果队列为空则线程等待这个条件  
            } catch (InterruptedException ie) {    
                notEmpty.signal();   
                throw ie;    
            }    
            E x = extract();    
            return x;    
        } finally {    
            lock.unlock();    
        }    
    }    
    
    private E extract() {    
        final E[] items = this.items;    
        E x = items[takeIndex];    
        items[takeIndex] = null;    
        takeIndex = inc(takeIndex);    
        --count;    
        notFull.signal();  //这里针对notFull这个condition，唤醒因该条件而等待的线程  
        return x;    
    }    
    ...    
}    
{% endhighlight %}

这里notEmpty和notFull作为lock的两个条件是可以分别负责管理想要加入元素的线程和想要取出元素的线程。例如put()方法在元素个数达到最大限制时会使用notFull条件把试图继续插入元素的线程都扔到等待集中，而执行了take()方法时如果顺利进入extract()则会空出空间，这时notFull负责随机的通知被其扔到等待集中的线程执行插入元素的操作。（这里没给出put方法，有兴趣的童鞋可以去查查源码，其实和take方法很类似） 

2) ReentrantLock提供了lockInterruptibly()方法可以优先考虑响应中断，而不是像synchronized那样不响应interrupt()操作。 
解释一下响应中断是什么意思：比如A、B两线程去竞争锁，A得到了锁，B等待，但是A有很多事情要处理，所以一直不返回。B可能就会等不及了，想中断自己，不再等待这个锁了，转而处理其他事情。在这种情况下，synchronized的做法是，B线程中断自己（或者别的线程中断它），我不去响应，继续让B线程等待，你再怎么中断，我全当耳边风。而lockInterruptibly()的做法是，B线程中断自己（或者别的线程中断它），ReentrantLock响应这个中断，不再让B等待这个锁的到来。 
有了这个机制，使用ReentrantLock时就不会像synchronized那样产生死锁了。 
                 
#### ReentrantLock与synchronized对比

由于ReentrantLock在提供了多样的同步功能（除了可响应中断，还能设置时间限制），因此在同步比较激烈的情况下，性能比synchronized大大提高。 
不过，在同步竞争不激烈的情况下，synchronized还是非常合适的（因为JVM会进行优化，具体不清楚怎么优化的）。

因此不能说ReentrantLock一定更好，只是两者适合情况不同而已，在同步竞争不激烈时用synchronized，激烈时用ReentrantLock。
换句话说，ReentrantLock的可伸缩性可并发性要更好一些。除非您对 ReentrantLock的某个高级特性有明确的需要，或者有明确的证据（而不是仅仅是怀疑）表明在特定情况下，同步已经成为可伸缩性的瓶颈，否则还是应当继续使用 synchronized。这里推荐一个帖子[可伸缩性的锁定机制](http://www.ibm.com/developerworks/cn/java/j-jtp10264/) 

再补充一点，使用ReentrantLock时，切记要在finally中释放锁，这是与synchronized使用方式很大的一个不同。
对于synchronized，JVM会自动释放锁，而ReentrantLock需要你自己来处理。 
{% highlight java %}
//synchronized   
public synchronized void increment() {  
    count++;  
}  
{% endhighlight %}

{% highlight java %}
//ReentrantLock  
public void increment() {  
    lock.lockInterruptibly();//上锁  
    try {  
        count++;  
    } finally {  
        lock.unlock();//手动释放锁  
    }  
}  
{% endhighlight %}


-EOF-
