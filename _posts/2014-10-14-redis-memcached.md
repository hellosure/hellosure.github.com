---
layout: post
title: 也来比较一下Redis与Memcached
category: Cache
tags: [Redis,Memcached]
---

### 传统MySQL+ Memcached架构遇到的问题
　　
实际MySQL是适合进行海量数据存储的，通过Memcached将热点数据加载到cache，加速访问，很多公司都曾经使用过这样的架构，但随着业务数据量的不断增加，和访问量的持续增长，我们遇到了很多问题：

1. MySQL需要不断进行拆库拆表，Memcached也需不断跟着扩容，扩容和维护工作占据大量开发时间。

2. Memcached与MySQL数据库数据一致性问题。

3. Memcached数据命中率低或down机，大量访问直接穿透到DB，MySQL无法支撑。

4. 跨机房cache同步问题。
　　
### redis和memcached区别

1. Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，zset，hash等数据结构的存储。

2. Redis支持数据的备份，即master-slave模式的数据备份。

3. Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。

在Redis中，并不是所有的数据都一直存储在内存中的。这是和Memcached相比一个最大的区别。

Redis只会缓存所有的key的信息，如果Redis发现内存的使用量超过了某一个阀值，将触发swap的操作，Redis根据“swappability = age*log(size_in_memory)”计算出哪些key对应的value需要swap到磁盘。

然后再将这些key对应的value持久化到磁盘中，同时在内存中清除。这种特性使得Redis可以保持超过其机器本身内存大小的数据。

当然，机器本身的内存必须要能够保持所有的key，毕竟这些数据是不会进行swap操作的。

同时由于Redis将内存中的数据swap到磁盘中的时候，提供服务的主线程和进行swap操作的子线程会共享这部分内存，所以如果更新需要swap的数据，Redis将阻塞这个操作，直到子线程完成swap操作后才可以进行修改。

使用Redis特有内存模型前后的情况对比：

    VM off: 300k keys, 4096 bytes values: 1.3G used
    VM on:  300k keys, 4096 bytes values: 73M used      //swap到磁盘中了
    VM off: 1 million keys, 256 bytes values: 430.12M used
    VM on:  1 million keys, 256 bytes values: 160.09M used
    VM on:  1 million keys, values as large as you want, still: 160.09M used  //说明value都在磁盘中

当从Redis中读取数据的时候，如果读取的key对应的value不在内存中，那么Redis就需要从swap文件中加载相应数据，然后再返回给请求方。 

这里就存在一个I/O线程池的问题。在默认的情况下，Redis会出现阻塞，即完成所有的swap文件加载后才会响应。

这种策略在客户端的数量较小，进行批量操作的时候比较合适。但是如果将Redis应用在一个大型的网站应用程序中，这显然是无法满足大并发的情况的。所以Redis运行我们设置I/O线程池的大小，对需要从swap文件中加载相应数据的读取请求进行并发操作，减少阻塞的时间。

### redis、memcache、mongoDB对比

1、性能

都比较高，性能对我们来说应该都不是瓶颈。

总体来讲，TPS方面redis和memcache差不多，要大于mongodb

2、操作的便利性

memcache数据结构单一。

redis丰富一些，数据操作方面，redis更好一些，较少的网络IO次数。

mongodb支持丰富的数据表达，索引，最类似关系型数据库，支持的查询语言非常丰富。

3、内存空间的大小和数据量的大小

redis在2.0版本后增加了自己的VM特性，突破物理内存的限制；可以对key value设置过期时间（类似memcache）

memcache可以修改最大可用内存,采用LRU算法

mongoDB适合大数据量的存储，依赖操作系统VM做内存管理，吃内存也比较厉害，服务不要和别的服务在一起

4、可用性（单点问题）

redis，依赖客户端来实现分布式读写；主从复制时，每次从节点重新连接主节点都要依赖整个快照,无增量复制，因性能和效率问题，
所以单点问题比较复杂；不支持自动sharding,需要依赖程序设定一致hash 机制。
一种替代方案是，不用redis本身的复制机制，采用自己做主动复制（多份存储），或者改成增量复制的方式（需要自己实现），一致性问题和性能的权衡

Memcache本身没有数据冗余机制，也没必要；对于故障预防，采用依赖成熟的hash或者环状的算法，解决单点故障引起的抖动问题。

mongoDB支持master-slave,replicaset（内部采用paxos选举算法，自动故障恢复）,auto sharding机制，对客户端屏蔽了故障转移和切分机制。

5、可靠性（持久化）

对于数据持久化和数据恢复：

redis支持（快照、AOF）：依赖快照进行持久化，aof增强了可靠性的同时，对性能有所影响

memcache不支持，通常用在做缓存,提升性能；

MongoDB从1.8版本开始采用binlog方式支持持久化的可靠性

6、数据一致性（事务支持）

Memcache 在并发场景下，用cas保证一致性

redis事务支持比较弱，只能保证事务中的每个操作连续执行

mongoDB不支持事务

7、数据分析

mongoDB内置了数据分析的功能(map-reduce),其他不支持

8、应用场景

redis：数据量较小的更性能操作和运算上

memcache：用于在动态系统中减少数据库负载，提升性能;做缓存，提高性能（适合读多写少，对于数据量比较大，可以采用sharding）

MongoDB:主要解决海量数据的访问效率问题

### Redis作者的观点

下面内容来自Redis作者在stackoverflow上的一个回答，对应的问题是《Is memcached a dinosaur in comparison to Redis?》（相比Redis，Memcached真的过时了吗？）

#### 性能

