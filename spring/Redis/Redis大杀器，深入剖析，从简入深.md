# Redis大杀器，深入剖析，从简入深
本文将以简洁的图文来阐述**Redis**，建立起对redis的透彻印象，同时也会对redis在企业实战中的一些优化来做深度剖析，目的是拿捏redis不求人。 

* * *

**Redis**是什么？ **（Remote Dictionary Server )-远程字典服务** 是一个高性能的键值对（key-value）存储系统。它通常被用作**缓存层**，以提升应用程序的性能。Redis是内存中的数据结构存储系统，这意味着所有的数据都存储在内存中，使得读取和写入操作都非常快速。

* * *

[redis安装参考地址](https://link.juejin.cn/?target=https%3A%2F%2Fmnpq-dgz.blog.csdn.net%2Farticle%2Fdetails%2F134233056%3Fspm%3D1001.2014.3001.5502 "https://mnpq-dgz.blog.csdn.net/article/details/134233056?spm=1001.2014.3001.5502")

* * *

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42f0a922cd4c40f6a0bf17c0c73a6e3b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=4000&h=1188&s=575100&e=png&b=f8faed)

*   字符串（string）：字符串是 Redis 最基本的数据类型，一个键最大的存储容量是 512MB。
*   哈希表（hash）：哈希表存储的是键值对，可以理解为一个字典。每个哈希表最多可以存储 2^32 - 1 个键值对（40多亿）。
*   列表（list）：列表是一个有序的字符串类型数据集合，可以理解为一个数组。一个列表最多可以包含 2^32 - 1 个元素（40多亿）。
*   集合（set）：集合是一个无序的字符串类型数据集合，不允许重复元素。
*   有序集合（sorted set）：有序集合是一个有序的字符串类型数据集合，每个元素都有一个分数值（score），可以理解为带权重的元素。
*   位图（bitmap）：由二进制位组成的数据类型，可以对其进行位运算。
*   地理位置（geospatial）：可以储存地理位置的数据类型，支持距离计算和地理范围查询。
*   Redis 5.0 版本引入了 Stream 数据结构，它提供了基于消息的处理方式。
*   HyperLogLog：用于进行基数统计，即统计不同元素的数量。

* * *

  虽然Redis主要存储在内存中，但为了防止数据丢失，你可以使用持久化功能。Redis支持两种持久化方法：RDB（Redis DataBase）和AOF（Append Only File）。RDB通过生成数据快照进行持久化，而AOF则通过记录每次写操作进行持久化。

**RDB持久化** ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed52c7b7035d4f6cac78bb860588d9e4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1117&h=780&s=55282&e=jpg&b=fffcfc)
 Redis某一时刻的数据持久化到磁盘中，是一种快照式的持久化方法。

Redis在进行数据持久化的过程中，会先将数据写入到一个临时文件中，待持久化过程都结束了，才会用这个临时文件替换上次持久化好的文件。正是这种特性，让我们可以随时来进行备份，因为快照文件总是完整可用的。

RDB优点：

*   RDB是一个非常紧凑的文件,它保存了某个时间点得数据集,非常适用于数据集的备份,比如你可以在每个小时报保存一下过去24小时内的数据,同时每天保存过去30天的数据,这样即使出了问题你也可以根据需求恢复到不同版本的数据集.
*   RDB是一个紧凑的单一文件,很方便传送到另一个远端数据中心或者亚马逊的S3（可能加密），非常适用于灾难恢复.
*   RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化redis的性能.

RDB缺点：

*   如果你希望在redis意外停止工作（例如电源中断）的情况下丢失的数据最少的话，那么RDB不适合你.虽然你可以配置不同的save时间点(例如每隔5分钟并且对数据集有100个写的操作),是Redis要完整的保存整个数据集是一个比较繁重的工作,你通常会每隔5分钟或者更久做一次完整的保存,万一在Redis意外宕机,你可能会丢失几分钟的数据.
*   RDB 需要经常fork子进程来保存数据集到硬盘上,当数据集比较大的时候,fork的过程是非常耗时的,可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续1秒,AOF也需要fork,但是你可以调节重写日志文件的频率来提高数据集的耐久度.

**AOF持久化**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3095bb4e0c749db9cff96413ade43a5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2715&h=1140&s=481457&e=png&b=ffffff)

AOF方式是将执行过的写指令记录下来，在数据恢复时按照从前到后的顺序再将指令都执行一遍。

> 实现方式：我们通过配置Redis.conf 中的appendonly yes 就可以打开AOF功能。如果有写操作(如SET 等)，Redis 就会被追加到AOF 文件的末尾。 AOF 持久化的方式： 默认的AOF 持久化策略是每秒钟fsync 一次(fsync是指把缓存中的写指令记录到磁盘中)，因为在这种情况下，Redis 仍然可以保持很好的处理性能，即使Redis 故障，也只会丢失最近1 秒钟的数据。   如果在追加日志时，恰好遇到磁盘空间满或断电等情况导致日志写入不完整，也没有关系，Redis 提供了**Redis-check-aof**工具，可以用来进行日志修复。因为采用了追加方式，如果不做任何处理的话，AOF 文件会变得越来越大，为此，Redis 提供了AOF 文件重写(rewrite)机制，即当AOF 文件的大小超过所设定的阈值时，Redis 就会启动AOF 文件的内容压缩，只保留可以恢复数据的最小指令集。举个例子或许更形象，假如我们调用了100 次INCR 指令，在AOF 文件中就要存储100 条指令，但这明显是很低效的，完全可以把这100条指令合并成一条SET 指令，这就是重写机制的原理。

