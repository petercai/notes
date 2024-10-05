# Java 线程池 ThreadPoolExecutor 类源码介绍
@\[TOC\]

前言
--

### 线程池是什么

随着现在计算机的飞速发展，现在的计算机基本都是多核 CPU 的了。在目前的系统中创建和销毁线程都是十分昂贵的，为了不频繁的创建和销毁，以及能够更好的管理线程，就出现了池化的思想。

### 线程池解决了哪些问题

线程池的出现，解决了系统中频繁创建和销毁线程的问题。从系统的角度看还解决了一些问题： 1、降低了资源损耗：线程池中会有一些线程，利用这些线程达到重复使用线程执行任务的作用，避免了频繁的线程创建和销毁，这样就大大降低了系统的资源损耗； 2、提高了线程的管理手段：通过监控线程池中线程的数量和状态，来进行调优和控制； 3、提高的响应速度：当一个任务给到线程池后，一旦有空闲的线程会立即执行该任务；

### 本文主要讲述什么

本文主要讲述线程池主要实现类 ThreadPoolExecutor 类的源码，让大家能够更加清楚明白 ThreadPoolExecutor 线程池内部的核心设计与实现，以及内部机制有哪些。

### 感谢读者

读者同学们如果发现了哪些不对的地方请评论或私信我，我会及时更正！非常感谢！！！

线程池 UML 类图
----------

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/ec5f6e350faf449e8aa9e0bc52f7f128~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5p2c5bCP6Iif:q75.awebp?rk3s=f64ab15b&x-expires=1728466727&x-signature=lfT7xU8BjrxQxHFwwLKjNZr62g8%3D)

*   Executor 接口：这是线程池顶级的接口，定义了一个 execute(Runnable command) 方法，用于提交任务。
*   ExecutorService 接口：继承自 Executor 接口，提供了更多管理线程生命周期的方法，如 shutdown()、shutdownNow()、submit()、invokeAll() 等。主要的作用是扩充执行任务的能力和提供了管控线程池的方法。
*   AbstractExecutorService 抽象类：这是 ExecutorService 的一个抽象实现类，提供了一些 ExecutorService 接口的默认实现，减少了实现 ExecutorService 接口的类的工作量。让下层实现类只需要关注执行任务的方法即可。
*   ThreadPoolExecutor 类：ThreadPoolExecutor 是线程池实现类。继承 AbstractExecutorService，主要用于维护线程池的生命周期、管理线程和任务。它提供了多种构造方法，允许我们对线程池的行为进行详细配置，包括核心线程数、最大线程数、线程空闲时间、任务队列等。

ThreadPoolExecutor 内部设计
-----------------------

### 核心参数

| 参数名称 | 参数解释 |
| --- | --- |
| corePoolSize | （必填）核心线程数，即线程池一直存活的最线程数量。但是将allowCoreThreadTimeOut参数设置为true后，核心线程处于空闲一段时间以上，也会被回收。 |
| maximumPoolSize | （必填）线程池中最大线程数，当核心线程都忙起来并且任务队列满了以后，就开始创建新线程，知道创建的数量到达设置的 maximumPoolSize 数。 |
| keepAliveTime | （必填）线程空闲时间，即当线程数大于核心线程数时，非核心线程的空闲时间超过这个时间后，就会被回收。将allowCoreThreadTimeOut参数设置为true后，核心线程也会被回收。 |
| unit | （必填）keepAliveTime 的时间单位。有：TimeUnit.DAYS（天）、TimeUnit.HOURS（小时）、TimeUnit.MINUTES（分钟）、TimeUnit.SECONDS（秒）、TimeUnit.MILLISECONDS（毫秒）、TimeUnit.MICROSECONDS（微秒）、TimeUnit.NANOSECONDS（纳秒）。 |
| workQueue | （必填）任务队列，用于保存等待执行的任务。 |
| threadFactory | 线程工厂，用于指定创建新线程的方式。 |
| handler | 拒绝策略，当任务过多而无法处理时，采取的处理策略。 |

### 内部类

在 ThreadPoolExecutor 中有五个核心的内部类，分别是 AbortPolicy、CallerRunsPolicy、DiscardOldestPolicy、DiscardPolicy 和 Worker。总的来说其实就两类：拒绝策略（Policy）和工作线程类（Worker）。

