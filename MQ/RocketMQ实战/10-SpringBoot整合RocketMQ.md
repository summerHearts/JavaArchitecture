# 10-SpringBoot整合RocketMQ

## 10.1 Maven依赖

### 10.1.1 父项目maven依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.clsaa.edu.rocketmq</groupId>
    <artifactId>trade-system</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>trade-order</module>
        <module>trade-pay</module>
        <module>trade-web</module>
        <module>trade-coupon</module>
        <module>trade-common</module>
    </modules>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.4.RELEASE</version>
    </parent>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.rockermq</groupId>
                <artifactId>rocketmq-client</artifactId>
                <version>3.2.6</version>
            </dependency>
            <!-- https://mvnrepository.com/artifact/commons-lang/commons-lang -->
            <dependency>
                <groupId>commons-lang</groupId>
                <artifactId>commons-lang</artifactId>
                <version>2.6</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>

```

### 10.1.2 子项目依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>trade-system</artifactId>
        <groupId>com.clsaa.edu.rocketmq</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>trade-common</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>3.2.6</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/commons-lang/commons-lang -->
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--<dependency>-->
            <!--<groupId>org.springframework.boot</groupId>-->
            <!--<artifactId>spring-boot-starter-jdbc</artifactId>-->
        <!--</dependency>-->
        <!--<dependency>-->
            <!--<groupId>mysql</groupId>-->
            <!--<artifactId>mysql-connector-java</artifactId>-->
        <!--</dependency>-->
        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

    </dependencies>

</project>
```

## 10.2 配置文件

```yaml
rocketmq:
  producer:
    # 发送同一类消息的设置为同一个group，保证唯一,默认不需要设置，rocketmq会使用ip@pid(pid代表jvm名字)作为唯一标示
    groupName: vehicleProducerGroup
    #mq的nameserver地址
    namesrvAddr: 123.206.175.47:9876;182.254.210.72:9876
    #如果需要同一个jvm中不同的producer往不同的mq集群发送消息，需要设置不同的instanceName
    instanceName: vehicleProducer
    #topic名称
    topic: TEST
    #根据实际情况设置消息的tag
    tag: TEST
    #消息最大长度
    maxMessageSize: 131072 # 1024*128
    #发送消息超时时间
    sendMsgTimeout: 10000
  consumer:
    namesrvAddr: 123.206.175.47:9876;182.254.210.72:9876
    groupName: vehicleProducerGroup
    topic: sms
    tag: verifycode
    consumeThreadMin: 20
    consumeThreadMax: 64
```

## 10.3 启动类

```
package com.clsaa.edu.trade.common.rocketmq;

import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;
import com.alibaba.rocketmq.client.exception.MQBrokerException;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.client.producer.DefaultMQProducer;
import com.alibaba.rocketmq.client.producer.SendResult;
import com.alibaba.rocketmq.common.message.Message;
import com.alibaba.rocketmq.remoting.exception.RemotingException;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ConfigurableApplicationContext;

/**
 * Created by eggyer on 2017/3/25.
 */
@SpringBootApplication
public class RocketMQApp {
    public static void main(String[] args) throws InterruptedException, RemotingException, MQClientException, MQBrokerException {

        ApplicationContext context = SpringApplication.run(RocketMQApp.class,args);
        DefaultMQProducer defaultMQProducer = context.getBean(DefaultMQProducer.class);
        Message msg = new Message("TEST",// topic
                "TEST",// tag
                "KKK",//key用于标识业务的唯一性
                ("Hello RocketMQ !!!!!!!!!!" ).getBytes()// body 二进制字节数组
        );
        SendResult result = defaultMQProducer.send(msg);
        System.out.println(result);
        DefaultMQPushConsumer consumer = context.getBean(DefaultMQPushConsumer.class);

    }
}

```

## 10.4 自定义异常

```
package com.clsaa.edu.trade.common.rocketmq.exception;

/**
 * Created by eggyer on 2017/3/25.
 */
public class RocketMQException extends Exception{
    public RocketMQException() {
        super();
    }

    public RocketMQException(String message) {
        super(message);
    }

    public RocketMQException(String message, Throwable cause) {
        super(message, cause);
    }

    public RocketMQException(Throwable cause) {
        super(cause);
    }

    protected RocketMQException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}

```

## 10.5 实现 MessageListenerConcurrently接口

```
package com.clsaa.edu.trade.common.rocketmq.conf;

import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import com.alibaba.rocketmq.common.message.MessageExt;

import java.util.List;

/**
 * Created by eggyer on 2017/3/25.
 */
public class MessageListener implements MessageListenerConcurrently{

    private MessageProcessor messageProcessor;

    public void setMessageProcessor(MessageProcessor messageProcessor) {
        this.messageProcessor = messageProcessor;
    }

    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
        for (MessageExt msg : msgs){
            boolean result = messageProcessor.handleMessage(msg);
            if (!result){
                return ConsumeConcurrentlyStatus.RECONSUME_LATER;
            }
        }
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
}

```

## 10.6 设计消息处理接口

```
package com.clsaa.edu.trade.common.rocketmq.conf;

import com.alibaba.rocketmq.common.message.MessageExt;

/**
 * Created by eggyer on 2017/3/25.
 */
public interface MessageProcessor {
    /**
     * 处理消息的接口
     * @param messageExt
     * @return
     */
    public boolean handleMessage(MessageExt messageExt);
}

```

