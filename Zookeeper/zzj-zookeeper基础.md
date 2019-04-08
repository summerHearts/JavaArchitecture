<!-- TOC -->

- [zookeeper](#zookeeper)
    - [1. 分布式协调技术](#1-%E5%88%86%E5%B8%83%E5%BC%8F%E5%8D%8F%E8%B0%83%E6%8A%80%E6%9C%AF)
        - [1.1 多进程调度](#11-%E5%A4%9A%E8%BF%9B%E7%A8%8B%E8%B0%83%E5%BA%A6)
        - [1.2 分布式锁的实现者](#12-%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E7%9A%84%E5%AE%9E%E7%8E%B0%E8%80%85)
    - [2. zookeeper](#2-zookeeper)
        - [2.1 zookeeper如何实现的？](#21-zookeeper%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%9A%84)
    - [3. ZooKeeper数据模型](#3-zookeeper%E6%95%B0%E6%8D%AE%E6%A8%A1%E5%9E%8B)
        - [3.1 ZooKeeper数据模型Znode](#31-zookeeper%E6%95%B0%E6%8D%AE%E6%A8%A1%E5%9E%8Bznode)
        - [3.2 Zookeeper中的时间](#32-zookeeper%E4%B8%AD%E7%9A%84%E6%97%B6%E9%97%B4)
        - [3.3 ZooKeeper节点属性](#33-zookeeper%E8%8A%82%E7%82%B9%E5%B1%9E%E6%80%A7)
        - [3.4 zookeeker服务中的操作](#34-zookeeker%E6%9C%8D%E5%8A%A1%E4%B8%AD%E7%9A%84%E6%93%8D%E4%BD%9C)
        - [3.5 watch触发器](#35-watch%E8%A7%A6%E5%8F%91%E5%99%A8)
            - [zookeeper监听(Watch触发条件)](#zookeeper%E7%9B%91%E5%90%ACwatch%E8%A7%A6%E5%8F%91%E6%9D%A1%E4%BB%B6)
    - [4.Zookeeper应用举例](#4zookeeper%E5%BA%94%E7%94%A8%E4%B8%BE%E4%BE%8B)
        - [4.1 分布式所应用场景](#41-%E5%88%86%E5%B8%83%E5%BC%8F%E6%89%80%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF)
        - [zookeeper解决方案](#zookeeper%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)
            - [1.master启动](#1master%E5%90%AF%E5%8A%A8)
            - [2.master故障](#2master%E6%95%85%E9%9A%9C)
            - [3.master恢复](#3master%E6%81%A2%E5%A4%8D)

<!-- /TOC -->


# zookeeper

[ZooKeeper学习第一期---Zookeeper简单介绍](https://www.cnblogs.com/sunddenly/p/4033574.html)         
[Paxos算法细节详解(一)--通过现实世界描述算法](https://www.cnblogs.com/endsock/p/3480093.html)       
[分布式系统Paxos算法](https://www.jdon.com/artichect/paxos.html)            
[分布式系统的Raft算法](https://www.jdon.com/artichect/raft.html)    

## 1. 分布式协调技术        
**布式协调技术**主要用来解决分布式环境当中**多个进程**之间的同步控制，让他们有序的去访问某种临界资源，防止造成"脏数据"的后果。      


### 1.1 多进程调度
分布式系统中如何对进程进行调度，我假设在第一台机器上挂载了一个资源，然后这三个物理分布的进程都要竞争这个资源，但我们又不希望他们同时进行访问，这时候我们就需要一个**协调器**，来让他们有序的来访问这个资源。这个协调器就是我们经常提到的那个**锁**，比如说"进程-1"在使用该资源的时候，会先去获得锁，"进程1"获得锁以后会对该资源保持独占，这样其他进程就无法访问该资源，"进程1"用完该资源以后就将锁释放掉，让其他进程来获得锁，那么通过这个锁机制，我们就能保证了分布式系统中多个进程能够有序的访问该临界资源。那么我们把这个分布式环境下的这个锁叫作**分布式锁**。这个分布式锁也就是我们分布式协调技术实现的核心内容。     

### 1.2 分布式锁的实现者
目前，在分布式协调技术方面做得比较好的就是Google的Chubby还有Apache的ZooKeeper他们都是分布式锁的实现者。有人会问既然有了Chubby为什么还要弄一个ZooKeeper，难道Chubby做得不够好吗？不是这样的，主要是Chbby是非开源的，Google自家用。       
后来雅虎模仿Chubby开发出了ZooKeeper，也实现了类似的分布式锁的功能，并且将ZooKeeper作为一种开源的程序捐献给了Apache，那么这样就可以使用ZooKeeper所提供锁服务。而且在分布式领域久经考验，它的可靠性，可用性都是经过理论和实践的验证的。

## 2. zookeeper
ZooKeeper是一种为分布式应用所设计的高可用、高性能且一致的开源**协调服务**，它提供了一项基本服务：**分布式锁**服务。

由于ZooKeeper的开源特性，后来我们的开发者在分布式锁的基础上，摸索了出了其他的使用方法：**配置维护、组服务、分布式消息队列、分布式通知/协调**等。        


>注意：ZooKeeper性能上的特点决定了它能够用在大型的、分布式的系统当中。从可靠性方面来说，它并不会因为一个节点的错误而崩溃。      
除此之外，它严格的序列访问控制意味着复杂的控制原语可以应用在客户端上。      
ZooKeeper在一致性、可用性、容错性的保证，也是ZooKeeper的成功之处，它获得的一切成功都与它采用的协议——Zab协议是密不可分的。       


### 2.1 zookeeper如何实现的？
ZooKeeper在实现这些服务时，     
首先它设计一种新的**数据结构**——Znode，         
然后在该数据结构的基础上定义了一些**原语**，也就是一些关于该数据结构的一些操作。有了这些数据结构和原语还不够，因为我们的ZooKeeper是工作在一个分布式的环境下，       
服务是通过消息以网络的形式发送给我们的分布式应用程序，所以还需要一个**通知机制**——Watcher机制。        

```
zookeeper = znode + 原语 + watcher      
```

## 3. ZooKeeper数据模型
### 3.1 ZooKeeper数据模型Znode      
ZooKeeper拥有一个层次的命名空间，这个和标准的文件系统非常相似，如下图:      

[![zookeeper-datamodel.png](https://s20.postimg.cc/tjpqckm4d/zookeeper-datamodel.png)](https://postimg.cc/image/yid8r3px5/)         


**引用方式**        
Zonde通过路径引用，如同Unix中的文件路径。路径必须是绝对的，因此他们必须由斜杠字符来开头。除此以外，他们必须是唯一的，也就是说每一个路径只有一个表示，因此这些路径不能改变。在ZooKeeper中，路径由Unicode字符串组成，并且有一些限制。字符串"/zookeeper"用以保存管理信息，比如关键配额信息。           

**Znode结构**           
ZooKeeper命名空间中的Znode，兼具文件和目录两种特点。既像文件一样维护着数据、元信息、ACL、时间戳等数据结构，又像目录一样可以作为路径标识的一部分。图中的每个节点称为一个Znode。 每个Znode由3部分组成:

① stat：此为状态信息, 描述该Znode的版本, 权限等信息

② data：与该Znode关联的数据

③ children：该Znode下的子节点

ZooKeeper虽然可以关联一些数据，但并没有被设计为常规的数据库或者大数据存储，相反的是，它用来管理调度数据，比如分布式应用中的配置文件信息、状态信息、汇集位置等等。这些数据的共同特性就是它们都是很小的数据，通常以KB为大小单位。ZooKeeper的服务器和客户端都被设计为严格检查并限制每个Znode的数据大小至多1M，但常规使用中应该远小于此值。     


**数据访问**            
ZooKeeper中的每个节点存储的数据要被原子性的操作。也就是说读操作将获取与节点相关的所有数据，写操作也将替换掉节点的所有数据。另外，每一个节点都拥有自己的ACL(访问控制列表)，这个列表规定了用户的权限，即限定了特定用户对目标节点可以执行的操作。      

**节点类型**            
ZooKeeper中的节点有两种，分别为临时节点和永久节点。节点的类型在创建时即被确定，并且不能改变。

① 临时节点：该节点的生命周期依赖于创建它们的会话。一旦会话(Session)结束，临时节点将被自动删除，当然可以也可以手动删除。虽然每个临时的Znode都会绑定到一个客户端会话，但他们对所有的客户端还是可见的。另外，ZooKeeper的临时节点不允许拥有子节点。

② 永久节点：该节点的生命周期不依赖于会话，并且只有在客户端显示执行删除操作的时候，他们才能被删除。


**顺序节点**            
当创建Znode的时候，用户可以请求在ZooKeeper的路径结尾添加一个递增的计数。这个计数对于此节点的父节点来说是唯一的，它的格式为"%10d"(10位数字，没有数值的数位用0补充，例如"0000000001")。当计数值大于(2^32)-1时，计数器将溢出。            

**观察**        
客户端可以在节点上设置watch，我们称之为监视器。当节点状态发生改变时(Znode的增、删、改)将会触发watch所对应的操作。当watch被触发时，ZooKeeper将会向客户端发送且仅发送一条通知，因为watch只能被触发一次，这样可以减少网络流量。

### 3.2 Zookeeper中的时间       
ZooKeeper有多种记录时间的形式，其中包含以下几个主要属性：

- 1.Zxid
致使ZooKeeper节点状态改变的每一个操作都将使节点接收到一个Zxid格式的时间戳，并且这个时间戳全局有序。也就是说，也就是说，每个对节点的改变都将产生一个唯一的Zxid。如果Zxid1的值小于Zxid2的值，那么Zxid1所对应的事件发生在Zxid2所对应的事件之前。实际上，ZooKeeper的每个节点维护者三个Zxid值，为别为：cZxid、mZxid、pZxid。
① cZxid： 是节点的创建时间所对应的Zxid格式时间戳。
② mZxid：是节点的修改时间所对应的Zxid格式时间戳。

实现中Zxid是一个64为的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个 新的epoch。低32位是个递增计数。     

- 2.版本号
对节点的每一个操作都将致使这个节点的版本号增加。每个节点维护着三个版本号，他们分别为：
① version：节点数据版本号
② cversion：子节点版本号
③ aversion：节点所拥有的ACL版本号

### 3.3 ZooKeeper节点属性
Znode节点属性结构

属性 | 描述 | 
---|----|-
czxid | 节点被创建的zxid
mzxid | 节点被修改的zxid
ctime | 节点被创建的时间
mtime | 节点被修的的时间
version | 节点的版本号
cversion | 子节点版本号
aversion | 节点所拥有的ACL版本号
ephemeralOwner | 如果此节点为临时节点，那么他的值为这个节点拥有者的会话ID;否则，他的值为0 | 
dataLength | 节点用的子节点长度 | 
pzxid | 最新修改的zxid | 

### 3.4 zookeeker服务中的操作       
zookeeper 类方法描述        

操作 | 描述
---|---
create | 创建znode | 
delete | 删除znode
exits | 测试znode是否存在，并获取他的元数据 | 
getACL/setACL | 为znode获取、设置ACL | 
getChildren | 获取znode所有的子节点的列表 | 
getData/setData | 获取、设置znode的相关数据
sync | 使客户端znode视图与zookeeper同步 | 


### 3.5 watch触发器         
**watch概述**           
ZooKeeper可以为所有的**读操作设置watch**，这些读操作包括：exists()、getChildren()及getData()。watch事件是**一次性**的触发器，当watch的对象状态发生改变时，将会触发此对象上watch所对应的事件。watch事件将被异步地发送给客户端，并且ZooKeeper为watch机制提供了**有序的一致性保证**。理论上，客户端接收watch事件的时间要**快于**其看到watch对象状态变化的时间。       

**watch类型**           
zookeeper管理的watch可以分为两类：      
- 数据watch(data watch)： getData 和  exits 负责设置数据 watch      
- 孩子watch(child watch):   getChildren 负责设置孩子watch       
  
可以通过操作**返回的数据** 来设置不同的watch:           
① getData和exists：返回关于节点的数据信息       
② getChildren：返回孩子列表     
因此        
① 一个成功的setData操作将触发Znode的数据watch       
② 一个成功的create操作将触发Znode的数据watch以及孩子watch       
③ 一个成功的delete操作将触发Znode的数据watch以及孩子watch       


**watch注册与触发器**       
- 1.exits操作上的watch,在被监视的znode**创建**、**删除**或数据**更新**时触发
- 2.getData操作上的watch,在被监视的znode**删除**或数据**更新**时被触发，在被创建时不能被触发，因为只有Znode一定存在，getData操作才会成功。  
- 3.getChildren操作上的watch，在被监视的Znode的子节点**创建或删除**，或是这个Znode自身被**删除**时被触发。

[![watch.png](https://s22.postimg.cc/lnp6rcqlt/watch.png)](https://postimg.cc/image/c35k4h19p/)

**注意的几点**          
Zookeeper的watch实际上要处理两类事件：      
- 1.连接状态事件(type=None, path=null)
这类事件不需要注册，也不需要我们连续触发，我们只要处理就行了。
- 2.节点事件
节点的建立，删除，数据的修改。它是one time trigger，我们需要不停的注册触发，还可能发生事件丢失的情况。

上面2类事件都在Watch中处理，也就是重载的`process(Event event)`      

节点事件的触发，通过函数exists，getData或getChildren来处理这类函数，有双重作用：
- 1.注册触发事件        
- 2.函数本身的功能      
  
函数的本身的功能又可以用异步的回调函数来实现,重载processResult()过程中处理函数本身的的功能。        



#### zookeeper监听(Watch触发条件)       
**连接状态监听**            

状态 | 说明 | 
---|----|-
KeeperStat.Expired | 在一定时间内客户端没有收到服务器的通知， 则认为当前的会话已经过期了。
KeeperStat.Disconnected | 断开连接的状态
KeeperStat.SyncConnected | 客户端和服务器端在某一个节点上建立连接，并且完成一次version、zxid同步
KeeperStat.authFailed | 授权失败


**事件监听**     

状态 | 说明
---|---
NodeCreated | 当节点被创建的时候，触发
NodeChildrenChanged | 表示子节点被创建、被删除、子节点数据发生变化
NodeDataChanged | 节点数据发生变化
NodeDeleted | 节点被删除
None | 客户端和服务器端连接状态发生变化的时候，事件类型就是None



## 4.Zookeeper应用举例      
**分布式所**        

### 4.1 分布式所应用场景        
在分布式锁服务中，有一种最典型应用场景，就是通过对集群进行Master选举，来解决分布式系统中的单点故障。

**单点故障**            
什么是分布式系统中的单点故障：通常分布式系统采用主从模式，就是一个主控机连接多个处理节点。主节点负责分发任务，从节点负责处理任务，当我们的主节点发生故障时，那么整个系统就都瘫痪了，那么我们把这种故障叫作单点故障。

[![image.png](https://s22.postimg.cc/6rqnjqs1t/image.png)](https://postimg.cc/image/z4m5a7drx/)     [![image.png](https://s22.postimg.cc/z4m5a7lht/image.png)](https://postimg.cc/image/j6dfk2r9p/)

### zookeeper解决方案       
#### 1.master启动       
在引入了Zookeeper以后我们启动了两个主节点，"主节点-A"和"主节点-B"他们启动以后，**都向ZooKeeper去注册一个节点**。我们假设"主节点-A"锁注册地节点是"master-00001"，"主节点-B"注册的节点是"master-00002"，**注册完以后进行选举**，编号最小的节点将在选举中获胜获得锁成为主节点，也就是我们的"主节点-A"将会获得锁成为**主节点**，然后"主节点-B"将被阻塞成为一个**备用节点**。那么，通过这种方式就完成了对两个Master进程的调度。      
[![mater.png](https://s22.postimg.cc/bqe5yaba9/mater.png)](https://postimg.cc/image/klf08t02l/)


#### 2.master故障       
如果"主节点-A"挂了，这时候他所注册的节点将被自动删除，ZooKeeper会自动感知节点的变化，然后再次发出选举，这时候"主节点-B"将在选举中获胜，替代"主节点-A"成为主节点。           
[![zookeeper-master-select.png](https://s22.postimg.cc/6ez9djuch/zookeeper-master-select.png)](https://postimg.cc/image/ao3zfpxlp/)

#### 3.master恢复
如果主节点恢复了，他会再次向ZooKeeper注册一个节点，这时候他注册的节点将会是"master-00003"，ZooKeeper会感知节点的变化再次发动选举，这时候"主节点-B"在选举中会再次获胜继续担任"主节点"，"主节点-A"会担任备用节点。
[![zookeeper-master-resume.png](https://s22.postimg.cc/4n6ain39t/zookeeper-master-resume.png)](https://postimg.cc/image/lnp6rbgb1/)








