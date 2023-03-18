---
layout: post
title: "Java Concepts"
permalink: /java-concepts
---





#### 面向对象OOP的特点

**封装**

将大量的数据和基于数据的操作封装在一个实体内部，使其构成一个不可分割的独立整体。数据及其操作的细节被隐藏了，只保留一些对外接口使之与外部发生联系。

优点包括：减少了耦合；方便开发者的维护，能够更好的对模块（对象）进行性能把握；提高软件的可重用性；降低了构建大型系统应用的风险

**继承**

继承应该遵循里氏替换原则，子类对象必须能够替换掉所有父类对象。

**多态**

多态分为编译时多态和运行时多态：编译时多态主要指方法的重载；运行时多态指程序中定义的对象引用所指向的具体类型在运行期间才确定

多态的特性使得OOP的灵活度和开发成本得到进一步的降低，通过没有把对象写死的方式来实现



#### 常见类图必须会

**继承 extends**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220512121616965.png" alt="image-20220512121616965" style="zoom:33%;" />

**实现 implements**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220512121655675.png" alt="image-20220512121655675" style="zoom:33%;" />

**聚合关系 理解为A类的内部有BCD的对象**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220512121850560.png" alt="image-20220512121850560" style="zoom:33%;" />

**组合关系 A不在了 BCD也就没了 理解为Java的内部类**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220512121958462.png" alt="image-20220512121958462" style="zoom:33%;" />

**关联关系 属于是虚拟关系**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220512122100360.png" alt="image-20220512122100360" style="zoom:33%;" />

**依赖关系 A是B的局部变量或者方法内的参数**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220512122146783.png" alt="image-20220512122146783" style="zoom:33%;" />



### Java语法

**装箱拆箱**

就是说Java基础类型大多都有对应的对象类型，二者通过装箱、拆箱机制互相转化

```java
Integer x = 2;     // 装箱
int y = x;         // 拆箱
```

Java基本类型中还有一个**缓存池**的概念，ValueOf( )方法每次都是调用缓存池中的数据而非新建对象

String是**不可变对象**，是线程安全的，且适合用来作为HashMap的Key值

强转的原则是**不抛弃**，所以只能父类强转为子类。因为子类中会包含父类中不存在的属性和方法，子类转父类的话回导致子类需要丢弃一些属性和方法。

**a==b** 是比较两个对象的引用，只有当 a 和 b 指向的是堆中的同一个对象才会返回 true，而 **a.equals(b)** 是进行逻辑比较，所以通常需要重写该方法来提供逻辑一致性的比较。equals( )是通过比较**hashCode**的值来实现的。





 **final、finalize 和 finally 的不同之处?**

final记住其编译器常量这个特性

finalize( )是垃圾回收

finally不管有没有异常都要处理



**super和this关键字**

this 只能放在方法中使用，<u>和super一样必须放在方法逻辑的第一行</u>

用于操作调用this所在方法的属性变量

用于调用构造方法（根据参数来区别）

用于调用当前对象的普通方法

用于返回当前对象 return this



```java
public class A{
  private String name = "xiaoming";
  public void setName(String name){
    // 由于形参和实参名都一样，不用this的话出于就近原则编译器并不会重置name的值
    this.name = name;
  }
}

public static void main(String[] args){
  A a = new A();
  thisTest.setName("xiaoma");
}

// use this to call the constructor
public class Student {
    private int age;
    private String name;
    public Student() {
    // when instantiate Student class without params, it'll turn out to be using the following params
        this("小马",50);
    }
  public Student(String name, int age) {
        this.name = name;
        this.age = age;
        System.out.println(name + "今年" + age + "岁了");
    }
}
```

super

调用父类的构造方法

调用父类的成员属性

调用父类的成员方法

super和this不能同时出现在一个方法中，因为二者都会初始化其父类



**局部变量为什么要初始化**

局部变量运行时被分配在栈中，量大，生命周期短，如果虚拟机给每个局部变量都初始化一下，是一笔很大的开销，但变量不初始化为默认值就使用是不安全的。出于速度和安全性两个方面的综合考虑，解决方案就是虚拟机不初始化，但要求编写者一定要在使用前给变量赋值。



#### **修饰符**

**private** : 在同一类内可见。使用对象：变量、方法。 **注意：不能修饰类（外部类）**

**protected** : 对同一包内的类和所有子类可见。使用对象：变量、方法。 **注意：不能修饰类（外部类）**

**static**：

- 该关键字用来声明独立于对象的**静态方法**，静态方法不能使用类的非静态变量，静态方法从参数列表得到数据，然后计算这些数据
- 同样也可以用来修饰**静态变量**，无论一个类实例化多少对象，它的静态变量只有一份拷贝。 静态变量也被称为类变量。局部变量不能被声明为 static 变量



#### **Java的深拷贝与浅拷贝**

**浅拷贝只是拷贝了源对象的地址，**所以源对象的值发生变化时，拷贝对象的值也会发生变化。而深拷贝则是拷贝了源对象的所有值，相当于属性值完全相同的新对象

执行深拷贝的方法：

1. 构造函数
2. Jackson序列化

```java
User copyUser = objectMapper.readValue(objectMapper.writeValueAsString(user), User.class);
```





#### 异常相关

throws和throw

throws是声明异常，用在方法名的结尾。若方法中存在检查异常，如果不对其捕获，那必须在方法头中显式声明该异常，以便于告知方法调用者此方法有异常，需要进行处理

throw则是抛出异常，通常配合try catch来使用

