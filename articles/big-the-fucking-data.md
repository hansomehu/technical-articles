---
layout: post
title: "大数据全家桶"
permalink: /big-the-fucking-data
---



# Hadoop Series

本章是关于Hadoop及其生态的内容





### Hadoop基本介绍

---

主要解决海量数据的存储和分析计算问题，这些功能由hadoop及其生态圈内的其他组件共同来完成，诸如Hive、HDFS等

 Doug Cutting为了解决海量数据的存储和检索难题，借鉴了Google的经验在Lucene基础上发展出来的大数据框架。Hadoop核心生态体系中包含三个组件，MapReduce计算组件、HBase数仓（对标谷歌的BigTable）、HDFS分布式文件存储（对标谷歌的GFS）

Apache 2006年成立

高可靠（多副本）、高可扩（动态扩展）、高效性（并行计算）、高容错性（自动将失败的任务重新分配）



**Hadoop的组成**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220522121758829.png" alt="image-20220522121758829" style="zoom: 25%;" />

Hadoop2.x提升了高内聚低耦合的思想，把计算和资源调度功能拆分开，引入了新的组件Yarn专门进行CPU和内存资源的调度，而MR则专注于计算。



**HDFS Hadoop Distributed File System**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220522134848495.png" alt="image-20220522134848495" style="zoom: 25%;" />

**Yarn架构概览**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220522141716347.png" alt="image-20220522141716347" style="zoom:25%;" />



**HDFS、MR、Yarn相互配合**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220522142235232.png" alt="image-20220522142235232" style="zoom:25%;" />



**大数据生态体系**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220522142740191.png" alt="image-20220522142740191" style="zoom: 25%;" />



**电商实时推荐系统流程**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220522143000132.png" alt="image-20220522143000132" style="zoom: 25%;" />



**Hadoop及其周边的安装与配置**

hadoop包下各文件夹的含义

(1) bin 目录:存放对 Hadoop 相关服务(hdfs，yarn，mapred)进行操作的脚本

(2)etc 目录:Hadoop 的配置文件目录，存放 Hadoop 的配置文件

(3) lib 目录:存放 Hadoop 的本地库(对数据进行压缩解压缩功能)

(4) sbin 目录:存放启动或停止 Hadoop 相关服务的脚本

(5) share 目录:存放 Hadoop 的依赖 jar 包、文档、和官方案例



### Hadoop安装及配置

---













### Hadoop HDFS

---

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220612121619835.png" alt="image-20220612121619835" style="zoom:40%;" />

#### 架构

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220611172306424.png" alt="image-20220611172306424" style="zoom: 50%;" />

**NameNode相当于Master，负责：**

(1)管理HDFS的名称空间; 

(2)配置副本策略;

 (3)管理数据块(Block)映射信息; 

(4)处理客户端读写请求

**DataNode相当于Worker，负责：**

(1)存储实际的数据块;

 (2)执行数据块的读/写操作

另外还有一个**SecondaryNameNode（2NN）**的存在，属于是NameNode的秘书，会备份NN的一部分数据，无法全备份。当NN宕机之后，2NN只能恢复部分数据，因而不是作为高可用角色而存在



#### pros

高容错，副本挂了自动恢复

大数据处理（磁盘大小和磁盘数量都可以很大）

可构建在廉价机器上

#### cons

不适合低延时数据查询

无法高效对大量小文件进行存储（因为会占用大量的NameNode、小文件的寻址时间超过读取时间 违背了hdfs设计原则）

不支持并发写入、不支持文件随机改（只能append）



#### 寻址传输时间问题



#### Shell命令（开发重点）



#### Java客户端集成（了解能这么做就行）

通过Java进行hdfs文件的CRUD

参数配置文件优先级hdfs-default.xml  ->  hdfs-site.xml



#### HDFS读写流程

**写数据流程**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220611224526205.png" alt="image-20220611224526205" style="zoom:50%;" />

**节点选择策略**



**机架感知**



**读数据流程**

核心关注问题是访问节点的距离和其负载能力

 <img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220611231019514.png" alt="image-20220611231019514" style="zoom:50%;" />



### Hadoop MapReduce

---

MapReduce 是一个***分布式运算程序编程框架***，是用户开发“基于 Hadoop 的数据分析 应用”的核心框架

MapReduce 核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的 分布式运算程序，并发运行在一个 Hadoop 集群上

用户编写计算业务代码，MapReduce自动完成底层的分布式计算

但是MapReduce的缺陷在于不能进行实时的数据处理，进行处理的数据必须要落盘在文件当中。像Spark这种流式框架则是应对源源不断的海量实时数据的处理



#### 序列化与反序列化(Hadoop语义下)

**Java中序列化的概念：**在网络中传输的时候需要将Java对象转换成字节码以适配对象在网络中传输，这个过程叫做序列化。将字节码转化为对象则是反序列化。为了能够原本地还原出Java对象，需要在字节码当中设置一些meta info以标注对象的相关信息。还有一个SerialVersionUID概念，相当于是一个check-sum，来校验反序列化的正确性

**Hadoop序列化：**写一个类实现Writable接口，必须有空参构造、write方法（序列化方法）、readFields方法（反序列化方法），并且字段的序列化和反序列化顺序必须相同



#### MapReduce架构

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220616102250542.png" alt="image-20220616102250542" style="zoom: 50%;" />

两个阶段MapTask、ReduceTask，两个阶段互不干扰。MapReduce 编程模型只能包含一个 Map 阶段和一个 Reduce 阶段，如果用户的业务逻辑非常复杂，那就只能多个 MapReduce 程序，串行运行

**MR的编程流程**：梳理业务逻辑 —> 明确Map和Reduce的K-V —> 进行逻辑代码的编写 —> Driver类的编写

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220616120023637.png" alt="image-20220616120023637" style="zoom: 50%;" />

MapTask 的并行度决定 Map 阶段的任务处理并发度，MapTask的并行度则取决于块的个数，块个数根本上取决于块大小。常用的块大小为128m，在大厂中性能极强的服务器块大小通常为256m。块大小和分块数量与性能上存在一个trade-off



#### Job提交流程

![image-20220616121808462](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220616121808462.png)

**waitForCompletion**

**JobSubmiter**

**Yarn调度**

**HDFS中查找文件**

**进行序列化并切片**

**将job相关参数写到文件**





#### **FileInputFormat**

MR的该组件负责对输入数据进行处理，其中包括了很重要的切片环节，即将一个很大的文件拆分为许多block-size大小的小型文件

**切片机制**

获取文件大小 —> 计算切片大小（=blocksize）在**computeSplitSize**方法中 —> 执行切片在**getSplit**中执行 —> 将切片信息写到一个切片规划文件当中去 —> 将切片规划提交到Yarn 

**切片大小计算公式**

Math.max(minSize, Math.min(maxSize, blockSize));

mapreduce.input.fileinputformat.split.minsize=1 默认值为1 

mapreduce.input.fileinputformat.split.maxsize= Long.MAXValue 默认值Long.MAXValue

因此，默认情况下，切片大小=blocksize。

**FileInputFormat实现类**

FileInputFormat 常见的接口实现类包括:TextInputFormat、KeyValueTextInputFormat、NLineInputFormat、CombineTextInputFormat 和自定义 InputFormat 等

**TextInputFormat**



**CombineTextInputFormat**



#### MapReduce工作流程

**Map&Shuffle阶段**

![image-20220616143815469](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220616143815469.png)

**Reduce阶段**

![image-20220616164253815](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220616164253815.png)



#### Shuffle机制

Map 方法之后，Reduce 方法之前的数据处理过程称之为 Shuffle

Shuffle机制的作用就是将Map阶段处理完的大批量数据进行分区、排序及合并

![image-20220616164502138](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220616164502138.png)



#### Partition分区

这里的分区指的是将最终的结果输出到哪几个文件当中去

先看看默认分区

```java
public int getPartition(K key, V value, int numReduceTasks) { 
  // 默认分区是根据key的hashCode对ReduceTasks个数取模得到的。用户没法控制哪个 key存储到哪个分区
  return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;

}
}
```

**自定义自己的分区器步骤：**

1. extends Partitioner<K, V>并重写getPartition方法
2. 在job中设置分区器的class
3. 在job中设置ReduceTasks数量（必须和你的分区器逻辑相匹配）

> 自定义了分区器一定要设置正确的ReduceTasks数量，否则报错或者达不到预期的效果。
>
> 例如逻辑中存在5个分区，如果只定义一个，则全部的数据都会去这个分区
>
> 如果定义超过5个，会产生多余个数的空文件
>
> 如果定义大于1个少与5个，会Exceptions，since the programme has no idea how to distribute those data



#### **WritableComparable** 排序

MapTask和ReduceTask都会进行默认的快速归并排序

MapTask的默认排序发生在环形缓冲区数据达到80%的时候

