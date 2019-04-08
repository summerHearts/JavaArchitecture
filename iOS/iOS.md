iOS
## 1、分类Category和扩展Extension有什么区别？可以分别用来做什么？分类有哪些局限性？分类的结构体里面有哪些成员？

- 1、属性、方法增加：

  - 1、分类**Category**中原则上只能增加方法（能添加属性的的原因只是通过runtime解决无setter/getter的问题而已）；
  
  - 2、类扩展**Extension**不仅可以增加方法，还可以增加实例变量（或者属性），只是该实例变量默认是@private类型的（用范围只能在自身类，而不是子类或其他地方）； 
- 2、动态性：
  - 3、类扩展**Extension**中声明的方法没被实现，编译器会报警，但是分类**Category**中的方法没被实现编译器是不会有任何警告的。这是因为类扩展**Extension**是在编译阶段被添加到类中，而分类**Category**是在运行时添加到类中。 
- 3、方法实现的局限性：
  - 4、类扩展**Extension**不能像类别那样拥有独立的实现部分（@implementation部分），也就是说，类扩展**Extension**所声明的方法必须依托对应类的实现部分来实现。 
  - 5、定义在 .m 文件中的类扩展**Extension**方法为私有的，定义在 .h 文件（头文件）中的类扩展方法为公有的。类扩展**Extension**是在 .m 文件中声明私有方法的非常好的方式。 
  - 最重要的还是类扩展**Extension**是在编译阶段被添加到类中，而分类**Category**是在运行时添加到类中。分类方法未实现，编译器也不会报警告。分类方法与原类中相同会因为顺序问题优先调用分类。

  分类：
    
  - 1、声明私有方法

  - 2、分解体积庞大的类文件
  - 3、把Framework的私有方法公开

  特点：
  
  - 1、运行时决议
  
  - 2、可以为系统类添加分类

  分类都可以添加哪些内容？
  
  - 1、实例方法
  
  - 2、类方法
  - 3、协议
  - 4、属性

  


## 2、讲一下atomic的实现机制；为什么不能保证绝对的线程安全（最好可以结合场景来说）？

```
id objc_getProperty(id self, SEL _cmd, ptrdiff_t offset, BOOL atomic) {
    if (offset == 0) {
        return object_getClass(self);
    }

    // Retain release world
    id *slot = (id*) ((char*)self + offset);
    if (!atomic) return *slot;
        
    // Atomic retain release world
    spinlock_t& slotlock = PropertyLocks[slot];
    slotlock.lock();
    id value = objc_retain(*slot);
    slotlock.unlock();
    
    // for performance, we (safely) issue the autorelease OUTSIDE of the spinlock.
    return objc_autoreleaseReturnValue(value);
}


static inline void reallySetProperty(id self, SEL _cmd, id newValue, ptrdiff_t offset, bool atomic, bool copy, bool mutableCopy) __attribute__((always_inline));

static inline void reallySetProperty(id self, SEL _cmd, id newValue, ptrdiff_t offset, bool atomic, bool copy, bool mutableCopy)
{
    if (offset == 0) {
        object_setClass(self, newValue);
        return;
    }

    id oldValue;
    id *slot = (id*) ((char*)self + offset);

    if (copy) {
        newValue = [newValue copyWithZone:nil];
    } else if (mutableCopy) {
        newValue = [newValue mutableCopyWithZone:nil];
    } else {
        if (*slot == newValue) return;
        newValue = objc_retain(newValue);
    }

    if (!atomic) {
        oldValue = *slot;
        *slot = newValue;
    } else {
        spinlock_t& slotlock = PropertyLocks[slot];
        slotlock.lock();
        oldValue = *slot;
        *slot = newValue;        
        slotlock.unlock();
    }

    objc_release(oldValue);
}
```

 - 当多个线程访问某个类时，不管运行环境采用何种调度方式或者这些进程将如何交替进行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么就称为线程安全的。 
	- 原子性： 提供互斥访问，同一时刻只能有一个线程来对它进行操作。 
	- 可见性： 一个线程对主内存的修改可以及时的被其他线程观察到。 
	- 有序性： 一个线程观察其他线程中的指令执行顺序，由于指令重排序的存在，该观察结果一般杂乱无章。 

