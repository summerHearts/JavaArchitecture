##redis-cli

```
//字符串键值结构:缓存,计数器播放量,分布式锁
127.0.0.1:6379> set hello world // 赋值
OK
127.0.0.1:6379> get hello  // 取值
"world"
127.0.0.1:6379>  incr playCount //当存储的字符串是整数形式时，redis提供了一个使用的命令 incr 作用是让当前的键值递增，并返回递增后的值
(integer) 1
127.0.0.1:6379> get playCount
"1"
127.0.0.1:6379> incrby playCount 99
(integer) 100
127.0.0.1:6379> decr playCount //当要操作的键不存在时会默认键值为 0  ，所以第一次递增后的结果是 1 ，当键值不是整数时 redis会提示错误
(integer) 99
127.0.0.1:6379> setnx hello word //类似添加操作，key不存在
(integer) 0
127.0.0.1:6379> get hello 
"world"
127.0.0.1:6379> set  hello java xx  //更新操作
OK
127.0.0.1:6379> get hello
"java"
127.0.0.1:6379> exists hello //判断是否存在
(integer) 1
127.0.0.1:6379> mset hello word java best php good //批量操作 
OK
127.0.0.1:6379> mget hello java php
1) "word"
2) "best"
3) "good"
127.0.0.1:6379> getset hello objc //设置新值，返回旧值
"word"
127.0.0.1:6379> append hello ",swift"  //拼接
(integer) 10
127.0.0.1:6379> get hello
"objc,swift"
127.0.0.1:6379> strlen hello //字符串长度
(integer) 10
127.0.0.1:6379> incrbyfloat count 3.8  //浮点型类型
"3.8"
127.0.0.1:6379> get count
"3.8"
127.0.0.1:6379> set hello javabest 
OK
127.0.0.1:6379> getrange hello 0 2 //截取对应部分的长度
"jav"
127.0.0.1:6379> setrange hello 5 A
(integer) 8
127.0.0.1:6379> get hello
"javabAst"
//实战：记录网站每个用户个人主页的访问量/缓存视频的基本信息/分布式id生成器
```





