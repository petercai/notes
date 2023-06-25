# Java开发实践：合理使用线程池及线程变量

> 本文将从线程池和线程变量的原理和使用出发，结合实例给出最佳使用实践，帮助各开发人员构建出稳定、高效的 java 应用服务。

随着计算技术的不断发展，3 纳米制程芯片已进入试产阶段，摩尔定律在现有工艺下逐渐面临巨大的物理瓶颈，通过多核处理器技术来提升服务器的性能成为提升算力的主要方向。

在服务器领域，基于 java 构建的后端服务器占据着领先地位，因此，掌握 java 并发编程技术，充分利用 CPU 的并发处理能力是一个开发人员必修的基本功，本文结合线程池源码和实践，简要介绍了线程池和线程变量的使用。

2.1 什么是线程池
----------

线程池是一种“池化”的线程使用模式，通过创建一定数量的线程，让这些线程处于就绪状态来提高系统响应速度，在线程使用完成后归还到线程池来达到重复利用的目标，从而降低系统资源的消耗。

2.2 为什么要使用线程池
-------------

总体来说，线程池有如下的优势：

1）降低资源消耗：通过重复利用已创建的线程降低线程创建和销毁造成的消耗。

2）提高响应速度：当任务到达时，任务可以不需要等到线程创建就能立即执行。

3）提高线程的可管理性：线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

3.1 线程池创建 &核心参数设置
-----------------

在 java 中，线程池的实现类是 ThreadPoolExecutor，构造函数如下：

