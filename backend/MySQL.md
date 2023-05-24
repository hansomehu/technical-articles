---
layout: post
title: "MySQL理论笔记"
permalink: /mysql-theory
---

从SELECT * 到删库跑路——MySQL最全笔记



## 1. 数据库的范式理论



#### **<u>什么是和为什么要？</u>**

数据库的设计存在四个问题：**冗余、删除、修改、插入异常**， 引入范式的原因就是要消除上述数据库的潜在问题。原则就是依赖分解，分解到最后，一张表里的主键就一个，比如学号-姓名，这样就完全不存在上面谈到的问题， 这就是第5范式。但第5范式毫无疑问是低效的。

设计数据库的时候记住，一个业务一张表，后续表关联即可，不要一张表怼进去一堆相互依赖的键。

不同级别的区别主要在于**字段之间依赖关系**， 高级别范式的依赖于低级别的范式，1NF 是最低级别的范式。

在实际生产当中时最高也就**满足第三范式**即可，高范式意味着高约束，会带来额外的开销，比如降低查询的效率是肯定的。



#### <u>**五大范式的简要概括**</u>

1NF：

**字段不可以再被拆分**，但要明白这中定义是存在主观性的，比如地址需不需要把省份和市单独拆出来？

例如：第一章表，key_id ｜name｜info(phone, address)；第二张表，id｜phone｜address 。第一张表中的info还可以再拆分，通常来说不满足1NF。

所以，标准就是，如果一张表中的一个字段包含多项信息，其中的某一项信息在另外一张表中是一个独立的字段，这种情况不满足1NF。

2NF：

**一张表是一个独立的消息**，在满足1NF的基础上，每一条记录都是被主键（可以是多字段的联合主键）唯一标识。如果主键有两个，存在某一条记录只单独依赖于主键元组中的某一个而与另外一个没关系，则这种表设计不满足2NF。解决的办法，把不被依赖的那个主键单独抽取出来和其他字段建立一张表

e.g ｜<u>学号</u>｜<u>课程号</u>｜成绩｜ 学号和课程号唯一标识成绩，满足2NF

3NF：

**非主键之间相互不能存在依赖**，第二范式中存在着A -> B -> <u>C</u> 这种循环依赖

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220416201046057.png" alt="image-20220416201046057" style="zoom:67%;" />

BCNF Boyce-Codd： 

**主键内的属性键不能存在依赖关系**。例如，在关系R中，主键U，A是主键U的一个属性，Y为U的主属性，如果A -> Y那么该表设计不符合BCNF

4NF：

消除了表中的多值依赖（非平凡的多值依赖）

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220416201425526.png" alt="image-20220416201425526" style="zoom:70%;" />

5NF：

完美范式



## 2. 存储引擎

引擎是相对于表来说的（表处理器）一个表能指定一种引擎

#### 1. InnoDB和MyISAM的区别

##### MyISAM：

- MySQL5.1及之前，MyISAM 是默认存储引擎，MyISAM 提供了大量的特性，包括全文索引、压缩、空间函数等；

- **事务缺陷。**不支持事务和行锁，最大的缺陷就是崩溃后无法安全恢复。对于只读的数据或者表比较小、可以忍受修复操作的情况仍然可以使用 MyISAM。支持整张表的排他锁，但是在表有读取查询的同时，也支持并发往表中插入新的记录；

- **三份文件。**MyISAM 将表存储在数据文件和索引文件中，分别以 `.MYD` 和 `.MYI` 作为扩展名，同时还有一个表结构文件。MyISAM 表可以包含<u>动态或者静态行</u>，MySQL 会根据表的定义决定行格式。MyISAM 表可以存储的行记录数一般**受限于可用磁盘空间或者操作系统中单个文件的最大尺寸**。

- 对于MyISAM 表，MySQL 可以**手动或自动执行检查和修复操作**，这里的修复和事务恢复以及崩溃恢复的概念不同。执行表的修复可能导致一些数据丢失，而且修复操作很慢。

- 支持多类型的全文索引。对于 MyISAM 表，即使是 BLOB 和 TEXT 等长字段，也可以基于其前 500 个字符创建索引。MyISAM 也**支持全文索引**，这是一种基于分词创建的索引，可以支持复杂的查询。

- **表锁问题开销大。**MyISAM 设计简单，数据以**紧密格式存储**，所以在某些场景下性能很好。MyISAM 最典型的性能问题还是**表锁问题**，如果所有的查询长期处于 Locked 状态，那么原因毫无疑问就是表锁。



##### InnoDB：

- 是 MySQL 默认的**事务型存储引擎**，只有在需要它不支持的特性时，才考虑使用其它存储引擎。

- 实现了四个标准的隔离级别，默认级别是可重复读(REPEATABLE READ)。在可重复读隔离级别下，通过多版本并发控制(MVCC)+ 间隙锁(Next-Key Locking)防止幻影读。

- 主索引是**聚簇索引**，在索引中保存了数据，从而避免直接读取磁盘，因此对查询性能有很大的提升。

- 内部做了很多优化，包括从磁盘读取数据时采用的**可预测性读**、能够加快读操作并且自动创建的自适应哈希索引、能够加速插入操作的插入缓冲区等。

- 支持真正的**在线热备份**。其它存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合场景中，停止写入可能也意味着停止读取。







#### 2. 二者的选择

InnoDB适合用于存在**大量事务操作、大表、频繁删改**等业务当中，由于InnoDB采用的**聚簇索引**在高性能的同时需要大量的内存开销，因此需要综合考虑业务和成本来进行选型。如果业务主要是**增查操作、小表**的话，使用MyISAM的性价比更高。



事务: InnoDB 是事务型的，可以使用 Commit 和 Rollback 语句。

并发: MyISAM 只支持表级锁，而 InnoDB 还支持行级锁。

外键: InnoDB 支持外键。

备份: InnoDB 支持在线热备份。

崩溃恢复: MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢。

其它特性: MyISAM 支持压缩表和空间数据索引。



## 3. 索引

主要知识点就是基于**聚簇索引**方式的**B+树索引**

MySQL的索引分两个，主键索引和二级索引。主键索引的叶子节点上存的的完整的数据行；二级索引则是根据后来添加的字段列来建立索引树，叶子节点存的是索引字段的值和主键值。后期回表的时候用主键ID去主键索引中进行查找。



#### 为什么用B+树，而不用B树或者红黑树？

> B+树出度多树高低、利用磁盘的预读特性、叶子节点链表连接

B+树是聚簇索引，且叶子节点以链表的形式相连，加速查找

同红黑树这种二叉树相比，B+树非叶子节点的出度都很高，在数据量相同的情况下，树高会大大降低，而这种树型结构的查找时间复杂度又和树高密切相关，故更快。

> 红黑树O(h) ，B+树 O(logN)，且 h = log d N

同B树相比较，B+树叶子节点以链表的形式存储，**方便范围查询**，而B树则做不到

B+树的插入时间复杂度很稳定，因为是从下往上进行插入的



B树叶子节点没链表，无法利用磁盘的预读





#### 普通索引与唯一索引的区别





#### Primary 主索引



#### Secondary（二级索引）





<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220417162650819.png" alt="image-20220417162650819" style="zoom:25%;" />

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220417163516576.png" alt="image-20220417163516576" style="zoom:25%;" />



#### 索引的数据结构：

record_type:  0 表示普通记录、 2 表示最小记 录、 3 表示最大记录、1 表示目录项中的数据

next_type：记录头信息的一项属性，表示下一条地址相对于本条记录的地址偏移量，我们用箭头来表明下一条记录是谁

存放用户记录的**叶子节点**代表的数据页1kb可以存放**100条**用户记录 ，所有存放目录项记录的内节点代表的数据页可以存放**1000条**目录项记录 



#### **为什么使用索引？**

- 大大减少了服务器需要扫描的数据行数，索引的使用**避免了全盘扫描**

- 帮助服务器**避免进行排序和分组**，也就**不需要创建临时表**(B+Tree 索引是有序的，可以用于 ORDER BY 和 GROUP BY 操作。临时表主要是在排序和分组过程中创建，因为不需要排序和分组，也就不需要创建临时表)

- 将随机 I/O 变为**顺序 I/O**(B+Tree 索引是有序的，也就将**相邻的数据都存储在一起**)



#### **索引在什么场景下使用？**

1. **字段的数值有唯一性的限制**。业务上具有唯一特性的字段，即使是组合字段，也必须建成唯一索引。(来源:Alibaba) 说明：不要以为唯一索引影响了 insert 速度，这个速度损耗可以忽略，但提高查找速度是明显的。
2. **频繁作为 WHERE 查询条件的列**。某个字段在SELECT语句的 WHERE 条件中经常被使用到，那么就需要给这个字段创建索引了。尤其是在数据量大的情况下，创建普通索引就可以大幅提升数据查询的效率。 比如student_info数据表(含100万条数据)，假设我们想要查询 student_id=123110 的用户信息。

3. **经常GROUP BY和ORDER BY的列**。索引就是让数据按照某种顺序进行存储或检索，因此当我们使用 GROUP BY 对数据进行分组查询，或者 使用 ORDER BY 对数据进行排序的时候，就需要对分组或者排序的字段进行索引 。如果待排序的列有多个，那么可以在这些列上建立 组合索引 。

4. **UPDATE、DELETE 的 WHERE 条件列**。对数据按照某个条件进行查询后再进行 UPDATE 或 DELETE 的操作，如果对 WHERE 字段创建了索引，就 能大幅提升效率。原理是因为我们需要先根据 WHERE 条件列检索出来这条记录，然后再对它进行更新或 删除。 **如果进行更新的时候，更新的字段是非索引字段，提升的效率会更明显，这是因为非索引字段更 新不需要对索引进行维护。**<u>也就是不推荐对修改的目标字段建索引</u>