对于atomic的属性，系统生成的 getter/setter 会保证 get、set 操作的完整性，不受其他线程影响。比如，线程 A 的 getter 方法运行到一半，线程 B 调用了 setter：那么线程 A 的 getter 还是能得到一个完好无损的对象。

而nonatomic就没有这个保证了。所以，nonatomic的速度要比atomic快。

不过atomic可并不能保证线程安全。如果线程 A 调了 getter，与此同时线程 B 、线程 C 都调了 setter——那最后线程 A get 到的值，3种都有可能：可能是 B、C set 之前原始的值，也可能是 B set 的值，也可能是 C set 的值。同时，最终这个属性的值，可能是 B set 的值，也有可能是 C set 的值。


## 3、被weak修饰的对象在被释放的时候会发生什么？是如何实现的？知道sideTable么？里面的结构可以画出来么？
- 被weak修饰的对象在被释放时候会置为nil，不同于assign； 
- Runtime维护了一个weak表，用于存储指向某个对象的所有weak指针。weak表其实是一个hash（哈希）表，Key是所指对象的地址，Value是weak指针的地址（这个地址的值是所指对象指针的地址）数组。 
	- 1、初始化时：runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。 
	- 2、添加引用时：objc_initWeak函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。 
	- 3、释放时，调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。
	
```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        NSObject *p = [[NSObject alloc] init];
        __weak NSObject *p1 = p;
    }
    return 0;
}

```
单步运行，发现会跳入 NSObject.mm 中的 objc_initWeak() 这个方法。在进行编译过程前，clang 其实对 __weak 做了转换，将声明方式做出了如下调整。
NSObject objc_initWeak(&p, 对象指针);
其中的对象指针，就是代码中的 [[NSObject alloc] init] ，而 p 是我们传入的一个弱引用指针。而对于 objc_initWeak() 方法的实现，在 runtime 中的源码如下：

```
id objc_initWeak(id *location, id newObj) {
    // 查看对象实例是否有效
    // 无效对象直接导致指针释放
    if (!newObj) {
        *location = nil;
        return nil;
    }
    // 这里传递了三个 bool 数值
    // 使用 template 进行常量参数传递是为了优化性能
    return storeWeakfalse/*old*/, true/*new*/, true/*crash*/>
        (location, (objc_object*)newObj);
}

```


可以看出，这个函数仅仅是一个深层函数的调用入口，而一般的入口函数中，都会做一些简单的判断（例如 objc_msgSend 中的缓存判断），这里判断了其指针指向的类对象是否有效，无效直接释放，不再往深层调用函数。
需要注意的是，当修改弱引用的变量时，这个方法非线程安全。所以切记选择竞争带来的一些问题。
继续阅读 objc_storeWeak() 的实现：

