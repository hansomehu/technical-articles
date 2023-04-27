---
layout: post
title: "es快速入门"
permalink: /es-crash
---



#### ES与Solr的选型









#### **ES的存储结构与传统数据库的对应关系**

为了强化ES作为**文档检索引擎**的特性，逐渐地去除了Type类型的作用，在ES7.x中已经完全废除了Type的使用

![image-20220517102806806](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220517102806806.png)



![image-20230325104428388](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230325104428388.png)





#### **倒排索引与正排**

倒排就是根据关键字去找到对应所在文档的ID

而正排就是比较传统的，先去海量文档中检索再返回文档ID

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220517103145456.png" alt="image-20220517103145456" style="zoom:33%;" />



#### REST操作

以下的操作都没有使用type这个概念，直接就是一个业务类型一个index（table/database），然后属于该业务类型的数据存放在这里面



##### **基本操作**

```js
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```

官方文档翻译版 https://www.elastic.co/guide/cn/elasticsearch/guide/current/data-in-data-out.html



GET /_cat/indices?v		展示所有的索引的详细信息



DELETE /indice_name		删除一个索引



POST /indice_name/_doc		向指定索引添加文档数据



POST /indice_name/_doc/id		指定id



GET /indice_name/_doc/id		根据id来查文档记录



GET /indeci_name/_search		查询索引下全部的数据



PUT /indice_name/_doc/id		修改指定数据，全量数据修改（幂等）



POST /indice_name/_update/id		局部更新

老语法 **POST /website/blog/1/_update**

**ES中的文档一旦创建是无法被修改的，**在后台更新操作执行的逻辑是先删除后新建文档

```json
{
  "doc":{
    "field":"value",
    ...
  }
}
```



DELETE /indice_name/_doc/id		删除指定数据

老语法 **DELETE /website/blog/123**



GET /website/blog (type的概念可以直接省略掉) /123?_source=title,text 查询指定字段

```http
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```



GET /website/blog/123/_source 不要任何元数据

```http
{
   "title": "My first blog entry",
   "text":  "Just trying this out...",
   "date":  "2014/01/01"
}
```



PUT /website/blog/123/_create 创建新的文档

如果创建失败则返回 409 Conflict



_mget批量查询

index相同

```http
GET /website/blog/_mget
{
   "ids" : [ "2", "1" ]
}
```

index不同

```http
GET /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```



bulk批量操作，在一个bulk中可以定义一个流式的操作群

```http
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} 
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} } 
```



GET /_search?size=5&from=10 分页查询



#### **高级查询**

https://www.elastic.co/guide/cn/elasticsearch/guide/current/_most_important_queries.html

https://www.elastic.co/guide/cn/elasticsearch/guide/current/combining-queries-together.html



GET /indice_name/_search		

条件检索

```json
{
  "query":{
    "match": {  // match_all 大括号中为空，即全量查询 match_phrase 匹配分词
      "title": "",
      "category": ""
    }
  },
  "from": , // 分页，多少页
  "size": , // 分页，每页多少数据
  "_source": [title],  // 只从title字段中做检索
	"sort": {
    "price":{
      "order": "desc" // order in desc
    }
  }
}
```

多条件检索

```json
{
  "query":{
    "bool":{
      "must":[ // should(or) all in precise
        {
          "match":{
            "field-1":"value"
          }
        },
        {
          "match":{
            "field-2":"value"
          }
        }
      ],
      "filter":{	// range query
        "range":{
          "price":{
            "gt":5000
          }
        }
      }
    }
  }
}
```

结果高量

```json
{
  "query":{
    
  },
  "highlight":{
    "fields":{
      "field-1":{
        // highlight style
      },
      ...
    }
  }
}
```

聚合操作

```json
{
  "aggs":{
    "group_name":{ // 本次聚合操作名称 随意起
      "terms":{	// 分组   avg求平均值
        "field":"price"	// 操作的字段
      }
    }
  }
}
```



编辑索引映射 

PUT /indice_name/_mapping

```json
{
  "properties":{
    "name":{
      "type":"text",  // Full text index
      "index":true
    },
    "sex":{
      "type":"keyword",		// keyword/phrases match
      "index":true
    },
    "tel":{
      "type":"keyword",
      "index":false   // this field does not support indexing
    }   
  }
}
```





