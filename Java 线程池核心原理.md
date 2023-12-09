# Java 线程池核心原理
简介
--

### 1.1 什么是线程池

线程池是一种管理和复用线程的机制，它可以在需要执行任务时分配线程，执行完任务后将线程放回线程池中，以便重复利用。线程池可以有效地控制并发线程的数量，提高系统的性能和资源利用率。

### 1.2 线程池的作用和优势

线程池的作用是管理和调度线程，它可以避免频繁地创建和销毁线程，减少系统开销，提高系统的稳定性和响应速度。线程池还可以限制并发线程的数量，防止系统资源被耗尽，提高系统的并发能力。

### 1.3 线程池在Java中的应用

#### 1.3.1 异步处理

举个例子：比如发送邮件需要找smtp服务器，发送短信需要找四大运营商。将这种允许延迟看到效果，甚至即便失败也🆗的任务，是完全可以搞成异步的。

我们一般在项目中完成这种操作的时候，是不用自己写线程池的，直接使用SpringBoot自带的@Async注解就🆗了。

但是SpringBoot自带的@Async注解的本质还是将任务投递给线程池处理，可能我们平时没有关注处理这个任务的线程池。

其实也是可以给异步处理的任务指定一个合理的线程池，从而提升效率：

```java
@Component
public class TestAsync {

    static AtomicInteger atomicInteger = new AtomicInteger(1);

    @Bean
    public Executor taskPool(){
        ExecutorService pool = Executors.newFixedThreadPool(100, new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r);
                thread.setName("task-" + atomicInteger.getAndIncrement());
                return thread;
            }
        });
        return pool;
    }


    @Async(value = "taskPool")
    public void task() throws InterruptedException {
        System.out.println(Thread.currentThread().getName() + "，发送邮件");
    }
}

```

#### 1.3.2 **并行IO处理**

这是项目中非常常见的一个情况，比如一个业务处理，需要调取到三个服务的数据进行整合，我们可以合理的实现线程池做多个IO的并行处理，提升效率。

一般都是CountDownLatch + ThreadPoolExecutor的方式去实现：

```ini
static ThreadPoolExecutor executor = new ThreadPoolExecutor(
        10, 10, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<>(10)
)

public Object test1() {
    CountDownLatch countDownLatch = new CountDownLatch(3)
    // 时间 = MAX(商品，库存，策略)  = 0.8s
    // 调用商品服务 - 拿点数据     0.5s
    executor.execute(() -> {
        Object itemData = itemFeignClient.getData()
        countDownLatch.countDown()
    })

    // 调用库存服务 - 拿点数据     0.8s
    executor.execute(() -> {
        Object stockData = stockFeignClient.getData()
        countDownLatch.countDown()
    })
    // 调用策略服务 - 拿点数据     0.7s
    executor.execute(() -> {
        Object strategyData = strategyFeignClient.getData()
        countDownLatch.countDown()
    })

    try {
        countDownLatch.await(1500, TimeUnit.MILLISECONDS)
        // 业务线程在这汇总
        Object result = itemData + stockData + strategyData

        return result
    } catch (InterruptedException e) {
        e.printStackTrace()
        // 超时
        throw new RuntimeException("超时啦~")
    }
}

```

#### 1.3.3 其他框架底层的线程池

*   RabbitMQ：默认情况下，消费者是单线程消费。我们可以基于RabbitMQ的配置，prefetch以及maxconcurrent去指定每次拉取消息的数量以及最大并行消费的线程数。
*   OpenFeign：底层也需要你去指定线程池的信息，不然OpenFeign慢的要死。
*   Tomcat：默认200个核心线程。

**线程池核心属性&参数扫盲**
----------------

### 2.1 线程池状态

线程池有5种状态：

```java
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;

```

*   RUNNING：一切正常，干活！
*   *   当创建线程池后，初始时，线程池处于RUNNING状态。
*   SHUTDOWN：接的活都能干，不接新活！
*   *   如果调⽤了shutdown()⽅法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执⾏完毕。
*   STOP：什么活都不能干了！
*   *   如果调⽤了shutdownNow()⽅法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终⽌正在执⾏的任务。
*   TIDYING：线程池准备倒闭！
*   *   在TIDYING状态下，线程池会尝试将工作线程销毁，并清空任务队列。一旦所有工作线程都被销毁，并且任务队列也被清空，线程池就会进入TERMINATED状态。
*   TERMINATED：线程池倒闭了！
*   *   当线程池处于SHUTDOWN或STOP状态，并且所有⼯作线程已经销毁，任务缓存队列已经清空或执⾏结束后，线程池被设置为TERMINATED状态。

