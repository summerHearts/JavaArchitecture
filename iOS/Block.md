## 1、Block相关面试题

- 1、block的原理是怎样的？本质是什么？

- 2、__block的作用是什么？有什么使用注意点？
- 3、block的属性修饰词为什么是copy？使用block有哪些使用注意？
- 4、block在修改NSMutableArray，需不需要添加__block？

## 2、对Block有一个基本的认识

- block本质上也是一个oc对象，他内部也有一个isa指针。block是封装了函数调用以及执行上下文的对象。

首先写一个简单的block

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        int age = 10;
        void(^block)(int ,int) = ^(int a, int b){
            NSLog(@"this is block,a = %d,b = %d",a,b);
            NSLog(@"this is block,age = %d",age);
        };
        block(3,5);
    }
    return 0;
}
```
使用命令行将代码转化为c++查看其内部结构，与OC代码进行比较

```
clang -arch arm64 -rewrite-objc main.m
```

![](https://upload-images.jianshu.io/upload_images/1434508-8c9c32b581bccf34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

上图中将c++中block的声明和定义分别与oc代码中相对应显示。将c++中block的声明和调用分别取出来查看其内部实现。

- 定义block变量

```
// 定义block变量代码
void(*block)(int ,int) = ((void (*)(int, int))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, age));
```

上述定义代码中，可以发现，block定义中调用了__main_block_impl_0函数，并且将__main_block_impl_0函数的地址赋值给了block。那么我们来看一下__main_block_impl_0函数内部结构。

- **_main_block_imp_0结构体**

![](https://upload-images.jianshu.io/upload_images/1434508-beb290d0acd377b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

==__main_block_imp_0== 结构体内有一个同名构造函数==__main_block_imp_0==，构造函数中对一些变量进行了赋值最终会返回一个结构体。

那么也就是说最终将一个==__main_block_imp_0==结构体的地址赋值给了block变量

==__main_block_impl_0==结构体内可以发现==__main_block_impl_0==构造函数中传入了四个参数。(void *)__main_block_func_0、&__main_block_desc_0_DATA、age、flags。其中flage有默认值，也就说flage参数在调用的时候可以省略不传。而最后的 age(_age)则表示传入的_age参数会自动赋值给age成员，相当于age = _age。

接下来着重看一下前面三个参数分别代表什么。

==(void *)__main_block_func_0==

![](https://upload-images.jianshu.io/upload_images/1434508-d61313d9e120ec13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

在==__main_block_func_0==函数中首先取出block中age的值，紧接着可以看到两个熟悉的NSLog，可以发现这两段代码恰恰是我们在block块中写下的代码。

那么==__main_block_func_0==函数中其实存储着我们block中写下的代码。而==__main_block_impl_0==函数中传入的是==(void *)__main_block_func_0==，也就说将我们写在block块中的代码封装成==__main_block_func_0==函数，并将==__main_block_func_0==函数的地址传入了==__main_block_impl_0==的构造函数中保存在结构体内。

**__main_block_desc_0_DATA**

![](https://upload-images.jianshu.io/upload_images/1434508-de58ac489063db39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

可以看到__main_block_desc_0中存储着两个参数，reserved和Block_size，并且reserved赋值为0而Block_size则存储着__main_block_impl_0的占用空间大小。最终将__main_block_desc_0结构体的地址传入__main_block_func_0中赋值给Desc。

**age**

age也就是我们定义的局部变量。因为在block块中使用到age局部变量，所以在block声明的时候这里才会将age作为参数传入，也就说block会捕获age，如果没有在block中使用age，这里将只会传入(void *)__main_block_func_0，&__main_block_desc_0_DATA两个参数


这里可以根据源码思考一下为什么当我们在定义block之后修改局部变量age的值，在block调用的时候无法生效。

```
int age = 10;
void(^block)(int ,int) = ^(int a, int b){
     NSLog(@"this is block,a = %d,b = %d",a,b);
     NSLog(@"this is block,age = %d",age);
};
     age = 20;
     block(3,5); 
     // log: this is block,a = 3,b = 5
     //      this is block,age = 10
```
因为block在定义的之后已经将age的值传入存储在__main_block_imp_0结构体中并在调用的时候将age从block中取出来使用，因此在block定义之后对局部变量进行改变是无法被block捕获的。


`此时回过头来查看__main_block_impl_0结构体`

![](https://upload-images.jianshu.io/upload_images/1434508-176664d595708892.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

首先我们看一下__block_impl第一个变量就是__block_impl结构体。

来到__block_impl结构体内部

![](https://upload-images.jianshu.io/upload_images/1434508-e6082d3ab970a01d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

我们可以发现__block_impl结构体内部就有一个isa指针。因此可以证明block本质上就是一个oc对象。而在构造函数中将函数中传入的值分别存储在__main_block_impl_0结构体实例中，最终将结构体的地址赋值给block。

接着通过上面对__main_block_impl_0结构体构造函数三个参数的分析我们可以得出结论：

- 1、 __block_impl结构体中isa指针存储着&_NSConcreteStackBlock地址，可以暂时理解为其类对象地址，block就是_NSConcreteStackBlock类型的。
- 2、 block代码块中的代码被封装成__main_block_func_0函数，FuncPtr则存储着__main_block_func_0函数的地址。
- 3、Desc指向__main_block_desc_0结构体对象，其中存储__main_block_impl_0结构体所占用的内存。

调用block执行内部代码

```
// 执行block内部的代码
((void (*)(__block_impl *, int, int))((__block_impl *)block)->FuncPtr)((__block_impl *)block, 3, 5);
```

通过上述代码可以发现调用block是通过block找到FunPtr直接调用，通过上面分析我们知道block指向的是__main_block_impl_0类型结构体，但是我们发现__main_block_impl_0结构体中并不直接就可以找到FunPtr，而FunPtr是存储在__block_impl中的，为什么block可以直接调用__block_impl中的FunPtr呢？

重新查看上述源代码可以发现，(__block_impl *)block将block强制转化为__block_impl类型的，因为__block_impl是__main_block_impl_0结构体的第一个成员，相当于将__block_impl结构体的成员直接拿出来放在__main_block_impl_0中，那么也就说明__block_impl的内存地址就是__main_block_impl_0结构体的内存地址开头。所以可以转化成功。并找到FunPtr成员。

上面我们知道，FunPtr中存储着通过代码块封装的函数地址，那么调用此函数，也就是会执行代码块中的代码。并且回头查看__main_block_func_0函数，可以发现第一个参数就是__main_block_impl_0类型的指针。也就是说将block传入__main_block_func_0函数中，便于重中取出block捕获的值。


`如何验证block的本质确实是__main_block_impl_0结构体类型。`

通过代码证明一下上述内容：
同样使用之前的方法，我们按照上面分析的block内部结构自定义结构体，并将block内部的结构体强制转化为自定义的结构体，转化成功说明底层结构体确实如我们之前分析的一样。

```
struct __main_block_desc_0 { 
    size_t reserved;
    size_t Block_size;
};
struct __block_impl {
    void *isa;
    int Flags;
    int Reserved;
    void *FuncPtr;
};
// 模仿系统__main_block_impl_0结构体
struct __main_block_impl_0 { 
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    int age;
};
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        int age = 10;
        void(^block)(int ,int) = ^(int a, int b){
            NSLog(@"this is block,a = %d,b = %d",a,b);
            NSLog(@"this is block,age = %d",age);
        };
