## 1、ThreadLocal是什么
ThreadLocal是一个本地线程副本变量工具类。主要用于将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，特别适用于各个线程依赖不通的变量值完成操作的场景。

**从数据结构入手**

下图为ThreadLocal的内部结构图
![](https://upload-images.jianshu.io/upload_images/7432604-ad2ff581127ba8cc.jpg)


![](https://upload-images.jianshu.io/upload_images/2184951-9611b7b31c9b2e20.png)

从上面的结构图，我们已经窥见ThreadLocal的核心机制：

 - 每个Thread线程内部都有一个Map。
 - Map里面存储线程本地对象（key）和线程的变量副本（value），但是，Thread内部的Map是由ThreadLocal维护的，由ThreadLocal负责向map获取和设置线程的变量值。

 - 所以对于不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离，互不干扰。


Thread线程内部的Map在类中描述如下：

```
public class Thread implements Runnable {
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

## 2、深入解析ThreadLocal
ThreadLocal类提供如下几个核心方法：

- public T get()
- public void set(T value)
- public void remove()


get()方法用于获取当前线程的副本变量值。
set()方法用于保存当前线程的副本变量值。
initialValue()为当前线程初始副本变量值。
remove()方法移除当前前程的副本变量值。

**get()方法**

```
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
 
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}


ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}

protected T initialValue() {
    return null;
}
```
步骤：

- 1.获取当前线程的ThreadLocalMap对象threadLocals
- 2.从map中获取线程存储的K-V Entry节点。
- 3.从Entry节点获取存储的Value副本值返回。
- 4.map为空的话返回初始值null，即线程变量副本为null，在使用时需要注意判断NullPointerException。

**set()方法**

```
/**
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of
 *        this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```
步骤：

- 1.获取当前线程的成员变量map
- 2.map非空，则重新将ThreadLocal和新的value副本放入到map中。
- 3.map空，则对线程的成员变量ThreadLocalMap进行初始化创建，并将ThreadLocal和value副本放入map中。

**remove()方法**

```
/**
 * Removes the current thread's value for this thread-local
 * variable.  If this thread-local variable is subsequently
 * {@linkplain #get read} by the current thread, its value will be
 * reinitialized by invoking its {@link #initialValue} method,
 * unless its value is {@linkplain #set set} by the current thread
 * in the interim.  This may result in multiple invocations of the
 * <tt>initialValue</tt> method in the current thread.
 *
 * @since 1.5
 */
public void remove() {
 ThreadLocalMap m = getMap(Thread.currentThread());
 if (m != null)
     m.remove(this);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```
remove方法比较简单，不做赘述。

**ThreadLocalMap**

ThreadLocalMap是ThreadLocal的内部类，没有实现Map接口，用独立的方式实现了Map的功能，其内部的Entry也独立实现。

![](https://upload-images.jianshu.io/upload_images/7432604-5bbe090d46789084.png)

在ThreadLocalMap中，也是用Entry来保存K-V结构数据的。但是Entry中key只能是ThreadLocal对象，这点被Entry的构造方法已经限定死了。

```
static class Entry extends WeakReference<ThreadLocal> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal k, Object v) {
        super(k);
        value = v;
    }
}
```
Entry继承自WeakReference（弱引用，生命周期只能存活到下次GC前），但只有Key是弱引用类型的，Value并非弱引用。

ThreadLocalMap的成员变量：

```
static class ThreadLocalMap {
    /**
     * The initial capacity -- MUST be a power of two.
     */
    private static final int INITIAL_CAPACITY = 16;

    /**
     * The table, resized as necessary.
     * table.length MUST always be a power of two.
     */
    private Entry[] table;

    /**
     * The number of entries in the table.
     */
    private int size = 0;

    /**
     * The next size value at which to resize.
     */
    private int threshold; // Default to 0
}
```

## 3、Hash冲突怎么解决

和HashMap的最大的不同在于，ThreadLocalMap结构非常简单，没有next引用，也就是说ThreadLocalMap中解决Hash冲突的方式并非链表的方式，而是采用线性探测的方式，所谓线性探测，就是根据初始key的hashcode值确定元素在table数组中的位置，如果发现这个位置上已经有其他key值的元素被占用，则利用固定的算法寻找一定步长的下个位置，依次判断，直至找到能够存放的位置。

ThreadLocalMap解决Hash冲突的方式就是简单的步长加1或减1，寻找下一个相邻的位置。

先看看ThreadLoalMap中插入一个key-value的实现

```
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

