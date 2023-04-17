---
layout: post
title: "kafka快速入门"
permalink: /kafka-crash
---



**问：Kafka和AMQP的RabbitMQ区别有哪些？**

Kafka支持消息的有序和持久化，吞吐量大

Rabbit的AMQP更加灵活，适合的业务场景比较广泛

AMQP模型：exchange --- route --- queue

AMQP模型有三层：module layer / session layer / transport layer



**Kafka能够起到的作用：**消息引擎，流处理平台，分布式存储系统



消息队列的两种工作模式：

点对点：即路由匹配，消费后即删除

发布/订阅：根据topic分类订阅，消费后不删除，供其他消费者进行消费



### 基础架构

---

![image-20220516231755246](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220516231755246.png)

重要知识点：

分区与消费者是多对一的关系

**一个消费者（组）只能订阅一个主题**

分区备份 leader和follower的区别

>  kafka中的**leader负责处理读写操作，而follower只负责副本数据的同步**如果leader出现故障，其他follower会被重新选举为leader，follower像一个consumer一样，拉取leader对应分区的数据，并保存到日志数据文件中。
>
> Kafka为什么不支持读写分离？
>
> 网络→主节点内存→主节点磁盘→网络→从节 点内存→从节点磁盘。这条链路很长，导致主从之间的延迟比较大，主机上的修改要经过很长的时延才能同步进从机的内存，so从机提供的读服务也是不太可靠的。

使用zookeeper做服务注册以及分区备份管理中心

Kraft模式 不依靠zk做管理中心



**Kafka部署在Linux上比Windows要拥有更高的IO性能**

JVM中IO在最底层会调用Selector模型，而Linux里的Selector的具体调用为epoll，而在Windows中它的具体调用为select，epoll的性能相较于select要高出很多。**原因在于epoll是被动模式，来一个处理一个；而select是主动轮询模式，会一遍遍地去轮询阻塞队列来获取请求进行处理。**



当数据在磁盘和网络进行传输时避免昂贵的内核态数据拷贝从而实现快速地数据传输。Linux平台实现了这样的**零拷贝机制**



**Producer永远要使用带有回调通知的发送API**，也就是说不要使用producer.send(msg)，而要使用 producer.send(msg, callback)



### 环境配置

---

通过shell脚本来启动集群

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220605215358929.png" alt="image-20220605215358929" style="zoom: 35%;" />



unclean.leader.election.enable默认为false，当集群出现分区partitioning时数据会丢失，Kafka在CAP当中选择了对分区的容忍性



**Max.poll.interval.ms**

**Max.poll.records**

上面这两个参数如果配置不正确会产生rebalance的问题，当records设置为一次拉去500条数据，但是consumer在处理这500条的数据耗费的时间超过了interval间隔，那么Kafka就认为consumer掉线了，会进行分区的rebalance



JVM的Heap Size改为6G为妥  KAFKA_HEAP_OPTS

收集器方面，如果CPU够强就CMS（Current Mark Sweep），否则ParallelGC；使用Java8的情况下，内存等不是很局促用默认的G1就够了，Full GC的数量少，时间效率高



#### 消息不丢失配置

<!--消息丢失包括两种情况：一种是消息发送机制的原因，比如失败后不做处理；另外一种是服务器在cache刷盘期间down了-->



消息丢失存在于三个方面，Producer、Broker、Consumer



在Producer上，Buffer满了会导致触发消息丢弃，解决方案：

- 异步发送消息改为同步发送消。或者service产生消息时，使用阻塞的线程池，并且线程数有一定上限。整体思路是控制消息产生速度。
- 扩大Buffer的容量配置。这种方式可以缓解该情况的出现，但不能杜绝。
- service不直接将消息发送到buffer（内存），而是将消息写到本地的磁盘中（数据库或者文件），由另一个（或少量）生产线程进行消息发送。相当于是在buffer和service之间又加了一层空间更加富裕的缓冲层。



