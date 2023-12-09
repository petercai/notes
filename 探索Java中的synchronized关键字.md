# 探索Java中的synchronized关键字

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80a54e34eae64242ac27583408b9d7f4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1256&h=391&s=57114&e=png&b=ffffff)
](https://link.juejin.cn/?target=https%3A%2F%2Fzimgs.com%2Fi%2FlipQuL "https://zimgs.com/i/lipQuL")

### 第1章：引言

咱们程序员在面对多线程编程时，经常会听到一个词——synchronized。这个词在Java世界里就像是一把万能钥匙，打开并发编程的大门。但是，你知道吗？虽然synchronized用得多，但真正深入理解它的人并不多。今天，小黑就带大家一起揭开synchronized的神秘面纱。

在Java的世界里，线程安全问题总是绕不开的话题。不管是在做Web应用、Android开发还是做一些高性能的后端服务，多线程和并发总是咱们的老朋友。而synchronized，作为Java内置的同步机制，是保证多线程环境下数据一致性和线程安全的重要工具。

### 第2章：synchronized关键字基础

那么，synchronized到底是个什么东西呢？简单来说，它是一个同步锁。当咱们在方法上或者代码块上使用synchronized关键字时，它就像是给代码加上了一道锁，确保同一时间只有一个线程可以执行这段代码。

举个例子来说，假设小黑有一个计数器，这个计数器会在多个线程中被访问和修改。如果不使用synchronized，就可能会出现计数错误的情况。看下面这个代码：

```java
public class Counter {
    private int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}

```

这个简单的计数器在多线程环境下就不安全了。因为当多个线程同时调用`increment`方法时，它们可能会看到`count`的旧值，从而导致计数不准确。这时，synchronized就派上用场了：

```java
public class SynchronizedCounter {
    private int count = 0;

    
    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}

```

这样，当一个线程在执行`increment`方法时，其他线程就必须等它执行完毕，才能继续执行，从而保证了线程安全。

但是，synchronized并不是万能的。它虽然解决了线程安全问题，但也带来了性能上的开销。因为当一个线程访问同步锁时，其他线程就必须等待，这就可能导致线程阻塞和等待时间过长的问题。所以，正确理解和使用synchronized，对于写出高效、安全的并发程序来说非常重要。

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2aa10cefacf04e578f172c5153020113~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=801&h=271&s=27776&e=png&b=fcfcfc)
](https://link.juejin.cn/?target=https%3A%2F%2Fzimgs.com%2Fi%2Flip7ZV "https://zimgs.com/i/lip7ZV")

> PS: 小黑收集整理了一份**超级全面的复习面试资料包**，**在这偷偷分享给你**~ 链接：[sourl.cn/gUV3UP](https://link.juejin.cn/?target=https%3A%2F%2Fsourl.cn%2FgUV3UP "https://sourl.cn/gUV3UP") 提取码：fjb3

### 第3章：synchronized的工作原理

#### 字节码层面的锁

当小黑在Java代码中使用synchronized时，这在字节码层面上对应着两种指令：`monitorenter`和`monitorexit`。这两个指令分别用于获取和释放锁。

#### 举个例子

来看个简单的示例，小黑有一个同步方法：

```java
public synchronized void syncMethod() {
    
}

```

在字节码层面，这个方法大概会被转换成：

```csharp

monitorenter
try {
    
} finally {
    monitorexit
}


```

#### 锁的内部工作机制

在Java中，每个对象都有一个监视器锁（Monitor）。当线程进入synchronized块时，它会尝试获取这个监视器锁。如果锁没有被其他线程持有，那么它会成功获取并持有这个锁，并继续执行同步块的代码。如果锁被其他线程持有，它会被阻塞，直到锁被释放。

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4db32c702b84b5f92aea9bf879b4a63~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=971&h=893&s=31843&e=png&b=ffffff)
](https://link.juejin.cn/?target=https%3A%2F%2Fzimgs.com%2Fi%2FliplRI "https://zimgs.com/i/liplRI")

#### 锁的优化：轻量级锁与重量级锁

Java虚拟机为了提高性能，实现了锁的升级机制。最初，当一个线程进入synchronized块时，它会使用轻量级锁。轻量级锁的获取和释放不需要从操作系统获得支持，它主要通过线程间的CAS操作实现。但如果有多个线程竞争同一个锁，轻量级锁就会升级为重量级锁。重量级锁需要操作系统的支持，它会导致线程阻塞。

#### 锁的状态

锁在Java中可以有多个状态，包括无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态。偏向锁是一种特殊的锁，它会偏向于第一个获取它的线程，以减少锁操作的开销。当有更多线程加入竞争时，偏向锁可以升级为轻量级锁，进而升级为重量级锁。

通过深入到字节码指令和锁的内部工作机制，咱们可以看到synchronized是如何在Java虚拟机中实现同步的。虽然它在字节码层面看起来很简单，但背后的优化机制却非常复杂。咱们在使用synchronized时，虽然不需要关心这些复杂的内部细节，但了解它们能帮助我们更好地理解Java的并发机制。

### 第4章：synchronized的使用场景

好啦，咱们已经了解了synchronized的基本概念和工作原理，那么接下来，小黑要聊聊synchronized的使用场景，以及和其他同步工具的比较。

#### 1\. 同步方法

首先，最常见的场景就是同步方法。这个大家都不陌生，就像前面提到的银行账户例子。在方法上添加synchronized关键字，可以确保同一时间只有一个线程可以执行这个方法。这适用于简单的操作，比如更新一个变量或者完成一个简单的事务。

```java
public synchronized void add(int value) {
    this.count += value;
}

```

#### 2\. 同步代码块

但是，如果你的方法里只有部分代码需要同步，那么用同步代码块可能更合适。这样可以减少锁的范围，提高效率。

```java
public void add(int value) {
    synchronized(this) {
        this.count += value;
    }
}

```

#### 3\. 对比其他同步工具

当然，除了synchronized，Java还提供了其他同步工具，比如ReentrantLock。与synchronized相比，ReentrantLock提供了更高的灵活性，比如可以尝试非阻塞地获取锁，或者在给定时间内等待锁。

#### 4\. 使用场景对比

那么，什么时候该用synchronized，什么时候用ReentrantLock呢？简单来说，如果你需要简单的同步机制，用synchronized就够了。但如果你需要更复杂的同步控制，比如锁的公平性、可中断的锁获取等，那么ReentrantLock可能是更好的选择。

#### 5\. 性能考量

还有个角度不能忽视，那就是性能。虽然synchronized在最新的Java版本中已经得到了很大的优化，但在某些高并发场景下，ReentrantLock可能会有更好的性能表现。

synchronized是一个非常强大且易用的同步机制。它适用于大多数的同步需求，尤其是那些不需要复杂同步策略的场景。但在选择synchronized之前，小黑建议咱们先考虑一下需求，确保它是最合适的工具。

### 第5章：synchronized的性能考量和优化

好了，来聊聊大家都关心的问题——性能。synchronized作为内置的同步机制，简单好用，但也有可能成为性能瓶颈。小黑这就来分析一下，同时给咱们一些优化的建议。

#### 1\. 性能影响

使用synchronized时，最大的性能问题就是线程等待。当一个线程持有锁时，其他需要这个锁的线程就会进入等待状态。在高并发的应用中，这种等待可能会导致严重的性能问题。

#### 2\. 减少锁的范围

一种常见的优化方法是减少锁的范围。比如，不是整个方法都加锁，而是只对需要同步的部分代码加锁。这样可以减少线程等待的时间。

```java
public void updateData() {
    
    processData();

    synchronized(this) {
        
        updateDatabase();
    }
}

```

#### 3\. 减少锁的粒度

另一个优化方法是减少锁的粒度。例如，如果有多个资源需要同步，可以为每个资源提供单独的锁，而不是一个锁同步所有资源。

```java
public class Resource {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void doSomething() {
        synchronized(lock1) {
            
        }

        synchronized(lock2) {
            
        }
    }
}

```

#### 4\. 使用其他并发工具

如果性能真的是一个大问题，可以考虑使用Java并发包里的其他工具，比如ReentrantLock。虽然它的使用比synchronized复杂一些，但提供了更多的控制，包括可中断的锁获取、公平性选择等。

#### 5\. 锁的优化

不要忘了Java虚拟机本身对synchronized的优化。自Java 6以来，JVM对synchronized做了很多优化，比如锁消除、锁粗化、自旋锁等。所以，在很多情况下，synchronized的性能已经足够好了。

synchronized在使用时确实需要考虑性能问题。但通过减少锁的范围和粒度，以及合理使用JVM的优化，可以大大减轻这些问题。在选择使用synchronized之前，小黑建议咱们先仔细考虑一下应用的实际需求，以及是否有更合适的并发工具。

### 第6章：synchronized的进阶话题

好的，咱们已经讲了不少关于synchronized的基础内容，现在小黑要带大家深入一些进阶话题，理解synchronized背后的更多秘密。

#### 1\. 锁的状态和优化

synchronized在JVM层面经历了不少优化，其中一个重要概念就是锁的状态。锁在Java中有几种不同的状态，包括无锁状态、偏向锁、轻量级锁和重量级锁。这些状态的转换基于锁竞争的程度，JVM会根据具体情况自动调整。

*   **偏向锁**：这是一种锁的状态，它假设锁主要被一个线程所使用，因此会有所优化。如果同一个线程多次获取锁，这将非常高效。
*   **轻量级锁**：当偏向锁失效时，锁会升级为轻量级锁。轻量级锁在多个线程交替获取锁时效率较高。
*   **重量级锁**：当有多个线程同时竞争同一个锁时，轻量级锁会升级为重量级锁。这是最传统的锁，涉及到操作系统层面的同步。

#### 2\. 锁的膨胀和退化

锁的状态不是固定不变的。在竞争激烈的情况下，锁可以从偏向锁膨胀为轻量级锁，甚至是重量级锁。反之，在竞争减少时，锁也可能退化。

#### 3\. Java内存模型（JMM）与synchronized

Java内存模型（JMM）是理解synchronized的另一个关键。JMM处理了多线程中变量的可见性问题，保证了一个线程写入的值对其他线程是可见的。synchronized在JMM中扮演着重要角色，通过提供内存屏障，它确保了变量的可见性和有序性。

#### 4\. 死锁问题

在讨论synchronized时，不能不提死锁。死锁发生在两个或以上的线程互相等待对方释放锁。理解死锁及其解决方法对于使用synchronized是非常重要的。

```java
public class DeadlockDemo {
    private final Object resource1 = new Object();
    private final Object resource2 = new Object();

    public void method1() {
        synchronized(resource1) {
            
            synchronized(resource2) {
                
            }
        }
    }

    public void method2() {
        synchronized(resource2) {
            
            synchronized(resource1) {
                
            }
        }
    }
}

```

synchronized虽然表面上看简单，但背后其实隐藏着复杂的机制。理解这些机制，可以帮助我们更好地使用synchronized，写出更高效、更安全的并发程序。

### 第7章：总结与展望

#### 1\. 总结

*   **synchronized的重要性**：小黑跟大家介绍了synchronized的基础概念、使用场景、性能考量、进阶知识，以及实际案例分析。咱们看到了synchronized在确保线程安全方面的重要作用，尤其是在多线程环境下操作共享资源时。
*   **性能和优化**：虽然synchronized可能导致性能问题，但通过减少锁的范围、降低锁的粒度，以及合理利用Java虚拟机的锁优化，咱们可以有效地减轻这些问题。

#### 2\. 展望

*   **并发编程的未来**：随着Java版本的更新，synchronized的性能持续提升。同时，Java并发编程还在不断发展，例如Project Loom的引入将为并发编程带来更轻量级的线程和更高效的性能。
*   **新的并发工具**：Java的并发工具箱也在不断丰富，比如CompletableFuture、StampedLock等。这些新工具为高效的并发编程提供了更多的选择。

#### 3\. 结语

作为Java程序员，了解和掌握synchronized是非常重要的。它不仅是实现线程安全的基本工具，也是理解Java并发编程的基石。当然，随着技术的发展，咱们也要持续学习新的工具和技术，保持技术的前瞻性。

希望这系列文章对大家有所帮助。小黑将持续关注Java并发编程的新动态，和大家一起成长。下次再见！

* * *
