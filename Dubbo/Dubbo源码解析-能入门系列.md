## 1、ZooKeeper 可视化工具 zkui
- 1、简介zkui它提供了一个管理界面，可以针对zookeepr的节点值进行CRUD操作，同时也提供了安全认证。

- 2、下载安装

  - 1、下载地址 https://github.com/DeemOpen/zkui

  - 2、mvn clean install，执行前需要安装 java 环境，maven环境，执行成功后会生成一个jar文件。

  - 3、将config.cfg复制到上一步生成的jar文件所在目录，然后修改配置文件中的zookeeper地址。
  
     ```
     zkServer=localhost:2181
     ```
  - 4、执行运行命令
  
     ```
     nohup java -jar zkui-2.0-SNAPSHOT-jar-with-dependencies.jar &
     ```
     
  - 5、测试，http://localhost:9090，如能看到如下页面则代表zookeeper安装运行正常。

     ```
     username:admin
     password:manager
     ```

     ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_11-32-48.png)
     
     ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_11-33-01.png)
      
  

        ```
        Register: dubbo://192.168.103.40:20880/com.of.wangpu.api.service.HelloService?anyhost=true&application=dubbo-provider&dubbo=2.4.10&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&pid=2054&side=provider&timestamp=1535511638784, dubbo version: 2.4.10, current host: 127.0.0.1
        
        Subscribe: provider://192.168.103.40:20880/com.of.wangpu.api.service.HelloService?anyhost=true&application=dubbo-provider&category=configurators&check=false&dubbo=2.4.10&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&pid=2054&side=provider&timestamp=1535511638784, dubbo version: 2.4.10, current host: 127.0.0.1
        2018-08-29 11:00:39.080  INFO 2054 --- 
        
        Notify urls for subscribe url provider://192.168.103.40:20880/com.of.wangpu.api.service.HelloService?anyhost=true&application=dubbo-provider&category=configurators&check=false&dubbo=2.4.10&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&pid=2054&side=provider&timestamp=1535511638784, urls: [empty://192.168.103.40:20880/com.of.wangpu.api.service.HelloService?anyhost=true&application=dubbo-provider&category=configurators&check=false&dubbo=2.4.10&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&pid=2054&side=provider&timestamp=1535511638784], dubbo version: 2.4.10, current host: 127.0.0.1
        2018-08-29 11:00:39.224  INFO 2054 ---
    ```
    
        ```
        Register: consumer://192.168.103.40/ com.of.wangpu.api.service.HelloService?application=dubbo-provider&category=consumers&check=false&connected=true&dubbo=2.4.10&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&owner=dubbo-provider&pid=2077&side=consumer&timestamp=1535511731989, dubbo version: 2.4.10, current host: 192.168.103.40
    
        Subscribe: consumer://192.168.103.40/com.of.wangpu.api.service.HelloService?application=dubbo-provider&category=providers,configurators,routers&connected=true&dubbo=2.4.10&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&owner=dubbo-provider&pid=2077&side=consumer&timestamp=1535511731989, dubbo version: 2.4.10, current host: 192.168.103.40
    
     Notify urls for subscribe url consumer://192.168.103.40/com.of.wangpu.api.service.HelloService?application=dubbo-provider&category=providers,configurators,routers&connected=true&dubbo=2.4.10&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&owner=dubbo-provider&pid=2077&side=consumer&timestamp=1535511731989, urls: [dubbo://192.168.103.40:20880/com.of.wangpu.api.service.HelloService?anyhost=true&application=dubbo-provider&dubbo=2.4.10&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&pid=2054&side=provider&timestamp=1535511638784, empty://192.168.103.40/com.of.wangpu.api.service.HelloService?application=dubbo-provider&category=configurators&connected=true&dubbo=2.4.10&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&owner=dubbo-provider&pid=2077&side=consumer&timestamp=1535511731989, empty://192.168.103.40/com.of.wangpu.api.service.HelloService?application=dubbo-provider&category=routers&connected=true&dubbo=2.4.10&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&owner=dubbo-provider&pid=2077&side=consumer&timestamp=1535511731989], dubbo version: 2.4.10, current host: 192.168.103.40
    
      Successed connect to server /192.168.103.40:20880 from NettyClient 192.168.103.40 using dubbo version 2.4.10, channel is NettyChannel [channel=[id: 0x27f1bbe0, /192.168.103.40:53619 => /192.168.103.40:20880]], dubbo version: 2.4.10, current host: 192.168.103.40
    
      Start NettyClient zuoyideMacBook-Pro.local/192.168.103.40 connect to the server /192.168.103.40:20880, dubbo version: 2.4.10, current host: 192.168.103.40
      
        Refer dubbo service com.of.wangpu.api.service.HelloService from url zookeeper://127.0.0.1:2181/com.alibaba.dubbo.registry.RegistryService?anyhost=true&application=dubbo-provider&check=false&connected=true&dubbo=2.4.10&inside.invoker.count=1&inside.invokers=dubbo%3A%2F%2F192.168.103.40%3A20880%2Fcom.of.wangpu.api.service.HelloService%3Fanyhost%3Dtrue%26application%3Ddubbo-provider%26dubbo%3D2.4.10%26interface%3Dcom.of.wangpu.api.service.HelloService%26methods%3DsayHello%26pid%3D2054%26side%3Dprovider%26timestamp%3D1535511638784&interface=com.of.wangpu.api.service.HelloService&methods=sayHello&owner=dubbo-provider&pid=2077&side=consumer&timestamp=1535511731989, dubbo version: 2.4.10, current host: 192.168.103.40
        ```

