---
title: 异步消息协议 JMS 、AMQP    
date: 2019-12-18 23:36:03  
tags:  
- JMS
- AMQP
---

### JMS

#### 简介
> 　**JMS（JAVA Message Service,java消息服务）** 是java的消息服务，JMS的客户端之间可以通过JMS服务进行异步的消息传输。JMS（JAVA Message Service，Java消息服务）API是一个消息服务的**标准**或者说是**规范**，允许应用程序组件基于JavaEE平台创建、发送、接收和读取消息。**它使分布式通信耦合度更低，消息服务更加可靠以及异步性。**

#### JMS两种消息模型  
- 点到点（P2P）模型  
> 使用**队列（Queue）作为消息通信载体；满足生产者与消费者模式**，一条消息只能被一个消费者使用，未被消费的消息在队列中保留直到被消费或超时。  
![image](https://note.youdao.com/yws/api/personal/file/83BCC612E61742519B866EFF1DE156DE?method=download&shareKey=0fb6ae11af28be870b37f1dafd774bd8)

- **发布/订阅（Pub/Sub）模型**  
> 发布订阅模型（Pub/Sub） 使用**主题（Topic）作为消息通信载体，类似于广播模式**；发布者发布一条消息，该消息通过主题传递给所有的订阅者，**在一条消息广播之后才订阅的用户则是收不到该条消息的**。    
![image](https://note.youdao.com/yws/api/personal/file/0D4246AE663145E2A48B6DFFD08B5C36?method=download&shareKey=fc3527441f3c73f2ea633b0a3bab4ed8)  

#### JSM的消息正文格式  
- **StreamMessage** ：  Java原始值的数据流  
- **MapMessage** ： 一套名称-值对  
- **TextMessage** ： 一个字符串对象  
- **ObjectMessage** ： 一个序列化的 Java对象  
- **BytesMessage** ： 一个字节的数据流  

### AMQP  
> **AMQP，即Advanced Message Queuing Protocol**，一个提供统一消息服务的应用层标准 **高级消息队列协议**（二进制应用层协议），是应用层协议的一个开放标准,为面向消息的中间件设计，兼容 JMS。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件同产品，不同的开发语言等条件的限制。

### JMS 对比 AMQP  
| 比方向    | JMS                              | AMQP                                                                                                                                        |
|--------|----------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| 定义     | Java API                         | 协议                                                                                                                                          |
| 跨语言    | 否                                | 是                                                                                                                                           |
| 跨平台    | 否                                | 是                                                                                                                                           |
| 支持消息类型 | 提供两种消息模型：①Peer\-2\-Peer;②Pub/sub | 提供了五种消息模型：①direct exchange；②fanout exchange；③topic change；④headers exchange；⑤system exchange。本质来讲，后四种和JMS的pub/sub模型没有太大差别，仅是在路由机制上做了更详细的划分； |
| 支持消息类型 | 支持多种消息类型 ，我们在上面提到过               | byte\[\]（二进制）                                                                                                                               |

#### 总结
- AMQP 为消息定义了**线路层（wire-level protocol）的协议**，而JMS所定义的是**API规范**。在 Java 体系中，多个client均可以通过JMS进行交互，不需要应用修改代码，但是其对跨平台的支持较差。而AMQP天然具有跨平台、跨语言特性。    
- JMS 支持**TextMessage、MapMessage** 等复杂的消息类型；而 AMQP 仅支持 **byte[] 消息类型**（复杂的类型可序列化后发送）。  
- 由于Exchange 提供的路由算法，AMQP可以提**供多样化的路由方式**来传递消息到消息队列，而 JMS 仅支持 队列 和 主题/订阅 方式两种。  

