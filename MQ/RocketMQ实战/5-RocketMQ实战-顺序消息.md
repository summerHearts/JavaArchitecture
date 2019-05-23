# 5-RocketMQ-顺序消息

[TOC]

- RockerMQ可以严格保证消息的顺序消费
    - 遵循全局顺序消费的时候使用一个queue,局部顺序的时候可以使用多个queue并行消费 

## 5.1 顺序消息代码实例

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
package com.clsaa.edu.rocketmq.ordermessage;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;
import java.util.logging.SimpleFormatter;

import com.alibaba.rocketmq.client.exception.MQBrokerException;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.client.producer.DefaultMQProducer;
import com.alibaba.rocketmq.client.producer.MQProducer;
import com.alibaba.rocketmq.client.producer.MessageQueueSelector;
import com.alibaba.rocketmq.client.producer.SendResult;
import com.alibaba.rocketmq.common.message.Message;
import com.alibaba.rocketmq.common.message.MessageQueue;
import com.alibaba.rocketmq.remoting.exception.RemotingException;


/**
 * Producer，发送顺序消息
 * 顺序消费的注意点:
 * 1. producer保证发送消息有序,并且发送到同一个队列
 * 2. consumer确保消费同一个队列
 */
public class Producer {
    public static void main(String[] args) {
        try {
            DefaultMQProducer producer = new DefaultMQProducer("order_producer");

            producer.setNamesrvAddr("123.206.175.47:9876;182.254.210.72:9876");

            producer.start();

            Date date = new Date();

            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

            String dateStr = sdf.format(date);

            for (int i = 0; i < 5; i++) {
                String body = dateStr + " hello rocketMQ " + i;
                Message msg = new Message("TopicOrder2", "TagA", "KEY" + i, body.getBytes());
                //发送数据:如果使用顺序消息,则必须自己实现MessageQueueSelector,保证消息进入同一个队列中去.
                SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                    @Override
                    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {

                        Integer id = (Integer) arg;
                        System.out.println("id : " + id);
                        return mqs.get(id);
                    }
                }, 1);//队列下标 //orderID是选定的topic中队列的下标

                System.out.println(sendResult + " , body : " + body);
            }
            for (int i = 0; i < 5; i++) {
                String body = dateStr + " hello rocketMQ " + i;
                Message msg = new Message("TopicOrder2", "TagA", "KEY" + i, body.getBytes());
                //发送数据:如果使用顺序消息,则必须自己实现MessageQueueSelector,保证消息进入同一个队列中去.
                SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                    @Override
                    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {

                        Integer id = (Integer) arg;
                        System.out.println("id : " + id);
                        return mqs.get(id);
                    }
                }, 2);//队列下标 //orderID是选定的topic中队列的下标

                System.out.println(sendResult + " , body : " + body);
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }

}


```

- consumer

```
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
package com.clsaa.edu.rocketmq.ordermessage;

import java.util.List;
import java.util.Random;
import java.util.concurrent.TimeUnit;

import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeOrderlyContext;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeOrderlyStatus;
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerOrderly;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;
import com.alibaba.rocketmq.common.message.MessageExt;


/**
 * 顺序消息消费，带事务方式（应用可控制Offset什么时候提交）
 */
public class Consumer {

    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("order_producer");
        consumer.setNamesrvAddr("123.206.175.47:9876;182.254.210.72:9876");
        /**
         * 设置Consumer第一次启动是从队列头部开始消费还是队列尾部开始消费<br>
         * 如果非第一次启动，那么按照上次消费的位置继续消费
         */
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        consumer.subscribe("TopicOrder2", "TagA");

        consumer.registerMessageListener(new MessageListenerOrderly() {
            private Random random = new Random();
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
                //设置自动提交
                context.setAutoCommit(true);

                for (MessageExt msg:msgs){
                    System.out.println(msg+ " , content : "+ new String(msg.getBody()));
                }
                try {
                    //模拟业务处理
                    TimeUnit.SECONDS.sleep(random.nextInt(5));
                }catch (Exception e){
                    e.printStackTrace();
                    return  ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT;
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });

        consumer.start();
        System.out.println("consume ! ");
    }
}
```


- 输出结果

```

