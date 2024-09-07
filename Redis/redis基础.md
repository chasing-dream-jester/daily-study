# Redis 基础

## Redis 简介
Redis（Remote Dictionary Server）是一个开源的内存数据库，遵守 BSD 协议，它提供了一个高性能的键值（key-value）存储系统，常用于缓存、消息队列、会话存储等应用场景

优势：
1. 性能极高 ：Redis 以其极高的性能而著称，能够支持每秒数十万次的读写操作24。这使得Redis成为处理高并发请求的理想选择，尤其是在需要快速响应的场景中，如缓存、会话管理、排行榜等。
2. 丰富的数据类型 ：Redis 不仅支持基本的键值存储，还提供了丰富的数据类型，包括字符串、列表、集合、哈希表、有序集合等。这些数据类型为开发者提供了灵活的数据操作能力，使得Redis可以适应各种不同的应用场景。
3. 原子性操作 ：Redis 的所有操作都是原子性的，这意味着操作要么完全执行，要么完全不执行。这种特性对于确保数据的一致性和完整性至关重要，尤其是在高并发环境下处理事务时。
4. 持久化 ： Redis 支持数据的持久化，可以将内存中的数据保存到磁盘中，以便在系统重启后恢复数据。这为 Redis 提供了数据安全性，确保数据不会因为系统故障而丢失。
5. 支持发布/订阅模式 ：Redis 内置了发布/订阅模式（Pub/Sub），允许客户端之间通过消息传递进行通信。这使得 Redis 可以作为消息队列和实时数据传输的平台
6. 单线程模型 ：尽管 Redis 是单线程的，但它通过高效的事件驱动模型来处理并发请求，确保了高性能和低延迟。单线程模型也简化了并发控制的复杂性。
7. 主从复制 ：Redis 支持主从复制，可以通过从节点来备份数据或分担读请求，提高数据的可用性和系统的伸缩性
8. 跨平台兼容性：Redis 可以在多种操作系统上运行，包括 Linux、macOS 和 Windows，这使得它能够在不同的技术栈中灵活部署。

## Redis 常用数据类型
1. string（字符串）： 基本的数据存储单元，可以存储字符串、整数或者浮点数。
2. hash（哈希）：一个键值对集合，可以存储多个字段
3. list（列表）：一个简单的列表，可以存储一系列的字符串元素
4. set（集合）：一个无序集合，可以存储不重复的字符串元素
5. zset(sorted set：有序集合): 类似于集合，但是每个元素都有一个分数（score）与之关联
6. 流（Streams）：用于消息队列和日志存储，支持消息的持久化和时间排序

## 启动 Redis 服务器
### Redis启动方式
1. 使用默认配置启动
   接运行 redis-server 命令将使用默认配置启动 Redis 服务器
   ```sh
   redis-server
   ```
2. 使用自定义配置文件启动
   ```sh
   redis-server /path/to/redis.conf
   ```
   配置文件示例(redis.conf)
   ```conf
    # 绑定到本地地址
    bind 127.0.0.1

    # 保护模式
    protected-mode yes

    # 端口
    port 6379

    # 日志文件
    logfile "/var/log/redis/redis-server.log"

    # 数据目录
    dir /var/lib/redis

    # 设置密码
    requirepass your_password

    # 最大内存
    maxmemory 256mb

    # 最大内存策略
    maxmemory-policy allkeys-lru

    # 持久化配置
    save 900 1
    save 300 10
    save 60 10000

    appendonly yes
    appendfilename "appendonly.aof"
    appendfsync everysec
   ```
### 常用命令行选项
1. --port ：指定 Redis 服务器监听的端口 `redis-server --port 6380`
2. --bind ：指定 Redis 服务器绑定的 IP 地址 `redis-server --bind 127.0.0.1`
3. --protected-mode ：启用或禁用保护模式 `redis-server --protected-mode no`
4. --requirepass ：设置访问密码 `redis-server --requirepass your_password`
5. 后台运行 : 可以在配置文件中设置 daemonize 选项 `daemonize yes` 或者直接运行命令 `redis-server /path/to/redis.conf --daemonize yes`
6. 关闭 Redis 服务器 `redis-cli shutdown`
7. 检查 Redis 服务器状态 `redis-cli info`


## Redis常用的**key**命令

