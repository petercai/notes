# 图解Redis，谈谈Redis的持久化，RDB快照与AOF日志
今天分享一下 Redis 的持久化、事务、管道相关的知识点，实现快速入门，丰富个人简历，提高面试 level，给自己增加一点谈资，秒变面试小达人，BAT 不是梦。

**Redis 是一个键值对数据库**，服务器中通常包含着任意个非空数据库，而每个非空数据库中又可以包含任意个键值对，我们将是服务器中的非空数据库以及它们的键值对统称为数据库状态。

![](_assets/eba3efbdc8cbf3899cfcd89c1c3c0d32.png)

**Redis 是内存数据库**，它将数据存储在内存中，如果不能将内存中的数据持久化到磁盘中，Redis 突然宕机，会导致数据丢失。

为了解决这个问题，Redis 提供了 RDB 持久化功能，RDB 持久化会将 Redis 在内存中的数据库状态保存到磁盘中，避免数据意外丢失。

### 一、RDB 持久化

RDB，英文全称 Redis DataBase，在指定的时间间隔，将内存中的数据写入磁盘，待恢复时再将磁盘中的数据写入内存。

#### 1、自动触发

redis.conf 配置文件中，`save <seconds> <changes>`，比如 save 60 20，在 60 秒内修改 20 次就自动触发 RDB 备份。

#### 2、手动触发

通过 save 和 bgsave 手动触发 RDB 备份。

（1）**save**，在主程序中执行会阻塞当前 Redis 服务器，直到 RDB 持久化完成，也就是说 save 持久化期间，Redis 就不能用了，禁止使用。

（2）**bgsave**，不阻塞当前 Redis 服务器，Redis 会 fork 一个子进程，异步进行快照操作。

禁用快照：`redis-cli config set save ""`。

#### 3、设置保存条件

服务器程序会根据 save 选项所设置的保存条件，设置服务器状态 **redisServer** 结构的 **saveparams** 属性。

**dirty 计数器**记录距离上次成功执行 save 命令后，服务器对数据状态进行了多少次修改。

**lastsave** 属性是一个 UNIX 时间戳，记录了服务器上一次成功执行 save 命令的时间。

属性是一个数组，数组中的每个元素都是一个 saveparam 结构，每个 saveparam 结构都保存了一个 save 选项设置的保存条件。

![](_assets/43c167b1616b956d39a3ed5673a4b518.png)

以上就是 Redis 服务器根据 save 选项所设置的保存条件，自动执行 bgsave 命令，进行间隔性数据保存的实现原理。

#### 4、加解密

RDB 持久化功能所生成的 RDB 文件是一个经过压缩的二进制文件，通过该文件可以还原生成 RDB 文件时的数据库状态。

#### 5、RDB 持久化优缺点

（1）优点

1.  适合大规模的数据备份、恢复；
    
2.  可以定时备份；
    
3.  对数据一致性、完整性要求不高的场景可以使用；
    
4.  RDB 文件在内存中的加载速度要比 AOF 块；
    

（2）缺点

1.  因为是定时备份，如果 Redis 宕机的话，会丢失一部分数据；
    
2.  因为是数据的全量同步，如果数据量过大，会产生大量 IO，严重影响服务器性能；
    
3.  RDB 持久化时，会 fork 一个子进程，如果数据过大， 可能会导致服务器的瞬间延迟；fork 的时候内存中的数据会被克隆一份，是否会造成内存溢出，值得考虑；
    

#### 6、哪些情况会触发 RDB 持久化？

（1）redis.conf 中的定时配置；

（2）手动执行 save、bgsave 命令；

（3）执行 flushall、flushdb 命令，产生的.rdb 文件是空的；

（4）执行 shutdown 命令，且没有设置 AOF 持久化；

（5）主动复制时，主节点自动触发；

注意：save 和 bgsave 不能同时执行，如果 bgsave 命令正在执行，那么客户端发送的 save 命令会被拒绝执行，服务器禁止 save 和 bgsave 同时执行是为了避免父进程和子进程同时执行两个 rdbSave 调用，防止产生竞争条件。

创建 RDB 文件的实际工作是由 rdb.c/rdbSave 函数完成，save 和 bgsave 会以不同的方式调用这个函数。

### 二、AOF 持久化

**AOF**，Append Only File，以日志的方式记录每一个操作命令，只追加不修改，Redis 启动时会读取该文件，重新执行一遍之前的写操作命令，达到恢复数据的效果。

默认不开启，需要更改为 appendonly yes，开启 AOF 快照功能。

#### 1、AOF 持久化过程

（1）在执行 Redis 命令时，Redis 会将这些命令放入 **AOF 缓存**中，AOF 缓存是位于内存中的一个区域，AOF 缓存中的命令达到一定数量后，批量写入磁盘，**避免频繁的磁盘 IO 操作**；