5. **DISTINCT 字段需要创建索引**。有时候我们需要对某个字段进行去重，使用 DISTINCT，那么对这个字段创建索引，也会提升查询效率。比如，我们想要查询课程表中不同的 student_id 都有哪些，如果我们对student_id 建立了索引，那么直接在索引中就能找到，并且结果都排好序的。

6. **多表 JOIN 连接操作时，对where字段建立索引，因为这才是最终的筛选字段**。同时应该注意，join的表不要超过3张，多重嵌套循环很影响效率。

7. **长文本建立索引时最好使用文本的一段前缀而非对整个文本建立索引**。这很好理解，长文本的重复率很低，比如博客文章，对前几个字符建立索引照样能精确匹配到结果，索引的时间和空间成本还都小了。

   `create table shop(address varchar(120) not null);`
   `alter table shop add index(address(12));`

8.  **区分度高(散列性高)的列适合作为索引**

9. **使用最频繁的列放到联合索引的左侧**



#### 什么场景下不适合建立索引？

1. **在where中使用不到的字段，不要设置索引**
2. **数据量小的表不要创建索引**
3. **有大量重复数据的列不创建索引**。例如要在 100 万行数据中查找其中的 50 万行(比如性别为男的数据)，一旦创建了索引，你需要先访问 50 万次索引，然后再访问 50 万次数据表，这样加起来的开销比不使用索引可能还要大
4. **避免对经常更改数据的表创建过多的索引**
5. **避免重复创建或者冗余创建索引**



#### 什么是联合索引？

联合索引是一种**非聚簇索引**，在设计联合索引时要关注最左匹配原则，索引文件具有 B-Tree 的最左前缀匹配特性，如果左边的值未确定，那么无法使用此索引。

联合索引就是在一个表的某项业务查询中的where子句中会用到多个字段，这些字段共同构建的索引叫做联合索引。联合索引的构建需要遵循一定的规则以达到最大效用，随意建立索引会使得索引的开销远大于索引所带来的回报。

为什么存在最左匹配原则：如果你建立一个`（a,b,c）`的联合索引，相当于建立了 `(a)、(a,b)、(a,b,c)` 三个索引。那么这个时候如果单独查一个b字段，必然是不走索引的了。

具体的规则：

```sql
# 在都是=匹配的情况下，索引遵循最左匹配原则，应该把区别度最大的字段放在联合索引的最前端index (a,b,c)
SELECT * FROM table WHERE a = 1 and b = 2 and c = 3; 

# 对(b,a)建立索引,如果你建立的是(a,b)索引，那么只有a字段能用得上索引，毕竟最左匹配原则遇到范围查询就停止匹配
SELECT * FROM table WHERE a > 1 and b = 2; 

# 将=匹配放在最左
SELECT * FROM `table` WHERE a > 1 and b = 2 and c > 3; 

# IN 可以等价为等值匹配
SELECT * FROM `table` WHERE a IN (1,2,3) and b > 1; 
```

// 关于最左匹配原则，以A、B、C三个键而建立的索引在B+树中会产生A、AB、ABC三个索引



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220416225235468.png" alt="image-20220416225235468" style="zoom:33%;" />



#### **使用索引的代价：**

**空间上的代价**

每建立一个索引都要为它建立一棵B+树，每一棵B+树的每一个节点都是一个数据页，一个页默认会占用**16KB** 的存储空间，一棵很大的B+树由许多数据页组成，那就是很大的一片存储空间。

**时间上的代价**

每次对表中的数据进行 操作时，都需要去修改各个B+树索引。而且我们讲过，B+树每 层节点都是按照索引列的值 而组成了双向链表 。不论是叶子节点中的记录，还是内节点中的记录(也就是不论是用户记录还是目录项记录)都是按照索引列的值从小到大的顺序而形成了一个单向链表。而增、删、改操作可能会对节点和记录的排序造成破坏，所以存储引擎需要额外的时间进行一些记录移位 ，页面分裂 、页面回收等操作来维护好节点和记录的排序。如果 我们建了许多索引，每个索引对应的B+树都要进行相关的维护操作，会给性能拖后腿。



#### 索引结构探析：B+树与B树与红黑树

> 7.29百度面试

1. 首先B+树的数据结构，非叶子节点保存两个关键数据，该节点以下的节点的索引值的最大值和最小值
2. B+树的叶子节点上存储数据（主键索引和非主键索引要区别开）
3. 叶子节点的数据用链表进行存储
4. B+ Tree 是基于 B Tree 和叶子节点顺序访问指针进行实现，它具有 B Tree 的平衡性，并且通过顺序访问指针来提高区间查询的性能。B树和B+树最大的区别就是，B树是将各种信息保存在所有节点中的，B+树是将各种信息保存在叶子中的。
5. **B树查找的时间复杂度是O(1)到O(h)；B+树的查找效率是O(logN)，插入和删除这是O(1)；B+树稳定在这个值**
6. 从查找的效率上面来看，B树要强于B+树。但是B+树的数据存储方式，全部集中中叶子节点，且用链表连接。这种结构更加适合数据库系统的数据查找，数据库可以快速定位到某个数据周围的数据的内存地址。当数据表很大的时候，这种快速定位带来的效率是大于B+树额外的查找速率的。

为什么不用红黑树？

红黑树是二叉树，每个节点的出度很低，导致树高h很高。查找的时间复杂度又同h相关，导致查询效率很低



#### **查找：**

进行查找操作时，首先在根节点进行二分查找，找到一个 key 所在的指针，然后递归地在指针所指向的节点进行查找。直到查找到叶子节点，然后在叶子节点上进行二分查找，找出 key 所对应的 data。

插入删除操作记录会破坏平衡树的平衡性，因此在插入删除操作之后，需要对树进行一个分裂、合并、旋转等操作来维护平衡性。插入和删除操作带来了**较大的开销**



#### 索引的种类：

1. 普通索引

在创建表的语句中增加 `INDEX(year_publication)`

2.  创建唯一索引

```sql
UNIQUE INDEX uk_idx_id(id)
SHOW INDEX FROM test1 \G
```

3. 主键索引

在创建表的时候随着主键的指定被指定，隐式和显式两种

```sql
# 删除主键索引
ALTER TABLE student
drop PRIMARY KEY ;
```

4. 单列索引

```sql
CREATE TABLE test2(
id INT NOT NULL,
name CHAR(50) NULL,
INDEX single_idx_name(name(20))
);
```

5. 组合索引

根据字段由前到后进行排序，假设index（a,b,c）

那么先以a进行排序然后再以b、c进行排序

```sql
CREATE TABLE test3(
id INT(11) NOT NULL,
name CHAR(30) NOT NULL,
age INT(11) NOT NULL,
info VARCHAR(255),
INDEX multi_idx(id,name,age)
);
```

6. 全文索引

```sql
CREATE TABLE `papers` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(200) DEFAULT NULL,
  `content` text,
  PRIMARY KEY (`id`),
  FULLTEXT KEY `title` (`title`,`content`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

# 检索方式较普通方式有区别
SELECT * FROM papers WHERE MATCH(title,content) AGAINST (‘查询字符串’);
```



#### 隐式索引

由于索引存在开销，有些小表或小数据不用索引反而效率更高，应该在实际业务中灵活使用索引。但是，索引一旦删除再重建可能带来一系列额外问题和麻烦，那么当不用的时候可以**手动隐藏索引**，之后再查询的时候不会走索引，当有需要的时候可以再开启索引。这样能使开发更加灵活。



#### limit深分页导致的索引失效问题

`select id,name,balance from account where create_time > '2020-09-19' limit 100000,10;`

取100000后面的10条记录，这条SQL看起来没什么问题，只是这种业务需求通过硬查询语句来做是低效的。它存在着一个深分页索引失效问题。安装mysql的逻辑走下去它的执行步骤如下：

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230118170642009.png" alt="image-20230118170642009" style="zoom:25%;" />

但是前面的十万次回表显然是不必要的，针对这种业务需求，单单从sql的层面下手很难圆满收场。一个比较好的思路是：在主键id上开启自增，然后select ... where id > 100000 limit 10;



#### 一页B+树索引可以存放多少数据？

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230118170941861.png" alt="image-20230118170941861" style="zoom: 33%;" />

B + 树叶子存的是数据，内部节点存的是键值 + 指针。索引组织表通过非叶子节点的二分查找法以及指针确定数据在哪个页中，进而再去数据页中找到需要的数据。假设 B + 树的高度为 2 的话，即有一个根结点和若干个叶子结点。这棵 B + 树的存放总记录数为 = 根结点指针数 * 单个叶子节点记录行数。

InnoDB中一页数据通常为16k，总数据量 = 叶子结点总数 * 16kb/单条数据大小



#### 索引下推 模糊like %

原本不能走索引的情况为了减少后序回表操作的频率，也尽量在索引当中再筛选一次

比如一个联合索引（a，b，c）

这个时候有筛选条件 where a=xx，b like “%xx”

按正常情况来看只能对a用索引，但是为了减少回表的次数，在索引中将b也顺带匹配一下的开销要比后期回表小得多，那么对b的筛选也走索引



#### 索引覆盖 select < index

覆盖索引（covering index ，或称为索引覆盖）即从非主键索引中就能查到的记录，而不需要查询主键索引中的记录，避免了回表的产生减少了树的搜索次数，显著提升性能。

比如，现在有联合索引（a，b，c），如果我select的字段为（a，b），那么这个时候MySQL不会再走回表的操作了而是直接返回索引里面存放的值



#### 索引失效的情况







#### 哈希索引

是一种K/V键值对类型的索引，在InnoDB引擎有一种机制叫做自适应哈希索引，当有一部分的数据被查询的频率非常高的时候，InnoDB引擎会自动地去为这些键创建哈希索引加快查找效率

