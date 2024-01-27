# Java并发编程：AQS篇
AQS的全称是：`AbstractQueuedSynchronizer`。主要用来构造锁和同步器。源码如下：

```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable

```

`AQS`是`JUC`同步框架的基石。AQS通过一个**FIFO队列维护线程同步状态**，实现类只要继承该类，并重写指定方法，就可以实现一套线程同步机制。

AQS根据资源互斥级别提供了**独占和共享**两种资源访问模式；同时定义Condition结构，提供了wait/signal等待唤醒机制。在JUC中， `ReentrantLock`，`Semaphore`， `ReentrantReadWriteLock`，`SynchronousQueue`，`CountDownLatch`等等皆是基于 AQS 的。

原理概述
----

`AQS`中维护了一个`volatile int state`变量和一个`CLH双向队列`。队列中的结点持有线程引用，每个结点都可以通过`getState()`、`setState()`、`compareAndSetState()`对state进行修改和访问。

当线程获取锁时，也就是试图对`state`变量做修改，如果修改成功则获取到锁；如果修改失败，则将线程包装为结点挂在到队列中，等待持有锁的线程释放锁并唤醒队列中的结点。

源码
--

### AQS中的重要属性

```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {

    
     * Head of the wait queue, lazily initialized.  Except for
     * initialization, it is modified only via method setHead.  Note:
     * If head exists, its waitStatus is guaranteed not to be
     * CANCELLED.
     * 头结点，就是当前持有锁的线程
     */
    private transient volatile Node head;

    
     * Tail of the wait queue, lazily initialized.  Modified only via
     * method enq to add new wait node.
     * 阻塞的尾节点，每个新的等待线程进来，都插入到最后，也就形成了一个链表
     */
    private transient volatile Node tail;

    
     * The synchronization state.
     * 0代表没有被占用，大于0代表有线程持有当前锁。
     */
    private volatile int state;

    
    private transient Thread exclusiveOwnerThread;
 }

```

### AQS模板方法

AQS内部封装了队列维护逻辑，给实现类提供了以下模板方法：

```java
tryAcquire(int); 尝试获取独占锁，可获取返回true，否则false
tryRelease(int); 尝试释放独占锁，可释放返回true，否则false
tryAcqureShared(int); 尝试以共享方式获取锁，失败返回负数，只能获取一次返回0，否则返回个数
tryReleaseShared(int);尝试释放共享锁，可获取返回true，否则false
isHeldExclusively();判断线程是否独占资源

```

如果实现类只需实现独占锁/共享锁，可以只实现tryAcquire/tryRelease或tryAcqureShared/tryReleaseShared

### AQS 状态值

AQS 的核心是状态值 `state`，它代表了锁或同步器的状态。在 AQS 中，通常将 `state` 划分为两部分：

*   高 16 位：用于表示状态信息，如获取锁的次数或资源数量。
*   低 16 位：用于表示线程的等待状态，如是否已经获取锁或在等待队列中等待。

### FIFO队列