// 将底层的结构体强制转化为我们自己写的结构体，通过我们自定义的结构体探寻block底层结构体
        struct __main_block_impl_0 *blockStruct = (__bridge struct __main_block_impl_0 *)block;
        block(3,5);
    }
    return 0;
}
```

通过打断点可以看出我们自定义的结构体可以被赋值成功，以及里面的值。

![](https://upload-images.jianshu.io/upload_images/1434508-6c88e7465aac23a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

接下来断点来到block代码块中，看一下堆栈信息中的函数调用地址。Debuf workflow -> always show Disassembly

![](https://upload-images.jianshu.io/upload_images/1434508-7020ffc3c6e733ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

通过上图可以看到地址确实和FuncPtr中的代码块地址一样。

`总结`

此时已经基本对block的底层结构有了基本的认识，上述代码可以通过一张图展示其中各个结构体之间的关系。

![](https://upload-images.jianshu.io/upload_images/1434508-1ecebe9ad79c8c11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

`block底层的数据结构也可以通过一张图来展示`

![](https://upload-images.jianshu.io/upload_images/1434508-b6f59468ae71b22b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

## 3、`Block的变量捕获`

为了保证block内部能够正常访问外部的变量，block有一个变量捕获机制。

对于`基本数据类型`的`局部变量`截获其值。

对于`对象`类型的局部变量`连同所有权修饰符`一起截获。

对`全局变量`、`静态全局变量`不截获。

`局部变量`

  - auto变量

    - 上述代码中我们已经了解过block对age变量的捕获。
    - auto自动变量，离开作用域就销毁，局部变量前面自动添加auto关键字。自动变量会捕获到block内部，也就是说block内部会专门新增加一个参数来存储变量的值。
    - auto只存在于局部变量中，访问方式为值传递，通过上述对age参数的解释我们也可以确定确实是值传递。

  - static变量

    - static 修饰的变量为指针传递，同样会被block捕获。

接下来分别添加aotu修饰的局部变量和static修饰的局部变量，重看源码来看一下他们之间的差别。

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        auto int a = 10;
        static int b = 11;
        void(^block)(void) = ^{
            NSLog(@"hello, a = %d, b = %d", a,b);
        };
        a = 1;
        b = 2;
        block();
    }
    return 0;
}
// log : block本质[57465:18555229] hello, a = 10, b = 2
// block中a的值没有被改变而b的值随外部变化而变化。
```

重新生成c++代码看一下内部结构中两个参数的区别。

