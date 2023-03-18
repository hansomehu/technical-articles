---

---



elk = elasticsearch + logstash + kibana



日志收集框架Flume、Logstash的对比

ELK日志框架生态

![image-20220803224953092](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220803224953092.png)

#### Flume

Flume是一种分布式、高可靠和高可用的服务，用于高效地收集、聚合和移动大量[日志数据](https://cloud.tencent.com/solution/cloudlog?from=10680)。它有一个简单而灵活的基于流数据流的体系结构。它具有可调的可靠性机制、故障转移和恢复机制，具有强大的容错能力。它使用一个简单的可扩展数据模型，允许在线分析应用程序。



#### Logstash

Logstash  是开源的服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到存储库中。数据从源传输到存储库的过程中，Logstash  过滤器能够解析各个事件，识别已命名的字段以构建结构，并将它们转换成通用格式，以便更轻松、更快速地分析和实现商业价值。



#### 二者相较

**Flume更注重于数据的传输，对于数据的预处理不如Logstash**。 在传输上Flume比Logstash更可靠一些，因为数据会持久化在channel中。 数据只有存储在sink端中，才会从channel中删除，这个过程是通过事物来控制的，保证了数据的可靠性

**Flume可靠性上强于Logstash，**要求数据严格不丢失的，必须采用Flume

**Flume的框架比较清晰，**比如source，channel，sink的概念，同时采用Java编写，背靠JVM调优便捷

**Logstash的插件丰富，**能扩展出多元化的应用

**Logstash紧贴elk生态，**和ES能完美配合



## Flume

Flume 是 Cloudera 提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传 输的系统。Flume 基于流式架构，灵活简单。

![image-20220714125106614](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714125106614.png)



Flume支持的消息格式只有txt，来源常用的就是日志输出；此外像文件系统的监控（文件的追加、目录下文件的新建）也相对常用。关键还是要掌握好**日志**输出进Flume进行过滤采集这块的内容





![image-20220714125412494](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714125412494.png)

在Flume的架构中，Source和Channel是多对多的关系，而Channel和Sink是一对一的关系

**Source的类型包括：**avro（Flume与Flume之间做连接使用）、thrift、exec、jms、spooling directory、netcat、taildir、 sequence generator、syslog、http、legacy。此外还可以通过基础AbstractSource自定义Source

**Channel的类型包括：**Memory Channel 和 File Channel

**Sink的类型包括：**hdfs、logger、avro、thrift、ipc、file、HBase、solr。Sink同样也可以自定义

**Event：**Flume数据传输的基本单位

![image-20220714125909942](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714125909942.png)

Header通常采用HashMap创建

Body则要注意String和Byte之间的转换



基本的使用都是通过配置文件的方式来完成的，涉及到需要自定义组件的时候则编写Java代码



#### 案例一：监听端口数据

```properties
# Name the components on this agent
# 定义组件名
a1.sources = r1
a1.sinks = k1
a1.channels = c1
# Describe/configure the source 
a1.sources.r1.type = netcat 
a1.sources.r1.bind = localhost 
a1.sources.r1.port = 44444
# Describe the sink
a1.sinks.k1.type = logger
# Use a channel which buffers events in memory 
a1.channels.c1.type = memory 
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100
# Bind the source and sink to the channel 
a1.sources.r1.channels = c1 
a1.sinks.k1.channel = c1
```



启动

```shell
bin/flume-ng agent 
# 表示配置文件存储在 conf/目录
--conf conf/ 
--name a1 
# 本次启动配置文件的读取地址
--conf-file job/flume-netcat-logger.conf 
- Dflume.root.logger=INFO,console
```



#### 案例二：监控一个日志文件，将更新的情况上传到HDFS

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714143544496.png" alt="image-20220714143544496" style="zoom:33%;" />

 

```properties
# Name the components on this agent 
a2.sources = r2
a2.sinks = k2
a2.channels = c2
# Describe/configure the source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /opt/module/hive/logs/hive.log
# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop102:9820/flume/%Y%m%d/%H #上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = logs-
#是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
#多少时间单位创建一个新的文件夹 
a2.sinks.k2.hdfs.roundValue = 1 
#重新定义时间单位 
a2.sinks.k2.hdfs.roundUnit = hour 
#是否使用本地时间戳 
a2.sinks.k2.hdfs.useLocalTimeStamp = true 
#积攒多少个 Event 才 flush 到 HDFS 一次 
a2.sinks.k2.hdfs.batchSize = 100 
#设置文件类型，可支持压缩 
a2.sinks.k2.hdfs.fileType = DataStream 
#多久生成一个新的文件 
a2.sinks.k2.hdfs.rollInterval = 60 
#设置每个文件的滚动大小 
a2.sinks.k2.hdfs.rollSize = 134217700 
#文件的滚动与 Event 数量无关 
a2.sinks.k2.hdfs.rollCount = 0
# Use a channel which buffers events in memory 
a2.channels.c2.type = memory a2.channels.c2.capacity = 1000 a2.channels.c2.transactionCapacity = 100
# Bind the source and sink to the channel 
a2.sources.r2.channels = c2 
a2.sinks.k2.channel = c2
```





#### 案例三：监控一个文件目录，时候更新文件到HDFS

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714154513378.png" alt="image-20220714154513378" style="zoom:50%;" />

在被监控的目录中，文件会被额外加上后缀标记。.Completed表示该文件已经上传至HDFS，.tmp表示该文件待上传

```properties
a3.sources = r3
a3.sinks = k3
a3.channels = c3
# Describe/configure the source 
a3.sources.r3.type = spooldir 
a3.sources.r3.spoolDir = /opt/module/flume/upload 
a3.sources.r3.fileSuffix = .COMPLETED 
a3.sources.r3.fileHeader = true
#忽略所有以.tmp 结尾的文件，不上传 
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)
# Describe the sink
a3.sinks.k3.type = hdfs 
a3.sinks.k3.hdfs.path = hdfs://hadoop102:9820/flume/upload/%Y%m%d/%H #上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload- 
#是否按照时间滚动文件夹 
a3.sinks.k3.hdfs.round = true 
#多少时间单位创建一个新的文件夹 
a3.sinks.k3.hdfs.roundValue = 1 
#重新定义时间单位 
a3.sinks.k3.hdfs.roundUnit = hour 
#是否使用本地时间戳 
a3.sinks.k3.hdfs.useLocalTimeStamp = true 
#积攒多少个 Event 才 flush 到 HDFS 一次 
a3.sinks.k3.hdfs.batchSize = 100
#设置文件类型，可支持压缩 
a3.sinks.k3.hdfs.fileType = DataStream 
#多久生成一个新的文件 
a3.sinks.k3.hdfs.rollInterval = 60 
#设置每个文件的滚动大小大概是 128M 
a3.sinks.k3.hdfs.rollSize = 134217700 
#文件的滚动与 Event 数量无关 
a3.sinks.k3.hdfs.rollCount = 0
# Use a channel which buffers events in memory 
a3.channels.c3.type = memory 
a3.channels.c3.capacity = 1000 
a3.channels.c3.transactionCapacity = 100
# Bind the source and sink to the channel 
a3.sources.r3.channels = c3 
a3.sinks.k3.channel = c3
```



### Flume架构原理



#### Flume事务

Flume的事务存在于两个阶段（Source的PUT和Sink的GET）

![image-20220714180155607](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714180155607.png)



#### Flume Agent（即整体架构）

![image-20220714180525189](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714180525189.png)

**1. ChannelSlector**

ChannelSelector 的作用就是选出 Event 将要被发往哪个 Channel。其共有两种类型， 分别是 Replicating(复制)和 Multiplexing(多路复用)。

**ReplicatingSelector** 会将同一个 Event 发往所有的 Channel，**Multiplexing** 会根据相 应的原则，将不同的 Event 发往不同的 Channel



**2. SinkProcessor**

SinkProcessor 共 有 三 种 类 型 ， 分 别 是 DefaultSinkProcessor 、 LoadBalancingSinkProcessor 和 FailoverSinkProcessor

**DefaultSinkProcessor** 对应的是单个的 Sink，**LoadBalancingSinkProcessor** 和 FailoverSinkProcessor对应的是Sink Group，LoadBalancingSinkProcessor可以实现负 载均衡的功能，**FailoverSinkProcessor** 可以错误恢复的功能（集群中某个Sink挂了会将其负载转移到其他Sink中去）。



#### Flume拓扑组合

**简单串联**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714182205909.png" alt="image-20220714182205909" style="zoom: 50%;" />



**多路复用**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714182235701.png" alt="image-20220714182235701" style="zoom: 50%;" />

![image-20220714204734889](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714204734889.png)

案例

```properties
# flume-file-flume.conf
# Name the components on this agent 
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1 c2
# 将数据流复制给所有 channel 
a1.sources.r1.selector.type = replicating
# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/module/hive/logs/hive.log 
a1.sources.r1.shell = /bin/bash -c
# Describe the sink
# sink端的avro是一个数据发送者 
a1.sinks.k1.type = avro 
a1.sinks.k1.hostname = hadoop102 
a1.sinks.k1.port = 4141
a1.sinks.k2.type = avro 
a1.sinks.k2.hostname = hadoop102 
a1.sinks.k2.port = 4142
# Describe the channel 
a1.channels.c1.type = memory 
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100
a1.channels.c2.type = memory 
a1.channels.c2.capacity = 1000 
a1.channels.c2.transactionCapacity = 100
# Bind the source and sink to the channel 
a1.sources.r1.channels = c1 c2 
a1.sinks.k1.channel = c1 
a1.sinks.k2.channel = c2


# flume-flume-hdfs.conf
# Name the components on this agent 
a2.sources = r1
a2.sinks = k1
a2.channels = c1
# Describe/configure the source # source端的avro是一个数据接收服务
a2.sources.r1.type = avro 
a2.sources.r1.bind = hadoop102 
a2.sources.r1.port = 4141
# Describe the sink
a2.sinks.k1.type = hdfs
a2.sinks.k1.hdfs.path = hdfs://hadoop102:9820/flume2/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k1.hdfs.filePrefix = flume2- 
#是否按照时间滚动文件夹 
a2.sinks.k1.hdfs.round = true 
#多少时间单位创建一个新的文件夹 
a2.sinks.k1.hdfs.roundValue = 1 
#重新定义时间单位 
a2.sinks.k1.hdfs.roundUnit = hour 
#是否使用本地时间戳 
a2.sinks.k1.hdfs.useLocalTimeStamp = true 
#积攒多少个 Event 才 flush 到 HDFS 一次 
a2.sinks.k1.hdfs.batchSize = 100 
#设置文件类型，可支持压缩
a2.sinks.k1.hdfs.fileType = DataStream 
#多久生成一个新的文件 
a2.sinks.k1.hdfs.rollInterval = 30 
#设置每个文件的滚动大小大概是 128M 
a2.sinks.k1.hdfs.rollSize = 134217700 
#文件的滚动与 Event 数量无关 
a2.sinks.k1.hdfs.rollCount = 0
# Describe the channel 
a2.channels.c1.type = memory 
a2.channels.c1.capacity = 1000 
a2.channels.c1.transactionCapacity = 100
# Bind the source and sink to the channel 
a2.sources.r1.channels = c1 
a2.sinks.k1.channel = c1


# flume-flume-dir.conf
# Name the components on this agent 
a3.sources = r1
a3.sinks = k1
a3.channels = c2
# Describe/configure the source 
a3.sources.r1.type = avro 
a3.sources.r1.bind = hadoop102 
a3.sources.r1.port = 4142
# Describe the sink
a3.sinks.k1.type = file_roll 
a3.sinks.k1.sink.directory = /opt/module/data/flume3
# Describe the channel 
a3.channels.c2.type = memory 
a3.channels.c2.capacity = 1000 
a3.channels.c2.transactionCapacity = 100
# Bind the source and sink to the channel 
a3.sources.r1.channels = c2 
a3.sinks.k1.channel = c2

# 启动时需要分别启动上面3个Flume进程
```



**负载均衡**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714182322538.png" alt="image-20220714182322538" style="zoom:50%;" />



**聚合**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714182414534.png" alt="image-20220714182414534" style="zoom:50%;" />



#### 自定义组件

**Interceptor的自定义**

案例需求：在控制台输出的日志中判断是否包含关键字，如果包含关键字走Channel_A，否则走Channel_B

![image-20220714235236123](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220714235236123.png)

```java
// 1. 实现接口
public class TypeInterceptor implements Interceptor
  
// 2. 实现方法，这里只展示核心逻辑
@Override
public Event intercept(Event event) {
//2.1.获取事件中的头信息
Map<String, String> headers = event.getHeaders();
//2.2.获取事件中的 body 信息
String body = new String(event.getBody());
//2.3.根据 body 中是否有"atguigu"来决定添加怎样的头信息 if (body.contains("atguigu")) {
//2.4.添加头信息
headers.put("type", "first"); } else {
//2.5.添加头信息
headers.put("type", "second"); }
   return event;
}
```



Flume的启动配置则根据拓扑图进行调配

```properties
# 采用了自定义之后的一些关键配置
a1.sources.r1.interceptors.i1.type = com.atguigu.flume.interceptor.CustomInterceptor$Builder 
a1.sources.r1.selector.type = multiplexing 
a1.sources.r1.selector.header = type 
a1.sources.r1.selector.mapping.first = c1 
a1.sources.r1.selector.mapping.second = c2
```



**Source的自定义**

案例需求：使用 flume 接收数据，并给每条数据添加前缀，输出到控制台。前缀可从 flume 配置文 件中配置

![image-20220715001434832](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220715001434832.png)

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220715001648093.png" alt="image-20220715001648093" style="zoom: 33%;" />



```java
// 关键代码

@Override
public Status process() throws EventDeliveryException {
try {
//创建事件头信息
HashMap<String, String> hearderMap = new HashMap<>(); //创建事件
SimpleEvent event = new SimpleEvent();
//循环封装事件
for (int i = 0; i < 5; i++) {
//给事件设置头信息 event.setHeaders(hearderMap); //给事件设置内容
event.setBody((field + i).getBytes()); //将事件写入 channel getChannelProcessor().processEvent(event); Thread.sleep(delay);
   }
} catch (Exception e) {
   e.printStackTrace();
  return Status.BACKOFF;
   }
   return Status.READY;
}
```



```properties
# conf
# Name the components on this agent 
a1.sources = r1
a1.sinks = k1
a1.channels = c1
# Describe/configure the source 
a1.sources.r1.type = com.atguigu.MySource 
a1.sources.r1.delay = 1000 
a1.sources.r1.field = atguigu
# Describe the sink
a1.sinks.k1.type = logger
# Use a channel which buffers events in memory 
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100
# Bind the source and sink to the channel 
a1.sources.r1.channels = c1 
a1.sinks.k1.channel = c1
```



**Sink的自定义**

Sink的自定义略微有些复杂，因为Sink是拉模式PULL，并且是批量进行的。这就使得Sink的运行过程是完全事务性的，一批中有一个Event失败，则该批中的Events全部回滚

案例需求：使用 flume 接收数据，并在 Sink 端给每条数据添加前缀和后缀，输出到控制台。前后 缀可在 flume 任务配置文件中配置

![image-20220715002416043](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220715002416043.png)



```java
// 关键代码
@Override
public Status process() throws EventDeliveryException {
//声明返回值状态信息 
  Status status;
//获取当前 Sink 绑定的 
  Channel Channel ch = getChannel();
//获取事务
Transaction txn = ch.getTransaction();
//声明事件 
  Event event;
//开启事务 
  txn.begin();
  //读取 Channel 中的事件，直到读取到事件结束循环 
  while (true) {
   event = ch.take();
   if (event != null) {
break; }
} try {
//处理事件(打印)
LOG.info(prefix + new String(event.getBody()) +
//事务提交 
         txn.commit();
status = Status.READY;
       } catch (Exception e) {
//遇到异常，事务回滚 
         txn.rollback();
status = Status.BACKOFF;
} finally {
//关闭事务
          txn.close();
       }
       return status;
   }

```

```properties
# Name the components on this agent 
a1.sources = r1
a1.sinks = k1
a1.channels = c1
# Describe/configure the source 
a1.sources.r1.type = netcat 
a1.sources.r1.bind = localhost 
a1.sources.r1.port = 44444
# Describe the sink
a1.sinks.k1.type = com.atguigu.MySink 
a1.sinks.k1.prefix = atguigu: 
a1.sinks.k1.suffix = :atguigu
# Use a channel which buffers events in memory 
a1.channels.c1.type = memory 
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100
# Bind the source and sink to the channel 
a1.sources.r1.channels = c1 
a1.sinks.k1.channel = c1
```











## Logstash



## 百万面试提问



#### 如何搭建一套海量日志系统

![img](https://static001.infoq.cn/resource/image/1d/8d/1d99a414f1f68fc44c1cd8ff1334a88d.jpg)

![img](https://static001.infoq.cn/resource/image/8f/8c/8f30ec5d9c9fc7f3921c020eb4e7288c.jpg)

1. 业务侧将消息以json形式写入日志
2. flume监听日志文件，增量推入kafka
3. kafka -> 批量写入es
4. java消费程序接收消息->清洗过滤->写入hdfs



1. 业务侧将消息以json形式写入日志
2. flume监听日志文件，增量推入kafka（多个消费者组订阅同一个Topic）
3. go消费程序接收消息->清洗过滤->批量写入es
4. java消费程序接收消息->清洗过滤->写入hdfs，不需要清洗的话直接flume到hdfs即可



Log_File ---> Flume ---> Kafka ---> Flink ---> HDFS/ES

当数据量巨大的时候，这条了路可以搞长一些，让一个组件专门负责一个任务。数据量洗哦啊可以直接Flume对接到ES，这样Flume这一个组件就承担了过滤清洗和发送。



Kafka发送数据到ES，可以通过**Kafka Connector**这个组件来完成

```properties
name=elasticsearch-sink
connector.class=io.confluent.connect.elasticsearch.ElasticsearchSinkConnector
tasks.max=1
#kafka主题名称，也是对应Elasticsearch索引名称
topics= test-elasticsearch-sink

key.ignore=true
#ES url信息
connection.url=http://110.18.6.20:9200
#ES type.name固定
type.name=kafka-connect
```