拒绝策略内部类（Policy）是当工作队列中的任务已到达最大限制，并且线程池中的线程数量也达到最大限制，这时如果有新任务提交进来，该如何处理呢？这里的拒绝策略，就是解决这个问题的。拒绝策略表示当任务队列满了且线程数也达到最大了，这时候再新加任务，线程池已经无法承受了，这些新来的任务应该按什么逻辑来处理。

工作线程内部类（Worker）是每一个 Worker 类就会绑定一个线程（Worker 类有一个成员属性持有一个线程对象），可以将 Worker 理解为线程池中执行任务的线程。但是实际上它并不仅仅是一个线程，Worker 里面还会有一些统计信息，存储一些相关的数据。

### 任务队列

*   SynchronousQueue：SynchronousQueue 是一个没有容量的队列（不知道能不能称为一个队列）。它是 BlockingQueue 的一个实现类，与其他实现 BlockingQueue 的实现类对比，它没有容量，更轻量级；入队和出队要配对，也就是说当你入队一个元素后，必须先出队才能入队，否则插入的线程会一直等待。
*   LinkedBlockingQueue：LinkedBlockingQueue 是一个支持并发操作的有界阻塞队列，底层数据结构是由一个单链表维护。当使用它的时候，不设置长度限制，则默认 Integer.MAX\_VALUE 做为最大容量。
*   ArrayBlockingQueue：ArrayBlockingQueue 也是一个支持并发操作的有界阻塞队列，与 LinkedBlockingQueue 不同的是，ArrayBlockingQueue 底层数据结构是由一个环形数组数据结构维护的。同样是当使用它的时候，不设置长度限制，则默认 Integer.MAX\_VALUE 做为最大容量。

另外，还支持另外 4 种队列：

*   PriorityBlockingQueue：PriorityBlockingQueue 是一个支持优先级处理操作的队列，你可以按照某个规则自定义这个优先级，以保证优先级高的元素放在队列的前面，进行出队的时候能够先出优先级高的元素。
*   DelayQueue：DelayQueue 是一个延迟队列，队列中每一个元素都会有一个自己的过期时间，每当使用出队操作的时候，只有过期的元素才会被出队，没有过期的依然会留在队列中。
*   LinkedBlockingDeque：LinkedBlockingDeque 是一个由链表结构组成的双向阻塞队列，可以头部和尾部进行操作。
*   LinkedTransferQueue：LinkedTransferQueue 由链表结构组成的无界阻塞队列。它的设计比较特别，是一种预约模式的设计。当出队时如果队列中有数据则直接取走，没有的话会在队列中占一个位子，这个位子的元素为 null，当有其他线程入队时，则会通知出队的这个线程，告诉它你可以拿走这个元素了。

### 拒绝策略

当添加任务时，如果线程池中的容量满了以后，线程池会做哪些策略？下面是线程池的一些策略：

*   AbortPolicy（默认）：AbortPolicy 策略会将新的任务丢弃并抛出 RejectedExecutionException 异常。
*   CallerRunsPolicy：CallerRunsPolicy 策略是直接运行这个任务的run方法，但并非是由线程池的线程处理，而是交由任务的调用线程处理。
*   DiscardPolicy：DiscardPolicy 策略是直接丢弃任务，不抛出任何异常。
*   DiscardOldestPolicy：DiscardOldestPolicy 策略是将当前处于等待队列列头的等待任务强行取出，然后再试图将当前被拒绝的任务提交到线程池执行。

ThreadPoolExecutor 源码
---------------------

### 线程池生命周期

```java

private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

private static final int COUNT_BITS = Integer.SIZE - 3;

private static final int CAPACITY   = (1 << COUNT_BITS) - 1;



private static final int RUNNING    = -1 << COUNT_BITS;

private static final int SHUTDOWN   =  0 << COUNT_BITS;

private static final int STOP       =  1 << COUNT_BITS;

private static final int TIDYING    =  2 << COUNT_BITS;

private static final int TERMINATED =  3 << COUNT_BITS;



private static int runStateOf(int c)     { return c & ~CAPACITY; }

private static int workerCountOf(int c)  { return c & CAPACITY; }

private static int ctlOf(int rs, int wc) { return rs | wc; }


private static boolean runStateLessThan(int c, int s) {
    return c < s;
}


private static boolean runStateAtLeast(int c, int s) {
    return c >= s;
}


private static boolean isRunning(int c) {
    return c < SHUTDOWN;
}

```

