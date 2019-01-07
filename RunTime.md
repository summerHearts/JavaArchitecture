##1、runtime（运行时机制）是什么

- runtime是属于OC的底层，是一套比较底层的纯C语言API, 属于1个C语言库, 包含了很多底层的C语言API，可以进行一些非常底层的操作(用OC是无法现实的, 不好实现)。 在我们平时编写的OC代码中, 程序运行过程时, 其实最终都是转成了runtime的C语言代码, runtime算是OC的幕后工作者。

- 例如：

    ```
    // OC : 
    [[MJPerson alloc] init] 
    // runtime : 
    objc_msgSend(objc_msgSend("MJPerson" , "alloc"), "init")
    ```

##2、runtime知识图谱

![](https://upload-images.jianshu.io/upload_images/325120-5c2405e9805ec022.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


##3、runtime数据结构

![](https://upload-images.jianshu.io/upload_images/325120-d4723349b859c269.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


- 3.1、objc_object

![](https://upload-images.jianshu.io/upload_images/325120-50e71da7bca91e8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

 ```
 struct objc_object {
    private:
        isa_t isa;
    
    public:
    
        // ISA() assumes this is NOT a tagged pointer object
        Class ISA();
    
        // getIsa() allows this to be a tagged pointer object
        Class getIsa();
    
        // initIsa() should be used to init the isa of new objects only.
        // If this object already has an isa, use changeIsa() for correctness.
        // initInstanceIsa(): objects with no custom RR/AWZ
        // initClassIsa(): class objects
        // initProtocolIsa(): protocol objects
        // initIsa(): other objects
        void initIsa(Class cls /*indexed=false*/);
        void initClassIsa(Class cls /*indexed=maybe*/);
        void initProtocolIsa(Class cls /*indexed=maybe*/);
        void initInstanceIsa(Class cls, bool hasCxxDtor);
    
        // changeIsa() should be used to change the isa of existing objects.
        // If this is a new object, use initIsa() for performance.
        Class changeIsa(Class newCls);
    
        bool hasIndexedIsa();
        bool isTaggedPointer();
        bool isClass();
    
        // object may have associated objects?
        bool hasAssociatedObjects();
        void setHasAssociatedObjects();
    
        // object may be weakly referenced?
        bool isWeaklyReferenced();
        void setWeaklyReferenced_nolock();
    
        // object may have -.cxx_destruct implementation?
        bool hasCxxDtor();
    
        // Optimized calls to retain/release methods
        id retain();
        void release();
        id autorelease();
    
        // Implementations of retain/release methods
        id rootRetain();
        bool rootRelease();
        id rootAutorelease();
        bool rootTryRetain();
        bool rootReleaseShouldDealloc();
        uintptr_t rootRetainCount();
    
        // Implementation of dealloc methods
        bool rootIsDeallocating();
        void clearDeallocating();
        void rootDealloc();
    
    private:
        void initIsa(Class newCls, bool indexed, bool hasCxxDtor);
    
        // Slow paths for inline control
        id rootAutorelease2();
        bool overrelease_error();
    
    #if SUPPORT_NONPOINTER_ISA
        // Unified retain count manipulation for nonpointer isa
        id rootRetain(bool tryRetain, bool handleOverflow);
        bool rootRelease(bool performDealloc, bool handleUnderflow);
        id rootRetain_overflow(bool tryRetain);
        bool rootRelease_underflow(bool performDealloc);
    
        void clearDeallocating_slow();
    
        // Side table retain count overflow for nonpointer isa
        void sidetable_lock();
        void sidetable_unlock();
    
        void sidetable_moveExtraRC_nolock(size_t extra_rc, bool isDeallocating, bool weaklyReferenced);
        bool sidetable_addExtraRC_nolock(size_t delta_rc);
        size_t sidetable_subExtraRC_nolock(size_t delta_rc);
        size_t sidetable_getExtraRC_nolock();
    #endif
    
        // Side-table-only retain count
        bool sidetable_isDeallocating();
        void sidetable_clearDeallocating();
    
        bool sidetable_isWeaklyReferenced();
        void sidetable_setWeaklyReferenced_nolock();
    
        id sidetable_retain();
        id sidetable_retain_slow(SideTable& table);
    
        uintptr_t sidetable_release(bool performDealloc = true);
        uintptr_t sidetable_release_slow(SideTable& table, bool performDealloc = true);
    
        bool sidetable_tryRetain();
    
        uintptr_t sidetable_retainCount();
    #if DEBUG
        bool sidetable_present();
    #endif
};
 ```
 
- 3.2、objc_class

![](https://upload-images.jianshu.io/upload_images/325120-94e6c7d2a487caa8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

```
struct objc_class : objc_object {
    Class superclass;
    const char *name;
    uint32_t version;
    uint32_t info;
    uint32_t instance_size;
    struct old_ivar_list *ivars;
    struct old_method_list **methodLists;
    Cache cache;
    struct old_protocol_list *protocols;
    // CLS_EXT only
    const uint8_t *ivar_layout;
    struct old_class_ext *ext;

    void setInfo(uint32_t set) {
        OSAtomicOr32Barrier(set, (volatile uint32_t *)&info);
    }

    void clearInfo(uint32_t clear) {
        OSAtomicXor32Barrier(clear, (volatile uint32_t *)&info);
    }


    // set and clear must not overlap
    void changeInfo(uint32_t set, uint32_t clear) {
        assert((set & clear) == 0);

        uint32_t oldf, newf;
        do {
            oldf = this->info;
            newf = (oldf | set) & ~clear;
        } while (!OSAtomicCompareAndSwap32Barrier(oldf, newf, (volatile int32_t *)&info));
    }

    bool hasCxxCtor() {
        // set_superclass propagates the flag from the superclass.
        return info & CLS_HAS_CXX_STRUCTORS;
    }

    bool hasCxxDtor() {
        return hasCxxCtor();  // one bit for both ctor and dtor
    }

    bool hasCustomRR() { 
        return true;
    }
    void setHasCustomRR(bool = false) { }
    void setHasDefaultRR() { }
    void printCustomRR(bool) { }

    bool hasCustomAWZ() { 
        return true;
    }
    void setHasCustomAWZ(bool = false) { }
    void setHasDefaultAWZ() { }
    void printCustomAWZ(bool) { }

    bool instancesHaveAssociatedObjects() {
        return info & CLS_INSTANCES_HAVE_ASSOCIATED_OBJECTS;
    }

    void setInstancesHaveAssociatedObjects() {
        setInfo(CLS_INSTANCES_HAVE_ASSOCIATED_OBJECTS);
    }

    bool shouldGrowCache() {
        return info & CLS_GROW_CACHE;
    }

    void setShouldGrowCache(bool grow) {
        if (grow) setInfo(CLS_GROW_CACHE);
        else clearInfo(CLS_GROW_CACHE);
    }

    bool shouldFinalizeOnMainThread() {
        return info & CLS_FINALIZE_ON_MAIN_THREAD;
    }

    void setShouldFinalizeOnMainThread() {
        setInfo(CLS_FINALIZE_ON_MAIN_THREAD);
    }

    // +initialize bits are stored on the metaclass only
    bool isInitializing() {
        return getMeta()->info & CLS_INITIALIZING;
    }

    // +initialize bits are stored on the metaclass only
    void setInitializing() {
        getMeta()->setInfo(CLS_INITIALIZING);
    }

    // +initialize bits are stored on the metaclass only
    bool isInitialized() {
        return getMeta()->info & CLS_INITIALIZED;
    }

    // +initialize bits are stored on the metaclass only
    void setInitialized() {
        getMeta()->changeInfo(CLS_INITIALIZED, CLS_INITIALIZING);
    }

    bool isLoadable() {
        // A class registered for +load is ready for +load to be called
        // if it is connected.
        return isConnected();
    }

    IMP getLoadMethod();

    bool isFuture();

    bool isConnected();

    const char *mangledName() { return name; }
    const char *demangledName() { return name; }
    const char *nameForLogging() { return name; }

    bool isMetaClass() {
        return info & CLS_META;
    }

    // NOT identical to this->ISA() when this is a metaclass
    Class getMeta() {
        if (isMetaClass()) return (Class)this;
        else return this->ISA();
    }

    // May be unaligned depending on class's ivars.
    uint32_t unalignedInstanceSize() {
        return instance_size;
    }

    // Class's ivar size rounded up to a pointer-size boundary.
    uint32_t alignedInstanceSize() {
        return (unalignedInstanceSize() + WORD_MASK) & ~WORD_MASK;
    }

    size_t instanceSize(size_t extraBytes) {
        size_t size = alignedInstanceSize() + extraBytes;
        // CF requires all objects be at least 16 bytes.
        if (size < 16) size = 16;
        return size;
    }

};
```
- 3.3、isa

![](https://upload-images.jianshu.io/upload_images/325120-a5db1f7484258d17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](https://upload-images.jianshu.io/upload_images/325120-f3ad3527bd30b4e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](https://upload-images.jianshu.io/upload_images/325120-8147a6de2b920561.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


  - cache_t是用来存放每一个类中的方法缓存的数据结构，其本质是一个可增量扩展的哈希表数据结构，使用cache_t可以快速查找方法的执行函数，也是局部性原理的最佳应用。试想一下，如果一个类中有众多的类方法与对象方法，有些方法可能在程序执行的生命周期中只会调用一次，而在其他方法每一次调用时，系统都会遍历类中所有的方法后找到其方法的相应IMP进行执行，是一个非常浪费性能的工作，所以以此原因引入了cache_t，缓存方法列表，系统将一些常用方法储存在类的缓存列表cache_t中，在每次方法调用时，先从缓存列表中查找，如果找到了对应方法，就直接调用其函数体，这是一种非常节约计算成本的方式。



![](https://upload-images.jianshu.io/upload_images/325120-424628a0fa6a0fd1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


  - 关于cache_t中的的数据结构，使用的一张哈希表存储的众多bucket_t数据结构，在bucket_t中包含的是方法的实现IMP与其对应的key，在一次消息传递的过程中，系统首先会在对应的类中进行缓存查找，利用发送消息的SEL查找cache_t哈希表，其过程是通过计算将SEL转换成一个cache_key_t对象，利用该"key"进行cache_t哈希表定位，随后取出bucket_t，直接拿到IMP指针，进行消息的发送。
  
![](https://upload-images.jianshu.io/upload_images/325120-b1e443883a36011d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

  - class_data_bits_t数据结构。class_data_bits_t主要的作用是包含类中的方法、成员变量、协议等诸多主要信息，随后再对一些零散信息的封装。其实class_data_bits_t的主要作用也是对class_rw_t的一个封装，重要的信息其实都在class_rw_t中

![](https://upload-images.jianshu.io/upload_images/325120-e949e2e903eebd97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

  - class_rw_t数据结构。class_rw_t代表了类相关的读写信息，以及对class_ro_t的封装。顾名思义，rw就是读写，ro就是只读。class_rw_t内的数据结构有,class_ro_t，protocols,properties,methods，第一个class_ro_t稍后详细说明。后面三个其实就是对协议、属性、方法的封装，这三个数据结构都是以二维数组的形式提现的，但是为什么是二维数组呢？我之前写过一篇关于分类理解的文章，里面说到分类方法添加的问题，就是分类在运行时决议的过程中，会把分类的方法都以数组的形式都添加到类方法中，其实过程就在这里，分类中众多method_t以数组[method1,method2,method3]的形式提现，但一个类可能有众多分类，那么分类1，分类2，分类3在objc_class的class_data_bits_t的class_rw_t中的体现就是[[method1,method2,method3],[method1,method2,method3],[method1,method2,method3]]，其都是method_t的数据形式。协议，属性的体现方式都是相似的，就同理后推就好了。所以到这里肯定会有一个疑问，为什么class_rw_t中没有成员变量只有属性。因为class_rw_t中包含的是一个读写数据的列表，换言之其实就是分类的列表，class_rw_t只管分类中的数据，之前分类那篇文章说到过，为分类添加属性是不会生成相应的成员变量的，如果要生成对应的成员变量必须用关联对象技术把使其达到一个类似于可读写效果，而其关联对象都储存在一个全局容器中，这也呼应了开头介绍objc_object数据结构中存储的相关关联对象的方法。好了，越绕越远了，现在继续介绍class_ro_t数据结构


![](https://upload-images.jianshu.io/upload_images/325120-dc58792793348ed0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

  - class_ro_t数据结构。这个数据结构包含的信息就是类本身的信息，包括name(类名),ivars(instence variables成员变量列表),properties(属性列表),protocols(协议列表),methodList(方法列表)。所以这下好理解了吧，class_ro_t中装的是类本身编译的信息，class_rw_t中装的是类中分类的信息，而class_data_bits_t封装了class_rw_t封装了class_ro_t。不过需要注意区别的是class_ro_t中包含的ivas,properties,protocols,methodList都是一位数组的形式存在（因为没有分类了嘛）。所以这里还需要注意一个点就是：我们没法向一个编译后的类动态添加信息，比如方法，成员变量，属性等，第一是因为这些都存在于class_ro_t中，其名称含义就告诉你readOnly，人好好的存在那，说了不让你改，你硬要改，那肯定不行啊，第二是因为所谓向一个类中动态添加信息，都是指的用runtime动态添加的类，通俗点讲就是用代码写的类

- 3.4、method_t

![](https://upload-images.jianshu.io/upload_images/325120-eab2d63d6bbf3854.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

```
struct method_t {
    SEL name;
    const char *types;
    IMP imp;

    struct SortBySELAddress :
        public std::binary_function<const method_t&,
                                    const method_t&, bool>
    {
        bool operator() (const method_t& lhs,
                         const method_t& rhs)
        { return lhs.name < rhs.name; }
    };
};
```

![](https://upload-images.jianshu.io/upload_images/325120-c38d8b3a273c59d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)



##4、类对象与元类对象&消息传递相关面试问题

- **类对象**存储实例方法列表等信息。

- **元类对象**存储类方法列表等信息。

![](https://upload-images.jianshu.io/upload_images/855021-567c23810f4827ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

为什么会有元类呢？

 - 因为在Objective-C中，对象的方法并没有存储在对象的结构体中(如果每个对象都存储自己的方法，那我们程序中无数对象就都要存储自己的方法，那内存肯定就不够用了)。当我们调用实例方法时，它通过自己的isa查找到对应的类，然后在class_data_bits_t结构体中查找对应的方法实现。每一个objc_class也有一个superClass指向自己的父类，可以查找到继承的方法。那么如果调用实例方法怎么查找呢？这时，就需要引入元类来保证无论调用类方法和实例方法，都可以以相同的方式来查找。

消息传递的机制

 - 消息传递机制在runtime中的提现是一个方法objc_msgSend(object,SEL,types),当我们在OC中用[]调起一个方法的同时，runtime内部就会去调取这个方法进行消息方法，还有一个方法是objc_msgSendSuper，参数还是一样，这个方法是我们用关键字super去调起方法时runtime内部所执行的方法，其内部包含一个变量receiver，这个变量指向的就是super的子类self自己。之前看过一个案例，是在init内部调用[self class]和[super class]，然后同时打印这两个类名。其实结果不出意外是一样的，但分析原因的话就是因为[super class]调起时，调用的是objc_msgSendSuper这个方法，其中包含一个指针receiver，指向的就是super对象调用者self自己，也就是消息的接收者还是self，而class方法都是存在于较高父类（系统类中的），所以在方法遍历的时候，[super class]和[self class]的区别就在于，super是从父类开始向上遍历直至找到class方法，而self是从本类开始向上遍历最后找到对应方法，而调用class最后的结果取决于消息的接收者，因为两个方法的消息接收者都是self，所以不出意外，最后打印出的结果都是self类本身。


而实际消息当一个方法调用的消息发出后，消息传递的机制为：

 - 首先判断方法为类方法还是实例方法 ->object找到父类class(若为类方法则再向上取到metaClass)->哈希查找objc_class的缓存列表cache_t试试能否用对用SEL直接找到函数体IMP->若找到，则直接进行函数体调用后结束消息放松，若找不到则遍历对应的类或者元类的方法列表，如果方法列表是排序好的则使用二分法遍历，如果没有排序好则使用普通遍历->如果找到了则进行函数体调用，结束消息传递流程，若找不到则继续根据superClass指针找到父类，再重复之前的工作(遍历缓存后遍历本类中方法列表)，在途中若找到了相应函数体实现IMP则进行调用，结束消息传递流程，如果至到NSObject中还没有找到则进行消息转发的流程


![](https://upload-images.jianshu.io/upload_images/325120-86ead71844c8aaaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


![](https://upload-images.jianshu.io/upload_images/325120-2d0b2e61eedefa90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](https://upload-images.jianshu.io/upload_images/325120-a9e0e4d9e03904a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


- **缓存查找**

  - 1、给定值是`SEL`，目标值是对应`bucket_t`中的`IMP`。

  ![](https://www.icheesedu.com/images/1.png)
  
  - 2、当前类中查找

     - 1、对于已排序好的列表，采用二分查找算法查找方法对应执行函数。
     
     - 2、对于没有排序的列表，采用一般遍历查找方法对应执行函数。
     
  - 3、父类逐级查找
  ![](https://www.icheesedu.com/images/2.png)


##5、消息转发流程

关于消息转发的流程，其实是系统给的一个消息再利用的过程，当消息传递流程中没有方法来响应此消息时，开发者可通过重写以下这四个方法来实现消息转发的过程，以及去让计算机做相应的工作。

- 1、+ (BOOL)resolveInstanceMethod:(SEL)sel

若返回YES，则表明消息已处理，结束流程，若返回NO则进行下一个方法的调用

- 2、- (id)forwardingTargetForSelector:(SEL)aSelector

此方法可以返回转发消息的目标，若返回为nil则调用下一个方法

- 3、- (NSMethodSignature *)methodSignatureForSelector

此方法可以返回方法的签名，若返回nil则直接抛出异常，表明方法函数体指针无法被找到，若返回方法签名，则调用下一个方法

- 4、- (void)forwardInvocation:(NSInvocation *)anInvocation

此方法内部会决定是否已处理方法，如果到这个方法也没法处理，则抛出异常，报错

以上四个方法是系统留给开发者处理消息转发的入口，开发者可以通过重写以上四个方法来手动处理消息转发的流程。

 ![](https://www.icheesedu.com/images/message.png)

 ![](https://www.icheesedu.com/images/message_forward_example.png)
   
##6、Method-Swizzling

![](https://www.icheesedu.com/images/Method-Swizzling.png)

![](https://www.icheesedu.com/images/Method-Swizzling_example.png)


##7、动态添加方法

![](https://www.icheesedu.com/images/动态添加test方法的实现.png)

##8、动态方法解析

 - @dynamic

   - 动态运行时语言将函数决议推迟到运行时。
   