哈希索引的一大特点就是，其受限较多，但是一旦某些数据场景符合使用哈希索引的条件，那么对效率的提升将是巨大的



其存在的缺点：

- 1、不能避免读取行

哈希索引只包含哈希值和行指针，而不存储字段值，所以不能使用索引中的值来避免读取行。不过，访问内存中的行的速度很快，所以大部分情况下这一点对性能的影响并不明显。

- 2、无法用于排序

哈希索引数据并不是按照索引值顺序存储的，所以也就无法用于排序。

- 3、无法使用部分索引列匹配查找

哈希索引也不支持部分索引列匹配查找，因为哈希索引始终是使用索引列的全部内容来计算哈希值的。例如，在数据列（A,B）上建立哈希索引，如果查询只有数据列A，则无法使用该索引。

- 4、只支持等值查找

哈希索引只支持等值比较查询，包括=、IN()、<=>（注意<>和<=>是不同的操作）。也不支持任何范围查询，例如WHERE price>100。

- 5、存在Hash冲突

访问哈希索引的数据非常快，除非有很多哈希冲突（不同的索引列值却有相同的哈希值）。当出现哈希冲突的时候，存储引擎必须遍历链表中所有的行指针，逐行进行比较，直到找到所有符合条件的行。







## 4. 性能优化 Performance Optimization

MySQL中出现的性能问题最大归因在于查询的时候没有匹配上索引，或者索引的设置不符合最佳实践。因此，性能优化需要往索引上面去想，建立正确的索引，正确地使用SQL匹配上索引

在MySQL的console中开启set global slow_query_log=on，然后在设置一下阈值和log文件dump的地址

下面mysqldumpslow这个插件，去分析一下log

拿到log的SQL之后，用explain去分析具体的原因

需要注意的就是不要一直开启慢查询日志功能，会很消耗性能，分析一段时间就可以关掉了



#### Explain 的使用

![image-20230221115946820](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230221115946820.png)

- type字段涵义：

  system：系统表，少量数据，往往不需要进行磁盘IO

  const：常量连接

  eq_ref：主键索引(primary key)或者非空唯一索引(unique not null)等值扫描

  ref：非主键非唯一索引等值扫描

  range：范围扫描

  index：索引树扫描

  ALL：全表扫描(full table scan)







#### 如何避免全表扫描？

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230118162057832.png" alt="image-20230118162057832" style="zoom:33%;" />

- like模糊查询
- 查询条件中包含询条件is null、is not null
- 查询条件中使用了不等于操作符（<>、!=）
- 组合索引使用不当（为遵循最左匹配、等值匹配等原则）
- in 和not in 导致全表
- where中存在表达式或者函数

总结就是WHERE中存在对字段进行筛选的必定需要全表扫描，因为要对每一行记录进行一个扫描比较



#### 表连接中存在的优化项

永远遵循小表驱动大表的原则

相当于：

fori=10 小表{

fori=1000 大表

}

1 LEFT JOIN 左连接，左边为驱动表，右边为被驱动表.

2 RIGHT JOIN 右连接，右边为驱动表，左 表.

3 INNER JOIN 内连接，mysql会选择数据量比较小的表作为驱动表，大表作为被驱动表.

4 可通过EXPLANIN查看SQL语句的执行计划,EXPLANIN分析的第一行的表即是驱动表.

in 后面要跟小表

exsits 后面要跟大表



#### 慢查询为什么发生？如何优化慢查询？

大量数据的时候索引反而会成为累赘，索引会有一个回表的过程增加耗时，这个无法避免。大数据量的慢查询当中能够人为进行干预的部分就是索引的设置是否合理，查询sql的设计是否高效命中索引。





#### SQL优化





#### 索引优化     别null、最左、等值优先、开头不能模糊、非函数

- 用不用索引和where中字段的顺序无关，比如我有联合索引（a，b，c），在where当中我是先b后a，还是先a后b都会匹配上这个索引
- 最左匹配，关键是不能跳过某个索引字段。比如我有联合索引（a，b，c），但是在where中我只用到了a、c，这显然就是不行的
- 主键auto-increment，插入新数据就是有序的，会减少开销
- 不针对索引字段使用函数、计算、类型转换，这些会导致索引失效。计算指的是where id+1=1000; 类型转换比如name是字符串类型，但是where name = 123
- 范围查询右边的字段索引会失效，将范围查询放在最后面。比如where a=xx, b=xx, c>xx，如果将c放在中间的话，那么b的索引会匹配不上
- 不等于操作会失效   ! =  <>  IS NOT NULL；实际操作中将字段设置为not null，然后用‘0’来代替null值
- 前后存在非索引的列会导致索引失效
- 模糊匹配开头如果就是模糊的话，那就是不行的，例如 "%xxx"，模糊在中间或者后面没问题可以使用到索引





#### 数据库哲学上的优化

- 比如范式这个事情，可以通过反范式的方式来让表冗余来减少JOIN表的数量





## 5. 事务 Transaction 

先看看事务的隔离级别和对应解决的事务问题

| *<u>Isolation Level</u>* | Dirty Read | Non-Repeatable Read | Phantom Read | Locked Read |
| :----------------------: | :--------: | :-----------------: | :----------: | :---------: |
|     READ UNCOMMITTED     |    Yes     |         Yes         |     Yes      |             |
|      READ COMMITTED      |     No     |         Yes         |     Yes      |             |
|     REPEATABLE READ      |     No     |         No          |     Yes      |             |
|       SERIALIZABLE       |     No     |         No          |      No      |             |



#### 事务的ACID特性是什么？

**原子性(**atomicity**):**

原子性是指事务是一个不可分割的工作单位，要么全部提交，要么全部失败回滚到最初状态

> 原子性是如何保证的

UNDO LOG 称为回滚日志 ，回滚行记录到某个特定版本，用来保证事务的原子性、一致性。undo log保证原子性和一致性的原理是在事务被判断为失败的时候根据log中的记录来将某事务过程中的全部操作进行一个undo操作，也就是“回滚”。Undo日志的作用有两个，回滚数据和实现MVCC（InnoDB的MVCC通过undo log来实现）。



**一致性(**consistency**)：**

“一致”是指数据库中的数据是正确的，不存在矛盾。事务的一致性是指事务执行前后，数据都是正确的，不存在矛盾。如果执行后数据是矛盾的，事务就会回滚到执行前的状态（执行前是一致的）。也就是经过事务操作之后，无论事务是正常提交了或者是回滚了，最终业务的数据都是符合预期的。

**隔离型(**isolation**):**

事务的隔离性是指一个事务的执行 ，即一个事务内部的操作及使用的数据对 并发 的 其他事务是隔离的，并发执行的各个事务之间不能互相干扰。

**持久性(**durability**):** 

持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的 ，接下来的其他操作和数据库故障不应该对其有任何影响。

持久性是通过 **事务日志** 来保证的。日志包括了 redo-log重做日志 和 回滚日志 。当我们通过事务对数据进行修改 的时候，首先会将数据库的变化信息记录到重做日志中，然后再对数据库中对应的行进行修改。这样做 的好处是，即使数据库系统崩溃，数据库重启后也能找到没有更新到数据库系统中的重做日志，重新执 行，从而使事务具有持久性。



#### 事务都有哪些状态？

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220418182740136.png" alt="image-20220418182740136" style="zoom:33%;" />



#### 事务有哪些启动方式？

1. 显示启动

```sql
mysql> BEGIN;
# 或者, START TRANSACTION启动方式能选择参数READ ONLY、WRITE ONLY、WITH CONSISTENT SNAPSHOT
mysql> START TRANSACTION;

# 提交事务。当提交事务后，对数据库的修改是永久性的。 
mysql> COMMIT;
# 回滚事务。即撤销正在进行的所有没有提交的修改 
mysql> ROLLBACK;
# 将事务回滚到某个保存点。
mysql> ROLLBACK TO [SAVEPOINT]

SAVEPOINT a;
```

2. 隐式启动

```sql
mysql> SHOW VARIABLES LIKE 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.01 sec)

SET autocommit = OFF; #或
SET autocommit = 0;
```



#### **mysql存在的事务问题？**

1. **脏写(Dirty Write)**

对于两个事务 Session A、Session B，如果事务Session A 修改了 另一个 未提交 事务Session B 修改过 的数 据，那就意味着发生了脏写。脏写由于问题过于严重，在mysql中默认的隔离级别都已经解决了这个问题。

2. **脏读(Dirty Read )**

对于两个事务 Session A、Session B，Session A 读取 了已经被 Session B 更新 但还 没有被提交 的字段。之后若 Session B 回滚 ，Session A 读取 的内容就是 临时且无效 的。

Session A和Session B各开启了一个事务，Session B中的事务先将studentno列为1的记录的name列更新 为'张三'，然后Session A中的事务再去查询这条studentno为1的记录，如果读到列name的值为'张三'，而 Session B中的事务稍后进行了回滚，那么Session A中的事务相当于读到了一个不存在的数据，这种现象 就称之为 脏读 。

3. **不可重复读(Non-Repeatable Read)**

对于两个事务Session A、Session B，Session A 读取 了一个字段，然后 Session B 更新 了该字段。 之后

Session A 再次读取 同一个字段， 值就不同 了。那就意味着发生了不可重复读。

我们在Session B中提交了几个 隐式事务 (注意是隐式事务，意味着语句结束事务就提交了)，这些事务都修改了studentno列为1的记录的列name的值，每次事务提交之后，如果Session A中的事务都可以查看 到最新的值，这种现象也被称之为 不可重复读 。

4. **幻读( Phantom )**

