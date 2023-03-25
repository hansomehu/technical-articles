layout: post
title: "Redis面试核心"
permalink: /redis-interview



![img](https://pdai.tech/images/db/redis/db-redis-overview.png)





Redis基本数据结构，一些特性，一些系统运用原理和实际业务中常见问题的解决



#### 本地缓存（补充知识点）

#### Caffeine

Caffeine是基于java8实现的新一代缓存工具，缓存性能接近理论最优。可以看作是Guava Cache的增强版，功能上两者类似，不同的是Caffeine采用了一种结合LRU、LFU优点的算法：W-TinyLFU，在性能上有明显的优越性。Caffeine的使用，首先需要引入maven包：

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>2.5.5</version>
</dependency>
复制代码
```

使用上和Guava Cache基本类似：

```arduino
public class CaffeineCacheTest {

    public static void main(String[] args) throws Exception {
        //创建guava cache
        Cache<String, String> loadingCache = Caffeine.newBuilder()
                //cache的初始容量
                .initialCapacity(5)
                //cache最大缓存数
                .maximumSize(10)
                //设置写缓存后n秒钟过期
                .expireAfterWrite(17, TimeUnit.SECONDS)
                //设置读写缓存后n秒钟过期,实际很少用到,类似于expireAfterWrite
                //.expireAfterAccess(17, TimeUnit.SECONDS)
                .build();
        String key = "key";
        // 往缓存写数据
        loadingCache.put(key, "v");

        // 获取value的值，如果key不存在，获取value后再返回
        String value = loadingCache.get(key, CaffeineCacheTest::getValueFromDB);

        // 删除key
        loadingCache.invalidate(key);
    }

    private static String getValueFromDB(String key) {
        return "v";
    }
}
复制代码
```

相比Guava Cache来说，Caffeine无论从功能上和性能上都有明显优势。同时两者的API类似，使用Guava Cache的代码很容易可以切换到Caffeine，节省迁移成本。需要注意的是，SpringFramework5.0（SpringBoot2.0）同样放弃了Guava Cache的本地缓存方案，转而使用Caffeine。





#### 基本数据类型

| 数据类型 |      可以存储的值      |                             操作                             |
| :------: | :--------------------: | :----------------------------------------------------------: |
|  STRING  | 字符串、整数或者浮点数 | 对整个字符串或者字符串的其中一部分执行操作对整数和浮点数执行自增或者自减操作get  del  set |
|   LIST   |          列表          | 从两端压入或者弹出元素 </br> 对单个或者多个元素进行修剪，<br />只保留一个范围内的元素<br />r/l push pop |
|   SET    |        无序集合        | 添加、获取、移除单个元素</br> 检查一个元素是否存在于集合中</br> 计算交集、并集、差集</br> 从集合里面随机获取元素<br />sadd srem |
|   HASH   | 包含键值对的无序散列表 | 添加、获取、移除单个键值对</br> 获取所有键值对</br> 检查某个键是否存在<br />hset hget field 1 "val1" ... hdel hgetall |
|   ZSET   |        有序集合        | 添加、获取、删除元素</br> 根据分值范围或者成员来获取元素</br> 计算一个键的排名<br />zadd zrem |



##### bitmap

底层是SDS转为二进制编码方式

基本的使用是SETBIT/GETBIT key offset value

用来打卡、登陆统计

最大的优点就是面对亿级的数据量所耗费的空间也就是MB级别



##### hyperloglog

自增的有状态数据



##### GEO

底层基于ZSET来实现，原理是通过把longitude、latitude等信息编码为整数，然后根据输入的经纬度信息范围最相近的一系列数据即可

```shell
# 动态统计车辆位置
GEOADD cars:locations 116.034579 39.030452 33
```

```shell
# 为乘客匹配附近车辆
GEORADIUS cars:locations 116.054579 39.030452 5 km ASC COUNT 10
```



#### Redis对象机制 RedisObject

Redis使用自己实现的对象机制（RedisObject）来实现类型判断、命令多态和基于引用次数的垃圾回收。拿最常见的命令操作来举例子，集合和哈希对应不同的底层存储，redis通过这些命令来匹配到目标数据结构。同时Redis也会<u>预分配</u>一些常用的数据对象，并通过共享这些对象来减少内存·占用，和避免频繁的为小对象分配内存。[图片来源](https://pdai.tech/md/db/nosql-redis/db-redis-x-redis-object.html)



在Redis的体系中，属于RedisObject的是键（Key），在每个键中都会包含该键值对存储的编码类型和底层数据结构。比如集合有多种编码类型，但是用户统一通过ZADD向其中添加元素，那么Redis会通过ZADD传入的键为它选择合适的处理方式。

```c
/*
 * Redis 对象
 */
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码方式
    unsigned encoding:4;

    // LRU - 24位, 记录最末一次访问时间（相对于lru_clock）; 或者 LFU（最少使用的数据：8位频率，16位访问时间）
    unsigned lru:LRU_BITS; // LRU_BITS: 24

    // 引用计数，垃圾回收时使用
    int refcount;

    // 指向底层数据结构实例
    void *ptr;

} robj;