ReduceTask则是在磁盘上两个大文件进行合并的时候进行一次归并排序，此外当全部的数据都写到了磁盘中的时候也会进行一次归并排序



**MR中几个排序概念：**全排序（Reduce阶段最后的排序）、部分排序（map阶段的排序）、辅助排序（相当于SQL里面的where第二第三个字段）、二次排序（排序的规则不止一个）



**自定义排序**

实现**WritableComparable**接口并重写其**compareTo**方法





#### Combiner

Combiner和Reducer相比其区别在于，CB运行在Map节点上，将MapTask的结果先进行一轮汇总，在ReduceTask拉取数据的时候能够减少网络的IO

使用步骤：

1. 写一个类继承Reducer并重写其reduce方法
2. 在job中setCombiner



#### OutPutFormat

这个类用来定义结果输出到文件的具体逻辑，例如日志中大于20字符的日志输出到logA，否则输出到logB



#### MapTask工作机制

![image-20220617003350719](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220617003350719.png)

#### ReduceTask工作机制

![image-20220617003416601](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220617003416601.png)

#### MapReduce源码解读（关注重点方法）



#### Join应用

文档数据关联，相当于SQL里面的表联接

Join的业务执行可以放到Map或者是Reduce阶段进行，为了防止数据倾斜应该平衡好二者处理的数据量

**Map端进行Join操作**

在 Map 端缓存多张表，提前处理业务逻辑，这样增加 Map 端业务，减少 Reduce 端数据的压力，尽可能的减少数据倾斜。Map端进行Join适合用来关联表中**存在小表的场景**

单纯的Join操作如果用不到Reduce部分的功能的话，在Job中将setReduceTasks为0

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220617214136312.png" alt="image-20220617214136312" style="zoom: 50%;" />

**Reduce端进行Join操作**



#### ETL数据清洗

在运行核心业务 MapReduce 程序之前，往往要先对数据进行清洗，清理掉不符合用户 要求的数据。清理的过程往往只需要运行 Mapper 程序，不需要运行 Reduce 程序。



#### Compact应用

在需要大量IO的业务中可以考虑对数据/文件进行一个压缩来减少开销



#### MR开发总结

**输入数据接口InputFormat**

**业务逻辑处理接口Mapper**

Mapper父类的一些方法及其运行逻辑：

- map
- setup
- cleanup
- run
- Mapper



**Partitioner分区**

**Comparable排序**

**Combiner合并**

**业务逻辑处理接口Reducer**

**输出数据接口OutputFormat**



### Hadoop Yarn

---

Yarn 是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式 的操作系统平台，而 MapReduce 等运算程序则相当于运行于操作系统之上的应用程序。

YARN 主要由 ResourceManager、NodeManager、ApplicationMaster 和 Container 等组件构成。

![image-20220619190626567](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220619190626567.png)



ResourceManager、NodeManager这两个相对较好理解

AppMst对应**一个Map和一个Reduce**任务，为其分配运行所需的资源

Container则是 YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、 网络等

> 某一个Container中的AppMstr不一定只管理本Container内的App



#### Yarn工作流程

![image-20220619195224340](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220619195224340.png)



#### HDFS & MapReduce 流程

![image-20220619205352306](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220619205352306.png)



#### HDFS、YARN、MapReduce三者关系

![image-20220619211045565](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220619211045565.png)

**todo：**总结三者之间相互配合的流程







#### Yarn调度器及调度算法

Hadoop 作业调度器主要有三种：**FIFO**、容量**（Capacity Scheduler）**和公平**（Fair Scheduler）**。Apache Hadoop3.x默认的资源调度器是 **Capacity Scheduler**

##### **用在哪？**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220619212755365.png" alt="image-20220619212755365" style="zoom:33%;" />

当用户将MR程序（Job）所需的资源提交到hdfs成功后，申请运行**mrAppMaster**。这个时候在**RM**中会生成一个Task（其实就是一个Job），将该Task放入队列，等待**NodeManager**领取任务并创建容器



##### **FIFO**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220619213352724.png" alt="image-20220619213352724" style="zoom:60%;" />

落地十分简单，甚至不支持多队列，生产环境下几乎不用



**容量调度器(Capacity Scheduler)**

优先选择资源利用率大的队列（就是占总资源的比例少的）

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220619213558146.png" alt="image-20220619213558146" style="zoom:67%;" />

多队列、容量保证（多次分配算法，上下限设置）、灵活性、多租户（能为指定租户设定指定大小的容量）

容量调度器进行三层分配，按照队列、作业（Task Job）、容器的顺序进行资源的分配。有剩余的资源则一步步向上借出

队列资源分配：从root开始，使用**深度优先算法**，优先选择资源占用率最低的队列分配资源

作业资源分配：默认按照提交作业的**优先级**和提交**时间顺序**分配资源

容器资源分配：按照容器的优先级分配资源; 如果**优先级**相同，按照数据**本地性原则**



##### 公平调度器(**Fair Scheduler**)

跟容量调度器相比最大的区别在于，优先选择对资源的缺额比例大的；并且每个队列都可以单独设置资源分配的方式。

这里的公平意味着，如果一个队列中有两个应用程序同时运行，则每个应用程序可得到1/2的资源;如果三个应用程序同时运行，则每个应用程序可得到1/3的资源。分配的时候也是按照着三层从上到下的顺序来进行。如果某个单位存在资源剩余的情况，则向上借出，但具体借给谁则根据下面的一些值来决定（**根本还是优先为缺额大的作业分配资源**）。

公平调度中需要比较的一些值：

➢ 实际最小资源份额:mindshare = Min(资源需求量，配置的最小资源) 12

➢ 是否饥饿:isNeedy = 资源使用量 < mindshare(实际最小资源份额) 1 2

➢ 资源分配比:minShareRatio = 资源使用量 / Max(mindshare, 1) 

➢ 资源使用权重比:useToWeightRatio = 资源使用量 / 权重



#### Yarn常用命令

```shell
# 查看在线app列表（和web查看是一致的）
# Application-Id Application-Name Application-Type User Queue State Final-State Progress
# Tracking-URL
yarn application -list
# 指定状态
yarn application -list -appStates FINISHED
# 杀死指定app
yarn application -kill application_1612577921195_0001

# ------ 说清楚，一个application是一个代码层面的任务，会包含很多个container ------

# 查看日志 app级别或者Container级别
yarn logs -applicationId application_1612577921195_0001 -containerId container_1612577921195_0001_01_000001
# 查看尝试运行的任务
yarn applicationattempt -list application_1612577921195_0001
# 打印状态
yarn applicationattempt -status appattempt_1612577921195_0001_000001
# 列出一个MR应用的全部Container
yarn container -list <ApplicationAttemptId>
# 打印 Container 状态
yarn container -status <ContainerId>
# 查看节点状态
yarn node -list -all
# 查看队列
yarn queue -status <QueueName>
# 更新配置
yarn rmadmin -refreshQueues

```



#### Yarn生产核心参数

**yarn-site.xml**这里面修改配置文件

`yarn.resourcemanager.scheduler.client.thread-count` 线程数，通常服务器一个CPU开2个线程

`yarn.nodemanager.resource.count-logical-processors-as-cores`  是否开启虚拟cpu（常用于cpu性能相差较大的集群）

`yarn.nodemanager.resource.pcores-vcores-multiplier` CPU虚拟倍数

`yarn.nodemanager.resource.memory-mb NodeManager` 使用内存，默认8G

`yarn.scheduler.minimum-allocation-mb` 容器最最小内存，默认1G

`yarn.scheduler.maximum-allocation-mb`  容器最最大内存，默认8G

`yarn.scheduler.minimum-allocation-vcores` 容器最小CPU核数，默认1个

`yarn.scheduler.maximum-allocation-vcores`  容器最大CPU核数，默认4个





#### Tool接口的使用

Tool接口是为了解决MR程序当中把获取程序运行参数（文件的输入输出路径）的代码写死了的问题。当命令行配置了很多参数的时候path[0] path[1]这两个参数就不能正确地匹配到文件的输入、输出路径了。这个时候可以采取Tool接口来重构参数位置

##### 具体用法：

- 写一个类实现Tool接口

`public class WordCount implements Tool`

实现Tool接口的run、getConf、setConf方法

run：基本就是为Job设置参数进去

- 以内部类的形式编写Mapper和Reducer的业务逻辑
- 编写Driver类，执行ToolRunner类（该类会引入WordCount类）的Run方法





### 源码

---

#### Hadoop RPC 通信原理

传入Hadoop的RPC协议，客户端和服务端的IP及端口，构建好RPCBuilder；最后通过Server类的start( )方法来启动服务



#### NameNode启动流程

NN负责在HDFS中对集群中的全部文件进行管理，包括备份、节点的分配、日志记录等工作

其中的核心逻辑包括：记录和更新日志、生成fsimage镜像、与2NN进行镜像的同步

