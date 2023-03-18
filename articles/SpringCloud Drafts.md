

拍个视频 英语面试练习CAP和BASE理论

看看把@RequestParam 在RestTemplate里面的运用总结一下

Dashborad的开启总结一下

idea改maven的jdk依赖，原创



Unable to make field private com.sun.tools.javac.processing.JavacProcessingEnvironment$DiscoveredProcessors com.sun.tools.javac.processing.JavacProcessingEnvironment.discoveredProcs accessible: module jdk.compiler does not "opens com.sun.tools.javac.processing" to unnamed module @6917bb4



大坑，做微服务的时候每次一点项目层级的改动，必须盯紧：**pom包及其版本、jdk版本的丝毫变动**



今天做的修改：CommonResult的两个参数构造方法删掉了， Slf4j的log业务语句删掉了





##### eureka集群搭建

![image-20220312123049161](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220312123049161.png)

集群之间是互相注册，不断发送心跳保持连接

建好7002工程之后改hosts文件 添加端口映射 sudo vi /etc/hosts 给127.0.0.1 添加多个url映射 意义是做区分

yml中改defaultZone以实现相互注册,多台机器逗号分隔（* 在实际的生产中不需要改hosts，根据实际机器的ip地址配上去就行了）

集群的使用，通过消费者80端口访问，

1、在80端口的服务端代码中将具体的端口改成http://service-name，

2、然后在AppContextConfig中restTemplate配置方法的开启@LoadBalance注解

@EnableDiscoveryClient 开启服务发现 注入 DiscoveryClient

eureka自我保护：当集群出现分区（网络）的时候， eureka不会删除该分区当中的服务信息，满足了CAP理论中的AP理论，出于保证集群高可用特性。相信任何故障都是暂时性的，也许一段时间后就恢复了

关闭自我保护：

服务端-业务：

```xml
lease-renewal-interval-in-seconds: 1  # 向server发送心跳的间隔
lease-expiration-duration-in-seconds: 2 # 多久没收和server通信就不再给server发心跳
```

服务端-eureka-server：

```xml
enable-self-preservation: false
eviction-interval-timer-in-ms: 2000  # 多久没收到心跳就把其驱逐掉
```



##### eureka总结谢幕



##### Zookeeper

使用Zookeeper作为注册中心方案之后，业务启动类加上@EnableDiscoveryClient注解

docker run --restart always 当docker重启的时候该容器也自动重启



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220312222959435.png" alt="image-20220312222959435" style="zoom: 33%;" />

<img src="/Users/handsomnehu/Library/Application Support/typora-user-images/image-20220312223401248.png" alt="image-20220312223401248" style="zoom:50%;" />

zk整合进Spring存在本级zk版本和Spring默认版本之间的冲突问题

在exclude了内嵌zookeeper之后，要自己引入本级版本对应的zk的maven依赖

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
</dependency>
```



https://blog.csdn.net/weixin_44595122/article/details/110097413 dokcer上查看zk注册服务

zk的服务节点是临时节点，eureka可以选择上是临时或非临时，但是这个能根据心跳、过期等时间来设置。总的来说，zk在这方面放弃节点比eu来得更加干脆

##### ZK集群搭建



##### Consul

用go语言开发的，zk由于用得相对少（作为微服务的注册中心）。因此，在SC的生态里consul可以同nacos一决高下

```xml
# pom
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
</dependency>

# yml
cloud:
	consul:
		host:
		port:
		discovery:
			service-name:

# main
@EnableDiscoveryClient
```



### 

##### zk docker

zk的一些基本操作，主要操作都在bin目录当中

docker进入bin的方式 **docker exec -it [name] bash  ->  cd bin**

zkCleanup　　清理Zookeeper历史数据，包括食物日志文件和快照数据文件

zkCli　　　　  Zookeeper的一个简易客户端

zkEnv　　　　设置Zookeeper的环境变量

zkServer　　  Zookeeper服务器的启动、停止、和重启脚本  **zkServer.sh start/status/stop/restart**

查看注册服务 **./zkCli.sh       ->      ls /services**

若连接不同的主机，可使用如下命令：**./zkCli.sh -server ip:port**

**ls /[path]** 查看当前目录下的节点列表

**get /[path]** 

**get /[path] [watch]**

**set /[path] key=value**

**delete /[path]**



docker的一些碎碎念之启动参数

docker run **-d** **--name** [name] **-p** local_p:virtual_p **-v** local:virtual **-e** k=v **--env-file** [env_addr] **--net** [net_type]  [image_name]



Eureka  zk  consul三者作为服务注册中心的区别

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220313163132231.png" alt="image-20220313163132231" style="zoom:50%;" />

Ribbon是进程内的负载均衡，本地负载均衡，逻辑集成进系统，属于**<u>消费方选地址</u>**

Nginx是进程外的负载均衡，第一道大门，独立设施，整个系统集中分配



**RestTemplate** 

forObject返回结果是JSON 

forEntity是更为精细，可以自定义请求头和请求体



#### Ribbon

**Ribbon自带的负载均衡算法集合**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220313195930630.png" alt="image-20220313195930630" style="zoom: 50%;" />



Ribbon 自定义的负载均衡设置方式：

1. 在@ComponentScan扫不到的包下创建Config类，把自定义的负载均衡类型（Ribbon自带的七种）bean到Spring容器里面去
2. 在启动类上加@DiscoveryClient ( name, class )

**Ribbon负载均衡算法实现大要：**

维护一个服务器节点list，每次rest接口请求序号a去跟服务器节点总数n取余，然后list把下标list[a / n]的节点推出来处理本次请求

在源码当中有这么个关键方法

```java
private int incrementAndGetModulo(int modulo){
  for(;;){
    int current = nextServerCyclicCounter.get();
    int next = (current + 1) % modulo;
    // if not true then step into next loop, in which the value of current and next will be updated
    if(nextServerCyclicCounter.compareAndSet(current, next)){
      return next;
    }
  }
}
```

```xml
// 客户端Ribbon设置自定义的超时时间，默认是1s
// 当前推荐使用OpenFeign的feign.client.config.... 来配置超时时间 并且可以制定为某个服务设置超时 
ribbon:
	ReadTimeout: 5000
	ConnectTimeout: 5000
```

**Ribbon日志级别：**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220314115634227.png" alt="image-20220314115634227" style="zoom:50%;" />











#### <u>服务调用OpenFeign方案</u>

##### OpenFeign基本使用

OpenFeign的使用理念就是微服务接口调用 + @FeignClient + 启动类上面@EnableFeignClients

Feign只需要在调用者80端口上面配即可

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220313223415071.png" alt="image-20220313223415071" style="zoom: 33%;" />



##### OpenFeign的面试常问

OpenFeign相较于Feign，支持了SpringMVC的接口，让进行远程调用像使用本地RestAPI接口一样便捷。

服务超时问题，这块需要注意Ribbon和OpenFeign都可能超时，在解决这个问题的时候要考虑好两边超时机制的配合。

传参问题，使用POJO对象来传参时额外加上一个@SpringQueryMap注解。









Hystrix 初步进行服务降级方案是把其他低资源的请求的负载分给高资源的请求，用Jmeter进行压测的结果表现为之需要0.0001s处理的请求开始变得响应延迟

##### Jmeter基本高并发请求

1. 先定义线程数量

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220314200038996.png" alt="image-20220314200038996" style="zoom:50%;" />

2. 编辑请求访问地址，全部完成之后点绿色开始按钮即可

   <img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220314200314192.png" alt="image-20220314200314192" style="zoom: 33%;" />

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220314200335831.png" alt="image-20220314200335831" style="zoom:33%;" />



#### <u>负载均衡SpringCloud LoadBalancer方案</u>

先上基本的使用

```java
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
  
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class RestaurantService{
   public static void main(String[] args) {
      SpringApplication.run(RestaurantService.class, args);
   }
}


@FeignClient(name = "customer-service")
@LoadBalancerClient(name = "customer-service",
configuration=LoadBalancerConfiguration.class)
public interface CustomerService {
   @RequestMapping("/customer/{id}")
   public Customer getCustomerById(@PathVariable("id") Long id);
}