```java
public static void method() throws IOException, FileNotFoundException{
    //something statements
}

public static double method(int value) {
    if(value == 0) {
        throw new ArithmeticException("参数不能为0"); //抛出一个运行时异常
    }
    return 5.0 / value;
}
```





#### SPI

SPI（Service Provider Interface），是JDK内置的一种 服务提供发现机制，可以用来启用框架扩展和替换组件，主要是被框架的开发人员使用，比如**java.sql.Driver接口**，其他不同厂商可以针对同一接口做出不同的实现。就是我们导入上述jdbc接口之后，如果我们本地用的是mysql的jar包，那么就可以选择使用MySQL的数据库服务，如果是其他的则亦然。

还有就是日志框架的选择也是走的SPI机制，slf4j是一个门面，我们在实际使用的过程当中会去查找它具体的实现。

Spring的自动装配也是通过SPI来做的

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220512173244144.png" alt="image-20220512173244144" style="zoom:33%;" />

它的实现原理在于，通过**ServiceLoader的load( )**方法，这个方法传入Class参数，然后前往META-INF中查找

然后在load方法当中会生成一个**iterator**

如果**providers**缓存里面存在实现对象，那么就iterate这些对象。如果不存在的话，那么就**hasNextSerive( )**方法取META-INF里面找实现了该接口的服务









#### 注解

注解的意义就是一种写给JVM的代码注释，为类或者方法添加额外的信息

Spring AOP的核心实现可以概括为**注解+反射**



##### 元注解

- @Retention - 标识这个注解怎么保存，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问。通常设置为RUNTIME级别
- @Documented - 标记这些注解是否包含在用户文档中
- @Target - 标记这个注解应该是哪种 Java 成员。类、方法、变量、接口、参数等等
- @Inherited - 标记这个注解是继承于哪个注解类(默认 注解并没有继承于任何子类)



##### example

```java
// 先定义一个注解，注解里头包括了我们需要的额外的类信息
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface UsingDataSource {

    String value() default ""; // 注解中只能定义基本类型（基础）字段，而没有对象类
}


@Aspect
@Component
public class DataSourceAspect {

    @Pointcut("@annotation(com.example.demo.datasource.UsingDataSource)")
    public void checkPointCut() {

    }

    @Before("checkPointCut()")
    public void checkBefore(JoinPoint joinPoint, UsingDataSource annotation) { // 注解作为其一个参数，可以根据注解内的值来对被注对象做定制化的操作

        try {
            Class<?> clazz = joinPoint.getTarget().getClass();
            String methodName = joinPoint.getSignature().getName();
            Class<?>[] parameterTypes = ((MethodSignature) joinPoint.getSignature()).getMethod().getParameterTypes();
            Method method = clazz.getMethod(methodName, parameterTypes);
            UsingDataSource usingDataSource = method.getAnnotation(UsingDataSource.class);
            String dataSourceKey = usingDataSource.value();
            DataSourceContextHolder.setKey(dataSourceKey);

        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }

    }

    @After("checkPointCut()")
    public void checkAfter(){
        DataSourceContextHolder.clearKey();
    }
}
```







#### 反射

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

在两个地方经常用到，一个是**动态代理**的时候；另外一个是**多态**的时候（声明的时候是用的父类或者接口，但是具体实例化这是通过反射创建具体的对象）



![image-20220512181041236](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220512181041236.png)

在Java中，**Class类**与**java.lang.reflect类**库一起对反射技术进行了全力的支持。在反射包中，我们常用的类主要有**Constructor类**表示的是Class 对象所表示的类的构造方法，利用它可以在运行时动态创建对象、**Field类**表示Class对象所表示的类的成员变量，通过它可以在运行时动态修改成员变量的属性值（包含private）、Method类表示Class对象所表示的类的成员方法，通过它可以动态调用对象的方法（包含private）

**通过JVM的堆空间中的对象信息获取到类相关的信息**





反射基于Java动态特性，具体的类型可以在运行之后再通过ClassName、ClassPath去确定

任何动态造对象底层都是通过**构造器**的方式去造的对象



**反射的两个方式，**可以在运行时获取到类的属性、方法、构造器、接口、父类方法等等

 <img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230219152406635.png" alt="image-20230219152406635" style="zoom:50%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230219152614435.png" alt="image-20230219152614435" style="zoom:50%;" />



**四种获取Class信息的方式**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230219160839166.png" alt="image-20230219160839166" style="zoom: 33%;" />







#### 动态代理

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230208164443436.png" alt="image-20230208164443436" style="zoom:33%;" />

https://juejin.cn/post/6844903744954433544



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230219194352403.png" alt="image-20230219194352403" style="zoom: 33%;" />



**静态代理**

指的是显示地指定一个类来作为代理类，局限性很大。并且代理类在编译期就确定下来了，很不灵活

动态代理就是编译期创建一个通用的类，然后在具体使用的时候再具体化

```java
interface ClothProduce(){
  void produce();
}

class ProxyClothProduce{
  void ProxyClothProduce(ClothProduce example){}
}

class NikeProducer implements ClothProduce{
  
}

new ProxyClothProduce(new NikeProducer()); 
```



**JDK**

基于接口实现的动态代理，具有一定的局限性

JDK 的动态代理是靠多态和反射来实现的，它生成的代理类需要实现你传入的接口，**并通过反射来得到接口的方法对象，并将此方法对象传参给增强类的 invoke 方法去执行**，从而实现了代理功能，故接口是 jdk 动态代理的核心实现方式，没有它就无法通过反射找到方法，所以这也是必须有接口的原因

JDK动态代理也是依靠字节码来实现的，先从JVM中获取到字节码，再转化成对象