```



#### 数据结构

Redis的五种基本数据类型（string、list、hash、set、zset.....）它们所对应的底层存储结构是不一样的，一种数据类型的底层可以由多种数据结构来实现，需要全部了解

String：**SDS**

List：**QuickList**

Hash：**ZipList (listpack)**、HashTable

Set：HashTable、IntSet

ZSet：ZipList (listpack)、**ZSkipList**



##### SDS

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220801000000851.png" alt="image-20220801000000851" style="zoom: 33%;" />

存在四种类型的头部

- `len` 保存了SDS保存字符串的长度
- `buf[]` 数组用来保存字符串的每个元素
- `alloc`分别以uint8, uint16, uint32, uint64表示整个SDS, 除过头部与末尾的\0, 剩余的字节数.
- `flags` 始终为一字节, 以低三位标示着头部的类型, 高5位未使用.

为什么使用SDS

- 常数复杂度获取字符串长；通过 `strlen key` 命令可以获取 key 的字符串长度
- 杜绝缓冲区溢出；在进行字符修改的时候，**会首先根据记录的 len 属性检查内存空间是否满足需求**，如果不满足，会进行相应的空间扩展，然后在进行修改操作，所以不会出现缓冲区溢出。而C语言中的字符串则没有这么多的校验
- 减少修改字符串的内存重新分配次数；通过空间欲分配和空间惰性释放来实现



##### ZipList 压缩列表

ziplist是为了**提高存储效率**而设计的一种特殊编码的**list**。它可以存储字符串或者整数，存储整数时是采用整数的二进制而不是字符串形式存储。它能在O(1)的时间复杂度下完成list两端的push和pop操作。但是因为每次操作都需要重新分配ziplist的内存，所以实际复杂度和ziplist的内存使用量相关

ZL最大的特点是**省内存：**

1. 普通列表的内存分配是根据所有元素中占内存最大的那个来进行分配的，**而ZL每个元素是不一样的**，根据其Encoding做到了精细化地分配
2. 这时候还需要解决的一个问题是遍历元素时如何定位下一个元素呢？在普通数组中每个元素定长，所以不需要考虑这个问题；但是ziplist中每个data占据的内存不一样，所以为了解决遍历，需要增加记录上一个元素的length，**所以增加了prelen字段**。

ZL的缺点是上述的设计会涉及到频繁扩容操作，有可能会引发连锁扩容，即列表中每个元素都进行一次扩容。根本的原因在于ZL不做空间预留这种浪费空间的操作。

它只适合从后往前去遍历，因为只有prelen而没有本节点的length

![img](https://pdai.tech/images/db/redis/db-redis-ds-x-6.png)



##### QuickList 快表

quicklist有自己的优点，也有缺点，对于使用者来说，其使用体验类似于线性数据结构。list作为最传统的双链表，结点通过指针持有数据，指针字段会耗费大量内存。 ziplist解决了耗费内存这个问题，但引入了新的问题，每次写操作 整个ziplist的内存都需要重分配。quicklist在两者之间做了一个平衡， 并且使用者可以通过自定义`quicklist.fill`，根据实际业务情况，经验主义调参。

它是一种以ziplist为结点的双端链表结构. 宏观上, quicklist是一个链表, 微观上, 链表中的每个结点都是一个ziplist。

双向链表加大了数据的检索效率，而ZipList则加大了内存的使用效率。

如果全部使用链表链接的话空间开销会很大，所以采取了一种类似折衷的方式



![image-20230208010912636](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230208010912636.png)



##### HashTable

Redis中的hashtable没有红黑树的概念，但是在这里顺便提一下，转化红黑树的意义在于减少节点查找的时间。链表的查找时间为O（n），而红黑树平均为O（logn）



哈希表的改良版，特点如下

**解决哈希冲突**：HashTable解决哈希冲突的方法是**链地址法**，通过字典里面的 *next 指针指向下一个具有相同索引值的哈希表节点



**扩容和收缩**：当哈希表保存的键值对太多或者太少时，就要通过 rehash(重新散列）来对哈希表进行相应的扩展或者收缩



**渐进式rehash：**

扩容和收缩操作不是一次性、集中式完成的，而是分多次、渐进式完成的。如果保存在Redis中的键值对只有几个几十个，那么 rehash 操作可以瞬间完成，但是如果键值对有几百万，几千万甚至几亿，那么要一次性的进行 rehash，势必会造成Redis一段时间内不能进行别的操作。所以Redis采用渐进式 rehash，这样在进行渐进式rehash期间，字典的删除查找更新等操作可能会在**两个哈希表**上进行，第一个哈希表没有找到，就会去第二个哈希表上进行查找。但是进行增加操作，一定是在新的哈希表上进行的

关于两个哈希表的问题，也就是dicht这个一级结构中存在两个哈希数组，一个用来保存数据，一个用做rehash的backup。具体的rehash过程就是，先将backup的哈希表扩容，然后将一部分的数据迁移进去

触发的机制是，当负载因子超过1之后，只有在不存在RDB/AOF的时候才rehash，当超过5之后，会立刻rehash

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230314201952451.png" alt="image-20230314201952451" style="zoom:33%;" />







##### ZSkipList 跳表

典型的以空间换时间的数据结构，通过大量的指针来加速元素的查找

ZKL通过多级索引，即多级链表来实现快速的访达

它的双向链表的指针是**一个next和一个backward**

但是，它的同列的元素结构设计使得**downward**查找的时候也拥有了指针

![image-20230301103729426](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230301103729426.png)

ZSET使用skiplist来做，为什么不用btree来做？

There are a few reasons from the author of Redis:

1. They are not very memory intensive. It's up to you basically. Changing parameters about the probability of a node to have a given number of levels will make then less memory intensive than btrees.（skiplist相比之下用的指针更少，节约内存）
2. A sorted set is often target of many ZRANGE or ZREVRANGE operations, that is, traversing the skip list as a linked list. With this operation the cache locality of skip lists is at least as good as with other kind of balanced trees. （范围查找方便）
3. They are simpler to implement, debug, and so forth. For instance thanks to the skip list simplicity I received a patch (already in Redis master) with augmented skip lists implementing ZRANK in O(log(N)). It required little changes to the code. （便于后来者学习和自定义）





##### listpack

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230228190828232.png" alt="image-20230228190828232" style="zoom:50%;" />

Redis7.0之后将ziplist换成了listpack

新的这种结构优化了存储空间和时间开销：

- 每个节点只记录自己的len，取消了prelen字段，避免了**连锁更新**问题
- 头节点上换成了**总字节数**和**总节点数**









#### Stream概念

Stream本身是Redis为了成为强大MQ而做的增强功能，但是真正需要强大MQ的时候用户也并非会选择MQ（面对大流量可靠性不足、使用上和常用MQ有差距等原因）。

Redis5.0 中增加了一个<u>数据类型Stream</u>，它借鉴了Kafka的设计，是一个新的强大的支持多播的可持久化的消息队列。Stream类型从字面上看是流类型，但其实从功能上看，应该是Redis对消息队列（MQ，Message Queue）的完善实现。

Stream设计目的是让Redis作为消息队列来用的时候功能更加强大，因为Redis本身只有SUB/PUB这类发布订阅机制是用来实现MQ的。但是一个强大的MQ还需要很多其他的支持，见下图（来源：美团技术团队博客）。Stream的诞生是为了支持下面的特性。

<img src="../assets/images/redis/image-20220427220932659.png" alt="image-20220427220932659" style="zoom:33%;" />

##### **Stream的数据结构：**

其数据结构是Stream名字的来源，Stream也是KV键值对类型的，Key是一个Stream结构的唯一标识。而Stream的值则是一个很长的数组，里面包含了很多数据项，这些数据项中则含有消息队列应该有的一些信息，例如”消费者组“

<img src="../assets/images/redis/image-20220427221121777.png" alt="image-20220427221121777" style="zoom:33%;" />









#### Redis持久化

持久化就是把内存中的数据放到磁盘中去，因为内存是volatile的，断电即失；并且内存的成本远大于磁盘，但处理速度远大于磁盘，所以需要不断将内存中老的数据拷贝到磁盘中去，以便腾出空间来接受新的数据。

##### **RDB持久化**

> **关键概念：bgsave（fork子线程）、CopyOnWrite**



Redis DataBase，中文名为**快照/内存快照**，RDB持久化是把当前进程数据生成快照保存到磁盘上的过程，由于是某一时刻的快照，那么快照中的值要早于或者等于内存中的值。简单说就是每隔一段时间，我就把内存中的数据拷贝到磁盘里面去，然后**清除**这部分数据。

RDB持久化有两种方式，一个是主线程阻塞执行save方法，或者fork一个子线程出来执行bgsave方法。

RDB执行持久化是通过fork一个**子线程**，让子线程来执行拷贝工作，redis主线程保持执行业务逻辑。持久化任务的<u>触发机制</u>有两个：1. 手动 2.自动

1. 手动方式是调用bgsave命令，或者save命令（该命令由主线程自行拷贝，不推荐）

2. 自动则是通过配置相关系统参数redis.conf中配置`save m n` ， 在m秒内执行了n次操作后即备份（可配置多条规则）。

   另外还有一些非主动要求的自动备份触发情景：shutdown命令、主从复制、debug reload命令会触发bgsave

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230208154236339.png" alt="image-20230208154236339" style="zoom:33%;" />

RDB存在的问题及解决方案：

1. 万年存在的数据一致性问题：RDB中的核心思路是**Copy-on-Write**。在子线程执行拷贝的这段时间，主线程如果对数据进行的修改，那么把修改过的那部分数据放到另一个新的内存区域，待快照操作结束后才会同步到原来的内存区域。

   <img src="https://pdai.tech/images/db/redis/redis-x-aof-42.jpg" alt="img" style="zoom: 50%;" />

2. 备份期间发生了服务崩溃（不是服务器物理崩溃）怎么办：采取一种类事务性的操作，只有全部的数据都完成了备份才能认定为本次快照成功，否则就是失败。如果失败了，则将以上一次完整的RDB快照文件作为恢复内存数据的参考。也就是说，在快照操作过程中不能影响上一次的备份数据。Redis服务会在磁盘上创建一个临时文件进行数据操作，待操作成功后才会用这个临时文件替换掉上一次的备份

3. 每秒做一次快照行不行？：行，但是没必要。快照过程对主线程来说最大的开销就是fork子线程和子线程生命周期管理这个过程。如果频率太高，那么必然会占用主线程用来处理业务逻辑的资源



pros & cons

RDB的复制、压缩方式很紧凑，节约资源；并且恢复速度远大于AOF；

but，RDB缺乏实时性；

并且**全程二进制读取**，人工排错几乎不可能；

fork这个过程对主线程的开销过大



##### **AOF持久化**

> 关键概念：写后日志、aof_buf、aof写入、aof重写

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230208155021846.png" alt="image-20230208155021846" style="zoom: 25%;" />

AOF持久化是一种采用日志记录的方式来实现持久化的方式，类似于MySQL的**statement**机制，通过将**命令语句**持久化进磁盘以达到数据不丢失的效果。

AOF属于**写后日志**，即在把数据写入内存之后，才记录该条命令进日志，这么设计的目的保证redis的高性能，保证可用性。但是也存在bug，就是写日志之前宕机了，会丢失数据。

AOF日志记录Redis的每个写命令，步骤分为：命令追加（append）、文件写入（write）和文件同步（sync）。其中写入和同步有三种方式，各有其利弊。

在redis.conf中配置AOF及其对应的一些参数，appendonly appendfilename appendfsync 等等



##### **AOF重写概念**

当Redis不断进行着操作时，其需要写入AOF的命令会越来越多，AOF文件的体积也不断膨胀。为了**减少AOF的体积**，加速AOF恢复的效率，引入了重写概念。重写就是创建一个新的AOF文件来**替换现有的AOF**，新旧两个AOF文件保存的数据相同，但新AOF文件会进过命令筛查过滤，**<u>将冗余的命令删除掉</u>**。

比如，在一个对一个元素先后执行LPUSH和RPOP，那么就相当于最终啥都没改变，那么这种命令就不再重写进新的AOF文件了。



1. AOF重写会阻塞主线程，但是仅仅是在fork阶段阻塞。子线程被fork出来第一件事是执行**bgrewriteof**命令，将主线程的内存区域拷贝一份，这个阶段会阻塞
2. 两个参数会控制重写的时机
   - `auto-aof-rewrite-min-size`表示运行AOF重写时文件的最小大小，默认为64MB
   - `auto-aof-rewrite-percentage`这个值的计算方式是，当前aof文件大小和上一次重写后aof文件大小的差值，再除以上一次重写后aof文件大小。也就是当前aof文件比上一次重写后aof文件的增量大小，和上一次重写后aof文件大小的比值。
3. 在写AOF期间，数据发生了修改，那么修改的数据进入**AOF重写缓冲区**，待到设定的AOF同步时间再进行一次AOF数据的同步。



##### **RDB结合AOF进行持久化**

二者结合的核心要义是：采用RDB方式进行大备份，全程使用AOF备份每个操作

二者结合使用时先检查AOF，因为AOF中保留的数据非常完整

<img src="https://pdai.tech/images/db/redis/redis-x-aof-5.png" alt="img" style="zoom: 50%;" />

优先采取AOF的方式来恢复数据的原因，它的实时的，每写一条数据进内存之后都会写一个日志。然而RDB是间隔性的，大概率会存在一段时间内的数据缺失。





#### Redis事务

一个事务包含了多个命令，服务器在执行事务期间，不会改去执行其它客户端的命令请求。事务中的全部命令要么全部成功要么全部失败。也遵循ACID原则。

Redis事务中的多个命令被一次性发送给服务器，而不是一条一条发送，这种方式被称为<u>流水线</u>，它可以减少客户端与服务器之间的网络通信次数从而提升性能。

Redis中事务的开启方式：把要执行的事务命令群包在muti和exec中间

**Redis不支持回滚**，官方的设计逻辑很签单。为了追求高性能，不为不该redis买单的错误的负责。怎么理解呢，回滚操作是存在开销的，而事务中的命令出现问题显然应该在程序员执行前就该解决。所以，在redis中，事务中有命令失败后不会回滚，而是直接执行剩余的命令。





#### Redis事件

Redis 服务器是一个事件驱动程序。事件驱动可以这么来理解，让驴拉磨，它不拉，你用鞭抽一下，它就开始拉了。然后又停了，你再抽一下，它又继续拉了。也就是系统做不做一件事件，是由某种信号来驱动的。在Redis当中，这种信号有两种，来自网络IO的文件、时间。文件驱动就是，网络IO传了一个文件给redis服务器，那么redis服务器需要去解析，然后找到合适的途径处理这个文件。时间驱动则是到达设定的时间节点，redis服务器就需要执行某项命令。

![image-20220428181319305](../assets/images/redis/image-20220428181319305.png)

时间驱动的运行逻辑就是不断循环监听，看看”驱动力“有没有到达

![image-20220428181408313](../assets/images/redis/image-20220428181408313.png)





#### 主从复制

> RDB全量复制、replica_buffer增量复制

Redis 提供了主从库模式，以保证数据副本的一致，主从库之间采用的是**读写分离**的方式。Redis的主从复制和mysql不同的地方在于，redis支持一主多从，多台从机共同提供读服务

redis的主从搭建方式较mysql更加简单，`replicaof ip port`

redis的复制分为全量复制和增量复制两种

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230209173239487.png" alt="image-20230209173239487" style="zoom:33%;" />



##### 全量复制

主从复制通常使用**RDB**进行，因为RDB采用二进制传输，并且压缩后文件大小很小，传输的开销低。AOF的话需要指定刷盘策略，由此带来无畏的开销（Always-同步写盘、EverySec-每秒写一次、No-机器自决）；此外，AOF会涉及到数据还原，相对RDB来说耗费的时间更大。

增量复制是在主从之间因为网络断开导致部分数据不同步，随后将缺失的那部分数据给补上。增量复制有两个关键的buffer。



##### **增量复制存在rep buf缓冲区被覆盖的情况**

`repl_backlog_buffer`是在主从之间找到彼此的差异之后，主机将从机没有的命令放到该环形缓冲区当中。如果该缓冲区因为数据太多，前面的数据被后面的数据覆盖了的话，唯一的解决方案就是进行一次大开销的全量复制。所以，这个缓冲区初始容量尽可能设置得大一些。

`replication buffer`是主机准备发送给从机的数据放入该缓冲区，该缓冲区相当于一个简单的发送缓冲区。



##### 从库的从库设计

所有的从库都是和主库连接，所有的全量复制也都是和主库进行的。现在，我们可以通过“主 - 从 - 从”模式**将主库生成 RDB 和传输 RDB 的压力，以级联的方式分散到从库上**。

这样一来，这些从库就会知道，在进行同步时，不用再和主库进行交互了，只要和级联的从库进行写操作同步就行了，这就可以减轻主库上的压力。

这些从库的从库不承担读访问，只是单纯作为备份机。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230209175912040.png" alt="image-20230209175912040" style="zoom:50%;" />



##### **读写分离存在的问题**

<u>数据不一致问题：</u>有命令传播时延，数据的不一致延迟无法避免。只能通过合理安排主从、将机器放在同一局域网当中以提高性能

<u>数据过期问题：</u>Redis 3.2中，从节点在读取数据时，增加了对数据是否过期的判断：如果该数据已过期，则不返回给客户端；将Redis升级到3.2可以解决数据过期问题

<u>故障切换问题：</u>用哨兵模式，或者其他分布式协调框架



#### 集群模式

集群的目标是通过部署大量的节点，以及保障高可拓展的能力。这其中涉及到主机之间的协调（分片），和集群的管理。

Redis的集群包括这几种：Standalone（即一主多从不使用哨兵）、一主多从使用哨兵、分片（多主多从）







#### 哨兵模式

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230209182049203.png" alt="image-20230209182049203" style="zoom:33%;" />

在部署了Sentinel的情况下，与客户端进行直接通信的则是Sentinel

**哨兵提供的功能**

<u>监控（Monitoring）：</u>哨兵会不断地检查主节点和从节点是否运作正常。

<u>自动故障转移（Automatic failover）：</u>当主节点不能正常工作时，哨兵会开始自动故障转移操作，它会将失效主节点的其中一个从节点升级为新的主节点，并让其他从节点改为复制新的主节点。

<u>配置提供者（Configuration provider）：</u>客户端在初始化时，通过连接哨兵来获得当前Redis服务的主节点地址。

<u>通知（Notification）：</u>哨兵可以将故障转移的结果发送给客户端。



<!--哨兵的通信方式 VIP-->

哨兵集群的实现是通过Redis提供的 **pub/sub 机制**，主库上建立一个哨兵频道，哨兵不断向其中发布自己的ip和端口，同时监听该频道上其他哨兵发布的ip和端口，实现互相监听的效果。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230209182652824.png" alt="image-20230209182652824" style="zoom:50%;" />

**主机下线判断**

每个哨兵和主机之间维持心跳，如果某台哨兵发现主机不回应心跳，则向其他哨兵发送`is-master-down-by-addr`。如果返回的结果中，已下线的回复大于`quorum`个，那么该主机客观下线。



**哨兵监视从库**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230209195340683.png" alt="image-20230209195340683" style="zoom:50%;" />



**哨兵leader选举**

只有当前主机挂掉之后才进行哨兵leader选举，被选中的那个哨兵来进行主从切换的一系列工作，选主的算法采用的是Raft算法：

1. 新主库选择，根据conf里面设置的**priority**选择
2. 将slave-1脱离原从节点（PS: 5.0 中应该是`replicaof no one`)，升级主节点
3. 将从节点slave-2指向新的主节点
4. 通知客户端主节点已更换
5. 将原主节点（oldMaster）变成从节点，指向新的主节点

选举本身算法很多（分布式共识），常用Raft算法，核心要义就是投票，然后得票数符合要求，投票时的偏好倾向是conf里面设置的优先级和主从同步的偏移量



**故障转移**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230209195650434.png" alt="image-20230209195650434" style="zoom:50%;" />



#### Redis分片

<!--分片机制 哈希槽-->

由于redis的主从复制只是将数据进行全量备份在多台机器（保证高可用），但是当数据量总量大到单台机器无法承载的时候就需要分片了（保证可扩展性）。分片是将数据划分为多个部分的方法，可以将数据存储到多台机器里面，这种方法在解决某些问题时可以获得线性级别的性能提升。

假设有 4 个 Redis 实例 R0，R1，R2，R3，还有很多表示用户的键 user:1，user:2，... ，有不同的方式来选择一个指定的键存储在哪个实例中。

- 最简单的方式是范围分片，例如用户 id 从 0~1000 的存储到实例 R0 中，用户 id 从 1001~2000 的存储到实例 R1 中，等等。但是这样需要维护一张映射范围表，维护操作代价很高。
- 还有一种方式是哈希分片，使用 CRC32 哈希函数将键转换为一个数字，再对实例数量求模就能知道应该存储的实例。这种方式也被称为**哈希槽（Hash Slot）**，槽位总数一般为16384个，然后除以N就是每个节点锁分到的槽位数，一般都是直接从0考16383开始分，不随机，

根据执行分片的位置，可以分为三种分片方式：

- 客户端分片：客户端使用一致性哈希等算法决定键应当分布到哪个节点。
- 代理分片：将客户端请求发送到代理上，由代理转发请求到正确的节点上。
- 服务器分片：Redis Cluster。



**节点定位——请求重定向**

**Moved重定向**

<img src="https://pdai.tech/images/db/redis/redis-cluster-3.png" alt="img" style="zoom: 50%;" />

**Ask重定向，**这类重定向更多地发生在集群节点变迁的阶段

<img src="https://pdai.tech/images/db/redis/redis-cluster-5.png" alt="img" style="zoom:50%;" />





<!--节点间通信-->

**Cluster总线**

每个Redis Cluster节点有一个额外的TCP端口用来接受其他节点的连接。这个端口与用来接收client命令的普通TCP端口有一个固定的offset。该端口等于普通命令端口加上10000.例如，一个Redis服务端在端口6379接受客户端连接，那么它的集群总线端口16379也会被打开。随后的集群间的各类通信都是通过这个端口来进行的。



**Gossip协议**

Gossip协议包括三个通信操作，PING、PONG、MEET，后者是当集群中出现新的节点时，该操作会将新节点联通后加入自己的节点拓扑，然后通过PING、PONG发送给其他的节点。

集群中的每个节点都会定期地向集群中的其他节点发送PING消息，以此交换各个节点状态信息，检测各个节点状态：**在线状态**、**疑似下线状态PFAIL**、**已下线状态FAIL**。

Redis节点会记录其向每一个节点上一次发出ping和收到pong的时间，心跳发送时机与这两个值有关。通过下面的方式既能保证及时更新集群状态，又不至于使心跳数过多：

- 每次Cron向所有未建立链接的节点发送ping或meet
- 每1秒从所有已知节点中随机选取5个，向其中上次收到pong最久远的一个发送ping
- 每次Cron向收到pong超过timeout/2的节点发送ping
- 收到ping或meet，立即回复pong



**消息包中包含的信息**

Header，发送者自己的信息 

- 所负责slots的信息
- 主从信息
- ip port信息
- 状态信息

Gossip，发送者所了解的部分其他节点的信息 

- ping_sent, pong_received
- ip, port信息
- 状态信息，比如发送者认为该节点已经不可达，会在状态信息中标记其为PFAIL或FAIL

一个节点在拿到来自其他节点的数据以后，会对**slots和epoch**等信息进行比较，如果发送发现自己的信息是旧的则会将自己的数据更新。



**新节点的加入**

![img](https://pdai.tech/images/db/redis/redis-cluster-1.png)



**故障恢复/重新选主**

当slave发现自己的master变为FAIL状态时，便尝试进行Failover，以期成为新的master。由于挂掉的master可能会有多个slave。Failover的过程需要经过类Raft协议的过程在整个集群内达到一致， 其过程如下：

- slave发现自己的master变为FAIL
- 将自己记录的集群currentEpoch加1，并广播Failover Request信息
- 其他节点收到该信息，只有master响应，判断请求者的合法性，并发送FAILOVER_AUTH_ACK，对每一个epoch只发送一次ack
- 尝试failover的slave收集FAILOVER_AUTH_ACK
- 超过半数后变成新Master
- 广播Pong通知其他集群节点



要注意的是，这种选举是在一主多从的情况之下进行的，在此之外的其他一主多从并没有这个权力去选别的一主多从下的主。

<img src="https://pdai.tech/images/db/redis/redis-cluster-2.png" alt="img" style="zoom:67%;" />



<!--除此之外其他的分片方案-->

豌豆荚开源的Codis，很想是MySQL中的MyCat，一个代理中间件，负责转发来自客户端的请求

分布式协调中间件，例如Keepalived、ZK也可以达到这种目的



#### Redis分布式锁的实现

分布式锁其实就是，控制分布式系统不同进程共同访问共享资源的一种锁的实现。如果不同的系统或同一个系统的不同主机之间共享了某个临界资源，往往需要互斥来防止彼此干扰，以保证一致性。其实本质也就是进程间的一种通信机制。

> **互斥性**: 任意时刻，只有一个客户端能持有锁。
>
> **锁超时释放**：持有锁超时，可以释放，防止不必要的资源浪费，也可以防止死锁。
>
> **可重入性**:一个线程如果获取了锁之后,可以再次对其请求加锁。
>
> **高性能和高可用**：加锁和解锁需要开销尽可能低，同时也要保证高可用，避免分布式锁失效。
>
> **安全性**：锁只能被持有的客户端删除，不能被其他客户端删除.



**Redis分布式锁方案一：SETNX + EXPIRE**

即先用setnx来抢锁，如果抢到之后，再用expire给锁设置一个过期时间，防止锁忘记了释放

SETNX 是SET IF NOT EXISTS的简写.日常命令格式是SETNX key value，如果 key不存在，则SETNX成功返回1，如果这个key已经存在了，则返回0

这种操作的效果是一个线程能在获取到锁的期间里独享一个key的写权限

这种方案的缺陷是不能做到原子性，如果setnx成功后机器crash掉了，那么就不会走到expire这一步。但是可以采用**lua脚本**将setnx和expire写成一个原子操作

解决释放锁的问题还可以**配合SETNX和SETEX**使用，redis自带的设置锁过期时间的操作。但是这种操作可能会带来的问题是锁过期了但是业务还么有执行完

在具体的实践当中更加倾向于set nx ex的方式来做，这样能够以原子性的方式设置过期时间

```java
// 1.1.1.1:6379> set name p7+ ex 100 nx
// 命令行方式