## 2、Dubbo相关知识点总结    
![](http://ovsiiuil2.bkt.clouddn.com/architecture.png)

- 服务容器Container负责启动，加载，运行服务提供者。
- 服务提供者Provider在启动时，向注册中心注册自己提供的服务。
- 服务消费者Consumer在启动时，向注册中心订阅自己所需的服务。
- 注册中心Registry返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

- 服务消费者Consumer，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
- 服务消费者Consumer和提供者Provider，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心Monitor。


- 1.Dubbo支持哪些协议，每种协议的应用场景，优缺点？

 - dubbo： 单一长连接和NIO异步通讯，适合大并发小数据量的服务调用，以及消费者远大于提供者。传输协议TCP，异步，Hessian序列化；
 - rmi： 采用JDK标准的rmi协议实现，传输参数和返回参数对象需要实现Serializable接口，使用java标准序列化机制，使用阻塞式短连接，传输数据包大小混合，消费者和提供者个数差不多，可传文件，传输协议TCP。 多个短连接，TCP协议传输，同步传输，适用常规的远程服务调用和rmi互操作。在依赖低版本的Common-Collections包，java序列化存在安全漏洞；
 - webservice： 基于WebService的远程调用协议，集成CXF实现，提供和原生WebService的互操作。多个短连接，基于HTTP传输，同步传输，适用系统集成和跨语言调用；
 - http： 基于Http表单提交的远程调用协议，使用Spring的HttpInvoke实现。多个短连接，传输协议HTTP，传入参数大小混合，提供者个数多于消费者，需要给应用程序和浏览器JS调用；
 - hessian： 集成Hessian服务，基于HTTP通讯，采用Servlet暴露服务，Dubbo内嵌Jetty作为服务器时默认实现，提供与Hession服务互操作。多个短连接，同步HTTP传输，Hessian序列化，传入参数较大，提供者大于消费者，提供者压力较大，可传文件；
 - memcache： 基于memcached实现的RPC协议
 - redis： 基于redis实现的RPC协议
- 2.Dubbo超时时间怎样设置？

   - Dubbo超时时间设置有两种方式：

    - 服务提供者端设置超时时间，在Dubbo的用户文档中，推荐如果能在服务端多配置就尽量多配置，因为服务提供者比消费者更清楚自己提供的服务特性。
    
    - 服务消费者端设置超时时间，如果在消费者端设置了超时时间，以消费者端为主，即优先级更高。因为服务调用方设置超时时间控制性更灵活。如果消费方超时，服务端线程不会定制，会产生警告。
- 3.Dubbo有些哪些注册中心？

   - Multicast注册中心： Multicast注册中心不需要任何中心节点，只要广播地址，就能进行服务注册和发现。基于网络中组播传输实现；
   
   - Zookeeper注册中心： 基于分布式协调系统Zookeeper实现，采用Zookeeper的watch机制实现数据变更；
   - redis注册中心： 基于redis实现，采用key/Map存储，住key存储服务名和类型，Map中key存储服务URL，value服务过期时间。基于redis的发布/订阅模式通知数据变更；
   - Simple注册中心
- 4.Dubbo集群的负载均衡有哪些策略　　

  - Dubbo提供了常见的集群策略实现，并预扩展点予以自行实现。

  -  Random LoadBalance: 随机选取提供者策略，有利于动态调整提供者权重。截面碰撞率高，调用次数越多，分布越均匀；
  
  - RoundRobin LoadBalance: 轮循选取提供者策略，平均分布，但是存在请求累积的问题；
  -  LeastActive LoadBalance: 最少活跃调用策略，解决慢提供者接收更少的请求；
  -  ConstantHash LoadBalance: 一致性Hash策略，使相同参数请求总是发到同一提供者，一台机器宕机，可以基于虚拟节点，分摊至其他提供者，避免引起提供者的剧烈变动；


## 3、SPI设计目标

- 面向对象的设计里，模块之间是基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体实现类，就违法了可拔插的原则。如果需要替换一种实现，就需要修改源码。

  Java SPI 就是提供了这样一个机制：

 - 为了某个接口寻找服务的实现的机制。优点类似IOC的思想，就是装配的控制权移到代码之外。

- SPI的具体约定如下：
  - 当服务的提供者，提供了服务接口的一种实现之后，在jar包的META-INF/services/目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。jdk提供服务实现查找的一个工具类：java.util.ServiceLoader
  
- JDK标准的SPI会一次性实例化扩展点所有实现，如果有扩展实现初始化很耗时，但如果没用上也加载，会很浪费资源.
- Dubbo增加了对扩展点IoC和AOP的支持，一个扩展点可以直接setter注入其它扩展点。

   dubbo spi有哪些约定？
    - spi 文件 存储路径 在 META-INF\dubbo\internal 目录下 并且文件名为接口的全路径名 就是=接口的包名+接口名
    - 每个spi文件里面的格式定义为： 扩展名=具体的类名，例如 dubbo=com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtoco


## 4、dubbo自己的SPI实现
- 1、为该接口 new 一个 ExtensionLoader,然后缓存起来。
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_14-20-50.png)