上面这些源码主要是写了当前线程池的状态、线程池的五种状态值定义、计算线程池的状态和判断线程池的状态。 这里再补充一下线程池的五种状态：

*   RUNNING 状态：线程池创建后，初始化时会将线程池的状态设置为 RUNNING，表示可接受新任务，并且执行队列中的任务；
*   SHUTDOWN 状态：当调用了 shutdown() 方法，则线程池处于 SHUTDOWN 状态，表示不接受新任务，但会执行队列中的任务；
*   STOP 状态：当调用了 shutdownNow() 方法，则线程池处于 STOP 状态，表示不接受新任务，并且不再执行队列中的任务，并且中断正在执行的任务；
*   TIDYING 状态：所有任务已经中止，且工作线程数量为 0，最后变迁到这个状态的线程将要执行 terminated() 钩子方法，只会有一个线程执行这个方法；
*   TERMINATED 状态：中止状态，已经执行完 terminated() 钩子方法；

线程池状态执行流程： ![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/43f02343c87d42ff92d9e9a0ef22f2bc~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5p2c5bCP6Iif:q75.awebp?rk3s=f64ab15b&x-expires=1728466727&x-signature=zxDaNUx1GJ54kZpxh1Vxr3iRVYg%3D)

  

### 线程池构造函数

当我们创建线程池时，不管是通过 Executor 工具类还是手动创建线程池，最终都会调用的下面这个构造方法来实现的。

```java

 * 构造方法，创建线程池。
 * 
 * @param corePoolSize    线程池的核心线程数量
 * @param maximumPoolSize 线程池的最大线程数
 * @param keepAliveTime   当线程数大于核心线程数时，多余的空闲线程存活的最长时间
 * @param unit            存活时间的时间单位
 * @param workQueue       任务队列，用来储存等待执行任务的队列
 * @param threadFactory   线程工厂，用来创建线程，一般默认即可
 * @param handler         拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
 */
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
                          TimeUnit unit, BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory, RejectedExecutionHandler handler) {
    if (corePoolSize < 0 || maximumPoolSize <= 0 || maximumPoolSize < corePoolSize || keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ? null : AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}

```

这里简单介绍一下线程池的内部结构设计： ![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/04dc8445fde44bc8b075867ec001609b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5p2c5bCP6Iif:q75.awebp?rk3s=f64ab15b&x-expires=1728466727&x-signature=MGREc%2BmYG%2FtfFMszXzpTVhKCe7E%3D)

在线程池中，如果执行的线程数超过了核心线程数，那么这些多余线程的最长存活时间，不会超过 keepAliveTime 参数。

  

### execute() 【提交任务】

execute() 方法的主要目的就是将任务执行起来。

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    
    if (workerCountOf(c) < corePoolSize) {
        
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        
        if (!isRunning(recheck) && remove(command))
            reject(command);
        
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    
    else if (!addWorker(command, false))
        
        reject(command);
}

```

上面这些源码和注释，主要讲述 execute() 方法是怎么做的校验和做了哪些步骤。用流程图表达： ![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/e97f27fe87f54897915d93951d8d0375~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5p2c5bCP6Iif:q75.awebp?rk3s=f64ab15b&x-expires=1728466727&x-signature=IpKH%2Fp7odUtfZRfaEppBqd8Z7bo%3D)

  

### addWorker() 方法 【添加工作线程并启动】

#### 了解 retry:

在了解 addWorker() 方法之前，先了解一下 retry: 下面是一个示例

```java
public static void main(String[] args) {
    int i = 0;
    
    retry:
    for (;;) {
        i++;
        System.out.println("i=" + i);
        int j = 0;
        for (;;) {
            System.out.println("j=" + j);
            j++;
            if (j == 3) {
                break retry;
            }
        }
    }
}

