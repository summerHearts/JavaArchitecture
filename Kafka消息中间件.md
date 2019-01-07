##Kafka应用场景

![image.png](https://upload-images.jianshu.io/upload_images/325120-cc3778c2bbb39ce8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

##Kafka简介

Kafka 是一种高吞吐的分布式发布订阅消息系统，能够替代传统的消息队列用于解耦合数据处理，缓存未处理消息等，同时具有更高的吞吐率，支持分区、多副本、冗余，因此被广泛用于大规模消息数据处理应用。Kafka 支持Java 及多种其它语言客户端，可与Hadoop、Storm、Spark等其它大数据工具结合使用。

##Kafka架构

![](https://upload-images.jianshu.io/upload_images/325120-e0e9685e7845f6d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](https://upload-images.jianshu.io/upload_images/325120-2a88ace956f1e83a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

**Broker**：Kafka集群包含一个或多个服务器，这些服务器就是Broker。

**Topic**：每条发布到Kafka集群的消息都必须有一个Topic。

**Partition**：是物理概念上的分区，为了提供系统吞吐率，在物理上每个Topic会分成一个或多个Partition，每个Partition对应一个文件夹。

**Producer**：消息产生者，负责生产消息并发送到Kafka Broker。

**Consumer**：消息消费者，向kafka broker读取消息并处理的客户端。

**Consumer Group**：每个Consumer属于一个特定的组，组可以用来实现一条消息被组内多个成员消费等功能。


##kafka集群安装

下载

```sh
$ cd /opt
$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/0.11.0.0/kafka_2.12-0.11.0.0.tgz
$ tar -zxvf kafka_2.12-0.11.0.0.tgz
$ cd kafka_2.12-0.11.0.0
```

在config目录下，可以看到很多的配置文件，修改server.properties

```sh
broker.id=0 #每个kafka节点的唯一标识
listeners=PLAINTEXT://192.168.5.28:9092 #监听端口 
log.dirs=/data/kafka-logs #日志地址
zookeeper.connect=192.168.5.28:2181,192.168.5.29:2181,192.168.5.30:2181/kafka #zookeeper地址
```
![](https://upload-images.jianshu.io/upload_images/325120-5fa6921fea7dc314.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

把配置复制到 192.168.252.122 ,192.168.252.123 两台服务器上。同时保证broker-id不同。

启动:

```
./bin/kafka-server-start.sh  -daemon config/server.properties 
```

同时看到zookeeper上的注册信息

```
cluster, controller, controller_epoch, brokers, zookeeper, admin, isr_change_notification, consumers, latest_producer_id_block, config
```
brokers  – kafka集群的broker信息 。

consumer  ids/owners/offsets

##Kafka创建topic

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

bin/kafka-topics.sh --list --zookeeper localhost:2181
```

![](https://upload-images.jianshu.io/upload_images/325120-82f7741b0eba75a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

```
bin/kafka-console-producer.sh --broker-list 192.168.5.28:9092 --topic test //localhost:9092 集群地址
```
![](https://upload-images.jianshu.io/upload_images/325120-2aec6bd76ea068fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

```
bin/kafka-console-consumer.sh  --bootstarp-server 192.168.5.29:9092  --topic test --from-beginning
```
![](https://upload-images.jianshu.io/upload_images/325120-cf0367bbedd57907.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

请参考Kafka官方文档

http://kafka.apache.org/documentation/#quickstart

##Kafka的实现细节

**消息**

消息是kafka中最基本的数据单元。消息由一串字节构成，其中主要由key和value构成，key和value也都是byte数组。key的主要作用是根据一定的策略，将消息路由到指定的分区中，这样就可以保证包含同一key的消息全部写入到同一个分区中，key可以是null。为了提高网络的存储和利用率，生产者会批量发送消息到kafka，并在发送之前对消息进行压缩。

![](https://upload-images.jianshu.io/upload_images/325120-a61983c53e64212b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

**topic&partition**

![](https://upload-images.jianshu.io/upload_images/325120-726ecd9668a77ed2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


Topic是用于存储消息的逻辑概念，可以看作一个消息集合。每个topic可以有多个生产者向其推送消息，也可以有任意多个消费者消费其中的消息。
每个topic可以划分多个分区（每个Topic至少有一个分区），同一topic下的不同分区包含的消息是不同的。每个消息在被添加到分区时，都会被分配一个offset（称之为偏移量），它是消息在此分区中的唯一编号，kafka通过offset保证消息在分区内的顺序，offset的顺序不跨分区，即kafka只保证在同一个分区内的消息是有序的。

![](https://upload-images.jianshu.io/upload_images/325120-a9ce26c0d16fe056.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

Partition是以文件的形式存储在文件系统中，存储在kafka-log目录下，命名规则是：`<topic_name>-<partition_id>`


**kafka的高吞吐量的因素**

- 1、	顺序写的方式存储数据 ； 
- 2、	批量发送；在异步发送模式中。kafka允许进行批量发送，也就是先讲消息缓存到内存中，然后一次请求批量发送出去。这样减少了磁盘频繁io以及网络IO造成的性能瓶颈

  - batch.size 每批次发送的数据大小
  - linger.ms  间隔时间

- 3、	零拷贝

  - 消息从发送到落地保存，broker维护的消息日志本身就是文件目录，每个文件都是二进制保存，生产者和消费者使用相同的格式来处理。在消费者获取消息时，服务器先从硬盘读取数据到内存，然后把内存中的数据原封不懂的通过socket发送给消费者。虽然这个操作描述起来很简单，但实际上经历了很多步骤：

     - 操作系统将数据从磁盘读入到内核空间的页缓存
     - 应用程序将数据从内核空间读入到用户空间缓存中
     - 应用程序将数据写回到内核空间到socket缓存中
     - 操作系统将数据从socket缓冲区复制到网卡缓冲区，以便将数据经网络发出

![](https://upload-images.jianshu.io/upload_images/325120-2f4a74bc1a150fb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

通过“零拷贝”技术可以去掉这些没必要的数据复制操作，同时也会减少上下文切换次数

![](https://upload-images.jianshu.io/upload_images/325120-3971ce782517e551.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


##Kafka日志策略

**日志保留策略**

无论消费者是否已经消费了消息，kafka都会一直保存这些消息，但并不会像数据库那样长期保存。为了避免磁盘被占满，kafka会配置响应的保留策略（retention policy），以实现周期性地删除陈旧的消息
kafka有两种“保留策略”：
1.	根据消息保留的时间，当消息在kafka中保存的时间超过了指定时间，就可以被删除；
2.	根据topic存储的数据大小，当topic所占的日志文件大小大于一个阀值，则可以开始删除最旧的消息


**日志压缩策略**

在很多场景中，消息的key与value的值之间的对应关系是不断变化的，就像数据库中的数据会不断被修改一样，消费者只关心key对应的最新的value。我们可以开启日志压缩功能，kafka定期将相同key的消息进行合并，只保留最新的value值 

![image.png](https://upload-images.jianshu.io/upload_images/325120-932f10d486825b85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

分区的创建

![](https://upload-images.jianshu.io/upload_images/325120-001d9e89a805518e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)



##Kafka消息可靠性机制

**消息发送可靠性**

生产者发送消息到broker，有三种确认方式（request.required.acks）

- acks = 0: producer不会等待broker（leader）发送ack 。因为发送消息网络超时或broker crash(1.Partition的Leader还没有commit消息 2.Leader与Follower数据不同步)，既有可能丢失也可能会重发。

- acks = 1: 当leader接收到消息之后发送ack，丢会重发，丢的概率很小
- acks = -1: 当所有的follower都同步消息成功后发送ack.  丢失消息可能性比较低。

**消息存储可靠性**

- 每一条消息被发送到broker中，会根据partition规则选择被存储到哪一个partition。如果partition规则设置的合理，所有消息可以均匀分布到不同的partition里，这样就实现了水平扩展。

- 在创建topic时可以指定这个topic对应的partition的数量。在发送一条消息时，可以指定这条消息的key，producer根据这个key和partition机制来判断这个消息发送到哪个partition。
- kafka的高可靠性的保障来自于另一个叫副本（replication）策略，通过设置副本的相关参数，可以使kafka在性能和可靠性之间做不同的切换。

  高可靠性的副本
  
  ```
  sh kafka-topics.sh --create --zookeeper 192.168.11.140:2181 --replication-factor 2 --partitions 3 --topic sixsix
  ```
--replication-factor表示的副本数.  实现交叉备份，保证消息安全。

![](https://upload-images.jianshu.io/upload_images/325120-dc8b553dbe6aea6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


**ISR（副本同步队列）**

维护的是有资格的follower节点

- 1、	副本的所有节点都必须要和zookeeper保持连接状态
- 2、	副本的最后一条消息的offset和leader副本的最后一条消息的offset之间的差值不能超过指定的阀值，这个阀值是可以设置的（replica.lag.max.messages）

![](https://upload-images.jianshu.io/upload_images/325120-e9877133b864429c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

**HW&LEO**

- 关于follower副本同步的过程中，还有两个关键的概念，HW(HighWatermark)和LEO(Log End Offset). 这两个参数跟ISR集合紧密关联。

- HW标记了一个特殊的offset，当消费者处理消息的时候，只能拉去到HW之前的消息，HW之后的消息对消费者来说是不可见的。也就是说，取partition对应ISR中最小的LEO作为HW，consumer最多只能消费到HW所在的位置。每个replica都有HW，leader和follower各自维护更新自己的HW的状态。对于leader新写入的消息，consumer不能立刻消费，leader会等待该消息被所有ISR中的replicas同步更新HW，此时消息才能被consumer消费。这样就保证了如果leader副本损坏，该消息仍然可以从新选举的leader中获取。
- LEO 是所有副本都会有的一个offset标记，它指向追加到当前副本的最后一个消息的offset。当生产者向leader副本追加消息的时候，leader副本的LEO标记就会递增；当follower副本成功从leader副本拉去消息并更新到本地的时候，follower副本的LEO就会增加。


##Kafka文件存储机制

- 在kafka文件存储中，同一个topic下有多个不同的partition，每个partition为一个目录，partition的名称规则为：topic名称+有序序号，第一个序号从0开始，最大的序号为partition数量减1，partition是实际物理上的概念，而topic是逻辑上的概念。

- partition还可以细分为segment，这个segment是什么呢？ 假设kafka以partition为最小存储单位，那么我们可以想象当kafka producer不断发送消息，必然会引起partition文件的无线扩张，这样对于消息文件的维护以及被消费的消息的清理带来非常大的挑战，所以kafka 以segment为单位又把partition进行细分。每个partition相当于一个巨型文件被平均分配到多个大小相等的segment数据文件中（每个setment文件中的消息不一定相等），这种特性方便已经被消费的消息的清理，提高磁盘的利用率
- segment file组成：由2大部分组成，分别为index file和data file，此2个文件一一对应，成对出现，后缀".index"和“.log”分别表示为segment索引文件、数据文件.
- segment文件命名规则：partion全局的第一个segment从0开始，后续每个segment文件名为上一个segment文件最后一条消息的offset值。数值最大为64位long大小，19位数字字符长度，没有数字用0填充

![](https://upload-images.jianshu.io/upload_images/325120-1f8fde310ffb5d38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](https://upload-images.jianshu.io/upload_images/325120-4a69ad5fb10901f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

查找方式

以上图为例，读取offset=170418的消息，首先查找segment文件，其中00000000000000000000.index为最开始的文件，第二个文件为00000000000000170410.index（起始偏移为170410+1=170411），而第三个文件为00000000000000239430.index（起始偏移为239430+1=239431），所以这个offset=170418就落到了第二个文件之中。其他后续文件可以依次类推，以其实偏移量命名并排列这些文件，然后根据二分查找法就可以快速定位到具体文件位置。其次根据00000000000000170410.index文件中的[8,1325]定位到00000000000000170410.log文件中的1325的位置进行读取。


##Kafka实践

引入pom文件

```
 <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>1.1.0</version>
</dependency>
```


![](https://upload-images.jianshu.io/upload_images/325120-e634f260aa6c55d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


创建生产者

![](https://upload-images.jianshu.io/upload_images/325120-06821a47f06555db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

创建消费者

![](https://upload-images.jianshu.io/upload_images/325120-87a9466551ddddf7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

##消息确认的几种方式

自动提交

![](https://upload-images.jianshu.io/upload_images/325120-631664bdbf35242b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

手动提交: 同步提交 异步提交

手动异步提交

```
consumer. commitASync() //手动异步ack
```
手动同步提交

```
consumer. commitSync() //手动异步ack
```

![](https://upload-images.jianshu.io/upload_images/325120-2ab264d83807944e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

 手动批量提交
 
![](https://upload-images.jianshu.io/upload_images/325120-c339ee6533dc8b2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


指定消费某个分区的消息

![](https://upload-images.jianshu.io/upload_images/325120-4dec37b6d24795e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

如何实现发布订阅功能，只需要设置不同的分组就可以。


##Kafka消息的消费原理

之前Kafka存在的一个非常大的性能隐患就是利用ZK来记录各个Consumer Group的消费进度（offset）。当然JVM Client帮我们自动做了这些事情，但是Consumer需要和ZK频繁交互，而利用ZK Client API对ZK频繁写入是一个低效的操作，并且从水平扩展性上来讲也存在问题。所以ZK抖一抖，集群吞吐量就跟着一起抖，严重的时候简直抖的停不下来。
新版Kafka已推荐将consumer的位移信息保存在Kafka内部的topic中，即__consumer_offsets topic。通过以下操作来看看__consumer_offsets_topic是怎么存储消费进度的，__consumer_offsets_topic默认有50个分区

- 1、计算consumer group对应的hash值


![](https://upload-images.jianshu.io/upload_images/325120-6bf0f864a690a30d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- 2、获得consumer group的位移信息

``` 
bin/kafka-simple-consumer-shell.sh --topic __consumer_offsets --partition 15 -broker-list 192.168.11.140:9092,192.168.11.141:9092,192.168.11.138:9092 --formatter kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter
```


##Kafka的分区分配策略


![](https://upload-images.jianshu.io/upload_images/325120-0cb5176660bce2b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- 在kafka中每个topic一般都会有很多个partitions。为了提高消息的消费速度，我们可能会启动多个consumer去消费； 同时，kafka存在consumer group的概念，也就是group.id一样的consumer，这些consumer属于一个consumer group，组内的所有消费者协调在一起来消费消费订阅主题的所有分区。当然每一个分区只能由同一个消费组内的consumer来消费，那么同一个consumer group里面的consumer是怎么去分配该消费哪个分区里的数据，这个就设计到了kafka内部分区分配策略（Partition Assignment Strategy）

- 在 Kafka 内部存在两种默认的分区分配策略：Range（默认） 和 RoundRobin。通过partition.assignment.strategy指定


**consumer rebalance**

当以下事件发生时，Kafka 将会进行一次分区分配：

- 1、	同一个consumer group内新增了消费者
- 2、	消费者离开当前所属的consumer group，包括shuts down 或crashes
- 3、	订阅的主题新增分区（分区数量发生变化）
- 4、	消费者主动取消对某个topic的订阅
- 5、	也就是说，把分区的所有权从一个消费者移到另外一个消费者上，这个是kafka consumer 的rebalance机制。如何rebalance就涉及到前面说的分区分配策略。

两种分区策略

**Range 策略（默认）**

0 ，1 ，2 ，3 ，4，5，6，7，8，9

c0 [0,3] c1 [4,6] c2 [7,9]

10(partition num/3(consumer num) =3

**roundrobin 策略**

0 ，1 ，2 ，3 ，4，5，6，7，8，9
c0,c1,c2
c0 [0,3,6,9]
c1 [1,4,7]
c2 [2,5,8]
Kafka 的key 为null， 是随机｛一个Metadata的同步周期内，默认是10分钟｝



##Kafka 的key 为null如何分配

是随机｛一个Metadata的同步周期内，默认是10分钟｝

计算之后分到不同的分区

![](https://upload-images.jianshu.io/upload_images/325120-c8ae60ae41c34a00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

强制全部发送到分区1

![](https://upload-images.jianshu.io/upload_images/325120-b19301781a22f61c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)






