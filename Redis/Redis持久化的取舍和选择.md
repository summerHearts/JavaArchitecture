##  1、Redis持久化的取舍和选择

- Redis是一种高级key-value数据库。它跟memcached类似，不仅数据可以持久化，而且支持的数据类型很丰富。有字符串，链表，集 合和有序集合。支持在服务器端计算集合的并，交和补集(difference)等，还支持多种排序功能。所以Redis也可以被看成是一个数据结构服务器。

- Redis的所有数据都是保存在内存中，然后不定期的通过异步方式保存到磁盘上(这称为“半持久化模式”)；也可以把每一次数据变化都写入到一个append only file(aof)里面(这称为“全持久化模式”)。 
 
- 由于Redis的数据都存放在内存中，如果没有配置持久化，redis重启后数据就全丢失了，于是需要开启redis的持久化功能，将数据保存到磁盘上，当redis重启后，可以从磁盘中恢复数据。redis提供两种方式进行持久化，一种是RDB持久化（原理是将Reids在内存中的数据库记录定时dump到磁盘上的RDB持久化），另外一种是AOF（append only file）持久化（原理是将Reids的操作日志以追加的方式写入文件）。那么这两种持久化方式有什么区别呢，改如何选择呢？网上看了大多数都是介绍这两种方式怎么配置，怎么使用，就是没有介绍二者的区别，在什么应用场景下使用。

- 1、RDB与AOF二者的区别

  - RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。

    ![](https://www.icheesedu.com/images/qiniu/388326-20170726161552843-904424952.png)
    
 - AOF持久化以日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录，以文本的方式记录，可以打开文件看到详细的操作记录。

    ![](https://www.icheesedu.com/images/qiniu/388326-20170726161604968-371688235.png)

##  2、RDB

   - RDB 是 Redis 默认的持久化方案。在指定的时间间隔内，执行指定次数的写操作，则会将内存中的数据写入到磁盘中。即在指定目录下生成一个dump.rdb文件。Redis 重启会通过加载dump.rdb文件恢复数据。
   - 触发机制
      - 主要有三种触发机制
         
         ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_15-54-00.png)
   
      - 1、save 
        ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_15-55-10.png)

      - 2、bgsave
         ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_16-14-56.png)
         
      - 3、两者之间的区别
         ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_16-15-50.png)
      
      - 4、不容忽略的方式

         - 1、全量复制
         - 2、debug reload
         - 3、shutdown
       这三种方式都将触发RDB持久化方式。
       
## 2.1 从配置文件了解RDB
 - 2.1、打开 redis.conf 文件，找到 SNAPSHOTTING 对应内容
   -  1、 RDB核心规则配置（重点）
     
     ```
     save <seconds> <changes>
     # save ""
     save 900 1
     save 300 10
     save 60 10000
     ```
     解说：save <指定时间间隔> <执行指定次数更新操作>，满足条件就将内存中的数据同步到硬盘中。官方出厂配置默认是 900秒内有1个更改，300秒内有10个更改以及60秒内有10000个更改，则将内存中的数据快照写入磁盘。
若不想用RDB方案，可以把 save "" 的注释打开，下面三个注释。
    
    - 2、指定本地数据库文件名，一般采用默认的 dump.rdb
    
     ```
     dbfilename dump.rdb
     ```
    - 3、 指定本地数据库存放目录

      ```
      dir ./opt/redis/data
      ```
    - 4、 默认开启数据压缩

      ```
      rdbcompression yes
      ```
      解说：配置存储至本地数据库时是否压缩数据，默认为yes。Redis采用LZF压缩方式，但占用了一点CPU的时间。若关闭该选项，但会导致数据库文件变的巨大。建议开启。
 - 触发RDB快照

    - 1 在指定的时间间隔内，执行指定次数的写操作
    - 2 执行save（阻塞， 只管保存快照，其他的等待） 或者是bgsave （异步）命令
    - 3 执行flushall 命令，清空数据库所有数据，意义不大。
    - 4 执行shutdown 命令，保证服务器正常关闭且不丢失任何数据，意义...也不大。

- 通过RDB文件恢复数据

     - 将dump.rdb 文件拷贝到redis的安装目录的bin目录下，重启redis服务即可。在实际开发中，一般会考虑到物理机硬盘损坏情况，选择备份dump.rdb 。可以从下面的操作演示中可以体会到。

 - RDB 的优缺点

     - 优点：
         - 1 适合大规模的数据恢复。
         - 2 如果业务对数据完整性和一致性要求不高，RDB是很好的选择。

     - 缺点：
         - 1 数据的完整性和一致性不高，因为RDB可能在最后一次备份时宕机了。
         - 2 备份时占用内存，因为Redis 在备份时会独立创建一个子进程，将数据写入到一个临时文件（此时内存中的数据是原来的两倍哦），最后再将临时文件替换之前的备份文件。
         
 所以Redis 的持久化和数据的恢复要选择在夜深人静的时候执行是比较合理的。
 
