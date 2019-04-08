## 1、ZAB简介
 - 1、zab协议是为分布式协调服务zookpeer专门设计的一种支持崩溃恢复的原子广播协议。

 - 2、在zookeeper中主要依赖ZAB协议来实现数据一致性，基于该协议zk实现了一种主备模式的系统架构来保证集群中各个副本之间数据的一致性。具体就是zk使用一个单一的主进程来接收并处理客户端的事务请求（就是写请求），并采用ZAB的原子广播协议，将服务器数据的状态变更以事务proposal的形式广播到所有的副本进程上去。
## 2、事务请求的处理方式
- 所有的事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被称为Leader服务器，而余下的其他服务器则是Follow服务器。Leader负责将一个事务请求转换成一个事务proposal，并将该proposal分发给集群中所有的Follow，之后Leader需要等待所有的Follow的反馈，一单超过半数Follow进行了正确的反馈后，那么Leader就会再次向所有的Follow发送commit消息，要求其将前一个proposal进行提交
- 注意：如果集群中非Leader服务器接收到事务请求，这些非Leader服务器会将事务请求转发给Leader服务器，只有Leader才能处理事务请求。

## 3、协议具体内容
- ZAB协议包括两种基本模式：分别是崩溃恢复和消息广播。

- 当整个集群启动过程中或者当Leader服务器出现网络中断，崩溃退出或重启等异常时，ZAB协议j就会进入恢复模式并选举产生新的Leader，当选举产生了新的Leader，同时集群中有过半的机器与该Leader服务器完成了状态同步（即数据同步）之后，ZAB协议就会退出崩溃恢复模式，进入消息广播模式。这时如果有一台遵守ZAB协议的服务器加入集群，因为此时集群中已经存在一个Leader服务器在广播消息，那么新加入的服务器自觉的进入恢复模式：找到Leader服务器并与之完成数据同步然后一起参与到消息广播流程中去。

- 当Leader出现崩溃退出或者机器重启，亦或是集群中已经不存在过半的服务器与Leader保持正常的通信，ZAB就会重新发一轮Leader选举并实现数据同步，最后又进入消息广播模式，接收事务请求。

- 消息必须是有顺序的
   - 在整个消息广播过程中，Leader会将每个事务请求转换成对应的proposal来进行广播，并且在广播事务proposal之前，Leader服务器会首先为这个事务proposal分配一个全局的单调递增的唯一ID，称之为事务ID（即ZXID），由于ZAB需要保证每一个消息严格的因果关系，因此必须将每一个proposal按照其ZXID的先后顺序来进行排序与处理。

- 消息广播
  - ZAB协议的消息广播使用的是一个原子广播协议，类似于二阶段提交，Leader接收事务请求，并转换成proposal广播给其他的Follow，然后过半的Follow ack消息，最后再给广播commit消息完成事务提交。
  - 具体的，在消息广播过程中，Leader为每个Follow服务器分配一个单独的队列，然后将需要广播的proposal依次放到队列中去，并且根据FIFO策略进行消息发送。每一个Follow接收到proposal后，都会首先将其以事务日志的形式写入本地磁盘中，并且写入成功后反馈Leader一个ACK响应。当Leader接收到超过半数的ACK响应后，就会广播一个commit消息给Follow已通知他们完成事务提交，同时Leader自身也会完成事务的提交。

- 崩溃恢复
  - Leader挂了之后，ZAB协议就自动进入崩溃恢复模式，选举出新的Leader，并完成数据同步，然后退出崩溃恢复模式进入消息广播模式。

- ZAB协议如何保证数据一致性
   - 异常情况：
     - 1、假设一个事务在Leader上提交了，并且过半Follow都响应ACK了，但是Leader将commit消息发出后就挂了
     - 2、假设一个事务在Leader提交了之后，Leader就挂掉了。
     - 要保证如果发生上述2种情况，数据还能保持一致性，那么ZAB协议选举算法必须确保已经提交的proposal（发送过commit消息），在Follow上也必须完成提交；并且丢弃已经被跳过的事务proposal。

  - 通过ZAB协议选举算法选举出来的Leader必须是拥有集群中最高编号（ZXID）proposal的机器，拥有最高编号说明新Leader一定具有所有已提交的提案。更为重要的是，如果让具有最高编号的事务proposal的机器成为Leader，就可以省去Leader服务器检查proposal的提交和丢弃工作的这一步操作了。

