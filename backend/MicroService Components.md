

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230509183502153.png" alt="image-20230509183502153" style="zoom:50%;" />



## RPC





### OpenFeign

**Feign**是Netflix公司写的，是SpringCloud组件中的一个轻量级RESTful的HTTP服务客户端，是SpringCloud中的第一代负载均衡客户端。 **OpenFeign**是SpringCloud自己研发的，在**Feign**的基础上支持了Spring MVC的注解，如@RequesMapping等等，让基于Java的远程调用更加便捷。

https://www.51cto.com/article/699142.html



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230209210738013.png" alt="image-20230209210738013" style="zoom: 50%;" />



#### OpenFeign的基本使用

```java
// 在启动类上注解 @EnableFeignClients  @EnableDiscoveryClient

// 在接口上开启@FeignClient
@FeignClient("stores")
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    List<Store> getStores();

    @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
    Store update(@PathVariable("storeId") Long storeId, Store store);
}

// 增加Hystrix的fallback支持
@FeignClient(name = "hello", fallback = HystrixClientFallback.class)
protected interface HystrixClient {
    @RequestMapping(method = RequestMethod.GET, value = "/hello")
    Hello iFailSometimes();
}

// 自定义配置，但是更推荐在配置中心使用配置
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    //..
}

// 增加对POJO参数的支持
@FeignClient("demo")
public class DemoTemplate {

    @GetMapping(path = "/demo")
    String demoEndpoint(@SpringQueryMap Params params);
}
```





#### 远程调用的发起流程（动态代理的过程）？

根据接口的type、name、url等信息

DefaultTargeter/HystrixTargeter调用target方法

在target中调用Feign类的builder方法

在builder中依赖于ReflectiveFeign去走JDK代理类生成的方式，然后注册进Spring容器

```java
// Class FeignClientFactoryBean

// target当中执行生成代理的核心逻辑
Targeter targeter = get(context, Targeter.class); 
return (T) targeter.target(this, builder, context, 
      new HardCodedTarget<>(this.type, this.name, url)); 

// 这部分代码实际创建了代理对象
public class ReflectiveFeign extends Feign { 
  // 为 feign client 接口中的每个接口方法创建一个 methodHandler 
 public <T> T newInstance(Target<T> target) { 
    for(...) { 
      methodToHandler.put(method, handler); 
    } 
    // 基于 JDK 动态代理的机制，创建了一个目标接口的动态代理，所有对接口的调用都会被拦截，然后转交给 handler 的方法。 
    InvocationHandler handler = factory.create(target, methodToHandler); 
    T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(), // JDK代理
          new Class<?>[] {target.type()}, handler); 
} 
```

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230209213044279.png" alt="image-20230209213044279" style="zoom: 33%;" />



#### MVC的解析过程

通过MethodMetaData类为每个对应的接口生成一个MethodHandler

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230209215124276.png" alt="image-20230209215124276" style="zoom:40%;" />



#### 请求的发送过程

在代理对象中会执行根据RequestTemplate来生成Request的过程，最后由OpenFeign发送这个Request请求

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230209215332557.png" alt="image-20230209215332557" style="zoom:40%;" />

Feign客户端这边会根据用户的请求自己生成一个Request，然后用这个Request去目的服务进行请求。这个过程是可以自定义拦截器来进行拦截的，在目的Request中加入我们需要的数据或者做出相应的更改。



### gRPC

















## Flow Control



### Hystrix

整体执行流程如下所示，大体上是先查询缓存，如果没有就走执行逻辑，如果健康检查失败则走fallback，如果正常逻辑执行失败也走fallback

```java
// 服务熔断代码示例
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),//是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),//请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),//时间范围
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60")//失败率%达到多少后跳闸
    })
    public String paymentCircuitBreaker(Integer id) {
        if (id < 0) {
            throw new RuntimeException("id 不能负数");
        }
        return Thread.currentThread().getName() + "\t" + "调用成功,流水号: " + IdUtil.fastSimpleUUID();
    }

// 也有使用Ribbon的RestTemplate进行服务间调用的，但是这种做法的解耦程度较差


// 在Netflix内部也会这么来以Service的方式定义一个Command，这种方式不符合主流的开发
public class GetStockServiceCommand extends HystrixCommand<String> { 
    private StockService stockService; 
    public GetStockServiceCommand() { 
        super(setter()); 
    } 
    private static Setter setter() { 
        //服务分组 
        HystrixCommandGroupKey groupKey = HystrixCommandGroupKey.Factory. asKey("stock"); 
        //服务标识 
        HystrixCommandKey commandKey =HystrixCommandKey.Factory. asKey("getStock"); 
        //线程池名称 
        HystrixThreadPoolKey threadPoolKey = HystrixThreadPoolKey.Factory. asKey("stock-pool"); 
        //线程池配置 
        HystrixThreadPoolProperties.Setter threadPoolProperties =HystrixThreadPoolProperties.Setter threadPoolProperties =HystrixThreadPoolProperties.Setter() 
               .withCoreSize(10) 
               .withKeepAliveTimeMinutes(5) 
               .withMaxQueueSize(Integer.MAX_VALUE) 
               .withQueueSizeRejectionThreshold(10000); 
  
        //命令属性配置 
        HystrixCommandProperties.Setter commandProperties = HystrixCommandProperties.Setter() 
               .withExecutionIsolationStrategy(HystrixCommandProperties.ExecutionIsolationStrategy.THREAD); 
        return HystrixCommand.Setter 
                        .withGroupKey(groupKey) 
                       .andCommandKey(commandKey) 
                       .andThreadPoolKey(threadPoolKey) 
                       .andThreadPoolPropertiesDefaults(threadPoolProperties) 
                        .andCommandPropertiesDefaults(commandProperties); 
    } 
   @Override 
    protectedString run() throws Exception { 
        return stockService.getStock(); 
    } 
} 
```



#### Overall

![image-20230211141809314](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230211141809314.png)





#### Circuit Breaker

通过两个条件判断是否open breaker，在一个滑动窗口期间用一个类似于队列的结构来统计一共有多少request以及error rate是多少

如果breaker处于开启状态，等待一个 sleepTimeout 的时间重新恢复访问

只有断路的功能，没有引入rate limiter 的各种限流算法

![image-20230211173958336](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230211173958336.png)

将一秒划分成10个窗口，每个窗口100ms，然后不断统计该窗口时期内的流量、成功率，如果超过阈值的话这个周期内的其他请求直接拒绝掉













#### TIPS

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230316104023178.png" alt="image-20230316104023178" style="zoom:33%;" />

解决方案





### Resilience4J

这个方案和Sentinel一样功能多，包括了限流算法和各种热启动，但是原生没做图形化的监控功能。



## Register Center



## Gateway



### Spring Cloud Gateway

