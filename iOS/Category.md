##1、Category的实现原理，以及Category为什么只能加方法不能加属性。

**分类的作用**   
    
  - 1、声明私有方法
  - 2、分解体积庞大的类文件
  - 3、把Framework的私有方法公开

**特点**
  
  - 1、运行时决议 
  - 2、可以为系统类添加分类

**分类都可以添加哪些内容？**
  
  - 1、实例方法
  - 2、类方法
  - 3、协议
  - 4、属性

```
//类
@interface Person : NSObject
@property (nonatomic ,copy) NSString *presonName;
@end

@implementation Person
- (void)doSomeThing{
   NSLog(@"Person");
}
@end 

// 分类
@interface Person(categoryPerson)
@property (nonatomic ,copy) NSString *categoryPersonName;
@end

@implementation Person(categoryPerson)

- (void)doSomeThing{
   NSLog(@"categoryPerson");
}
@end 
```

**分类的结构体**

```
struct _category_t {
    const char *name;//类名
    struct _class_t *cls;//类
    const struct _method_list_t *instance_methods;//category中所有给类添加的实例方法的列表(instanceMethods)
    const struct _method_list_t *class_methods;//category中所有添加的类方法的列表(classMethods)
    const struct _protocol_list_t *protocols;//category实现的所有协议的列表(protocols)
    const struct _prop_list_t *properties;//category中添加的所有属性(instanceProperties)
};
struct category_t {
    const char *name; // 类名
    classref_t cls; // 分类所属的类
    struct method_list_t *instanceMethods; // 实例方法列表
    struct method_list_t *classMethods; // 类方法列表
    struct protocol_list_t *protocols; // 遵循的协议列表
    struct property_list_t *instanceProperties; // 属性列表
    // 如果是元类,就返回类方法列表;否则返回实例方法列表
    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) {
            return classMethods;
        } else {
            return instanceMethods;
        }
    }
    // 如果是元类,就返回 nil,因为元类没有属性;否则返回实例属性列表,但是...实例属性
    property_list_t *propertiesForMeta(bool isMeta) {
        if (isMeta) {
            return nil; // classProperties;
        } else {
            return instanceProperties;
        }
    }
};
```

**编译时的分类**
 
 - **属性列表**
 
```
// Person(categoryPerson) 属性列表
static struct /*_prop_list_t*/ {
    unsigned int entsize; // sizeof(struct _prop_t)
    unsigned int count_of_properties;
    struct _prop_t prop_list[1];
} _OBJC_$_PROP_LIST_Person_$_categoryPerson __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_prop_t),
    1,
    {{"categoryPersonName","T@NSString",C,N"}}
};
// Person 属性列表
static struct /*_prop_list_t*/ {
    unsigned int entsize; // sizeof(struct _prop_t)
    unsigned int count_of_properties;
    struct _prop_t prop_list[1];
} _OBJC_$_PROP_LIST_Person __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_prop_t),
    1,
    {{"presonName","T@NSString/",C,N,V_presonName"}}
};
```
- **分类的实例变量**

```
// Person 实例变量列表
static struct /*_ivar_list_t*/ {
    unsigned int entsize; // sizeof(struct _prop_t)
    unsigned int count;
    struct _ivar_t ivar_list[1];
} _OBJC_$_INSTANCE_VARIABLES_Person __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_ivar_t),
    1,
    {{(unsigned long int *)&;OBJC_IVAR_$_Person$_presonName, "_presonName", "@/"NSString/"", 3, 8}}
};

```
因为 _category_t 这个结构体中并没有 _ivar_list_t,所以在编译时系统没有Person(categoryPerson) 没有生成类似的相应结构体,也没有生成 _categoryPersonName。

- **分类的实例方法**

```
// Person 实例方法结构体
static struct /*_method_list_t*/ {
    unsigned int entsize; // sizeof(struct _objc_method)
    unsigned int method_count;
    struct _objc_method method_list[3];
} _OBJC_$_INSTANCE_METHODS_Person __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_objc_method),
    3,
    {{(struct objc_selector *)"doSomeThing", "aliyunzixun@xxx.com:8", (void *)_I_Person_doSomeThing},
        {(struct objc_selector *)"presonName", "@aliyunzixun@xxx.com:8", (void *)_I_Person_presonName},
        {(struct objc_selector *)"setPresonName:", "aliyunzixun@xxx.com:aliyunzixun@xxx.com", (void *)_I_Person_setPresonName_}}
};
// Person(categoryPerson )实例方法结构体
static struct /*_method_list_t*/ {
    unsigned int entsize; // sizeof(struct _objc_method)
    unsigned int method_count;
    struct _objc_method method_list[1];
} _OBJC_$_CATEGORY_INSTANCE_METHODS_Person_$_categoryPerson __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_objc_method),
    1,
    {{(struct objc_selector *)"doSomeThing", "aliyunzixun@xxx.com:8", (void *)_I_Person_categoryPerson_doSomeThing}}
}; 
```

