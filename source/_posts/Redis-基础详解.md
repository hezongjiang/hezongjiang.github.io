---
title: Redis 基础详解
catalog: true
date: 2019-11-01 00:38:10
subtitle:
header-img: timg.jpeg
tags: 服务端开发
---
## 一、Redis 是什么

Redis 是一个使用 C 语言写成的，开源的、key-value 结构的、非关系型数据库。它支持存储的 value 类型相对更多，包括 String(字符串)、List(列表)、Set(集合)、Sorted Set(有序集合) 和 Hash(哈希)，而且这些操作都是原子性的。在此基础上，Redis 支持各种不同方式的排序。为了保证效率，数据都是缓存在内存中。Redis 可以周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

**使用 Redis 有哪些好处？**
1. 速度快，因为数据存在内存中；

2. 支持丰富数据类型，支持 string，list，set，sorted set，hash等；

3. 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行；

4. 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除；

## 二、单线程？

你可能听说过 Redis 是单线程的，那会不会很慢呢？

**为什么 Redis 是单线程的？**

Redis 的数据存储在内存中，如果数据全都在内存里，单线程的去操作就是效率最高的。

为什么呢？因为多线程的本质就是 CPU 模拟出来多个线程的情况，这种模拟出来的情况就有一个代价，就是上下文的切换，对于一个内存的系统来说，它没有上下文的切换就是效率最高的。因为上下文切换所花费的时间远大于直接从内存中读取数据所花费的时间。

Redis 用单个 CPU 绑定一块内存的数据，然后针对这块内存的数据进行多次读写的时候，都是在一个CPU上完成的，所以它是单线程处理这个事。在内存的情况下，这个方案就是最佳方案。

这里我们一直在强调的单线程，只是在处理我们的网络请求的时候只有一个线程来处理，一个正式的 Redis Server 运行的时候肯定是不止一个线程的，例如 Redis 进行持久化的时候会以子进程或者子线程的方式执行。



## 三、Redis 数据类型
**1、String**
String 数据结构是简单的 key-value 类型，value 其实不仅可以是 String，也可以是数字。需要注意是一个键值最大存储 512MB。
```
> set myName "redis"
OK
> get myName
redis
> set myName "memcache"
OK
> get myName
memcache
> set count 1
OK
> get count
1
> incr count
2
```

**2、Hash（哈希）**
Hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。 比如我们可以 Hash 数据结构来存储用户信息，商品信息等等。
```
> hmset lilei name "LiLei" age 26 title "Senior"
OK
> hget lilei age
26
> hget lilei title
Senior
> hset lilei title "Primary"
> hget lilei title
Primary
```

**3、List（列表）**
是 Redis 的简单的字符串列表。list 的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。

```
> lpush mylist aaa
(integer) 1
> lpush mylist bbb
(integer) 2
> rpush mylist ccc
(integer) 3
> llen mylist
(integer) 3
> lrange mylist 0 2
1) "bbb"
1) "aaa"
1) "ccc"
```

**4、Set**
Set 对外提供的功能与 list 类似是一个列表的功能，特殊之处在于 set 不允许重复元素，可以自动排重的，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。

在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis 可以非常方便的实现如共同关注、共同喜好、二度好友等功能。
```
> sadd myset 111
(integer) 1
> sadd myset 222
(integer) 1
> sadd myset 333
(integer) 1
> sadd myset 222
(integer) 0
> smembers myset
1) "111"
2) "222"
3) "333"
```

**5、Sorted Set**
与 set 相比，Sorted set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列。

举例： 在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息，适合使用 Sorted set 结构进行存储。
```
> zadd myzset 3 a
(integer) 1
> zadd myzset 1 b
(integer) 1
> zadd myzset 2 c
(integer) 1
> zadd myzset 2 c
(integer) 0
> zadd myzset 1 d
(integer) 1
> zrangebyscore myzset 0 3
1) "b"
2) "d"
3) "c"
4) "a"
```

## 四、Redis 应用
**例1： Redis 中有 1 亿个 key，其中有 10 万个 key 是已知的某个固定前缀，如何将找出这些 key。**

当问题是：如何找出某一前缀的 key。这里需要注意数据规模是多大，再结合实际场景去解决。

如果数据量小，可以使用 `keys` 命令来查询。

```
> keys key*
 1) "key9"
 2) "key3"
 3) "key1"
 4) "key2"
 5) "key5"
 6) "key8"
 7) "key7"
 8) "key6"
 9) "key"
10) "key4"
```