AOF 优点：我们通过一个场景再现来说明。某同学在操作Redis 时，不小心执行了FLUSHALL，导致Redis 内存中的数据全部被清空了，这是很悲剧的事情。不过这也不是世界末日，只要Redis 配置了AOF 持久化方式，且AOF文件还没有被重写(rewrite)，我们就可以用最快的速度暂停Redis 并编辑AOF文件，将最后一行的FLUSHALL 命令删除，然后重启Redis，就可以恢复Redis的所有数据到FLUSHALL 之前的状态了。是不是很神奇，这就是AOF 持久化方式的好处之一。但是如果AOF 文件已经被重写了，那就无法通过这种方法来恢复数据了。 AOF缺点：比如在同样数据规模的情况下，AOF文件要比RDB文件的体积大。而且，AOF 方式的恢复速度也要慢于RDB 方式。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79553e17946141e19121ff41744cc3d7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1137&h=413&s=41404&e=webp&b=d9d9d9)

企业级的持久化的配置策略

RDB 和 AOF 是 Redis 内部的两种数据持久化策略，这是两种不同的持久化策略，一种是基于内存快照，一种是基于操作日志。

在企业中，RDB的生成策略，用默认的也差不多

save 60 10000：如果你希望尽可能确保说，RDB最多丢1分钟的数据，那么尽量就是每隔1分钟都生成一个快照，低峰期，数据量很少，也没必要

具体是10000->生成RDB，1000->RDB，这个生成rdb操作的阈值大小，这个根据你自己的应用和业务的数据量，你自己去决定

AOF一定要打开，设置appendsync = everysec

auto-aof-rewrite-percentage 100: 就是当前AOF大小膨胀到超过上次100%，上次的两倍

auto-aof-rewrite-min-size 64mb: 根据你的数据量来定，16mb，32mb

**redis数据备份恢复**

[redis数据备份恢复参考地址](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_42555019%2Farticle%2Fdetails%2F100118260%3Fops_request_misc%3D%25257B%252522request%25255Fid%252522%25253A%252522170174114216800197047783%252522%25252C%252522scm%252522%25253A%25252220140713.130102334.pc%25255Fall.%252522%25257D%26request_id%3D170174114216800197047783%26biz_id%3D0%26utm_medium%3Ddistribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~hot_rank-10-100118260-null-null.142%255Ev96%255Epc_search_result_base6%26utm_term%3Dredis%25E5%25A4%2587%25E4%25BB%25BD%26spm%3D1018.2226.3001.4187 "https://blog.csdn.net/weixin_42555019/article/details/100118260?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170174114216800197047783%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=170174114216800197047783&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~hot_rank-10-100118260-null-null.142%5Ev96%5Epc_search_result_base6&utm_term=redis%E5%A4%87%E4%BB%BD&spm=1018.2226.3001.4187")

[redis数据迁移参考地址](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_44566795%2Farticle%2Fdetails%2F130562562 "https://blog.csdn.net/weixin_44566795/article/details/130562562")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cccb554d28b4e60bf844e66de79e7a6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=518&s=252291&e=png&b=fefdfd)

    学习一门技术，关键是为我所用，能过通过技术驱动业务，有实际的业务产出，提高企业变现的能力或整个生产流程的效率，这是每个技术人必须落实的理念！

    作为史上最大的Redis集群，有必要透过新浪微博这家企业来看redis在企业中的使用借鉴！

> 新浪微博目前使用的98%都是持久化的应用，2%的是缓存，用到了600+服务器 Redis中持久化的应用和非持久化的方式不会差别很大： 非持久化的为8-9万tps，那么持久化在7-8万tps左右； 当使用持久化时，需要考虑到持久化和写性能的配比，也就是要考虑redis使用的内存大小和硬盘写的速率的比例计算；

🍉 5.1 使用历程
-----------

2009年, 使用memcache(用于非持久化内容), memcacheDB(用于持久化+计数), memcacheDB是新浪在memcache的基础上，使用BerkeleyDB作为数据持久化的存储实现；

**面临的问题**

> 数据结构(Data Structure)需求越来越多, 但memcache中没有, 影响开发效率性能需求, 随着读操作的量的上升需要解决。 经历的过程有: 数据库读写分离(M/S)-->数据库使用多个Slave-->增加Cache (memcache)-->转到Redis解决写的问题：水平拆分，对表的拆分，将有的用户放在这个表，有的用户放在另外一个表； 可靠性需求: Cache的"雪崩"问题让人纠结 Cache面临着快速恢复的挑战 开发成本需求: Cache和DB的一致性维护成本越来越高(先清理DB, 再清理缓存, 不行啊, 太慢了!) 开发需要跟上不断涌入的产品需求: 硬件成本最贵的就是数据库层面的机器，基本上比前端的机器要贵几倍，主要是IO密集型，很耗硬件； 维护性复杂: 一致性维护成本越来越高； BerkeleyDB使用B树，会一直写新的，内部不会有文件重新组织；这样会导致文件越来越大；大的时候需要进行文件归档，归档的操作要定期做； 这样，就需要有一定的down time；

🥝 5.2 寻找开源软件的方式及评判标准
---------------------