MessageExt [queueId=1, storeSize=168, queueOffset=20, sysFlag=0, bornTimestamp=1490337644787, bornHost=/112.28.174.121:25028, storeTimestamp=1490337647075, storeHost=/123.206.175.47:10911, msgId=7BCEAF2F00002A9F0000000000008083, commitLogOffset=32899, bodyCRC=505147113, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicOrder2, flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=21, KEYS=KEY0, WAIT=true, TAGS=TagA}, body=36]] , content : 2017-03-24 14:40:44 hello rocketMQ 0
MessageExt [queueId=2, storeSize=168, queueOffset=0, sysFlag=0, bornTimestamp=1490337644908, bornHost=/112.28.174.121:25028, storeTimestamp=1490337647173, storeHost=/123.206.175.47:10911, msgId=7BCEAF2F00002A9F00000000000083CB, commitLogOffset=33739, bodyCRC=505147113, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicOrder2, flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=1, KEYS=KEY0, WAIT=true, TAGS=TagA}, body=36]] , content : 2017-03-24 14:40:44 hello rocketMQ 0
MessageExt [queueId=2, storeSize=168, queueOffset=1, sysFlag=0, bornTimestamp=1490337644927, bornHost=/112.28.174.121:25028, storeTimestamp=1490337647192, storeHost=/123.206.175.47:10911, msgId=7BCEAF2F00002A9F0000000000008473, commitLogOffset=33907, bodyCRC=1763499647, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicOrder2, flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=2, KEYS=KEY1, WAIT=true, TAGS=TagA}, body=36]] , content : 2017-03-24 14:40:44 hello rocketMQ 1
MessageExt [queueId=1, storeSize=168, queueOffset=21, sysFlag=0, bornTimestamp=1490337644831, bornHost=/112.28.174.121:25028, storeTimestamp=1490337647098, storeHost=/123.206.175.47:10911, msgId=7BCEAF2F00002A9F000000000000812B, commitLogOffset=33067, bodyCRC=1763499647, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicOrder2, flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=22, KEYS=KEY1, WAIT=true, TAGS=TagA}, body=36]] , content : 2017-03-24 14:40:44 hello rocketMQ 1
MessageExt [queueId=2, storeSize=168, queueOffset=2, sysFlag=0, bornTimestamp=1490337644944, bornHost=/112.28.174.121:25028, storeTimestamp=1490337647210, storeHost=/123.206.175.47:10911, msgId=7BCEAF2F00002A9F000000000000851B, commitLogOffset=34075, bodyCRC=1880461253, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicOrder2, flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=3, KEYS=KEY2, WAIT=true, TAGS=TagA}, body=36]] , content : 2017-03-24 14:40:44 hello rocketMQ 2
MessageExt [queueId=2, storeSize=168, queueOffset=3, sysFlag=0, bornTimestamp=1490337644964, bornHost=/112.28.174.121:25028, storeTimestamp=1490337647229, storeHost=/123.206.175.47:10911, msgId=7BCEAF2F00002A9F00000000000085C3, commitLogOffset=34243, bodyCRC=118669139, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicOrder2, flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=4, KEYS=KEY3, WAIT=true, TAGS=TagA}, body=36]] , content : 2017-03-24 14:40:44 hello rocketMQ 3
MessageExt [queueId=1, storeSize=168, queueOffset=22, sysFlag=0, bornTimestamp=1490337644851, bornHost=/112.28.174.121:25028, storeTimestamp=1490337647117, storeHost=/123.206.175.47:10911, msgId=7BCEAF2F00002A9F00000000000081D3, commitLogOffset=33235, bodyCRC=1880461253, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicOrder2, flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=23, KEYS=KEY2, WAIT=true, TAGS=TagA}, body=36]] , content : 2017-03-24 14:40:44 hello rocketMQ 2
MessageExt [queueId=2, storeSize=168, queueOffset=4, sysFlag=0, bornTimestamp=1490337644982, bornHost=/112.28.174.121:25028, storeTimestamp=1490337647247, storeHost=/123.206.175.47:10911, msgId=7BCEAF2F00002A9F000000000000866B, commitLogOffset=34411, bodyCRC=427174640, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicOrder2, flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=5, KEYS=KEY4, WAIT=true, TAGS=TagA}, body=36]] , content : 2017-03-24 14:40:44 hello rocketMQ 4
MessageExt [queueId=1, storeSize=168, queueOffset=23, sysFlag=0, bornTimestamp=1490337644870, bornHost=/112.28.174.121:25028, storeTimestamp=1490337647135, storeHost=/123.206.175.47:10911, msgId=7BCEAF2F00002A9F000000000000827B, commitLogOffset=33403, bodyCRC=118669139, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicOrder2, flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=24, KEYS=KEY3, WAIT=true, TAGS=TagA}, body=36]] , content : 2017-03-24 14:40:44 hello rocketMQ 3
MessageExt [queueId=1, storeSize=168, queueOffset=24, sysFlag=0, bornTimestamp=1490337644889, bornHost=/112.28.174.121:25028, storeTimestamp=1490337647154, storeHost=/123.206.175.47:10911, msgId=7BCEAF2F00002A9F0000000000008323, commitLogOffset=33571, bodyCRC=427174640, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicOrder2, flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=25, KEYS=KEY4, WAIT=true, TAGS=TagA}, body=36]] , content : 2017-03-24 14:40:44 hello rocketMQ 4
```