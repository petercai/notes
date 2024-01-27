# 深入理解Java中的锁机制

[![](_assets/cc27fc75a3774af98cadb0ec68660729~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)
](https://link.juejin.cn/?target=https%3A%2F%2Fimgse.com%2Fi%2FpiyCGcV "https://imgse.com/i/piyCGcV")

### 引言

大家好，我是小黑。今天咱们来聊聊Java中的锁机制，这可是并发编程的核心。你知道吗，在并发编程的世界里，正确地使用锁就像是掌握了一把神奇的钥匙，它能帮咱们在多线程的混战中保持秩序，防止数据被乱改。但如果用错了，那可就是自找麻烦了。所以，这篇博客的目标就是让咱们一起深入浅出地理解Java中的锁机制，无论你是新手还是有经验的开发者，相信都能从中学到一些东西。

### 基础知识回顾

在咱们深入研究之前，让我们先复习一下并发编程的基础知识。并发编程，简单来说，就是让多个任务同时运行。但这里的“同时”并不意味着字面上的同一时刻，而是指在一个较短的时间内，任务看起来像是同时执行的。就像咱们用电脑听音乐边写代码一样，虽然CPU在不停地在这两者之间切换，但对咱们来说，就像它们是同时进行的。

现在，让我们来聊聊锁。在Java中，锁是用来控制多线程访问共享资源的一种机制。为啥需要锁呢？想象一下，如果两个线程同时修改同一个数据，结果可就乱套了。锁的作用，就是确保在任一时刻，只有一个线程能访问特定的资源。这就像是咱们家的洗手间，锁上门的时候，别人就不能进来，从而保证了使用的独立性和安全。

下面是一个简单的Java代码示例，展示了如何使用synchronized关键字来实现锁的功能：

```java
public class Counter {
    private int count = 0;

    
    public synchronized void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}

```

这个例子中，`increment` 方法被 `synchronized` 修饰，这意味着在任一时刻，只有一个线程可以访问这个方法。这就是最基础的锁机制在Java中的应用。别着急，后面咱们还会看到更多复杂和强大的锁的使用方法。

[![](_assets/7a1a10dfe51d4e6894ba84143525266d~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)
](https://link.juejin.cn/?target=https%3A%2F%2Fimgse.com%2Fi%2FpiyC3pq "https://imgse.com/i/piyC3pq")

### Java中的锁类型

首先，咱们得知道，Java里的锁大致分为两种：内部锁（synchronized）和显式锁（如ReentrantLock）。咱们先聊聊内部锁。

**内部锁（synchronized）** 小黑告诉你，synchronized是Java原生支持的锁机制，用起来挺方便的。它基于对象监视器原理，可以用于代码块或方法。比如说：

```java
synchronized(this) {
    
}

public synchronized void method() {
    
}

```

这里，`synchronized`确保只有一个线程能执行同步代码块或方法。但别忘了，它是非公平锁，所以线程抢占的顺序可能是随机的。

接下来，咱们来看显式锁。

**显式锁（ReentrantLock等）** 显式锁更灵活一些，提供了更多的功能。ReentrantLock是个好例子。它可以实现公平锁或非公平锁，还可以尝试锁定，限时等待，还有条件变量支持。看下面这个例子：

```java
ReentrantLock lock = new ReentrantLock(true); 
try {
    lock.lock();
    
} finally {
    lock.unlock();
}

```

这个锁可以手动上锁和解锁，所以控制权更在咱们手上，但也更容易忘记解锁，造成问题。

#### 锁类型

**1\. 内部锁（synchronized）**

*   用法：同步方法和同步块。
*   特点：内置于Java对象中，简单易用，但功能相对有限。

**2\. 重入锁（ReentrantLock）**

*   用法：显式地加锁和解锁。
*   特点：比synchronized更灵活，支持公平锁和非公平锁，提供了条件变量（Condition）的支持。

**3\. 读写锁（ReadWriteLock）**

*   用法：分为读锁（共享锁）和写锁（排他锁）。
*   特点：允许多个读操作同时进行，但写操作是排他的。

**4\. 偏向锁、轻量级锁、重量级锁**

*   这些是synchronized的锁状态，Java虚拟机（JVM）为了提高性能，会根据竞争情况动态地改变锁的状态。

**5\. 原子变量类锁（如AtomicInteger）**

*   用法：在java.util.concurrent.atomic包中。
*   特点：提供原子操作，用于小范围的同步，性能高于锁。

**6\. StampedLock**

*   是ReadWriteLock的增强版，提供了一种乐观的读锁模式，可以在没有写操作时提高并发度。

**7\. 分段锁**

*   用于提高在一些容器，如ConcurrentHashMap中的并发性能。

### 锁的高级特性

聊完了锁的类型，咱们来看看锁的一些高级特性。

**公平性和非公平性** 公平锁意味着等待最久的线程会先获取锁。这听起来很公平，但实际上会增加开销。为什么呢？因为系统得维护一个等待队列，来确保顺序。相反，非公平锁可能会让新请求的线程先得到锁，这样更高效，但也可能造成某些线程一直得不到锁。

**可重入性** 所谓可重入锁，就是指同一个线程可以多次获取同一个锁，而不会发生死锁。小黑举个例子：

```java
public class ReentrantExample {
    public synchronized void outerMethod() {
        innerMethod();
    }

    public synchronized void innerMethod() {
        
    }
}

```

这里，同一个线程可以在`outerMethod`中调用`innerMethod`，而不会发生死锁，因为它已经持有了那个锁。

**锁的优化和性能考量** 锁的使用是个技术活儿。小黑建议，尽量减少锁的范围和持有时间，避免嵌套锁，这样可以减少死锁风险，提高性能。

### 避免死锁

咱们聊聊死锁，这可是Java并发编程中的一个大难题。首先，让小黑来告诉你什么是死锁。简单来说，死锁就像是两个小孩互不相让，抓着对方的玩具不放手，结果谁也玩不成。在Java中，如果多个线程相互等待对方释放锁，而这个锁正是对方需要的，那么就会发生死锁。

那么，咱们怎么避免死锁呢？首先，小黑推荐几个简单的策略：

1.  **避免一次性申请所有资源**：就像排队买饭，不要一次性抢所有的菜，一道菜一道菜来。
2.  **使用定时锁**：给锁一个超时时间，如果时间到了还没获取到锁，就放弃吧。
3.  **死锁检测**：就像安装一个监控相机，一旦发现死锁，就采取措施。

来，看个例子：

```java
public class DeadlockDemo {
    private final Object resource1 = new Object();
    private final Object resource2 = new Object();

    public void method1() {
        synchronized (resource1) {
            System.out.println("Resource 1 locked by method1");
            synchronized (resource2) {
                System.out.println("Resource 2 locked by method1");
            }
        }
    }

    public void method2() {
        synchronized (resource2) {
            System.out.println("Resource 2 locked by method2");
            synchronized (resource1) {
                System.out.println("Resource 1 locked by method2");
            }
        }
    }
}

```

这里，`method1` 和 `method2` 分别锁定了两个资源，但顺序相反，容易造成死锁。

[![](_assets/8e94a90c8c7341dbb86ecedaf4788e5d~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)
](https://link.juejin.cn/?target=https%3A%2F%2Fimgse.com%2Fi%2FpiyC810 "https://imgse.com/i/piyC810")

### Java并发工具类中的锁

现在咱们来看看Java的并发工具包`java.util.concurrent`里的锁。这个包里的锁，简直就像是各种高级工具，让并发编程变得简单又高效。

一个重要的类是`ReadWriteLock`。这个锁允许多个读操作同时进行，但写操作就需要独占了。这非常适合读多写少的场景。看看下面的例子：

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockDemo {
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private int value;

    public void read() {
        rwLock.readLock().lock();
        try {
            System.out.println("Reading value: " + value);
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public void write(int newValue) {
        rwLock.writeLock().lock();
        try {
            value = newValue;
            System.out.println("Writing value: " + value);
        } finally {
            rwLock.writeLock().unlock();
        }
    }
}

```

在这个例子里，`read`方法可以被多个线程同时调用，但是一旦一个线程开始执行`write`方法，所有的读写操作都得等它完成。

### 案例分析

咱们来聊聊Java中锁的实际应用，通过一些真实场景的案例分析，理解锁的作用和实际操作。假设小黑正在开发一个在线购物平台，要处理用户下单的并发请求。在这种情况下，锁就显得尤为重要了。

```java

public class Inventory {
    private int stock;  

    
    public synchronized void decreaseStock() {
        if (stock > 0) {
            stock--;
            System.out.println("库存减少，当前库存：" + stock);
        } else {
            System.out.println("库存不足");
        }
    }

    
    public synchronized void increaseStock(int amount) {
        stock += amount;
        System.out.println("库存增加，当前库存：" + stock);
    }
}

```

在这个例子中，小黑使用`synchronized`关键字来保证在减少库存（`decreaseStock`）和增加库存（`increaseStock`）时的线程安全。如果没有锁，多个用户同时下单可能会导致库存数不准确。

