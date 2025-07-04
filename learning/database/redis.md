# Redis学习文档（面试方向）

## 1. Redis有几种数据结构，底层分别是什么

### 1.1 String（字符串）

Redis的字符串是动态字符串，是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配。

常见操作：

* `SET key value`：设置指定 key 的值
* `GET key`：获取指定 key 的值
* `DEL key`：删除指定 key 的值
* `INCR key`：将 key 中储存的数字值增一
* `DECR key`：将 key 中储存的数字值减一

内部编码：

* `int`：如果字符串的值是整数，且可以用`long`类型表示，那么就用`int`编码。
* `embstr`：如果字符串的值是字符串，且长度小于等于39字节，那么就用`embstr`编码。
* `raw`：如果字符串的值是字符串，且长度大于39字节，那么就用`raw`编码。

应用场景：

* 缓存：字符串类型是最常用的数据结构，通常用来缓存数据。
* 计数器：字符串类型可以用来实现计数器，例如访问次数、点赞数等。
* 分布式锁：字符串类型可以用来实现分布式锁，例如使用 SETNX 命令来实现。
* 限流：字符串类型可以用来实现限流，例如使用 INCR 命令来实现。
* 位存储：字符串类型可以用来实现位存储，例如使用 SETBIT 命令来实现。

### 1.2 Hash（哈希）

Redis hash 是一个键值对集合。

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

常见操作：

* `HSET key field value`：将哈希表 key 中的字段 field 的值设为 value 。
* `HGET key field`：获取存储在哈希表中指定字段的值。
* `HGETALL key`：获取在哈希表中指定 key 的所有字段和值。
* `HDEL key field`：删除哈希表 key 中的一个或多个指定字段。

内部编码：

* `ziplist`：当哈希类型元素个数小于`hash-max-ziplist-entries`配置（默认512个），同时所有值都小于`hash-max-ziplist-value`配置（默认64字节）时，Redis会使用ziplist作为哈希的内部实现。
* `hashtable`：当哈希类型无法满足ziplist的条件时，Redis会使用hashtable作为哈希的内部实现。

应用场景：

* 缓存：哈希类型可以用来缓存对象，例如用户信息、商品信息等。
* 计数器：哈希类型可以用来实现计数器，例如统计用户的点赞数、评论数等。
* 存储对象：哈希类型可以用来存储对象，例如用户对象、订单对象等。

## 1.3 List（列表）

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

常见操作：

* `LPUSH key value`：将一个或多个值插入到列表头部。
* `RPUSH key value`：将一个或多个值插入到列表尾部。
* `LPOP key`：移除并返回列表的第一个元素。
* `RPOP key`：移除并返回列表的最后一个元素。
* `LRANGE key start stop`：返回列表中指定区间内的元素。

内部编码：

* `ziplist`：当列表的元素个数小于`list-max-ziplist-entries`配置（默认512个），同时列表中每个元素的值都小于`list-max-ziplist-value`配置（默认64字节）时，Redis会使用ziplist作为列表的内部实现。
* `linkedlist`：当列表类型无法满足ziplist的条件时，Redis会使用linkedlist作为列表的内部实现。

应用场景：

* 消息队列：列表类型可以用来实现消息队列，例如使用 LPUSH 和 RPOP 命令来实现生产者-消费者模型。
* 任务队列：列表类型可以用来实现任务队列，例如使用 LPUSH 和 RPOP 命令来实现异步任务处理。
* 最新列表：列表类型可以用来实现最新列表，例如记录用户最近的操作、最近的评论等。

## 1.4 Set（集合）

Redis的Set是string类型的无序集合。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

常见操作：

* `SADD key member`：将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。
* `SMEMBERS key`：返回集合中的所有成员。
* `SISMEMBER key member`：判断 member 元素是否是集合 key 的成员。
* `SCARD key`：返回集合的成员数。
* `SRANDMEMBER key count`：返回集合中的 count 个随机元素。
* `SPOP key`：移除并返回集合中的一个随机元素。
* `SREM key member`：移除集合 key 中的一个或多个 member 元素，不存在的 member 元素会被忽略。

内部编码：

* `intset`：当集合中的元素都是整数且元素个数小于`set-max-intset-entries`配置（默认512个）时，Redis会使用intset作为集合的内部实现。
* `hashtable`：当集合类型无法满足intset的条件时，Redis会使用hashtable作为集合的内部实现。

应用场景：

