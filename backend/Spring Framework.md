---
layout: post
title: "Spring基础原理问题答题思路"
permalink: /spring-interview-basic
---
这篇文章分享了关于Spring面试中常见原理问题的答题思路，由于这种框架原理在初级中级开发人员招聘中不是重点，所以这篇原理文章捋明白了足够应付校招！

在整理思路框架的时候牢记**总分结构**

不管是要说什么话，记住对应场景下的套路表达框架总能事半功倍！！！

**总**：先对问题做一个简要的概括

**分**：分点来描述细节，不清楚的点可以选择性略过

1

2

3...
会说的点能稍微多谈些

突出技术名词，核心概念（**核心概念、接口、类、关键方法**）

## 1、对IOC的理解
说到IOC就要想到两个关键的概念：控制反转IOC 和 依赖注入DI

Inverse Of Control：理论思想，原来的对象是由使用者来进行控制，有了Spring之后，可以把整个对象交给Spring来帮我们进行管理

DI：@Autowired populateBean，把对应属性注入到具体的Bean中去

天天说的Spring容器是什么？ -->  可以理解成一个存储对象的数据结构，而且存的对象限定为Bean对象，它使用map KV 键值对形式来存储。在spring中一般存在三级缓存，三级的划分标准就是Bean对象的不同生命周期阶段，例如singletonObjeacts级别（一级缓存）的空间用来存放完整的Bean对象即已经初始化完成且populate了数据的Bean对象。整个Bean的生命周期，从Bean的创建到使用到销毁的过程全部都是由这个Spring容器来进行管理的。

到底啥是Bean对象呢？  -->  Bean对象其实就是普通的Java对象，只不过遵守了一些特定的规则，比如必须有构造方法、getter和setter方法。
说到底，Spring容器就是一个专门管理Java对象的一套程序，比如在使用@Autowired的时候，我们不需要手动引入对象，Spring会帮我们自动引入在Spring容器中已有的该类对象。如果没有的话，则需要通过@Bean、@Service、@Component等把一个类对象交给Spring容器来进行管理。



## **2、IOC的底层实现**

IOC的底层实现可以理解为Spring容器是如何来管理Bean对象的，因为IOC主要就是解决了Java对象的管理问题。所以这个问题可以以Bean对象的生命周期来答，重点突出一些关键点：如反射、工厂、设计模式、重要的方法等。

带do的是实际执行逻辑的方法，不带do的都是在包在外层的facet

1. 先通过**createBeanFactory** 创建一个Bean工厂(DefaultListableBeanFactory)

2. 开始循环创建对象，因为容器中的bean默认都是单例的，所以优先通过getBean，doGetBean从容器中查找，找不到的话

3. 通过createBean，**doCreateBean**方法，以**反射**的方式创建对象，一般情况下使用的是无参的构造器(getDeclaredConstructor(),newinstance)。在创建对象的过程中需要BeanDefinition对象，其中记录了对象的各种信息

   <img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220510165705276.png" alt="image-20220510165705276" style="zoom: 33%;" />

3. 进行对象的属性填充populateBean

5. 进行其他的初始化操作(initializingBean)



## **3、描述Bean的生命周期**
<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220411151731744.png" alt="image-20220411151731744" style="zoom: 25%;" />

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220510170243988.png" alt="image-20220510170243988" style="zoom:33%;" />

