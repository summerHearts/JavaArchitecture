##Redis慢查询
- 1、什么是SlowLog
  - SlowLog是Redis用来记录慢查询执行时间的日志系统。由于SlowLog只保存在内存中，所以SlowLog的效率非常高，

  - 所以你不用担心会影响到你Redis的性能问题。SlowLog是Redis 2.2.12版本之后才引入的一条命令。
- 2、SlowLog设置
  - 1、redis.conf配置文件设置

     ```
     slowlog-log-slower-than 10000
     slowlog-max-len 128
     ```
         
    - 其中slowlog-log-slower-than表示slowlog的划定界限，只有query执行时间大于slowlog-log-slower-than的才会定义成慢查询，才会被slowlog进行记录。slowlog-log-slower-than设置的单位是微秒，默认是10000微秒，也就是10毫秒。

    - slowlog-max-len表示慢查询最大的条数，当slowlog超过设定的最大值后，会将最早的slowlog删除，是个FIFO队列。
    - 都保存在内存中

 - 2、使用config方式动态设置slowlog
     -  查看当前slowlog-log-slower-than设置

         ```
         127.0.0.1:6379> CONFIG GET slowlog-log-slower-than
         1) "slowlog-log-slower-than"
         2) "10000"
         ```
      - 设置slowlog-log-slower-than为100ms
        
         ```
         127.0.0.1:6379> CONFIG SET slowlog-log-slower-than 100000
         OK
         ```
       
     - 设置slowlog-max-len为1000
       
         ```
        127.0.0.1:6379> CONFIG SET slowlog-max-len 1000
        OK
         ```
- 3、运维经验
  - 1、slowlog-max-len不要设置过大，默认10ms，通常设置1ms.
  
  - 2、slowlog-max-slower-than不要设置过小，通常设置1000左右。
  - 3、slowlog-max-len :线上建议调大慢查询列表,记录慢查询时 Redis 会对长命令做阶段操作,并不会占用大量内存.增大慢查询列表可以减缓慢查询被剔除的可能,例如线上可设置为 1000 以上.

  - 4、slowlog-log-slower-than :默认值超过 10 毫秒判定为慢查询,需要根据 Redis 并发量调整该值.由于 Redis 采用单线程相应命令,对于高流量的场景,如果命令执行时间超过 1 毫秒以上,那么 Redis 最多可支撑 OPS 不到 1000 因此对于高OPS场景下的 Redis 建议设置为 1 毫秒.

  - 5、慢查询只记录命令的执行时间,并不包括命令排队和网络传输时间.因此客户端执行命令的时间会大于命令的实际执行时间.因为命令执行排队机制,慢查询会导致其他命令级联阻塞,因此客户端出现请求超时时,需要检查该时间点是否有对应的慢查询,从而分析是否为慢查询导致的命令级联阻塞.

  - 6、由于慢查询日志是一个先进先出的队列,也就是说如果慢查询比较多的情况下,可能会丢失部分慢查询命令,为了防止这种情况发生,可以定期执行 slowlog get 命令将慢查询日志持久化到其他存储中(例如: MySQL 、 ElasticSearch 等),然后可以通过可视化工具进行查询.
   


