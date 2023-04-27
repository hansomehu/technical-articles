---
layout: post
title: "Java JUC"
permalink: /java-juc
---

这篇文档介绍Java并发框架包Java.Util.Concurrent下的并发支持工具以及Java对高并发、线程安全的支持

关键字 + CAS + JUC包内工具



### JUC



![image-20220725225308513](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220725225308513.png)

https://blog.csdn.net/weixin_43767015/article/details/106952220

并发队列就是各种的blocking queue



#### ConcurrentHashMap

JUC包的Collections子包

JDK1.7中用Segment方式实现加锁，就是把全部数据放在**16个Segment**里面（默认16），一次加锁只锁一个Segment，提高了资源效率。Segment最大就是16，但是每个Segment的容量可以扩容，初始值是2，负载因子是0.75，元素个数达到1.5个时（采用溢出形式）会按照原先的**2倍**进行扩容

在JDK1.8中，ConcurrentHashMap的实现原理摒弃了Segment的设计，而是选择了与HashMap类似的**数组+链表+红黑树**的方式实现，而加锁则采用**CAS和synchronized**实现

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220511144716206.png" alt="image-20220511144716206" style="zoom: 33%;" />

sizeCtl = 【 (1.5 * initialCapacity + 1)，然后向上取最近的 2 的 n 次方】。初始容量是10，然后按照上述的公式进行扩容，每次扩容为原来的**2倍**，这边要明确扩容是扩的数组的容

如果数组长度小于 **64** 的时候通常不会考虑建立红黑树，因为开销很大。这里并没有一个绝对的建立红黑树的机制，关键是看红黑树和数组扩容之间的trade-off

在对sizeCtl进行操作的时候用到了CAS锁的方式，具体的过程就是先比较sizeCtl的值和原值是否相等（调这个compareAndSwapInt方法），相等则进行替换；

<!--CHM获取锁的这个流程无比重要-->

在PUT操作的过程中，同时用到了CAS和Synchorized，具体流程：

0. 进入一个**for**循环

1. 对要插入的Key进行hash，确定在数组上的槽位（获得数组链表的下一个节点Node）
2. 如果数组为空，则先initTable
3. 如果数组的该槽位为空，通过CAS方式插入该节点Node，如果**CAS失败会break出去进入下一轮的循环**，这个时候Compare的值会更新
4. 如果不为空则用**synchorized锁住这个链表头节点**执行插入操作

> 7.29百度面试：CHM获取锁的过程







#### LockSupport

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230227125854279.png" alt="image-20230227125854279" style="zoom:50%;" />

LockSupport是一套相较于传统的wait/await/notigy/signal更加灵活的线程阻塞和唤醒方案

传统的方案在进行线程的阻塞/唤醒时存在一些限制：

- 必须同现在同步代码块或者Lock之下
- 必须成对出现，否则抛异常



而LockSupport的park/unpark方法则更加灵活，它内部包含一个全局的permit成员变量调用park方法会使得permit-1，而调用unpark会使得permit+1，当permit=0的时候该线程阻塞，permit初始值为1，它的最大值为1。这种方式让全局只关注volatile 的 permit的值即可，不需要放在同步代码块当中或者要求park/unpark方法成对出现

LockSupport的park/unpark方法由Native方法提供底层的上锁机制

LockSupport也是构成Java并发框架核心类AQS的基石







#### AQS AbstractQueuedSynchronizer

[美团技术文章](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)

AQS是一个用来**构建锁和同步器的框架**，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。



AQS定义两种资源共享方式

- **Exclusive(独占)：**只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：
  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- **Share(共享)：**多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatCh、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230226110143594.png" alt="image-20230226110143594" style="zoom: 33%;" />

AQS为Java并发安全开发规范了一个体系，AQS是一个抽象类，也是所有的JUC包下的锁、同步器等等工具所依赖的类。在AQS 里面定义了用来开发锁所需要的很多顶层方法，比如tryAcquire、acquire、release等，我们通过这些方法就可以构造出简单的锁工具出来。

**AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。**如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用**CLH（Craig, Landin, and Hagersten）队列锁**实现的，即将暂时获取不到锁的线程加入到队列中。CLH是一个虚拟的双向队列，只维护了节点之间的关系，并没有实例。

非公平的实现就是在任务开始时会先CAS一下来尝试获取资源，如果获取到了则占有资源，如果没获取到则入CLH队列





#### **CountDownLatch** 

**主线程等待其他线程执行完毕**