| 命令               | 用途                                      | 示例                              |
|--------------------|-------------------------------------------|-----------------------------------|
| `DEL key [key ...]`| 删除一个或多个键                          | `DEL mykey`                       |
| `EXISTS key [key ...]` | 检查一个或多个键是否存在                  | `EXISTS mykey`                    |
| `EXPIRE key seconds` | 为键设置过期时间（秒）                     | `EXPIRE mykey 60`                 |
| `EXPIREAT key timestamp` | 为键设置过期时间（Unix 时间戳）            | `EXPIREAT mykey 1672531199`       |
| `TTL key`          | 获取键的剩余生存时间（秒）                 | `TTL mykey`                       |
| `PTTL key`         | 获取键的剩余生存时间（毫秒）               | `PTTL mykey`                      |
| `PERSIST key`      | 移除键的过期时间                          | `PERSIST mykey`                   |
| `RENAME key newkey` | 重命名键                                  | `RENAME mykey newkey`             |
| `RENAMENX key newkey` | 重命名键（仅当新键名不存在时）              | `RENAMENX mykey newkey`           |
| `MOVE key db`      | 将键移动到另一个数据库                    | `MOVE mykey 1`                    |
| `RANDOMKEY`        | 返回一个随机键                            | `RANDOMKEY`                       |
| `TYPE key`         | 返回键的存储类型                          | `TYPE mykey`                      |
| `DUMP key`         | 序列化键                                  | `DUMP mykey`                      |
| `RESTORE key ttl serialized-value` | 反序列化键并存储                     | `RESTORE mykey 0 "\x00\x00..."`   |
| `MGET key [key ...]` | 获取多个键的值                            | `MGET key1 key2 key3`             |
| `MSET key value [key value ...]` | 设置多个键值对                          | `MSET key1 value1 key2 value2`    |
| `MSETNX key value [key value ...]` | 设置多个键值对（仅当所有键都不存在时） | `MSETNX key1 value1 key2 value2`  |
| `SCAN cursor [MATCH pattern] [COUNT count]` | 增量迭代数据库中的键          | `SCAN 0 MATCH mykey:*`            |
| `OBJECT subcommand [arguments [arguments ...]]` | 检查键的内部数据             | `OBJECT REFCOUNT mykey`           |

这个表格汇总了 Redis 中常用的键操作命令，包括它们的用途和示例，便于参考和使用。


## Redis 字符串(String)

Redis 字符串命令

1. SET key value ：设置指定 key 的值 
2. GET key ：获取key的值
3. GETRANGE key start end：返回 key 中字符串值的子字符 包含 end
4. GETSET key value ：将给定 key 的值设为 value ，并返回 key 的旧值(old value)。
5. GETBIT key offset 用于获取存储在指定键的字符串值的特定位的值
   ```sh
    # 设置位
    SETBIT mykey 0 1
    SETBIT mykey 1 0
    SETBIT mykey 2 1
    SETBIT mykey 3 0
    SETBIT mykey 4 1
    SETBIT mykey 5 0
    SETBIT mykey 6 1
    SETBIT mykey 7 0

    # 获取位
    GETBIT mykey 0  # 输出: 1
    GETBIT mykey 1  # 输出: 0
    GETBIT mykey 2  # 输出: 1
    GETBIT mykey 3  # 输出: 0
    GETBIT mykey 4  # 输出: 1
    GETBIT mykey 5  # 输出: 0
    GETBIT mykey 6  # 输出: 1
    GETBIT mykey 7  # 输出: 0

   ```
   使用场景
   * 位图（Bitmap）操作 ：GETBIT 命令常用于位图操作，例如用户活跃标记、权限管理等。
   * 稀疏数据存储 ：位图适用于存储稀疏数据，通过 GETBIT 和 SETBIT 命令可以高效地存取数据。
   * 布隆过滤器 ：位图是布隆过滤器的一种实现方式，GETBIT 命令可以用于查询某个元素是否可能存在。
  
   注意事项
   * 偏移量 ：偏移量从 0 开始，且不能为负数。如果指定的偏移量超出了字符串的长度，Redis 会自动将字符串扩展，并填充零。
   * 性能 ：GETBIT 和 SETBIT 操作在 Redis 中非常高效，但对于非常大的位图（例如数十亿位），需要注意内存占用。

