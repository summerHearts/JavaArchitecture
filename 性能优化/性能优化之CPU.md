## 性能优化之CPU
- 1、什么是性能优化
   - 性能优化就是发挥机器本来的性能
- 2、性能的几个维度之 CPU
   
   
   命令 vmstat
    
  ```
  mpstat -P ALL  和  sar -P ALL 
  ```

 说明：sar -P ALL > aaa.txt   重定向输出内容到文件 aaa.txt
 
 ![](http://files.jb51.net/file_images/article/201308/201308230923349.gif)
 
 vmstat(选项)(参数)
 
 ```
a：显示活动内页；
f：显示启动后创建的进程总数；
m：显示slab信息；
n：头信息仅显示一次；
s：以表格方式显示事件计数器和内存状态；
d：报告磁盘状态；
p：显示指定的硬盘分区状态；
S：输出信息的单位。
 ```
 参数
 
   - 事件间隔：状态信息刷新的时间间隔；
   -  次数：显示报告的次数。

  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-149_17-43-00.png)
  
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-149_17-46-47.png)
 

 
   命令 Top
   
     - :top命令经常用来监控Linux的系统状况，比如cpu、内存的使用，程序员基本都知道这个命令，但比较奇怪的是能用好它的人却很少，例如top监控视图中内存数值的含义就有不少的曲解。
     
     -  :本文通过一个运行中的WEB服务器的top监控截图，讲述top视图中的各种数据的含义，还包括视图中各进程（任务）的字段的排序。



  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-149_17-01-40.png)
  
  第一行：
  
     - 10:01:23 — 当前系统时间
     
     - 126 days, 14:29 — 系统已经运行了126天14小时29分钟（在这期间没有重启过）
     - 2 users — 当前有2个用户登录系统
     - load average: 1.15, 1.42, 1.44 — load average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。
     - load average数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了。

  第二行：
  
     - Tasks — 任务（进程），系统现在共有183个进程，其中处于运行中的有1个，182个在休眠（sleep），stoped状态的有0个，zombie状态（僵尸）的有0个。

  第三行：cpu状态
  
     - 6.7% us — 用户空间占用CPU的百分比。
     
     - 0.4% sy — 内核空间占用CPU的百分比。
     - 0.0% ni — 改变过优先级的进程占用CPU的百分比
     - 92.9% id — 空闲CPU百分比
     - 0.0% wa — IO等待占用CPU的百分比
     - 0.0% hi — 硬中断（Hardware IRQ）占用CPU的百分比
     - 0.0% si — 软中断（Software Interrupts）占用CPU的百分比

 第四行：内存状态

     - 8306544k total — 物理内存总量（8GB）
     
     - 7775876k used — 使用中的内存总量（7.7GB）
     - 530668k free — 空闲内存总量（530M）
     - 79236k buffers — 缓存的内存量 （79M）

 第五行：swap交换分区
 
     - 2031608k total — 交换区总量（2GB）
     
     - 2556k used — 使用的交换区总量（2.5M）
     - 2029052k free — 空闲交换区总量（2GB）
     - 4231276k cached — 缓冲的交换区总量（4GB）

 这里要说明的是不能用windows的内存概念理解这些数据，如果按windows的方式此台服务器“危矣”：8G的内存总量只剩下530M的可用内存。Linux的内存管理有其特殊性，复杂点需要一本书来说明，这里只是简单说点和我们传统概念（windows）的不同。
 
  第四行中使用中的内存总量（used）指的是现在系统内核控制的内存数，空闲内存总量（free）是内核还未纳入其管控范围的数量。纳入内核管理的内存不见得都在使用中，还包括过去使用过的现在可以被重复利用的内存，内核并不把这些可被重新使用的内存交还到free中去，因此在linux上free内存会越来越少，但不用为此担心。

 如果出于习惯去计算可用内存数，这里有个近似的计算公式：第四行的free + 第四行的buffers + 第五行的cached，按这个公式此台服务器的可用内存：530668+79236+4231276 = 4.7GB。

 对于内存监控，在top里我们要时刻监控第五行swap交换分区的used，如果这个数值在不断的变化，说明内核在不断进行内存和swap的数据交换，这是真正的内存不够用了。
 
 
 
  第七行以下：各进程（任务）的状态监控
  
     - PID — 进程id
     
     - USER — 进程所有者
     - PR — 进程优先级
     - NI — nice值。负值表示高优先级，正值表示低优先级
     - VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
     - RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
     - SHR — 共享内存大小，单位kb
     - S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
     - %CPU — 上次更新到现在的CPU时间占用百分比
     - %MEM — 进程使用的物理内存百分比
     - TIME+ — 进程使用的CPU时间总计，单位1/100秒
     - COMMAND — 进程名称（命令名/命令行）


  多U多核CPU监控
  
     - 在top基本视图中，按键盘数字“1”，可监控每个逻辑CPU的状况：
     
    ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-149_17-22-42.png)
    
 如何确定是哪一个线程出现的问题
 
  ```
  jstack  pid > a.txt // pid是进程号 
  print "%x \n " pid 
  vi a.txt
  shift h //显示线程
  ```
  注意： 这里使用print "%x \n " pid   查询返回的是线程数 。在a.txt中查询 `/threadId`
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-149_17-29-32.png)
  
  
  
    
  
  [Top命令](http://man7.org/linux/man-pages/man1/top.1.html)
  
  ```
  cat  /proc/cpuinfo
  ```
  
  ![](http://ovsiiuil2.bkt.clouddn.com/Xnip2018-05-149_17-07-43.png)
  



