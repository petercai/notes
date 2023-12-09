# 深入理解和使用volatile关键字

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8abf526fcd144ed9097aa58b62a4f10~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=979&h=429&s=56630&e=png&b=ffffff)
](https://link.juejin.cn/?target=https%3A%2F%2Fzimgs.com%2Fi%2Flip4Xq "https://zimgs.com/i/lip4Xq")

### 第1章：引言

在Java的世界里，掌握并发编程是一项必备技能，尤其是当咱们处理多线程应用时。你可能听说过这样的情况：即使你的代码看起来毫无问题，但在并发环境下，它们就像是刚从床上起来的头发，乱七八糟！为什么会这样呢？原因在于多线程操作时存在的一些难以察觉的陷阱，比如变量的可见性问题、操作的原子性问题等等。

Java提供了多种机制来处理这些问题，其中volatile关键字就是一个重要的工具。可能有人会问，这个volatile到底是个什么东西？简单来说，它是Java提供的一种轻量级的同步机制。但别小看了这个“轻量级”，它在确保变量在多线程环境下的可见性方面，可是有着不可小觑的作用。在接下来的内容中，小黑将带你深入了解volatile，以及它在Java并发编程中的应用和局限性。

### 第2章：并发编程中的挑战

当咱们谈到并发编程时，就不得不提到几个经典的问题：可见性、原子性和有序性。这些问题看起来可能有点晦涩，但别担心，小黑来帮你一一捋清。

#### 可见性问题

**可见性问题**：这是指当多个线程访问同一个变量时，一个线程对这个变量的修改，其他线程可能不会立即看到。想象一下，你在看一个直播，而直播的内容实际上是延迟的，你看到的并不是实时发生的事情。这就是可见性问题，非常狡猾对吧？

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d69ef4c73a6448786eaf119258314a4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=889&h=649&s=23308&e=png&b=ffffff)
](https://link.juejin.cn/?target=https%3A%2F%2Fzimgs.com%2Fi%2FlipzVU "https://zimgs.com/i/lipzVU")

**线程1**(Thread 1) 向共享变量写入数据。 数据被写入主内存(Memory)。 主内存确认写入操作。 线程1通知线程2关于变量的更改。 **线程2**(Thread 2) 可能仍然在其缓存中有旧值。 线程2尝试读取共享变量。主内存返回的值可能是过时的。

#### 原子性问题

**原子性问题**：原子性是指一个操作要么完全执行，要么完全不执行，不能停在中间步骤。比如，你在网上订餐，要么整个订单处理完成，要么就是没有任何变化，不能出现订了一半的情况。在并发编程中，如果没有适当的措施，就可能导致原子性问题。

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63110c6241a54a0084cc73bf8faebb87~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=759&h=649&s=23572&e=png&b=ffffff)
](https://link.juejin.cn/?target=https%3A%2F%2Fzimgs.com%2Fi%2FliptlZ "https://zimgs.com/i/liptlZ")

*   **线程1(Thread 1)** 尝试访问共享资源。
*   线程1试图修改资源。
*   如果操作是原子的（Atomic Operation）：
    *   资源向线程1确认操作成功。
*   如果操作非原子性（Non-Atomic Operation）：
    *   **线程2(Thread 2)** 同时尝试访问资源。
    *   这可能导致数据竞争。
    *   资源通知线程1操作失败。
    *   资源通知线程2操作成功。

#### 有序性问题

**有序性问题**：在Java程序中，代码的执行顺序可能与编写顺序不同，这是因为编译器和处理器可能会对指令进行重排序，以优化程序性能。但这种优化有时候会导致意想不到的问题。

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4ae15cc0de84d86981936c0805542bd~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=653&h=647&s=19865&e=png&b=ffffff)
](https://link.juejin.cn/?target=https%3A%2F%2Fzimgs.com%2Fi%2FlipYiH "https://zimgs.com/i/lipYiH")

*   **线程 1 (Thread 1)** 尝试访问资源。
*   **线程 2 (Thread 2)** 也尝试访问同一个资源。
*   由于没有同步机制，两个线程的访问顺序变得不可预测，导致竞态条件（Race Condition）。
*   线程 1 修改资源。
*   线程 2 同时修改资源。
*   由于缺乏同步，资源状态变得不一致。

针对这些问题，Java提供了一系列的解决方案，而volatile关键字正是其中一个重要的工具。它主要用来解决可见性问题，但使用时也有一些限制。

> PS: 小黑收集整理了一份超级全面的**复习面试资料包**，在这偷偷分享给你~ 链接：[sourl.cn/CjagkK](https://link.juejin.cn/?target=https%3A%2F%2Fsourl.cn%2FCjagkK "https://sourl.cn/CjagkK") 提取码：yqwt

### 第3章：什么是volatile关键字

好了，现在咱们来深入了解一下volatile这个“神秘”的关键字。在Java中，volatile是一种用于声明变量的修饰符。它告诉JVM和编译器，这个变量可能会被多个线程同时访问，而且还不通过锁来控制。这听起来有点像是给变量加了一个“注意”标签，让它在并发环境下表现得更好。

首先，小黑给大家强调一下，volatile主要解决的是可见性问题。可见性，就像它字面上的意思，确保当一个线程修改了volatile变量的值时，其他线程能够立即知道这个改变。这听起来很简单，但在并发编程中，这个特性非常重要。为什么呢？因为在多线程环境中，每个线程可能在自己的工作内存中保留了变量的副本，这就导致了一个线程对变量的修改，其他线程不一定能立即看到。

再来看看volatile如何工作。当你把一个变量声明为volatile后，Java虚拟机就会确保所有的读写操作都是直接在主内存中进行的。这样一来，就不会存在线程内部缓存变量副本的问题了，任何一个线程对这个变量的修改都会立即反映到主内存中，同时，其他线程对这个变量的读取也都是直接从主内存进行的。

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7ccf018850b48b3b8d44b5878136d24~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=653&h=647&s=19865&e=png&b=ffffff)
](https://link.juejin.cn/?target=https%3A%2F%2Fzimgs.com%2Fi%2FlipYiH "https://zimgs.com/i/lipYiH")

下面小黑用一个小例子来展示volatile的使用。假设有一个简单的场景，我们有一个标志位变量，控制着一个线程的运行状态：

```java
public class VolatileExample {
    private volatile boolean flag = false;

    public void startThread() {
        new Thread(() -> {
            while (!flag) {
                
            }
        }).start();
    }

    public void stopThread() {
        flag = true;
    }
}

```

在这个例子中，`flag`变量被声明为volatile。这意味着，当`stopThread`方法被调用，将`flag`设置为`true`时，正在运行的线程会立即看到这个改变，并退出while循环。

volatile是Java并发编程中一个非常有用的工具，尤其是在处理可见性问题时。但是它并不是万能的，有它的局限性。

### 第4章：volatile的内部工作原理

咱们继续深入探讨volatile关键字。要理解volatile的内部工作原理，咱们得先聊聊Java内存模型（JMM）。在Java中，每个线程都有自己的工作内存（线程栈），用于存储它使用的变量的副本。而volatile关键字的作用，就是确保变量直接从主内存读取和写入，而不是使用线程工作内存中的副本。这样一来，就解决了可见性问题，但同时也带来了一些性能开销。

**1\. 保证可见性**：当小黑把一个变量声明为volatile后，就像是在这个变量上打上了一个不可忽视的标记。这个标记确保每次访问变量时都会从主内存中读取，每次修改变量时都会立即写回主内存。这样，无论哪个线程在访问这个变量，都能看到最新的值。

**2\. 禁止指令重排序**：这是volatile另一个重要的特性。在Java程序中，为了提高性能，编译器和处理器可能会对指令进行重新排序。但是，当涉及到volatile变量时，JVM会确保对这些变量的读写操作不会与其他内存操作进行重排序。这就保证了操作的有序性，避免了一些难以发现的并发问题。

但需要注意的是，**volatile并不保证原子性**。这意味着，尽管对volatile变量的单次读/写操作是原子的，但复合操作（如递增操作）不是原子的。来看个例子：

```java
public class VolatileCounter {
    private volatile int counter = 0;

    public void increment() {
        counter++;  
    }

    public int getCounter() {
        return counter;
    }
}

```

在这个例子中，`counter++`实际上是一个复合操作，包括读取变量、增加变量的值和写回新值三个步骤。在并发环境中，这可能导致不一致的行为。即使`counter`被声明为volatile，它也不能保证递增操作的原子性。

### 第5章：volatile的使用场景和限制

好啦，接下来咱们聊聊volatile的使用场景和它的限制。了解这些对于合理使用volatile来说非常关键。

首先，**何时使用volatile**？简单地说，当咱们想在多个线程之间共享变量时，而且这个变量满足以下条件，就可以考虑使用volatile：

*   变量不依赖于其当前值，或者只有单一的线程修改变量的值。
*   变量没有包含在具有其他变量的不变式中。

举个例子，如果有一个标志位，用来指示某个条件是否满足，而这个条件会影响多个线程的行为，那么把这个标志位声明为volatile是合适的：

```java
public class SharedFlag {
    private volatile boolean flag = false;

    public void setFlag(boolean value) {
        flag = value;
    }

    public boolean isFlagSet() {
        return flag;
    }
}

```

在这个例子中，`flag`变量被声明为volatile，这保证了所有线程都能看到它的最新值。

但是，volatile并不是万能的，它也有自己的局限性。主要局限性是，**volatile不保证原子性**。对于复合操作，比如自增操作`i++`，volatile就无能为力了。如果需要原子性，那就得考虑用`synchronized`或者`java.util.concurrent.atomic`包中的原子类了。

另一个局限性是，volatile不适用于变量的当前值依赖于其先前值的情况。例如，当计数器或累加器等需要根据之前的值来更新时，单纯使用volatile是不够的。这时候，咱们可能需要使用锁或者原子变量。

那么，**volatile和synchronized的区别是什么呢**？简单地说，synchronized不仅解决了可见性问题，还解决了原子性问题。但synchronized的代价是更高的性能开销。所以，如果只需要解决可见性问题，没有原子性要求，使用volatile是一个更轻量级的选择。

### 第6章：代码示例：探索volatile的实际应用

到了这一章，小黑将用一些代码示例来展示volatile在实际应用中的效果。这些例子会帮助大家更直观地理解volatile的使用方法和效果。

**例1：状态标志**

首先，让咱们看一个简单的例子，其中用volatile变量作为一个线程的运行状态标志：

```java
public class StatusFlag {
    private volatile boolean running = true;

    public void runExample() {
        new Thread(() -> {
            while (running) {
                
            }
        }).start();
    }

    public void stop() {
        running = false; 
    }
}

```

在这个例子里，`running`变量被声明为volatile。这确保了当`stop`方法被调用时，改变`running`的值能够立即对所有线程可见，从而安全地停止线程。

**例2：单例模式中的双重检查锁定**

volatile在单例模式的双重检查锁定（Double-Checked Locking）中也很常见。这种模式可以减少同步的开销，同时保证了单例的延迟初始化。

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

```

在这个例子中，`instance`变量被声明为volatile，这防止了指令重排序，确保在对象初始化完成后才设置`instance`变量，从而安全地实现延迟初始化。

**例3：观察volatile变量的内存效果**

最后，让咱们通过一个简单的实验来观察volatile变量的内存效果。这个实验将展示非volatile变量和volatile变量在多线程环境下的行为差异：

```java
public class VolatileDemo {
    int normalVar = 0;
    volatile int volatileVar = 0;

    public void increment() {
        normalVar++;
        volatileVar++;
    }

    public void printValues() {
        System.out.println("Normal Variable: " + normalVar);
        System.out.println("Volatile Variable: " + volatileVar);
    }
}

```

在这个实验中，`normalVar`是一个普通变量，而`volatileVar`是一个volatile变量。通过对比这两个变量在多线程环境中的表现，咱们可以观察到volatile变量在确保可见性方面的效果。

通过这些例子，咱们可以看到volatile在实际编程中的应用场景。它是一个强大的工具，但要记住它的局限性和合适的使用场景。咱们在编写并发程序时，应该根据具体需求选择合适的同步机制。

### 第7章：性能考量

当咱们谈论volatile时，一个不可避免的话题就是性能。虽然volatile在某些场景下是必需的，但它也带来了一些性能开销。让小黑带大家一起来了解一下这方面的情况。

首先，要明白volatile变量的一个重要特性：每次访问都要从主内存中读取或写入。这意味着，与普通变量相比，volatile变量的操作可能会更慢一些，因为它防止了变量值在本地线程缓存中的存储和获取。这种不使用本地缓存的特性，虽然提高了数据的可见性和一致性，但也增加了内存访问的成本。

再来说说volatile的另一个影响：禁止指令重排序。虽然这保证了程序的正确性，但同时也意味着编译器和处理器在优化代码时的灵活性降低了。这种情况下，可能会导致程序的执行效率不如不使用volatile时高。

那么，怎样才能平衡正确性和性能呢？关键在于只在必要时使用volatile。例如，如果你正在处理只由单个线程修改、由多个线程读取的变量，那么使用volatile是合适的。但如果一个变量频繁地被多个线程读写，那么可能需要考虑其他同步机制，比如`synchronized`或`java.util.concurrent`包中的锁机制。

还有一点很重要，那就是测试。在实际应用中，评估不同同步机制对性能的影响是必不可少的。通过性能测试，咱们可以更好地了解在特定场景下使用volatile的成本，以及它是否真的是最合适的解决方案。

### 第8章：总结和最佳实践

走到这里，咱们已经一起探讨了volatile的方方面面，从基本概念到实际应用，再到性能考量。现在，小黑来总结一下关于volatile的关键点，并提供一些最佳实践的建议。

**关键点总结**：

1.  **可见性保证**：volatile确保变量的更新对所有线程立即可见。
2.  **禁止指令重排序**：volatile防止编译器和处理器对相关代码的重排序，保障了代码执行的有序性。
3.  **不保证原子性**：volatile不适用于那些需要原子性保证的操作。

**最佳实践建议**：

1.  **合理应用场景**：当需要确保变量的修改对所有线程立即可见时，使用volatile。例如，状态标志、单例的双重检查锁定。
2.  **避免滥用**：不要在每个变量上都使用volatile。理解其适用场景，并仅在必要时使用。
3.  **配合其他同步工具**：对于复合操作，考虑使用`synchronized`或`java.util.concurrent.atomic`包中的类，如`AtomicInteger`。
4.  **性能测试**：在使用volatile时，进行性能测试，了解其对应用性能的影响。
5.  **代码清晰**：即使使用volatile，也保持代码逻辑清晰和简单。避免过于复杂的并发逻辑，这有助于降低出错的风险。
