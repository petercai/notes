# Java并发编程之线程池十八问
Java并发编程之线程池十八问
---------------

   经过之前的学习，我们知道在Java中创建一个线程需要调用操作系统内置API，操作系统要为创建的线程分配一系列的资源，成本很高，如果在一个程序中，我们频繁的创建线程和销毁线程，资源占用巨大，性能很差，因此，便诞生了 **`池化思想`** ，将创建的线程放入一个池中管理，在Java中除了线程池，数据库连接、HTTP连接也用到了池化思想！

   我们基于此，整理了线程池相关的常用面试题集，合计十八道，通过这样的方式充分的了解和学习线程池。

### 第一问：什么是线程池？

   所谓 **`线程池`**，就是一个可以管理若干线程的容器，当有任务需要处理时，会提交到线程池的任务队列中，由线程池分配空闲的线程处理任务，处理完任务的线程不会被销毁，而是在线程池中等待下一个任务。

### 第二问：为什么要用线程池？

至于为什么要用线程池，可以从如下几点回答面试官：

*   **降低资源消耗：**  频繁的创建与销毁线程，占用大量资源，线程池的出现避免了这种情况，减少了资源的消耗；
*   **提高响应速度：**  因为线程池中的线程处于待命状态，有任务进来无需等待线程的创建就能立即执行（前提是有空闲线程，任务量巨大，还是需要排队的哈）；
*   **更好的管理线程：**  线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

### 第三问：如何创建一个线程池，为什么不推荐使用Executors？

在这里我们提供2种构造线程池的方法：

**方法一：**  通过ThreadPoolExecutor构造函数来创建（首选）    这是JDK中最核心的线程池工具类，在JDK1.8中，它提供了丰富的可设置的线程池构造参数，供我们设计不同的线程池，如下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6a141dcd7a842c0aa920564787be82a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=912&s=222404&e=png&b=2b2b2b)
 通过构造方法 ，可以给整个线程池设置大小、等待队列、非核心线程存活时间、创建线程的工厂类、拒绝策略等，具体参数描述可见 **`第六问`**，它们在线程池中所对应的关系，可见下图。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49e3027d0a254323a4f7ea678325f393~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1666&h=1042&s=356684&e=png&b=e4ecf9)

**方法二：**  通过 Executor 框架的工具类 Executors 来创建（不推荐）    Executors 是java并发工具包中的一个静态工厂类，在JDK1.5时被创造出来，提供了丰富的创造线程池的方法，通过它可以创建多种类型的线程池。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5744fb6ab9804a8b82306dbd5159f514~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1434&h=758&s=160142&e=png&b=21262c)

*   newFixedThreadPool：创建定长线程池，该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。当线程发生错误结束时，线程池会补充一个新的线程；
*   newCachedThreadPool：创建可缓存的线程池，如果线程池的容量超过了任务数，自动回收空闲线程，任务增加时可以自动添加新线程，所有线程在当前任务执行完毕后，将返回线程池进行复用，线程池的容量不限制；
*   newScheduledThreadPool：创建定长线程池，可执行周期性的任务；
*   newSingleThreadExecutor：创建单线程的线程池，只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务，线程异常结束，会创建一个新的线程，能确保任务按提交顺序执行；
*   newWorkStealingPool：任务可窃取线程池，不保证执行顺序，当有空闲线程时会从其他任务队列窃取任务执行，适合任务耗时差异较大的场景。

**`为何很多大厂都禁止使用Executors 创建线程池呢？`**

   如果大家跟入到Executors这些方法的底层实现中去看一眼的话，立马就知道原因了，像FixedThreadPool 和 SingleThreadExecutor这两个方法内使用的是无界的 LinkedBlockingQueue存储任务，任务队列最大长度为 Integer.MAX_VALUE,这样可能会堆积大量的请求，从而导致 OOM。

   而CachedThreadPool使用的是同步队列 SynchronousQueue, 允许创建的线程数量也为 Integer.MAX_VALUE ，如果任务数量过多且执行速度较慢，可能会创建大量的线程，从而导致 OOM，其他的方法所提供的均是这种`无界任务队列`，在高并发场景下导致OOM的风险很大，故大部分的公司已经不建议采用Executors提供的方法创建线程池了。

### 第四问：如何给线程池命名？

   如果我们的项目模块较多，在运行时调用了不同模块的线程池，为了在发生异常后快速定位问题，我们一般会在构建线程池时给它一个名字，这里我们提供几种线程池命名的方法。

**方法一：**  通过Spring 框架提供的CustomizableThreadFactory命名