### 2.2 线程池核心参数

```arduino
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {...}

```

解释⼀下构造⽅法中涉及到的参数：

1.  corePoolSize：核⼼线程数。即池中⼀直保持存活的线程数，即使这些线程处于空闲。但是将allowCoreThreadTimeOut参数设置为true后，核⼼线程处于空闲⼀段时间以上，也会被回收。
2.  maximumPoolSize：池中允许的最⼤线程数。当核⼼线程全部繁忙且任务队列打满之后，线程池会临时追加线程，直到总线程数达到maximumPoolSize这个上限。
3.  keepAliveTime：线程空闲超时时间。当⾮核⼼线程处于空闲状态的时间超过这个时间后，该线程将被回收。将allowCoreThreadTimeOut参数设置为true后，核⼼线程也会被回收。
4.  unit：keepAliveTime参数的时间单位。有：TimeUnit.DAYS（天）、TimeUnit.HOURS（⼩时）、TimeUnit.MINUTES（分钟）、TimeUnit.SECONDS（秒）、TimeUnit.MILLISECONDS（毫秒）、TimeUnit.MICROSECONDS（微秒）、TimeUnit.NANOSECONDS（纳秒）
5.  workQueue：任务队列，采⽤阻塞队列实现。当核⼼线程全部繁忙时，后续由execute⽅法提交的Runnable将存放在任务队列中，等待被线程处理。
6.  threadFactory：线程⼯⼚。指定线程池创建线程的⽅式。
7.  handler：拒绝策略。当线程池中线程数达到maximumPoolSize且workQueue打满时，后续提交的任务将被拒绝，handler可以指定⽤什么⽅式拒绝任务。

#### 2.2.1 任务队列

在ThreadPoolExecutor线程池的API文档中，一共推荐了三种等待队列：

1.  SynchronousQueue：同步队列。这是⼀个内部没有任何容量的阻塞队列，任何⼀次插⼊操作的元素都要等待相对的删除/读取操作，否则进⾏插⼊操作的线程就要⼀直等待，反之亦然。
2.  LinkedBlockingQueue：⽆界队列（严格来说并⾮⽆界，上限是Integer.MAX_VALUE），基于链表结构。使⽤⽆界队列后，当核⼼线程都繁忙时，后续任务可以⽆限加⼊队列，因此线程池中线程数不会超过核⼼线程数。这种队列可以提⾼线程池吞吐量，但代价是牺牲内存空间，甚⾄会导致内存溢出。另外，使⽤它时可以指定容量，这样它也就是⼀种有界队列了。
3.  ArrayBlockingQueue：有界队列，基于数组实现。在线程池初始化时，指定队列的容量，后续⽆法再调整。这种有界队列有利于防⽌资源耗尽，但可能更难调整和控制。

另外，Java还提供了另外4种队列：

1.  PriorityBlockingQueue：⽀持优先级排序的⽆界阻塞队列。存放在PriorityBlockingQueue中的元素必须实现Comparable接⼝，这样才能通过实现compareTo()⽅法进⾏排序。优先级最⾼的元素将始终排在队列的头部；PriorityBlockingQueue不会保证优先级⼀样的元素的排序，也不保证当前队列中除了优先级最⾼的元素以外的元素，随时处于正确排序的位置。
2.  DelayQueue：延迟队列。基于⼆叉堆实现，同时具备：⽆界队列、阻塞队列、优先队列的特征。DelayQueue延迟队列中存放的对象，必须是实现Delayed接⼝的类对象。通过执⾏时延从队列中提取任务，时间没到任务取不出来。
3.  LinkedBlockingDeque：双端队列。基于链表实现，既可以从尾部插⼊/取出元素，还可以从头部插⼊元素/取出元素。
4.  LinkedTransferQueue：由链表结构组成的⽆界阻塞队列。这个队列⽐较特别的时，采⽤⼀种预占模式，意思就是消费者线程取元素时，如果队列不为空，则直接取⾛数据，若队列为空，那就⽣成⼀个节点（节点元素为null）⼊队，然后消费者线程被等待在这个节点上，后⾯⽣产者线程⼊队时发现有⼀个元素为null的节点，⽣产者线程就不⼊队了，直接就将元素填充到该节点，并唤醒该节点等待的线程，被唤醒的消费者线程取⾛元素。