```
// HaveOld:  true - 变量有值
//          false - 需要被及时清理，当前值可能为 nil
// HaveNew:  true - 需要被分配的新值，当前值可能为 nil
//          false - 不需要分配新值
// CrashIfDeallocating: true - 说明 newObj 已经释放或者 newObj 不支持弱引用，该过程需要暂停
//          false - 用 nil 替代存储
template bool HaveOld, bool HaveNew, bool CrashIfDeallocating>
static id storeWeak(id *location, objc_object *newObj) {
    // 该过程用来更新弱引用指针的指向
    // 初始化 previouslyInitializedClass 指针
    Class previouslyInitializedClass = nil;
    id oldObj;
    // 声明两个 SideTable
    // ① 新旧散列创建
    SideTable *oldTable;
    SideTable *newTable;
    // 获得新值和旧值的锁存位置（用地址作为唯一标示）
    // 通过地址来建立索引标志，防止桶重复
    // 下面指向的操作会改变旧值
  retry:
    if (HaveOld) {
        // 更改指针，获得以 oldObj 为索引所存储的值地址
        oldObj = *location;
        oldTable = &SideTables()[oldObj];
    } else {
        oldTable = nil;
    }
    if (HaveNew) {
        // 更改新值指针，获得以 newObj 为索引所存储的值地址
        newTable = &SideTables()[newObj];
    } else {
        newTable = nil;
    }
    // 加锁操作，防止多线程中竞争冲突
    SideTable::lockTwoHaveOld, HaveNew>(oldTable, newTable);
    // 避免线程冲突重处理
    // location 应该与 oldObj 保持一致，如果不同，说明当前的 location 已经处理过 oldObj 可是又被其他线程所修改
    if (HaveOld  &&  *location != oldObj) {
        SideTable::unlockTwoHaveOld, HaveNew>(oldTable, newTable);
        goto retry;
    }
    // 防止弱引用间死锁
    // 并且通过 +initialize 初始化构造器保证所有弱引用的 isa 非空指向
    if (HaveNew  &&  newObj) {
        // 获得新对象的 isa 指针
        Class cls = newObj->getIsa();
        // 判断 isa 非空且已经初始化
        if (cls != previouslyInitializedClass  &&  
            !((objc_class *)cls)->isInitialized()) {
            // 解锁
            SideTable::unlockTwoHaveOld, HaveNew>(oldTable, newTable);
            // 对其 isa 指针进行初始化
            _class_initialize(_class_getNonMetaClass(cls, (id)newObj));
            // 如果该类已经完成执行 +initialize 方法是最理想情况
            // 如果该类 +initialize 在线程中 
            // 例如 +initialize 正在调用 storeWeak 方法
            // 需要手动对其增加保护策略，并设置 previouslyInitializedClass 指针进行标记
            previouslyInitializedClass = cls;
            // 重新尝试
            goto retry;
        }
    }
    // ② 清除旧值
    if (HaveOld) {
        weak_unregister_no_lock(&oldTable->weak_table, oldObj, location);
    }
    // ③ 分配新值
    if (HaveNew) {
        newObj = (objc_object *)weak_register_no_lock(&newTable->weak_table, 
                                                      (id)newObj, location, 
                                                      CrashIfDeallocating);
        // 如果弱引用被释放 weak_register_no_lock 方法返回 nil 
        // 在引用计数表中设置若引用标记位
        if (newObj  &&  !newObj->isTaggedPointer()) {
            // 弱引用位初始化操作
            // 引用计数那张散列表的weak引用对象的引用计数中标识为weak引用
            newObj->setWeaklyReferenced_nolock();
        }
        // 之前不要设置 location 对象，这里需要更改指针指向
        *location = (id)newObj;
    }
    else {
        // 没有新值，则无需更改
    }
    SideTable::unlockTwoHaveOld, HaveNew>(oldTable, newTable);
    return (id)newObj;
引用计数和弱引用依赖表-SideTable
struct SideTable {
    // 保证原子操作的自旋锁
    spinlock_t slock;
    // 引用计数的 hash 表
    RefcountMap refcnts;
    // weak 引用全局 hash 表
    weak_table_t weak_table;
}
```

上面 objc_storeWeak 方法中，取出实例的方法变成了 &SideTables()[xxxObj]; 这种方式。查看方法的实现，发现了如下函数：

```
static StripedMapSideTable>& SideTables() {
    return *reinterpret_castStripedMapSideTable>*>(SideTableBuf);
}

```
在取出实例方法的实现中，使用了 C++ 标准转换运算符 reinterpret_cast ，其表达方式为：
reinterpret_cast new_type> (expression)
用来处理无关类型之间的转换。该关键字会产生一个新值，并保证与原参数（expression）拥有完全相同的比特位。
而 StripedMap 是一个模板类（Template Class），通过传入类（结构体）参数，会动态修改在该类中的一个 array 成员存储的元素类型，并且其中提供了一个针对于地址的 hash 算法，用作存储 key。可以说， StripedMap 提供了一套拥有将地址作为 key 的 hash table 解决方案，而该方案采用了模板类，是拥有泛型性的。
介绍了与对象相关联的 SideTable 检索方式，再来看 SideTable 的成员和作用。
对于 slock 和 refcnts 两个成员不用多说，第一个是为了防止竞争选择的自旋锁，第二个是协助对象的 isa 指针的 extra_rc 共同引用计数的变量（对于对象结果，在今后的文中提到）。这里主要看 weak 全局 hash 表的结构与作用。

