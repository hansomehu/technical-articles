---
9layout: post
title: "im快速入门"
permalink: /im-crash
---





## WebSocket

WS是一种浏览器与服务器进行全双工通讯的网络技术，属于应用层协议。它基于TCP传输协议，并复用HTTP的握手通道

WS和Socket没有关系，它是从HTTP协议升级而来，先建立HTTP然后发送一个upgrade请求，再经历一次握手，成功后即建立连接

双通道、长连接、实时性强

<img src="http://www.52im.net/data/attachment/forum/201605/25/172321pg747a7ru4lghzwu.png" alt="WebSocket详解（三）：深入WebSocket通信协议细节_2.png" style="zoom: 67%;" />



#### 掩码机制 Masking

WebSocket协议中，数据掩码的作用是增强协议的安全性。但数据掩码并不是为了保护数据本身，因为算法本身是公开的，运算也不复杂。除了加密通道本身，似乎没有太多有效的保护通信安全的办法。

那么为什么还要引入掩码计算呢，除了增加计算机器的运算量外似乎并没有太多的收益（这也是不少同学疑惑的点）。

答案还是两个字：安全，但并不是为了防止数据泄密，而是为了防止早期版本的协议中存在的**代理缓存污染攻击**（proxy cache poisoning attacks）等问题。

但真正意义上的安全还需要基于WS进行安全化的改造，比如加上类似于HTTPS的SSL安全机制，构建一个**安全的连接环境+数据加密方式**。





#### Sec-WebSocket-Key/Accept 的作用

Sec-WebSocket-Key/Sec-WebSocket-Accept在主要作用在于提供基础的防护，减少恶意连接、意外连接。因为它是基于HTTP的，会存在服务端意外应答了WS相关的连接请求而造成了端口资源的浪费，因而通过在headers中加入这些字段来做一个保证

- 1）避免服务端收到非法的websocket连接（比如http客户端不小心请求连接websocket服务，此时服务端可以直接拒绝连接）；
- 2）确保服务端理解websocket连接。因为ws握手阶段采用的是http协议，因此可能ws连接是被一个http服务器处理并返回的，此时客户端可以通过Sec-WebSocket-Key来确保服务端认识ws协议。（并非百分百保险，比如总是存在那么些无聊的http服务器，光处理Sec-WebSocket-Key，但并没有实现ws协议。。。）；
- 3）用浏览器里发起ajax请求，设置header时，Sec-WebSocket-Key以及其他相关的header是被禁止的。这样可以避免客户端发送ajax请求时，意外请求协议升级（websocket upgrade）；
- 4）可以防止反向代理（不理解ws协议）返回错误的数据。比如反向代理前后收到两次ws连接的升级请求，反向代理把第一次请求的返回给cache住，然后第二次请求到来时直接把cache住的请求给返回（无意义的返回）；
- 5）Sec-WebSocket-Key主要目的并不是确保数据的安全性，因为Sec-WebSocket-Key、Sec-WebSocket-Accept的转换计算公式是公开的，而且非常简单，最主要的作用是预防一些常见的意外情况（非故意的）。



#### 跨站点 WebSocket 劫持漏洞

大家仔细观察上文的握手 Get 请求，可以看到 Cookie 头部把域名下的 Cookie 都发送到服务器端。如果有机会阅读 WebSocket 协议（10.5 章客户端身份认证）就发现，WebSocket 协议没有规定服务器在握手阶段应该如何认证客户端身份。服务器可以采用任何 HTTP 服务器的客户端身份认证机制，譬如 cookie，HTTP 基础认证，TLS 身份认证等。因此，对于绝大多数 Web 应用来说，客户端身份认证应该都是 SessionID 等 Cookie 或者 HTTP Auth 头部参数等。熟悉跨站点请求伪造攻击 Cross Site Request Forgery（CSRF）的朋友到这里应该就可以联想到黑客可能伪造握手请求来绕过身份认证。因为 WebSocket 的客户端不仅仅局限于浏览器，因此 WebSocket 规范没有规范 Origin 必须相同，这就为漏洞埋下了伏笔。

解决方案：

在服务器端的代码中增加 Origin 检查，如果客户端发来的 Origin 信息来自不同域，建议服务器端拒绝这个请求，发回 403 错误响应拒绝连接。

或者在消息体中加入立牌等验证机制，但这种方案的开销会略大。









## Netty