AQS使用一个FIFO队列来管理线程。这个队列由Node对象组成，每个Node包含了一个等待线程及一个等待状态。 AQS中的等待队列是一个双向链表，如下图所示，但是等待队列中不包含头节点，头节点是已经拿到了锁的节点。 ![](_assets/a97537d186954a82b56768750fcd4829~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

**Node的源码如下**

```java
static final class Node {
    
    
    static final Node SHARED = new Node();
    
    
    static final Node EXCLUSIVE = null;
    
    
    
    
    
    static final int CANCELLED =  1;
    
    
    static final int SIGNAL    = -1;
    
    
    static final int CONDITION = -2;
    
     * waitStatus value to indicate the next acquireShared should
     * unconditionally propagate
     */
    static final int PROPAGATE = -3;
    
    
    volatile int waitStatus;
    
    volatile Node prev;
   
    
    volatile Node next;
    
    
    volatile Thread thread;
    
    Node nextWaiter;

```

*   `SIGNAL`属性表示当前node的后继结点对应的线程需要被唤醒，也就是说：当前Node是需要被其前继节点来唤醒的。
*   `Node`的数据结构，目前只需要记住有：waitStatus+thread+pre+next四个属性

### acquire方法

```java
public final void acquire(int arg) {
    
    
    if (!tryAcquire(arg) &&
        
        
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

```

### addWaiter 方法

上述`acquire`方法中，如果获取锁失败，调用的`addWaiter`方法如下：

```java
private Node addWaiter(Node mode) {
    
    Node node = new Node(Thread.currentThread(), mode);
    
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    
    
    
    enq(node);
    return node;
}

```

ReentrantLock
-------------

### 使用ReentrantLock

```java
class LockReentrant implements Runnable{
    private final Lock lock = new ReentrantLock();

    public void method1() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "method1()");
            method2();
        } finally {
            lock.unlock();
        }
    }
}


```

上述的lock.lock调用的是ReentrantLock的lock方法

```java
public void lock() {
    sync.lock();
}

```

#### Sync类

sync.lock调用的方法如下

ReentrantLock使用Sync管理锁的加锁与释放，Sync继承AbstractQueuedSynchronizer，Sync有两个实现类，分别是非公平锁和公平锁。

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = -5179523762034025860L;

    
     * Performs {@link Lock#lock}. The main reason for subclassing
     * is to allow fast path for nonfair version.
     */
    abstract void lock();

    
     * Performs non-fair tryLock.  tryAcquire is implemented in
     * subclasses, but both need nonfair try for trylock method.
     */
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) 
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

```

#### 加锁（公平锁）

`static final class FairSync extends Sync`中的方法

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    
    
    final void lock() {
        
        acquire(1);
    }

    
     * Fair version of tryAcquire.  Don't grant access unless
     * recursive call or no waiters or is first.
     */
    
    protected final boolean tryAcquire(int acquires) {
        
        final Thread current = Thread.currentThread();
        
        int c = getState();
        
        if (c == 0) {
            
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        
        else if (current == getExclusiveOwnerThread()) {
            
            int nextc = c + acquires;
            
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            
            setState(nextc);
            return true;
        }
        
        return false;
    }
}


```

// 这个方法好像不是显示调用的，业务代码里都是直接调用lock方法，没见到 protected final boolean tryAcquire

#### 解锁

```java

public void unlock() {
    
    sync.release(1);
}

public final boolean release(int arg) {
    
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}


protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    
    boolean free = false;
    
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}



private void unparkSuccessor(Node node) {
    
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    
    
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        
        
        
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        
        LockSupport.unpark(s.thread);
}

```

在并发环境下，加锁和解锁需要三个部件的协调：

1.  锁状态，也就是锁state值，这个值就是表示是否有线程持有锁的标志位。加锁就是CAS操作这个state值加1，解锁就是CAS操作这个state值减1。
2.  线程的挂起和唤醒，AQS中使用了LockSupport的park()方法来挂起线程，unpark方法来唤醒线程。
3.  阻塞队列，争抢锁的线程可能有很多个所以会存在阻塞队列去管理这些没有抢到锁的线程，AQS使用的是一个FIFO的队列，双向链表的形式去管理。AQS采用了CLH锁的变体来实现。

Semaphore 信号量
-------------

synchronized和ReentrantLock都是一次只允许一个线程访问某个资源，而Semaphore可以让多个线程同时访问共享资源。通过acquire()获取一个许可，如果没有就等到。通过release()释放一个许可。比如：

```scss

final Smaphore semaphore = new Semaphore(5);
semaphore.acquire();
semephore.release()

```

### 构造方法

```java
public Semaphore(int permits) { 
    
    sync = new NonfairSync(permits); 
} 

public Semaphore(int permits, boolean fair) { 
    
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits); 
}

```

### 其他重要方法

```java
public void acquire() throws InterruptedException { } 
public void acquire(int permits) throws InterruptedException { } 
public void release() {} 
public void release(int permits) {} 

```

上述4个方法都会被阻塞，需要等待获取到结果之后才返回。如果想要立即得到执行结果，需要以下几个方法

```java
public boolean tryAcquire() { }; 
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { }; 
tryAcquire(int permits) { }; 
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; 

```

### 实战

```java
package multithread.aqs.semaphore;

import java.util.Random;
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

public class Worker implements Runnable {
    private static int count = 0;
    private final int id = count++;
    private int finished = 0;
    private Random random = new Random(47);
    private Semaphore semaphore;
    public Worker(Semaphore semaphore) {
        this.semaphore = semaphore;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                semaphore.acquire();
                System.out.println(this +"占用一个机器在生产...  ");
                TimeUnit.MILLISECONDS.sleep(random.nextInt(2000));
                synchronized (this) {
                    System.out.println("已经生产了" + (++finished) + "个产品，" + "释放出机器");
                }
                semaphore.release();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public String toString() {
        return getClass().getSimpleName() + "_" + id;
    }
}


public class WorkerAppMain {
    public static void main(String[] args) {
        
        int N = 8;
        
        Semaphore semaphore = new Semaphore(5);
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < N; i++) {
            exec.execute(new Worker(semaphore));
        }
        exec.shutdown();
    }
}


```

CountDownLatch 倒计时器
-------------------

可以实现类似计数器的功能。比如有一个任务A，要等到其他4个任务都执行完毕之后才能执行，可以利用`CountDownLatch`来实现。`CountDownLatch`是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对该值进行设置，当`CountDownLatch`使用完毕后，它不能再次被使用。

### 重要方法

```java
public void await() throws InterruptedException { }; 
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { }; 
public void countDown() { 
    sync.releaseShared(1)
}; 

```

### 原理

`CountDownLatch`是共享锁的一种实现，默认构造AQS的state值为count。

```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}

private static final class Sync extends AbstractQueuedSynchronizer {
    Sync(int count) {
        setState(count);
    }
  
}

```

当线程调用`countDown()`方法时，其实使用了`tryReleaseShare`方法以`CAS`的操作来减少`state`，直至`state`为0。当`state`为0时，表示所有的线程都调用了`countDown`方法，那么在`CountDownLatch`上等待的线程就会被唤醒并继续执行。

### 实战

两种典型用法

*   某一线程在开始运行前等待n个线程执行完毕。比如：启动一个服务时，主线程需要等待多个组件加载完成之后再继续执行
    *   将`CountDownLatch`的计数器初始化为1（`new CountDownLatch(n)`），每当一个任务线程执行完毕，就将计数器减1（`countdownlatch.countDown()`）。
    *   当计数器的值变为0时，在`CountDownLatch`上`await`的线程就会被唤醒。
*   实现多个线程开始执行任务的最大并行性。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开炮。
    *   初始化一个共享的CountDownLatch对象，将其计数器初始化为1，多个线程在开始执行任务前首先`countdownlatch.await()`。当主线程调用countDown()时，计数器变为0，多个线程同时被唤醒。

代码示例

```java
public class CountDownLatchExample {
  
  private static final int THREAD_COUNT = 550;

  public static void main(String[] args) throws InterruptedException {
    
    
    ExecutorService threadPool = Executors.newFixedThreadPool(300);
    final CountDownLatch countDownLatch = new CountDownLatch(THREAD_COUNT);
    for (int i = 0; i < THREAD_COUNT; i++) {
      final int threadNum = i;
      threadPool.execute(() -> {
        try {
          test(threadNum);
        } catch (InterruptedException e) {
          e.printStackTrace();
        } finally {
          
          countDownLatch.countDown();
        }

      });
    }
    countDownLatch.await();
    threadPool.shutdown();
    System.out.println("finish");
  }

  public static void test(int threadnum) throws InterruptedException {
    Thread.sleep(1000);
    System.out.println("threadNum:" + threadnum);
    Thread.sleep(1000);
  }
}


```

CyclicBarrier 循环栅栏
------------------

`CyclicBarrier` 和 `CountDownLatch` 非常类似，它也可以实现线程间的技术等待，但是它的功能比 `CountDownLatch` 更加复杂和强大。主要应用场景和 `CountDownLatch` 类似。

> `CountDownLatch` 的实现是基于 AQS 的，而 `CycliBarrier` 是基于 `ReentrantLock`(`ReentrantLock` 也属于 AQS 同步器)和 `Condition` 的。

`CyclicBarrier` 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是：让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。

**CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：** 

*   CountDownLatch 一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；
*   CyclicBarrier 一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；
*   CountDownLatch 是不能够重用的，而 CyclicBarrier 是可以重用的。

> 1.  [掘金：AQS源码分析——一步一步带你走进AQS的世界](https://juejin.cn/post/7225915335785644092 "https://juejin.cn/post/7225915335785644092")
> 2.  [掘金：【深入AQS原理】我画了35张图就是为了让你深入 AQS](https://juejin.cn/post/6844904146127044622 "https://juejin.cn/post/6844904146127044622")
> 3.  [知乎：AQS源码分析看这一篇就够了](https://link.juejin.cn/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F147850966 "https://zhuanlan.zhihu.com/p/147850966")
> 4.  [掘金：Java并发之AQS详解](https://juejin.cn/post/7006895386103119908 "https://juejin.cn/post/7006895386103119908")
> 5.  [腾讯云：CountDownLatch、CyclicBarrier、Semaphore的区别，你知道吗？](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1603768 "https://cloud.tencent.com/developer/article/1603768")
> 6.  [JavaGuide面试题](https://link.juejin.cn/?target=https%3A%2F%2Fjavaguide.cn%2Fjava%2Fconcurrent%2Faqs.html%23semaphore-%25E4%25BF%25A1%25E5%258F%25B7%25E9%2587%258F "https://javaguide.cn/java/concurrent/aqs.html#semaphore-%E4%BF%A1%E5%8F%B7%E9%87%8F")