在Broker上，是服务器在cache刷盘期间down了，这个只能通过调整消息的持久化机制来做到尽可能地保证

设置副本数

设置ack=1的最少副本落盘数



在Consumer上，尽量开始手动提交



1. 不要使用producer.send(msg)，而要使用**producer.send(msg, callback)**。记住，一定要使用带有回调通 知的send方法。

   > 这边Callback原生不带消息展示，只是在日志中记录exception的相关信息。但是可以自己去写，重写onFailure、onSuccess等方法即可。

2. 设置**acks = all**。acks是Producer的一个参数，代表了你对“已提交”消息的定义。如果设置成all，则表明所有副本Broker都要接收到消息，该消息才算是“已提交”。这是最高等级的“已提交”定义。

3. 设置**retries**为一个较大的值。这里的retries同样是Producer的参数，对应前面提到的Producer自动重试。当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了retries > 0的Producer能够自动重试消 息发送，避免消息丢失。

4. 设置**unclean.leader.election.enable = false**。这是Broker端的参数，它控制的是哪些Broker有资格竞选 分区的Leader。如果一个Broker落后原先的Leader太多，那么它一旦成为新的Leader，必然会造成消息的丢失。故一般都要将该参数设置成false，即不允许这种情况的发生。

5. 设置**replication.factor >= 3**。这也是Broker端的参数。其实这里想表述的是，最好将消息多保存几份， 毕竟目前防止消息丢失的主要机制就是冗余。

6. 设置**min.insync.replicas > 1**。这依然是Broker端参数，控制的是消息至少要被写入到多少个副本才算 是“已提交”。设置成大于1可以提升消息持久性。在实际环境中千万不要使用默认值1。

7. 确保**replication.factor > min.insync.replicas**。如果两者相等，那么只要有一个副本挂机，整个分区就无 法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。推 荐设置成replication.factor = min.insync.replicas + 1。

5. 确保消息消费完成再提交。Consumer端有个参数**enable.auto.commit**，最好把它设置成false，并采用手 动提交位移的方式。就像前面说的，这对于单Consumer多线程处理的场景而言是至关重要的。



### 命令行操作

---

Kafka的命令行操作都是通过运行脚本+参数的方式来进行。生产者端、Topics服务器、消费者端都分别对应着有一个shell脚本

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220605220028230.png" alt="image-20220605220028230" style="zoom:35%;" />

#### **kafka-topic.sh**

对主题进行操作有个要点，就是分区数partitions只能增加而不能减少



#### **kafka-consumer.sh**

可以指定数据的消费是全量或者增量



### Kafka Producer

---

![image-20220605233240759](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220605233240759.png)

kafka对批处理的原生支持体现点1，支持消息的批量发送



#### **异步发送**

普通异步、回调异步（以Callback内部lambda方法作为send的入参）



#### **同步发送**

在同步send的之后调用其get( )方法，最终效果就是每次数据的发送必须要等待上一次完成



#### **分区策略**

1. 显式通过分区号指定分区
2. 妹指定分区号，则通过key的hash值对分区个数取余
3. 分区号和key都没设置，那么根据“黏合”规则随即分配给一个合适的分区，并一直在此分区当中
4. 自定义分区器partitoner，然后properties.put(**args, myPatitioner)



**kafka同一个topic是无法保证数据的顺序性的，但是同一个partition中的数据是有顺序的。如果想保证多业务都是有序的话，应该使用分区而非主题来区别每一个业务**

```java
public class ProductService {
    /**
     * data为需要发送的数据
     */
    public void partitionSend(String topic, int partition, JSONObject data) {
         // 获取设备id
        String deviceId = data.getString("deviceId");
        // 获取自增数 如果是新设备会创建一个并放入缓存中
        int inc = getDeviceInc(deviceId);
        // 如果分区数为3 设备自增id为1 取模结果为1 就是发送到1分区 这样1000个设备就可以保证每个分区发送数据量是1000 / 3
        int targetPartition = Math.floorMod(inc, partition);
        // 分区发送时候 需要指定一个唯一k 可以使用uuid或者百度提供的雪花算法获取id 字符串即可
        kafkaTemplate.send(topic, targetPartition, getUuid(), data.toJSONString());
    }
}
```



