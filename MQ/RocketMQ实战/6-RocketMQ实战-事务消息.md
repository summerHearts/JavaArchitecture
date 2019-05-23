# 6-RocketMQ实战-事务消息

[TOC]

## 6.1 简介

[http://www.jianshu.com/p/453c6e7ff81c](http://www.jianshu.com/p/453c6e7ff81c)

- 事务模式:支持事务方式对消息进行提交处理,在rocket里事务非为两个阶段
    - 第一个阶段为把消息传递给MQ只不过消费端不可见,但是数据其实已经发送到broker上
    - 第二个阶段为本地消息回调处理,如果成功的话返回commitmessage,则在broker上的数据对消费端可见,失败则为rollbackmessage,消费端不可见.
    - 如果确认消息发送失败了怎么办？RocketMQ会定期扫描消息集群中的事物消息，如果发现了Prepared消息，它会向消息发送端(生产者)确认.
    - 如果消费失败怎么办？阿里提供给我们的解决方法是：人工解决。大家可以考虑一下，按照事务的流程，因为某种原因Smith加款失败，那么需要回滚整个流程。如果消息系统要实现这个回滚流程的话，系统复杂度将大大提升，且很容易出现Bug，估计出现Bug的概率会比消费失败的概率大很多。这也是RocketMQ目前暂时没有解决这个问题的原因，在设计实现消息系统时，我们需要衡量是否值得花这么大的代价来解决这样一个出现概率非常小的问题，这也是大家在解决疑难问题时需要多多思考的地方。
- 传统方式:两阶段提交协议(2PC)经常被用来处理分布式事务,一般氛围协调器C和若干事务执行者Si两种角色,哲理的事务执行者就是具体的数据库,协调器可以和事务执行在一台机器上.
    - 两阶段提交涉及多次节点间的网络通信,通信时间太长
    - 事务相对时间变长,对预定资源的占用时间变长
![image](http://roclsaarocketrrimgbed-10042610.cos.myqcloud.com/1338803383_5864.jpg)
![image](http://roclsaarocketrrimgbed-10042610.cos.myqcloud.com/1338803474_8977.jpg)

## 6.2 代码示例

- producer

```
package com.clsaa.edu.rocketmq.transaction;

import com.alibaba.rocketmq.client.producer.LocalTransactionExecuter;
import com.alibaba.rocketmq.client.producer.LocalTransactionState;
import com.alibaba.rocketmq.common.message.Message;


/**
 * 执行本地事务
 */
public class TransactionExecuterImpl implements LocalTransactionExecuter {


    @Override
    public LocalTransactionState executeLocalTransactionBranch(final Message msg, final Object arg) {
        System.out.println(msg.toString());
        System.out.println("msg = " + new String(msg.getBody()));
        System.out.println("arg = " + arg);
        String tag = msg.getTags();
        System.out.println("这里执行入库操作....入库成功");

        return LocalTransactionState.COMMIT_MESSAGE;
    }
}

```

-TransactionExecuterImpl 

```

package com.clsaa.edu.rocketmq.transaction;

import com.alibaba.rocketmq.client.producer.LocalTransactionExecuter;
import com.alibaba.rocketmq.client.producer.LocalTransactionState;
import com.alibaba.rocketmq.common.message.Message;


/**
 * 执行本地事务
 */
public class TransactionExecuterImpl implements LocalTransactionExecuter {


    @Override
    public LocalTransactionState executeLocalTransactionBranch(final Message msg, final Object arg) {
        System.out.println(msg.toString());
        System.out.println("msg = " + new String(msg.getBody()));
        System.out.println("arg = " + arg);
        String tag = msg.getTags();
        System.out.println("这里执行入库操作....入库成功");

        return LocalTransactionState.COMMIT_MESSAGE;
    }
}


```

## 6.3参考文档

- 分布式开放消息系统(RocketMQ)的原理与实践 - 简书[http://www.jianshu.com/p/453c6e7ff81c](http://www.jianshu.com/p/453c6e7ff81c)