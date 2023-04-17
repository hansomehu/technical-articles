---
layout: post
title: "机器学习大数据推荐平台"
permalink: /ml-recom-platform
---



#### 项目整体技术架构

![image-20220709154155347](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220709154155347.png)



#### 数据流转图

![image-20220709154231837](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220709154231837.png)



【pre】通过SparkSQL将系统初始化数据（业务数据相关的txt文档）加载到MongoDB中

**离线部分**

【1.0】通过调度框架（如Azkaban），**定时**将产生的离线数据通过HQL导入HDFS中进行机器学习的计算

【1.1】离线统计服务从MongoDB中加载数据，将“商品平均评分统计”、“商品评分个数统计”、“最近商品评分个数统计”三个统计算法进行运行实现，并将计算结果回写到 MongoDB 中；离线推荐服务从 MongoDB 中加载数据，通过 ALS 算法分别将**“用户离线推荐结果矩阵”、“商品相似度矩阵”**回写到 MongoDB 中

**实时部分**

【2.0】Flume 从综合业务服务的运行日志中读取日志更新，并将更新的日志（商品评分操作的日志）实时推送到Kafka 中

【2.1】Kafka 在收到这些日志之后，通过 kafkaStream 程序对获取的日志信息 进行过滤处理，获取用户评分数据流【UID|MID|SCORE|TIMESTAMP】，并发送到 另外一个 Kafka 队列

【2.3】Spark Streaming 监听 Kafka 队列，实时获取 Kafka 过滤出 来的用户评分数据流，融合存储在 Redis 中的用户最近评分队列数据，提交给实时 推荐算法，完成对用户新的**基于行为的推荐结果**计算

【2.4】计算完成之后，将新的推荐结构和 MongDB 数据库中的推荐**结果进行合并**

**系统业务部分**

【3】推荐结果展示部分，从MongoDB中将离线推荐结果、实时推荐结果、内容推荐结果进行混合，综合给出相		  对应的数据

【3.1】商品信息查询服务通过对接MongoDB实现对商品信息的查询操作，一个简单的CRUD

【3.2】商品评分部分，获取用户通过UI给出的评分动作，后台服务进行数据库记录后，一方面将数据推动到Redis 			 中，另一方面，通过预设的日志框架输出到 Tomcat 中的日志中（随后由Flume提取）

【3.3】商品标签部分，项目提供用户对商品打标签服务。