#### **提高生产者端的吞吐量**

下面这些参数配置进行合适的调配能够提高生产者的吞吐量

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220606123218662.png" alt="image-20220606123218662" style="zoom: 33%;" />



#### **生产者端的数据可靠性**

ack级别

1. **-1**(**all**):生产者发送过来的数据，**Leader**和**ISR**队列里面的所有节点（Follower）收齐数据后应答。别名At Least Once
2. **0**:生产者发送过来的数据，不需要等数据落盘应答 atguigu Hello
3. **1**:生产者发送过来的数据，**Leader**收到数据后应答。别名Only Once

数据完全可靠条件 = ACK级别设置为-1 + 分区副本大于等于2 + ISR里应答的最小副本数量大于等于2。**这其中与性能有一个trade off**



#### **避免消息的重复发送**

幂等性原则，依靠消息中的三个参数来实现 ProducerID、PartitionID、Seq Num

生产发送的消息没有收到正确的broke响应，导致producer重试。producer发出一条消息，broke落盘以后因为网络等种种原因发送端得到一个发送失败的响应或者网络中断，然后producer收到一个可恢复的Exception重试消息导致消息重复。 

解决办法最快的就是开启kafka自身带的幂等配置：**enable.idempotence=true ； ack=all ； retries>1**



#### **保证数据的有序性**

保证单分区下的有序，参数开启max.in.flight.requests.per.connection=1即可，无需其他设置

保证多分区下的有序，参数max.in.flight.requests.per.connection需要设置小于等于5，同时开启幂等性。基本原理：kafka集群当中开启了消息缓存，会按序将消息落盘后再发出，但是缺陷在于每次只能缓存5个数据

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220606160928977.png" alt="image-20220606160928977" style="zoom: 33%;" />



#### 拦截器

类似于Spring的Interceptor，在Producer和Consumer之间都存在

当前Kafka拦截器的设置方法是通过参数配置完成的。生产者和消费者两端有一个相同的参数，名字叫 **interceptor.classes**，它指定的是一组类的列表，每个类就是特定逻辑的拦截器实现类。也可以在Java代码中的KafkaConfig类中进行配置，拿上面的例子来 说，假设第一个拦截器的完整类路径是 com.yourcompany.kafkaproject.interceptors.AddTimeStampInterceptor，第二个类是 com.yourcompany.kafkaproject.interceptors.UpdateCounterInterceptor，那么你需要按照以下方法在 Producer端指定拦截器:

![image-20230202190108153](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230202190108153.png)



Producer端有两个周期方法：onSend、onAcknowledgement

Consumer端：onConsume、onCommit



#### 生产者的TCP管理

1. KafkaProducer实例创建时启动Sender线程，从而创建与bootstrap.servers中所有Broker的TCP连接。

2. KafkaProducer实例首次更新元数据信息之后，还会再次创建与集群中所有Broker的TCP连接。

3. 如果Producer端发送消息到某台Broker时发现没有与该Broker的TCP连接，那么也会立即创建连接。

4. 如果设置Producer端connections.max.idle.ms参数大于0，则步骤1中创建的TCP连接会被自动关闭;如

   果设置该参数=-1，那么步骤1中创建的TCP连接将无法被关闭，从而成为“僵尸”连接。



#### 生产者的事务与幂等（Idempotence）

当enable.idempotence被设置成true后，Producer自动升级成幂等性Producer，其他所有的代码逻辑都不需 要改变。Kafka自动帮你做消息的重复去重。底层具体的原理很简单，就是经典的用空间去换时间的优化思路，**即在Broker端多保存一些字段**。当Producer发送了具有相同字段值的消息后，Broker能够自动知晓这 些消息已经重复了，于是可以在后台默默地把它们“丢弃”掉。当然，实际的实现原理并没有这么简单，但你大致可以这么理解。