（2）根据 AOF 缓冲区同步文件的三种写回策略，将命令写到磁盘的 AOF 文件中；

（3）随着写入 AOF 内容的增加，会根据规则进行命令的合并，起到压缩 AOF 文件的效果；

（4）当 Redis 重启时，会根据 AOF 文件，依此执行命令，恢复内存数据；

很多人会问，如果加入到 AOF 缓存的数据还没写入磁盘，此时服务器宕机了，数据不会产生丢失吗？

系统提供了 fsync 和 fdatasync 两个同步函数，他们可以强制操作系统立即将缓存区中的数据写入到磁盘里，确保写入数据的完整性。

#### 2、appendfsync 的选项值

（1）**Always**，同步写回，每个写命令执行完立刻同步到磁盘；最多只会丢失一个命令的数据，最安全，但是效率最低；

（2）**everysec**，每秒写回；它最多丢失 1 秒的数据；

（3）**no**，操作系统控制的写回，每个命令执行完，存到 AOF 缓存中，由操作系统决定，啥时候将命令写入到磁盘 AOF 文件中；

#### 3、AOF 持久化优缺点

（1）优点

更好的保护数据不丢失，性能高，可做紧恢复。

（2）缺点

1.  等量级的命令，AOF 持久化文件要大于 RDB 持久化文件；
    
2.  恢复速度慢于 RDB；
    

#### 4、数据恢复顺序和加载流程

当同时开启 AOF 和 RDB 的时候，会优先加载 AOF 文件来恢复数据，因为通常情况下 AOF 文件比 RDB 文件备份的数据要完整。

![](_assets/18b1185771a219cb0a106962b346fc0a.png)

### 三、Redis 事务

#### 1、Redis 事务是什么？

可以一次性顺序执行多个命令，不可被其它命令插队，但是，Redis 的事务不能回滚。

#### 2、Redis 事务常用命令

（1）**discard**，取消事务，放弃执行事务内的所有命令；

（2）**exec**，执行所有命令；

（3）**multi**，标记一个事务块的开始；

（4）**unwatch**，取消 watch 命令对所有 key 的监视；

（5）**watch key**，监视某个 key，如果这个 key 在事务执行之前被其它命令改动，那么事务将被打断；

#### 3、事务命令执行顺序

![](_assets/999aa7dd40c291176a2c990e131b3c2f.png)

#### 4、事务的 ACID 特性

在 Redis 中，事务总是具有**原子性（Atomicity）**、**一致性（Consistency）**、**隔离性（Isolation）**，并且 Redis 运行在某些特殊模式下，也具有**持久性（Durability）**。

**（1）原子性**

事务具有原子性，指的是事务中的多个操作当做一个整体来执行，服务器要么就执行事务中的全部操作，要么就一个操作也不执行。

**（2）一致性**

如果数据库在执行事务之前是一致的，那么在事务执行之后，无论事务执行是否成功，数据库也应该是一致的。

**（3）隔离性**

隔离性指即使数据库中有多个事务并发地执行，各个事务之间也不会互相影响，并且在并发状态下执行的事务和串行执行的事务产生的结果完全相同。

**（4）持久性**

当一个事务执行完毕时，执行这个事务所得的结果已经被保存到硬盘里，即使服务器此时宕机，执行事务所得的结果也不会丢失。

### 四、Redis 管道

#### 1、Redis 管道释义

Redis 是一种基于客户端-服务端模型以及请求响应协议的 TCP 服务，每个请求会遵循如下操作：

（1）客户端向服务端发送命令，① 发送命令；② 命令排队；③ 命令执行；④ 返回结果，监听 Socket 返回，通常以阻塞模

（2）式等待服务端响应；

服务端处理命令，并将结果返回给客户端；

上面两步走，称为 RTT，数据往返于两端的时间，Round Trip Time。

如果需要执行大量命令，那么新的命令要等待上一个命令执行完毕才能执行，会频繁的调用系统 IO，频繁发送网络请求，同时需要 Redis 调用多次 read()和 write()系统方法，系统方法会将数据从用户态转移到内核态，性能不佳。

#### 2、注意事项

（1）pipeline 缓存的指令只会依此执行，不保证原子性，如果执行中发生异常，也会继续执行后面的指令；

（2）使用 pipeline 组装的命令个数不能太多，不然可能会造成阻塞时间过长的现象，同时服务端也会被迫回复一个队列答复，占用过多内存；

#### 3、代码实战

编写一个命令脚本，添加几个不同类型的数据。

通过 `cat run.txt | redis-cli -a 111111 --pipe` 命令，体验 Redis 管道效果。