// Java API的方式则是构建一个执行语句然后execute
public boolean setAndExpireIfAbsent(final String key, final Serializable value, final long expire) {

        Boolean result = (Boolean) RedisTemplateHolder.execute(new Statement() {
            @Override
            public Object prepare(RedisTemplate redisTemplate) {
                return redisTemplate.execute(new RedisCallback<Boolean>() {
                    @Override
                    public Boolean doInRedis(RedisConnection connection) throws DataAccessException {
                        Object obj = connection.execute("set", serialize(key), serialize(value), SafeEncoder.encode("NX"), SafeEncoder.encode("EX"), Protocol.toByteArray(expire));
                        return obj != null;
                    }
                });
            }
        }, redisTemplate);

        return result;
    }

// 在单机多线程安全方面可以使用semaphore
				/**
         * 最大有20个redis连接被使用，其他的连接要等待令牌释放
         * 令牌数量自己定义，这个令牌是为了避免高并发下，获取redis连接数时，抛出的异常
         * 在压力测试下，性能也很可观
         */
        private static Semaphore semaphore = new Semaphore(20);

        private RedisTemplateHolder() {
        }

        public static RedisTemplate getRedisTemplate(RedisTemplate redisTemplate) {
            try {
                semaphore.acquire();
                return redisTemplate;
            } catch (Exception e) {
                throw new IllegalStateException(e);
            }
        }

        public static void release() {
            semaphore.release();
        }

