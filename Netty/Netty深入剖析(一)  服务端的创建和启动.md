##Netty深入剖析(一)  服务端的创建和启动

- Netty是什么

   - 1、异步事件驱动框架，用于快速开发高性能服务端和客户端
   
   - 2、封装了JDK底层BIO和NIO模型，提高高度可用的API 
   
   - 3、自带编码解码器解决拆包沾包问题，用户只需要关注业务逻辑

   - 4、精心设计的reactor线程模型支持高并发海量连接

   - 5、自带各种协议栈让你处理任何一种通信协议都几乎不用亲自动手
   
- 目前使用Netty的框架都有哪些呢？

 ```
 Dubbo、RocketMQ 、Spark 、Elasticsearch 、Netty-SocketIO 、 Spring5 、 Grpc 
 ```

- Netty服务端的启动


       -  1、调用ChannelFactory对象生成ServerBootstrap 的channel(Class<? extends C> channelClass)方法中设置的channelClass对象即NioServerSocketChannel对象，也就是Netty重新封装过的ServerSocketChannel对象。
       
       -  2、 初化NioServerSocketChannel对象，将我们在创建ServerBootstrap对象中设置的option 和attr值设置到NioServerSocketChannel对象中的config和arrs属性中。
       
       -  3、生成ChannelPromise对象，这个对象主要是对channel的注册状态进行监听
       
       -  4、 取eventLoop设置到channel中，并调用AbstractChannel$AbstractUnsafe的register0方法奖channel注册到eventLoop中的selector，并将ChannelPromise状态设置为成功：

    ![](https://www.icheesedu.com/images/qiniu/Xnip2018-06-168_11-42-34.png)
    
    ```
     ChannelFuture f = b.bind(8888).sync();
     
     |
     V
     
     this.doBind(localAddress);
     
     |
     V
     
     ChannelFuture regFuture = this.initAndRegister();
     
     |
     V
     
     channel = this.channelFactory.newChannel();
     
     |
     V
     
     this.channelFactory((io.netty.channel.ChannelFactory)(new ReflectiveChannelFactory(channelClass)));
     
     |
     V
     
     newChannel---> (Channel)this.clazz.newInstance();

    ```
    
    ![](https://www.icheesedu.com/images/qiniu/Xnip2018-06-168_11-32-35.png)
    
    ```
    this(newSocket(DEFAULT_SELECTOR_PROVIDER));
    
    |
    V
    
    provider.openServerSocketChannel();
    
    |
    V
    
    NioServerSocketChannelConfig(this, this.javaChannel().socket());
    
    |
    V
    
    ch.configureBlocking(false);
    
    |
    V
    
    protected AbstractChannel(Channel parent) {
        this.parent = parent;
        this.id = this.newId();
        this.unsafe = this.newUnsafe();
        this.pipeline = this.newChannelPipeline();
    }
    ```
    
    ![](https://www.icheesedu.com/images/qiniu/Xnip2018-06-168_11-43-52.png)
    
    ```
     final Map<ChannelOption<?>, Object> options = options0();
        synchronized (options) {
            channel.config().setOptions(options);
        }
     }
     
     |
     V
     
    synchronized (childOptions) {
            currentChildOptions = childOptions.entrySet().toArray(newOptionArray(childOptions.size()));
    }
    synchronized (childAttrs) {
            currentChildAttrs = childAttrs.entrySet().toArray(newAttrArray(childAttrs.size()));
    }
 
     |
     V
    
     ChannelHandler handler = config.handler();
     pipeline.addLast(handler);
     
     |
     V
     
    ch.eventLoop().execute(new Runnable() {
                    @Override
                    public void run() {
                        pipeline.addLast(new ServerBootstrapAcceptor(
                                currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                    }
    });
    ```
    
   注册selector
   
       ```
        private void register0(ChannelPromise promise) {
            try {
                // check if the channel is still open as it could be closed in the mean time when the register
                // call was outside of the eventLoop
                if (!promise.setUncancellable() || !ensureOpen(promise)) {
                    return;
                }
                boolean firstRegistration = neverRegistered;
                doRegister();
                neverRegistered = false;
                registered = true;

                // Ensure we call handlerAdded(...) before we actually notify the promise. This is needed as the
                // user may already fire events through the pipeline in the ChannelFutureListener.
                pipeline.invokeHandlerAddedIfNeeded();

                safeSetSuccess(promise);
                pipeline.fireChannelRegistered();
                // Only fire a channelActive if the channel has never been registered. This prevents firing
                // multiple channel actives if the channel is deregistered and re-registered.
                if (isActive()) {
                    if (firstRegistration) {
                        pipeline.fireChannelActive();
                    } else if (config().isAutoRead()) {
                        // This channel was registered before and autoRead() is set. This means we need to begin read
                        // again so that we process inbound data.
                        //
                        // See https://github.com/netty/netty/issues/4805
                        beginRead();
                    }
                }
            } catch (Throwable t) {
                // Close the channel directly to avoid FD leak.
                closeForcibly();
                closeFuture.setClosed();
                safeSetFailure(promise, t);
            }
        }
       ```
   
   ![](https://www.icheesedu.com/images/qiniu/Xnip2018-06-168_11-59-54.png)
   
   绑定端口
   
  通过initAndRegister方法将serverchannel注册到selector后调用doBind0方法注册端口
  
  ```
  private static void doBind0(  
            final ChannelFuture regFuture, final Channel channel,  
            final SocketAddress localAddress, final ChannelPromise promise) {  

  
        channel.eventLoop().execute(new Runnable() { //(1)  
            @Override  
            public void run() {  
                if (regFuture.isSuccess()) {  
                    channel.bind(localAddress, promise).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);  
                } else {  
                    promise.setFailure(regFuture.cause());  
                }  
            }  
        });  
}  
  ```
   
   
   
      
  
 
 


