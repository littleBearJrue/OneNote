# redis面试要点

#### 1. redis基本数据结构有哪些？

答：redis的基本数据结构有：string，list，hash，set 和 zset

之外还有一些数据结构包括: Bitmap, GEO，HyperLogLog，RedisTimeSeries甚至可以自定义数据结构类型

这里分别介绍下各个数据结构的意义：

1. string：存储字符串类型，内部又SDS实现
2. List：双向循环列表，内部由双向链表和压缩列表实现
3. Hash:key-value的哈希表，内部由哈希表和压缩列表实现
4. zset：排序的不重复数据集合，内部由跳表和压缩列表实现
5. set:不重复数据集合，内部由数组和哈希表组成
6. BitMap:储存的是二进制数据
7. HyperLogLog:储存的是基数数据
8. GEO：存储的是经纬度



#### 2. redis的基础命令行有哪些？

答：

1. string：

| 操作命令                      | 命令作用                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| SET key value                 | 设置指定的key的值                                            |
| GET key                       | 获取指定key的值                                              |
| GETRANGE key start end        | 获取key中字符串的子串                                        |
| SETRANGE key offset value     | 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始 |
| GETSET key value              | 设置key的新值，并返回旧值                                    |
| MGET key,key...               | 获取多个key的值                                              |
| MSET key value, key value...  | 设置多个key的值                                              |
| STRLENG key                   | 返回key字符串的长度                                          |
| SETNX key value               | 只有key不存在时才设置成功                                    |
| SETEX key seconds value       | 设置key 对应的值在second                                     |
| MSETNX key value,key value... | 设置多个key value值，需所有的值都不存在才设置成功            |
| PSETEX key millseconds value  | 同setex，它以毫秒为单位                                      |
| INCR key                      | key值自增1                                                   |
| DECR key                      | key自减1                                                     |
| INCRBY key increment          | key增加到给定的增量值                                        |
| DECRBY key decrement          | key减少到给定的减量值                                        |
| APPEND key value              | 指定key的值追加值到value                                     |

2. List

   | 操作命令                        | 命令作用                                               |
   | ------------------------------- | ------------------------------------------------------ |
   | LPOP key                        | 移除并获取列表的第一个元素                             |
   | LPUSH key value,key value...    | 插入值到列表头部,可插入多个                            |
   | RPOP key                        | 移除并获取列表的最后一个元素                           |
   | RPUSH key value,key value...    | 插入值到列表的尾部，可插入多个                         |
   | BLPOP key timeout               | 移除并获取列表第一个元素，没有则阻塞等待，直到超时     |
   | BRPOP key timeout               | 移除并获取列表最后一个元素，没有则阻塞等待，直到超时   |
   | LINDEX key index                | 获取列表中索引的值                                     |
   | LLEN key                        | 获取列表长度                                           |
   | LINSERT key before\|after value | 在列表元素的前\|后插入                                 |
   | LRANGE key start end            | 获取列表指定范围内的元素                               |
   | LREM key count value            | 移除列表元素（指定多少个）                             |
   | LSET key index value            | 通过索引设置列表元素的值                               |
   | LTRIM key start end             | 修剪列表，保留列表指定区间的元素                       |
   | RPOPLPUSH source destination    | 移除列表最后一个元素，并将元素添加到另外一个列表中返回 |

3. Hash

   | 操作命令                         | 命令作用                                              |
   | -------------------------------- | ----------------------------------------------------- |
   | HSET key field value             | 设置哈希表中key的字段field的值                        |
   | HGET key field                   | 获取哈希表中key的字段field的值                        |
   | HDEL key field,field2...         | 删除一个，多个哈希表中字段的值                        |
   | HEXISTS key field                | 查看哈希表中的key中field字段是否存在                  |
   | HETALL key                       | 获取哈希表中key所有的字段和值                         |
   | HMGET key field,field2...        | 获取多个哈希表中key指定字段field的值                  |
   | HMSET key field,key2 field2...   | 设置哈希表中key多个field的值                          |
   | HLEN key                         | 获取哈希表中字段的数量                                |
   | HSETNX key field value           | 当字段field不存在时，设置哈希表字段的值               |
   | HKEYS key                        | 获取哈希表key中所有字段                               |
   | HVALS key                        | 获取哈希表key中的所有值                               |
   | HSCAN key cursor                 | 迭代获取哈希表中的键值对                              |
   | HINCRBY key field increment      | 为哈希表 key 中的指定字段的整数值加上增量 increment   |
   | HINCRBYFLOAT key field increment | 为哈希表 key 中的指定字段的浮点数值加上增量 increment |

