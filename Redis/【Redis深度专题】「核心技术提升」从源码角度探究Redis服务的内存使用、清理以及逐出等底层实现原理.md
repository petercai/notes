# 【Redis深度专题】「核心技术提升」从源码角度探究Redis服务的内存使用、清理以及逐出等底层实现原理
背景介绍
----

Redis作为一种高性能的内存NoSQL数据库，其容量受限于最大内存的限制。用户在使用阿里云Redis时，除了对性能和稳定性有较高的要求外，对内存占用也非常敏感。然而，在实际使用中，一些用户可能会发现他们的线上实例的内存占用比预期的要大。

### 内存较高的场景

在使用Redis时，以下是一些可能导致内存占用较高的因素：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8818dd76dfd548f7b085eb1408b0a85a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=961&h=558&s=41897&e=png&b=c6dde2)

*   数据存储格式：Redis支持不同的数据结构，如字符串、哈希、列表等。不同数据结构的存储方式和编码格式可能会导致不同的内存占用。确保选择合适的数据结构和存储方式，可以有效地控制内存占用。
    
*   数据过期时间：Redis允许为存储的数据设置过期时间。如果数据没有正确设置过期时间或过期时间设置不合理，可能会导致数据过多地积累在内存中，从而增加了内存占用。
    
*   内存碎片化：Redis使用内存分配机制来存储数据，而这可能导致内存碎片化。当有键过期或被删除时，内存不会立即释放，而是会留下一些碎片空间。内存碎片化可能导致总体内存占用率的增加。
    

### 降低Redis的内存占用

通过合理的数据结构选择、设置合理的数据过期时间和优化内存使用等措施，可以帮助降低Redis的内存占用，提高性能和稳定性。定期监控和优化内存使用是持续保持Redis在高性能和低内存占用之间平衡的关键。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce2d67afe2b74f058a1d76890323b84c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1262&h=768&s=61560&e=png&b=ffffff)

*   优化数据结构和存储方式：根据实际需求选择合适的数据结构和对应的存储方式，例如使用压缩列表来存储较短的列表。
    
*   合理设置数据过期时间：为存储的数据设置合适的过期时间，确保数据按需进行过期，并及时释放内存空间。
    
*   使用字节淘汰机制：配置合适的内存淘汰机制，以便及时回收和释放过期数据和不活跃数据占用的内存空间。
    
*   监控和优化内存使用：定期监控Redis实例的内存使用情况，及时发现和解决占用较高的问题，并进行适当的调优。
    

### 额外内存较高的场景

实际上，在Redis实例中，除了保存原始键值对所需的内存开销之外，还存在一些运行时产生的额外内存开销。其中，两个主要因素是：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02b4012088fc4f43bdd7b8fc9e9ddbd5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=817&h=218&s=29858&e=png&b=f4ecfa)

*   垃圾数据和过期键所占空间：在Redis中，当键被删除或过期时，并不会立即释放相应的内存空间。因此，即使键已过期或被删除，仍然会占用内存空间，这可能导致垃圾数据和过期键占据内存。
    
*   渐进式Rehash导致的未及时删除：当Redis进行渐进式Rehash过程时，会在新哈希槽中分配内存空间，并逐步迁移旧哈希槽中的键。然而，在迁移期间，如果有新的写入操作，可能会导致旧哈希槽中的键未被及时删除，从而占据了额外的内存空间。
    

为了优化和降低Redis实例的内存占用，可以考虑以下优化措施：

#### 定期清理垃圾数据和过期键

通过适当设置数据的过期时间，或使用定期任务进行清理操作，及时清理过期的键和垃圾数据，以释放占用的内存空间。

#### 执行Rehash后进行内存回收

在Redis执行Rehash或使用槽位分配器时，可以通过执行KEYS命令或使用SCAN迭代器等方式，确保未及时迁移的键被删除，从而释放旧哈希槽中的内存空间。

> **监控和优化Redis的内存使用情况是关键。定期检查内存占用，并根据需求进行调整和优化，有助于提高Redis实例的性能和稳定性，并避免额外的内存开销。同时，合理设置键的过期时间，可以避免不必要的内存占用**。

### 渐进式Rehash操作

Redis中的字典渐进式Rehash是指在字典扩容时，Redis会逐步迁移旧的哈希桶中的键值对到新的更大的哈希桶中的过程。

#### 渐进式Rehash操作的原因

