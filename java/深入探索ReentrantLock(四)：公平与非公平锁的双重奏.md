# 深入探索ReentrantLock(四)：公平与非公平锁的双重奏
前言
--

在并发编程中，锁是管理共享资源访问的关键机制之一。Java并发包（`java.util.concurrent`）中的`ReentrantLock`类提供了一个比内置`synchronized`关键字更灵活的锁实现。它不仅支持重入性（即同一个线程可以多次获得锁），还提供了公平锁和非公平锁两种模式，以满足不同场景下的需求。本文将深入探讨这两种锁模式的差异、应用场景以及性能考量。

* * *

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/1e9a6735a2d442dab65ce195c8a87347~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5qmY5a2Q6Z2S6KGr:q75.awebp?rk3s=f64ab15b&x-expires=1728553807&x-signature=HVsDonV7H4tAHJP4S3N2ZsFD9AU%3D)

一、ReentrantLock**非公平锁**
-----------------------

### 1.介绍

在并发编程领域，非公平锁作为锁机制的一种重要形式，尤其在Java的ReentrantLock类中，被设定为默认模式。非公平锁的设计哲学在于，它并不遵循严格的先进先出（FIFO）原则来分配锁资源。当锁被当前持有者释放时，系统不会依据线程请求锁的顺序来授予锁，而是允许任何正在尝试获取该锁的线程直接参与竞争，锁的分配更多地依赖于线程的调度策略和操作系统的即时状态。这种机制可能导致所谓的“插队”现象，即新到达的线程可能先于长时间等待的线程获得锁，从而增加了锁获取的不确定性。

```csharp

ReentrantLock lock = new ReentrantLock();

```

源码如下：

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/d9db590bd07647f0a6275a3fae51b78b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5qmY5a2Q6Z2S6KGr:q75.awebp?rk3s=f64ab15b&x-expires=1728553807&x-signature=%2FmAdfNmex2D2muBrQD6G1dOO2GM%3D)

### 2.优点与适用场景

**优点解析**：

*   高性能：非公平锁的核心优势在于其高效性。由于无需维护一个严格的等待队列，并减少了对队列操作的开销（如入队、出队、排序等），这使得非公平锁在大多数并发场景下能够展现出更优的性能表现。特别是在高并发环境中，减少这些额外操作可以显著提升系统的整体吞吐量。
*   简单高效：从实现角度来看，非公平锁的设计更为简洁，减少了系统资源的消耗。它直接利用操作系统的线程调度机制来管理锁的分配，避免了复杂的队列管理逻辑，从而提高了系统的运行效率。
*   灵活性：在某些应用场景中，锁的获取顺序并非关键因素，而系统的响应速度和吞吐量更为重要。非公平锁提供了这种灵活性，允许系统根据当前的实际情况快速响应，而不必拘泥于严格的顺序要求。

**适用场景**：

*   对锁获取顺序不敏感的场景：在这些场景中，锁的主要作用是保护共享资源，防止数据竞争，而锁的获取顺序并不会对业务逻辑产生显著影响。
*   高性能要求的应用：在对响应时间有严格要求的系统中，如实时交易系统、高频交易系统等，非公平锁能够凭借其高效性优势，帮助系统实现更快的响应速度和更高的吞吐量。
*   高并发环境：在高并发场景下，系统的负载较重，维护一个有序的等待队列可能会成为性能瓶颈。此时，采用非公平锁可以有效减轻系统的负担，提升系统的整体性能。

###  3.案例

> 以下示例展示了ReentrantLock非公平锁在实际应用中的使用效果。
> 
> ```java
> import java.util.concurrent.locks.ReentrantLock;
> 
> public class ReentrantLockExample {
> 
>     public static void main(String[] args) {
>         Task task = new Task();
>         new Thread(task, "Thread-1").start();
>         new Thread(task, "Thread-2").start();
>         new Thread(task, "Thread-3").start();
>         new Thread(task, "Thread-4").start();
>         new Thread(task, "Thread-5").start();
>     }
> }
> 
> class Task implements Runnable {
> 
>     
>     ReentrantLock lock = new ReentrantLock();
> 
>     @Override
>     public void run() {
>         String name = Thread.currentThread().getName();
>         try {
>             lock.lock();
>             System.out.println(name + " 已获得锁");
>         } finally {
>             lock.unlock();
>         }
>     }
> }
> 
> ```
> 
> 运行结果：
> 
> ![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/d3e2b6d32cbb42269e8e886b8578a94b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5qmY5a2Q6Z2S6KGr:q75.awebp?rk3s=f64ab15b&x-expires=1728553807&x-signature=QSMETvXJn%2BZR5aYeMWP1jDs5qPg%3D)
> ​