具有计数器的功能，等待其他线程执行完毕，**主线程**再继续执行，用于**监听某些初始化操作**，并且线程进行阻塞，等初始化执行完毕后，通知主线程继续工作执行

维护了一个计数器 cnt（当前线程总数），每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220511161233382.png" alt="image-20220511161233382" style="zoom:33%;" />

```java
// 用CountDownLatch的时候保证await调用之前的逻辑先执行（也就是end要最后一个打印）
public class CountDownLatchExample {

    public static void main(String[] args) throws InterruptedException {
        final int totalThread = 10;
        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("run..");
//                countDownLatch.countDown();
            });
        }
//        countDownLatch.await();
        System.out.println("end");
        executorService.shutdown();
    }
}
```



##### 当有线程挂了如何保证countdown顺利完成，以及其他一些使用注意事项？

用try catch代码块来做业务操作，在finally当中countdown就能保证一定会退出了，还有就是设置timeout



#### **CyclicBarrier**  

**所有的线程相互等待**

用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

```java
public class CyclicBarrierExample {

    public static void main(String[] args) {
        final int totalThread = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("before..");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.print("after..");
            });
        }
        executorService.shutdown();
    }
}
```





### Java多线程



#### ThreadLocal

ThreadLocal是一个将在多线程中为每一个线程创建单独的变量副本的类; 当使用ThreadLocal来维护变量时, ThreadLocal会为每个线程创建单独的变量副本, 避免因多线程操作共享变量而导致的数据不一致的情况。



##### TLM ——ThreadLocal的管理器

Thread类维护了一个**ThreadLocalMap**的数据结构，它的key为当前线程对象，value为存储的数据对象。这个Map具备如下一些特性：

- 它没有实现Map接口

- 它没有public的方法, 最多有一个default的构造方法, 因为这个ThreadLocalMap的方法仅仅在ThreadLocal类中调用, 属于静态内部类

- ThreadLocalMap的Entry实现继承了**WeakReference<ThreadLocal<?>>**该方法仅仅用了一个Entry数组来存储Key, Value; Entry并不是链表形式, 而是每个bucket里面仅仅放一个Entry



获取到Value的过程：

```java
// 首先调用get
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap threadLocals = getMap(t);
    if (threadLocals != null) {
        ThreadLocalMap.Entry e = threadLocals.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

// 如果没有则走initiate流程，该方法可以重写
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}

```



还可以在一个多线程类中定义一个ThreadLocal对象，为其设置一个能够getter、setter的方法，然后在外部通过这些途径来获取该对象

```java
public class ThreadLocalTest implements Runnable{
    
    ThreadLocal<Student> StudentThreadLocal = new ThreadLocal<Student>();

    @Override
    public void run() {
        String currentThreadName = Thread.currentThread().getName();
        System.out.println(currentThreadName + " is running...");
        Random random = new Random();
        int age = random.nextInt(100);
        System.out.println(currentThreadName + " is set age: "  + age);
        Student Student = getStudentt(); //通过这个方法，为每个线程都独立的new一个Student对象，每个线程的的Student对象都可以设置不同的值
        Student.setAge(age);
        System.out.println(currentThreadName + " is first get age: " + Student.getAge());
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println( currentThreadName + " is second get age: " + Student.getAge());
        
    }
  
  private Student getStudentt() {
        Student Student = StudentThreadLocal.get();
        if (null == Student) {
            Student = new Student();
            StudentThreadLocal.set(Student);
        }
        return Student;
    }
}

```



**内存泄露问题，**首先ThreadLocal是**WeakReference**，只有在GC的时候才会消除

其次，在使用线程池来启动线程，它不会自动销毁掉线程，这会导致该部分的引用不会回收

所以，为了安全地使用ThreadLocal，必须要像每次使用完锁就解锁一样，在每次使用完ThreadLocal后都要调用remove()来清理无用的Entry



##### InheritableThreadLocal

通过这个类定义的数据可以被子线程所共享







#### **手动启用多线程的方式**

- Callable接口

- Runnable接口

- Thread类

实现接口的开销比继承整个类要小，故建议通过接口的方式来实现多线程



#### **ThreadPoolExecutor**

说白了就是一个**线程集合workerSet**和一个**阻塞队列workQueue**。当用户向线程池提交一个任务(也就是线程)时，线程池会先将任务放入workQueue中。workerSet中的线程会不断的从workQueue中获取线程然后执行。主要任务就是负责工作线程的生命周期，开始、任务执行、结束。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220731171322571.png" alt="image-20220731171322571" style="zoom: 33%;" />



