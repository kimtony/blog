
### redis 有什么优点
```
读写快，Redis 使用单线程的多路 IO 复用模型。 （Redis 6.0 引入了多线程 IO ）
```

### 数据结构
* String  （set,get,decr,incr,mget）
```
String 数据结构是简单的 key-value 类型，value 其实不仅可以是 String，也可以是数字。 常规 key-value 缓存应用； 常规计数：微博数，粉丝数等。
```

* Hash    (hget,hset,hgetall)
```
hash 是一个 string 类型的 field 和 value 的映射表 特别适合用于存储对象,存储用户信息，商品信息等等

key=JavaUser293847
value={
  "id": 1,
  "name": "SnailClimb",
  "age": 22,
  "location": "Wuhan, Hubei"
}
```

* List  ( lpush,rpush,lpop,rpop,lrange )
```
list 就是链表 关注列表，粉丝列表，消息列表等功能都可以用 Redis 的 list 结构来实现。

Redis list 的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。

lrange 命令，就是从某个元素开始读取多少个元素，可以基于 list 实现分页查询，这个很棒的一个功能，基于 Redis 实现简单的高性能分页。
```

* Set  （sadd,spop,smembers,sunion）
```
Redis 中的 set 类型是一种无序集合，集合中的元素没有先后顺序。

当你需要存储一个列表数据，又不希望出现重复数据时，set 是一个很好的选择，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。可以基于 set 轻易实现交集、并集、差集的操作

Redis 可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能
```

* Sorted Set  (zadd,zrange,zrem,zcard)
```
和 set 相比，sorted set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列。

举例： 在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息，适合使用 Redis 中的 Sorted Set 结构进行存储。
```


### redis 持久化机制 怎么保证挂掉之后重启数据可以进行恢复
```
有两种：一:RDB（快照） 二：AOF(追加文件)

AOF 持久化 的实时性更好

开启：appendonly yes

开启 AOF 持久化后每执行一条会更改 Redis 中的数据的命令，Redis 就会将该命令写入硬盘中的 AOF 文件
```




### 缓存雪崩
```
缓存同一时间大面积的失效，所以，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩掉


事前：尽量保证整个 Redis 集群的高可用性，发现机器宕机尽快补上。选择合适的内存淘汰策略。
事中：本地 ehcache 缓存 + hystrix 限流&降级，避免 MySQL 崩掉
事后：利用 Redis 持久化机制保存的数据尽快恢复缓存
```


### 缓存穿透
```
缓存穿透说简单点就是大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有经过缓存这一层

布隆过滤器解决缓存穿透
```


### 缓存同步
```
1、在更新数据库时，确保及时更新相应的缓存。这意味着在数据库更新操作成功后，立即更新缓存中相应的数据。
2、将数据库更新操作和缓存更新操作放入消息队列中异步处理，确保数据库和缓存之间的操作顺序一致性。
```


https://blog.csdn.net/liuming690452074/article/details/126070611
