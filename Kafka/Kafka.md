## Kafka
有时候，我们学习了一些东西，过了一段时间，又忘记了。只记得当时学过。那么再次回顾的时候，最好能带上几个问题，这样才能让自己记忆的更加清晰。

今日我们来回顾Kafka消息中间件，那么下边是我提出的几个问题，大家看看有几个是能回答上来的。

- 1、**kafka节点之间如何复制备份的？**

- 2、**kafka消息是否会丢失？为什么？**
- 3、**kafka最合理的配置是什么？**
- 4、**kafka的leader选举机制是什么？**
- 5、**kafka对硬件的配置有什么要求？**
- 6、**kafka的消息保证有几种方式？**

![](https://www.icheesedu.com/images/qiniu/%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0-2.png)


## 1、简介
 -  Kafka 是分布式发布-订阅消息系统。它最初由 LinkedIn 公司开发，使用 Scala语言编写,之后成为 Apache 项目的一部分。Kafka 是一个分布式的，可划分的，多订阅者,冗余备份的持久性的日志服务。它主要用于处理活跃的流式数据。
 
 -  消息队列的性能好坏，其文件存储机制设计是衡量一个消息队列服务技术水平和最关键指标之一。下面将从Kafka文件存储机制和物理结构角度，分析Kafka是如何实现高效文件存储，及实际应用效果。

## 2、 Kafka的特点:

 - **高吞吐量、低延迟**：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个topic可以分多个partition, consumer group 对partition进行consume操作。
 
- **可扩展性**：kafka集群支持热扩展
- **持久性、可靠性**：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失
- **容错性**：允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）
- **高并发**：支持数千个客户端同时读写


## 3、 Kafka的使用场景：
- **日志收集**：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等。

- **消息系统**：解耦和生产者和消费者、缓存消息等。
- **用户活动跟踪**：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。
- **运营指标**：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。
- **流式处理**：比如spark streaming和storm
- **事件源**

　　
## 4、架构

![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-190_11-13-31.png)

![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-190_11-13-45.png)

## 5、Kafka专用术语：
 
- **Broker**：消息中间件处理结点，一个Kafka节点就是一个broker，多个broker可以组成一个Kafka集群。

- **Topic**：一类消息，Kafka集群能够同时负责多个topic的分发。
   - Topic是用于存储消息的逻辑概念，可以看作一个消息集合。每个topic可以有多个生产者向其推送消息，也可以有任意多个消费者消费其中的消息
      - 每个topic可以划分多个分区（每个Topic至少有一个分区），同一topic下的不同分区包含的消息是不同的。每个消息在被添加到分区时，都会被分配一个offset（称之为偏移量），它是消息在此分区中的唯一编号，kafka通过offset保证消息在分区内的顺序，offset的顺序不跨分区，即kafka只保证在同一个分区内的消息是有序的；

- **Partition**：topic物理上的分组，一个topic可以分为多个partition，每个partition是一个有序的队列。

     - Partition是以文件的形式存储在文件系统中，存储在kafka-log目录下，命名规则是：`<topic_name>-<partition_id>`

- **Segment**：partition物理上由多个segment组成。

- **offset**：每个partition都由一系列有序的、不可变的消息组成，这些消息被连续的追加到partition中。partition中的每个消息都有一个连续的序列号叫做offset，用于partition唯一标识一条消息
- **消息**   - 消息是kafka中最基本的数据单元。消息由一串字节构成，其中主要由key和value构成，key和value也都是byte数组。key的主要作用是根据一定的策略，将消息路由到指定的分区中，这样就可以保证包含同一key的消息全部写入到同一个分区中，key可以是null。为了提高网络的存储和利用率，生产者会批量发送消息到kafka，并在发送之前对消息进行压缩


- 4.1、topic & partition
   - 在Kafka文件存储中，同一个topic下有多个不同partition，每个partition为一个目录，partiton命名规则为topic名称+有序序号，第一个partiton序号从0开始，序号最大值为partitions数量减1。

   - 这里也就是broker——>topic——>partition——>segment 

   - segment file组成：由2大部分组成，分别为index file和data file，此2个文件一一对应，成对出现，后缀".index"和“.log”分别表示为segment索引文件、数据文件。
 
     ![](https://www.icheesedu.com/images/qiniu/932932-20170818102025709-1227763455.jpg.png)
     
     　segment文件命名规则：partion全局的第一个segment从0开始，后续每个segment文件名为上一个segment文件最后一条消息的offset值。数值最大为64位long大小，19位数字字符长度，没有数字用0填充。
     　
     　segment中index与data file对应关系物理结构如下：
     　
     　![](https://www.icheesedu.com/images/qiniu/932932-20170818102555537-956883187.jpg.png)
 
    索引文件存储大量元数据，数据文件存储大量消息，索引文件中元数据指向对应数据文件中message的物理偏移地址。

　　 其中以索引文件中元数据3,497为例，依次在数据文件中表示第3个message（在全局partiton表示第368772个message），以及该消息的物理偏移地址为497。
　　 
　　 ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-190_15-34-57.png)
　　 
## 6、kafka高吞吐量的因素

![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-190_14-32-56.png)

 - 1、 顺序写的方式存储数据 
  - 2、批量发送；在异步发送模式中。kafka允许进行批量发送，也就是先讲消息缓存到内存中，然后一次请求批量发送出去。这样减少了磁盘频繁io以及网络IO造成的性能瓶颈      - batch.size 每批次发送的数据大小      - linger.ms  间隔时间
       - 3、零拷贝    - 消息从发送到落地保存，broker维护的消息日志本身就是文件目录，每个文件都是二进制保存，生产者和消费者使用相同的格式来处理。在消费者获取消息时，服务器先从硬盘读取数据到内存，然后把内存中的数据原封不动的通过socket发送给消费者。传统操作经历了很多步骤       - 操作系统将数据从磁盘读入到内核空间的页缓存       - 应用程序将数据从内核空间读入到用户空间缓存中       - 应用程序将数据写回到内核空间到socket缓存中       -  操作系统将数据从socket缓冲区复制到网卡缓冲区，以便将数据经网络发出  通过“零拷贝”技术可以去掉这些没必要的数据复制操作，同时也会减少上下文切换次数

   ![1](https://www.icheesedu.com/images/qiniu/Xnip2018-07-190_14-40-19.png)
   
   ![2](https://www.icheesedu.com/images/qiniu/Xnip2018-07-190_14-44-02.png)
   
   避免了内核态与用户态的上下文切换动作
   
## 7、日志策略- 日志保留策略   - 无论消费者是否已经消费了消息，kafka都会一直保存这些消息，但并不会像数据库那样长期保存。为了避免磁盘被占满，kafka会配置响应的保留策略（retention policy），以实现周期性地删除陈旧的消息   - kafka有两种“保留策略”：        - 1、根据消息保留的时间，当消息在kafka中保存的时间超过了指定时间，就可以被删除；       - 2、 根据topic存储的数据大小，当topic所占的日志文件大小大于一个阀值，则可以开始删除最旧的消息- 日志压缩策略   - 在很多场景中，消息的key与value的值之间的对应关系是不断变化的，就像数据库中的数据会不断被修改一样，消费者只关心key对应的最新的value。我们可以开启日志压缩功能，kafka定期将相同key的消息进行合并，只保留最新的value值 。　　 

## 8、消息发送可靠性

 - 生产者发送消息到broker，有三种确认方式（request.required.acks）    - acks = 0: producer不会等待broker（leader）发送ack 。因为发送消息网络超时或broker crash(1.Partition的Leader还没有commit消息 2.Leader与Follower数据不同步)，既有可能丢失也可能会重发。
        - acks = 1: 当leader接收到消息之后发送ack，丢会重发，丢的概率很小    - acks = -1: 当所有的follower都同步消息成功后发送ack.  丢失消息可能性比较低。
    ## 9、消息存储可靠性   - 每一条消息被发送到broker中，会根据partition规则选择被存储到哪一个partition。如果partition规则设置的合理，所有消息可以均匀分布到不同的partition里，这样就实现了水平扩展。
      - 在创建topic时可以指定这个topic对应的partition的数量。在发送一条消息时，可以指定这条消息的key，producer根据这个key和partition机制来判断这个消息发送到哪个partition。   - kafka的高可靠性的保障来自于另一个叫副本（replication）策略，通过设置副本的相关参数，可以使kafka在性能和可靠性之间做不同的切换。两两之间相互备份。
   
      ```
      sh kafka-topics.sh --create --zookeeper 192.168.11.140:2181 --replication-factor 2 --partitions 3 --topic sixsix      --replication-factor表示的副本数
      ```
      
## 10、副本机制

![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-190_14-59-50.png)

- **ISR（副本同步队列）**
     维护的是有资格的follower节点  - 1、 副本的所有节点都必须要和zookeeper保持连接状态  - 2、副本的最后一条消息的offset和leader副本的最后一条消息的offset之间的差值不能超过指定的阀值，这个阀值是可以设置的（replica.lag.max.messages）


- **HW&LEO**  - 关于follower副本同步的过程中，还有两个关键的概念，HW(HighWatermark)和LEO(Log End Offset). 这两个参数跟ISR集合紧密关联。HW标记了一个特殊的offset，当消费者处理消息的时候，只能拉去到HW之前的消息，HW之后的消息对消费者来说是不可见的。也就是说，取partition对应ISR中最小的LEO作为HW，consumer最多只能消费到HW所在的位置。每个replica都有HW，leader和follower各自维护更新自己的HW的状态。对于leader新写入的消息，consumer不能立刻消费，leader会等待该消息被所有ISR中的replicas同步更新HW，此时消息才能被consumer消费。这样就保证了如果leader副本损坏，该消息仍然可以从新选举的leader中获取。  - LEO 是所有副本都会有的一个offset标记，它指向追加到当前副本的最后一个消息的offset。当生产者向leader副本追加消息的时候，leader副本的LEO标记就会递增；当follower副本成功从leader副本拉去消息并更新到本地的时候，follower副本的LEO就会增加

     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-190_15-03-14.png)




## 11、查看kafka数据文件内容- 在使用kafka的过程中有时候需要我们查看产生的消息的信息，这些都被记录在kafka的log文件中。由于log文件的特殊格式，需要通过kafka提供的工具来查看

 ``` ./bin/kafka-run-class.sh kafka.tools.DumpLogSegments --files /tmp/kafka-logs/*/000**.log  --print-data-log {查看消息内容}
 ```
 - 高可用副本机制回顾   - 在kfaka0.8版本前，并没有提供这种High Availablity机制，也就是说一旦一个或者多个broker宕机，则在这期间内所有的partition都无法继续提供服务。如果broker无法再恢复，则上面的数据就会丢失。所以在0.8版本以后引入了High Availablity机制   - 关于leader election       - 在kafka引入replication机制以后，同一个partition会有多个Replica。那么在这些replication之间需要选出一个Leader，Producer或者Consumer只与这个Leader进行交互，其他的Replica作为Follower从leader中复制数据（因为需要保证一个Partition中的多个Replica之间的数据一致性，其中一个Replica宕机以后其他的Replica必须要能继续提供服务且不能造成数据重复和数据丢失）。 如果没有leader，所有的Replica都可以同时读写数据，那么就需要保证多个Replica之间互相同步数据，数据一致性和有序性就很难保证，同时也增加了Replication实现的复杂性和出错的概率。在引入leader以后，leader负责数据读写，follower只向leader顺序fetch数据，简单而且高效- 如何将所有的Replica均匀分布到整个集群    - 为了更好的做到负载均衡，kafka尽量会把所有的partition均匀分配到整个集群上。如果所有的replica都在同一个broker上，那么一旦broker宕机所有的Replica都无法工作。kafka分配Replica的算法       - 1、 把所有的Broker（n）和待分配的Partition排序
           - 2、 把第i个partition分配到 （i mod n）个broker上       - 3、把第i个partition的第j个Replica分配到 ( (i+j) mod n) 个broker上- 如何处理所有的Replica不工作的情况    - 在ISR中至少有一个follower时，Kafka可以确保已经commit的数据不丢失，但如果某个Partition的所有Replica都宕机了，就无法保证数据不丢失了       - 1、等待ISR中的任一个Replica“活”过来，并且选它作为Leader       - 2、选择第一个“活”过来的Replica（不一定是ISR中的）作为Leader这就需要在可用性和一致性当中作出一个简单的折衷。    - 如果一定要等待ISR中的Replica“活”过来，那不可用的时间就可能会相对较长。而且如果ISR中的所有Replica都无法“活”过来了，或者数据都丢失了，这个Partition将永远不可用。    - 选择第一个“活”过来的Replica作为Leader，而这个Replica不是ISR中的Replica，那即使它并不保证已经包含了所有已commit的消息，它也会成为Leader而作为consumer的数据源（前文有说明，所有读写都由Leader完成）。    - Kafka0.8.*使用了第二种方式。Kafka支持用户通过配置选择这两种方式中的一种，从而根据不同的使用场景选择高可用性还是强一致性