![img](https://img-blog.csdnimg.cn/20210529213228111.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4ODI2MDE5,size_16,color_FFFFFF,t_70)

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/BF6FDF73-D96B-4C6C-87AE-C56560AFD26B.JPG" style="zoom: 25%;" />

在表述的时候不要只说图片中有的关键点，要学会扩展描述

1. 实例化Bean对象，采用反射的方式生成对象。这里重要的是生成**BeanDefinition对象**，从里面获取到创建的Bean所具有的属性

2. 填充bean的属性：**populateBean()**， 循环依赖的问题(三级缓存)

3. 调用aware接口相关的方法：invokeAwareMethod(完成BeanName可以获取容器bean的名称，BeanFactory获取当前bean factory这也可以调用容器的服务，BeanClassLoader对象的属性设置)

4. 调用**BeanPostProcessor中的Before**前置处理方法：使用比较多的有(ApplicationContextPostProcessor设置ApplicationContext，Environment,ResourceLoader,EmbeddValueResolver等对象)

5. 调用**init-method**方法：invokeInitmethod()，判断是否实现了InitializingBean接口，如果有，调用afterPropertiesSet方法

6. 调用**BeanPostProcessor的After**后置处理方法：spring的aop就是在此处实现的，AbstractAutoProxyCreator注册Destuction相关的回调接口。Spring主要提供了两类扩展点BeanPostProcessor和BeanFactoryPostProcessor。前者是操作bean的实例，后者使对bean的元数据定义进行扩展。**AOP功能就是IOC扩展点的一个经典应用。**

7. 获取到完整的对象，可以通过getBean的方式来进行对象的获取

8. 销毁流程 ①判断是否实现了DispoableBean接口②调用**destroyMethod**方法



## **4、对AOP底层实现的理解**

总得来说，AOP就是一种功能增强机制，官方名字叫面向切面编程，功能是在不改变原有代码的逻辑基础上为原方法新增一些业务逻辑。AOP是通过**动态代理**的方式来进行增强的，AOP实际上就是基于IOC的特性做的一些扩展。

什么是动态代理？ 通俗点说就是：无需声明式的创建java代理类，而是在运行过程中生成"虚拟"的代理类，被ClassLoader加载。从而避免了静态代理那样需要声明大量的代理类。如果我们为Spring的某个bean配置了切面，那么Spring在创建这个bean的时候，**实际上创建的是这个bean的一个代理对象，我们后续对bean中方法的调用，实际上调用的是代理类重写的代理方法**。也就是在有AOP逻辑的方法上，Spring在生成对应的Bean对象的时候有两套逻辑，即原逻辑+切面逻辑。

- 它有**三种增强方式，前后和环绕，**分别对应在方法执行流程不同环节上进行增强。其中两个概念，切面和切点，分表代表着某个逻辑要加在哪个方法上面。
- 它通过动态代理来实现增强，有jdk和cglib两种方式。cglib的执行逻辑是
  - 在执行方法调用的时候，会调用到生成的字节码文件中，直接回找到DynamicAdvisoredInterceptor类中的intercept方法，从此方法开始执行
  - 根据之前定义好的通知来生成拦截器
  - 从拦截器链中依次获取每一个通知开始进行执行，在执行过程中，为了方便找到下一个通知是哪个，会有一个CglibMethodInvocation的对象，找的时候是从-1的位置依次开始查找并且执行的
- 它和IoC的关系，AOP是基于IoC的特性实现的。IoC在设计的时候为高可扩展性做了铺垫，在Bean生命周期的**BeanPostProcessor方法**中能够为Bean对象增加增强功能。
- 应用场景，比如缓存和日志，这类高频但是会涉及很多重复代码的场景就适合用AOP来做



#### AOP的各种通知模式的执行顺序

![image-20230226181053650](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230226181053650.png)



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230320113419177.png" alt="image-20230320113419177" style="zoom: 33%;" />

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230320115727078.png" alt="image-20230320115727078" style="zoom: 33%;" />

权限修饰符、类名、方法名可以使用*符号来进行全包含



#### example

```java
@Aspect //将一个java类定义为切面类
@Component  //将该类将入Spring bean factory
@Slf4j
public class WebLogAspect {
    //解决线程访问共享变量的安全问题
    ThreadLocal<Long> startTime = new ThreadLocal<Long>();

    @Pointcut("execution(* com.zerah.init..*.*(..))")
    public void webLog(){}

    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable{
        startTime.set(System.currentTimeMillis());

        //接受请求，记录请求内容
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        //切面逻辑：记录请求内容
        // 记录下请求内容
        log.info("URL : " + request.getRequestURL().toString());
        log.info("HTTP_METHOD : " + request.getMethod());
        log.info("IP : " + request.getRemoteAddr());
        log.info("CLASS_METHOD : " + joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
        log.info("ARGS : " + Arrays.toString(joinPoint.getArgs()));
        log.info(": "+joinPoint.getSignature().getName()+joinPoint.getSignature().getModifiers());
    }
    @AfterReturning(returning = "ret",pointcut = "webLog()")
    public void doAfterReturning(Object ret) throws Throwable{
        // 处理完请求，返回内容
        log.info("Response: "+ ret);
        log.info("Spend time :"+ (System.currentTimeMillis() -startTime.get()));
    }

    @Around("webLog()")
    public Object doArround(ProceedingJoinPoint joinPoint) throws Throwable{
        log.info("Arround 之前执行");
        String proceed = (String)joinPoint.proceed();
        proceed += "增强返回值";
        log.info("Arround 之后执行"+proceed);
        return proceed;
    }
}
```







## **5、循环依赖的发生与解决**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220411232515633.png" alt="image-20220411232515633" style="zoom: 25%;" />

循环依赖就是由于两个Bean对象还没有生成的时候或者在生命周期还**没有完成初始化**的过程中，二者彼此都有了依赖，导致初始化赋值无法继续下去，因为依赖的对方还没有初始化完成，也即在内存空间中还没有分配地址（也就没有引用）。

解决的话就是三级缓存（三个map）用来存放处于不同阶段的Bean（成品与半成品的区别，有没有填充属性的区别）。**需要引入其他Bean的时候先去缓存而非全局Bean容器中找，找到之后引用其代理，**后续再对这个半成品的对象进行属性的填充。

一级缓存：完整Bean对象

二级缓存：还没有填充属性的半成品

三级缓存：三级缓存的value类型是ObjectFactory，是一个函数式接口，存在的意义是保证在整个容器的运行过程中同名的bean对象只能有一个

普通对象和代理对象是不能同时出现在容器中的，因此当一个对象需要被代理的时候，就要使用代理对象覆盖掉之前的普通对象，在实际的调用过程中，是没有办法确定什么时候对象被使用，所以就要求某个对象被调用的时候，优先判断此对象是否需要被代理，类似于一种回调机制的实现，因此传入lambda表达式的时候，可以通过lambda表达式来执行对象的覆盖过程，getEarlyBeanReference()

因此，所有的bean对象在创建的时候要优先放到三级缓存中，在后续的使用过程中，如果需要被代理则返回代理对象，如果不需要被代理，则直接返回普通对象

各级缓存放置的时间：

三级缓存：createBeanInstance之后：addSingletonFactory

二级缓存：第一次从三级缓存确定对象是代理对象还是不同对象的时候，同时删除三级缓存getSingleton

一级缓存：生成完整对象之后放到一级缓存，删除二三级缓存：addSingleton



## **6、BeanFactory和FactoryBean的区别**

相同点：都是用来创建Bean对象的

不同点：使用BeanFactory创建对象的时候，必须要遵循严格的生命周期流程，太复杂了，如果想要简单的自定义某个对象的创建，同时创建完成的对象想交给spring来管理，那么就需要实现FactoryBean接口的方法

总结就是，如果需要按照标准化来批量生产，则用BeanFactory，见名知意就是一个生产bean的工厂；如果需要个性化定义则用FactoryBean

## **7、Spring中用到的设计模式**

参考《图解设计模式》过一下Spring中的几个重点设计模式



#### 单例模式：bean默认都是单例的

- 单例模式的意义：

1. 节约内存资源，避免重复浪费
2. 对于某些应用需求，只允许有一个类的实例生成

- Spring中实现单例：

定义了一个Bean的HashMap，每次要生成实例的时候先进入到这个HashMap中检查该类的实例对象是否存在，如果存在的话从中取出即可不走新建的流程





原型模式：指定作用域为prototype

#### 工厂模式：BeanFactory



模板模式：postProcessBeanFactory，onRefresh，initPropertyValue

策略模式：XmlBeanDefinitionReader，PropertiesBeanDefinitionReader

观察者模式：listener，event，multicast

<u>适配器模式</u>：Adapter - SecurityConfigurerAdapter

装饰者模式：BeanWrapper

责任链模式：使用MVC的时候Interceptor的拦截链条，preHandle、postHandle、onCompletion

<u>代理模式</u>：动态代理

委托者模式：delegate



## **8、Spring事务是如何实现回滚的**

优质文章https://xie.infoq.cn/article/52f38883e28821c9cf0a608ea

Spring事务的实现是通过AOP来给目标方法的执行前后增加一套TransactionalInterceptor，在事务开始的时候Spring会同数据库进行通信set autocommit = 0 也就是关闭掉自动提交

在事务需要回滚的时候进行rollback操作

隔离级别默认的是和数据库一致，如果不一致会将本次操作的数据库事务设置为Spring对应的级别

```java
// Apply specific isolation level, if any.
Integer previousIsolationLevel = null;
if (definition != null && definition.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT) {
	if (logger.isDebugEnabled()) {
		logger.debug("Changing isolation level of JDBC Connection [" + con + "] to " +
				definition.getIsolationLevel());
	}
	int currentIsolation = con.getTransactionIsolation();
	if (currentIsolation != definition.getIsolationLevel()) {
		previousIsolationLevel = currentIsolation;
		con.setTransactionIsolation(definition.getIsolationLevel());
	}
}
```

特别注意使用Spring事务的类不能是final，其中的方法不能是private，这样无法被代理



## **9、Spring事务传播**



#### 传播特性有几种？七种

**Required，Required_new，nested, Support，Not_Support，Never，Mandatory**

**默认是Required**



#### 某一个事务嵌套另外一个事务的时候如何选择合适的传播方式？

这个问题实际上只需要考虑内层事务是什么传播类型即可，因为外层事务失效了内层事务必挂（不一定）也就没有什么讨论的必要性；而内层事务挂了该如何影响外层事务就需要我们进行深入的讨论了

**Required：**如果当前没有事务，则自己新建一个事务，如果当前存在事务，则加入这个事务

**Supports：**当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行

**Mandatory：**当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常

**Requires_new：**在执行时，不论当前是否存在事务，总是会新建一个事务。也就是外层出事了但是不影响到内层

**Not_Supported：**始终不以事务的方式来运行

**Never：**不使用事务，如果存在事务则抛出异常

**Nested：**如果当前事务存在，则在嵌套事务中执行，否则REQUIRED的操作一样（开启一个事务）。但是和Required相比，这类传播可以允许调用方catch被嵌套事务的异常





## 10、SpringBoot自动装配原理

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220413142340845.png" alt="image-20220413142340845" style="zoom:50%;" />

简单讲就是SpringBootApplication注解当中核心组成部分**@EnableAutoConfiguration**里面导入了两个东西，一是**@Import(AutoConfigurationPackagesRegistrar.class)**该包导入了导包规则；

其次是**@Import(AutoConfigurationImportSelector.class)**，这个包则是指定从从META-INF/spring.factories中按照Registrar的规则来讲依赖包进行导入。



现在看来啊就是一个SPI机制，核心就在于**AutoConfigurationImportSelector.class**这个类中的**selectImports**方法，然后开启一系列的方法调用来完成装配：

1. 首先从全部的配置文件中load类信息，以及注解上的ConditionXXX等信息，生成一个需要真正load进来的Class列表

2. 然后调用ClassLoader把这些jar包文件load到JVM里面来

3. 最后就是基于这些class文件来创建对象



```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
  if (!isEnabled(annotationMetadata)) {
    return NO_IMPORTS;
  }
  AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
  return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
  if (!isEnabled(annotationMetadata)) {
    return EMPTY_ENTRY;
  }
  ...
    
  List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);

  ...
}
```

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
  // 这里就是关键，使用SpringFactoriesLoader加载所有配置类，是不是像我们SPI的ServicesLoader
  List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
                                                                       getBeanClassLoader());
  Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
                  + "are using a custom packaging, make sure that file is correct.");
  return configurations;
}
```

```java
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
  return EnableAutoConfiguration.class;
}
```

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
  String factoryTypeName = factoryType.getName();
  return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
}
```