* 去重：集合类型可以用来实现去重，例如统计用户的访问IP、统计页面的UV等。
* 标签：集合类型可以用来实现标签，例如用户的标签、商品的标签等。
* 好友关系：集合类型可以用来实现好友关系，例如用户的好友列表、用户的共同好友等。

## 1.5 Zset（有序集合）

Redis有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。

常见操作：
* `ZADD key score member`：将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
* `ZRANGE key start stop`：返回有序集 key 中，下标在 start 到 stop 之间的元素。
* `ZSCORE key member`：返回有序集 key 中，成员 member 的 score 值。
* `ZREM key member`：移除有序集 key 中的一个或多个成员，不存在的成员将被忽略。
* `ZRANGEBYSCORE key min max`：返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。
* `ZINCRBY key increment member`：为有序集 key 的成员 member 的 score 值加上增量 increment 。
* `ZCOUNT key min max`：返回有序集 key 中， score 值在 min 和 max 之间(包括等于 min 或 max )的成员的数量。
* `ZRANK key member`：返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增(从小到大)顺序排列。
* `ZREVRANK key member`：返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递减(从大到小)顺序排列。

内部编码：

* `ziplist`：当有序集合的元素个数小于`zset-max-ziplist-entries`配置（默认128个），同时每个元素的值都小于`zset-max-ziplist-value`配置（默认64字节）时，Redis会使用ziplist作为有序集合的内部实现。
* `skiplist + hashtable`：当有序集合类型无法满足ziplist的条件时，Redis会使用skiplist作为有序集合的内部实现。

常见场景：
* 排行榜：有序集合可以用来存储用户的排名，例如根据用户的积分排名、根据用户的等级排名等。
* 延迟队列：有序集合可以用来存储任务的延迟时间，当任务的延迟时间到达时，可以从有序集合中取出任务进行执行。

## 2. Redis的三种特殊数据类型

### 2.1 位图（bitmap）

位图（bitmap）是一种特殊的数据结构，它可以用来存储多个位（bit），每个位只能是0或1。

位图的应用场景：

* 统计用户活跃情况：位图可以用来记录用户的活跃情况，例如每天用户是否登录、是否签到等。
* 统计用户在线情况：位图可以用来记录用户的在线情况，例如用户是否在线、用户的在线时长等。
* 统计用户权限：位图可以用来记录用户的权限，例如用户是否有管理员权限、用户是否有读写权限等。

### 2.2  HyperLogLog

HyperLogLog 是一种用来统计基数（不重复元素的数量）的数据结构，它的特点是占用的内存空间固定，并且统计的误差比较小。

HyperLogLog 的应用场景：

* 统计 UV：HyperLogLog 可以用来统计网站的 UV（Unique Visitor），即独立访客数。
* 统计 PV：HyperLogLog 可以用来统计网站的 PV（Page View），即页面访问量。
* 统计在线用户数：HyperLogLog 可以用来统计在线用户数，即当前在线的用户数。
* 统计在线 IP 数：HyperLogLog 可以用来统计在线 IP 数，即当前访问网站的 IP 数量。

### 2.3 地理空间（geospatial）

geospatial 是一种特殊的数据结构，它可以用来存储地理位置信息，并支持根据地理位置信息进行查询。

geospatial 的应用场景：

* 定位服务：geospatial 可以用来实现定位服务，例如根据用户的地理位置信息，推荐附近的商家、商品等。
* 范围查询：geospatial 可以用来实现范围查询，例如根据用户的地理位置信息，查询一定范围内的商家、商品等。
* 附近的人：geospatial 可以用来实现附近的人功能，例如根据用户的地理位置信息，查询一定范围内的其他用户等。

## 3. Redis有几种持久化类型？什么实现的？

### 3.1 RDB 持久化 (Redis DataBase)

RDB 持久化是指将 Redis 在内存中的数据定时 dump 到磁盘上，以实现数据的持久化。

RDB 持久化的实现：

* `SAVE` 命令：当客户端发送 SAVE 命令时，Redis 会阻塞客户端，直到 RDB 文件创建完成。
* `BGSAVE` 命令：当客户端发送 BGSAVE 命令时，Redis 会创建一个子进程，子进程负责将数据 dump 到磁盘上，父进程继续处理客户端请求。

### 3.2 AOF 持久化 (Append Only File)

AOF 持久化是指将 Redis 的写命令追加到文件中，以实现数据的持久化。

