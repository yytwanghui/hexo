---
title: 02-Java操作ActiveMQ
date: 2020-07-14 23:35:30
categories:
- ActiveMQ
- JMS API
- Spring
tags:
---

<center><font size=4 color="red">Java操作ActiveMQ</font></center>

<!--more-->



## 原生JMS API操作ActiveMQ

#### PTP模式

##### 引入坐标

```xml
<dependency>
  <groupId>org.apache.activemq</groupId>
  <artifactId>activemq-all</artifactId>
  <version>5.14.5</version>
</dependency>
```

##### 代码实现

**发送消息**

```
1.创建连接工厂
2.创建连接
3.打开连接
4.创建session
5.创建目标地址（Queue：点对点消息，Topic：发布订阅信息）
6.创建消息生产者
7.创建消息
8.发送消息
9.释放资源
```

com.hui.producer.PTP_Producer

```java
package com.hui.producer;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 演示点对点模式---消息生产者
 */
public class PTP_Producer {

    public static void main(String[] args) throws JMSException {

        //1.创建连接工厂
        ConnectionFactory factory=new ActiveMQConnectionFactory("tcp://192.168.25.181:61616");

        //2.创建连接
        Connection connection = factory.createConnection();

        //3.打开连接
        connection.start();

        //4.创建session
        /**
         * createSession：（后续讲解）
         *      参数一：是否开启事务
         *      参数二：消息确认机制
         */
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        //5.创建目标地址（Queue：点对点消息，Topic：发布订阅信息）,也是队列地址，指定地址名称是：queue01
        Queue queue = session.createQueue("queue01");

        //6.创建消息生产者,并指定要发送的队列：queue
        MessageProducer producer = session.createProducer(queue);

        //7.创建消息,createTextMessage是文本类型
        TextMessage testMessage = session.createTextMessage("test message");

        //8.发送消息
        producer.send(testMessage);

        System.out.println("消息发送完成");

        //9.释放资源
        session.close();
        connection.close();
    }
}
```

执行该代码，在后台页面可以查看到有一个消息已经发送上了

![queue01](queue01.png)

**接收消息-第一种**

```
1.创建连接工厂
2.创建连接
3.打开连接
4.创建session
5.指定目标地址（Queue：点对点消息，Topic：发布订阅信息）
6.创建消息消费者
7.接收消息
8.释放资源
```

com.hui.consumer.PTP_Consumer

```java
package com.hui.consumer;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 演示点对点模式---接收消息，第一种方案
 */
public class PTP_Consumer {
    public static void main(String[] args) throws JMSException {

        //1.创建连接工厂
        ConnectionFactory factory=new ActiveMQConnectionFactory("tcp://192.168.25.181:61616");

        //2.创建连接
        Connection connection = factory.createConnection();

        //3.打开连接
        connection.start(); 

        //4.创建session
        /**
         * createSession：
         *      参数一：是否开启事务
         *      参数二：消息确认机制
         */
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        //5.指定目标地址（Queue：点对点消息，Topic：发布订阅信息）保持和生产者一样
        Queue queue = session.createQueue("queue01");

        //6.创建消息消费者,并指定从哪个队列接收消息
        MessageConsumer consumer = session.createConsumer(queue);

        //7.接收消息
        while (true){
            Message message = consumer.receive();

            //如果没有消息，就直接结束
            if (message==null){
                break;
            }

            //如果有消息，判断是否是文本类型的消息
            if (message instanceof TextMessage){
                TextMessage textMessage=(TextMessage) message;
                System.out.println("接收的消息是："+textMessage.getText());
            }
        }

        //8.释放资源
        session.close();
        connection.close();
    }
}
```

**接收消息-第二种，监听器接收，常用**

> 注意：在监听器的模式下千万不要关闭连接，一旦关闭，消息无法接收

```java
package com.hui.consumer;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 演示点对点模式---接收消息，第二种方案---常用
 */
public class PTP_ConsumerListener {

    public static void main(String[] args) throws JMSException {

        //1.创建连接工厂
        ConnectionFactory factory=new ActiveMQConnectionFactory("tcp://192.168.25.181:61616");

        //2.创建连接
        Connection connection = factory.createConnection();

        //3.打开连接
        connection.start();

        //4.创建session
        /**
         * createSession：
         *      参数一：是否开启事务
         *      参数二：消息确认机制
         */
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        //5.指定目标地址（Queue：点对点消息，Topic：发布订阅信息）保持和生产者一样
        Queue queue = session.createQueue("queue01");

        //6.创建消息消费者,并指定从哪个队列接收消息
        MessageConsumer consumer = session.createConsumer(queue);

        //7.设置监听器来接收消息
        consumer.setMessageListener(new MessageListener() {
            //处理信息
            public void onMessage(Message message) {
                //如果有消息，判断是否是文本类型的消息
                if (message instanceof TextMessage){
                    TextMessage textMessage=(TextMessage) message;
                    try {
                        System.out.println("接收的消息是："+textMessage.getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
            }
        });

        //注意：在监听器的模式下千万不要关闭连接，一旦关闭，消息无法接收
    }
}
```