会出现渐进式Rehash的原因是当字典中的键值对数量增加时，为了保持哈希表的性能，Redis会定期扩容字典。而在字典扩容的过程中，Redis会同时创建一个新的更大的哈希桶，并将旧哈希桶中的键值对逐步迁移到新的哈希桶中。

##### 具体的渐进式Rehash过程

渐进式Rehash，Redis可以避免在一次性进行哈希桶迁移时造成的性能问题，同时可以保证旧的哈希桶仍然可以被正常访问。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ec64810bd9c4b84949bd648c2e17404~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=3714&h=2346&s=327033&e=png&a=1&b=fefefe)

1.  Redis在旧哈希桶中选择一个非空的桶进行rehash。
    
2.  将选择的桶中的键值对逐个迁移到新的哈希桶中。
    
3.  在迁移的过程中，对于新哈希桶中的每个桶，将新桶指向旧桶的下一个桶，使得迁移的键值对能够被访问到。
    
4.  当选择的桶中的所有键值对都迁移到新哈希桶后，将旧哈希桶删除。
    
5.  重复以上步骤，直到所有的桶都被迁移到新哈希桶中。
    

渐进式Rehash可以在很长的时间内进行，逐渐将键值对从旧哈希桶迁移到新哈希桶，减少了操作的复杂性和对性能的影响。

### 其他方面开销

Redis在管理数据时涉及到底层数据结构的开销、客户端信息和读写缓冲区等方面的管理。此外，在Redis的主从复制过程中，进行bgsave操作可能会产生一些额外开销。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34d348ac1f43413d954cac6091c2ef03~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=926&h=512&s=42084&e=png&b=261f1c)

*   客户端信息管理：在使用Redis时，应合理管理客户端的连接和会话信息。避免创建过多的无效连接或长时间闲置的连接，以减轻服务器负载，并提高资源利用率。
    
*   读写缓冲区管理：在数据读写过程中，Redis使用缓冲区来提高性能。可以通过适当配置缓冲区的大小和处理策略，以平衡内存使用和性能需求。根据实际情况，可以调整Redis的配置参数，如 `client-output-buffer-limit` 和 `client-query-buffer-limit`，以优化缓冲区管理。
    
*   Redis主从复制：在Redis的主从复制过程中，进行bgsave操作可能会导致额外的开销，因为bgsave会将数据快照保存到磁盘中。为了减少对主节点的影响，可以考虑在从节点上执行bgsave操作，避免主节点的性能影响。此外，合理设置Redis的复制策略和配置参数可对主从复制效率进行优化。
    

Redis过期数据清理策略
-------------

为了避免大量过期键的一次性清理对Redis服务性能造成负面影响，Redis已经优化，在空闲时清理过期键。这种策略可以有效减少对服务性能的干扰，使得清理工作更加平稳和高效。

### 过期数据清理时机

1.  Redis在逐出过期键时，具体的时机是在访问键时进行判断，如果发现键已过期，则将其逐出。这种机制确保了对过期键的及时清理，同时也保证了数据访问时的高效性和实时性。

```c
robj *lookupKeyRead(redisDb *db, robj *key) {
    robj *val;
    expireIfNeeded(db,key);
    val = lookupKey(db,key);
    ...
    return val;
}

```

2.  在CPU空闲时，定期的serverCron任务会逐出部分过期的Key。

```c
aeCreateTimeEvent(server.el, 1, serverCron, NULL, NULL)
    int serverCron(struct aeEventLoop *eventLoop, long long id, 
        void *clientData) {
            ...
            databasesCron();
            ...
        }
        void databasesCron(void) {
            
             + as master will synthesize DELs for us. */
            if (server.active_expire_enabled && server.masterhost == NULL)
                activeExpireCycle(ACTIVE_EXPIRE_CYCLE_SLOW);
            ...
        }
}

```

3.  每次事件循环执行时，会逐出部分过期的Key。

```c
        void aeMain(aeEventLoop *eventLoop) {
            eventLoop->stop = 0;
            while (!eventLoop->stop) {
                if (eventLoop->beforesleep != NULL)
                    eventLoop->beforesleep(eventLoop);
                aeProcessEvents(eventLoop, AE_ALL_EVENTS);
            }
        }
        void beforeSleep(struct aeEventLoop *eventLoop) {
            ...
            
             - ASAP if a fast cycle is not needed). */
            if (server.active_expire_enabled && server.masterhost == NULL)
                activeExpireCycle(ACTIVE_EXPIRE_CYCLE_FAST);
            ...
        }

```

