# Redis

 Redis（REmote DIctionary Server）是用C语言开发的一个开源的高性能键值对（key-value）数据库

**特征**：

1. 数据间没有必然的关联关系
2. 内部采用单线程机制进行工作
3. 高性能
4. 多数据类型支持：

* 字符串类型	string		
* 列表类型 list
* 散列类型 hash
* 集合类型 set
* 有序集合类型 sorted_set

## 1.常用的数据类型及操作

### 1.1 redis数据存储格式

* redis自身是一个Map，其中所有的数据都是采用key:value的形式存储
* **数据类型**指的是存储的数据的 类型，也就是value部分的类型，key部分永远都是字符串

#### 1.1.1 string

* 存储的数据：单个数据，最简单的数据存储类型
* 存储数据的格式：一个存储空间保存一个数据
* 存储内容：通常使用字符串，如果字符串以整数的形式展示，可以作为数字操作使用

```redis
* 添加/修改数据
set key value

* 获取数据
get key

* 删除数据
del key

* 添加/删除多个数据
mset key1 value1 key2 value2 ...

* 获取多个数据
mget key1 key2 ...

* 获取数据字符个数（字符串长度）
strlen key

* 追加信息到原始信息后部(不存在就创建)
append key value
```

**string类型的扩展操作**

```redis
* 设置数值数据增加指定范围的值
incr key
incrby key increment
incrbyfloat key increment

* 设置数值数据减少指定范围的值
decr key
decrby key increment
```

* string在redis内部存储默认就是一个字符串，遇到增减操作会转换成数值型进行计算
* redis所有的操作都是原子性的，采用单线程处理所有业务，命令是一个一个执行，因此无需考虑并发带来的数据影响

**设置数据具有指定的生命周期**

```redis
setex key seconds value
psetex key milliseconds value
```

**key设置**

![](F:\学习笔记\redis\string中的key格式.jpg)

#### 1.1.2 hash

![hash](F:\学习笔记\redis\hash.jpg)



* 新的存储需求：对一系列存储的数据进行编组，方便管理，典型应用存储对象信息
* 需要的存储结构：一个存储空间保存多个键值对数据
* hash类型：
  * 如果field数据比较少，采用类数组的方式进行存储
  * 如果field数据比较多，采用Hashmap进行存储

```redis
* 添加/修改数据
hset key field value

* 获取数据
hget key field
hgetall key

* 删除数据
hdel key field1 [field2]

* 添加/修改多个数据
hmset key field1 value1 field2 value2

* 获取多个数据 
hmget key field1 field2 ...

* 获取哈希表中字段的数量
hlen key

* 获取哈希表中是否存在指定的字段
hexists key field

```

##### **hash类型数据扩展操作**

```redis
* 获取哈希表中所有的字段名或字段值
hkeys key
hvals key

* 设置指定字段的数值数据增加指定范围的值
hincrby key field increment
hincrbyfloat key field increment

hsetnx key field value
```

##### **hash使用注意事项**

* hash类型下的value只能存储字符串，不允许存储其他数据类型，不存在嵌套现象。如果数据未获取到，对应的值为（nil）
* 每个hash可以存储2<sup>32</sup>-1个键值对
* hash类型贴近对象的数据存储形式，并且可以灵活添加删除对象属性。但hash设计初衷不是为了存储大量对象而设计，不可滥用，更不可以当作对象列表使用
* hgetall操作可以获取全部属性，如果内部field过多，遍历整体数据效率就会很低

##### hash类型应用场景

![hash购物车应用场景](F:\学习笔记\redis\hash购物车应用场景.jpg)

解决方案：

* 以用户id作为key，每个客户创建一个hash存储结构存储对应的购物车信息
* 以商品编号作为field，购买数量作为value进行存储
* 添加商品：追加全新的field与value
* 浏览：遍历hash
* 更改数量：自增/自减，设置value值
* 删除商品：删除field
* 清空：删除key

#### 1.1.3 List