- 操作演示

    ```
    [root@ikenvin bin]# vim redis.conf
save 900 1
save 120 5
save 60 10000
[root@ikenvin bin]# ./redis-server redis.conf
[root@ikenvin bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set key1 value1
OK
127.0.0.1:6379> set key2 value2
OK
127.0.0.1:6379> set key3 value3
OK
127.0.0.1:6379> set key4 value4
OK
127.0.0.1:6379> set key5 value5
OK
127.0.0.1:6379> set key6 value6
OK
127.0.0.1:6379> SHUTDOWN
not connected> QUIT
[root@ikenvin bin]# cp dump.rdb dump_bk.rdb
[root@ikenvin bin]# ./redis-server redis.conf
[root@ikenvin bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> FLUSHALL 
OK
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> SHUTDOWN
not connected> QUIT
[root@ikenvin bin]# cp dump_bk.rdb  dump.rdb
cp: overwrite `dump.rdb'? y
[root@ikenvin bin]# ./redis-server redis.conf
[root@ikenvin bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> keys *
1) "key5"
2) "key1"
3) "key3"
4) "key4"
5) "key6"
6) "key2"
    ```
第一步：vim 修改持久化配置时间，120秒内修改5次则持久化一次。
第二步：重启服务使配置生效。
第三步：分别set 5个key，过两分钟后，在bin的当前目录下会自动生产一个dump.rdb文件。（set key6 是为了验证shutdown有触发RDB快照的作用）
第四步：将当前的dump.rdb 备份一份（模拟线上工作）。
第五步：执行FLUSHALL命令清空数据库数据（模拟数据丢失）。
第六步：重启Redis服务，恢复数据.....咦？？？？( ′◔ ‸◔`)。数据是空的？？？？这是因为FLUSHALL也有触发RDB快照的功能。
第七步：将备份的 dump_bk.rdb 替换 dump.rdb 然后重新Redis。

注意点：SHUTDOWN 和 FLUSHALL 命令都会触发RDB快照，这是一个坑，请大家注意。
其他命令：

  - keys * 匹配数据库中所有 key
  - save 阻塞触发RDB快照，使其备份数据
  - FLUSHALL 清空整个 Redis 服务器的数据(几乎不用)
  - SHUTDOWN 关机走人（很少用）

## 2.2 RDB 总结
  - 1、RDB是Redis内存到硬盘的快照，用于持久化。
  - 2、save通常会阻塞redis。
  - 3、bgsave不会阻塞redis,但是会fork新进程。
  - 4、save自动配置满足任意一就会被执行。
  - 5、有些触发机制不容忽视。

## 3、AOF 
- RDB 现存的问题
   - 1、耗时，耗性能。
   ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_17-11-16.png)
   - 2、不可控，容易丢失数据。
   ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_17-11-38.png)
   
- AOF ：Redis 默认不开启。它的出现是为了弥补RDB的不足（数据的不一致性），所以它采用日志的形式来记录每个写操作，并追加到文件中。Redis 重启的会根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

    ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_17-13-28.png)
    
- 1:AOF持久化的三种策略
  - 1、Always
     - 服务器每写入一个命令,就调用一次fdatasync,将缓冲区里面的命令写入到磁盘里面,在这种模式下,服务器即使遭遇意外停机,也不会丢失任何自己已经成功执行的命令数据
  - 2、Everysec
     - 服务器每一秒重调用一次fdatasync,将缓冲区里面的命令写入到磁盘里面,在这种模式写,服务器即使遭遇意外停机时,最多只丢失一秒钟内执行的命令数据
  - 3、NO
     - 服务器不主动调用fdatasync,由操作系统决定任何将缓冲区里面的命令写入磁盘里面,在这种模式写,服务器遭遇意外停机时,丢失命令的数据是不确定的
 
 always的速度慢,everysec和no都很快,默认值:everysec
 
## 3.1  为什么要有AOF重写策略
 
   - AOF 持久化是通过保存被执行的写命令来记录数据库状态的，所以AOF文件的大小随着时间的流逝一定会越来越大；影响包括但不限于：对于Redis服务器，计算机的存储压力；AOF还原出数据库状态的时间增加；
  - 为了解决AOF文件体积膨胀的问题，Redis提供了AOF重写功能：Redis服务器可以创建一个新的AOF文件来替代现有的AOF文件，新旧两个文件所保存的数据库状态是相同的，但是新的AOF文件不会包含任何浪费空间的冗余命令，通常体积会较旧AOF文件小很多。

  - 作用：
     - 1、减少磁盘占用量
     - 2、加速恢复速度