AOF 持久化的实现：

* `appendonly` 配置：当 `appendonly` 配置为 yes 时，Redis 会开启 AOF 持久化。
* `appendfsync` 配置：`appendfsync` 配置决定了 AOF 持久化的 fsync 策略，有三种策略：always、everysec、no。always 表示每次写命令都执行 fsync，everysec 表示每秒执行一次 fsync，no 表示不执行 fsync，由操作系统决定何时执行。

### 3.3 混合持久化 (Redis 4.0 版本开始引入)

混合持久化是指 Redis 4.0 版本开始引入的一种持久化方式，它同时使用 RDB 和 AOF 两种持久化方式。

混合持久化的实现：

* `aof-use-rdb-preamble` 配置：当 `aof-use-rdb-preamble` 配置为 yes 时，Redis 会在 AOF 文件的开头添加 RDB 文件的内容，以实现数据的恢复。
* `aof-load-truncated` 配置：当 `aof-load-truncated` 配置为 yes 时，Redis 会在 AOF 文件加载时，忽略文件开头的 RDB 文件内容，只加载 AOF 文件的写命令。

## 4. Redis 集群模式

Redis 集群模式是指 Redis 提供的一种分布式数据库模式，它可以将数据分布在多个节点上，以实现数据的高可用、高性能。

## 4.1 集群模式的特点

* 数据分片：Redis 集群模式将数据分片（sharding）到多个节点上，每个节点负责存储一部分数据。
* 数据复制：Redis 集群模式使用异步复制（async replication）来实现数据的复制，每个节点都有一个或多个副本节点。
* 故障转移：当某个节点发生故障时，Redis 集群模式会自动将该节点的副本节点提升为主节点，以实现故障转移。
* 客户端分片：Redis 集群模式提供了一种客户端分片的方式，客户端可以根据键的哈希值，将请求路由到对应的节点上。

## 4.2 redis集群模式和kafka的数据复制的区别

* 数据分片：Redis 集群模式将数据分片到多个节点上，每个节点负责存储一部分数据。Kafka 则是将数据分片到多个分区上，每个分区负责存储一部分数据。（redis节点和kafka broker的区别，redis分片是在客户端完成的，而kafka分片在服务端完成）

* 数据复制：Redis 集群模式使用异步复制来实现数据的复制，每个节点都有一个或多个副本节点。Kafka 则是使用同步复制来实现数据的复制，每个分区都有一个或多个副本节点。

## 4.3 redis集群模式和kafka的区别

* 数据有序性：Redis 集群模式不保证数据的有序性。Kafka 则是保证数据的有序性，每个分区中的消息都是有序的。

* 数据可靠性：Redis 集群模式不保证数据的可靠性。Kafka 则是保证数据的可靠性，每个分区都有多个副本节点，保证数据的可靠性。

## 4.4 redis集群模式和kafka的区别总结

Redis 集群模式和 Kafka 都是分布式数据库，都可以用来存储数据。它们的区别在于数据分片、数据复制、数据有序性、数据可靠性等方面。在实际应用中，需要根据业务场景选择合适的分布式数据库。

| 区别 | Redis 集群模式 | Kafka |
| --- | --- | --- |
| 数据分片 | Redis 集群模式将数据分片到多个节点上，每个节点负责存储一部分数据。 | Kafka 则是将数据分片到多个分区上，每个分区负责存储一部分数据。 |
| 数据复制 | Redis 集群模式使用异步复制来实现数据的复制，每个节点都有一个或多个副本节点。 | Kafka 则是使用同步复制来实现数据的复制，每个分区都有一个或多个副本节点。 |
| 数据有序性 | Redis 集群模式不保证数据的有序性。 | Kafka 则是保证数据的有序性，每个分区中的消息都是有序的。 |
| 数据可靠性 | Redis 集群模式不保证数据的可靠性。 | Kafka 则是保证数据的可靠性，每个分区都有多个副本节点，保证数据的可靠性。 |

## 4.5 redis集群模式和kafka的应用场景对比

#### 4.5.1 redis集群模式的应用场景