![](https://upload-images.jianshu.io/upload_images/1434508-36df0d4129800a40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

从上述源码中可以看出，a,b两个变量都有捕获到block内部。但是a传入的是值，而b传入的则是地址。

为什么两种变量会有这种差异呢，因为自动变量可能会销毁，block在执行的时候有可能自动变量已经被销毁了，那么此时如果再去访问被销毁的地址肯定会发生坏内存访问，因此对于自动变量一定是值传递而不可能是指针传递了。而静态变量不会被销毁，所以完全可以传递地址。而因为传递的是值得地址，所以在block调用之前修改地址中保存的值，block中的地址是不会变得。所以值会随之改变。

  - `全局变量`

    - 我们同样以代码的方式看一下block是否捕获全局变量

  ```
  int a = 10;
static int b = 11;
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        void(^block)(void) = ^{
            NSLog(@"hello, a = %d, b = %d", a,b);
        };
        a = 1;
        b = 2;
        block();
    }
    return 0;
}
// log hello, a = 1, b = 2
  ```
  
同样生成c++代码查看全局变量调用方式

![](https://upload-images.jianshu.io/upload_images/1434508-fd98622fbe7b533b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

通过上述代码可以发现，__main_block_imp_0并没有添加任何变量，因此block不需要捕获全局变量，因为全局变量无论在哪里都可以访问。

局部变量因为跨函数访问所以需要捕获，全局变量在哪里都可以访问 ，所以不用捕获。

最后以一张图做一个总结

![](https://upload-images.jianshu.io/upload_images/1434508-fc81811bcf0e5398.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

总结：局部变量都会被block捕获，自动变量是值捕获，静态变量为地址捕获。全局变量则不会被block捕获

`疑问：以下代码中block是否会捕获变量呢？`

```
#import "Person.h"
@implementation Person
- (void)test
{
    void(^block)(void) = ^{
        NSLog(@"%@",self);
    };
    block();
}
- (instancetype)initWithName:(NSString *)name
{
    if (self = [super init]) {
        self.name = name;
    }
    return self;
}
+ (void) test2
{
    NSLog(@"类方法test2");
}
@end
```

同样转化为c++代码查看其内部结构

![](https://upload-images.jianshu.io/upload_images/1434508-16cb7c2601805d3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

上图中可以发现，self同样被block捕获，接着我们找到test方法可以发现，test方法默认传递了两个参数self和_cmd。而类方法test2也同样默认传递了类对象self和方法选择器_cmd。

![](https://upload-images.jianshu.io/upload_images/1434508-3798a099a1c26fe1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

不论对象方法还是类方法都会默认将self作为参数传递给方法内部，既然是作为参数传入，那么self肯定是局部变量。上面讲到局部变量肯定会被block捕获。

接着我们来看一下如果在block中使用成员变量或者调用实例的属性会有什么不同的结果。

```
- (void)test{
    void(^block)(void) = ^{
        NSLog(@"%@",self.name);
        NSLog(@"%@",_name);
    };
    block();
}
```
![](https://upload-images.jianshu.io/upload_images/1434508-747d16e5f612f9c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

上图中可以发现，即使block中使用的是实例对象的属性，block中捕获的仍然是实例对象，并通过实例对象通过不同的方式去获取使用到的属性


`block的类型`

- block对象是什么类型的，之前稍微提到过，通过源码可以知道block中的isa指针指向的是_NSConcreteStackBlock类对象地址。那么block是否就是_NSConcreteStackBlock类型的呢？

我们通过代码用class方法或者isa指针查看具体类型。

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // __NSGlobalBlock__ : __NSGlobalBlock : NSBlock : NSObject
        void (^block)(void) = ^{
            NSLog(@"Hello");
        };
        
        NSLog(@"%@", [block class]);
        NSLog(@"%@", [[block class] superclass]);
        NSLog(@"%@", [[[block class] superclass] superclass]);
        NSLog(@"%@", [[[[block class] superclass] superclass] superclass]);
    }
    return 0;
}
```
打印内容

![](https://upload-images.jianshu.io/upload_images/1434508-dfe29f5f66092925.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

从上述打印内容可以看出block最终都是继承自NSBlock类型，而NSBlock继承于NSObjcet。那么block其中的isa指针其实是来自NSObject中的。这也更加印证了block的本质其实就是OC对象。


`block的3种类型`

```
__NSGlobalBlock__ （ _NSConcreteGlobalBlock ）
__NSStackBlock__ （ _NSConcreteStackBlock ）
__NSMallocBlock__ （ _NSConcreteMallocBlock ）
```

通过代码查看一下block在什么情况下其类型会各不相同

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // 1. 内部没有调用外部变量的block
        void (^block1)(void) = ^{
            NSLog(@"Hello");
        };
        // 2. 内部调用外部变量的block
        int a = 10;
        void (^block2)(void) = ^{
            NSLog(@"Hello - %d",a);
        };
       // 3. 直接调用的block的class
        NSLog(@"%@ %@ %@", [block1 class], [block2 class], [^{
            NSLog(@"%d",a);
        } class]);
    }
    return 0;
}
```
通过打印内容确实可以发现block的三种类型

![](https://upload-images.jianshu.io/upload_images/1434508-cd8b531d22172049.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

但是我们上面提到过，上述代码转化为c++代码查看源码时却发现block的类型与打印出来的类型不一样，c++源码中三个block的isa指针全部都指向_NSConcreteStackBlock类型地址。

我们可以猜测runtime运行时过程中也许对类型进行了转变。最终类型当然以runtime运行时类型也就是我们打印出的类型为准。


`block在内存中的存储`

通过下面一张图看一下不同block的存放区域

![](https://upload-images.jianshu.io/upload_images/1434508-180cead6473c3ca8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

上图中可以发现，根据block的类型不同，block存放在不同的区域中。

- 数据段中的__NSGlobalBlock__直到程序结束才会被回收，不过我们很少使用到__NSGlobalBlock__类型的block，因为这样使用block并没有什么意义。

- __NSStackBlock__类型的block存放在栈中，我们知道栈中的内存由系统自动分配和释放，作用域执行完毕之后就会被立即释放，而在相同的作用域中定义block并且调用block似乎也多此一举。

- __NSMallocBlock__是在平时编码过程中最常使用到的。存放在堆中需要我们自己进行内存管理。


`block是如何定义其类型`

- block是如何定义其类型，依据什么来为block定义不同的类型并分配在不同的空间呢？首先看下面一张图

![](https://upload-images.jianshu.io/upload_images/1434508-131f0569fa47538a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

接着我们使用代码验证上述问题，首先关闭ARC回到MRC环境下，因为ARC会帮助我们做很多事情，可能会影响我们的观察。

```
// MRC环境！！！
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // Global：没有访问auto变量：__NSGlobalBlock__
        void (^block1)(void) = ^{
            NSLog(@"block1---------");
        };   
        // Stack：访问了auto变量： __NSStackBlock__
        int a = 10;
        void (^block2)(void) = ^{
            NSLog(@"block2---------%d", a);
        };
        NSLog(@"%@ %@", [block1 class], [block2 class]);
        // __NSStackBlock__调用copy ： __NSMallocBlock__
        NSLog(@"%@", [[block2 copy] class]);
    }
    return 0;
}
```
查看打印内容

![](https://upload-images.jianshu.io/upload_images/1434508-832c592e0d5ee523.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

通过打印的内容可以发现正如上图中所示。

- 没有访问auto变量的block是__NSGlobalBlock__类型的，存放在数据段中。
- 访问了auto变量的block是__NSStackBlock__类型的，存放在栈中。
- __NSStackBlock__类型的block调用copy成为__NSMallocBlock__类型并被复制存放在堆中。

上面提到过__NSGlobalBlock__类型的我们很少使用到，因为如果不需要访问外界的变量，直接通过函数实现就可以了，不需要使用block。

但是__NSStackBlock__访问了auto变量，并且是存放在栈中的，上面提到过，栈中的代码在作用域结束之后内存就会被销毁，那么我们很有可能block内存销毁之后才去调用他，那样就会发生问题，通过下面代码可以证实这个问题。

```
void (^block)(void);
void test(){
    // __NSStackBlock__
    int a = 10;
    block = ^{
        NSLog(@"block---------%d", a);
    };
}
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        test();
        block();
    }
    return 0;
}
```

此时查看打印内容

![](https://upload-images.jianshu.io/upload_images/1434508-2d35a43dfac6548b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

可以发现a的值变为了不可控的一个数字。为什么会发生这种情况呢？因为上述代码中创建的block是__NSStackBlock__类型的，因此block是存储在栈中的，那么当test函数执行完毕之后，栈内存中block所占用的内存已经被系统回收，因此就有可能出现乱得数据。查看其c++代码可以更清楚的理解。

![](https://upload-images.jianshu.io/upload_images/1434508-b535042c8eff56a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

为了避免这种情况发生，可以通过copy将NSStackBlock类型的block转化为NSMallocBlock类型的block，将block存储在堆中，以下是修改后的代码。

```
void (^block)(void);
void test()
{
    // __NSStackBlock__ 调用copy 转化为__NSMallocBlock__
    int age = 10;
    block = [^{
        NSLog(@"block---------%d", age);
    } copy];
    [block release];
}
```
此时在打印就会发现数据正确

![](https://upload-images.jianshu.io/upload_images/1434508-7f08293fa997bad1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

那么其他类型的block调用copy会改变block类型吗？下面表格已经展示的很清晰了。

![](https://upload-images.jianshu.io/upload_images/1434508-74811a740c972d8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

所以在平时开发过程中MRC环境下经常需要使用copy来保存block，将栈上的block拷贝到堆中，即使栈上的block被销毁，堆上的block也不会被销毁，需要我们自己调用release操作来销毁。而在ARC环境下系统会自动copy，是block不会被销毁。

`ARC帮我们做了什么`

在ARC环境下，编译器会根据情况自动将栈上的block进行一次copy操作，将block复制到堆上。

什么情况下ARC会自动将block进行一次copy操作？
以下代码都在ARC环境下执行。

`1. block作为函数返回值时`

```
typedef void (^Block)(void);
Block myblock()
{
    int a = 10;
    // 上文提到过，block中访问了auto变量，此时block类型应为__NSStackBlock__
    Block block = ^{
        NSLog(@"---------%d", a);
    };
    return block;
}
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Block block = myblock();
        block();
       // 打印block类型为 __NSMallocBlock__
        NSLog(@"%@",[block class]);
    }
    return 0;
}
```
看一下打印的内容

![](https://upload-images.jianshu.io/upload_images/1434508-1d18acbfdbb984c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

上文提到过，如果在block中访问了auto变量时，block的类型为__NSStackBlock__，上面打印内容发现blcok为__NSMallocBlock__类型的，并且可以正常打印出a的值，说明block内存并没有被销毁。

上面提到过，block进行copy操作会转化为__NSMallocBlock__类型，来讲block复制到堆中，那么说明RAC在 block作为函数返回值时会自动帮助我们对block进行copy操作，以保存block，并在适当的地方进行release操作。

`2. 将block赋值给__strong指针时`

block被强指针引用时，RAC也会自动对block进行一次copy操作

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // block内没有访问auto变量
        Block block = ^{
            NSLog(@"block---------");
        };
        NSLog(@"%@",[block class]);
        int a = 10;
        // block内访问了auto变量，但没有赋值给__strong指针
        NSLog(@"%@",[^{
            NSLog(@"block1---------%d", a);
        } class]);
        // block赋值给__strong指针
        Block block2 = ^{
          NSLog(@"block2---------%d", a);
        };
        NSLog(@"%@",[block1 class]);
    }
    return 0;
}
```
查看打印内容可以看出，当block被赋值给__strong指针时，RAC会自动进行一次copy操作。

![](https://upload-images.jianshu.io/upload_images/1434508-13e4a43cfdd88247.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

`3. block作为Cocoa API中方法名含有usingBlock的方法参数时`

例如：遍历数组的block方法，将block作为参数的时候。

```
NSArray *array = @[];
[array enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
            
}];
```

`4. block作为GCD API的方法参数时`

- 例如：GDC的一次性函数或延迟执行的函数，执行完block操作之后系统才会对block进行release操作。

```
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
            
});        
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            
});
```

`block声明写法`

通过上面对MRC及ARC环境下block的不同类型的分析，总结出不同环境下block属性建议写法。

MRC下block属性的建议写法

- `@property (copy, nonatomic) void (^block)(void);`

ARC下block属性的建议写法

- `@property (strong, nonatomic) void (^block)(void);`
- `@property (copy, nonatomic) void (^block)(void);`


## 4、 探寻block的本质

`block对对象变量的捕获`

- block一般使用过程中都是对对象变量的捕获，那么对象变量的捕获同基本数据类型变量相同吗？
查看一下代码思考：当在block中访问的为对象类型时，对象什么时候会销毁？

```
typedef void (^Block)(void);
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Block block;
        {
            Person *person = [[Person alloc] init];
            person.age = 10;
            
            block = ^{
                NSLog(@"------block内部%d",person.age);
            };
        } // 执行完毕，person没有被释放
        NSLog(@"--------");
    } // person 释放
    return 0; 
}
```
大括号执行完毕之后，person依然不会被释放。上一篇文章提到过，person为aotu变量，传入的block的变量同样为person，即block有一个强引用引用person，所以block不被销毁的话，peroson也不会销毁。

查看源代码确实如此

![](https://upload-images.jianshu.io/upload_images/1434508-781587dce871adee?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

将上述代码转移到MRC环境下，在MRC环境下即使block还在，person却被释放掉了。因为MRC环境下block在栈空间，栈空间对外面的person不会进行强引用。

```
//MRC环境下代码
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Block block;
        {
            Person *person = [[Person alloc] init];
            person.age = 10;
            block = ^{
                NSLog(@"------block内部%d",person.age);
            };
            [person release];
        } // person被释放
        NSLog(@"--------");
    }
    return 0;
}
```
block调用copy操作之后，person不会被释放。

```
block = [^{
   NSLog(@"------block内部%d",person.age);
} copy];
```

上文中也提到过，只需要对栈空间的block进行一次copy操作，将栈空间的block拷贝到堆中，person就不会被释放，说明堆空间的block可能会对person进行一次retain操作，以保证person不会被销毁。堆空间的block自己销毁之后也会对持有的对象进行release操作。

也就是说栈空间上的block不会对对象强引用，堆空间的block有能力持有外部调用的对象，即对对象进行强引用或去除强引用的操作。

`__weak`

- __weak添加之后，person在作用域执行完毕之后就被销毁了。

```
typedef void (^Block)(void);

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Block block;
        {
            Person *person = [[Person alloc] init];
            person.age = 10;
            
            __weak Person *waekPerson = person;
            block = ^{
                NSLog(@"------block内部%d",waekPerson.age);
            };
        }
        NSLog(@"--------");
    }
    return 0;
}
```
将代码转化为c++来看一下上述代码之间的差别。
__weak修饰变量，需要告知编译器使用ARC环境及版本号否则会报错，添加说明-fobjc-arc -fobjc-runtime=ios-8.0.0

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc -fobjc-arc -fobjc-runtime=ios-8.0.0 main.m
```

