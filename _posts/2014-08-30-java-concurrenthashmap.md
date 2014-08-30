---
layout: post
title: 以ConcurrentHashMap为例小议并发集合类
category: Java
tags: [Java,多线程,并发,ConcurrentHashMap,Hashtable,synchronizedMap]
---

迁移自[以ConcurrentHashMap为例小议并发集合类](http://hellosure.iteye.com/blog/1143942)

### 题纲：

+ Hashtable
+ Collections.synchronizedMap
+ ConcurrentHashMap

三者都是线程安全的，其中Hashtable和 Collections.synchronizedMap是同步的，由于使用map范围的锁因此可伸缩性较差。ConcurrentHashMap则利用一系列精妙的设计提供了很好的并发性。 

### Hashtable：

在Java类库中出现的第一个关联的集合类是 Hashtable ，它是JDK 1.0的一部分。 Hashtable 提供了一种易于使用的、线程安全的、关联的map功能。 
为什么线程安全？因为Hashtable 的所有读写方法都是同步的（各方法的含义从方法名就能得知，因此不作累述）：

{% highlight java %}
public synchronized int size(){...}  
public synchronized boolean isEmpty(){...}  
public synchronized Enumeration<K> keys(){...}  
public synchronized Enumeration<V> elements(){...}  
public synchronized boolean contains(Object value){...}  
public synchronized boolean containsKey(Object key){...}  
public synchronized boolean contains(Object value){...}  
public synchronized V get(Object key){...}  
public synchronized V put(K key, V value){...}  
public synchronized V remove(Object key){...}  
public synchronized void putAll(Map<? extends K, ? extends V> t){...}  
public synchronized void clear(){...}  
public synchronized Object clone(){...}  
public synchronized String toString(){...}  
{% endhighlight %}

为什么说Hashtable实现了map的功能呢？其实从上面的方法中也可窥一二，事实上，Hashtable维护了一个数组 
{% highlight java %}
private transient Entry[] table;  
{% endhighlight %}

而从这个Entry的实现可以看出：其实这个Entry[]是一个单项列表，其中的每个元素是键值对 
{% highlight java %}
private static class Entry<K,V> implements Map.Entry<K,V> {  
    int hash;  
    K key;  
    V value;  
    Entry<K,V> next;  
        ...  
}  
{% endhighlight %}
然而，它的同步的实现是以很弱的并发性为代价的。这个后面会提到。 

### Collections.synchronizedMap 

Collections.synchronizedMap是一个同步的包装器，它和Hashtable一样也是线程安全的。 
SynchronizedMap有一个统一的对象锁，所有的方法都对这个锁同步： 

{% highlight java %}
private static class SynchronizedMap<K,V>  
    implements Map<K,V>, Serializable {  
        private final Map<K,V> m;   
        final Object  mutex;//锁  
        public int size() {  
        synchronized(mutex) {return m.size();}  
        }  
  
        public V get(Object key) {  
        synchronized(mutex) {return m.get(key);}  
        }  
  
    public V put(K key, V value) {  
        synchronized(mutex) {return m.put(key, value);}  
        }  
        ......  
}  
{% endhighlight %}

注意：这种线程安全并不是无条件的，**虽然所有单个的操作都是线程安全的，但是多个操作组成的操作序列却可能导致数据争用**，因为在操作序列中控制流取决于前面操作的结果。比如put-if-absent（空则放入）： 
{% highlight java %}
Map m = Collections.synchronizedMap(new HashMap());  
if (!map.containsKey(key))  
      map.put(key, value);  
{% endhighlight %}

如果一个条目不在 Map 中，那么添加这个条目。不幸的是，在 containsKey() 方法返回到 put() 方法被调用这段时间内，可能会有另一个线程也插入一个带有相同键的值。如果您想确保只有一次插入，您需要用一个对 Map m 进行同步的同步块将这一对语句包装起来。 
这种情况就会造成一种信任的错觉： 
开发者会这样想，“因为这些集合都是同步的，所以它们都是线程安全的”，这样一来他们对于正确地同步混合操作这件事就会疏忽。其结果是尽管表面上这些程序在负载较轻的时候能够正常工作，但是一旦负载较重，它们就会开始抛出 NullPointerException 或 ConcurrentModificationException 。 

另一方面，**同步的集合包装器Collections.synchronizedMap以及早期的 Hashtable类带来的更大的问题是，它们在单个的锁上进行同步。这意味着一次只有一个线程可以访问集合，如果有一个线程正在读一个 Map ，那么所有其他想要读或者写这个 Map 的线程就必须等待。** 

### ConcurrentHashMap 

由上面的分析可以知道，同步的集合包装器Collections.synchronizedMap以及早期的 Hashtable类有两个问题：**一是有条件的线程安全性，二是可伸缩性问题。**其实简单来说，就是它们的并发性很弱。这种可伸缩性的主要障碍是它使用了一个 map 范围的锁，为了保证插入、删除或者检索操作的完整性必须保持这样一个锁，而且有时候甚至还要为了保证迭代遍历操作的完整性保持这样一个锁。这样一来，只要锁被保持，就阻止了其他线程访问 Map，即使处理器有空闲也不能访问，这样大大地限制了并发性。 
为了解决高并发的问题，JDK 1.5引入了java.util.concurrent.ConcurrentHashMap，它也是对 Map 的线程安全的实现，比起 synchronizedMap 来，它提供了好得多的并发性。 

ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术。它使用了多个锁来控制对hash表的不同部分进行的修改。ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的hash table，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。 
{% highlight java %}
//每个Segment其实就是一个小的hash表    
final Segment<K,V>[] segments;    
{% endhighlight %}

有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁。这里“按顺序”是很重要的，否则极有可能出现死锁。 

ConcurrentHashMap完全允许多个读操作并发进行，读操作并不需要加锁。（`事实上，ConcurrentHashMap支持完全并发的读以及一定程度并发的写。`）如果使用传统的技术，如HashMap中的实现，如果允许可以在hash链的中间添加或删除元素，读操作不加锁将得到不一致的数据。但是ConcurrentHashMap实现技术是保证HashEntry几乎是不可变的。HashEntry代表每个hash链中的一个节点，其结构如下所示： 
{% highlight java %}
static final class HashEntry<K,V> {    
    final K key;    
    final int hash;    
    volatile V value;    
    final HashEntry<K,V> next;    
}    
{% endhighlight %}

可以看到除了value不是final的，其它值都是final的，这意味着不能从hash链的中间或尾部添加或删除节点，因为这需要修改next引用值，所有的节点的修改只能从头部开始。对于put操作，可以一律添加到Hash链的头部。但是对于remove操作，可能需要从中间删除一个节点，这就需要将要删除节点的前面所有节点整个复制一遍，最后一个节点指向要删除结点的下一个结点。这在讲解删除操作时还会详述。为了确保读操作能够看到最新的值，将value设置成volatile，这避免了加锁。 
HashEntry 类的 value 域被声明为 Volatile 型，Java 的内存模型可以保证：某个写线程对 value 域的写入马上可以被后续的某个读线程“看”到。在 ConcurrentHashMap 中，不允许用 null作为键和值，当读线程读到某个 HashEntry 的 value 域的值为 null 时，便知道产生了冲突——发生了重排序现象，需要加锁后重新读入这个 value 值。这些特性互相配合，使得读线程即使在不加锁状态下，也能正确访问 ConcurrentHashMap。 

关于Hash表的基础数据结构，这里不想做过多的探讨。Hash表的一个很重要方面就是如何解决hash冲突，ConcurrentHashMap和HashMap使用相同的方式，都是将hash值相同的节点放在一个hash链中。与HashMap不同的是，ConcurrentHashMap使用多个子Hash表，也就是段(Segment)。 

#### Segment

每个Segment相当于一个子Hash表，它的数据成员如下： 
{% highlight java %}
static final class Segment<K,V> extends ReentrantLock implements Serializable {    
        transient volatile int count;    
        transient int modCount;    
        transient int threshold;    
        transient volatile HashEntry<K,V>[] table;   
        final float loadFactor;    
}    
{% endhighlight %}

Segment 类继承于 ReentrantLock 类，从而使得 Segment 对象能充当锁的角色。每个 Segment 对象用来守护其（成员对象 table 中）包含的若干个桶。 
table 是一个由 HashEntry 对象组成的数组。table 数组的每一个数组成员就是散列映射表的一个桶。如果散列时发生碰撞，碰撞的 HashEntry 对象就以链表的形式链接成一个链表。 
count用来统计该段数据的个数，它是volatile，它用来协调修改和读取操作，以保证读取操作能够读取到几乎最新的修改。协调方式是这样的，每次修改操作做了结构上的改变，如增加/删除节点(修改节点的值不算结构上的改变)，都要写count值，每次读取操作开始都要读取count的值。modCount统计段结构改变的次数，主要是为了检测对多个段进行遍历过程中某个段是否发生改变，在讲述跨段操作时会还会详述。threashold用来表示需要进行rehash的界限值。table数组存储段中节点，每个数组元素是个hash链，用HashEntry表示。table也是volatile，这使得能够读取到最新的table值而不需要同步。loadFactor表示负载因子。 

关于ConcurrentHashMap各种操作（remove，put，get，containsKey，以及跨段操作size、containsValaue和isEmpty等）在帖子[ConcurrentHashMap之实现细节](http://www.iteye.com/topic/344876)中有精彩的描述。 
另外推荐一个帖子[探索 ConcurrentHashMap 高并发性的实现机制](http://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/index.html)对ConcurrentHashMap的实现也有很精彩的描述。 

### 知识点小结 

这个文章总结起来感觉蛮乱的，现在总结一下自己的一点小收获： 

同步和并发：在某种程度来说，两者不可兼得的。同步要求原子性，所以需要一个全局的锁来保证这种原子性。比如用一个map范围的锁来使这整个map成为一个原子。使用一个全局的锁来同步不同线程间的并发访问。同一时间点，只能有一个线程持有锁，也就是说在同一时间点，只能有一个线程能访问容器。这虽然保证多线程间的安全并发访问，但同时也导致对容器的访问变成串行化的了。（串行和并行当然是相悖的两个概念）但是这样一来就不可避免的造成了可伸缩性差，就是说在并发线程数增加时争用的情况非常激烈，这就导致了并发性较差。对于并发性高的环境就需要顾及到并发性的实现，那么就要在某种程度上打破这种原子性，即进行锁分离，使用多个锁来分段维护map。

同步也不是无条件的线程安全：虽然所有单个的操作都是线程安全的，但是多个操作组成的操作序列却可能导致数据争用。

在实际的应用中，散列表一般的应用场景是：除了少数插入操作和删除操作外，绝大多数都是读取操作，而且读操作在大多数时候都是成功的。正是基于这个前提，ConcurrentHashMap 针对读操作做了大量的优化。通过 HashEntry 对象的不变性和用 volatile 型变量协调线程间的内存可见性，使得ConcurrentHashMap支持完全并发的读以及一定程度并发的写

ConcurrentHashMap 的高并发性主要来自于三个方面：
1) 用分离锁实现多个线程间的更深层次的共享访问。使用分离锁，减小了请求同一个锁的频率。 
2) 用 HashEntery 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。 
3) 通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。 
    
-EOF-