```java
public class DynamicProxy {
    
    class ProxyFactory{

        public Object getProxyInstance(Object obj){
            MyProxyHandler handler = new MyProxyHandler(obj);

            return Proxy.newProxyInstance(obj.getClass().getClassLoader(),
                    obj.getClass().getInterfaces(),
                    handler);
        }
    }

    class MyProxyHandler implements InvocationHandler {

        private Object obj;

        public MyProxyHandler(Object obj) {
            this.obj = obj;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            return null;
        }
    }
}
```



**CGLib**

通过字节码框架ASM来获取到类信息，它是通过生成子类来实现的，代理对象既可以赋值给实现类，又可以赋值给接口，度比jdk动态代理更快，性能更好



#### **ThreadLocal**

tl是一种无同步方案类型的线程安全机制，核心是在每个线程中保存一个**本地内存**，这个内存只能由该线程来操作，通过这种方式避免了线程不安全事件的发生。ThreadLocal会为每个线程创建单独的变量副本, 避免因多线程操作共享变量而导致的数据不一致的情况。

 

Thread类型的对象中有一个ThreadLocalMap类型的变量threadlocals，在new ThreadLocal时候对象名就是key，然后该线程的业务对象是value（实际的存储都在该线程的本地内存中）。在整个JVM层面有这么一个map来存储全部的ThreadLocals。

TLM对象：

- 没有实现Map接口
- 它没有public的方法，最多有一个default的构造方法，因为这个ThreadLocalMap的方法仅仅在ThreadLocal类中调用，属于静态内部类
- WeakReference
- 仅仅用了一个Entry数组来存储Key，Value

 

内存泄露问题：

- 不被销毁的线程，其ThreadLocal内存里面的数据会一直在那里导致了内存的无端浪费
- 在线程业务执行的最后调用remove( )方法清除线程中的对象

 

运用场景：

- 多并发情况下保证线程安全
- 将状态(例如用户ID、事务ID)与线程关联起来，可以手动为每个线程设置一个serialNum来对应一个类



#### 内存模型

内存模型主要包括了下面一些部分，内存模型的抽象结构、happens-before原则、volatile可见性规则

**抽象结构**

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/E44aHibktsKbkcTibzPYSrl5EiaBvFO80Z4BribQ5ry83ibA709DoWv0Hibgj4VX0XSTiaeYjy6Z22CWA36Yegiayicicr9w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 50%;" />

Java线程的一次读写操作涉及8种操作，这些操作共同定义了线程读写数据在本地内存和主内存之间交互的流程

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/E44aHibktsKbkcTibzPYSrl5EiaBvFO80Z437Z9zoWaOlddfByaj0N8NBrVkhicVASCib71f9RzTPCXIECKDWA4pf5A/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 50%;" />

**happens-before原则**

前一个操作的结果对后一个操作应该是可见的。Java中为了保证这些重排序采用了8条规则，比如传递性、volatile变量规则、程序顺序规则、监视器锁的规则等等。我们写的代码只要在这些规则下，前一个操作的结果对后续操作是可见的，是不会发生重排序的。

**volatile关键字**

这个关键字用来解决有序性和可见性问题，用到了我们上面说的两个技术点，但是**不能实现原子性（部分操作）**

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/E44aHibktsKbkcTibzPYSrl5EiaBvFO80Z4N0mu8oBWBOBlCVoLOf9DRxZg57tWiahXmicSmSZnKIfq7VmRrHpEpMOQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:33%;" />

Java内存模型为了实现volatile有序性和可见性，定义了4种内存屏障的「规范」，分别是LoadLoad/LoadStore/StoreLoad/StoreStore。说白了就是在读写的前后加入了保护机制。最终的效果其实就是：对一个 volatile 变量的写操作相对于后续对这个 volatile 变量的读操作可见





#### String**类**

二者都继承了AbstractStringBuilder抽象类，间接实现了CharSequence接口

和String相比，Buffer和Builder都是可变对象，不需要生成大量的中间对象，因此效率高、节约空间

最底层的东西是final char[]



**StringBuilder**

线程不安全

最底层的东西是char[]



**StringBuffer**

线程安全，但是开销大，在多线程操作的时候使用

最底层的东西是char[]

同时增删改的方法都是Synchronized修饰的



大多数通过**synchronized**关键字来实现线程安全

```java
public synchronized StringBuffer append(String str) {
  toStringCache = null;
  super.append(str);
  return this;
}
```



在new一个buffer或者builder的时候新建了一个大小为16的char[]



扩容的时候是扩到原数组的2倍+2，然后再执行一个copyOf操作 





#### Java的基本类型

- 整数类型：byte，short，int，long.
- 浮点数类型：float，double.
- 字符类型：char.
- 布尔类型：boolean.



#### Java 泛型

**java 中泛型标记符：**

- **E** - Element (在集合中使用，因为集合中存放的是元素)
- **T** - Type（Java 类）
- **K** - Key（键）
- **V** - Value（值）
- **N** - Number（数值类型）
- **？** - 表示不确定的 java 类型



##### **集合泛型**

```java
// 泛型方法 printArray                         
   public static < E > void printArray( E[] inputArray )
   {
      // 输出数组元素            
         for ( E element : inputArray ){        
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }

	public List<E> transferData(E[] arr){
    
    return list;
  }
```



##### **类型泛型**

```java
public static <T extends Comparable<T>> T maximum(T x, T y, T z)
   {                     
      T max = x; // 假设x是初始最大值
      if ( y.compareTo( max ) > 0 ){
         max = y; //y 更大
      }
      if ( z.compareTo( max ) > 0 ){
         max = z; // 现在 z 更大           
      }
      return max; // 返回最大对象
   }
```



