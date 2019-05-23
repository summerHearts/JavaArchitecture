# 4-RocketMQ-HelloWorld

[TOC]

## 4.1 HelloWorld基本模型

- 我们要使用rockerMQ主要非为以下步骤
- 生产者
    1. 创建DefaultMQProducer类并设定生产者名称,设置serNamesrvAddr,集群模式使用";"进行分割,调用start方法启动即可
    2. 使用message类进行实例化消息,参数分别为:主题/标签/内容
    3. 调用send方法发送消息,并且关闭生产者即可
- 消费者
    1. 创建deaultMQPushConsumer类并设定消费者名称,设置setNamesrvAddr,集群模式用";"进行分割.
    2. 设置defaultMQPushConsumer实例的订阅主题,一个消费者对象可以订阅多个主题,使用subscribe方法订阅(参数1为主题名,参数2为标签内容,可以使用"||"对标签内容进行合并获取)
    3. 消费者实例进行注册监听:设置registerMessageListener方法
    4. 监听类实现MessageListenerConcurrently接口即可,重写conbsumeMessage方法接受数据.ConsumeConcurrently接口即可,重写consumeMessage方法接受数据.(ConsumeConcurrentlyStatus.RECONSUME_LATER;ConsumeConcurrentlyStatus.CONSUME_SUCCESS)
    5. 启动消费者实例对象,调用start方法即可


## 4.2 RocketMQ-HelloWorld代码

- 引入pom文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>com.alibaba.rocketmq</groupId>
        <artifactId>rocketmq-all</artifactId>
        <version>3.2.6</version>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <packaging>jar</packaging>
    <artifactId>rocketmq-example</artifactId>
    <name>rocketmq-example ${project.version}</name>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>${project.groupId}</groupId>
            <artifactId>rocketmq-client</artifactId>
        </dependency>
        <dependency>
            <groupId>${project.groupId}</groupId>
            <artifactId>rocketmq-srvutil</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <dependency>
            <groupId>jboss</groupId>
            <artifactId>javassist</artifactId>
        </dependency>
    </dependencies>
</project>

```

- producer

```java
/**
 * Copyright (C) 2010-2013 Alibaba Group Holding Limited
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.clsaa.edu.rocketmq.quickstart;

import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.client.producer.DefaultMQProducer;
import com.alibaba.rocketmq.client.producer.SendResult;
import com.alibaba.rocketmq.common.message.Message;


/**
 * Producer，发送消息
 * 
 */
public class Producer {
    public static void main(String[] args) throws MQClientException, InterruptedException {
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");

        /**
         * producer配置项
         *  producerGroup DEFAULT_PRODUCER (一个线程下只能有一个组,但是一个组下面可以有多个实例,生产者组)
         *  Producer组名,多个producer如果属于一个应用,发送同样的消息,则应该将他们视为同一组
         *  createTopicKey WBW102 在发送消息时,自动创建服务器不存在的topic,需制定key
         *  defaultTopicQueueNums4 发送消息时,自动创建服务器不存在的topic,默认创建的队列数
         *  sendMsgTimeout 10000 发送消息超时时间,单位毫秒
         *  compressMsgBodyOverHowmuch 4096 消息body超过多大开始压缩(Consumer收到消息会自动解压缩),单位:字节
         *  retryTimesWhenSendFailed 重试次数 (可以配置)
         *  retryAnotherBrokerWhenNotStoreOK FALSE 如果发送消息返回sendResult,但是sendStatus!=SEND_OK,是否重发
         *  maxMessageSize 131072 客户端限制消息的大小,超过报错,同时服务端也会限制(默认128k)
         *  transactionCheckListener 事务消息回查监听器,如果发送事务消息,必须设置
         *  checkThreadPoolMinSize 1 Broker回查Producer事务状态时,线程池大小
         *  checkThreadPoolMaxSize 1 Broker回查Producer事务状态时,线程池大小
         *  checkRequestHoldMax 2000 Broker回查Producer事务状态,Producer本地缓冲请求队列大小
         */

        producer.setNamesrvAddr("123.206.175.47:9876;182.254.210.72:9876");
        producer.start();

        for (int i = 1; i <= 10; i++) {
            try {
                Message msg = new Message("TopicQuickStart",// topic
                    "TagA",// tag
                        "KKK",//key用于标识业务的唯一性
                    ("Hello RocketMQ " + i).getBytes()// body 二进制字节数组
                        );
                SendResult sendResult = producer.send(msg); //ACK确认反馈,通过result判断消息发送成功还是失败
                System.out.println(sendResult); //msgID会在msg经过msgQueue逻辑结构之后才会有ID
            }
            catch (Exception e) {
                e.printStackTrace();
                Thread.sleep(1000);
            }
        }

        producer.shutdown();
    }
}