```java
ThreadFactory springThreadFactory = new CustomizableThreadFactory("Spring线程池:");
ExecutorService exec = new ThreadPoolExecutor(1, 1,
         0L, TimeUnit.MILLISECONDS,
         new LinkedBlockingQueue<Runnable>(10),springThreadFactory);
 exec.submit(() -> {
     log.info(exec.toString());
 });

```

**方法二：**  通过Google guava工具类提供的ThreadFactoryBuilder命名

```java

ThreadFactory guavaThreadFactory = new ThreadFactoryBuilder().setNameFormat("guava线程池:").build();
ExecutorService exec = new ThreadPoolExecutor(1, 1,
          0L, TimeUnit.MILLISECONDS,
          new LinkedBlockingQueue<Runnable>(10),guavaThreadFactory );
  exec.submit(() -> {
      log.info(exec.toString());
  });

```

   其实还有一个是Apache commons-lang3 提供的 BasicThreadFactory工厂类，也可以给线程池命名，咱这里就不贴代码了，原因是他们的本质都是通过Thread 的setName()方法实现的！所以，我们其实自己也可以设计一个工厂类也实现线程池的命名操作！

**方法三：**  自定义工厂类实现线程池命名

先定义一个工厂类，通过实现ThreadFactory的newThread方法，完成命名。

```java
public class MyThreadFactory implements ThreadFactory {

    private final AtomicInteger threadNum = new AtomicInteger();
    private final String name;

    
     * 创建一个带名字的线程池生产工厂
     */
    public MyThreadFactory(String name) {
        this.name = name;
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setName(name + "-" + threadNum.incrementAndGet());
        return t;
    }
}

```

调用一下看看结果：

```java
@Slf4j
public class Test {
    public static void main(String[] args) {
        MyThreadFactory myThreadFactory = new MyThreadFactory("javaBuild-pool");
        ExecutorService exec = new ThreadPoolExecutor(1, 1,
                0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>(10),myThreadFactory);
        exec.submit(() -> {
            log.info(exec.toString());
        });
    }
}

```

**输出：** 

```java
17:46:37.387 [javaBuild-pool-1] INFO com.javabuild.server.pojo.Test - java.util.concurrent.ThreadPoolExecutor@1ee7d6d6[Running, pool size = 1, active threads = 1, queued tasks = 0, completed tasks = 0]

```

### 第五问：如何设置线程池的大小？

   我们在创建线程池的时候，线程池的大小是值得关注的点，线程池过小的话，在高并发场景下，同一时间有大量的任务请求处理，处理线程不够用，大量的任务堆积在任务队列中，CPU没有得到充分的使用，任务量过大时还可能带来OOM问题；线程池过大的话也会带来问题，大量的线程可能会在同一时间竞争CPU资源，带来频繁的上下文切换，导致相应时间过长，效率低下。

那么如何设置一个比较合适的线程池大小呢？ 我们在这里推荐一个公式:**`最佳线程数 = N（CPU 核心数）∗（1+WT（线程等待时间）/ST（线程计算时间）），其中 WT（线程等待时间）=线程运行总时间 - ST（线程计算时间）。`**

对于**CPU密集型任务**来说，WT/ST近似于0，故最佳线程数为N，不过一般情况下为了防止任务异常暂停导致CPU空闲，会多加一个线程，也就是N+1。

对于**IO密集型任务**来说，大部分时间都在做IO处理工作，线程几乎都在等待，这时WT/ST会很大，上述公式就会算出一个很大的线程数，但为了避免线程过多带来上下文切换问题，建议最佳线程数为2N。

### 第六问：线程池的常见参数有哪些？

在线程池中我们常见的参数如下：

*   corePoolSize：线程池中用来工作的核心线程数量，也可理解为线程池保有的最小线程数；
*   maximumPoolSize：最大线程数，线程池允许创建的最大线程数；
*   keepAliveTime：超出 corePoolSize 后创建的线程存活时间或者是所有线程最大存活时间，一个线程如果在一段时间内，都没有执行任务，说明很闲，keepAliveTime 和 unit 就是用来定义这个“一段时间”的参数。也就是说，如果一个线程空闲了keepAliveTime & unit这么久，而且线程池的线程数大于 corePoolSize ，那么这个空闲的线程就要被回收了；
*   unit：keepAliveTime 的时间单位；
*   workQueue：任务队列，是一个阻塞队列，当线程数达到核心线程数后，会将任务存储在阻塞队列中；
*   threadFactory：线程池内部创建线程所用的工厂，可以自定义如何创建线程，如给线程指定name。
*   handler：自定义任务的拒绝策略。线程池中所有线程都在忙碌，且任务队列已满，线程池就会拒绝接收再提交的任务（后面的问题中会详细讲）。