Netty is an synchoronous event-deriven newtwork application framework  [官网](https://netty.io/index.html)

Netty 是一个 Java 开源框架。Netty 提供 ***并发、异步、事件驱动*** 的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序。也就是说，Netty 是一个基于 NIO 的客户、服务器端编程框架，使用Netty 可以确保你快速和简单的开发出一个网络应用，例如实现了某种协议的客户，服务端应用。Netty 相当简化和流线化了网络应用的编程开发过程，例如，TCP 和 UDP 的 Socket 服务开发。







底层基于TCP/IP，同时也适用于P2P大数据通信

Netty的层次关系

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220531143158786.png" alt="image-20220531143158786" style="zoom:45%;" />

#### Netty高性能之道

异步非阻塞

零拷贝机制，直接从网卡的buffer中拷贝到Channel中去，调用native方法allocator

内存池，使用堆外内存，避免了ByteBuf的频繁回收

高效的Reactor模型

无锁化的串行设计，采用多线程来合理利用IO空闲时间

大量使用高效的并发编程以及线程安全容器，CAS、volatile

高效的序列化框架





#### Netty应用场景

- 基础通信组件，被RPC框架使用，例如Dubbo（默认使用Netty实现进程节点间的内部通信）

- 游戏行业，用来作为高性能通信组件；提供了TCP/IP/HTTP协议栈
- 地图服务器高性能通信
- 大数据领域中Hadoop的Avro通信框架使用Netty实现

> 参考书：《Netty in Action》（实战）、《Netty权威指南》（原理）
>
> 学习Netty要积极复习Java IO和BIO、NIO、AIO（异步非阻塞）知识！！！



#### Netty工作流程

![image-20230320001329983](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230320001329983.png)



#### Netty的设计结构

![image-20230320200002510](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230320200002510.png)



### Netty核心组件

**Event**

Netty在内部使用了回调来处理事件，当一个回调被触发时，相关的事件可以交由一个ChannelHandler的实现处理，它的本质就是一次网络请求。



**Channel**

Channel是Netty传输API的核心，被用于所有的I/O操作，Channel 接口所提供的API大大降低了Java中直接使用Socket类的复杂性。Channel对象维护一个连接的信息，每次网络调用立即返回一个 ChannelFuture 实例，通过注册监听器到 ChannelFuture 上，可以 I/O 操作成功、失败或取消时回调通知调用方。具体的Channel有NioSocketChannel、NioServerSocketChannel。



**ChannelFuture**

Future提供了一种在操作完成时通知应用程序的方式，可以看作是一个异步操作的结果的占位符，它将在未来的某个时刻完成，并提供对其结果的访问。Netty提供了自己的实现——ChannelFuture，由ChannelFutureListener提供的通知机制消除了手动检查对应操作是否完成的步骤。



**ChannelHandler**

ChannelHandler 是一个接口，处理 I/O 事件或拦截 I/O 操作，并将其转发到其 ChannelPipeline（业务处理链）中的下一个处理程序。ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类：ChannelInboundHandler 用于处理入站 I/O 事件、ChannelOutboundHandler 用于处理出站 I/O 操作



##### ChannelPipeline

保存 ChannelHandler 的 List，用于处理或拦截 Channel 的入站事件和出站操作。ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 Channel 中各个的 ChannelHandler 如何相互交互。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230320195025350.png" alt="image-20230320195025350" style="zoom:33%;" />

在具体的数据结构方面，采用了链表的机制来构建这个处理链条

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230320195302384.png" alt="image-20230320195302384" style="zoom: 25%;" />







### Reactor模型

Event-Driven模型

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230320195655170.png" alt="image-20230320195655170" style="zoom:25%;" />

Reactor的事件驱动

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230320195747840.png" alt="image-20230320195747840" style="zoom:25%;" />





#### Reactor的线程模型

**单线程**

一个Reactor进行接待和IO处理

**多线程**

一个Reactor进行接待，后面由一个Reactor线程池进行IO处理

**主从线程池**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220531143224135.png" alt="image-20220531143224135" style="zoom:45%;" />

Reactor主线程池负责连接处理、安全验证、请求转发；而从线程池负责具体的IO处理



### 























#### WS + Netty 作为即时消息传递的解决方案

![image-20220531143246900](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220531143246900.png)



前端方面是在页面的created的生命周期方法中就向Netty请求建立WS连接，然后调用WS的onXXXX方法进行业务操作。服务端中也是同步监听onXXXX方法，并设置对应的Handler。onOpen onMessage onClose

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220531143254652.png" alt="image-20220531143254652" style="zoom:33%;" />







## MQTT协议





















## 消息存储/同步架构

#### 双库设计

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230318233158119.png" alt="image-20230318233158119" style="zoom:33%;" />

当前的通用设计都是双库，当服务端拿到消息之后首先存在消息储存库中，之后再放入一个长度有限的消息同步队列中去（这个长度比如微信的99+）。这个同步库就好比一个滑动窗口，里面的消息都存储一个特定的时间和数量。



#### Timeline虚拟数据结构

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230318233546079.png" alt="image-20230318233546079" style="zoom:33%;" />

以两个场景来举例子：

- 多端同步场景，这时候的同步index根据最大的已同步index来计算。也就是如果APP端同步到了第100条数据，PAD只同步到了80条，但是从全局来看也还是按照第100条为准
- 群聊场景，这个时候需要统计每个接收方各自的index，根据这个index来统计已读未读人数总和



#### 单聊存储

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230318234218970.png" alt="image-20230318234218970" style="zoom:33%;" />

如图例子所示，A与B/C/D/E/F均发生了会话，每个会话对应一个独立的Timeline





<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230318234734578.png" alt="image-20230318234734578" style="zoom: 33%;" />

IM场景下，一条消息只会产生一次，但是会被读取多次，是典型的读多写少的场景，消息的读写比例大概是10:1。若使用读扩散同步模式，整个系统的读写比例会被放大到100:1。

一个优化的好的系统，必须从设计上去平衡这种读写压力，避免读或写任意一维触碰到天花板。所以IM系统这类场景下，通常会应用写扩散这种同步模式，来平衡读和写，将100:1的读写比例平衡到30:30。

当然写扩散这种同步模式，还需要处理一些极端场景，例如万人大群。针对这种极端写扩散的场景，会退化到使用读扩散。一个简单的IM系统，通常会在产品层面限制这种大群的存在，而对于一个高级的IM系统，会采用读写扩散混合的同步模式，来满足这类产品的需求。采用混合模式，会根据数据的不同类型和不同的读写负载，来决定用写扩散还是读扩散。





## 推送业务架构





## 单聊/群聊业务架构

这类业务的基本架构都是这样的，服务端存在一个**转发中心、存储中心和业务处理中心**来应对来自客户端的请求

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230319000440484.png" alt="image-20230319000440484" style="zoom:33%;" />



## 美团 Pike

![图4 Pike 2.0产品全景图](https://p1.meituan.net/travelcube/28d21d62765640c18ddf2145ac7e38eb791076.png)