##### **泛型类**

```java
public class Box<T> {
   
  private T t;
 
  public void add(T t) {
    this.t = t;
  }
 
  public T get() {
    return t;
  }
 
  public static void main(String[] args) {
    Box<Integer> integerBox = new Box<Integer>();
    Box<String> stringBox = new Box<String>();
 
    integerBox.add(new Integer(10));
    stringBox.add(new String("菜鸟教程"));
 
    System.out.printf("整型值为 :%d\n\n", integerBox.get());
    System.out.printf("字符串为 :%s\n", stringBox.get());
  }
}
```



##### **通配符/上下限**

```java
public static void getData(List<?> data) {
      System.out.println("data :" + data.get(0));
   }

List<?> listUknown = new ArrayList<A>();
List<? extends A> listUknown = new ArrayList<A>(); // 必须是A的子类
List<? super A> listUknown = new ArrayList<A>(); // 必须是A的父类
```



##### 类型擦除

Java泛型这个特性是从JDK 1.5才开始加入的，因此为了兼容之前的版本，Java泛型的实现采取了“**伪泛型**”的策略，即Java在语法上支持泛型，但是在编译阶段会进行所谓的“**类型擦除**”（Type Erasure），将所有的泛型表示（尖括号中的内容）都替换为具体的类型（其对应的原生态类型），就像完全没有泛型一样。理解类型擦除对于用好泛型是很有帮助的，尤其是一些看起来“疑难杂症”的问题，弄明白了类型擦除也就迎刃而解了。



擦除的原则：

- 消除类型参数声明，即删除`<>`及其包围的部分。

- 根据类型参数的上下界推断并替换所有的类型参数为原生态类型：如果类型参数是无限制通配符或没有上下界限定则替换为Object，如果存在上下界限定则根据子类替换原则取类型参数的最左边限定类型（即父类）。

- 为了保证类型安全，必要时插入强制类型转换代码。

- 自动产生“桥接方法”以保证擦除类型后的代码仍然具有泛型的“多态性”。



<img src="https://pdai.tech/images/java/java-basic-generic-1.png" alt="img" style="zoom: 50%;" />

<img src="https://pdai.tech/images/java/java-basic-generic-2.png" alt="img" style="zoom: 50%;" />

<img src="https://pdai.tech/images/java/java-basic-generic-3.png" alt="img" style="zoom:50%;" />





##### 泛型中的多态问题，桥接解决方案

类型擦除会造成多态的冲突，而JVM解决方法就是桥接方法。

这个问题的本质就是类型擦除导致泛型类型最终变成了Object，然后子类继承它的重写变成了重载，从而造成了多态的冲突。JVM使用了一种叫做“桥接”的方式来将针对这个场景下的重载变回重写，编译器自己生成的桥方法，然后这个桥方法代替了我们客观上的“重载”。

https://pdai.tech/md/java/basic/java-basic-x-generic.html#%E6%B3%9B%E5%9E%8B%E7%9A%84%E4%B8%8A%E4%B8%8B%E9%99%90



##### 泛型使用的几点注意事项

- 在异常的捕获和抛出中不能使用泛型机制，因为类型擦除机制会让它们都是Object或者某个共同的父类/子类
- 泛型类中的静态方法和静态变量不可以使用泛型类所声明的泛型类型参数，因为泛型类在建立对象时才产生，而静态方法在类初始化时便会执行 
- 泛型类不能实例化，否则就是new Object，没意义





#### Static关键字

##### 1.静态变量

```java
private static int y; 
```

也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它，并且静态变量在内存中只存在一份。而常规的实例变量这是每个对象都存在一份。



##### **2. 静态方法**

静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法(abstract)。它只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字。



##### **3. 静态代码块**

静态代码块在类初始化时运行一次。



##### **4. 静态内部类**

非静态内部类依赖于外部类的实例，而静态内部类不需要。

```java
public static void main(String[] args) {
        // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
```





#### Java初始化执行顺序

- 父类(静态变量、静态语句块)
- 子类(静态变量、静态语句块)
- 父类(实例变量、普通语句块)
- 父类(构造函数)
- 子类(实例变量、普通语句块)
- 子类(构造函数)





#### **类加载流程**

**装载**：

将class文件字节码内容加载到内存中，并将静态数据转化为方法区中的运行时数据结构，在堆中生成一个代表这个类的java.lang.Class对象，作为方法区类数据的访问入口



**连接**：

将java累的二进制代码合并到JVM的运行状态中
    验证：确保加载的类信息符合JVM规范
    准备：正式为类变量分配内存并设置初始值（初始化static变量），这些内存都在方法区中分配
    解析：JVM常量池的符号引用替换为直接引用



**初始化**：

​    执行类构造方法init()方法。该方法由编译器自动收集勒种所有变量的赋值动作和静态语句块(static块)中的语句合并产生
​    当初始化一个类的时候，如果发现其父类还没有初始化，则需要先触发其父类的初始化。
​    虚拟机会保证一个类的init()方法在多线程的环境下被正确的加锁和同步。
​    <u>当访问一个Java类的静态域时，只有真正声明这个域的类才会被初始化。</u>



**使用**：访问该类的属性和方法





#### **被动引用**

当含静态代码块的类是以被动引用的方式被引用的话，这个时候并不会走类加载逻辑，也就没有代码块初始化这个步骤了

被动引用包括：引用父类的静态变量；引用静态产量；以数组的形式而非new声明出来