Executors有下面一些默认的线程池类型：

> 但是不推荐这么来做

- CachedThreadPool：一个任务创建一个线程，导致线程过多容易OOM；
- FixedThreadPool：所有任务只能使用固定大小的线程，请求堆积容易导致OOM；
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

FutureTask + ExecutorService是比较推荐的方案，具备线程管理而且还带返回值



但同时，Executors还支持自定义线程池的规格，详细的参数如下。上面哪些默认的线程池不过是把某些参数写死了的自定义线程池罢了。

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize, 
                        // 线程空闲时的存活时间，即当线程没有任务执行时，该线程继续存活的时间
                              long keepAliveTime,
                          // 上面的单位
                              TimeUnit unit, 
                          // 阻塞队列
                              BlockingQueue<Runnable> workQueue,
                         // 线程池的饱和策略，当线程池满了还有任务来了应该如何处理
                              RejectedExecutionHandler handler) 

  
  // 怎么用 直接new 然后填参数 最后再丢runnable对象进去
  // Executors直接调内部的方法就好
```



#### 合理的线程数量配置

分为CPU密集型和IO密集型

针对**CPU密集型**，CPU一直在全力运行，这个时候多线程没有多大的实际效用。CPU核心+1就够了，多一个线程出来做备份

针对**IO密集型**，这个时候如果线程不够多，CPU时间就会被等待给耗尽。最常用的方式是CPU数量 * 2，这种方式能保证大量场景下发挥比较的性能

要追求更仔细的话，就要看平均每个IO耗时情况，计算它的阻塞系数（IO中断在一秒内的占比），然后CPU核数 / 1 - 阻塞系数



#### 阻塞队列

ArrayBlockingQueue

LinkedBlockingQueue

PriorityQueue 这个是能够自己定义优先级

DelayBlockQueue 这个能自己定义任务的延期



#### RejectedHandler

`AbortPolicy`: 直接抛出异常，默认策略；

`CallerRunsPolicy`: 用调用者所在的线程来执行任务；

`DiscardOldestPolicy`: 丢弃阻塞队列中靠最前的任务，并执行当前任务；

`DiscardPolicy`: 直接丢弃任务；

总结来说都是丢弃一些线程，区别在于丢弃什么地方的线程



####  Execute原理

当一个任务提交至线程池之后：

1. 线程池首先当前运行的线程数量是否少于**corePoolSize**。如果是，则创建一个新的工作线程来执行任务。如果都在执行任务，则进入BQ
2. 判断**BlockingQueue**是否已经满了，倘若还没有满，则将线程放入BlockingQueue。否则进入3.
3. 如果创建一个新的工作线程将使当前运行的线程数量超过**maximumPoolSize**，则交给RejectedExecutionHandler来处理任务。

当ThreadPoolExecutor创建新线程时，通过CAS来更新**线程池的状态ctl.**

```java
// 线程池的内部状态
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }

```



**Execute执行流程**

submit -> execute –> addWorker –> runWorker (getTask)

execute：这步里面最重要的事情是对线程池的状态做一个double check，保证线程池处于running状态。在多线程环境下，线程池的状态时刻在变化，而ctl.get()是非原子操作，很有可能刚获取了线程池状态后线程池状态就改变了

addWorker：创建工作线程，这步需要获取全局锁（ReentrantLock）

runWorker：任务的具体执行；继承了AQS类，可以方便的实现工作线程的中止操作；实现了Runnable接口，可以将自身作为一个任务在工作线程中执行

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220511123911768.png" alt="image-20220511123911768" style="zoom:67%;" />



##### **为什么线程池不允许使用Executors去创建? 推荐方式是什么?**

线程池不允许使用Executors去创建，而是**通过ThreadPoolExecutor**的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：Executors各个方法的弊端：

- new FixedThreadPool和new SingleThreadExecutor:  主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至**OOM**。 此外，newFixedThreadPool线程池的线程数量达corePoolSize后，即使线程池没有可执行任务时，也**不会释放线程**
- new CachedThreadPool和new ScheduledThreadPool:  主要问题是线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至**OOM**
- 在阻塞队列方面，它的队列是没设置大小的，容易导致**OOM**





##### **ScheduledThreadPoolExecutor**

随着业务量的不断增大，我们可能需要多个工作线程运行任务来尽可能的增加产品性能，或者是需要更高的灵活性来控制和监控这些周期业务。这些都是 ScheduledThreadPoolExecutor 诞生的必然性。

Executors通过newScheduledThreadPool(1, threadFactory) 和new SingleThreadScheduledExecutor来创建。但二者并非前者的第一个参数设置为1就等价于后者，前者的线程数可以根据setCoreSize进行更换，而后者是固定的。

有两种工作模式，一是以**固定周期执行**，二是以**固定延迟时间执行**

核心方法：

**Schedule** :用于执行一次性的延迟任务

**run:**  setNextRunTime 设置下一次执行的时间；reExecutePeriodic()周期任务重新入队等待下一次执行；继承自其父类FutureTask

**scheduleAtFixedRate**

**scheduleWithFixedDelay**



#### 监控线程池的状态

可以使用ThreadPoolExecutor以下方法：

- `getTaskCount()` Returns the approximate total number of tasks that have ever been scheduled for execution
- `getCompletedTaskCount()` Returns the approximate total number of tasks that have completed execution. 返回结果少于getTaskCount()
- `getLargestPoolSize()` Returns the largest number of threads that have ever simultaneously been in the pool. 返回结果小于等于maximumPoolSize
- `getPoolSize()` Returns the current number of threads in the pool
- `getActiveCount()` Returns the approximate number of threads that are actively executing tasks





### Java并发关键字与锁



#### 锁到底锁住了什么？原子性揭秘

原子性在C/C++这类底层语言中指的是一个操作只涉及到**对寄存器的一次操作**

原子性就和数据库的原子性一样的，**一个或多个操作，要么全部执行且在执行过程中不被任何因素打断，要么全部不执行**。**在Java中当我们讨论一个操作具有原子性问题是一般就是指这个操作会被线程的随机调度打断**

Java线程安全的三大特性：**原子性、可见效性、有序性**

Java保证原子性的方法：

- CAS
- Lock锁
- Synchronous
- Atomic类
- 基本类型原生支持原子性；但是，对long和double的操作无法保证原子性，因为这种类型包含了两个部分，正部和负部



#### JMM

**目的是保障线程执行的安全性，使其满足下面的三大特性**



**1 原子性**
        由Java内存模型来直接保证的原子性操作包括read,load,assign.use,store,和write。

        实现：

synchronized

reentrantlock

readWriteLock

concurrent包 
**2 可见性**
        可见性，是指线程之间的可见性，一个线程修改的状态对另一个线程是可见的。也就是一个线程修改的结果。另一个线程马上就能看到。比如：用volatile修饰的变量，就会具有可见性。

        实现：

volatile
synchronized
final
        注意：volatile只能让被他修饰内容具有可见性，但不能保证它具有原子性。比如 volatile int a = 0；之后有一个操作 a++；这个变量a具有可见性，但是a++ 依然是一个非原子操作，也就是这个操作同样存在线程安全问题。

**3 有序性**
        Java 语言提供了 volatile 和 synchronized 两个关键字来保证线程之间操作的有序性，volatile 是因为其本身包含“禁止指令重排序”的语义，synchronized 是由“一个变量在同一个时刻只允许一条线程对其进行 lock 操作”这条规则获得的，此规则决定了持有同一个对象锁的两个同步块只能串行执行。

        实现：

volatile：关键字来保证一定的“有序性”
synchronized
lock+unlock

final



**禁止重排序的案例，**在多线程的环境下这样能保证结果的唯一性，在使用了volatile的变量上会通过内存屏障的机制去禁止它的重排序

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230218125120253.png" alt="image-20230218125120253" style="zoom:33%;" />





#### 锁基础

较为主流的两把锁

##### 乐观锁：CAS + 自旋

##### 悲观锁：synchorized、ReentrantLock



##### **死锁条件和预防**

你说破坏互斥条件，那必须要有资源保持互斥怎么办（加锁的时候设置超时时间，超过超时时间就放弃重新竞争锁，长时间没能获取的资源的进程释放已有的资源）

<u>互斥：</u>某种资源一次只允许一个进程访问，即该资源一旦分配给某个进程，其他进程就不能再访问，直到该进程访问结束
<u>占有且等待：</u>一个进程本身占有资源（一种或多种），同时还有资源未得到满足，正在等待其他进程释放该资源
<u>不可抢占：</u>别人已经占有了某项资源，你不能因为自己也需要该资源，就去把别人的资源抢过来
<u>循环等待：</u>存在一个进程链，使得每个进程都占有下一个进程所需的至少一种资源

破坏占有及等待条件：该方法是对第一种方法的改进，允许进程只获得运行初期需要的资源，便开始运行，在运行过程中逐步释放掉分配到的已经使用完毕的资源，然后再去请求新的资源

破坏不可抢占条件：当一个已经持有了一些资源的进程在提出新的资源请求没有得到满足时，它必须释放已经保持的所有资源，待以后需要使用的时候再重新申请

破坏循环等待条件：就是给线程设一个优先级



**锁的状态**

JDK 1.6 引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：**无锁状态**（unlocked）、**偏向锁状态**（biasble）、**轻量级锁状态**（lightweight locked）和**重量级锁状态**（inflated）。



#### 锁底层

Java中锁的底层原理，临界资源只允许单一线程进行访问是如何做到的？

CPU由相关的指令，针对某些寄存器，某时刻只允许单一线程进行访问

在内核态，相关的程序负责线程的调度，将该临界资源分配给它

在用户态，主要是通过AQS的CLH队列来实现，需要访问临界资源的线程先通过CAS对其进行访问；如果访问失败，则进入CLH队列等待



<!--这块内容少，不需要太深入-->

<!--锁膨胀：偏向锁	-	轻量级锁（不断轮询）-	重量级锁（阻塞）   -->

<!--JVM的锁优化：锁消除（编译的时候取消无意义的锁）、锁粗化（锁住更大的范围）-->

#### 锁膨胀

JVM中在处理synchorized的时候会即根据实际情况一步步地去升级锁，由轻到重

具体实现方式就是对象Mark Word当中的标记位，在偏向锁和轻量级锁中，MW指向特定的标记位则意味着获取到了特定的锁，而不需要去互斥锁住资源



> 无锁 → 偏向锁 → 轻量级锁 → 重量级锁 (此过程是不可逆的)

无锁：初始状态

偏向锁：

轻量级锁：利用逃逸原理，不断自旋，在最短的时间内获取到锁

重量级锁：就是阻塞原理

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220807171640518.png" alt="image-20220807171640518" style="zoom:33%;" />



#### JVM锁优化，让锁的性能更高

> **锁消除**

锁消除是指虚拟机即时编译器 **JIT** 在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除。锁消除的主要判定依据来源于逃逸分析的数据支持。

例如String类，把这种不可变资源锁住是没有多大意义的。

> **锁粗化**

在连串的一系列操作都对同一个对象反复加锁和解锁，甚至加锁操作时出现在循环体中的，那即使没有线程竞争，频繁的进行互斥同步操作也会导致不必要的性能操作。在这种情况下，JVM会将一系列加锁操作升级为一个大锁。

```java
// 会直接锁住StringBuilder而非连续对append操作加锁
public static String test04(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```





> Synchorized和Reentrantlock在Java中的悲观锁里较为常用，需要熟悉到底层工作流程

#### **Synchronized**

由JVM实现，也是可重入的，它只有非公平这个选项

注意事项：

一把锁只能同时被一个线程获取，没有获得锁的线程只能等待；

每个实例都对应有自己的一把锁(this),不同实例之间互不影响；但是存在例外：锁对象是*.class以及synchronized修饰的是static方法的时候，所有对象公用同一把锁

synchronized修饰的方法，无论方法正常执行完毕还是抛出异常，都会释放锁



锁对象（变量）、代码块、方法

```java
// **************************Synchronized的使用方式**************************
// 1.用于代码块
synchronized (this) {}
// 2.用于对象
synchronized (object) {}
// 3.用于方法
public synchronized void test () {}
// 4.可重入
for (int i = 0; i < 100; i++) {
	synchronized (this) {}
}
// **************************ReentrantLock的使用方式**************************
public void test () throw Exception {
	// 1.初始化选择公平锁、非公平锁
	ReentrantLock lock = new ReentrantLock(true);
	// 2.可用于代码块
	lock.lock();
	try {
		try {
			// 3.支持多种加锁方式，比较灵活; 具有可重入特性
			if(lock.tryLock(100, TimeUnit.MILLISECONDS)){ }
		} finally {
			// 4.手动释放锁
			lock.unlock()
		}
	} finally {
		lock.unlock();
	}
}
```



##### 可重入（递归锁）的实现

`Monitorenter`和`Monitorexit`指令，会让对象在执行，使其锁计数器加1或者减1。每一个对象在同一时间只与一个monitor(锁)相关联，而一个monitor在同一时间只能被一个线程获得，一个对象在尝试获得与这个对象相关联的Monitor锁的所有权的时候，monitorenter指令会发生如下3中情况之一：

- monitor计数器为0，意味着目前还没有被获得，那这个线程就会立刻获得然后把锁计数器+1，一旦+1，别的线程再想获取，就需要等待
- 如果这个monitor已经拿到了这个锁的所有权，又重入了这把锁，那锁计数器就会累加，变成2，并且随着重入的次数，会一直累加
- 这把锁已经被别的线程获取了，等待锁释放

`monitorexit指令`：释放对于monitor的所有权，释放过程很简单，就是讲monitor的计数器减1，如果减完以后，计数器不是0，则代表刚才是重入进来的，当前线程还继续持有这把锁的所有权，如果计数器变成0，则代表当前线程不再拥有该monitor的所有权，即释放锁

##### monitor的可见性

对monitor对象的监控用到了happens-before原则来保证其可见性，即线程A对锁的释放happens before线程B对锁的获取

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230224204856402.png" alt="image-20230224204856402" style="zoom:25%;" />

这种可重入的机制可以避免死锁，但是在实际的编码中可重入可以理解为避免死锁的必要条件而非充分条件（当内部的锁已经被其他线程占有了的话就没有办法）





#### **ReentrantLock**

JDK发行版实现，不同的JDK会不一定有这个功能，并且不具备自动释放的功能

他的公平锁和非公平锁都是基于AQS来做的，在ReentrantLock的源码中只停留在了Acquire方法上，进一步的源码需要去AQS中学习

```java
// java.util.concurrent.locks.ReentrantLock#NonfairSync

// 非公平锁
static final class NonfairSync extends Sync {
	...
	final void lock() {
		if (compareAndSetState(0, 1))
			setExclusiveOwnerThread(Thread.currentThread()); // 区别在于这里，非公平有这么一个抢占的逻辑
		else
			acquire(1);
		}
  ...
}

// java.util.concurrent.locks.ReentrantLock#FairSync

static final class FairSync extends Sync {
  ...  
	final void lock() {
		acquire(1);  // 公平的实现则需要交给AQS来完成
	}
  ...
}
```

<img src="https://p0.meituan.net/travelcube/412d294ff5535bbcddc0d979b2a339e6102264.png" alt="img" style="zoom:50%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230226111348654.png" alt="image-20230226111348654" style="zoom: 50%;" />



##### AQS中主要的结构 State + CLH队列

```java
public class AbstractQueuedSynchronized{
  
  private volatile int state; //资源状态
  
  class Node {
    volatile Node prev;  
    volatile Node next;    
    volatile Thread thread;
    volatile int waitStatus; // Node等待状态 0 CANCELED CONDITION PROPAGATE -1SIGNAL
  }
}

/*
公平与非公平的实现：
在具体的之类中，通过 new FairSync 或者 new NonFairSync 实现
具体到方法则是 tryAcquire() 和 tryNonFairAcuqire()
公平的方式多了下面这个判断方法
*/

// 公平锁的实现，区别就在于循环判断 !hasQueuedPredecessors() 是否成立
protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }


