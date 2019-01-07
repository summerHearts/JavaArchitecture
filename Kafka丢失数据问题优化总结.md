##1、Kafka丢失数据问题优化总结
- 数据丢失是一件非常严重的事情事，针对数据丢失的问题我们需要有明确的思路来确定问题所在，针对这段时间的总结，我个人面对kafka 数据丢失问题的解决思路如下：


  - 1、是否真正的存在数据丢失问题，比如有很多时候可能是其他同事操作了测试环境，所以首先确保数据没有第三方干扰。

  - 2、理清你的业务流程，数据流向，数据到底是在什么地方丢失的数据，在kafka 之前的环节或者kafka之后的流程丢失？比如kafka的数据是由flume提供的，也许是flume丢失了数据，kafka 自然就没有这一部分数据。

  - 3、如何发现有数据丢失，又是如何验证的。从业务角度考虑，例如：教育行业，每年高考后数据量巨大，但是却反常的比高考前还少，或者源端数据量和目的端数据量不符

  - 4、定位数据是否在kafka之前就已经丢失还事消费端丢失数据的

      -   kafka支持数据的重新回放功能(换个消费group)，清空目的端所有数据，重新消费。
    
      -   如果是在消费端丢失数据，那么多次消费结果完全一模一样的几率很低。
      
      -   如果是在写入端丢失数据，那么每次结果应该完全一样(在写入端没有问题的前提下)。

##2、kafka环节丢失数据，常见的kafka环节丢失数据的原因有：
  -  1、如果auto.commit.enable=true，当consumer fetch了一些数据但还没有完全处理掉的时候，刚好到commit interval出发了提交offset操作，接着consumer crash掉了。这时已经fetch的数据还没有处理完成但已经被commit掉，因此没有机会再次被处理，数据丢失。

  - 2、网络负载很高或者磁盘很忙写入失败的情况下，没有自动重试重发消息。没有做限速处理，超出了网络带宽限速。kafka一定要配置上消息重试的机制，并且重试的时间间隔一定要长一些，默认1秒钟并不符合生产环境（网络中断时间有可能超过1秒）。

  - 3、如果磁盘坏了，会丢失已经落盘的数据

  - 4、单批数据的长度超过限制会丢失数据，报kafka.common.MessageSizeTooLargeException异常
解决：

  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-07-191_14-50-17.png)

   
 - 5、partition leader在未完成副本数follows的备份时就宕机的情况，即使选举出了新的leader但是已经push的数据因为未备份就丢失了！
     - kafka是多副本的，当你配置了同步复制之后。多个副本的数据都在PageCache里面，出现多个副本同时挂掉的概率比1个副本挂掉的概率就很小了。（官方推荐是通过副本来保证数据的完整性的）

- kafka的数据一开始就是存储在PageCache上的，定期flush到磁盘上的，也就是说，不是每个消息都被存储在磁盘了，如果出现断电或者机器故障等，PageCache上的数据就丢失了。
     - 可以通过log.flush.interval.messages和log.flush.interval.ms来配置flush间隔，interval大丢的数据多些，小会影响性能但在0.8版本，可以通过replica机制保证数据不丢，代价就是需要更多资源，尤其是磁盘资源，kafka当前支持GZip和Snappy压缩，来缓解这个问题 是否使用replica取决于在可靠性和资源代价之间的balance

同时kafka也提供了相关的配置参数，来让你在性能与可靠性之间权衡（一般默认）：

 - 当达到下面的消息数量时，会将数据flush到日志文件中。默认10000

     ```
    log.flush.interval.messages=10000
     ```

- 当达到下面的时间(ms)时，执行一次强制的flush操作。interval.ms和interval.messages无论哪个达到，都会flush。默认3000ms

  ```
  log.flush.interval.ms=1000
  ```
- 检查是否需要将日志flush的时间间隔

  ```
  log.flush.scheduler.interval.ms = 3000
  ```

##3、Kafka的优化建议