* 数据存储需求：存储多个数据，并对数据进入存储空间的顺序进行区分
* 需要的存储结构: 一个存储空间保存多个数据，且通过数据可以体现进入顺序
* list类型： 保存多个数据，底层使用**双向链表**存储结构实现

```redis
* 添加数据
lpush key value1 [value2]...
rpush key value1 [value2]...

* 获取数据
lrange key start stop
lindex key index
llen key

* 获取并移除数据
lpop key
rpop key

* 规定时间内获取并移除数据
blpop key1 [key2] timeout
brpop key1 [key2] timeout
```

##### 扩展操作

```redis
* 移除指定数据
lrem key count value
```

应用场景：微信点赞

#### 1.1.4 set

* 新的存储需求：存储大量的数据，在查询方面提供更高的效率
* 需要的存储结构：能够保存大量的数据，高校的内部存储机制，便于查询

* set类型：与hash存储结构完全相同，仅存储键，不存储值（nil），并且值是不允许重复的

```redis
* 添加数据
sadd key member1 [member2]

* 获取全部数据
smembers key

* 删除数据
srem key member1 [member2]

* 获取集合数据总量
scard key

* 判断集合中是否包含指定数据
sismember key member
```

**扩展操作**

```redis
* 随机获取集合中指定数量的数据
srandmember key [count]

* 随机获取集合中的某个数据并将该数据移出集合
spop key

* 求两个集合的交、并、差集
sinter key1 [key2]
sunion key1 [key2]
sdiff  key1 [key2]

* 求两个集合的交、并、差并存到指定集合中
sinterstore key1 [key2]
sunionstore key1 [key2]
sdiffstore key1 [key2]

* 将指定数据从原始集合中移动到目标集合中
smove source destination member
```

应用场景：权限校验、网站访问量统计、黑白名单

#### 1.1.5 sroted_set

* 新的存储需求：数据排序有利于数据的有效展示，需要提供一种可以根据自身特征进行排序的方式
* 需要的存储结构：新的存储模型，可以保存可排序的数据
* sorted_set类型：在set的存储结构基础上添加可排序字段

```redis
* 添加数据
zadd key score1 member1 [score2 member2]

* 获取全部数据
zrange key start stop [withscores]
zrevrange key start stop [withscores]

* 删除数据
zrem key member [member ...]

* 按条件获取数据
zrangebyscore key min max [withscores] [limit]
zrerangeby score key max min [withscores] [limit]

* 条件删除数据
zremrangebyrank key start stop
zremrangebyscore key min max

* 获取集合数据总量
zcard key
zcount key min max

* 集合交、并操作
zinterstore destination numkeys key [key...]
zunionstore destination numbkeys key [key...]
```

**扩展操作**

```redis
* 获取数据对应的索引（排名）
zrank key member
zrevrank key member

* score值获取与修改
zscore key member
zincrby key increment member
```



#### 1.1.6 key通用操作

```redis
* 删除指定key
del key

* 获取key是否存在 
exists key

* 获取key的类型
type key

* 为key改名
rename key newkey
renamex key newkey

* 对所有key排序
sort

```

##### 时效性控制

```redis
* 为指定key设置有效期
expire key seconds
pexpire key milliseconds
expireat key timestamp
expireat key milliseconds-timestamp

* 获取key的有效时间
ttl key  // 不存在-2，存在没有时长-1，存在有时长返回时长
pttl key

* 将key从时效性转换为永久性
perstst key
```

##### 查询模式

```redis
* 查询key
keys parttern
* 匹配任意数量的任意符号  ? 符合一个任意符号  [] 匹配一个指定符号
```

#### 1.1.7db通用操作

```redis
* 切换数据库（0-15）
select index

* 其他操作
quit
ping
echo message

* 数据移动
move key db

* 数据清楚
dbsize
flushdb
flushall
```

## 1.2 Linux下redis操作

以指定的端口号启动redis-server

```redis
redis-server --port 6380
```

### 1.2.1 持久化

RDB（快照）  AOF（日志）

#### RDB

##### **RDB启动方式**---save指令想观配置

* dbfilename dump.rdb

  ​		说明： 设置本地数据库文件名，默认值为dump.rdb

  ​		经验：通常设置为:dump-**端口号**.rdb

