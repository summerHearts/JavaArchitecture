## 1.并发编程中的三个概念

  在并发编程中，我们通常会遇到以下三个问题：原子性问题，可见性问题，有序性问题。我们先看具体看一下这三个概念：

**原子性**：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

**可见性**：是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

```
//线程1执行的代码
int i = 0;
i = 10;
```

```
//线程2执行的代码
j = i;
```

假若执行线程1的是CPU1，执行线程2的是CPU2。由上面的分析可知，当线程1执行 i =10这句时，会先把i的初始值加载到CPU1的高速缓存中，然后赋值为10，那么在CPU1的高速缓存当中i的值变为10了，却没有立即写入到主存当中。此时线程2执行 j = i，它会先去主存读取i的值并加载到CPU2的缓存当中，注意此时内存当中i的值还是0，那么就会使得j的值为0，而不是10.

这就是可见性问题，线程1对变量i修改了之后，线程2没有立即看到线程1修改的值。

**有序性**：即程序执行的顺序按照代码的先后顺序执行。

```
//线程1:
context = loadContext();   //语句1
inited = true;             //语句2
``` 

```
//线程2:
while(!inited ){
  sleep()
}
doSomethingwithconfig(context);
```

上面代码中，由于语句1和语句2没有数据依赖性，因此可能会被重排序。假如发生了重排序，在线程1执行过程中先执行语句2，而此是线程2会以为初始化工作已经完成，那么就会跳出while循环，去执行doSomethingwithconfig(context)方法，而此时context并没有被初始化，就会导致程序出错。
从上面可以看出，指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性。也就是说，要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。

## 2.volatile关键字的两层语义

一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

 - 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。
 - 禁止进行指令重排序。

假如线程1先执行，线程2后执行：

```
//线程1
boolean stop = false;
while(!stop){
    doSomething();
}
```

```
//线程2
stop = true;
```

使用volatile修饰之后：

- 1、使用volatile关键字会强制将修改的值立即写入主存；

- 2、使用volatile关键字的话，当线程2进行修改时，会导致线程1的工作内存中缓存变量stop的缓存行无效（反映到硬件层的话，就是CPU的L1或者L2缓存中对应的缓存行无效）；

- 3、由于线程1的工作内存中缓存变量stop的缓存行无效，所以线程1再次读取变量stop的值时会去主存读取。

那么在线程2修改stop值时（当然这里包括2个操作，修改线程2工作内存中的值，然后将修改后的值写入内存），会使得线程1的工作内存中缓存变量stop的缓存行无效，然后线程1读取时，发现自己的缓存行无效，它会等待缓存行对应的主存地址被更新之后，然后去对应的主存读取最新的值。线程1读取到的就是最新的正确的值。

## 3.volatile保证内存可见性

对于可见性，Java提供了volatile关键字来保证可见性。

当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。

而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。

另外，通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性。

