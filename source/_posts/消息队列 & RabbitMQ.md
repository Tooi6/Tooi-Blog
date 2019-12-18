---
title: 消息队列 & RabbitMQ    
date: 2019-12-18 23:00:36  
tags:  
- 消息队列
- RabbitMQ
---
### 简介  
#### 什么是MQ？  
> Message Queue（MQ），消息队列中间件。消息队列就是在消息传输过程中保存消息的容器。消息队列是分布式系统中重要的组件，**使用消息队列主要是为了通过异步处理提高系统性能和削峰、降低系统耦合性。**  

#### 为什么使用MQ？  
- **提高系统响应速度**  
> 使用了消息队列，生产者一方，把消息往队列里一扔，就可以立马返回，响应用户了。无需等待处理结果
- **提高系统稳定性**
> 考虑电商系统下订单，发送数据给生产系统的情况。电商系统和生产系统之间的网络有可能掉线，生产系统可能会因维护等原因暂停服务。如果不使用消息队列，电商系统数据发布出去，顾客无法下单，影响业务开展。两个系统间不应该如此紧密耦合。应该通过消息队列解耦。同时让系统更健壮、稳定。  
- **降低系统耦合性**
![image](https://note.youdao.com/yws/api/personal/file/A7176E3B46CC43D88753827F3A2F30D0?method=download&shareKey=726b280fbca9438a4ff3fcc180705236)
> 消息队列使利用发布-订阅模式工作，消息发送者（生产者）发布消息，一个或多个消息接受者（消费者）订阅消息。 **可以看到消息发送者（生产者）和消息接受者（消费者）之间没有直接耦合。**

#### 使用MQ带来的问题
- **系统可以性降低**   
> 需要考虑消息队列挂掉的情况。  
- **系统复杂性提高**
> 需要保证消息没有被重复消费、处理消息丢失的情况、保证消息传递的顺序性等等问题！  
- **一致性问题**
> 消息队列带来的异步确实可以提高系统响应速度。但是，万一消息的真正消费者并没有正确消费消息怎么办？这样就会导致数据不一致的情况了!

#### MQ框架 
> MQ框架非常之多，比较流行的有**RabbitMq、ActiveMq、ZeroMq、kafka**，以及阿里开源的**RocketMQ**  

#### RabbitMQ的概念 
- **生成者和消费者**  
> Producer：消息的生产者  
Consumer：消息的消费者  

- **Queue**  
> **消息队列**，提供了 **FIFO** 的处理机制，具有缓存消息的能力。RabbitMQ中，队列消息可以设置为持久化，临时或者自动删除。  
设置为**持久化的队列**，Queue中的消息会在Server本地硬盘存储一份，防止系统Crash，数据丢失  
设置为**临时队列**，Queue 中的数据在系统重启之后就会丢失  
设置为**自动删除的队列**，当不存在用户连接到 Server，队列中的数据会被自动删除  

- **ExChange**  
> Exchange 类似于数据通信网络中的**交换机**，提供消息路由策略。RabbitMQ 中，Producer 不是通过信道直接将消息发送给 Queue，而是先发送给 ExChange。一个 ExChange 可以和多个 Queue 进行绑定，Producer 在传递消息的时候，会传递一个 **ROUTING_KEY**，ExChange 会根据这个 ROUTING_KEY 按照特定的路由算法，将消息路由给指定的 Queue。和 Queue 一样，ExChange 也可设置为持久化，临时或者自动删除  

- **ExChange 的 4 种类型**   
>**direct（默认）**：直接交换器，工作方式类似于单播，ExChange 会将消息发送完全匹配 ROUTING_KEY 的 Queue（key 就等于 queue）  
**fanout**：广播是式交换器，不管消息的 ROUTING_KEY 设置为什么，ExChange 都会将消息转发给所有绑定的 Queue（无视 key，给所有的 queue 都来一份）  
**topic**：主题交换器，工作方式类似于组播，ExChange 会将消息转发和 ROUTING_KEY 匹配模式相同的所有队列（key 可以用“宽字符”模糊匹配 queue），比如，ROUTING_KEY 为 user.stock 的 Message 会转发给绑定匹配模式为 * .stock,user.stock， * . * 和 #.user.stock.# 的队列。（ * 表是匹配一个任意词组，# 表示匹配 0 个或多个词组）  
**headers**：消息体的 header 匹配，无视 key，通过查看消息的头部元数据来决定发给那个 queue（AMQP 头部元数据非常丰富而且可以自定义）  

- **Binding**  
> 所谓绑定就是将一个特定的 ExChange 和一个特定的 Queue 绑定起来。ExChange 和 Queue 的绑定可以是多对多的关系

- **Virtual Host**  
> **在 RabbitMQ Server 上可以创建多个虚拟的 Message Broker**，又叫做 Virtual Hosts (vhosts)。每一个 vhost 本质上是一个 mini-rabbitmq server，分别管理各自的 ExChange，和 bindings。vhost 相当于物理的 Server，可以为不同 app 提供边界隔离，使得应用安全的运行在不同的 vhost 实例上，相互之间不会干扰。Producer 和 Consumer 连接 rabbit server 需要指定一个 vhost  

#### RabbitMQ 使用步骤  
- 客户端连接到消息队列服务器，打开一个 Channel。  
- 客户端声明一个 ExChange，并设置相关属性。  
- 客户端声明一个 Queue，并设置相关属性。  
- 客户端使用 Routing Key，在 ExChange 和 Queue 之间建立好绑定关系。  
- 客户端投递消息到 ExChange。  
- ExChange 接收到消息后，就根据消息的key和已经设置的binding，进行消息路由，将消息投递到一个或多个队列里   


### 安装RabbitMQ

#### 使用 Docker 安装  

```
## docker-compose.yml 文件
version: '3.1'
services:
  rabbitmq:
    restart: always
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - 5672:5672
      - 15672:15672
    environment:
      TZ: Asia/Shanghai
      RABBITMQ_DEFAULT_USER: rabbit
      RABBITMQ_DEFAULT_PASS: 123456
    volumes:
      - ./data:/var/lib/rabbitmq
```

### DEMO
> 下面是一个Springboot整合RabbitMQ的demo

- **pom.xml**  

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit-test</artifactId>
    <scope>test</scope>
</dependency>
```

- **application.yml**
> 生成者和消费者配置一样

```
spring:
  rabbitmq:
    host: 192.168.213.133
    port: 5672
    username: rabbit
    password: 123456
```

- **创建队列配置**

```
@Configuration
public class RabbitMQConfiguration {
    @Bean
    public Queue queue(){
        return new Queue("helloRabbit");
    }
}
```

- **创建消息提供者**

```
@Component
public class HelloRabbitProvider {
    @Autowired
    private AmqpTemplate amqpTemplate;

    public void send() {
        String context = "hello" + new Date();
        System.out.println("Provider:" + context);
        amqpTemplate.convertAndSend("helloRabbit", context);
    }
}
```

- **创建测试用例**

```
@SpringBootTest(classes = DemoRabbitmqApplication.class)
@RunWith(SpringRunner.class)
class DemoRabbitmqApplicationTests {

    @Autowired
    private HelloRabbitProvider helloRabbitProvider;

    @Test
    void testSender() {
        for (int i=0;i<10;i++){
            helloRabbitProvider.send();
        }
    }

}
```

- **运行，查看 Queen**
![image](https://note.youdao.com/yws/api/personal/file/B0660A2DF01F429EA41593279160CDA6?method=download&shareKey=40cb944866adefe7f634afeb133780ce)  

- **创建消息消费者**

```
@Component
@RabbitListener(queues = "helloRabbit")
public class HelloRabbitConsumer {

    @RabbitHandler
    public void process(String message){
        System.out.println(message);
    }
}
```

- **启动消费者SpringBoot项目即可收到消息**  
