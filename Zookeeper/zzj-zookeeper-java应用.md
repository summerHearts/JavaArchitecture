
# java应用zookeeper

## 1. java原生api


## 2. zkClient
pom.xml导入如下的包
```xml
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.7</version>
</dependency>
```
创建，删除，订阅等功能，接近原生api。


## 3. curator       
Curator 本身是 Netflix 公司开源的zookeeper客户端；     
curator提供了各种应用场景的实现封装       
curator-framework  提供了fluent风格api（链式操作，build）        
curator-replice     提供了实现封装          



### 3.1 curator连接的重试策略
ExponentialBackoffRetry()  衰减重试 
RetryNTimes 指定最大重试次数
RetryOneTime 仅重试一次
RetryUnitilElapsed 一直重试知道规定的时间

### 3.2 curator独有的事务操作       
### 3.3 curator的三种watcher监听        


## 4. zookeeper应用     
订阅发布
>watcher机制
>统一配置管理（disconf）

分布式锁
>redis  
zookeeper
数据库   

负载均衡
ID生成器
分布式队列
统一命名服务
master选举


### 4.1 订阅发布/配置中心       
实现配置信息的集中式管理和数据的动态更新

实现配置中心有两种模式：push 、pull。

zookeeper采用的是推拉相结合的方式。 客户端向服务器端注册自己需要关注的节点。一旦节点数据发生变化，那么服务器端就会向客户端
发送watcher事件通知。客户端收到通知后，主动到服务器端获取更新后的数据

1.	数据量比较小
2.	数据内容在运行时会发生动态变更
3.	集群中的各个机器共享配置


### 4.2 分布式锁
通常实现分布式锁有几种方式
1.	redis。 setNX 存在则会返回0， 不存在
2.	数据方式去实现
创建一个表， 通过索引唯一的方式
create table (id , methodname …)   methodname增加唯一索引
insert 一条数据XXX   delete 语句删除这条记录
mysql  for update
3.	zookeeper实现
排他锁
共享锁（读锁）