```

在执行完下面代码后，通过控制台可以看到下面这个输出

```java
i=1
j=0
j=1
j=2

```

接下来把示例代码改一下

```java
public static void main(String[] args) {
    int i = 0;
    for (;;) {
        i++;
        System.out.println("i=" + i);
        int j = 0;
        
        retry:
        for (;;) {
            System.out.println("j=" + j);
            j++;
            if (j == 3) {
                break retry;
            }
        }
    }
}

```

输出的内容如下

```java
0
j=1
j=2
i=232781
j=0
j=1
j=2
i=232782
j=0
j=1
j=2
i=232783
...

```

通过两个示例，能够发现，retry 相当于一个标记，只用在循环里面，break 的时候会到 retry 标记处。如果 retry 没有在循环（for，while）里面，在执行到 retry 时，就会跳出整个循环。如果 retry 在循环里面，可以理解为跳到了关键字处执行，不管几层循环。continue理解也是一样。 **需要注意的是：**  retry: 需要放在for，whlie，do...while的前面声明，变量只跟在 break 和 continue 后面。

  

#### addWordker() 方法讲述

addWordker() 方法主要的目的是创建一个线程，然后启动执行，期间也会判断线程池状态和工作线程数量等各种检测。

```java

 * addWordker() 方法主要做的事情是创建一个线程，然后启动执行，期间也会判断线程池状态和工作线程数量等各种检测。
 *
 * @param firstTask 创建线程后执行的任务
 * @param core      该参数决定了线程池容量的约束条件，即当前线程数量以何值为极限值。参数为 true 则使用 corePollSize 作为约束值，否则使用 maximumPoolSize。
 * @return 是否创建线程并启动成功
 */
private boolean addWorker(Runnable firstTask, boolean core) {
    
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        
        
        
        
        if (rs >= SHUTDOWN && !(rs == SHUTDOWN && firstTask == null && !workQueue.isEmpty()))
            return false;

        
        for (;;) {
            
            int wc = workerCountOf(c);
            
            if (wc >= CAPACITY || wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            
            if (compareAndIncrementWorkerCount(c))
                break retry;
            
            c = ctl.get();
            
            if (runStateOf(c) != rs)
                continue retry;
        }
    }

    
    
    boolean workerStarted = false;
    
    boolean workerAdded = false;
    
    Worker w = null;
    try {
        
        
        w = new Worker(firstTask);
        
        final Thread t = w.thread;
        
        if (t != null) {
            
            final ReentrantLock mainLock = this.mainLock;
            
            mainLock.lock();
            try {
                
                
                int rs = runStateOf(ctl.get());
                
                
                
                
                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) 
                        throw new IllegalThreadStateException();
                    
                    
                    workers.add(w);
                    
                    int s = workers.size();
                    
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                
                
                t.start();
                
                workerStarted = true;
            }
        }
    } finally {
        
        if (! workerStarted)
            addWorkerFailed(w);
    }
    
    return workerStarted;
}

```

在 addWorker 方法中，如果添加 Worker 并且启动线程失败，则会做失败后的处理：

```java
private void addWorkerFailed(Worker w) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        
        if (w != null)
            workers.remove(w);
        
        decrementWorkerCount();
        
        tryTerminate();
    } finally {
        mainLock.unlock();
}