![image-20220620235434177](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220620235434177.png)

##### 全流程

![image-20220620235804185](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220620235804185.png)

以上流程简单的文字描述：

在**NameNode**类中进入其**main**方法，在其中会调用**createNameNode**方法

**startHttpServer**启动9870端口服务

**loadNamesystem**加载镜像文件和编辑日志

**startCommonServices** NN启动资源检查

在createNameNode中最后一步是new 一个 NameNode对象，该对象的**Initialize( )**方法调用

**createRpcServer**初始化NN的RPC服务端

 

#### DataNode启动流程

![image-20220621111855167](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220621111855167.png)

![image-20220621111927153](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220621111927153.png)



文字描述：

在**DataNode**类当中有一个**secureMain**方法，这个方法是DN启动的入口

在该方法内调用**createDataNode**方法，在其内调用**instantiateDataNode**方法，该方法按照如下流程来初始化DataNode：**makeInstance** —> **new DataNode** —> **startDataNode**。

在startDataNode方法中执行核心的初始化逻辑：**initDataXceiver**初始化**DataXceiverServer** —> **startInfoServer**初始化HTTP服务 —> **initIpcServer**初始化DN的 RPC服务端 —> **refreshNamenodes** DN向NN注册 —> **offerService**（sendHeartBeat 通过RPC发送 给NN）向NN发送心跳



#### HDFS上传源码

![image-20220621121505217](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220621121505217.png)

**ps** 客户端只同DN1建立通道，dn1-3之间通道的建立及后续的数据传递由dn1来负责

![image-20220621123530057](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220621123530057.png)







#### Yarn源码

![image-20220621140752933](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220621140752933.png)

##### 源码流程

![image-20220621141837039](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220621141837039.png)



#  Spark Series

Spark与Hadoop最大的区别就是，**Spark是基于内存的**，在性能有余的时候能够带来超强的性能

Spark在用户使用上区别于Hadoop最大的地方在于提供了MapReduce需要的很多High Level API，通过调用方法的方式就能实现一个MR程序

**Spark Core（基础、核心功能）、Spark SQL（SQL内容、与Hive结合）、Spark Streaming（流式计算）**是三个必须掌握的模块。像Spark MLlib（用于机器学习）、Spark GraphL（用于图计算）暂时不用学习



### **环境配置**

增加Scala插件，通过Idea的Plugins中心添加即可

Spark官方入门案例 https://spark.apache.org/examples.html



**并行度**   一个核一个任务；通常在Spark当中一个分区由一个核来进行执行

**并发度**  一个核执行多个程序



## Spark运行架构

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220623164004510.png" alt="image-20220623164004510" style="zoom:50%;" />



#### Driver

Spark 驱动器节点，用于执行 Spark 任务中的 main 方法，负责实际代码的协调指挥工作，所谓的 Driver 就是驱使整个应用运行起来的程序，也称之为 Driver 类。 Driver 在 Spark 作业执行时主要负责：

1. 将用户程序转化为作业(job)
2. 在Executor之间调度任务(task)
3. 跟踪Executor的执行情况



#### Executor

Spark Executor 是集群中工作节点(Worker)中的一个 JVM 进程，负责在 Spark 作业中运行具体任务(Task)，任务彼此之间相互独立。Spark 应用启动时，Executor 节点被同时启动，并且始终伴随着整个 Spark 应用的生命周期而存在。如果有 Executor 节点发生了 故障或崩溃，Spark 应用也可以继续执行，会将出错节点上的任务调度到其他 Executor 节点 上继续运行。正常情况下**一个CPU核对应一个Executor**

Executor 有两个核心功能:

1. 负责运行**组成Spark应用**的任务，并将结果返回给驱动器进程

2. 它们通过自身的块管理器(Block Manager)为用户程序中要求缓存的 RDD 提供内存式存储。**RDD 是直接缓存在 Executor 进程内的**，因此任务可以在运行时充分利用缓存 数据加速运算。



#### **Master & Worker**

Saprk是集计算和资源管理一体的框架，它的资源管理部分涉及到的核心组件就是Master 和 Worker，这里的 Master 是一个进程，主要负责资源的调度和分配，并进行集群的监控等职责，类似于 Yarn 环境中的 **RM**, 而 Worker 呢，也是进程，一个 Worker 运行在集群中的一台服务器上，由 Master 分配资源对 数据进行并行的处理和计算，类似于 Yarn 环境中 **NM**。

但是在实际的生产中通常用更加高效的Yarn或者Mesos来进行资源的调度，所以M&W用得并不多



#### **ApplicationMaster**

ResourceManager(资源)和 Driver(计算)之间的解耦合靠的就是 ApplicationMaster。一个Driver程序在创建任务的时候通过AM去向RM申请资源，在Hadoop中集群申请资源也是通过AppMstr来申请资源容器Container



#### 提交流程

基于**Yarn资源管理器**的提交流程

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220623171718434.png" alt="image-20220623171718434"  />

**ps** Yarn还有一个Client模式和Cluster模式的知识点，二者简单来说就是yarn是运行在本地还是集群的区别





## Spark核心编程



Spark 计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于 处理不同的应用场景。三大数据结构分别是：

1. **RDD**：弹性分布式数据集
2. **累加器**：分布式共享只写变量
3. **广播变量**：分布式共享只读变量



### RDD

RDD(Resilient Distributed Dataset)叫做弹性分布式数据集，是 Spark 中**最基本的**数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行 计算的集合

特性：

1. 弹性，表现在存储（内存与磁盘的自动切换）、容错（数据丢失自动恢复）、计算（计算出错重试）、分片（根据需要可重新分片）
2. 分布式
3. 数据集，RDD只保留了计算的逻辑并不复杂数据，具体的数据还是要去数据原本的内存地址获取
4. 数据抽象，RDD是一个抽象类，需要子类具体实现
5. 不可变，要一个新的计算逻辑还是得重新创建一个RDD
6. 支持分区、并行计算



##### RDD核心属性

**getPartitions**

**compute**

**getDependencies**

**partitioner**

**getPreferredLocations** 计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算



##### RDD执行原理

启动Yarn 确定RM和NM ——>

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220623175947400.png" alt="image-20220623175947400" style="zoom:25%;" />

 Spark 通过申请资源创建调度节点和计算节点(确定承载Driver和Executor的NM) ——> 

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220623175925391.png" alt="image-20220623175925391" style="zoom:25%;" />

Spark 框架根据需求将计算逻辑根据分区划分成不同的任务 ——> 

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220623175816603.png" alt="image-20220623175816603" style="zoom: 25%;" />

调度节点将任务根据计算节点状态发送到对应的计算节点进行计算 ——end



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220623175841442.png" alt="image-20220623175841442" style="zoom: 25%;" />





#### RDD的创建

四种创建方式：**集合**（内存）之间创建、从**外部文件**创建（支持HDFS、Hive等）、通过**其他的RDD**转化而来、直接**new一个RDD**（为Spark框架自身的保留用法）

```scala
// pre
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)

// external files
sparkContext.textFile(" addr ")
// memory
sparkContext.makeRDD( List(1,2,3,4) , [param : partitions] )
```



#### RDD转换算子

RDD 根据数据处理方式的不同将算子整体上分为 **Value** 类型、**双 Value** 类型和 **Key-Value** 类型

**Value类型 map**



**mapPartitions**



> map与mapPartitions的区别：Map 算子是分区内一个数据一个数据的执行，类似于串行操作。而 mapPartitions 算子 是以分区为单位进行批处理操作
>
> Map 算子主要目的将数据源中的数据进行转换和改变。但是不会减少或增多数据。 MapPartitions 算子需要传递一个迭代器，返回一个迭代器，没有要求的元素的个数保持不变， 所以可以增加或减少数据。这就说明mp可以进行filter操作
>
> 从性能的角度来看，Map 算子因为类似于串行操作，所以性能比较低，而是 mapPartitions 算子类似于批处理，所以性能较高。但是 mapPartitions 算子会长时间占用内存，所以需要trade-off思想来考虑选型

Spark分区规则和分区算法的源码

```java
// the rule of partitioning
def positions(length: Long, numSlices: Int): Iterator[(Int, Int)] = {
  (0 until numSlices).iterator.map { i =>
   val start = ((i * length) / numSlices).toInt
   val end = (((i + 1) * length) / numSlices).toInt
   (start, end)
}

// choose output partitions
protected long computeSplitSize(long goalSize, long minSize,
                               long blockSize) {
  return Math.max(minSize, Math.min(goalSize, blockSize));
}
```





**mapPartitionsWithIndex**



**flatMap**



**glom**

将同一个分区的数据直接转换为相同类型的内存数组进行处理，分区不变。

函数签名 **def glom(): RDD[Array[T]]**

```scala
val rddGlom: RDD[Array[Int]] = rdd.glom()
var sum = 0

// wrong example ...
rddGlom.foreach(
  sum += _.max
)
}
```