除了在构造线程池时进行参数的初始化配置，在ThreadPoolExecutor中还提供了参数动态配置的方法：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b29b062aef184d61b7188f68f795b775~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1256&h=266&s=50431&e=png&b=fffefe)

### 第七问：说一说线程池的5种状态？

在ThreadPoolExecutor的源码中定义了5个常量，用来标识线程池在整个任务处理周期中的状态。

```java
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

```

1.  **RUNNING：**  线程池一旦被创建，就处于 RUNNING 状态，任务数为 0，能够接收新任务，对已排队的任务进行处理。
    
2.  **SHUTDOWN：**  不接收新任务，但能处理已排队的任务。调用线程池的 shutdown() 方法，线程池由 RUNNING 转变为 SHUTDOWN 状态。
    
3.  **STOP：**  不接收新任务，不处理已排队的任务，并且会中断正在处理的任务。调用线程池的 shutdownNow() 方法，线程池由(RUNNING 或 SHUTDOWN ) 转变为 STOP 状态。
    
4.  **TIDYING：**  1）SHUTDOWN 状态下，任务数为 0， 其他所有任务已终止，线程池会变为 TIDYING 状态。
    
    2）线程池在 SHUTDOWN 状态，任务队列为空且执行中任务为空，线程池就会由 SHUTDOWN 转变为 TIDYING 状态。
    
    3）线程池在 STOP 状态，线程池中执行中任务为空时，就会由 STOP 转变为 TIDYING 状态。
    
5.  **TERMINATED：**  线程池彻底终止。线程池在 TIDYING 状态执行完 terminated() 方法就会由 TIDYING 转变为 TERMINATED 状态。
    

具体的状态转换可见下图：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2edf187a83a04127b2638e2ccf257259~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=294&s=68858&e=png&b=fcfcfc)

### 第八问：请你说一说线程池的运行原理（重要）？

上面聊了那么多，我们应该对于线程池的作用有了一个大致的了解，现在来看一下它整个生命周期内是如何运行的。（以ThreadPoolExceutor为例）

1、刚new出来的线程池里默认是没有线程的，只有一个传入的阻塞队列；

2、当我们执行execute提交一个方法后，会判断当前线程池中线程数是否小于核心线程数（corePoolSize），如果小于，那么就直接通过 ThreadFactory 创建一个线程来执行这个任务，当任务执行完之后，线程不会退出，而是会去阻塞队列中获取任务；

3、如果当前运行的线程数等于或大于核心线程数，但是小于最大线程数，那么就把该任务放入到任务队列里等待执行。

4、如果向任务队列投放任务失败（任务队列已经满了），但是当前运行的线程数是小于最大线程数的，就新建一个线程来执行任务。

5、如果当前运行的线程数已经等同于最大线程数了，新建线程将会使当前运行的线程超出最大线程数，那么当前任务会被拒绝，调用RejectedExecutionHandler.rejectedExecution()方法。

我们跟入到execute方法的源码中，去看看它是如何实现的。

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
        
        if (! isRunning(recheck) && remove(command))  
            reject(command);
        
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    
    else if (!addWorker(command, false))
        
        reject(command);
}

```

这段源码里通过workerCountOf()来计算当前工作线程数，通过addWorker()来添加线程执行任务，通过reject()来拒绝任务。

### 第九问：线程池的拒绝策略有哪些？

   上面的多个问题中都有提及线程池的拒绝策略，当线程池中所有线程都在忙碌，且任务队列已满，线程池就会拒绝接收再提交的任务，合理的配置拒绝策略对于一个线程池来说至关重要！

在JDK中提供了RejectedExecutionHandler接口的4种实现作为我们构建线程池传参使用：

1.  AbortPolicy：默认的拒绝策略，丢弃任务并抛出throws RejectedExecutionException；
2.  CallerRunsPolicy：由提交任务的线程自己去执行该任务；
3.  DiscardPolicy：直接丢弃任务，不抛出任何异常；
4.  DiscardOldestPolicy：从队列中剔除最先进入队列的任务，然后再次提交任务。

除了这4种拒绝策略之外，我们也可以自己实现RejectedExecutionHandler接口，设计自己需要的拒绝方式哈。

### 第十问：如果不允许丢弃任务，应该选什么拒绝策略？

根绝上一问中描述的几种拒绝策略的特点，在这里我们果断选择CallerRunsPolicy，直接在调用execute方法的线程中运行被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。

```java
public static class CallerRunsPolicy implements RejectedExecutionHandler {