```

- consumer

```java
/**
 * Copyright (C) 2010-2013 Alibaba Group Holding Limited
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.clsaa.edu.rocketmq.quickstart;

import java.util.List;

import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;
import com.alibaba.rocketmq.common.message.MessageExt;


/**
 * Consumer，订阅消息
 */
public class Consumer {

    public static void main(String[] args) throws InterruptedException, MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name");
        /**
         * Push Consumer设置
         * messageModel CLUSTERING 消息模型,支持以下两种1.集群消费2.广播消费
         * consumeFromWhere CONSUME_FROM_LAST_OFFSET Consumer启动后,默认从什么位置开始消费
         * allocateMessageQueueStrategy
         * allocateMessageQueueAveragely Rebalance 算法实现策略
         * Subsription{} 订阅关系
         * messageListener 消息监听器
         * offsetStore 消费进度存储
         * consumeThreadMin 10 消费线程池数量
         * consumeThreadMax 20 消费线程池数量
         * pullThresholdForQueue 1000 拉去消息本地队列缓存消息最大数
         * pullInterval 拉消息间隔,由于是轮训,所以为0,但是如果用了流控,也可以设置大于0的值,单位毫秒
         * consumeMessageBatchMaxSize 1 批量消费,一次消费杜少条消息
         * pullBatchSize 32 批量拉消息,一次最多拉多少条
         *
         */

        /**
         * 设置Consumer第一次启动是从队列头部开始消费还是队列尾部开始消费<br>
         * 如果非第一次启动，那么按照上次消费的位置继续消费
         */

        consumer.setNamesrvAddr("123.206.175.47:9876;182.254.210.72:9876");
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        consumer.subscribe("TopicQuickStart", "TagA");

        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                    ConsumeConcurrentlyContext context) {
                //System.out.println(Thread.currentThread().getName() + " Receive New Messages: " + msgs);
                MessageExt msg = msgs.get(0);
                try {
                    String topic = msg.getTopic();
                    String msgBody = new String(msg.getBody(),"utf-8");
                    String tags = msg.getTags();
                    System.out.println("get massage : " + " topic : " + topic + " tags : " + tags + " msg : " +msgBody);
                }catch (Exception e){
                    e.printStackTrace();
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER; //requeue 一会再消费
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS; // response broker ack
            }
        });

        consumer.start();

        System.out.println("Consumer Started.");
    }
}

```

## 4.2 较为重要的配置项说明

* producer配置项
    *  producerGroup DEFAULT_PRODUCER (一个线程下只能有一个组,但是一个组下面可以有多个实例,生产者组)
    *  Producer组名,多个producer如果属于一个应用,发送同样的消息,则应该将他们视为同一组
    *  sendMsgTimeout 10000 发送消息超时时间,单位毫秒
    *  compressMsgBodyOverHowmuch 4096 消息body超过多大开始压缩(Consumer收到消息会自动解压缩),单位:字节
    *  retryTimesWhenSendFailed 重试次数 (可以配置)
    *  retryAnotherBrokerWhenNotStoreOK FALSE 如果发送消息返回sendResult,但是sendStatus!=SEND_OK,是否重发
    *  maxMessageSize 131072 客户端限制消息的大小,超过报错,同时服务端也会限制(默认128k)

* Push Consumer设置
    * messageModel CLUSTERING 消息模型,支持以下两种1.集群消费2.广播消费
    * consumeFromWhere CONSUME_FROM_LAST_OFFSET Consumer启动后,默认从什么位置开始消费
    * allocateMessageQueueStrategy
    * allocateMessageQueueAveragely Rebalance 算法实现策略
    * Subsription 订阅关系
    * messageListener 消息监听器
    * offsetStore 消费进度存储

## 4.3 坑与问题

- com.alibaba.rocketmq.client.exception.MQClientException: Send [3] times, still failed, cost [49]ms
    1. 检查nameserver和broker是否启动成功(查看两个日志)
    2. 查看broker的IP地址是否正确,若不正确参照RockerMQ控制台一节的问题与坑对配置文件进行修改