##### **groupBy**

将数据根据指定的规则进行分组, 分区默认不变，但是数据会被打乱重新组合，我们将这样 的操作称之为 shuffle。极限情况下，数据可能被分在同一个分区中

groupBy方法内部逻辑返回值就作为分组的依据

函数签名 **def groupBy[K (f: T => K)(implicit kt: ClassTag[K]): RDD[(K, Iterable[T])]**

```scala
val dataRDD = sparkContext.makeRDD(List(1,2,3,4),1)
val dataRDD1 = dataRDD.groupBy( _%2 )
```



##### **filter**

将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。 当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出现**数据倾斜**

函数签名 **def filter(f: T => Boolean): RDD[T]** 函数逻辑内boolean类型的返回值作为过滤的依据

```scala
val rddFiltered: RDD[Int] = rdd.filter(
    _ == 2
)
```



##### sample

根据指定的规则从数据集中抽取数据

def sample(withReplacement: Boolean,fraction: Double,seed: Long = Utils.random.nextLong): RDD[T]

```scala
val dataRDD = sparkContext.makeRDD(List(
   1,2,3,4
),1)
// 抽取数据不放回(伯努利算法)
// 伯努利算法:又叫 0、1 分布。例如扔硬币，要么正面，要么反面。
// 具体实现:根据种子和随机算法算出一个数和第二个参数设置几率比较，小于第二个参数要，大于不 要
// 第一个参数:抽取的数据是否放回，false:不放回
// 第二个参数:抽取的几率，范围在[0,1]之间,0:全不取;1:全取;
// 第三个参数:随机数种子
val dataRDD1 = dataRDD.sample(false, 0.5)
// 抽取数据放回(泊松算法)
// 第一个参数:抽取的数据是否放回，true:放回;false:不放回
// 第二个参数:重复数据的几率，范围大于等于 0.表示每一个元素被期望抽取到的次数
// 第三个参数:随机数种子
val dataRDD2 = dataRDD.sample(true, 2)
```



##### extincet

对数据进行去重

函数签名  **def distinct(numPartitions: Int -optional- )(implicit ord: Ordering[T] = null): RDD[T]**

```scala
// deduplicate in particular partition
val dataRDD2 = dataRDD.distinct(2)
```



##### **coalesce**

根据数据量**缩减分区**，用于大数据集过滤后，提高小数据集的执行效率当 spark 程序中，存在过多的小任务的时候，可以通过 coalesce 方法，收缩合并分区，减少分区的个数，减小任务调度成本

函数签名 

def coalesce ( **numPartitions: Int**, 

shuffle: Boolean = false,

 partitionCoalescer: Option[PartitionCoalescer] = Option.empty)

( implicit ord: Ordering[T] = null ) : RDD[T]

```scala
// reduce partitions to 2 
val dataRDD1 = dataRDD.coalesce(2)
```



##### **repartition**

重新分区；该操作内部其实执行的是 **coalesce** 操作，参数 shuffle 的默认值为 true。无论是将分区数多的 RDD 转换为分区数少的 RDD，还是将分区数少的 RDD 转换为分区数多的 RDD，repartition 操作都可以完成，因为无论如何都会经 shuffle 过程。

函数签名 **def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]**



##### **sortBy**

该操作用于排序数据。在排序之前，可以将数据通过 f 函数进行处理，之后按照 f 函数处理 的结果进行排序，**默认为升序排列**。排序后新产生的 RDD 的分区数与原 RDD 的分区数一 致。中间存在 shuffle 的过程

函数签名  def sortBy 「K」 ( f: 

**(T) => K,**

**ascending: Boolean = true,**
 **numPartitions: Int = this.partitions.length**

) (implicit ord: Ordering[K], ctag: ClassTag[K]): RDD[T]

```scala
val dataRDD1 = dataRDD.sortBy(num=>num, false, 4)
```



#### 双Value类型

##### **intersection**

求交集

```scala
val dataRDD = dataRDD1.intersection(dataRDD2)
```



##### Union

求并集

```scala
val dataRDD = dataRDD1.union(dataRDD2)
```





##### Subtract

以一个 RDD 元素为主，去除两个 RDD 中重复元素，将其他元素保留下来。求差集

```scala
val dataRDD = dataRDD1.subtract(dataRDD2)
```



##### zip

将两个 RDD 中的元素，以键值对的形式进行合并。其中，键值对中的 Key 为第 1 个 RDD 中的元素，Value 为第 2 个 RDD 中的相同位置的元素。

```scala
val dataRDD = dataRDD1.zip(dataRDD2)
```



#### 键值对类型K-V

##### **partitionBy**

函数签名   def partitionBy(partitioner: Partitioner): RDD[(K, V)]

按照传入的分区器进行重新分区



##### **reduceByKey**



##### **groupByKey**

> **groupByKey**与**reduceByKey**的区别
>
> 二者都有一个Shuffle的阶段，但是reduceByKey在Shuffle之前会进行一个combine聚合阶段，最后的数据会减少一部分，相对讲高效些。但是groupByKey则只是单纯分组
>
> reduceByKey 其实包含分组和聚合的功能。GroupByKey 只能分组，不能聚 合，所以在分组聚合的场合下，推荐使用 reduceByKey，如果仅仅是分组而不需要聚合。那 么还是只能使用 groupByKey



##### **aggregateByKey**

函数签名 

def aggregateByKey -U: ClassTag- (zeroValue: U)(seqOp: (U, V) => U,combOp: (U, U) => U): RDD[(K, U)]

两个参数列表  (zeroValue: U) 初始值，主要用于遇见第一个key的时候进行计算

and

 (seqOp: (U, V) => U,combOp: (U, U) => U)；第一个是分区内处理规则，第二个是分区间的处理规则



##### **foldByKey**



##### 上述几个ByKey类型算子的区别

上面谈到的几个算子的工作流程都是：先分区内计算 ——> 再分区间计算

并且最底层调用的方法都是**combineByKeyWithClassTag**，不同是在于传入的参数

**ReduceByKey：**相同 key 的第一个数据不进行任何计算，分区内和分区间计算规则相同。就是单纯的wordcount

**FoldByKey：**相同 key 的第一个数据和初始值进行分区内计算，分区内和分区间计算规则相同

**AggregateByKey：**相同 key 的第一个数据和初始值进行分区内计算，分区内和分区间计算规则可以不相同

**CombineByKey：**当计算时，发现数据结构不满足要求时，可以让第一个数据转换结构。分区内和分区间计算规则不相同



**sortByKey**

返回一个按照key排序的RDD，Key必须实现Ordered接口（scala中的介质）

```scala
val sortRDD1: RDD[(String, Int)] = dataRDD1.sortByKey(true)
val sortRDD1: RDD[(String, Int)] = dataRDD1.sortByKey(false)
```



##### join

def join -W- (other: RDD[(K, W)]): RDD[(K, (V, W))]

在类型为(K,V)和(K,W)的 RDD 上调用，返回一个相同 key 对应的所有元素连接在一起的 (K,(V,W))的 RDD

```scala
val rdd: RDD[(Int, String)] = sc.makeRDD(Array((1, "a"), (2, "b"), (3, "c")))
val rdd1: RDD[(Int, Int)] = sc.makeRDD(Array((1, 4), (2, 5), (3, 6)))
rdd.join(rdd1).collect().foreach(println)
```



##### **leftOuterJoin**

类似于SQL中的左外连接



##### **cogroup**

函数签名  def cogroup -W- (other: RDD[(K, W)]): RDD[(K, (Iterable[V], Iterable[W]))]

在类型为(K,V)和(K,W)的 RDD 上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的 RDD

Iterable<V>把同一个分区中的数据放入了一个迭代器

```scala
dataRDD1.cogroup(dataRDD2)
```



#### RDD行动算子action

RDD行动算子的结果不是RDD，而是普通的数组等类型

RDD行动算子是触发任务开始执行的算子



##### reduce

聚集 RDD 中的所有元素，先聚合分区内数据，再聚合分区间数据

函数签名   def reduce(f: (T, T) => T): T

```scala
// 聚合数据
val reduceResult: Int = rdd.reduce(_+_)
```



##### collect

函数签名  def collect(): Array[T]

在驱动程序中，以数组 Array 的形式返回数据集的所有元素

```scala
// 收集数据到 
Driver rdd.collect().foreach(println)
```



##### count

函数签名  def count(): Long

返回 RDD 中元素的个数

```scala
// 返回 RDD 中元素的个数
val countResult: Long = rdd.count()
```



##### first

返回第一个值 

函数签名   def first(): T



##### take

返回前n个元素   def take(num: Int): Array[T]



##### takeOrdered

返回该 RDD 排序后的前 n 个元素组成的数组

函数签名   def takeOrdered(num: Int)(implicit ord: Ordering[T]): Array[T]



