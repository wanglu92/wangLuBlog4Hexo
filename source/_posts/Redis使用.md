---
title: Redis使用
date: 2021-02-05 08:55:01
tags:
---

 ## NoSql概述

为什么要有NoSql？

> 用户个人信息，社交网络，地理位置等，用户自己产生的数据，用户日志等可以很好的使用NoSql来解决。

大数据一般的数据库没有办法分析处理！

> 1. 单机MySql年代
>    + 如果数据量太大，一个机器放不下了
>    + 数据的索引，一个机器内存放不下
>    + 访问量大，一个服务器承受不了
> 2. Memcached缓存 + MySql + 垂直拆分（读写分离）
>    + 网站80%都是都是在查询，使用缓存减少数据库压力
> 3. 分库分表 + 水平拆分（MySql集群、数据分片）
>    + MyISAM引擎：表锁
>    + Innodb引擎：行锁
>    + 使用分库分表来解决写的压力
>    + MySql推出了表分区和集群
>    + 现在MySql等关系型数据库不够使用
>    + 有专门的数据库存储大文件、博客、图片等，MySql效率变高
> 4. 目前基本的互联网项目

什么是NoSql

> not only sql 泛指非关系型数据库
>
> 关系型数据库：使用行和列

NoSql的特点

> 1. 方便扩展（数据之间没有关系，好扩展）
> 2. 大数据量高性能（Redis一秒可以写8万次，读11万次，NoSql缓存记录，是一种细粒度的缓存，性能会比较高！）
> 3. 数据类型是多样型的！（不需要事先设计数据库，随取随用！如果是数据库量是十分大的表，很多人无法设计）
> 4. 传统的RDBMS关系型数据库和NoSql
>    + 结构化组织
>    + SQL
>    + 数据和关系都存在单独的表中
>    + 操作，数据定义语言
>    + 严格的一致性
>    + 基础的事务
>    + Nosql不仅仅是数据
>    + 没有固定的查询语言
>    + 键值对存储，列存储，文档存储，图形数据库
>    + 最终一致性
>    + CAP定理和BASE理论异地多活
>    + 高性能，高可用，高可扩？？？

3V+3高？？？（大数据）hadoop？？？

> 1. 海量Volume
> 2. 多样Variety
> 3. 实时Velocity
> 4. 高并发
> 5. 高可拓（水平拆分，可以随时拆分服务器解决）
> 6. 高性能

公司中使用：NoSql + RDBMS一起使用

## NoSql的四大分类

KV键值对：

+ 新浪：Redis
+ 美团：Redis + Tair
+ 阿里、百度：Redis + memcache

文档型数据库（bson格式和json一样）

+ MongDB（一般必须掌握）
  + 基于分布式文件存储的数据库，用C++编写
  + 主要用来处理大量的文档
  + MongoDB是一个介于关系型数据库和非关系型数据之间的产品。
  + NoSql关系型数据库中功能最丰富，最像关系型数据库的
+ ConthDB：国外的

列存储数据库

+ HBase
+ 分布式文件系统

图关系数据库

+ 用来存储关系，朋友圈，社交网络，广告推广。
+ Neo4j，InfoGrid

## Redis是什么

Redis（Remote Dictionary Server )，即远程字典服务。

Redis能做什么

> 1. 内存存储、持久化，内存中断电即失，持久化很重要。持久化技术 rdb、aof https://juejin.cn/post/6844903939339452430
> 2. 效率高，可以用于高速缓存
> 3. 可以发布订阅系统
> 4. 地图信息分析
> 5. 计时器、计数器（浏览量等）

Redis特性

> 1. 多样的数据类型
> 2. 持久化
> 3. 集群
> 4. 事务

Redis配置

> 1. 官网
> 2. 下载地址

## Redis安装

测试连接

```
ping -> pong
```

## redis-benchmar性能测试工具

## Redis基础知识说明

```
默认有16个数据库
默认使用0数据库
可以通过 select 数据库号 来切换
```

```
key * 查看所有key
flushdb 清除当前数据库
flushall 清空全部数据库
```

