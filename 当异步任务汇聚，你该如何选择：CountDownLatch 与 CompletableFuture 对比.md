# 当异步任务汇聚，你该如何选择：CountDownLatch 与 CompletableFuture 对比

当我们需要执行多个异步任务，并且需要等待它们全部完成才可以继续时，可以使用以下两种实现方案：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/761aefc46d31429da9a646da48cfa069~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1030&h=362&s=30830&e=png&b=fffefe)

方案一：CountDownLatch
------------------

CountDownLatch是一个同步工具类，可以用来实现多个线程之间的同步。它可以让一个线程等待其他线程完成某些工作后再继续执行。

实现方案如下：

```arduino
import java.util.concurrent.CountDownLatch;

public class CountDownLatchDemo {

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(3);

        
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                
                System.out.println(Thread.currentThread().getName() + " 执行任务");
                latch.countDown();
            }).start();
        }

        
        latch.await();

        
        System.out.println("所有任务完成");
    }
}

```

输出结果：

```null
Thread-1 执行任务
Thread-0 执行任务
Thread-2 执行任务
所有任务完成

```

方案二：CompletableFuture
---------------------

CompletableFuture是一个异步编程的工具类，可以用来执行异步任务并获取任务执行结果。

实现方案如下：

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CompletableFutureDemo {

    public static void main(String[] args) throws InterruptedException, 
    ExecutionException {
        
        ExecutorService executorService = Executors.newFixedThreadPool(3);

        
        CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> {
            
            System.out.println("任务1 执行");
        }, executorService);
        CompletableFuture<Void> future2 = CompletableFuture.runAsync(() -> {
            
            System.out.println("任务2 执行");
        }, executorService);
        CompletableFuture<Void> future3 = CompletableFuture.runAsync(() -> {
            
            System.out.println("任务3 执行");
        }, executorService);

        
        CompletableFuture.allOf(future1, future2, future3).get();

        
        System.out.println("所有任务完成");
    }
}

```

如果任务比较多的情况下就可以使用数组列表来存储创建的任务，CompletableFuture.allOf接收的是CompletableFuture<?>\[\]数组。

输出结果：

```null
任务1 执行
任务2 执行
任务3 执行
所有任务完成

```

*   CountDownLatch 方案中，我们使用 latch.await() 方法来等待所有任务完成。
*   CompletableFuture 方案中，我们使用 CompletableFuture.allOf() 方法来等待所有任务完成。

两种方案各有优缺点：

*   CountDownLatch 方案简单易用，但需要额外创建一个 CountDownLatch 对象。
*   CompletableFuture 方案可以获取任务的执行结果，但需要额外创建一个 CompletableFuture 对象。

在实际使用中，可以根据具体需求选择合适的方案。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c6e96a5b0324fa6a34becdb6ac7e5c2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1060&h=259&s=67451&e=png&b=fffefe)

CountDownLatch 和 CompletableFuture 都是用于在多线程环境中同步线程的工具。它们都具有各自的优缺点，适用于不同的场景。

| 特性 | CountDownLatch | CompletableFuture |
| --- | --- | --- |
| 功能 | 简单的同步 | 丰富的同步、异步、通信功能 |
| 使用难度 | 简单 | 复杂 |
| 适用场景 | 简单的同步场景 | 各种同步、异步场景 |

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/575c1db91ca84b809ad1690d2f711585~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1020&h=219&s=33026&e=png&b=dbe4f2)

CountDownLatch
--------------

CountDownLatch 是一个计数器，它可以让一个线程等待其他线程完成某些操作。CountDownLatch 的计数器初始值为某个整数，当计数器的值减为 0 时，所有等待的线程都会被唤醒。

CountDownLatch 的优点是简单易用，适用于一些简单的同步场景。例如，可以使用 CountDownLatch 来实现线程池的启动和关闭。

CountDownLatch 的缺点是它只能实现简单的同步场景。例如，如果需要在多个线程之间传递数据，CountDownLatch 就无法实现。

CompletableFuture
-----------------

CompletableFuture 是一个异步任务处理框架，它可以用于在多线程环境中异步执行任务、同步多个异步任务的结果、以及实现线程间通信等。

CompletableFuture 的优点是功能丰富，适用于各种同步场景。例如，可以使用 CompletableFuture 来实现线程池的异步执行、多个异步任务的结果同步、以及线程间数据传递等。

CompletableFuture 的缺点是使用起来比较复杂，需要一定的学习成本。

1.  CountDownLatch 的使用场景有哪些？

答案：CountDownLatch 可以用于以下场景：

*   等待其他线程完成任务后再执行某些操作。例如，可以使用 CountDownLatch 来实现线程池的启动和关闭。
*   让多个线程同步执行某些操作。例如，可以使用 CountDownLatch 来实现多个线程并发读取文件。

2.  CompletableFuture 是什么？

答案：CompletableFuture 是一个异步任务处理框架，它可以用于在多线程环境中异步执行任务、同步多个异步任务的结果、以及实现线程间通信等。

3.  CompletableFuture 的使用场景有哪些？

答案：CompletableFuture 可以用于以下场景：

*   异步执行任务。例如，可以使用 CompletableFuture 来实现异步 HTTP 请求。
*   同步多个异步任务的结果。例如，可以使用 CompletableFuture 来实现多个线程并发计算的结果求和。
*   实现线程间通信。例如，可以使用 CompletableFuture 来实现线程间的数据传递。

4.  CompletableFuture 的常用方法有哪些？

答案：CompletableFuture 提供了丰富的 API，常用的方法包括：

*   thenApply()：将当前 CompletableFuture 的结果作为参数传递给一个函数，并将函数的结果作为新的 CompletableFuture。
*   thenAccept()：将当前 CompletableFuture 的结果作为参数传递给一个函数，但不返回任何结果。
*   thenRun()：不依赖当前 CompletableFuture 的结果，执行一个函数。
*   allOf()：将多个 CompletableFuture 组合成一个新的 CompletableFuture，只有当所有 CompletableFuture 都完成时，新的 CompletableFuture 才会完成。
*   anyOf()：将多个 CompletableFuture 组合成一个新的 CompletableFuture，当任意一个 CompletableFuture 完成时，新的 CompletableFuture 就会完成。

5.  CompletableFuture 的异常处理机制是怎样的？

答案：CompletableFuture 的异常处理机制如下：

*   如果 CompletableFuture 的执行过程中抛出异常，则异常会被封装在一个 CompletionException 中，并传递给 thenApply()、thenAccept() 等方法。
*   如果 CompletableFuture 的执行过程中没有抛出异常，则 thenApply()、thenAccept() 等方法不会抛出异常。

6.  CompletableFuture 和 CountDownLatch 的区别是什么？

答案：CompletableFuture 和 CountDownLatch 都是用于在多线程环境中同步线程的工具，但它们有以下区别：

*   功能：CompletableFuture 具有更丰富的功能，可以用于异步执行任务、同步多个异步任务的结果、以及实现线程间通信等，而 CountDownLatch 只能用于简单的同步场景。
*   使用难度：CompletableFuture 的使用难度比 CountDownLatch 高，需要一定的学习成本。
*   适用场景：CompletableFuture 适用于各种同步场景，而 CountDownLatch 适用于简单的同步场景。