- 2、getAdaptiveExtension() 获取一个扩展类，如果@Adaptive注解在类上就是一个装饰类；如果注解在方法上就是一个动态代理类，例如Protocol$Adaptive对象。


  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_14-22-21.png)

- 3、getExtension(String name) 获取一个指定对象。

  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_14-29-39.png)


```
public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
    if (type == null)//拓展点类型非空判断
        throw new IllegalArgumentException("Extension type == null");
    if(!type.isInterface()) {//拓展点类型只能是接口
        throw new IllegalArgumentException("Extension type(" + type + ") is not interface!");
    }
    if(!withExtensionAnnotation(type)) {//需要添加spi注解,否则抛异常
        throw new IllegalArgumentException("Extension type(" + type + 
                ") is not extension, because WITHOUT @" + SPI.class.getSimpleName() + " Annotation!");
    }
    //从缓存EXTENSION_LOADERS中获取,如果不存在则新建后加入缓存
    //对于每一个拓展,都会有且只有一个ExtensionLoader与其对应
    ExtensionLoader<T> loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
    if (loader == null) {
        EXTENSION_LOADERS.putIfAbsent(type, new ExtensionLoader<T>(type));
        loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
    }
    return loader;
}

private ExtensionLoader(Class<?> type) {
    this.type = type;
    //这里会存在递归调用,ExtensionFactory的objectFactory为null,其他的均为AdaptiveExtensionFactory
    //AdaptiveExtensionFactory的factories中有SpiExtensionFactory,SpringExtensionFactory
    //getAdaptiveExtension()这个是获取一个拓展装饰类对象.细节在下篇讲解
    objectFactory = (type == ExtensionFactory.class ? null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
}
```

其次我们从Dubbo的启动开始分析

```
ExtensionLoader.getExtensionLoader(Container.class);
    -->this.type = type;
    -->objectFactory = (type == ExtensionFactory.class ? null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
       -->ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension())
           -->this.type = type;
           -->objectFactory = null;
```

```
private final ConcurrentMap<String, Holder<Object>> cachedInstances = new ConcurrentHashMap<String, Holder<Object>>();
```
objectFactory作用，为dubbo的IOC提供所有对象。

继续看还有个很重要的类Holder,这个类用于保存一个值,并且给值添加volatile来保证线程的可见性.

![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_14-57-10.png)


## 5、SPI机制的adpative原理 getAdaptiveExtension

![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_15-24-57.png)

![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_15-26-522.png)


-  adaptive注解在类和方法上的区别
  -  1、注解在类上： 代表人工实现编码，即实现了一个装饰类。例如 ExtensionFactory
  -  2、注解在方法上：代表自动生成和编译一个动态的adpatice类，例如： Protocol$Adaptive