*   对于开源软件，首先看其能做什么，但更多的需要关注它不能做什么，它会有什么问题？
*   上升到一定规模后，可能会出现什么问题，是否能接受？
*   google code上, 国外论坛找材料(国内比国外技术水平滞后5年）
*   观察作者个人的代码水平

🍓 5.3 业务使用方式
-------------

*   hash sets: 关注列表, 粉丝列表, 双向关注列表(key-value(field), 排序)
*   string(counter): 微博数, 粉丝数, ...(避免了select count(*) from ...)
*   sort sets(自动排序): TopN, 热门微博等, 自动排序
*   lists(queue): push/sub提醒,...

上述四种, 从精细化控制方面: hash sets和string(counter)推荐使用, sort sets和lists(queue)不推荐使用 还可通过二次开发，进行精简。比如: 存储字符改为存储整形, 16亿数据, 只需要16G内存 存储类型保存在3种以内，建议不要超过3种； 将memcache +myaql 替换为Redis： Redis作为存储并提供查询，后台不再使用mysql，解决数据多份之间的一致性问题；

### 🍎 5.3.1 List

**微博和微信公众号消息流**

```powershell
LPUSH msg:{id} 10086

```

**查看最新的微博消息**

```powershell
//LRANGE:返回列表key中指定区间内的元素,区间以偏移量start和stop指定
LRANGE  msg:{id} 0 5

```

**微信、微博在线用户抽奖**

```sql
//思路
1)点击参与抽奖加入集合
SADD key {userID}
2)查看参与抽奖所有用户
SMEMBERS key
3)抽count名中奖者
如果希望中奖用户依然保留在抽奖池里面
SRANDMENMBER key [count]
SRANDMENMBER 从集合key中选出count个元素,元素不从key中删除
如果不希望中奖用户依然保留在抽奖池里面
SPOP key [count]
SPOP从集合key中选出count个元素,元素从key中删除

```

**微信、微博、抖音点赞、收藏标签**

```sql
//思路
1)抖音大V视频点赞量
SADD like:{messageId}  {userId}
2)取消点赞
SREM like:{messageId} {userId}
移除集合 key 中的一个或多个 member 元素，不存在的 member 元素会被忽略
3)检查用户是否点过赞
SISMEMBER like:{messageId} {userId}
4)获取点赞的用户列表
SISMEMBER like:{messageId}
5)获取点赞用户数
返回集合 key 的基数(集合中元素的数量)
SCARD like:{messageId}

```

**微博、QQ共同关注的人,我关注的人也关注他,可能认识的人**

```sql
//思路
共同关注的人:
//两个集合的交集
SINTER simaSet yanguoSet
我关注的人也关注他
//我关注的人在他的集合里面,可以运用SISMENBER
SISMENBER simaSet yanguo
我可能认识的人
//他关注的人,不在我的集合里面,但是在我的朋友集合里面
SDIFF simaSet yanguoSet

```

### 🍅 5.3.2 Zset

**微博热搜、排行榜**

```sql
//思路
1)热搜
//新闻点击量
ZINCRBY HOTNEWS:20191115 1 李小璐离婚
为有序集合key中元素member的分值加上increment
2)展示当日排行版前十
ZREVRANGE hotNews:20191115 0 10 WITHSCORES
ZREVRANGE倒序获取有序集合key中start下标到stop下标的元素
3)七日搜索榜单计算
ZUNIONSTORE hotNews:20190722 7
并集计算
4)展示七日排行前十
ZREVRANGE hotNews:20190716-20190722 0 10 WITHSCORES
交集计算

```

### 🥒 5.3.3 Geo

**附近的人**

```sql
//思路
微博、微信、默默<附近的人>
滴滴打车、哈啰单车、美团单车<附近的车>
美团和饿了吗<附近的餐馆>

基于地理位置
比如GEORADIUS
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2

redis> GEORADIUS Sicily 15 37 200 km WITHDIST
1) 1) "Palermo"
   2) "190.4424"
2) 1) "Catania"
   2) "56.4413"

redis> GEORADIUS Sicily 15 37 200 km WITHCOORD
1) 1) "Palermo"
   2) 1) "13.361389338970184"
      2) "38.115556395496299"
2) 1) "Catania"
   2) 1) "15.087267458438873"
      2) "37.50266842333162"

redis> GEORADIUS Sicily 15 37 200 km WITHDIST WITHCOORD
1) 1) "Palermo"
   2) "190.4424"
   3) 1) "13.361389338970184"
      2) "38.115556395496299"
2) 1) "Catania"
   2) "56.4413"
   3) 1) "15.087267458438873"
      2) "37.50266842333162"

```

### 🍒 5.3.4 String

**微信公众号中文章阅读量、抖音中的城市访问量**

```sql
Redis提供了基于原子的加减操作,
这里面我们可以用计数器功能,
运用INCR对feild进行加加操作统计数据, 
INCR  article:readcount:{articleId}.

```

🌽 5.4 其他高频通用场景
---------------

**会话缓存（Session Cache）or 全页缓存（FPC） or 数据缓存（Data Cache）** ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/792a7d560f56422683da7fe86e497251~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=803&h=500&s=108403&e=png&b=fefefe)

