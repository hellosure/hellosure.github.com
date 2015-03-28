---
layout: post
title: 生产者/消费者模式
category: Java
tags: [Java,多线程,同步]
---

### 简介

        生产者/消费者模式其实是一种很经典的线程同步模型，很多时候，并不是光保证多个线程对某共享资源操作的互斥性就够了，往往多个线程之间都是有协作的。

        假设有这样一种情况，有一个桌子，桌子上面有一个盘子，盘子里只能放一颗鸡蛋，A专门往盘子里放鸡蛋，如果盘子里有鸡蛋，则一直等到盘子里没鸡蛋，B专门 从盘子里拿鸡蛋，如果盘子里没鸡蛋，则等待直到盘子里有鸡蛋。其实盘子就是一个互斥区，每次往盘子放鸡蛋应该都是互斥的，A的等待其实就是主动放弃锁，B 等待时还要提醒A放鸡蛋。

#### 如何让线程主动释放锁

很简单，调用锁的wait()方法就好。wait方法是从Object来的，所以任意对象都有这个方法。看这个代码片段：

{% highlight java %}

Object lock=new Object();//声明了一个对象作为锁
   synchronized (lock) {
       balance = balance - num;
       //这里放弃了同步锁，好不容易得到，又放弃了
       lock.wait();
}

{% endhighlight %}

如果一个线程获得了锁lock，进入了同步块，执行lock.wait()，那么这个线程会进入到lock的阻塞队列。如果调用 lock.notify()则会通知阻塞队列的某个线程进入就绪队列。

#### 声明一个盘子，只能放一个鸡蛋

{% highlight java %}

import java.util.ArrayList;
import java.util.List;
 
public class Plate {
 
    List<Object> eggs = new ArrayList<Object>();
 
    public synchronized Object getEgg() {
        while(eggs.size() == 0) {
            try {
                wait();
            } catch (InterruptedException e) {
            }
        }
 
        Object egg = eggs.get(0);
        eggs.clear();// 清空盘子
        notify();// 唤醒阻塞队列的某线程到就绪队列
        System.out.println("拿到鸡蛋");
        return egg;
    }
 
    public synchronized void putEgg(Object egg) {
        while(eggs.size() > 0) {
            try {
                wait();
            } catch (InterruptedException e) {
            }
        }
        eggs.add(egg);// 往盘子里放鸡蛋
        notify();// 唤醒阻塞队列的某线程到就绪队列
        System.out.println("放入鸡蛋");
    }
     
    static class AddThread extends Thread{
        private Plate plate;
        private Object egg=new Object();
        public AddThread(Plate plate){
            this.plate=plate;
        }
         
        public void run(){
            for(int i=0;i<5;i++){
                plate.putEgg(egg);
            }
        }
    }
     
    static class GetThread extends Thread{
        private Plate plate;
        public GetThread(Plate plate){
            this.plate=plate;
        }
         
        public void run(){
            for(int i=0;i<5;i++){
                plate.getEgg();
            }
        }
    }
     
    public static void main(String args[]){
        try {
            Plate plate=new Plate();
            Thread add=new Thread(new AddThread(plate));
            Thread get=new Thread(new GetThread(plate));
            add.start();
            get.start();
            add.join();
            get.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("测试结束");
    }
}

{% endhighlight %}

  执行结果：
放入鸡蛋
拿到鸡蛋
放入鸡蛋
拿到鸡蛋
放入鸡蛋
拿到鸡蛋
放入鸡蛋
拿到鸡蛋
放入鸡蛋
拿到鸡蛋
测试结束

声明一个Plate对象为plate，被线程A和线程B共享，A专门放鸡蛋，B专门拿鸡蛋。假设

1 开始，A调用plate.putEgg方法，此时eggs.size()为0，因此顺利将鸡蛋放到盘子，还执行了notify()方法，唤醒锁的阻塞队列 的线程，此时阻塞队列还没有线程。
2 又有一个A线程对象调用plate.putEgg方法，此时eggs.size()不为0，调用wait()方法，自己进入了锁对象的阻塞队列。
3 此时，来了一个B线程对象，调用plate.getEgg方法，eggs.size()不为0，顺利的拿到了一个鸡蛋，还执行了notify()方法，唤 醒锁的阻塞队列的线程，此时阻塞队列有一个A线程对象，唤醒后，它进入到就绪队列，就绪队列也就它一个，因此马上得到锁，开始往盘子里放鸡蛋，此时盘子是 空的，因此放鸡蛋成功。
4 假设接着来了线程A，就重复2；假设来料线程B，就重复3。 

整个过程都保证了放鸡蛋，拿鸡蛋，放鸡蛋，拿鸡蛋。

-EOF-