* 缓存：Redis 集群模式可以用来作为缓存，存储一些热点数据，减少数据库的负载。
* 计数器：Redis 集群模式可以用来实现计数器，例如统计网站的访问量、用户注册量等。
* 排行榜：Redis 集群模式可以用来实现排行榜，例如根据用户的积分排名、根据商品的销售量排名等。
## 4.5.2 kafka的应用场景
* 日志收集：Kafka 可以用来收集日志数据，例如应用程序的日志、系统的日志等。
* 流处理：Kafka 可以用来进行流处理，例如实时数据分析、实时推荐等。
* 事件驱动：Kafka 可以用来实现事件驱动架构，例如订单系统、支付系统等。

## 5. 多线程下，redis如何保证线程安全？

在多线程环境下，为了保证 Redis 集群模式的线程安全，需要注意以下几点：

* 避免多个线程同时操作同一个 key。可以使用分布式锁来避免多个线程同时操作同一个 key。
* 使用 pipeline 批量操作。pipeline 可以将多个命令打包发送到服务器端，减少网络开销，提高效率。
* 使用 Lua 脚本。Lua 脚本是 Redis 的原子性执行脚本，可以保证多个命令的原子性执行。
* 使用主从复制。可以使用多个从节点，每个从节点负责读取数据，避免主节点成为瓶颈。

## 5.1 分布式锁？

分布式锁是指在分布式系统中，用于协调多个进程或线程对共享资源的访问的一种机制。在 Redis 集群模式中，可以使用 SETNX 命令来实现分布式锁。

使用 SETNX 命令实现分布式锁的基本流程如下：

1. 客户端向 Redis 服务器发送 SETNX 命令，请求获取锁。
2. Redis 服务器收到 SETNX 命令后，判断 key 是否存在。
   * 如果 key 不存在，则设置 key 的值为 value，并返回 1，表示获取锁成功。
   * 如果 key 已存在，则不设置 key 的值，并返回 0，表示获取锁失败。
3. 客户端获取到锁后，执行业务逻辑。
4. 客户端执行完业务逻辑后，向 Redis 服务器发送 DEL 命令，释放锁。
5. Redis 服务器收到 DEL 命令后，判断 key 是否存在。
   * 如果 key 不存在，则返回 0，表示释放锁成功。
   * 如果 key 存在，则删除 key，并返回 1，表示释放锁成功。
6. 客户端收到释放锁的响应后，判断是否释放锁成功。
   * 如果返回 1，表示释放锁成功。
   * 如果返回 0，表示释放锁失败，需要重新获取锁。

其他客户端在获取锁时，需要判断是否获取到锁。如果获取到锁，则执行业务逻辑；如果没有获取到锁，则需要等待锁释放。
（在没有获取到锁的时候，其他客户端在干嘛？）
其他客户端在没有获取到锁的时候，会在一个循环中不断地尝试获取锁，直到获取到锁或者超过了超时时间。如果超过了超时时间，还没有获取到锁，则需要放弃获取锁，避免浪费资源。


使用 EXPIRE 命令设置锁的过期时间。

EXPIRE 命令的作用是：设置 key 的过期时间。过期时间是指 key 从创建到过期的时间，单位是秒。

使用 EXPIRE 命令设置锁的过期时间的基本流程如下：

1. 客户端向 Redis 服务器发送 EXPIRE 命令，设置锁的过期时间。
2. Redis 服务器收到 EXPIRE 命令后，设置 key 的过期时间。
3. 当 key 过期时，Redis 服务器会自动删除 key。


## 6. Redis挂了怎么办？

通过Redis的持久化进行恢复（RDB和AOF），或者搭建主从复制，实现数据的高可用。


-----
REDIS 分布式锁实例
```
SET lock:user:123 <unique_id> NX EX 10
```
lock:user:123 表示锁的名称
<unique_id>: 表示锁的唯一标识。
NX: 表示只有当 key 不存在时，才会设置 key 的值。
EX 10: 表示设置 key 的过期时间为 10 秒。

释放时，使用lua保证原子性
```python
   r = redis.StrictRedis()
   lua = """
   if redis.call("get", KEYS[1]) == ARGV[1] then
      return redis.call("del", KEYS[1])
   else
      return 0
   end
   """
   r.eval(lua, 1, lock_key, unique_id)
```


### 7. REDIS安全，常见的攻击方式与预防？
* 1. Redis命令注入(Command Injection)
```python
user_input = "abc\r\nDEL key2\r\n"
cmd = f"SET user:{user_input} 123"
# 实际发送：
# SET user:abc\r\nDEL key2\r\n 123
```
预防方式：
- 使用官方的redis客户端
- 者对所有的用户输入做字符过滤
- 限制用户的权限，避免使用管理员权限的命令
- 禁用高危命令，如FLUSHALL、FLUSHDB等

