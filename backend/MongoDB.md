---
layout: post
title: "MongoDB快速入门"
permalink: /mongodb-crash
---



#### **什么情况下使用MongoDB？**

数据量大

读写都频繁

数据的价值性不高

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220519173727294.png" alt="image-20220519173727294" style="zoom: 25%;" />

使用RDBMS处理海量数据时，系统响应时间变慢。为了解决此问题，当然可以通过升级现有硬件来“纵向扩展”我们的系统。但这个成本很高。这个问题的替代方案是在负载增加时将数据库负载分配到多个主机上。这种方法称为**“横向扩展”**。关系型数据库由于normalization的限制，水平扩展受到限制，故在大数据情况下业界更加青睐NoSQL。

NoSQL有两个特点，非关系和动态架构。后者保证了MongoDB不需要对数据架构进行任何形式的定义，并能提供同一域中的**异构数据结构**。

NoSQL在分布式环境中以**牺牲部分安全性换取了高性能**，节点之间几乎没有同步复制，多为异步多主复制，对等，HDFS复制。仅提供最终的一致性。无共享架构。这样可以减少协调并提高分布。

[SpringBoot快速整合MongoDB教程](https://blog.csdn.net/newCheng/article/details/77747492)



#### MongoDB数据结构

**集合** 

这是MongoDB文档的分组。集合等效于在任何其他RDMS（例如Oracle或MS SQL）中创建的**表**。集合存在于单个数据库中。从介绍中可以看出，集合不强制执行任何结构。上面提到的异构数据指的是在一个集合当中（表），其里面的记录并不需要遵循特点的格式，也就是并非要保持字段上的一致。而关系型数据库中，一张表里面每条记录的结构必定是一样的。

Capped Collections 固定大小的集合

**文档**

MDB是面向文档的，由于MongoDB是NoSQL类型的数据库，它不是以关系类型的格式存储数据，而是将数据存储在文档中。这使得MongoDB非常灵活，可以适应实际的业务环境和需求。开发人员经常会说他们的类不是行和列，而是具有键值对的清晰结构，MDB面向文档的特性很好地迎合了生产实际。

在MDB中一条记录就一个文档，文档里面是诸如JSON的键值对数据。对标MySQL的行和字段概念。

**_id** 

这是每个MongoDB文档中必填的字段。_id字段表示MongoDB文档中的唯一值。_id字段类似于文档的主键。如果创建的新文档中没有_id字段，MongoDB将自动创建该字段。

**字段** 

文档中的名称/值对。一个文档具有零个或多个字段。字段类似于关系数据库中的列。

![image-20220727091604867](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220727091604867.png)



#### WiredTiger引擎

存储引擎要做的事情无外乎是将磁盘上的数据读到内存并返回给应用，或者将应用修改的数据由内存写到磁盘上。如何设计一种高效的数据结构和算法是所有存储引擎要考虑的根本问题，目前大多数流行的存储引擎是基于B-Tree或LSM(Log Structured Merge) Tree这两种数据结构来设计的。

**WT数据结构**

<u>磁盘数据结构</u>

对于WiredTiger存储引擎来说，集合所在的数据文件和相应的索引文件都是按**B-Tree**结构来组织的，不同之处在于数据文件对应的B-Tree叶子结点上除了存储键名外（keys），还会存储真正的集合数据（values），所以数据文件的存储结构也可以认为是一种**B+Tree**，其整体结构如下图所示

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220519200814927.png" alt="image-20220519200814927" style="zoom:25%;" />

<u>内存数据结构</u>

WiredTiger会按需将磁盘的数据**以page为单位**加载到内存，同时在内存会构造相应的B-Tree来存储这些数据。为了高效的支撑CRUD等操作以及将内存里面发生变化的数据持久化到磁盘上，WiredTiger也会在内存里面维护其它几种数据结构，如下图所示

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220519200923445.png" alt="image-20220519200923445" style="zoom:25%;" />

**Page的生命周期**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220519211959199.png" alt="image-20220519211959199" style="zoom:33%;" />

磁盘 —— 加载进内存 —— 在内存中被更新 —— 在内存中reconcile —— 落盘 



### Quizs

#### MySQL和MongoDB的区别

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230119003110460.png" alt="image-20230119003110460" style="zoom: 33%;" />

Mongo一些独到的特点：

1. 数据的异构性
2. 热数据保存在内存当中
3. 不支持事务
4. 由于数据的异构性，基于数据统计的目的原生引入了map-reduce



#### MongoDB比MySQL快的原因，压测验证

1. 首先是内存映射机制，数据不是持久化到存储设备中的，而是暂时存储在内存中，这就提高了在IO上效率以及操作系统对存储介质之间的性能损耗(毕竟内存读取最快)

2. 其次，NoSQL并不是不使用sql，只是不使用关系。没有关系的存在，就表示每个数据都好比是拥有一个单独的存储空间，然后一个**聚集索引**来指向，搜索性能一定会提高的。
3. 使用javascript语法进行操作更加高效、直接。这些是MongoDB针对关系型数据库的效率要高的原因
   



[压测case](https://www.jianshu.com/p/9e582b470810)

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230119153554233.png" alt="image-20230119153554233" style="zoom: 33%;" />

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230119154622167.png" alt="image-20230119154622167" style="zoom:33%;" />

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230119154643744.png" alt="image-20230119154643744" style="zoom:33%;" />

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230119154703954.png" alt="image-20230119154703954" style="zoom:33%;" />

1. 相比较MySQL，**MongoDB数据库更适合那些读作业较重的任务模型**。MongoDB能充分利用机器的内存资源。如果机器的内存资源丰富的话，MongoDB的查询效率会快很多。

2. 在带”id”插入数据的时候，MongoDB的插入效率其实并不高。如果想充分利用MongoDB性能的话，**推荐采取不带”id”的插入方式，然后对相关字段作索引来查询**。
3. MongoDB在稳定性上输MySQL很多，因此C端应用需要考虑用其他的一些手段来counter这个不稳定性

