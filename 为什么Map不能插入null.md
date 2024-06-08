# 为什么Map不能插入null
在 Java 中，Map 是属于 java.util 包下的一个接口（interface），所以说“为什么 Map 不能插入 null？”这个问题本身问的不严谨。Map 部分类关系图如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dce819cb9eb84bf19289dc70bae1ea0a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=950&h=1031&s=157991&e=png&b=3d3223)
 所以，这里面试官其实想问的是：**为什么 ConcurrentHashMap 不能插入 null？**

1.HashMap和ConcurrentHashMap的区别
------------------------------

HashMap 和 ConcurrentHashMap 在对待 null 的态度上是不同的，在 Java 中，HashMap 是允许 key 和 value 值都为 null 的，如下代码所示：

```java
HashMap<String, Object> map = new HashMap();
map.put(null, null);
if (map.containsKey(null)) {
    System.out.println("存在 null");
} else {
    System.out.println("不存在 null");
}

```

以上程序的执行结果如下：

> 存在 null

从上述结果可以看出，HashMap 是允许 key 和 value 值都为 null 的。

但 ConcurrentHashMap 就不同了，它不但 key 不能为 null，而且 value 也不能为 null，如以下代码所示：

```java
ConcurrentHashMap<String, String> concurrentHashMap = new ConcurrentHashMap();
concurrentHashMap.put(null, "javacn.site");
System.out.println(concurrentHashMap.get(null));

```

在运行以上程序时就会报错，如下图所示： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92860f5d793242059cd69168d4c61055~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2129&h=560&s=105999&e=png&b=232427)
 当然，当你为 ConcurrentHashMap 的 value 值设置 null 时也会报错，如下代码所示：

```java
String key = "www.Javacn.site";
ConcurrentHashMap<String, String> concurrentHashMap = new ConcurrentHashMap();
concurrentHashMap.put(key, null);
System.out.println(concurrentHashMap.get(key));

```

在运行以上程序时就会报错，如下图所示： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/706b4854fc2a4c69bc558311e552cabc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2139&h=562&s=105757&e=png&b=232427)
 因此，我们可以得出结论：

1.  在 HashMap 中，key 和 value 值都可以为 null。
2.  在 ConcurrentHashMap 中，key 或者是 value 值都不能为 null。

2.为什么不能插入null？
--------------

如果我们查看 ConcurrentHashMap 的源码，就能发现为什么 ConcurrentHashMap 不能插入 null 了，以下是 ConcurrentHashMap 添加元素时的部分核心源码：

```java

public V put(K key, V value) {
    return putVal(key, value, false);
}
final V putVal(K key, V value, boolean onlyIfAbsent) {
    
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    
}

```

从上述 ConcurrentHashMap 添加元素的第一行源码就可以看出，当 key 或 value 为 null 时，会直接抛出空指针异常，这就是 ConcurrentHashMap 之所以不能插入 null 的根本原因了，因为源码就是这样设计的。

3.更深层次的原因
---------

那么问题来了，为什么 ConcurrentHashMap 的实现源码中，不允许为 key 或者是 value 设置 null 呢？

这就要从 ConcurrentHashMap 的使用场景说起了，在 Java 中，ConcurrentHashMap 是用于并发环境中执行的线程安全的容器，而 HashMap 是用于单线程环境下执行的非线程安全的容器，而并发环境下的运行更复杂，**如果我们允许 ConcurrentHashMap 的 key 或者是 value 为 null 的情况下，就会存在经典的“二义性问题”**。

### 3.1 什么是二义性问题？

所谓的二义性问题指的是代码或表达式存在多种理解或解释，导致程序的含义不明确或模糊。

以 ConcurrentHashMap 不允许为 null 的二义性问题来说，null 其实有以下两层含义：

1.  这个值本身设置的是 null，null 在这里表示的是一种具体的“null”值状态。
2.  null 还表示“没有”的意思，因为没有设置，所以啥也没有。

所以，如果 ConcurrentHashMap 允许插入 null 值，那么就会存在二义性问题。

那就有同学会问了，为什么 HashMap 允许插入 null，它就不怕有二义性问题吗？

### 3.1 可证伪的HashMap

HashMap 之所以不怕二义性问题的原因是，**HashMap 的设计是给单线程使用的，而单线程下的二义性问题是能被证明真伪的，所以也就不存在二义性问题了（能被证明的问题就不是二义性问题）**。

例如，当我们给 HashMap 的 key 设置为 null 时，我们可以通过 hashMap.containsKey(key) 的方法来区分这个 null 值到底是存入的 null？还是压根不存在的 null？这样二义性问题就得到了解决，所以 HashMap 的二义性问题可被证明真伪，所以就不怕二义性问题，因此也就可以给 key 或者 value 设置 null 了。

### 3.2 不可证伪的ConcurrentHashMap

而 ConcurrentHashMap 就不一样了，**因为 ConcurrentHashMap 是设计在多线程下使用的，而多线程下的二义性问题是不能被证明真伪的，所以二义性问题是真实存在的**。

因为在你在证明二义性问题的同时，可能会有另一个线程影响你的执行结果，所以它的二义性问题就一直存在。

例如，当 ConcurrentHashMap 未设置 key 为 null 时，会有这样一个场景，当一个线程 A 调用了 concurrentHashMap.containsKey(key)，我们期望返回的结果是 false，但在我们调用 concurrentHashMap.containsKey(key) 之后，未返回结果之前，线程 B 又调用了 concurrentHashMap.put(key,null) 存入了 null 值，那么线程 A 最终返回的结果就是 true 了，这个结果和我们之前预想的 false 完全不一样，这就是不能被证伪的二义性问题。

所以说，多线程的执行比较复杂，在多线程下 null 的二义性问题是不能被证明真伪的（因为在一个线程执行验证时，可能会有另一个线程改动结果，造成结果不准确），所以 ConcurrentHashMap 为了避免这个二义性问题，所以就在源码中禁用了 null 值作为 key 或 value。

课后思考
----

除了 ConcurrentHashMap 之后，还有哪些容器不允许使用 null 作为 key 或者 value 呢？
