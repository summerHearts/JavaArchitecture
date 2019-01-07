##1、内存布局

![](https://www.icheesedu.com/images/内存分布.png)

- `stack`:方法调用

- `heap`:通过alloc等分配对象

- `bss`:未初始化的全局变量等。

- `data`:已初始化的全局变量等。

- `text`:程序代码。

内存条中主要分为几大类：栈区(stack)、堆区(heap)、常量区、代码区(.text)、保留区。常量区分为未初始化区域(.bss)和已初始化区域(.data)，栈区stack存储顺序是由高地址存向低地址，而堆区是由低地址向高地址存储。内存条中地址由低到高的区域分别为：保留区，代码区，已初始化区(.data)，未初始化区(.bss)，堆区(heap)，栈区(stack)，内核区。而程序员操作的主要是栈区与堆区还有常量区。


申请后的系统是如何响应的？

`栈：`

  - 只要栈的剩余空间大于所申请空间，系统将为程序提供内存，否则将报异常提示栈溢出。
  
`堆：`

  - 首先应该知道操作系统有一个记录空闲内存地址的链表。当系统收到程序的申请时，会遍历该链表，寻找第一个空间大于所申请空间的堆结点，然后将该结点从空闲结点链表中删除，并将该结点的空间分配给程序。由于找到的堆结点的大小不一定正好等于申请的大小，系统会自动的将多余的那部分重新放入空闲链表中。

`申请大小的限制是怎样的？`

  - 栈：栈是向低地址扩展的数据结构，是一块连续的内存的区域。是栈顶的地址和栈的最大容量是系统预先规定好的，栈的大小是2M（也有的说是1M，总之是一个编译时就确定的常数 ) ,如果申请的空间超过栈的剩余空间时，将提示overflow。因此，能从栈获得的空间较小。
  - 堆：堆是向高地址扩展的数据结构，是不连续的内存区域。这是由于系统是用链表来存储的空闲内存地址的，自然是不连续的，而链表的遍历方向是由低地址向高地址。堆的大小受限于计算机系统中有效的虚拟内存。由此可见，堆获得的空间比较灵活，也比较大。

`申请效率的比较？`

  - 栈：由系统自动分配，速度较快。但程序员是无法控制的。
  - 堆：是由new分配的内存，一般速度比较慢，而且容易产生内存碎片,不过用起来最方便.


##2、内存管理方案

- 1、`TaggedPointer`:小对象，比如NSNumber

  其主要的原理就是在对象的指针中加入特定需要记录的信息，以及对象所对应的值，在64位的系统中，一个指针所占用的内存空间为8个字节，已足以存下一些小型的数据量了，当对象指针的空间中存满后，再对指针所指向的内存区域进行存储，这就是taggedPointer。距离NSNumber，最低4位用于标记是什么类型的数据（long为3，float则为4，Int为2，double为5），而最高4位的“b”表示是NSNumber类型；其余56位则用来存储数值本身内容。


