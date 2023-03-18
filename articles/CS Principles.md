crash面试中的计算机基础知识



## 计算机网络

#### TCP与UDP的区别



TCP有哪些保证可靠传输的手段

TCP粘包现象

TCP的三次握手及其标记位

TCP的四次挥手系列及其11种状态

CLOSE_WAIT状态多久出现，出现原因是什么 

TIME_WAIT状态的详细分析，何时出现，何处出现，为何出现，是否有弊端，如何避免

ping的底层 

 linux查看网络状态的命令 

TCP协议中超时重传能保证数据的可靠传输，但是丢包之后一定会有延迟，有没有什么办法来降低这个延迟？

TCP为什么是三次握手？



#### 访问百度域名过程中发生了什么，涉及到了什么协议

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220807172903412.png" alt="image-20220807172903412" style="zoom: 25%;" />



内网和外网的区别？ 

讲一下NAT



讲一下epoll,select,poll三者的区别和适用的场景



#### HTTP SERIES

HTTP1.1

##### HTTPS

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220807213537259.png" alt="image-20220807213537259" style="zoom: 25%;" />

TLS1.2 TLS1.3 RSA Diffie-Hellman



HTTP2.0

HTTP3.0

HTTP请求头的一些参数

HTTP的状态码（响应码）





#### RPC

Remote Procedure Call，它类似于一种应用层协议，对标http，但是实际上更像是一个调用方式。在本地调用远程服务器的方法像调用本地方法一样便捷，屏蔽掉中间存在的网络相关的细节。

其实HTTP和RPC都能做到这种远过程调用，之所以在还搞一个RPC而不直接使用HTTP的原因如下：

1. HTTP协议的头部和Body部分存在较多的冗余信息
2. HTTP因为冗余信息多，在序列化的时候采用JSON序列化，而效率更高开销更小的Protobuf则不可以
3. HTTP需要考虑服务器和浏览器之间的各种情况，例如30X和40X
4. HTTP无法保持长连接

RPC更多是在HTTP的基础上做减法，定制化的程度更高

当然上面说的 HTTP，其实 特指的是现在主流使用的 HTTP1.1，HTTP2在前者的基础上做了很多改进，所以 性能可能比很多 RPC 协议还要好，甚至连gRPC底层都直接用的HTTP2



TCP/IP五层协议与七层协议





#### IP地址的计算问题

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222112343668.png" alt="image-20230222112343668" style="zoom: 33%;" />

![image-20230222112528905](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222112528905.png)

广播地址为主机号全1，网络地址主机号全0，子网划分最少要保证留出一个主机位出来，也就是最后的数量-3



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230222114941378.png" alt="image-20230222114941378" style="zoom: 33%;" />



## 操作系统

#### 我们知道操作系统定义了进程和线程，那么区别出进程和线程的目的是什么？ 



进程的通信方式

进程的调度，包括其用到的数据结构

线程和协程的区别



#### 讲一下epoll,select,poll三者的区别和适用的场景

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220413221335475.png" alt="image-20220413221335475" style="zoom:33%;" />

select/poll/epoll 都是 I/O 多路复用的具体实现，也就是NIO中具体处理者（单线程），需要多个任务来复用这个线程处理任务。

这里重点关注Reactor模型的**epoll**实现（同时epoll也是最新的），大多数业务都跑在Linux的服务器上，同时epoll的性能相对最好且对Java的支持性也高。

Reactor模型的核心思想就是**业务处理与IO分离**。可以理解为一个主线程负责监听端口上的请求然后线程池中的其他线程负责具体IO和具体业务的处理，这就使得网络IO和业务处理能够同时进行不发生阻塞。具体到Reactor模型的话，mainReactor负责监听网络端口，让subReactor负责协同线程池和Acceptor（专门进行客户端连接的获取）进行任务的处理（交由Reactor中的Handler组建进行）

**下图是多线程的Reactor模型（回顾一下）**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220414001520121.png" alt="image-20220414001520121" style="zoom: 25%;" />

Reactor模型的实现分为单线程和多线程两种，在单线程情况下任务执行速度并没有得到提高，但是由于对工作职责的精细划分，使得整体效率得到提高。

1. Reactor模型是以事件进行驱动的，其能够将接收客户端连接、网络读和网络写、以及业务计算进行拆分，从而极大的提升处理效率

2. Reactor模型是异步非阻塞模型，工作线程在没有网络事件时可以处理其他的任务，而不用像传统IO那样必须阻塞等待

##### **epoll**

##### poll

##### select







讲一下Linux的文件系统



怎么查看进程切换？如何统计进程上下文切换的次数？

讲一下虚拟内存到物理内存的映射，TLB，MMU

讲一下 page fault， major page fault， minor page fault

内存什么时候会出现一个锯齿状的波动，举一例子，为什么会造成这种情况 





## 计算机硬件（组成原理）



