# 分布式锁自己写，不用Redssion，结果翻车了
近日遇到了一个分布锁线上问题，导致用户获取锁一直失败，被阻拦提单近2H。 bug原因是Redis setnx获取锁超时，但实际写入成功，出现超时后并没有释放锁。由于锁维度是userId维度，导致用户再次提单一直无法获取锁，提单一直失败，客服同学说：用户情绪很激动，骂骂咧咧的。

为了避免锁超时导致并发问题，系统设置的超时时间过长——2小时。这导致用户2个小时都无法提单。由于系统日订单量在千万，即便超时比例非常低，每天也会有10单以上。用户找客服投诉的意愿十分强烈，并且态度很差。

在解决问题前，先回顾下我们分布式锁的技术实现方案

*   加锁命令： redis setnx 命令在key存在时返回false，key不存在返回成功。由于redis命令的原子性，可保证多个只有1个客户端可成功。
*   解锁命令：del key 删除key，就是释放锁

锁漏释放问题
------

加锁或解锁异常的情况下，可能出现漏被漏释放。 如果未设置超时时间，那么就会导致锁一直未释放，像我们的提单的场景，如果用户一直无法申请锁，一直提单，用户绝对会有打人的冲动。

超时时间设置长短由业务场景决定，业务耗时短，超时就设置短一些，如果持锁时间长，锁超时设置长一些。

设置超时时间的命令是 EXPIRE KEY timeout 。由于redis并未提供原子命令同时 setnx, expire 。所以要使用lua脚本 改进setnx 命令。（建议公司Redis Client统一提供这个原子命令）

```js
if (redis.call('setnx',KEYS[1],ARGV[1]) < 1) then 
    return 0; 
end; 
redis.call('EXPIRE',KEYS[1],tonumber(ARGV[2])); 
return 1;

```

有朋友反馈目前Redis Set命令支持同时设置超时时间，无需 再写LUA脚本。我亲自试验了一下，确实如此。 实验Redis版本要求在2.6.12以后。命令和实验截图如下

```arduino
set key1 value nx ex 10

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad48fde5ecfd4a0384bb17d81b133cce~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1440&h=206&s=121796&e=png&b=010101)

锁误释放问题
------

为什么会出现锁误释放呢？

假如客户端1持锁时间超过了锁超时时间，redis key被过期删除。然后客户端2申请锁成功。而客户端1 在执行完成后，释放锁。此时会导致 客户端2 申请的锁被客户端1 释放。 如图

Client1RedisClient2Client3Client 1误释放了Client申请的锁，导致Client3 可申请锁成功，未起到防并发的目的setnx 申请锁执行业务逻辑超时锁过期被删除setnx 申请锁成功释放锁。setnx 抢占锁成功。Client1RedisClient2Client3

Client 1误释放了Client2 申请的锁，导致Client3 可申请锁成功，未起到防并发的目的。由此可见，**删除锁的时候不能盲目删除。删除前应该保证是自己申请的锁，才能释放。** 

如何实现呢？首先setnx 需要保存value。 value中记录了客户端信息，可以使用毫秒级时间戳、UUID或者 机器ip+线程名称等。（毫秒级时间戳足够用了，一般业务场景分布式锁争抢没那么严重）。

删除操作需要实现 Compare AND DELETE （CAD）原子命令。 redis没有提供，需要使用LUA脚本实现

```js
if(redis.call('GET',KEYS[1]) ~= false ) then
        local v = redis.call('GET', KEYS[1]);
        if(v~= KEYS[2]) then
            return -1;
        end;
            local res = redis.call('DEL', KEYS[1]);
            if(res==1) then
                return 1;
           else
                return -2;
           end;

end;

return 0;

```

使用redis-cli加载lua脚本：

```js
redis-cli  script load "$(cat cad.lua)"

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/606c6c33554c4cbbbf889e2a256c96cb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1230&h=188&s=50225&e=png&b=010101)

执行lua脚本，指定参数参数，传入KEY、Value

EVALSHA 304be2df131dda36b189882adef40856da7b0259 2 KEY VALUE

返回值说明：

```yaml
 1:  值相等，可以删除，且删除成功
  0: 当 key1 不存在时
  -1： value1 和key1对应的value不相等
   -2： value1 和key1对应的value相等，但是DEL执行失败。因为redis lua原子化执行，之前已经判断key1存在，所以这种场景几乎不可能，但预留下

```

Value还有其他更重要的用途
---------------

分布式锁为什么会存Value，如果只看到持有锁超时场景，防止误删除是不够的。

回到一开始的问题，如果setnx阶段出现了超时，客户端是不清楚锁是否写入成功。需要重试一下，才能确认。（如果不重试就会出现锁漏释放，导致后续一直无法获取锁。）

超时后，可以get查询一下锁是否存在，如果存在就说明刚才加锁成功了吗？并不能，有可能是其他Client加的锁。此时就需要判断value是否一致，如果一致说明是自己获取成功的，否则说明是其他Client获取成功的。

Client1Redis防止超时，GET也要重试setnx key value1 获取锁获取锁超时GET KEYvalue一致则获取锁成功，否则获取锁失败。Client1Redis

总结
--

redis 加锁和释放锁都需要设置value，为了防止漏释放锁，更是为了保证加锁超时后，能查询确认上一次加锁是否成功。

到现在，我们线上的问题解决方案已经出来了，我们需要在加锁超时后，重新GET 确认是否加锁成功。