##### **aggregate**

分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合

def aggregate -U: ClassTag- (zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U): U



**fold**  折叠操作，aggregate 的简化版操作，区内外操作一样

**countByKey**  统计每种 key 的个数

**save**  将数据保存到不同格式的文件中















#### **RDD** 序列化

从计算的角度，算子以外的代码都是在 **Driver** 端执行, 算子里面的代码都是在 **Executor** 端执行。那么在 scala 的函数式编程中，就会导致算子内经常会用到算子外的数据，这样就 形成了闭包的效果，如果使用的算子外的数据无法序列化，就意味着无法传值给 Executor 端执行，就会发生错误，<u>所以需要在执行任务计算前，检测闭包内的对象是否可以进行序列化</u>，这个操作我们称之为**闭包检测**。



#### **RDD** 依赖关系

**A ——> B ——> C ——> DataSource** 这种连续的依赖在Spark中叫做血缘关系

由于RDD中本身不存储数据，那么A如何根据”血缘“关系来推算出自己最终的输出应该是什么数据呢？

![image-20220625101247058](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220625101247058.png)



##### 宽窄依赖

**窄依赖**表示每一个父(上游)RDD 的 Partition 最多被子(下游)RDD 的一个 Partition 使用，窄依赖我们形象的比喻为独生子女。

```scala
// 宽窄依赖只能在类的程度定义，比较粗
class OneToOneDependency[T](rdd: RDD[T]) extends NarrowDependency[T](rdd)
```



**宽依赖**表示同一个父(上游)RDD 的 Partition 被多个子(下游)RDD 的 Partition 依赖，会 引起 Shuffle，总结:宽依赖我们形象的比喻为多生

```scala
class ShuffleDependency extends Dependency[Product2[K, V]]
```



##### RDD的DAG状态链阶段划分 stage

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220625101955209.png" alt="image-20220625101955209" style="zoom: 50%;" />



##### 任务划分

- Application：初始化一个SparkContext即生成一个Application;
- Job：一个Action算子就会生成一个Job;
- Stage：Stage等于宽依赖(ShuffleDependency)的个数加1;
- Task：一个Stage阶段中，最后一个RDD的分区个数就是Task的个数。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220625104738858.png" alt="image-20220625104738858" style="zoom: 50%;" />





#### **RD**D持久化

##### **RDD Cache** <u>缓存</u>

RDD 通过 Cache 或者 Persist 方法将前面的计算结果缓存，默认情况下会把数据以缓存 在 **JVM 的堆内存**中。但是并不是这两个方法被调用时立即缓存，而是**触发后面的 action 算子**时，该 RDD 将会被缓存在计算节点的内存中，并供后面重用

```scala
// 数据缓存
wordToOneRdd.cache()
// 可以更改存储级别
mapRdd.persist(StorageLevel.MEMORY_AND_DISK_2)
```



**缓存有可能丢失**，或者存储于内存的数据由于内存不足而被删除，RDD 的缓存容错机 制保证了即使缓存丢失也能保证计算的正确执行。通过基于 RDD 的一系列转换，丢失的数 据会被重算，由于 RDD 的各个 Partition 是相对独立的，因此只需要计算丢失的部分即可， 并不需要重算全部 Partition。



Spark 会自动对一些 **Shuffle 操作的中间数据做持久化**操作(比如:reduceByKey)。这样 做的目的是为了当一个节点 Shuffle 失败了避免重新计算整个输入。但是，在实际使用的时 候，如果想重用数据，仍然建议调用 persist 或 cache



##### **RDD CheckPoint** <u>检查点</u>

所谓的检查点其实就是通过**将 RDD 中间结果写入磁盘**。由于血缘依赖过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果检查点 之后有节点出现问题，可以从检查点开始重做血缘，减少了开销。 对 RDD 进行 checkpoint 操作并不会马上被执行，必须**执行 Action 操作才能触发**。

```scala
// 设置检查点路径 
sc.setCheckpointDir("./checkpoint1")
...
// 增加缓存,避免再重新跑一个 job 做 
checkpoint wordToOneRdd.cache()
// 数据检查点:针对 wordToOneRdd 做检查点计算 
wordToOneRdd.checkpoint()

```



##### 缓存和检查点区别

1. Cache 缓存只是将数据保存起来，不切断血缘依赖。Checkpoint 检查点切断血缘依赖。 

2. Cache 缓存的数据通常存储在磁盘、内存等地方，可靠性低。Checkpoint 的数据通常存 储在 HDFS 等容错、高可用的文件系统，可靠性高。
3. 建议对 checkpoint()的 RDD 使用 Cache 缓存，这样 checkpoint 的 job 只需从 Cache 缓存 中读取数据即可，否则需要再从头计算一次 RDD。因为checkpoint是由一个单独的job线程来做的，要么从缓存拿数据要么自己计算一遍



### 分区器

Spark 目前支持 Hash 分区和 Range 分区，和用户自定义分区。Hash 分区为当前的默认分区

**Hash分区**：对于给定的 key，计算其 hashCode,并除以分区个数取余

**Range 分区**：将一定范围内的数据映射到一个分区中，尽量保证每个分区数据均匀，而 且分区间有序

**自定义分区**：diy











### 累加器

累加器用来把 Executor 端变量信息聚合到 Driver 端。在 Driver 程序中定义的变量，在 Executor 端的每个 Task 都会得到这个变量的一份新的副本，每个 task 更新这些副本的值后， 传回 Driver 端进行 merge。



### 广播变量

广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个 或多个 Spark 操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表， 广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是 Spark 会为每个任务 分别发送。

```scala
// 声明广播变量
val broadcast: Broadcast[List[(String, Int)]] = sc.broadcast(list)
...
// 使用广播变量
for ((k, v) <- broadcast.value) { }
```





## Spark Streaming



### 基本概念

![image-20220630232737478](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220630232737478.png)

Spark Streaming支持的数据输入形式多样，除了图中标出的外，还支持TCP连接进行数据的输入



**Streaming中的数据对象DStream概念：**

和 Spark 基于 RDD 的概念很相似，Spark Streaming 使用离散化流(discretized stream)作为抽 象表示，叫作 DStream。DStream 是随时间推移而收到的数据的序列。在内部，每个时间区间收 到的数据都作为 RDD 存在，而 DStream 是由这些 RDD 所组成的序列(因此得名“离散化”)。所以 简单来将，DStream 就是对 RDD 在实时数据处理场景的一种封装。可以理解为DStream为RDD在时间轴上的积分



**Streaming架构：**

![image-20220630233424073](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220630233424073.png)

老版本的Streaming的接收器运行在一个独立的节点，该节点专门负责数据的接收和分发。由此带来的一个问题就是数据接收和处理（性能）的速度不匹配，为了解决这个问题Streaming引入了态配制参数“**spark.streaming.receiver.maxRate**”的值来实现，此举虽然可以通过限制接收速率，来适配当前 的处理能力，防止内存溢出，但也会引入其它问题。就是资源的闲置浪费

为了更好的协调数据接收速率与资源处理能力，1.5 版本开始 Spark Streaming 可以动态控制 数据接收速率来适配集群数据处理能力。**背压机制（即 Spark Streaming Backpressure）**: 根据 JobScheduler 反馈作业的执行信息来动态调整 Receiver 数据接收率





### DStream入门



#### DS执行流程

![image-20220701111321489](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220701111321489.png)

DS就是一段时间内的数据之和，体现出了Spark的特点（并非严格意义上的实时）

数据以一个批次一个批次地发到Spark Engine中去执行

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220701111434085.png" alt="image-20220701111434085" style="zoom: 50%;" />



#### DS的创建

可以通过使用 **ssc.queueStream(queueOfRDDs)**来创建 DStream，每一个推送到这个队列中的 RDD，都会作为一个 DStream 处理

```scala
object RDDStream {
  def main(args: Array[String]) {
//1.初始化 Spark 配置信息
val conf = new SparkConf().setMaster("local[*]").setAppName("RDDStream")
//2.初始化 SparkStreamingContext
val ssc = new StreamingContext(conf, Seconds(4))
//3.创建 RDD 队列
val rddQueue = new mutable.Queue[RDD[Int]]()
//4.创建 QueueInputDStream
val inputStream = ssc.queueStream(rddQueue,oneAtATime = false)
//5.处理队列中的 RDD 数据
val mappedStream = inputStream.map((_,1))
val reducedStream = mappedStream.reduceByKey(_ + _)
//6.打印结果 reducedStream.print()
//7.启动任务 ssc.start()
//8.循环创建并向 RDD 队列中放入 RDD for (i <- 1 to 5) {
     rddQueue += ssc.sparkContext.makeRDD(1 to 300, 10)
     Thread.sleep(2000)
   }
   ssc.awaitTermination()
} }
```



#### 自定义数据接收器