>   Redis数据库是一种基于缓存的数据库，通过缓存来提升访问速度。   当我们访问某个网站时，第一个访问者向服务器发起请求，服务器返回一些静态文件。   如果某些静态文件的内容不经常变化，就可以将它们存储到Redis中，以便下一次访问时直接从Redis中返回，节省查找时间。如果Redis中没有需要的数据，就在文件中查找并返回数据，同时将数据存储到Redis中，以便下一个人访问时可以快速返回。   由于Redis基于内存存储，存储空间有限，因此需要对数据进行区分和设置存储时间，避免存储满后无法继续存储。对于访问频率高的数据，其存储时间会被不断更新延长，而对于访问频率低的数据，则会因为长时间未被访问而自动删除。这种数据存储方式非常常见，但需要根据实际情况进行设置和调整。

**bitMap(大数据处理）**

  源于redis特殊的数据结构位图（bitmap），用于数据量上亿的场景下，例如几亿用户系统的签到，去重登录次数统计，某用户是否在线状态等等。腾讯10亿用户，要几个毫秒内查询到某个用户是否在线，能怎么做？千万别说给每个用户建立一个key，然后挨个记（你可以算一下需要的内存会很恐怖，而且这种类似的需求很多。这里要用到位操作——使用setbit、getbit、bitcount命令。原理是：   redis内构建一个足够长的数组，每个数组元素只能是0和1两个值，然后这个数组的下标index用来表示用户id（必须是数字哈），那么很显然，这个几亿长的大数组就能通过下标和元素值（0和1）来构建一个记忆系统。

**分布式锁（重点）**

**基于Redis分布式锁实现**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/baf6f3084ba44ee38499612cde534ce9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1830&h=1000&s=196128&e=png&b=fffdfd)

**分布式锁对比**

| 分类 | 方案 | 实现原理 | 优点 | 缺点 |
| --- | --- | --- | --- | --- |
| 基于数据库 | 基于mysql 表唯一索引 | 1.表增加唯一索引 2.加锁：执行insert语句，若报错，则表明加锁失败 3.解锁：执行delete语句 | 完全利用DB现有能力，实现简单 | 1.锁无超时自动失效机制，有死锁风险 2.不支持锁重入，不支持阻塞等待 3.操作数据库开销大，性能不高 |
| 基于MongoDB findAndModify原子操作 | 1.加锁：执行findAndModify原子命令查找document，若不存在则新增  
2.解锁：删除document | 实现也很容易，较基于MySQL唯一索引的方案，性能要好很多 | 1.大部分公司数据库用MySQL，可能缺乏相应的MongoDB运维、开发人员 2.锁无超时自动失效机制 |  |
| 基于分布式协调系统 | 基于ZooKeeper | 1.加锁：在/lock目录下创建临时有序节点，判断创建的节点序号是否最小。若是，则表示获取到锁；否，则则watch /lock目录下序号比自身小的前一个节点 2.解锁：删除节点 | 1.由zk保障系统高可用 2.Curator框架已原生支持系列分布式锁命令，使用简单 | 需单独维护一套zk集群，维保成本高 |
| 基于缓存 | 基于redis命令 | 1\. 加锁：执行setnx，若成功再执行expire添加过期时间 2. 解锁：执行delete命令 | 实现简单，相比数据库和分布式系统的实现，该方案最轻，性能最好 | 1.setnx和expire分2步执行，非原子操作；若setnx执行成功，但expire执行失败，就可能出现死锁 2.delete命令存在误删除非当前线程持有的锁的可能 3.不支持阻塞等待、不可重入 |
| 基于redis Lua脚本能力 | 1\. 加锁：执行SET lock\_name random\_value EX seconds NX 命令  
2\. 解锁：执行Lua脚本，释放锁时验证random\_value -- ARGV\[1\]为random\_value, KEYS\[1\]为lock_nameif redis.call("get", KEYS\[1\]) == ARGV\[1\] then return redis.call("del",KEYS\[1\])else return 0end | 同上；实现逻辑上也更严谨，除了单点问题，生产环境采用用这种方案，问题也不大。 | 不支持锁重入，不支持阻塞等待 |  |

**分布式锁需满足四个条件**

*   互斥性。在任意时刻，只有一个客户端能持有锁。
*   不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
*   解铃还须系铃人，加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了，即不能误解锁。
*   具有容错性。只要大多数Redis节点正常运行，客户端就能够获取和释放锁。

**Redisson分布式锁的实现**

```java

Config config = new Config();
config.useSingleServer().setAddress("redis://127.0.0.1:5379").setPassword("123456").setDatabase(0);

RedissonClient redissonClient = Redisson.create(config);

RLock rLock = redissonClient.getLock(lockKey);
try {
    
     * 4.尝试获取锁
     * waitTimeout 尝试获取锁的最大等待时间，超过这个值，则认为获取锁失败
     * leaseTime   锁的持有时间,超过这个时间锁会自动失效（值应设置为大于业务处理的时间，确保在锁有效期内业务能处理完）
     */
    boolean res = rLock.tryLock((long)waitTimeout, (long)leaseTime, TimeUnit.SECONDS);
    if (res) {
        
    }
} catch (Exception e) {
    throw new RuntimeException("aquire lock fail");
}finally{
    
    rLock.unlock();
}

```

redisson这个框架重度依赖了Lua脚本和Netty，代码很牛逼，各种Future及FutureListener的异步、同步操作转换。

> 如果要手写一个分布式锁组件，怎么做？肯定要定义2个接口：加锁、解锁；大道至简，redisson的作者就是在加锁和解锁的执行层面采用Lua脚本，逼格高，而且重要有原子性保证。当然，redisson的作者毕竟牛逼，加锁和解锁过程中还巧妙地利用了redis的发布订阅功能。