6. SETBIT key offset value ：对key所储存的字符串值，设置或清除指定偏移量上的位(bit) 操作见上。
7. MGET  key1 [key2..]
8. SETEX key seconds value ：将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。
9. SETNX key value 只有在 key 不存在时设置 key 的值
10. SETRANGE key offset value 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。
11. STRLEN key 返回 key 所储存的字符串值的长度。
12. MSET key value [key value ...] ：同时设置一个或多个 key-value 对
13. MSETNX key value [key value ...]：同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
14. PSETEX key milliseconds value ：这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。
15. INCR key ： 将 key 中储存的数字值增一
    
    如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。

    如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

    本操作的值限制在 64 位(bit)有符号数字表示之内
16. INCRBY key increment :将 key 所储存的值加上给定的增量值（increment） 。
17. INCRBYFLOAT key increment :将 key 所储存的值加上给定的浮点增量值（increment） 。
18. DECR key:将 key 中储存的数字值减一。
19. DECRBY key decrement :key 所储存的值减去给定的减量值（decrement） 。
20. APPEND key value :如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。

## Redis 哈希(Hash)
Redis 哈希（Hash）是一种用于存储键值对集合的数据结构，类似于一个小型的键值对数据库。哈希特别适合用于表示对象，例如用户信息、配置项等。每个哈希本身是一个键，并且包含多个字段（field）及其对应的值（value）

以下是 Redis 哈希的常用命令及其说明：
1. HDEL key field1 [field2]:删除一个或多个哈希表字段
2. DEL key：:删除哈希表
3. HEXISTS key field :查看哈希表 key 中，指定的字段是否存在
4. HGET key field :获取存储在哈希表中指定字段的值。
5. HGETALL key :获取在哈希表中指定 key 的所有字段和值
6. HINCRBY key field increment:为哈希表 key 中的指定字段的整数值加上增量 increment 。原值必须是integer
7. HINCRBYFLOAT key field increment:将哈希表 key 中字段 field 的值加上浮点数 increment。原值需要是float
8. HKEYS key：获取哈希表中的所有字段
9. HLEN key：获取哈希表中字段的数量
10. HMGET key field1 [field2]：获取所有给定字段的值
11. HMSET key field1 value1 [field2 value2 ] ：同时将多个 field-value (域-值)对设置到哈希表 key 中 已弃用，使用下面hset
12. HSET key field value：将哈希表 key 中的字段 field 的值设为 value
13. HSETNX key field value：只有在字段 field 不存在时，设置哈希表字段的值。
14. HVALS key：获取哈希表中所有值
15. HSCAN key cursor [MATCH pattern] [COUNT count] ：迭代哈希表中的键值对
    ```txt
    key：要遍历的哈希表的键。
    cursor：游标，初始值为 0，每次迭代返回下一个游标值，直到游标值为 0 表示扫描结束。
    MATCH pattern：可选参数，指定匹配模式，用于过滤字段名。
    COUNT count：可选参数，指定每次迭代返回的元素数量，默认值为 10。
    ```
    ```sh
    HSET user:1000 name "John Doe"
    HSET user:1000 age 30
    HSET user:1000 email "john.doe@example.com"
    HSET user:1000 phone "123-456-7890"
    HSET user:1000 address "123 Main St"

    # 使用 HSCAN 命令遍历哈希表
    HSCAN user:1000 0

    # 使用 MATCH 过滤字段
    HSCAN user:1000 0 MATCH a*

    # 使用 COUNT 指定返回数量
    HSCAN user:1000 0 COUNT 2
    ```
## Redis 列表(List)
Redis 列表（List）是一种简单的链表数据结构，支持在两端插入和删除元素。列表中的元素可以是字符串类型，并且列表是有序的。Redis 列表非常适合用于实现队列和栈等数据结构

1. LPUSH key value1 [value2]:将一个或多个值插入到列表头部
2. LPOP key:移出并获取列表的第一个元素
3. LPUSHX key value 将一个值插入到已存在的列表头部
4. DEL key 删除LIST
5. LLEN key :获取列表长度
6. LINDEX key index : 通过索引获取列表中的元素
7. LINSERT key BEFORE|AFTER pivot value :在列表的元素前或者后插入元素
   ```txt
    key：要操作的列表的键。
    BEFORE|AFTER：指定新元素插入的位置，是在 pivot 元素之前（BEFORE）还是之后（AFTER）。
    pivot：列表中现有元素，新的元素将插入到其之前或之后。
    value：要插入的新元素。

    返回值：
    如果命令执行成功，返回列表的长度。
    如果没有找到 pivot 元素，返回 -1。
    如果 key 不存在，返回 0。
   ```
