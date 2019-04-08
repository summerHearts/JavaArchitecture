## 1、Zookeeper简介
  ![](https://upload-images.jianshu.io/upload_images/325120-c57e91504fb6272c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
- 1、ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

- 2、ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。
- 3、是一个中间件，提供协调服务。作用于分布式系统，发挥其优势，可为大数据服务。支持Java 和 C语言。
![](https://upload-images.jianshu.io/upload_images/325120-0e5799eb6316280c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
## 2、什么是分布式系统
![](https://upload-images.jianshu.io/upload_images/325120-f2bb4b0f789b998e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
![](https://upload-images.jianshu.io/upload_images/325120-d347784fd824be96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
## 3、分布式系统的瓶颈以及zk的相关特性

- 1、一致性：数据一致性，数据按顺序分批入库。

- 2、原子性：事务要么都成功，要么都失败，不会局部化。
- 3、单一视图：客户端连接集群中的任一zk节点，数据都是一致的。
- 4、可靠性：每次对zk的操作状态都保存在服务器中。
- 5、实时性：客户端可以读取zk服务端的最新数据。

## 4、Zookeeper中zoo.cfg配置
- 1、tickTime:用于计算的时间单元。比如session超时：N*tickTime.

- 2、initLimit:用于集群，允许从节点连接并同步到 master节点的初始化连接时间，以tickTime 的倍数来表示.
- 3、syncLimit:用于集群， master主节点与 从节点 之间发送消息，请求和应答 时间长度（心跳机制）
- 4、dataDir:必须配置。
- 5、dataLogDir:日志目录。
- 6、clientPort：连接服务器的端口，默认2181

## 5、Zookeeper基本数据类型

![](https://upload-images.jianshu.io/upload_images/325120-3cb5079669382f8b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- 1、是一个树型结构，类似前端开发中的tree.js组件。

- 2、每个节点称之为znode,它可以有子节点，也可以有数据。
- 3、每个节点分为临时节点和永久节点，临时节点会在客户端断开后消失。
- 4、每个zk节点都各自的版本号，可以通过命令行来显示节点信息。
- 5、每个节点数据发生变化，那么节点的版本号会累加(乐观锁)。
- 6、删除、修改过期节点，版本号不匹配则会报错。
- 6、每个zk节点存储的数据不宜过大，几K即可。
- 7、节点可以设置acl,可以通过权限来控制用户的访问。

## ## 6、zk的作用体现
 - 1、master节点选举，主节点挂了以后，从节点就会接手工作，并且保证这个节点是唯一的，这也就是所谓的首脑模式，从而保证我们的集群是高可用的。
 
 - 2、统一配置文件管理，即只需要部署一台服务器，则可以把相同的配置文件同步更新到其他所有服务器，此操作在云计算的特别多。
 ![](https://upload-images.jianshu.io/upload_images/325120-27b9777658b77b5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
- 3、发布与订阅，类似消息队列MQ（amq,rmq）,dubbo发布者把数据存在znode上，订阅者会读取这个数据。

- 4、提供分布式锁，分布式环境中不同进程之间争夺资源，类似多线程中的锁。

  ![](https://upload-images.jianshu.io/upload_images/325120-5e20f0ec3e6fafa7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
- 5、集群管理，集群中保证数据的强一致性。

  ![](https://upload-images.jianshu.io/upload_images/325120-1277457a6fdb405f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
## 7、Linux的ZK客户端命令行学习

```
ZooKeeper -server host:port cmd args
	stat path [watch]   //stat命令用于查看节点的状态信息
	set path data [version]  //set命令用于设置节点的数据
	ls path [watch]     //ls命令用于获取路径下的节点信息，注意路径为绝对路径
	delquota [-n|-b] path //delquota命令用于删除配额，-n为子节点个数，-b为节点数据长度
	ls2 path [watch]    //ls2命令是ls命令的增强版，比ls命令多输出本节点信息
	setAcl path acl     //　setAcl命令用于设置节点Acl,Acl由三部分构成：1为scheme，2为user，3为permission，一般情况下表示为scheme:id:permissions
	setquota -n|-b val path  //setquota命令用于设置节点个数以及数据长度的配额
	history             //history用于列出最近的命令历史，可以和redo配合使用
	redo cmdno          //redo命令用于再次执行某个命令
	printwatches on|off  //printWatchers命令用于设置和显示监视状态，值为on或则off
	delete path [version]  //delete命令用于删除节点，如delete /nodeDelete
	sync path           //sync命令用于强制同步，由于请求在半数以上的zk server上生效就表示此请求生效，那么就会有一些zk server上的数据是旧的。sync命令就是强制同步所有的更新操作。
	listquota path       //查看指定znode的配额
	rmr path             //递归删除
	get path [watch]     //get命令用于获取节点的信息，注意节点的路径必须是以/开头的绝对路径。如get /
	create [-s] [-e] path data acl   //create命令用于创建节点，其中-s为顺序充点，-e临时节点
	addauth scheme auth  //addauth命令用于节点认证，使用方式：如addauth digest username:password
	quit                 //退出客户端
	getAcl path          //获取节点的Acl，如getAcl /node1
	close                //close命令用于关闭与服务端的链接
	connect host:port    //连接zk服务端，与close命令配合使用可以连接或者断开zk服务端
```
返回信息的具体含义

```
    cZxid = 0x0       //节点创建时的zxid
    ctime = Thu Jan 01 08:00:00 CST 1970   //节点创建时间
    mZxid = 0x0       //节点最近一次更新时的zxid
    mtime = Thu Jan 01 08:00:00 CST 1970   //节点最近一次更新的时间
    pZxid = 0x2c      //子节点的id
    cversion = 10     //子节点数据更新次数
    dataVersion = 0   //本节点数据更新次数
    aclVersion = 0    //节点ACL(授权信息)的更新次数
    ephemeralOwner = 0x0  //如果该节点为临时节点,ephemeralOwner值表示与该节点绑定的session id. 如果该节点不是临时节点,ephemeralOwner值为0
    dataLength = 0    //节点数据长度，本例中为hello world的长度
    numChildren = 10  //子节点个数
```

## 8、Zookeeper-session的基本原理

 - 1、客户端与服务端之间的连接存在会话
 
 - 2、每个会话都可以设置一个超时时间
 - 3、心跳结束，session则过期
 - 4、session过期，则临时节点znode则会被抛弃
 - 5、心跳机制：客户端向服务端的ping包请求
 
## 9、Zookeeper-watcher机制
- 1、针对每个节点的操作，都会有一个监督者->wathcer。

- 2、当监控的某个对象(znode)发生了变化。则触发wathcer事件。
- 3、zk中的wathcer是一次性的，触发后立即销毁。
- 4、父节点、子节点 增删改查都能够触发其wathcer
- 5、针对不同类型的操作，触发的wathcer事件是不同的。子节点的创建事件，子节点的删除事件，子节点数据变化事件

- 6、ls 为父节点设置watcher,创建子节点触发：NodeChildChanged
- 7、ls 为父节点设置watcher，删除子节点触发：NodeChildChanged
- 8、ls 为父节点设置watcher，修改子节点不触发事件

## ## 10、watcher使用场景
- 1、主机更新节点为新的配置信息

![](https://upload-images.jianshu.io/upload_images/325120-943972fba3ba2b20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

## 11、ACL权限控制
- 1、针对节点可以设置相关的读写等权限，目的为了保证数据安全性。

- 2、权限*permissions*可以指定不同的权限范围以及角色
- 3、zk的acl通过[scheme:id:permissions]来构成权限列表
   - 1、*scheme* : 代表采用的某种权限机制
   - 2、*id*    : 代表允许访问的用户
   - 3、*permissions*: 权限组合字符串
   - 4、*world*: world下只有一个id,即只有一个用户，也就是anyone,那么组合的写法就是 world:anyone:[permissions]

   - 5、*auth*: 代表认证登录，需要注册用户有权限就可以，形式为auth:user:password:[permissions]
   - 6、*digest*:需要对密码加密才能访问，组合形式为digest:username:BASE64(SHA1(password)):[permissions]
   - 7、简而言之，*auth/digest*的区别就是，前者明文，后者密文。 `setAcl /path auth:lee:lee:cdrwa` 与  `setAcl /path digest:lee:BASE64(SHA1(password)):cdrwa`是等价的，在通过 `addauth digest lee:lee` 后都能操作指定节点的权限。
   - 8、*ip*:当设置为ip指定的ip地址，此时限制ip进行访问，比如 `ip:192.168.1.12:[permissions]`
   - 9、*super*:超级管理员，拥有所有的权限。
   
   
- 4、acl的构成-permissions
   - 1、`crdwa`权限字符串： *CREATE*:创建子节点； *READ*:获取节点/子节点；*WRITE*:设置节点数据；*DELETE*:删除子节点； *ADMIN*:设置权限。
   
   
- 5、 `word:anyone:cdrwa` 添加权限 /`auth:user:pwd:cdrwa` 注册 / `digest:lee:BASE64(SHA1(password)):cdrwa` 顾客访问 / `addauth digest lee:lee` 登录
- 6、*ip*: `setAcl /tapps/names/ip ip:192.168.103.214:cdrwa`
- 7、 *Super* : 1、修改zkServer.sh 增加super管理员；2、重启 zkServer.sh

- 8、ACL的常用使用场景

    - 开发/测试环境分离，开发者无权操作测试库的节点，只能看。
    
    - 生产环境上的控制指定ip的服务可以访问相关节点，防止混乱。

## ## 12、Zookeeper四字命令 Four Letter Words
- 1、zk可以通过它自身提供的简写命令来和服务器进行交互。
- 2、需要使用 nc命令，安装： `yum install nc`
- 3、`echo [command] | nc [ip] [port]`

 ![](https://upload-images.jianshu.io/upload_images/325120-8a6f9a37867f9270.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
- *stat*

```
echo stat | nc localhost 2181

Zookeeper version: 3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
Clients:
 /127.0.0.1:52506[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/32
Received: 1260
Sent: 1264
Connections: 1
Outstanding: 0
Zxid: 0x5a
Mode: standalone
Node count: 33
```
- *ruok*

```
echo ruok | nc localhost 2181

imok
```
- *dump*

```
echo dump | nc localhost 2181

SessionTracker dump:
Session Sets (0):
ephemeral nodes dump:
Sessions with Ephemerals (0):
```
- *conf*

```
echo conf | nc localhost 2181

clientPort=2181
dataDir=/developer/setup/zookeeper-3.4.10/data/version-2
dataLogDir=/developer/setup/zookeeper-3.4.10/logs/version-2
tickTime=2000
maxClientCnxns=60
minSessionTimeout=4000
maxSessionTimeout=40000
serverId=0
```
- *cons*

```
echo cons | nc localhost 2181

 /127.0.0.1:52570[0](queued=0,recved=1,sent=0)
```
- *envi*

```
echo envi | nc localhost 2181

Environment:
zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
host.name=localhost
java.version=1.8.0_161
java.vendor=Oracle Corporation
java.home=/usr/java/jdk1.8.0_161/jre
java.class.path=/developer/setup/zookeeper-3.4.10/bin/../build/classes:/developer/setup/zookeeper-3.4.10/bin/../build/lib/*.jar:/developer/setup/zookeeper-3.4.10/bin/../lib/slf4j-log4j12-1.6.1.jar:/developer/setup/zookeeper-3.4.10/bin/../lib/slf4j-api-1.6.1.jar:/developer/setup/zookeeper-3.4.10/bin/../lib/netty-3.10.5.Final.jar:/developer/setup/zookeeper-3.4.10/bin/../lib/log4j-1.2.16.jar:/developer/setup/zookeeper-3.4.10/bin/../lib/jline-0.9.94.jar:/developer/setup/zookeeper-3.4.10/bin/../zookeeper-3.4.10.jar:/developer/setup/zookeeper-3.4.10/bin/../src/java/lib/*.jar:/developer/setup/zookeeper-3.4.10/bin/../conf:.:/usr/java/jdk1.8.0_161/jre/lib/rt.jar:/usr/java/jdk1.8.0_161/lib/dt.jar:/usr/java/jdk1.8.0_161/lib/tools.jar
java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
java.io.tmpdir=/tmp
java.compiler=<NA>
os.name=Linux
os.arch=amd64
os.version=2.6.32-696.6.3.el6.x86_64
user.name=root
user.home=/root
user.dir=/developer/setup/zookeeper-3.4.10/bin
```
- *mntr*

```
echo mntr | nc localhost 2181

zk_version	3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
zk_avg_latency	0
zk_max_latency	32
zk_min_latency	0
zk_packets_received	1266
zk_packets_sent	1270
zk_num_alive_connections	1
zk_outstanding_requests	0
zk_server_state	standalone
zk_znode_count	33
zk_watch_count	0
zk_ephemerals_count	0
zk_approximate_data_size	766
zk_open_file_descriptor_count	31
zk_max_file_descriptor_count	65535
```
- *wchs* 

```
echo wchs | nc localhost 2181

0 connections watching 0 paths
Total watches:0
```