// as for the self defined conf, there is a lot more need to be explored
@Configuration
public class LoadBalancerConfiguration {
   @Bean
   public ServiceInstanceListSupplier
discoveryClientServiceInstanceListSupplier(
         ConfigurableApplicationContext context) {
      System.out.println("Configuring Load balancer to prefer same instance");
      return ServiceInstanceListSupplier.builder()
               .withBlockingDiscoveryClient()
               .withSameInstancePreference()
               .build(context);
      }
}
```



#### <u>服务降级Hystrix</u>

降级是思想，在降级之下存在诸如兜底、熔断、链路恢复等具体策略 

导致降级的情况：运行时异常、超时、服务熔断触发服务降级、线程池/信号池打满

降级的意思就是原本用来处理这个请求的业务逻辑是给你一辆特斯拉model x，但是发现满足不了你，然后就退而求其次给你一辆五菱宏光mini（兜底方法 handler）

##### Hystrix单点兜底

在8001Service业务逻辑上（业务提供侧）加@HystrixCommand注解

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220314212206119.png" alt="image-20220314212206119" style="zoom:50%;" />

在 *<u>80用户端和8001服务端</u>* 的启动类上加@EnableCircuitBreaker注解

##### Hystrix全局服务降级 global fallback

1

 在80用户端的Controller 类上添加@DefaultProperties( DefaultFallback = "handler_name" )，需要细致的参数将*commandProperties*配置进该注解

在该类里添加对应方法的兜底方法实现，这种弊端的one for all，一个兜底方法给全部接口使用

2

用户端80上的服务引用接口实 现一个统一的异常处理类

如果没有注释 @HystrixCommand且是指定处理方法的那种会到@HystrixCommand指定fallback处理，全局的不会

写在controller类内部的处理，并加上默认或者指定降级处理。如果存在一个继承service的降级处理，他处理的是客户端自身的异常

他们俩都能处理服务端异常，但是如果都存在，则service优先处理服务端出现的异常。

写在接口里只能处理服务器端的异常，客户端没用写在controller类内部的处理，并加上默认或者指定降级处理。如果没有继承service的降级处理，他处理的是服务端与客户端异常

### 

##### Hystrix 熔断

在一定时间内达到服务调用失败的上限即触发熔断，就是直接返回人为的500错误，在一段时间服务恢复之后又恢复该链路的访问（和物理建筑中的“熔断”概念最大的区别就在这里，物理上熔断之后需要人为干预去恢复）

自动恢复的过程是渐渐放开一部分访问，当访问量大到一个阈值且都能正常访问之后遍全局开启链路恢复

```java
@HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback", commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
            // in 10 seconds if 60% of all requests failed then we enable circuit breaker
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60"),
    })
// how to find all the properties in IDEA
// 1. open "find" window in IDEA
// 2. serach "HystrixCommandProperties" class in "Classes" sub tag
```

Hystrix限流放在Sentinel里面学习

Hystrix Flow Chart 

![img](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/hystrix-command-flow-chart.png)



#### 网关Gateway

Zuul1.0是阻塞式

Zuul2.0设计理念采用非阻塞的Netty框架，但由于商业运营问题迟迟未上线，Spring在Zuul2.0的理念上提出了基于WebFlux和Netty的Gateway网关

**Gateway三大概念：**

Route 路由匹配规则

Predicate 断言/匹配条件

Filter 过滤/拦截器

上述三大配置项在官方文档中有https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories



#### 配置中心Spring Cloud Config

解决配置文件冗余，集中的**外部配置**支持 通过Git进行配置文件管理  

github绑定1980- 的qq邮箱 用户名是handsomehu  之后就专门用这个GitHub账号了

application.yml是用户级配置

bootstrap.yml是系统级配置



Config 更新后客户端自动更新配置

```xml
# client的controller类上
@RefreshScope

# bootstrap.yml
# 配置自动更新
management:
  endpoints:
    web:
      exposure:
        include: "*"

# 激活3355 client
curl -X POST "http://localhost:3355/actuator/refresh"
```

Config这种半手动客户端配置更新存在着如下一些问题，将在Bus中得到解决：

每个微服务都需要手动POST刷新配置

不能做到一次广播，处处生效

不能做到大范围自动更新

##### SpringCloud Bus

在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息。

支持两种MQ rabbit 和 kafka

核心理念就是通过MQ来自动发送刷新消息

总线就是一种消息整体通知，发送给订阅了相关主题的所有节点





pining41youngladY!



##### rabbitmq再复习

Ubuntu用apt可以直接install erlang 和 rabbitmq

```shell
# systemctl 是操作系统层面的命令
systemctl start rabbitmq-server
systemctl stop rabbitmq-server
systemctl restart rabbitmq-server
systemctl status rabbitmq-server.service