```
Redis是单线程
Redis是基于内存操作的，CPU不是Redis的性能瓶颈，Redis的性能瓶颈是根据机器的内存和网络带宽。
Redis为什么是单线程还这么快
1. 误区高性能的服务器一定是多线程的?
2. 误区多线程一定比单线程效率高？多线程需要CPU上下文切换。
redis将全部数据放在内存中，所以说使用单线程效率最高，如果没有上下文切换效率就是最高的。
```

```
端口号 6379 由来
```

## Redis基本命令

> 五大基本数据类型

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://www.redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://www.redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://www.redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://www.redis.cn/topics/lru-cache.html)，[事务（transactions）](http://www.redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://www.redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://www.redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://www.redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。

> redis-key

```
keys *
EXISTS key_name
move key_name 数据库号
ttl key_name 看过期时间
EXPIRE age 10 设置过期时间
type name 判断key的类型
```

## String字符串

```
set key1 v1
get key1
APPEND key1 hello
STRLEN key1 查看key长度
INCR views 自增
DECR views 自减
INCRBY views 10
DECRBY views 5
GETRANGE key1 0 5 查看
GETRANGE key1 0 -1 查看全部
SETRANGE key1 0 *** 替换指定位置的字符串
SETEX key2 30 hello 设置key并设置过期时间
SETNX key2 30 不存在的时候设置（分布式时设置）
mset k1 v1 k2 v2 k3 v3 批量设置 
mget k1 k2 k3
MSETNX k1 vv1 k4 v4 同时设置多个值，原子性操作
GETSET db redis 取出并设置
```

使用json格式可以保存一个对象

String使用场景

1. 缓存
2. 锁
3. 统计数量

## List

基本数据类型，列表

所有的list命令都是已l开头的

```
LPUSH list one 插入头部
LRANGE list 0 -1 获取
LRANGE list 0 1
RPUSH list four 从右侧插入
LPOP list 移除第一个元素
RPOP list
LINDEX list 1 通过下标获取list中的某一个值
LLEN list 返回列表的长度
LREM list 1 haha 移除指定个数的value值
LTRIM list 1 2 截取指定的长度
RPOPLPUSH list mylist 移除列表中的元素加到新的列表中
LSET list 0 item 更改index为0的为item
EXISTS list 是否存在
LINSERT list BEFORE "a" "haha" 在指定位置前或后插入
```

list实际上是一个链表，left和right都可以插入值

如果key不存在可以创建新的链表

消息队列

## set

```
sadd set hello
SMEMBERS set 查看set内容
SISMEMBER set hello 查看元素是否在set中
SCARD set 看集合中的个数
SREM set hello 移除指定元素
SRANDMEMBER set 2 随机获取set中指定个数的元素
SPOP set 随机删除一个元素
SMOVE myset myset2 hello 将一个集合转移到另外的一个集合中
SDIFF seta setb 两个集合的差集
SINTER seta setb 两个集合的交集
SUNION seta setb 两个集合的并集

```

使用：共同关注

## hash

```
hset my name wang age 3
hget my age
HMSET my name lu 覆盖值
HMGET my name age
HGETALL my 获取全部的值
HDEL my name 删除指定的值
HLEN my 查看hash中有多少值
HEXISTS my name 判断是否有指定的值
HKEYS my 查看所有的字段
HVALS my
HINCRBY my age 1 指定字段增加
HSETNX my name wang 不存在可以设置
```

hash的应用：存储变更的数据

更适合对象的存储

## Zset（有序集合）

在set的基础上增加了一个值，优先级

```
zadd zset 1 one 添加
ZRANGE zset 0 -1 获取
ZRANGEBYSCORE zset -inf +inf 按照socre排序
ZRANGEBYSCORE zset -inf +inf withscores 带socres排序显示
ZREVRANGEBYSCORE zset +inf -inf withscores 从小到大排列
ZREM zset one 移除
ZCARD zset 查看数量
ZCOUNT zset 1 2 获取指定范围的数量
```

set可以存储需要排序的

排行榜应用实现获取Top10

## geospatial地理位置

朋友的定位，附近的人，打车距离计算？

