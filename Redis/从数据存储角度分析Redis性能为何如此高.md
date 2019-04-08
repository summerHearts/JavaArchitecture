## 1、Redis性能如此高的原因：

* 纯内存操作
* 单线程
* 高效的数据结构
* 合理的数据编码
* 其他方面的优化

## 2、在Redis中，常用的 5 种数据结构和应用场景如下：

* String：缓存、计数器、分布式锁等。
* List：链表、队列、微博关注人时间轴列表等。
* Hash：用户信息、Hash 表等。
* Set：去重、赞、踩、共同好友等。
* Zset：访问量排行榜、点击量排行榜等。

## 3、特殊的字符串结构**SDS**

 - Redis 是用 C 语言开发完成的，但在 Redis 字符串中，并没有使用 C 语言中的字符串，而是用一种称为 SDS（Simple Dynamic String）的结构体来保存字符串。


![](https://www.icheesedu.com/images/blog/v2-c04de91ae2ec8948c8961e1884acbe6d_hd.jpg)

```
struct sdshdr {
      int len;
    int free;
    char buf[];
}
```

 - SDS 的结构如上图：

* len：用于记录 buf 中已使用空间的长度。
* free：buf 中空闲空间的长度。
* buf[]：存储实际内容。


例如：执行命令 set key value，key 和 value 都是一个 SDS 类型的结构存储在内存中。

**SDS 与 C 字符串的区别**

 - ①常数时间内获得字符串长度

   C 字符串本身不记录长度信息，每次获取长度信息都需要遍历整个字符串，复杂度为 O(n)；C 字符串遍历时遇到 '\0' 时结束。

   SDS 中 len 字段保存着字符串的长度，所以总能在常数时间内获取字符串长度，复杂度是 O(1)。

 - ②避免缓冲区溢出

   假设在内存中有两个紧挨着的两个字符串，s1=“xxxxx”和 s2=“yyyyy”。

   由于在内存上紧紧相连，当我们对 s1 进行扩充的时候，将 s1=“xxxxxzzzzz”后，由于没有进行相应的内存重新分配，导致 s1 把 s2 覆盖掉，导致 s2 被莫名其妙的修改。

   但 SDS 的 API 对 zfc 修改时首先会检查空间是否足够，若不充足则会分配新空间，避免了缓冲区溢出问题。

 - ③减少字符串修改时带来的内存重新分配的次数

   在 C 中，当我们频繁的对一个字符串进行修改（append 或 trim）操作的时候，需要频繁的进行内存重新分配的操作，十分影响性能。

   如果不小心忘记，有可能会导致内存溢出或内存泄漏，对于 Redis 来说，本身就会很频繁的修改字符串，所以使用 C 字符串并不合适。而 SDS 实现了空间预分配和惰性空间释放两种优化策略：

   空间预分配：当 SDS 的 API 对一个 SDS 修改后，并且对 SDS 空间扩充时，程序不仅会为 SDS 分配所需要的必须空间，还会分配额外的未使用空间。

   分配规则如下：如果对 SDS 修改后，len 的长度小于 1M，那么程序将分配和 len 相同长度的未使用空间。举个例子，如果 len=10，重新分配后，buf 的实际长度会变为 10(已使用空间)+10(额外空间)+1(空字符)=21。如果对 SDS 修改后 len 长度大于 1M，那么程序将分配 1M 的未使用空间。

   惰性空间释放：当对 SDS 进行缩短操作时，程序并不会回收多余的内存空间，而是使用 free 字段将这些字节数量记录下来不释放，后面如果需要 append 操作，则直接使用 free 中未使用的空间，减少了内存的分配。

- ④二进制安全

  在 Redis 中不仅可以存储 String 类型的数据，也可能存储一些二进制数据。

  二进制数据并不是规则的字符串格式，其中会包含一些特殊的字符如 '\0'，在 C 中遇到 '\0' 则表示字符串的结束，但在 SDS 中，标志字符串结束的是 len 属性。

## 4、**字典**

与 Java 中的 HashMap 类似，Redis 中的 Hash 比 Java 中的更高级一些。

Redis 本身就是 KV 服务器，除了 Redis 本身数据库之外，字典也是哈希键的底层实现。

字典的数据结构如下所示：

```
typedef struct dict{
      dictType *type;
    void *privdata;
    dictht ht[2];
    int trehashidx;
}
```
```
typedef struct dictht{
    //哈希表数组
    dectEntrt **table;
    //哈希表大小
    unsigned long size;
    //
    unsigned long sizemask;
    //哈希表已有节点数量
    unsigned long used;
}
```

重要的两个字段是 dictht 和 trehashidx，这两个字段与 rehash 有关，下面重点介绍 rehash。

**Rehash**

  - 学过 Java 的朋友都应该知道 HashMap 是如何 rehash 的，在此处我就不过多赘述，下面介绍 Redis 中 Rehash 的过程。

  - 由上段代码，我们可知 dict 中存储了一个 dictht 的数组，长度为 2，表明这个数据结构中实际存储着两个哈希表 ht[0] 和 ht[1]，为什么要存储两张 hash 表呢？

当然是为了 Rehash，Rehash 的过程如下：

* 为 ht[1] 分配空间。如果是扩容操作，ht[1] 的大小为第一个大于等于 ht[0].used*2 的 2^n；如果是缩容操作，ht[1] 的大小为第一个大于等于 ht[0].used 的 2^n。
* 将 ht[0] 中的键值 Rehash 到 ht[1] 中。
* 当 ht[0] 全部迁移到 ht[1] 中后，释放 ht[0]，将 ht[1] 置为 ht[0]，并为 ht[1] 创建一张新表，为下次 Rehash 做准备。


**渐进式 Rehash**

  - 由于 Redis 的 Rehash 操作是将 ht[0] 中的键值全部迁移到 ht[1]，如果数据量小，则迁移过程很快。但如果数据量很大，一个 Hash 表中存储了几万甚至几百万几千万的键值时，迁移过程很慢并会影响到其他用户的使用。

  - 为了避免 Rehash 对服务器性能造成影响，Redis 采用了一种渐进式 Rehash 的策略，分多次、渐进的将 ht[0] 中的数据迁移到 ht[1] 中。

前一过程如下：

* 为 ht[1] 分配空间，让字典同时拥有 ht[0] 和 ht[1] 两个哈希表。
* 字典中维护一个 rehashidx，并将它置为 0，表示 Rehash 开始。
* 在 Rehash 期间，每次对字典操作时，程序还顺便将 ht[0] 在 rehashidx 索引上的所有键值对 rehash 到 ht[1] 中，当 Rehash 完成后，将 rehashidx 属性+1。当全部 rehash 完成后，将 rehashidx 置为 -1，表示 rehash 完成。


注意，由于维护了两张 Hash 表，所以在 Rehash 的过程中内存会增长。另外，在 Rehash 过程中，字典会同时使用 ht[0] 和 ht[1]。

所以在删除、查找、更新时会在两张表中操作，在查询时会现在第一张表中查询，如果第一张表中没有，则会在第二张表中查询。但新增时一律会在 ht[1] 中进行，确保 ht[0] 中的数据只会减少不会增加。

## 5、**跳跃表**

  - Zset 是一个有序的链表结构，其底层的数据结构是跳跃表 skiplist，其结构如下：

```
typedef struct zskiplistNode {
    //成员对象
   robj *obj;
    //分值
   double score;
    //后退指针
   struct zskiplistNode *backward;
    //层
   struct zskiplistLevel {
        struct zskiplistNode *forward;//前进指针
        unsigned int span;//跨度
   } level[];
 } zskiplistNode;

typedef struct zskiplist {
    //表头节点和表尾节点
   struct zskiplistNode *header, *tail;
    //表中节点的的数量
   unsigned long length;
    //表中层数最大的节点层数
   int level;
 } zskiplist;
```

![](https://www.icheesedu.com/images/blog/v2-136c8cfb45a9ccb3320524b88effa30e_hd.png)

- **前进指针**：用于从表头向表尾方向遍历。

- **后退指针**：用于从表尾向表头方向回退一个节点，和前进指针不同的是，前进指针可以一次性跳跃多个节点，后退指针每次只能后退到上一个节点。

- **跨度**：表示当前节点和下一个节点的距离，跨度越大，两个节点中间相隔的元素越多。

在查询过程中跳跃着前进。由于存在后退指针，如果查询时超出了范围，通过后退指针回退到上一个节点后仍可以继续向前遍历。


## 6、**压缩列表**

压缩列表 ziplist 是为 Redis 节约内存而开发的，是列表键和字典键的底层实现之一。

当元素个数较少时，Redis 用 ziplist 来存储数据，当元素个数超过某个值时，链表键中会把 ziplist 转化为 linkedlist，字典键中会把 ziplist 转化为 hashtable。

ziplist 是由一系列特殊编码的连续内存块组成的顺序型的数据结构，ziplist 中可以包含多个 entry 节点，每个节点可以存放整数或者字符串。


由于内存是连续分配的，所以遍历速度很快。

![](https://www.icheesedu.com/images/blog/v2-dbd7e77409687191f06a1d634933110f_hd.jpg)

## 7、**编码转化**

Redis 使用对象（redisObject）来表示数据库中的键值，当我们在 Redis 中创建一个键值对时，至少创建两个对象，一个对象是用做键值对的键对象，另一个是键值对的值对象。

例如我们执行 SET MSG XXX 时，键值对的键是一个包含了字符串“MSG“的对象，键值对的值对象是包含字符串"XXX"的对象。

redisObject 的结构如下：

```
typedef struct redisObject{
    //类型
   unsigned type:4;
   //编码
   unsigned encoding:4;
   //指向底层数据结构的指针
   void *ptr;
    //...
 }robj;
```
 
其中 type 字段记录了对象的类型，包含字符串对象、列表对象、哈希对象、集合对象、有序集合对象。

当我们执行 type key 命令时返回的结果如下：

![](https://www.icheesedu.com/images/blog/v2-437460722a1b6bc407a8c6030e9ef753_hd.jpg)


ptr 指针字段指向对象底层实现的数据结构，而这些数据结构是由 encoding 字段决定的，每种对象至少有两种数据编码：

![](https://www.icheesedu.com/images/blog/v2-cb773032a696d1e39c1b8e75e42d739d_hd.jpg)

可以通过 object encoding key 来查看对象所使用的编码：

![](https://www.icheesedu.com/images/blog/v2-ccc5b2e52cdeba6cb85d73a6c5f19079_hd.jpg)

细心的读者可能注意到，list、hash、zset 三个键使用的是 ziplist 压缩列表编码，这就涉及到了 Redis 底层的编码转换。

上面介绍到，ziplist 是一种结构紧凑的数据结构，当某一键值中所包含的元素较少时，会优先存储在 ziplist 中，当元素个数超过某一值后，才将 ziplist 转化为标准存储结构，而这一值是可配置的。

**String 对象的编码转化**

String 对象的编码可以是 int 或 raw，对于 String 类型的键值，如果我们存储的是纯数字，Redis 底层采用的是 int 类型的编码，如果其中包括非数字，则会立即转为 raw 编码：

```
127.0.0.1:6379> set str 1
OK
127.0.0.1:6379> object encoding str
"int"
127.0.0.1:6379> set str 1a
OK
127.0.0.1:6379> object encoding str
"raw"
127.0.0.1:6379>
```

**List 对象的编码转化**

List 对象的编码可以是 ziplist 或 linkedlist，对于 List 类型的键值，当列表对象同时满足以下两个条件时，采用 ziplist 编码：

列表对象保存的所有字符串元素的长度都小于 64 字节。
列表对象保存的元素个数小于 512 个。
如果不满足这两个条件的任意一个，就会转化为 linkedlist 编码。注意：这两个条件是可以修改的，在 redis.conf 中：

```
list-max-ziplist-entries 512
list-max-ziplist-value 64
```

**Set 类型的编码转化**

Set 对象的编码可以是 intset 或 hashtable，intset 编码的结婚对象使用整数集合作为底层实现，把所有元素都保存在一个整数集合里面。

```
127.0.0.1:6379> sadd set 1 2 3
(integer) 3
127.0.0.1:6379> object encoding set
"intset"
127.0.0.1:6379>
如果 set 集合中保存了非整数类型的数据时，Redis 会将 intset 转化为 hashtable：

127.0.0.1:6379> sadd set 1 2 3
(integer) 3
127.0.0.1:6379> object encoding set
"intset"
127.0.0.1:6379> sadd set a
(integer) 1
127.0.0.1:6379> object encoding set
"hashtable"
 127.0.0.1:6379>
```
 
当 Set 对象同时满足以下两个条件时，对象采用 intset 编码：

* 保存的所有元素都是整数值（小数不行）。
* Set 对象保存的所有元素个数小于 512 个。

不能满足这两个条件的任意一个，Set 都会采用 hashtable 存储。注意：第两个条件是可以修改的，在 redis.conf 中：

```
set-max-intset-entries 512
```

**Hash 对象的编码转化**

 Hash 对象的编码可以是 ziplist 或 hashtable，当 Hash 以 ziplist 编码存储的时候，保存同一键值对的两个节点总是紧挨在一起，键节点在前，值节点在后：


 当 Hash 对象同时满足以下两个条件时，Hash 对象采用 ziplist 编码：

* Hash 对象保存的所有键值对的键和值的字符串长度均小于 64 字节。
* Hash 对象保存的键值对数量小于 512 个。


如果不满足以上条件的任意一个，ziplist 就会转化为 hashtable 编码。注意：这两个条件是可以修改的，在 redis.conf 中：

```
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
```

**Zset 对象的编码转化**

Zset 对象的编码可以是 ziplist 或 zkiplist，当采用 ziplist 编码存储时，每个集合元素使用两个紧挨在一起的压缩列表来存储。

第一个节点存储元素的成员，第二个节点存储元素的分值，并且按分值大小从小到大有序排列。


当 Zset 对象同时满足一下两个条件时，采用 ziplist 编码：

* Zset 保存的元素个数小于 128。
* Zset 元素的成员长度都小于 64 字节。


如果不满足以上条件的任意一个，ziplist 就会转化为 zkiplist 编码。注意：这两个条件是可以修改的，在 redis.conf 中：

```
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
```

思考：Zset 如何做到 O(1) 复杂度内元素并且快速进行范围操作？Zset 采用 skiplist 编码时使用 zset 结构作为底层实现，该数据结构同时包含了一个跳跃表和一个字典。

其结构如下：

```
typedef struct zset{
    zskiplist *zsl;
   dict *dict;
}
```

Zset 中的 dict 字典为集合创建了一个从成员到分值之间的映射，字典中的键保存了成员，字典中的值保存了成员的分值，这样定位元素时时间复杂度是 O(1)。

Zset 中的 zsl 跳跃表适合范围操作，比如 ZRANK、ZRANGE 等，程序使用 zkiplist。

另外，虽然 Zset 中使用了 dict 和 skiplist 存储数据，但这两种数据结构都会通过指针来共享相同的内存，所以没有必要担心内存的浪费。

## 8、过期数据的删除对 Redis 性能影响

当我们对某些 key 设置了 expire 时，数据到了时间会自动删除。如果一个键过期了，它会在什么时候删除呢？

下面介绍三种删除策略：

* 定时删除：在这是键的过期时间的同时，创建一个定时器 Timer，让定时器在键过期时间来临时立即执行对过期键的删除。
* 惰性删除：键过期后不管，每次读取该键时，判断该键是否过期，如果过期删除该键返回空。
* 定期删除：每隔一段时间对数据库中的过期键进行一次检查。


- 定时删除：对内存友好，对 CPU 不友好。如果过期删除的键比较多的时候，删除键这一行为会占用相当一部分 CPU 性能，会对 Redis 的吞吐量造成一定影响。

- 惰性删除：对 CPU 友好，内存不友好。如果很多键过期了，但在将来很长一段时间内没有很多客户端访问该键导致过期键不会被删除，占用大量内存空间。

- 定期删除：是定时删除和惰性删除的一种折中。每隔一段时间执行一次删除过期键的操作，并且限制删除操作执行的时长和频率。

**具体的操作如下：**

 - Redis 会将每一个设置了 expire 的键存储在一个独立的字典中，以后会定时遍历这个字典来删除过期的 key。除了定时遍历外，它还会使用惰性删除策略来删除过期的 key。
 - Redis 默认每秒进行十次过期扫描，过期扫描不会扫描所有过期字典中的 key，而是采用了一种简单的贪心策略。
 - 从过期字典中随机选择 20 个 key；删除这 20 个 key 中已过期的 key；如果过期 key 比例超过 1/4，那就重复步骤 1。
 - 同时，为了保证在过期扫描期间不会出现过度循环，导致线程卡死，算法还增加了扫描时间上限，默认不会超过 25ms。
## 总结

- 总而言之，Redis 为了高性能，从各方各面都进行了优化，下次小伙伴们面试的时候，面试官问 Redis 性能为什么如此高，可不能傻傻的只说单线程和内存存储了。


**特别感谢** 向代码致敬  原文地址：[从数据存储角度分析Redis性能为何如此高](https://mp.weixin.qq.com/s/MXRzbdnHVXGf_of3H8tvkQ)