#### Java API操作

当前的Java API推荐使用下面的新Client，但是也不排除仍然有很多公司在使用HighRestClientAPI

https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/introduction.html







#### 环境配置

**Linux单机配置**

`修改/opt/module/es/config/elasticsearch.yml文件`

```bash
# 开启远程访问所需配置
cluster.name: elasticsearch
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"]
```

`修改/etc/security/limits.conf`

```bash
# 在文件末尾中增加下面内容
# 每个进程可以打开的文件数的限制
es soft nofile 65536 
es hard nofile 65536
```

`修改/etc/security/limits.d/20-nproc.conf`

```bash
# 在文件末尾中增加下面内容
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
# 操作系统级别对每个用户创建的进程数的限制
* hard nproc 4096
# 注： * 带表 Linux 所有用户名称
```

`修改/etc/sysctl.conf`

```bash
# 在文件中增加下面内容
# 一个进程可以拥有的 VMA(虚拟内存区域)的数量,默认值为 65536
vm.max_map_count=655360

# 最后刷新配置
sysctl -p
```

**集群配置**

首先创建用户

```bash
useradd es #新增 es 用户
passwd es #为 es 用户设置密码
userdel -r es #如果错了，可以删除再加
chown -R es:es /opt/module/es #文件夹所有者
```

修改配置文件 `/opt/module/es/config/elasticsearch.yml` 

```bash
# 加入如下配置
#集群名称
cluster.name: cluster-es
#节点名称， 每个节点的名称不能重复
node.name: node-1
#ip 地址， 每个节点的地址不能重复
network.host: linux1
#是不是有资格主节点
node.master: true
node.data: true
http.port: 9200
# head 插件需要这打开这两个配置
http.cors.allow-origin: "*"
http.cors.enabled: true
http.max_content_length: 200mb
#es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举 master
cluster.initial_master_nodes: ["node-1"]
#es7.x 之后新增的配置，节点发现
discovery.seed_hosts: ["linux1:9300","linux2:9300","linux3:9300"]
gateway.recover_after_nodes: 2
network.tcp.keep_alive: true
network.tcp.no_delay: true
transport.tcp.compress: true
#集群内同时启动的数据任务个数，默认是 2 个
cluster.routing.allocation.cluster_concurrent_rebalance: 16
#添加或删除节点及负载均衡时并发恢复的线程个数，默认 4 个
cluster.routing.allocation.node_concurrent_recoveries: 16
#初始化数据恢复时，并发恢复线程的个数，默认 4 个
cluster.routing.allocation.node_initial_primaries_recoveries: 16
```

修改`/etc/security/limits.conf` 

```bash
# 在文件末尾中增加下面内容
es soft nofile 65536
es hard nofile 65536
```

修改`/etc/security/limits.d/20-nproc.conf`

```bash
# 在文件末尾中增加下面内容
es soft nofile 65536
es hard nofile 65536
\* hard nproc 4096
\# 注： * 带表 Linux 所有用户名称
```

修改`/etc/sysctl.conf`

```bash
# 在文件中增加下面内容
vm.max_map_count=655360

sysctl -p
```





#### ES核心概念

**索引 index**

为了提高查询的性能，索引的数量是任意多的

**文档 Doc**

一条数据就是一个doc，以JSON的形式进行存储

**字段 Field**

对应与关系型数据库的数据列

**映射 Mapping**

类似于MySQL的表结构信息，数据类型、默认值、分析器、是否被索引等等

**分片 Shards**

相当于数据库的分表，允许对分片数据进行并行操作，提高系统的吞吐量

**副本 Replicas**

即备份，针对分片的维度进行备份。要注意主备不能再同一台机器上，一旦机器挂了就BBQ了



#### ES数据读写

**数据写入**

 <img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220518120849759.png" alt="image-20220518120849759" style="zoom:50%;" />

在客户端收到成功响应时，文档变更已经 在主分片和所有副本分片执行完成，变更是安全的。有一些可选的请求参数允许您影响这个过程，可能以数据安全为代价提升性能。这些选项很少使用，因为 Elasticsearch 已经很快，但是为了完整起见， 请参考下文：