当被动引用的时候，Java的类加载方法forName的initialize参数会被设置为**false**从而不会进行初始化



#### NIO

Java Non-Blocking IO

这种非阻塞的IO基于缓冲区来做的，以往在创建字节/字符流的时候都是byte by byte传输的，NIO中可以通过buffer来一段一段的传输，具备缓冲的功能

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230220003840612.png" alt="image-20230220003840612" style="zoom: 33%;" />

**buffer --- channel --- buffer**

NIO2当中提供了Paths、Files等工具类来更好地支持了对文件的操作



Java的阻塞IO的操作方式就比较就是两种，面向字节流的InputSream/OutputStream和面向字符流的InputSreamReader/OutputStreamReader



#### Java引用类型

**强引用**

我们用new方式创建的引用就是强引用。

```java
Client client = new Client()
```

只要一对象有强引用，垃圾回收机制不会回收



**软引用**

SoftReference实例来保存一个对象的软引用。

```java
SoftReference aSoftRef = new SoftReference(aRef);  
```

在内存不够用时，会优先回收只有软引用的内存空间，主要是做**缓存**用。



**弱引用**

只要是触发垃圾回收机制进行回收，只具有弱引用的对象就会被回收

- tomcat中就是使用的弱应用
- ThreadLocal也是使用弱应用 如：weakhashmap



**虚引用**

PhantomReference在调用get方法的时候总是返回Null，也就说明它不是旨在给Java用户来服务的

主要是**管理堆外内存**的，主要是给写jvm的使用

主要检测队列配合使用，虚引用API无法get到值，主要是通知对象已经被回收，去清理堆外的内存



**四种引用比较**

| 引用类型 | 被垃圾回收时间 | 用途               | 生存时间          |
| -------- | -------------- | ------------------ | ----------------- |
| 强引用   | 从来不会       | 对象的一般状态     | JVM停止运行时终止 |
| 软引用   | 当内存不足时   | 对象缓存           | 内存不足时终止    |
| 弱引用   | 正常垃圾回收时 | 对象缓存           | 垃圾回收后终止    |
| 虚引用   | 正常垃圾回收时 | 跟踪对象的垃圾回收 | 垃圾回收后终止    |



**ReferenceQueue**

WeakReference、PhantomReference在GC后都会给放到引用队列当中去

目的在于引用在被回收之后能让用户知道（通过监听这个queue），然后做出一些业务上的处理逻辑



#### StackOverFlow / OutOfMemeory

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230223164910920.png" alt="image-20230223164910920" style="zoom:33%;" />

![image-20230223163947131](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230223163947131.png)



##### **SOFE**

方法的调用过多，导致方法栈溢出

也可以Java内部用户定义出来的栈结构，但是这种可能性不是很大

这是一个Error，顶级父类是Error



##### OOM - Java Heap Space

堆空间满了，这个时候要明白什么数据放在堆空间中，主要是对象的信息，包括变量等

我们复现的方式可以是，在类中定义一个String变量，然后不断给它append新的字符上去，早晚会导致OOM



##### OOM - GC Overhead Limit Exceeded

这个问题的本质是由于堆空间内存不够GC非常频繁，但是最终却没能腾出太多的内存出来（2%），JVM认为当前系统有性价比不高，便退出运行

复现例子就是，定义一个运行时常量，把它的大小设置为巨大（～Xmx），它在Meta Space当中，GC再怎么去回收也触及不到这些内存



##### OOM - Direct Buffer Memeory

这个问题在Netty、Mina等NIO框架中很常见，根本的问题就是OS的本地内存不够了，但是JVM有没有办法去管理这部分的内存（堆外内存）

为了加速IO效率，我们会调用Native方法来为ByteBuffer分配本地OS内存，然后通过JVM中的DirectByteBuffer引用来调用这些数据

ByteBuffer.allocateDirect( capacity )

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230223172549240.png" alt="image-20230223172549240" style="zoom:25%;" />



##### # OOM - Unable To Create Native Thread

高并发的情况下会发生这种错误

单个进程Linux能够设置的最大线程数量就是1024，一般也就是900来个就挂了



##### OOM - Meta Space

JVM的Hotspot版本的元空间不再使用JVM内存，而是使用本地内存

元空间主要是类的框架

复现可以通过Enhancer类来不断创建类，直到达到元空间的大小上限





### JVM常问问题

#### JVM内存区域分布

![image-20220802212314085](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220802212314085.png)

---

DESCREPTION：分两大部分，一个是线程私有数据区，另外一个是共享数据区。私有区的话就是线程执行要用到的局部变量表、操作数栈，PC，本地方法栈。而共享区就是放的整个Java运行所需要的数据，包括运行时常量池、类信息、堆空间。

在堆空间中就是Java运行时所有生成的对象。每个对象的内存空间当中包括对象的实例数据（name、value）、本地线程私有空间的指针、class对象名、字符串常量池。

---

**程序计数器（Program Counter Register）：**当前线程所执行的字节码的行号指示器，字节码解析器的工作是通过改变这个计数器的值，来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能，都需要依赖这个计数器来完成；

**Java 虚拟机栈（Java Virtual Machine Stacks）：**用于存储局部变量表、操作数栈、动态链接、方法出口等信息；

**本地方法栈（Native Method Stack）：**与虚拟机栈的作用是一样的，只不过虚拟机栈是服务 Java 方法的，而本地方法栈是为虚拟机调用 Native 方法服务的；

---

**Java 堆（Java Heap）：**Java 虚拟机中内存最大的一块，是被所有线程共享的，几乎所有的对象实例都在这里分配内存；