对比上述方法可以看到:虽然分类可以声明属性,但是编译时,系统并没有生成分类属性的 get/set 方法,所以,这就是为什么分类要利用runtime 动态添加属性,如何动态添加属性,有兴趣的同学可以查看下面文章 iOS分类中通过runtime添加动态属性

- **分类的结构体**

```
// Person(categoryPerson ) 结构体
static struct _category_t _OBJC_$_CATEGORY_Person_$_categoryPerson __attribute__ ((used, section ("__DATA,__objc_const"))) =
{
    "Person",
    0, // &;OBJC_CLASS_$_Person,
    (const struct _method_list_t *)&;_OBJC_$_CATEGORY_INSTANCE_METHODS_Person_$_categoryPerson,
    0,
    0,
    (const struct _prop_list_t *)&;_OBJC_$_PROP_LIST_Person_$_categoryPerson,
};
```
这是系统在编译时实例化 _category_t 生成的_OBJC_$_CATEGORY_Person_$_categoryPerson


- **分类数组**

```
static struct _category_t *L_OBJC_LABEL_CATEGORY_$ [1] __attribute__((used, section ("__DATA, __objc_catlist,regular,no_dead_strip")))= {
    &;_OBJC_$_CATEGORY_Person_$_categoryPerson,
};
```
编译器最后生成了一个数组,数组的元素就是我们创建的各个分类,用来在运行时加载分类。


##1.1、分类的加载

首先来到runtime初始化函数