**consistency**

即一致性。在默认设置下，即使仅仅是在试图执行一个写操作之前，主分片都会要求必须要有规定数量quorum（或者换种说法，也即必须要有大多数）的分片副本处于活跃可用状态，才会去执行写操作（其中分片副本 可以是主分片或者副本分片）。这是为了避免在发生网络分区故障（network partition）的时候进行写操作，进而导致数据不一致。 规定数量即： int((primary + number_of_replicas) / 2 ) + 1 consistency 参数的值可以设为：

- one ：只要主分片状态 ok 就允许执行写操作。

- all：必须要主分片和所有副本分片的状态没问题才允许执行写操作。

- quorum：默认值为quorum , 即大多数的分片副本状态没问题就允许执行写操作。

  注意，规定数量的计算公式中number_of_replicas指的是在索引设置中的设定副本分片数，而不是指当前处理活动状态的副本分片数。如果你的索引设置中指定了当前索引拥有3个副本分片，那规定数量的计算结果即：int((1 primary + 3 replicas) / 2) + 1 = 3，如果此时你只启动两个节点，那么处于活跃状态的分片副本数量就达不到规定数量，也因此您将无法索引和删除任何文档。



**timeout**

如果没有足够的副本分片会发生什么？Elasticsearch 会等待，希望更多的分片出现。默认情况下，它最多等待 1 分钟。 如果你需要，你可以使用timeout参数使它更早终止：100是100 毫秒，30s是30秒。
新索引默认有1个副本分片，这意味着为满足规定数量应该需要两个活动的分片副本。 但是，这些默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当number_of_replicas 大于1的时候，规定数量才会执行。



**数据读取**

读取从主节点或者其副本进行读取都可以   这块的重点是为了实现负载均衡而对主副做轮询

![image-20220518122122145](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220518122122145.png)



**数据更新及批量操作**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220518132518970.png" alt="image-20220518132518970" style="zoom:33%;" />

1. 请求到达node1
2. 随后被转发到数据所在分片的主份所在的节点上node3（更新是写操作，需要在主机上完成）
3. 获得数据后需要在操作时间内不断进行更新，因为中间可能有其他写请求，需要等待锁
4. 同步副本



#### ES索引

**倒排索引**

分片是ES最小的工作单元，插入数据后进行倒排索引

分词策略   通常借助分词器完成

词条：索引中最小存储和查询单元

词典：字典，词条的集合

倒排表：词条和其所在文档编号的映射关系表



倒排索引写入磁盘后不会再被修改， ”不可变性“，所以对索引的修改开销很大

但这种特性带来优势有：

1. 不需要锁
2. 被读入缓存后不需要进行频繁的内存交换来更新数据
3. 写入单个大倒排索引时允许被压缩，节约IO和内存资源



新的索引作为补充索引合并进主倒排索引	 

按段查询，每一段是一个倒排索引，最早的段会被先查询

删除只是逻辑删除 清除其.del标记

多个倒排合并为一个大索引后才进行物理删除



**写入延时发生示意图**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220518152526135.png" alt="image-20220518152526135" style="zoom:33%;" />



**索引从生成到落盘起效的流程**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220518152411570.png" alt="image-20220518152411570" style="zoom:33%;" />

最后在磁盘上还会发生合并操作



**启用ik分词器**

1. 下载ik分词器解压缩
2. 将ik分词器拷贝到es的plugins目录中去

测试

```json
 GET /_analyze
{
"text":"测试文本"，
"analyzer":"ik_max_word"
}
```

ik_max_word	最细粒度

ik_smart	最粗粒度



**自定义分词词典**

1. 进入es plugins的ik config目录下，新增词典文本文件xxxx.dic

2. 在IKAnalyzer.cfg.xml中将xxxx.dic配置进去 <entry key="ext dict">xxxx.dic </entry>



**在创建索引的时候生成过滤器**

PUT /indice_name

<img src="../assets/images/2022-05-18-elasticsearch.assets/image-20220518155431818.png" alt="image-20220518155431818" style="zoom:33%;" />



**文档冲突**

<img src="../assets/images/2022-05-18-elasticsearch.assets/image-20220518155752044.png" alt="image-20220518155752044" style="zoom:33%;" />