        public CallerRunsPolicy() { }

        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                
                r.run();
            }
        }
    }

```

但这种策略会存在问题，如果我们抛给主线程的任务很耗时的话，会严重影响其他任务的提交速度，影响程序的整体性能，一般情况下不建议使用。

### 第十一问：线程池中的线程如何实现复用的？

我们知道线程池的核心功能就是实现线程的重复利用，那么线程池是如何实现线程的复用呢？ 我们在上面的第八问中知道了线程池通过addWorker()方法添加任务，而在这个方法的底层会将任务和线程一起封装到一个Worker对象中，Worker 继承了 AQS，也就是具有一定锁的特性。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/078aba321b86459493052fc5f8fa63d7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=781&h=124&s=6674&e=png&b=2d2d2d)
 然后Worker中有一个run方法，执行时会去调用runWorker()方法来执行任务，我们来看一下这个方法的源码：

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

在这段源码中我们看到了线程被复用的原因了，就是这个while的循环，当有任务需要执行或者能够从任务队列中获取到任务时，工作线程就会持续运行；如果从 getTask 获取不到方法的话，就会调用 finally 中的 processWorkerExit 方法，将线程退出。

### 第十二问：线程池中的线程是如何获取任务的？

线程获取任务的操作，在上一问中已经可以窥见了，就是这个getTask()方法

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

整个源码中的核心代码是workQueue的poll与take方法的选择问题，当timed为true时，则采用poll()方法获取队列中的头部任务，参数keepAliveTime也就是构造线程池时传入的空闲时间，这个方法的意思就是从队列中阻塞 keepAliveTime 时间来获取任务，获取不到就会返回 null，否则采用take(）无线期等待获取任务，知道获取到。

而这里的通过 `boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;`这句也给timed赋值，要么将allowCoreThreadTimeOut 设置为true，或者工作线程数大于核心线程数时，可以设置超时时间的去获取任务。

### 第十三问：线程池常用的阻塞队列有哪些？

在Executors中不同的创建线程池方法中采用了不同的阻塞队列，在此通过源码中的使用情况，汇总一下这些队列以及特点：

```java

public static ExecutorService newFixedThreadPool(int nThreads) {

    return new ThreadPoolExecutor(nThreads, nThreads,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());

}


public static ExecutorService newSingleThreadExecutor() {

    return new FinalizableDelegatedExecutorService (new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>()));

}


public static ExecutorService newCachedThreadPool() {

    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>());

}


public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}

```

### 第十四问：线程池中的线程异常后，是销毁还是复用呢？

对于这个问题我们要分两种情况去分析，第一种是通过 _**execute()**_ 提交任务时，在执行过程中抛出异常，且没有在任务内被捕获，当前线程会因此终止，异常信息会记录在日志或控制台中，并且线程池会移除异常线程，重新创建一个线程补上去。

我们在第十一问中讨论线程复用时，去分析过runWorker的源码，在源码的最后有个finally中调用了processWorkerExit方法，我们跟入进去后会发现，当任务执行抛出异常后，当前工作线程和任务均被移除，并创建新的线程。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df6f0c7e17c0421a911c040d7676d330~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=838&h=540&s=158464&e=png&b=fefdfd)
 第二种通过submit()提交任务时，如果在任务执行中发生异常，这个异常不会直接打印出来。相反，异常会被封装在由submit()返回的Future对象中。当调用Future.get()方法时，可以捕获到一个ExecutionException。在这种情况下，线程不会因为异常而终止，它会继续存在于线程池中，准备执行后续的任务。

我们通过submit()的底层源码发现，其实它的内部封装的是execute方法，只不过它的任务被放在了RunnableFuture对象里。

```java
  public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
     RunnableFuture<Void> ftask = newTaskFor(task, null);
     execute(ftask);
     return ftask;
  }