* 2. Key 注入 / Key 冲突
```
key = "user:{}".format(user_id)
# 实际发送：
# SET user:123 123
# SET user:123\r\nDEL key2\r\n 123
```
预防方式：
- 对 key 进行编码，避免使用特殊字符
- 使用 Redis 提供的命名空间功能，将不同的业务数据划分到不同的命名空间中
- 限制 key 的长度，避免 key 过长

* 3. 缓存穿透
```
# 缓存穿透是指查询一个不存在的 key，导致每次请求都要去数据库查询，从而浪费数据库资源。
# 例如，查询一个不存在的用户 id，每次都要去数据库查询，浪费数据库资源。
```
预防方式：
- 缓存空对象：当数据库不存在该对象时，缓存一个空对象，设置较短的过期时间，避免缓存穿透。
- 布隆过滤器：使用布隆过滤器预先判断 key 是否存在，避免缓存穿透。

* 4. 缓存击穿
```
# 缓存击穿是指一个热点 key 在缓存中过期，但在同一时间有大量请求访问该 key，导致每次请求都要去数据库查询，从而浪费数据库资源。
# 例如，一个热门的商品 id，在缓存中过期，但在同一时间有大量请求访问该商品 id，每次都要去数据库查询，浪费数据库资源。
```
预防方式：
- 缓存热点数据：将热点数据缓存到 Redis 中，设置较长的过期时间，避免缓存击穿。
- 加互斥锁：当缓存失效时，只有一个线程去数据库查询数据，并将查询到的数据缓存到 Redis 中，其他线程等待锁释放后，从缓存中读取数据。


### 8. Redis如何实现权限控制？
* requirepass
- 可以在redis.conf中配置requirepass参数，设置密码
- 也可以在启动redis时，使用--requirepass参数，设置密码
- 客户端连接时，使用AUTH命令进行身份验证

* redis6.0+ 多用户 ACL （Access Control List）
- 可以在redis.conf中配置aclfile参数，设置acl文件路径
- 也可以在启动redis时，使用--aclfile参数，设置acl文件路径
- 客户端连接时，使用AUTH命令进行身份验证
```bash
# 添加一个只读用户，只能执行 GET
ACL SETUSER readonly on >readonly_password ~* +GET +MGET +EXISTS +TTL +PTTL -@all
```
| 参数                              | 含义                |
| ------------------------------- | ----------------- |
| `on`                            | 启用用户              |
| `>readonly_password`            | 设置密码（`>` 后跟密码）    |
| `~*`                            | 允许访问所有 key（可指定前缀） |
| `+GET +MGET +EXISTS +TTL +PTTL` | 允许执行这些只读命令        |
| `-@all`                         | 拒绝所有其它命令（否则默认全开放） |



### 9. 讲解一下Redis的RDB和AOF?
RDB（Redis Database File）也可以称作快照，是一种指定时间间隔，将整体数据结构写入磁盘的机制，默认文件类型就是.rdb
触发方式可以分成手动触发(save关键字)和自动触发两种形式
自动触发：save 90 10 # 90秒内，发生10次写操作通常情况下，我们可以编写多条规则共同作用
save 900 1     # 15分钟内有1次写
save 300 10    # 5分钟内有10次写
save 60 10000  # 1分钟内有10000次写
AOF（Append Only File）是一种追加日志持久化技术，把每一次写操作记录下来写在.aof文件中，本质类似mysql的binlog
通过编写appendfsync来进行控制
appendfsync always       # 每个命令都 fsync（慢，最安全）
appendfsync everysec     # 每秒 fsync 一次（默认，性能与安全兼顾）
appendfsync no           # 由 OS 自己决定何时写盘（性能高但不安全）

重写机制(AOF ReWrite)
AOF 重写机制是指在 AOF 持久化过程中，当 Redis 数据库中的数据发生了变化时，会将这些变化记录到 AOF 文件中。但是，随着时间的推移，AOF 文件会变得越来越大，因为 Redis 数据库中的数据会不断变化。为了避免 AOF 文件过大，Redis 会定期对 AOF 文件进行重写，即对 AOF 文件中的命令进行压缩，从而减少 AOF 文件的大小。
AOF 重写机制的触发条件同样可以分成手动和自动两种
手动触发：可以使用 BGREWRITEAOF 命令手动触发 AOF 重写机制。
自动触发条件：
auto-aof-rewrite-min-size 64mb
auto-aof-rewrite-percentage 100