// 判断完成后返回上一级
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) //添加队列等待
            selfInterrupt();
    }

// 入队列代码
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node); // 当前队列为空的入队列逻辑，包含队列的初始化
        return node;
    }

// 入队列的具体实现
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }

// 节点信息入完队列之后再次判断能否抢占到锁，如果依旧不能，则调用park方法阻塞该线程，等到unlock的唤醒
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) { // 非公平就体现在这里，在真正被阻塞之前再抢一次锁，如果成功的话，对其他已经出于阻塞状态的线程来说就是不公平的
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())  // 真正进入阻塞状态
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }


/*
释放锁的过程
*/
// tryRelease的工作完成后就调用LockSupport的unparkSuccessor正式唤醒线程
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

// 主要的任务就是设置state变量为0（空闲），然后将占有线程置为null
protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```



队列中的初始节点为一个pseudo的哨兵节点，当B线程开始占有资源的时候，这个时候CLH队列的头节点（节点）就变为了节点B，以此类推

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230226151623574.png" alt="image-20230226151623574" style="zoom:33%;" />





<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230226114324373.png" alt="image-20230226114324373" style="zoom: 25%;" />





#### Lock

主要的时候方式`lock.lock() / lock.tryLock() `前者没获取到锁会阻塞，直到拥有锁；后者则是直接返回

Lock是一个接口，它的实现主要有ReentrantLock和ReadWriteLock

任何Lock类的实现类，在里面都**聚合了一个AQS的子类，**通过调用AQS的lock、unlock方法来实现目的

 

#### CAS自旋锁 ABA问题

##### **Compare and Swap**

和上面的两种阻塞同步方式对比，CAS这种**非阻塞**方式的**乐观锁**开销更低。CAS操作需要输入两个数值，一个旧值(期望操作前的值)和一个新值，在操作期间先比较下在旧值有没有发生变化，如果没有发生变化，才交换成新值，发生了变化则不交换。CAS的实现方式是基于硬件平台的汇编指令，就是说**CAS是靠硬件实现**的，一个**寄存器**在一个特定的时间节点只允许一条CPU指令对其进行操作。

```java
Unsafe.getUnsafe().compareAndSwapInt()
// 自旋就是丢进一个for循环，失败不断重试
// 但是一定要设置退出的条件，或者要限制次数
```



**AtomicReference**可以为任何类实现CAS操作

`AtomicReference<T> demo = new AtomicReference<>();` 然后调用它的set、compareAndSet方法来设置具体的值 



##### **CAS存在的并发问题**

<u>ABA问题：</u>

A=3，某CAS做C的时候读到的是3，然后另外一个线程把A改为了4，随后又改为了3，这个时候第一个线程到了S阶段正常执行。然后问题在于有些业务不允许A做过更改，所以CAS以结果为导向来看没有问题，但是以过程为导向来看就存在问题了

好解决，给数据加**版本号**即可。Java中的原子操作类**AtomicStampedReference** / **AtomicMarkableReference**就很有用了，这允许一对变化的元素进行原子操作，其本质也是类似于给数据加版本

`AtomicStampedReference<T> demo = new AtomicStampedReference<T> (initVal, initVersion);`



<u>保证对多个变量的原子操作：</u>

有两个共享变量i = 2，j = a，合并一下ij = 2a，然后用CAS来操作ij



#### **final**

修饰类：当某个类的整体定义为final时，就表明了你不能打算继承该类；且final类中的方法都是隐式final的

修饰方法：private自带final；final方法可以被重载（即改变参数）但是不能被重写

修饰变量：final型的变量并非都是编译期的常量，但是确定的是一旦它的值被确定之后就不能再修改了（比如给一个final类型的变量赋上随机值）

修饰参数：Java允许在参数列表中以声明的方式将参数指明为final，这意味这你无法在方法中更改参数引用所指向的对象



final关键字的实现都是通过storestore和loadload**内存屏障**来做到的，**原理还是防止重排序**

**写final域的禁止重排序**

编译器会在final域写之后，构造函数return之前，插入一个**storestore**屏障。这个屏障可以禁止处理器把final域的写重排序到**构造函数**之外，就是final域的写只能在构造函数当中完成，最终就是体现为不可变性

在一个线程中，初次读对象引用和初次读该对象包含的final域，JMM会禁止这两个操作的重排序。(注意，这个规则仅仅是针对处理器)，处理器会在读final域操作的前面插入一个LoadLoad屏障

**读final域的禁止重排序**

在一个线程中，初次读对象引用和初次读该对象包含的final域，JMM会禁止这两个操作的重排序。(注意，这个规则仅仅是针对处理器)，处理器会在读final域操作的前面插入一个LoadLoad屏障。也就是对final域的读操作中，必须先读对象的引用，之后才能读该对象中的final域的





#### **volatile**

**作用**

<u>防重排序：</u>一个对象的生成过程是需要遵循特定顺序的，但是构成这些操作的计算机指令会根据硬件来重排序，因而很有可能导致软件层面的顺序被重排了

<u>实现可见性：</u>可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——线程工作内存

<u>不保证原子性：</u>volatile不能保证完全的原子性，只能保证单次的读/写操作具有原子性。两个非常经典的例子 **1）**i++无法保证原子性，因为这是读和写两个操作  **2）**对long和double的操作无法保证原子性，因为它们是由正负位两部分构成的



**原理**

<u>防重排实现：</u>内存屏障，又称**内存栅栏（Memory Barrier）**，是一个 CPU 指令。JMM 为了保证在不同的编译器和 CPU 上有相同的结果，通过插入特定类型的内存屏障来禁止+ 特定类型的编译器重排序和处理器重排序，插入一条内存屏障会告诉编译器和 CPU：不管什么指令都不能和这条 Memory Barrier 指令重排序。

<u>可见性实现：</u>内存一致性，线程的本地内存之间（其物理映射）都会监听**内存总线**，当有内存空间发生了修改操作会在总线上发送对应的消息



**案例：单例模式下用volatile来保证线程安全**

单例模式只在单线程的环境下能保证对象创建的唯一，而在多线程的情况下还需要额外处理

```java
public class DCLSingleton {