#### 发布/订阅模式

> 注意发布/订阅模式一定要先启动消费者，再启动生产者。就是一定要先订阅，再发送。

**订阅消息**

因为常用监听模式，这里只使用监听的做演示

com.hui.consumer.PS_ConsumerListener

```java
package com.hui.consumer;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 演示发布/订阅模式---接收消息，第二种方案---常用
 */
public class PS_ConsumerListener {

    public static void main(String[] args) throws JMSException {

        //1.创建连接工厂
        ConnectionFactory factory=new ActiveMQConnectionFactory("tcp://192.168.25.181:61616");

        //2.创建连接
        Connection connection = factory.createConnection();

        //3.打开连接
        connection.start();

        //4.创建session
        /**
         * createSession：
         *      参数一：是否开启事务
         *      参数二：消息确认机制
         */
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        //5.指定目标地址（Queue：点对点消息，Topic：发布订阅信息）保持和生产者一样
        Topic topic = session.createTopic("topic01");

        //6.创建消息消费者,并指定从哪个队列接收消息
        MessageConsumer consumer = session.createConsumer(topic);

        //7.设置监听器来接收消息
        consumer.setMessageListener(new MessageListener() {
            //处理信息
            public void onMessage(Message message) {
                //如果有消息，判断是否是文本类型的消息
                if (message instanceof TextMessage){
                    TextMessage textMessage=(TextMessage) message;
                    try {
                        System.out.println("接收的消息是："+textMessage.getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
            }
        });

        //注意：在监听器的模式下千万不要关闭连接，一旦关闭，消息无法接收
    }
}
```

**发送消息**

com.hui.producer.PS_Producer

```java
package com.hui.producer;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 演示发布/订阅模式---消息生产者
 */
public class PS_Producer {

    public static void main(String[] args) throws JMSException {

        //1.创建连接工厂
        ConnectionFactory factory=new ActiveMQConnectionFactory("tcp://192.168.25.181:61616");

        //2.创建连接
        Connection connection = factory.createConnection();

        //3.打开连接
        connection.start();

        //4.创建session
        /**
         * createSession：
         *      参数一：是否开启事务
         *      参数二：消息确认机制
         */
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        //5.创建目标地址（Queue：点对点消息，Topic：发布订阅信息）,也是队列地址，指定地址名称是：topic01
        Topic topic = session.createTopic("topic01");

        //6.创建消息生产者,并指定要发送的队列：queue
        MessageProducer producer = session.createProducer(topic);

        //7.创建消息,createTextMessage是文本类型
        TextMessage testMessage = session.createTextMessage("test message--topic");

        //8.发送消息
        producer.send(testMessage);

        System.out.println("消息发送完成");

        //9.释放资源
        session.close();
        connection.close();
    }
}
```

## Spring操作ActiveMQ

#### 添加pom依赖

```xml
<properties>
    <spring.version>5.0.2.RELEASE</spring.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jms</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.2</version>
    </dependency>

    <!--这个jar包是必须的
            5.0.2.RELEASE版本的Spring需要使用JMS 2.0版本，但spring-jms的依赖没有自动导入JMS 2.0，而activemq-core会导入JMS 1.1的依赖
            所以需要添加以下依赖解决
        -->
    <dependency>
        <groupId>javax.jms</groupId>
        <artifactId>javax.jms-api</artifactId>
        <version>2.0.1</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-all</artifactId>
        <version>5.14.5</version>
    </dependency>
</dependencies>
```

#### 发送消息

##### Spring整合ActiveMQ的xml文件