```
ExtensionLoader.getExtensionLoader(ProxyFactory.class).getAdaptiveExtension();//为cachedAdaptiveInstance赋值
   --->createAdaptiveExtension()
       --->getAdaptiveExtensionClass()
           --->getExtensionClasses() //为cachedClasses对象赋值。
             --->loadFile()   //加载配置文件
           --->createAdaptiveExtensionClass。//自动生成和编译一个动态Adaptive类，这是一个代理类。
               --->ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();//获取代码编译器
               --->ompiler.compile(code, classLoader);//加载代理
```

主要做了以下几件事情：

- 1.loadExtensionClasses方法判断ExtensionLoader类中的传入的type接口是否标注了SPI注解，并获取SPI注解的值，这个值为接口的默认实现标记。
- 2.loadFile方法用来加载配置路径下的接口的实现类。比如在调用loadFile方法时，传入的参数DUBBO_INTERNAL_DIRECTORY，DUBBO_DIRECTORY，SERVICES_DIRECTORY。他们都描述了接口实现类配置文件路径，看看3个属性的值如下：

  ```
  private static final String SERVICES_DIRECTORY = "META-INF/services/";
  private static final String DUBBO_DIRECTORY = "META-INF/dubbo/"; 
  private static final String DUBBO_INTERNAL_DIRECTORY = DUBBO_DIRECTORY + "internal/";
  ```
  这里我就以com.alibaba.dubbo.rpc.Protocol接口来研究！哟！是不是感觉跟JDK里面配置不太一样，它按照key=value的形式来保存的，在分析下loadFile方法中的代码，它也是按照key=value的格式来解析出接口的具体实现，将最终解析的数据保存到了传入的map参数extensionClasses中。大家应该感到好奇为什么要做个key=value的配置。打个比方Protocol协议接口在dubbo框架里实现有hession，http,rmi,webservice，dubbo等好几种实现，在程序运行中我们根据配置来使用具体的协议，比方我要使用rmi协议，那我就配置rmi，我想使用dubbo 我就配置dubbo。配置好以后会根据这个属性配置取找相关的具体协议实现。所以这里的key=value应该就是做这个事情的。看看Protocol接口的实现配置
  
  ```
  redis=com.alibaba.dubbo.rpc.protocol.redis.RedisProtocol
  rmi=com.alibaba.dubbo.rpc.protocol.rmi.RmiProtocol
  com.alibaba.dubbo.rpc.protocol.http.HttpProtocol
  hessian=com.alibaba.dubbo.rpc.protocol.hessian.HessianProtocol
  injvm=com.alibaba.dubbo.rpc.protocol.injvm.InjvmProtocol
  memcached=com.alibaba.dubbo.rpc.protocol.memcached.MemcachedProtocol
  thrift=com.alibaba.dubbo.rpc.protocol.thrift.ThriftProtocol
  com.alibaba.dubbo.rpc.protocol.webservice.WebServiceProtocol
  ```
 
 ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_16-01-51.png)


 - 关于loadfile的一些细节
   - 目的： 通过把配置文件META-INF/dubbo/internal/com.alibaba.dubbo.rpc.protocol.webservice.WebServiceProtocol的内容，存储到缓存变量中。

   - cachedAdaptiveClass = clazz; //如果这个class含有Adaptive注解就赋值。例如ExtensionFactory.Protocol在这个环节是没有的。
   - cachedWrapperClasses = new ConcurrentHashSet<Class<?>>();//当这个class不含有Adaptive注解，并且包含目标接口(type)类型。例如protocol里边的spi就只有ProtocolFilterWrapper和ProcolListerWrapper能命中。
   - cacheActivates. //剩下的类，包含Activate注解。
   - cacheNames. // 剩下的类就存储在这里。

         ```
         private Class<?> getAdaptiveExtensionClass() {
                getExtensionClasses();
                if (cachedAdaptiveClass != null) {
                    return cachedAdaptiveClass;
                }
                return cachedAdaptiveClass = createAdaptiveExtensionClass();
         }
         ```
    - 如果cachedAdaptiveClass存在，就直接返回。不存在就生成动态类。

      ```
    private Class<?> createAdaptiveExtensionClass() {
        String code = createAdaptiveExtensionClassCode();//创建接口的代理类实现
        ClassLoader classLoader = findClassLoader();//获取当前使用的类加载器
        com.alibaba.dubbo.common.compiler.Compiler compiler = ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();//获取代码编译器
        return compiler.compile(code, classLoader);//加载代理
    }
      ```