4. sort

   | 操作命令                   | 命令作用                                        |
   | -------------------------- | ----------------------------------------------- |
   | SADD key value,value2      | 向集合添加一个或者多个成元素                    |
   | SCARD key                  | 获取集合的所有元素                              |
   | SDIFF key1 key2            | 返回第一个集合与其他集合之间的差异              |
   | SDIFFSTORE dest key1 key2  | 返回给定的属于偶偶集合的差集储存在dest集合中    |
   | SINTER key1 key2           | 返回指定所有集合的差集                          |
   | SINTERSTORE dest key1 key2 | 返回指定所有集合的差集存储到dest集合中          |
   | SISMENBER key value        | 判断value是否是几个key的元素                    |
   | SMEMBERS key               | 返回集合中的所有元素                            |
   | SMOVE source dest member   | 将member元素从source集合移动到desitnation集合中 |
   | SPOP key                   | 移除并返回集合中的一个随机元素                  |
   | SRANDMEMBER key [count]    | 返回集合中一个或者多个随机元素                  |
   | SREM key value value2      | 移除集合中一个或者多个元素                      |
   | SUNION key1 key2           | 返回所有集合中的                                |
   | SUNIONSTORE dest key1 key2 | 所有给定集合的并集存储在 destination 集合中     |
   | SSCAN key cursor           | 迭代集合中的元素                                |