```

通过上面的源码，可以知道所有的线程都是放在 Worker 这个类中的，到目前为止，任务还并没有执行， 真正的执行是在 Worker 内部类中执行的，在下面会聊到 Worker 内部类中是怎么执行任务的，注意一下这里的 t.start() 这个语句，启动时会调用 Worker 类中的run方法，Worker 本身实现了 Runnable 接口，所以一个 Worker 类型的对象也是一个线程。现在先来总结创建一个线程执行工作任务 addWorker() 这个方法。

流程图表示： ![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/7ba7c75e266b4def9ba2031c4f90c7e5~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5p2c5bCP6Iif:q75.awebp?rk3s=f64ab15b&x-expires=1728466727&x-signature=G%2Bq0HPpHSqhFTnySYX1UJL1KStU%3D)

  

### Worker 类

通过上面的分析，就可以知道每一个任务都会放在 Worker 类中，由 Worker 类中的线程去最终执行任务。 下面就来看一下 Worker 类的源码：

```java
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable
{

    private static final long serialVersionUID = 6138294804551838833L;

    
    final Thread thread;
    
    Runnable firstTask;
    
    volatile long completedTasks;


    
    
    Worker(Runnable firstTask) {
        
        
        setState(-1); 
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);
    }

    
    public void run() {
        runWorker(this);
    }

    
    protected boolean isHeldExclusively() {
        return getState() != 0;
    }

    
    protected boolean tryAcquire(int unused) {
        
        if (compareAndSetState(0, 1)) {
            
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }

    
    protected boolean tryRelease(int unused) {
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }
    
    
    public void lock()        { acquire(1); }
    
    public boolean tryLock()  { return tryAcquire(1); }
    
    public void unlock()      { release(1); }
    
    public boolean isLocked() { return isHeldExclusively(); }

    
    void interruptIfStarted() {
        Thread t;
        
        if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
            try {
                
                t.interrupt();
            } catch (SecurityException ignore) {
            }
        }
    }
}

```

  

### runWorker 【执行任务】

通过 Worker 类中的源码可以得知，在类中最终是调用了 runWorker 方法，执行的任务，那么接下来就来看一下 runWorker 方法。

```java
final void runWorker(Worker w) {
    
    Thread wt = Thread.currentThread();
    
    Runnable task = w.firstTask;
    w.firstTask = null;
    
    w.unlock(); 
    
    boolean completedAbruptly = true;
    try {
        
        
        while (task != null || (task = getTask()) != null) {
            
            w.lock();
            
            
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                
                wt.interrupt();
            
            try {
                
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    
                    afterExecute(task, thrown);
                }
            } finally {
                
                task = null;
                
                w.completedTasks++;
                
                w.unlock();
            }
        }
        
        completedAbruptly = false;
    } finally {
        
        processWorkerExit(w, completedAbruptly);
    }
}

```

通过上面的源码可以得知在 runWorker 方法内部的主要流程主要如下：

1.  首先解锁 worker，允许线程中断；
2.  通过 while 循环判断当前任务是否为空，若为空则调用 getTask() 方法从阻塞队列中获取待处理的任务；
3.  获取到任务后工作线程直接加锁，同时根据线程池状态判断是否中断当前线程；
4.  执行任务，在任务执行前后执行自定义扩展方法；
5.  重复第 2 步的逻辑，直到阻塞队列中的任务全部执行完毕，最后调用 processWorkerExit() 方法处理 worker 线程退出、销毁空闲线程和控制线程池中线程数量的大小变化； 下面使用比较直观详细的流程图来表示出来：

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/ce7afc0d0fff48268eee23986f30ff7d~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5p2c5bCP6Iif:q75.awebp?rk3s=f64ab15b&x-expires=1728466727&x-signature=EVtzgMAvfP6S5gnFeefe0iKPBUg%3D)

  

### getTask 【获取任务】

getTask() 方法的主要作用是从任务队列中获取任务。

```java
private Runnable getTask() {
    
    boolean timedOut = false; 

    
    for (;;) {
        int c = ctl.get();
        
        int rs = runStateOf(c);

        
        
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            
            decrementWorkerCount();
            
            return null;
        }
        
        
        int wc = workerCountOf(c);

        
        
        
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        
         * 当前工作线程数 > maximumPoolSize
         * 或
         * timed && timedOut 为 true
         *
         * 并且
         *
         * 工作线程数大于1
         * 或
         * 阻塞队列为空
         */
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        
        try {
            
            
            
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            
            if (r != null)
                
                return r;
            
            timedOut = true;
        } catch (InterruptedException retry) {
            
            timedOut = false;
        }
    }
}