## 3.2 AOF重写实现的两种方式
- 1、bgrewriteaof
     - Redis Bgrewriteaof 命令用于异步执行一个 AOF（AppendOnly File） 文件重写操作。重写会创建一个当前 AOF 文件的体积优化版本
     
     - 即使 Bgrewriteaof 执行失败，也不会有任何数据丢失，因为旧的 AOF 文件在 Bgrewriteaof 成功之前不会被修改。
注意：从 Redis 2.4 开始， AOF 重写由 Redis 自行触发， BGREWRITEAOF 仅仅用于手动触发重写操作。
- 2、AOF重写配置

    - 配置重写触发机制

      ```
      auto-aof-rewrite-percentage 100
      auto-aof-rewrite-min-size 64mb
      ```
    解说：当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。一般都设置为3G，64M太小了。
 
      ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_17-26-53.png)
    
      ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_17-27-37.png)
    
- 3、重写流程
 
       ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_17-27-53.png)
-  4、从配置文件了解AOF

   - 打开 redis.conf 文件，找到 APPEND ONLY MODE 对应内容
   - 1 redis 默认关闭，开启需要手动把no改为yes

     ```
     appendonly yes
     ```
     
   - 2、指定本地数据库文件名，默认值为 appendonly.aof

     ```
     appendfilename "appendonly.aof"
     ```

## 3.3 实战操作

```
[root@ikenvin bin]# vim appendonly.aof
appendonly yes
[root@ikenvin bin]# ./redis-server redis.conf
[root@ikenvin bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set keyAOf valueAof
OK
127.0.0.1:6379> FLUSHALL 
OK
127.0.0.1:6379> SHUTDOWN
not connected> QUIT
[root@ikenvin bin]# ./redis-server redis.conf
[root@ikenvin bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> keys *
1) "keyAOf"
127.0.0.1:6379> SHUTDOWN
not connected> QUIT
[root@ikenvin bin]# vim appendonly.aof
fjewofjwojfoewifjowejfwf
[root@ikenvin bin]# ./redis-server redis.conf
[root@ikenvin bin]# ./redis-cli -h 127.0.0.1 -p 6379
Could not connect to Redis at 127.0.0.1:6379: Connection refused
not connected> QUIT
[root@ikenvin bin]# redis-check-aof --fix appendonly.aof 
'x              3e: Expected prefix '*', got: '
AOF analyzed: size=92, ok_up_to=62, diff=30
This will shrink the AOF from 92 bytes, with 30 bytes, to 62 bytes
Continue? [y/N]: y
Successfully truncated AOF
[root@ikenvin bin]# ./redis-server redis.conf
[root@ikenvin bin]# ./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> keys *
1) "keyAOf"
```
第一步：修改配置文件，开启AOF持久化配置。
第二步：重启Redis服务，并进入Redis 自带的客户端中。
第三步：保存值，然后模拟数据丢失，关闭Redis服务。
第四步：重启服务，发现数据恢复了。（额外提一点：有教程显示FLUSHALL 命令会被写入AOF文件中，导致数据恢复失败。我安装的是redis-4.0.2没有遇到这个问题）。
第五步：修改appendonly.aof，模拟文件异常情况。
第六步：重启 Redis 服务失败。这同时也说明了，RDB和AOF可以同时存在，且优先加载AOF文件。
第七步：校验appendonly.aof 文件。重启Redis 服务后正常。

补充点：aof 的校验是通过 redis-check-aof 文件，那么rdb 的校验是不是可以通过 redis-check-rdb 文件呢

## 4、总结

- Redis 默认开启RDB持久化方式，在指定的时间间隔内，执行指定次数的写操作，则将内存中的数据写入到磁盘中。
- RDB 持久化适合大规模的数据恢复但它的数据一致性和完整性较差。
- Redis 需要手动开启AOF持久化方式，默认是每秒将写操作日志追加到AOF文件中。
- AOF 的数据完整性比RDB高，但记录内容多了，会影响数据恢复的效率。
- Redis 针对 AOF文件大的问题，提供重写的瘦身机制。
- 若只打算用Redis 做缓存，可以关闭持久化。
- 若打算使用Redis 的持久化。建议RDB和AOF都开启。其实RDB更适合做数据的备份，留一后手。AOF出问题了，还有RDB。


##  5、推荐阅读

[持久化的取舍和选择](http://www.imooc.com/article/255833)