rabbitmqctl add_user admin admin
rabbitmqctl set_user_tags admin administrator

rabbitmq-plugins enable rabbitmq_management  # web port 15672

# set_permissions [-p <vhostpath>] <user> <conf> <write> <read> -p / 表示赋予全路径的权限
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```









Nextcloud Config

```shell
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-nextcloud-on-ubuntu-20-04

'trusted_domains' => array(
        0 => '127.0.0.1',
        1 => preg_match('/cli/i',php_sapi_name())?'127.0.0.1':$_SERVER['SERVER_NAME'],
),

https://github.com/yukaiji/buildVpn

```

Bus进行总控通知的两种方案

1.

2.

为什么第二种更靠谱

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220316161445421.png" alt="image-20220316161445421" style="zoom: 33%;" />



ubuntu tips:

dpkg -L +软件包的名字，可以知道这个软件包包含了哪些文件

系统安装软件一般在/usr/share，可执行的文件在/usr/bin，配置文件可能安装到了/etc下等

查看所有服务端口 netstat -ap

**cmd + c** abort current bash task

handshu 19990627a

snapd

snapcraft

```sql
# 解决mysql密码无法设置问题
use mysql;
update user set plugin="mysql_native_password";
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
flush privileges;
quit;

# 设置远程访问
use mysql;
update user set host='%' where user='root';
```

Bus定点通知

curl -X POST "http://host:port/actuator/bus-refresh/config-client:3355"

### 

##### Stream

 <img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220316221215771.png" alt="image-20220316221215771" style="zoom: 33%;" />

  

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220317145651972.png" alt="image-20220317145651972" style="zoom: 33%;" />

Channel是对队列Queue的一种抽象，IM中的消息的存储和发送都是通过Channel来完成的

##### 重复消费问题

Stream在默认情况下所有标了Input的消费者都能消费到该消息，存在重复消费问题（实际生产中一个业务部署多台机器，但是一条消息只能被同服务下的一个实例消费）。解决方案是分组，同组内的消费者实现消息唯一消费

在服务的配置文件当中注明该服务的分组

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220317195840133.png" alt="image-20220317195840133" style="zoom: 33%;" />

这块本质是还是MQ基本原理的运用，又得复习MQ了 : )

##### 持久化

在设置了分组之后同时也开启了持久化功能，就是当服务关机期间有来自MQ的消息到达之后，在服务开机之后也能正常获取到

##### sleuth & zipkin

sleuth内部集成了zipkin，后者主要负责监控调用的线路和响应时间，而前者负责what？sleuth算是zipkin pro，Spring收编了zipkin之后微调后推出了Sleuth

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/648afd5a54f548b897c4c84b3060a254~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp" alt="image.png" style="zoom: 50%;" />

```yml
# 下jar包
java -jar zipkin-server-2.12.9-exec.jar

# 消费 和 服务端都引入依赖
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>

# 消费 和 服务端yml
spring:
  application:
    name: cloud-stream-provider
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
    #采样率值介于 0 到 1 之间，1 则表示全部采集
    probability: 1

```

ubuntu 20.04更新jdk

1. sudo apt install openjdk-8-jdk

2. sudo vim ~/.bashrc

3. export JAVA_HOME=/usr/lib/jvm/openjdk-8-jdk

   export JRE_HOME=${JAVA_HOME}/jre  

   export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  

   export PATH=${JAVA_HOME}/bin:$PATH

4. cd 到 /usr/lib/jvm 之后source ~./bashrc

java-8-openjdk-amd64



##### nacos-discovery

nacos的配置和入门案例参见官网就行了

在分布式理论领域里面，nacos有两套理论可以根据业务的需要进行切换，CP和AP理论

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220319134800640.png" alt="image-20220319134800640" style="zoom: 25%;" />

##### nacos-config

配置文件application（保证配置的个性）和bootstrap（保证配置的共性）都要有

application加载的优先级要高于boostrap









<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220319155659999.png" alt="image-20220319155659999" style="zoom:33%;" />

Namespace默认是public

Namespace主要用来隔离，开发、测试、生产这种不同类型的环境就可以用namespace来进行隔离：

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220319163131984.png" alt="image-20220319163131984" style="zoom:33%;" />

在nacos管理页面配置完之后，再去项目的bootstrap包里面指定namespace

Group默认是DEFAULT_GROUP 相当于是包名：

在bootstrap里面注明group名，在application里注明active类型

Srvice类比于服务，一个服务包含多个集群，主要用在做异地多活上。以杭州（HZ）、深圳（SZ）等名字来命名service之间可以实现互相调用

Cluster默认是DEFAULT

DataId 每个配置文件的名字，做每个具体配置文件的隔离



##### nacos集群

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220319165741941.png" alt="image-20220319165741941" style="zoom:33%;" />

**nacos高可用 = nacos集群 + nginx主备 + mysql集群**

###### nacos持久化切换为mysql

Tencent  <u>mysql_master</u> 121.5.143.186

Aliyun  <u>nginx_master</u>

Azure

```
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://121.5.143.186:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```