但是，如果数据量大，而 Redis 正在线上运行，使用 `keys` 命令很可能会阻塞 Redis，使其不能提供服务。
这时可以使用 `SCAN cursor [MATCH pattern] [COUNT count]` 命令。需要注意的是条命令返回的数量是不可控的，只能大概符合 count。
```
> scan 0 match key* count 5
1) "12"
2) 1) "key9"
   2) "key7"
   3) "key4"
> scan 12 match key* count 5
1) "14"
2) 1) "key8"
   2) "key6"
   3) "key5"
> scan 14 match key* count 5
1) "0"
2) 1) "key3"
   2) "key1"
   3) "key2"
   4) "key"
```
cursor 从 0 开始，这里第一个返回值为 cursor 用于下一次迭代，当 cursor 再次为 0 时，说明迭代完成。

**例2：如何通过 Redis 实现分布式锁**

首先想到的方法是使用 `SETNX key value`，如果 key 不存在，则创建并赋值，返回 1；若 key 已存在，则创建失败，返回 0。

当某一个线程通过 `SETNX key value` 设置成功后占用该资源，其他线程执行该命令就会设置失败，说明已经有线程占用该资源，通过这种方式来实现分布式锁。

```
> setnx mylock 11
(integer) 1
> setnx mylock 22
(integer) 0
> get mylock
"11"
```

同时，为了防止死锁，还需要设置 key 的过期时间 `EXPIRE key second`，设置 key 的过期时间，经过 second 秒后 key 会被删除。
```
> expire mylock 11
(integer) 1
```

虽然 Redis 中每个命令都满足原子性，但两个命令组合起来不满足了，例如若设置锁后，程序挂了，来不及设置过期时间，那么这个锁就无法释放了。

所以我们需要把上面两个命令结合起来的命令 `SET key value [EX seconds] [PX milliseconds] [NX|XX]`，其中 `NX` 是 key 不存在时设置成功，`XX` 是 key 存在时设置成功。例如：

```
> set mylock 123 ex 10 nx
OK
```

这里还需要注意，不要让大量的 key 在同一时间过期，因为删除大量的 key 很耗时，会出现卡顿现象。所以我们可以在设置 key 的过期时间时，加上一个随机值来避免。

**例3：使用 Redis 实现消息队列**

使用 List 作为队列，`RPUSH` 生产消息，`LPOP` 消费消息。
```
rpush testlist aaa
(integer) 1
> rpush testlist bbb
(integer) 2
> rpush testlist ccc
(integer) 3
> lpop testlist
"aaa"
> lpop testlist
"bbb"
> lpop testlist
"ccc"
```
但是这样有个缺点，就是只能一对一的消息通信，所以还可以使用 pub/sub 主题订阅者模式，发送者(pub)使用 `PUBLISH channel message` 发送消息，订阅者(sub)使用 `SUBSCRIBE channel` 接受消息，并且订阅者可以接收任意数量的 channel。

但是消息的发布是无状态的，无法保证可达性。

## 五、Redis 数据持久化存储
上面说了 Redis 是基于内存的数据库，一旦进程退出，数据就会丢失，所以我们需要它也可以把数据写到磁盘上，当 Redis 重启后，可以从磁盘中恢复数据。

Redis 提供了两种解决方案将内存中的数据保存到磁盘上。一种是 RDB 持久化，原理是将 Reids 在内存中的全部数据库记录定时 dump 到磁盘上的 RDB 持久化；另外一种是 AOF 持久化，原理是将 Reids 的操作日志以追加的方式写入文件。

**1、RDB(Redis DataBase)**

RDB 是 Redis 用来进行持久化的一种方式，是把当前内存中的数据集快照写入磁盘，也就是 Snapshot 快照所有数据。恢复时是将快照文件直接读到内存里。

RDB 存储的劣势：

1. RDB 方式数据没办法做到实时持久化，该方式是每间隔一段时间做一次备份。因为 bgsave 每次运行都要执行 fork 操作创建子进程，属于重量级操作，频繁执行成本过高。所以如果 Redis 意外挂掉，就会丢失最后一次快照后的所有修改；

2. RDB 是内存数据的全量同步，数据量大会由于 I/O 而严重影响性能；

2. RDB 文件使用特定二进制格式保存，Redis 版本演进过程中有多个格式的 RDB 版本，旧版本无法兼容新版的格式。

**2、AOF(Append Only File)**

 AOF 持久化：记录除了查询以外，所有变更数据库状态的指令，以 append 的形式追加保存到 AOF 文件中。AOF 文件通常会比 RDF 文件体积更大。

**3、RDB-AOF 混合模式**
混合模式是先使用 bgsave 以 RDB 形式将内存中的全部数据写入磁盘，之后当有新的数据时，再使用 AOF 的形式追加到文件中。