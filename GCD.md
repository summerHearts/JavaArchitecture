##1、GCD简介
- Grand Central Dispatch(GCD) 是 Apple 开发的一个多核编程的较新的解决方法。它主要用于优化应用程序以支持多核处理器以及其他对称多处理系统。它是一个在线程池模式的基础上执行的并发任务。

- GCD是一个替代诸如NSThread等技术的很高效和强大的技术。GCD完全可以处理诸如数据锁定和资源泄漏等复杂的异步编程问题。GCD的工作原理是让一个程序，根据可用的处理资源，安排他们在任何可用的处理器核心上平行排队执行特定的任务。这个任务可以是一个功能或者一个程序段。
 
 ![GCD相关的一些基本概念.png](https://upload-images.jianshu.io/upload_images/325120-f5a8069d80a31cad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)
 
 [GCD整体结构](http://naotu.baidu.com/file/e35c8bfc41ad0b4da9a1bf4d2df72ff0?token=b600d7a92fe01fb7)
 
##2、GCD优点：

- 可用于多核的并行运算

- 会自动利用更多的 CPU 内核（比如双核、四核）
- 会自动管理线程的生命周期（创建线程、调度任务、销毁线程）
- 程序员只需要告诉 GCD 想要执行什么任务，不需要编写任何线程管理代码

##3、GCD 任务和队列

- 任务：执行操作的意思，换句话说就是你在线程中执行的那段代码。在 GCD 中是放在 block 中的。执行任务有两种方式：同步执行（sync）和异步执行（async）。

   - `同步执行（sync）`：同步添加任务到指定的队列中，在添加的任务执行结束之前，会一直等待，直到队列里面的任务完成之后再继续执行。只能在当前线程中执行任务，不具备开启新线程的能力。
   
   - `异步执行（async）`：异步添加任务到指定的队列中，它不会做任何等待，可以继续执行任务。可以在新的线程中执行任务，具备开启新线程的能力。
   - 备注：`异步执行（async）`虽然具有开启新线程的能力，但是并不一定开启新线程。这跟任务所指定的队列类型有关。
   - 两者的主要区别是：是否等待队列的任务执行结束，以及是否具备开启新线程的能力。

- 队列:这里的队列指执行任务的等待队列，即用来存放任务的队列。队列是一种特殊的线性表，采用 `FIFO（先进先出）`的原则，即新任务总是被插入到队列的末尾，而读取任务的时候总是从队列的头部开始读取。每读取一个任务，则从队列中释放一个任务。队列的结构可参考下图：

  ![](https://upload-images.jianshu.io/upload_images/1933747-133958d375f52a96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
   - `串行队列（Serial Dispatch Queue）`：每次只有一个任务被执行。让任务一个接着一个地执行。（只开启一个线程，一个任务执行完毕后，再执行下一个任务）
   
     ![](https://upload-images.jianshu.io/upload_images/1933747-e9cca07d70322d1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
     
   - `并发队列（Concurrent Dispatch Queue）`：可以让多个任务并发（同时）执行，可以开启多个线程，并且同时执行任务
  
     ![](https://upload-images.jianshu.io/upload_images/1933747-944bb1286f287992.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
     
     - 备注：并发队列的并发功能只有在`异步（dispatch_async）`函数下才有效

关于同步异步、串行并行和线程的关系，下面通过一个表格来总结：

![](https://upload-images.jianshu.io/upload_images/4139766-db407f51ea0f83d2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
##4、GCD 的使用步骤

- 创建一个队列（串行队列或并发队列）

- 将任务追加到任务的等待队列中，然后系统就会根据任务类型执行任务（同步执行或异步执行）  

**4.1、队列的创建方法/获取方法**

 可以使用dispatch_queue_create来创建队列，需要传入两个参数，第一个参数表示队列的唯一标识符，用于 DEBUG，可为空，Dispatch Queue 的名称推荐使用应用程序 ID 这种逆序全程域名；第二个参数用来识别是串行队列还是并发队列。
 
 - `DISPATCH_QUEUE_SERIAL` 表示串行队列
 - `DISPATCH_QUEUE_CONCURRENT` 表示并发队列

 ![](https://upload-images.jianshu.io/upload_images/325120-ed4bb01c5ea4aa77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- 对于串行队列，GCD 提供了的一种特殊的串行队列：`主队列（Main Dispatch Queue）`。
所有放在主队列中的任务，都会放到主线程中执行。

  可使用`dispatch_get_main_queue()`获得主队列。
  
  ![](https://upload-images.jianshu.io/upload_images/325120-efa9ea275413dddf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
- 对于并发队列，GCD 默认提供了全局并发队列`（Global Dispatch Queue）`。

  可以使用`dispatch_get_global_queue`来获取。需要传入两个参数。第一个参数表示队列优先级，一般用DISPATCH_QUEUE_PRIORITY_DEFAULT。第二个参数暂时没用，用0即可。
  
  ![](https://upload-images.jianshu.io/upload_images/325120-080038a8be208f32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

**4.2、 任务的创建方法**

GCD 提供了同步执行任务的创建方法`dispatch_sync`和异步执行任务创建方法`dispatch_async`。

![](https://upload-images.jianshu.io/upload_images/325120-e3560ca6a53fd84a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

不同组合方式

![](https://upload-images.jianshu.io/upload_images/325120-015c99392daa4106.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

##5、 栅栏方法(dispatch_barrier_async和dispatch_barrier_sync) 

实现多读单写

![](https://upload-images.jianshu.io/upload_images/325120-57457aff9126fcfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](https://upload-images.jianshu.io/upload_images/325120-b562cf15d1fdbdce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

```
输出的结果：
2018-09-21 14:36:35.946008+0800 GCDDemo[3665:625310] aa, <NSThread: 0x600002f013c0>{number = 1, name = main}
2018-09-21 14:36:35.946012+0800 GCDDemo[3665:625532] ----1-----<NSThread: 0x600002f57dc0>{number = 3, name = (null)}
2018-09-21 14:36:35.946027+0800 GCDDemo[3665:625531] ----2-----<NSThread: 0x600002f5d3c0>{number = 4, name = (null)}
2018-09-21 14:36:35.946139+0800 GCDDemo[3665:625310] bb, <NSThread: 0x600002f013c0>{number = 1, name = main}
2018-09-21 14:36:35.946161+0800 GCDDemo[3665:625531] ----barrier-----<NSThread: 0x600002f5d3c0>{number = 4, name = (null)}
2018-09-21 14:36:35.946258+0800 GCDDemo[3665:625532] ----4-----<NSThread: 0x600002f57dc0>{number = 3, name = (null)}
2018-09-21 14:36:35.946277+0800 GCDDemo[3665:625531] ----3-----<NSThread: 0x600002f5d3c0>{number = 4, name = (null)}
```

从打印中可以总结如下：

 - aa和bb都在主线程进行输出。
 
 - 先执行完barrier之前的任务，然后再执行自己的任务（barrier），最后执行barrier之后的任务（注意这里说的是任务不是下一行代码）。
 - 将自己的任务（barrier）插入到队列之后，不会等待自己的任务结束，它会继续把后面的任务（3、4）插入到队列，然后执行任务。

 ![](https://upload-images.jianshu.io/upload_images/325120-56f3742a2fa5d524.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

从打印中可以总结如下：

 - aa和bb都在主线程进行输出。
 
 - barrier队列里面任务在主线程中执行。
 - 先执行完barrier之前的任务，然后再执行自己的任务（barrier），最后执行barrier之后的任务（注意这里说的是任务不是下一行代码）。✨✨✨
 - 需要等待自己的任务（barrier）结束之后，才会继续添加并执行写在barrier后面的任务（3、4），然后执行后面的任务。✨✨✨
 - 注意点：在使用栅栏函数时，需使用自定义队列才有意义,如果用的是串行队列或者系统提供的全局并发队列,这个栅栏函数的作用等同于一个同步函数的作用。

 
不同点：

 - `dispatch_barrier_async`将自己的任务（barrier）插入到队列之后，不会等待自己的任务结束，它会继续把后面的任务（3、4）插入到队列，然后执行任务。
 
 - `dispatch_barrier_sync`需要等待自己的任务（barrier）结束之后，才会继续添加并执行写在barrier后面的任务（3、4），然后执行后面的任务。

##6、GCD队列组（dispatch_group）

- 1、dispatch_group_notify

- 2、dispatch_group_wait
- 3、dispatch_group_enter和dispatch_group_leave

![](https://upload-images.jianshu.io/upload_images/325120-c7a3199bdd8d077d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

##7、GCD信号量（dispatch_semaphore）

![GCD 信号量：dispatch_semaphore.png](https://upload-images.jianshu.io/upload_images/325120-bce807a95de0ecfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

##常见面试题

- 1、已经在主线程了，还同步向主线程扔了一个任务

![](https://upload-images.jianshu.io/upload_images/325120-a68cf22289eedf7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

假设当前我们的代码正在queue0中执行。然后我们调用disptach_sync将一个任务block1扔到queue0中执行，`dispatch_sync(queue0, block1);`

这时，dispatch_sync将等待queue0排队执行完block1，然后才能继续执行下一行代码。But，当前代码执行的环境也是queue0。假设当前执行的任务为block0。也就是说，block0在执行到一半时，需要等到自己的下一个任务block1执行完，自己才能继续执行。而block1排队在后面，需要等block0执行完才能执行。这时死锁就产生了，block0和block1互相等待执行，当前线程就卡死在dispatch_sync这行代码处。

安全方法

YYKit中提供了一个同步扔任务到主线程的安全方法：

![](https://upload-images.jianshu.io/upload_images/325120-8284955df66a0035.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

其方式就是在扔任务给主线程之前，先检查当前线程是否已经是主线程，如果是，就不用调用GCD的队列调度接口dispatch_sync了，直接执行即可；如果不是主线程，那么调用GCD的dispatch_sync也不会卡死。

但事实上并不是这样的，dispatch_sync_on_main_queue也可能会卡死，这个安全接口并不安全。这个接口只能保证两个block之间不因互相等待而死锁。多于两个block的互相依赖就束手无策了。

举个例子，假设queue0是一个子线程的队列：

![](https://upload-images.jianshu.io/upload_images/325120-a3ef1e5ea5ec2fcc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

在上述代码中，block0正在主线程中执行，并且同步等待子线程执行完block1。block1又同步等待主线程执行完block2。而当前主线程正在执行block0，即block2的执行需要等到block0执行完。这样就成了`block0-->block1-->block2-->block0...`这样一个循环等待，即死锁。由于block1的环境是子线程，所以安全API的线程判断不起任何作用。

![](https://upload-images.jianshu.io/upload_images/325120-1fc788bc3c621f51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


由于通知NSNotification的执行是同步的，这里会出现和上一例一样的死锁情况：`block0-->block1-->block2-->block0...`

- 2、输入顺序

**串行队列+同步执行**

 - 特点：不会开启新线程，在当前线程执行任务，并且任务是串行的，执行完一个任务，再执行下一个任务。
 
![](https://upload-images.jianshu.io/upload_images/325120-b5c86806d5bf73c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

viewDidLoad是在主队列中主线程执行的。同步提交一个任务到串行队列，此时，该任务将会在主线程执行完毕，然后继续执行主队列中的其他操作。

从打印中可以总结如下几点：

  - 所有任务都是在主线程中执行的，并没有开启新的线程（同步执行不具备开启新线程的能力），由于串行队列，所以按顺序一个一个执行下去。
  
  - 所有任务都在打印的begin-- 和end-- 之间，这说明任务是添加到队列中马上执行的，而且同步任务需要等待队列的任务执行结束,才可以执行下一个任务。
  - 任务是按顺序执行的，这说明串行队列每次只有一个任务被执行，任务一个接一个按顺序执行下去。

**串行队列+异步执行**

- 特点：会开启新线程，但是因为任务是串行的，所以执行完一个任务之后，才会再执行下一个任务。

![](https://upload-images.jianshu.io/upload_images/325120-a4bf10d4bc3778a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

从打印中可以总结如下几点：

  - 开启了一条新线程(异步执行具备开启新线程的能力，但是串行队列只能开启一个线程)，但是任务由于是串行的，所以任务还是一个一个的执行下去（串行队列每次只能有一个任务被执行，任务一个接着一个顺序执行）。
  
  - 所有任务是在打印的begin-- 和end-- 之后才开始执行的,说明任务是将所有任务添加到队列之后才开始同步执行，而不是马上执行（异步执行不会做任何等待，可以继续执行任务）。


**并发队列+同步执行**

![](https://upload-images.jianshu.io/upload_images/325120-d102c6f5330e3888.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

```
GCD[2591:161026] 1
GCD[2591:161026] 2
GCD[2591:161026] 3
GCD[2591:161026] 4
GCD[2591:161026] 5
```
从打印中可以总结如下几点：

 - 所有任务都是在当前线程（主线程）中执行的，没有开启新的线程（同步执行不具备开启新线程的能力）。
 - 所有任务按顺序执行的。
 - 所有任务都在打印的begin--和end--之间，由于同步任务需要等待队列的任务执行结束才能执行下一个任务。

疑问点：并发队列具备开启多个线程能力为什么没不能同时执行任务呢？

  - 答：虽然并发队列可以开启多个线程，并且可以同时执行多个任务。但是其本身不能创建新线程，因为同步任务不具备开启新线程的能力，所以只有当前线程这一个线程，所以也就不存在并发。而且当前线程只有等待当前队列中正在执行的任务执行完毕之后，才能继续接着执行下面的操作（同步任务需要等待队列的任务执行结束）。所以任务只能一个接一个按顺序执行，不能同时被执行。

**并发队列+异步执行**

特点：可以开启多个线程，任务交替（同时）执行。

![](https://upload-images.jianshu.io/upload_images/325120-209cf2dd4173eb1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

从打印中可以总结如下几点：

 - 除了主线程，又开启了3个线程，并且任务是交替着同时执行的,由于异步执行具备开启新线程的能力,且并发队列可开启多个线程，同时执行多个任务.
 
 - 所有任务是在打印的begin-- 和end-- 之后才开始执行的,说明任务不是马上执行，而是将所有任务添加到队列之后才开始异步执行,另外当前线程并没有等待，而是直接开启了新的线程，在新线程中执行任务,由于异步执行不做等待，所以可以继续执行其他任务.

**主队列+同步执行**

 - 特点：1.主线程调用：互等卡主不执行。 2.其他线程调用：不会开启新线程，执行完一个任务，再执行下一个任务。

![](https://upload-images.jianshu.io/upload_images/325120-d068212ca7c0af99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

从打印中可以总结如下：

  - 程序崩溃，只打印出begin-- 主队列 + 同步执行
  - 疑问点:为什么任务没有执行完，而且程序了崩溃呢？
  - 答：我们在主线程中执行syncMain方法，相当于把syncMain任务放到了主线程的队列中，主线程正在处理syncMain这个任务 而syncMain方法中又有同步事件需要处理 ,造成会相互等待,所以死锁了，所以任务不会执行完。

 其他线程调用
 
![](https://upload-images.jianshu.io/upload_images/325120-5cb822efa7a692a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

 从打印中可以总结如下：

  - 所有任务都是在主线程中执行的，并没有开启新的线程（所有放在主队列中的任务，都会放到主线程中执行），而且由于主队列是串行队列，所以按顺序一个一个执行。
  
  - 所有任务都在打印的begin-- 和end-- 之间，这说明任务是添加到队列中马上执行的。
  - 疑问点:为什么在其他线程调用不会崩溃卡住呢？
  - 答：因为syncMain任务放到了其他线程里，而syncMain方法中的几个任务都追加到主队列中，因为主队列现在没有正在执行的任务，所以会直接执行主队列的任务，一个个执行下去，所以这里不会卡住线程。

**主队列+异步执行**

 - 特点:只在主线程中执行任务，执行完一个任务，再执行下一个任务。

 ![](https://upload-images.jianshu.io/upload_images/325120-b2054ea32df755ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

从打印中可以总结如下：

 - 1.所有任务都是在主线程中执行的，并没有开启新的线程,虽然异步执行具备开启线程的能力，但因为是主队列，所以所有任务都在主队列(主队列是串行队列)中,并且一个接一个的执行下去.
 
 - 2.所有任务是在打印的begin-- 和end-- 之后才开始执行的，说明任务并不是马上执行，而是将所有任务添加到队列之后才开始同步执行。


**PerformSelector**

![](https://upload-images.jianshu.io/upload_images/325120-bc3096332a765e89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

`PerformSelector` 相关,同步在当前线程执行的。会阻塞当前线程。可在主线程或者子线程执行。

`PerformSelector:afterDelay` 任务会在默认的系统底层的一个线程上执行，而这个线程是没有开启runLoop。需要一个提交到runloop的逻辑的，所以该方法会失效。

```
GCD[2672:168690] 4
GCD[2672:168728] 1
GCD[2672:168728] 3
```

**输出题(自行检测)**

![](https://upload-images.jianshu.io/upload_images/325120-ce0db528aa95f06a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

```
GCD[1874:105153] 1
GCD[1874:105153] 5
GCD[1874:105241] 2
```
上图解释如下：因为是异步并发，创建了子线程执行任务。之后由于在主队列提交了一个同步任务，主线程的ViewDidLoad需要等待该同步任务的执行完成，同时该任务也需要主线程执行完成，产生死锁。

![](https://upload-images.jianshu.io/upload_images/325120-3dfce94cfbcdf861.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

```
GCD[1918:108523] 1
GCD[1918:108523] 5
GCD[1918:108602] 2
GCD[1918:108602] 3
GCD[1918:108602] 4
```
上图解释如下：在执行完任务的子线程中，随后模式是 串行+同步，按顺序执行，先执行2，然后提交在串行队列的同步任务，等任务返回，再执行之后的任务。

![](https://upload-images.jianshu.io/upload_images/325120-76364a7ab6681080.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

```
GCD[1918:108523] 1
GCD[1918:108523] 5
GCD[1918:108602] 2
GCD[1918:108602] 3
GCD[1918:108602] 4
```
上图解释如下：并发队列+异步执行，创建子线程执行任务。在子线程执行主队列的同步任务。不会影响主线程的执行。

![](https://upload-images.jianshu.io/upload_images/325120-4f6999a159268364.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

程序崩溃，出现了死锁。串行队列+异步执行，会开启新线程，但是因为任务是串行的，所以执行完一个任务之后，才会再执行下一个任务。然后在串行队列中又有任务，产生死锁。

**dispatch_group_enter和dispatch_group_leave**

 - 特点：`dispatch_group_enter`和`dispatch_group_leave`总是成对出现的。
 - dispatch_group_enter:用于添加对应任务组中的未执行完毕的任务数，执行一次，未执行完毕的任务数加1，当未执行完毕任务数为0的时候，才会使dispatch_group_wait解除阻塞和dispatch_group_notify的block执行。
 - dispatch_group_leave:用于减少任务组中的未执行完毕的任务数，执行一次，未执行完毕的任务数减1，dispatch_group_enter和dispatch_group_leave要匹配，不然系统会认为group任务没有执行完毕。