### 过期数据清理算法

通过动态调整清理频率、引入并行化处理机制、优化内存碎片整理策略以及持续监控和优化系统性能，可以有效提升Redis过期Key清理的性能，并在不影响正常服务的情况下实现长时间服务的性能最优化。

#### 优化方案分析

*   **根据系统负载和流量特征调整定时清理的频率**：通过监控系统负载和Key的过期频率，动态调整清理的时间间隔，确保能够及时清理过期Key，同时避免频繁的清理操作影响正常服务。
    
*   **通过合理设置最大时间限制并进行并行处理**： 针对清理操作的最大时间限制，可以引入并行化处理机制，将清理操作分批进行，并行处理以提高清理效率，可以确保清理操作在限定时间内完成，并且不会对正常服务产生明显影响。
    
*   **通过合理设置Redis的内存碎片整理策略**：可以减少内存碎片对过期Key清理效率的影响，进一步优化清理性能。
    
*   **建议监控系统的运行情况**：定期检查系统的清理效率和对服务的影响程度，并根据实际情况进行调整和优化，以确保系统长时间运行时的性能最优。
    

#### 清理数据具体的流程

1.  Redis配置项 `hz`，定义了 `serverCron` 任务的执行周期，默认为 10，即在 CPU 空闲时每秒执行 10 次。
    
2.  每次过期键清理的时间不超过 CPU 时间的 25%。例如，如果 `hz=1`，则每次清理的最大时间为 250 毫秒；如果 `hz=10`，则每次清理的最大时间为 25 毫秒。
    
3.  清理过程会依次遍历所有的数据库（db）。
    
4.  在清理过程中，从每个数据库随机选择 20 个键，并判断它们是否过期。如果键过期，则会被删除。如果有超过 5 个键过期，会重新执行步骤 4；否则，会继续遍历下一个数据库。
    
5.  在执行清理过程时，如果达到了 CPU 时间的 25%，就会退出清理过程。
    

#### 清理内容的实现

Redis的清理过程，包括执行周期、过期键清理的时间限制、遍历数据库和随机选择键进行过期判断的步骤。同时，提及了在清理过程中限制 CPU 时间的机制，以保证不过度消耗 CPU 资源。这样有助于理解清理过程的工作原理并优化 Redis 的性能。

### 关于hz参数的一些讨论

> **调高hz参数可以提升清理的频率，过期key可以更及时的被删除，但hz太高会增加CPU时间的消耗**;

代码分析如下:

```c
void activeExpireCycle(int type) {
    ...
    
     * per iteration. Since this function gets called with a frequency of
     * server.hz times per second, the following is the max amount of
     * microseconds we can spend in this function. */
    
    
    
    timelimit = 1000000*ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC/server.hz/100;
    ...
    
    for (j = 0; j < dbs_per_call; j++) {

        int expired;

        redisDb *db = server.db+(current_db % server.dbnum);

        
         * in the current DB we'll restart from the next. This allows to
         * distribute the time evenly across DBs. 
         */

        current_db++;

        
         * of the keys were expired. 
         */
        do {
            ...
            
            if (num > ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP)
                num = ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP;
            while (num--) {
                dictEntry *de;
                long long ttl;

                if ((de = dictGetRandomKey(db->expires)) == NULL) break;
                ttl = dictGetSignedIntegerVal(de)-now;
                if (activeExpireCycleTryExpire(db,de,now)) expired++;
            }

            if ((iteration & 0xf) == 0) { 
                long long elapsed = ustime()-start;
                latencyAddSampleIfNeeded("expire-cycle",elapsed/1000);
                if (elapsed > timelimit) timelimit_exit = 1;
            }

            if (timelimit_exit) return;

            
             * found expired in the current DB. */
            
            

        } while (expired > ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP/4);
    }
}

```

这是一个基于概率的简单算法，基本的假设是抽出的样本能够代表整个key空间，redis持续清理过期的数据直至将要过期的key的百分比降到了25%以下。这也意味着在长期来看任何给定的时刻已经过期但仍占据着内存空间的key的量最多为每秒的写操作量除以4.

由于算法采用的随机取key判断是否过期的方式，故几乎不可能清理完所有的过期Key;

### Redis数据逐出策略

#### 数据逐出时机

