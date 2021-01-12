---
title: 01-ActiveMQ简介及JMS
date: 2020-07-14 23:27:39
categories:
- ActiveMQ
tags:
- ActiveMQ
- JMS
---

<center><font size=4 color="red">ActiveMQ简介及JMS</font></center>

<!--more-->

## ActiveMQ简介及JMS

#### 什么是ActiveMQ

[activeMQ官网](http://activemq.apache.org/)

#### ActiveMQ的应用场景

ActiveMQ可以用于做：异步处理、应用解耦、流量削锋

#### 什么是JMS

JMS是java平台上有关面向消息中间件的技术规范，它便于消息系统中的java应用程序进行信息交换，并且通过提供标准的产生、发送、接受消息的接口简化企业应用的开发。

##### JMS消息模型

消息中间件一般有两种传递模式：点对点模式（P2P）和发布-订阅模式（Pub、Sub）

1. P2P点对点模式：Queue队列模式
2. Publish/Subscribe发布/订阅模式：Topic主题模式

**点对点模式**

生产者和消费者之间的消息往来

> clientA是发布者，clientB是消费者，一个消息只有一个消费者

![点对点模式](点对点模式.png)

每个消息都被发送到特定的消息队列，接受者从队列中获取消息。队列保留着消息，知道他们被消费或超时

特点：

* 每个消息只有一个消费者（consumer），即一旦被消费，消息就不再在队列中了
* 发送者和接收者之间在时间上没有依赖性，也就是说当发送者发送了消息之后，不管接收者有没有正在运行，它不会影响到消息被发送到队列。
* 接收者在成功接收消息之后需向队列应答成功

**发布/订阅模式**

包含三个角色：主题（Topic），发布者（Publisher)，订阅者（Subscriber），多个发布者将消息发送到topic，系统将这些消息投递到订阅此topic的订阅者

> clientA是发送者，clientB和ClientC是订阅者，一个消息可以有多个订阅者

![发布-订阅模式](发布-订阅模式.png)

发送者发送到topic的消息，只有订阅了topic的订阅者才会收到消息。topic实现了发布和订阅，当你发布一个消息，所有订阅这个topic的服务都能得到这个消息。，所以从1到N个订阅者都能得到这个消息的拷贝。

特点：

* 每个消息可以有多个消费者
* 发布者和订阅者之间有时间上的依赖性，要先订阅主题，再来发送消息（不能先发送，再订阅）
* 订阅者必须保持运行的转态，才能接受发布者发布的消息

##### JMS编程API

| 要素              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| Destination       | 表示消息所有通道的目标定义，用来定义消息从发送端发出后要走的通道，而不是接收方。Destination属于管理类对象 |
| ConnectionFactory | 顾名思义，用于创建连接对象。ConnectionFactory属于管理类对象  |
| Connection        | 连接接口，所负责的重要工作是创建Session                      |
| Session           | 会话接口，这是一个非常重要的对象，消息发送者、消息接收者以及消息对象本身，都是通过这个会话对象创建的 |
| MessageConsumer   | 消息的消费者，也就是订阅消息并处理消息的对象                 |
| MessageProducer   | 消息的生产者，也就是用来发送消息的对象                       |

* Destination：Destination的意思是消息生产者的消息发送目标或者说消息消费者的消息来源，对于消息生产者来说，它的Destination是某个队列（Queue）或某个主题（Topic）；对于消费者来说，它的Destination也是某个队列或主题（即消息来源）。所以，Destination实际上就是两种类型的对象：Queue、Topic
* ConnectionFactory：创建Connection对象的工厂，针对两种不同jms消息模型，分别有QueueConnectionFactory和TopicConnectionFactory两种
* Connection：表示在客户端和JMS系统之间建立的连接（对TCP/IP socket的包装），Connection可以产生一个或多个Session
* Session：Session是我们对消息进行操作的接口，可以通过session创建生产者、消费者、消息等。Session提供了事务的功能，如果需要使用session发送/接收多个消息时，可以将这些发送/接收动作放到一个事务中
* Producer（消息生产者）：消息生产者由Session创建，并用于将消息发送到Destination。同样，消息生产者分两种类型：QueueSender和TopicPublisher。可以调用消息生产者的方法（send和publish方法）发送消息
* Consumer（消息消费者）：消息消费者由Session创建，用于接收被发送到Destination的消息。两种类型：QueueReceiver和TopicSubscriber。可分别通过session的createReceiver（Queue）或createSubscriber（Topic）来创建。当然，也可以通过session的createDurableSubscriber方法来创建持久化的订阅者
* MessageListener：消息监听器。如果注册了消息监听器，一旦消息送到，将自动调用监听器的onMessage方法。EJB中的MDB（Message-Driven Bean）就是一种MessageListener

**整个API的图解**

![JMS的API图解](JMS的API图解.png)

## ActiveMQ的安装

#### 安装

activeMQ的[下载地址](http://activemq.apache.org/download-archives)

```powershell
# 第一步，安装jdk

# 第二步，把activeMQ的压缩包（apache-Activemq-5.14.5-bin.tar.gz）上传到linux系统

# 第三步 解压压缩包
$ tar -zxvf apache-Activemq-5.14.5-bin.tar.gz

# 第四步 进入apache-Activemq-5.14.5的bin目录
$ cd apache-Activemq-5.14.5/bin

# 第五步 启动activemq
./activemq start

# 第六步 停止activemq
./activemq stop
```

##### 访问

```
页面控制台：http://192.168.25.180:8161/   (监控)
请求地址：tcp:ip:61616   （java代码访问消息中间件）
```

账号密码都是：admin

![activemq](activemq.png)