自定义的数据接收器需要继承 Receiver，并实现 onStart、onStop 方法来自定义数据源采集

```scala
// 自定义数据接收器
class CustomerReceiver(host: String, port: Int) extends
Receiver[String](StorageLevel.MEMORY_ONLY) {
  //最初启动的时候，调用该方法，作用为:读数据并将数据发送给 Spark 
  override def onStart(): Unit = {
  new Thread("Socket Receiver") {
   override def run() {
receive() }
}.start() }
//读数据并将数据发送给 Spark 
def receive(): Unit = {
//创建一个 Socket
var socket: Socket = new Socket(host, port)
//定义一个变量，用来接收端口传过来的数据 
var input: String = null
//创建一个 BufferedReader 用于读取端口传来的数据
   val reader = new BufferedReader(new InputStreamReader(socket.getInputStream,
StandardCharsets.UTF_8))
//读取数据
input = reader.readLine()
//当 receiver 没有关闭并且输入数据不为空，则循环发送数据给 Spark 
while (!isStopped() && input != null) {
  // store方法保存input的内容 在调用该接收器时获取到保存的数据内容
     store(input)
     input = reader.readLine()
   }
//跳出循环则关闭资源 
reader.close() socket.close()
//重启任务
   restart("restart")
  }
  override def onStop(): Unit = {}
}

// 接收器的使用
object FileStream{
  
  ...
  
  //3.创建自定义 receiver 的 Streaming
val lineStream = ssc.receiverStream(new CustomerReceiver("hadoop102", 9999))
  
  ...
  
}
```

**以Kafka作为数据源**

Kafka作为数据源存在两种速率控制的方式，ReceiverAPI和DirectAPI方式

**ReceiverAPI：**需要一个专门的 Executor 去接收数据，然后发送给其他的 Executor 做计算。存在 的问题，接收数据的 Executor 和计算的 Executor 速度会有所不同，特别在接收数据的 Executor 速度大于计算的 Executor 速度，会导致计算数据的节点内存溢出。早期版本中提供此方式，当 前版本不适用

**DirectAPI：**是由计算的 Executor 来主动消费 Kafka 的数据，速度由自身控制。



### DStream的转换

DS和RDD一样存在丰富的转换函数

##### Transform

Transform 允许 DStream 上执行任意的 RDD-to-RDD 函数。即使这些函数并没有在 DStream 的 API 中暴露出来，通过该函数可以方便的扩展 Spark API。该函数每一批次调度一次（map则是一个Job只执行一次）。其实也就是对 DStream 中的 RDD 应用转换。



##### Join

两个流之间的 join 需要两个流的批次大小一致，这样才能做到同时触发计算。计算过程就是 对当前批次的两个流中各自的 RDD 进行 join，与两个 RDD 的 join 效果相同。

(K, V1) join (K, V2)  =>  ( K, (V1, V2))



##### **UpdateStateByKey**

UpdateStateByKey 原语用于记录历史记录，有时，我们需要在 DStream 中跨批次维护状态（例如流计算中累加 wordcount）。updateStateByKey()为我们提供了对一个状态变量的访问，用于键值对形式的 DStream。给定一个由**（键，事件）对构成的 DStream**，并传递一个指 定如何根据新的事件更新每个键对应状态的函数，它可以构建出一个新的 DStream，**其内部数据为（键，状态）对**。

updateStateByKey 操作使得我们可以在用新信息进行更新时保持任意的状态。为使用这个功 能，需要做下面两步:

1. 定义状态，状态可以是一个任意的数据类型。

2. 定义状态更新函数，用此函数阐明如何使用之前的状态和来自输入流的新值对状态进行更 新。

实例代码

```scala
object WorldCount {
  def main(args: Array[String]) {
// 定义更新状态方法，参数 values 为当前批次单词频度，state 为以往批次单词频度 
    val updateFunc = (values: Seq[Int], state: Option[Int]) => {
    	val currentCount = values.foldLeft(0)(_ + _)
      val previousCount = state.getOrElse(0)
     	sum(currentCount + previousCount)
}
    
 // 使用该状态对
  ...
 // Count each word in each batch
  val pairs = words.map(word => (word, 1))
// 使用 updateStateByKey 来更新状态，统计从运行开始以来单词总的次数 
  val stateDstream = pairs.updateStateByKey[Int](updateFunc)
  stateDstream.print()
	...

```



##### WindowOperations

所有基于窗口的操作都需要两个参数，分别为窗口时长以及滑动步长。常用来计算一个特定时间范围内的累计值。

**reduceByKeyAndWindow**

```scala
// Split each line into words
   val words = lines.flatMap(_.split(" "))
// Count each word in each batch
   val pairs = words.map(word => (word, 1))
   val wordCounts = pairs.reduceByKeyAndWindow((a:Int,b:Int) => (a +
b),Seconds(12), Seconds(6))
   // Print the first ten elements of each RDD generated in this DStream to the
	//console
   wordCounts.print()
```



**其他窗口相关函数**

1. window(windowLength, slideInterval): 基于对源 DStream 窗化的批次进行计算返回一个

新的 Dstream;

2. countByWindow(windowLength, slideInterval): 返回一个滑动窗口计数流中的元素个数;
3. reduceByWindow(func, windowLength, slideInterval): 通过使用自定义函数整合滑动区间流元素来创建一个新的单元素流;
4. reduceByKeyAndWindow(func, windowLength, slideInterval, [numTasks]): 当在一个(K,V)对的 DStream 上调用此函数，会返回一个新(K,V)对的 DStream，此处通过对滑动窗口中批次数 据使用 reduce 函数来整合每个 key 的 value 值。
5. reduceByKeyAndWindow(func, invFunc, windowLength, slideInterval, [numTasks]): 这个函 数是上述函数的变化版本，每个窗口的 reduce 值都是通过用前一个窗的 reduce 值来递增计算。 通过 reduce 进入到滑动窗口数据并”反向 reduce”离开窗口的旧数据来实现这个操作。<u>一个例子是随着窗口滑动对 keys 的“加”“减”计数。</u>   这个比较抽象



### DStream输出

输出操作指定了对流数据经转化操作得到的数据所要执行的操作(例如把结果推入外部数据库 或输出到屏幕上)。与 RDD 中的惰性求值类似（例如collect操作是驱动执行的事件），如果一个 DStream 及其派生出的 DStream 都没 有被执行输出操作，那么这些 DStream 就都不会被求值（输出事件就是DStream的驱动事件）。如果 StreamingContext 中没有设定输出操作，整个 context 就都不会启动。

- print():在运行流程序的驱动结点上打印DStream中每一批次数据的最开始10个元素。这

  用于开发和调试。在 Python API 中，同样的操作叫 print()。

- saveAsTextFiles(prefix, [suffix]):以 text 文件形式存储这个 DStream 的内容。每一批次的存

  储文件名基于参数中的 prefix 和 suffix。”prefix-Time_IN_MS[.suffix]”。

- saveAsObjectFiles(prefix, [suffix]):以 Java 对象序列化的方式将 Stream 中的数据保存为

  SequenceFiles . 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]". Python

  中目前不可用。

- saveAsHadoopFiles(prefix, [suffix]):将 Stream 中的数据保存为 Hadoop files. 每一批次的存

  储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]"。Python API 中目前不可用。

- foreachRDD(func):这是最通用的输出操作，即将函数 func 用于产生于 stream 的每一个

  RDD。其中参数传入的函数 func 应该实现将每一个 RDD 中数据推送到外部系统，如将

  RDD 存入文件或者通过网络将其写入数据库。

**一些开发注意事项：**

1. 连接不能写在 driver 层面（序列化）
2. 如果写在 foreach 则每个 RDD 中的每一条数据都创建，得不偿失
3. 增加 foreachPartition，在分区创建（获取）



### 优雅关闭

流式任务需要 7*24 小时执行，但是有时涉及到升级代码需要主动停止程序，但是分布式程序，没办法做到一个个进程去杀死，所有配置优雅的关闭就显得至关重要了。 使用外部文件系统来控制内部程序关闭。

示例代码

```scala
// 系统监控器，随着SparkStreaming启动一道启动
class MonitorStop(ssc: StreamingContext) extends Runnable {
  override def run(): Unit = {
   val fs: FileSystem = FileSystem.get(new URI("hdfs://linux1:9000"), new
Configuration(), "atguigu")
   while (true) {
     try
       Thread.sleep(5000)
     catch {
       case e: InterruptedException =>
        e.printStackTrace()
     }
     val state: StreamingContextState = ssc.getState
     val bool: Boolean = fs.exists(new Path("hdfs://linux1:9000/stopSpark"))
     if (bool) {
       if (state == StreamingContextState.ACTIVE) {
        ssc.stop(stopSparkContext = true, stopGracefully = true)
        System.exit(0)
       }
} }
}
}

// main
def main{
  ...
  
  new Thread(new MonitorStop(ssc)).start()
	ssc.start()
  ssc.awaitTermination()
  
  ...
}
```



