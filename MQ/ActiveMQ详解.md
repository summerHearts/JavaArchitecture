## 1、消息中间件之JMS规范

**什么是Java消息服务**

- Java消息服务指的是两个应用程序之间进行异步通信的API，它为标准消息协议和消息服务提供了一组通用接口，包括创建、发送、读取消息等，用于支持JAVA应用程序开发。在J2EE中，当两个应用程序使用JMS进行通信时，它们之间并不是直接相连的，而是通过一个共同的消息收发服务连接起来，可以达到解耦的效果。

**MOM**

- 面向消息的中间件，使用消息传送提供者来协调消息传输操作。 MOM需要提供API和管理工具。 客户端调用api。 把消息发送到消息传送提供者指定的目的地。
- 在消息发送之后，客户端会技术执行其他的工作。并且在接收方收到这个消息确认之前，提供者一直保留该消息。


**JMS的概念和规范**

![](https://upload-images.jianshu.io/upload_images/325120-304f1d511cde3d23.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

**为什么需要JMS**

- 在JAVA中，如果两个应用程序之间对各自都不了解，甚至这两个程序可能部署在不同的大洲上，那么它们之间如何发送消息呢？举个例子，一个应用程序A部署在印度，另一个应用程序部署在美国，然后每当A触发某件事后，B想从A获取一些更新信息。当然，也有可能不止一个B对A的更新信息感兴趣，可能会有N个类似B的应用程序想从A中获取更新的信息。

- 在这种情况下，JAVA提供了最佳的解决方案-JMS，完美解决了上面讨论的问题。

- JMS同样适用于基于事件的应用程序，如聊天服务，它需要一种发布事件机制向所有与服务器连接的客户端发送消息。JMS与RMI不同，发送消息的时候，接收者不需要在线。服务器发送了消息，然后就不管了；等到客户端上线的时候，能保证接收到服务器发送的消息。这是一个很强大的解决方案，能处理当今世界很多普遍问题。

**JMS五种不同的消息正文格式**

- JMS定义了五种不同的消息正文格式，以及调用的消息类型，允许你发送并接收以一些不同形式的数据，提供现有消息格式的一些级别的兼容性。

  - StreamMessage -- Java原始值的数据流
  - MapMessage--一套名称-值对
  - TextMessage--一个字符串对象
  - ObjectMessage--一个序列化的 Java对象
  - BytesMessage--一个字节的数据流


**JMS的优势**

- 异步

  - JMS天生就是异步的，客户端获取消息的时候，不需要主动发送请求，消息会自动发送给可用的客户端。

- 可靠
  - JMS保证消息只会递送一次。大家都遇到过重复创建消息问题，而JMS能帮你避免该问题。

**JMS消息传送模型**

- 在JMS API出现之前，大部分产品使用“点对点”和“发布/订阅”中的任一方式来进行消息通讯。JMS定义了这两种消息发送模型的规范，它们相互独立。任何JMS的提供者可以实现其中的一种或两种模型，这是它们自己的选择。JMS规范提供了通用接口保证我们基于JMS API编写的程序适用于任何一种模型。

**点对点消息传送模型**

- 在点对点消息传送模型中，应用程序由消息队列，发送者，接收者组成。每一个消息发送给一个特殊的消息队列，该队列保存了所有发送给它的消息(除了被接收者消费掉的和过期的消息)。点对点消息模型有一些特性，如下：

  - 每个消息只有一个接收者；
  - 消息发送者和接收者并没有时间依赖性；
  - 当消息发送者发送消息的时候，无论接收者程序在不在运行，都能获取到消息；
  -  当接收者收到消息的时候，会发送确认收到通知（acknowledgement）。
  -  如果session关闭时，有一些消息已经收到，但还没有被签收，那么当消费者下次连接到相同的队列时，消息还会被签收
  -  如果用户在receive方法中设定了消息选择条件，那么不符合条件的消息会留在队列中不会被接收
  -  	队列可以长久保存消息直到消息被消费者签收。消费者不需要担心因为消息丢失而时刻与jms provider保持连接状态

  ![](https://upload-images.jianshu.io/upload_images/325120-7978db5023e22f6e.gif?imageMogr2/auto-orient/strip)

**发布/订阅消息传递模型**

- 发布/订阅消息模型中，发布者发布一个消息，该消息通过topic传递给所有的客户端。在这种模型中，发布者和订阅者彼此不知道对方，是匿名的且可以动态发布和订阅topic。topic主要用于保存和传递消息，且会一直保存消息直到消息被传递给客户端。

- 发布/订阅消息模型特性如下：

  - 一个消息可以传递给多个订阅者
  - 发布者和订阅者有时间依赖性，只有当客户端创建订阅后才能接受消息，且订阅者需一直保持活动状态以接收消息。
  - 为了缓和这样严格的时间相关性，JMS允许订阅者创建一个可持久化的订阅。这样，即使订阅者没有被激活（运行），它也能接收到发布者的消息。

  - 订阅可以分为非持久订阅和持久订阅
  - 当所有的消息必须接收的时候，则需要用到持久订阅。反之，则用非持久订阅
    
   ![](https://upload-images.jianshu.io/upload_images/325120-4fbfec5b12ee6377.gif?imageMogr2/auto-orient/strip)



## 2、JMS编码接口之间的关系

![](https://markdownimge.oss-cn-beijing.aliyuncs.com/markdown/JMS%E7%BC%96%E7%A0%81%E6%8E%A5%E5%8F%A3%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB.jpeg)

- ConnectionFactory：创建Connection对象的工厂，针对两种不同的jms消息模型，分别有QueueConnectionFactory和TopicConnectionFactory两种。可以通过JNDI来查找ConnectionFactory对象。
- Connection：Connection表示在客户端和JMS系统之间建立的链接（对TCP/IP socket的包装）。Connection可以产生一个或多个Session。跟ConnectionFactory一样，Connection也有两种类型：QueueConnection和TopicConnection。
- Session：Session是操作消息的接口。可以通过session创建生产者、消费者、消息等。Session提供了事务的功能。当需要使用session发送/接收多个消息时，可以将这些发送/接收动作放到一个事务中。同样，也分QueueSession和TopicSession。
- MessageProducer：消息生产者由Session创建，并用于将消息发送到Destination。同样，消息生产者分两种类型：QueueSender和TopicPublisher。可以调用消息生产者的方法（send或publish方法）发送消息。
- MessageConsumer ：消息消费者由Session创建，用于接收被发送到Destination的消息。两种类型：QueueReceiver和TopicSubscriber。可分别通过session的createReceiver(Queue)或createSubscriber(Topic)来创建。当然，也可以session的creatDurableSubscriber方法来创建持久化的订阅者。
- Destination：Destination的意思是消息生产者的消息发送目标或者说消息消费者的消息来源。对于消息生产者来说，它的Destination是某个队列（Queue）或某个主题（Topic）;对于消息消费者来说，它的Destination也是某个队列或主题（即消息来源）。
- MessageListener： 消息监听器。如果注册了消息监听器，一旦消息到达，将自动调用监听器的onMessage方法。


**JMS的可靠性机制**

- JMS消息之后被确认后，才会认为是被成功消费。
- 消息的消费包含三个阶段： *客户端接收消息*、*客户端处理消息*、*消息被确认*

**事务性会话**
 
 ```
Session session=connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
 ```
 设置为true的时候，消息会在session.commit以后自动签收
 
**非事务性会话**

```
Session session=connection.createSession(Boolean.FALSE, Session.AUTO_ACKNOWLEDGE);
```
在该模式下，消息何时被确认取决于创建会话时的应答模式


**AUTO_ACKNOWLEDGE**

- 当客户端成功从recive方法返回以后，或者[MessageListener.onMessage] 方法成功返回以后，会话会自动确认该消息

**CLIENT_ACKNOWLEDGE**

- 客户端通过调用消息的textMessage.acknowledge();确认消息。
在这种模式中，如果一个消息消费者消费一共是10个消息，那么消费了5个消息，然后在第5个消息通过textMessage.acknowledge()，那么之前的所有消息都会被确认。

**DUPS_OK_ACKNOWLEDGE**

- 延迟确认


**本地事务**

- 在一个JMS客户端，可以使用本地事务来组合消息的发送和接收。JMS Session 接口提供了commit和rollback方法。
- JMS Provider会缓存每个生产者当前生产的所有消息，直到commit或者rollback，commit操作将会导致事务中所有的消息被持久存储；rollback意味着JMS Provider将会清除此事务下所有的消息记录。在事务未提交之前，消息是不会被持久化存储的，也不会被消费者消费
- 事务提交意味着生产的所有消息都被发送。消费的所有消息都被确认； 
- 事务回滚意味着生产的所有消息被销毁，消费的所有消息被恢复，也就是下次仍然能够接收到发送端的消息，除非消息已经过期了。


## 3、ActiveMQ的应用场景

- 异步消息


![](https://upload-images.jianshu.io/upload_images/325120-4c963c5150f7e9d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

- 应用解耦

![](https://upload-images.jianshu.io/upload_images/325120-1b06e7636715a1be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

- 流量削锋

![](https://upload-images.jianshu.io/upload_images/325120-28b0e44e27832409.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

## 3.1、安装

 - 1、下载：apache-activemq-5.14.0-bin.tar.gz

    [http://activemq.apache.org/activemq-5140-release.html](http://activemq.apache.org/activemq-5140-release.html)
    
    ![](http://upload-images.jianshu.io/upload_images/325120-e2a0f7f19d01fe1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

- 2、安装activemq

  - 1、gz文件拷贝到/usr/local/src目录

  - 2、解压启动

    ```
    tar -zxvf apache-activemq-5.14.0-bin.tar.gz
    
    cd apache-activemq-5.14.0
    
    cd bin
    
    ./activemq start
    
     netstat -anp|grep 61616
    
    ./activemq stop
    ```

## 3.2、代码实践


**点对点模型通信**

Message生产者

```java
public class JmsSender {

    public static void main(String[] args) {
        ConnectionFactory connectionFactory=new ActiveMQConnectionFactory("" +
                "tcp://192.168.11.140:61616");
        Connection connection=null;
        try {
            //创建连接
            connection=connectionFactory.createConnection();
            connection.start();

            Session session=connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);

            //创建队列（如果队列已经存在则不会创建， first-queue是队列名称）
            //destination表示目的地
            Destination destination=session.createQueue("first-queue");
            //创建消息发送者
            MessageProducer producer=session.createProducer(destination);

            TextMessage textMessage=session.createTextMessage("hello, 菲菲,我是帅帅的mic");
            producer.send(textMessage);
            session.commit();
            session.close();
        } catch (JMSException e) {
            e.printStackTrace();
        }finally {
            if(connection!=null){
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
Message消费者

```java
public class JmsReceiver {

    public static void main(String[] args) {
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("" +
                "tcp://192.168.11.140:61616");
        Connection connection = null;
        try {
            //创建连接
            connection = connectionFactory.createConnection();
            connection.start();

            Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);

            //创建队列（如果队列已经存在则不会创建， first-queue是队列名称）
            //destination表示目的地
            Destination destination = session.createQueue("first-queue");
            //创建消息接收者
            MessageConsumer consumer = session.createConsumer(destination);

            TextMessage textMessage = (TextMessage) consumer.receive();
            System.out.println(textMessage.getText());
            session.commit();
            session.close();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

**测试发布/订阅（Pub/Sub）模型通信**

Topic生产者

```java
public class JmsTopicSender {

    public static void main(String[] args) {
        ConnectionFactory connectionFactory=new ActiveMQConnectionFactory("" +
                "tcp://192.168.11.140:61616");
        Connection connection=null;
        try {
            //创建连接
            connection=connectionFactory.createConnection();
            connection.start();

            Session session=connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);

            //创建队列（如果队列已经存在则不会创建， first-queue是队列名称）
            //destination表示目的地
            Destination destination=session.createTopic("first-topic");
            //创建消息发送者
            MessageProducer producer=session.createProducer(destination);
            TextMessage textMessage=session.createTextMessage("今天心情，晴转多云");
            producer.send(textMessage);
            session.commit();
            session.close();
        } catch (JMSException e) {
            e.printStackTrace();
        }finally {
            if(connection!=null){
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

Topic消费者

```java
public class JmsTopicReceiver {

    public static void main(String[] args) {
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("" +
                "tcp://192.168.11.140:61616");
        Connection connection = null;
        try {
            //创建连接
            connection = connectionFactory.createConnection();
            connection.start();

            Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);

            //创建队列（如果队列已经存在则不会创建， first-queue是队列名称）
            //destination表示目的地
            Destination destination = session.createTopic("first-topic");
            //创建消息接收者
            MessageConsumer consumer = session.createConsumer(destination);
            TextMessage textMessage = (TextMessage) consumer.receive();
            System.out.println(textMessage.getText());
            session.commit();
            session.close();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

持久化存储

```java
public class JmsTopicPersistenteReceiver {

    public static void main(String[] args) {
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("" +
                "tcp://192.168.11.140:61616");
        Connection connection = null;
        try {
            //创建连接
            connection = connectionFactory.createConnection();
            connection.setClientID("DUBBO-ORDER"); //设置持久订阅
            connection.start();

            Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);

            //创建队列（如果队列已经存在则不会创建， first-queue是队列名称）
            //destination表示目的地
            Topic topic = session.createTopic("first-topic");
            //创建消息接收者
//            MessageConsumer consumer = session.createConsumer(destination);
            MessageConsumer consumer = session.createDurableSubscriber(topic,"DUBBO-ORDER");
            TextMessage textMessage = (TextMessage) consumer.receive();
            System.out.println(textMessage.getText());
            session.commit();
            session.close();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            if (connection != null) {
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```

点对点通信和发布订阅通信模式的区别就是创建生产者和消费者对象时提供的Destination对象不同，如果是点对点通信创建的Destination对象是Queue，发布订阅通信模式通信则是Topic。

## 3.3、整合Spring使用

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jms</artifactId>
    <version>4.2.7.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>4.2.7.RELEASE</version>
</dependency>
```
比如我们在我们的系统中现在有两个服务，第一个服务发送消息，第二个服务接收消息，我们下面看看这是如何实现的

- 发送消息

发送消息的配置文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
    http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.2.xsd">

    <!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://192.168.25.155:61616" />
    </bean>
    <!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
    <bean id="connectionFactory"
        class="org.springframework.jms.connection.SingleConnectionFactory">
        <!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
        <property name="targetConnectionFactory" ref="targetConnectionFactory" />
    </bean>
    <!-- 配置生产者 -->
    <!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->
        <property name="connectionFactory" ref="connectionFactory" />
    </bean>
    <!--这个是队列目的地，点对点的 -->
    <bean id="queueDestination" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg>
            <value>spring-queue</value>
        </constructor-arg>
    </bean>
    <!--这个是主题目的地，一对多的 -->
    <bean id="topicDestination" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg value="topic" />
    </bean>
</beans>
```

发送消息的测试方法：

```
@Test
public void testSpringActiveMq() throws Exception {
    //初始化spring容器
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext-activemq.xml");
    //从spring容器中获得JmsTemplate对象
    JmsTemplate jmsTemplate = applicationContext.getBean(JmsTemplate.class);
    //从spring容器中取Destination对象
    Destination destination = (Destination) applicationContext.getBean("queueDestination");
    //使用JmsTemplate对象发送消息。
    jmsTemplate.send(destination, new MessageCreator() {
        
        @Override
        public Message createMessage(Session session) throws JMSException {
            //创建一个消息对象并返回
            TextMessage textMessage = session.createTextMessage("spring activemq queue message");
            return textMessage;
        }
    });
}
```

- 接收消息

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
    http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.2.xsd">

    <!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://192.168.25.168:61616" />
    </bean>
    <!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
    <bean id="connectionFactory"
        class="org.springframework.jms.connection.SingleConnectionFactory">
        <!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
        <property name="targetConnectionFactory" ref="targetConnectionFactory" />
    </bean>
    <!--这个是队列目的地，点对点的 -->
    <bean id="queueDestination" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg>
            <value>spring-queue</value>
        </constructor-arg>
    </bean>
    <!--这个是主题目的地，一对多的 -->
    <bean id="topicDestination" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg value="topic" />
    </bean>
    <!-- 接收消息 -->
    <!-- 配置监听器 -->
    <bean id="myMessageListener" class="cn.e3mall.search.listener.MyMessageListener" />
    <!-- 消息监听容器 -->
    <bean class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="destination" ref="queueDestination" />
        <property name="messageListener" ref="myMessageListener" />
    </bean>
</beans>
```

创建一个MessageListener的实现类

```
public class MyMessageListener implements MessageListener {

    @Override
    public void onMessage(Message message) {
        
        try {
            TextMessage textMessage = (TextMessage) message;
            //取消息内容
            String text = textMessage.getText();
            System.out.println(text);
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }

}
```
测试接收消息的代码

```
@Test
public void testQueueConsumer() throws Exception {
    //初始化spring容器
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext-activemq.xml");
    //等待
    System.in.read();
}
```

## 4、消息的发送策略

**持久化消息**

 - 默认情况下，生产者发送的消息是持久化的。消息发送到broker以后，producer会等待broker对这条消息的处理情况的反馈

 - 可以设置消息发送端发送持久化消息的异步方式

   ```
   connectionFactory.setUseAsyncSend(true);
   ```
 - 回执窗口大小设置

  ```
  connectionFactory.setProducerWindowSize();
  ```
- 非持久化消息
 
 ```
 textMessage.setJMSDeliveryMode(DeliveryMode.NON_PERSISTENCE);
 ```
 
 非持久化消息模式下，默认就是异步发送过程，如果需要对非持久化消息的每次发送的消息都获得broker的回执的话

 ```
 connectionFactory.setAlwaysSyncSend();
 ```

## 5、consumer获取消息是pull还是（broker的主动 push）

- 默认情况下，mq服务器（broker）采用异步方式向客户端主动推送消息(push)。也就是说broker在向某个消费者会话推送消息后，不会等待消费者响应消息，直到消费者处理完消息以后，主动向broker返回处理结果
 
- broker端一旦有消息，就主动按照默认设置的规则推送给当前活动的消费者。 每次推送都有一定的数量限制，而这个数量就是prefetchSize预取数量

**Queue**

- 持久化消息   prefetchSize=1000

- 非持久化消息1000

**topic**

- 持久化消息        100

- 非持久化消息      32766

假如prefetchSize=0 . 此时对于consumer来说，就是一个pull模式


## 6、acknowledge为什么能够在第5次主动执行ack以后，把前面的消息都确认掉

![](https://upload-images.jianshu.io/upload_images/325120-ad14a135ccec68db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

deliveredMessages 表示已经被consumer接收但是未被确认的消息

- 1、MessageDispatch消息分发信息

```
public static final byte DATA_STRUCTURE_TYPE = CommandTypes.MESSAGE_DISPATCH;

protected ConsumerId consumerId;
protected ActiveMQDestination destination;
protected Message message;
protected int redeliveryCounter;

protected transient long deliverySequenceId;
protected transient Object consumer;
protected transient Runnable transmitCallback;
protected transient Throwable rollbackCause;
```

- 2、自己主动确认。是再接受到消息，反馈给上层应用之前就给确认

  - 在afterMessageIsConsumed方法中
  - 先deliveredMessages.clear();
  - 接着 session.sendAck(ack);
   
- 3、在从tcp取到消息后放到unconsumedMessages等待消费

- 4、在从unconsumedMessages取出消息预处理后。在beforeMessageIsConsumed方法将消息加到deliveredMessages 里面。unconsumedMessages; 消费者从mq接受到消息存储的位置，还没有消费。

- 5、receive内部消费之前

```
 private void beforeMessageIsConsumed(MessageDispatch md) throws JMSException {
        md.setDeliverySequenceId(session.getNextDeliveryId());
        lastDeliveredSequenceId = md.getMessage().getMessageId().getBrokerSequenceId();
        if (!isAutoAcknowledgeBatch()) {
            synchronized(deliveredMessages) {
                deliveredMessages.addFirst(md);
            }
            if (session.getTransacted()) {
                if (transactedIndividualAck) {
                    immediateIndividualTransactedAck(md);
                } else {
                    ackLater(md, MessageAck.DELIVERED_ACK_TYPE);
                }
            }
        }
}
```

- 6、receive内部消费之后

```
private void afterMessageIsConsumed(MessageDispatch md, boolean messageExpired) throws JMSException {
        if (unconsumedMessages.isClosed()) {
            return;
        }
        if (messageExpired) {
            acknowledge(md, MessageAck.DELIVERED_ACK_TYPE);
            stats.getExpiredMessageCount().increment();
        } else {
            stats.onMessage();
            if (session.getTransacted()) {
                // Do nothing.
            } else if (isAutoAcknowledgeEach()) {
                if (deliveryingAcknowledgements.compareAndSet(false, true)) {
                    synchronized (deliveredMessages) {
                        if (!deliveredMessages.isEmpty()) {
                            if (optimizeAcknowledge) {
                                ackCounter++;

                                // AMQ-3956 evaluate both expired and normal msgs as
                                // otherwise consumer may get stalled
                                if (ackCounter + deliveredCounter >= (info.getPrefetchSize() * .65) || (optimizeAcknowledgeTimeOut > 0 && System.currentTimeMillis() >= (optimizeAckTimestamp + optimizeAcknowledgeTimeOut))) {
                                    MessageAck ack = makeAckForAllDeliveredMessages(MessageAck.STANDARD_ACK_TYPE);
                                    if (ack != null) {
                                        deliveredMessages.clear();
                                        ackCounter = 0;
                                        session.sendAck(ack);
                                        optimizeAckTimestamp = System.currentTimeMillis();
                                    }
                                    // AMQ-3956 - as further optimization send
                                    // ack for expired msgs when there are any.
                                    // This resets the deliveredCounter to 0 so that
                                    // we won't sent standard acks with every msg just
                                    // because the deliveredCounter just below
                                    // 0.5 * prefetch as used in ackLater()
                                    if (pendingAck != null && deliveredCounter > 0) {
                                        session.sendAck(pendingAck);
                                        pendingAck = null;
                                        deliveredCounter = 0;
                                    }
                                }
                            } else {
                                MessageAck ack = makeAckForAllDeliveredMessages(MessageAck.STANDARD_ACK_TYPE);
                                if (ack!=null) {
                                    deliveredMessages.clear();
                                    session.sendAck(ack);
                                }
                            }
                        }
                    }
                    deliveryingAcknowledgements.set(false);
                }
            } else if (isAutoAcknowledgeBatch()) {
                ackLater(md, MessageAck.STANDARD_ACK_TYPE);
            } else if (session.isClientAcknowledge()||session.isIndividualAcknowledge()) {
                boolean messageUnackedByConsumer = false;
                synchronized (deliveredMessages) {
                    messageUnackedByConsumer = deliveredMessages.contains(md);
                }
                if (messageUnackedByConsumer) {
                    ackLater(md, MessageAck.DELIVERED_ACK_TYPE);
                }
            }
            else {
                throw new IllegalStateException("Invalid session state.");
            }
        }
}
```

## 7、消息确认

- ACK_TYPE，消费端和broker交换ack指令的时候，还需要告知broker  ACK_TYPE。 
- ACK_TYPE表示确认指令的类型，broker可以根据不同的ACK_TYPE去针对当前消息做不同的应对策略

  - REDELIVERED_ACK_TYPE (broker会重新发送该消息)  重发侧策略
  - DELIVERED_ACK_TYPE  消息已经接收，但是尚未处理结束
  - STANDARD_ACK_TYPE  表示消息处理成功


## 8、ActiveMQ支持的传输协议

- client端和broker端的通讯协议： TCP、UDP 、NIO、SSL、Http（s）、vm

## 9、ActiveMQ持久化存储

![](https://upload-images.jianshu.io/upload_images/325120-a433345dd7f002aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

**ActiveMQ提供了插件式的消息存储，主要有有如下几种:**

- 1.AMQ消息存储-基于文件的存储方式，是以前的默认消息存储

- 2.KahaDB消息存储-提供了容量的提升和恢复能力，是现在的默认存储方式

- 3.JDBC消息存储-消息基于JDBC存储的

- 4.Memory消息存储-基于内存的消息存储


**1、kahaDB  默认的存储方式**

```
<persistenceAdapter>
    <kahaDB directory="${activemq.data}/kahadb"/>
</persistenceAdapter>
```

可用的属性有:

  - 1、 director: KahaDB存放的路径，默认值activemq-data
  - 2、indexWriteBatchSize: 批量写入磁盘的索引page数量,默认值为1000
  - 3、 indexCacheSize: 内存中缓存索引page的数量,默认值10000
  - 4、enableIndexWriteAsync: 是否异步写出索引,默认false
  - 5、 journalMaxFileLength: 设置每个消息data log的大小，默认是32MB
  - 6、enableJournalDiskSyncs: 设置是否保证每个没有事务的内容，被同步写入磁盘，JMS持久化的时候需要，默认为true
  - 7、 cleanupInterval: 在检查到不再使用的消息后,在具体删除消息前的时间,默认30000
  - 8、checkpointInterval: checkpoint的间隔时间，默认是5000
  - 9、 ignoreMissingJournalfiles: 是否忽略丢失的消息日志文件，默认false
  - 10、 checkForCorruptJournalFiles: 在启动的时候，将会验证消息文件是否损坏，默认false
  - 11、checksumJournalFiles: 是否为每个消息日志文件提供checksum,默认false
  - 12、 archiveDataLogs: 是否移动文件到特定的路径，而不是删除它们，默认false
  - 13、 directoryArchive: 定义消息已经被消费过后，移动data log到的路径，默认null
  - 14、 databaseLockedWaitDelay: 获得数据库锁的等待时间(used by shared master/slave),默认10000
  - 15、 maxAsyncJobs: 设置最大的可以存储的异步消息队列，默认值10000，可以和concurrent MessageProducers设置成一样的值。
  - 16、 concurrentStoreAndDispatchTransactions:是否分发消息到客户端，同时事务存储消息，默认true
  - 17、 concurrentStoreAndDispatchTopics: 是否分发Topic消息到客户端，同时进行存储，默认true
  - 18、concurrentStoreAndDispatchQueues: 是否分发queue消息到客户端，同时进行存储，默认true

**2、AMQ 基于文件的存储方式**

　　AMQ Message Store是ActiveMQ5.0缺省的持久化存储，它是一个基于文件、事务存储设计为快速消息存储的一个结构，该结构是以流的形式来进行消息交互的。
  
   这种方式中，Messages被保存到data logs中，同时被reference store进行索引以提高存取速度。Data logs由一些简单的data log文件组成，缺省的文件大小是32M，如果某个消息的大小超过了data log文件的大小，那么可以修改配置以增加data log文件的大小。如果某个data log文件中所有的消息都被成功消费了，那么这个data log文件将会被标记，以便在下一轮的清理中被删除或者归档。
   
AMQ Message Store配置示例：

  - 写入速度很快，容易恢复。
  - 文件默认大小是32M
  
  ![](https://upload-images.jianshu.io/upload_images/325120-99c13adfbba3dc83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

**3、JDBC 基于数据库的存储**

  配置如下：
  
   ![image.jpeg](https://upload-images.jianshu.io/upload_images/325120-ec6ed97b2891af26.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


  ![image.jpeg](https://upload-images.jianshu.io/upload_images/325120-7f58b36f639754cb.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  
  
  JDBC Message Store with ActiveMQ Journal
  
  这种方式克服了JDBC Store的不足，使用快速的缓存写入技术，大大提高了性能。
  配置如下：

  ```
  <beans>
　　<broker brokerName="test-broker" xmlns="http://activemq.apache.org/schema/core">
　　　　<persistenceFactory>
　　　　<journalPersistenceAdapterFactory journalLogFiles="4" journalLogFileSize="32768"
　　　　　　useJournal="true" useQuickJournal="true" dataSource="#mysql_ds" dataDirectory="activemq-data">
　　　　</persistenceFactory>
　　</broker>
  </beans>
  ```

JDBC Store和JDBC Message Store with ActiveMQ Journal的区别：

  - 1. JDBC with journal的性能优于jdbc
  - 2. JDBC用于master/slave模式的数据库分享
  - 3. JDBC with journal不能用于master/slave模式
  - 4. 一般情况下，推荐使用jdbc with journal 

**生成的数据库表**

  - ACTIVEMQ_ACKS ： 存储持久订阅的信息
  - ACTIVEMQ_LOCK ： 锁表（用来做集群的时候，实现master选举的表）
  - ACTIVEMQ_MSGS ： 消息表

   1.消息表,缺省表明为ACTIVEMQ_MSGS, queue和topic都存储在里面，结构如下:
  ![image.png](https://upload-images.jianshu.io/upload_images/325120-7a9608eb32ed4951.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

   2.ACTIVEMQ_ACKS表存储持久订阅的信息和最后一个持久订阅接收的消息ID,结构如下:
  ![image.png](https://upload-images.jianshu.io/upload_images/325120-04cb3cf196fd3b09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   
   3.锁定表，缺省表名为ACTIVEMQ_LOCK，用来确保在某一时刻，只能有一个ActiveMQ broker实例来访问数据库，结构如下：
  ![image.png](https://upload-images.jianshu.io/upload_images/325120-423c24508a117327.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**4、Memory Message Store**

- 内存消息存储主要是存储所有的持久化的消息在内存中。这里没有动态的缓存存在，所以你必须注意设置你的broker所在的JVM和内存限制。

```xml
<beans>
 　<broker brokerName="test_broker" persistent="false" xmlns="http://activemq.apache.org/schema/core">
　　　　<transportConnectors>
　　　　<transportConnector uri="tcp://localhost:61635">
  　　　</transportConnectors>
</beans>
```


## 10、静态网络连接

![](https://upload-images.jianshu.io/upload_images/325120-2592af606fe7c849.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

**ActiveMQ的networkConnector是什么**

 - 在某些情况下，需要多个ActiveMQ的Broker做集群，那么就涉及到Broker到Broker的通信，这个就称为ActiveMQ的networkConnector.

 - ActiveMQ的networkConnector默认是单向的，一个Broker在一端发送消息，另一个Broker在另一端接收消息，这就是所谓的"桥接"。ActiveMQ也支持双向链接，创建一个双向的通道对于两个Broker不仅发送消息而且也能从相同的通道接收消息，通常作为duplex connector来映射，如下：

![](https://upload-images.jianshu.io/upload_images/325120-1e64a0dc8f38d5ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

**有两种配置Client到Broker的链接方式**

![image.png](https://upload-images.jianshu.io/upload_images/325120-694b601d3367d8aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 第一种： Client通过Staticlly配置的方式去连接Broker(静态链接)

- 第二种:  Client通过discover agent来dynamically的发现Brokers(动态链接)

**1、Static networks:**
 
 - Static networkConnector是用于创建一个静态的配置对于网络中的多个Broker,这种协议用于复合url,一个复合url包括多个url地址，格式如下:

    >static:(uri1,uri2,uri3, ...)?key=value

   ```xml
   <networkConnectors>
        <networkConnector name="local network" uri="static://(tcp://ip:prot,tcp://ip:port)"/>
   </networkConnectors>
   ```
   
   在activemq.xml配置如下：

   ![](https://upload-images.jianshu.io/upload_images/325120-a713e4a03889020f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


**常用networkConnector配置的可用属性：**

- conduitSubscriptions ：默认true，是否把同一个broker的多个consumer当做一个来处理

- duplex ：默认false，设置是否能双向通信


## 11、**“丢失”的消息**

 - broker1和broker2通过networkConnector连接，一些consumers连接到broker2，消费broker1上的消息。消息先被broker2从broker1上消费掉，然后转发给这些consumers。不幸的是转发部分消息的时候broker2重启了，这些consumers发现broker2连接失败，通过failover连接到broker1上去了，但是有一部分他们还没有消费的消息被broker1已经分发到了broker2上去了。这些消息，就好像是消失了。

 - broker1 中my-queue4 接收到20条消息。

 ![](https://upload-images.jianshu.io/upload_images/325120-3a582297c5d8adae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
 
 - broker1通过静态网络与broker2连接，与broker2相连的消费者消费后，broker1中Number of Pending Messages为0，即消息先被broker2从broker1上消费掉。

 ![](https://upload-images.jianshu.io/upload_images/325120-e14da443cd6dd5d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 一些consumers连接到broker1，没法从broker1获取消息消费。

  ![](https://upload-images.jianshu.io/upload_images/325120-c7643689d178fced.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
**针对“丢失”的消息，配置replayWhenNoConsumers选项**

 这个选项使得broker1上有需要转发的消息但是没有消费者时，把消息回流到它原始的broker。同时把enableAudit设置为false，为了防止消息回流后被当做重复消息而不被分发。
 
 ```xml
 <policyEntries>
        <policyEntry queue=">" enableAudit="false">
                <networkBridgeFilterFactory>
                        <conditionalNetworkBridgeFilterFactory replayWhenNoConsumers="true"/>
                </networkBridgeFilterFactory>
        </policyEntry>
</policyEntries>
 ```
 
 **容错的链接--Failover**

  - Failover协议实现了自动重新链接的逻辑。默认情况下，这种协议用于随机的去选择一个链接去链接，如果链接失败了，那么会链接到其他的Broker上。默认的配置定义了延迟重新链接，意味着传输将会在10秒后自动的去重新链接可用的broker。当然所有的重新链接参数都可以根据应用的需要而配置。

    ```java
    ConnectionFactory connectionFactory=new ActiveMQConnectionFactory("failover:(tcp://192.168.174.104:61616,tcp://192.168.174.104:61676)?randomize=false");
    ```
  randomize：使用随机链接，以达到负载均衡的目的，默认true。