**加锁和解锁Lua脚本剖析**:

1、加锁Lua脚本

**脚本参数**

| 参数 | 示例值 | 含义 |
| --- | --- | --- |
| KEY个数 | 1 | KEY个数 |
| KEYS\[1\] | my\_first\_lock_name | 锁名 |
| ARGV\[1\] | 60000 | 持有锁的有效时间：**毫秒** |
| ARGV\[2\] | 58c62432-bb74-4d14-8a00-9908cc8b828f:1 | **唯一标识**：获取锁时set的唯一值，实现上为redisson客户端**ID(UUID)+线程ID** |

**脚本内容**

```powershell
-- 若锁不存在：则新增锁，并设置锁重入计数为1、设置锁过期时间
if (redis.call('exists', KEYS[1]) == 0) then
    redis.call('hset', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
end;
 
-- 若锁存在，且唯一标识也匹配：则表明当前加锁请求为锁重入请求，故锁重入计数+1，并再次设置锁过期时间
if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then
    redis.call('hincrby', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
end;
 
-- 若锁存在，但唯一标识不匹配：表明锁是被其他线程占用，当前线程无权解他人的锁，直接返回锁剩余过期时间
return redis.call('pttl', KEYS[1]);

```

**脚本解读**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9995fe462bc453a9c39d1b7e09b0a20~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1426&h=1006&s=116243&e=png&b=ffffff)

当且仅当返回nil，才表示加锁成功；客户端需要感知加锁是否成功的结果

2、解锁Lua脚本

**脚本入参**

| 参数 | 示例值 | 含义 |
| --- | --- | --- |
| KEY个数 | 2 | KEY个数 |
| KEYS\[1\] | my\_first\_lock_name | 锁名 |
| KEYS\[2\] | redisson\_lock\_\_channel:{my\_first\_lock_name} | **解锁消息PubSub频道** |
| ARGV\[1\] | 0 | **redisson定义0表示解锁消息** |
| ARGV\[2\] | 30000 | 设置锁的过期时间；默认值30秒 |
| ARGV\[3\] | 58c62432-bb74-4d14-8a00-9908cc8b828f:1 | 唯一标识；同加锁流程 |

**脚本内容**

```powershell
-- 若锁不存在：则直接广播解锁消息，并返回1
if (redis.call('exists', KEYS[1]) == 0) then
    redis.call('publish', KEYS[2], ARGV[1]);
    return 1; 
end;
 
-- 若锁存在，但唯一标识不匹配：则表明锁被其他线程占用，当前线程不允许解锁其他线程持有的锁
if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then
    return nil;
end; 
 
-- 若锁存在，且唯一标识匹配：则先将锁重入计数减1
local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); 
if (counter > 0) then 
    -- 锁重入计数减1后还大于0：表明当前线程持有的锁还有重入，不能进行锁删除操作，但可以友好地帮忙设置下过期时期
    redis.call('pexpire', KEYS[1], ARGV[2]); 
    return 0; 
else 
    -- 锁重入计数已为0：间接表明锁已释放了。直接删除掉锁，并广播解锁消息，去唤醒那些争抢过锁但还处于阻塞中的线程
    redis.call('del', KEYS[1]); 
    redis.call('publish', KEYS[2], ARGV[1]); 
    return 1;
end;
 
return nil;

```

**脚本解读** ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c50fbe19cc5541349778324221be1ece~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1688&h=1334&s=149212&e=png&b=ffffff)
 广播解锁消息有什么用？ 是为了通知其他争抢锁阻塞住的线程，从阻塞中解除，并再次去争抢锁。

返回值0、1、nil有什么不一样？ 当且仅当返回1，才表示当前请求真正触发了解锁Lua脚本；

**加锁源码**

加锁源码中tryAcquire(leaseTime, unit, threadId)方法执行了加锁Lua脚本。 直接进入org.redisson.RedissonLock#tryLock(long, long, java.util.concurrent.TimeUnit)源码

