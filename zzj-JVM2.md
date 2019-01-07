# jvm运行机制

[参考](https://www.cnblogs.com/smyhvae/p/4748392.html)   

### JVM启动流程

[参考](https://www.cnblogs.com/muffe/p/3540001.html)   

![微信截图_20181101200639](E:\OneDriver\OneDrive\Pictures\java\微信截图_20181101200639.png)

[![20181101200639.png](https://i.postimg.cc/cJQ5C4Wy/20181101200639.png)](https://postimg.cc/zLGkPN30)



#### 启动过程：

JVM工作原理和特点主要是指操作系统装入JVM，是通过jdk中Java.exe来完成,通过下面4步来完成JVM环境.

1. 创建JVM装载环境和配置

2. 装载JVM.dll

3. 初始化JVM.dll并挂界到JNIENV(JNI调用接口)实例

4. 调用JNIEnv实例装载并处理class类。





### JVM基本结构

[参考](http://www.cnblogs.com/smyhvae/p/4748392.htm#undefined)   

![JVM内部结构](E:\OneDriver\OneDrive\Pictures\java\JVM内部结构.png)

[![JVM.png](https://i.postimg.cc/52pSRYft/JVM.png)](https://postimg.cc/DSJGWzJV)

JVM体系主要是两个JVM的内部体系结构分为**三个子系统**和**两大组件**，分别是：

**子系统**：类装载器（ClassLoader）子系统、执行引擎子系统GC子系统，

**组件 **: 是内存运行数据区域和本地接口,



#### PC寄存器

- 每个线程拥有一个PC寄存器
- 在线程创建时创建
- 指向下一条指令的地址
- 执行本地方法时，PC的值为undefined

#### 方法区

- 保存装载的类信息，类的元信息
  - 类型的常量池
  - 字段，方法信息
  - 方法字节码
- 通常和永久区（Perm）关联在一起

#### java堆

- 和程序开发密切相关
- 应用系统对象都保存在java堆中
- 所有线程共享java堆
- 对分代GC来说，堆也是分代的（eden,  survivor,  tenured）
- GC的主要工作区间

#### java栈

- 线程私有
- 栈由一序列帧组成（因此java栈也叫栈帧）
- 帧保存一个方法的局部变量、操作数栈、常量池指针
- 每一次方法调用创建一个帧，并入栈

##### 局部变量表、操作数栈、[栈上分配](https://my.oschina.net/u/3572607/blog/1840581)   

**栈上分配的好处**：

- 小对象，在没有逃逸的情况下，可以直接分配在栈上
- 直接分配在站上，可以自动回收，减轻GC压力
- 大对象或者逃逸对象无法栈上分配



#### 栈、堆、方法区交互

![栈堆方法区交互](E:\OneDriver\OneDrive\Pictures\java\栈堆方法区交互.png)

[![image.png](https://i.postimg.cc/8kKH4rPP/image.png)](https://postimg.cc/XX5d7qCT)







### JVM内存模型

- 每个线程有一个工作内存和主存独立
- 工作内存存放驻村中变量的值的拷贝

#### 可见性

- volatile
- synchronized
- final

#### 有序性

#### 指令重排序



### 编译和解释运行的概念

#### 解释执行

- 解释执行以解释方式运行字节码
- 解释执行的意思是：读一句执行一句

#### 编译执行（JIT）（快）

- 将字节码编译成机器码
- 直接执行机器码
- 运行时编译
- 编译后性能有数量级的提升