8. LRANGE key start stop: 获取列表指定范围内的元素
9. LREM key count value 移除列表元素
   ```txt
    key：要操作的列表的键。
    count：指定删除的方向和数量：
    count > 0：从头到尾搜索并删除最多 count 个元素。
    count < 0：从尾到头搜索并删除最多 count 个元素。
    count = 0：删除所有匹配的元素。
    value：要删除的元素的值
   ```
10. LSET key index value:通过索引设置列表元素的值
11. LTRIM key start stop :对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
12. RPUSH key value1 [value2] :在列表中添加一个或多个值到列表尾部
13. RPOP key: 移除列表的最后一个元素，返回值为移除的元素。
14. RPOPLPUSH source destination :移除列表的最后一个元素，并将该元素添加到另一个列表并返回
15. RPUSHX key value:为已存在的列表添加值
16. BLPOP key1 [key2 ] timeout:移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
17. BRPOP key1 [key2 ] timeout:移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
18. BRPOPLPUSH source destination timeout:从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
   

## Redis 集合(Set)
Redis 集合（Set）是一种无序的字符串集合，**集合中的元素是唯一**的，即不存在重复的元素。集合的操作非常高效，适合用于需要快速查找、添加和删除元素的场景。Redis 集合还支持集合间的数学运算，如交集、并集和差集

以下是 Redis 集合的常用命令及其说明
1. SADD key member1 [member2]: 向集合添加一个或多个成员
2. DEL key 删除集合
3. SCARD key: 获取集合的成员数
4. SDIFF key1 [key2]:返回第一个集合与其他集合之间的差异 (差集)
5. SINTER key1 [key2]：返回给定所有集合的交集 （交集）
6. SUNION key1 [key2]：返回所有给定集合的并集（并集）
7. SDIFFSTORE destination key1 [key2] ：返回给定所有集合的差集并存储在 destination 中
8. SINTERSTORE destination key1 [key2]：返回给定所有集合的交集并存储在 destination 中
9. SUNIONSTORE destination key1 [key2]：所有给定集合的并集存储在 destination 集合中
10. SISMEMBER key member ：判断 member 元素是否是集合 key 的成员
11. SMEMBERS key：返回集合中的所有成员
12. SMOVE source destination member ：将 member 元素从 source 集合移动到 destination 集合
13. SPOP key：移除并返回集合中的一个随机元素
14. SRANDMEMBER key [count]：返回集合中一个或多个随机数
15. SREM key member1 [member2] ：移除集合中一个或多个成员
16. SSCAN key cursor [MATCH pattern] [COUNT count]
    ```txt
    key：要遍历的哈希表的键。
    cursor：游标，初始值为 0，每次迭代返回下一个游标值，直到游标值为 0 表示扫描结束。
    MATCH pattern：可选参数，指定匹配模式，用于过滤字段名。
    COUNT count：可选参数，指定每次迭代返回的元素数量，默认值为 10。
    ```
## Redis 有序集合(sorted set)
Redis 有序集合（Sorted Set）是一种将成员（元素）与分数（score）关联的集合，成员是唯一的，但分数可以重复。与普通集合不同，有序集合中的元素按分数排序，这使得有序集合特别适合需要按一定顺序存储和访问数据的场景，如排行榜、优先级队列等。