- 2、`NONPOINTER_ISA`:objc_objcet对象中isa指针分为指针型isa与非指针型isa(NONPOINTER_ISA)，运用的便是类似这种技术。

  在一个64位的指针内存中，第0位存储的是indexed标识符，它代表一个指针是否为NONPOINTER型，0代表不是，1代表是。第1位has_assoc，顾名思义，1代表其指向的实例变量含有关联对象，0则为否。第2位为has_cxx_dtor，表明该对象是否包含C++相关的内容或者该对象是否使用ARC来管理内存，如果含有C++相关内容或者使用了ARC来管理对象，这一块都表示为YES，第3-35位shiftcls存储的就是这个指针的地址。第42位为weakly_referenced，表明该指针对象是否有弱引用的指针指向。第43位为deallocing，表明该对象是否正在被回收。第44位为has_sidetable_rc，顾名思义，该指针是否引用了sidetable散列表。第45-63位extra_rc装的就是这个实例变量的引用计数，当对象被引用时，其引用计数+1，但少量的引用计数是不会直接存放在sideTables表中的，对象的引用计数会先存在NONPOINTER_ISA的指针中的45-63位，当其被存满后，才会相应存入sideTables散列表中。
  
  ![](https://www.icheesedu.com/images/NONPOINTER_ISA-0-31.png)
  
  ![](https://www.icheesedu.com/images/NONPOINTER_ISA-47-64.png)

- 3、散列表

    散列表在系统中的提现是一个`sideTables`的哈希映射表，其中所有对象的引用计数（除上述存在NONPOINTER_ISA中的外）都存在这个`sideTables`散列表中，而一个散列表中又包含众多`sideTable`。每个SideTable中又包含了三个元素，spinlock_t自旋锁，RefcountMap引用计数表，weak_table_t弱引用表。所以既然SideTables是一个哈希映射的表，为什么不用SideTables直接包含自旋锁，引用技术表和弱引用表呢？因为在众多线程同时访问这个SideTables表的时候，为了保证数据安全，需要给其加上自旋锁，如果只有一张SideTable的表，那么所有数据访问都会出一个进一个，单线程进行，非常影响效率，而且会带来不好的用户体验，针对这种情况，将一张SideTables分为多张表的SideTable，再各自加锁保证数据的安全，这样就增加了并发量，提高了数据访问的效率，所以这就是一张SideTables表下涵盖众多SideTable表的原因。
    
    基于此，我们进行SideTable的表分析，那么当一个对象的引用计数增加或减少时，需要去查找对应的SideTable并进行引用计数或者弱引用计数的操作时，系统又是怎样实现的呢。
    
    当一个对象访问SideTables时，首先会取到对象的地址，将地址进行哈希运算，与SideTables的个数取余，最后得到的结果就是该对象所要访问的SideTable所在SideTables中的位置，随后在取到的SideTable中的RefcountMap表中再次进行一次哈希查找，找到该对象在引用计数表中所对应的位置，如果该位置存在对应的引用计数，则对其进行操作，如果没有对应的引用计数，则创建一个对应的size_t对象，其实就是一个uint类型的无符号整型。
       
   ![](https://www.icheesedu.com/images/sidetables.png)
   
   ![](https://www.icheesedu.com/images/sidetable.png)
   

##3、数据结构

 
对于Spinlock_t自旋锁，其本质是一种“忙等”的锁，所谓“忙等”就是当一条线程被加上Spinlock自旋锁后，当线程执行时，会不断的去获取这个锁的信息，一旦获取到这个锁，便进行线程的执行。这对于一般的高性能锁比如信号量不同，信号量是当线程获取到信号量小于等0时，便自动进行休眠，当信号量发出时，对线程进行唤醒操作，这样就致使了两种锁的性质不同。Spinlock自旋锁只适用于一些小型数据操作，耗时很少的线程操作。
    
对于每张SideTable表中的弱引用表weak_table_t，其也是一张哈希表的结构，其内部包含了每个对象对应的弱引用表weak_entry_t，而weak_entry_t是一个结构体数组，其中包含的则是每一个对象弱引用的对象所对应的弱引用指针。

 ![](https://www.icheesedu.com/images/分离锁.png)
   
 ![](https://www.icheesedu.com/images/引用计数表.png)
   
 ![](https://www.icheesedu.com/images/size_t数据结构含义.png)
   
 ![](https://www.icheesedu.com/images/weak_table_t.png)


##4、ARC & MRC

![](https://www.icheesedu.com/images/mrc.png)

- **MRC**:手动管理对象引用计数的方式，但这也是内存管理的立足之本.在MRC中可以调用`alloc`,`retain`,`release`,`retainCount`,`dealloc`等方法，这些方法在ARC中只能调用alloc方法，调用其他的会引起编译报错，不过在ARC模式中可以重写dealloc方法。


![](https://www.icheesedu.com/images/arc.png)

- **ARC**:就是现代程序员常用的对象引用计数管理方式，ARC是由编译器和runtime协作，共同完成对对象引用计数的控制，而不需要程序员自己手动控制。ARC中**禁止手动调用**`retain/realse/retainCount/dealloc`。相比起MRC，在ARC中新增了weak和strong等属性关键字。

##5、引用计数

- `alloc`：这个方法实质上是经过了一系列调用,最终调用了C函数的calloc，需要注意的是调用该方法时，对象的引用计数并没有+1.

    ```
    id
    objc_object::sidetable_retain()
    {
    #if SUPPORT_NONPOINTER_ISA
        assert(!isa.indexed);
    #endif
        SideTable& table = SideTables()[this];
    
        if (table.trylock()) {
            size_t& refcntStorage = table.refcnts[this];
            if (! (refcntStorage & SIDE_TABLE_RC_PINNED)) {
                refcntStorage += SIDE_TABLE_RC_ONE;
            }
            table.unlock();
            return (id)this;
        }
        return sidetable_retain_slow(table);
    }
    ```
    ![](https://www.icheesedu.com/images/retain.png)

- `retain`：这个方法是先在SideTables中通过哈希查找找到对象所在的那张SideTable表，随后在SideTable中的引用计数表中再次通过哈希查找找到对象所对应的size_t，再加上一个系统的（引用计数+1宏）。为什么这里没有+1而是加上一个系统的宏呢，因为在size_t结构中，前两位不是储存引用计数的，第一位存储的是是否有弱引用指针指向，第二位存储的是对象是否在被回收中。所以，在增加其引用计数时需要右移两位再进行增加，所以用到了这个系统的宏SIDE_TABLE_RC_ONE。

  ```
  uintptr_t 
objc_object::sidetable_release(bool performDealloc)
{
#if SUPPORT_NONPOINTER_ISA
    assert(!isa.indexed);
#endif
    SideTable& table = SideTables()[this];

    bool do_dealloc = false;

    if (table.trylock()) {
        RefcountMap::iterator it = table.refcnts.find(this);
        if (it == table.refcnts.end()) {
            do_dealloc = true;
            table.refcnts[this] = SIDE_TABLE_DEALLOCATING;
        } else if (it->second < SIDE_TABLE_DEALLOCATING) {
            // SIDE_TABLE_WEAKLY_REFERENCED may be set. Don't change it.
            do_dealloc = true;
            it->second |= SIDE_TABLE_DEALLOCATING;
        } else if (! (it->second & SIDE_TABLE_RC_PINNED)) {
            it->second -= SIDE_TABLE_RC_ONE;
        }
        table.unlock();
        if (do_dealloc  &&  performDealloc) {
            ((void(*)(objc_object *, SEL))objc_msgSend)(this, SEL_dealloc);
        }
        return do_dealloc;
    }

    return sidetable_release_slow(table, performDealloc);
}
  ```

- `release`：这个方法跟retain方法原理一样，只不过是减一个系统的宏SIDE_TABLE_RC_ONE


  ```
  uintptr_t
objc_object::sidetable_retainCount()
{
    SideTable& table = SideTables()[this];

    size_t refcnt_result = 1;
    
    table.lock();
    RefcountMap::iterator it = table.refcnts.find(this);
    if (it != table.refcnts.end()) {
        // this is valid for SIDE_TABLE_RC_PINNED too
        refcnt_result += it->second >> SIDE_TABLE_RC_SHIFT;
    }
    table.unlock();
    return refcnt_result;
}
  ```
  
- `retainCount`：这个方法的实现同样是先查找系统的SideTables表，并找到对象对应的SideTable表，但在之前要先申明一个size_t为1的对象，随后在对应的引用计数表中找到了对象对应的引用计数后，通过右移找到的count对象，与之前创建好的1相加，最后返回其结果便是引用计数。所以这就是为什么系统在调用alloc方法后并没有给对象的引用计数+1，但retainCount方法调用后对象的引用计数就是1的原因。

  ```
  inline void
objc_object::rootDealloc()
{
        assert(!UseGC);
        if (isTaggedPointer()) return;
    
        if (isa.indexed  &&  
            !isa.weakly_referenced  &&  
            !isa.has_assoc  &&  
            !isa.has_cxx_dtor  &&  
            !isa.has_sidetable_rc)
        {
            assert(!sidetable_present());
            free(this);
        } 
        else {
            object_dispose((id)this);
        }
}
  ```
  
  ```
      id 
    object_dispose(id obj)
    {
        if (!obj) return nil;
    
        objc_destructInstance(obj);
        
    #if SUPPORT_GC
        if (UseGC) {
            auto_zone_retain(gc_zone, obj); // gc free expects rc==1
        }
    #endif
    
        free(obj);
    
        return nil;
    }
  ```
  
  ```
  void *objc_destructInstance(id obj) {
    if (obj) {
        // Read all of the flags at once for performance.
        bool cxx = obj->hasCxxDtor();
        bool assoc = !UseGC && obj->hasAssociatedObjects();
        bool dealloc = !UseGC;

        // This order is important.
        if (cxx) object_cxxDestruct(obj);
        if (assoc) _object_remove_assocations(obj);
        if (dealloc) obj->clearDeallocating();
    }

    return obj;
}
  ```
  
  ```
    inline void 
    objc_object::clearDeallocating()
    {
        if (!isa.indexed) {
            // Slow path for raw pointer isa.
            sidetable_clearDeallocating();
        }
        else if (isa.weakly_referenced  ||  isa.has_sidetable_rc) {
            // Slow path for non-pointer isa with weak refs and/or side table data.
            clearDeallocating_slow();
        }
    
        assert(!sidetable_present());
    }
  ```
  
  ```
    void 
objc_object::sidetable_clearDeallocating()
{
    SideTable& table = SideTables()[this];

    // clear any weak table items
    // clear extra retain count and deallocating bit
    // (fixme warn or abort if extra retain count == 0 ?)
    table.lock();
    RefcountMap::iterator it = table.refcnts.find(this);
    if (it != table.refcnts.end()) {
        if (it->second & SIDE_TABLE_WEAKLY_REFERENCED) {
            //将指向该对象的弱引用指针置为nil
            weak_clear_no_lock(&table.weak_table, (id)this);
        }
        //从引用计数表中擦除该对象引用计数
        table.refcnts.erase(it);
    }
    table.unlock();
}
  ```
  
  ```
    void 
weak_clear_no_lock(weak_table_t *weak_table, id referent_id) 
{
    objc_object *referent = (objc_object *)referent_id;

    weak_entry_t *entry = weak_entry_for_referent(weak_table, referent);
    if (entry == nil) {
        /// XXX shouldn't happen, but does with mismatched CF/objc
        //printf("XXX no entry for clear deallocating %p\n", referent);
        return;
    }

    // zero out references
    weak_referrer_t *referrers;
    size_t count;
    
    if (entry->out_of_line) {
        referrers = entry->referrers;
        count = TABLE_SIZE(entry);
    } 
    else {
        referrers = entry->inline_referrers;
        count = WEAK_INLINE_COUNT;
    }
    
    for (size_t i = 0; i < count; ++i) {
        objc_object **referrer = referrers[i];
        if (referrer) {
            if (*referrer == referent) {
                *referrer = nil;
            }
            else if (*referrer) {
                _objc_inform("__weak variable at %p holds %p instead of %p. "
                             "This is probably incorrect use of "
                             "objc_storeWeak() and objc_loadWeak(). "
                             "Break on objc_weak_error to debug.\n", 
                             referrer, (void*)*referrer, (void*)referent);
                objc_weak_error();
            }
        }
    }
    
    weak_entry_remove(weak_table, entry);
}
  ```
  
   ![](https://www.icheesedu.com/images/dealloc.png)

- `dealloc`：对象在被回收时，就会调用dealloc方法，其内部实现流程首先要调用一个_objc_rootDealloc()方法，再方法内部再调用一个rootDealloc()方法，此时在rootDealloc中会判断该对象的isa指针，依次判断指针内的内容：nonpointer_isa，weakly_referenced,has_assoc,has_cxx_dtor,has_sidetable_rc，如果判断结果为：该isa指针不是非指针型的isa指针，没有弱引用的指针指向，没有相应的关联对象，没有c++相关的内容，没有使用ARC模式，没有关联到散列表中，即判断的内容都为否，则可以直接调用c语言中的free()函数进行相应的内存释放，否则就会调用objc_dispose()这个函数。

##6、弱引用

当我们创建一个弱引用变量weakPointer的时候在编译器中可以这么写

```
id __weak weakPointer = object;
```

这行代码实际上在系统内部实现的时候转化为了两行代码：

```
id weakPointer;

objc_initWeak(&weakPointer,object);
```

首先定义了一个变量weakPointer，其次调用objc_initWeak方法来给weakPointer的这个弱引用指针来赋值。

```
id
objc_initWeak(id *location, id newObj)
{
    if (!newObj) {
        *location = nil;
        return nil;
    }

    return storeWeak<false/*old*/, true/*new*/, true/*crash*/>
        (location, (objc_object*)newObj);
}
```

```
storeWeak(id *location, objc_object *newObj)
{
    assert(HaveOld  ||  HaveNew);
    if (!HaveNew) assert(newObj == nil);

    Class previouslyInitializedClass = nil;
    id oldObj;
    SideTable *oldTable;
    SideTable *newTable;

    // Acquire locks for old and new values.
    // Order by lock address to prevent lock ordering problems. 
    // Retry if the old value changes underneath us.
 retry:
    if (HaveOld) {
        oldObj = *location;
        oldTable = &SideTables()[oldObj];
    } else {
        oldTable = nil;
    }
    if (HaveNew) {
        newTable = &SideTables()[newObj];
    } else {
        newTable = nil;
    }

    SideTable::lockTwo<HaveOld, HaveNew>(oldTable, newTable);

    if (HaveOld  &&  *location != oldObj) {
        SideTable::unlockTwo<HaveOld, HaveNew>(oldTable, newTable);
        goto retry;
    }

    // Prevent a deadlock between the weak reference machinery
    // and the +initialize machinery by ensuring that no 
    // weakly-referenced object has an un-+initialized isa.
    if (HaveNew  &&  newObj) {
        Class cls = newObj->getIsa();
        if (cls != previouslyInitializedClass  &&  
            !((objc_class *)cls)->isInitialized()) 
        {
            SideTable::unlockTwo<HaveOld, HaveNew>(oldTable, newTable);
            _class_initialize(_class_getNonMetaClass(cls, (id)newObj));

            // If this class is finished with +initialize then we're good.
            // If this class is still running +initialize on this thread 
            // (i.e. +initialize called storeWeak on an instance of itself)
            // then we may proceed but it will appear initializing and 
            // not yet initialized to the check above.
            // Instead set previouslyInitializedClass to recognize it on retry.
            previouslyInitializedClass = cls;

            goto retry;
        }
    }

    // Clean up old value, if any.
    if (HaveOld) {
        weak_unregister_no_lock(&oldTable->weak_table, oldObj, location);
    }

    // Assign new value, if any.
    if (HaveNew) {
        newObj = (objc_object *)weak_register_no_lock(&newTable->weak_table, 
                                                      (id)newObj, location, 
                                                      CrashIfDeallocating);
        // weak_register_no_lock returns nil if weak store should be rejected

        // Set is-weakly-referenced bit in refcount table.
        if (newObj  &&  !newObj->isTaggedPointer()) {
            newObj->setWeaklyReferenced_nolock();
        }

        // Do not set *location anywhere else. That would introduce a race.
        *location = (id)newObj;
    }
    else {
        // No new value. The storage is not changed.
    }
    
    SideTable::unlockTwo<HaveOld, HaveNew>(oldTable, newTable);

    return (id)newObj;
}
```

```
id 
weak_register_no_lock(weak_table_t *weak_table, id referent_id, 
                      id *referrer_id, bool crashIfDeallocating)
{
    objc_object *referent = (objc_object *)referent_id;
    objc_object **referrer = (objc_object **)referrer_id;

    if (!referent  ||  referent->isTaggedPointer()) return referent_id;

    // ensure that the referenced object is viable
    bool deallocating;
    if (!referent->ISA()->hasCustomRR()) {
        deallocating = referent->rootIsDeallocating();
    }
    else {
        BOOL (*allowsWeakReference)(objc_object *, SEL) = 
            (BOOL(*)(objc_object *, SEL))
            object_getMethodImplementation((id)referent, 
                                           SEL_allowsWeakReference);
        if ((IMP)allowsWeakReference == _objc_msgForward) {
            return nil;
        }
        deallocating =
            ! (*allowsWeakReference)(referent, SEL_allowsWeakReference);
    }

    if (deallocating) {
        if (crashIfDeallocating) {
            _objc_fatal("Cannot form weak reference to instance (%p) of "
                        "class %s. It is possible that this object was "
                        "over-released, or is in the process of deallocation.",
                        (void*)referent, object_getClassName((id)referent));
        } else {
            return nil;
        }
    }

    // now remember it and where it is being stored
    weak_entry_t *entry;
    if ((entry = weak_entry_for_referent(weak_table, referent))) {
        append_referrer(entry, referrer);
    } 
    else {
        weak_entry_t new_entry;
        new_entry.referent = referent;
        new_entry.out_of_line = 0;
        new_entry.inline_referrers[0] = referrer;
        for (size_t i = 1; i < WEAK_INLINE_COUNT; i++) {
            new_entry.inline_referrers[i] = nil;
        }
        
        weak_grow_maybe(weak_table);
        weak_entry_insert(weak_table, &new_entry);
    }

    // Do not set *referrer. objc_storeWeak() requires that the 
    // value not change.

    return referent_id;
}
```

```
static weak_entry_t *
weak_entry_for_referent(weak_table_t *weak_table, objc_object *referent)
{
    assert(referent);

    weak_entry_t *weak_entries = weak_table->weak_entries;

    if (!weak_entries) return nil;

    size_t index = hash_pointer(referent) & weak_table->mask;
    size_t hash_displacement = 0;
    while (weak_table->weak_entries[index].referent != referent) {
        index = (index+1) & weak_table->mask;
        hash_displacement++;
        if (hash_displacement > weak_table->max_hash_displacement) {
            return nil;
        }
    }
    
    return &weak_table->weak_entries[index];
}
```

内部原理与上面相似，当弱引用指针指向这个objcet变量时，首先去SideTables散列表中通过哈希查找，来找到object这个对象的SideTable表，再通过一次哈希查找，利用object对象的地址，在SideTable中的弱引用表中找到其对应的弱引用结构体数组，如果这个数组存在则在里面添加一个之前weakPointer的地址作为弱引用指针指向object，如果没有这个结构体数组，则创建一个数组，将这个指针添加到第0个元素，同时给第2，3，4个元素设置为nil。这样就完成了一个弱引用指针的定义实现过程了。

关于如何清除弱引用指针的，Dealloc方法调用过程已经说的很明白了，过程也与上面一行说的类似，就是在最后调用sidetable_clearDeallocating()方法中将对象对应的弱引用列表找到，将所有弱引用指针置为nil的时候就把相应的弱引用指针擦除了，这样说就一目了然了。


##7、自动释放池

因为现在大家都在使用ARC模式下进行编程，一个很重要的问题也是最容易被大家所忽视的问题就是自动释放池，大部分程序员尤其是刚入行的都只是知道有这么一个东西，但具体是什么，工作的原理是什么，在什么时候使用它都一概不知。所以写一篇文章，记录一下个人对自动释放池的一些理解。

我们新建一个OC项目，在main函数中可以看到这么一串代码：

```
int main(int argc, char * argv[]) {

  @autoreleasepool {

    return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
  }
}
```

编译器会将`@autoreleasepool{}`改写为：

```
//创建一个无类型指针的哨兵对象
void *pool = objc_autoreleasePoolPush();

//执行@autoreleasepool{}中对应{}里所书写的代码
{}中的代码

//释放哨兵对象所分隔区域内所有对象的引用计数
objc_autoreleasePoolPop(pool);
```

由以上可以看出，autoreleasepool工作的原理就是一个压栈，一个出栈，push方法创建哨兵对象作为标记，pop操作作批量引用计数释放的工作。

autoreleasepool是以栈为节点，通过双向链表的形式组合而成的，autoreleasepool是与线程一一对应的。

那么什么是双向链表呢，双向链表的单位是节点，从头结点开始一直到尾节点，每一个节点都会有一个父指针和子指针，父指针指向前一个节点，子指针指向后一个节点，而头节点的父指针指向NULL，尾节点的子指针指向NULL。这样就把栈空间通过双向链表的形式连接起来了。

而autoreleasepoolPage就是双链链表中每一个节点的长度了，可以说，autoreleasepoolPage把栈空间中一些大小的空间分割成一个个单位，称为节点，然后通过双向链表的形式连接起来，这就成了一个整体的autoreleasepool的结构了。

![](https://www.icheesedu.com/images/双向链表.png)

![](https://www.icheesedu.com/images/栈.png)


![](https://www.icheesedu.com/images/AutoreleasepoolPage.png)

在AutoreleasePool中有四个变量分别是：

 - 1、id *next一个id类型的指针，指向的是下一个可存储对象的位置，
 - 2、*AutoreleasepoolPage* const parent;这就是当前page父节点page的地址指针。
 - 3、*AutoreleasepoolPage* child，同理，是子节点表地址的指针。4.pthread_t const thread;这个变量中就记录了线程的情况，所以说自动释放池是与线程一一对应的关系。


执行逻辑：

- 1、调用objc_autoreleasepoolPush()方法，在当前autoreleasepoolPage中的next指针位置创建一个为nil的哨兵对象，随后将next指针的位置指向下一个内存地址。

- 2、第二步就是执行代码了，在自动释放池范围内的代码被执行，给逐个对象调用[object autorelease]方法。其实在autorelease方法内部实现的步骤为：
   - 1.判断next指针是否已经在栈顶了，如果是，则增加一个栈节点到链表上，随后增加一个对象到新的栈节点链表中，如果不是的话则在next指针所指的位置添加一个调用autorelease方法的对象。

- 3、第三步就是执行objc_autoreleasepoolPop()方法,该方法会根据传入的哨兵对象找到对应的内存位置，然后根据哨兵对象的位置给上次push后添加的对象依次发送release消息，然后回退next指针到正确的位置。

以上三个步骤就是在autorelease自动释放池中进行的操作，也是这三个步骤构成了自动释放池。

总结一下

- main函数中的autoreleasepool是在runloop结束的时候调用objc_autoreleasepoolPop的方法的

- 多层嵌套的autoreleasepool其实就是在栈中多次插入哨兵对象
- 而在我们开发的过程中，通过for循环加载一些占用内存较大的对象时可以嵌套使用autoreleasepool，在这些对象使用完毕的时候及时被释放掉，这样就不会造成内存过大或过多浪费的情况啦~


##8、循环引用

- 循环引用分三种：
   - 1.自循环引用
   
   - 2.相互循环引用
   - 3.多循环引用

- 循环引用出现的地方多数是在block，NSTimer中，代理中如果代理对象没有设置为weak也会产生循环引用。

- 破环的方法无非是将一方引用的方式改为弱引用，但在OC中，引用一个对象而不增加其引用计数一共有三种关键字可以实现：1.`weak`，2.`block`，3.`_unsafe_unretained`

  - 第一种weak之前的文章已经详细叙述了其工作原理，如何对对象进行添加弱引用指针以及弱引用指针如何在对象被销毁时进行回收的，这里就不多写了。

  - 第二种__block关键字，其在MRC模式下，使用__block关键字修饰一个对象，不会增加其引用计数，从而可以避免循环引用。但是在ARC中，__block关键字修饰的对象会被强引用，就没法避免循环引用了。这就是__block和__weak的区别

  - 第三种_unsafe_unretained关键字，使用之后确实不会增加引用对象的引用计数，但是当引用对象被释放的时候，会产生悬垂指针，从而发生一些不可预见的错误，所以是不安全的，不可取。

以上就是这三种关键字的区别了。

最常见的循环引用案例——NSTimer

- 在NSTimer被创建的时候，由于系统常驻的runloop对其进行强引用，其又会对当前对象进行强引用，当前对象又会对其进行循环引用，而VC又会对当前对象进行循环引用，这时候就造成了一个环，导致内存泄漏。这时候如果将VC中创建的对象timer的关键字改为weak也无济于事，因为当VC释放对象的时候，timer并不会对系统中持有的定时器对象进行释放，是因为runloop对系统Timer对象还有一个强引用，导致系统中的Timer不会被正常释放，而系统Timer又对当前timer对象有一个强引用，这样就导致了当前对象timer无法被正常的释放了。遇到这种问题，大部分图省事的做法就是手动破环，当VC被回收的时候手动将timer调用废弃方法，回收系统对象。这种做法也可取，但是在VC非常多的时候，而且使用定时器的地方非常多的时候，稍不注意忘记回收timer就会导致内存的泄漏，而且这种方式也是非常耗费大量无用功的。所以我自己封装了一个weakTimer的自定义定时器。

- 将系统Timer实例对象，与VC中timer对象以及VC的关系变了一下，在中间加了一个过渡层对象。将过度对象弱引用VC为target，然后弱引用系统Timer，并且调用target中真正需要实现的方法，每次定时器的回调方法时，过度对象都会去判断这个target是否为nil，如果为nil就将定时器置空，如果对象还在，就会先判断目标方法是否相应，随后调用真正的方法。这种破环的模式即安全，又对调用者来说简单，且不易出错，比较提高效率。

```
#import <Foundation/Foundation.h>

@interface NSTimer (WeakTimer)

+ (NSTimer *)scheduledWeakTimerWithTimeInterval:(NSTimeInterval)interval
                                         target:(id)aTarget
                                       selector:(SEL)aSelector
                                       userInfo:(id)userInfo
                                        repeats:(BOOL)repeats;

@end
```

```
#import "NSTimer+WeakTimer.h"

@interface TimerWeakObject : NSObject
@property (nonatomic, weak) id target;
@property (nonatomic, assign) SEL selector;
@property (nonatomic, weak) NSTimer *timer;

- (void)fire:(NSTimer *)timer;
@end

@implementation TimerWeakObject

- (void)fire:(NSTimer *)timer
{
    if (self.target) {
    
        if ([self.target respondsToSelector:self.selector]) {
            #pragma clang diagnostic push
            #pragma clang diagnostic ignored "-Warc-performSelector-leaks"
            [self.target performSelector:self.selector withObject:timer.userInfo];
            #pragma clang diagnostic pop
        }
    }
    else{
        [self.timer invalidate];
    }
}

@end

@implementation NSTimer (WeakTimer)

+ (NSTimer *)scheduledWeakTimerWithTimeInterval:(NSTimeInterval)interval
                                         target:(id)aTarget
                                       selector:(SEL)aSelector
                                       userInfo:(id)userInfo
                                        repeats:(BOOL)repeats
{
    TimerWeakObject *object = [[TimerWeakObject alloc] init];
    object.target = aTarget;
    object.selector = aSelector;
    object.timer = [NSTimer scheduledTimerWithTimeInterval:interval target:object selector:@selector(fire:) userInfo:userInfo repeats:repeats];
    
    return object.timer;
}

@end
```