```

通过上面的 getTask() 方法原发分析，大致的主要的过程就是：

1.  校验线程池状态和工作线程数是否合规；
2.  根据条件选择从任务队列中超时等待获取，还是从任务队列中阻塞获取；

下面用一个直观的流程图来表示：

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/181be28d8da9453dab613806a6d2b5a9~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5p2c5bCP6Iif:q75.awebp?rk3s=f64ab15b&x-expires=1728466727&x-signature=RzQYR19IzaZK4jMGTlhY91WcSzU%3D)

  

### shutdown() 【停止所有线程】

shutdown() 方法会把状态修改成 SHUTDOWN。并且这里肯定会成功，因为修改时是通过 自旋 的方式，不成功不退出。

```java
public void shutdown() {
    
    final ReentrantLock mainLock = this.mainLock;
    
    mainLock.lock();
    try {
        
        checkShutdownAccess();
        
        advanceRunState(SHUTDOWN);
        
        interruptIdleWorkers();
        onShutdown();
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
}
    
private void advanceRunState(int targetState) {
    
    for (;;) {
        
        int c = ctl.get();
        
        
        if (runStateAtLeast(c, targetState) ||
                ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))
            break;
    }
}

```

  

### shutdownNow() 【停止所有任务，中断执行的任务】

shutdownNow() 方法会把线程池的状态改成 STOP，线程池不接受新任务，且不再执行队列中的任务，且中断正在执行的任务，同时标记所有线程为中断状态。

```java
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    
    final ReentrantLock mainLock = this.mainLock;
    
    mainLock.lock();
    try {
        
        checkShutdownAccess();
        
        advanceRunState(STOP);
        
        interruptWorkers();
        
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }
    
    tryTerminate();
    
    return tasks;
}

private List<Runnable> drainQueue() {
    BlockingQueue<Runnable> q = workQueue;
    ArrayList<Runnable> taskList = new ArrayList<Runnable>();
    
    q.drainTo(taskList);
    if (!q.isEmpty()) {
        for (Runnable r : q.toArray(new Runnable[0])) {
            if (q.remove(r))
                taskList.add(r);
        }
    }
    return taskList;
}

```

  

### tryTerminate() 【根据线程池状态进行判断是否结束线程池】

所有任务已经中止，且工作线程数量为0，就会进入这个状态。最后变迁到这个状态的线程将要执行 terminated() 钩子方法，只会有一个线程执行这个方法。

```java
final void tryTerminate() {
    
    for (;;) {
        int c = ctl.get();
        
        
        
        
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;
        
        
        if (workerCountOf(c) != 0) {
            
            interruptIdleWorkers(ONLY_ONE);
            return;
        }

        
        final ReentrantLock mainLock = this.mainLock;
        
        mainLock.lock();
        try {
            
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    
                    terminated();
                } finally {
                    
                    
                    ctl.set(ctlOf(TERMINATED, 0));
                    
                    termination.signalAll();
                }
                return;
            }
        } finally {
            
            mainLock.unlock();
        }
    }
}

```

更新成 TIDYING 或者 TERMINATED 状态的代码都在 tryTerminate() 方法中，而 tryTerminate() 方法在很多个地方被调用，比如 shutdown()、shutdownNow()、线程退出时，所以说几乎每个线程最后消亡的时候都会调用 tryTerminate() 方法，但最后只会有一个线程真正执行到修改状态为 TIDYING 的地方。 修改状态为 TIDYING 后执行 terminated() 方法，最后修改状态为 TERMINATED，标志着线程池真正消亡了。

总结
--

最后，使用流程图来呈现一下 ThreadPoolExecutor 运行机制的图： ![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/2123fcba66d946a1bf9d483c342fc6a2~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5p2c5bCP6Iif:q75.awebp?rk3s=f64ab15b&x-expires=1728466727&x-signature=wtM%2B9fywTqdnSLOR3xykzl97ou8%3D)

参考与感谢
-----

*   [美团线程池文章](https://juejin.cn/post/6844904113952522253?searchId=20240911155802C83E46D9F717398777C4#heading-14 "https://juejin.cn/post/6844904113952522253?searchId=20240911155802C83E46D9F717398777C4#heading-14")
*   [Agodival 博主的线程池文章](https://juejin.cn/post/6844903475197788168?searchId=2024091115522526026C16DF1430868C51#heading-0 "https://juejin.cn/post/6844903475197788168?searchId=2024091115522526026C16DF1430868C51#heading-0")
*   [云深i不知处 博主的线程池文章](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fmu_wind%2Farticle%2Fdetails%2F113806680 "https://blog.csdn.net/mu_wind/article/details/113806680")

非常感谢以上的博主！！！

End