## JVM垃圾回收
-  1、如何判定对象是垃圾对象
    -  1:   引用计数 
    -  2：可达性分析法
-   2、如何回收
   -  1、回收策略
       -  1:标记-清除算法
       -  2:复制算法
       -  3:标记-整理算法
       -  4:分代收集算法
   -  2、垃圾回收器
       -  1：Serial
       -  2:  Parnew
       -  3:  cms
       -  4: G1

## 引用计数器

- 简单的来说就是判断对象的引用数量。实现方式：给对象共添加一个引用计数器，每当有引用对他进行引用时，计数器的值就加1，当引用失效，也就是不在执行此对象是，他的计数器的值就减1，若某一个对象的计数器的值为0，那么表示这个对象没有人对他进行引用，也就是意味着是一个失效的垃圾对象，就会被gc进行回收。但是这种简单的算法在当前的jvm中并没有采用，原因是他并不能解决对象之间循环引用的问题。假设有A和B两个对象之间互相引用，也就是说A对象中的一个属性是B，B中的一个属性时A,这种情况下由于他们的相互引用，从而是垃圾回收机制无法识别。


  ![](https://upload-images.jianshu.io/upload_images/1534431-982627fe64dfa349.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)
  
 -  优点：简单，高效，Objective-c用的就是这种算法。
 
 -  缺点：很难处理循环引用，比如图中相互引用的两个对象则无法释放。但是也有解决办法，想知道自行搜索。

-  参数命令 
    - verbose:gc
    - -xx:+PrintGCDetail
 
     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-204_14-27-08.png)
   
     最后面两句将object1和object2赋值为null，也就是说object1和object2指向的对象已经不可能再被访问，但是由于它们互相引用对方，导致它们的引用计数器都不为0，那么垃圾收集器就永远不会回收它们。
  
