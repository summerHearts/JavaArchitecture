##RocketMQ消息中间件

Apache RocketMQ是一款具有低延迟，高性能和可靠性，数十亿容量和灵活可扩展性的分布式消息传递和流媒体平台。它由四部分组成：Name Servers，brokers，producers和consumers。 它们中的每一个都可以在没有单点故障的情况下进行水平扩展。

![](https://upload-images.jianshu.io/upload_images/325120-5b3ef079eaaea7ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

**NameServer集群：** Name Servers提供轻量级服务发现和路由。每个Name Server记录完整的路由信息，提供相应的读写服务，并支持快速存储扩展。

**Broker集群：**Brokers通过提供轻量级的TOPIC和QUEUE机制来实现消息存储。 它们支持Push和Pull模式，包含容错机制（2个或3个副本），并提供强大的峰值填充和按原始时间顺序累积数千亿条消息的能力。此外，broker提供灾难恢复，丰富的指标统计数据和警报机制，而传统的消息传递系统都缺乏这些机制。

![](https://upload-images.jianshu.io/upload_images/325120-d3f78545d292eda5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


如上图：Broker 服务器重要的子模块：
远程处理模块是 broker 的入口，处理来自客户的请求。
Client manager，管理客户（生产者/消费者）并维护消费者的主题订阅。
Store Service，提供简单的 API 来存储或查询物理磁盘中的消息。
HA 服务，提供主代理和从代理之间的数据同步功能。
索引服务，通过指定键为消息建立索引，并提供快速的消息查询。

**Producer集群：**Producer集群支持分布式部署。分布式producer通过多种负载均衡模式向Broker集群发送消息。发送过程支持fast failure并具有低延迟。

**Consumer集群：** Consumer也支持Push和Pull模型的分布式部署。 它还支持群集消费和消息广播。 它提供了实时的消息订阅机制，可以满足大多数消费者的需求。