事务型Producer能够保证将消息原子性地写入到多个分区中。这批消息要么全部写入成功，要么全部失败。另外，事务型Producer也不惧进程的重启。Producer重启回来后，Kafka依然保证它们发送消息的精确一次处理。

设置事务型Producer的方法也很简单，满足两个要求即可:

和幂等性Producer一样，开启enable.idempotence = true，或者

设置Producer端参数transctional. id。最好为其设置一个有意义的名字。

也可以在业务代码中手动指明一系列操作的事务要求：

```java
producer.initTransactions();
   try {
               producer.beginTransaction();
               producer.send(record1);
               producer.send(record2);
               producer.commitTransaction();
   } catch (KafkaException e) {
               producer.abortTransaction();
}
```



### Kafka Broker(ZK dependent)

---

ZK在kafka业务下的节点描述

总体工作流程

#### **服役新节点**

先准备好节点机器并配置好集群参数，之后将zk和kafka都停掉，最后按序启动



#### **退役新节点**

数据转移 —> 停止接受消息 —> 退出



#### **Kafka副本**

默认为1份，但生产上建议2份，性能和可靠性的trade off

kafka的副本分为leader和follower，主机只会把数据发给leader副本，follower副本需要主动向leader副本进行数据的同步

kafka中的副本AR = ISR（正常的follower） + OSR（延迟过高的follower）



#### **分区节点选主**

![image-20220606213829290](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220606213829290.png)

上述的场景是一个topic四个分区，每个分区有四个副本。选主机制是在isr中找replicas最靠前的那个节点



#### **Leader和Follower的故障处理**

两个概念：

LEO（log end offset）：每个副本的最后一个offset，LEO其实就是最新的offset + 1

HW（High Watermark)：所有副本中最小的LEO

当Follower发生故障以后，其他的节点都退到HW线上（即所有节点都保证有的最低点），从这里开始同步

**Kafka只能保证故障后节点间*数据的一致性*，但是无法保证数据的不丢失**



#### **自定义分区（分区及其副本具体放哪个broker）**

修改这个配置文件increase-replication-factor.json

在其中指定每个partition的副本的具体存储位置

运行kafka-reassign-partition.sh脚本



#### **副本因子**

针对一个分区增加其副本的数量，kafka不能通过命令行来修改replica的数量

1. 修改配置文件 increase-replication-factor.json 将目标partition的分区数增加
2. 运行kafka-reassign-partition.sh脚本



#### **数据存储**（全部的消息）

<u>消息落盘后的索引</u>

> 下图中的索引查找流程很有参考价值

![image-20220606232913097](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220606232913097.png)

在index文件里面用的是相对索引（相对于文件名中的序号）

根据index找到消息的position后，再去到.log文件中根据position找到具体的那条log



#### 文件的清理与压缩



**delete**

**compact**



#### **kafka高效读写的原因** 

> 1. kafka本身的设计是就是分布式的，原生支持
> 2. 读数据采用稀疏索引，可以快速定位要消费的数据。即上面的图片所展示的
> 3. 顺序写磁盘，采用追加的方式让数据物理有序，能够极大加快磁盘读写
> 4. 页缓存 + 零拷贝

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220607091244808.png" alt="image-20220607091244808" style="zoom:33%;" />





### Kafka Consumer

---

![image-20220607161450974](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220607161450974.png)

#### 消费方式

MQ的消费方式中常见的是两种，pull模式和push模式

**kafka采用的consumer主动的pull模式，**因为consumer节点的性能不尽相同，采用broker统一定义的push速率可能会导致节点的性能旱涝不一



#### 消费者组