```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit timeUnit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

可以通过 new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory,handler)来创建一个线程池。

*   **corePoolSize 参数**
    

在构造函数中，corePoolSize 为线程池核心线程数。默认情况下，核心线程会一直存活，但是当将 allowCoreThreadTimeout 设置为 true 时，核心线程超时也会回收。

*   **maximumPoolSize 参数**
    

在构造函数中，maximumPoolSize 为线程池所能容纳的最大线程数。

*   **keepAliveTime 参数**
    

在构造函数中，keepAliveTime 表示线程闲置超时时长。如果线程闲置时间超过该时长，非核心线程就会被回收。如果将 allowCoreThreadTimeout 设置为 true 时，核心线程也会超时回收。

*   **timeUnit 参数**
    

在构造函数中，timeUnit 表示线程闲置超时时长的时间单位。常用的有：TimeUnit.MILLISECONDS（毫秒）、TimeUnit.SECONDS（秒）、TimeUnit.MINUTES（分）。

*   **blockingQueue 参数**
    

在构造函数中，blockingQueue 表示任务队列，线程池任务队列的常用实现类有：

*   ArrayBlockingQueue ：一个数组实现的有界阻塞队列，此队列按照 FIFO 的原则对元素进行排序，支持公平访问队列。
    
*   LinkedBlockingQueue ：一个由链表结构组成的可选有界阻塞队列，如果不指定大小，则使用 Integer.MAX_VALUE 作为队列大小，按照 FIFO 的原则对元素进行排序。
    
*   PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列，默认情况下采用自然顺序排列，也可以指定 Comparator。
    
*   DelayQueue：一个支持延时获取元素的无界阻塞队列，创建元素时可以指定多久以后才能从队列中获取当前元素，常用于缓存系统设计与定时任务调度等。
    
*   SynchronousQueue：一个不存储元素的阻塞队列。存入操作必须等待获取操作，反之亦然。
    
*   LinkedTransferQueue：一个由链表结构组成的无界阻塞队列，与 LinkedBlockingQueue 相比多了 transfer 和 tryTranfer 方法，该方法在有消费者等待接收元素时会立即将元素传递给消费者。
    
*   LinkedBlockingDeque：一个由链表结构组成的双端阻塞队列，可以从队列的两端插入和删除元素。
    

*   **threadFactory 参数**
    

在构造函数中，threadFactory 表示线程工厂。用于指定为线程池创建新线程的方式，threadFactory 可以设置线程名称、线程组、优先级等参数。如通过 Google 工具包可以设置线程池里的线程名：

```
new ThreadFactoryBuilder().setNameFormat("general-detail-batch-%d").build()
```

*   **RejectedExecutionHandler 参数**
    

在构造函数中，rejectedExecutionHandler 表示拒绝策略。当达到最大线程数且队列任务已满时需要执行的拒绝策略，常见的拒绝策略如下：

*   ThreadPoolExecutor.AbortPolicy：默认策略，当任务队列满时抛出 RejectedExecutionException 异常。
    
*   ThreadPoolExecutor.DiscardPolicy：丢弃掉不能执行的新任务，不抛任何异常。
    
*   ThreadPoolExecutor.CallerRunsPolicy：当任务队列满时使用调用者的线程直接执行该任务。
    
*   ThreadPoolExecutor.DiscardOldestPolicy：当任务队列满时丢弃阻塞队列头部的任务（即最老的任务），然后添加当前任务。
    

3.2 线程池状态转移图
------------

ThreadPoolExecutor 线程池有如下几种状态：

*   RUNNING：运行状态，接受新任务，持续处理任务队列里的任务；
    
*   SHUTDOWN：不再接受新任务，但要处理任务队列里的任务；
    
*   STOP：不再接受新任务，不再处理任务队列里的任务，中断正在进行中的任务；
    
*   TIDYING：表示线程池正在停止运作，中止所有任务，销毁所有工作线程，当线程池执行 terminated()方法时进入 TIDYING 状态；
    
*   TERMINATED：表示线程池已停止运作，所有工作线程已被销毁，所有任务已被清空或执行完毕，terminated()方法执行完成。
    

![](_assets/c7555cd2ee3842964a0e894ea88c326f.png)

3.3 线程池任务调度机制
-------------

![](_assets/547ee1a1f4eca5d1df052e19a32d490e.png)

线程池提交一个任务时任务调度的主要步骤如下：

1）当线程池里存活的核心线程数小于 corePoolSize 核心线程数参数的值时，线程池会创建一个核心线程去处理提交的任务；

2）如果线程池核心线程数已满，即线程数已经等于 corePoolSize，新提交的任务会被尝试放进任务队列 workQueue 中等待执行；

3）当线程池里面存活的线程数已经等于 corePoolSize 了，且任务队列 workQueue 已满，再判断当前线程数是否已达到 maximumPoolSize，即最大线程数是否已满，如果没到达，创建一个非核心线程执行提交的任务；

4）如果当前的线程数已达到了 maximumPoolSize，还有新的任务提交过来时，执行拒绝策略进行处理。

核心代码如下：

```
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```

3.4 Tomcat 线程池分析
----------------

### Tomcat 请求处理过程

![](_assets/a9abf7b780c82a903a3c00495d519a9e.png)

Tomcat 的整体架构包含连接器和容器两大部分，其中连接器负责与外部通信，容器负责内部逻辑处理。在连接器中：

1）使用 ProtocolHandler 接口来封装 I/O 模型和应用层协议的差异，其中 I/O 模型可以选择非阻塞 I/O、异步 I/O 或 APR，应用层协议可以选择 HTTP、HTTPS 或 AJP。ProtocolHandler 将 I/O 模型和应用层协议进行组合，让 EndPoint 只负责字节流的收发，Processor 负责将字节流解析为 Tomcat Request/Response 对象，实现功能模块的高内聚和低耦合，ProtocolHandler 接口继承关系如下图示。

2）通过适配器 Adapter 将 Tomcat Request 对象转换为标准的 ServletRequest 对象。

![](_assets/3c91c3a6f100eda349dadac98f4172e2.png)

Tomcat 为了实现请求的快速响应，使用线程池来提高请求的处理能力。下面我们以 HTTP 非阻塞 I/O 为例对 Tomcat 线程池进行简要的分析。

### Tomcat 线程池创建

![](_assets/4982cbc66b4f3580a75710c4e1603dbf.png)

在 Tomcat 中，通过 AbstractEndpoint 类提供底层的网络 I/O 的处理，若用户没有配置自定义公共线程池，则 AbstractEndpoint 通过 createExecutor 方法来创建 Tomcat 默认线程池。

核心部分代码如下：

```
public void createExecutor() {
        internalExecutor = true;
        TaskQueue taskqueue = new TaskQueue();
        TaskThreadFactory tf = new TaskThreadFactory(getName() + "-exec-", daemon, getThreadPriority());
        executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), 60, TimeUnit.SECONDS,taskqueue, tf);
        taskqueue.setParent( (ThreadPoolExecutor) executor);
    }