applicationContext-producer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:amq="http://activemq.apache.org/schema/core"
       xmlns:jms="http://www.springframework.org/schema/jms"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/aop/spring-aop.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/jms
       http://www.springframework.org/schema/jms/spring-jms.xsd
       http://activemq.apache.org/schema/core
       http://activemq.apache.org/schema/core/activemq-core.xsd">

    <!--创建连接工厂-->
    <amq:connectionFactory
            id="amqConnectionFactory"
            brokerURL="tcp://192.168.25.181:61616"
            userName="admin"
            password="admin"/>

    <!--创建缓存连接工厂-->
    <bean id="cachingConnectionFactory" class="org.springframework.jms.connection.CachingConnectionFactory">
        <!--注入连接工厂-->
        <property name="targetConnectionFactory" ref="amqConnectionFactory"/>
        <!--缓存消息信息-->
        <property name="sessionCacheSize" value="5"/>
    </bean>

    <!--创建用于点对点发送的JmsTemplate-->
    <bean id="jmsQueueTemplate" class="org.springframework.jms.core.JmsTemplate">
        <!--注入缓存连接工厂-->
        <property name="connectionFactory" ref="cachingConnectionFactory"/>
        <!--指定是否为点对点模式 为false就是点对点模式-->
        <property name="pubSubDomain" value="false"/>
    </bean>

    <!--创建用于发布/订阅发送的JmsTemplate-->
    <bean id="jmsTopicTemplate" class="org.springframework.jms.core.JmsTemplate">
        <!--注入缓存连接工厂-->
        <property name="connectionFactory" ref="cachingConnectionFactory"/>
        <!--指定是否为发布/订阅模式 为true就是发布/订阅模式-->
        <property name="pubSubDomain" value="true"/>
    </bean>
</beans>
```

**发送消息代码实现**

**点对点模式**

com.hui.producer.Spring_PTPProducer

```java
package com.hui.producer;

import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Session;
import javax.jms.TextMessage;

/**
 * spring与activemq整合的点对点发送演示
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext-producer.xml")
public class Spring_PTPProducer {

    //@Resource(name = "JmsQueueTemplate")
    @Autowired
    @Qualifier("jmsQueueTemplate")
    private JmsTemplate jmsQueueTemplate;

    @Test
    public void ptpSender(){
        /**
         * 参数一：指定队列的名称
         * 参数二：MessageCreator接口，我们需要提供该接口的匿名内部实现
         */
        jmsQueueTemplate.send("spring-ptp", new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                TextMessage textMessage = session.createTextMessage("spring-queue message");
                return textMessage;
            }
        });
        System.out.println("queue消息发送成功");
    }
}
```

**发布/订阅模式**

com.hui.producer.Spring_PSProducer

```java
package com.hui.producer;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Session;
import javax.jms.TextMessage;

/**
 * spring与activemq整合的发布/订阅发送演示
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext-producer.xml")
public class Spring_PSProducer {

    @Resource(name = "jmsTopicTemplate")
    private JmsTemplate jmsTopicTemplate;

    @Test
    public void pcSender(){
        /**
         * 参数一：指定队列的名称
         * 参数二：MessageCreator接口，我们需要提供该接口的匿名内部实现
         */
        jmsTopicTemplate.send("spring-pc", new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                TextMessage textMessage = session.createTextMessage("spring-topic message");
                return textMessage;
            }
        });
        System.out.println("topic消息发送成功");
    }
}
```

#### 接收消息

##### Spring整合ActiveMQ的xml文件

applicationContext-consumer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:amq="http://activemq.apache.org/schema/core"
       xmlns:jms="http://www.springframework.org/schema/jms"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/aop/spring-aop.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/jms
       http://www.springframework.org/schema/jms/spring-jms.xsd
       http://activemq.apache.org/schema/core
       http://activemq.apache.org/schema/core/activemq-core.xsd">

    <!--创建连接工厂-->
    <amq:connectionFactory
            id="amqConnectionFactory"
            brokerURL="tcp://192.168.25.181:61616"
            userName="admin"
            password="admin"/>

    <!--创建缓存连接工厂-->
    <bean id="cachingConnectionFactory" class="org.springframework.jms.connection.CachingConnectionFactory">
        <!--注入连接工厂-->
        <property name="targetConnectionFactory" ref="amqConnectionFactory"/>
        <!--缓存消息信息-->
        <property name="sessionCacheSize" value="5"/>
    </bean>

    <!--配置消息监听主键扫描-->
    <context:component-scan base-package="com.hui.listener"/>

    <!--配置监听器（点对点）-->
    <!--
        destination-type:目标的类型（queue：点对点模式，topic：发布/订阅模式）
    -->
    <jms:listener-container connection-factory="cachingConnectionFactory" destination-type="queue">
        <!--
            destination:监听的队列名称
            ref：监听类，需要代码编写，使用component注解注入到容器中
        -->
        <jms:listener destination="spring-ptp" ref="queueListener"/>
    </jms:listener-container>

    <!--配置监听器（发布/订阅）-->
    <!--
        destination-type:目标的类型（queue：点对点模式，topic：发布/订阅模式）
    -->
    <jms:listener-container connection-factory="cachingConnectionFactory" destination-type="topic">
        <!--
            destination:监听的队列名称
            ref：监听类，需要代码编写，使用component注解注入到容器中
        -->
        <jms:listener destination="spring-pc" ref="topicListener"/>
    </jms:listener-container>
</beans>
```