Consumer Group(CG)：消费者组，由多个consumer组成。形成一个消费者组的条件，是所有消费者的groupid相同。在一个指定的消费者组内，每个消费者负责消费不同分区的数据，并且一个消费者可以消费多个分区，但是一个分区不能由同一消费者组内的消费者共同消费。消费者组之间互不影响。**所有的消费者都属于某个消费者组**，即消费者组是逻辑上的一个订阅者

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220607161855834.png" alt="image-20220607161855834" style="zoom:33%;" />

**CG的初始化**

![image-20220607162300830](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220607162300830.png)

一个CG配置对应着一个coordinator，每个consumer都会同coordinator保持心跳。consumer会选出一个leader来进行消费方案、分区分配方案的制定



#### 消费者/分区分配策略

**Range**

当分不均匀的时候，某些节点按序多消费一个分区

一个节点挂掉的时候，45s后，某个节点继承消费挂掉节点的全部分区



**Roundbin**

轮询消费



***Sticky***

均匀且随机分配



#### ***offset***

按照offset消费，也就是按照时间消费（把时间转换为对应的offset）



#### **漏消费和重复消费**

offset的自动/手动提交会带来造成漏/重复消费的问题



#### 数据积压

消费能力不足导致，增加分区数量，使得分区数=消费者数

下游消费速度不够，增加每次批拉取数据的大小



#### offset主题

ZooKeeper这类元框架其实并不适合进行频繁的写更新，而 Consumer Group的位移更新却是一个非常频繁的操作。这种大吞吐量的写操作会极大地拖慢ZooKeeper集群的性能。

在新版本的Consumer Group中，Kafka社区重新设计了Consumer Group的位移管理方式，采用了将位移保存在Kafka内部主题的方法。这个内部主题就是让人既爱又恨的**_consumer_offsets**。



#### Kafka Rebalance问题

Rebalance本质上是一种协议，规定了一个Consumer Group下的所有Consumer如何达成一致，来分配订阅 Topic的每个分区。比如某个Group下有20个Consumer实例，它订阅了一个具有100个分区的Topic。正常情 况下，Kafka平均会为每个Consumer分配5个分区。这个分配的过程就叫Rebalance。



Rebalance的触发条件有3个。

1. **组成员数发生变更。**比如有新的Consumer实例加入组或者离开组，抑或是有Consumer实例崩溃被“踢 出”组
2. **订阅主题数发生变更。**Consumer Group可以使用正则表达式的方式订阅主题，比如 consumer.subscribe(Pattern.compile(“t.*c”))就表明该Group订阅所有以字母t开头、字母c结尾的主 题。在Consumer Group的运行过程中，你新创建了一个满足这样条件的主题，那么该Group就会发生 Rebalance
3. **订阅主题的分区数发生变更。**Kafka当前只能允许增加一个主题的分区数。当分区数增加时，就会触发订 阅该主题的所有Group开启Rebalance



控制好Rebalance的几个参数：max.records/ max.interval，也就是消费者一次从broker拉取数据的条数和从从broker拉取的频率。当一次拉的太多，下游处理时间过长导致没在interval再次从broker拉数据，这个时候broker会认为这个consumer挂了，便会触发rebalance



#### 位移主题  __consumer_offsets



当Kafka集群中的第一 个Consumer程序启动时，Kafka会自动创建位移主题。我们说过，位移主题就是普通的Kafka主题，那么它自然也有对应的分区数。但如果是Kafka自动创建的，分区数是怎么设置的呢?这就要看Broker端参数 offsets.topic.num.partitions 的取值了。

位移主题的副本数量也是遵循offsets.topic.replication.factor参数的配置。

enable.auto.commit是否开启自动提交       auto.commit.interval.ms自动提交间隔

也可以在业务代码中手动提交位移

Compact策略整理已经过时了的位移记录



社区于0.11.0.0版本推出 了StickyAssignor，即有粘性的分区分配策略。所谓的有粘性，是指每次Rebalance时，该策略会尽可能地保留之前的分配方案，尽量实现分区分配的最小变动。