**乐观锁并发控制案例**

在更新的时候传入seq_no和primary_term，只有版本匹配才能更新

`POST /indice_name/id?if_seq_no=1&if_primary_term=1`

 任何老版本的修改请求都将被舍弃，假设现在有一条_version=3的请求操作成功了，此时来了一条_version=2的请求，那么这条请求将被直接舍弃





#### 性能优化

**分片策略**













#### 数据库同步

**通过数据库同步热词**

通过文本编辑来管理词库不方便管理，推荐从数据库中去加载词库

这块涉及到改代码，描述一下大致的流程

1. 在dic包下找到Dictionary类
2. 在该类中新增一个loadFromJDBC方法，该方法中撰写从数据库中查询的逻辑（当然也包括连接，不建议用mbp等插件，会加大源码的大小，采用最原始的conn即可）
3. 在initial方法里面添加一个线程来执行loadFromJDBC方法 



**数据库同步ES doc**

采用**binlog**实现mysql与elasticsearch的同步

存在两种抽象方式：通过消息队列或者直连（都是针对数据同步工具和ES二者而言）

![img](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/87d39a36d0a720843d41ebbd4ae7a583.png)

![img](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/ac57f80fd140be932ba93bc887196ff0.png)



**增量同步**

如何同步？（能同步binlog就行，下面是一些常见的开源/商业化方案）

数据同步工具可以选择**Canal**（only 增量）

选择流式计算框架可以用**Flink**

MQ也可以用**kafka**

各大云厂商的**DTS**



**全量同步**

使用**logstash**

注意ES的版本要和logstash保持一致

```bash
# 复制配置文件，并重命名   ！！！都是在logstash目录下进行操作
cp config/logstash-sample.conf mysql/logstash-mysql.conf
# 编辑配置文件
vim mysql/logstash-mysql.conf
```

```bash
# mysql/logstash-mysql.conf
input {
  # 用于接收控制台输入数据，进行测试
  stdin { }
  jdbc {
      # 数据库  数据库名称为elk，表名为book_table
      jdbc_connection_string => "jdbc:mysql://localhost:3306/elk?characterEncoding=UTF-8"
      # 数据库用户名
      jdbc_user => "root"
      # 数据库密码
      jdbc_password => "root"
      # jar包的位置
      jdbc_driver_library => "mysql/mysql-connector-java-5.1.40.jar"
      # mysql的Driver
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      # 开启分页
      jdbc_paging_enabled => "true"
      # 每页条数
      jdbc_page_size => "100"
      # 需要执行的mysql文件的位置，会执行文件中的sql语句
      #statement_filepath => "mysql/test.sql"
      statement => "select * from book_table where id > :sql_last_value"
      # 调度时间，默认为1分钟执行一次
      schedule => "* * * * *"
      #是否记录上次执行结果, 如果为true,将会把上次执行到的 tracking_column 字段的值记录下来,保存到 last_run_metadata_path 指定的文件中
      record_last_run => true

      #是否需要记录某个column 的值,如果 record_last_run 为true,可以自定义我们需要 track 的 column 名称，此时该参数就要为 true. 否则默认 track 的是 timestamp 的值.
      use_column_value => true
 
      #如果 use_column_value 为true,需配置此参数. track 的数据库 column 名,该 column 必须是递增的.比如：ID.
      tracking_column => id
 
      #指定文件,来记录上次执行到的 tracking_column 字段的值
      #比如上次数据库有 10000 条记录,查询完后该文件中就会有数字 10000 这样的记录,下次执行 SQL 查询可以从 10001 条处开始.
      #我们只需要在 SQL 语句中 WHERE MY_ID > :sql_last_value 即可. 其中 :sql_last_value 取得就是该文件中的值(10000).
      last_run_metadata_path => "config/metadata"
 
      #是否清除 last_run_metadata_path 的记录,如果为true那么每次都相当于从头开始查询所有的数据库记录
      clean_run => false
 
      #是否将 column 名称转小写
      #lowercase_column_names => false
      # 标识输入的类型，自定义字符串，可以用于输出类型选择
      type => "book_table"
    }
}
filter {
 # 默认使用的是Unix时间，我们需要将logstash同步数据到ES时间+8小时
#  数据库中的时间字段也可以这样设置
   ruby {
      code => "event.set('@timestamp',event.get('@timestamp').time.localtime + 8*60*60)"
   }
}

output {
 if [type] == "book_table" {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    user => "elastic"
        password => "changeme"
    #按分钟
    #index => "mysql-%{+YYYY.MM.dd.HH.mm}"
    #按小时
    # 索引名字，必须小写
    index => "mysql-%{+YYYY.MM.dd.HH}"
    #index => "logstash-%{[fields][document_type]}-%{+YYYY.MM.dd}"
        #index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    # 数据唯一索引（建议使用数据库主键 KeyID）
    document_id => "%{id}"
  }
  # 以json格式输出到控制台
  stdout {
        codec => json_lines
  }
 }
}
```