没有必要过多的关心性能，因为二者的性能都已经足够高了。由于Redis只使用单核，而Memcached可以使用多核，所以在比较上，平均每一个核上Redis在存储小数据时比Memcached性能更高。而在100k以上的数据中，Memcached性能要高于Redis，虽然Redis最近也在存储大数据的性能上进行优化，但是比起Memcached，还是稍有逊色。说了这么多，结论是，无论你使用哪一个，每秒处理请求的次数都不会成为瓶颈。（比如瓶颈可能会在网卡）

> 对于性能，Redis作者的说法是平均到单个核上的性能，在单条数据不大的情况下Redis更好。为什么这么说呢，理由就是Redis是单线程运行的。
因为是单线程运行，所以和Memcached的多线程相比，整体性能肯定会偏低。
因为是单线程运行，所以IO是串行化的，网络IO和内存IO，因此当单条数据太大时，由于需要等待一个命令的所有IO完成才能进行后续的命令，所以性能会受影响。

#### 内存使用效率

如果要说内存使用效率，使用简单的key-value存储的话，Memcached的内存利用率更高，而如果Redis采用hash结构来做key-value存储，由于其组合式的压缩，其内存利用率会高于Memcached。当然，这和你的应用场景和数据特性有关。

> 就内存使用上来说，目前Redis结合了tcmalloc和jemalloc两个内存分配器，基本上和Memcached不相伯仲。如果是简单且有规律的key value存储，那么用Redis的hash结构来做，内存使用上会惊人的变小，优势是很明显的。

#### 选择Redis的理由

如果你对数据持久化和数据同步有所要求，那么推荐你选择Redis，因为这两个特性Memcached都不具备。即使你只是希望在升级或者重启系统后缓存数据不会丢失，选择Redis也是明智的。

当然，最后还得说到你的具体应用需求。Redis相比Memcached来说，拥有更多的数据结构和并支持更丰富的数据操作，通常在Memcached里，你需要将数据拿到客户端来进行类似的修改再set回去。这大大增加了网络IO的次数和数据体积。在Redis中，这些复杂的操作通常和一般的GET/SET一样高效。所以，如果你需要缓存能够支持更复杂的结构和操作，那么Redis会是不错的选。

### 知乎上一些观点

相比memcached：

1、redis具有持久化机制，可以定期将内存中的数据持久化到硬盘上。

2、redis具备binlog功能，可以将所有操作写入日志，当redis出现故障，可依照binlog进行数据恢复。

3、redis支持virtual memory，可以限定内存使用大小，当数据超过阈值，则通过类似LRU的算法把内存中的最不常用数据保存到硬盘的页面文件中。

4、redis原生支持的数据类型更多，使用的想象空间更大。

5、前面有位朋友所提及的一致性哈希，用在redis的sharding中，一般是在负载非常高需要水平扩展时使用。我们还没有用到这方面的功能，一般的项目，单机足够支撑并发了。redis 3.0将推出cluster，功能更加强大。

### robbin说iteye和CSDN的使用

ITeye(JavaEye)和CSDN现在都用到了Redis，采用memcached比较出色的网站有ITeye(JavaEye)。

我们用memcached非常好，唯一不爽的地方是万一服务器掉电重启以后的缓存预热过程比较长，导致网站负载过高。Redis的持久化存储可以解决这个问题。

### Instagram的Redis实践：节约内存

场景：Instagram的照片数量已经达到3亿，而在Instagram里，我们需要知道每一张照片的作者是谁，下面就是Instagram团队如何使用Redis来解决这个问题并进行内存优化的。

需求：这个通过图片ID反查用户UID的应用有以下几点需求：

1. 查询速度要足够快

2. 数据要能全部放到内存里，最好是一台EC2的 high-memory 机型就能存储（17GB或者34GB的，68GB的太浪费了）

3. 支持持久化，这样在服务器重启后不需要再预热

技术选型：

Instagram的开发者首先否定了数据库存储的方案，他们保持了KISS原则（Keep It Simple and Stupid），因为这个应用根本用不到数据库的update功能，事务功能和关联查询等等牛X功能，所以不必为这些用不到的功能去选择维护一个数据库。

于是他们选择了Redis，Redis是一个支持持久化的内存数据库，所有的数据都被存储在内存中（忘掉VM吧），而最简单的实现就是使用Redis的String结构来做一个key-value存储就行了。像这样：

    SET media:1155315 939
    GET media:1155315
    > 939

其中1155315是图片ID，939是用户ID，我们将每一张图片ID为作key，用户uid作为value来存成key-value对。然后他们进行了测试，将数据按上面的方法存储，1,000,000数据会用掉70MB内存，300,000,000张照片就会用掉21GB的内存。对比预算的17GB还是超支了。

于是Instagram的开发者向Redis的开发者之一Pieter Noordhuis询问优化方案，得到的回复是使用Hash结构。具体的做法就是将数据分段，每一段使用一个Hash结构存储，由于Hash结构会在单个Hash元素在不足一定数量时进行压缩存储，所以可以大量节约内存。这一点在上面的String结构里是不存在的。而这个一定数量是由配置文件中的hash-zipmap-max-entries参数来控制的。经过开发者们的实验，将hash-zipmap-max-entries设置为1000时，性能比较好，超过1000后HSET命令就会导致CPU消耗变得非常大。

于是他们改变了方案，将数据存成如下结构：

    HSET "mediabucket:1155" "1155315" "939"
    HGET "mediabucket:1155" "1155315"
    > "939"

通过取7位的图片ID的前四位为Hash结构的key值，保证了每个Hash内部只包含3位的key，也就是1000个。
再做一次实验，结果是每1,000,000个key只消耗了16MB的内存。总内存使用也降到了5GB，满足了应用需求。

-EOF-