##### 监听器代码实现

**点对点模式监听**

com.hui.listener.QueueListener

```java
package com.hui.listener;

import org.springframework.stereotype.Component;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

/**
 * 点对点模式的监听类
 */
@Component
public class QueueListener implements MessageListener {

    //接收消息
    public void onMessage(Message message) {
        if (message instanceof TextMessage){
            TextMessage testMessage = (TextMessage)message;
            try {
                System.out.println("topic的消息："+testMessage.getText());
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}
```

**发布/订阅模式监听**

com.hui.listener.TopicListener

```java
package com.hui.listener;

import org.springframework.stereotype.Component;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

/**
 * 发布/订阅模式的监听类
 */
@Component
public class TopicListener implements MessageListener {

    //接收消息
    public void onMessage(Message message) {
        if (message instanceof TextMessage){
            TextMessage testMessage = (TextMessage)message;
            try {
                System.out.println("topic的消息："+testMessage.getText());
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}
```

##### 启动监听器

com.hui.consumer.SpringConsumer

```java
package com.hui.consumer;

import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.io.IOException;

/**
 * 用于启动消费方监听
 */
public class SpringConsumer {
    public static void main(String[] args) throws IOException {
        //1.加载配置文件
        ClassPathXmlApplicationContext context=
                new ClassPathXmlApplicationContext("classpath:applicationContext-consumer.xml");

        //2.启动
        context.start();

        //3.阻塞方法，让程序一直处于等待状态，不停止
        System.in.read();
    }
}
```

## SpringBoot操作ActiveMQ

#### pom文件的依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-activemq</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
</dependencies>
```

#### appliction.properties配置

> 注意：如果想要切换点对点模式和发布/订阅模式，只需要修改spring.jms.pub-sub-domain既可，代码不用改

```properties
server.port=8080
# 服务名称（与SpringCloud整合使用）
spring.application.name=activemq-producer

# springboot与activemq整合配置
spring.activemq.broker-url=tcp://192.168.25.181:61616
spring.activemq.user=admin
spring.activemq.password=admin

# 指定发送模式 false是点对点模式  true是发布/订阅模式
spring.jms.pub-sub-domain=false
```

#### 发送消息

com.hui.producer.SpringBootProducer

```java
package com.hui.producer;

import com.hui.springbootactivemq.SpringbootactivemqApplication;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/**
 * 演示springboot与activemq的整合--消息生产者
 */
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = SpringbootactivemqApplication.class)
public class SpringBootProducer {

    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;
    
    @Autowired
    private JmsTemplate jmsTemplate;

    @Test
    public void ptpSender(){

        /**
         * 第一种发送消息的方法
         * 参数一：队列的名称
         * 参数二：消息内容
         */
        jmsMessagingTemplate.convertAndSend("springboot-queue","springboot queue message");
        System.out.println("消息发送成功");
    }
    
        @Test
    public void ptpSender2(){
	   /**
         * 第二种发送消息的方法
         */
        jmsTemplate.send("springboot_queue2", new MessageCreator() {
            @Override
            public Message createMessage(Session session) throws JMSException {
                TextMessage textMessage = session.createTextMessage("springboot queue message2");
                return textMessage;
            }
        });
        System.out.println("消息发送成功");
    }
}
```

#### 接收消息

> 注意：接收消息的监听包必须和启动类在一个包下：springbootactivemq

com.hui.springbootactivemq.listener.MessageListener

```java
package com.hui.springbootactivemq.listener;

import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.TextMessage;

/**
 * 用于监听消息类，既可用于队列监听，也可以用于主题监听
 */
@Component
public class MessageListener {

    @JmsListener(destination = "springboot_queue")
    public void receiveMessage(Message message){
        if (message instanceof TextMessage){
            TextMessage textMessage = (TextMessage) message;
            try {
                System.out.println("接收到的消息："+textMessage.getText());
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}
```
