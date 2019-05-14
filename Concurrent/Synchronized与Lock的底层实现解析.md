# Synchronized与Lock的底层实现解析

在JDK1.5之前，我们在编写并发程序的时候无一例外都是使用synchronized来实现线程同步的，而synchronized在JDK1.5之前同步的开销较大效率较低，因此在JDK1.5之后，推出了代码层面的Lock接口（synchronized为jvm层面）来实现与synchronized同样功能的同步锁，并且针对不同的并发场景也加入了许多个性锁功能。

在java.util.concurrent.locks包中有很多Lock的实现类，常用的有ReentrantLock、ReadWriteLock（实现类ReentrantReadWriteLock），其实现都依赖java.util.concurrent.AbstractQueuedSynchronizer类简称AQS，他是实现Lock接口所有锁的核心。

Synchronized与Lock的区别

![](https://upload-images.jianshu.io/upload_images/325120-e3401c5935e0197f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1、synchronized锁源码分析

对于synchronized来说，其实没有什么所谓的源码去分析，synchronized是Java中的一个关键字，他的实现时基于jvm指令去实现的下面我们写一个简单的synchronized示例

![](https://oscimg.oschina.net/oscnet/c083b4974d8443f03f5cc30b16cb502b2af.jpg)

我们点击查看SyncDemo.java的源码SyncDemo.class，可以看到如下： 

![](https://oscimg.oschina.net/oscnet/be3a03ab292c6c6a5069d1d23fe7904b4c3.jpg)

其实synchronized映射成字节码指令就是增加来两个指令：monitorenter和monitorexit。我们看到上面的class文件第9、13、19行，分别加入了monitorenter、monitorexit、monitorexit。当一条线程进行执行的遇到monitorenter指令的时候，它会去尝试获得锁，如果获得锁那么锁计数+1（为什么会加一呢，因为它是一个可重入锁，所以需要用这个锁计数判断锁的情况），如果没有获得锁，那么阻塞。当它遇到monitorexit的时候，锁计数器-1，当计数器为0，那么就释放锁。


## 2、Lock接口源码逐个分析


针对Lock接口中的锁，我们来一个个来深入分析，首先需要了解下面的名词概念

- 独占锁、共享锁
- 公平锁、非公平锁、重入锁
- 条件锁
- 读写锁

**ReentrantLock 可重入锁深入源码分析**

ReentrantLock相当于是对synchronized的一个实现，他与synchronized一样是一个可重入锁并且是一个独占锁，但是synchronized是一个非公平锁，任何处于竞争队列的线程都有可能获取锁，而ReentrantLock既可以为公平锁，又可以为非公平锁。


![](https://oscimg.oschina.net/oscnet/51c4399b6ff0df8f43e20a01d5b344b612f.jpg)

根据上面的源码我们可知，ReentrantLock继承于Lock接口，并且是并发编程大师Doug Lea所创作（向大师致敬）并且在源码中我们可以发现，很多操作都是基于Sync这个类实现的，而Sync是一个内部抽象静态类，继承AQS类。而Sync又有两个子类

非公平锁子类：

```
static final class NonfairSync extends Sync
```

公平锁子类：

```
static final class FairSync extends Sync
```

他们两个的实现大致相同，差别不大，并且ReentrantLock的默认是采用非公平锁，非公平锁相对公平锁而言在吞吐量上有较大优势，我们分析源码也主要从非公平锁入手。

本人主要通过ReentrantLock的加锁，到解锁的源码流程来分析。ReentrantLock的类图如下

![](https://oscimg.oschina.net/oscnet/b7213e4efb8314ce768e1a425bbdb8dcc46.jpg)

ReentrantLock的实现依赖于Java同步器框架AbstractQueuedSynchronizer（本文简称之为AQS）。AQS使用一个整型的volatile变量（命名为state）来维护同步状态，马上我们会看到，这个volatile变量是所有Lock实现锁的关键。加锁、解锁的状态全都围绕这个状态位去实现。

![](https://oscimg.oschina.net/oscnet/63ef6416f4c72c26dd56f4dd8c6f92d58c2.jpg)

闲话不多说，首先是ReentrantLock的非公平锁的加锁方法lock()

- 1.第一步，加锁lock()

![](https://oscimg.oschina.net/oscnet/62c809f47cad42f1a1aec0601c81273b81e.jpg)

  公平锁的加锁方法lock()如下

  ![](https://oscimg.oschina.net/oscnet/139dccde4bb0bba6583a866d65c955858b2.jpg)

**我们可以看到非公平锁与公平锁的加锁区别在于，非公平锁首先会进行一次CAS，去尝试修改AQS中的锁标记state字段，将其从0（无锁状态），修改为1（锁定状态）（注：ReentrantLock用state表示“持有锁的线程已经重复获取该锁的次数”。当state等于0时，表示当前没有线程持有锁），如果成功，就设置ExclusiveOwnerThread的值为当前线程（Exclusive是独占的意思，ReentrantLock用exclusiveOwnerThread表示“持有锁的线程”），如果成功，执行setExclusiveOwnerThread方法将持有锁的线程（ownerThread）设置为当前线程，否则就执行acquire方法，而公平锁线程不会尝试去获取锁，直接执行acquire方法。**

- 2.acquire方法

![](https://oscimg.oschina.net/oscnet/20c58e39d74999d19a738c2b356b99b50e2.jpg)

根据acquire方法的注释大概能知道他的作用：

获取独占模式，忽略中断。通过调用至少一次tryAcquire方法，成功则返回。否则，线程可能排队。重复阻塞和解除阻塞，调用tryAcquire直到成功。

acquire方法的执行逻辑为，首先调用tryAcquire尝试获取锁，如果获取不到，则调用addWaiter方法将当前线程加入包装为Node对象加入队列队尾，之后调用acquireQueued方法不断的自旋获取锁。

其中tryAcquire方法、addWaiter方法、acquireQueued方法我们接下来逐个分析。

- 3.tryAcquire方法

非公平锁中的tryAcquire方法直接调用Sync的nofairTryAcquire方法，源码如下：

![](https://oscimg.oschina.net/oscnet/b235e3f7be563a027f8beaed6a83ed857f1.jpg)

nofairTryAcquire方法的逻辑：

  - 获取当前的锁标记位state，如果state为0表名此时没有线程占用锁，直接进入if中获取锁的逻辑，与非公平锁lock方法的前半部分一样，将state标记CAS改变为1，设置获取独占锁的线程为当前线程。

  - 如果state锁标记位的state不为0，说明有线程占用了该锁，此时需要判断占用锁的线程是否为当前线程（getExclusiveQwnerThread方法获取占用锁的线程），如果是当前线程，则将state锁标记位+1 表示重入，并修改status值，但因为没有竞争，获取锁的线程还是当前线程，所以通过setStatus修改，而非CAS，也就是说这段代码实现了类似偏向锁的功能，并且实现的非常漂亮。

  - 如果state锁标记位既不为0，也不是当前线程，表示其他线程来争夺锁，结果当然是失败。

我们回到第2步的acquire方法，当tryAcquire方法返回true，说明没有线程占用锁，当前线程获取锁成功，后续的addWaiter方法与acquireQueued方法不再执行并返回，线程执行同步块中的方法。若tryAcquire方法返回false，说明当前有其他线程占用锁，此时将会触发执行addWaiter方法与acquireQueued方法。


公平锁中的tryAcquire方法与非公平锁基本相同，只不过比非公平锁在第一次获取锁时的判断中多了hasQueuedPredecessors方法

![](https://oscimg.oschina.net/oscnet/bfeeeef06e9e14fba19d1417f740af7167f.jpg)

hasQueuedPredecessors方法用于判断当前线程是否为head节点的后续节点线程（预备获取锁的线程节点）。

或者说：判断“当前线程”是不是CLH队列中的第一个线程线程（head节点的后置节点），若是的话返回false，不是返回true。


![](https://oscimg.oschina.net/oscnet/4bd8d4ba4e96ab6d995958dcc6f03fad86b.jpg)

说明： 通过代码，能分析出，hasQueuedPredecessors() 是通过判断"当前线程"是不是在CLH队列的队首，来返回AQS中是不是有比“当前线程”等待更久的线程。下面对head、tail和Node进行说明。


- 4.addWaiter方法

addWaiter方法的主要目的是将当前线程包装为一个独占模式的Node隐式队列，在分析方法前我们需要了解Node类的几个重要的参数：

![](https://oscimg.oschina.net/oscnet/1e54cfd5aeba4ee99e15df42af706c6bb27.jpg)

prev：前置节点；

next：后置节点；

waitStatus：是等待链表（队列）中的状态，状态分一下几种


![](https://oscimg.oschina.net/oscnet/2f8c38cc476e2c2e1e8a178ca6a687e2cee.jpg)

而在AQS中，记录了Node的头结点head，和尾节点tail

![](https://oscimg.oschina.net/oscnet/ae5b6e965114bf4d67ba2f91259c5e3b1c2.jpg)

首先将当前线程保障成一个独占模式的Node节点对象，然后判断当前队尾（tail节点）是否有节点，如果有，则通过CAS将队尾节点设置为当前节点，并将当前节点的前置节点设置为上一个尾节点。

如果tail尾节点为null，说明当前节点为第一个入队节点，或者CAS设置当前节点为尾节点失败，将调用enq方法。

![](https://oscimg.oschina.net/oscnet/f62403f2457baa450ed8e3da76a74a00ce1.jpg)

enq方法也是一个CAS方法，当第一次循环tail尾节点为null时，说明当前节点为第一个入队节点，此时将新建一个空Node节点为傀儡节点，并将其设置为队首，然后再次循环时，将当前节点设置为tail尾节点，失败将循环设定直至成功。若是addWaiter方法中设置tail尾节点失败的话，进入enq方法后直接将进入else模块将当前节点设置为tail尾节点，循环设定直至成功。

接了方便理解addWaiter方法的作用，以及后续acquireQueued的理解，我们通过3个线程来画图演示从第1步到第4步AQS中Node队列的情况：

-  1.假设当前有一个线程 thread-1执行lock方法，由于此时没有其他线程占用锁，thread-1得到了锁

-  2.此时thread-2执行lock方法，由于此时thread-1占用锁，因此thread-2执行acquire方法，并且thread-1不释放锁，tryAcquire方法失败，执行addWaiter方法
 
 ![](https://oscimg.oschina.net/oscnet/1996b85884117419ecdd7efd5cefd414704.jpg)

由于thread-2第一个进入队列，此时AQS中head以及tail为null，因此进入执行enq方法，根据上面描述的enq方法逻辑，执行之后等待队列为

![](https://oscimg.oschina.net/oscnet/a45a7aa0aca85747c69e4f562c2ea56debb.jpg)

-  3.接下来thread-3执行lock方法，thread-2依然没有释放锁，此时对接就变成这样

![](https://oscimg.oschina.net/oscnet/f2dda3375208c854718b0096ace147cdf39.jpg)

addWaiter方法的的作用就是将一个个没有获取锁的线程，包装成为一个等待队列。


- **5.acquireQueued方法**

![](https://oscimg.oschina.net/oscnet/059beefa78c7a32cd7f7c7b7fc1e4424e43.jpg)

acquireQueued方法的作用就是CAS循环获取锁的方法，并且如果当前节点为head节点的后续节点，则尝试获取锁，如果获取成功则将当前节点置为head节点，并返回，如果获取失败或者当前节点并不是head节点的后续节点，则调用shouldParkAfterFailedAcquire方法以及parkAndCheckInterrupt方法，将当前节点的前置节点状态位置为SIGNAL(-1) ，并阻塞当前节点。


- 6.shouldParkAfterFailedAcquire方法

![](https://oscimg.oschina.net/oscnet/dd36199dcff571aee3bd73e5f4d6e73ccb2.jpg)

shouldParkAfterFailedAcquire方法根据源码分许我们可以得知，该方法就是用来设置前置节点的状态位为SIGNAL(-1)，而SIGNAL(-1)表示节点的后置节点处于阻塞状态。首次进入该方法时，前置节点的waitStatus为0，因此进入else代码块中，通过CAS将waitStatus设置为-1，当外围方法acquireQueued再次循环时，将直接返回true。这时将满足判断条件执行parkAndCheckInterrupt方法。


![](https://oscimg.oschina.net/oscnet/81d326a0a7b6607b5f2ed52c4099af024fe.jpg)


而中间这块判断逻辑则是前置节点状态为CANCELLED(1)，则继续查找前置节点的前驱节点，因为当head节点唤醒时，会跳过CANCELLED(1)节点（CANCELLED(1)：因为超时或中断或异常，该线程已经被取消）。

摘取网上大神的shouldParkAfterFailedAcquire方法的逻辑总结：

 - 如果前继的节点状态为SIGNAL，表明当前节点需要unpark，则返回成功，此时acquireQueued方法的第12行（parkAndCheckInterrupt）将导致线程阻塞
 - 如果前继节点状态为CANCELLED(ws>0)，说明前置节点已经被放弃，则回溯到一个非取消的前继节点，返回false，acquireQueued方法的无限循环将递归调用该方法，直至规则1返回true，导致线程阻塞（parkAndCheckInterrupt）
 - 如果前继节点状态为非SIGNAL、非CANCELLED，则设置前继的状态为SIGNAL，返回false后进入acquireQueued的无限循环，与第2点相同
 - 总体看来，shouldParkAfterFailedAcquire就是靠前继节点判断当前线程是否应该被阻塞，如果前继节点处于CANCELLED状态，则顺便删除这些节点重新构造队列。

- 7.parkAndCheckInterrupt方法

![](https://oscimg.oschina.net/oscnet/ada9f54c986fd2857ed401d1618335fe66d.jpg)

很简单，就是讲将前线程中断，返回中断状态。

那么此时我们来通过画图，总结下步骤5~步骤7：

在thread-2与thread-3执行完步骤4后

![](https://oscimg.oschina.net/oscnet/b685e85970f4d6a9261ed324d178e0bc4ad.jpg)

此时thread-2执行步骤5，由于他的前置节点为Head节点因此它有了一次tryAcquire获取锁的机会，如果成功则设置thread-2的Node节点为head节点然后返回，由于当前节点没有被中断，因此返回的中断标记位为false。

如果tryAcquire获取锁依然失败，则调用shouldParkAfterFailedAcquire方法以及parkAndCheckInterrupt方法对线程进行阻塞，直到持有锁的线程释放锁时被唤醒（具体后续说明，现在只需要知道前置节点获取释放锁后，会唤醒他的后置节点的线程），此时执行了shouldParkAfterFailedAcquire方法后将会变成这样

![](https://oscimg.oschina.net/oscnet/6f1836d1eda65b8683e63fc994ad6249497.jpg)

当锁被释放thread-2被唤醒后再次执行tryAcquire获取锁，此时由于锁已释放获取将会成功，但由于当前节点被中断过interrupted为true，因此返回的中断标记位为true。

![](https://oscimg.oschina.net/oscnet/980aa862121d9bf02e0192fb7dcfb0f665c.jpg)

回到上面步骤2中，此时将会执行selfInterrupt方法将当前线程从阻塞状态唤醒。

![](https://oscimg.oschina.net/oscnet/ce4a861d194e6bc197de96bf6841cae0e1b.jpg)

而thread-3则和thread-2经历差不多，区别在于thread-3的前置节点不是head节点，因此进入acquireQueued方法后thread-3直接被阻塞，直到thread-2获取锁后变为head节点并且释放锁之后，thread-3才会被唤醒。thread-3进入acquireQueued方法后变为

（为了避免大家理解不了，此处再次说明，前置节点的waitStatus为-1时表示当前节点处于阻塞态）

![](https://oscimg.oschina.net/oscnet/e887c7f49fb0a7a287b867e746686010a9c.jpg)

下面来说说步骤5中acquireQueued方法的finally代码块

cancelAcquire方法：如果出现异常或者出现中断，就会执行finally的取消线程的请求操作，核心代码是node.waitStatus = Node.CANCELLED;将线程的状态改为CANCELLED。

```
private void cancelAcquire(Node node) {
    // Ignore if node doesn't exist
    if (node == null)
        return;

    node.thread = null;

    // 获取前置节点并判断状态，如果前置节点已被取消，则将其丢弃重新指向前置节点，直到指向一个距离当前节点最近的有效节点，这种处理非常巧妙让人佩服
    Node pred = node.prev;
    while (pred.waitStatus > 0)
        node.prev = pred = pred.prev;

    //获取新的前置节点的后置节点（此时新的前置节点的next还没有指向当前节点）
    Node predNext = pred.next;

    //将当前节点设置为取消状态
    node.waitStatus = Node.CANCELLED;

    // 如果当前节点为尾部节点，直接丢弃
    if (node == tail && compareAndSetTail(node, pred)) {
        compareAndSetNext(pred, predNext, null);
    } else {
        //如果前置节点不是head，则后置节点需要一个状态，来对标记当前节点的状态，此处是设置新的前置节点的waitStatus为SIGNAL(-1)，并且将新的前置节点的next指向当前节点，当前节点不会再此处被丢弃，而是在shouldParkAfterFailedAcquire方法中丢弃
        int ws;
        if (pred != head &&
            ((ws = pred.waitStatus) == Node.SIGNAL ||
             (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) {
            Node next = node.next;
            if (next != null && next.waitStatus <= 0)
                compareAndSetNext(pred, predNext, next);
        } else {
            //如果前置节点为head，则直接唤醒距离当前节点最近的有效后置节点
            unparkSuccessor(node);
        }

        node.next = node; // help GC
    }
}
```

unparkSuccessor方法源码如下：

首先，将当前节点waitStatus设置为0，然后获取距离当前节点最近的有效后置节点，最后unpark唤醒后置节点的线程，此时后置节点线程就有机会获取锁了

![](https://oscimg.oschina.net/oscnet/c065b4e4bd73f07c3b73a1630aaac832efe.jpg)

至此，所有的lock逻辑全部走完了，下面来说说解锁。

ReentrantLock的unlock方法

![](https://oscimg.oschina.net/oscnet/1ed48762afb1d92eba85cd75b1e39202638.jpg)

其实是调用的AQS的release方法

![](https://oscimg.oschina.net/oscnet/854226d4eb966f48d1d4429ddc26190c1a8.jpg)

release方法的逻辑为，首先调用tryRelease方法，如果返回true，就执行unparkSuccessor方法唤醒后置节点线程。接下来我们看看tryRelease方法，在ReentrantLock中实现。

![](https://oscimg.oschina.net/oscnet/27c3de5aee62720cf652e50329d8a0382c4.jpg)

我们看到在tryRelease方法中首先会获取state锁标记，将其进行-1操作，并且返回结果根据state锁标记位是否为0，如果为0则返回true，否则返回false。

我们知道ReentrantLock是一个可重入锁，前面分析了同一个线程，每次获取锁，重入锁，都会为state锁标记+1，state记录了线程获取了多少次锁。那么同一个线程获取了多少次锁，就要进行多少次解锁，直到全部解锁，state锁标记为0时，表示解锁成功，tryRelease方法返回true，后续唤醒后置节点线程