**方法区（Methed Area）：**包括运行时常量池和类信息

**类信息：**类的元数据，诸如类型、方法、字段等信息

**运行时常量池：**基本类型的数字、文本字符串、类的全限定名

---

##### 类加载的过程（区别于对象的初始化）

类加载分为以下 5 个步骤：

**加载：**根据查找路径找到相应的 class 文件然后导入；

**检查：**检查加载的 class 文件的正确性；

**准备：**给类中的**静态变量**分配内存空间；

**解析：**虚拟机将常量池中的符号引用替换成直接引用的过程。符号引用就理解为一个标示，而在直接引用直接指向内存中的地址（在运行时常量池当中）；

**初始化：**对静态变量和静态代码块执行初始化工作。

---

#### **创建对象过程** 

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220728184948792.png" alt="image-20220728184948792" style="zoom: 25%;" />

- 类加载检查

  当遇到new指令之后，首先会去到**静态常量池**中看看能否找到这个指令所对应的**符号引用**，然后会检查**符号引用**所对应的**类**是否被**加载——连接——初始化**，如果有的话就进行第二步，如果没有就要先进行类的加载。

- 内存分配

  当类加载检查通过后，对象的大小在类加载完成之后就可以确定，所以首先会为准备新创建的对象根据对象大小**分配内存**，这块内存就是在堆中划分。那么如何进行内存的分配呢，一般情况下会有两种情况：**“指针碰撞” 和 “空闲列表” 两种**，选择那种分配方式由 **Java 堆是否规整**决定，而 **Java 堆是否规整**又由所采用的**垃圾收集器是否带有压缩整理功能决定**。

  <u>指针碰撞：</u>假设Java堆的内存是绝对规整的，所有用过的内存都放一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅把那个指针向空闲空间那边挪动一段与对象大小相等的距离。

  <u>空闲列表：</u>用表记录下空闲的内存块

- 值的初始化

  在准备过程中会将**final修饰**的静态变量直接赋初值，对**static修饰**的静态变量赋零值。虚拟机需要将分配到的内存空间中的数据类型都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

- 设置对象头

  初始化零值完成之后，虚拟机要对对象进行必要的设置，例如这个对象是**那个类的实例**、如何才能找到类的元数据信息、**对象的哈希码**、对象的 **GC 分代年龄**等信息。 这些信息存放在对象头中。 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

- 执行init

​		为对象的属性赋值，执行构造方法

---

#### 类加载器

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1447c58b0cc945fa98b4aeda2eb8eb68~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?" alt="bea.png" style="zoom: 50%;" />

**（父类）启动类加载器（Bootstrap ClassLoader）：**是虚拟机自身的一部分，用来加载Java_HOME/lib/目录中的，或者被 -Xbootclasspath 参数所指定的路径中并且被虚拟机识别的类库；

**扩展类加载器（Extension ClassLoader）：**负责加载\lib\ext目录或Java. ext. dirs系统变量指定的路径中的所有类库；

**应用程序类加载器（Application ClassLoader）：**负责加载用户类路径（classpath）上的指定类库，我们可以直接使用这个类加载器。一般情况，如果我们没有自定义类加载器默认就是用这个加载器。

---

#### 双亲委派机制

ClassLoader类中的loadClass方法的主要职责就是实现双亲委派机制：首先检查这个类是不是已经被加载过了，如果加载过了直接返回。否则委派给父加载器加载，这是一个递归调用，一层一层向上委派，最顶层的类加载器（启动类加载器）无法加载该类时，再一层一层向下委派给子类加载器加载

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220802220210974.png" alt="image-20220802220210974" style="zoom:25%;" />

java虚拟机只会类名相同且加载该类的加载器均相同的情况下才会判定这是一个类。 如果没有双亲委派机制，同一个类可能就会被多个类加载器加载，如此类就可能会被识别为两个不同的类，相互赋值时问题就会出现

相当与是遍历了全部的类加载器，固定的链路保证一个类的加载一直是由同一个类加载器来完成



##### 原生类加载的流程

(1) 类加载器从已加载的类中查询该类是否已加载，如果已加载则直接返回。

(2) 如果在已加载的类中未找到该类，则委托给父类加载器去加载`c = parent.loadClass(name, false)`，父类也会采用同样的策略查看自己加载的类中是否包含该类，如果没有则委托给父类，以此类推一直到启动类加载起。

(3) 如果启动类加载器加载失败（例如在`$JAVA_HOME/jre/lib`里未查找到该`class`），会使用拓展类加载器来尝试加载，继续失败则会使用`AppClassLoader`来加载，继续失败则会抛出一个异常`ClassNotFoundException`，然后再调用当前加载器的`findClass()`方法进行加载。



##### 自定义类加载

推荐重写findClass而非loadClass

因为loadClass会涉及到调用原生的findClass方法，最后的修改幅度会比较大

在使用上调用Class类的forName方法，然后传入name、loader