#### 2.2.2 拒绝策略

线程池提供了4种策略:

| AbortPolicy（默认） | 丢弃任务并抛出RejectedExecutionException异常 |
| --- | --- |
| CallerRunsPolicy | 直接运⾏这个任务的run⽅法，但并⾮是由线程池的线程处理，⽽是交由任务的调⽤线程处理 |
| DiscardPolicy | 直接丢弃任务，不抛出任何异常 |
| DiscardOldestPolicy | 将当前处于等待队列列头的等待任务强⾏取出，然后再试图将当前被拒绝的任务提交到线程池执⾏ |

**线程池提交任务流程**
-------------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2714aabe79ec42e3ba432bb8d36144ab~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1562&h=1467&s=184964&e=png&b=ffffff)

### **3.1 构建好的线程池内部，有多少个工作线程？**

0个，工作线程是懒加载，随着任务的提交才会构建。

### **3.2** 任务提交到线程池的几种处理方式？

*   **工作线程数 < 核心线程数：**  直接构建核心线程来处理当前任务。只要满足前面的条件，就直接构建核心线程，不存在其他工作线程是否空闲。
*   **工作线程数 ≥ 核心线程数：**  将任务投递到阻塞队列，空闲的工作线程都会来当前的阻塞队列获取任务处理。
*   *   **当任务丢到阻塞队列之后，可能会出现没有工作线程的情况。**  此时会构建一个非核心线程去处理阻塞队列中的任务，避免任务长时间在阻塞队列中不被处理！
*   **阻塞队列已满：**  会创建非核心线程来处理刚刚投递过来的任务。等刚刚投递过来的任务处理完，才会去阻塞队列找之前排队的任务处理。
*   **工作线程数 ≥ 最大线程数 && 阻塞队列已满：**  执行拒绝策略。

### **3.3** 核心线程与非核心线程的区别 **？**

没有区别！线程池构建线程，也是基于Thread thread = new Thread()去构建，虽然在构建前，有一些判断上的不同，但是构建出来的都是线程，不分核心还是非核心！

线程池最终只确保数量满足参数要求即可。无论线程构建之初，被认定为核心还是非核心，到后面都是工作线程，而且线程池再多维护一个工作线程的属性，很麻烦，而且对效率有影响。

**线程池后续处理任务流程**
---------------

### 4.1 处理任务

线程池中的工作线程，在处理完手头的任务之后，都会去阻塞队列尝试拿新任务。在尝试的过程中处于阻塞转状态。

*   有的工作线程会死等，没任务，我就死等，等到有任务。（核心线程。 **工作线程没有超过核心线程数时，都是死等！** ） WAITING，take方法。
*   有的工作线程会等待最大空闲时间，这段时间没拿到任务，这个线程就销毁。（非核心线程。 **工作线程超过核心线程数之后，都是等一会！** ）TIMED_WAITING，poll（最大空闲时间）方法。

### **4.2 线程销毁**

无论是线程池的线程，还是自己new的Thread都是同一种线程，都是start起来的。

销毁的方式很简单，直接run方法结束。

线程池也是一样的操作，让run方法结束即可，而结束的方式，是让runWorker方法里的while循环结束。（工作线程可以一直干活的原因，就是在一个while循环中不停的去阻塞队列获取任务）

### **4.3线程池回收**

当咱们在局部位置，方法内部构件一个线程池去处理任务后，即便方法弹栈了，线程池引用没了，工作线程依然无法被正常回收，严重化会导致OOM问题。

在方法内部使用完毕线程池后，一定要基于执行shutdown方法，先让线程池将工作线程回收，否则存活的工作线程会让线程池对象无法被GC回收。

如何使用线程池
-------