```



**方案二：Redisson框架**

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/367cd1a7a3fb4d398988e4166416d71d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp" alt="img" style="zoom: 25%;" />

Redisson框架主要解决了**锁过期但是还没有执行完业务**的问题。Redisson核心包括两个部分：循环请求去获取锁；当锁获取成功后开启一个看门狗**watch dog**，每10s检查一次线程是否还持有锁，如果仍然持有锁则延迟锁持有时间

```java
// redisson的使用
@Configuration
public class RedisConfig {

  //这里在application.yml中填写你对应的redis的ip:port
    @Value("${redis.address}")
    private String redisAddress;

    @Bean
    public RedissonClient redissonClient() {
        Config config = new Config();
        config.useSingleServer().setAddress(redisAddress);
        return Redisson.create(config);
    }
}

//首先获取分布式的锁
  RLock lock = redissonClient.getLock("deductProduct");
  try{
    LOGGER.info("分布式锁加锁!");
    //尝试对redis的分布式锁进行加锁
    boolean success = lock.tryLock(30, TimeUnit.SECONDS);
    if (!success){
      //加锁失败，直接返回
      return false;
    }finally {
    //最后一定记得释放锁资源
    LOGGER.info("分布式锁释放！");
    lock.unlock();
  }

    
```

底层源码部分

```java
if (acquired) {
      if (leaseTime != -1) {
        // 明确指定了租约时间，则更新类相应的租约时间即可
        internalLockLeaseTime = unit.toMillis(leaseTime);
      } else {
        // 否则将当前的ThreadId保存到一个相应的ConcurrentMap中，
        // 开启守护线程，定期刷新对应线程ID持有锁的过期时间。避免出现锁过期被释放的问题
        // watch dog机制的来源
        scheduleExpirationRenewal(threadId);
      }
    }

// 最底层，也就是获取锁的那个方法，本质就是lua语句
<T> RFuture<T> tryLockInnerAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
  //这个命令的逻辑相对清晰，首先判断当前的key值是否存在
  //==0则代表哈希的key不存在，则此时新增哈希的key及field对象
  //==1则代表哈希key及对应的field对象存在，刷新其过期时间，同时会返回其剩余的超时时间。
  return evalWriteAsync(getRawName(), LongCodec.INSTANCE, command,
                        "if (redis.call('exists', KEYS[1]) == 0) then " +
                        "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                        "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                        "return nil; " +
                        "end; " +
                        "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                        "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                        "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                        "return nil; " +
                        "end; " +
                        "return redis.call('pttl', KEYS[1]);",
                        Collections.singletonList(getRawName()), unit.toMillis(leaseTime), getLockName(threadId));
}
```





**方案三：分布式解决方案RedLock+Redisson**

上述方案都是在单机情况下，在分布式多机环境下应该使用Redlock。在最新的Redisson中集成了Redlock，这就是一种分布式的协同算法 

- 按顺序向5个master节点请求加锁
- 根据设置的超时时间来判断，是不是要跳过该master节点。
- 如果大于等于三个节点加锁成功，并且使用的时间小于锁的有效期，即可认定加锁成功啦。
- 如果获取锁失败，解锁！

```java
Config config1 = new Config();
config1.useSingleServer().setAddress("redis://localhost:5378")
        .setPassword("").setDatabase(0);