  // prevent reordering of instructions
    private static volatile DCLSingleton instance = null;

    public DCLSingleton() {
        System.out.println("Instantiated...");
    }

    public static DCLSingleton getInstance(){
        if (instance == null){

            // double check
            synchronized (DCLSingleton.class){
                if (instance == null){
                    instance = new DCLSingleton();
                }
            }
        }

        return instance;
    }
}

```



volatile的正确使用场景应该是，如果对一个对象的的改操作不在乎它的当前值的话

其次，volatile变量不应该存在于其他变量的等式当中

volatile最适合来做全局唯一的标记为，很好地发挥其可见性的特质，同时也避免了它在原子性上面的缺乏



#### Java多线程相关方法

![image](https://pdai.tech/images/pics/ace830df-9919-48ca-91b5-60b193f593d2.png)

无限期等待Waiting的触发方式是不带参数的wait()方法，或者join()方法；结束这是notify或者notifyAll



**yield()**

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。



**interrupted()**

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。常用在while（！interrupted）当中



**join()**

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。



**wait() / notify() / notifyAll()**

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

**只能用在同步方法或者同步控制块中配合使用，**否则会在运行时抛出 IllegalMonitorStateExeception。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。



**await() / signal() / signalAll()**

java.util.concurrent 类库中提供了 **Condition** 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。这一套方式可以在Lock加锁的情况下使用，相比同步代码块也会更加灵活一些。

```java
public class AwaitSignalExample {
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```













#### Condition

可以根据条件来控制由哪个线程来执行，具体的使用查看上面的await、signal等内容



#### **Semaphore**

设定临界资源能够同时被多少线程访问

最后的release( )正常来讲也需要放在finally里面，任何用锁的都要这么来做，除了JVM提供的synchronized关键字





#### CompletableFuture

进行线程编排，控制一系列线程的执行顺序并加以利用

和Future相比它存在一个非常重要的优化，那就是采用了完全的non-blocking模式。Future在等待IO的时候会阻塞当前线程，但是CompletableFuture不会，采用Reactive的方式来进行

CompletableFuture同时实现了两个接口，分别为Future和CompletionStage，CompletionStage是CompletableFuture提供的一些非常丰富的接口，可以借助这些接口来实现非常复杂的异步计算工作

在CompletableFuture中，如果没有显示指定的Executor的参数，则会**调用默认的ForkJoinPool.commonPool( )**

```java
    public T get() throws InterruptedException, ExecutionException {
        Object r;
        return reportGet((r = result) == null ? waitingGet(true) : r);
    }
```



##### **Threads Choreography 线程编排**

每个CF任务会生成一个Completion对象，这些对象一起通过链表的方式进行关联管理

后续的Completion结点返回的CompletableFuture, 将拥有的stack里面的所有结点都压入了当前CompletableFuture的stack里面，重新构成了一个链表结构，后续也按照前面的逻辑操作，如此反复，便会遍历完所有的CompletableFuture, 这些CompletableFuture(叶子结点)的stack为空，也是结束条件。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230313100055225.png" alt="image-20230313100055225" style="zoom:33%;" />

```java
// the field in CF —— stack 
volatile Completion stack;    // Top of Treiber stack of dependent actions


// Completion Class
abstract static class Completion extends ForkJoinTask<Void> implements Runnable, AsynchronousCompletionTask {
        volatile Completion next; // 无锁并发栈

        /**
         * 钩子方法，有三种模式，postComplete()方法里面使用的是NESTED模式，避免过深的递归调用 SYNC, ASYNC, or NESTED
         */
        abstract CompletableFuture<?> tryFire(int mode); // run()和exec()都调用了这个钩子方法

        /** cleanStack()方法里有用到 */
        abstract boolean isLive();

        public final void run() {
            tryFire(ASYNC);
        }

        public final boolean exec() {
            tryFire(ASYNC);
            return true;
        }

        public final Void getRawResult() {
            return null;
        }

        public final void setRawResult(Void v) {
        }
    }
```



##### The APIs 





