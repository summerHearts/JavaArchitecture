## Redis消息订阅与发布
- Redis 发布订阅(pub/sub)是一种消息通信模式，可以用于消息的传输，Redis的发布订阅机制包括三个部分，发布者，订阅者和Channel。适宜做在线聊天、消息推送等。

- 发布者和订阅者都是Redis客户端，Channel则为Redis服务器端，发布者将消息发送到某个的频道，订阅了这个频道的订阅者就能接收到这条消息，客户端可以订阅任意数量的频道

   ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_14-33-04.png)

  - 使用方式
    
     ![](https://www.icheesedu.com/images/qiniu/Xnip2018-07-183_14-38-18.png)

     ```
       //publish channel message：发布者向指定的频道发布消息
        127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"
        (integer) 1
        127.0.0.1:6379> PUBLISH redisChat "Learn redis by runoob.com"
        (integer) 1
        
        //sbuscribe channel [channel2 ... n]：订阅者订阅某个或几个频道，监听channel的消息
        127.0.0.1:6379> subscribe redisChat
        Reading messages... (press Ctrl-C to quit)
        1) "subscribe"
        2) "redisChat"
        3) (integer) 1
        1) "message"
        2) "redisChat"
        3) "Redis is a great caching technique"
        1) "message"
        2) "redisChat"
        3) "Learn redis by runoob.com"
        
        
        127.0.0.1:6379> publish redis-server "AKyS"
        (integer) 1
        127.0.0.1:6379> publish redis-client "BLANK"
        (integer) 1
        
        
        //psubscribe pattern [pattern2 .. n]：根据匹配模式订阅频道，可订阅多个匹配的频道
        127.0.0.1:6379> psubscribe redis*
        Reading messages... (press Ctrl-C to quit)
        1) "psubscribe"
        2) "redis*"
        3) (integer) 1
        1) "pmessage"
        2) "redis*"
        3) "redis-server"
        4) "AKyS"
        1) "pmessage"
        2) "redis*"
        3) "redis-client"
        4) "BLANK"
        
        //pubsub channels [pattern]：列出活跃的频道
        127.0.0.1:6379> pubsub channels
        1) "sohu:tv"
        2) "redisChat"
        
        //pubsub subnum channel [channel2 .. n]：查询频道订阅者数量
        127.0.0.1:6379> pubsub numsub redisChat
        1) "redisChat"
    2) "1"


     ```