手动提交位移的方式

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230205213630622.png" alt="image-20230205213630622" style="zoom: 50%;" />





<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230205213944037.png" alt="image-20230205213944037" style="zoom:50%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230205214328715.png" alt="image-20230205214328715" style="zoom:50%;" />



![image-20230205214439409](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230205214439409.png)





### Kafka Eagle 监控

---

MySQL

修改分配的内存

页面访问8048

demo示意图

![image-20220608123036479](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220608123036479.png)



### Kafka-Kraft模式

---

Kafka新架构示意图

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220608123535817.png" alt="image-20220608123535817" style="zoom: 50%;" />



### Kafka集成Flume

---

#### What's Flume

Flume是一个高可用、高可靠，分布式的海量日志采集、聚合和传输的系统。Flume支持在日志系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。其中Flume-NG是Flume的一个分支，其目的是要明显简单，体积更小，更容易部署。在kafka当中可以作为生产者或者是消费者

Flume-NG（Flume的一个分支）由一个个Agent来组成，而每个Agent由Source、Channel、Sink三个模块组成，其中Source负责接收数据，Channel负责数据的传输，Sink则负责数据向下一端的发送。Flume也可以配置成多个Source、Channel、Sink，如下图：

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/Flume%E6%98%AF%E4%BB%80%E4%B9%882.jpg" alt="Flume是什么2" style="zoom: 67%;" />

当然，一台机器也可以配置多个Agent相互配合，架构如下图所示：

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/Flume%E6%98%AF%E4%BB%80%E4%B9%883.jpg" alt="Flume是什么3" style="zoom: 67%;" />





### Kafka集成Flink

---

#### Whats Flink

Apache Flink is a framework and **distributed processing engine** for **stateful** computations over ***unbounded and bounded* data streams**. Flink has been designed to run in *all common cluster environments*, perform computations at *in-memory speed* and at *any scale*.