## 可达性分析算法（根搜索算法）
- 为了解决上面的循环引用问题，Java采用了一种新的算法：可达性分析算法。
从GC Roots（每种具体实现对GC Roots有不同的定义）作为起点，向下搜索它们引用的对象，可以生成一棵引用树，树的节点视为可达对象，反之视为不可达。

 ![](https://upload-images.jianshu.io/upload_images/1534431-a9dda2d42cd749cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/465)
 
-  Java定义的GC Roots对象:
  - 虚拟机栈（帧栈中的本地变量表）中引用的对象。
  
  - 方法区中静态属性引用的对象。
  - 方法区中常量引用的对象。
  - 本地方法栈中JNI引用的对象。

  如果出现循环引用了，只要没有被GC Roots引用了就会被回收，完美解决！
  

## 圾回收算法-标记清除算法
- 标记-清除法分为两个阶段：标记阶段和清除阶段。标记阶段的任务是标记出所有需要被回收的对象，清除阶段就是回收被标记的对象所占用的空间。

 ![](https://upload-images.jianshu.io/upload_images/1534431-43bc311c23427d67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)

- 优点：是简单，容易实现。

- 缺点：容易产生内存碎片，碎片太多可能会导致后续过程中需要为大对象分配空间时无法找到足够的空间而提前触发新的一次垃圾收集动作。

## 圾回收算法-复制算法
- 复制算法将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用的内存空间一次清理掉，这样一来就不容易出现内存碎片的问题。

   ![](https://upload-images.jianshu.io/upload_images/1534431-9bf508a2e534d864.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/511)

- 优点：实现简单，运行高效且不容易产生内存碎片，适用于存活对象很少。回收对象多

- 缺点：内存空间的使用做出了高昂的代价，因为能够使用的内存缩减到原来的一半，如果存活对象很多，那么Copying算法的效率将会大大降低。

复制算法：适用于存活对象很少。回收对象多

## 标记整理算法
- 该算法标记阶段和标记清除法一样，但是在完成标记之后，它不是直接清理可回收对象，而是将存活对象都向一端移动，然后清理掉端边界以外的内存。

![](https://upload-images.jianshu.io/upload_images/1534431-4b6066a7e03fb403.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/679)

- 优点：不会出现内存碎片问题，适用于存活对象多，回收对象少的情况使用

- 缺点：整理时间长，容易导致卡顿。

标记整理算法: 适用用于存活对象多，回收对象少
## 分代回收算法
- 分代回收算法其实不算一种新的算法，而是根据复制算法和标记整理算法的使用场景综合使用。

- 分代算法就是根据对象存活的生命周期将内存划分为若干个不同的区域。一般情况下将堆区划分为老年代（Old Generation）和新生代（Young Generation），老年代的特点是每次垃圾收集时只有少量对象需要被回收所以采用标记整理法。而新生代的特点是每次垃圾回收时都有大量的对象需要被回收所以采用复制算法（改良的复制算法，不是按1：1分配）。

   ![](https://upload-images.jianshu.io/upload_images/1534431-df6da09763c72d95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
   
- Eden空间和两块Survivor空间的工作流程

 - 分配了一个又一个对象放到Eden区
 - 不好，Eden区满了，只能GC(新生代GC：Minor GC)了，把Eden区的存活对象copy到Survivor A区，然后清空Eden区（本来Survivor B区也需要清空的，不过本来就是空的）
 -  又分配了一个又一个对象放到Eden区
 - 不好，Eden区又满了，只能GC(新生代GC：Minor GC)了，把Eden区和Survivor A区的存活对象copy到Survivor B区，然后清空Eden区和Survivor A区
 - 又分配了一个又一个对象 放到Eden区
 -  不好，Eden区又满了，只能GC(新生代GC：Minor GC)了，把Eden区和Survivor B区的存活对象copy到Survivor A区，然后清空Eden区和Survivor B区
 - 有的对象来回在Survivor A区或者B区呆了比如15次，就被分配到老年代Old区
 - 有的对象太大，超过了Eden区，直接被分配在Old区
 -  有的存活对象，放不下Survivor区，也被分配到Old区
 
 -  在某次Minor GC的过程中突然发现：
 -  不好，老年代Old区也满了，这是一次大GC(老年代GC：Major GC)
 - Old区慢慢的整理一番，空间又够了
 - 继续Minor GC
 - ...

为什么要分新生代和老年代？
  综合使用算法，最优采用分代算法所以分为新生代和老年代两块区域。具体为什么1：2？应该是根据实践测试得出的结果，也可以调整。

## 垃圾收集器-serial收集器详解
 
- 在GC机制中，起重要作用的是垃圾收集器，垃圾收集器是GC的具体实现，Java虚拟机规范中对于垃圾收集器没有任何规定，所以不同厂商实现的垃圾 收集器各不相同，HotSpot 1.6版使用的垃圾收集器如下图（图来源于《深入理解Java虚拟机：JVM高级特效与最佳实现》，图中两个收集器之间有连线，说明它们可以配合使用）


![](https://upload-images.jianshu.io/upload_images/3951014-88ef411ba4fd7a08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/651)

不同垃圾收集器的区分：

  - 新生代、老年代垃圾收集器，

  - 单线程、多线程并行垃圾收集器，

  - stop the world和并发(GC线程和用户线程同时运行)的垃圾收集器

  - 如果你运行在JVM的客户端模式（Client）下，JVM默认垃圾收集器是串行垃圾收集器（Serial GC，-XX:+USeSerialGC）；在JVM服务器模式（Server）下默认垃圾收集器是并行垃圾收集器（Parallel GC，-XX:+UseParallelGC）。


- **serial、serial old收集器：**
    - 单线程，不存在线程之间切换，适用于client模式。
    
    - Serial old会作为CMS收集器concurrent mode failure失败时的替代收集器。

     ![](https://upload-images.jianshu.io/upload_images/3951014-749e350358001883.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

- **ParNew收集器**

  - ParNew收集器其实就是Serial收集器的多线程版本（也是一个新生代收集器），除了使用多条线程进行垃圾收集外，其余行为都与Serial收集器完全一样，共用了相当多的代码。

  - ParNew垃圾收集器作为CMS收集器的默认新生代收集器，也可以通过-XX:UseParNewGC选项来强制指定，使用-XX:ParallelGCThreads参数来限制垃圾收集的线程数。
 
 - 桌面应用还是很适合的
 
     ![](https://upload-images.jianshu.io/upload_images/3951014-fe284fa276097773.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

- **Parallel Scavenge**

  -  Parallel Scavenge也是新生代并行垃圾收集器。和ParNew不同的是，Parallel Scavenge收集器的目标是达到可控的吞吐量，吞吐量优先，吞吐量=运行用户代码时间 / （运行用户代码时间 + 垃圾收集时间）。主要适合在后台运算而不需要太多交互的任务。

  -  -XX:MaxGCPauseMillis 控制最大垃圾收集停顿时间，允许的值是一个大于零的毫秒数，收集器将尽可能地保证内存回收花费的时间不超过设定值（但是不是说这个值设置的越小，垃圾收集的速度就越快，垃圾收集停顿时间缩短是以牺牲吞吐量和新生代空间换取的：系统把新生代调小一些，收集300M新生代肯定比收集500M快，这也导致垃圾收集发生的更频繁，原来十秒收集一次、每次停顿一百毫秒，现在变成五秒收集一次、每次停顿七十毫秒，停顿时间的确下降了，但是吞吐量也下降了）。

  -  -XX:GCTimeRatio 直接设置吞吐量大小，参数的值应当是一个大于零小于一百的整数，相当于吞吐量的倒数。如果此参数值为19，那允许垃圾收集的时间就占总时间的5%（即1/(1+19)），默认值是99

  -  -XX:UseAdaptiveSizePolicy 这是一个开关参数。当这个参数打开之后，就不需要手工指定其他参数等细节了，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最何时的停顿时间或者最大的吞吐量，这种调节方式称为GC自适应的调节策略（GC Ergonomics）。这是一个不错的选择，只需要把基本的内存数据设置好（如-Xmx设置最大堆），然后使用MaxGCPauseMillis参数（更关注最大停顿时间）或GCTimeRatio（更关注吞吐量）参数给虚拟机设立一个优化目标，具体细节就不用管了。

- **Parallel Old收集器**
    - 此收集器是Parallel Scavenge收集器的老年代版本，这个收集器出现之后，“吞吐量优先”收集器终于有了名副其实的应用组合。其运行过程和Parallel Scavenge类似。

- **CMS收集器**

  - 此收集器用户线程和GC线程可以并发进行，是一种以获得最短回收停顿时间为目标的收集器。大量用在B/S系统的服务器上，因为这类应用注重响应速度，给用户较好的体验。

 - -XX:+UseConcMarkSweepGC：激活CMS收集器，默认情况下使用ParNew + CMS + Serial Old的收集器组合进行内存回收，Serial Old作为CMS出现“Concurrent Mode Failure”失败后的后备收集器使用。

    ![](https://upload-images.jianshu.io/upload_images/3951014-be8529efa3a2aa6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
    
    - 初始标记
       - CMS算法中两个会触发Stop the World事件中的一个，这个阶段会标记所有与GC Roots直接相关联的对象，以及被存活的青年代对象所直接引用的对象。
   
       ![](https://upload-images.jianshu.io/upload_images/3951014-eaaf83b2b90ccfb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/634)

    - 并发标记
      - GC在运行的过程中用户的应用线程并不会停止工作。该阶段GC收集器会从第一步“初始标记”中所标记出来的对象开始逐步遍历这些对象（与GCRoot直接相连或与存活的青年代对象直接相关联的对象）的所引用的对象，并将这些被引用的对象加上标记。（需要注意的是，这一步中，会漏掉一下老年代的存活对象，这是因为在并发的过程中，用户应用线程可能会对老年代的对象产生引用上的改变。某一些被改变的标记可能会被遗漏。）

 
       ![](https://upload-images.jianshu.io/upload_images/3951014-049c9b75d98c5c41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)
       
  - 并发预清理

       - 并发预清理是Java1.5被加入进来的。主要目的是减少重标记（Remark）步骤Stop-the-World的时间。这一步同样也是并发的，不会停止用户应用线程。在前面的并发标记中，一些引用被改变了。当某一块块（Card）中的对象引用发生改变时，JVM会标记这个空间为“脏块”（Dirty Card）。
 
        
         ![](https://upload-images.jianshu.io/upload_images/3951014-3c72009f597e7a70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/641)
         
       - 在预清理阶段，JVM根据之前记录的这些“脏对象”重新标记了他们新的可达对象。这一步结束后空间重新进入clean状态。另外，一些必要的最终重标记之前的准备步骤也会在这一步做好。

          ![](https://upload-images.jianshu.io/upload_images/3951014-604639a1a4710422.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/643)

       - 预清理步骤将会不断重复一直到Eden区的占用量达到某个指定的阈值。设定这个阈值作为结束条件的原因主要是为了防止YoungGC产生的Stop-the-World和下一阶段的Remark同时产生，导致系统产生一个更长的停滞。设定了这个阈值之后基本可以保证Remark阶段可以在两次YoungGC之间进行。

  - 重新标记

       - 这是CMS算法中第二个会触发Stop-the-World事件的步骤，由于前一步是一个并发的步骤，预清理的速度可能会赶不上用户应用对对象改变的速度，所以需要一个Stop-the-World的暂停来完整的标记所有对象结束整个标记阶段。

       - 通常CMS会在年轻代为空时来运行重标记阶段，以此避免一个接一个的Stop-the-World阶段。

  - 并发清理

       - 这一阶段程序并发地工作，目的是移除所有不用的对象，并且重新声明内存空间的归属等候将来使用。

         ![](https://upload-images.jianshu.io/upload_images/3951014-f691897a331d5add.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/642)
         
       - 这一阶段程序并发地工作，目的是移除所有不用的对象，并且重新声明内存空间的归属等候将来使用。

  - 并发重置

       - 并发地重置所有算法需要的内部数据结构，为下一次GC做准备。

- CMS缺点

   - CMS收集器对CPU资源非常敏感。
   
   - 在并发阶段，它虽然不会导致用户线程停顿，但是会因为占用了一部分线程（或者说是CPU资源）而导致应用程序变慢，总吞吐量会降低。CMS默认启动的回收线程数是（CPU数量 + 3）/4(-XX:ParallelCMSThreads={x}：设置)，也就是当CPU在四个以上时（此时回收一个线程），并发回收时垃圾收集线程不少于25%的CPU资源，并随着CPU数量的增加而下降。但是当CPU不足四个时，CMS对用户程序的影响就可能变得很大，为了应对这种情况，虚拟机提供了一种称为“增量式并发收集器”的CMS收集器变种，其实就是一种抢占式来模拟多任务机制，让多个线程交替运行，尽量减少GC线程独占资源的时间，这样整个垃圾收集的过程会变长，但是对用户程序的影响会变小，速度下降也就没那么明显了。实践证明，此变种很一般，现在已不再提倡使用。

   - CMS收集器无法处理浮动垃圾（Floating Garbage），可能出现“Concurrent Mode Failure”失败而导致另一次Full GC产生。由于CMS并发清理阶段用户线程还在运行，伴随程序运行自然就还会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS无法在档次收集中处理掉它们，只好留待下一次GC时再清理掉。这一部分垃圾称为“浮动垃圾”。正是由于垃圾收集过程中，用户线程还要执行，那么也就需要预留足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全填满后再进行垃圾收集，需要预留一部分空间提供并发收集时的程序运作使用。可以使用参数-XX:CMSInitiatingOccupancyFraction来设置老年代的阈值，在JDK1.5中设置为当老年代使用了68%就会被激活进行垃圾收集，JDK1.6中设置为92%。要是CMS运行期间预留的内存无法满足程序需要，居会出现一次“Concurrent Mode Failure”失败，这时虚拟机将启动后备预案：临时启用Serial Old收集器重新进行老年代的垃圾收集，此时停顿时间就会很长了。所以该参数设置得太高可能反而会降低性能。

   - 由于CMS是一款基于“标记-清除”算法实现的收集器，在收集结束后会有大量的空间碎片产生。这将会给大对象分配带来很大麻烦，往往会出现老年代还有很大空间剩余，但是无法找到足够大的连续空间来分配当前对象不得不提前触发一次Full GC。为了解决此问题，CMS收集器提供了一个-XX:+UseCMSCompactAtFullCollection开关参数（默认是开启），用于在CMS收集器顶不住要进行Full GC时开启内存碎片的合并整理过程，内存整理的过程是无法并发执行的，空间碎片问题没有了，但停顿时间不得不变长。还提供了参数-XX:CMSFullGCsBeforeCompaction，这个参数用于设置执行多少次不压缩（不整理）的Full GC后，跟着来一次带压缩的（默认为零，表示每次进入Full GC时都进行碎片整理）。

- G1收集器  
    ![](https://upload-images.jianshu.io/upload_images/3951014-9ae309e9446ea98c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/677)
     
   -  G1将新生代，老年代的物理空间划分取消了，取而代之的是，G1算法将堆划分为若干个区域（Region），区域分为新生代、老年代，它仍然属于分代收集器。

   -  新生代的垃圾收集依然采用暂停所有应用线程的方式，将存活对象拷贝到老年代或者Survivor空间。

   -  老年代也分成很多区域，G1收集器通过将对象从一个区域复制到另外一个区域，完成了清理工作，避免CMS内存碎片问题。

   -  在G1中，还有一种特殊的区域，叫Humongous区域，专门用于存储巨大对象。如果一个对象占用的空间超过了分区容量50%以上，G1收集器就认为这是一个巨型对象。这些巨型对象，默认直接会被分配在年老代，但是如果它是一个短期存在的巨型对象，就会对垃圾收集器造成负面影响。

   -  对象分配情况分三种：TLAB(Thread Local Allocation Buffer)线程本地分配缓冲区、Eden区中分配、Humongous区分配。

   -  G1提供了两种GC模式，Young GC和Mixed GC，两种都是Stop The World(STW)的。

- Young GC

    - Young GC主要是对Eden区进行GC，它在Eden空间耗尽时会被触发。在这种情况下，Eden空间的数据移动到Survivor空间中，如果Survivor空间不够，Eden空间的部分数据会直接晋升到年老代空间(Survivor区的数据移动到新的Survivor区中，也有部分数据晋升到老年代空间中。)。最终Eden空间的数据为空，GC停止工作，应用线程继续执行。


     ![](https://upload-images.jianshu.io/upload_images/3951014-bd4fffe978f4655a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)


   - 如果仅仅GC 新生代对象，我们如何找到所有的根对象呢？ 老年代的所有对象都是根么？那这样扫描下来会耗费大量的时间。于是，G1引进了RSet的概念。它的全称是Remembered Set，作用是跟踪指向某个heap区内的对象引用。这样就避免扫描整个老年代。(在CMS中，也有RSet的概念，在老年代中有一块区域用来记录指向新生代的引用。)


     ![](https://upload-images.jianshu.io/upload_images/3951014-96563921ad907ed6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/531)
     
    - 如果引用的对象很多，赋值器需要对每个引用做处理，赋值器开销会很大，为了解决赋值器开销这个问题，在G1 中又引入了另外一个概念，卡表（CardTable）。一个CardTable将一个分区在逻辑上划分为固定大小的连续区域，每个区域称之为卡。卡通常较小，介于128到512字节之间。CardTable通常为字节数组，由Card的索引（即数组下标）来标识每个分区的空间地址。默认情况下，每个卡都未被引用。当一个地址空间被引用时，这个地址空间对应的数组索引的值被标记为”0″，即标记为脏被引用，此外RSet也将这个数组下标记录下来。一般情况下，这个RSet其实是一个HashTable，Key是别的Region的起始地址，Value是一个集合，里面的元素是Card Table的Index。

   - Young GC 阶段：

     - 阶段1：根扫描：静态和本地对象被扫描
 
     - 阶段2：更新RS：处理dirty card队列更新RS

     - 阶段3：处理RS：检测从年轻代指向年老代的对象

     - 阶段4：对象拷贝：拷贝存活的对象到survivor/old区域

     - 阶段5：处理引用队列：软引用，弱引用，虚引用处理

  - Mix GC
     - Mix GC不仅进行正常的新生代垃圾收集，同时也回收部分后台扫描线程标记的老年代分区。

     它的GC步骤分2步：

     - 全局并发标记（global concurrent marking）

     - 拷贝存活对象（evacuation）

     - 全局并发标记主要是为Mixed GC提供标记服务的，但并不是一次GC过程的一个必须环节。

    ![](https://upload-images.jianshu.io/upload_images/3951014-66524a374c966c35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/682)