![image.png](https://upload-images.jianshu.io/upload_images/325120-e9fb52c50e7c2654.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 4.volatile禁止指令重排

volatile关键字提供内存屏障的方式来防止指令被重排，编译器在生成字节码文件时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。

在Java内存模型中说过，为了性能优化，编译器和处理器会进行指令重排序；也就是说java程序天然的有序性可以总结为：如果在本线程内观察，所有的操作都是有序的；如果在一个线程观察另一个线程，所有的操作都是无序的。在单例模式的实现上有一种双重检验锁定的方式（Double-checked Locking）。代码如下：

```
public class Singleton {
    private Singleton() { }
    private volatile static Singleton instance;
    public Singleton getInstance(){
        if(instance==null){
            synchronized (Singleton.class){
                if(instance==null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

这里为什么要加volatile了？我们先来分析一下不加volatile的情况，有问题的语句是这条：instance = new Singleton();
这条语句实际上包含了三个操作：

- 1.分配对象的内存空间；
- 2.初始化对象；
- 3.设置instance指向刚分配的内存地址。但由于存在重排序的问题，可能有以下的执行顺序：

![](http://swiftlet.net/wp-content/uploads/2018/11/20181123113924_13549.png)

如果2和3进行了重排序的话，线程B进行判断if(instance==null)时就会为true，而实际上这个instance并没有初始化成功，显而易见对线程B来说之后的操作就会是错的。而用volatile修饰的话就可以禁止2和3操作重排序，从而避免这种情况。volatile包含禁止指令重排序的语义，其具有有序性。

通过生成汇编代码，可以清晰的看到加入volatile和未加入volatile的差别。volatile变量修饰的共享变量，在进行写操作的时候会多出一个lock前缀的汇编指令，这个指令会触发总线锁或者缓存锁，通过缓存一致性协议来解决可见性问题。（可从Java内存模型简介了解缓存一致性协议）

```
0x01a3de1d:movb $0x0,0x1104800(%esi)  ; ...c6860048 100100
0x01a3de24:lock addl $0x0,(%esp)  ; ...f0830424 00
```

再比如下边的例子

![image.png](https://upload-images.jianshu.io/upload_images/325120-5d519da5984300cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 5.volatile如何保证有序性

- 在分析保证有序性前，有必要了解一下内存屏障。内存屏障（Memory Barriers，Intel称之Memory Fence）指令是指，重排序时不能把后面的指令重排序到内存屏障之前的位置。CPU把内存屏障分成三类：写屏障(store barrier)、读屏障(load barrier)和全屏障(Full Barrier)。

- 写屏障store barrier相当于storestore barrier, 强制所有在storestore内存屏障之前的所有执行，都要在该内存屏障之前执行，并发送缓存失效的信号。所有在storestore barrier指令之后的store指令，都必须在storestore barrier屏障之前的指令执行完后再被执行。

![](https://upload-images.jianshu.io/upload_images/3972450-2a45652d869b9be3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/351)

读屏障load barrier相当于loadload barrier，强制所有在load barrier读屏障之后的load指令，都在loadbarrier屏障之后执行。


![](https://upload-images.jianshu.io/upload_images/3972450-c5302d3aeefd18af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/330)


全屏障full barrier相当于storeload，是一个全能型的屏障，因为它同时具备前面两种屏障的效果。强制了所有在storeload barrier之前的store/load指令，都在该屏障之前被执行，所有在该屏障之后的的store/load指令，都在该屏障之后被执行。

![](https://upload-images.jianshu.io/upload_images/3972450-2b4276d2a17a677c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/340)

在JMM中把内存屏障指令分为4类：

- LoadLoad Barriers，load1 ; LoadLoad; load2 ，确保load1数据的装载优先于load2及所有后续装载指令的装载。
- StoreStore Barriers，store1; storestore;store2 ，确保store1数据对其他处理器可见优先于store2及所有后续存储指令的存储。
- LoadStore Barries， load1;loadstore;store2，确保load1数据装载优先于store2以及后续的存储指令刷新到内存。
- StoreLoad Barries， store1; storeload;load2， 确保store1数据对其他处理器变得可见， 优先于load2及所有后续装载指令的装载；这条内存屏障指令是一个全能型的屏障同时具有其他3条屏障的效果。

编译器在生成字节码时，会在volatile指令序列中插入内存屏障来禁止特定类型的处理器重排序。对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎不可能。为此，JMM采取保守策略。下面是基于保守策略的JMM内存屏障插入策略。

 - 在每个volatile写操作的前面插入一个StoreStore屏障。
 - 在每个volatile写操作的后面插入一个StoreLoad屏障。
 - 在每个volatile读操作的前面插入一个LoadLoad屏障。
 - 在每个volatile读操作的后面插入一个LoadStore屏障。






## 6.volatile无法保证原子性

```
public class Test {
    public volatile int inc = 0;
     
    public void increase() {
        inc++;
    }
     
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
         
        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

自增操作不是原子性操作，而且volatile也无法保证对变量的任何操作都是原子性的。

把上面的代码改成以下任何一种都可以达到效果：

（1）采用synchronized

```
public class Test {
    public  int inc = 0;
    
    public synchronized void increase() {
        inc++;
    }
    
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
        
        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

（2）采用Lock：

```
public class Test {
    public  int inc = 0;
    Lock lock = new ReentrantLock();
    
    public  void increase() {
        lock.lock();
        try {
            inc++;
        } finally{
            lock.unlock();
        }
    }
    
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
        
        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```
## 7.volatile的原理和实现机制
前面讲述了源于volatile关键字的一些使用，下面我们来探讨一下volatile到底如何保证可见性和禁止指令重排序的。

“观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：

 - 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
 - 它会强制将对缓存的修改操作立即写入主存；
 - 如果是写操作，它会导致其他CPU中对应的缓存行无效。
 
## 8.volatile适用场景
- synchronized关键字是防止多个线程同时执行一段代码，那么就会很影响程序执行效率，而volatile关键字在某些情况下性能要优于synchronized，是一种比synchronized 关键字更轻量级的同步机制。但是要注意volatile关键字是无法替代synchronized关键字的，因为volatile关键字无法保证操作的原子性。通常来说，使用volatile必须具备以下2个条件：

     - 对变量的写操作不依赖于当前值
     - 该变量没有包含在具有其他变量的不变式中
 
- 实际上，这些条件表明，可以被写入 volatile 变量的这些有效值独立于任何程序的状态，包括变量的当前状态。上面的2个条件需要保证操作是原子性操作，才能保证使用volatile关键字的程序在并发时能够正确执行。

 

## 9.思维导图如下：

![](https://upload-images.jianshu.io/upload_images/325120-9665950c3ee38180.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