```java
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.nio.ByteBuffer;
import java.nio.channels.Channels;
import java.nio.channels.FileChannel;
import java.nio.channels.WritableByteChannel;
 
public class MyClassLoader extends ClassLoader
{
    public MyClassLoader(){}
    
    public MyClassLoader(ClassLoader parent)
    {
        super(parent);
    }
    
    protected Class<?> findClass(String name) throws ClassNotFoundException
    {
    	File file = new File("D:/People.class");
        try{
            byte[] bytes = getClassBytes(file);
            //defineClass方法可以把二进制流字节组成的文件转换为一个java.lang.Class
            Class<?> c = this.defineClass(name, bytes, 0, bytes.length);
            return c;
        } 
        catch (Exception e)
        {
            e.printStackTrace();
        }
        
        return super.findClass(name);
    }
    
    private byte[] getClassBytes(File file) throws Exception
    {
        // 这里要读入.class的字节，因此要使用字节流
        FileInputStream fis = new FileInputStream(file);
        FileChannel fc = fis.getChannel();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        WritableByteChannel wbc = Channels.newChannel(baos);
        ByteBuffer by = ByteBuffer.allocate(1024);
        
        while (true){
            int i = fc.read(by);
            if (i == 0 || i == -1)
            break;
            by.flip();
            wbc.write(by);
            by.clear();
        }
        fis.close();
        return baos.toByteArray();
    }
}

// 使用
MyClassLoader mcl = new MyClassLoader(); 
Class<?> clazz = Class.forName("People", true, mcl); 
Object obj = clazz.newInstance();
```



##### 如何破坏双亲委派机制

`Java.`这类的官方类不可以使用自定义的加载器来加载，必须由BootstrapClassLoader进行加载，这一点是无法破坏的

如果想自定义类加载器，就需要继承ClassLoader，并重写**findClass**。如果想不遵循双亲委派的类加载顺序，还需要重写**loadClass**，改变加载规则。破坏加载顺序是发生在AppClassLoader及以下的加载器级别中，从其往上走到Bootstrap是无法改变的



<u>Tomcat破坏双亲委派机制：</u>

....

最后每个WebApp里面的类都有自己所属的类加载器进行加载，做到了隔离性，可以实现一个Tomca中包含多个程序



<u>Tomcat需要解决两个问题</u>

1. 一个web容器可能需要部署两个应用程序，不同的应用程序可能会依赖同一个第三方类库的不同版本，不能要求同一个类库在同一个服务器只有一份，因此要保证每个应用程序的类库都是独立的，保证相互隔离
2. 部署在同一个web容器中相同的类库相同的版本可以共享。否则，如果服务器有10个应用程序，那么要有10份相同的类库加载进虚拟机，这是扯淡的



<img src="https://pica.zhimg.com/80/v2-371ed70e75e64b7d6fed16a05db59e4e_1440w.jpg?source=1940ef5c" alt="img" style="zoom:50%;" />

每个Tomcat的webappClassLoader加载自己的目录下的class文件，不会传递给父类加载器



##### 自定义类加载器的意义？

**加密**：如果你不想自己的代码被反编译的话。（类加密后就不能再用ClassLoader进行加载了，这时需要自定义一个类加载器先对类进行解密，再加载）

**从非标准的来源加载代码**：如果你的字节码存放在数据库甚至是云端，就需要自定义类加载器，从指定来源加载类

---



#### 垃圾回收系列

##### 垃圾如何被定义为垃圾？

<u>GCRoots资源可达性分析</u>

通过一系列的**“GC Roots”对象**作为起始点，从起始点开始向下搜索到对象的路径。搜索所经过的路径称为**引用链(Reference Chain)**，当一个对象到任何GC Roots都没有引用链时，则表明对象“**不可达”**，即该对象是不可用的。

GC Root Set，它是一个集合，一系列的对象可以作为起始对象，如下：

- 虚拟机栈（本地变量表）中引用的对象。也就是方法当中引用的对象
- 方法区中的静态属性引用的对象 private static
- 方法区中的常量引用的对象
- JNI本地方法 Native方法引用的对象

<img src="https://pic2.zhimg.com/80/v2-1b5f1a5869d24b3f1c31b69d853327d5_1440w.jpg" alt="img" style="zoom:50%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230223162339588.png" alt="image-20230223162339588" style="zoom: 25%;" />







<u>引用计数法</u>

给每个创建的对象添加一个引用计数器，每当此对象被某个地方引用时，计数值+1，引用失效时-1，所以当计数值为0时表示对象已经不能被使用。引用计数算法大多数情况下是个比较不错的算法，简单直接，但是它很难解决对象直接**相互循环引用**的问题。

```java
// 两个对象的引用永远都不为0，永远都不会存在于内存当中，因此是一种明显的内存泄露

public class MyObject {
 
public Object ref = null;
 
public static void main(String[] args) {
 
MyObject myObject1 = new MyObject();
 
MyObject myObject2 = new MyObject();
 
myObject1.ref = myObject2;
 
myObject2.ref = myObject1;
 
myObject1 = null;
 
myObject2 = null;
} 
}
```



##### 引用类型介绍

强引用：发生 gc 的时候不会被回收。

软引用：有用但不是必须的对象，在发生内存溢出之前会被回收。

弱引用：有用但不是必须的对象，在下一次GC时会被回收。

虚引用（幽灵引用/幻影引用）：无法通过虚引用获得对象，用 PhantomReference 实现虚引用，虚引用的用途是在 gc 时返回一个通知。



##### 垃圾回收算法

<u>标记-清除算法：</u>标记无用对象，然后进行清除回收。缺点：效率不高，无法清除垃圾碎片

<u>标记-整理-清除算法：</u>标记无用对象，让所有存活的对象都向一端移动，然后直接清除掉端边界以外的内存

<u>复制算法：</u>按照容量划分二个大小相等的内存区域，当一块用完的时候将活着的对象复制到另一块上，然后再把已使用的内存空间一次清理掉。缺点：内存使用率不高，只有原来的一半



<u>引用计数：</u>按照该对象是否还存在引用来判断是否给予清除 UseSerialOldGC 已经淘汰了





##### **关于FullGC和MinorGC的问题**

**MinorGC：**在Eden区和Survivor空间不足的时候触发，开销比较小，执行的过程也很快速

