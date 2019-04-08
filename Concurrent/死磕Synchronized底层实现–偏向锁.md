
 
### 偏向锁入口
 
 目前网上的很多文章，关于偏向锁源码入口都找错地方了，导致我之前对于偏向锁的很多逻辑一直想不通，走了很多弯路。
 
 `synchronized`分为`synchronized`代码块和`synchronized`方法，其底层获取锁的逻辑都是一样的，本文讲解的是`synchronized`代码块的实现。上篇文章也说过，`synchronized`代码块是由`monitorenter`和`monitorexit`两个指令实现的。
 
 关于HotSpot虚拟机中获取锁的入口，网上很多文章要么给出的方法入口为
 [interpreterRuntime.cpp#monitorenter](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/share/vm/interpreter/interpreterRuntime.cpp#l608)，要么给出的入口为[bytecodeInterpreter.cpp#1816](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/share/vm/interpreter/bytecodeInterpreter.cpp#l1816)。包括占小狼的这篇[文章](https://www.jianshu.com/p/c5058b6fe8e5)关于锁入口的位置说法也是有问题的（当然文章还是很好的，在我刚开始研究`synchronized`的时候，小狼哥的这篇文章给了我很多帮助）。
 
 要找锁的入口，肯定是要在源码中找到对`monitorenter`指令解析的地方。在HotSpot的中有两处地方对`monitorenter`指令进行解析：一个是在[bytecodeInterpreter.cpp#1816](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/share/vm/interpreter/bytecodeInterpreter.cpp#l1816) ，另一个是在[templateTable_x86_64.cpp#3667](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/cpu/x86/vm/templateTable_x86_64.cpp#l3667)。
 
 前者是JVM中的字节码解释器(`bytecodeInterpreter`)，用C++实现了每条JVM指令（如`monitorenter`、`invokevirtual`等），其优点是实现相对简单且容易理解，缺点是执行慢。后者是模板解释器(`templateInterpreter`)，其对每个指令都写了一段对应的汇编代码，启动时将每个指令与对应汇编代码入口绑定，可以说是效率做到了极致。模板解释器的实现可以看这篇[文章](https://zhuanlan.zhihu.com/p/33886967)，**在研究的过程中也请教过文章作者‘汪先生’一些问题，这里感谢一下。**
 
 在HotSpot中，只用到了模板解释器，字节码解释器根本就没用到，R大的[读书笔记](https://book.douban.com/annotation/31407691/)中说的很清楚了，大家可以看看，这里不再赘述。
 
 所以`montorenter`的解析入口在模板解释器中，其代码位于[templateTable_x86_64.cpp#3667](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/cpu/x86/vm/templateTable_x86_64.cpp#l3667)。通过调用路径：`templateTable_x86_64#monitorenter`-`interp_masm_x86_64#lock_object`进入到偏向锁入口`macroAssembler_x86#biased_locking_enter`，在这里大家可以看到会生成对应的汇编代码。需要注意的是，不是说每次解析`monitorenter`指令都会调用`biased_locking_enter`,而是只会在JVM启动的时候调用该方法生成汇编代码，之后对指令的解析是通过直接执行汇编代码。
 
 其实`bytecodeInterpreter`的逻辑和`templateInterpreter`的逻辑是大同小异的，因为`templateInterpreter`中都是汇编代码，比较晦涩，所以看`bytecodeInterpreter`的实现会便于理解一点。但这里有个坑，在jdk8u之前，`bytecodeInterpreter`并没有实现偏向锁的逻辑。我之前看的JDK8-87ee5ee27509这个版本就没有实现偏向锁的逻辑，导致我看了很久都没看懂。在这个[commit](https://github.com/sourcemirror/jdk-9-hotspot/commit/0148b64cabaad1ac686a74e453531d453c2e3a16?diff=unified)中对`bytecodeInterpreter`加入了偏向锁的支持，我大致了看了下和`templateInterpreter`对比**除了[栈结构](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/cpu/x86/vm/frame_x86.hpp#l36)不同外，其他逻辑大致相同**，所以下文**就按bytecodeInterpreter中的代码对偏向锁逻辑进行讲解**。`templateInterpreter`的汇编代码讲解可以看这篇[文章](https://zhuanlan.zhihu.com/p/34662715)，其实汇编源码中都有英文注释，了解了汇编几个基本指令的作用再结合注释理解起来也不是很难。
 
### 偏向锁获取流程
 下面开始偏向锁获取流程分析，代码在[bytecodeInterpreter.cpp#1816](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/share/vm/interpreter/bytecodeInterpreter.cpp#l1816)。**注意本文代码都有所删减**。
 
 ```c++
 CASE(_monitorenter): {
   // lockee 就是锁对象
   oop lockee = STACK_OBJECT(-1);
   // derefing's lockee ought to provoke implicit null check
   CHECK_NULL(lockee);
   // code 1：找到一个空闲的Lock Record
   BasicObjectLock* limit = istate-monitor_base();
   BasicObjectLock* most_recent = (BasicObjectLock*) istate-stack_base();
   BasicObjectLock* entry = NULL;
   while (most_recent != limit ) {
     if (most_recent-obj() == NULL) entry = most_recent;
     else if (most_recent-obj() == lockee) break;
     most_recent++;
   }
   //entry不为null，代表还有空闲的Lock Record
   if (entry != NULL) {
     // code 2：将Lock Record的obj指针指向锁对象
     entry-set_obj(lockee);
     int success = false;
     uintptr_t epoch_mask_in_place = (uintptr_t)markOopDesc::epoch_mask_in_place;
 	// markoop即对象头的mark word
     markOop mark = lockee-mark();
     intptr_t hash = (intptr_t) markOopDesc::no_hash;
     // code 3：如果锁对象的mark word的状态是偏向模式
     if (mark-has_bias_pattern()) {
       uintptr_t thread_ident;
       uintptr_t anticipated_bias_locking_value;
       thread_ident = (uintptr_t)istate-thread();
      // code 4：这里有几步操作，下文分析
       anticipated_bias_locking_value =
         (((uintptr_t)lockee-klass()-prototype_header() | thread_ident) ^ (uintptr_t)mark) &
         ~((uintptr_t) markOopDesc::age_mask_in_place);
 	 // code 5：如果偏向的线程是自己且epoch等于class的epoch
       if  (anticipated_bias_locking_value == 0) {
         // already biased towards this thread, nothing to do
         if (PrintBiasedLockingStatistics) {
           (* BiasedLocking::biased_lock_entry_count_addr())++;
         }
         success = true;
       }
        // code 6：如果偏向模式关闭，则尝试撤销偏向锁
       else if ((anticipated_bias_locking_value & markOopDesc::biased_lock_mask_in_place) != 0) {
         markOop header = lockee-klass()-prototype_header();
         if (hash != markOopDesc::no_hash) {
           header = header-copy_set_hash(hash);
         }
         // 利用CAS操作将mark word替换为class中的mark word
         if (Atomic::cmpxchg_ptr(header, lockee-mark_addr(), mark) == mark) {
           if (PrintBiasedLockingStatistics)
             (*BiasedLocking::revoked_lock_entry_count_addr())++;
         }
       }
          // code 7：如果epoch不等于class中的epoch，则尝试重偏向
       else if ((anticipated_bias_locking_value & epoch_mask_in_place) !=0) {
         // 构造一个偏向当前线程的mark word
         markOop new_header = (markOop) ( (intptr_t) lockee-klass()-prototype_header() | thread_ident);
         if (hash != markOopDesc::no_hash) {
           new_header = new_header-copy_set_hash(hash);
         }
         // CAS替换对象头的mark word  
         if (Atomic::cmpxchg_ptr((void*)new_header, lockee-mark_addr(), mark) == mark) {
           if (PrintBiasedLockingStatistics)
             (* BiasedLocking::rebiased_lock_entry_count_addr())++;
         }
         else {
           // 重偏向失败，代表存在多线程竞争，则调用monitorenter方法进行锁升级
           CALL_VM(InterpreterRuntime::monitorenter(THREAD, entry), handle_exception);
         }
         success = true;
       }
       else {
          // 走到这里说明当前要么偏向别的线程，要么是匿名偏向（即没有偏向任何线程）
        	// code 8：下面构建一个匿名偏向的mark word，尝试用CAS指令替换掉锁对象的mark word
         markOop header = (markOop) ((uintptr_t) mark & ((uintptr_t)markOopDesc::biased_lock_mask_in_place |(uintptr_t)markOopDesc::age_mask_in_place |epoch_mask_in_place));
         if (hash != markOopDesc::no_hash) {
           header = header-copy_set_hash(hash);
         }
         markOop new_header = (markOop) ((uintptr_t) header | thread_ident);
         // debugging hint
         DEBUG_ONLY(entry-lock()-set_displaced_header((markOop) (uintptr_t) 0xdeaddead);)
         if (Atomic::cmpxchg_ptr((void*)new_header, lockee-mark_addr(), header) == header) {
            // CAS修改成功
           if (PrintBiasedLockingStatistics)
             (* BiasedLocking::anonymously_biased_lock_entry_count_addr())++;
         }
         else {
           // 如果修改失败说明存在多线程竞争，所以进入monitorenter方法
           CALL_VM(InterpreterRuntime::monitorenter(THREAD, entry), handle_exception);
         }
         success = true;
       }
     }
 
     // 如果偏向线程不是当前线程或没有开启偏向模式等原因都会导致success==false
     if (!success) {
       // 轻量级锁的逻辑
       //code 9: 构造一个无锁状态的Displaced Mark Word，并将Lock Record的lock指向它
       markOop displaced = lockee-mark()-set_unlocked();
       entry-lock()-set_displaced_header(displaced);
       //如果指定了-XX:+UseHeavyMonitors，则call_vm=true，代表禁用偏向锁和轻量级锁
       bool call_vm = UseHeavyMonitors;
       // 利用CAS将对象头的mark word替换为指向Lock Record的指针
       if (call_vm || Atomic::cmpxchg_ptr(entry, lockee-mark_addr(), displaced) != displaced) {
         // 判断是不是锁重入
         if (!call_vm && THREAD-is_lock_owned((address) displaced-clear_lock_bits())) {		//code 10: 如果是锁重入，则直接将Displaced Mark Word设置为null
           entry-lock()-set_displaced_header(NULL);
         } else {
           CALL_VM(InterpreterRuntime::monitorenter(THREAD, entry), handle_exception);
         }
       }
     }
     UPDATE_PC_AND_TOS_AND_CONTINUE(1, -1);
   } else {
     // lock record不够，重新执行
     istate-set_msg(more_monitors);
     UPDATE_PC_AND_RETURN(0); // Re-execute
   }
 }
 ```
 
 再回顾下对象头中mark word的格式：![image](https://camo.githubusercontent.com/ba0c739510c9092e06a37903670441072239d7c7/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31312f32382f313637353964643162306239363236383f773d37323026683d32353026663d6a70656726733d3337323831)
 
 JVM中的每个类也有一个类似mark word的prototype_header，用来标记该class的epoch和偏向开关等信息。上面的代码中`lockee-klass()-prototype_header()`即获取class的prototype_header。
 
 `code 1`，从当前线程的栈中找到一个空闲的`Lock Record`（**即代码中的BasicObjectLock，下文都用Lock Record代指**），判断`Lock Record`是否空闲的依据是其obj字段 是否为null。注意这里是按内存地址从低往高找到最后一个可用的`Lock Record`，换而言之，就是找到内存地址最高的可用`Lock Record`。
 
 `code 2`，获取到`Lock Record`后，首先要做的就是为其obj字段赋值。
 
 `code 3`，判断锁对象的`mark word`是否是偏向模式，即低3位是否为101。
 
 `code 4`，这里有几步位运算的操作` anticipated_bias_locking_value = (((uintptr_t)lockee-klass()-prototype_header() | thread_ident) ^ (uintptr_t)mark) & ​ ~((uintptr_t) markOopDesc::age_mask_in_place);` 这个位运算可以分为3个部分。
 
 第一部分`((uintptr_t)lockee-klass()-prototype_header() | thread_ident)` 将当前线程id和类的prototype_header相或，这样得到的值为（当前线程id + prototype_header中的（epoch + 分代年龄 + 偏向锁标志 + 锁标志位）），注意prototype_header的分代年龄那4个字节为0
 
 第二部分 `^ (uintptr_t)mark` 将上面计算得到的结果与锁对象的markOop进行异或，相等的位全部被置为0，只剩下不相等的位。
 
 第三部分 ` & ~((uintptr_t) markOopDesc::age_mask_in_place)` markOopDesc::age_mask_in_place为...0001111000,取反后，变成了...1110000111,除了分代年龄那4位，其他位全为1；将取反后的结果再与上面的结果相与，将上面异或得到的结果中分代年龄给忽略掉。
 
 `code 5`，`anticipated_bias_locking_value==0`代表偏向的线程是当前线程且`mark word`的epoch等于class的epoch，这种情况下什么都不用做。
 
 `code 6`，`(anticipated_bias_locking_value & markOopDesc::biased_lock_mask_in_place) != 0`代表class的prototype_header或对象的`mark word`中偏向模式是关闭的，又因为能走到这已经通过了`mark-has_bias_pattern()`判断，即对象的`mark word`中偏向模式是开启的，那也就是说class的prototype_header不是偏向模式。
 
 然后利用CAS指令`Atomic::cmpxchg_ptr(header, lockee-mark_addr(), mark) == mark`撤销偏向锁，我们知道CAS会有几个参数，1是预期的原值，2是预期修改后的值 ，3是要修改的对象，与之对应，cmpxchg_ptr方法第一个参数是预期修改后的值，第2个参数是修改的对象，第3个参数是预期原值，方法返回实际原值，如果等于预期原值则说明修改成功。
 
 code 7，如果epoch已过期，则需要重偏向，利用CAS指令将锁对象的`mark word`替换为一个偏向当前线程且epoch为类的epoch的新的`mark word`。
 
 code 8，CAS将偏向线程改为当前线程，如果当前是匿名偏向则能修改成功，否则进入锁升级的逻辑。
 
 code 9，这一步已经是轻量级锁的逻辑了。从上图的`mark word`的格式可以看到，轻量级锁中`mark word`存的是指向`Lock Record`的指针。这里构造一个无锁状态的`mark word`，然后存储到`Lock Record`（`Lock Record`的格式可以看第一篇文章）。设置`mark word`是无锁状态的原因是：轻量级锁解锁时是将对象头的`mark word`设置为`Lock Record`中的`Displaced Mark Word`，所以创建时设置为无锁状态，解锁时直接用CAS替换就好了。
 
 code 10， 如果是锁重入，则将`Lock Record`的`Displaced Mark Word`设置为null，起到一个锁重入计数的作用。
 
 以上是偏向锁加锁的流程（包括部分轻量级锁的加锁流程），如果当前锁已偏向其他线程||epoch值过期||偏向模式关闭||获取偏向锁的过程中存在并发冲突，都会进入到`InterpreterRuntime::monitorenter`方法， 在该方法中会对偏向锁撤销和升级。
 
### 偏向锁的撤销
 这里说的撤销是指在获取偏向锁的过程因为不满足条件导致要将锁对象改为非偏向锁状态；释放是指退出同步块时的过程，释放锁的逻辑会在下一小节阐述。请读者注意本文中**撤销与释放的区别**。
 
 如果获取偏向锁失败会进入到[InterpreterRuntime::monitorenter](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/share/vm/interpreter/interpreterRuntime.cpp#l608)方法
 
 ```c++
 IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorenter(JavaThread* thread, BasicObjectLock* elem))
   ...
   Handle h_obj(thread, elem-obj());
   assert(Universe::heap()-is_in_reserved_or_null(h_obj()),
          "must be NULL or an object");
   if (UseBiasedLocking) {
     // Retry fast entry if bias is revoked to avoid unnecessary inflation
     ObjectSynchronizer::fast_enter(h_obj, elem-lock(), true, CHECK);
   } else {
     ObjectSynchronizer::slow_enter(h_obj, elem-lock(), CHECK);
   }
   ...
 IRT_END
 ```
 
 可以看到如果开启了JVM偏向锁，那会进入到`ObjectSynchronizer::fast_enter`方法中。
 
 ```c++
 void ObjectSynchronizer::fast_enter(Handle obj, BasicLock* lock, bool attempt_rebias, TRAPS) {
  if (UseBiasedLocking) {
     if (!SafepointSynchronize::is_at_safepoint()) {
       BiasedLocking::Condition cond = BiasedLocking::revoke_and_rebias(obj, attempt_rebias, THREAD);
       if (cond == BiasedLocking::BIAS_REVOKED_AND_REBIASED) {
         return;
       }
     } else {
       assert(!attempt_rebias, "can not rebias toward VM thread");
       BiasedLocking::revoke_at_safepoint(obj);
     }
     assert(!obj-mark()-has_bias_pattern(), "biases should be revoked by now");
  }
 
  slow_enter (obj, lock, THREAD) ;
 }
 ```
 
 如果是正常的Java线程，会走上面的逻辑进入到`BiasedLocking::revoke_and_rebias`方法，如果是VM线程则会走到下面的`BiasedLocking::revoke_at_safepoint`。我们主要看`BiasedLocking::revoke_and_rebias`方法。这个方法的主要作用像它的方法名：撤销或者重偏向，第一个参数封装了锁对象和当前线程，第二个参数代表是否允许重偏向，这里是true。
 
 ```c++
 BiasedLocking::Condition BiasedLocking::revoke_and_rebias(Handle obj, bool attempt_rebias, TRAPS) {
   assert(!SafepointSynchronize::is_at_safepoint(), "must not be called while at safepoint");
     
   markOop mark = obj-mark();
   if (mark-is_biased_anonymously() && !attempt_rebias) {
      //如果是匿名偏向且attempt_rebias==false会走到这里，如锁对象的hashcode方法被调用会出现这种情况，需要撤销偏向锁。
     markOop biased_value       = mark;
     markOop unbiased_prototype = markOopDesc::prototype()-set_age(mark-age());
     markOop res_mark = (markOop) Atomic::cmpxchg_ptr(unbiased_prototype, obj-mark_addr(), mark);
     if (res_mark == biased_value) {
       return BIAS_REVOKED;
     }
   } else if (mark-has_bias_pattern()) {
     // 锁对象开启了偏向模式会走到这里
     Klass* k = obj-klass();
     markOop prototype_header = k-prototype_header();
     //code 1： 如果对应class关闭了偏向模式
     if (!prototype_header-has_bias_pattern()) {
       markOop biased_value       = mark;
       markOop res_mark = (markOop) Atomic::cmpxchg_ptr(prototype_header, obj-mark_addr(), mark);
       assert(!(*(obj-mark_addr()))-has_bias_pattern(), "even if we raced, should still be revoked");
       return BIAS_REVOKED;
     //code2： 如果epoch过期
     } else if (prototype_header-bias_epoch() != mark-bias_epoch()) {
       if (attempt_rebias) {
         assert(THREAD-is_Java_thread(), "");
         markOop biased_value       = mark;
         markOop rebiased_prototype = markOopDesc::encode((JavaThread*) THREAD, mark-age(), prototype_header-bias_epoch());
         markOop res_mark = (markOop) Atomic::cmpxchg_ptr(rebiased_prototype, obj-mark_addr(), mark);
         if (res_mark == biased_value) {
           return BIAS_REVOKED_AND_REBIASED;
         }
       } else {
         markOop biased_value       = mark;
         markOop unbiased_prototype = markOopDesc::prototype()-set_age(mark-age());
         markOop res_mark = (markOop) Atomic::cmpxchg_ptr(unbiased_prototype, obj-mark_addr(), mark);
         if (res_mark == biased_value) {
           return BIAS_REVOKED;
         }
       }
     }
   }
   //code 3：批量重偏向与批量撤销的逻辑
   HeuristicsResult heuristics = update_heuristics(obj(), attempt_rebias);
   if (heuristics == HR_NOT_BIASED) {
     return NOT_BIASED;
   } else if (heuristics == HR_SINGLE_REVOKE) {
     //code 4：撤销单个线程
     Klass *k = obj-klass();
     markOop prototype_header = k-prototype_header();
     if (mark-biased_locker() == THREAD &&
         prototype_header-bias_epoch() == mark-bias_epoch()) {
       // 走到这里说明需要撤销的是偏向当前线程的锁，当调用Object#hashcode方法时会走到这一步
       // 因为只要遍历当前线程的栈就好了，所以不需要等到safepoint再撤销。
       ResourceMark rm;
       if (TraceBiasedLocking) {
         tty-print_cr("Revoking bias by walking my own stack:");
       }
       BiasedLocking::Condition cond = revoke_bias(obj(), false, false, (JavaThread*) THREAD);
       ((JavaThread*) THREAD)-set_cached_monitor_info(NULL);
       assert(cond == BIAS_REVOKED, "why not?");
       return cond;
     } else {
       // 下面代码最终会在VM线程中的safepoint调用revoke_bias方法
       VM_RevokeBias revoke(&obj, (JavaThread*) THREAD);
       VMThread::execute(&revoke);
       return revoke.status_code();
     }
   }
 	
   assert((heuristics == HR_BULK_REVOKE) ||
          (heuristics == HR_BULK_REBIAS), "?");
    //code5：批量撤销、批量重偏向的逻辑
   VM_BulkRevokeBias bulk_revoke(&obj, (JavaThread*) THREAD,
                                 (heuristics == HR_BULK_REBIAS),
                                 attempt_rebias);
   VMThread::execute(&bulk_revoke);
   return bulk_revoke.status_code();
 }
 ```
 
 会走到该方法的逻辑有很多，我们只分析最常见的情况：假设锁已经偏向线程A，这时B线程尝试获得锁。
 
 上面的`code 1`，`code 2`B线程都不会走到，最终会走到`code 4`处，如果要撤销的锁偏向的是当前线程则直接调用`revoke_bias`撤销偏向锁，否则会将该操作push到VM Thread中等到`safepoint`的时候再执行。
 
 关于VM Thread这里介绍下：在JVM中有个专门的VM Thread，该线程会源源不断的从VMOperationQueue中取出请求，比如GC请求。对于需要`safepoint`的操作（VM_Operationevaluate_at_safepoint返回true）必须要等到所有的Java线程进入到`safepoint`才开始执行。 关于`safepoint`可以参考下这篇[文章](https://blog.csdn.net/ITer_ZC/article/details/41892567)。
 
 接下来我们着重分析下`revoke_bias`方法。第一个参数为锁对象，第2、3个参数为都为false
 
 ```c++
 static BiasedLocking::Condition revoke_bias(oop obj, bool allow_rebias, bool is_bulk, JavaThread* requesting_thread) {
   markOop mark = obj-mark();
   // 如果没有开启偏向模式，则直接返回NOT_BIASED
   if (!mark-has_bias_pattern()) {
     ...
     return BiasedLocking::NOT_BIASED;
   }
 
   uint age = mark-age();
   // 构建两个mark word，一个是匿名偏向模式（101），一个是无锁模式（001）
   markOop   biased_prototype = markOopDesc::biased_locking_prototype()-set_age(age);
   markOop unbiased_prototype = markOopDesc::prototype()-set_age(age);
 
   ...
 
   JavaThread* biased_thread = mark-biased_locker();
   if (biased_thread == NULL) {
      // 匿名偏向。当调用锁对象的hashcode()方法可能会导致走到这个逻辑
      // 如果不允许重偏向，则将对象的mark word设置为无锁模式
     if (!allow_rebias) {
       obj-set_mark(unbiased_prototype);
     }
     ...
     return BiasedLocking::BIAS_REVOKED;
   }
 
   // code 1：判断偏向线程是否还存活
   bool thread_is_alive = false;
   // 如果当前线程就是偏向线程 
   if (requesting_thread == biased_thread) {
     thread_is_alive = true;
   } else {
      // 遍历当前jvm的所有线程，如果能找到，则说明偏向的线程还存活
     for (JavaThread* cur_thread = Threads::first(); cur_thread != NULL; cur_thread = cur_thread-next()) {
       if (cur_thread == biased_thread) {
         thread_is_alive = true;
         break;
       }
     }
   }
   // 如果偏向的线程已经不存活了
   if (!thread_is_alive) {
     // 允许重偏向则将对象mark word设置为匿名偏向状态，否则设置为无锁状态
     if (allow_rebias) {
       obj-set_mark(biased_prototype);
     } else {
       obj-set_mark(unbiased_prototype);
     }
     ...
     return BiasedLocking::BIAS_REVOKED;
   }
 
   // 线程还存活则遍历线程栈中所有的Lock Record
   GrowableArray<MonitorInfo** cached_monitor_info = get_or_compute_monitor_info(biased_thread);
   BasicLock* highest_lock = NULL;
   for (int i = 0; i < cached_monitor_info-length(); i++) {
     MonitorInfo* mon_info = cached_monitor_info-at(i);
     // 如果能找到对应的Lock Record说明偏向的线程还在执行同步代码块中的代码
     if (mon_info-owner() == obj) {
       ...
       // 需要升级为轻量级锁，直接修改偏向线程栈中的Lock Record。为了处理锁重入的case，在这里将Lock Record的Displaced Mark Word设置为null，第一个Lock Record会在下面的代码中再处理
       markOop mark = markOopDesc::encode((BasicLock*) NULL);
       highest_lock = mon_info-lock();
       highest_lock-set_displaced_header(mark);
     } else {
       ...
     }
   }
   if (highest_lock != NULL) {
     // 修改第一个Lock Record为无锁状态，然后将obj的mark word设置为执行该Lock Record的指针
     highest_lock-set_displaced_header(unbiased_prototype);
     obj-release_set_mark(markOopDesc::encode(highest_lock));
     ...
   } else {
     // 走到这里说明偏向线程已经不在同步块中了
     ...
     if (allow_rebias) {
        //设置为匿名偏向状态
       obj-set_mark(biased_prototype);
     } else {
       // 将mark word设置为无锁状态
       obj-set_mark(unbiased_prototype);
     }
   }
 
   return BiasedLocking::BIAS_REVOKED;
 }
 ```
 
 需要注意下，当调用锁对象的`Object#hash`或`System.identityHashCode()`方法会导致该对象的偏向锁或轻量级锁升级。这是因为在Java中一个对象的hashcode是在调用这两个方法时才生成的，如果是无锁状态则存放在`mark word`中，如果是重量级锁则存放在对应的monitor中，而偏向锁是没有地方能存放该信息的，所以必须升级。具体可以看这篇[文章](https://www.aimoon.site/blog/2018/05/21/biased-locking/)的`hashcode()方法对偏向锁的影响`小节（注意：该文中对于偏向锁的加锁描述有些错误），另外我也向该文章作者请教过一些问题，他很热心的回答了我，**在此感谢一下！**
 
 言归正传，`revoke_bias`方法逻辑：
 
 1. 查看偏向的线程是否存活，如果已经不存活了，则直接撤销偏向锁。JVM维护了一个集合存放所有存活的线程，通过遍历该集合判断某个线程是否存活。
 2. 偏向的线程是否还在同步块中，如果不在了，则撤销偏向锁。我们回顾一下偏向锁的加锁流程：每次进入同步块（即执行`monitorenter`）的时候都会以从高往低的顺序在栈中找到第一个可用的`Lock Record`，将其obj字段指向锁对象。每次解锁（即执行`monitorexit`）的时候都会将最低的一个相关`Lock Record`移除掉。所以可以通过遍历线程栈中的`Lock Record`来判断线程是否还在同步块中。
 3. 将偏向线程所有相关`Lock Record`的`Displaced Mark Word`设置为null，然后将最高位的`Lock Record`的`Displaced Mark Word` 设置为无锁状态，最高位的`Lock Record`也就是第一次获得锁时的`Lock Record`（这里的第一次是指重入获取锁时的第一次），然后将对象头指向最高位的`Lock Record`，这里不需要用CAS指令，因为是在`safepoint`。 执行完后，就升级成了轻量级锁。原偏向线程的所有Lock Record都已经变成轻量级锁的状态。这里如果看不明白，请回顾上篇文章的轻量级锁加锁过程。
 
### 偏向锁的释放
 偏向锁的释放入口在[bytecodeInterpreter.cpp#1923](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/share/vm/interpreter/bytecodeInterpreter.cpp#l1923)
 
 ```c++
 CASE(_monitorexit): {
   oop lockee = STACK_OBJECT(-1);
   CHECK_NULL(lockee);
   // derefing's lockee ought to provoke implicit null check
   // find our monitor slot
   BasicObjectLock* limit = istate-monitor_base();
   BasicObjectLock* most_recent = (BasicObjectLock*) istate-stack_base();
   // 从低往高遍历栈的Lock Record
   while (most_recent != limit ) {
     // 如果Lock Record关联的是该锁对象
     if ((most_recent)-obj() == lockee) {
       BasicLock* lock = most_recent-lock();
       markOop header = lock-displaced_header();
       // 释放Lock Record
       most_recent-set_obj(NULL);
       // 如果是偏向模式，仅仅释放Lock Record就好了。否则要走轻量级锁or重量级锁的释放流程
       if (!lockee-mark()-has_bias_pattern()) {
         bool call_vm = UseHeavyMonitors;
         // header!=NULL说明不是重入，则需要将Displaced Mark Word CAS到对象头的Mark Word
         if (header != NULL || call_vm) {
           if (call_vm || Atomic::cmpxchg_ptr(header, lockee-mark_addr(), lock) != lock) {
             // CAS失败或者是重量级锁则会走到这里，先将obj还原，然后调用monitorexit方法
             most_recent-set_obj(lockee);
             CALL_VM(InterpreterRuntime::monitorexit(THREAD, most_recent), handle_exception);
           }
         }
       }
       //执行下一条命令
       UPDATE_PC_AND_TOS_AND_CONTINUE(1, -1);
     }
     //处理下一条Lock Record
     most_recent++;
   }
   // Need to throw illegal monitor state exception
   CALL_VM(InterpreterRuntime::throw_illegal_monitor_state_exception(THREAD), handle_exception);
   ShouldNotReachHere();
 }
 ```
 
 上面的代码结合注释理解起来应该不难，偏向锁的释放很简单，只要将对应`Lock Record`释放就好了，而轻量级锁则需要将`Displaced Mark Word`替换到对象头的mark word中。如果CAS失败或者是重量级锁则进入到`InterpreterRuntime::monitorexit`方法中。该方法会在轻量级与重量级锁的文章中讲解。
 
### 批量重偏向和批量撤销
 批量重偏向和批量撤销的背景可以看上篇文章，相关实现在`BiasedLocking::revoke_and_rebias`中：
 
 ```c++
 BiasedLocking::Condition BiasedLocking::revoke_and_rebias(Handle obj, bool attempt_rebias, TRAPS) {
   ...
   //code 1：重偏向的逻辑
   HeuristicsResult heuristics = update_heuristics(obj(), attempt_rebias);
   // 非重偏向的逻辑
   ...
       
   assert((heuristics == HR_BULK_REVOKE) ||
          (heuristics == HR_BULK_REBIAS), "?");	
    //code 2：批量撤销、批量重偏向的逻辑
   VM_BulkRevokeBias bulk_revoke(&obj, (JavaThread*) THREAD,
                                 (heuristics == HR_BULK_REBIAS),
                                 attempt_rebias);
   VMThread::execute(&bulk_revoke);
   return bulk_revoke.status_code();
 }
 ```
 
 在每次撤销偏向锁的时候都通过`update_heuristics`方法记录下来，以类为单位，当某个类的对象撤销偏向次数达到一定阈值的时候JVM就认为该类不适合偏向模式或者需要重新偏向另一个对象，`update_heuristics`就会返回`HR_BULK_REVOKE`或`HR_BULK_REBIAS`。进行批量撤销或批量重偏向。
 
 先看`update_heuristics`方法。
 
 ```c++
 static HeuristicsResult update_heuristics(oop o, bool allow_rebias) {
   markOop mark = o-mark();
   //如果不是偏向模式直接返回
   if (!mark-has_bias_pattern()) {
     return HR_NOT_BIASED;
   }
  
   // 锁对象的类
   Klass* k = o-klass();
   // 当前时间
   jlong cur_time = os::javaTimeMillis();
   // 该类上一次批量撤销的时间
   jlong last_bulk_revocation_time = k-last_biased_lock_bulk_revocation_time();
   // 该类偏向锁撤销的次数
   int revocation_count = k-biased_lock_revocation_count();
   // BiasedLockingBulkRebiasThreshold是重偏向阈值（默认20），BiasedLockingBulkRevokeThreshold是批量撤销阈值（默认40），BiasedLockingDecayTime是开启一次新的批量重偏向距离上次批量重偏向的后的延迟时间，默认25000。也就是开启批量重偏向后，经过了一段较长的时间（=BiasedLockingDecayTime），撤销计数器才超过阈值，那我们会重置计数器。
   if ((revocation_count = BiasedLockingBulkRebiasThreshold) &&
       (revocation_count <  BiasedLockingBulkRevokeThreshold) &&
       (last_bulk_revocation_time != 0) &&
       (cur_time - last_bulk_revocation_time = BiasedLockingDecayTime)) {
     // This is the first revocation we've seen in a while of an
     // object of this type since the last time we performed a bulk
     // rebiasing operation. The application is allocating objects in
     // bulk which are biased toward a thread and then handing them
     // off to another thread. We can cope with this allocation
     // pattern via the bulk rebiasing mechanism so we reset the
     // klass's revocation count rather than allow it to increase
     // monotonically. If we see the need to perform another bulk
     // rebias operation later, we will, and if subsequently we see
     // many more revocation operations in a short period of time we
     // will completely disable biasing for this type.
     k-set_biased_lock_revocation_count(0);
     revocation_count = 0;
   }
 
   // 自增撤销计数器
   if (revocation_count <= BiasedLockingBulkRevokeThreshold) {
     revocation_count = k-atomic_incr_biased_lock_revocation_count();
   }
   // 如果达到批量撤销阈值则返回HR_BULK_REVOKE
   if (revocation_count == BiasedLockingBulkRevokeThreshold) {
     return HR_BULK_REVOKE;
   }
   // 如果达到批量重偏向阈值则返回HR_BULK_REBIAS
   if (revocation_count == BiasedLockingBulkRebiasThreshold) {
     return HR_BULK_REBIAS;
   }
   // 没有达到阈值则撤销单个对象的锁
   return HR_SINGLE_REVOKE;
 }
 ```
 
 当达到阈值的时候就会通过VM 线程在`safepoint`调用`bulk_revoke_or_rebias_at_safepoint`, 参数`bulk_rebias`如果是true代表是批量重偏向否则为批量撤销。`attempt_rebias_of_object`代表对操作的锁对象`o`是否运行重偏向，这里是`true`。
 
 ```c++
 static BiasedLocking::Condition bulk_revoke_or_rebias_at_safepoint(oop o,
                                                                    bool bulk_rebias,
                                                                    bool attempt_rebias_of_object,
                                                                    JavaThread* requesting_thread) {
   ...
   jlong cur_time = os::javaTimeMillis();
   o-klass()-set_last_biased_lock_bulk_revocation_time(cur_time);
 
 
   Klass* k_o = o-klass();
   Klass* klass = k_o;
 
   if (bulk_rebias) {
     // 批量重偏向的逻辑
     if (klass-prototype_header()-has_bias_pattern()) {
       // 自增前类中的的epoch
       int prev_epoch = klass-prototype_header()-bias_epoch();
       // code 1：类中的epoch自增
       klass-set_prototype_header(klass-prototype_header()-incr_bias_epoch());
       int cur_epoch = klass-prototype_header()-bias_epoch();
 
       // code 2：遍历所有线程的栈，更新类型为该klass的所有锁实例的epoch
       for (JavaThread* thr = Threads::first(); thr != NULL; thr = thr-next()) {
         GrowableArray<MonitorInfo** cached_monitor_info = get_or_compute_monitor_info(thr);
         for (int i = 0; i < cached_monitor_info-length(); i++) {
           MonitorInfo* mon_info = cached_monitor_info-at(i);
           oop owner = mon_info-owner();
           markOop mark = owner-mark();
           if ((owner-klass() == k_o) && mark-has_bias_pattern()) {
             // We might have encountered this object already in the case of recursive locking
             assert(mark-bias_epoch() == prev_epoch || mark-bias_epoch() == cur_epoch, "error in bias epoch adjustment");
             owner-set_mark(mark-set_bias_epoch(cur_epoch));
           }
         }
       }
     }
 
     // 接下来对当前锁对象进行重偏向
     revoke_bias(o, attempt_rebias_of_object && klass-prototype_header()-has_bias_pattern(), true, requesting_thread);
   } else {
     ...
 
     // code 3：批量撤销的逻辑，将类中的偏向标记关闭，markOopDesc::prototype()返回的是一个关闭偏向模式的prototype
     klass-set_prototype_header(markOopDesc::prototype());
 
     // code 4：遍历所有线程的栈，撤销该类所有锁的偏向
     for (JavaThread* thr = Threads::first(); thr != NULL; thr = thr-next()) {
       GrowableArray<MonitorInfo** cached_monitor_info = get_or_compute_monitor_info(thr);
       for (int i = 0; i < cached_monitor_info-length(); i++) {
         MonitorInfo* mon_info = cached_monitor_info-at(i);
         oop owner = mon_info-owner();
         markOop mark = owner-mark();
         if ((owner-klass() == k_o) && mark-has_bias_pattern()) {
           revoke_bias(owner, false, true, requesting_thread);
         }
       }
     }
 
     // 撤销当前锁对象的偏向模式
     revoke_bias(o, false, true, requesting_thread);
   }
 
   ...
   
   BiasedLocking::Condition status_code = BiasedLocking::BIAS_REVOKED;
 
   if (attempt_rebias_of_object &&
       o-mark()-has_bias_pattern() &&
       klass-prototype_header()-has_bias_pattern()) {
     // 构造一个偏向请求线程的mark word
     markOop new_mark = markOopDesc::encode(requesting_thread, o-mark()-age(),
                                            klass-prototype_header()-bias_epoch());
     // 更新当前锁对象的mark word
     o-set_mark(new_mark);
     status_code = BiasedLocking::BIAS_REVOKED_AND_REBIASED;
     ...
   }
 
   ...
 
   return status_code;
 }
 ```
 
 该方法分为两个逻辑：批量重偏向和批量撤销。
 
 先看批量重偏向，分为两步：
 
 `code 1` 将类中的撤销计数器自增1，之后当该类已存在的实例获得锁时，就会尝试重偏向，相关逻辑在`偏向锁获取流程`小节中。
 
 `code 2` 处理当前正在被使用的锁对象，通过遍历所有存活线程的栈，找到所有正在使用的偏向锁对象，然后更新它们的epoch值。也就是说不会重偏向正在使用的锁，否则会破坏锁的线程安全性。
 
 批量撤销逻辑如下：
 
 `code 3`将类的偏向标记关闭，之后当该类已存在的实例获得锁时，就会升级为轻量级锁；该类新分配的对象的`mark word`则是无锁模式。
 
 `code 4`处理当前正在被使用的锁对象，通过遍历所有存活线程的栈，找到所有正在使用的偏向锁对象，然后撤销偏向锁。

