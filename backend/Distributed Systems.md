语雀 https://www.yuque.com/hansonhu/lwxhbb/wwx8uz69csu7e8sd/



### 分布式事务

2PC

3PC

ATC

Seata



### 分布式协调

Zookeeper



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