**MajorGC/Full GC：**前者只回收老年代，而后者是整堆回收，通常混淆二者的用法是因为它们的触发场景非常的类似，下面是Full GC触发的场景

- 老年代空间不足
- MetaSpace空间不足
- 调用gc方法
- 由MinorGC进入老年代的对象的平均大小 大于 老年代当前的可用空间
- 大对象直接进入老年代而导致老年代的内存不足





##### 垃圾回收器的分类

分代与不分代的区别在于逻辑上有没有换代这种操作，对象在eden、from/to survivor、old区之间不断地变动

新生代回收器与老年代回收器的区别，**新生代一般使用的复制算法**，优先是效率高，缺点是内存利用率低；**而老年代一般使用标记-清除/标记-整理算法**，内存的利用率很高。JDK8默认的是Parallel就已经够用了

现在最新的是G1，从三个地方进行了优化：一个是并行度，加快了时间效率；二个是空间整理算法，加大了空间的利用率；第三个是仍然保持了新生代和老年代区别优化的设计



<u>分代垃圾回收器</u>

​	新生代回收器：Serial、Parallel Scavenge

​	老年代回收器：CMS、Parallel Old

<u>不分代垃圾回收器</u>：ZGC、Epsilon

> jdk1.8 默认垃圾收集器Parallel Scavenge（新生代）+Parallel Old（老年代）



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224140900226.png" alt="image-20230224140900226" style="zoom: 33%;" />

命令行 java -XX:+PrintCommandLineFlags -version

-XX:+PrintGCDetails 

-XX:+UseCMSGC 虚拟机参数



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224142059845.png" alt="image-20230224142059845" style="zoom:33%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224145521421.png" alt="image-20230224145521421" style="zoom:33%;" />

-XX：+UseParNewGC是生产中比较常用的（新生代用并行，老年代用串行）

-XX:ParallelGCThreads 限制GC并行的线程数，默认为CPU的核数

ParallelScavenge和ParNew的区别有：1. 前者可控GC的吞吐量，也就是多少CPU时间用于垃圾清除  2. 前者可以根据系统负载来调节GC的停顿时间 -XX:MaxGCPauseMillis（生产上推荐是设置为100ms）





##### Concurrent Mark Swap

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224170112906.png" alt="image-20230224170112906" style="zoom:33%;" />

-XX:+UseConcMarkSweepGC

大型互联网应用最佳收集器（G1除外）



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224170735476.png" alt="image-20230224170735476" style="zoom:25%;" />

缺点：并发执行对CPU的压力比较大，它必须在老年代堆内存耗尽之前完成回收，否则会退化到Serial Old





##### G1底层原理

![image-20230224174013671](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224174013671.png)

G1还有个特性便是可以手动设置STW stop the world的间隔时间 ，化整为零，更加灵活



在内存块划分的这个问题上采用了一个**Region**的概念，一个Region块专属于一个代

![image-20230224174942980](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224174942980.png)



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224175524246.png" alt="image-20230224175524246"  />



新生代的收集策略也还是将所有用户线程暂停，然后把命中对象拷贝到Survivor或者Tenured区中，一块一块整体地拷贝过去；对于**巨型对象Humongous**，会分配好几个连续的块存放

![image-20230224175901499](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224175901499.png)



G1收集器的工作流程

![image-20230224180121156](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224180121156.png)







<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224172320723.png" alt="image-20230224172320723" style="zoom:33%;" />

![image-20230224171406439](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224171406439.png)









#### JVM调优常用参数

参数值的类型通常有两种，一是数值类型，而是布尔类型



-Xms2g：初始化推大小为 2g；等价于 **-XX:InitialHeapSize**

-Xmx2g：堆最大内存为 2g；等价于 **-XX:MaxHeapSize**

-Xmn500M 设置年轻代为500m;

-XX:PermSize=500M ； 1.8之后采用 MetaspaceSize

-XX:MaxPermSize=500M ； 1.8之后采用 MaxMetaspaceSize

##### **-XX:NewRatio=4**

设置年轻的和老年代的内存比例为 1:4

默认是2，即Old占2，New占1

##### **-XX:SurvivorRatio=8**

设置新生代 Eden 和 Survivor 比例为 8:2

具体的比例两个S区是相同的，即8 : 1 : 1

##### **-XX:MaxTenuringThreshold**

Survivor区中From和To来回交换的次数

交换超过这个次数就进入老年代，Java8中写死为15了



–XX:+UseParNewGC：指定使用 ParNew + Serial Old 垃圾回收器组合；

-XX:+UseParallelOldGC：指定使用 ParNew + ParNew Old 垃圾回收器组合；

-XX:+UseConcMarkSweepGC：指定使用 CMS + Serial Old 垃圾回收器组合；

-XX:+PrintGC：开启打印 gc 信息；

-XX:+PrintGCDetails：打印 gc 详细信息；

-XX:+PrintCommandLineFlags：打印JVM的基本信息；

-XX:MaxDirectMemorySize : 设置直接内存的大小。

调优工具：IDEA + JProfiler



命令行 jinfo -flags [pid] 查看JVM全部参数

MaxHeapSize通常为最大内存的1/4

-xx:OldSize    -xx:NewSize 可以更为精细化地去调节堆内存



##### -XX:MetaSpaceSize

元空间不在JVM当中，它存在于本地内存里，但是也需要给其分配合理的大小。元空间默认的大小一般也就是几十MB，在大型业务下很快就会被撑爆了

一般设置为1G就差不多了，元空间里面一般都是类的信息（框架）不需要太大的内存空间
