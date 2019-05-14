# 浅谈AtomicInteger实现原理

AtomicInteger位于java.util.concurrent.atomic包下，是对int的封装，提供原子性的访问和更新操作，其原子性操作的实现是基于CAS。

- 1. CAS

  CAS(compare-and-swap)直译即比较并交换，提供原子化的读改写能力，是Java 并发中所谓 lock-free 机制的基础。compareAndSet 传入的为执行方法时获取到的 value 属性值，next 为加 1 后的值，compareAndSet 所做的为调用 Sun 的 UnSafe 的 compareAndSwapInt 方法来完成，此方法为 native 方法，compareAndSwapInt 基于的是 CPU 的 CAS 指令来实现的。所以基于 CAS 的操作可认为是无阻塞的，一个线程的失败或挂起不会引起其它线程也失败或挂起。并且由于 CAS 操作是 CPU 原语，所以性能比较好。
  
  CAS的思想很简单：三个参数，一个当前内存值V、旧的预期值A、即将更新的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false。
可能会有面试官问 CAS 底层是如何实现的，在JAVA中,CAS通过调用C++库实现，由C++库再去调用CPU指令集。不同体系结构中，cpu指令还存在着明显不同。比如，x86 CPU 提供 cmpxchg 指令；而在精简指令集的体系架构中，（如“load and reserve”和“store conditional”）实现的，在大多数处理器上 CAS 都是个非常轻量级的操作，这也是其优势所在。

CAS的缺点有以下几个方面：

- ABA问题

  如果某个线程在CAS操作时发现，内存值和预期值都是A，就能确定期间没有线程对值进行修改吗？答案未必，如果期间发生了 A -> B -> A 的更新，仅仅判断数值是 A，可能导致不合理的修改操作。针对这种情况，Java 提供了 AtomicStampedReference 工具类，通过为引用建立类似版本号（stamp）的方式，来保证 CAS 的正确性。
  
- 循环时间长开销大
  CAS中使用的失败重试机制，隐藏着一个假设，即竞争情况是短暂的。大多数应用场景中，确实大部分重试只会发生一次就获得了成功。但是总有意外情况，所以在有需要的时候，还是要考虑限制自旋的次数，以免过度消耗 CPU。

- 3.只能保证一个共享变量的原子操作

  当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。


## 2.AtomicInteger原理浅析

```
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
}
```
![image.png](https://upload-images.jianshu.io/upload_images/325120-8184328d9f15ee99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从 AtomicInteger 的内部属性可以看出，它依赖于Unsafe 提供的一些底层能力，进行底层操作；如根据valueOffset代表的该变量值在内存中的偏移地址，从而获取数据的。
变量value用volatile修饰，保证了多线程之间的内存可见性。

下面以getAndIncrement为例，说明其原子操作过程

```
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

//unsafe.getAndAddInt
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

![image.png](https://upload-images.jianshu.io/upload_images/325120-fedd5090f7cbaa58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/325120-f118f16139a77971.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 假设线程1和线程2通过getIntVolatile拿到value的值都为1，线程1被挂起，线程2继续执行
- 线程2在compareAndSwapInt操作中由于预期值和内存值都为1，因此成功将内存值更新为2
- 线程1继续执行，在compareAndSwapInt操作中，预期值是1，而当前的内存值为2，CAS操作失败，什么都不做，返回false
- 线程1重新通过getIntVolatile拿到最新的value为2，再进行一次compareAndSwapInt操作，这次操作成功，内存值更新为3

## 3、原子操作的实现原理
- Java中的CAS操作正是利用了处理器提供的CMPXCHG指令实现的。自旋CAS实现的基本思路就是循环进行CAS操作直到操作成功为止。
- 在CAS中有三个操作数：分别是内存地址（在Java中可以简单理解为变量的内存地址，用V表示）、旧的预期值（用A表示）和新值（用B表示）。CAS指令执行时，当且仅当V符合旧的预期值A时，处理器才会用新值B更新V的值，否则他就不执行更新，但无论是否更新了V的值，都会返回V的旧值。