```powershell
@Override
    public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        // 获取锁能容忍的最大等待时长
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        final long threadId = Thread.currentThread().getId();
 
        // 【核心点1】尝试获取锁，若返回值为null，则表示已获取到锁
        Long ttl = tryAcquire(leaseTime, unit, threadId);
        // lock acquired
        if (ttl == null) {
            return true;
        }
 
        // 还可以容忍的等待时长=获取锁能容忍的最大等待时长 - 执行完上述操作流逝的时间
        time -= (System.currentTimeMillis() - current);
        if (time <= 0) {
            acquireFailed(threadId);
            return false;
        }
 
        current = System.currentTimeMillis();
        // 【核心点2】订阅解锁消息，见org.redisson.pubsub.LockPubSub#onMessage
        /**
     * 4.订阅锁释放事件，并通过await方法阻塞等待锁释放，有效的解决了无效的锁申请浪费资源的问题：
     * 基于信息量，当锁被其它资源占用时，当前线程通过 Redis 的 channel 订阅锁的释放事件，一旦锁释放会发消息通知待等待的线程进行竞争
     * 当 this.await返回false，说明等待时间已经超出获取锁最大等待时间，取消订阅并返回获取锁失败
     * 当 this.await返回true，进入循环尝试获取锁
     */
        final RFuture<RedissonLockEntry> subscribeFuture = subscribe(threadId);
        //await 方法内部是用CountDownLatch来实现阻塞，获取subscribe异步执行的结果（应用了Netty 的 Future）
        if (!await(subscribeFuture, time, TimeUnit.MILLISECONDS)) {
            if (!subscribeFuture.cancel(false)) {
                subscribeFuture.addListener(new FutureListener<RedissonLockEntry>() {
                    @Override
                    public void operationComplete(Future<RedissonLockEntry> future) throws Exception {
                        if (subscribeFuture.isSuccess()) {
                            unsubscribe(subscribeFuture, threadId);
                        }
                    }
                });
            }
            acquireFailed(threadId);
            return false;
        }
 
        // 订阅成功
        try {
            // 还可以容忍的等待时长=获取锁能容忍的最大等待时长 - 执行完上述操作流逝的时间
            time -= (System.currentTimeMillis() - current);
            if (time <= 0) {
                // 超出可容忍的等待时长，直接返回获取锁失败
                acquireFailed(threadId);
                return false;
            }
 
            while (true) {
                long currentTime = System.currentTimeMillis();
                // 尝试获取锁；如果锁被其他线程占用，就返回锁剩余过期时间【同上】
                ttl = tryAcquire(leaseTime, unit, threadId);
                // lock acquired
                if (ttl == null) {
                    return true;
                }
 
                time -= (System.currentTimeMillis() - currentTime);
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }
 
                // waiting for message
                currentTime = System.currentTimeMillis();
 
                // 【核心点3】根据锁TTL，调整阻塞等待时长；
                // 注意：这里实现非常巧妙，1、latch其实是个信号量Semaphore，调用其tryAcquire方法会让当前线程阻塞一段时间，避免了在while循环中频繁请求获取锁；
               //2、该Semaphore的release方法，会在订阅解锁消息的监听器消息处理方法org.redisson.pubsub.LockPubSub#onMessage调用；当其他线程释放了占用的锁，会广播解锁消息，监听器接收解锁消息，并释放信号量，最终会唤醒阻塞在这里的线程。
                if (ttl >= 0 && ttl < time) {
                    getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                } else {
                    getEntry(threadId).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
                }
 
                time -= (System.currentTimeMillis() - currentTime);
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }
            }
        } finally {
            // 取消解锁消息的订阅
            unsubscribe(subscribeFuture, threadId);
        }
    }

```

tryAcquire的实现，真的就是执行Lua脚本！

```powershell
private Long tryAcquire(long leaseTime, TimeUnit unit, long threadId) {
    // tryAcquireAsync异步执行Lua脚本，get方法同步获取返回结果
    return get(tryAcquireAsync(leaseTime, unit, threadId));
}
 
//  见org.redisson.RedissonLock#tryAcquireAsync
private <T> RFuture<Long> tryAcquireAsync(long leaseTime, TimeUnit unit, final long threadId) {
    if (leaseTime != -1) {
        // 实质是异步执行加锁Lua脚本
        return tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
    }
    RFuture<Long> ttlRemainingFuture = tryLockInnerAsync(commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
    ttlRemainingFuture.addListener(new FutureListener<Long>() {
        @Override
        public void operationComplete(Future<Long> future) throws Exception {
            //先判断这个异步操作有没有执行成功，如果没有成功，直接返回，如果执行成功了，就会同步获取结果
            if (!future.isSuccess()) {
                return;
            }
 
            Long ttlRemaining = future.getNow();
            // lock acquired
            //如果ttlRemaining为null，则会执行一个定时调度的方法scheduleExpirationRenewal
            if (ttlRemaining == null) {
                scheduleExpirationRenewal(threadId);
            }
        }
    });
    return ttlRemainingFuture;
}
 
// 见org.redisson.RedissonLock#tryLockInnerAsync
<T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
    internalLockLeaseTime = unit.toMillis(leaseTime);
 
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
              "if (redis.call('exists', KEYS[1]) == 0) then " +
                  "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                  "return nil; " +
              "end; " +
              "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                  "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                  "return nil; " +
              "end; " +
              "return redis.call('pttl', KEYS[1]);",
                Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
}

```

**加锁总结**

> 尝试获取锁，这一步是通过执行加锁Lua脚本来做； 若第一步未获取到锁，则去订阅解锁消息，当获取锁到剩余过期时间后，调用信号量方法阻塞住，直到被唤醒或等待超时 一旦持有锁的线程释放了锁，就会广播解锁消息。于是，第二步中的解锁消息的监听器会释放信号量，获取锁被阻塞的那些线程就会被唤醒，并重新尝试获取锁。

RedissonLock中的变量internalLockLeaseTime,默认值是30000毫秒 调用tryLockInnerAsync()传入的一个从连接管理器获取的getLockWatchdogTimeout(),默认值也是30000毫秒 这些都和redisson官方文档所说的**watchdog**机制有关，看门狗，还是很形象的描述这一机制

**看门狗到底做了什么?**

先思考一个问题:

> 假设在一个分布式环境下，多个服务实例请求获取锁，其中服务实例1成功获取到了锁，在执行业务逻辑的过程中，服务实例突然挂掉了或者hang住了，那么这个锁会不会释放，什么时候释放？回答这个问题，自然想起来之前我们分析的lua脚本，其中第一次加锁的时候使用pexpire给锁key设置了过期时间，默认30000毫秒，由此来看如果服务实例宕机了，锁最终也会释放，其他服务实例也是可以继续获取到锁执行业务。但是要是30000毫秒之后呢，要是服务实例1没有宕机但是业务执行还没有结束，所释放掉了就会导致线程问题，这个redisson是怎么解决的呢？

