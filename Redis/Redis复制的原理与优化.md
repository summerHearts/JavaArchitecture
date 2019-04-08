## Redis复制的原理与优化
- 1、单机有什么问题
  - 机器故障 ：机器故障，导致数据损坏，服务无法提供。
  - 容量瓶颈  ：16G内存，redis需要12G，是否需要购买一个128G内存的？
  - QPS瓶颈： 10万QPS，需求为100万QPS？怎么解决？

- 2、主从复制的作用
  -  Redis持久化保证了即使redis服务重启也不会丢失数据，因为redis服务重启后会将硬盘上持久化的数据恢复到内存中，但是当redis服务器的硬盘损坏了可能会导致数据丢失，如果通过redis的主从复制机制就可以避免这种单点故障，如下图：
    ![](https://www.icheesedu.com/images/qiniu/fd36c6cfc40adf5907b9d5016c3354b3d8c0e2c1.png)
    
   -  说明
       -  主redis中的数据有两个副本（replication）即从redis1和从redis2，即使一台redis服务器宕机其它两台redis服务也可以继续提供服务。
       -  主redis中的数据和从redis上的数据保持实时同步，当主redis写入数据时通过主从复制机制会复制到两个从redis服务上。
       -  只有一个主redis，可以有多个从redis。
       -   主从复制不会阻塞master，在同步数据时，master 可以继续处理client 请求。

- 3、主从复制配置
  - 1、命令实现
     
     - 在6380 服务器上执行 slaveof 127.0.0.1 6379
     
      ![](https://www.icheesedu.com/images/qiniu/ffddf97389495a8bd761d7ea0e7d817b_934x366.png)

     - 取消复制，不想成为任何服务器的从 -- 在6380 服务器上执行 slaveof no one

      ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-184_09-44-54.png)

 - 2、配置实现

     ```
     slaveof   ip    port
     slave-read-only  yes   //只做读操作，保障保障与主服务器数据一致
     ```
 - 3、比较

        | 方式 | 命令 | 配置 |
        | --- | --- | --- |
        | 优点 | 无需重启 | 统一配置 |
        | 缺点 | 不便于管理 | 需要重启 |
        
  - 4、实践
   
   ```
 cp redis.conf redis-6380.conf
 vim  redis.conf
 daemonize yes
 pidfile /var/run/redis-6379.pid
 port 6379
 logfile   "6379.log"
  //关闭自动触发机制
 -- save 900 1
 -- save 300 10
 -- save 60 10000
 dbfilename dump-6379.rdb  //多节点使用不同的db.
 slave-read-only yes  //只做读操作，保障保障与主服务器数据一致
      
  //启动6379配置
  redis-server  redis-6379.conf  
  ps -ef | grep redis
  501 84962 84609   0  9:51上午 ttys000    0:00.46 redis-server *:6379
  redis-cli  
  127.0.0.1:6379> info replication
    # Replication
    role:master
    connected_slaves:0
    master_repl_offset:0
    repl_backlog_active:0
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:0
    repl_backlog_histlen:0
    
     // 启动6380配置
     redis-server redis-6380.conf 
     redis-cli -p 6380
     27.0.0.1:6380> info replication
    # Replication
    role:slave
    master_host:127.0.0.1
    master_port:6379
    master_link_status:up
    master_last_io_seconds_ago:9
    master_sync_in_progress:0
    slave_repl_offset:183
    slave_priority:100
    slave_read_only:1
    connected_slaves:0
    master_repl_offset:0
    repl_backlog_active:0
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:0
    repl_backlog_histlen:0
 ```
此时在从节点执行  修改数据是不被允许的。只能在主节点更新数据。

  ```
  127.0.0.1:6380> set hello java
  (error) READONLY You can't write against a read only slave.
  ```
打开log日志查看在主从复制的过程中都发生了什么操作
 
 ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-184_10-22-18.png)

  
- 4、runid和复制偏移量
   - redis每次启动的时候都会有一个随机的id来保障redis的标识，重启后消失。
   - 查看runid
   
      ```
      redis-cli -p 6379 info server | grep run 
      run_id:cb50caa478465c82a1e184dc7992891c2f2903a7
      
      redis-cli -p 6380 info server | grep run 
      run_id:d4a994556cfe63bab8e52ad5bdae6fa17a913c8e
      ```
  当首次或者发现主节点的run_id发生了改变，一般都会全部更新。
  
  - 2、偏移量
      -  一个数据写入量的字节，记录写了多少数据。主服务器会把偏移量同步给从服务器，当主从的偏移量一致，则数据是完全同步的。
      - 如果主从服务的偏移量大于从服务器，则主从不同步。
     
     比如： 
     
     ```
     redis-cli -p 6379 info replication 
    # Replication
    role:master
    connected_slaves:1
    slave0:ip=127.0.0.1,port=6380,state=online,offset=2621,lag=0
    master_repl_offset:2621
    repl_backlog_active:1
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:2
    repl_backlog_histlen:2620
     ```
     master_repl_offset:2621  偏移量为2621 。主从偏移量大，说明主从复制出现问题。
     
- 5、全量复制和部分复制      
  - 1、全量复制
     -  流程
         - slave 向 master 传递命令 psync? -1 （因为第一次通信不知道master的runid和偏移量，所以传-1）
         - master 向 slave 返回runid 和偏移量
         - slave 保存 master 的信息
         - master 执行 bgsave 生产RDB快照
         - master 做send RDB 操作 向 slave 同步快照信息
         - master 做 send buffer 操作 ， 向 slave 同步 生成快照过程中的 缓存命令
         - slave 加载 RDB文件及数据
     - 开销
         - bgsave时间
         - RDB文件网络传输时间
         - 从节点清空数据时间
         - 从节点加载RDB的时间
         - 可能的AOF重写时间

     ![](https://www.icheesedu.com/images/qiniu/faba5668620f687f257cdc069f066e57_849x480.png) 
     
     
    - 2、部分复制

      - redis 2.8后的功能，当网络发生抖动断开后，会用到部分复制的功能
      - 当网络发生抖动，slave会与master断开
      - master 写命令时，会写一份复制缓冲区的命令
      - 当slave在此连接master时 ，传递命令 psync {offset} {runid} ,告诉 master 自己当前的偏移量是多少
      - master 向 slave 返回CONTINUE 把 缺失的内容 传递过去。

      ![](https://www.icheesedu.com/images/qiniu/afcde1f61a86395226fb6159fcb5bdda_725x462.png)
      
- 6、故障处理
  - 故障是不可避免的。
  - 无自动故障转移，是多么痛苦的。一般都是在晚上出现问题。
  
   - 从服务宕机
   
     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-184_10-56-59.png)
  
      修改配置，启动另一台为从服务。
   
    - 主服务宕机
    
     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-184_10-58-31.png)
     
     选择一个从服务，让你作为主服务，并且能够进行读写。
     
     
- 7、开发与运维中的问题

    - 1、读写分离
     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-184_11-05-33.png)

    - 2、主从配置不一致

      ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-184_11-07-54.png)
      
    - 3、规避全量复制
     
       ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-184_11-10-17.png)
       
    - 4、规避复制风暴
    
      ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-184_11-13-01.png)