对于两个事务Session A、Session B, Session A 从一个表中 读取 了一个字段, 然后 Session B 在该表中 插入了一些新的行。 之后, 如果 Session A 再次读取 同一个表, 就会多出几行。那就意味着发生了幻读。Session A中的事务先根据条件 studentno > 0这个条件查询表student，得到了name列值为'张三'的记录；之后Session B中提交了一个 隐式事务 ，该事务向表student中插入了一条新记录;之后Session A中的事务 再根据相同的条件 studentno > 0查询表student，得到的结果集中包含Session B中的事务新插入的那条记 录，这种现象也被称之为 幻读 。我们把新插入的那些记录称之为幻影记录 。



#### mysql的隔离级别有哪些？

**READ UNCOMMITTED :**

读未提交，在该隔离级别，所有事务都可以看到其他未提交事务的执行结 果。不能避免脏读、不可重复读、幻读。

**READ COMMITTED :**

读已提交，它满足了隔离的简单定义:一个事务只能看见已经提交事务所做 的改变。这是大多数数据库系统的默认隔离级别(但不是MySQL默认的)。可以避免脏读，但不可 重复读、幻读问题仍然存在。

**REPEATABLE READ :**

可重复读，事务A在读到一条数据之后，此时事务B对该数据进行了修改并提交，那么事务A再读该数据，读到的还是原来的内容。可以避免脏读、不可重复读，但幻读问题仍然存在。这是MySQL的默认隔离级别。RR隔离级别下，通过MVCC和间隙锁（行锁的一种）杜绝了大多数的幻读问题。

**SERIALIZABLE :**

可串行化，确保事务可以从一个表中读取相同的行。在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作。所有的并发问题都可以避免，但性能十分低下。能避免脏读、不可重复读和幻读。



#### 在RR隔离级别下是如何解决幻读问题的？

RR隔离级别下幻读的解决是通过MVCC的**快照读**，即在一个时刻上的MySQL数据库指定数据行的备份中进行操作。这种操作能解决部分幻读问题，即读的问题可以完全解决，但是写的问题还可能发生幻读的情况。例如，Thread A 在1-10行数据上进行快照读，Thread B 这个时候插入了一条数据7，那么Thread A 对1-10的读是没有问题的，但是当Thread A 想往7上插数据时候则会冲突报错。

MVCC还有一个当前读，当前读就是对要读取的数据行上锁，开销较大。



#### mysql的事务机制是如何实现的？

<!--通过undo log和redo log来实现，前者用来保证原子性和一致性，后者保证持久性-->

UNDO LOG 称为回滚日志 ，回滚行记录到某个特定版本，用来保证事务的原子性、一致性。undo log保证原子性和一致性的原理是在事务被判断为失败的时候根据log中的记录来将某事务过程中的全部操作进行一个undo操作，也就是“回滚”。Undo日志的作用有两个，回滚数据和实现MVCC（InnoDB的MVCC通过undo log来实现）。

**undo log的存储结构：**

InnoDB对undo log的管理采用段的方式，也就是每个**rollback segment 回滚段**记录了 1024个undo log segment，而在每个undo log segment段中进行undo页的申请。从1.1版本开始InnoDB支持最大 128个rollback segment ，故其支持同时在线的事务限制提高到 了 **128*1024** 。

<u>p.s InnoDB引擎存储结构</u>

![img](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/524341-20160416104932379-1446683950.jpg)



**undo log页是重复使用的**，当我们开启一个事务需要写undo log的时候，先去undo log segment中找到一个空闲的位置，然后申请undo页，每个undo页是16kb。当一个事务执行完成并commit了undo log之后，系统会判断当前的undo页剩余大小是否大于1/4，如果是的话下一个事务仍会被安排到该页中，以节约内存空间。

undo log根据业务类型被划分成了insert和update两种。其中在undo log执行删除操作的时候，这两种undo log的逻辑略有不同，insert直接删除，因为这类操作只对本事务可见。而update型的undo log可能需要提供MVCC机制，因此不能在事务提交时就进行删除。提交时放入undo log链表，等待purge线程进行最后的删除。



**<u>REDO LOG 称为重做日志 ，</u>**提供再写入操作，恢复提交事务修改的页操作，用来保证事务的**持久性**。在一个事务执行成功之后，相关的数据只是在内存中进行了修改，而没有同步到磁盘上去，这个时候把修改记录放在redo log当中，一定时间再一起将磁盘上的数据做修改，这样的话能够保证最大的性能，如果同步改磁盘的话会由于随机IO造成严重的性能损害。通常也会在节点重启的时候再通过redo log来将数据的修改持久化到磁盘中去。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220419004221989.png" alt="image-20220419004221989" style="zoom:33%;" />

Redo log可以简单分为以下两个部分：重做日志的缓冲 (redo log buffer) ，保存在内存中，是易失的。重做日志文件 (redo log file) ，保存在硬盘中，是持久的。前者通过某种持久化策略定期将内容持久化到磁盘中。

redo log的工作流程.jpg

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220419004556227.png" alt="image-20220419004556227" style="zoom: 25%;" />

**redo log的刷盘策略**：InnoDB给出 innodb_flush_log_at_trx_commit 参数，共有三种策略。

设置为0:表示每次事务提交时不进行刷盘操作。(系统默认masterthread每隔1s进行一次重做日 志的同步)

设置为1:表示每次事务提交时都将进行同步，刷盘操作（系统默认值）

设置为2:表示每次事务提交时都只把 redo log buffer 内容写入 page cache，不进行同步。由os自己决定什么时候同步到磁盘文件。

**redo log的存储结构：**

mini transaction概念，一个事务可以包含若干条语句，每一条语句其实是由若干个 mtr 组成，每一个 mtr 又可以包含若干条redo日志

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220419112627979.png" alt="image-20220419112627979" style="zoom:33%;" />



**redo log以文件组的形式进行存储：**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220419113026566.png" alt="image-20220419113026566" style="zoom: 33%;" />

ib_logfile0-n指的是innodb中的文件组的数量编号，按环形进行存储。为了避免先前写过的数据被新数据覆盖，innodb设计了checkpoint概念，当目前的write pos到达checkpoint的时候说明**日志文件组**满了。系统将清空部分文件，之后将checkpoint和write pos移到空白区和已写区的开始。



**<u>mysql锁实现了事务的隔离性，</u>**即线程之间对同一资源的操作是互不可见的





## 6. MySQL锁相关问题

#### **mysql哪些操作需要加锁，加什么锁？**

**读-读**情况下由于不设计数据的修改，每条线程读操作都具有幂等性，不需要加锁

**写-写**操作是最需要并非控制的情况，需要严格通过悲观锁机制来实现

**写-读**操作相对较为灵活，其中**读部分可以通过MVCC**来以较低开销的方式实现，而写**部分还是得通过加锁**的方式来实现。一般情况下我们当然愿意采用 MVCC 来解决 读-写 操作并发执行的问题



#### **InnoDB引擎都有哪些锁？**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220419151114654.png" alt="image-20220419151114654" style="zoom: 50%;" />

#### 共享锁与排他锁

可以采取手动上锁的方式

`SELECT IN SHARE MODE`

`SELECT FOR UPDATE`

在MySQL8.0当中，UPDATE有一些新的参数比如`NOWAIT`  `SKIP LOCKED`  ，前者在目标数据被锁定的时候直接返回，而后者返回未被锁定的数据行



**INSERT**：插入操作的时候是不加锁的，但是会有一个叫做“隐式锁”的东西来保证这个操作在本事务提交之前不被其他事务获取



#### MySQL**行锁**

<!--细分一下：共享锁、独占锁；行锁（Record Lock / Gap Lock / Next-Key Lock）-->

多个事务操作同一行数据时，后来的事务处于阻塞等待状态。这样可以避免了脏读等数据一致性的问题。后来的事务可以操作其他行数据，解决了表锁高并发性能低的问题。

InnoDB只有在通过索引条件检索数据时使用行级锁，否则使用表锁！而模拟操作正是通过id去作为检索条件，而id又是MySQL自动创建的唯一索引，所以才忽略了行锁变表锁的情况。为了避免开启表锁，可以先给修改中需要where的字段添加索引。

但是在一张表里面执行大量更新的时候，MySQL为了减少不断加行锁、减行锁的操作锁带来的开销，会直接升级成表锁，不论是否对查找字段建立了索引。

**MySQL事务用的不是锁，而是读快照方式！默认的RR级别中，不涉及到给数据加锁**



MySQL中的三类行锁

- **Record Lock（记录锁）** ：锁住某一行记录
- **Gap Lock（间隙锁）** ：锁住一段左开右开的区间
- **Next-key Lock（临键锁）** ：锁住一段左开右闭的区间，这是加锁的基本单位，当查找的记录是不存在的时候会退化为双开的Gap Lock，它是加锁的基本单位  (a, b]



**共享锁：**select * from tableName where ... + lock in share more，可读可写但是不能抢占锁

**独占锁：**select * from tableName where ... + for update，其他线程不能读也不能写

**自动上锁：**当DML中设计到update、delete、insert等修改语句时会自动上写锁



**间隙锁：**当我们用范围条件检索数据，并请求共享或排他锁时，InnoDB会**给符合条件的已有数据记录的索引项加锁**。对于键值在条件范围内但并不存在的记录，叫做”间隙(GAP)"，InnoDB也会对这个"间隙"加锁，如下图所示。间隙锁的主要作用是为了防止出现幻读，但是它会把锁定范围扩大，也就是在无记录的间隙中上锁，·防止这些空间在一个事务的执行过程当中被插入数据导致幻读。

间隙锁的出现主要集中在同一个事务中**先delete后 insert的情况**下， 当我们通过一个参数去删除一条记录的时候， 如果参数在数据库中存在，那么这个时候产生的是普通行锁，锁住这个记录， 然后删除， 然后释放锁。

