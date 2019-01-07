##JVM之内存分配与回收策略 - 内存分配-逃逸分析与栈上分配

- **Java体系中所提倡的自动内存管理最终可以归结为自动化地解决两个问题：**

   - 给对象分配内存
   
   - 以及回收分配给对象的内存
   
- **GC的区别**

   - 新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。
   
   - 老年代GC（Major GC/Full GC）：指发生在老年代的GC，出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器策略里就有直接进行Major GC的策略选择过程）。Major GC的速度一般会比Minor GC慢10倍以上。
- **对象优先在Eden分配**

   - 大多数情况下，对象在新生代Eden区中分配。当Eden去没有足够空间进行分配时，虚拟机将发起一次Minor GC。

    - 目前主流的垃圾收集器都会采用分代回收算法，因此需要将堆内存分为新生代和老年代。

    - 在新生代中为了防止内存碎片问题，因此垃圾收集器一般都选用“复制”算法。因此，堆内存的新生代被进一步分为：Eden区＋Survior1区＋Survior2区。

    - 每次创建对象时，首先会在Eden区中分配。 
    - 若Eden区已满，则在Survior1区中分配。 
    - 若Eden区＋Survior1区剩余内存太少，导致对象无法放入该区域时，就会启用“分配担保”，将当前Eden区＋Survior1区中的对象转移到老年代中，然后再将新对象存入Eden区。 

- **大对象直接进入老年代**

   - 大对象是指需要大量连续内存空间的Java对象，最典型的大对象是很长的字符串以及数组。经常出现大对象容易导致内存还有不少空间时就提前触发垃圾收集以获取足够的连续空间来安置它们。
   -  -XX:PretenureSizeThreshold=6M 指定大对象的大小

- **长期存活的对象将进入老年代**

   - 虚拟机给每个对象定义了一个对象年龄计算器。如果对象在Eden出生并经过一次Minor GC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并且对象年龄设为1。对象在Survivor区中每“熬过”一次Minor GC，年龄就增加一岁，当它的年龄增加得到一定程度（默认为15岁），就会被晋升到老年代中。对象晋升老奶奶年代的年龄阈值，可以通过参数配置 -XX:MaxTernuringThreshold设置。

- **动态对象年龄判定**

   - 为了能更好地适应不同程序的内存状况，虚拟机并不是永远地要求对象的年龄必须达到了MaxTenuringTreshold才能晋升老年代，如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一般，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到MaxtenuringThreshold中要求的年龄。