```

根据上一种方法的解释，我们知道execute方法会抛出异常终止线程的，为什么submit中不会呢，猫腻肯定在RunnableFuture里，经过一顿跟踪发现（图片来源：京东技术），这个Future中实现的run方法，对异常进行了捕获，所以并不会往上抛出，也就不会移除异常线程以及新建线程了。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/baf42afbbbd144f69ced9176df22f6e3~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=836&s=247318&e=png&b=fefdfd)

### 第十五问：如何对线程池进行监控？

为了更好的检测线程池的运行情况，以及出现问题时的快速定位，ThreadPoolExecutor中提供了一些方法来获取线程池的运行状态：

1.  getCompletedTaskCount：获取已经执行完成的任务数量；
2.  getLargestPoolSize：获取线程池里曾经创建过的最大的线程数量。这个主要是用来判断线程是否满过；
3.  getActiveCount：获取正在执行任务的线程数据；
4.  getPoolSize：获取当前线程池中线程数量的大小。

除此之外，还有不少的方法就不一一列举了，看图！

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07e2fafe3faa4a72afcecba7269bc788~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1034&h=460&s=51769&e=png&b=3c3f41)
 这里补充一点，其实细心的小伙伴应该也已经发现了，在第十一问，里runWorker源码中，在执行任务之前会回调 beforeExecute 方法，执行任务之后会回调 afterExecute 方法，而这些方法默认都是空实现，这为我们提供了重新实现的空间！

### 第十六问：如何合理的关闭一个线程池？

在JDK 1.8 中，线程池的停止一般使用 shutdown()、shutdownNow()这两种方法。

**方法一：**  shutdown()

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

```

在shutdown的源码中，会启动一次顺序关闭，在这次关闭中，执行器不再接受新任务，但会继续处理队列中的已存在任务，当所有任务都完成后，线程池中的线程会逐渐退出。

**方法二：**  shutdown()

```java

 * 尝试停止所有正在执行的任务，停止处理等待的任务，
 * 并返回等待处理的任务列表。
 *
 * @return 从未开始执行的任务列表
 */
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

```

与shutdown不同的是shutdownNow会尝试终止所有的正在执行的任务，清空队列，停止失败会抛出异常，并且返回未被执行的任务列表。

### 第十七问：说一说线程池的应用场景？

我们在真正的java开发过程中，会经常使用到多线程场景，特别是并发量较大的系统中，我们不可能通过频繁的创建与切换线程来处理大量任务，因此，线程池无疑是一个很好的选择，既可管理任务，又能高效利用线程，诸如多请求的WEB服务器、并行计算、异步处理等场景下，使用线程池可达到事半功倍的效果！

**我们这里以并行计算为例，写一个小demo感受一下哈**

```java
public class Test {
    public static void main(String[] args) {
        
        ExecutorService exec = buildThreadPoolExecutor();
        
        Callable<String> task = new Callable<String>() {
            @Override
            public String call() {
                
                Integer res = 2+2;
                return "[thread-name:" + Thread.currentThread().getName() + ",计算结果:" + res + "]";
            }
        };
        
        List<Future<String>> results = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            results.add(exec.submit(task));
        }
        
        for (Future<String> result : results) {
            try {
                System.out.println(result.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        }
        
        exec.shutdown();
    }

    
     * 构建线程池
     * @return
     */
    public static ExecutorService buildThreadPoolExecutor() {
        return new ThreadPoolExecutor(5, 10, 10, TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(100), new ThreadFactoryBuilder().setNameFormat("javabuild-%s").build()
                , new ThreadPoolExecutor.CallerRunsPolicy());
    }
}

```

**输出：** 

```java
[thread-name:javabuild-0,计算结果:4]
[thread-name:javabuild-1,计算结果:4]
[thread-name:javabuild-2,计算结果:4]
[thread-name:javabuild-3,计算结果:4]
[thread-name:javabuild-4,计算结果:4]
[thread-name:javabuild-0,计算结果:4]
[thread-name:javabuild-0,计算结果:4]
[thread-name:javabuild-0,计算结果:4]
[thread-name:javabuild-0,计算结果:4]
[thread-name:javabuild-0,计算结果:4]

```

通过输出我们看到，我们重命名了线程池，且不同的线程都执行完成了这个计算任务，并输出正确的结果。

### 第十八问：请你设计一个根据任务优先级执行的线程池？

为了考察面试者对于线程池的掌握程度，很多面试官可能会让你设计一个线程池，比如：`请你设计一个根据任务优先级执行的线程池`

拿到这样的问题后，我们结合自己的所学冷静分析，首先，我们的任务存储在哪里？在构造线程池会传入一个阻塞队列，作为我们任务存放的容器，在所有的队列中有一个优先级队列：**PriorityBlockingQueue** ，它的底层是通过小顶堆形式实现，即值最小的元素优先出队。

在选好任务队列后，我们要在队列中对任务进行排序，这个排序规则需要和面试官进一步沟通，但排序的实现可以使用2种方式。

1.  创建 PriorityBlockingQueue 时传入一个 Comparator 对象来指定任务之间的排序规则（比如众多异步运算任务中，按照乘法、除法、减法、加法的顺序优先执行任务）；
2.  或者对提交的任务实现Comparable 接口，并重写 compareTo 方法来指定任务之间的优先级比较规则。