![img](https://flink.apache.org/img/bounded-unbounded.png)



**Features:**

1. **Deploy Applications Anywhere**

> Flink integrates with all common cluster resource managers such as [Hadoop YARN](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/YARN.html), [Apache Mesos](https://mesos.apache.org), and [Kubernetes](https://kubernetes.io/) but can also be setup to run as a stand-alone cluster. Besides, Flink's communication to submit or control an application happens via REST calls, which is widely supported

2. **Run Applications at any Scale**

3. **Leverage In-Memory Performance**

> Flink guarantees exactly-once state consistency in case of failures by  periodically and asynchronously checkpointing the local state to durable storage.

<img src="https://flink.apache.org/img/local-state.png" alt="img"  />





### Kafka集成Spark

---

#### Whats **Spark**

Spark is an Unified engine for large-scale data analytics. It supports multi-language and can run on single-node machines or clusters





### Kafka生产调优

---

#### 生产者调优

通过参数来控制发送速率、批次大小、确认应答机制等等



#### Broker调优

Follower被踢出ISR的阈值实践 ，不要太长

再平衡

日志保存时间

删除策略

各组件的线程数量

手动调整broker的分区负载，一机一议

提前创建主题及其分区数等参数，避免使用系统默认的



#### 消费者调优



#### 总体调优策略

提高生产者的吞吐量

增加分区数

提高消费者的吞吐量

提高消费者下游的处理速率



推荐使用Kafka自带的压测脚本





### 面试常问

---

#### Kafka和RocketMQ的区别

Kafka的定位是流式数据处理，目标是高吞吐量。RocketMQ生而为互联网应用解决异步、削峰、解耦等问题，拥有更高的可靠性

- 数据吞吐量方面，Kafka远强于RocketMQ

  1、kafka单机写入TPS月在百万条/秒，消息大小为10个字节
   2、RocketMQ单机写入TPS单实例约7万条/秒，若单机部署3个broker，可以跑到最高12万条/秒，消息大小为10个字节

  主要原因是Kafka在Broker层面进行了消息合并，批量发送。而Rocket处于稳定性考虑，把消息的合并放到了业务层去做

  消息的合并批量发送同时也涉及到了消息的堆积，Rocket用的Java，消息堆积过多会带来GC的压力

- 数据可靠性方面，**Rocket采用同步刷盘**，而Kafka这是异步刷盘。Rocket最大限度地保证了机器Crash掉但是不会丢数据
- 队列数量方面，**Rocket支持上万队列**，并且推荐大集群运维，符合互联网的应用场景。相比Kafka最大推荐64个分区，在日志处理方面是够用了
- **Rocket支持消息消费失败后重试**，Kafka则不支持
- **Rocket保证严格的消息有序**，Kafka只保证在Partition内的消息有序
- 在互联网应用的角度上，Rocket有着如下的优势
  - 支持分布式事务
  - 支持消息的定时发送
  - 支持消息的查看和回溯



 

#### JMS模型与AMQP模型的区别

| 项目         | AMQP                                                         | JMS                                                          |
| ------------ | :----------------------------------------------------------- | ------------------------------------------------------------ |
| 定义         | 线级协议                                                     | Java API                                                     |
| 跨平台       | 是                                                           | 否                                                           |
| 跨语言       | 是                                                           | 否                                                           |
| 消息收发模型 | 4种消息收发模型：                                                               Direct Exchange<br />Fanout Exchange<br />Topic Exchange            <br />Header Exchange | 2种消息收发模型：<br />P2P<br />Pub/Sub                      |
| 消息类型     | 二进制数据类型                                               | 5种消息类型：                                                               <br />Text message<br />Object message     <br />Bytes message   <br />Stream message   <br /> Map message |
| 消息流       | Producer将消息发送到Exchange，Exchange将消息路由到Queue，Consumer从Queue中消费消息。 | Producer将消息发送到Queue或者Topic，Consumer从Queue或Topic中消费消息。 |



#### Kafka如何保证消息不丢失

消息在生产者、Broker、消费者三处都有可能丢失，那么需要在三处分别设置防丢才行

**Producer**

采用异步带回调函数的send方法send(msg, callback)，再设置参数为发送消息失败不停地重试。

<u>acks=all</u>，这个参数有可以配置0|1|all

<u>0</u>	表示生产者写入消息不管服务器的响应，可能消息还在网络缓冲区，服务器根本没有收到消息，当然会丢失消息

<u>1</u>	表示至少有一个副本收到消息才认为成功，一个副本那肯定就是集群的Leader副本了，但是如果刚好Leader副本所在的节点挂了，Follower没有同步这条消息，消息仍然丢失了

<u>all</u>	表示所有ISR都写入成功才算成功，那除非所有ISR里的副本全挂了，消息才会丢失

<u>retries=N，</u>设置一个非常大的值，可以让生产者发送消息失败后不停重试



**Broker**

kafka因为消息写入是通过PageCache异步写入磁盘的，在刷盘没完成之前内存中的数据都是存在丢失风险的

因此针对kafka自身丢失的可能设置参数：

<u>replication.factor=N，</u>设置一个比较大的值，保证至少有2个或者以上的副本

<u>min.insync.replicas=N，</u>代表消息如何才能被认为是写入成功，设置大于1的数，保证至少写入1个或者以上的副本才算写入消息成功

<u>unclean.leader.election.enable=false，</u>这个设置意味着没有完全同步的分区副本不能成为Leader副本，如果是true的话，那些没有完全同步Leader的副本成为Leader之后，就会有消息丢失的风险。



**Consumer**

改为手动提交OffSet即可，在代码逻辑中消费了消息之后再提交OffSet。



#### Kafka 消费者比分区数要多如何来处理

会有几个消费者不能正常消费到消息处于闲置状态

这种做的目的是为了保证有效的有序性，一个分区不能同时被多个消费者消费





#### Kafka为什么能做到高吞吐且快
