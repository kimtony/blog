
## redis 有什么优点
```
读写快，Redis 使用单线程的多路 IO 复用模型。 （Redis 6.0 引入了多线程 IO ） 
```

## 数据结构
### String结构  （set,get,decr,incr,mget）
* 字符串常用操作
```
SET key value                     //存入字符串键值对
MSET key value [key value ...]    //批量存储字符串键值对
SETNX key value                   //存入一个不存在的字符串键值对
GET key                           //获取一个字符串键值
MGET key [key...]                 //批量获取字符串键值
DEL key [key...]                  //删除一个键
EXPIRE key seconds                //设置一个键的过期时间(秒)
```
* 原子加减
```
INCR key                   //将key中存储的数字值加1
DECR key                   //将key中存储的数字值减1
INCRBY key increment       //将key所存储的值加上increment
DECRBY key decrement       //将key所存储的值减去decrement
```
### String常见应用场景
```
String 数据结构是简单的 key-value 类型，value 其实不仅可以是 String，也可以是数字。
常规 key-value 缓存应用； 常规计数：微博数，粉丝数等。
```
* 单值缓存
SET key value        APPEND key value
* 对象缓存
```
1) set user:1 `{"name":"roy","balance":1888}`
2) mset user:1:name roy user:1:balance:1888
   megt user:1:name user:1:balance
```
* 分布式锁
```
SETNX product:10001 true          //返回1代表获取锁成功    第一次拿到锁才是1 
SETNX product:10001 true          //返回0代表获取锁失败

DEL product:10001                 //执行完业务释放锁
SET product:10001 true ex 10 nx   //防止程序意外终止导致死锁
```


### Hash数据结构 (hget,hset,hgetall)
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
* Hash常用操作
```
HSET key field value                        //存储一个哈希表key的键值
HSETNX key field value                      //存储一个不存在的哈希表key的键值
HMSET key field value[field value...]       //在一个哈希表key中存储多个键值对
HGET key field                              //获取哈希表key对应的field键值
HMGET key field [field...]                  //批量获取哈希表key中多个field键值
HDEL key field [field...]                   //删除哈希表key中的field键值
HLEN key                                    //返回哈希表key中所有的键值
HGETALL key                                 //返回哈希表key中所有的键值

HINCRBY key field increment                 //为哈希表key中的field键的值加上增量increment  
```
### Hash应用场景
* 对象缓存
```
//一个id对应一个key
HSET user:1 name roy balance 1888
HMGET user:1 name balance 1888

//一个user key 包含所有user_id  （会出现大key的问题）
HSET user 1:name roy 1:blance 1888
HMGET user 1:name 1：balance
```
* 电商购物车  (大型的电商就不能使用这种结构)
1)用户id为key
2)商品id为field
3)商品数量为value
```
//购物车操作
hset cart:1001 10088 1       // 1) 添加商品
hincrby cart:1001 10088 1    // 2) 增加数量
hlen cart:1001               // 3) 商品总数
hdel cart:1001 10088         // 4) 删除商品
hgetall cart:1001            // 5) 获取购物车所有商品        
```      

### Hash结构优缺点
* 优点
1) 同类数据归类整合存储，方便数据管理
2) 相比string操作消化内存与cpu更小
3) 相比stting存储更节省空间
* 缺点
1) 过期功能不能使用在field上，只能用在key上
2) Redis集群架构下不适合大规模使用


### List数据结构  ( lpush,rpush,lpop,rpop,lrange )
```
list 就是链表 关注列表，粉丝列表，消息列表等功能都可以用 Redis 的 list 结构来实现。

Redis list 的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。

lrange 命令，就是从某个元素开始读取多少个元素，可以基于 list 实现分页查询，这个很棒的一个功能，基于 Redis 实现简单的高性能分页。
```
* List 常用操作
```
LPUSH key value[value...]       //将一个或多个值value插入到key列表的表头(最左边)
RPUSH key value[value...]       //将一个或多个值value插入到key列表的表尾(最右边)
LPOP key                        //移除并返回key列表的头元素
RPOP key                        //移除并返回key列表的尾元素
LRANGE key start stop           //返回列表key中指定区间内的元素，区间以偏移量start和stop值定
BLPOP key [key...] timeout      //从key列表表头弹出一个元素，若列表中没有元素，阻塞等待 timeout秒，如果timeout=0，一直阻塞等待
BRPOP key [key...] timeout      //从key列表表尾弹出一个元素，若列表中没有元素，阻塞等待timeout秒，如果timeout=-，一直阻塞等待 
```
### List应用场景
* 常用数据结构
Stack(栈) = LPUSH + LPOP
Queue(队列) = LPUSH + RPOP
Blocking MQ(阻塞队列) = LPUSH + BRPOP
* 常见应用场景
视频列表、签到列表
排队机
简化版的MQ

### List类型注意点
1) 一个list的容量是2的32次方减1个元素，大概40多亿，但是在应用是，要注意大key的问题
2) list的底层是一个双向链表，对双端的操作性能很高，但是通过索引下表直接操作某一个中间节点的性能就会比较低。
   


### Set数据结构  （sadd,spop,smembers,sunion）
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