### 案例实操

前往IDEA查看代码



























# Hive Series

### Hive基本概念

Hive 是基于 Hadoop 的一个数据仓库工具，可以将结构化的数据文件映射为一张表（一张表在HDFS中属于一个文件目录，该目录下的全部数据文件都属于这张表中的数据；例如一次MR操作之后有四个分区文件产生，这四个分区文件的数据格式完全相同），并提供类 SQL 查询功能

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220630165245097.png" alt="image-20220630165245097" style="zoom: 50%;" />



Hive的场景局限性：Hive 的执行延迟比较高，因此 Hive 常用于数据分析，对实时性要求不高的场合。Hive 优势在于处理大数据，对于处理小数据没有优势，因为 Hive 的执行延迟比较高。此外，HQL还存在表达能力有限，迭代式算法无法表达，调优效率较低等缺陷。总结来说，Hive只合适做回顾性的数据分析和挖掘



#### Hive架构

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220630165714793.png" alt="image-20220630165714793" style="zoom:50%;" />

**元数据：Metastore**
 元数据包括:表名、表所属的数据库(默认是 default)、表的拥有者、列/分区字段、表的类型(是否是外部表)、表的数据所在目录等。元数据是一系列的表，每个表中存储着一个业务信息，例如tbls存储着各个数据表的基本信息，诸如表中数据的行数（rows_num）

**驱动器：Driver**

(1)解析器(SQL Parser):将 SQL 字符串转换成抽象语法树 AST，这一步一般都用第 三方工具库完成，比如 antlr;对 AST 进行语法分析，比如表是否存在、字段是否存在、SQL 语义是否有误

(2)编译器(Physical Plan):将 AST 编译生成逻辑执行计划

(3)优化器(Query Optimizer):对逻辑执行计划进行优化。 (4)执行器(Execution):把逻辑执行计划转换成可以运行的物理计划（对于 Hive 来说，就是 MR/Spark）



#### 运行机制

![image-20220630170428172](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220630170428172.png)

Hive 通过给用户提供的一系列交互接口，接收到用户的指令(SQL)，使用自己的 Driver， 结合元数据(MetaStore)，将这些指令翻译成 MapReduce，提交到 Hadoop 中执行，最后，将执行返回的结果输出到用户交互接口。



由于 Hive 是针对数据仓库应用设计的，而数据仓库的内容是读多写少的。因此，**Hive中不建议对数据的改写**，所有的数据都是在加载的时候确定好的。

Hive 在查询数据的时候，由于没有索引，需要扫描整个表，因此**延迟较高**。另外一个导 致Hive 执行延迟高的因素是 MapReduce 框架。由于 MapReduce 本身具有较高的延迟，因此 在利用 MapReduce 执行 Hive 查询时，也会有较高的延迟。和数据库进行比较的话，Hive的性能优势体现在巨量数据时数据库系统难以承受压力，但是Hive确依旧坚挺



#### Hive的访问方式

使用元数据服务进行访问

使用JDBC进行访问  依赖于hiveserver2客户端

 

#### Hive配置文件

配置文件、命令行参数、参数声明（在 HQL 中使用 SET 关键字设定参数）

Hive的默认配置文件:hive-default.xml，用户自定义配置文件:hive-site.xml。注意，用户自定义配置会覆盖默认配置。另外，Hive 也会读入 Hadoop 的配置，因为 Hive是作为 Hadoop 的客户端启动的，Hive 的配置会覆盖 Hadoop 的配置。配置文件的设定对本机启动的所有 Hive 进程都有效

自定义的hive-site.xml  > 默认的hive-default.xml > Hadoop配置



### Hive数据类型及DDL操作

#### 基本数据类型

![image-20220630174954831](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220630174954831.png)

数据类型转换上，HQL支持隐式地向下转换。强制转换需要使用CAST函数，例如CAST('1' AS INT)，但是强制转换也需要逻辑上合理



#### 集合数据类型

![image-20220630175022024](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220630175022024.png)

Struct类型相当于Java中的Bean



##### 数据类型小案例

```sql
# 表结构创建
create table test(
name string,
friends array<string>,
children map<string, int>,
address struct<street:string, city:string>
)
# 定义数据的分隔符
row format delimited fields terminated by ','
collection items terminated by '_'
map keys terminated by ':'
lines terminated by '\n';

# 导入数据文件
load data local inpath '/opt/module/hive/datas/test.txt' into table test;

# 示例hql
hive (default)> select friends[1],children['xiao song'],address.city from
test
where name="songsong";
```

```
// 本地测试文件txt
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long
guan_beijing
yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing

// 一行数据以JSON格式的展示
{
"name": "songsong",
"friends": ["bingbing" , "lili"] , //列表 Array,
   "children": {
       "xiao song": 18 ,
       "xiaoxiao song": 19
   }
   "address": {
       "street": "hui long guan",
       "city": "beijing"
   }
}
```



#### DDL数据定义



##### 管理表与外部表

默认创建的表都是所谓的管理表，有时也被称为内部表。因为这种表，Hive 会(或多或少地)控制着数据的生命周期。Hive 默认情况下会将这些表的数据存储在由配置项hive.metastore.warehouse.dir(例如，/user/hive/warehouse)所定义的目录的子目录下。**当我们删除一个管理表时，Hive 也会删除这个表中数据**。管理表不适合和其他工具共享数据。当需要多源用户/业务共享一个数据表的时候应当把表的类型设置为**外部表（External Table）**。外部表的理论思想就是，Hive认为自己并非完全拥有这张表，故对该的数据操作会规避delete all等概念。

修改表的外部性

```sql
# 注意:('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写!
alter table <table_name> set tblproperties('EXTERNAL'='TRUE');
```





##### 创建表

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[(col_name data_type [COMMENT col_comment], ...)]
[COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
[CLUSTERED BY (col_name, col_name, ...)
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
[ROW FORMAT row_format]
[STORED AS file_format]
[LOCATION hdfs_path]
[TBLPROPERTIES (property_name=property_value, ...)]
[AS select_statement]
```



##### 修改表

```sql
# 更新列
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name
column_type [COMMENT col_comment] [FIRST|AFTER column_name]

# 增加和替换列
# 针对删除列这种操作，只能巧用replace
ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT
col_comment], ...)
```



附：[Hive中表与数据文件映射的流程](https://blog.csdn.net/xxydzyr/article/details/100915053)



### DML

#### 数据写入

##### load

向表中装载数据

put增加数据时count * 操作不会走MR，不会更改元数据 numRows、numFiles还是0

put和hdfs没什么关系，load则是操作hdfs

load操作时候hdfs文件（非local模式）速度很快，因为不涉及到数据的拷贝，只是将元数据中的地址和其他信息进行了一个更改

```sql
load data [local] inpath 'data path' [overwrite] into table student [partition (partcol1=val1,...)];
# local:表示从本地加载数据到 hive 表;否则从 HDFS 加载数据到 hive 表
```



##### insert

通常从一个表A中查出一些数据放到表B中的

```sql
insert [overwrite] into table_B
Select * from table_A where ...

# select twice
from table_B
# insert into partitions or tables, canot be same one
insert into table_A
select ...
insert into table_B/table_A(partition = "xxx")
select ...
```

```sql
# as ... select
create table if not exsit table_name
as
select * from table_A
```

##### location关键字

```sql
# location 建表时指定数据文件的地址（当数据文件没有放在table_name目录下的时候适用）
# 通常用在外部表
create external table table_A location 'file path'
```



#### 数据导出

##### Insert

```sql
# 导到hdfs上则不加local
insert overwrite local directory
'/opt/module/hive/data/export/student1'
# 不需要格式化则不加
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
select * from student;
```



##### HDFS命令导出

```
dfs -get 'orig/example.txt' 'dest/example.txt' 
```



### 数据查询

这部分内容大都是常规SQL

常用函数 count、max、min、sum、avg

limit、where；比较运算Between/In/ Is Null；like、rlike（like和Java正则表达式的结合）

逻辑运算and/or/not

... 下面就重点列出HQL中所独有的

**sort by**      每个reduce内部排序

**distribute by**      控制某个特定行应该到哪个 reducer

```sql
insert overwrite local directory
'/opt/module/data/distribute-result' select * from emp distribute by
deptno sort by empno desc;
```

**cluster by**      当 distribute by 和 sorts by 字段相同时，可以使用 cluster by 方式



### 分区和分桶

分区表实际上就是对应一个 HDFS 文件系统上的**独立的文件夹，**该文件夹下是该分区所有的数据文件。Hive 中的分区就是分目录，把一个大的数据集根据业务需要分割成小的数据 集。在查询时通过 WHERE 子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。

**单分区**

**二级分区**

**分区表与数据产生关联的三种方式：**上传数据后修复、上传数据后添加分区、创建文件夹后load数据到分区



#### 动态分区

关系型数据库中，对分区表 Insert 数据时候，数据库自动会根据分区字段的值，将数据 插入到相应的分区中



#### 分桶表

分区提供一个隔离数据和优化查询的便利方式。不过，并非所有的数据集都可形成合理的分区。对于一张表或者分区，Hive 可以进一步组织成桶，也就是**更为细粒度**的数据范围划分。



Hive 的分桶采用对分桶字段的值进行哈希，然后除以桶的个数求余的方式决定该条记录存放在哪个桶当中

使用分桶表时reduce 的个数设置为-1,让 Job 自行决定需要用多少个 reduce 或者将 reduce 的个 数设置为大于等于分桶表的桶数

同时建议从hdfs中load数据，以免找不到数据



创建桶的示例代码

```sql
create table stu_buck(id int, name string)
clustered by(id)
into 4 buckets
row format delimited fields terminated by '\t';