如果这条记录不存在，问题就来了， 数据库会扫描索引，发现这个记录不存在， 这个时候的delete语句获取到的就是一个间隙锁，然后数据库会向左扫描扫到第一个比给定参数小的值，向右扫描扫描到第一个比给定参数大的值， 然后以此为界，构建一个区间， 锁住整个区间内的数据， 一个特别容易出现死锁的间隙锁诞生了。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230211205227615.png" alt="image-20230211205227615" style="zoom: 33%;" />

四种情况（在特定的方向上找到第一个不满足的记录，然后进行锁的退化）：

- 唯一索引的等值查询：找到了就是Record Lock，找不到就是双开的Gap Lock
- 唯一索引的范围查询：比如id > 20，就是间隙锁(20, ~)
- 非唯一索引的等值查询：找到了的话，查找记录的左区间加 Next-key Lock，右区间加 Gap lock；没找到就是Gap Lock
- 非唯一索引的非等值查询：不会退化为Gap Lock，比如id > 20，就是[20, ~)







#### MySQL**表锁**

MyISAM只支持表锁，InnoDB正常情况下是以行锁为准

表锁的优势：开销小；加锁快；无死锁

表锁的劣势：锁粒度大，发生锁冲突的概率高，并发处理能力低

加锁的方式：**自动加锁。**查询操作（SELECT），会自动给涉及的所有表加读锁，**更新操作（UPDATE、DELETE、INSERT），会自动给涉及的表加写锁**



`LOCK TABLE t WRITE`

`LOCK TABLE t READ`

`UNLOCK TABLE t / TABLES`

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230227173028083.png" alt="image-20230227173028083" style="zoom: 33%;" />







**mysql锁表的解决，简而言之就是查看MySQL当前的process，然后将占有锁的那个process kill掉**
#查看进程id，然后用kill id杀掉进程

show processlist;
或
SELECT * FROM information_schema.PROCESSLIST；
#查询正在执行的进程

SELECT * FROM information_schema.PROCESSLIST where length(info) >0 ;

#查询是否锁表

show OPEN TABLES where In_use > 0;

#查看被锁住的

SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
#等待锁定

SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
#杀掉锁表进程

kill 5601



也可以显示加锁：

共享读锁：lock table tableName read;

独占写锁：lock table tableName write;

 批量解锁：unlock tables;



#### 意向锁（表级）

意向锁是由MySQL自动加的，当给一张表中的一些记录加上X或S锁的时候，会自动为其加上IX或IS锁

加锁的过程是先加意向锁后加真正意义上的锁

意向锁的意义是让行级锁与表级锁共存，协调二者之间的关系，让锁的粒度更加多样化，让行锁的性能更高

一个具体的场景：当我们给表中的一条数据上了X锁，引擎会自动地去给页或表加上一个IX锁。这个时候如果另外一个用户来想给这个表加表级的X锁，那么会先检查这个表是否存在IX锁，如果存在则放弃加表X锁。在没有intention lock的时候，需要全表遍历去查看是否有记录处于X锁当中，这样做的开销很大

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230227174434070.png" alt="image-20230227174434070" style="zoom: 33%;" />





#### 其他表锁

##### 自增锁AUTO-INC

设置了auto increment的字段的表在插入的时候给表上这个锁，目的是避免主键id重复

innodb_autoinc_lock_mode= 0/1/2

0 传统模式，加上AUTO-INC，直接给表加锁

1 连续模式，bulk insert 加上 AUTO-INC

2 交错模式，并发度比较高，不加锁，通过区间插入来保证主键不重复



##### MDL锁

Meta Data Lock元数据锁

在修改表结构的时候会加这个锁，这个锁和X/S锁都是阻塞的



#### MySQL死锁

死锁是并发系统中常见的问题，同样也会出现在数据库MySQL的并发读写请求场景中。当两个及以上的事务，双方都在等待对方释放已经持有的锁或因为加锁顺序不一致造成循环等待锁资源，就会出现“死锁”。常见的报错信息为 `Deadlock found when trying to get lock...`。

回顾一下死锁的四个条件，这里的死锁和操作的系统是一个道理的，互斥条件、不可剥夺条件、请求与保持条件和循环等待条件。要解决死锁很困难，开销很大，最直接的方式就是撤销操作，**应该重点从死锁的预防上面去考虑**。

1. 尽量让数据表中的数据检索都通过索引来完成，避免无效索引导致行锁升级为表锁。
2. 合理设计索引，尽量缩小锁的范围。
3. 尽量减少查询条件的范围，尽量避免间隙锁或缩小间隙锁的范围。
4. 尽量控制事务的大小，减少一次事务锁定的资源数量，缩短锁定资源的时间。
5. 如果一条SQL语句涉及事务加锁操作，则尽量将其放在整个事务的最后执行。
6. 尽可能使用低级别的事务隔离机制。



在MySQL中死锁并不是什么致命问题，设置一个合理的timeout就可以了，但最为重要的是从业务设计的角度出发在每次上锁的时候锁更少的数据



**死锁情况的复刻：**

```mysql
# 客户端A锁住id=1
mysql> start transaction; 
Query OK, 0 rows affected (0.00 sec) 
 
mysql> select * from account where id =1 for update; 
+----+--------+---------+ 
| id | name   | balance | 
+----+--------+---------+ 
|  1 | 张三   |     300 | 
+----+--------+---------+ 
1 row in set (0.00 sec) 

# 客户端B锁住id=2
mysql> start transaction; 
Query OK, 0 rows affected (0.00 sec) 
 
mysql> select * from account where id =2 for update; 
+----+--------+---------+ 
| id | name   | balance | 
+----+--------+---------+ 
|  2 |  李四   |     350 | 
+----+--------+---------+ 
1 row in set (0.00 sec) 

# 然后客户端A和B分别请求update2和1
mysql> select * from account where id =2 for update; 
mysql> select * from account where id =1 for update; 
```





#### 悲观锁与乐观锁

这两个概念偏向于锁的设计思想

悲观锁使用在写的场景较多，必须通过传统的加锁方式来实现

乐观锁在读的场景较多，通常使用 version字段 + CAS操作 控制一下就够了





#### 锁结构



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230228111512966.png" alt="image-20230228111512966" style="zoom:33%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230228111839449.png" alt="image-20230228111839449" style="zoom:33%;" />





#### 锁监控

MySQL中存在一些一些语句能观察锁的运行状况例如：

`show status like 'innodb_row_lock%';`



同时information_schema这个库中的INNODB_TRX这张表保存了锁在执行过程中的一些信息









#### **聊一聊MVCC的原理，它是如何实现并发控制的？**

<!--MVCC可以解决部分的幻读问题-->

MVCC 的实现依赖于：隐藏字段、Undo Log、Read View。其中两个隐藏字段记录了每条记录的事务ID（trx_id）和指向该记录的上一个版本的**Roll Pointer**回滚指针（能定位到这个数据上一次被修改之前的值），这两个字段对用户不可见

Undo Log则是多版本的保证，当多个事务对某条记录进行操作之后，在undo log里都有体现，能够根据版本做还原。最后，Read View是做版本的管理，它是根据隐藏字段的事务ID和当前事务的ID来进行版本管理

**MVCC解决不可重复读**问题的方案就是，从Read View中查出历史版本的数据（快照读）并返回给用户

MVCC和行锁一道，在RR隔离级别下能**解决部分的幻读问题**。读操作只读该事务开始前的数据库的快照，可以解决脏读，幻读，不可重复读等事务隔离问题，但不能解决更新丢失问题。**这是说MVCC不能解决在多事务环境下的对数据库的修改操作，修改是直接改数据，而不会走快照**

Read View和事务是一对一的关系，维护了当前活跃事务的ID，也就是还没有结束的事务。MVCC无法实现`READ UNCOMMITED`和`SERIALISABLE`，前者是无差别返回最新版本数据，而后者通过高开销的锁方式实现

![image-20220419205638616](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220419205638616.png)

READ VIEW中的重要参数：

`creator_trx_id` 创建这个 Read View 的事务 ID

`trx_ids` 表示在生成ReadView时当前系统中活跃的读写事务的事务id列表 

`up_limit_id` 活跃的事务中最小的事务 ID

`low_limit_id` 表示生成ReadView时系统中应该分配给下一个事务的 id 值。low_limit_id 是系 统最大的事务id值，这里要注意是系统中的事务id，需要区别于正在活跃的事务ID

MVCC如何解决不可重复读和幻读问题：

总结起来，就是只能返回已经提交了的事务的数据给到用户。

具体流程：

每当我们进行一个事务操作的时候都会生成一个事务trx_id（可以粗粒度这么理解），同时数据库中的每条记录同时也有一个事务trx_id，这个id是最后一次修改过该数据的事务的id。

假设我们事务要select  * from [table_name]，数据库第一步会真的select全部数据出来，接下来就是判断哪些数据能返回给当前用户，就是通过事务数据和用户的事务id来进行判断

1、数据库会维护一个当前活跃事务（即还未commit的事务）的id列表，如果select * 的数据中有事务id大于`low_limit_id`的话，这些数据是不会返回的，因为说明正在操作该数据的事务甚至在当前用户开始事务之后才被开始

2、如果数据的事务id小于`up_limit_id`那么这些数据可以返回，因为这些操作这些数据的事务已经提交了

3、如果数据的事务id处在`up_limit_id`和`low_limit_id`之间，并且事务id还存在于`trx_ids`表中，那么这些数据是不能被返回的，因为正在操作它们的事务还没有commit

4、如果数据的事务id等于`creator_trx_id`，也就是当前事务的id，那么这些数据能返回，因为该事务永远能访问自己正在操作的数据

*事务id的生成是随着时间的增长而单调递增的



##### MVCC是如何解决部分幻读问题的？

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220805231047588.png" alt="image-20220805231047588" style="zoom:25%;" />