```

其中，TaskQueue、ThreadPoolExecutor 分别为 Tomcat 自定义任务队列、线程池实现。

### Tomcat 自定义 ThreadPoolExecutor

Tomcat 自定义线程池继承于 java.util.concurrent.ThreadPoolExecutor，并新增了一些成员变量来更高效地统计已经提交但尚未完成的任务数量（submittedCount），包括已经在队列中的任务和已经交给工作线程但还未开始执行的任务。

```
* Same as a java.util.concurrent.ThreadPoolExecutor but implements a much more efficient
 * {@link #getSubmittedCount()} method, to be used to properly handle the work queue.
 * If a RejectedExecutionHandler is not specified a default one will be configured
 * and that one will always throw a RejectedExecutionException
 *
 */
public class ThreadPoolExecutor extends java.util.concurrent.ThreadPoolExecutor {
​
    
     * The number of tasks submitted but not yet finished. This includes tasks
     * in the queue and tasks that have been handed to a worker thread but the
     * latter did not start executing the task yet.
     * This number is always greater or equal to {@link #getActiveCount()}.
     */
    
    private final AtomicInteger submittedCount = new AtomicInteger(0);
    private final AtomicLong lastContextStoppedTime = new AtomicLong(0L);
    
    public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory,
            RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
        
        prestartAllCoreThreads();
    }
​
}
```

Tomcat 在自定义线程池 ThreadPoolExecutor 中重写了 execute()方法，并实现对提交执行的任务进行 submittedCount 加一。Tomcat 在自定义 ThreadPoolExecutor 中，当线程池抛出 RejectedExecutionException 异常后，会调用 force()方法再次向 TaskQueue 中进行添加任务的尝试。如果添加失败，则 submittedCount 减一后，再抛出 RejectedExecutionException。

```
@Override
    public void execute(Runnable command) {
        execute(command,0,TimeUnit.MILLISECONDS);
    }
​
    public void execute(Runnable command, long timeout, TimeUnit unit) {
        submittedCount.incrementAndGet();
        try {
            super.execute(command);
        } catch (RejectedExecutionException rx) {
            if (super.getQueue() instanceof TaskQueue) {
                final TaskQueue queue = (TaskQueue)super.getQueue();
                try {
                    if (!queue.force(command, timeout, unit)) {
                        submittedCount.decrementAndGet();
                        throw new RejectedExecutionException("Queue capacity is full.");
                    }
                } catch (InterruptedException x) {
                    submittedCount.decrementAndGet();
                    throw new RejectedExecutionException(x);
                }
            } else {
                submittedCount.decrementAndGet();
                throw rx;
            }
​
        }
    }
```

### Tomcat 自定义任务队列

在 Tomcat 中重新定义了一个阻塞队列 TaskQueue，它继承于 LinkedBlockingQueue。在 Tomcat 中，核心线程数默认值为 10，最大线程数默认为 200，**为了避免线程到达核心线程数后后续任务放入队列等待，Tomcat 通过自定义任务队列 TaskQueue 重写 offer 方法实现了核心线程池数达到配置数后线程的创建。** 

具体地，从线程池任务调度机制实现可知，当 offer 方法返回 false 时，线程池将尝试创建新新线程，从而实现任务的快速响应。TaskQueue 核心实现代码如下：

```
* As task queue specifically designed to run with a thread pool executor. The
 * task queue is optimised to properly utilize threads within a thread pool
 * executor. If you use a normal queue, the executor will spawn threads when
 * there are idle threads and you wont be able to force items onto the queue
 * itself.
 */
public class TaskQueue extends LinkedBlockingQueue<Runnable> {
​
    public boolean force(Runnable o, long timeout, TimeUnit unit) throws InterruptedException {
        if ( parent==null || parent.isShutdown() ) throw new RejectedExecutionException("Executor not running, can't force a command into the queue");
        return super.offer(o,timeout,unit); 
    }
​
    @Override
    public boolean offer(Runnable o) {
        
      
        if (parent==null) return super.offer(o);
        
        
        if (parent.getPoolSize() == parent.getMaximumPoolSize()) return super.offer(o);
        
        
        if (parent.getSubmittedCount()<=(parent.getPoolSize())) return super.offer(o);
        
        
        if (parent.getPoolSize()<parent.getMaximumPoolSize()) return false;
        
        
        return super.offer(o);
    }   
}
```

### Tomcat 自定义任务线程

Tomcat 中通过自定义任务线程 TaskThread 实现对每个线程创建时间的记录；使用静态内部类 WrappingRunnable 对 Runnable 进行包装，用于对 StopPooledThreadException 异常类型的处理。

```
* A Thread implementation that records the time at which it was created.
 *
 */
public class TaskThread extends Thread {
​
    private final long creationTime;
​
    public TaskThread(ThreadGroup group, Runnable target, String name) {
        super(group, new WrappingRunnable(target), name);
        this.creationTime = System.currentTimeMillis();
    }
​
​
    
     * Wraps a {@link Runnable} to swallow any {@link StopPooledThreadException}
     * instead of letting it go and potentially trigger a break in a debugger.
     */
    private static class WrappingRunnable implements Runnable {
        private Runnable wrappedRunnable;
        WrappingRunnable(Runnable wrappedRunnable) {
            this.wrappedRunnable = wrappedRunnable;
        }
        @Override
        public void run() {
            try {
                wrappedRunnable.run();
            } catch(StopPooledThreadException exc) {
                
                
                log.debug("Thread exiting on purpose", exc);
            }
        }
​
    }
​
}
```

3.5 思考 &小结
----------

**1）Tomcat 为什么要自定义线程池和任务队列实现？**

JUC 原生线程池在提交任务时，当工作线程数达到核心线程数后，继续提交任务会尝试将任务放入阻塞队列中，只有当前运行线程数未达到最大设定值且在任务队列任务满后，才会继续创建新的工作线程来处理任务，因此 JUC 原生线程池无法满足 Tomcat 快速响应的诉求。

**2）Tomcat 为什么使用无界队列？**

Tomcat 在 EndPoint 中通过 acceptCount 和 maxConnections 两个参数来避免过多请求积压。其中 maxConnections 为 Tomcat 在任意时刻接收和处理的最大连接数，当 Tomcat 接收的连接数达到 maxConnections 时，Acceptor 不会读取 accept 队列中的连接；这时 accept 队列中的线程会一直阻塞着，直到 Tomcat 接收的连接数小于 maxConnections（maxConnections 默认为 10000，如果设置为-1，则连接数不受限制）。acceptCount 为 accept 队列的长度，当 accept 队列中连接的个数达到 acceptCount 时，即队列满，此时进来的请求一律被拒绝，默认值是 100（基于 Tomcat 8.5.43 版本）。因此，通过 acceptCount 和 maxConnections 两个参数作用后，Tomcat 默认的无界任务队列通常不会造成 OOM。

```
* Allows the server developer to specify the acceptCount (backlog) that
 * should be used for server sockets. By default, this value
 * is 100.
 */
private int acceptCount = 100;
​
private int maxConnections = 10000;
```

3.6 最佳实践
--------

### 避免用 Executors 的创建线程池

![](_assets/79af84f1b4c905d5ec046e0c07778c5b.png)

Executors 常用方法有以下几个：

1）newCachedThreadPool()：创建一个可缓存的线程池，调用 execute 将重用以前构造的线程（如果线程可用）。如果没有可用的线程，则创建一个新线程并添加到线程池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。CachedThreadPool 适用于并发执行大量短期耗时短的任务，或者负载较轻的服务器；

2）newFiexedThreadPool(int nThreads)：创建固定数目线程的线程池，线程数小于 nThreads 时，提交新的任务会创建新的线程，当线程数等于 nThreads 时，提交新的任务后任务会被加入到阻塞队列，正在执行的线程执行完毕后从队列中取任务执行，FiexedThreadPool 适用于负载略重但任务不是特别多的场景，为了合理利用资源，需要限制线程数量；

3）newSingleThreadExecutor() 创建一个单线程化的 Executor，SingleThreadExecutor 适用于串行执行任务的场景，每个任务按顺序执行，不需要并发执行；

4）newScheduledThreadPool(int corePoolSize) 创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代 Timer 类。ScheduledThreadPool 中，返回了一个 ScheduledThreadPoolExecutor 实例，而 ScheduledThreadPoolExecutor 实际上继承了 ThreadPoolExecutor。从代码中可以看出，ScheduledThreadPool 基于 ThreadPoolExecutor，corePoolSize 大小为传入的 corePoolSize，maximumPoolSize 大小为 Integer.MAX_VALUE，超时时间为 0，workQueue 为 DelayedWorkQueue。实际上 ScheduledThreadPool 是一个调度池，其实现了 schedule、scheduleAtFixedRate、scheduleWithFixedDelay 三个方法，可以实现延迟执行、周期执行等操作；

5）newSingleThreadScheduledExecutor() 创建一个 corePoolSize 为 1 的 ScheduledThreadPoolExecutor；

6）newWorkStealingPool(int parallelism)返回一个 ForkJoinPool 实例，ForkJoinPool 主要用于实现“分而治之”的算法，适合于计算密集型的任务。

**Executors 类看起来功能比较强大、用起来还比较方便，但存在如下弊端：** 

1）FiexedThreadPool 和 SingleThreadPool 任务队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM；

2）CachedThreadPool 和 ScheduledThreadPool 允许创建的线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM；

使用线程时，可以直接调用 ThreadPoolExecutor 的构造函数来创建线程池，并根据业务实际场景来设置 corePoolSize、blockingQueue、RejectedExecuteHandler 等参数。

### 避免使用局部线程池

使用局部线程池时，若任务执行完后没有执行 shutdown()方法或有其他不当引用，极易造成系统资源耗尽。

### 合理设置线程池参数

在工程实践中，通常使用下述公式来计算核心线程数：

**nThreads=(w+c)/c\*n\*u=(w/c+1)\*n\*u**

其中，w 为等待时间，c 为计算时间，n 为 CPU 核心数（通常可通过 Runtime.getRuntime().availableProcessors()方法获取），u 为 CPU 目标利用率（取值区间为\[0, 1\]）；在最大化 CPU 利用率的情况下，当处理的任务为计算密集型任务时，即等待时间 w 为 0，此时核心线程数等于 CPU 核心数。

上述计算公式是理想情况下的建议核心线程数，而不同系统/应用在运行不同的任务时可能会有一定的差异，因此最佳线程数参数还需要根据任务的实际运行情况和压测表现进行微调。

### 增加异常处理

为了更好地发现、分析和解决问题，建议在使用多线程时增加对异常的处理，异常处理通常有下述方案：

*   在任务代码处增加 try...catch 异常处理；
    
*   如果使用的 Future 方式，则可通过 Future 对象的 get 方法接收抛出的异常；
    
*   为工作线程设置 setUncaughtExceptionHandler，在 uncaughtException 方法中处理异常。
    

### 优雅关闭线程池

```
public void destroy() {
        try {
            poolExecutor.shutdown();
            if (!poolExecutor.awaitTermination(AWAIT_TIMEOUT, TimeUnit.SECONDS)) {
                poolExecutor.shutdownNow();
            }
        } catch (InterruptedException e) {
            
            pool.shutdownNow();
            
            Thread.currentThread().interrupt();
        }
    }
```

为了实现优雅停机的目标，我们应当先调用 shutdown 方法，调用这个方法也就意味着，这个线程池不会再接收任何新的任务，但是已经提交的任务还会继续执行。之后我们还应当调用 awaitTermination 方法，这个方法可以设定线程池在关闭之前的最大超时时间，如果在超时时间结束之前线程池能够正常关闭则会返回 true，否则，超时会返回 false。通常我们需要根据业务场景预估一个合理的超时时间，然后调用该方法。

如果 awaitTermination 方法返回 false，但又希望尽可能在线程池关闭之后再做其他资源回收工作，可以考虑再调用一下 shutdownNow 方法，此时队列中所有尚未被处理的任务都会被丢弃，同时会设置线程池中每个线程的中断标志位。shutdownNow 并不保证一定可以让正在运行的线程停止工作，除非提交给线程的任务能够正确响应中断。

### 鹰眼上下文参数传递

```
* 在主线程中，开启鹰眼异步模式，并将ctx传递给多线程任务
**/

RpcContext_inner ctx = EagleEye.getRpcContext();

ctx.setAsyncMode(true);
​
​

* 在线程池任务线程中，设置鹰眼rpc环境
**/
private void runTask() {
    try {
        EagleEye.setRpcContext(ctx);
        
​
    } catch (Exception e) {
        log.error("requestError, params: {}", this.params, e);
    } finally {
        
        if (mainThread != Thread.currentThread()) {
            EagleEye.clearRpcContext();
        }
    }
​
}
```

4.1 什么是 ThreadLocal
-------------------

ThreadLocal 类提供了线程本地变量（thread-local variables），这些变量不同于普通的变量，访问线程本地变量的每个线程（通过其 get 或 set 方法）都有其自己的独立初始化的变量副本，因此 ThreadLocal 没有多线程竞争的问题，不需要单独进行加锁。

4.2 ThreadLocal 使用场景
--------------------

*   每个线程都需要有属于自己的实例数据（线程隔离）；
    
*   框架跨层数据的传递；
    
*   需要参数全局传递的复杂调用链路的场景；
    
*   数据库连接的管理，在 AOP 的各种嵌套调用中保证事务的一致性。
    

对于 ThreadLocal 而言，常用的方法有 get/set/initialValue 三个方法。

众所周知，在 java 中 SimpleDateFormat 有线程安全问题，为了安全地使用 SimpleDateFormat，除了 1）创建 SimpleDateFormat 局部变量；和 2）加同步锁 两种方案外，我们还可以使用 3）ThreadLocal 的方案。

```
* 使用 ThreadLocal 定义一个全局的 SimpleDateFormat
*/
private static ThreadLocal<SimpleDateFormat> simpleDateFormatThreadLocal = new
ThreadLocal<SimpleDateFormat>() {
@Override
protected SimpleDateFormat initialValue() {
return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
}
};

String dateString = simpleDateFormatThreadLocal.get().format(calendar.getTime());
```

5.1 ThreadLocal 原理
------------------

Thread 内部维护了一个 ThreadLocal.ThreadLocalMap 实例(threadLocals)，ThreadLocal 的操作都是围绕着 threadLocals 来操作的。

### threadLocal.get()方法

```
* Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
public T get() {
    
    Thread t = Thread.currentThread();
    
    ThreadLocalMap map = getMap(t);
    
    if (map != null) {
        
        ThreadLocalMap.Entry e = map.getEntry(this);
        
        if (e != null) {
            
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    
    return setInitialValue();
}
​
 
     * Variant of set() to establish initialValue. Used instead
     * of set() in case user has overridden the set() method.
     *
     * @return the initial value
     */
private T setInitialValue() {
    
    T value = initialValue();
    
    Thread t = Thread.currentThread();
    
    ThreadLocalMap map = getMap(t);
    if (map != null)
        
        map.set(this, value);
    else
        
        createMap(t, value);
    return value;
}
​

 * Create the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param t the current thread
 * @param firstValue value for the initial entry of the map
 */
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
​

 * Construct a new map initially containing (firstKey, firstValue).
 * ThreadLocalMaps are constructed lazily, so we only create
 * one when we have at least one entry to put in it.
 */
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    
    table = new Entry[INITIAL_CAPACITY];
    
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    
    table[i] = new Entry(firstKey, firstValue);
    
    size = 1;
    
    setThreshold(INITIAL_CAPACITY);
}

 * Set the resize threshold to maintain at worst a 2/3 load factor.
 */
private void setThreshold(int len) {
    threshold = len * 2 / 3;
}
```

### threadLocal.set()方法

```
* Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        
        Thread t = Thread.currentThread();
        
        ThreadLocalMap map = getMap(t);
        if (map != null)
            
            map.set(this, value);
        else
            
            createMap(t, value);
    }
```

### ThreadLocalMap

从 JDK 源码可见，ThreadLocalMap 中的 Entry 是弱引用类型的，这就意味着如果这个 ThreadLocal 只被这个 Entry 引用，而没有被其他对象强引用时，就会在下一次 GC 的时候回收掉。

```
static class ThreadLocalMap {
​
        
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            
            Object value;
​
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
    
    
}
```

5.2 ThreadLocal 示例
------------------

### 鹰眼链路 ThreadLocal 的使用

EagleEye（鹰眼）作为全链路监控系统在集团内部被广泛使用，traceId、rpcId、压测标等信息存储在 EagleEye 的 ThreadLocal 变量中，并在 HSF/Dubbo 服务调用间进行传递。EagleEye 通过 Filter 将数据初始化到 ThreadLocal 中，部分相关代码如下：

```
EagleEyeHttpRequest eagleEyeHttpRequest = this.convertHttpRequest(httpRequest);

EagleEyeRequestTracer.startTrace(eagleEyeHttpRequest, false);
​
try {
    chain.doFilter(httpRequest, httpResponse);
} finally {
    
    EagleEyeRequestTracer.endTrace(this.convertHttpResponse(httpResponse));
}
```

在 EagleEyeFilter 中，通过 EagleEyeRequestTracer.startTrace 方法进行初始化，在前置入参转换后，通过 startTrace 重载方法将鹰眼上下文参数存入 ThreadLocal 中，相关代码如下：

![](_assets/5609c20e12081b0d526376cc734f3805.png)

![](_assets/ee91282cd4fe330204339938c8f284bd.png)

EagleEyeFilter 在 finally 代码块中，通过 EagleEyeRequestTracer.endTrace 方法结束调用链，通过 clear 方法将 ThreadLocal 中的数据进行清理，相关代码实现如下：

![](_assets/84290c781d229af4c31694f1387716ed.png)

### Bad case：XX 项目权益领取失败问题

在某权益领取原有链路中，通过 app 打开一级页面后才能发起权益领取请求，请求经过淘系无线网关(Mtop)后到达服务端，服务端通过 mtop sdk 获取当前会话信息。

在 XX 项目中，对权益领取链路进行了升级改造，在一级页面请求时，通过服务端同时发起权益领取请求。具体地，服务端在处理一级页面请求时，同时通过调用 hsf/dubbo 接口来进行权益领取，因此在发起 rpc 调用时需要携带用户当前会话信息，在服务提供端将会话信息进行提取并注入到 mtop 上下文，从而才能通过 mtop sdk 获取到会话 id 等信息。某开发同学在实现时，因 ThreadLocal 使用不当造成下述问题：

*   问题 1：因 ThreadLocal 初始化时机不当，造成获取不到会话信息，进而导致权益领取失败。
    
*   问题 2：请求完成时，因未清理 ThreadLocal 中的变量值，导致脏数据。
    

**问题 1：权益领取失败分析**

在权益领取服务中，该应用构建了一套高效和线程安全的依赖注入框架，基于该框架的业务逻辑模块通常抽象为 xxxModule 形式，Module 间为网状依赖关系，框架会按依赖关系自动调用 init 方法（其中，被依赖的 module 的 init 方法先执行）。

在应用中，权益领取接口的主入口为 CommonXXApplyModule 类，CommonXXApplyModule 依赖 XXSessionModule。当请求来临时，会按依赖关系依次调用 init 方法，因此 XXSessionModule 的 init 方法会优先执行；而开发同学在 CommonXXApplyModule 类中的 init 方法中通过调用 recoverMtopContext()方法来期望恢复 mtop 上下文，因 recoverMtopContext()方法的调用时机过晚，从而导致 XXSessionModule 模块获取不到正确的会话 id 等信息而导致权益领取失败。

![](_assets/d39ba42545577a0bc84b8d1cc874a759.png)

**问题 2：脏数据分析**

权益领取服务在处理请求时，若当前线程曾经处理过权益领取请求，因 ThreadLocal 变量值未被清理，此时 XXSessionModule 通过 mtop SDK 获取会话信息时得到的是前一次请求的会话信息，从而造成脏数据。

**解决方案**

在依赖注入框架入口处 AbstractGate#visit（或在 XXSessionModule 中）通过 recoverMtopContext 方法注入 mtop 上下文信息，并在入口方法的 finally 代码块清理当前请求的 threadlocal 变量值。

5.3 思考 &小结
----------

**1）ThreadLocalMap 中的 Entry 为什么要设计为弱引用类型？**

若使用强引用类型，则 threadlocal 的引用链为：Thread -> ThreadLocal.ThreadLocalMap -> Entry\[\] -> Entry -> key（threadLocal 对象）和 value；在这种场景下，只要这个线程还在运行（如线程池场景），若不调用 remove 方法，则该对象及关联的所有强引用对象都不会被垃圾回收器回收。

**2）使用 static 和不使用 static 修饰 threadlocal 变量有和区别？**

若使用 static 关键字进行修饰，则一个线程仅对应一个线程变量；否则，threadlocal 语义变为 perThread-perInstance，容易引发内存泄漏，如下述示例：

```
public class ThreadLocalTest {
    public static class ThreadLocalDemo {
        private ThreadLocal<String> threadLocalHolder = new ThreadLocal();
​
        public void setValue(String value) {
            threadLocalHolder.set(value);
        }
​
        public String getValue() {
            return threadLocalHolder.get();
        }
    }
​
    public static void main(String[] args) {
        int count = 3;
        List<ThreadLocalDemo> list = new LinkedList<>();
        for (int i = 0; i < count; i++) {
            ThreadLocalDemo demo = new ThreadLocalDemo();
            demo.setValue("demo-" + i);
            list.add(demo);
        }
        System.out.println();
    }
}
```

在上述 main 方法第 22 行 debug，可见线程的 threadLocals 变量中有 3 个 threadlocal 实例。在工程实践中，使用 threadlocal 时通常期望一个线程只有一个 threadlocal 实例，因此，若不使用 static 修饰，期望的语义发生了变化，同时易引起内存泄漏。

![](_assets/63c569c02cda052b0f3b5f0d64ae0bc9.png)

5.4 最佳实践
--------

### ThreadLocal 变量值初始化和清理建议成对出现

如果不执行清理操作，则可能会出现：

1）内存泄漏：由于 ThreadLocalMap 的中 key 是弱引用，而 Value 是强引用。这就导致了一个问题，ThreadLocal 在没有外部对象强引用时，发生 GC 时弱引用 Key 会被回收，而 Value 不会回收，从而 Entry 里面的元素出现<null,value>的情况。如果创建 ThreadLocal 的线程一直持续运行，那么这个 Entry 对象中的 value 就有可能一直得不到回收，这样可能会导致内存泄露。

2）脏数据：由于线程复用，在用户 1 请求时，可能保存了业务数据在 ThreadLocal 中，若不清理，则用户 2 的请求进来时，可能会读到用户 1 的数据。

建议使用 try...finally 进行清理。

### ThreadLocal 变量建议使用 static 进行修饰

我们在使用 ThreadLocal 时，通常期望的语义是 perThread，若不使用 static 进行修饰，则语义变为 perThread-perInstance；在线程池场景下，若不用 static 进行修饰，创建的线程相关实例可能会达到 M * N 个（其中 M 为线程数，N 为对应类的实例数），易造成内存泄漏([https://errorprone.info/bugpattern/ThreadLocalUsage)](https://xie.infoq.cn/link?target=https%3A%2F%2Ferrorprone.info%2Fbugpattern%2FThreadLocalUsage))。

### 谨慎使用 ThreadLocal.withInitial

在应用中，谨慎使用 ThreadLocal.withInitial(Supplier<? extends S> supplier)这个工厂方法创建 ThreadLocal 对象，一旦不同线程的 ThreadLocal 使用了同一个 Supplier 对象，那么隔离也就无从谈起了，如：

```
private static ThreadLocal<Obj> threadLocal = ThreadLocal.withIntitial(() -> obj);
```

在 java 工程实践中，线程池和线程变量被广泛使用，因线程池和线程变量的不当使用经常造成安全生产事故，因此，正确使用线程池和线程变量是每一位开发人员必须修炼的基本功。本文从线程池和线程变量的使用出发，简要介绍了线程池和线程变量的原理和使用实践，各开发人员可结合最佳实践和实际应用场景，正确地使用线程和线程变量，构建出稳定、高效的 java 应用服务。