* dir

  ​        说明:   设置存储.rdb文件的路径

  ​        经验：通常设置成存储空间较大的目录中，目录名称data

* rdbcompresssion yes

  ​	    说明：设置存储至本地数据库时是否压缩数据，默认为yes，采用**LZF压缩**

  ​		经验：通常默认为开启状态，如果设置为no，可以解约CPU运行时间，但会使存储的文件变大（**巨大**）

* rdbchecksum yes

  ​		说明： 设置是否进行RDB文件格式校验，该校验过程在写文件和读文件过程均进行

  ​        经验：通常默认为开启状态，如果设置为no，可以节约读写过程约10%时间消耗，但是存在一定的数据损坏风险

```redis
* 后台save
bgsave
```

* 配置

```redis
save second changes
```

* 作用：满足限定时间范围内key的变化数量达到指定数量即进行持久化
* 参数：second:监控时间范围  changes:监控key的变化量
* 位置：在conf文件中配置

![RDB](F:\学习笔记\redis\RDB.jpg)

##### **RDB优缺点**

**优点**：（1）RDB是一个紧凑压缩的二进制文件，存储效率比较高（2）RDB内部存储的是redis在某个**时间点**的数据快照，非常适合用于数据备份，全量复制等场景（3）RDB恢复数据的速度要比AOF快很多（4）应用：服务器每X小时执行bgsave备份，并将RDB文件口拷贝到远程机器中，用于灾难恢复

**缺点**：（1）RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据（2）bgsave指令每次运行要执行fork操作创建子进程（3）Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务质之间数据格式无法兼容的现象

#### AOF

* 不写全数据，仅记录部分数据
* 改记录数据为记录操作过程
* 对所有操作均进行记录，排除丢失数据的风险



AOF持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令达到恢复数据的目的。与RDB相比可以简单描述为改记录数据为记录数据产生的过程，AOF解决了数据持久化的实时性，目前已经是Redis持久化的主流方式。

##### **三种策略**

* always（每次）

  每次写入操作均同步到AOF文件中，**数据零误差，性能较低**

* everysec（每秒）

  每秒将缓冲区中的指令同步到AOF文件中，**数据准确性较高，性能较高**

  在系统突然当即的情况下丢失1秒的数据

* no（系统控制）

  由操作系统控制每次同步到AOF文件的周期，整体过程**不可控**

```redis
![RDBVSAOF](F:\学习笔记\redis\RDBVSAOF.jpg)* 配置
appendonly yes|no
作用：是否开启AOF持久化功能，默认为不开启
 
*配置
appendfsync always|everysec|no
作用：AOF写数据策略

*配置
appendfilename filename
作用：AOF持久化文件名，默认文件名为appendonly.aof,建议配置为appendonly-端口号.aof
```

##### AOF重写

随着命令不断写入AOF，文件会越来越大，AOF文件重写是将Redis紧成内的数据转化为写命令同步到新AOF文件的过程。简单来说就是对同一个数据的若干条命令执行结果转化成最终结果数据对应的指令进行记录

* 进程内已经超时的数据不再写入文件
* 忽略无效指令，重写时使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令
* 对统一数据的多条写命令合并为一条命令：lpush list1 a 、 lpush list1 b、 lpush list1 c可以转化为：lpush list1 a b c。（每条指令最多写入64个元素）

```redis
* 手动重写
bgrewriteaof
* 自动重写
auto-aof-rewrite-min-size size
auto-aof-rewrite-percentage percentage
```

#### RDB VS AOF

![RDBVSAOF](F:\学习笔记\redis\RDBVSAOF.jpg)

### 1.2.2 事务

* 开启事务

```redis
multi  ---设置事务的开启位置，此指令执行后，后续的所有指令均加入到事务中
```

* 执行事务

```redis
exec ---设定事务的结束位置，同时执行事务
```

注意：加入事务的命令暂时进入到任务队列中，并没有立即执行，而是等到执行命令后一起执行

* 取消事务

```redis
discard ---终止当前事务的定义，发生再multi之后，exec之前
```