增删改属于当前读，上的是写锁

MVCC用的快照读，能保证读的时候走快照，但是增删改并不会走快照。那么导致的问题就是在执行增删改操作的时候会操作到快照以外的数据

所以要做到完全的解决幻读只能给数据上行锁，实现串行化操作





##### **你了解mysql事务锁的结构吗，它的原理是什么样的？**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220419214102431.png" alt="image-20220419214102431" style="zoom: 25%;" />



在type_mode字段中，通过lock_mode的不同值来表示X、S、IS、IX等锁类型

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220419215435915.png" alt="image-20220419215435915" style="zoom:33%;" />



## 7. 并发与大流量应对



#### <u>**如何看待MySQL的高并发问题？**</u>

数据库的并发问题和其他服务的并发问题存在一些区别。数据库在高并发下是有“倾向”的，读的并发是远大于写的。所以在解决MySQL并发问题时应该将读写分开看待，给予读更多的资源。常用的架构就是基于**主从复制**的**读写分离**。主从复制指的是一台主机配合多台从服务器，并且主服务器负责的功能有两个，一方面是写bin log主导主从之间的复制和同步，另外一方面是接受写数据的流量。



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220420211025207.png" alt="image-20220420211025207" style="zoom:50%;" />

从较为宽泛的概念上来说，MySQL的性能优化应该**首先从SQL优化上来考虑**，也就是设计好合理的表体系和结构以及索引。上文中提到，数据库系统的读请求通常都是高于修改请求的，所以通过合理的索引设计能够改善大部分的流量并且这也是成本最低的方式。下表是一些各种优化高并发trade off。

![image-20220420215214737](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220420215214737.png)



#### <u>MySQL集群/主从都存在哪些拓扑？</u>





#### <u>MySQL主从复制的原理是什么，如何实现的？</u>

![image-20220420211507664](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220420211507664.png)

主从同步的原理是基于bin log进行数据同步的。在主从复制过程中，会基于 3 个线程（主机log dump线程、从机I/O线程、从机SQL线程）来操作，一个主库线程，两个从库线程。

全过程的概述：

1. 主机负责写（修改）操作，每次的修改操作都备案到bin log当中
2. 主机定时或或按照其他策略向其他从服务器做bin log同步，从服务器拿到bin log后写入本地的relay log
3. 最后SQL线程从relay log中拿取进行到记录之后将数据写入从库

![image-20220420212607338](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220420212607338.png)



#### <u>基于MyCat的一主一从实践</u>

参见MyCat部分



#### <u>binlog的格式</u>

**STATEMENT模式** (基于SQL语句的复制(statement-based replication, SBR))，是mysql中默认的binlog格式

pros：

1. 不需要记录每一行的变化，减少了binlog日志量
2. 文件较小 binlog中包含了所有数据库更改信息，可以据此来审核数据库的安全等情况 （出事了看看问题sql是谁写的，明确锅谁来背）
3. binlog可以用于实时的还原，而不仅仅用于复制

cons：

1. 不是所有的UPDATE语句都能被复制，尤其是包含不确定操作的时候，以及当数据发生了改变再update的情况
2. 使用以下函数的语句也无法被复制:LOAD_FILE()、UUID()、USER()、FOUND_ROWS()、SYSDATE()
3. 更多的**行级锁产生**，开销大
   1. INSERT ... SELECT 会产生比 RBR **更多的行级锁**，需要遍历表从而生成了锁
   2. 复制需要进行全表扫描(WHERE 语句中没有使用到索引)的 UPDATE 时，需要比 RBR 请求**更多的行级锁**

4. 执行复杂语句得到的结果为简单数据，这种情况凭白增大了系统的开销
5. 数据表必须严格主服务器保持一致才行，否则通常语句来进行还原很有可能会导致复制出错



**Row模式，基于行的复制(row-based replication, RBR)**

直接把数据记录下来，包括哪条数据被修改了，修改成什么样了。

pros：

1. 执行 INSERT，UPDATE，DELETE 语句时**锁更少**。因为通过行的精确匹配来做到免除遍历加锁这个过程。
2. 任何情况都可以被复制，这对复制来说是最 安全可靠的，不存在数据错误的情况。(比如:不会出现某些特定情况下 的存储过程、function、trigger的调用和触发无法被正确复制的问题)
3. 从服务器上采用**多线程来执行复制**成为可能，因为是直接插入数据，不需要执行语句走加锁这个过程

cons：

1. binlog 大了很多，因为直接记录的数据
2. 复杂的回滚时 binlog 中会包含大量的数据
3. 主服务器上执行 UPDATE 语句时，所有发生变化的记录都会写到 binlog 中，而 SBR 只会写一 次，这会导致<u>频繁发生 binlog 的并发写</u>问题
4. 无法从 binlog 中看到都复制了些什么语句



**MIXED模式，混合模式复制(mixed-based replication, MBR)**

在Mixed模式下，一般的语句修改使用statment格式保存binlog。如一些函数，statement无法完成主从复 制的操作，则采用row格式保存binlog。

MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志形式，也就是在Statement和Row之间选 择一种。

在开销和性能中间采取一种平衡。



#### <u>主从数据同步的一致性问题是如何产生的又该如何解决？</u>

主从数据不一致的根本是对从机没能及时从relay log中恢复数据。诱因主要有下面两个：

1. 进行主从同步的内容是二进制日志，它是一个文件，在进行**网络传输**的过程中就一定会产生延迟，这样就可能造成用户在从库上读取的数据不是最新的数据，也就是主从同步中的不一致性问题
2. 从机在消费relay log的时候耗时太长，可能是**从机的性能**问题导致的。要么是从机本身配置不够，或者是大部分资源被用来处理用户请求了



**关于减少主从延迟**

1. 降低多线程大事务并发的概率，优化业务逻辑
2. 优化SQL，避免慢SQL， 减少批量操作 
3. 提高物理带宽、从机服务器的配置
4. 实时性要求的业务读强制走主库，从库只做灾备，备份



**读写分离的背景下解决数据一致性问题**

读写分离情况下，解决主从同步中数据不一致的问题， 就是解决主从之间**数据复制方式**的问题，如果按照数据一致性从弱到强来进行划分，有以下 3 种复制方式。

**异步复制，弱一致性**

![image-20220421132412220](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220421132412220.png)

在主机写完bin log并且事务提交之前，从机读bin log到relay log后执行数据同步操作。很显然，当写完bin log后事务失败，或者是log的传输、消费耗时很久，都无法保证数据一致性。



**半同步复杂，勉强能接受的一致性**

![image-20220421132638484](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220421132638484.png)

半同步复制和异步复制的核心区别在于，主机要收到从机完成relay log的ack后才提交事务，这样一来保证了事务过程中的一致。但是这种硬等待开销大，并且在实时性要求高的场景下难以满足要求。



**组复制，强一致性**

![image-20220421132846457](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220421132846457.png)

首先我们将多个节点共同组成一个复制组，在 执行读写(RW)事务 的时候，需要通过一致性协议层 (Consensus 层)的同意，也就是读写事务想要进行提交，必须要经过组里“大多数人”(对应 Node 节 点)的同意，大多数指的是同意的节点数量需要大于 (N/2+1)，这样才可以进行提交，而不是原发起 方一个说了算。而针对 只读(RO)事务 则不需要经过组内同意，直接 COMMIT 即可。

在一个复制组内有多个节点组成，它们各自维护了自己的数据副本，并且在一致性协议层实现了原子消息和全局有序消息，从而保证组内数据的一致性。

MGR 将 MySQL 带入了数据强一致性的时代，是一个划时代的创新，其中一个重要的原因就是MGR 是基 于 Paxos 协议的。**Paxos** 算法是由 2013 年的图灵奖获得者 Leslie Lamport 于 1990 年提出的，有关这个算 法的决策机制可以搜一下。事实上，Paxos 算法提出来之后就作为 被广泛应用，比如 Apache 的 ZooKeeper 也是基于 Paxos 实现的。

组复制需要条件，在较大规模的场景下组复制更加能够发挥出其优势，保证集群当中又一半以上的机器持有最新的数据。这种机制使得从机对relay log的消费延迟得到容忍，也能保证最新的数据是冗余备份的，能够被访问到且安全。







## 8. 架构及运行原理

 <img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220417182655660.png" alt="image-20220417182655660" style="zoom:67%;" />

MySQL8以上已经把Caches&Buffers这块给砍掉了（鸡肋），其他部分依旧



## 9. MyCat

MyCat相当于是MySQL访问的中间件，配置MyCat后用户对MySQL的访问不再直接走MySQL的端口，而是MyCat端口；MyCat会对用户的请求数据进行拦截，然后根据配置的规则（分库、表、读写分离等）将请求发送到对应的MySQL Server中去

MyCat能提供到的功能包括

1. 读写分离
2. 分布式数据库（数据库/表拆分）
3. 整合多数据源（Not Only MySQL）



#### 原理图，核心概念——”拦截“

![image-20220424222523798](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220424222523798.png)

https://github.com/MyCATApache/Mycat-Server

通过配置文件的方式来定义好分库分表的规则（当然你得自己先建好数据库及对应的表），然后再代码层面做到数据和业务代码解耦

mycat的数据库逻辑配置、分库分表等通过schema.xml来确定逻辑库和物理库的对应关系



MyCat属于MySQL的一种server，所以登录mysql的方式就是用mysql登录命令，但是指定到mycat的端口。

mysql -umycat -p123456 -P8066 -h[your ip]      数据端口

mysql -umycat -p123456 -P9066 -h[your ip]      管理员端口

mycat的主从复制从连接点开始进行复制，而redis是从头开始把全部数据写入RDB传给从机进行复制



配置主从同步

配置读写分离