```
struct weak_table_t {
    // 保存了所有指向指定对象的 weak 指针
    weak_entry_t *weak_entries;
    // 存储空间
    size_t    num_entries;
    // 参与判断引用计数辅助量
    uintptr_t mask;
    // hash key 最大偏移值
    uintptr_t max_hash_displacement;
};
```

这是一个全局弱引用表。使用不定类型对象的地址作为 key ，用 weak_entry_t 类型结构体对象作为 value 。其中的 weak_entries 成员，从字面意思上看，即为弱引用表入口。其实现也是这样的。
弱引用的初始化，从上文的分析中可以看出，主要的操作部分就在弱引用表的取键、查询散列、创建弱引用表等操作，可以总结出如下的流程图
这个图中省略了很多情况的判断，但是当声明一个 __weak 会调用上图中的这些方法。当然， storeWeak 方法不仅仅用在 __weak 的声明中，在 class 内部的操作中也会常常通过该方法来对 weak 对象进行操作。

## 4、关联对象有什么应用，系统如何管理关联对象？其被释放的时候需要手动将所有的关联对象的指针置空么？
- 可以不改变源码的情况下增加实例变量。 
- 可与分类配合使用，为分类增加属性。（类别是不能添加成员变量的（property本质也是成员变量 = var + setter、getter），原因是因为一个类的内存大小是固定的，一个雷在load方法执行前就已经加载在内存之中，大小已固定） 
在关联中最主要的一个类就是 AssociationsManager

```
class AssociationsManager {
    static OSSpinLock _lock;
    static AssociationsHashMap *_map;               // associative references:  object pointer -> PtrPtrHashMap.
public:
    AssociationsManager()   { OSSpinLockLock(&_lock); }
    ~AssociationsManager()  { OSSpinLockUnlock(&_lock); }

    AssociationsHashMap &associations() {
        if (_map == NULL)
            _map = new AssociationsHashMap();
        return *_map;
    }
};
```

AssociationsManager里面是由一个静态AssociationsHashMap来存储所有的关联对象的。这相当于把所有对象的关联对象都存在一个全局map里面。而map的的key是这个对象的指针地址（任意两个不同对象的指针地址一定是不同的），而这个map的value又是另外一个AssociationsHashMap，里面保存了关联对象的kv对。
销毁
在obj dealloc时候会调用object_dispose，检查有无关联对象，有的话

```
_object_remove_assocations删除
id object_dispose(id obj){
    if (!obj) return nil;
    // 销毁对象
    objc_destructInstance(obj);    
  // 释放内存
    free(obj);

    return nil;
}

void *objc_destructInstance(id obj) {
    if (obj) {
        // Read all of the flags at once for performance.
        bool cxx = obj->hasCxxDtor();
        bool assoc = obj->hasAssociatedObjects();

        // This order is important.
        // C++ 析构
        if (cxx) object_cxxDestruct(obj);
        // 移除 Associated Object
        if (assoc) _object_remove_assocations(obj);
        // ARC 下调用实例变量的 release 方法，移除 weak 引用
        obj->clearDeallocating();
    }

    return obj;
}
```

## 5、KVO的底层实现？如何取消系统默认的KVO并手动触发（给KVO的触发设定条件：改变的值符合某个条件时再触发KVO）？
- 当观察某对象 A 时，KVO 机制动态创建一个对象A当前类的子类，并为这个新的子类重写了被观察属性 keyPath 的 setter 方法。setter 方法随后负责通知观察对象属性的改变状况。 
- Apple 使用了 isa 混写（isa-swizzling）来实现 KVO 。当观察对象A时，KVO机制动态创建一个新的名为：NSKVONotifying_A 的新类，该类继承自对象A的本类，且 KVO 为 NSKVONotifying_A 重写观察属性的 setter 方法，setter 方法会负责在调用原 setter 方法之前和之后，通知所有观察对象属性值的更改情况。 
修改：使用方法,可实现取消系统kvo，自己触发，也就可控。