```c

int processCommand(redisClient *c) {
        ...
        
        **
        First we try to free some memory if possible (if there are volatile
        * keys in the dataset). If there are not the only thing we can do
        * is returning an error. */
        if (server.maxmemory) {
            int retval = freeMemoryIfNeeded();
            ...
    }
    ...
}

```

#### 数据逐出算法

在逐出算法中，根据用户设置的逐出策略，选出待逐出的key，直到当前内存小于最大内存值为主.

##### 可选逐出策略如下：

*   volatile-lru：从已设置过期时间的数据集（server.db\[i\].expires）中挑选最近最少使用的数据淘汰
*   volatile-ttl：从已设置过期时间的数据集（server.db\[i\].expires）中挑选将要过期的数据淘汰
*   volatile-random：从已设置过期时间的数据集（server.db\[i\].expires）中任意选择数据 淘汰
*   allkeys-lru：从数据集（server.db\[i\].dict）中挑选最近最少使用的数据淘汰
*   allkeys-random：从数据集（server.db\[i\].dict）中任意选择数据淘汰
*   no-enviction（驱逐）：禁止驱逐数据

具体代码如下

```c
int freeMemoryIfNeeded() {
    ...
    
    mem_used = zmalloc_used_memory();
    ...

    
    if (mem_used <= server.maxmemory) return REDIS_OK;

    
    if (server.maxmemory_policy == REDIS_MAXMEMORY_NO_EVICTION)
        return REDIS_ERR; 

    mem_freed = 0;
    mem_tofree = mem_used - server.maxmemory;
    long long start = ustime();
    latencyStartMonitor(latency);
    while (mem_freed < mem_tofree) {
        int j, k, keys_freed = 0;
        for (j = 0; j < server.dbnum; j++) {
            
            long bestval = 0; 
            sds bestkey = NULL;
            struct dictEntry *de;
            redisDb *db = server.db+j;
            dict *dict;

            if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_LRU ||
                server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_RANDOM)
            {
                dict = server.db[j].dict;
            } else {
                dict = server.db[j].expires;
            }
            if (dictSize(dict) == 0) continue;

            
            if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_RANDOM ||
                server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_RANDOM)
            {
                de = dictGetRandomKey(dict);
                bestkey = dictGetKey(de);
            }

            
            else if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_LRU ||
                server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_LRU)
            {
                for (k = 0; k < server.maxmemory_samples; k++) {
                    sds thiskey;
                    long thisval;
                    robj *o;

                    de = dictGetRandomKey(dict);
                    thiskey = dictGetKey(de);
                    
                     * to locate the real key, as dict is set to db->expires.  **/
                    if (server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_LRU)
                        de = dictFind(db->dict, thiskey);
                    o = dictGetVal(de);
                    thisval = estimateObjectIdleTime(o);

                    
                    if (bestkey == NULL || thisval > bestval) {
                        bestkey = thiskey;
                        bestval = thisval;
                    }
                }
            }

            
            else if (server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_TTL) {
                for (k = 0; k < server.maxmemory_samples; k++) {
                    sds thiskey;
                    long thisval;

                    de = dictGetRandomKey(dict);
                    thiskey = dictGetKey(de);
                    thisval = (long) dictGetVal(de);

                    
                     * candidate for deletion **/
                    if (bestkey == NULL || thisval < bestval) {
                        bestkey = thiskey;
                        bestval = thisval;
                    }
                }
            }

            
            
            if (bestkey ) {
                ...
                delta = (long long) zmalloc_used_memory();
                dbDelete(db,keyobj);
                delta -= (long long) zmalloc_used_memory();
                mem_freed += delta;
                ...
            }
        }
        ...
    }
    ...
    return REDIS_OK;
}

```

总结归纳
----

*   不要放垃圾数据，及时清理无用数据：定期清理垃圾数据和下线的业务数据，避免占用不必要的内存空间和资源。
    
*   设置过期时间：对具有时效性的键设置合理的过期时间，利用Redis自身的过期键清理策略，降低过期键对内存的占用，并避免手动清理的麻烦。
    
*   控制单个键的大小：避免单个键过大，防止网络传输延迟和内存资源消耗过多。通过业务拆分、数据压缩等方式，控制键的大小。
    
*   使用不同逻辑数据库（db）分隔业务：在不同业务共享Redis时，最好使用不同的逻辑数据库进行分隔。这有助于过期键的及时清理，并方便问题排查和无用数据的下线。