常用命令
1. `ZADD key score1 member1 [score2 member2]` : 向有序集合添加一个或多个成员，或者更新已存在成员的分数
2. `ZCARD key` ：获取有序集合的成员数
3. `ZCOUNT key min max` :计算在有序集合中指定区间分数的成员数
4. `ZINCRBY key increment member`:有序集合中对指定成员的分数加上增量 increment
5. `ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]`:计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 destination 中 。（交集） numkeys：参与交集运算的有序集合的数量。
6. `ZDIFFSTORE destination numkeys key [key ...]`(差集) numkeys：参与交集运算的有序集合的数量。
7. `ZUNIONSTORE destination numkeys key [key ...]`:计算给定的一个或多个有序集的并集，并存储在新的 key 中
   ```txt
    destination：结果存储的有序集合的键名。
    numkeys：参与交集运算的有序集合的数量。
    key [key ...]：参与交集运算的有序集合的键名。
    WEIGHTS weight [weight ...]：可选参数，指定每个有序集合的权重。默认权重为 1。
    AGGREGATE SUM|MIN|MAX：可选参数，指定计算结果分数的方法。默认方法是 SUM。
      SUM：结果集合中每个成员的分数是所有集合中对应成员分数的和。
      MIN：结果集合中每个成员的分数是所有集合中对应成员分数的最小值。
      MAX：结果集合中每个成员的分数是所有集合中对应成员分数的最大值。
   ```

   ```sh
    ZADD zset1 1 "one" 2 "two" 3 "three"
    ZADD zset2 2 "one" 3 "two" 4 "four"

    # 计算交集并存储结果
    ZINTERSTORE out 2 zset1 zset2
    # out 有序集合的内容是
    ZRANGE out 0 -1 WITHSCORES
    # 输出:
    # 1) "one"
    # 2) "3"
    # 3) "two"
    # 4) "5"

    # 使用权重计算交集
    ZINTERSTORE out 2 zset1 zset2 WEIGHTS 2 3   
    # 在这个示例中，"one" 的分数是 1*2 + 2*3 = 8，"two" 的分数是 2*2 + 3*3 = 13。
    ZRANGE out 0 -1 WITHSCORES
    # 输出:
    # 1) "one"
    # 2) "8"
    # 3) "two"
    # 4) "13"
   ```
8. `ZLEXCOUNT key min max` :在有序集合中计算指定字典区间内成员数量,范围是按字典序（lexicographical order）计算的，而不是按分数（score）。该命令特别适用于需要按字典顺序统计成员的场景。 注意：需要确保分数相同 
   ```txt
    key：要操作的有序集合的键。
    min：字典序范围的最小成员。
    max：字典序范围的最大成员。
    字典序范围的表示
    使用 [ 表示包含（inclusive）。
    使用 ( 表示不包含（exclusive）。
    使用 - 表示最小值。
    使用 + 表示最大值。
   ```
9. `ZRANGE key start stop [WITHSCORES]`：通过索引区间返回有序集合指定区间内的成员
   ```sh
   ZRANGE myzset 0 -1 WITHSCORES
   ```
10. `ZRANGEBYLEX key min max [LIMIT offset count]`:通过字典区间返回有序集合的成员
   
   用于按字典顺序（lexicographical order）获取有序集合（sorted set）中指定成员范围内的成员。该命令特别适用于需要按字典顺序获取成员的场景。
   ```txt
    key：有序集合的键。
    min：字典序范围的最小成员。
    max：字典序范围的最大成员。
    LIMIT offset count：可选参数，用于指定结果的偏移量和数量。
    字典序范围的表示
    使用 [ 表示包含（inclusive）。
    使用 ( 表示不包含（exclusive）。
    使用 - 表示最小值。
    使用 + 表示最大值
   ```
11. `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]` :通过分数返回有序集合指定区间内的成员
12. `ZRANK key member` :返回有序集合中指定成员的索引 (Redis Zrank 返回有序集中指定成员的排名。其中有序集成员按分数值递增(从小到大)顺序排列。)
13. `ZREM key member [member ...]` :移除有序集合中的一个或多个成员
14. `ZREMRANGEBYLEX key min max` :移除有序集合中给定的字典区间的所有成员
15. `ZREMRANGEBYRANK key start stop`:移除有序集合中给定的排名区间的所有成员
16. `ZREMRANGEBYSCORE key min max`:移除有序集合中给定的分数区间的所有成员
17. `ZREVRANGE key start stop` [WITHSCORES]:返回有序集中指定区间内的成员，通过索引，分数从高到低
18. `ZREVRANGEBYSCORE key max min [WITHSCORES]`:返回有序集中指定分数区间内的成员，分数从高到低排序
19. `ZREVRANK key member`:返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
20. `ZSCORE key member`:返回有序集中，成员的分数值
21. `ZSCAN key cursor [MATCH pattern] [COUNT count]`:迭代有序集合中的元素（包括元素成员和元素分值）
    ```txt
    key：要遍历的哈希表的键。
    cursor：游标，初始值为 0，每次迭代返回下一个游标值，直到游标值为 0 表示扫描结束。
    MATCH pattern：可选参数，指定匹配模式，用于过滤字段名。
    COUNT count：可选参数，指定每次迭代返回的元素数量，默认值为 10。
    ```