- ZAB是如何数据同步？
     - 完成Leader选举后（新Leader具有最高编号），在正式开始工作之前（接收事务请求，然后提出新的proposal）,Leader服务器会首先确认事务日志中的所有的Proposal是否都已经被集群中过半的提交了。

     - Leader服务器需要确保所有的Follow服务器能够接收到每一条事务proposal, 并且能将所有已经提交的事务proposal应用到内存数据库中去。等到Follow将所有尚未同步的事务proposal都从Leader服务器上同步过来并应用到内存数据库中去，Leader才会把该Follow加入到真正可用的Follow列表中。

 - ZAB是如何处理需要丢弃的Proposal？
     - 在ZAB协议的事务编号ZXID设计中，ZXID是一个64位的数字，其中低32位可以看作成一个简单的单调递增计数器了，针对客户端每一个事务请求，Leader在产生新的事务Proposal时，都会对该计数器加1，而高32位则代表了Leader周期的epoch编号（可以理解为选举的届期），每当选举产生一个新的Leader，就会从这个Leader服务器上取出本地事务日志中最大的编号proposal的ZXID，并从ZXID中解析得到对应epoch编号，然后再对其进行加1，之后就以此编号作为新的epoch值，并将地32位置为0开始生成新的ZXID，ZAB协议通过epoch编号来区分Leader变化周期，能够有效的避免了不同的Leader错误的使用了相同的ZXID编号提出了不一样的proposal的异常情况。

     - 基于这样的策略，当一个包含了上一个Leader周期中尚未提交过的事务proposal的服务器启动时，当这台机器加入集群中，以Follow角色连接上Leader服务器后，Leader服务器会根据自己服务器上最后提交的proposal来和Follow服务器的proposal进行比对，比对的结果肯定是Leader要求Follow进行一个回退操作，回退到一个确实已经被集群中过半机器提交的最新proposal。
- ZAB协议特性：

     - 1、ZAB协议需要确保那些已经在Leader服务器上提交（commit）的事务最终被所有的服务器提交 
　 
     - 2、ZAB协议需要确保丢弃那些只在Leader上被提出的事务(只是被提出还没有被提交)

