# JAVA并发编程ReentrantLock核心原理剖析
> JAVA并发编程系列以及陆续出了5篇，第六篇的主角ReentrantLock该出场了。之前[《一文看懂全部锁机制》](https://juejin.cn/post/7412487295729942528 "https://juejin.cn/post/7412487295729942528")谈到可重入锁、[《JAVA并发编程AQS原理剖析》](https://juejin.cn/post/7412606099722338356 "https://juejin.cn/post/7412606099722338356")谈到了JUC灵魂AQS，那么AQS的思想优秀实践者ReentrantLock是怎么实现AQS的呢？

#### 1、ReentrantLock是什么，有哪些优点

ReentrantLock英文翻译以及顾名思义：可重入锁。之前文章说过，还有synchronized也是可重入锁。竟然JDK最开始有了synchronized这个可重入锁，而且JDK1.6之后也对synchronized进行锁优化，性能堪比JUC里的其他Lock()。为什么还提供了ReentrantLock呢？在[《JAVA并发编程volatile核心原理》](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F2449423%3Ffrom_column%3D20421%26from%3D20421 "https://cloud.tencent.com/developer/article/2449423?from_column=20421&from=20421")文中开头我们就简单的列了synchronized的几个缺点，包括：阻塞时间过长，不可中断、是非公平锁。

所以，同样是可重入锁，ReentrantLock必须有一些亮点。它的优点就是较好的补足synchronized的缺点，提供了非常灵活多变的特性，满足系统项目研发的需求。

优点有：

1、支持公平锁+非公平锁。（synchronized仅支持非公平锁）

2、支持响应中断、超时。（synchronized不支持超时）

3、支持线程尝试获取锁。（synchronized不支持）

#### 2、具体说说ReentrantLock的原理，看过ReentrantLock源码吗

话说到这份上，好像不看源码不行了。上源码，不要慌，它的核心源码就这几行，先扫一眼：

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/aaf81754170941518356f813323c52ad~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5ouJ5LiB6Kej54mb6K-05oqA5pyv:q75.awebp?rk3s=f64ab15b&x-expires=1728552551&x-signature=4fXS9blShrhRJX9vBu3Pi7bhL7Q%3D)

刚面试官问：具体说说它的原理？

少废话，一句话总结它的原理：**ReentrantLock是AQS的具体实现，实现了公平锁和不公平锁。ReentrantLock源码里有三个内部类。** 

**一个是Sync抽象类；sync 继承了AQS 队列同步器。所以ReentrantLock是AQS的实现。** 

**一个是FailSync类，实现了Sync类，实现的是公平锁。** 

**一个是NonFairSync类，也是实现Sync类，实现的是非公平锁。** 

“好家伙，总结的还不错”面试官心嘀咕着，继续问：

##### 2.1 公平锁是怎么实现的？

**我们直接看源码，公平锁的lock加锁是acquire(1)方法。** 

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/bd9503e96108453596e23c66359711e7~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5ouJ5LiB6Kej54mb6K-05oqA5pyv:q75.awebp?rk3s=f64ab15b&x-expires=1728552551&x-signature=K9Pm1wDSI6UHx4VhX4VR3pK5LLg%3D)

接下来源码是，就看到这里就很清晰了。

```scss
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

```

首先，公平锁加锁可能有三部逻辑执行。

\*\*第一步：\*\*tryAcquire(1)，尝试去获取锁。如果成功了，就没有后面两个步骤的事。

\*\*第二步：\*\*如果第一步获取失败，把当前线程封装为独占锁类型NODE 进入AQS 的FIFO 队列，**等待被按顺序唤醒。** 

\*\*第三步：\*\*第二步执行后，也会调用selfInterrupt()，里面的源码就是线程自我中断，修改中断标识位为：true。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/d5f7b61f1d9d4f0f934c107d68d187f4~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5ouJ5LiB6Kej54mb6K-05oqA5pyv:q75.awebp?rk3s=f64ab15b&x-expires=1728552551&x-signature=jVV3l%2Brc35V3jKsDGFcsW0ZaKok%3D)

**那具体再说说第一步tryAcquire(1)，是如何获取线程锁的？（候选人内心有点破防：面试官问题咋那么多...）**

具体就是：

1、先判断当前AQS队列的state是否为0，如果是0，说明现在可以竞争锁。

2、接下来继续判断AQS的同步队列，如果队列是空、或者队列不为空且队列里的节点是不是自己，就去竞争锁。

3、如果竞争锁成功，通过CAS去设置AQS的state值为1，并帮AQS的当前占有锁的NODE 引用线程改完自己。

源码如下：

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/be4a02ccb16e4e4ca2df801ca0bd15b0~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5ouJ5LiB6Kej54mb6K-05oqA5pyv:q75.awebp?rk3s=f64ab15b&x-expires=1728552551&x-signature=NWffKp64%2BYN%2BFB%2BNUcQVuBEcw8E%3D)

##### 2.2 非公平锁又是怎么实现的？

new ReentrantLock()默认的就是非公平锁。

```csharp
    public ReentrantLock() {
        sync = new NonfairSync();
    }

```

非公平锁的竞争锁就比较暴力，上来就直接通过CAS去竞争，

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/2fc0ecb382ba4fdcbeb7bc8c703030bd~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5ouJ5LiB6Kej54mb6K-05oqA5pyv:q75.awebp?rk3s=f64ab15b&x-expires=1728552551&x-signature=9RPPEGQ5KVqP9NtLNE4E3iMQDjU%3D)

如果获取失败，进入acquire(1)，继续去尝试获取锁。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/08c2f56550114deeba7706d1fa96424b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5ouJ5LiB6Kej54mb6K-05oqA5pyv:q75.awebp?rk3s=f64ab15b&x-expires=1728552551&x-signature=b4ANPQKjNCa4JOjsOuyz1Zvu8Do%3D)

这里和公平锁也是不一样，它没有判断队列情况，继续用CAS方式去看能否修改state获取锁。

如果竞争失败，就会进入AQS队列，等待被唤醒....

今天暂时分享到这，其实可重入锁里面的源码还有很多内容，篇幅有限，等源码专区出来再全部分享。