```
+(BOOL)automaticallyNotifiesObserversForKey:(NSString *)key{
    if ([key isEqualToString:@"name"]) {
        return NO;
    }else{
        return [super automaticallyNotifiesObserversForKey:key];
    }
}
-(void)setName:(NSString *)name{
    
    if (_name!=name) {
        
        [self willChangeValueForKey:@"name"];
        _name=name;
        [self didChangeValueForKey:@"name"];
    }
      
}
```

## 6、class_ro_t 和 class_rw_t 的区别？
- ObjC 类中的属性、方法还有遵循的协议等信息都保存在 class_rw_t 中： 
- 其中还有一个指向常量的指针 ro，其中存储了当前类在编译期就已经确定的属性、方法以及遵循的协议。

## 7、iOS 中内省的几个方法？class方法和objc_getClass方法有什么区别?
- 	内省方法
	- 判断对象类型:
   	- 1、-(BOOL) isKindOfClass: 判断是否是这个类或者这个类的子类的实例
   	- 2、-(BOOL) isMemberOfClass: 判断是否是这个类的实例
- 判断对象or类是否有这个方法
	- -(BOOL) respondsToSelector: 判读实例是否有这样方法
	- +(BOOL) instancesRespondToSelector: 判断类是否有这个方法
	- object_getClass:获得的是isa的指向
	- self.class:当self是实例对象的时候，返回的是类对象，否则则返回自身。
	- 类方法class，返回的是self，所以当查找meta class时，需要对类对象调用object_getClass方法
