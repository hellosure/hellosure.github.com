---
layout: post
title: 小议时序调度Timer和Quartz
category: Java
tags: [Java,Timer,Quartz]
---

迁移自[小议时序调度Timer和Quartz](http://hellosure.iteye.com/blog/1135175)

### 题纲：

+ Timer
+ Quartz

### Timer：

想实现定时调度，最简单的方法是使用Timer 
还是先给个使用范例： 

{% highlight java %}
long PERIOD = 60*1000;//一分钟  
Timer timer = new Timer("sure's timer");  
timer.schedule(new TimerTask() {  
    @Override  
    public void run() {  
        if(!doSomething()){  
            timer.cancel();  
        }  
    }  
}, 0, PERIOD);  
{% endhighlight %}

上面的代码实现了一个从当前时间开始执行的，以一分钟为周期的定时器。执行的内容在doSomething()方法中实现，这个方法返回是否继续执行的布尔值，因此可以在需要的时候将定时器cancel掉。 

让我们看一看Timer的源码。 

{% highlight java %}
public class Timer {   
    private TaskQueue queue = new TaskQueue();  
    private TimerThread thread = new TimerThread(queue);  
}  
{% endhighlight %}

Timer主要属性有以上两个，先来看看TimerThread，TimerThread是Timer的内部类，主要代码如下： 
{% highlight java %}
class TimerThread extends Thread {  
    private TaskQueue queue;//任务队列  
  
    public void run() {  
        try {  
            mainLoop();  
        } finally {  
            // Someone killed this Thread, behave as if Timer cancelled  
            synchronized(queue) {  
                newTasksMayBeScheduled = false;  
                queue.clear();    
            }  
        }  
    }  
  
    private void mainLoop() {  
        while (true) {  
            try {  
                TimerTask task;  
                boolean taskFired;  
                synchronized(queue) {  
                    // 如果队列为空则等待  
                    while (queue.isEmpty() && newTasksMayBeScheduled)  
                        queue.wait();  
                    if (queue.isEmpty())  
                        break; // 如果不再会非空，则跳出  
  
                    // 队列非空，则取第一个任务执行  
                    long currentTime, executionTime;  
                    task = queue.getMin();//取第一个任务  
                    synchronized(task.lock) {  
                        if (task.state == TimerTask.CANCELLED) {  
                            queue.removeMin();  
                            continue;  // 任务已取消则继续取任务  
                        }  
                        currentTime = System.currentTimeMillis();//当前时间  
                        executionTime = task.nextExecutionTime;//任务将开始执行的时间  
                        if (taskFired = (executionTime<=currentTime)) {  
                            if (task.period == 0) {   
                                queue.removeMin();  
                                task.state = TimerTask.EXECUTED;//已执行完成  
                            } else { //重复执行（***）  
                                queue.rescheduleMin(  
                                  task.period<0 ? currentTime   - task.period  
                                                : executionTime + task.period);  
                            }  
                        }  
                    }  
                    if (!taskFired) // 还没到时间，则等到执行开始时间  
                        queue.wait(executionTime - currentTime);  
                }  
                if (taskFired)  // 执行  
                    task.run();  
            } catch(InterruptedException e) {  
            }  
        }  
    }  
}  
{% endhighlight %}

可以看到这代码不亏是出自大师手笔啊考虑的非常详细周到。 
从这个代码可以看出， 
1）首先，TimerThread是个线程 

2）TimerThread自己维护了一个任务队列，也就是TaskQueue 

3）对队列的操作是线程安全的 

4）***处代码说明很重要的一点：如果前一个任务在执行完的时间已经超过了当前任务原定的开始时间，将当前任务的开始时间重新设置一个值然后执行。这同时也从侧面说明了，定时器的周期并不能结束正在执行的任务。 
> 这话比较拗口，这样说吧，如果定时器的周期是1分钟，但是任务A执行时间是1分零十秒，当到达1分钟时，本来应该执行任务B，但是这时任务A还未执行完，这时会将任务A执行完，然后在1分钟零十秒的时候重新计算任务B的开始执行时间，设为2分钟时，那么任务B会在2分钟时开始执行。 

在来看看TaskQueue，TaskQueue也是Timer的一个内部类： 
{% highlight java %}
class TaskQueue {  
    private TimerTask[] queue = new TimerTask[128];  
    ...  
}  
{% endhighlight %}

通过源码可以看到TaskQueue其实就是在维护一个TimerTask[]。 
那TimerTask又是什么呢？就是我们要定时的任务。部分源码如下： 
{% highlight java %}
public abstract class TimerTask implements Runnable {  
    final Object lock = new Object();//锁  
  
    public abstract void run();  
  
    public boolean cancel() {  
        synchronized(lock) {  
            boolean result = (state == SCHEDULED);  
            state = CANCELLED;  
            return result;  
        }  
    }  
}  
{% endhighlight %}

我们所要做的第一步，就是实现一个TimerTask的对象。 
然后所要做的就是，将这个对象作为参数传入Timer的schedule方法，请看源码： 
{% highlight java %}
public void schedule(TimerTask task, long delay, long period) {  
        if (delay < 0)  
            throw new IllegalArgumentException("Negative delay.");  
        if (period <= 0)  
            throw new IllegalArgumentException("Non-positive period.");  
        sched(task, System.currentTimeMillis()+delay, -period);  
    }  
  
private void sched(TimerTask task, long time, long period) {  
        if (time < 0)  
            throw new IllegalArgumentException("Illegal execution time.");  
  
        synchronized(queue) {  
            if (!thread.newTasksMayBeScheduled)  
                throw new IllegalStateException("Timer already cancelled.");  
  
            synchronized(task.lock) {  
                if (task.state != TimerTask.VIRGIN)  
                    throw new IllegalStateException(  
                        "Task already scheduled or cancelled");  
                task.nextExecutionTime = time;//设置该任务的开始时间  
                task.period = period;//设置该任务的周期  
                task.state = TimerTask.SCHEDULED;//将该任务的状态设为SCHEDULED  
            }  
  
            queue.add(task);//将该任务加入到任务队列中  
            if (queue.getMin() == task)  
                queue.notify();//当列队的第一个任务是该任务时，唤醒  
        }  
    }  
{% endhighlight %}
再看看我们是如何使用Timer帮助我们实现定时调度的，我们可以发现Timer封装的很好，作为使用者可以不用关注TimerThread是如果对TaskQueue中的一个个TimerTask进行调度的。我们只需要创建一个定时任务然后交给TImer管理即可。但是了解了Timer的内部实现之后，使用起来就更加得心应手了。

### Quartz

下面再来说说Quartz，Quartz提供了比Timer更加强大的时序调度功能。 
关于Quartz，我不想说太多，原因在于：[Quartz官方](http://www.quartz-scheduler.org/)提供的[15个例子](http://dl.iteye.com/topics/download/de6149ff-41f6-3c24-9abb-02526ba37c12)太经典，这15个例子看看自己跑跑试试，学起来又快又轻松。

为了再降低一下看代码的门槛，这里提供一些关键的概念性的描述。

#### Job

接口，只有一个方法void execute(JobExecutionContext context)，开发者实现该接口定义Job所运行的任务，JobExecutionContext类提供了调度上下文的各种信息。Job运行时的信息保存在JobDataMap实例中。

#### JobDetail

Quartz在每次执行Job时，都重新创建一个Job实例，所以它不直接接受一个Job的实例，相反它接收一个Job实现类，以便运行时通过newInstance()的反射机制实例化Job。因此需要通过一个类来描述Job的实现类及其它相关的静态信息，如Job名字、描述、关联监听器等信息，JobDetail承担了这一角色。通过该类的构造函数可以更具体地了解它的功用：JobDetail(java.lang.String name, java.lang.String group, java.lang.Class jobClass)，该构造函数要求指定Job的实现类，以及任务在Scheduler中的组名和Job名.

#### Trigger

是一个类，描述触发Job执行的时间触发规则。主要有SimpleTrigger和CronTrigger这两个子类。当仅需触发一次或者以固定时间间隔周期执行，SimpleTrigger是最适合的选择；而CronTrigger则可以通过Cron表达式定义出各种复杂时间规则的调度方案：如每早晨9:00执行，周一、周三、周五下午5:00执行等

#### Calendar

org.quartz.Calendar和java.util.Calendar不同，它是一些日历特定时间点的集合。一个Trigger可以和多个Calendar关联，以便排除或包含某些时间点

#### Scheduler

Trigger和JobDetail可以注册到Scheduler中，两者在Scheduler中拥有各自的组及名称，组及名称是Scheduler查找定位容器中某一对象的依据。Scheduler可以将Trigger绑定到某一JobDetail中，这样当Trigger触发时，对应的Job就被执行。一个Job可以对应多个Trigger，但一个Trigger只能对应一个Job。可以通过SchedulerFactory创建一个Scheduler实例。Scheduler拥有一个SchedulerContext，它类似于ServletContext，保存着Scheduler上下文信息，Job和Trigger都可以访问SchedulerContext内的信息。

#### ThreadPool

Scheduler使用一个线程池作为任务运行的基础设施，任务通过共享线程池中的线程提高运行效率。

      

-EOF-