![](https://upload-images.jianshu.io/upload_images/1434508-0b43b0c4f1a2f9e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

接着我们来到 &map_images读取模块（images这里代表模块），来到map_images_nolock函数中找到_read_images函数，在_read_images函数中我们找到分类相关代码

**1、加载流程**

```
_objc_init
  └──map_2_images
    └──map_images_nolock
     └──_read_images 
       └── remethodizeClass
```

分类加载的调用栈如上述

- _objc_init 算是整个 objc4 的入口,进行了一些初始化操作,注册了镜像状态改变时的回调函数 
   
- map_2_images 主要是加锁并调用 map_images_nolock 

- map_images_nolock 在这个函数中,完成所有 class 的注册、fixup等工作,还有初始化自动释放池、初始化 side table 等工作并在函数后端调用了 _read_images 

- _read_images 方法干了很多苦力活,比如加载类、Protocol、Category,加载分类的代码就写在 _read_images 函数的尾部 

**2、_read_images 中加载分类的源码**

  加载分类的源码主要做了两件事：
  
  -  1、把category的实例方法、协议以及属性添加到类上 
  -  2、把category的类方法和协议添加到类的metaclass上 


  ![](https://upload-images.jianshu.io/upload_images/325120-0cd5cd18b343761e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

  ![](https://upload-images.jianshu.io/upload_images/325120-8140028a12ed81a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
    ```
    static category_list *
    unattachedCategoriesForClass(Class cls, bool realizing)
    {
        runtimeLock.assertWriting();
        return (category_list *)NXMapRemove(unattachedCategories(), cls);
    }
    ```
  
  ![](https://upload-images.jianshu.io/upload_images/325120-228845c08e31af43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


  ![](https://upload-images.jianshu.io/upload_images/325120-aa07b7ccfd83a752.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


  ![](https://upload-images.jianshu.io/upload_images/325120-38d8f88e3645e372.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
  
  - 1、addUnattachedCategoryForClass(cat, cls, hi) 为类添加未依附的分类
  - 2、从 cats 列表中找到 cls 对应的 unattached 分类的列表
  - 3、将新来的分类 cat 添加刚刚开辟的位置上
  - 4、将新的 list 重新插入 cats 中,会覆盖老的 list
  - 5、执行完上述过程后,系统将这个分类放到了一个 cls 对应的 unattached 分类的 list 中(有点绕口....),这个 list 会在 remethodizeClass(cls) 方法用到

**3、remethodizeClass(cls)**
  
  - 1、取得 cls 类的 unattached 未整合的分类列表
  - 2、将 unattached 的分类列表 attach 到 cls 类上
  - 3、执行完上述过程后,系统就把category的实例方法、协议以及属性添加到类上
  
 ![](https://upload-images.jianshu.io/upload_images/325120-d7f5854046ccc757.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

 ![](https://upload-images.jianshu.io/upload_images/325120-25db5a5edc3b048c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

 ![](https://upload-images.jianshu.io/upload_images/325120-7c0746ab6a0d437f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
 
 下面我们图示经过memmove和memcpy方法过后的内存变化。
 
 ![](https://upload-images.jianshu.io/upload_images/1434508-de4cc8308b362d72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

经过memmove方法之后，内存变化为

![](https://upload-images.jianshu.io/upload_images/1434508-d57c524988754a9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

经过memmove方法之后，我们发现，虽然本类的方法，属性，协议列表会分别后移，但是本类的对应数组的指针依然指向原始位置。

memcpy方法之后，内存变化

![](https://upload-images.jianshu.io/upload_images/1434508-2f70771f3deffad7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

Category可以添加属性，但是并不会自动生成成员变量及set/get方法。因为category_t结构体中并不存在成员变量。通过之前对对象的分析我们知道成员变量是存放在实例对象中的，并且编译的那一刻就已经决定好了。而分类是在运行时才去加载的。那么我们就无法再程序运行时将分类的成员变量中添加到实例对象的结构体中。因此分类中不可以添加成员变量。


##2、Category中有load方法吗？load方法是什么时候调用的？load 方法能继承吗？

- load方法会在程序启动就会调用，当装载类信息的时候就会调用。

调用顺序看一下源代码。

![](https://upload-images.jianshu.io/upload_images/1434508-708c460c656a6f6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

通过源码我们发现是优先调用类的load方法，之后调用分类的load方法。

我们通过代码验证一下：

我们添加Student继承Presen类，并添加Student+Test分类，分别重写只+load方法，其他什么都不做通过打印发现

![](https://upload-images.jianshu.io/upload_images/1434508-1d4aee32a833695e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

确实是优先调用类的load方法之后调用分类的load方法，不过调用类的load方法之前会保证其父类已经调用过load方法。

之后我们为Preson、Student 、Student+Test 添加initialize方法。

我们知道当类第一次接收到消息时，就会调用initialize，相当于第一次使用类的时候就会调用initialize方法。调用子类的initialize之前，会先保证调用父类的initialize方法。如果之前已经调用过initialize，就不会再调用initialize方法了。当分类重写initialize方法时会先调用分类的方法。但是load方法并不会被覆盖，首先我们来看一下initialize的源码。

![](https://upload-images.jianshu.io/upload_images/1434508-ec80b8d39337c614.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/669)

上图中我们发现，initialize是通过消息发送机制调用的，消息发送机制通过isa指针找到对应的方法与实现，因此先找到分类方法中的实现，会优先调用分类方法中的实现。

我们再来看一下load方法的调用源码

![](https://upload-images.jianshu.io/upload_images/1434508-6e323496c3792af8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

我们看到load方法中直接拿到load方法的内存地址直接调用方法，不在是通过消息发送机制调用。

![](https://upload-images.jianshu.io/upload_images/1434508-ad02c1839e1ec5e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

我们可以看到分类中也是通过直接拿到load方法的地址进行调用。因此正如我们之前试验的一样，分类中重写load方法，并不会优先调用分类的load方法，而不调用本类中的load方法了。

因此得出结论：

**Category中有load方法，load方法在程序启动装载类信息的时候就会调用。load方法可以继承。调用子类的load方法之前，会先调用父类的load方法**

##3、load、initialize的区别，以及它们在category重写的时候的调用的次序。

- 区别在于调用方式和调用时刻

  - 调用方式：load是根据函数地址直接调用，initialize是通过objc_msgSend调用

  - 调用时刻：load是runtime加载类、分类的时候调用（只会调用1次），initialize是类第一次接收到消息的时候调用，每一个类只会initialize一次（父类的initialize方法可能会被调用多次）

  - 调用顺序：先调用类的load方法，先编译那个类，就先调用load。在调用load之前会先调用父类的load方法。分类中load方法不会覆盖本类的load方法，先编译的分类优先调用load方法。initialize先初始化父类，之后再初始化子类。如果子类没有实现+initialize，会调用父类的+initialize（所以父类的+initialize可能会被调用多次），如果分类实现了+initialize，就覆盖类本身的+initialize调用。