## 8、一个int变量被__block修饰与否的区别？
- 没有修饰，被block捕获，是值拷贝。 - 使用__block修饰,会生成一个结构体，复制int的引用地址。达到修改数据。
	[iOS Block 详解](https://juejin.im/entry/588075132f301e00697f18e0)

## 9、哪些场景可以触发离屏渲染？

- 设置了以下属性时，都会触发离屏绘制：
  - shouldRasterize（光栅化）
  - masks（遮罩）
  - shadows（阴影）
  - edge antialiasing（抗锯齿）
  - group opacity（不透明）
  - 复杂形状设置圆角等
  - 渐变

  
## 10、App 启动优化策略？最好结合启动流程来说
[App 启动优化策略](https://mp.weixin.qq.com/s/Kf3EbDIUuf0aWVT-UCEmbA)


## 11、App 无痕埋点的思路了解么？你认为理想的无痕埋点系统应该具备哪些特点？

```
#import <Foundation/Foundation.h>
#import "Sender.h"
#import "TrackRequest.h"

static dispatch_once_t onceToken;

@interface Sender()

@property (strong, nonatomic) dispatch_queue_t sendQueue;
@property (nonatomic, assign) BOOL running;
@property (nonatomic, strong) NSMutableArray *dataArray;
@property (nonatomic, strong) dispatch_semaphore_t semaphore;
@end

@implementation Sender

+ (id)sharedInstance{
    static Sender* sharedInstance;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[Sender alloc] init];
    });
    return sharedInstance;
}

- (id)init{
    if (self = [super init]) {
        self.semaphore = dispatch_semaphore_create(0);
        self.dataArray = [self loadFromFile];
        self.sendQueue = dispatch_queue_create("sendQueue", NULL);
        self.running = YES;
        dispatch_async(self.sendQueue, ^{
            [self performSelector:@selector(executeLoop)];
        });
    }
    return self;
}

- (void)executeLoop{
    while(self.running){
        @synchronized(self) {
            if([self.dataArray count] > 0){
                NSString* data = [self.dataArray objectAtIndex:0];
       
                [TrackRequest sendTrackOTSDataSync:data];
            
                BOOL sendResult = [TrackRequest sendTrackDataSync:data];
                if(sendResult){
                    [self.dataArray removeObjectAtIndex:0];
                }else{
                    [self.dataArray addObject:data];
                    [self.dataArray removeObjectAtIndex:0];
                }
            }
        }
        sleep(1);
        if([self.dataArray count] == 0){
            dispatch_time_t timeout = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(60.0 * NSEC_PER_SEC));
            dispatch_semaphore_wait(self.semaphore, timeout);
        }
        NSLog(@"%@", @"in loop~~~~~~");
    }
}

- (NSMutableArray*)loadFromFile{
    NSString* path = [Sender getSaveFilePath];
    NSMutableArray* dataArray = [NSMutableArray arrayWithContentsOfFile:path];
    if(dataArray == nil){
        dataArray = [NSMutableArray array];
    }
    return dataArray;
}

- (void)saveToFile{
    NSString* path = [Sender getSaveFilePath];
    @synchronized(self) {
        [self.dataArray writeToFile:path atomically:YES];
    }
}

- (void)addToSendingList:(NSString*)data {
    @synchronized(self) {
        [self.dataArray addObject:data];
    }
    dispatch_semaphore_signal(self.semaphore);
}

- (void)exit{
    [self saveToFile];
    self.running = NO;
    onceToken = 0;
}

//获取存储文件路径
+ (NSString*)getSaveFilePath{
    NSArray * paths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    NSString* sandBoxPath = [paths objectAtIndex:0];
    NSString* dataPath = [sandBoxPath stringByAppendingPathComponent:@"track_data"];
    NSFileManager* fm = [NSFileManager defaultManager];
    if(![fm fileExistsAtPath:dataPath]){
        [fm createFileAtPath:dataPath contents:nil attributes:nil];
    }
    return dataPath;
}
@end
```

## 12、你知道有哪些情况会导致app崩溃，分别可以用什么方法拦截并化解？

[NSObjectSafe](https://github.com/jasenhuang/NSObjectSafe/blob/master/NSObjectSafe/NSObjectSafe.m)

- 1、NSInvalidArgumentException 异常,向容器加入nil，引起的崩溃。hook容器添加方法，进行判断。 https://github.com/jasenhuang/NSObjectSafe 
- 2、 SIGSEGV 异常
	- SIGSEGV是当SEGV发生的时候，让代码终止的标识。 当去访问没有被开辟的内存或者已经被释放的内存时，就会发生这样的异常。另外，在低内存的时候，也可能会产生这样的异常。
- 3、 NSRangeException 异常
	- 造成这个异常，就是越界异常了，在iOS中我们经常碰到的越界异常有两种，一种是数组越界，一种字符串截取越界
- 4、SIGPIPE 异常
	- 先解释一下什么是SIGPIPE异常，通俗一点的描述是这样的：对一个端已经关闭的socket调用两次write，第二次write将会产生SIGPIPE信号，该信号默认结束进程。 
	- SIGABRT 异常 这是一个让程序终止的标识，会在断言、app内部、操作系统用终止方法抛出。通常发生在异步执行系统方法的时候。如CoreData、NSUserDefaults等，还有一些其他的系统多线程操作。 注意：这并不一定意味着是系统代码存在bug，代码仅仅是成了无效状态，或者异常状态。 

## 13、App 网络层有哪些优化策略？
- 1、优化DNS解析和缓存
- 2、网络质量检测（根据网络质量来改变策略）
- 3、提供网络服务优先级和依赖机制
- 4、提供网络服务重发机制
- 5、减少数据传输量
- 6、优化海外网络性能

实践

- 每个网络绑定到vc，vc销毁，网络请求销毁。
- 数据请求优先级高于图片请求。网络层做区分。

## 14、TCP为什么要三次握手，四次挥手

[TCP的三次握手与四次挥手（详解+动图)](https://blog.csdn.net/qzcsu/article/details/72861891?utm_source=blogxgwz2)

- 为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误。client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据。但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。 
- tcp是全双工模式，接收到FIN时意味将没有数据再发来，但是还是可以继续发送数据。 