```
GEOADD
GEODIST 获取指定位置的经度维度
GEOHASH 将二维的经纬度转换为hash值
GEOPOS 查看两个人直接的距离
GEORADIUS 已给定的经度纬度为中心查到半径范围内的
GEORADIUSBYMEMBER 已某个给定地点查询半径范围
```

底层的实现原理是Zset，可以使用Zset命令来实现geo

可以使用zset命令来操作geo

## Hyperloglog基数统计

什么是基数-一个集合中不重复的元素？

一个网站一个人访问多次只计算一次

Hyperloglog占用的内存十分小12KB

会有0.81%的错误率

实现原理布隆过滤器？？？

```
PFADD mykey a b c d e f g h i j
PFCOUNT mykey
PFMERGE mykey3 mykey mykey2 合并
```

## bitmaps位图

位存储

统计用户信息：活跃、不活跃！登录、未登录！打卡，未打卡！

只有两个状态的可以使用。

```
SETBIT sign 0 0 第index为状态
GETBIT sign 1
BITCOUNT sign 查看为1的个数
```

## redis基本的事务操作

redis单条命令是保证原子性，但是事务不保证多条命令的原子性

> redis事务的本质：一组命令的集合！

一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序执行！

> redis事务没有隔离级别的概念

所有命令在事务中，并没有被直接执行！只有在发起执行的时候才会执行

> redis的事务：一次性、顺序性、排他性
>
> + 开启事务（MULTI）
> + 命令入队（需要执行的命令）
> + 执行事务（EXEC）
> + 放弃事务（DISCARD）

> 1. 错误的命令，类似编译的问题，所有的命令都不会执行
> 2. 如果运行的时候逻辑错误导致错误，其他命令会正常执行，所以没有原子性

## 监控（面试）

悲观锁：

+ 无论做什么都加锁。

乐观锁：

+ 默认拿数据时别人不会修改，更新数据的时候判断数据是否有人修改过。
+ 获取version
+ 比较version

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> DECRBY money 20
QUEUED
127.0.0.1:6379> INCRBY out 20
QUEUED
127.0.0.1:6379> EXEC
1) (integer) 80
2) (integer) 20
```

开启redis事务之后，如果有其他线程修改了watch监视的值，multi事务会执行失败。

```
unwatch 放弃监视
```

## Jedis

使用java来操作Redis

> jedis是redis官方推荐的java连接开发工具！

## SpringBoot整合

SpringBoot操作数据：spring-data jpa jdbc mongodb redis

SpringData和SpringBoot齐名的项目

SpringBoot2.x之后，原来使用的jedis被替换为lettuce

jedis：采用的是直接连接，多个线程操作的话是不安全的。需要使用jedis pool连接池解决。BIO模式。

lettuce：采用netty，实例可以再多个线程中共享，不存在线程不安全的问题，可以减少线程数量。NIO模式。

## redis配置文件

启动的时候就需要redis.conf

> NETWORK 网络配置

```xml
NETWORK 网络配置
bind 127.0.0.1 ::1 绑定ip
protected-mode yes 保护模式
port 6379 端口

GENERAL 通用
daemonize yes 默认是no，需要改为yes后台
pidfile /var/run/redis_6379.pid 如果以后台方式运行，就需要指定一个pid文件
loglevel notice 日志级别
logfile "" 输出的文件名
databases 16 数据库的数量

SNAPSHOTTING 快照 -- redis是内存数据，数据断电即失
save 900 1 持久化规则
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes 持久化失败后是否继续工作
rdbcompression yes 是否压缩rdb文件
rdbchecksum yes 保存rdb文件的时候是否校验rdb文件
dir /usr/local/var/db/redis/ rdb保存的文件夹

REPLICATION 主从配置 --  Master 主机  Replica 从机
replicaof <masterip> <masterport> 配置主机信息

SECURITY 安全
requirepass foobared 设置密码

CLIENTS 客户端