1. 在schema.xml中定义了逻辑库和物理库对应
2. 在schema.xml中定义datahost的balance，1为单主单从，3为双主双从



双主双从

1. 各自的主从配置
2. 两个主机之间配置主备复制
3. 将四台及其都配置进schema.xml并将balance值改为1（3也可以，参考下表中mycat配置常用参数）



**示例配置**

datanode：该标签定义了MyCat中的数据节点，也就是我们通常所说的数据分片。一个datanode标签就是一个独立的数据分片

datahost：该标签则具体定义分片下的一个物理数据库

```xml
// user.xml
<user name="admin">
    <property name="password">admin</property>
    <property name="schemas">TESTDB</property>
</user>
<user name="user">
    <property name="password">user</property>
    <property name="schemas">TESTDB</property>
    <property name="readOnly">true</property>
</user>

// schema.xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">
    
    <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
        <!-- 这里不配置，代表所有的表分片到dn1节点-->
    </schema>

    <dataNode name="dn1" dataHost="localhost1" database="sync_test" />
    
    <dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <!-- can have multi write hosts -->
        <writeHost host="hostM1" url="192.168.10.1:3306" user="root"  password="123456">
            <!-- can have multi read hosts -->
            <readHost host="hostS2" url="192.168.10.2:3306" user="root" password="123456" />
        </writeHost>
    </dataHost>

</mycat:schema>
```



``` bash
# balance属性负载均衡类型

balance=”0”, 不开启读写分离机制，所有读操作都发送到当前可用的 writeHost 上
balance=”1”，全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡
balance=”2”，所有读操作都随机的在 writeHost、 readhost 上分发。
balance=”3”， 所有读请求随机的分发到 wiriterHost 对应的 readhost 执行,writerHost 不负担读压力


# writeType属性负载均衡类型，目前的取值有3种：

1.writeType="0", 所有写操作发送到配置的第一个writeHost，第一个挂了切到还生存的第二个writeHost，重新启动后已切换后的为准，切换记录在配置文件中:dnindex.properties.

2.writeType="1"，所有写操作都随机的发送到配置的writeHost，1.5以后废弃不推荐。

3.writeType="2"，不执行写操作

#switchType属性

-1 表示不自动切换(普通的读写分离最好是不要自动切换，避免了将数据写进slave的可能性。除非是双主)

1 默认值，自动切换

2 于MySQL主从同步的状态决定是否切换,心跳语句为 show slave status

3 基于MySQLgalarycluster的切换机制（适合集群）（1.4.1）心跳语句为show status like‘wsrep%’

# dbType属性
指定后端连接的数据库类型，目前支持二进制的mysql协议，还有其他使用
JDBC连接的数据库。例如：mongodb、oracle、spark等
```



#### MySQL 8.0 新增的JSON字段

可以选择直接把字段设置为JSON；或者字段类型为text然后用JSON相关函数把数据转化为JSON格式存入data字段

**json字段格式**

```
["abc", 10, null, true, false]

{"k1": "value", "k2": 10}

["12:18:29.000000", "2015-07-29", "2015-07-29 12:18:29.000000"]

嵌套也ok
[99, {"id": "HK500", "cost": 75.99}, ["hot", "cold"]]

{"k1": "value", "k2": [10, 20]}
```

**基本操作**

```sql
# 创建表  字段定义为JSON
mysql> CREATE TABLE t1 (jdoc JSON);
Query OK, 0 rows affected (0.20 sec)

# 手动输入标准JSON形式  如上面代码块中所示
mysql> INSERT INTO t1 VALUES('{"key1": "value1", "key2": "value2"}');
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO t1 VALUES('[1, 2,');
ERROR 3140 (22032) at line 2: Invalid JSON text:
# output:"Invalid value." at position 6 in value (or column) '[1, 2,'.

```



**更加推荐使用函数，方便快捷**

The `JSON_TYPE()` function expects a JSON argument and attempts to parse it into a JSON value. It returns the value's JSON type if it is valid and produces an error otherwise:

```sql
mysql> SELECT JSON_TYPE('["a", "b", 1]');
+----------------------------+
| JSON_TYPE('["a", "b", 1]') |
+----------------------------+
| ARRAY                      |
+----------------------------+

mysql> SELECT JSON_TYPE('"hello"');
+----------------------+
| JSON_TYPE('"hello"') |
+----------------------+
| STRING               |
+----------------------+

mysql> SELECT JSON_TYPE('hello');
# ERROR 3146 (22032): Invalid data type for JSON data in argument 1
# to function json_type; a JSON string or JSON type is required.
```



`JSON_ARRAY()` takes a (possibly empty) list of values and returns a JSON array containing those values:

```sql
mysql> SELECT JSON_ARRAY('a', 1, NOW());
+----------------------------------------+
| JSON_ARRAY('a', 1, NOW())              |
+----------------------------------------+
| ["a", 1, "2015-07-27 09:43:47.000000"] |
+----------------------------------------+
```