5. zsort

   | 操作命令                                        | 命令作用                                                     |
   | ----------------------------------------------- | ------------------------------------------------------------ |
   | ZADD key score member,score2 member2            | 向有序集合中添加一个或者多个成员                             |
   | ZCARD key                                       | 获取有序集合的成员数                                         |
   | ZCOUNT key min max                              | 获取有序集合中指定score区间的成员数                          |
   | ZINCRBY key increment member                    | 向有序集合指定成员的score加上增量incrememt                   |
   | ZLEXCOUNT key min max                           | 在有序集合中计算指定字典区间内成员数量                       |
   | ZRANGE key start end[withscores]                | 通过索引区间返回有序集合指定区间内的成员                     |
   | ZRANGEBYSCORE key min max [WITHSCORES\] [LIMIT] | 通过分数返回有序集合指定区间内的成员                         |
   | ZRANK key member                                | 返回有序集合中指定成员的索引                                 |
   | ZREM key member member2                         | 移除有序集合中一个或者多个成员                               |
   | ZREMRANGEBYLEX key min max                      | 移除有序集合中给定的字典区间的所有成员                       |
   | ZREMRANGEBYRANK key start stop                  | 移除有序集合中给定的排名区间的所有成员                       |
   | ZREMRANGEBYSCORE key min max]                   | 移除有序集合中给定的分数区间的所有成员                       |
   | ZREVRANGE key start stop [WITHSCORES]           | 返回有序集中指定区间内的成员，通过索引，分数从高到低         |
   | ZREVRANGEBYSCORE key max min [WITHSCORES        | 返回有序集中指定分数区间内的成员，分数从高到低排序           |
   | ZREVRANK key member                             | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
   | ZSCORE key member                               | 返回有序集中，成员的分数值                                   |
   | ZUNIONSTORE destination numkeys key [key ...\]  | 计算给定的一个或多个有序集的并集，并存储在新的 key 中        |
   | ZSCAN key cursor [MATCH pattern\] [COUNT count] | 迭代有序集合中的元素（包括元素成员和元素分值）               |




#### 3. 为什么redis储存可以做到这么快？相当于传统的关系数据有什么优势？

答：Redis是基于内存的操作的key-value形式的非关系型数据库。之所以这么快速高效是因为：

1. 本身基于内存进行的操作，内存操作具备高效性
2. 归功于对数据结构的重新设计和优化
3. 采用IO多路复用的机制，使其在网络IO中能并发处理大量请求



#### 4. redis持久化的方式有哪些以及对其的理解？

答：Redis的持久化包括RDB（redis database）和AOF（append only file）

#### 5. rbd的持久化具体的流程是怎么样的？

答：一种是通过save命令在主线程中执行，会导致阻塞；另一种是通过bgsave fork一个子进程，专门用于RDB文件的写入；一般这些都可以在配置表中做好配置工作，可以设置规定时间内修改了多少次key值执行RDB写入的操作；在写入过程中主线程也要正常接收写操作，此时可以借助操作系统提供的写时复制（cow）,在执行快照的同时，正常处理写操作，主线程修改的数据会对应生成数据副本，子进程把副本写入RDB文件中。

#### 6. aof的持久化具体流程是怎么样的？

答：aof的持久化也可以通过配置的方式修改，配置表中提供appendfsync配置项，包括always,EverySec和no三种配置选项；其中always同步写回，命令执行完立即写入日志到磁盘中；EverySec则是每秒写回，命令执行优先写入缓冲区（消息队列）中，此时使用子线程从每秒从缓冲区（消息队列）中读取数据落入磁盘中

#### 7. aof的重写流程是怎么样进行的？

答： AOF重写的流程。

1. 执行AOF重写请求。
   如果当前进程正在执行bgsave操作，重写命令会等待bgsave执行完后再执行。
2. 父进程执行fork创建子进程。
3. fork操作完成后，主进程会继续响应其它命令。所有修改命令依然会写入到aof_buf中，并根据appendfsync策略持久化到AOF文件中。
4. 因fork操作运用的是写时复制技术，所以子进程只能共享fork操作时的内存数据，对于fork操作后，生成的数据，主进程会单独开辟一块aof_rewrite_buf保存。
5. 子进程根据内存快照，按照命令合并规则写入到新的AOF文件中。每次批量写入磁盘的数据量由aof-rewrite-incremental-fsync参数控制，默认为32M，避免单次刷盘数据过多造成硬盘阻塞。
6. 新AOF文件写入完成后，子进程发送信号给父进程，父进程更新统计信息。
7. 父进程将aof_rewrite_buf（AOF重写缓冲区）的数据写入到新的AOF文件中。
8. 使用新AOF文件替换老文件，完成AOF重写。

#### 8. 什么是主从复制？为什么要做主从复制？

答：

1. 主从复制，就是避免单机请求量过大或者单机宕机等引起的redis数据丢失等问题，额外开一个或者多个从服务，实现主从模式读写分离，主机只做写操作请求，从机只做读操作请求，继而分担服务器压力。

2. 引入主从复制的原因是，单机执行redis的读写操作，当读写请求量足够大时，单机性能很难满足要求，于是引入多个从服务，所有的读请求都从从服务中获取，将请求量分散在多台子服务器中，大大减少主机的请求压力；而且存在主从服务器的情况下，不用担心主机宕机无法请求数据且存在数据丢失的问题，一台主机无法工作，其他从服务器能接手工作，实现服务的可靠性，高可用性

#### 9. 主从复制的流程是怎么样的？

答：主从复制的流程如下：

1. 在从库中执行replicationof或者slaveof命令（replicationof 主服务ip 主服务port）,使主从库建立联系，主库会单独为从库开辟replication buffer缓冲区（完成主从复制后一旦主服务有数据的写请求时，会同步将数据写入缓冲区buffer中，然后再把buffer中的数据通过socket传递给从服务，实现数据的交互，从而写请求的数据都会同步储存在从服务器中）

2. 从库发送psync命令携带从库唯一的runid和从库的复制进度slave_offset给主库

3. 主库回复fullresync命令携带主库的唯一runid和主库的复制进度master_offset给从库

4. 主库开启**子线程**生成RDB文件，并在文件生成完成后发送给从库

5. 从库接收到RDB文件后，在**主线程**中**清空数据库**，加载RDB文件

6. 主库生成并发送给从库RDB文件过程中，主库主线程还有继续接收写命令，并保存在replication buffer中，等待主库发送完RDB文件后，将缓冲区的数据发送给从库


#### 10. 为什么要使用哨兵模式？怎么理解哨兵模式？

答：redis拥有了主从复制后，可以在一定程度上解放服务器的压力和单机宕机数据丢失问题，但没法解决问题包含：

1. 服务下线后便无法恢复服务，导致服务不可用，需要运维感知并手动拉起

2. 主从复制下，master节点服务宕机后，所有的写请求都请求失败，丢失数据，也需要在运维感知的情况下拉起主服务，会丢失这段时间的所有写请求数据

   而哨兵可以主动在非人为干扰和感知的情况下完成所有的服务拉起的工作



   哨兵模式下，哨兵可在主从服务中充当几个重要角色：监控，选主，通知

   1. 监控：哨兵可以判断主master或者从slave服务是否在线，可判断服务是否主观下线了，通过哨兵集群不同哨兵确定了主观下线后，决定主服务客观下线
   2. 选择新的主服务：当客观判定服务客观下线后，需要从slave服务中选举出新的master服务
   3. 通知：主库变更后，通知其他从库关联的主库ip和port的变更，也通知关联的client告知其服务的ip和端口的变更通知


#### 11. 哨兵集群是怎么工作的？

答：为了完成哨兵的角色任务（监控，选主，通知），则务必在哨兵建立工作前完成对以上任务的布置工作：

1. sentinel和master：通过sentinel monitor <master Name> <ip><port><quorum>同主库建立连接

2. Sentinel和slave：跟主库建立连接后，通过info命令，可以获取：

   * 主库的本身运行的信息
   * 从库salve的运行信息，之后哨兵通过获取的信息同从库建立订阅连接

3. sentinel和sentinel：跟主库建立连接后，订阅sentinel:hello频道，利用pub/sub发布订阅的机制，实现获取其他哨兵的信息，通过此频道为接下来后续的所有哨兵间的信息交换做了准备工作

   哨兵集群建立完对master,slave和其他sentinel的联系后，会定时发送一些监控命令：

   1. 每10s每个sentinel向master、slave发送INFO命令
   2. 每2s每个sentinel通过master节点的sentinel:hello频道交换信息
   3. 每1s每个sentinel向其他sentinel和master、slave发送ping命令，用于心跳检测判断是否存活，心跳的检测依据是down_after_millsecond时间内未回复心跳包，则判断主观下线
   4. 主观下线和客观下线（发现故障）
      * 主观下线（subjectively down，SDOWN）：当前 Sentinel 实例认为某个 redis服务为”不可用”状态。Sentinel 向 redis master 数据节点发送消息后30s（down-after-milliseconds） 内没有收到有效回复（+PONG、-LOADING 或者-MASTERDOWN），Sentinel 会将 master 标记为下线（打开 master 结构中 flags 的 SRI_S_DOWN 标记）
      * 客观下线（objectively down，ODOWN）：多个 Sentinel 实例都认为 master 处于 SDOWN 状态，那么此时 master 将处于 ODOWN， ODOWN可以简单理解为master已经被集群确定为”不可用”,将会开启故障转移机制。向其他 sentinel 节点发送
        sentinel is-master-down-by-addr 消息询问数据节点情况，得知达到 quorum 数量的 sentinel节点

#### 12. 哨兵是怎么选择主从切换任务执行的？

答：当主机被检测为客观下线时，在哨兵模式下，客观下线的判定需要哨兵集群中得到你N/2+1的依据判定master已经主观下线，则执行。那哨兵就要选择其他从服务之一作为新的主服务提供服务。选举的依据有：

1. 如果在down-after-millseconds毫秒内，哨兵都没有的带从节点的心跳回复，且次数超过了10次，则可判定从库网络并不佳，不适合作为新主库

2. 优先选择优先级最高的从库（slave-priority）

3. 同旧主库同步程度最接近的从库优先级高（通过比较master_repl_offset和slave_repl_offset）

4. ID号最小的从库优先级高

   那选举的依据如上，各个哨兵是怎么选举的呢？

   1. 哨兵获得仲裁需要大于quorum赞成票数
   2. leader的选举，需要拿到半数以上的选票，或者拿到超过哨兵配置的quorum
   3. 如果在一轮投票环节中不产生leader，哨兵会等待一段时间再重新选举
   4. 务必部署三个以上的哨兵集群，否则投票环节无法进行

   过程：在认为主节点`客观下线`的情况下,哨兵节点节点间会发起一次选举，命令还是上面的命令`SENTINEL is-master-down-by-addr <ip> <port> <current_epoch> <run_id>`,只是`run_id`这次会将`自己的run_id`带进去，希望接受者将自己设置为主节点。如果超过半数以上的节点返回将该节点标记为leader的情况下，会有该leader对故障进行迁移

#### 13. 什么是集群模式？为什么要做成集群？

答：

1. 为什么要做集群？

   - 主从复制不能实现真正意义上的服务高可用，当请求量很大的时候，做不了水平的拓展

   - 当请求量越来越多，业务需要支撑更高的QPS，主从复制中单机的QPS可能无法做到满足业务的需求

   - 纵向拓展，单纯的价内存不能达到要求

   - 网络流量上的的需求：业务的流量已经超过服务器的网卡上限值，可以考虑使用分布式来进行分流

2. 集群模式即为redis-cluster,采用无中心的结构，每个节点保存数据和整个集群状态，每个节点和其他所有节点进行连接。内部采用虚拟槽的方式分区其中主节点之间是怎么进行通讯的呢？

   * 一个切片集群共有16384个哈希槽，根据CRC16算法得到哈希值跟16384取模映射到一个哈希槽中

   * 使用cluster meet命令建立主节点实例间的通讯连接

     a. .节点A会为节点B创建一个clusterNode结构，并将该结构添加到自己的clusterState.nodes字典里面

     b. 节点A根据CLUSTER MEET命令给定的IP地址和端口号，向节点B发送一条MEET消息

     c. 节点B接收到节点A发送的MEET消息，节点B会为节点A创建一个clusterNode结构，并将该结构添加到自己的clusterState.nodes字典里面

     d. 节点B向节点A返回一条PONG消息

     e. 节点A将收到节点B返回的PONG消息，通过这条PONG消息节点A可以知道节点B已经成功的接收了自己发送的MEET消息

     f. 节点A将向节点B返回一条PING消息

     g. 节点B将接收到的节点A返回的PING消息，通过这条PING消息节点B可以知道节点A已经成功的接收到了自己返回的PONG消息，握手完成

     h. 节点A会将节点B的信息通过Gossip协议传播给集群中的其他节点，让其他节点也与节点B进行握手，最终，经过一段时间后，节点B会被集群中的所有节点认识

   * 使用cluster addslots命令，指定给每个实例分配哈希槽个数

   * 这里涉及到重定向的概念，客户端给一个实例发送数据读写操作时，这个实例上并没有相应的数据，客户端要再给一个新实例发送操作命令。

     a. 哈希槽的数据已经迁移到新节点，则回复MOVED命令，通知客户端向新实例发送请求数据，并缓存新实例地址

     b. 哈希槽数据在迁移过程中，则回复ASK命令，通知客户端向新实例请求数据，暂不缓存新实例地址 

3. 故障的转移方案：

   发现故障节点：

   1. 集群内的节点会向其他节点发送PING命令，检查是否在线
   2. 如果未能在规定时间内做出PONG响应，则会把对应的节点标记为疑似下线
   3. 集群中一半以上`负责处理槽的主节点`都将主节点X标记为疑似下线的话，那么这个主节点X就会被认为是`已下线`
   4. 向集群广播主节点X`已下线`,大家收到消息后都会把自己维护的结构体里的主节点X标记为`已下线`

   从节点选举：

   1. 当从节点发现自己复制的主节点已下线了，会向集群里面广播一条消息，要求所有有投票权的节点给自己投票(`所有负责处理槽的主节点都有投票权`)
   2. 主节点会向第一个给他发选举消息的从节点回复支持
   3. 当支持数量超过N/2+1的情况下，该从节点当选新的主节点

   故障的迁移:

   1. 新当选的从节点执行 `SLAVEOF no one`,修改成主节点
   2. 新的主节点会撤销所有已下线的老的主节点的槽指派，指派给自己
   3. 新的主节点向集群发送命令，通知其他节点自己已经变成主节点了，负责哪些槽指派
   4. 新的主节点开始处理自己负责的槽的命


#### 14. 如何提高缓存的命中率？

答：提高缓存的命中率：

1. 做数据预热

2. 加长数据的过期时间

3. 调整缓存的粒度，缓存单个对象比缓存整个对象的集合会提升命中率

4. 使用分布式的水平扩容缓存，增加缓存的容量上限，避免引起缓存频繁的失效和淘汰



   缓存的命中率的查看方式：

   使用info命令，可以打印如下数据：
   keyspace_hits:14414110  
   keyspace_misses:3228654  
   used_memory:433264648  
   expired_keys:1333536  
   evicted_keys:1547380
   其中缓存的命中率可以通过keyspace_hits/(keyspace_hits+keyspace_misses)得出



#### 15. Redis的淘汰key的策略有什么（替换策略）？

答： Redis中的淘汰策略包括定期删除+惰性删除+淘汰策略：

1. 定期删除：redis 会将每个设置了过期时间的 key 放入到一个独立的字典中，以后会定期遍历这个字典来删除到期的 key。Redis 默认会每秒进行十次过期扫描（100ms一次），过期扫描不会遍历过期字典中所有的 key，而是采用了一种简单的贪心策略：
   * 从过期字典中随机20个key
   * 删除这20个key中已经过期的key
   * 如果过期key比率超过1/4，重复步骤1
2. 惰性删除：在客户端访问这个key的时候，redis对key的过期时间进行检查，如果过期了就立即删除，不会给你返回任何东西。定期删除可能会导致很多过期key到了时间并没有被删除掉。所以就有了惰性删除
3. **内存淘汰策略**：解决以上定期删除和惰性删除这两张不精准删除方案下，还是会存在很多过期key未删除的场景。
   * noeviction：当内存使用超过配置的时候会返回错误，不会驱逐任何键
   * allkeys-lru：加入键的时候，如果过限，首先通过LRU算法驱逐最久没有使用的键
   * volatile-lru：加入键的时候如果过限，首先从设置了过期时间的键集合中驱逐最久没有使用的键
   * allkeys-random：加入键的时候如果过限，从所有key随机删除
   * volatile-random：加入键的时候如果过限，从过期键的集合中随机驱逐
   * volatile-ttl：从配置了过期时间的键中驱逐马上就要过期的键
   * volatile-lfu：从所有配置了过期时间的键中驱逐使用频率最少的键
   * allkeys-lfu：从所有键中驱逐使用频率最少的键

#### 16. Redis中可能存在的阻塞的点有哪些？

答：Redis可能存在的阻塞的点有：

1. 全量查询和聚合数据
2. 清空数据库，flushall命令
3. bigkey删除
4. AOF日志的同步写
5. 加载RDB文件

#### 17. Redis可能存在的缓存异常有哪些？

答：可能存在的缓存异常以及解决方案如下：

1. 缓存穿透：短时间内查询redis缓存和数据库中均没有的数据，导致大量的请求打到数据库中，数据库承受不了此请求压力。

   解决办法：

   ​	a. 使用布隆过滤器，判断查询的key是否存在

   ​	b. 从数据库中读到nil的数据时也同步写到缓存中，设置合适的过期时间

2. 缓存击穿：高并发请求查询情况下，某个key刚好失效了，导致请求都达到数据库中

   解决办法：

   ​	a. 使用互斥锁，分布式锁

   ​	b. 分布式集群中，使用redlock

3. 缓存雪崩：短时间内高并发请求，多个key缓存过期，导致所有的请求都打到数据库中

   解决办法：

   ​	a. 预防： 实现服务的高可用，建立集群模式，对高并发做分流请求。即使主机宕机，在集群模式下，依		  然可以稳定的提供服务可用; 做好热备份和数据的预热，提前预约

   ​       b. 雪崩中：实现服务的熔断和降级方案

   ​      c. 雪崩发生后：对已热备的数据做恢复，重启服务

#### 18. 怎么实现Redis的分布式锁？

答： 

​	基于单个redis节点的分布式锁：

 1. 使用SETNX命令加锁，DEL命令解锁

 2. 为了预防在加锁操作后发生数据异常导致DEL锁命令一直没执行，给锁设置一个合适的过期时间

 3. 在并发情况下，为了保证加锁的唯一性，需要客户点在执行SETNX加锁命令的时候使用唯一的key，同理解锁操作也是如此

    基于多个redis节点的分布式锁:

    使用redlock分布式锁算法，步骤如下：

    1. 客户端获取当前时间

    2. 客户端按顺序依次向N个Redis实例执行加锁操作，加锁操作要设置超时时间，加锁的时间要远远小于锁的有效时间

    3. 一旦客户端完成了所有redis实例的加锁操作，客户端要计算整个加锁过程的总耗时，满足以下条件则加锁成功：

       a. 客户端从超过N/2+1的Redis实例中成功获取到锁

       b. 客户端获取锁的总耗时没有超过锁的有效时间

    4. 如果不满足以上两个条件之，则客户端要向所有Redis释放锁操作

#### 19. Redis常见性能问题和解决方案？

答：

1. Master最好不要做任何持久化工作，包括内存快照和AOF日志文件，特别是不要启用内存快照做持久化。
2. 如果数据比较关键，某个Slave开启AOF备份数据，策略为每秒同步一次。
3. 为了主从复制的速度和连接的稳定性，Slave和Master最好在同一个局域网内。
4. 尽量避免在压力较大的主库上增加从库
5. Master调用BGREWRITEAOF重写AOF文件，AOF在重写的时候会占大量的CPU和内存资源，导致服务load过高，出现短暂服务暂停现象。
6. 为了Master的稳定性，主从复制不要用图状结构，用单向链表结构更稳定，即主从关系为：Master<–Slave1<–Slave2<–Slave3…，这样的结构也方便解决单点故障问题，实现Slave对Master的替换，也即，如果Master挂了，可以立马启用Slave1做Master，其他不变。

#### 20. 假如Redis里面有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来？

答： 

- 使用keys指令可以扫出指定模式的key列表。
- 对方接着追问：如果这个redis正在给线上的业务提供服务，那使用keys指令会有什么问题？
  这个时候你要回答redis关键的一个特性：redis的单线程的。keys指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用scan指令，scan指令可以无阻塞的提取出指定模式的key列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用keys指令长。

#### 21. 使用Redis做过异步队列吗，是如何实现的

答： 使用list类型保存数据信息，rpush生产消息，lpop消费消息，当lpop没有消息时，可以sleep一段时间，然后再检查有没有信息，如果不想sleep的话，可以使用blpop, 在没有信息的时候，会一直阻塞，直到信息的到来。redis可以通过pub/sub主题订阅模式实现一个生产者，多个消费者，当然也存在一定的缺点，当消费者下线时，生产的消息会丢失

#### 22. Redis如何实现延时队列

答：使用sortedset，使用时间戳做score, 消息内容作为key,调用zadd来生产消息，消费者使用zrangbyscore获取n秒之前的数据做轮询处理







