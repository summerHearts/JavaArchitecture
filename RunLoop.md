## RunLoop
- 1、什么是RunLoop
    1、 run loop是通过内部维护的事件循环来对事件/消息进行管理的一个对象。
       1、没有消息需要处理时，休眠以避免资源占用。
       2、有消息需要处理时，立即被唤醒。
    2、事件循环，绝对不止是死循环这么简单的一个回答。实质上就是runloop内部状态的转换。
       1.用户态：应用程序都是在用户态，平时开发用到的api等都是用户态的操作
       2.内核态：系统调用，牵涉到操作系统，底层内核相关的指令。
- 2、RunLoop的数据结构
  - NSRunloop是CFRunLoop的封装，提供了面向对象的API。
     - 1.CFRunloop
     - 2.CFRunloopMode
     - 3.Source/Timer/Observer
  - CFRunLoop主要包括 
     - 1、pthead ----> 一一对应，runloop内部一个线程
     - 2、currentMode ---->   当前mode  CFRunloopMode数据结构
     - 3、models  modes多个model的集合 ----> NSMutableSet<CFRunLoopMode*>
     - 4、commonModels ----> NSMutableSet<NSString*>
     - 5、commonModeltems ----> Observer,Timer,Source

   - CFRunLoopMode主要结构
   
     ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-122_14-47-53.jpg)
     
     - name  字符串  比如NSDefaultRunloopMode  别名定义，字符串名称 去找到对应的mode
     
     - source0,source1 都是无序集合
         - source0  需要手动唤醒线程  非基于Port的，用于用户主动触发的事件 
    常见的source事件：比如用户点击按钮，拖拽，手势等事件。
         - source1  具备唤醒线程能力 通过内核和其它线程相互发送消息
     -  timers都是数组 
     -  Observers   CFRunloopObserver观测时间点
     
         - kCFRunloopEntry          准备启动runloop
         
         - KCFRunloopBeforeTimers   通知观察者将要处理timer了
         - kCFRunloopBeforeSources  通知观察者将要处理source了
         - kCFRunloopBeforeWaiting  将要休眠了 之后就是用户态切换到内核态
         - kCFRunloopAfterWaiting   用户态切换到内核态不久
         - kCFRunloopExit           runloop退出
         - kCFRunloopAllActivities  观察所有事件
         
     - 3.modes    NSMutableSet<CFRunloopMode *>的集合
     - 4.commonModes  NSMutableSet<NSString *>内部的每一个字符串的名称对应了每一中Mode。

         - 1、kCFRunLoopDefaultMode：App的默认Mode，通常主线程是在这个Mode下运行
         
         - 2、UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView追踪触摸滑动，保证界面滑动时不受其他Mode 影响
         - 3、UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用
         - 4、GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到
         - 5、kCFRunLoopCommonModes: 这是一个占位用的Mode，不是一种真正的Mode

     - 5.commonModeItems
         -  里面有多个Observer，Timer，Source


- 3、各个数据结构的关系
  - 1.一个runloop对应了多种mode   ，每个mode下又有多种source，timer。Observer
  
  - 2.每次RunLoop启动时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode
  - 3.如果需要切换Mode，只能退出Loop，再重新指定一个Mode进入,这样做主要是为了分隔开不同组的Source/Timer/Observer，让其互不影响

     ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-122_15-22-23.jpg)

     runloop在同一时间，只能处理，一种mode下的事件
     
   - 由此引入一个面试经常会问到的问题：
      
     * 问题： runloop在同一时间，只能处理，一种mode下的事件*
     * 答案： imer的mode从defaultmode，换到CommonMode就可以了
    
    那么为什么切换就可以了呢，这里就要说到CommonMode了
    
    

- 4、CommonMode
       
 ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-122_15-26-43.jpg)      
 ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-122_15-26-56.jpg)
 
  CommonMode并不是一种实际的模式，和defaultmode完全不是一回事。简单的理解就是，如果设置了CommonMode模式，那么runloop在切换mode的同时，也把timer的事件也带走了，所以无论切到哪种mode下，timer的事件都是可以处理的。

 此时，我们就可以回答为什么放在commonMode下就可以执行的原因了

   - 1.默认滚动事件和timer事件都是在kCFRunloopDefaultMode下的
   - 2.此时tableView一滚动，mode切换到UITrackingRunloopMode中，上面说，runloop同一时间只能处理一种mode的事件，那么在DefaultMode的timer就无法响应了。
   - 3.CommonMode又是一个同步多种mode的技术方案，此模式下，当runloop到UITrackingRunloopMode的时候，他把timer的事件也转移到UITrackingRunloopMode下，那么timer就能走了。

   
另外扩展一下： 订单倒计时的问题，比如说，我下了一个订单，在订单支付界面有倒计时，那么如果此时App进入后台，定时器就不工作了。在服务器请求任务繁重的时候，如果本地判断订单是否超时的问题呢
   
  ![](https://upload-images.jianshu.io/upload_images/325120-79cac009031ce7ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)
  
 首先，需要将App长时间挂起，保证不被杀死。可以在进入后台的时候一直执行某个操作即可。
 其次，GCD的定时器不受Runloop的mode的影响。所以可以使用下边的方式解决
 
![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-122_15-42-49.jpg)
  



- 5、CommonMode不是一种实际的模式 
  
   源码分析：VoidCFRunloopAddTimer（runloop,timer,commoMode）
   这个方法流程：
     - 1.大致意思就是，如果是commonmodes将runloop和commonmode封装成一个context。
     
     - 2.commonModes  NSMutableSet<NSString *>内部的每一个字符串的名称对应了每一中Mode。在介绍下，这是一个集合。也就是一个set，里面包含了所有mode下的字符串。
     - 3.将set，和contex作为参数，调用__CFRunloopAddItemToCommonModes方法。
     - 4.这里面我们能取到runloop具体在什么mode下，然后再重新调用VoidCFRunloopAddTimer（runloop,timer,commoMode）。
     - 注意：这里不会进行循环调用，因为此时标记的CommonMode已经变成了具体的UITrackingRunloopMode，我们将timer加到UITrackingRunloopMode，timer就能响应了。


- 6、四：事件循环的实现机制

  当你被问到，runloop具体操作流程的时候，你该怎么回答。
  
  当我们手动点击屏幕，UI操作，实际runloop是这样来的

  ![](https://img-blog.csdn.net/20180427182835206)
  
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-122_15-49-55.jpg)
  
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-122_15-51-43.jpg)

  那么内核态和用户态的转换，实质是调用了mach_msg()函数。


- 7 、runloop和多线程之间的关系。怎么实现一个常驻线程。

  - 1、为当前线程开启一个RunLoop
  - 2、向该RunLoop中添加一个 port/source等维持RunLoop的事件循环。
  - 3、启动RunLoop

   ![](https://file.zsxq.com/1b3/46/b346d19963f2c4607a27c66aa1cb7b2ea8c3616bb1a52957de6088e7e317cbd0.jpg)
   
   
- 8、如何保证子线程数据回来更新UI的时候不打断用户的滑动操作？
 
   - tableView在滑动，处于UITrackingRunloopMode模式下。
   - 子线程请求的数据，那么在和主线程处理的时候，我们将更新的逻辑加载defaultMode下。那么defaultMode下的操作是不会执行的。
   - 滑动结束了，runloop由UITrackingRunloopMode又回到defaultMode下了，那么defaultMode下的更新操作就能执行了

