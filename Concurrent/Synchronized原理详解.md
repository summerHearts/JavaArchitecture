# 1、Synchronized原理详解

- Synchronized主要作用是在多个线程操作共享数据的时候，保证对共享数据访问的线程安全性。比如两个线程对于i这个共享变量同时做i++递增操作，那么这个时候对于i这个值来说就存在一个不确定性，也就是说理论上i的值应该是2，但是也可能是1。而导致这个问题的原因是线程并行执行i++操作并不是原子的，存在线程安全问题。所以通常来说解决办法是通过加锁来实现线程的串行执行，而synchronized就是java中锁的实现的关键字。

- synchronized在并发编程中是一个非常重要的角色，在JDK1.6之前，它是一个重量级锁的角色，但是在JDK1.6之后对synchronized做了优化，优化以后性能有了较大的提升

## 2、Synchronized的使用
synchronized有三种使用方法，这三种使用方法分别对应三种不同的作用域，代码如下：

①修饰普通同步方法

将synchronized修饰在普通同步方法，那么该锁的作用域是在当前实例对象范围内,也就是说对于 SyncDemosd=newSyncDemo();这一个实例对象sd来说，多个线程访问access方法会有锁的限制。如果access已经有线程持有了锁，那这个线程会独占锁，直到锁释放完毕之前，其他线程都会被阻塞。

```
public SyncDemo{

    Object lock =new Object();
    //形式1
    public synchronized void access(){
    
    }
    //形式2,作用域等同于形式1
    public void access1(){
        synchronized(lock){
        
        }
    }
}
```
②修饰静态同步方法

修饰静态同步方法或者静态对象、类，那么这个锁的作用范围是类级别。举个简单的例子，{SyncDemo sd=SyncDemo();SyncDemo sd2=new SyncDemo();} 两个不同的实例sd和sd2， 如果sd这个实例访问access方法并且成功持有了锁，那么sd2这个对象如果同样来访问access方法，那么它必须要等待sd这个对象的锁释放以后，sd2这个对象的线程才能访问该方法，这就是类锁；也就是说类锁就相当于全局锁的概念，作用范围是类级别。

```
public SyncDemo{
    static Object lock=new Object();
    //形式1
    public synchronized static void access(){
  
    }
    //形式2等同于形式1
    public void access1(){
        synchronized(lock){
    
        }
    }
    //形式3等同于前面两种
    public void access2(){
        synchronzied(SyncDemo.class){
        
        }
    }
}
```

③.同步方法块
同步方法块，是范围最小的锁，锁的是synchronized括号里面配置的对象。这种锁在实际工作中使用得比较频繁，毕竟锁的作用范围越大，那么对性能的影响就越严重。

```
public SyncDemo{
    Object lock=new Object();
    public void access(){
    //do something
        synchronized(lock){
        //
        }
    }
}
```
## 3、Synchronized的实现原理分析

   synchronized实现的锁是存储在Java对象头里，什么是对象头呢？在Hotspot虚拟机中，对象在内存中的存储布局，可以分为三个区域:对象头(Header)、实例数据(Instance Data)、对齐填充(Padding)
   
   当我们在Java代码中，使用new创建一个对象实例的时候，（hotspot虚拟机）JVM层面实际上会创建一个 instanceOopDesc对象。instanceOopDesc的定义在Hotspot源码中的 instanceOop.hpp文件中，另外，arrayOopDesc的定义对应 arrayOop.hpp
   
