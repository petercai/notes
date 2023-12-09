# Java多线程在实际工作中遇到的坑
Java多线程大家在实际项目中应该都有用到过，今天这篇文章不讲那些烂大街的八股文及多线程基础知识，聊聊在实际应用过程中一些需要注意的点（踩过的坑）。 

1.线程池
-----

Java使用多线程一般就两种方式，一种是直接new Thread()，另一种就是使用线程池了，使用线程池的好处有**降低资源消耗、** **提高响应速度、** **提高线程的可管理性等。**  但在使用线程池时特别需要注意：**队列类型大小**、**拒绝策略的选择。** 

**下面详细说说这两点带来的致命问题**

### 1.1队列的选择及大小

我们知道，当提交一个任务给线程池时，核心线程已满的话会将任务放到阻塞队列中，阻塞队列jdk有以下几种实现：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f396c52bea974d4f968aff4bac741a47~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1522&h=606&s=186498&e=png&b=2e261a)

一般用的比较多的就两种，数组类型和链表类型的，如果队列数量设置的比较小的情况下可以使用ArrayBlockingQueue，因为ArrayBlockingQueue底层是数组，它的内存是连续的，可以很好的利用到cpu的缓存，但也正因为内存连续，如果队列数量比较大的，会产生大对象，从而造成频繁GC甚至OOM，所以这里一般都用的链表类型，控制好size。如果设置的很大或者不设置，那么最大线程数是用不到的，并且会造成OOM，基于这点，也是很多jdk自带的线程池我们不推荐（不能）使用的原因，很多都用的无界队列。

### 1.2线程池拒绝策略

线程池拒绝策略jdk自带有四种：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ea66871b1584d86aff3bba9fddc0e06~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1460&h=600&s=162187&e=png&b=202020)

这里简单解释一下这四种拒绝策略

*   **ThreadPoolExecutor.AbortPolicy**：抛出 RejectedExecutionException来拒绝新任务的处理。
*   **ThreadPoolExecutor.CallerRunsPolicy**：调用执行自己的线程运行任务，也就是直接在调用execute方法的线程中运行(run)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
*   **ThreadPoolExecutor.DiscardPolicy**：不处理新任务，直接丢弃掉。
*   **ThreadPoolExecutor.DiscardOldestPolicy**：此策略将丢弃最早的未处理的任务请求。

这里重点关注一下**AbortPolicy**，他也是ThreadPoolExecutor默认的拒绝策略

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1f4712f3a05420fb361216d77124436~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1006&h=216&s=17703&e=jpg&b=202020)

**AbortPolicy**拒绝策略会直接抛出异常，这种处理方式不仅会造成我们的任务“丢失”，而且当我们需要等待多个线程完毕，如下图这种代码时，主线程会进入假死状态，这种问题在生产环境中将会带来灾难性的后果。所以我们一般会使用 **CallerRunsPolicy**，调用执行自己的线程运行任务运行任务，或者实现RejectedExecutionHandler接口，自定义拒绝策略，加入监控告警等措施。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b412d327bb6420b9a232fc9a0d331ac~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1422&h=1816&s=289758&e=png&b=202020)

2.线程变量传递-ThreadLocal
--------------------

ThreadLocal相信大家都不陌生，它的底层实现原理及它存在的问题也是面试过程中问的比较多的（容易造成内存泄露，变量在子线程中获取不到），因为它的自身缺陷，一般我们项目中都会用阿里的[transmittable-thread-loca](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Falibaba%2Ftransmittable-thread-local "https://github.com/alibaba/transmittable-thread-local")。

使用方式也比较简单：

1.  导入maven包，将原有的ThreadLocal替换为TransmittableThreadLocal。
2.  线程池使用TtlExecutors.getTtlExecutorService()进行包装。

这里特别重要的是第二步，很多时候会被忽略导致数据错乱。

我们项目中的自定义线程池虽然都进行了包装，但是某处业务代码在进行异步操作的时候使用了第三包里的线程池，从而造成了数据在并发环境下的错乱。

[感兴趣的也可以看下我写的这篇TransmittableThreadLocal源码解析](https://juejin.cn/spost/7308620787329564723 "https://juejin.cn/spost/7308620787329564723")

当在Java中使用多线程时，尽管它有助于提升程序运行效率，但不恰当和不合理的使用可能带来灾难性的后果。因此，在实际项目中，强烈建议谨慎使用多线程。如果问题可以通过单线程解决，最好避免引入多线程，因为多线程的潜在问题确实不少且比较隐蔽。

多线程编程中常见的问题包括竞态条件、死锁、上下文切换开销、资源限制以及线程间通信和协调等。这些问题可能影响程序的正确性和性能。因此，在讨论区，大家可以分享自己在实际项目中遇到的多线程问题，相互交流经验，共同进步。谨慎使用多线程并了解其潜在风险是确保程序稳定性和性能的关键。感谢大家。