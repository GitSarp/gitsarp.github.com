### 缓存
数据量大，不怎么变的数据，如评论等
### redis
value支持的数据类型：
基本命令：
- set key value;get key

- ** mset key1 value1 … keyN valueN 一次设置多个key的值 **

- ** mget key1 key2 … keyN 一次获取多个key的值** 

- rename oldkey newkey    改名字

- dbsize 返回当前数据库的key数量

- keys pattern           返回匹配指定正则表达式的所有key，如**keys *返回所有key**

- type key                返回给定key的value类型

- exists key           测试指定key是否存在

- del key 

- **expire key n**      为key指定过期时间n秒

- **ttl key**                 返回key的剩余过期秒数

- **select db-index**         选择数据库，不能超过15

- move key db-index      将key从当前数据库移动到指定数据库

- **flushdb**                删除当前数据库的所有key

- flushall               删除所有数据库的所有key

- ** incr key 对key的值做加加操作，并返回新的值**

- **incrby key n 加n，返回新值**

- **decr key 同上，但做的是减减操作**

- **decrby key n 减n，返回新值 **

- append key value 给指定key的字符串追加value 

- substr key start end 返回截取过的key的字符串值 ，如substr key 0 4返回前五位


#### list操作指令

**lpush key string         在key对应list的头部添加字符串元素**

**lpop key                      从list的头部删除元素，并返回删除元素** 

**rpush key string         在key对应list的尾部添加字符串元素** 

**rpop key                 从list的尾部删除元素，并返回删除元素** 

**llen key 返回 key        对应list的长度，key不存在返回0，如果key对应类型不是list返回错误** 

**lrange key start end     返回指定区间内的元素，下标从0开始** 

**ltrim key start end         截取list，保留指定区间内元素** 

可以实现排行榜，将前n名的数据存储在list中

#### Set操作

最多包含2^23-1个元素

**sadd key member           添加一个string元素到key对应的set集合中，成功返回1，如果元素已经在集合中返回0，key对应的set不存在，则创建一个只包含member元素的集合。**

**srem key member [member]  从key对应set中移出给定元素，成功返回1**

**smove p1 p2 member        从p1移到p2**

**scard key                 返回set的元素个数**

**sismember key member      判断member是否在set中**

**sinter key1 key2...keyN   返回所有给定key的交集 **

**sunion key1 key2...keyN   返回所有给定key的并集 **

**sdiff key1 key2...keyN    返回在k1不在k2的元素集 **

**smembers key              返回key对应set的所有元素，结果是无序的** 

可以实现共同好友（交集）

#### SortSet

元素有权值，可以实现有序

**zadd key score member            添加元素到集合，元素在集合中存在则更新对应score** 

**zrem key member                  删除指定元素，1表示成功，如果元素不存在返回0 **

**zincrby key incr member          按照incr幅度增加对应member的score值，返回score值 **

**zrank key member                 返回指定元素在集合中的排名(下标)，集合中元素是按score从小到大排序的 zrevrank key member              同上，但是集合中元素是按score从大到小排序**
**zrange key start end             类似lrange操作是从集合中去指定区间的元素。返回的是有序的结果 **

**zrevrange key start end          同上，返回结果是按score逆序的 **

**zcard key                        返回集合中元素个数 zscore key element               返回给定元素对应的score **
**zremrangebyrank key min max      删除集合中排名在给定区间的元素(权值从小到大排序)** 

#### Hash操作
类似一条表记录
![](https://img-blog.csdn.net/20161128164051439?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**hset key field value             设置hash field为指定值，如果key不存在，则先创建 **

**hget key field                   获取指定的hash field **

**hmget key field1...fieldN        获取多个指定的hash field **

**hmset key field1 value1...fieldN  valueN       同时设置hash的多个field **

**hincrby key field integer        将指定的hash field加上给定值 **

**hexists key field                测试指定的field是否存在 **

**hdel key field                   删除指定的hash field  **

**hlen key                         返回指定hsah的field数量 **

**hkeys key                        返回hash的所有field **

**hvals key                        返回hash的所有value **

**hgetall key                      返回hsah的所有field和value** 