MEMORY MANAGEMENT 内存配置
maxmemory <bytes> 最大内存数
maxmemory-policy noeviction 达到最大内存后的策略
  # volatile-lru -> Evict（驱逐） using approximated（接近的） LRU, only keys with an expire set.
	# allkeys-lru -> Evict any key using approximated LRU.
	# volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
	# allkeys-lfu -> Evict any key using approximated LFU.
	# volatile-random -> Remove a random key having an expire set.
	# allkeys-random -> Remove a random key, any key.
	# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
	# noeviction -> Don't evict anything, just return an error on write operations.
  
APPEND ONLY MODE 附加模式
appendonly no 默认不开启
appendfilename "appendonly.aof" aof持久化文件名
appendfsync everysec 每秒执行一次
```

## redis持久化（面试）

> RDB（redis database）：单独创建一个子进程来持久化快照。将数据写入到临时文件中。保存到dump.rdb。因为是单独fork的子进程，所以效率较高。
>
> 触发机制：
>
> 1. sava的规则满足
> 2. 执行flushall
> 3. 退出redis
>
> 会自动生成dump.rdb文件
>
> 如何恢复rdb文件
>
> 1. 只需要将rdb文件放入到redis的启动目录就可以。
> 2. 目录 /usr/local/bin 启动目录下？
>
> 优点：
>
> 1. 大规模的数据恢复！
> 2. 如果对数据的完整性要求不高。
>
> 缺点：
>
> 1. 需要一定的时间间隔进行操作，如果redis宕机，最后修改的数据就没有了
> 2. 需要fork子进程，会消耗一些内存。
>
> 如果是在生产环境，会将dump.rdb备份



> AOF (append only file)：将所有操作的命令都记录下来，history，回复的时候就将这个文件全部执行一遍。只记录写的操作。保存的文件是appendonly.aof
>
> 默认是不开启的，需要手动进行配置。
>
> 可以通过 redis-check - aof 自动修复aof文件？
>
> 优点：
>
> 1. 每秒修改同步，文件完整性会更好
> 2. 每秒同步一次，可能会丢失数据
> 3. 从不同步，效率高
>
> 缺点：
>
> 1. 相对rdb文件来说，aof远大于rdb。
> 2. aof运行是同步运行，每次都要IO，所以效率比较慢。
>
> rewrite 重写规则
>
> 如果aof文件大于配置中的规则，太大了，fork一个进程，将文件进行重写，来缩小文件大小。、

在主从复制中，rdb就是从机使用的。

## redis订阅发布

redis发布订阅（pub/sub）是一种消息通信模式。

微信、微博、关注系统

1. 消息发送者
2. 频道
3. 消息订阅者

使用场景：

1. 实时消息系统
2. 实时聊天
3. 订阅，关注系统

稍微复杂的场景使用消息中间件MQ来实现

## redis集群环境搭建

> redis主从复制
>
> 将一台master/leader节点的数据复制到所有的从机上。
>
> 数据的复制都是单向的。
>
> 通过写三个配置文件该端口号就可以启动不同的。

> 作用：
>
> 1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式
> 2. 故障恢复：当主节点出现问题，可以由从节点提供服务，实现快速的故障恢复，实际上是一种服务的冗余
> 3. 负载均衡：在中从复制基础上，配合读写分离，可以由主节点提供写服务，从节点提供读服务，分担服务器压力。
> 4. 高可用（集群）：主从复制是哨兵和集群能够实施的基础，因此说主从复制是redis高可用的基础。

单台redis最大的内存不应该超过20G

在公司中，主从复制是必须的，集群模式一主而从。

> 环境配置

只配置从库，不用配置主库。

```
info replication 查看集群配置信息
role:master
connected_slaves:0
master_replid:dbd9200404d271c52925388668881c443e5db6a7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

> 修改信息
>
> 1. 端口号
> 2. pid名字
> 3. log文件名字
> 4. dump.rdb名字