```java
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {

  try {
    // FACTORIES_RESOURCE_LOCATION的值为：META-INF/spring.factories
    // 这步就是意味中读取classpath下的META-INF/spring.factories文件
    Enumeration<URL> urls = (classLoader != null ?
                             classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
                             ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
    ...
}
```









## 11、Spring小组件

### Spring Cache

缓存小工具，通过注解的方式来代替在代码中编写缓存的各种逻辑，起到一个简化开发的作用

相关注解：

**@Cacheable** 

将方法的返回结果放入缓存，两个参数：一个是分区name，另一个是key。key的话可以写死一个字符串，也可以通过reg表达式进行简单的构建



**@CacheEvict**

缓存删除，通常放在update或者delete的操作方法当中



**@CachePut**

更新缓存，放在update操作的方法当中



**@Caching**

聚合方法，通过@Caching({@Cacheable().... @CachePut....})等来为一个方法添加多个词、缓存逻辑



**Spring Cache在读模式下能够做到安全，在多读少写的场景下可以放心使用**

解决缓存穿透：开启cache-null-values=true

解决缓存击穿 ：@Cacheable有个sync参数，开启之后的get方法是走synchronized

缓存雪崩：设置参数spring.cache.redis.time-to-live



**然而，Spring Cache在写多的场景下做不到安全，仍然需要通过手写的方式去实现**