## 4、实际情况分析
 -  Zookeeper 客户端会随机连接到 Zookeeper 集群的一个节点，如果是读请求，就直接从当前节点中读取数据；如果是写请求，那么节点就会向 leader 提交事务，leader 会广播事务，只要有超过半数节点写入成功，该写请求就会被提交（类 2PC 协议）。

  问题如下：

  - 1、主从架构下，leader 崩溃，数据一致性怎么保证？
  - 2、选举 leader 的时候，整个集群无法处理写请求的，如何快速进行 leader 选举？
   带着这两个问题，我们来看看 ZAB 协议是如何解决的。
  - 在分析流程之前，我们先来了解一下几个基本的知识点，方便后边的理解。
     **1、ZAB 中的节点有三种状态**
     
        -  following：当前节点是跟随者，服从 leader 节点的命令
        -  leading：当前节点是 leader，负责协调事务
        -  election/looking：节点处于选举状态
 
      代码实现中多了一种：observing 状态，这是 Zookeeper 引入 Observer 之后加入的，Observer 不参与选举，是只读节点，跟 ZAB 协议没有关系。
      
     **2、节点的持久状态**
    
      - history：当前节点接收到事务提议的 log
      - acceptedEpoch：follower 已经接受的 leader 更改年号的          NEWEPOCH 提议
      - currentEpoch：当前所处的年代
      - lastZxid：history 中最近接收到的提议的 zxid （最大的）

      - 在 ZAB 协议的事务编号 Zxid 设计中，Zxid 是一个 64 位的数字，其中低 32 位是一个简单的单调递增的计数器，针对客户端每一个事务请求，计数器加 1；而高 32 位则代表 Leader 周期 epoch 的编号，每个当选产生一个新的 Leader 服务器，就会从这个 Leader 服务器上取出其本地日志中最大事务的ZXID，并从中读取 epoch 值，然后加 1，以此作为新的 epoch，并将低 32 位从 0 开始计数。
         
      - epoch：可以理解为当前集群所处的年代或者周期，每个 leader 就像皇帝，都有自己的年号，所以每次改朝换代，leader 变更之后，都会在前一个年代的基础上加 1。这样就算旧的 leader 崩溃恢复之后，也没有人听他的了，因为 follower 只听从当前年代的 leader 的命令。


  -   3、 ZAB 的四个阶段
      -  Leader election（选举阶段）

      ![](http://upload-images.jianshu.io/upload_images/325120-8b94c3b0c925dbc4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

       - 节点在一开始都处于选举阶段，只要有一个节点得到超半数节点的票数，它就可以当选准 leader。只有到达 Phase 3 准 leader 才会成为真正的 leader。这一阶段的目的是就是为了选出一个准 leader，然后进入下一个阶段。 协议并没有规定详细的选举算法，后面我们会提到实现中使用的 Fast Leader Election。
       
       - 2、Discovery（发现阶段）
        
      ![](http://upload-images.jianshu.io/upload_images/325120-f746fb71e4c2efba?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
        -  在这个阶段，followers 跟准 leader 进行通信，同步 followers 最近接收的事务提议。这个一阶段的主要目的是发现当前大多数节点接收的最新提议，并且准 leader 生成新的 epoch，让 followers 接受，更新它们的 acceptedEpoch。
         - 一个 follower 只会连接一个 leader，如果有一个节点 f 认为另一个 follower p 是 leader，f 在尝试连接 p 时会被拒绝，f 被拒绝之后，就会进入Leader election。
       
       - 3、Synchronization（同步阶段）
   
      ![](http://upload-images.jianshu.io/upload_images/325120-5362b9bcea4859f5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
          - 同步阶段主要是利用 leader 前一阶段获得的最新提议历史，同步集群中所有的副本。只有当 quorum 都同步完成，准 leader 才会成为真正的 leader。follower 只会接收 zxid 比自己的 lastZxid 大的提议。
        
       - 4、Broadcast（广播阶段）

      ![](http://upload-images.jianshu.io/upload_images/325120-0cc90896f5d61102?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

         - 到了这个阶段，Zookeeper 集群才能正式对外提供事务服务，并且 leader 可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。
         - 值得注意的是，ZAB 提交事务并不像 2PC 一样需要全部 follower 都 ACK，只需要得到 quorum （超过半数的节点）的 ACK 就可以了。
     
- 4、协议实现
   - 协议的 Java 版本实现跟上面的定义有些不同，选举阶段使用的是 Fast Leader Election（FLE），它包含了 Phase 1 的发现职责。因为 FLE 会选举拥有最新提议历史的节点作为 leader，这样就省去了发现最新提议的步骤。实际的实现将 Phase 1 和 Phase 2 合并为 Recovery Phase（恢复阶段）。所以，ZAB 的实现只有三个阶段：
      - Fast Leader Election
      - Recovery Phase
      - Broadcast Phase
  - Fast Leader Election
      - 前面提到 FLE 会选举拥有最新提议历史（lastZixd最大）的节点作为 leader，这样就省去了发现最新提议的步骤。这是基于拥有最新提议的节点也有最新提交记录的前提。
      - 成为 leader 的条件
          - 选epoch最大的
          - epoch相等，选 zxid 最大的
          - epoch和zxid都相等，选择server id最大的（就是我们配置zoo.cfg中的myid）
      -  节点在选举开始都默认投票给自己，当接收其他节点的选票时，会根据上面的条件更改自己的选票并重新发送选票给其他节点，当有一个节点的得票超过半数，该节点会设置自己的状态为 leading，其他节点会设置自己的状态为 following。

     - 选举过程

   ![](http://upload-images.jianshu.io/upload_images/325120-5f1064170ba77fe0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

- Recovery Phase （恢复阶段）
    - 这一阶段 follower 发送它们的 lastZixd 给 leader，leader 根据 lastZixd 决定如何同步数据。这里的实现跟前面 Phase 2 有所不同：Follower 收到 TRUNC 指令会中止 L.lastCommittedZxid 之后的提议，收到 DIFF 指令会接收新的提议。
        - history.lastCommittedZxid：最近被提交的提议的 zxid
        - history:oldThreshold：被认为已经太旧的已提交提议的 zxid
   
  ![](http://upload-images.jianshu.io/upload_images/325120-6161a8841f76c51e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

- 总结
   - 主从架构下，leader 崩溃，数据一致性怎么保证？leader 崩溃之后，集群会选出新的 leader，然后就会进入恢复阶段，新的 leader 具有所有已经提交的提议，因此它会保证让 followers 同步已提交的提议，丢弃未提交的提议（以 leader 的记录为准），这就保证了整个集群的数据一致性。
   - 选举 leader 的时候，整个集群无法处理写请求的，如何快速进行 leader 选举？这是通过 Fast Leader Election 实现的，leader 的选举只需要超过半数的节点投票即可，这样不需要等待所有节点的选票，能够尽早选出 leader。