> 默认情况下，每个redis服务都是主节点
>
> 一般情况下只需要配置从机就可以
>
> ```
> SLAVEOF localhost 6379 
> 
> # Replication
> role:slave
> master_host:localhost
> master_port:6379
> master_link_status:up
> master_last_io_seconds_ago:0
> master_sync_in_progress:0
> slave_repl_offset:28
> slave_priority:100
> slave_read_only:1
> connected_slaves:0
> master_replid:215e7b4e5a421b5f28fc9dcac59709acfffc4e6a
> master_replid2:0000000000000000000000000000000000000000
> master_repl_offset:28
> second_repl_offset:-1
> repl_backlog_active:1
> repl_backlog_size:1048576
> repl_backlog_first_byte_of
> 
> # Replication
> role:master
> connected_slaves:1 #多了从机的配置
> slave0:ip=127.0.0.1,port=6380,state=online,offset=98,lag=1
> master_replid:215e7b4e5a421b5f28fc9dcac59709acfffc4e6a
> master_replid2:0000000000000000000000000000000000000000
> master_repl_offset:112
> second_repl_offset:-1
> repl_backlog_active:1
> repl_backlog_size:1048576
> repl_backlog_first_byte_offset:1
> repl_backlog_histlen:112
> ```

> 细节：
>
> 从机只能读不能写。

> 复制原理
>
> slave启动成功连接到master之后，会发送一个sync之后。
>
> master会将所有的数据持久化，master将整个数据文件到slave，并完成一次同步。
>
> 全量复制
>
> 增量复制

> 集群配置可以多种形式：可以层层连接

```
手动配置主节点
SLAVEOF no one 
```

## 哨兵模式 自动选取主节点（面试）

sentinel

能够监控主机是否有故障，如果故障将自动将从库转换为主库。

单独的进程，通过发送命令判断主机是否健康。

多哨兵模式，哨兵直接也要互相监控。

```
redis-sentinel 命令
```

仅仅当一个哨兵发现主服务器不可用时，并不会马上进行选举，这个过程称为**主管下线**，需要后面的哨兵也检测到主服务不可用，并且达到一定数量后，哨兵直接回进行一次投票，投票的结果由一个哨兵发起，进行failover故障转移操作。切换成功后，会通过发布订阅模式，让哥哥哨兵把自己监控的从服务器切换为主机，这个过程称为**客观下线**.

1. 哨兵需要配置文件 sentinel.conf

   ```
   # sentinel monitor 被监控的名称 host port 1主机挂了slave投票让谁接替主机，票数最多称为主机。
   sentinel monitor myredis 127.0.0.1 6379 1
   ```

2. 启动哨兵
3. 如果master节点断开，这个时候会从从机中根据投票算法决定出一个新的主机。

> 优点：
>
> 1. 哨兵集群，基于主从复制，所有主从配置的优点都有。
> 2. 主动可以切换，故障可以转移，系统的可用性更好
> 3. 哨兵模式就是主从模式的升级，手动到自动，更加健壮。
>
> 缺点：
>
> 1. redis不好在线扩容，如果集群容量一旦达到上线，在线扩容就十分麻烦
> 2. 实现哨兵模式的配置其实是很麻烦，里面有跟多选择。

```
哨兵模式的全部配置
如果有哨兵集群需要配置每个哨兵的端口
定义工作目录
默认主机的节点
配置主机的密码
配置默认的延迟操作
故障转移的操作
可以设置默认的转移时间
出现问题可以执行通知脚本
```

## 缓存穿透和雪崩（面试 工作）

服务的高可用。

> 缓存穿透
>
> 用户查询数据，但是缓存中没有，数据库中也没有，查询透过缓存直接到数据库，数据库承受不了压力。
>
> 解决方案：
>
> 1. 布隆过滤器
> 2. 缓存空对象
>    1. 需要存储很多空值，浪费存储空间
>    2. 空值设置了过期时间，存在窗口时间。如果库中有数据了，会造成数据不一致。

> 缓存击穿
>
> 非常热的点，在不停抗大量的并发。在很短的时间内大量的请求到数据库中，导致数据库不可用。
>
> 解决方案：
>
> 1. 设置热点数据永不过期
> 2. 加互斥锁，只允许一个线程去查询数据库，其余的进行等待。

> 缓存雪崩
>
> 在一定时间段，缓存集中失效，redis宕机。
>
> 1. 服务降级，限流降级，在流量高峰停掉一些服务，保证主要的服务。
> 2. 多做几个集群，异地多活。
> 3. 数据预热