加分布式锁

canal异步同步

或者直接不走缓存





### Spring Scheduling

























## SpringBoot小题

#### @Resource、@Value和@Autowired的区别

**@Autowired** 相对来说比较死板，只能byType进行匹配，对于一个Class多个instance的情况就有点儿棘手

**@Resource** 由Java来提供，然后支持byName，很灵活

@Resource注解中有一个name属性，针对name属性是否有值，@Resource依赖注入的底层流程是不同的
如果name属性有值，那么Spring会直接根据所指定的name值去Spring容器中找Bean对象，如果找到了则成功否者就会抛出异常
如果name属性没有值
1、先判断该属性名字在Spring容器中是否存在Bean对象
2、如果存在，则成功找到Bean对象进行注入
3、如果不存在，则根据属性类型去Spring容器中找Bean对象，找到一个则进行注入，没有则抛异常

**@Value**是用来从Properties文件中来获取值的



#### 如何在 Spring Boot 启动的时候运行一些特定的代码？

可以实现接口 ApplicationRunner 或者 CommandLineRunner，这两个接口实现方式一样，它们都只提供了一个 run 方法



#### bootstrap.properties如何理解

bootstrap (. yml 或者 . properties)：boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 Spring Cloud 配置就会使用这个文件。**且 boostrap 里面的属性不能被覆盖**。