**自动延长锁有效期的机制**

异步执行完lua脚本执行完成之后，设置了一个监听器，来处理异步执行结束之后的一些工作。在操作完成之后会去执行operationComplete方法，先判断这个异步操作有没有执行成功，如果没有成功，直接返回，如果执行成功了，就会同步获取结果，如果ttlRemaining为null，则会执行一个定时调度的方法**scheduleExpirationRenewal**,回想一下之前的lua脚本，当加锁逻辑处理结束，返回了一个nil;如此说来 就一定会走定时任务了。

> **定时调度scheduleExpirationRenewal代码**

```powershell
private void scheduleExpirationRenewal(final long threadId) {
        if (expirationRenewalMap.containsKey(getEntryName())) {
            return;
        }
 
        Timeout task = commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
            @Override
            public void run(Timeout timeout) throws Exception {
                
                RFuture<Boolean> future = commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
                        "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                            "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                            "return 1; " +
                        "end; " +
                        "return 0;",
                          Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
                
                future.addListener(new FutureListener<Boolean>() {
                    @Override
                    public void operationComplete(Future<Boolean> future) throws Exception {
                        expirationRenewalMap.remove(getEntryName());
                        if (!future.isSuccess()) {
                            log.error("Can't update lock " + getName() + " expiration", future.cause());
                            return;
                        }
                        
                        if (future.getNow()) {
                            // reschedule itself
                            scheduleExpirationRenewal(threadId);
                        }
                    }
                });
            }
        }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);
 
        if (expirationRenewalMap.putIfAbsent(getEntryName(), task) != null) {
            task.cancel();
        }
    }

```

首先，会先判断在expirationRenewalMap中是否存在了entryName，这是个map结构，主要还是判断在这个服务实例中的加锁客户端的锁key是否存在，如果已经存在了，就直接返回；第一次加锁，肯定是不存在的，接下来就是搞了一个TimeTask，延迟internalLockLeaseTime/3之后执行，这里就用到了文章一开始就提到奇妙的变量，算下来就是大约10秒钟执行一次，调用了一个异步执行的方法：

```powershell
                RFuture<Boolean> future = commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
                        "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                            "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                            "return 1; " +
                        "end; " +
                        "return 0;",
                          Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));

```

> 任务调度异步执行并设置监听器，更新锁过期时间失败则返回；获取异步执行结果为true则每隔10秒执行一次，续约锁的过期时间到30000毫秒，保证锁一直在手中。

> 加锁成功后，在执行业务期间，其他服务实例或者当前客户端的其他线程若尝试加锁，都会被阻塞。加锁lua代码中，若锁key存在且当前客户端唯一key也存在，则自增重入次数，设置过期时间，并返回nil，实现锁的可重入性。定时调度任务会继续执行，完成锁key过期时间的续约。

> 如果没有出现锁续约的情况，直接返回当前锁的剩余有效期，不执行续约逻辑。若加锁成功，则返回；若加锁失败，则死循环尝试加锁，等待一段时间后重试，直到第一个服务实例释放锁。这样实现了锁的互斥。

锁释放的逻辑

锁释放的逻辑，调用lock.unlock()会异步执行lua脚本，判断锁key是否存在，不存在则返回；存在则判断唯一key的值是否存在，不存在返回nil；存在则自减一并判断是否大于零，大于零返回0，否则删除锁key并返回1。如果返回1，取消之前的续约任务，失败则设置状态。

**源码解读**

org.redisson.RedissonLock#unlock代码

```powershell
@Override
    public void unlock() {
        // 执行解锁Lua脚本，这里传入线程id，是为了保证加锁和解锁是同一个线程，避免误解锁其他线程占有的锁
        Boolean opStatus = get(unlockInnerAsync(Thread.currentThread().getId()));
        if (opStatus == null) {
            throw new IllegalMonitorStateException("attempt to unlock lock, not locked by current thread by node id: "
                    + id + " thread-id: " + Thread.currentThread().getId());
        }
        if (opStatus) {
            cancelExpirationRenewal();
        }
    }
 
// 见org.redisson.RedissonLock#unlockInnerAsync
protected RFuture<Boolean> unlockInnerAsync(long threadId) {
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            "if (redis.call('exists', KEYS[1]) == 0) then " +
                "redis.call('publish', KEYS[2], ARGV[1]); " +
                "return 1; " +
            "end;" +
            "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
                "return nil;" +
            "end; " +
            "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
            "if (counter > 0) then " +
                "redis.call('pexpire', KEYS[1], ARGV[2]); " +
                "return 0; " +
            "else " +
                "redis.call('del', KEYS[1]); " +
                "redis.call('publish', KEYS[2], ARGV[1]); " +
                "return 1; "+
            "end; " +
            "return nil;",
            Arrays.<Object>asList(getName(), getChannelName()), LockPubSub.unlockMessage, internalLockLeaseTime, getLockName(threadId));
 
}

```

**加锁&解锁流程串起来**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbb61d84058d47a7906e8dab1521d55f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2104&h=1548&s=364369&e=png&b=ffffff)
 总结

*   线程A抢到锁，线程B订阅解锁消息再尝试获取锁并阻塞等待。
*   线程A释放锁并广播解锁消息。
*   LockPubSub收到消息后释放信号量，线程B被唤醒后再次抢到锁并干完活解锁。

