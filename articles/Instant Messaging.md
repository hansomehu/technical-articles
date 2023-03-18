---
layout: post
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

底层基于TCP/IP，同时也适用于P2P大数据通信

Netty的层次关系

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220531143158786.png" alt="image-20220531143158786" style="zoom:45%;" />

#### Netty应用场景

- 基础通信组件，被RPC框架使用，例如Dubbo（默认使用Netty实现进程节点间的内部通信）

- 游戏行业，用来作为高性能通信组件；提供了TCP/IP/HTTP协议栈
- 地图服务器高性能通信
- 大数据领域中Hadoop的Avro通信框架使用Netty实现

> 参考书：《Netty in Action》（实战）、《Netty权威指南》（原理）
>
> 学习Netty要积极复习Java IO和BIO、NIO、AIO（异步非阻塞）知识！！！



#### Netty线程模型

**单线程**

一个Reactor进行接待和IO处理

**多线程**

一个Reactor进行接待，后面由一个Reactor线程池进行IO处理

**主从线程池**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220531143224135.png" alt="image-20220531143224135" style="zoom:45%;" />

Reactor主线程池负责连接处理、安全验证、请求转发；而从线程池负责具体的IO处理



#### Netty核心组件

**Channel**

Channel是Netty传输API的核心，被用于所有的I/O操作，Channel 接口所提供的API大大降低了Java中直接使用Socket类的复杂性

**回调**

Netty在内部使用了回调来处理事件，当一个回调被触发时，相关的事件可以交由一个ChannelHandler的实现处理

**Future**

Future提供了一种在操作完成时通知应用程序的方式，可以看作是一个异步操作的结果的占位符，它将在未来的某个时刻完成，并提供对其结果的访问。Netty提供了自己的实现——ChannelFuture，由ChannelFutureListener提供的通知机制消除了手动检查对应操作是否完成的步骤。

**事件和ChannelHandler**

Netty使用不同的事件来通知我们状态的改变，这使得我们能够基于已经发生的事件来触发适当的动作。每个事件都可以被分发给ChannelHandler类，ChannelHandler类中提供了自定义的业务逻辑，架构上有助于保持业务逻辑与网络处理代码的分离



#### WS + Netty 作为即时消息传递的解决方案

![image-20220531143246900](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220531143246900.png)





前端方面是在页面的created的生命周期方法中就向Netty请求建立WS连接，然后调用WS的onXXXX方法进行业务操作。服务端中也是同步监听onXXXX方法，并设置对应的Handler。onOpen onMessage onClose

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220531143254652.png" alt="image-20220531143254652" style="zoom:33%;" />







## MQTT协议





















## 消息存储/同步架构







## 推送业务架构





## 单聊/群聊业务架构

这类业务的基本架构都是这样的，服务端存在一个**转发中心、存储中心和业务处理中心**来应对来自客户端的请求