- producer端：

   - 设计上保证数据的可靠安全性，依据分区数做好数据备份，设立副本数等。
        - push数据的方式：同步异步推送数据：权衡安全性和速度性的要求，选择相应的同步推送还是异步推送方式，当发现数据有问题时，可以改为同步来查找问题。

   - flush是kafka的内部机制,kafka优先在内存中完成数据的交换,然后将数据持久化到磁盘.kafka首先会把数据缓存(缓存到内存中)起来再批量flush.
      - 可以通过log.flush.interval.messages和log.flush.interval.ms来配置flush间隔

   - 可以通过replica机制保证数据不丢.
      - 代价就是需要更多资源,尤其是磁盘资源,kafka当前支持GZip和Snappy压缩,来缓解这个问题
是否使用replica(副本)取决于在可靠性和资源代价之间的balance(平衡)

   - broker到 Consumer kafka的consumer提供两种接口.
       - high-level版本已经封装了对partition和offset的管理，默认是会定期自动commit offset，这样可能会丢数据的

       - low-level版本自己管理spout线程和partition之间的对应关系和每个partition上的已消费的offset(定期写到zk)
并且只有当这个offset被ack后，即成功处理后，才会被更新到zk，所以基本是可以保证数据不丢的即使spout线程crash(崩溃)，重启后还是可以从zk中读到对应的offset

  - 异步要考虑到partition leader在未完成副本数follows的备份时就宕机的情况，即使选举出了新的leader但是已经push的数据因为未备份就丢失了！
      - 不能让内存的缓冲池太满，如果满了内存溢出，也就是说数据写入过快，kafka的缓冲池数据落盘速度太慢，这时肯定会造成数据丢失。
      - 尽量保证生产者端数据一直处于线程阻塞状态，这样一边写内存一边落盘。
      - 异步写入的话还可以设置类似flume回滚类型的batch数，即按照累计的消息数量，累计的时间间隔，累计的数据大小设置batch大小。
  - 设置合适的方式，增大batch 大小来减小网络IO和磁盘IO的请求，这是对于kafka效率的思考。
    - 不过异步写入丢失数据的情况还是难以控制
    - 还是得稳定整体集群架构的运行，特别是zookeeper，当然正对异步数据丢失的情况尽量保证broker端的稳定运作吧

   kafka不像hadoop更致力于处理大量级数据，kafka的消息队列更擅长于处理小数据。针对具体业务而言，若是源源不断的push大量的数据（eg：网络爬虫），可以考虑消息压缩。但是这也一定程度上对CPU造成了压力,还是得结合业务数据进行测试选择。


- broker端：

   - topic设置多分区，分区自适应所在机器，为了让各分区均匀分布在所在的broker中，分区数要大于broker数。分区是kafka进行并行读写的单位，是提升kafka速度的关键。

      - 1、broker能接收消息的最大字节数的设置一定要比消费端能消费的最大字节数要小，否则broker就会因为消费端无法使用这个消息而挂起。

      - 2、broker可赋值的消息的最大字节数设置一定要比能接受的最大字节数大，否则broker就会因为数据量的问题无法复制副本，导致数据丢失


- comsumer端：

   - 关闭自动更新offset，等到数据被处理后再手动跟新offset。
   
   - 在消费前做验证前拿取的数据是否是接着上回消费的数据，不正确则return先行处理排错。
   - 一般来说zookeeper只要稳定的情况下记录的offset是没有问题，除非是多个consumer group 同时消费一个分区的数据，其中一个先提交了，另一个就丢失了。

  **常见的配置**

    ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-07-191_15-08-37.png)

##4、Kafka重复消费原因

- 强行kill线程，导致消费后的数据，offset没有提交，partition就断开连接。比如，通常会遇到消费的数据，处理很耗时，导致超过了Kafka的session timeout时间（0.10.x版本默认是30秒），那么就会re-blance重平衡，此时有一定几率offset没提交，会导致重平衡后重复消费。

- 如果在close之前调用了consumer.unsubscribe()则有可能部分offset没提交，下次重启会重复消费

- kafka数据重复 kafka设计的时候是设计了(at-least once)至少一次的逻辑，这样就决定了数据可能是重复的，kafka采用基于时间的SLA(服务水平保证)，消息保存一定时间（通常为7天）后会被删除
kafka的数据重复一般情况下应该在消费者端,这时log.cleanup.policy = delete使用定期删除机制