* 对key添加监视锁，在执行exec前如果key发生变化，终止事务执行

```redis
watch key1 [key2...]
```

* 取消对所有key的监视

```redis
unwatch
```

* 使用setnx设置一个公共锁

```redis
setnx lock-key value -- 利用setnx命令的返回值特征，有值则返回设置失败，无值则返回设置成功
```

**死锁解决方案**

* 使用expire为锁key添加时间限定，到时不释放，放弃锁

```redis
expire lock-key second
pexpire lock-key milliseconds
```

由于操作通常都是微秒或者毫秒级，因此该锁定时间不宜设置过大。具体时间需要业务测试后确认。

* 例如：持有锁的操作最长时间127ms，最短执行时间7ms。
* 测试百万次最长执行时间命令的最大耗时，测试百万次网络延迟平均耗时
* 锁时间设定推荐：最大耗时*120%+平均网络延迟\*110%
* 如果业务最大耗时<<网络平均延迟，通常为两个数量级，取其中单个耗时较长的即可

### 1.2.3 删除策略

#### 定时删除

* 创建一个定时器，当key设置有过期时间，且过期时间到达时，由定时器任务立即执行对键的删除操作
* 优点：节约内存，到时就删除，快速四方掉不必要的内存占用
* 缺点：CPU压力很大，无论CPU此时负载量多高，均会占用CPU，会影响redis服务器响应时间和指令吞吐量
* 总结：用CPU性能换取存储空间（时间换空间）

#### 惰性删除

* 数据到达过期时间，不做处理。等下次访问该数据时
  	* 如果未过期，返回数据
  	* 发现已过期，删除，返回不存在

* 优点：节约CPU性能，等到必须删除时才删除
* 缺点：内存压力很大，出现长期占用内存的数据

* 总结：存储空间换CPU性能（空间换时间）

#### 定期删除

![定期删除](F:\学习笔记\redis\定期删除.jpg)

#### **逐出算法**

```redis
* 最大可使用内存
maxmemory

* 每次选取待删除数据的个数
maxmemory-samples

* 删除策略
maxmemory-policy volatile-lru
```

* 检查易失数据：

1. volatile-lru：挑选最近最少使用的数据淘汰
2. volatile-lfu
3. volatile-ttl：挑选将要过期的数据淘汰
4. volatile-random： 任意选择数据淘汰

* 检测全库数据：

5. allkeys-lru
6. allkeys-random
7. allkeys-lfu

* 放弃数据驱逐

8. no-enviction

## 1.3 服务器配置文件解析

* 设置服务器以守护进程的方式进行

  ```redis
  daemonize yes|no
  ```

* 绑定主机地址

  ```redis
  bind 127.0.0.1
  ```

* 设置服务器端口号

  ```
  port 6379
  ```

* 设置数据库数量

  ```redis
  database 16
  ```

### 日志配置

* 设置服务器以指定日志记录级别

  ```redis
  loglevel debug|verbose|notice|warning
  ```

### 客户端配置

* 设置同一时间最大客户端连接数，默认无限制。当客户端连接到达上限，Redis关闭新的连接

  ```redis
  maxclients 0
  ```

* 客户端闲置等待最大时长，达到最大值后关闭

  ````redis
  timeout 3600
  ````

### 多服务器快捷配置

* 导入并加载指定配置文件信息，用于快速创建Redis公共配置较多的redis实例配置文件，便于维护

  ```redis
  include /path/server-端口号.conf
  ```

## 1.4 高级数据类型

* Bitmaps
* HyperLogLog
* GEO

### Bitmaps

**基础操作**

* 获取指定key对应偏移量上的bit值

  ```redis
  getbit key offset
  ```

* 设置指定key对应偏移量上的bit值，value只能是1或0

  ```redis
  setbit key offset value
  ```

#### 扩展操作

* 对指定key按位进行交、并、非、异或操作，并将结果保存到destKey中

  ```redis
  bitop op destKey key1 [key2...]
  ```

* 统计指定key中1的数量

  ```redis
  bitcount key [start end]
  ```

  

