# Redis 入门 - 五大基础类型及其指令学习
_01_、字符串（String）
----------------

Redis中字符串类型是二进制安全的数据类型。可以把字符串理解成一个字符数组，这个数组里存放着很多特定编码的字符，因此这种设计，所有Redis中的字符串可以存储认识数据类型：整数、小数、字符串、图片、序列化对象、二进制数据等。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/271b86f2dae84300812aaf7f4babafb2~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=jeD0Bl6mS1AopdpYoIL%2FTmuOVb8%3D)

我们简单讲解几个最常见指令。

1.设置指定key的值，语法：**set key value**。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/ad58f9971d7c49daa7e6629b4c103399~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=usmcaIbmF20n%2B1PTarg3aq52iA8%3D)

2.获取指定key的值，语法：**get key**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/e85562f1af5741889de66b2f81ee4a84~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=zrnHPbQioZ4i65Px4a9p1%2FMI%2Bo4%3D)

3.删除指定key，语法：**del key**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/dd00f8823473486f88da734c5c2b4c8c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=HoRmt7H7zEI6rEkOn9Hn153fm9w%3D)

当然字符串还有很多其他指令，这里就不一一列举了，有兴趣的可以自己试试。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/393f24f130434f6da2c9dba4013eba08~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=fpx6uSUBPPpd66Qe5KvGUJz2%2F6o%3D)

_02_、集合（Set）
------------

Redis中的集合类型可以理解为存放着一组无序的、无重复的元素的合集。你可以对元素进行增删查，也可以进行差集、交集、并集运算。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/b29c5a95b8764904869c1b705b74a948~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=EZxBKW8xMaJyB4lMaiEn0ZtD8X0%3D)

我们简单讲解几个最常见指令。

1.向指定key集合添加一个或多个元素，语法：**sadd key value1 value2…**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/ef815e8b49034d22a26c7d9ca0a8b2ab~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=XxwL23rTattb1PeCgh3VIxcwf5w%3D)

2.获取指定key集合中所有元素，语法：**smembers key**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/3354aafca09d472eb80d528a553718a4~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=VZPvPzrf7yDwrC9tjIXyosYd4sg%3D)

3.删除指定key集合中的一个或多个元素，语法：**srem key value1 value2…**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/3208d4828656418e8fb1bf5fefc17590~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=BzKWhPj3IZdJ%2FU6tenyNRickhG8%3D)

当然集合还有很多其他指令，这里就不一一列举了，有兴趣的可以自己试试。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/4c000dfcb3b644d988848d938a91114c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=w2mFfBul0bHd35X1yU5%2FuDcBxW4%3D)

_03_、有序集合（Sorted Set）
---------------------

Redis中的有序集合类型可以理解为集合类型+有序，即每个元素都对应一个分值，因此集合类型有的功能，有序集合类型基本也都有，同时还多了对分值进行聚合、筛选、排序等功能。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/d5784251472c4323a63ddaab008b39aa~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=zLWwPnM%2B%2FocucJSzc1LcwGt2T2c%3D)

我们简单讲解几个最常见指令。

1.向指定key有序集合添加一对或多对元素及其分值，语法：**zadd key score1 value1 score2 value2…**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/ac2ec99b899b460d9883c15a5baa1f24~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=QbDbiLtV%2BxhhkDqSDDszzTKvukA%3D)

2.获取指定key有序集合中指定元素的分值，语法：

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/ce61212871dd4a62aa127ca61617723a~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=wrnqc1pV0h3VT3TpQa4oJk%2BLo5w%3D)

3.删除指定key有序集合中指定元素，语法：**zrem key value**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/7a7b021240554484923e16fb4eb6cb59~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=uLlvgAI6sQjevP6CBCJtRg3%2BXP4%3D)

当然有序集合还有很多其他指令，这里就不一一列举了，有兴趣的可以自己试试。

_04_、列表（List）
-------------

Redis中的列表类型是一个严格按照元素先后插入的顺序排列的字符串集合，并且可以通过在这个集合的两端进行插入和移除操作，还可以通过元素值或索引进行查找元素或移除元素。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/e1291e92e3d544efb677dfb288172c52~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=6%2B3944BMvW4IHKCIjUm5zpZkMkY%3D)

我们简单讲解几个最常见指令。

1.从左边向指定key列表插入一个或多个元素，语法：**lpush key value1 value2 value3**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/2f04fbc1596f4a67ae0ba5e4961e6a5b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=%2FT6E9qkQQ6HrNKROcDmoZXy9nxg%3D)

2.从右边移除并获取指定key列表的第一个元素，语法：**rpop key**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/3644a0d7782649f2b58bcc0eefe36117~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=GPO5l98JnMyOXLPVrqypblOS1Mk%3D)

当然列表还有很多其他指令，这里就不一一列举了，有兴趣的可以自己试试。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/bcdc3e7cab1547528c7c59fbddcadaae~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=zEMW9XNbZMlnvtNVWeFpSYSu0rM%3D)

_05_、哈希（Hash）
-------------

Redis中的哈希类型可以理解成是一组键值对集合，键表示一个字符串字段，值表示数据对象，并且支持添加、获取或删除单个项即键值对，也可以获取整个哈希集合等功能。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/a14f318acf0143b1aadaa9aa4cd54a06~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=h7KemwkBcBMwZcm9SBSqyAXpFmo%3D)

我们简单讲解几个最常见指令。

1.向指定key哈希中添加一对或多对键值对，语法：**hset key field1 value1 field2 value2**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/ef183d7c8a9f420a9914df2d8828ed41~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=OAVQkuHMQJ1pi17JcJBQ%2Bl6rKuQ%3D)

2.获取指定key哈希中指定键对应的值，语法：**hget key filed**

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/e00f1b653d594bd9a8806439ba8f9773~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=lG%2FThOHOFRQ%2FohbloOot%2B8pe6jk%3D)

当然哈希还有很多其他指令，这里就不一一列举了，有兴趣的可以自己试试。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/83fbe5efcc1f49229d060be0a3e4cd80~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5Yaz5paX5bCP6aW85bmy:q75.awebp?rk3s=f64ab15b&x-expires=1728465484&x-signature=cjr5GqUJfnEKRpkcL%2BGzaaVQjgA%3D)

当然Redis不止这五种数据类型，还有其他更高级的数据类型，我们作为入门级教程，还是先掌握好这五大基本类型。只有掌握好了这些基础知识，只能Redis有什么，能做什么，才好在项目上熟练使用Redis，才好用Redis来解决各种复杂问题。

万丈高楼平地起，打好基础最重要，因此文章中没有列举到的指令也需要大家自己多去试试，亲自感受一下，才能更好的理解、记住、掌握。

> **文章转载自：**  [IT规划师](https://link.juejin.cn/?target=https%3A%2F%2Fhome.cnblogs.com%2Fu%2Fhugogoos%2F "https://home.cnblogs.com/u/hugogoos/")
> 
> **原文链接：**  [www.cnblogs.com/hugogoos/p/…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fhugogoos%2Fp%2F18407559 "https://www.cnblogs.com/hugogoos/p/18407559")
> 
> **体验地址：**  [www.jnpfsoft.com/?from=xitu](https://link.juejin.cn/?target=http%3A%2F%2Fwww.jnpfsoft.com%2F%3Ffrom%3Dxitu "http://www.jnpfsoft.com/?from=xitu")