# 探秘redis持久化流程

## 一. 概述redis持久化的俩种模式

随着redis越来越流行，使用者不在仅仅满足其只是一个内存数据库，同时也期望其能将内存数据落磁盘，这样重启服务就不会导致缓存数据丢失了

redis持久化有俩种模式模式，分别如下:

![](https://mmbiz.qpic.cn/mmbiz_png/tnZGrhTk4dfW1b7hKInQV8x2ZTCpUgSzNzElibKsMXIprTR1zNsOVibPKqpGUKjLgmEqtrot93ad8fCXRGCWiaRvQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 二. rdb持久化模式

- 1、**rdb持久化核心思路**

  获取当前内存快照并将快照数据保存到磁盘实现当前数据全量备份

- 2、**rdb持久化难点**

  - 1、如何获取当前内存数据快照
   
     redis利用linux fork子进程后( cow机制:https://juejin.im/post/5bd96bcaf265da396b72f855 )子进程拥有父进程的内存快照来获取内存快照

  - 2、保存数据落盘时io操作阻塞其他请求
  
     redis rdb持久化落盘操作只在独立的子进程中进行不会影响到主进程中的其他请求

- 3.**rdb持久化后rdb文件的格式**

  1、redis rdb文件整体格式如图1所示

    - 开头为固定的REDIS 5个字符，标示这是一个redis rdb文件
    - 5.0.0标示保存改rdb文件的redis服务为5.0.0版本
    - 64标示保存rdb文件的redis所在机器为64位机
    - 1554103539为该rdb文件的创建时间
    - 23343563为该rdb内存快照需占用内存大小
    - 接着保存的为复制相关标示和数据库数据(见图2)
    - EOF标志代表rdb文件结尾
    - 63562735845846756为rdb完整性检查的校验和

  ![](https://mmbiz.qpic.cn/mmbiz_png/tnZGrhTk4dfW1b7hKInQV8x2ZTCpUgSz6qTVaiaN2Wa1bUdkb3pLgDUPibgQsfMZ4S8g5m4sicuNJtEbCtCURCYcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 2、redis database在rdb文件中的整体格式如图2所示

    - SELECTDB代表一个数据库的开始标志
    - 0代表数据库编号，标示接下来保存的为0号数据库的内容
    - 3456为当前数据库的总元素个数
    - 24为当前数据库过期元素个数
    - KeyValuePair保存具体的数据库中k-v的值

  ![](https://mmbiz.qpic.cn/mmbiz_png/tnZGrhTk4dfW1b7hKInQV8x2ZTCpUgSz5jUf6BuBqk3YbxJ5GTQ5bZ00QBicCXtrZI8tr4qeKOrsV2mJRwtzKqw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)


- 4、**rdb持久化触发条件**

rdb持久化为redis开启的默认持久化方式，可通过配置文件灵活配置，默认如下

>
save 900 1          //15分钟内有一条数据写入时触发rdb持久化
save 300 10        //5分钟内有10条数据写入时触发rdb持久化
save 60 10000     //1分钟内有10000条数据写入时触发rdb持久化


除了默认触发之外，我们也可以通过redis客户端发送命令主动触发rdb持久化，具体命令如下：

>save          //主进程执行rdb持久化操作
>bgsave      //生成后台进程执行rdb持久化(默认方式)

- 5、**rdb持久化数据安全性**

> 疑问: rdb持久化模式下，当机器宕机，会丢失多少数据？
> 答: 从最近一次rdb保存到宕机时的数据都会丢失

- 6、**rdb持久化核心源码(bgsave为例)**

```
int rdbSaveBackground(char *filename, rdbSaveInfo *rsi) {
    pid_t childpid;
    long long start;

    //如果已经存在aof重写进程或rdb持久化进程则返回错误
    if (server.aof_child_pid != -1 || server.rdb_child_pid != -1) return C_ERR;

    server.dirty_before_bgsave = server.dirty;
    server.lastbgsave_try = time(NULL);
    //打开与子进程的进程间通信管道(用于子进程完成持久化后给父进程发送消息)
    openChildInfoPipe();

    start = ustime();
    if ((childpid = fork()) == 0) {
        int retval;

        //子进程
        //关闭监听的连接
        closeListeningSockets(0);   
        //设置子进程名字
        redisSetProcTitle("redis-rdb-bgsave");  
        //保存全量内存信息
        retval = rdbSave(filename,rsi);        
        if (retval == C_OK) {

            //发送cow内存大小信息给父进程
            sendChildInfo(CHILD_INFO_TYPE_RDB);   
        }
        //退出rdb持久化对应进程
        exitFromChild((retval == C_OK) ? 0 : 1);
    } else {
        //父进程
        //统计fork进程花费的时间，当redis占用过多内存空间fork进程可能会很慢
        server.stat_fork_time = ustime()-start;
        server.stat_fork_rate = (double) zmalloc_used_memory() * 1000000 / server.stat_fork_time / (1024*1024*1024); /* GB per second. */

        server.rdb_save_time_start = time(NULL);
        server.rdb_child_pid = childpid;
        server.rdb_child_type = RDB_CHILD_TYPE_DISK;
        //重新设置dict是否可以执行resize操作标志
        updateDictResizePolicy();
        return C_OK;
    }
    return C_OK;
}
```

## 三、 AOF持久化模式

- 1、**aof持久化核心思路**

  将服务器收到的更新命令直接写入文件尾部，恢复时直接重放文件中命令即可

- 2、**aof持久化难点**

  - 1、aof文件由于赘余数据，会变得很庞大(占用磁盘空间、重启load数据变慢)
redis会定期重写aof文件(下文中会详细介绍)，抛弃赘余

  - 2、保存数据到aof文件落盘时io操作阻塞其他请求
   redis aof文件写入在主线程，比较耗时的刷盘操作(fsync)则由专门的异步线程来处理，保证主线程不被io阻塞

- 3、**aof持久化文件格式**

   aof持久化直接将redis命令协议写入，如图3所示

   ![](https://mmbiz.qpic.cn/mmbiz_png/tnZGrhTk4dfW1b7hKInQV8x2ZTCpUgSzVkUtD8O0LMC8LQR7jGAXE7gBFRRSHF3BPqaicibPEEa9d4luopsWwFrw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- 4、**aof持久化触发条件**

  redis默认不开启aof持久化方式，可通过修改如下配置开启

  > appendonly yes #yes开启no关闭

  aof持久化刷盘频率控制

  > appendfsync always  //每次命令都刷盘
  appendfsync everysec //每秒刷盘一次(默认)
  appendfsync no  //从不刷盘，由内核去刷盘
  
  
- 5、**aof持久化对redis性能影响**

  >疑问: aof持久化涉及磁盘io操作(write数据到aof文件)，那么该操作是否会严重影响redis吞吐量?
   答: 在aof文件追加过程中，write调用是在redis主线程内，而fsync(将文件数据从内核缓存区buffer中真正写入磁盘)控制刷盘却是在一个独立的后台线程中来完成，所以并不需要担心开启aof会十分影响redis性能

- **6、aof文件重写**

  - 1、aof重写解决的问题
  
     aof文件重写是为了解决aof命令添加持久化模式下，会有大量赘余命令(比如设置一个k-v，之后又删除)导致aof文件过大。不仅浪费系统资源(主要是磁盘)，而且会导致load数据到内存重放变慢
    
  - 2、aof重写核心原理
  
     aof重写过程会有大量io操作，所以redis会像写rdb文件一样为其分配一个独立的进程去执行重写。重写的过程中redis会将新完成的命令写入一段临时buf。当aof重写完成，将buf中的命令数据批量写入aof文件中

     我们知道aof重写是去掉赘余命令(例如先增加后删除)，如果通过原文件内容去逐一进行逻辑计算找出赘余必然会耗费大量计算和存储资源。redis是通过直接将当前内存k-v数据拼装成命令协议格式并写入aof文件，如此便十分优雅的去掉原aof文件中的赘余命令

  - 3、aof重写客户端触发命令:

   >bgrewriteaofCommand
   
   
- 7、**aof持久化的安全性**

  >疑问: aof持久化模式下，机器宕机，最多丢失多少数据
  答:aof根据appendfsync配置的不同，丢失数据情况分别如下:

   ![](https://mmbiz.qpic.cn/mmbiz_png/tnZGrhTk4dfW1b7hKInQV8x2ZTCpUgSzLkLN1fsO2UD4qqbOAdRicJicw4sUGoCyHakxEicibibWH3buqiaILhribsShA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)


 -  对于always参数，由于刷盘并不是每次更新命令都执行一次，而是一次epollwait触发后只批量执行一次。故其也会丢失数据
 -  everysec虽然是每s执行一次刷盘，理论上最大丢失1s数据，但是由于在刷盘过程中有个特殊逻辑，当某次刷盘前，判断刷盘线程已在工作(处理之前的刷盘任务)，则本次刷盘延后2s。故该模式下最大丢失其实是2s的数据量
 -  no啥时候刷盘，完全有内核管理，对应用系统来说存在很大的不确定性，故很难评估其数据安全性


## 四、rdb持久化在恢复过程中会比aof快很多么？

网上各种资料显示，rdb在恢复过程中会优于aof文件。事实也的确如此，因为aof文件中会存在大量赘余命令，当然会恢复的慢。但是不要忘了，aof文件可是会定期重写(刚重写后同样没赘余)的，如果是刚重写后不久的aof文件，那么其恢复就不会相对rdb慢很多了(理论分析)。所以可以说只要我们重写aof的频次不低，那么aof恢复就不怕他相较rdb有明显的速度差异。