![](https://upload-images.jianshu.io/upload_images/1434508-adb46896addb1c61?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

__weak修饰的变量，在生成的__main_block_impl_0中也是使用__weak修饰。

`__main_block_copy_0 和 __main_block_dispose_0`

- 当block中捕获对象类型的变量时，我们发现block结构体__main_block_impl_0的描述结构体__main_block_desc_0中多了两个参数copy和dispose函数，查看源码：

![](https://upload-images.jianshu.io/upload_images/1434508-99a6aef47cd003b3?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

copy和dispose函数中传入的都是__main_block_impl_0结构体本身。

  - copy本质就是__main_block_copy_0函数，__main_block_copy_0函数内部调用_Block_object_assign函数，_Block_object_assign中传入的是person对象的地址，person对象，以及8。
  
  - dispose本质就是__main_block_dispose_0函数，__main_block_dispose_0函数内部调用_Block_object_dispose函数，_Block_object_dispose函数传入的参数是person对象，以及8。

`_Block_object_assign函数调用时机及作用`

  - 当block进行copy操作的时候就会自动调用__main_block_desc_0内部的__main_block_copy_0函数，__main_block_copy_0函数内部会调用_Block_object_assign函数。

  - _Block_object_assign函数会自动根据__main_block_impl_0结构体内部的person是什么类型的指针，对person对象产生强引用或者弱引用。可以理解为_Block_object_assign函数内部会对person进行引用计数器的操作，如果__main_block_impl_0结构体内person指针是__strong类型，则为强引用，引用计数+1，如果__main_block_impl_0结构体内person指针是__weak类型，则为弱引用，引用计数不变。

`_Block_object_dispose函数调用时机及作用`

  - 当block从堆中移除时就会自动调用__main_block_desc_0中的__main_block_dispose_0函数，__main_block_dispose_0函数内部会调用_Block_object_dispose函数。

  - _Block_object_dispose会对person对象做释放操作，类似于release，也就是断开对person对象的引用，而person究竟是否被释放还是取决于person对象自己的引用计数。


`总结`

 - 一旦block中捕获的变量为对象类型，block结构体中的__main_block_desc_0会出两个参数copy和dispose。因为访问的是个对象，block希望拥有这个对象，就需要对对象进行引用，也就是进行内存管理的操作。比如说对对象进行retarn操作，因此一旦block捕获的变量是对象类型就会会自动生成copy和dispose来对内部引用的对象进行内存管理。

 - 当block内部访问了对象类型的auto变量时，如果block是在栈上，block内部不会对person产生强引用。不论block结构体内部的变量是__strong修饰还是__weak修饰，都不会对变量产生强引用。

 - 如果block被拷贝到堆上。copy函数会调用_Block_object_assign函数，根据auto变量的修饰符（__strong，__weak，unsafe_unretained）做出相应的操作，形成强引用或者弱引用

 - 如果block从堆中移除，dispose函数会调用_Block_object_dispose函数，自动释放引用的auto变量。



## 5、问题

- 1、下列代码person在何时销毁 ？

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    Person *person = [[Person alloc] init];
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"%@",person);
    });
    NSLog(@"touchBegin----------End");
}
```
打印内容

![](https://upload-images.jianshu.io/upload_images/1434508-251a97e702f24ee7?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

答：上文提到过ARC环境中，block作为GCD API的方法参数时会自动进行copy操作，因此block在堆空间，并且使用强引用访问person对象，因此block内部copy函数会对person进行强引用。当block执行完毕需要被销毁时，调用dispose函数释放对person对象的引用,person没有强指针指向时才会被销毁。

- 2、下列代码person在何时销毁 ？

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    Person *person = [[Person alloc] init];
    
    __weak Person *waekP = person;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"%@",waekP);
    });
    NSLog(@"touchBegin----------End");
}
```