Redis 的主从复制（Master-Slave Replication）是 Redis 构建**高可用（High Availability）**和**读写分离**的核心机制之一，也是很多面试中重点问的点。下面我们系统讲解：

---

# ✅ 一、什么是 Redis 主从复制？

> Redis 主从复制是一种让一个 Redis 实例（**主节点 master**）的数据被自动复制到一个或多个 Redis 实例（**从节点 slave**）的机制。

### 目的包括：

* 实现数据冗余，提高容错性
* 扩展读能力（读写分离）
* 准备故障切换（sentinel、cluster）

---

# ✅ 二、主从复制的基本原理

Redis 采用**异步复制（asynchronous replication）**，从节点会主动向主节点发送 `PSYNC` 请求，请求复制数据。

---

## 🔄 复制过程分为两个阶段：

### 🔹 1. 初次全量复制（Full Resynchronization）

> 适用于新从节点、主从断线重新连接、复制偏移不一致的场景。

流程如下：

```
slave → master: PSYNC ? -1
master 生成 RDB 快照 + 发送快照
master 同时将期间写入的命令缓存在复制缓冲区中
slave 加载 RDB + 回放缓冲区命令 → 数据一致
```

关键词：

* `RDB`: 主节点生成快照
* `replication backlog buffer`: 主节点维护的固定长度命令缓冲区（默认 1MB）

---

### 🔹 2. 增量复制（Partial Resynchronization）

> 如果主从连接短暂断开后恢复，且主节点还保留了断开期间的写命令缓冲，就可以增量复制，**只补丢的命令**，不用再发全量 RDB。

流程如下：

```
slave → master: PSYNC <replicationid> <offset>
master → slave: 只发送 offset 之后的写命令
```

---

# ✅ 三、主从复制命令示例

```bash
# 设置从节点（在 slave 机器上执行）
SLAVEOF <master_ip> <master_port>

# 取消主从关系
SLAVEOF NO ONE
```

---

# ✅ 四、主从复制结构图

```
       Write        Write
Client -----> Master Redis
               |
       Replication (异步)
              /  \
         Slave1  Slave2
         (Read)  (Read)
```

---

# ✅ 五、主从复制中的关键数据结构

| 名称               | 说明                 |
| ---------------- | ------------------ |
| `run_id`         | 主节点的唯一标识           |
| `replication ID` | PSYNC 协议使用的 ID     |
| `offset`         | 当前主从数据复制到的位置       |
| `backlog buffer` | 主节点的命令环形缓冲区，用于增量复制 |

---

# ✅ 六、主从复制的常见问题

### 1. ❓ 主从数据一致性如何保证？

* 异步复制，不保证强一致性
* 可配置为**半同步**（`min-slaves-to-write` 配置项）

### 2. ❓ 多个 slave 如何连接一个 master？

* 支持一个 master 对多个 slave，slave 之间不通信

### 3. ❓ Redis 是强一致吗？

* ❌ 不是，复制是异步，主挂时从的数据可能略有滞后

### 4. ❓ 从节点是否可以写？

* 默认不能写（`read-only`），但可以强制配置为可写（不推荐）

---

# ✅ 七、实际部署相关配置

```ini
# redis.conf 中的配置项
replicaof <master_ip> <master_port>
replica-read-only yes        # 从节点是否只读（默认 yes）
repl-backlog-size 1mb        # backlog 大小
repl-timeout 60              # 超时时间
```

---

# ✅ 面试高频追问问题（含答案）

| 问题               | 答案                                              |
| ---------------- | ----------------------------------------------- |
| 主从复制是同步还是异步？     | 默认异步，存在数据延迟                                     |
| 如何实现主从切换？        | 配合 Redis Sentinel 使用                            |
| 如何避免频繁全量复制？      | 启用 `repl-backlog-buffer` 支持增量同步                 |
| 主节点重启后，能否自动恢复复制？ | 可以，slave 会通过 PSYNC 重建连接                         |
| 复制延迟大怎么办？        | 调整 `repl-disable-tcp-nodelay`, 优化网络, 或用 cluster |

---

如果你希望我补充：

* Redis Sentinel + 主从复制 + 故障转移
* Redis Cluster 的复制模型（主从分片）
* Python 实现主从同步 demo

我可以继续帮你扩展。是否继续？