## 10.7 实现消息处理接口

```
package com.clsaa.edu.trade.common.rocketmq.conf;

import com.alibaba.rocketmq.common.message.MessageExt;
import org.springframework.stereotype.Component;

/**
 * Created by eggyer on 2017/3/26.
 */
@Component
public class MessageProcessorImplTest implements MessageProcessor {
    @Override
    public boolean handleMessage(MessageExt messageExt) {
        System.out.println("receive : " + messageExt.toString());
        return true;
    }
}

```

## 10.8 实现生产者

```
package com.clsaa.edu.trade.common.rocketmq.conf;

import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.client.producer.DefaultMQProducer;
import com.clsaa.edu.trade.common.rocketmq.exception.RocketMQException;
import org.apache.commons.lang.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;

/**
 * Created by eggyer on 2017/3/25.
 */
@SpringBootConfiguration
public class RocketMQProducerConfiguration {
    public static final Logger LOGGER = LoggerFactory.getLogger(RocketMQProducerConfiguration.class);
    @Value("${rocketmq.producer.groupName}")
    private String groupName;
    @Value("${rocketmq.producer.namesrvAddr}")
    private String namesrvAddr;
    @Value("${rocketmq.producer.instanceName}")
    private String instanceName;
    @Value("${rocketmq.producer.maxMessageSize}")
    private int maxMessageSize ; //4M
    @Value("${rocketmq.producer.sendMsgTimeout}")
    private int sendMsgTimeout ;

    @Bean
    public DefaultMQProducer getRocketMQProducer() throws RocketMQException {
        if (StringUtils.isBlank(this.groupName)) {
            throw new RocketMQException("groupName is blank");
        }
        if (StringUtils.isBlank(this.namesrvAddr)) {
            throw new RocketMQException("nameServerAddr is blank");
        }
        if (StringUtils.isBlank(this.instanceName)){
            throw new RocketMQException("instanceName is blank");
        }
        DefaultMQProducer producer;
        producer = new DefaultMQProducer(this.groupName);
        producer.setNamesrvAddr(this.namesrvAddr);
        producer.setInstanceName(instanceName);
        producer.setMaxMessageSize(this.maxMessageSize);
        producer.setSendMsgTimeout(this.sendMsgTimeout);
        try {
            producer.start();
            LOGGER.info(String.format("producer is start ! groupName:[%s],namesrvAddr:[%s]"
                    , this.groupName, this.namesrvAddr));
        } catch (MQClientException e) {
            LOGGER.error(String.format("producer is error {}"
                    , e.getMessage(),e));
            throw new RocketMQException(e);
        }
        return producer;
    }
}



```

## 10.9 实现消费者

```
package com.clsaa.edu.trade.common.rocketmq.conf;

import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.clsaa.edu.trade.common.rocketmq.exception.RocketMQException;
import org.apache.commons.lang.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;


/**
 * Created by eggyer on 2017/3/25.
 */
@SpringBootConfiguration
public class RocketMQConsumerConfiguration {
    public static final Logger LOGGER = LoggerFactory.getLogger(RocketMQConsumerConfiguration.class);
    @Value("${rocketmq.consumer.namesrvAddr}")
    private String namesrvAddr;
    @Value("${rocketmq.consumer.groupName}")
    private String groupName;
    @Value("${rocketmq.consumer.topic}")
    private String topic;
    @Value("${rocketmq.consumer.tag}")
    private String tag;
    @Value("${rocketmq.consumer.consumeThreadMin}")
    private int consumeThreadMin;
    @Value("${rocketmq.consumer.consumeThreadMax}")
    private int consumeThreadMax;

    @Autowired
    @Qualifier("messageProcessorImplTest")
    private MessageProcessor messageProcessor;

    @Bean
    public DefaultMQPushConsumer getRocketMQConsumer() throws RocketMQException {
        if (StringUtils.isBlank(groupName)){
            throw new RocketMQException("groupName is null !!!");
        }
        if (StringUtils.isBlank(namesrvAddr)){
            throw new RocketMQException("namesrvAddr is null !!!");
        }
        if (StringUtils.isBlank(topic)){
            throw new RocketMQException("topic is null !!!");
        }
        if (StringUtils.isBlank(tag)){
            throw new RocketMQException("tag is null !!!");
        }
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer(groupName);
        consumer.setNamesrvAddr(namesrvAddr);
        consumer.setConsumeThreadMin(consumeThreadMin);
        consumer.setConsumeThreadMax(consumeThreadMax);
        MessageListener messageListener = new MessageListener();
        messageListener.setMessageProcessor(messageProcessor);
        consumer.registerMessageListener(messageListener);
        try {
            consumer.subscribe(topic,this.tag);
            consumer.start();
            LOGGER.info("consumer is start !!! groupName:{},topic:{},namesrvAddr:{}",groupName,topic,namesrvAddr);
        }catch (MQClientException e){
            LOGGER.error("consumer is start !!! groupName:{},topic:{},namesrvAddr:{}",groupName,topic,namesrvAddr,e);
            throw new RocketMQException(e);
        }
        return consumer;
    }

}

```