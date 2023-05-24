语雀 https://www.yuque.com/hansonhu/lwxhbb/wwx8uz69csu7e8sd/



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230504095112275.png" alt="image-20230504095112275" style="zoom: 67%;" />



### 分布式事务

#### 2PC

2PC的设计目的在于每个子节点在打算提交之前和检查自己的资源情况，如果不能够完成操作，那么回复coordinator一个false，说明这个节点出于unprepared状态，这样整个事务就不会往下执行了，不至于去浪费资源。也就是给每个节点一个**主动放弃的机会**

![image.png](https://cdn.nlark.com/yuque/0/2023/png/32438377/1678359171158-73d1589a-269d-42ed-b932-8a29850f60c1.png?x-oss-process=image%2Fresize%2Cw_642%2Climit_0)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/32438377/1678359213209-d1715fd4-d146-4582-82eb-fc3e11ee0dc5.png?x-oss-process=image%2Fresize%2Cw_643%2Climit_0)



##### 两阶段提交的缺陷

总结将就是2PC没有引入恰当的意外处理机制，比如单点故障之后如何处理，以及某些节点网络异常超时后如何应该如何来处理

1. **同步阻塞问题。**执行过程中，所有参与节点都是事务阻塞型的。当参与者占有公共资源时，其他第三方节点访问公共资源不得不处于阻塞状态。

2. **单点故障。**由于协调者的重要性，一旦协调者发生故障。参与者会一直阻塞下去。尤其在第二阶段，协调者发生故障，那么所有的参与者还都处于锁定事务资源的状态中，而无法继续完成事务操作。（如果是协调者挂掉，可以重新选举一个协调者，但是无法解决因为协调者宕机导致的参与者处于阻塞状态的问题）

3. **数据不一致。**在二阶段提交的阶段二中，当协调者向参与者发送commit请求之后，发生了局部网络异常或者在发送commit请求过程中协调者发生了故障，这回导致只有一部分参与者接受到了commit请求。而在这部分参与者接到commit请求之后就会执行commit操作。但是其他部分未接到commit请求的机器则无法执行事务提交。于是整个分布式系统便出现了数据部一致性的现象。



##### 嵌套事务

![image-20230504105326265](../../../Library/Application Support/typora-user-images/image-20230504105326265.png)

处理方法有两种，一是层次化管理，由每个节点去管理自己的子事务；二是平面化管理，由一个协调节点统一管理，会存在一个abortList的结构用来存储已经放弃了的节点，如果一个对canCommit返回true的节点的任意父节点在这个abortList当中，它最终也会被拒绝执行



#### 3PC

三阶段提交协议在协调者和参与者中都引入超时机制，并且把两阶段提交协议的第一个阶段拆分成了两步：**询问**，然后再**锁资源**，最后真正**提交**。并且在第二、三步添加了超时机制来保证系统的稳定性和数据一致性

![image-20230504104244464](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230504104244464.png)

三阶段提交是“非阻塞”协议。三阶段提交在两阶段提交的第一阶段与第二阶段之间插入了一个准备阶段，使得原先在两阶段提交中，参与者在投票之后，由于协调者发生崩溃或错误，而导致参与者处于无法知晓是否提交或者中止的“不确定状态”所产生的可能相当长的延时的问题得以解决。

⭐️ 在preCommit完成之后，如果协调者不响应了，那么Cohorts在设定时间内会自动执行Commit，但在2PC中Cohorts会一直等待阻塞下去。



##### 总结

2PC存在的最大问题就是阻塞问题，当协调者挂了之后Cohorts会一直在那里做轮询而导致系统阻塞。3PC希望解决这个阻塞的问题，当协调者收集完全部Cohorts的信息后下达执行事务的指令。当协调者即便在这期间挂掉了，由于引入了超时机制，Cohorts依旧会继续执行commit。为了保证这种机制的正确性就需要保证commit前所有的cohorts都能够正常执行commit，由此便引入了preCommit阶段。在preCommit阶段中进行资源锁定操作，在资源锁定都正常的情况下执行commit能保证全局的数据唯一性，由此便可以放心地启动超时自动commit机制。



#### AT

https://seata.io/zh-cn/docs/dev/mode/at-mode.html

Seata的AT（Automatic Transaction）模式是当前最为常用的，它是依赖于数据库自己的回滚特性来实现了，和Spring一样通过在底层开启begin、commit、rollback来进行事务的控制。

在细节上有一个**全局锁**的问题，提交本地事务时要先拿到全局锁，如果锁被占用了的话就一直做自旋操作直到拿到锁或者整个事务操作被取消。



#### TCC

<img src="https://seata.io/img/saga/sofameetup3_img/5.jpeg" alt="16_16_51__08_13_2019.jpg" style="zoom: 67%;" />

把2PC中的try-confirm-cancel三个过程抽象为接口，由用户自己进行业务逻辑定义，Seata提供流程上和通信上的支持。TCC模式适合自定义事务的实现，并且也不依赖于数据库的事务功能。

TCC需要Seata来做好流程细节上的控制让用户专注于业务逻辑，其中包括如下重要特性：允许空回滚、防悬挂控制、幂等控制



#### SAGA

https://seata.io/zh-cn/blog/seata-at-tcc-saga.html

<img src="https://seata.io/img/saga/sofameetup3_img/20.png" alt="图片8.png" style="zoom:67%;" />

Sagas主推一个补偿机制，将事务做成一个调用链（使用蚂蚁金服为此专研的状态机引擎）。如果某个步骤（事务）失败了的话，则开始进入方向补偿机制，直到还原整个全局事务的状态。





### 分布式协调

Zookeeper

etcd

Nacos

Eureka



### 分布式共识算法

Paxos

Raft



### # 数据一致性理论



#### CAP





#### BASE





#### 一致性哈希

**核心思想：**

-  将哈希的节点在哈希链上构成环
- 在哈希环上将每个节点均等地复制，使得节点在环上的分布是相对均匀的
- 在对请求进行哈希的时候按照一个方面匹配上最近的哈希节点，这样一来可以使得以一个很简单的算法实现负载均衡



一致哈希算法是对 **2^32** 进行取模运算的结果值组织成一个圆环，就像钟表一样，钟表的圆可以理解成由 60 个点组成的圆，而此处我们把这个圆想象成由 2^32 个点组成的圆，这个圆环被称为**哈希环**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222164718833.png" alt="image-20230222164718833" style="zoom: 25%;" />



为了避免单点负载过高，我们将节点复制后均等分布在哈希环上，这样尽最大的可能”均衡“

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222164809839.png" alt="image-20230222164809839" style="zoom: 25%;" />

然后额外地我们还需要考虑的问题就是单点故障后，该节点上的数据如何迁移，新进来的请求如何分配：可以重新为每个节点分配点位，依旧是做到每个节点之间的间隙是均等的；原故障节点上的数据根据哈希算法分配到各个节点当中去