[`JSON_OBJECT()`](https://dev.mysql.com/doc/refman/8.0/en/json-creation-functions.html#function_json-object) takes a (possibly empty) list of key-value pairs and returns a JSON object containing those pairs:

```sql
mysql> SELECT JSON_OBJECT('key1', 1, 'key2', 'abc');
+---------------------------------------+
| JSON_OBJECT('key1', 1, 'key2', 'abc') |
+---------------------------------------+
| {"key1": 1, "key2": "abc"}            |
+---------------------------------------+
```





## 10. MyBatisPlus篇

### MyBatis是核心，MBP只是基于此的扩展

每次复习MyBatis的时候回顾一下这些问题

![image-20230314235827136](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230314235827136.png)





https://pdai.tech/md/framework/orm-mybatis/mybatis-overview.html













**为什么用mbp？**

1. 简化了数据库的连接和CRUD操作，强大的sql构造器
2. 支持主键自动生成，覆盖多种主键策略模式
3. 对字段进行额外操作，比如时间生成，自动填充等（比如对updateTime这种字段做自动填充）
4. 额外的功能，例如分页查询和性能分析插件
5. 安全机制，例如内置 Sql 注入剥离器



**MBP对表的匹配**

通过BaseMapper的泛型，然后指定数据库表前缀，保证对象与表中字段同名

```java
public interface UserMapper extends BaseMapper<User> {
  // 
}
```

基本操作及其返回值

```java
// insert操作返回该操作影响的行数
baseMapper.insert()
// update返回匹配的行数
baseMapper.update
```



**乐观锁**

在表中设置一个version字段，并为对象类中该字段使用@Version注解



**元对象处理器**

进行字典填充的时候，一方面需要在实体类字段上进行注解，同时也要在元对象处理器上做填充类型声明

```java
public class MyMetaObjectHandler implements MetaObjectHandler
  
this.setFieldValByName("deleted", 0, metaObject);

```



**性能监控设置**



**获取自增主键值**

在实体类上的自增主键中用@TableID注解，并声明是自增主键



**全局配置规则**

@TableName 用来声明表名，但可以在全局配置中指定表名与实体类名的对应逻辑

配置全局的主键自增



**常用注解**

@TableName(value = )

@TableField(value = , exsit = , select = , fill = )

@TableId(value = , type = )

@TableLogic 逻辑删除

@Version 乐观锁的版本控制



#### MBP**的缓存机制**

mybatis会缓存先前执行过的SQL语句的返回结果，减少SQL执行带来的损耗

缓存是由sqlsession对象来完成的，所以只有在同一个session中查相同语句才会实现缓存

**二级缓存机制**

一级由sqlsession属于用户线程缓存

二级由sqlsessionFactory属于全局缓存

![img](https://img2018.cnblogs.com/blog/1515111/201908/1515111-20190810211431621-1800471104.png)

 sqlsession关闭的时候，它所属的缓存会托管到factory当中去



**mybatisplus缓存**

一级缓存是**SqlSession级别**的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。 一级缓存是默认开启的不用配置。

二级缓存是**mapper级别**的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。二级缓存的开启（实体类必须序列化），然后在配置文件里面配置。

<img src="https://img2018.cnblogs.com/blog/1515111/201908/1515111-20190810211454258-1861395227.png" alt="img" style="zoom: 50%;" />

**第一级缓存失效问题**

1. 探究更新对一级缓存失效的影响： 由上面的分析结论可知，我们每次执行 **update** 方法时，都会先刷新一级缓存，因为是同一个 SqlSession, 所以是由同一个 Map 进行存储的，所以此时一级缓存会失效
2. 探究不同的 SqlSession 对一级缓存的影响： 这个也就比较好理解了，因为不同的 SqlSession 会有不同的Map 存储一级缓存，然而 SqlSession 之间也不会共享，所以此时也就不存在相同的一级缓存
3. 同一个 SqlSession 使用不同的查询操作： 这个论点就需要从缓存的构成角度来讲了，我们通过 cacheKey 可知，一级缓存命中的必要条件是两个 cacheKey 相同，要使得 cacheKey 相同，就需要使 cacheKey 里面的值相同。<u>createCacheKey(ms, parameter, rowBounds, boundSql);</u>

```java
// 有sql请求来了先query构建缓存cachekey
// 将cachekey和sql发给query重载方法，判断是走缓存还是数据库
// 执行queryFromDatabase，如果走数据库的话

@Override
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
  BoundSql boundSql = ms.getBoundSql(parameter);
  // 创建缓存
  CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
  return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
}

@SuppressWarnings("unchecked")
@Override
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  ...
  list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
  if (list != null) {
      // 这个主要是处理存储过程用的。
      handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
      } else {
      list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
  }
  ...
}

// queryFromDatabase 方法
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  List<E> list;
  localCache.putObject(key, EXECUTION_PLACEHOLDER);
  try {
    list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
  } finally {
    localCache.removeObject(key);
  }
  localCache.putObject(key, list);
  if (ms.getStatementType() == StatementType.CALLABLE) {
    localOutputParameterCache.putObject(key, parameter);
  }
  return list;
}
```





第一级缓存清除

最终 是调用map.clear( )方法来对hashmap做清除。在执行update语句的时候会自动清除原来的缓存

![img](https://img2018.cnblogs.com/blog/1515111/201908/1515111-20190808213656063-467330727.png)





**第二级缓存失效问题**

下列情况下容易导致第二级缓存的失效

多表联查容易导致脏数据，联查的sql被看成是一个pattern，某个表数据修改之后并不会通知另外一个表的mapper。**解决方案：**如果是两个mapper命名空间的话，可以使用 `<cache-ref>`来把一个命名空间指向另外一个命名空间，从而消除上述的影响，再次执行，就可以查询到正确的数据。也就是两个Mapper公用一个Cache，这样有变动就能做到更新告知了



**第二级缓存源码解析**

![image-20220526140429612](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220526140429612.png)



**是否应该使用第二级缓存？**

一级缓存直接用hashmap来存效率高，开销也不大；但是二级缓存是表级的缓存，开销比较大，而且在**表连接场景下会大量产生脏数据**，可能最后ROI并不高。





#### MyBatis的流程原理

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220721233749751.png" alt="image-20220721233749751" style="zoom: 33%;" />

分两个部分来看这个问题，一个是初始化流程，另外一个是运行流程

##### 初始化流程





##### 运行流程





#### MBP的AR模式

Active Record(活动记录)，是一种领域模型模式，特点是一个模型类对应关系型数据库中的一个表，而模型类的一个实例对应表中的一行记录。支持pActiveRecord形式的调用，实体类只需要继承Model类即可进行强大的CRUD操作

AR模式就是把一个表转化为一个对象，表中的每条记录相当于是该对象的一个实例

**AR模式下的开启：**

1、创建项目继承MP

2、创建数据库实体类，继承Model类

3、实体类中复写pkVal方法

4、创建Mapper接口并继承BaseMapper接口

5、创建Spring对象，让Spring对象完成对Mapper层的实例化扫描

6、创建实体类对象，直接调用实体类从Model中继承的数据库方法完成数据库操作



#### **MyBatisPlus自动匹配分库分表**

MBP通过拦截器对sql进行解析，将sql查询自动地匹配到对应的表中去（动态地更换表名）

MyBatis中拦截器分下面这些类型，但是没有官方实现分库分表，还得看MBP：

1. **Executor** (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed) 拦截执行器的方法；
2. **ParameterHandler** (getParameterObject, setParameters) 拦截参数的处理；
3. **ResultSetHandler** (handleResultSets, handleOutputParameters) 拦截结果集的处理；
4. **StatementHandler** (prepare, parameterize, batch, update, query) 拦截Sql语法构建的处理；



**涉及到的类和方法：**

PaginationInterceptor 主入口

abstract class AbstractSqlParserHandler

class DynamicTableNameParser

parser方法



**具体的步骤：**

1. 配置表的映射关系，在ITableNameHandler的dynamicTableName方法中编写替换逻辑， 将映射关系加到 Map<String, ITableNameHandler> tableNameHandlerMap中
2. 然后构建一个DynamicTableNameParser类，将map定义进去
3. 再将DynamicTableNameParser加到 **PaginationInterceptor**的变量sqlParserList集合中即可



**面试回答：**

主要用的还是MyBatis的StatementHandler拦截器，对sql进行拦截解析

在MBP的PaginationInterceptor类里面有一个sqlParserList集合，里面放了具体的解析处理类DynamicTableNameParser，在parser里面会定义一个hashMap，在这个类的parser方法里头进行表名的匹配



## 11. MySQL分库分表的实践

#### **不分库会导致什么样的问题？**

- 数据量大，IO瓶颈
- 数据量大很多时候伴随着访问量大，单台机器的网络IO成为瓶颈

- 索引开销过大，树高变大，叶子节点占有的存储空间变大



#### 分库分表都有哪些方案？

**垂直拆分：**

将一个库当中的很多表按照业务的不同划分进不同的数据库

将一张表的字段进行拆分，划分成几个不同的业务，放到不同的表中去

总的来说，垂直拆封就是库和表的结构发生了变化

**水平拆分：**

库和表的结构不发生变化，只是按照规则将一定的表/数据放到新的数据库/表中去

表的水平拆分可以根据对某个字段进行range或者hash来进行

range速度很快很直接，但是可能会导致热点数据集中在一台机器上，带来不必要的负担

hash不会导致热点数据集中在一张表里的问题，但是再次分表的时候会导致历史数据的迁移

> 一致性哈希加强数据的平衡



#### 分库分表后会带来的问题及解决方案

**带来的问题：**

0. 事务问题

​		只能采用分布式事务，可以借助缓存或者seata这种集成式的中间件

1. 跨库关联

   解决这一问题可以分两次查询实现

2. 排序问题

   跨节点的count,order by,group by以及聚合函数等问题：可以分别在各个节点上得到结果后在应用程序端进行合并

3. 分页问题

   在每个节点查到对应结果后，在代码端汇聚再分页

4. 分布式ID

   UUID或者SnowFlake算法



#### Example：亿级表进行不停机热拆分

数据均衡问题

分表方案

双写设计

拆分技术选型





## 12. Canal

Canal就是一个管道，伪装成一台MySQL Slave从Master那边通过**dump**协议获取**binlog**得到数据增量

与Kafka对接的案例 https://github.com/alibaba/canal/wiki/Canal-Kafka-RocketMQ-QuickStart

<img src="https://camo.githubusercontent.com/63881e271f889d4a424c55cea2f9c2065f63494fecac58432eac415f6e47e959/68747470733a2f2f696d672d626c6f672e6373646e696d672e636e2f32303139313130343130313733353934372e706e67" alt="img" style="zoom: 50%;" />

![image-20220803211546399](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220803211546399.png)



## 13. SQL语法

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230217003414319.png" alt="image-20230217003414319" style="zoom:50%;" />

#### SQL的执行顺序

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230218142850610.png" alt="image-20230218142850610" style="zoom:67%;" />

SELECT

###

FROM

JOIN

WHERE

GROUP BY

###

HAVING

LIMIT





#### 多表查询/关联查询

<img src="https://www.runoob.com/wp-content/uploads/2019/01/sql-join.png" alt="img" style="zoom:50%;" />

99版语法中JOIN的七种操作方式



##### 笛卡尔积错误



##### 等值连接VS非等值连接

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230501214858040.png" alt="image-20230501214858040" style="zoom:33%;" />



##### 内连接VS外连接

现在假设A、B两张表做连接，A表中有一条记录在B表中是不存在的（其关联字段的值为NULL），B中也有记录是A中不存在的

内连接返回AB列表中ON条件后存在关联的全部字段 JOIN / INNER JOIN

左外连接返回A表中的全部字段，包括与B不存在关联的 LEFT JOIN / LEFT OUTER JOIN

右外连接返回B表中的全部字段，包括与A不存在关联的 RIGHT JOIN / RIGHT OUTER JOIN

满外连接相当于是内连接、左右外连接加一起后去重 FULL OUTER JOIN



##### 自连接VS非自连接

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230501232711306.png" alt="image-20230501232711306" style="zoom:33%;" />



##### 小表驱动大表



##### cases

连多张表

![image-20230502173601562](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230502173601562.png)



自连接

![image-20230502175435143](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230502175435143.png)



外连接只取差值

![image-20230502180050206](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230502180050206.png)



#### 聚合函数

SUM

AVG

MIN

MAX

COUNT



#### GROUP BY



#### HAVING

放在GROUP BY后面

可以使用聚合函数做筛选AVG(salary) > 10,000

HAVING的执行效率要高于WHERE，在SQL的执行顺序当中是先WHERE再GROUP BY。WHERE先进行筛选后会让更少的数据进行GROUP BY操作，而在分组之后进行HAVING会处理更多的数据



#### 分页操作

92/99版本的SQL在处理分页上面有一点点语法层面的不同，如下：

LIMIT [offset], [number_per_page]

LIMIT [number_per_page] OFFSET [offset]



#### 子查询

**Example**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222150133974.png" alt="image-20230222150133974" style="zoom:33%;" />

```sql
# 这个用表连接也能做，表连接的开销较大
SELECT e1.last_name, e1.salary
FROM employee e1, employee e2
WHERE e1.salary > e2.salary AND e2.last_name = 'Abel';
```



**分类**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222150829618.png" alt="image-20230222150829618" style="zoom:33%;" />

**单行子查询**

子查询返回单值，通常被当作外层循环的where匹配条件



**多行子查询**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222153352193.png" alt="image-20230222153352193" style="zoom:33%;" />

聚合函数嵌套的处理

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222154337550.png" alt="image-20230222154337550" style="zoom:33%;" />



**相关子查询**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222155619352.png" alt="image-20230222155619352" style="zoom:33%;" />

![image-20230222160154511](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222160154511.png)

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222160859769.png" alt="image-20230222160859769" style="zoom: 33%;" />



#### UNION / ALL

UNION用于纵向地将两个SELECT查询的数据结合起来，**必须保证两个SELECT中的字段是相同的**

UNION会去重

UNION ALL展示出全部的数据不去重





##### Binding Peek  绑定sql

这样一来这一系列的sql会复用同一个执行计划execute schema，能够节约指定执行计划这个步骤所带来的开销

```java
PreparedStatement pstmt = con.prepareStatement("UPDATE employees SET salay = ? WHERE id = ?"); 
pstmt.setBigDecimal(1, 15.00); 
pstmt.setInt(2, 110592); //result statmement: UPDATE employees SET salay = 15.00 WHERE id = 110592 pstmt.executeQuery();
```