![](https://upload-images.jianshu.io/upload_images/1434508-c4d999795453674c?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

答：block中对waekP为__weak弱引用，因此block内部copy函数会对person同样进行弱引用，当大括号执行完毕时，person对象没有强指针引用就会被释放。因此block块执行的时候打印null。

- 3、通过示例代码进行总结。

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    Person *person = [[Person alloc] init];
    
    __weak Person *waekP = person;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        
        NSLog(@"weakP ----- %@",waekP);
        
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"person ----- %@",person);
        });
    });
    NSLog(@"touchBegin----------End");
}
```

打印内容

![](https://upload-images.jianshu.io/upload_images/1434508-25e0302dfd848278?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

`block内修改变量的值`

本部分分析基于下面代码。

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        int age = 10;
        Block block = ^ {
            // age = 20; // 无法修改
            NSLog(@"%d",age);
        };
        block();
    }
    return 0;
}
```
默认情况下block不能修改外部的局部变量。通过之前对源码的分析可以知道。

age是在main函数内部声明的，说明age的内存存在于main函数的栈空间内部，但是block内部的代码在__main_block_func_0函数内部。__main_block_func_0函数内部无法访问age变量的内存空间，两个函数的栈空间不一样，__main_block_func_0内部拿到的age是block结构体内部的age，因此无法在__main_block_func_0函数内部去修改main函数内部的变量。

`方式一：age使用static修饰。`

前文提到过static修饰的age变量传递到block内部的是指针，在__main_block_func_0函数内部就可以拿到age变量的内存地址，因此就可以在block内部修改age的值。

`方式二：__block`

__block用于解决block内部不能修改auto变量值的问题，__block不能修饰静态变量（static） 和全局变量。

```
__block int age = 10;
```
编译器会将__block修饰的变量包装成一个对象，查看其底层c++源码。