![](https://segmentfault.com/img/bVbsc99?w=554&h=300)

从instanceOopDesc代码中可以看到 instanceOopDesc继承自oopDesc，oopDesc的定义载Hotspot源码中的 oop.hpp文件中。

![](https://segmentfault.com/img/bVbsdag?w=554&h=208)


在普通实例对象中，oopDesc的定义包含两个成员，分别是 _mark和 _metadata，其中_mark表示对象标记、属于markOop类型，也就是Mark World，它记录了对象和锁有关的信。_metadata表示类元信息，类元信息存储的是对象指向它的类元数据(Klass)的首地址，其中Klass表示普通指针、 _compressed_klass表示压缩类指针。

Mark Word
前面说的普通对象的对象头由两部分组成，分别是markOop以及类元信息，markOop官方称为Mark Word 。在Hotspot中，markOop的定义在 markOop.hpp文件中，代码如下

![](https://segmentfault.com/img/bVbsdap?w=554&h=292)

Mark word记录了对象和锁有关的信息，当某个对象被synchronized关键字当成同步锁时，那么围绕这个锁的一系列操作都和Mark word有关系。Mark Word在32位虚拟机的长度是32bit、在64位虚拟机的长度是64bit。 Mark Word里面存储的数据会随着锁标志位的变化而变化。
锁标志位的表示意义
1.锁标识 lock=00 表示轻量级锁
2.锁标识 lock=10 表示重量级锁
3.偏向锁标识 biased_lock=1表示偏向锁
4.偏向锁标识 biased_lock=0且锁标识=01表示无锁状态

https://www.jianshu.com/p/e62fa839aa41

在synchronized锁中，这个存储在对象头的Mark Word中的锁信息是一个指针，它指向一个monitor对象（也称为管程或监视器锁）的起始地址。这样，我们就通过对象头，将每一个对象与一个monitor关联了起来，它们的关系如下图所示：

![](https://raw.githubusercontent.com/ChiuCheng/images/master/fat-lock.png)

图片的最左边是线程的调用栈，它引用了堆中的一个对象，该对象的对象头部分记录了该对象所使用的监视器锁，该监视器锁指向了一个monitor对象。

那么这个monitor对象是什么呢？ 在Java虚拟机(HotSpot)中，monitor是由ObjectMonitor实现的，其主要数据结构如下:

```
 ObjectMonitor() {
    _header       = NULL;
    _count        = 0;
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL;
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ;
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
    _previous_owner_tid = 0;
 }
```
上面这些字段中，我们只需要重点关注三个字段:

_owner : 当前拥有该 ObjectMonitor 的线程
_EntryList: 当前等待锁的集合
_WaitSet: 调用了Object.wait()方法而进入等待状态的线程的集合，即我们上一篇一直提到的wait set
在java中，每一个等待锁的线程都会被封装成ObjectWaiter对象，当多个线程同时访问一段同步代码时，首先会被扔进 _EntryList 集合中，如果其中的某个线程获得了monitor对象，他将成为 _owner，如果在它成为 _owner之后又调用了wait方法，则他将释放获得的monitor对象，进入 _WaitSet集合中等待被唤醒。

![](https://www.javatpoint.com/images/core/interthread.gif)

另外，因为每一个对象都可以作为synchronized的锁，所以每一个对象都必须支持wait()，notify，notifyAll方法，使得线程能够在一个monitor对象上wait, 直到它被notify。这也就解释了这三个方法为什么定义在了Object类中——这样，所有的类都将持有这三个方法。


**总结:**

- 每一个java对象都和一个ObjectMonitor对象相关联，关联关系存储在该java对象的对象头里的Mark Word中。
- 每一个试图进入同步代码块的线程都会被封装成ObjectWaiter对象，它们或在ObjectMonitor对象的_EntryList中，或在 _WaitSet 中，等待成为ObjectMonitor对象的owner. 成为owner的对象即获取了监视器锁。
- 所以，说是每一个java对象都可以作为锁，其实是指将每一个java对象所关联的ObjectMonitor作为锁，更进一步是指，大家都想成为 某一个java对象所关联的、ObjectMonitor对象的、_owner，所以你可以把这个_owner看做是铁王座，所有等待在这个监视器锁上的线程都想坐上这个铁王座，谁拥有了它，谁就有进入由它锁住的同步代码块的权利。



## 4.锁的升级
前面提到了锁的几个概念，偏向锁、轻量级锁、重量级锁。在JDK1.6之前，synchronized是一个重量级锁，性能比较差。从JDK1.6开始，为了减少获得锁和释放锁带来的性能消耗，synchronized进行了优化，引入了 偏向锁和 轻量级锁的概念。所以从JDK1.6开始，锁一共会有四种状态，锁的状态根据竞争激烈程度从低到高分别是:无锁状态->偏向锁状态->轻量级锁状态->重量级锁状态。这几个状态会随着锁竞争的情况逐步升级。为了提高获得锁和释放锁的效率，锁可以升级但是不能降级。

**偏向锁**
在大多数的情况下，锁不仅不存在多线程的竞争，而且总是由同一个线程获得。因此为了让线程获得锁的代价更低引入了偏向锁的概念。偏向锁的意思是如果一个线程获得了一个偏向锁，如果在接下来的一段时间中没有其他线程来竞争锁，那么持有偏向锁的线程再次进入或者退出同一个同步代码块，不需要再次进行抢占锁和释放锁的操作。偏向锁可以通过 -XX:+UseBiasedLocking开启或者关闭。

- 偏向锁的获取

偏向锁的获取过程非常简单，当一个线程访问同步块获取锁时，会在对象头和栈帧中的锁记录里存储偏向锁的线程ID，表示哪个线程获得了偏向锁，结合前面分析的Mark Word来分析一下偏向锁的获取逻辑
1首先获取目标对象的Mark Word，根据锁的标识为和epoch去判断当前是否处于可偏向的状态
2如果为可偏向状态，则通过CAS操作将自己的线程ID写入到MarkWord，如果CAS操作成功，则表示当前线程成功获取到偏向锁，继续执行同步代码块
3如果是已偏向状态，先检测MarkWord中存储的threadID和当前访问的线程的threadID是否相等，如果相等，表示当前线程已经获得了偏向锁，则不需要再获得锁直接执行同步代码；如果不相等，则证明当前锁偏向于其他线程，需要撤销偏向锁。

- 偏向锁的撤销

当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放偏向锁，撤销偏向锁的过程需要等待一个全局安全点(所有工作线程都停止字节码的执行)。
1首先，暂停拥有偏向锁的线程，然后检查偏向锁的线程是否为存活状态
2如果线程已经死了，直接把对象头设置为无锁状态
3如果还活着，当达到全局安全点时获得偏向锁的线程会被挂起，接着偏向锁升级为轻量级锁，然后唤醒被阻塞在全局安全点的线程继续往下执行同步代码

**轻量级锁**

前面我们知道，当存在超过一个线程在竞争同一个同步代码块时，会发生偏向锁的撤销。偏向锁撤销以后对象会可能会处于两种状态

1.一种是不可偏向的无锁状态，简单来说就是已经获得偏向锁的线程已经退出了同步代码块，那么这个时候会撤销偏向锁，并升级为轻量级锁

2.一种是不可偏向的已锁状态，简单来说就是已经获得偏向锁的线程正在执行同步代码块，那么这个时候会升级到轻量级锁并且被原持有锁的线程获得锁

- 轻量级锁加锁

  - 1.JVM会先在当前线程的栈帧中创建用于存储锁记录的空间(LockRecord)
  - 2.将对象头中的Mark Word复制到锁记录中，称为Displaced Mark Word.
  - 3.线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针
  - 4.如果替换成功，表示当前线程获得轻量级锁，如果失败，表示存在其他线程竞争锁，那么当前线程会尝试使用CAS来获取锁，当自旋超过指定次数(可以自定义)时仍然无法获得锁，此时锁会膨胀升级为重量级锁
  
- 轻量锁解锁
  - 1.尝试CAS操作将所记录中的Mark Word替换回到对象头中
  - 2.如果成功，表示没有竞争发生
  - 3.如果失败，表示当前锁存在竞争，锁会膨胀成重量级锁
  - 一旦锁升级成重量级锁，就不会再恢复到轻量级锁状态。当锁处于重量级锁状态，其他线程尝试获取锁时，都会被阻塞，也就是 BLOCKED状态。当持有锁的线程释放锁之后会唤醒这些现场，被唤醒之后的线程会进行新一轮的竞争

**重量级锁**

重量级锁依赖对象内部的monitor锁来实现，而monitor又依赖操作系统的MutexLock（互斥锁），假设Mutex变量的值为1，表示互斥锁空闲，这个时候某个线程调用lock可以获得锁，而Mutex的值为0表示互斥锁已经被其他线程获得，其他线程调用lock只能挂起等待。

为什么重量级锁的开销比较大呢？原因是当系统检查到是重量级锁之后，会把等待想要获取锁的线程阻塞，被阻塞的线程不会消耗CPU，但是阻塞或者唤醒一个线程，都需要通过操作系统来实现，也就是相当于从用户态转化到内核态，而转化状态是需要消耗时间的。