[参考地址1 redis性能优化](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fagonie201218%2Farticle%2Fdetails%2F130424297%3Fops_request_misc%3D%25257B%252522request%25255Fid%252522%25253A%252522170179053816800226510313%252522%25252C%252522scm%252522%25253A%25252220140713.130102334.pc%25255Fall.%252522%25257D%26request_id%3D170179053816800226510313%26biz_id%3D0%26utm_medium%3Ddistribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~hot_rank-5-130424297-null-null.142%255Ev96%255Epc_search_result_base2%26utm_term%3Dredis%25E6%2580%25A7%25E8%2583%25BD%25E4%25BC%2598%25E5%258C%2596%26spm%3D1018.2226.3001.4449 "https://blog.csdn.net/agonie201218/article/details/130424297?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170179053816800226510313%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=170179053816800226510313&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~hot_rank-5-130424297-null-null.142%5Ev96%5Epc_search_result_base2&utm_term=redis%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96&spm=1018.2226.3001.4449")

[参考地址2 redis性能优化](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fzhengzhaoyang122%2Farticle%2Fdetails%2F112293953%3Fops_request_misc%3D%25257B%252522request%25255Fid%252522%25253A%252522170179053816800226510313%252522%25252C%252522scm%252522%25253A%25252220140713.130102334.pc%25255Fall.%252522%25257D%26request_id%3D170179053816800226510313%26biz_id%3D0%26utm_medium%3Ddistribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~hot_rank-6-112293953-null-null.142%255Ev96%255Epc_search_result_base2%26utm_term%3Dredis%25E6%2580%25A7%25E8%2583%25BD%25E4%25BC%2598%25E5%258C%2596%26spm%3D1018.2226.3001.4449 "https://blog.csdn.net/zhengzhaoyang122/article/details/112293953?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170179053816800226510313%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=170179053816800226510313&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~hot_rank-6-112293953-null-null.142%5Ev96%5Epc_search_result_base2&utm_term=redis%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96&spm=1018.2226.3001.4449")

* * *

[Redis高可用参考地址](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_44183721%2Farticle%2Fdetails%2F126195582%3Fops_request_misc%3D%25257B%252522request%25255Fid%252522%25253A%252522170179108616800213076720%252522%25252C%252522scm%252522%25253A%25252220140713.130102334.pc%25255Fall.%252522%25257D%26request_id%3D170179108616800213076720%26biz_id%3D0%26utm_medium%3Ddistribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~hot_rank-7-126195582-null-null.142%255Ev96%255Epc_search_result_base2%26utm_term%3Dredis%25E9%25AB%2598%25E5%258F%25AF%25E7%2594%25A8%26spm%3D1018.2226.3001.4187 "https://blog.csdn.net/weixin_44183721/article/details/126195582?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170179108616800213076720%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=170179108616800213076720&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~hot_rank-7-126195582-null-null.142%5Ev96%5Epc_search_result_base2&utm_term=redis%E9%AB%98%E5%8F%AF%E7%94%A8&spm=1018.2226.3001.4187")

[Redis事务](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fqq_37284798%2Farticle%2Fdetails%2F128774826%3Fops_request_misc%3D%25257B%252522request%25255Fid%252522%25253A%252522170179146516800227449697%252522%25252C%252522scm%252522%25253A%25252220140713.130102334.pc%25255Fall.%252522%25257D%26request_id%3D170179146516800227449697%26biz_id%3D0%26utm_medium%3Ddistribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~hot_rank-4-128774826-null-null.142%255Ev96%255Epc_search_result_base2%26utm_term%3Dredis%25E4%25BA%258B%25E5%258A%25A1%26spm%3D1018.2226.3001.4187 "https://blog.csdn.net/qq_37284798/article/details/128774826?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170179146516800227449697%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=170179146516800227449697&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~hot_rank-4-128774826-null-null.142%5Ev96%5Epc_search_result_base2&utm_term=redis%E4%BA%8B%E5%8A%A1&spm=1018.2226.3001.4187")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ec2950cd2074f1981007d8b2fd88573~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1440&h=720&s=1102034&e=png&a=1&b=e67e77)

> Redis（Remote Dictionary Server）是一个开源的高性能键值对（key-value）存储系统，通常用作数据库、缓存和消息代理。它支持多种数据结构，如字符串、哈希表、列表、集合和有序集合。Redis具有原子性操作，能够在单个命令中实现数据的读取、写入和删除，这使得它在处理高并发请求时表现出色。

*   高速缓存：Redis使用内存存储数据，因此它可以在大多数操作中提供非常快的响应时间。
*   原子性操作：Redis的所有操作都是原子的，这意味着它们要么完全执行，要么完全不执行。
*   数据结构多样性：Redis支持多种数据结构，使得它可以用作多种用途，如数据库、缓存和消息代理。
*   持久化：虽然主要使用内存存储，但Redis也支持将数据持久化到磁盘。
*   发布/订阅：Redis支持发布/订阅模型，使其成为实现实时应用的理想选择。
*   事务处理：Redis支持事务处理，即一系列按顺序执行的命令，要么全部执行，要么全部不执行。
*   Lua脚本处理：Redis支持Lua脚本处理，可以在服务器端执行一段Lua脚本。
*   分布式：通过Redis的分片，可以很容易地实现数据的分布式存储和处理。

Redis是一个功能丰富、性能卓越的键值对存储系统，适用于各种应用场景，无论是作为数据库、缓存还是消息代理，都能提供出色的性能和可靠性。