###### nginx搭建主备高可用集群

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220319204428245.png" alt="image-20220319204428245" style="zoom: 25%;" />

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/zh-cn_image_0285681028.png" alt="img" style="zoom:50%;" />

目前来讲，主备两台机要实现集群都得在同一个子网里面

安装完nginx记得开启disable 禁止开机自启

`sudo apt install keepalived`

`https://www.cnblogs.com/binghe001/p/13599263.html`

`https://support.huaweicloud.com/bestpractice-vpc/bestpractice_0010.html`

###### nacos集群搭建

![deployDnsVipMode.jpg](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/deployDnsVipMode.jpg)

1. 在/nacos/conf 里面修改.properties文件，开启mysql存储方式

2. cluster.conf中将集群中的全部的ip:port都填进去

3. 在nginx的conf文件中增加upstream{ }，将nacos运行的三台机器ip都放进去

   <img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220320193524208.png" alt="image-20220320193524208" style="zoom: 50%;" />

   

##### Sentinel

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220320202311904.png" alt="image-20220320202311904" style="zoom: 25%;" />

sentinel组件 = 后台控制 + 前台操作(8858)

@SentinelResource注解关于配置错误处理方法的四种方法

- 无兜底，直接返回whitelabel错误提示
- fallback兜底
- blockHandler兜底
- fallback + blockHandler兜底

在兜底方法的设置中，可以直接在Controller中@SentinelResource写兜底方法。但更加项目化的方式是新建兜类，在类中写方法，然后在@SentinelResource里面写明*<u>类</u>*和*<u>方法</u>*

##### Seata

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/145942191-7a2d469f-94c8-4cd2-8c7e-46ad75683636.png" alt="image" style="zoom:25%;" />



**WHAT**

Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata = 1个全局唯一分布式id + 3套组件（TC、TM、RM）。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。

**TC (Transaction Coordinator) - 事务协调者**

维护全局和分支事务的状态，驱动全局事务提交或回滚。

**TM (Transaction Manager) - 事务管理器**

定义全局事务的范围：开始全局事务、提交或回滚全局事务。

**RM (Resource Manager) - 资源管理器**

管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

全局提交完成后删除undo_log和image，这时业务无法自动回退了



mysql8链接地址后面加

```properties
?useUnicode=true&characterEncoding=UTF-8&serverTimeZone=UTC
?useUnicode=true&characterEncoding=utf-8&useSSL=false
```



maven的parent标签意义是依赖继承，DependencyManagement的意义是版本继承



**Seata事务原理**

注册分支，获取全局锁

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220322170925602.png" alt="image-20220322170925602" style="zoom:25%;" />

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220322165137812.png" alt="image-20220322165137812" style="zoom: 33%;" />

全局写入

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220322165102014.png" alt="image-20220322165102014" style="zoom: 25%;" />



尝试拿本地锁 ——> 执行业务逻辑 ——> 尝试拿全局锁 ——> 写入本地库



在数据库本地事务隔离级别 **读已提交（Read Committed）** 或以上的基础上，Seata（AT 模式）的默认全局隔离级别是 **读未提交（Read Uncommitted）** 。

如果应用在特定场景下，必需要求全局的 **读已提交** ，目前 Seata 的方式是通过 SELECT FOR UPDATE 语句的代理。







Nginx 搭建图片服务器

121.5.143.186/root/nginx/images