此时回忆一下Protocol结构

```
 /**
     * 暴露远程服务：<br>
     * 1. 协议在接收请求时，应记录请求来源方地址信息：RpcContext.getContext().setRemoteAddress();<br>
     * 2. export()必须是幂等的，也就是暴露同一个URL的Invoker两次，和暴露一次没有区别。<br>
     * 3. export()传入的Invoker由框架实现并传入，协议不需要关心。<br>
     *
     * @param <T>     服务的类型
     * @param invoker 服务的执行体
     * @return exporter 暴露服务的引用，用于取消暴露
     * @throws RpcException 当暴露服务出错时抛出，比如端口已占用
     */
    @Adaptive
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;

    /**
     * 引用远程服务：<br>
     * 1. 当用户调用refer()所返回的Invoker对象的invoke()方法时，协议需相应执行同URL远端export()传入的Invoker对象的invoke()方法。<br>
     * 2. refer()返回的Invoker由协议实现，协议通常需要在此Invoker中发送远程请求。<br>
     * 3. 当url中有设置check=false时，连接失败不能抛出异常，并内部自动恢复。<br>
     *
     * @param <T>  服务的类型
     * @param type 服务的类型
     * @param url  远程服务的URL地址
     * @return invoker 服务的本地代理
     * @throws RpcException 当连接服务提供方失败时抛出
     */
    @Adaptive
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;

    /**
     * 释放协议：<br>
     * 1. 取消该协议所有已经暴露和引用的服务。<br>
     * 2. 释放协议所占用的所有资源，比如连接和端口。<br>
     * 3. 协议在释放后，依然能暴露和引用新的服务。<br>
     */
```
动态生成的协议代理类

```
	public class Protocol$Adpative implements com.alibaba.dubbo.rpc.Protocol {
		public void destroy() {throw new UnsupportedOperationException("method public abstract void com.alibaba.dubbo.rpc.Protocol.destroy() of interface com.alibaba.dubbo.rpc.Protocol is not adaptive method!");
		}
		public int getDefaultPort() {throw new UnsupportedOperationException("method public abstract int com.alibaba.dubbo.rpc.Protocol.getDefaultPort() of interface com.alibaba.dubbo.rpc.Protocol is not adaptive method!");
		}
		public com.alibaba.dubbo.rpc.Exporter export(com.alibaba.dubbo.rpc.Invoker arg0) throws com.alibaba.dubbo.rpc.RpcException {
			if (arg0 == null) throw new IllegalArgumentException("com.alibaba.dubbo.rpc.Invoker argument == null");
 
			if (arg0.getUrl() == null) throw new IllegalArgumentException("com.alibaba.dubbo.rpc.Invoker argument getUrl() == null");
 
			com.alibaba.dubbo.common.URL url = arg0.getUrl();
 
			//默认选择dubbo协议，否则根据url中带的协议属性来选择对应的协议处理对象，这样可以动态选择不同的协议
			String extName = ( url.getProtocol() == null ? "dubbo" : url.getProtocol() );
 
			if(extName == null) throw new IllegalStateException("Fail to get extension(com.alibaba.dubbo.rpc.Protocol) name from url(" + url.toString() + ") use keys([protocol])");
 
			//根据拿到的协议key从缓存的map中取协议对象
			com.alibaba.dubbo.rpc.Protocol extension = (com.alibaba.dubbo.rpc.Protocol)ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.rpc.Protocol.class).getExtension(extName);
 
			return extension.export(arg0);
		}
		public com.alibaba.dubbo.rpc.Invoker refer(java.lang.Class arg0, com.alibaba.dubbo.common.URL arg1) throws com.alibaba.dubbo.rpc.RpcException {
 
			if (arg1 == null) throw new IllegalArgumentException("url == null");
 
			com.alibaba.dubbo.common.URL url = arg1;
 
			String extName = ( url.getProtocol() == null ? "dubbo" : url.getProtocol() );
 
			if(extName == null) throw new IllegalStateException("Fail to get extension(com.alibaba.dubbo.rpc.Protocol) name from url(" + url.toString() + ") use keys([protocol])");
 
			//根据拿到的协议key从缓存的map中取协议对象
			com.alibaba.dubbo.rpc.Protocol extension = (com.alibaba.dubbo.rpc.Protocol)ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.rpc.Protocol.class).getExtension(extName);
 
			return extension.refer(arg0, arg1);
		}
	}
```
到这里接口的代理已经生成啦！再回退到createAdaptiveExtension方法中。

