##1、简介

- NSOperation、NSOperationQueue 是苹果提供给我们的一套多线程解决方案。实际上 NSOperation、NSOperationQueue 是基于 GCD 更高一层的封装，完全面向对象。

- NSOperation提供任务的封装，NSOperationQueue顾名思义，执行队列，可以自动实现多核并行计算，自动管理线程的生命周期，如果是并发的情况，其底层也使用线程池模型来管理。

- 优点：
  - 1、添加任务依赖`addDependency`
   
  - 2、任务执行状态控制
     - isReady

     - isExecuting
     - isFinished
     - isCancelled

       如果只重写**main**方法，底层控制变更任务执行完成状态，以及任务退出。
    
       如果重写**start**方法，自行控制任务状态。

  - 3、控制最大并发量

##2、NSOperation

- 操作对象（operation object）是NSOperation类的实例，你能够使用它执行应用中你想执行的任务。NSOperation是一个抽象的基类，开发中我们很少使用，一般使用它的子类`NSInvocationOpeation`和`NSBlockOperation`或者继承`NSOperation自定义子类`来执行任务。虽然是基类但是`NSOperation`实现了很多重要的逻辑来确保执行任务的安全，我们只需要专注实际的任务即可。

- NSOperation是与单个任务关联的代码和数据的抽象类，任务捆绑在NSOperation实例里，可以简单地认为NSOperation是单个的工作单元。
- 它在使用上，更加符合面向对象的思想，更加方便的为任务添加依赖关系，同时提供了四个支持KVO监听的代表当前任务执行状态的属性`cancelled`、`executing`、`finished`、`ready`。NSOperation内部对这四个状态行为作了预处理，根据任务的不同状态这四个属性的值会自动改变。



![](https://upload-images.jianshu.io/upload_images/325120-e7b8686917609f9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](https://upload-images.jianshu.io/upload_images/325120-f35d990ae76419ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](https://upload-images.jianshu.io/upload_images/325120-44be291faf502bce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)