### 5.1 ThreadPoolExecutor

```csharp
public class ThreadPoolExecutorTest {
    public static void main(String args[]) {
        
        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(3,5,5, TimeUnit.SECONDS,
            new ArrayBlockingQueue<Runnable>(5));
    
        
        for (int i = 0; i < threadPool.getCorePoolSize(); i++) {
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    for (int j = 0; j < 2; j++) {
                        System.out.println(Thread.currentThread().getName() + ":" + j);
                        try {
                            Thread.sleep(2000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            });
        }
        
        threadPool.shutdown(); 
        
    }
}








```

### **5.2 Executors封装线程池**

#### 5.2.1 FixedThreadPool

```arduino
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>());
}

```

固定容量线程池。其特点是最⼤线程数就是核⼼线程数，意味着线程池只能创建核⼼线程，keepAliveTime为0，即线程执⾏完任务⽴即回收。任务队列未指定容量，代表使⽤默认值Integer.MAX_VALUE。适⽤于需要控制并发线程的场景。

#### 5.2.2 SingleThreadExecutor

```csharp
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

```

单线程线程池。特点是线程池中只有⼀个线程（核⼼线程），线程执⾏完任务⽴即回收，使⽤有界阻塞队列（容量未指定，使⽤默认值Integer.MAX_VALUE）

#### 5.2.3 ScheduledThreadPool

```arduino
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

```

定时线程池。指定核⼼线程数量，普通线程数量⽆限，线程执⾏完任务⽴即回收，任务队列为延时阻塞队列。这是⼀个⽐较特别的线程池，适⽤于执⾏定时或周期性的任务。

#### 5.2.4 CachedThreadPool

```csharp
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
}

```

缓存线程池。没有核⼼线程，普通线程数量为Integer.MAX_VALUE（可以理解为⽆限），线程闲置60s后回收，任务队列使⽤SynchronousQueue这种⽆容量的同步队列。适⽤于任务量⼤但耗时低的场景。

线程池的最佳实践
--------

### 6.1 核心线程数

*   CPU密集：你的线程需要CPU尽可能的去调度他，核心线程数一般就要设置到CPU内核数+1。
*   I/O密集：很多书上有一些IO密集时，核心线程数设置的公式，但是实际的情况和公式还有出入的，IO密集的时间不确定的。这里一定要基于压测的形式，找到一个合适的中间值，在短时间内可以按照任务的RT时间以及任务数来决定核心线程数的大小。

### 6.2 阻塞队列长度

*   根据任务允许的延迟时间，基于每秒任务大致多少的数量，来预测队列最长时，任务会延迟多久。
*   JVM内存问题，如果大量的任务扔到阻塞队列中，占用的内存也不小，尽可能不要占用太多。

### 6.3 最大线程数

*   强烈推荐就跟核心线程数的大小设置为一样的。如果设置的超过核心线程数，反而会造成线程池处理任务的效率降低，因为CPU频繁的上下文切换，导致浪费CPU资源，这个是完全没有必要的。

### 6.4 拒绝策略

根据业务场景去使用：

*   例如记录日志，丢了就丢了，可以采用DiscardPolicy。
*   任务必须要求处理完的，就得设置成CallerRunsPolicy。
*   或者AbortPolicy抛出异常也可以，日志有记录的信息，会知道线程池处理的能力不足。
*   任务允许延迟，但是又不能影响当前效率，记录信息扔到数据库，后面定时处理。

### 6.5 避免线程泄漏

*   确保及时关闭线程池，避免长时间运行的任务导致线程池无法释放线程资源。
*   使用 try-with-resources 或者 finally 块来确保在任务执行完毕后关闭线程池。

### 6.6 处理异常

*   在任务执行过程中，及时捕获并处理异常，避免异常导致线程池中断或异常终止。
*   可以使用 Thread.UncaughtExceptionHandler 来处理未捕获的异常，确保线程池的稳定运行。

### 6.7 优雅地关闭线程池

*   调用线程池的 shutdown() 方法来关闭线程池，该方法会等待所有任务执行完毕后关闭线程池。
*   如果需要立即关闭线程池，可以使用 shutdownNow() 方法来中断所有任务并立即关闭线程池。