# insert为桶表添加数据
insert into table stu_buck select * from student_insert;
```



##### 抽样查询

抽样查询基于分桶表来进行

TABLESAMPLE(BUCKET x OUT OF y)





### 函数

查询所有函数

show functions

desc function [extended] func_name



函数根据输入输出数据的行数进行分类

UDF 一进一出    这类函数可以随意嵌套

UDAF 多进一出

UDTF 一进多出



**case when then else end** 语法的用法

这种场景也可以用if三元表达式来处理 **if (expr1, expr2, expr3)**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220703224401382.png" alt="image-20220703224401382" style="zoom: 25%;" />



Join语句 左右内外连接



排序 

order by 全局排序 只有一个reducer（默认禁止）

sort by 区内排序  

分区表中，分区字段在mysql元数据中，普通数据在文件当中



行转列、列转行

更好地理解方式就是多行变一行或者一行变多行的区别

concat拼接，主要可以拼接表的字段

concat_ws (separator, str1 ...)

也支持放字符串类型的数组



collect_list/collect_set聚合函数  将多行合成一行 

select collect_list/collect_set(field_name) from table_name





explode  将一列复杂的array或者map拆分成多行  udtf类的函数

案例场景：电影分类由原先的一对多平化为一对一





窗口函数 

OVER  指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变。用窗口函数就必须用到over 

LAG(col,n,default_val):往前第 n 行数据

LEAD(col,n, default_val):往后第 n 行数据 

NTILE(n)：把有序窗口的行分发到指定数据的组中，各个组有编号，编号从 1 开始，对于每一行，NTILE 返回此行所属的组的编号。发到n个7-=组中

> 需求案例：前20%时间的订单信息
>
> ```sql
> select * from (
>    select name,orderdate,cost, ntile(5) over(order by orderdate) sorted
>    from business
> )t
> where sorted = 1;
> ```





排名函数

RANK

DENSE_RANK

ROW_NUMBER

> ```sql
> select
> name
> (
>   select
>   *,
> 	rank() over( partition by subject order by score ) rk
> 	from score) t1
> 	where rk <=3;
> ```
>
> 三种排名函数的区别：rank-668，dense_rank-667，row-678



不用reducer的时候，map阶段经过环形缓冲区之后值是反的



集合操作

size 返回集合中元素个数

map_keys

map_values

array_contains

sort_array 





自定义UDF和UDFT函数

继承GenericUDF/UDTF，然后load function进Hive即可使用

`create temporary function func_name as "jar.package.name"`



grouping sets

自动union

grouping sets((a,b), a) => group by (a,b) **union** group by a

```sql
# example
select gender,deptId,count(*) from staff group by name,deptId grouping sets((gender, deptId),gender, deptId,())
```









### 综合大案例

*tips：比较复杂的SQL查询可以拆分为多个子查询



```sql
# gulivideo_orc表结构 HQL
create table gulivideo_ori(
videoId string,
uploader string,
age int,
category array<string>,
length int,
views int,
rate float,
ratings int,
comments int,
relatedId array<string>)
row format delimited fields terminated by "\t"
collection items terminated by "&"
stored as textfile;

# user tbl
create table gulivideo_user_ori(
uploader string,
videos int,
friends int)
row format delimited
fields terminated by "\t"
stored as textfile;
```



**需求一：视频播放数量top10**

```sql
SELECT
    videoId,
views FROM
    gulivideo_orc
ORDER BY
    views DESC
LIMIT 10;
```



**需求二：视频类别热度top10（该类别下视频数量最高）**

(1)即统计每个类别有多少个视频，显示出包含视频最多的前 10 个类别。

 (2)我们需要按照类别 group by 聚合，然后 count 组内的 videoId 个数即可。

 (3)因为当前表结构为:一个视频对应一个或多个类别。所以如果要 group by 类别，需要先将类别进行列转行(展开)，然后再进行 count 即可。

 (4)最后按照热度排序，显示前 10 条。

```sql
SELECT
   t1.category_name ,
   COUNT(t1.videoId) hot
FROM ( SELECT
videoId,
   category_name
FROM
   gulivideo_orc
   lateral VIEW explode(category) gulivideo_orc_tmp AS category_name
) t1
GROUP BY
   t1.category_name
ORDER BY hot
DESC LIMIT 10
```



**需求三：统计视频观看数top20的视频所对应的类别，以及这些类别下各有多少视频处于播放top20当中**

(1)先找到观看数最高的 20 个视频所属条目的所有信息，降序排列

 (2)把这 20 条信息中的 category 分裂出来(列转行) 

(3)最后查询视频分类名称和该分类下有多少个 Top20 的视频

```sql
SELECT  
  t2.category_name,
   COUNT(t2.videoId) video_sum
FROM
( SELECT
t1.videoId,
   category_name
FROM
( SELECT
   videoId,
   views ,
   category
FROM
gulivideo_orc
ORDER BY
views DESC
LIMIT 20
) t1
lateral VIEW explode(t1.category) t1_tmp AS category_name
) t2
GROUP BY t2.category_name
```



**需求四：统计视频观看数top50所关联视频的所属类别排行**

```sql
SELECT
  t6.category_name,
  t6.video_sum,
  rank() over(ORDER BY t6.video_sum DESC ) rk
FROM ( SELECT
  t5.category_name,
  COUNT(t5.relatedid_id) video_sum
FROM
( SELECT
  t4.relatedid_id,
  category_name
FROM
( SELECT
t2.relatedid_id ,
  t3.category
FROM
( SELECT
  relatedid_id
FROM
( SELECT
  videoId,
  views,
  relatedid
FROM
  gulivideo_orc
 ORDER BY
  views
DESC
LIMIT 50
)t1
lateral VIEW explode(t1.relatedid) t1_tmp AS relatedid_id
)t2
JOIN
  gulivideo_orc t3
ON
 t2.relatedid_id = t3.videoId
) t4
lateral VIEW explode(t4.category) t4_tmp AS category_name
) t5
GROUP BY
  t5.category_name
ORDER BY
  video_sum
DESC ) t6
```



**需求五：每个类别中的视频热度top10，以MUSIC为例**

(1)要想统计 Music 类别中的视频热度 Top10，需要先找到 Music 类别，那么就需要将 category 展开，所以可以创建一张表用于存放 categoryId 展开的数据。

(2)向 category 展开的表中插入数据。

 (3)统计对应类别(Music)中的视频热度。

```sql
SELECT
   t1.videoId,
t1.views,
   t1.category_name
FROM
( SELECT
   videoId,
   views,
   category_name
FROM gulivideo_orc
lateral VIEW explode(category) gulivideo_orc_tmp AS category_name
)t1
WHERE
   t1.category_name = "Music"
ORDER BY
   t1.views
DESC
LIMIT 10
```



**需求六：统计每个类别视频观看数的top10**

```sql
SELECT
  t2.videoId,
  t2.views,
  t2.category_name,
  t2.rk
FROM ( SELECT
  t1.videoId,
  t1.views,
  t1.category_name,
  rank() over(PARTITION BY t1.category_name ORDER BY t1.views DESC ) rk
FROM (
SELECT
   videoId,
views,
   category_name
FROM gulivideo_orc
lateral VIEW explode(category) gulivideo_orc_tmp AS category_name
)t1
)t2
WHERE t2.rk <=10
```



**需求七：统计上传视频数最多的用户top10以及他们上传的视频观看数在总top20的视频**

(1)求出上传视频最多的 10 个用户

(2)关联 gulivideo_orc 表，求出这 10 个用户上传的所有的视频，按照观看数取前 20

```sql
SELECT
  t2.videoId,
  t2.views,
  t2.uploader
FROM (
SELECT
  uploader,
  videos
FROM gulivideo_user_orc
ORDER BY
videos DESC
LIMIT 10
) t1
JOIN gulivideo_orc t2
ON t1.uploader = t2.uploader
ORDER BY
  t2.views
DESC
LIMIT 20
```