- **空间分配担保**

   - 在发生Minor GC之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那么Minor GC可以确保安全的。如果不成立，则虚拟机会查看HandlePromotionFailure设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者HandlePromotionFailure设置不允许冒险，那这时也要改为进行一次Full GC。
   
   - 上面提到了Minor GC依然会有风险，是因为新生代采用复制收集算法，假如大量对象在Minor GC后仍然存活（最极端情况为内存回收后新生代中所有对象均存活），而Survivor空间是比较小的，这时就需要老年代进行分配担保，把Survivor无法容纳的对象放到老年代。老年代要进行空间分配担保，前提是老年代得有足够空间来容纳这些对象，但一共有多少对象在内存回收后存活下来是不可预知的，因此只好取之前每次垃圾回收后晋升到老年代的对象大小的平均值作为参考。使用这个平均值与老年代剩余空间进行比较，来决定是否进行Full GC来让老年代腾出更多空间。

   - 取平均值仍然是一种概率性的事件，如果某次Minor GC后存活对象陡增，远高于平均值的话，必然导致担保失败，如果出现了分配担保失败，就只能在失败后重新发起一次Full GC。虽然存在发生这种情况的概率，但大部分时候都是能够成功分配担保的，这样就避免了过于频繁执行Full GC。

   - 注意： 
      - 1、分配担保是老年代为新生代作担保。
      
      - 2、新生代中使用“复制”算法实现垃圾回收，老年代中使用“标记-清除”或“标记-整理”算法实现垃圾回收，只有使用“复制”算法的区域才需要分配担保，因此新生代需要分配担保，而老年代不需要分配担保。
      
   **Android堆内存分代回收模型**

     ![](http://odoy84a4u.bkt.clouddn.com/image/android/memory_mode_generation.png)


##内存分配-逃逸分析与栈上分配
- 逃逸分析：

   -  逃逸分析是一种为其他优化手段提供依据的分析技术，其基本行为是分析对象动态作用域：当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他方法中，称为方法逃逸；也有可能被其外部线程访问到，如复制给类变量或者可以在其他线程中访问的实例变量，称为线程逃逸。逃逸分析（个人理解）：就是方法内的对象，可以被方法外所访问。
   - 如果一个对象不会逃逸到方法或者线程之外，则可以对这个对象进行一些高效的优化：

   - 栈上分配Stack Allocation：如果一个对象不会逃逸到方法之外，那么可以让这个对象在栈上分配内存，以提高执行效率，对象所占内存会随着栈帧出栈而销毁。在一般应用中，无逃逸的局部变量对象所占的比例较大，如果能使用栈上分配，那么大量的对象就会随着方法的结束而自动销毁，GC压力减小很多。栈上分配：就是把没发生逃逸的对象，在栈分配空间。（一般对象分配空间是在堆）逃逸

    - 二者联系：jvm根据对象是否发生逃逸，会分配到不同（堆或栈）的存储空间。

   - 同步消除SynchronizationElimination：线程同步是一个相对耗时的过程，如果逃逸分析能够确定一个变量不会逃逸出线程，无法被其他线程访问，那么该变量的读写不存在竞争关系，即可以消除掉对这个变量的同步措施

   - 如果对象发生逃逸，那会分配到堆中。（因为对象发生了逃逸，就代表这个对象可以被外部访问，换句话说，就是可以共享，能共享数据的，无非就是堆或方法区，这里就是堆。）

   - 如果对象没发生逃逸，那会分配到栈中。（因为对象没发生逃逸，那就代表这个对象不能被外部访问，换句话说，就是不可共享，这里就是栈。）

   - 那我们再想深一层，为什么会有逃逸分析，有栈上分配这些东西？

   - 当然是为了主体的考虑，主体就是jvm，或者直接说为了GC考虑也不为过。大家想想，GC主要回收的对象是堆和方法区。GC不会对栈、程序计数器这些进行回收的，因为没东西可以回收。

   - 说回来，如果方法逃逸，那么对象就会分配在堆中，这个时候，GC就要工作了。如果没发生方法逃逸，那么对象就分配在栈中，当方法结束后，资源就自动释放了，GC压根不用操心。所以哈，方法逃逸这东西，主要也是为GC打工的，或者说为GC服务吧！说到这里，可能有人会问，那方法逃逸和性能还是没关系哈？emmm!!!其实有，想深一层，GC不运行的时候，程序的性能肯定会好点，不会占用程序运行的时间。虽然GC清扫垃圾的速度很快，但是当一个程序足够大的时候，对象就自然多了，垃圾也自然多了，这个时候GC就忙了。

   - 能在方法内创建对象，就不要在方法外创建对象。毕竟这是为了GC好，也是为了提高性能。


- 标量替换：

   - 标量：指的是一个数据已经无法再分解成更小的数据来表示了，Java虚拟机的原始数据类型（int,float等数值类型以及reference类型）都不能再进行进一步的分解
   - 聚合量：相对于标量，如果一个数据可继续分解，则可以称作聚合量，Java对象是典型的聚合量。
   - 如果把一个Java对象拆散，根据程序访问的情况，将其使用到的成员变量恢复原始类型来访问，这过程成为标量替换
   - 如果逃逸分析可以确定一个对象不会被外部访问，且这个对象可以被拆散，那程序真正执行的时候，可以不创建这个对象，而是直接创建它的成员变量来替换这个对象。将对象拆分后，可以在栈上分配内存


- 测试实例：
  
  ```
/**
* Description：新建10000000个对象，计算执行时间，再配置不同JVM参数
* 比较执行结果
*    -XX:-DoEscapeAnalysis  关闭逃逸分析
*    -XX:-EliminateAllocations 关闭标量替换
*    -XX:-UseTLAB 关闭线程本地内存
*    -XX:-PrintGC 打印GC信息
*/
public class JVMTest1 {
    
    class User{
        int id;
        String name;
        
        User(int id,String name){
            this.id = id;
            this.name = name;
            
        }
    }
    
    void alloc(int i){
        new User(i,"name"+i);
    }

    public static void main(String[] args) {
        
            JVMTest1 t = new JVMTest1();
            long s1 = System.currentTimeMillis();
            for(int i = 0;i<10000000;i++){
                t.alloc(i);
            }
            long s2 = System.currentTimeMillis();
            System.out.println(s2-s1);
       }
  }
  ```
run As --> run configuration --> Argument --> VM argument ：填入如下配置：

    ```
    -XX:-DoEscapeAnalysis -XX:-EliminateAllocations -XX:-UseTLAB -XX:+PrintGC
    ```
结果分析：

   - a.无逃逸分析、无栈上分配、不使用线程本地内存：
   
     ```
     -XX:-DoEscapeAnalysis -XX:-EliminateAllocations -XX:-UseTLAB -XX:+PrintGC
     ```
     
     ```
    [GC (Allocation Failure)  49152K->688K(188416K), 0.0010012 secs]
    [GC (Allocation Failure)  49840K->728K(188416K), 0.0009848 secs]
    [GC (Allocation Failure)  49880K->640K(188416K), 0.0007432 secs]
    [GC (Allocation Failure)  49792K->672K(237568K), 0.0008412 secs]
    [GC (Allocation Failure)  98976K->640K(237568K), 0.0012708 secs]
    [GC (Allocation Failure)  98944K->656K(328704K), 0.0008696 secs]
    [GC (Allocation Failure)  197264K->624K(328704K), 0.0017397 secs]
    [GC (Allocation Failure)  197232K->624K(320512K), 0.0003312 secs]
    791
     ```
  - b.使用线程本地内存，无需在eden区分配内存时加锁，效率变高

      ```
      -XX:-DoEscapeAnalysis -XX:-EliminateAllocations -XX:+UseTLAB -XX:+PrintGC
      ```
控制台输出：
  
     ```
    [GC (Allocation Failure)  49760K->640K(188416K), 0.0007129 secs]
    [GC (Allocation Failure)  49792K->624K(237568K), 0.0008062 secs]
    [GC (Allocation Failure)  98928K->608K(237568K), 0.0014966 secs]
    [GC (Allocation Failure)  98912K->728K(328704K), 0.0008608 secs]
    [GC (Allocation Failure)  197336K->588K(328704K), 0.0016310 secs]
    [GC (Allocation Failure)  197196K->620K(525312K), 0.0003275 secs]
    528
     ```
  - c.开启逃逸分析、使用标量替换、使用线程本地内存、效率变高

        ```
        -XX:+DoEscapeAnalysis -XX:+EliminateAllocations -XX:+UseTLAB -XX:+PrintGC
        ```

     控制台输出：


     ```
     [GC (Allocation Failure)  49152K->688K(188416K), 0.0010576 secs]
[GC (Allocation Failure)  49840K->640K(188416K), 0.0009443 secs]
[GC (Allocation Failure)  49792K->640K(188416K), 0.0007502 secs]
[GC (Allocation Failure)  49792K->696K(237568K), 0.0008981 secs]
[GC (Allocation Failure)  99000K->656K(237568K), 0.0011229 secs]
[GC (Allocation Failure)  98960K->608K(328704K), 0.0010558 secs]
[GC (Allocation Failure)  197216K->644K(328704K), 0.0015396 secs]
486
     ```
问题分析：开启逃逸分析存在开销，有时效率不如未开逃逸分析时的效率高


