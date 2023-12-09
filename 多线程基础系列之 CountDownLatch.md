# 多线程基础系列之 CountDownLatch
📒一、线程工具
--------

### 背景故事

> 🔊 很久以前面试大厂，被面试官问到有没有使用过 CountDownLatch，我说使用过；面试官追问，那你可以说一下具体使用场景吗，脑袋一懵，竟然答不上来。是的，我背的八股文已经忘了，之前我也确实没有使用过。

最近碰到有一个同事错误地使用了 CountdownLatch，不由的想谈一谈 CountdownLatch 的使用。

### 官方解释

**同步工具类，允许一个或多个线程一直等待，直到其他线程运行完成后再执行** _（A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.）_

| 单线程等待执行 | 多线程同步等待执行 |
| --- | --- |
| ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7998776f8f2a4e6aa9dc5ad56c474f6e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=551&h=553&s=28916&e=png&b=ffffff)
 | ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80527cc86ee349b49c9d31fcceb7d394~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=845&h=535&s=35274&e=png&b=ffffff)
 |

有了上面理解，接下来再看看应用场景

📚二、应用场景
--------

举几个实际应用用到的例子，不会使用 CountDownLatch 的请抄作业了。

### 单线程等

RPC 远程调用场景

1.  RPC 底层通信调用是一个异步行为。
2.  在调用之前设置 CountDownLatch
3.  异步调用结束后，回调 countDown 方法
4.  这个时候主方法继续向下执行

```Java

final CountDownLatch latch = new CountDownLatch(1);
final RpcResult result = new RpcResult();


remoteInvoke(getHost(requestLine), request, new CallBack(timeout) {
    @Override
    public void handled(Response response) throws Exception {
        try {
            Response.Status status = response.startLine();
            byte[] bytes = response.body().bytes();
            ......
            result.setStateCode(status.code());
        } catch (Exception e) {
            result.setCause(t);
        } finally {
            
            latch.countDown();
        }
    }
});

```

可以把 remoteInvoke 理解为线程操作。

很多框架的源码启动类，都会有异步加载过程并同步等待，就是下面这个模板代码，试着在你的工程中用一用。

```Java
  final CountDownLatch launchLatch = new CountDownLatch(1);
  Thread launcherThread = new Thread(() -> {
      try {
        .......
      } finally {
          launchLatch.countDown();
      }
  });
   .....
 latch.await(timeout, TimeUnit.MILLISECONDS);

```

### 多线程等

多线程等待的场景，比如并发下载，并发查询等等

```Java
final CountDownLatch latch = new CountDownLatch(nTasks);
for (int i = 0; i < nTasks; ++i) {
    executorService.submit(
            new Runnable() {
                @Override
                public void run() {
                    try {
                        
                    } finally {
                        latch.countDown();
                    } 
                }
            });
}
latch.await();

```

### 补充场景

*   启动类，异步加载资源
*   源码中的注释也有两个详细的例子

📑三、其他补充
--------

### 源码浅析

内部实现一个 AbstractQueuedSynchronizer（AQS 队列同步器），源码并不多，主要，通过改变 `private volatile int state` 值来实现线程之间的协助。 state 不为 0 继续等待，为 0 向下执行。调用 countDown 则 state 减一

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00885e22a5734a25a829911fbad35170~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1331&h=955&s=98484&e=png&b=2b2b2b)

_学会慢慢的把这些工具类应用到你的工程里面，别再做 CRUD boy 了。_

### 同类工具

1.  Thread#join
2.  Future#get
3.  CompletableFuture

 纸上得来终觉浅，绝知此事要躬行.