二、ReentrantLock公平锁
------------------

### 1.介绍

公平锁是一种并发控制机制，旨在确保在多线程环境中，所有请求锁的线程都能按照它们请求锁的顺序来获得锁，从而避免了线程间的饥饿现象。在ReentrantLock类中，当构造函数被显式地设置为true时，该锁即进入公平锁模式。这一模式下，系统内部维护了一个FIFO（先进先出）的等待队列，严格按照线程请求锁的顺序来管理和分配锁资源。

```csharp

ReentrantLock lock = new ReentrantLock(true);

```

源码如下：

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/1ecaacb9498445b6897d6567c4975384~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5qmY5a2Q6Z2S6KGr:q75.awebp?rk3s=f64ab15b&x-expires=1728553807&x-signature=UQaK9GhwUN%2BygDERD7DYWJ%2Fp1ZA%3D)

### 2.优缺点与适用场景

**优点解析**：

*   公平性保证：公平锁的核心优势在于其严格的锁分配策略，确保了所有等待的线程都有平等的机会获得锁，从而有效防止了“饥饿”线程的出现，即某些线程因长时间无法获得锁资源而导致执行延迟或停滞。
*   可预测性增强：在需要高度确定性和可预测性的系统设计中，公平锁能够提供一个稳定且可预测的锁获取顺序，这对于维持系统的稳定性和提高调试效率至关重要。特别是在需要严格控制任务执行顺序的场景中，公平锁的优势尤为明显。

**缺点剖析**：

*   性能开销增大：尽管公平锁带来了诸多优势，但其实现机制也带来了额外的性能负担。每次锁释放时，系统都需要执行复杂的队列操作以确保锁的公平分配，这不可避免地增加了系统的响应时间和整体开销。
*   资源消耗增加：为了维护有序的等待队列，公平锁需要额外的数据结构和复杂的逻辑处理来管理队列中的线程。这些额外的数据结构（如链表或队列）和相应的算法实现，都会增加系统的内存占用和CPU消耗，从而可能影响系统的整体性能。

**适用场景**：

*   高度依赖公平性的业务场景：在需要确保线程公平访问共享资源的业务逻辑或算法实现中，公平锁是不可或缺的工具。例如，金融交易系统、票务预订系统等，都需要保证操作的公平性和有序性。
*   调试与测试阶段：在软件开发的调试和测试阶段，公平锁可以帮助开发者更准确地分析锁竞争和死锁等问题。通过模拟和观察线程间的锁请求和获取过程，开发者可以更容易地定位和修复潜在的并发问题。

###  3.案例

> 以下示例展示了ReentrantLock公平锁在实际应用中的使用效果。
> 
> ```java
> import java.util.concurrent.locks.ReentrantLock;
> 
> public class ReentrantLockExample {
> 
>     public static void main(String[] args) {
>         Task task = new Task();
>         new Thread(task, "Thread-1").start();
>         new Thread(task, "Thread-2").start();
>         new Thread(task, "Thread-3").start();
>         new Thread(task, "Thread-4").start();
>         new Thread(task, "Thread-5").start();
>     }
> }
> 
> class Task implements Runnable {
> 
>     
>     ReentrantLock lock = new ReentrantLock(true);
> 
>     @Override
>     public void run() {
>         String name = Thread.currentThread().getName();
>         try {
>             lock.lock();
>             System.out.println(name + " 已获得锁");
>         } finally {
>             lock.unlock();
>         }
>     }
> }
> 
> ```
> 
> 运行结果：
> 
> ![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/e60a66e59d434a02bd44878ca4dd6053~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5qmY5a2Q6Z2S6KGr:q75.awebp?rk3s=f64ab15b&x-expires=1728553807&x-signature=Y%2Boz6NC5FkD52W8mvlf%2FbEL8RB4%3D)
> ​

* * *

总结
--

选择ReentrantLock的公平锁或非公平锁模式是一项需要综合考虑多方面因素的决策，开发者应根据实际应用的需求、性能要求以及系统负载的波动，灵活调整锁的策略，并结合合理的并发设计，来构建高效、稳定的并发应用。本文主要介绍了ReentrantLock的公平锁与非公平锁，助力开发者构建高效稳定的并发应用。