在mysql目录中下载mysql-connector-java-version.jar用于连接mysql数据库

最后`./bin/logstash`启动即可



#### Java语言检索高亮

```java
@Service
public class ContentService {

    @Autowired
    private RestHighLevelClient restHighLevelClient;
    
    //实现搜索功能
    public List<Map<String, Object>> searchHighlightPage(String keyword, int page, int size) throws IOException {
        if (page <= 1) {
            page = 1;
        }
        //创建搜索请求
        SearchRequest searchRequest = new SearchRequest("索引名");
        //构造搜索参数
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
		//设置需要精确查询的字段
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("filed", keyword);
        searchSourceBuilder.query(termQueryBuilder);
        searchSourceBuilder.from((page - 1) * size);
        searchSourceBuilder.size(size);
        searchSourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));
        //高亮
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        //设置高亮字段
        highlightBuilder.field("filed");
        //如果要多个字段高亮,这项要为false
        highlightBuilder.requireFieldMatch(true);
        highlightBuilder.preTags("<span style='color:red'>");
        highlightBuilder.postTags("</span>");
        
		//下面这两项,如果你要高亮如文字内容等有很多字的字段,必须配置,不然会导致高亮不全,文章内容缺失等   
		highlightBuilder.fragmentSize(800000); //最大高亮分片数
   		highlightBuilder.numOfFragments(0); //从第一个分片获取高亮片段
        searchSourceBuilder.highlighter(highlightBuilder);

        searchRequest.source(searchSourceBuilder);
        SearchResponse response = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
        List<Map<String, Object>> list = new ArrayList<>();
        for (SearchHit hit : response.getHits().getHits()) {
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            //解析高亮字段
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            HighlightField field= highlightFields.get("field");
            if(field!= null){
                Text[] fragments = field.fragments();
                String n_field = "";
                for (Text fragment : fragments) {
                    n_field += fragment;
                }
                //高亮标题覆盖原标题
                sourceAsMap.put("field",n_field);
            }
            list.add(hit.getSourceAsMap());
        }
        return list;
    }
}
```





### 源码与原理

---

使用方法见上文，主要明白如何ES中的索引是如何工作的，如何建立索引、查询数据以及自定义分词即可。本节专注在ES的工作机制上面



#### 分片机制



##### 物理分片

```http
PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```