```
injectExtension((T) getAdaptiveExtensionClass().newInstance());
```
作用：

  - 进入IOC反转控制机制判断接口代理类中是否有需要注入的属性。

## 6、dubbo自己的IOC和AOP原理
getExtension(String name)

```
getExtension(String name) //指定对象缓存在cachedInstances；get出来的对象wrapper对象，例如protocol就是ProtocolFilterWrapper和ProtocolListenerWrapper其中一个。
  -->createExtension(String name)
    -->getExtensionClasses()
    -->injectExtension(T instance)//dubbo的IOC反转控制，就是从spi和spring里面提取对象赋值。
      -->objectFactory.getExtension(pt, property)
        -->SpiExtensionFactory.getExtension(type, name)
          -->ExtensionLoader.getExtensionLoader(type)
          -->loader.getAdaptiveExtension()
        -->SpringExtensionFactory.getExtension(type, name)
          -->context.getBean(name)
    -->injectExtension((T) wrapperClass.getConstructor(type).newInstance(instance))//AOP的简单设计
```

## 7、dubbo的动态编译

上文我们讲到编译类，那么Dubbo是怎么实现的呢


```
com.alibaba.dubbo.common.compiler.Compiler compiler = ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();//获取代码编译器
        return compiler.compile(code, classLoader);//加载代理
```

![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_21-14-31.png)

第一步
![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_21-16-39.png)
第二步
![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_21-21-55.png)
第三步
![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_21-22-05.png)
第四步
![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_21-21-31.png)
第五步
![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-08-29_21-30-12.png)

在看JavassistCompiler 编译类之前，首先需要移动的javac基础知识。下边我们看一个例子

```
public class CompilerByJavassist {

	public static void main(String[] args) throws Exception {

		// ClassPool：CtClass对象的容器
		ClassPool pool = ClassPool.getDefault();

		// 通过ClassPool生成一个public新类Emp.java
		CtClass ctClass = pool.makeClass("com.study.javassist.Emp");

		// 添加属性
		// 首先添加属性private String ename
		CtField enameField = new CtField(pool.getCtClass("java.lang.String"),
				"ename", ctClass);
		enameField.setModifiers(Modifier.PRIVATE);
		ctClass.addField(enameField);

		// 其次添加熟悉privtae int eno
		CtField enoField = new CtField(pool.getCtClass("int"), "eno", ctClass);
		enoField.setModifiers(Modifier.PRIVATE);
		ctClass.addField(enoField);

		// 为属性ename和eno添加getXXX和setXXX方法
		ctClass.addMethod(CtNewMethod.getter("getEname", enameField));
		ctClass.addMethod(CtNewMethod.setter("setEname", enameField));
		ctClass.addMethod(CtNewMethod.getter("getEno", enoField));
		ctClass.addMethod(CtNewMethod.setter("setEno", enoField));

		// 添加构造函数
		CtConstructor ctConstructor = new CtConstructor(new CtClass[] {},
				ctClass);
		// 为构造函数设置函数体
		StringBuffer buffer = new StringBuffer();
		buffer.append("{\n").append("ename=\"yy\";\n").append("eno=001;\n}");
		ctConstructor.setBody(buffer.toString());
		// 把构造函数添加到新的类中
		ctClass.addConstructor(ctConstructor);

		// 添加自定义方法
		CtMethod ctMethod = new CtMethod(CtClass.voidType, "printInfo",
				new CtClass[] {}, ctClass);
		// 为自定义方法设置修饰符
		ctMethod.setModifiers(Modifier.PUBLIC);
		// 为自定义方法设置函数体
		StringBuffer buffer2 = new StringBuffer();
		buffer2.append("{\nSystem.out.println(\"begin!\");\n")
				.append("System.out.println(ename);\n")
				.append("System.out.println(eno);\n")
				.append("System.out.println(\"over!\");\n").append("}");
		ctMethod.setBody(buffer2.toString());
		ctClass.addMethod(ctMethod);

		//最好生成一个class
		Class<?> clazz = ctClass.toClass();
		Object obj = clazz.newInstance();
		//反射 执行方法
		obj.getClass().getMethod("printInfo", new Class[] {})
				.invoke(obj, new Object[] {});

		// 把生成的class文件写入文件
		byte[] byteArr = ctClass.toBytecode();
		FileOutputStream fos = new FileOutputStream(new File("D://Emp.class"));
		fos.write(byteArr);
		fos.close();
	}
}
```
此时就不难理解上边第五步的代码了。