```
/**
 * Increment i modulo len.
 */
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}

/**
 * Decrement i modulo len.
 */
private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

每个ThreadLocal对象都有一个hash值threadLocalHashCode，每初始化一个ThreadLocal对象，hash值就增加一个固定的大小0x61c88647。
在插入过程中，根据ThreadLocal对象的hash值，定位到table中的位置i，过程如下：

- 1、如果当前位置是空的，那么正好，就初始化一个Entry对象放在位置i上；
- 2、不巧，位置i已经有Entry对象了，如果这个Entry对象的key正好是即将设置的key，那么重新设置Entry中的value；
- 3、很不巧，位置i的Entry对象，和即将设置的key没关系，那么只能找下一个空位置；
这样的话，在get的时候，也会根据ThreadLocal对象的hash值，定位到table中的位置，然后判断该位置Entry对象中的key是否和get的key一致，如果不一致，就判断下一个位置
- 可以发现，set和get如果冲突严重的话，效率很低，因为ThreadLoalMap是Thread的一个属性，所以即使在自己的代码中控制了设置的元素个数，但还是不能控制其它代码的行为。

- 显然ThreadLocalMap采用线性探测的方式解决Hash冲突的效率很低，如果有大量不同的ThreadLocal对象放入map中时发送冲突，或者发生二次冲突，则效率很低。
- 所以这里引出的良好建议是：每个线程只存一个变量，这样的话所有的线程存放到map中的Key都是相同的ThreadLocal，如果一个线程要保存多个变量，就需要创建多个ThreadLocal，多个ThreadLocal放入Map中时会极大的增加Hash冲突的可能。



## 4、ThreadLocal可能导致内存泄漏，为什么？

```
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */p
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

- ThreadLocalMap中的Entry弱引用（key）到的是 threadLocal 对象，此处的弱引用意图是，当外界不再持有对 threadLocal 对象的强引用的时候，threadLocal 对象可以被GC。此处Entry对 threadLocal 的弱引用不会引起内存泄露。

	![](https://images.zsxq.com/FvCsNCGvwWX_7Qk5j4BBLHSs-GVT?e=1906272000&token=kIxbL07-8jAj8w1n4s9zv64FuZZNEATmlU_Vm6zD:eNqxt-GJw9aH3MU51an7kZaNOkA=)
	
**如何避免内存泄露**

既然已经发现有内存泄露的隐患，自然有应对的策略，在调用ThreadLocal的get()、set()可能会清除ThreadLocalMap中key为null的Entry对象，这样对应的value就没有GC Roots可达了，下次GC的时候就可以被回收，当然如果调用remove方法，肯定会删除对应的Entry对象。
如果使用ThreadLocal的set方法之后，没有显示的调用remove方法，就有可能发生内存泄露，所以养成良好的编程习惯十分重要，使用完ThreadLocal之后，记得调用remove方法。

```
ThreadLocal<Session> threadLocal = new ThreadLocal<Session>();
try {
    threadLocal.set(new Session(1, "Misout的博客"));
    // 其它业务逻辑
} finally {
    threadLocal.remove();
}
```

## 5、ThreadLocal与线程同步机制的比较

线程同步机制通过对象的锁机制保证同一时间只有一个线程去访问变量，该变量时多个线程共享的。ThreadLocal则为每一个线程提供了一个变量副本，从而隔离了多个线程访问数据的冲突，ThreadLocal提供了线程安全的对象封装，在编写多线程代码时，可以把不安全的代码封装进ThreadLocal。概括的说，对于多线程资源共享的问题，线程同步机制采取了时间换空间的方式，访问串行化，对象共享化；而ThreadLocal采取了空间换时间的方式，访问并行化，对象独享化。

## 6、ThreadLocal实战使用

![](https://images.zsxq.com/FpJPgZR25nWEwZrYm8hpuEAqLrXO?e=1906272000&token=kIxbL07-8jAj8w1n4s9zv64FuZZNEATmlU_Vm6zD:BWOuDBEiVEBBFv2VdjAetEn-C7I=)

## 6、总结

- 每个ThreadLocal只能保存一个变量副本，如果想要上线一个线程能够保存多个副本以上，就需要创建多个ThreadLocal。
- ThreadLocal内部的ThreadLocalMap键为弱引用，会有内存泄漏的风险。
- 适用于无状态，副本变量独立后不影响业务逻辑的高并发场景。如果如果业务逻辑强依赖于副本变量，则不适合用ThreadLocal解决，需要另寻解决方案。

## 7、推荐阅读

https://mp.weixin.qq.com/s/IXYk-6PmsgAkBnTlcWGGpQ

https://www.jianshu.com/p/377bb840802f


![](https://upload-images.jianshu.io/upload_images/325120-00fcb7f4184230cd.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

详情介绍:

[更新主题详情](https://www.jianshu.com/p/e56fcbc75b56)

420天以来，Java架构更新了 888个主题，已经有156+位同学加入。微信扫码关注java架构，获取Java面试题和架构师相关题目和视频。上述相关面试题答案，尽在Java架构中。