![](https://upload-images.jianshu.io/upload_images/1434508-882a3af49e2cfa65?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

上述源码中可以发现

首先被__block修饰的age变量声明变为名为age的__Block_byref_age_0结构体，也就是说加上__block修饰的话捕获到的block内的变量为__Block_byref_age_0类型的结构体。

通过下图查看__Block_byref_age_0结构体内存储哪些元素。

![](https://upload-images.jianshu.io/upload_images/1434508-6ff119d1f9610b77?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

__isa指针 ：__Block_byref_age_0中也有isa指针也就是说__Block_byref_age_0本质也一个对象。

__forwarding ：__forwarding是__Block_byref_age_0结构体类型的，并且__forwarding存储的值为(__Block_byref_age_0 *)&age，即结构体自己的内存地址。

__flags ：0

__size ：sizeof(__Block_byref_age_0)即__Block_byref_age_0所占用的内存空间。

age ：真正存储变量的地方，这里存储局部变量10。

接着将__Block_byref_age_0结构体age存入__main_block_impl_0结构体中，并赋值给__Block_byref_age_0 *age;

![](https://upload-images.jianshu.io/upload_images/1434508-da97b752ce4c8a0d?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

之后调用block，首先取出__main_block_impl_0中的age，通过age结构体拿到__forwarding指针，上面提到过__forwarding中保存的就是__Block_byref_age_0结构体本身，这里也就是age(__Block_byref_age_0)，在通过__forwarding拿到结构体中的age(10)变量并修改其值。

后续NSLog中使用age时也通过同样的方式获取age的值。

![](https://upload-images.jianshu.io/upload_images/1434508-ec22eca9aad829ac?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


`为什么要通过__forwarding获取age变量的值？`

__forwarding是指向自己的指针。这样的做法是为了方便内存管理，之后内存管理章节会详细解释。

到此为止，__block为什么能修改变量的值已经很清晰了。__block将变量包装成对象，然后在把age封装在结构体里面，block内部存储的变量为结构体指针，也就可以通过指针找到内存地址进而修改变量的值。

`__block修饰对象类型`

那么如果变量本身就是对象类型呢？通过以下代码生成c++源码查看

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        __block Person *person = [[Person alloc] init];
        NSLog(@"%@",person);
        Block block = ^{
            person = [[Person alloc] init];
            NSLog(@"%@",person);
        };
        block();
    }
    return 0;
}
```

通过源码查看，将对象包装在一个新的结构体中。结构体内部会有一个person对象，不一样的地方是结构体内部添加了内存管理的两个函数__Block_byref_id_object_copy和__Block_byref_id_object_dispose

![](https://upload-images.jianshu.io/upload_images/1434508-428bb6dbe75665d8?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

__Block_byref_id_object_copy和__Block_byref_id_object_dispose函数的调用时机及作用在__block内存管理部分详细分析。



`以下代码是否可以正确执行`

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSMutableArray *array = [NSMutableArray array];
        Block block = ^{
            [array addObject: @"5"];
            [array addObject: @"5"];
            NSLog(@"%@",array);
        };
        block();
    }
    return 0;
}
```

答：可以正确执行，因为在block块中仅仅是使用了array的内存地址，往内存地址中添加内容，并没有修改arry的内存地址，因此array不需要使用__block修饰也可以正确编译。

因此当仅仅是使用局部变量的内存地址，而不是修改的时候，尽量不要添加__block，通过上述分析我们知道一旦添加了__block修饰符，系统会自动创建相应的结构体，占用不必要的内存空间。

`上面提到过__block修饰的age变量在编译时会被封装为结构体，那么当在外部使用age变量的时候，使用的是__Block_byref_age_0结构体呢？还是__Block_byref_age_0结构体内的age变量呢？`

为了验证上述问题

同样使用自定义结构体的方式来查看其内部结构

```
typedef void (^Block)(void);

struct __block_impl {
    void *isa;
    int Flags;
    int Reserved;
    void *FuncPtr;
};

struct __main_block_desc_0 {
    size_t reserved;
    size_t Block_size;
    void (*copy)(void);
    void (*dispose)(void);
};

struct __Block_byref_age_0 {
    void *__isa;
    struct __Block_byref_age_0 *__forwarding;
    int __flags;
    int __size;
    int age;
};
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    struct __Block_byref_age_0 *age; // by ref
};

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        __block int age = 10;
        Block block = ^{
            age = 20;
            NSLog(@"age is %d",age);
        };
        block();
        struct __main_block_impl_0 *blockImpl = (__bridge struct __main_block_impl_0 *)block;
        NSLog(@"%p",&age);
    }
    return 0;
}
```

打印断点查看结构体内部结构

![](https://upload-images.jianshu.io/upload_images/1434508-6a30e785a10d8512?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

通过查看blockImpl结构体其中的内容，找到age结构体，其中重点观察两个元素：

__forwarding其中存储的地址确实是age结构体变量自己的地址
age中存储这修改后的变量20。
上面也提到过，在block中使用或修改age的时候都是通过结构体__Block_byref_age_0找到__forwarding在找到变量age的。

另外apple为了隐藏__Block_byref_age_0结构体的实现，打印age变量的地址发现其实是__Block_byref_age_0结构体内age变量的地址。

![](https://upload-images.jianshu.io/upload_images/1434508-0a6b05321bd08463?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

通过上图的计算可以发现打印age的地址同__Block_byref_age_0结构体内age值的地址相同。也就是说外面使用的age，代表的就是结构体内的age值。所以直接拿来用的age就是之前声明的int age。

## 6、__block内存管理


上文提到当block中捕获对象类型的变量时，block中的__main_block_desc_0结构体内部会自动添加copy和dispose函数对捕获的变量进行内存管理。

那么同样的当block内部捕获__block修饰的对象类型的变量时，__Block_byref_person_0结构体内部也会自动添加__Block_byref_id_object_copy和__Block_byref_id_object_dispose对被__block包装成结构体的对象进行内存管理。

当block内存在栈上时，并不会对__block变量产生内存管理。当blcok被copy到堆上时
会调用block内部的copy函数，copy函数内部会调用_Block_object_assign函数，_Block_object_assign函数会对__block变量形成强引用（相当于retain）

首先通过一张图看一下block复制到堆上时内存变化

![](https://upload-images.jianshu.io/upload_images/1434508-2a51e0fcd8cf91de?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

当block被copy到堆上时，block内部引用的__block变量也会被复制到堆上，并且持有变量，如果block复制到堆上的同时，__block变量已经存在堆上了，则不会复制。

当block从堆中移除的话，就会调用dispose函数，也就是__main_block_dispose_0函数，__main_block_dispose_0函数内部会调用_Block_object_dispose函数，会自动释放引用的__block变量。

![](https://upload-images.jianshu.io/upload_images/1434508-b035c93f31e19bff?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


block内部决定什么时候将变量复制到堆中，什么时候对变量做引用计数的操作。

__block修饰的变量在block结构体中一直都是强引用，而其他类型的是由传入的对象指针类型决定。

一段代码更深入的观察一下。

```
typedef void (^Block)(void);
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        int number = 20;
        __block int age = 10;
        
        NSObject *object = [[NSObject alloc] init];
        __weak NSObject *weakObj = object;
        
        Person *p = [[Person alloc] init];
        __block Person *person = p;
        __block __weak Person *weakPerson = p;
        
        Block block = ^ {
            NSLog(@"%d",number); // 局部变量
            NSLog(@"%d",age); // __block修饰的局部变量
            NSLog(@"%p",object); // 对象类型的局部变量
            NSLog(@"%p",weakObj); // __weak修饰的对象类型的局部变量
            NSLog(@"%p",person); // __block修饰的对象类型的局部变量
            NSLog(@"%p",weakPerson); // __block，__weak修饰的对象类型的局部变量
        };
        block();
    }
    return 0;
}
```

将上述代码转化为c++代码查看不同变量之间的区别

```
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  
  int number;
  NSObject *__strong object;
  NSObject *__weak weakObj;
  __Block_byref_age_0 *age; // by ref
  __Block_byref_person_1 *person; // by ref
  __Block_byref_weakPerson_2 *weakPerson; // by ref
  
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _number, NSObject *__strong _object, NSObject *__weak _weakObj, __Block_byref_age_0 *_age, __Block_byref_person_1 *_person, __Block_byref_weakPerson_2 *_weakPerson, int flags=0) : number(_number), object(_object), weakObj(_weakObj), age(_age->__forwarding), person(_person->__forwarding), weakPerson(_weakPerson->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

上述__main_block_impl_0结构体中看出，没有使用__block修饰的变量（object 和 weadObj）则根据他们本身被block捕获的指针类型对他们进行强引用或弱引用，而一旦使用__block修饰的变量，__main_block_impl_0结构体内一律使用强指针引用生成的结构体。

接着我们来看__block修饰的变量生成的结构体有什么不同

```
struct __Block_byref_age_0 {
  void *__isa;
__Block_byref_age_0 *__forwarding;
 int __flags;
 int __size;
 int age;
};

struct __Block_byref_person_1 {
  void *__isa;
__Block_byref_person_1 *__forwarding;
 int __flags;
 int __size;
 void (*__Block_byref_id_object_copy)(void*, void*);
 void (*__Block_byref_id_object_dispose)(void*);
 Person *__strong person;
};

struct __Block_byref_weakPerson_2 {
  void *__isa;
__Block_byref_weakPerson_2 *__forwarding;
 int __flags;
 int __size;
 void (*__Block_byref_id_object_copy)(void*, void*);
 void (*__Block_byref_id_object_dispose)(void*);
 Person *__weak weakPerson;
};
```

如上面分析的那样，__block修饰对象类型的变量生成的结构体内部多了__Block_byref_id_object_copy和__Block_byref_id_object_dispose两个函数，用来对对象类型的变量进行内存管理的操作。而结构体对对象的引用类型，则取决于block捕获的对象类型的变量。weakPerson是弱指针，所以__Block_byref_weakPerson_2对weakPerson就是弱引用，person是强指针，所以__Block_byref_person_1对person就是强引用。

```
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {
    _Block_object_assign((void*)&dst->age, (void*)src->age, 8/*BLOCK_FIELD_IS_BYREF*/);
    _Block_object_assign((void*)&dst->object, (void*)src->object, 3/*BLOCK_FIELD_IS_OBJECT*/);
    _Block_object_assign((void*)&dst->weakObj, (void*)src->weakObj, 3/*BLOCK_FIELD_IS_OBJECT*/);
    _Block_object_assign((void*)&dst->person, (void*)src->person, 8/*BLOCK_FIELD_IS_BYREF*/);
    _Block_object_assign((void*)&dst->weakPerson, (void*)src->weakPerson, 8/*BLOCK_FIELD_IS_BYREF*/);
}
```


__main_block_copy_0函数中会根据变量是强弱指针及有没有被__block修饰做出不同的处理，强指针在block内部产生强引用，弱指针在block内部产生弱引用。被__block修饰的变量最后的参数传入的是8，没有被__block修饰的变量最后的参数传入的是3。

当block从堆中移除时通过dispose函数来释放他们。

```
static void __main_block_dispose_0(struct __main_block_impl_0*src) {
    _Block_object_dispose((void*)src->age, 8/*BLOCK_FIELD_IS_BYREF*/);
    _Block_object_dispose((void*)src->object, 3/*BLOCK_FIELD_IS_OBJECT*/);
    _Block_object_dispose((void*)src->weakObj, 3/*BLOCK_FIELD_IS_OBJECT*/);
    _Block_object_dispose((void*)src->person, 8/*BLOCK_FIELD_IS_BYREF*/);
    _Block_object_dispose((void*)src->weakPerson, 8/*BLOCK_FIELD_IS_BYREF*/);
    
}
```

`__forwarding指针`

上面提到过__forwarding指针指向的是结构体自己。当使用变量的时候，通过结构体找到__forwarding指针，在通过__forwarding指针找到相应的变量。这样设计的目的是为了方便内存管理。通过上面对__block变量的内存管理分析我们知道，block被复制到堆上时，会将block中引用的变量也复制到堆中。

我们重回到源码中。当在block中修改__block修饰的变量时。

```
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  __Block_byref_age_0 *age = __cself->age; // bound by ref
            (age->__forwarding->age) = 20;
            NSLog((NSString *)&__NSConstantStringImpl__var_folders_jm_dztwxsdn7bvbz__xj2vlp8980000gn_T_main_b05610_mi_0,(age->__forwarding->age));
        }
```


通过源码可以知道，当修改__block修饰的变量时，是根据变量生成的结构体这里是__Block_byref_age_0找到其中__forwarding指针，__forwarding指针指向的是结构体自己因此可以找到age变量进行修改。

当block在栈中时，__Block_byref_age_0结构体内的__forwarding指针指向结构体自己。

而当block被复制到堆中时，栈中的__Block_byref_age_0结构体也会被复制到堆中一份，而此时栈中的__Block_byref_age_0结构体中的__forwarding指针指向的就是堆中的__Block_byref_age_0结构体，堆中__Block_byref_age_0结构体内的__forwarding指针依然指向自己。

此时当对age进行修改时

```
// 栈中的age
__Block_byref_age_0 *age = __cself->age; // bound by ref
// age->__forwarding获取堆中的age结构体
// age->__forwarding->age 修改堆中age结构体的age变量
(age->__forwarding->age) = 20;
```

通过__forwarding指针巧妙的将修改的变量赋值在堆中的__Block_byref_age_0中。

我们通过一张图展示__forwarding指针的作用

![](https://upload-images.jianshu.io/upload_images/1434508-f1196104d11e8d0b?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

因此block内部拿到的变量实际就是在堆上的。当block进行copy被复制到堆上时，_Block_object_assign函数内做的这一系列操作。

**被__block修饰的对象类型的内存管理**

使用以下代码，生成c++代码查看内部实现

```
typedef void (^Block)(void);
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        __block Person *person = [[Person alloc] init];
        Block block = ^ {
            NSLog(@"%p", person);
        };
        block();
    }
    return 0;
}
```
来到源码查看__Block_byref_person_0结构体及其声明

```
__Block_byref_person_0结构体

typedef void (*Block)(void);
struct __Block_byref_person_0 {
  void *__isa;  // 8 内存空间
__Block_byref_person_0 *__forwarding; // 8
 int __flags; // 4
 int __size;  // 4
 void (*__Block_byref_id_object_copy)(void*, void*); // 8
 void (*__Block_byref_id_object_dispose)(void*); // 8
 Person *__strong person; // 8
};
// 8 + 8 + 4 + 4 + 8 + 8 + 8 = 48
```

```
// __Block_byref_person_0结构体声明

__attribute__((__blocks__(byref))) __Block_byref_person_0 person = {
    (void*)0,
    (__Block_byref_person_0 *)&person,
    33554432,
    sizeof(__Block_byref_person_0),
    __Block_byref_id_object_copy_131,
    __Block_byref_id_object_dispose_131,
    
    ((Person *(*)(id, SEL))(void *)objc_msgSend)((id)((Person *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("Person"), sel_registerName("alloc")), sel_registerName("init"))
};
```

之前提到过__block修饰的对象类型生成的结构体中新增加了两个函数void (*__Block_byref_id_object_copy)(void*, void*);和void (*__Block_byref_id_object_dispose)(void*);。这两个函数为__block修饰的对象提供了内存管理的操作。

可以看出为void (*__Block_byref_id_object_copy)(void*, void*);和void (*__Block_byref_id_object_dispose)(void*);赋值的分别为__Block_byref_id_object_copy_131和__Block_byref_id_object_dispose_131。找到这两个函数

```
static void __Block_byref_id_object_copy_131(void *dst, void *src) {
 _Block_object_assign((char*)dst + 40, *(void * *) ((char*)src + 40), 131);
}
static void __Block_byref_id_object_dispose_131(void *src) {
 _Block_object_dispose(*(void * *) ((char*)src + 40), 131);
}
```

上述源码中可以发现__Block_byref_id_object_copy_131函数中同样调用了_Block_object_assign函数，而_Block_object_assign函数内部拿到dst指针即block对象自己的地址值加上40个字节。并且_Block_object_assign最后传入的参数是131，同block直接对对象进行内存管理传入的参数3，8都不同。可以猜想_Block_object_assign内部根据传入的参数不同进行不同的操作的。

通过对上面__Block_byref_person_0结构体占用空间计算发现__Block_byref_person_0结构体占用的空间为48个字节。而加40恰好指向的就为person指针。

也就是说copy函数会将person地址传入_Block_object_assign函数，_Block_object_assign中对Person对象进行强引用或者弱引用。

![](https://upload-images.jianshu.io/upload_images/1434508-29bf2e512e34f437?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

如果使用__weak修饰变量查看一下其中的源码

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *person = [[Person alloc] init];
        __block __weak Person *weakPerson = person;
        Block block = ^ {
            NSLog(@"%p", weakPerson);
        };
        block();
    }
    return 0;
}
```

```
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __Block_byref_weakPerson_0 *weakPerson; // by ref
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_weakPerson_0 *_weakPerson, int flags=0) : weakPerson(_weakPerson->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```
__main_block_impl_0中没有任何变化，__main_block_impl_0对weakPerson依然是强引用，但是__Block_byref_weakPerson_0中对weakPerson变为了__weak指针。

```
struct __Block_byref_weakPerson_0 {
  void *__isa;
__Block_byref_weakPerson_0 *__forwarding;
 int __flags;
 int __size;
 void (*__Block_byref_id_object_copy)(void*, void*);
 void (*__Block_byref_id_object_dispose)(void*);
 Person *__weak weakPerson;
};
```

也就是说无论如何block内部中对__block修饰变量生成的结构体都是强引用，结构体内部对外部变量的引用取决于传入block内部的变量是强引用还是弱引用。

![](https://upload-images.jianshu.io/upload_images/1434508-6ea9bd1eb694b9ea?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

mrc环境下，尽管调用了copy操作，__block结构体不会对person产生强引用，依然是弱引用。

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        __block Person *person = [[Person alloc] init];
        Block block = [^ {
            NSLog(@"%p", person);
        } copy];
        [person release];
        block();
        [block release];
    }
    return 0;
}
```
上述代码person会先释放

```
block的copy[50480:8737001] -[Person dealloc]
block的copy[50480:8737001] 0x100669a50
```
当block从堆中移除的时候。会调用dispose函数，block块中去除对__Block_byref_person_0 *person;的引用，__Block_byref_person_0结构体中也会调用dispose操作去除对Person *person;的引用。以保证结构体和结构体内部的对象可以正常释放。

## 7、循环引用

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *person = [[Person alloc] init];
        person.age = 10;
        person.block = ^{
            NSLog(@"%d",person.age);
        };
    }
    NSLog(@"大括号结束啦");
    return 0;
}
```
运行代码打印内容

```
block的copy[55423:9158212] 大括号结束啦
```

可以发现大括号结束之后，person依然没有被释放，产生了循环引用。

通过一张图看一下他们之间的内存结构

![](https://upload-images.jianshu.io/upload_images/1434508-15830a121b61ca4e?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

上图中可以发现，Person对象和block对象相互之间产生了强引用，导致双方都不会被释放，进而造成内存泄漏。

- 解决循环引用问题 - ARC

  首先为了能随时执行block，我们肯定希望person对block对强引用，而block内部对person的引用为弱引用最好。

  使用__weak 和 __unsafe_unretained修饰符可以解决循环引用的问题

  我们上面也提到过__weak会使block内部将指针变为弱指针。block对person对象为弱指针的话，也就不会出现相互引用而导致不会被释放了。
  
  ![](https://upload-images.jianshu.io/upload_images/1434508-710648fce77ee7d0?imageMogr2/auto-orient/strip%7CimageView2/2/w/561)
  
  __weak 和 __unsafe_unretained的区别。

    - __weak不会产生强引用，指向的对象销毁时，会自动将指针置为nil。因此一般通过__weak来解决问题。
 
    - __unsafe_unretained不会产生前引用，不安全，指向的对象销毁时，指针存储的地址值不变。

使用__block也可以解决循环引用的问题。

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        __block Person *person = [[Person alloc] init];
        person.age = 10;
        person.block = ^{
            NSLog(@"%d",person.age);
            person = nil;
        };
        person.block();
    }
    NSLog(@"大括号结束啦");
    return 0;
}
```
上述代码之间的相互引用可以使用下图表示

![](https://upload-images.jianshu.io/upload_images/1434508-1562170389513df5?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

上面我们提到过，在block内部使用变量使用的其实是__block修饰的变量生成的结构体__Block_byref_person_0内部的person对象，那么当person对象置为nil也就断开了结构体对person的强引用，那么三角的循环引用就自动断开。该释放的时候就会释放了。但是有弊端，必须执行block，并且在block内部将person对象置为nil。也就是说在block执行之前代码是因为循环引用导致内存泄漏的。

- 解决循环引用问题 - MRC

  - 使用__unsafe_unretained解决。在MRC环境下不支持使用__weak，使用原理同ARC环境下相同，这里不在赘述。

  - 使用__block也能解决循环引用的问题。因为上文__block内存管理中提到过，MRC环境下，尽管调用了copy操作，__block结构体不会对person产生强引用，依然是弱引用。因此同样可以解决循环引用的问题。

- __strong 和 __weak

 ```
 __weak typeof(self) weakSelf = self;
person.block = ^{
    __strong typeof(weakSelf) myself = weakSelf;
    NSLog(@"age is %d", myself->_age);
};
 ```
 在block内部重新使用__strong修饰self变量是为了在block内部有一个强指针指向weakSelf避免在block调用的时候weakSelf已经被销毁。