![拥有两个节点的集群](https://www.elastic.co/guide/cn/elasticsearch/guide/current/images/elas_0203.png)

ES的分片机制与Kafka的十分类似，都是一台机器上保持一个index的某些分片（Shards）的数据以及除此之外的其他分片的备份（Replicas）数据

当数据量不断加大时，可以动态调整部署机器的数量，以及每个Shards中的数据大小

```http
PUT /blogs/_settings
{
   "number_of_replicas" : 2
}
```



总体看来，ES的分片具备routing和loadbalancing的功能。用户请求到任意一台机器都可以被该机器转发到正确的地址，并且这个转发的过程符合负载均衡原则。此外，读写都是在Shard上进行，Replica的意义只保留为灾备。



##### 文档更新与持久化



##### 段合并

<img src="https://www.elastic.co/guide/cn/elasticsearch/guide/current/images/elas_1110.png" alt="Two commited segments and one uncommited segment in the process of being merged into a bigger segment" style="zoom: 50%;" />

小段合并成大段的时候会将之前因删除/更新而废弃的老文档彻底删掉，在此之前都是通过维护一个isDel字段来判断状态



#### 评分机制（排序与相关性）



#### 分布式检索



#### 索引管理



##### 分词器管理 Analyzer

一个 *分析器* 就是在一个包里面组合了三种函数的一个包装器， 三种函数按照顺序被执行：字符过滤器、分词器、词单元过滤器

自定义分词器可以通过HTTP请求的方式进行，但是这种方式只适合定义复杂度极小的分词器

```http
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "&=> and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
}}}
```

如果我们需要自定义一个更大的分词器，生产级别的话，需要将分词表等规则以配置文件的方式加载进ES



##### 文档类型Type及其映射

之所以会存在type这个分类机制是由于ES的底层每个片段采用一个Lucene引擎来做，每个Lucene实例中的每个文档都有一个_type字段来对它们进行分类。为了让ES更像是NoSQL，在高版本中已经谈化不提type这个概念了。

https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping.html   



### 面试题

---

##### **Elasticsearch 中的倒排索引是什么？**

倒排索引是搜索引擎的核心。搜索引擎的主要目标是在查找发生搜索条件的文档时提供快速搜索。ES中的倒排索引其实就是 lucene 的倒排索引，区别于传统的正向索引， 倒排索引会再存储数据时将关键词和数据进行关联，保存到倒排表中，然后查询时，将查询内容进行分词后在倒排表中进行查询，最后匹配数据即可。



##### **为什么要使用 Elasticsearch？**

系统中的数据， 随着业务的发展，时间的推移， 将会非常多， 而业务中往往采用模糊查询进行数据的搜索， 而模糊查询会导致查询引擎放弃索引，导致系统查询数据时都是全表扫描，在百万级别的数据库中，查询效率是非常低下的，而我们使用 ES 做一个全文索引，将经常查询的系统功能的某些字段，比如说电商系统的商品表中商品名，描述、价格还有 id 这些字段我们放入 ES 索引库里，可以提高查询速度。

文本搜索能力强（继承了lucece的衣钵）、多维度筛选能力强、社区活跃



**Elasticsearch 的 master 选举流程？**

Elasticsearch的选主是**ZenDiscovery模块**负责的，主要包含**Ping**（节点之间通过这个RPC来发现彼此）和Unicast（单播模块包含-一个主机列表以控制哪些节点需要ping通）这两部分。

对所有可以成为master的节点（node master: true）**根据nodeId字典排序**，每次选举每个节点都把自己所知道节点排一次序，然后选出第一个（第0位）节点，暂且认为它是master节点。

如果对某个节点的**投票数**达到一定的值（可以成为master节点数n/2+1）并且该节点自己也选举自己，那这个节点就是master。否则重新选举一直到满足上述条件。

master节点的职责主要包括集群、节点和索引的管理，不负责文档级别的管理；data节点可以关闭http功能。




**Elasticsearch 集群脑裂问题？**

“脑裂”问题可能的成因：

- 网络问题：集群间的网络延迟导致一些节点访问不到master, 认为master 挂掉了从而选举出新的master,并对master上的分片和副本标红，分配新的主分片。
- 节点负载：主节点的角色既为master又为data,访问量较大时可能会导致ES停止响应造成大面积延迟，此时其他节点得不到主节点的响应认为主节点挂掉了，会重新选取主节点。
- 内存回收：data 节点上的ES进程占用的内存较大，引发JVM的大规模内存回收，造成ES进程失去响应。



脑裂问题解决方案：

- 减少误判：discovery.zen ping_ timeout 节点状态的响应时间，默认为3s，可以适当调大，如果master在该响应时间的范围内没有做出响应应答，判断该节点已经挂掉了。调大参数（如6s，discovery.zen.ping_timeout:6），可适当减少误判。

- 选举触发：discovery.zen.minimum. _master_ nodes:1，该参數是用于控制选举行为发生的最小集群主节点数量。当备选主节点的个數大于等于该参数的值，且备选主节点中有该参数个节点认为主节点挂了，进行选举。官方建议为(n / 2) +1, n为主节点个数（即有资格成为主节点的节点个数）。

- 角色分离：即master节点与data节点分离，限制角色

  - 主节点配置为：node master: true，node data: false

   - 从节点配置为：node master: false，node data: true
     
     

##### **Elasticsearch 索引文档（indexing）的流程？**

索引建立后是如何进入内存然后刷盘的

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/1bdc6c30d1be9b1bff83a683c64d2ac7.png" alt="img" style="zoom: 67%;" />

协调节点默认使用文档 ID 参与计算（也支持通过 routing），以便为**路由**提供合适的分片：shard = hash(document_id) % (num_of_primary_shards)

当分片所在的节点接收到来自协调节点的请求后，会将请求写入到 Memory Buffer，然后定时（默认是每隔 1 秒）写入到 **Filesystem Cache**，这个从 Memory Buffer 到 Filesystem Cache 的过程就叫做 refresh；

当然在某些情况下，存在 Momery Buffer 和 Filesystem Cache 的数据可能会丢失， ES 是通过 translog的机制来保证数据的可靠性的。其实现机制是接收到请求后，同时也会写入到 **translog** 中，当 Filesystemcache 中的数据写入到磁盘中时，才会清除掉，这个过程叫做 **flush**；

在 flush 过程中，内存中的缓冲将被清除，内容被写入一个新段，段的 fsync 将创建一个新的提交点，并将内容刷新到磁盘，旧的 translog 将被删除并开始一个新的 translog。

flush 触发的时机是定时触发（默认 30 分钟）或者 translog 变得太大（默认为 512M）时；



##### **Elasticsearch 更新和删除文档的流程？**

删除和更新也都是写操作，但是 Elasticsearch 中的文档是不可变的，因此不能被删除或者改动以展示其变更；
磁盘上的每个段都有一个相应的.del 文件。当删除请求发送后，文档并没有真的被删除，而是在.del文件中被标记为删除。该文档依然能匹配查询，但是会在结果中被过滤掉。当段合并时，在.del 文件中被标记为删除的文档将不会被写入新段。
在新的文档被创建时， Elasticsearch 会为该文档指定一个版本号，当执行更新时，旧版本的文档在.del文件中被标记为删除，新版本的文档被索引到一个新段。旧版本的文档依然能匹配查询，但是会在结果中被过滤掉。



##### **Elasticsearch 搜索的流程？**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/053a14eee04ace7b4e5aec0ce53a5284.png" alt="img" style="zoom:67%;" />

搜索被执行成一个两阶段过程，我们称之为 Query Then Fetch；

在初始查询阶段时，查询会广播到索引中每一个分片拷贝（主分片或者副本分片）。 每个分片在本地执行搜索并构建一个匹配文档的大小为 from + size 的优先队列。 PS：在搜索的时候是会查询Filesystem Cache 的，但是有部分数据还在 Memory Buffer，所以搜索是近实时的。

每个分片返回各自优先队列中所有文档的 ID 和排序值给协调节点，它合并这些值到自己的优先队列中来产生一个全局排序后的结果列表。

接下来就是取回阶段， 协调节点辨别出哪些文档需要被取回并向相关的分片提交多个 GET 请求。每个分片加载并丰富文档，如果有需要的话，接着返回文档给协调节点。一旦所有的文档都被取回了，协调节点返回结果给客户端。

Query Then Fetch 的搜索类型在文档相关性打分的时候参考的是本分片的数据，这样在文档数量较少的时候可能不够准确， DFS Query Then Fetch 增加了一个预查询的处理，询问 Term 和 Document frequency，这个评分更准确，但是性能会变差。



##### **在并发情况下， Elasticsearch 如果保证读写一致？**

可以通过**版本号**使用乐观并发控制，以确保新版本不会被旧版本覆盖，由应用层来处理具体的冲突；

另外对于写操作，一致性级别支持 quorum/one/all，默认为 quorum，即只有当大多数分片可用时才允许写操作。但即使大多数可用，也可能存在因为网络等原因导致写入副本失败，这样该副本被认为故障，分片将会在一个不同的节点上重建。

对于读操作，可以设置 replication 为 sync(默认)，这使得操作在主分片和副本分片都完成后才会返回；如果设置 replication 为 async 时，也可以通过设置搜索请求参数_preference 为 primary 来查询主分片，确保文档是最新版本。