#### SpringBoot如何配置多数据源

方法一：

在配置文件中配置两个数据源，从名字上区别开

在配置类中创建DataSource的Bean，需要指定配置文件的prefix来确定走的是哪个数据源

配置类的Bean需要@Primary注解上默认数据源

如何使用ORM框架，mapper需要放在两个不同的路径下，然后ORM的配置类中@MapperScan扫对应的包



方法二：

使用Druid连接池，ORM层还是一样的

```java
@Value("${spring.datasource.datasource2.jdbc-url}")
private String url;
@Value("${spring.datasource.datasource2.driver-class-name}")
private String driverClassName;
@Value("${spring.datasource.datasource2.username}")
private String username;
@Value("${spring.datasource.datasource2.password}")
private String password;
@Value("${spring.datasource.datasource2.initialSize}")
private int initialSize;
@Value("${spring.datasource.datasource2.minIdle}")
private int minIdle;
@Value("${spring.datasource.datasource2.maxActive}")
private int maxActive;

@Bean(name = "dataSource2")
public DataSource dataSource() {
    DruidDataSource dataSource = new DruidDataSource();
    dataSource.setUrl(url);
    dataSource.setDriverClassName(driverClassName);
    dataSource.setUsername(username);
    dataSource.setPassword(password);
    dataSource.setInitialSize(initialSize);
    dataSource.setMinIdle(minIdle);
    dataSource.setMaxActive(maxActive);
    return dataSource;
}
```



方法三：动态数据源

通过继承DynamicDataSource类然后重写相关的方法的实现动态数据源







#### 如何使用 Spring Boot 实现全局异常处理？



#### Spring事务在哪些场景下会失效，又如何来解决呢？

异常未捕获的情况下事务失效

A：在catch内加上异常处理逻辑即可

private或者protected关键字的方法上

A：这个没有办法

AOP里面的逻辑在pjp.proceed( )执行之前报错了

A：在AOP里面加上try catch，或者直接throw掉



#### 如何提高SpringBoot应用的性能？

- 如果项目比较大，类比较多，不使用@SpringBootApplication，采用@Compoment指定扫包范围

- 在项目启动时设置JVM初始内存和最大内存相同
- 将Tomcat服务器换成undertow



### spring boot 核心配置文件是什么？bootstrap.properties 和 application.properties 有何区别 ?

- 单纯做 Spring Boot 开发，可能不太容易遇到 bootstrap.properties 配置文件，但是在结合 Spring Cloud 时，这个配置就会经常遇到了，特别是在需要加载一些远程配置文件的时侯。
- spring boot 核心的两个配置文件：
  1. bootstrap (. yml 或者 . properties)：boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 Spring Cloud 配置就会使用这个文件。且 boostrap 里面的属性不能被覆盖；
  2. application (. yml 或者 . properties)： 由ApplicatonContext 加载，用于 spring boot 项目的自动化配置。
  3. 通过@ImportResource注解也可以实现倒入xml配置文件



#### SpringBoot配置文件的加载顺序

1. properties文件；

2. YAML文件；

3. 系统环境变量；

4. 命令行参数



#### Async异步调用的如何使用

1. 在启动类上@EnableAsync注解
2. 在需要异步调用的方法定义上注解@Async，在该方法被实际调用的时候则不会影响原来的主线程



## SpringMVC小题

#### 什么是SpringMVC，它的意义是什么？

它基于Spring的MVC开发框架，属于请求驱动型的web框架。在功能上将数据模型、视图和控制器三部分内容进行了解耦和，便于开发者之间的合作

在前后端不分离的时代，这种MVC框架有着很积极的作用。但是现在主流开发是前后端分离的，后端开发者只需要将数据以JSON等形式传给前端即可，不需要经过时图渲染这个过程，只保留了对M和C两层的使用



#### SpringMVC的工作原理

![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/14/171744675337b0ec~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 常用的注解

@RestController = @Controller + @ResponseBody

@RequestBody

@ResponseBody

@RequestMapping

@RequestParam与@PathVariable