RedissonClient redissonClient1 = Redisson.create(config1);

Config config2 = new Config();
config2.useSingleServer().setAddress("redis://localhost:5379")
        .setPassword("").setDatabase(0);
RedissonClient redissonClient2 = Redisson.create(config2);

Config config3 = new Config();
config3.useSingleServer().setAddress("redis://localhost:5380")
        .setPassword("").setDatabase(0);
RedissonClient redissonClient3 = Redisson.create(config3);

// 初始化出集群全部节点
String lockKey = "REDLOCK";
RLock lock1 = redissonClient1.getLock(lockKey);
RLock lock2 = redissonClient2.getLock(lockKey);
RLock lock3 = redissonClient3.getLock(lockKey);

// 通过redisson来搭建redlock环境
RedissonRedLock redLock = new RedissonRedLock(lock1, lock2, lock3);
try {
    if (redLock.tryLock(10, 5, TimeUnit.SECONDS)) {
        // TODO if get lock success, do something;
    }
 }catch(Exception e){
    
 }
```



#### Redis问题排查

Redis这种缓存中间件的设计初衷是为了分担DB的压力，正常的应用都是查询频繁型的，有可能DB服务器难以承受大量的访问而导致崩溃。具体的流程就是，用户需要查询什么数据，先去redis缓存中进行查询，如果缓存中不存在，那么再去访问DB。

**缓存穿透**  **Cache Penetration** 	指的是查同一条数据  更多指向人为攻击 

缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求。缓存中的数据一定是DB中存在的数据（前提是合理的缓存、DB删除一致策略），用户查询某个数据，该数据在DB而不在缓存中，则系统会被动地将该数据载入缓存。所以，当用户频繁查询缓存中没有的数据时则会导致频繁的查库行为，冲破了缓存搭建的保护墙。

这种情况的发生有两种情况，要么是大量数据范围下部分冷门数据被查询，要么是人为攻击（比如用id=-1去查询缓存）

解决方案：

1. 接口层**增加校验**，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
2. 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为**key-null**，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击；
3. 设置**bloomfilter**，它类似于一个hash set，用于快速判某个元素是否存在于集合中。将需要屏蔽掉的规则直接过滤了去



##### **缓存击穿  ** Cache Breakdown

通常考虑业务

缓存击穿是指<u>缓存中没有但数据库中有的数据</u>（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。

解决方案：

1. 针对系统自身设计问题，可以采取预热机制，即在某项业务中，将可预见的数据先载入缓存。比如在中秋周围，人们会大量采购月饼，那么把type=”月饼“的商品id先载入缓存
2. 加互斥锁，但这样会带来性能问题
3. 接口限流与熔断，降级；这种方法会降低可用性



##### **缓存雪崩**  

缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，<u>缓存击穿指并发查同一条数据</u>，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

解决方案：

1. 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
2. 如果缓存数据库是分布式部署，将热点数据均匀分布在不同的缓存数据库中。
3. 设置热点数据永远不过期。

总结就是要对自己开发的系统的业务足够熟悉，明白什么时间点会存在峰值，产品经理很少会关注因down机无法提供服务问题（或者说主动去预防这个问题）



##### **缓存污染（淘汰）**

这个问题集中在把缓存中哪些久久没被用到的数据淘汰出去。缓存数据放在内存上比较宝贵，如果多次没被命中还会拖垮DB。是一个在实际开发中非常常见且实用的问题。

基本上就是操作系统那几个页面置换算法一样。涉及到淘汰谁，要么就是根据业务实际情况，选择优先淘汰权重低的；要么就是按时间，先来先走，先来后走等。



##### **缓存与DB数据一致性问题**

当需要对数据进行修改的时候会发现，不管是先改缓存的数据还是先改数据库的数据，从数据库到缓存都存在一个时延。如果在时延期间对数据进行访问，那么二者的数据必然不一致。所以，不管是先写MySQL数据库，再删除Redis缓存；还是先删除缓存，再写库，都有可能出现数据不一致的情况。

节选最最常用的<u>Cache Aside Pattern</u>, 总结来说就是

- 读的时候，先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应。
- 更新的时候，先更新数据库，然后再删除缓存。

这种处理方式最大程度上规避了读旧数据的问题。如果用户操作是更新数据库中的数据，那么在数据库更新完成后会对该数据位做标记，后续的查询操作再把数据从数据库中拉出来。这样虽然会带来一定的性能问题，但是解决了脏读情况的发生。



#### Redis和MySQL经典缓存一致性

缓存一致性是一个相对的概念，追求绝对的缓存缓存一致性是很难的，在单服务器上缓存一致性的努力方向是尽可能减少不一致的出现时间。而在分布式系统当中，努力的方向应该是保证最终的一致性。

**靠谱办法一：先更新数据库，接着删除缓存**

操作方法：将数据库的更新与删缓存打包成一个事务；或者开启一个程序监听binlog，但是这样延迟可能较高

还可以在业务上进行一定的处理，比如买东西的时候，在下单时不走缓存

**办法二：带版本写入**

对缓存层的数据写入需要校验数据版本，只有当版本比之前的高的时候数据才能写入成功





#### **Redis为什么快**

（1）完全基于内存，数据存在内存中，绝大部分请求是纯粹的内存操作，非常快速，跟传统的磁盘文件数据存储相比，避免了通过磁盘IO读取到内存这部分的开销

（2）数据结构简单，对数据操作也简单。Redis中的数据结构是专门进行设计的，每种数据结构都有一种或多种数据结构来支持。Redis正是依赖这些灵活的数据结构，来提升读取和写入的性能

（3）采用**单线程**，省去了很多上下文切换的时间以及CPU消耗，不存在竞争条件，不用去考虑各种锁的问题，不存在加锁释放锁操作，也不会出现死锁而导致的性能消耗

（4）使用基于**IO多路复用**机制的线程模型，可以处理并发的链接。Reactor事件监听机制 event driven，并且Redis的多路复用用的是epoll时候，是一种被动机制，节省了很多轮询请求的时间

<img src="https://img-blog.csdnimg.cn/20210603044938131.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDUyMzM3MDA=,size_16,color_FFFFFF,t_70" alt="img" style="zoom:33%;" />

**但后来又开始使用多线程了**

redis使用多线程并非是完全摒弃单线程，redis还是使用单线程模型来处理客户端的请求，只是使用多线程来处理数据的读写和协议解析，执行命令还是使用单线程。

这样做的目的是因为redis的性能瓶颈在于网络IO而非CPU，使用多线程能提升IO读写的效率，从而整体提高redis的性能。



##### 单线程下的阻塞问题

当执行一些CPU密集型的操作，比如AOF持久化，RDB持久化中的fork()操作，大key操作，getAll操作等等都会导致CPU满负荷运转而产生阻塞问题



#### OOM

maxmemory这个配置项可以设置最大占用内存，可以通过改配置文件的方式来设置，也可以直接在redis的console里面configure set memory <bits>

64位的机器默认是无限占用，32位最大占用默认是3GB



Redis默认的OOM处理是no eviction，也就是不处理，等待用户来手动处理

比较常用的两个删除算法是LRU和LRF，一个基于时间，另一个基于频率；另外一个纬度是，删除设置了过期时间的key还是全部都执行定期删除

configure set memory- policy



删除的策略有三种（具体如何来删除一个key）：

1. 立即删除，这样对CPU不友好
2. 惰性删除，OOM了才去专门执行删除，这样对业务不友好
3. 定期删除，每隔一定的频率，随机抽取一部分的key检查是否过期，如果过期则立刻删除

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230217184830070.png" alt="image-20230217184830070" style="zoom: 33%;" />





#### 性能调优案例

这个东西必须吹说自己有过调优的经验，没办法！

https://pdai.tech/md/db/nosql-redis/db-redis-x-performance.html
