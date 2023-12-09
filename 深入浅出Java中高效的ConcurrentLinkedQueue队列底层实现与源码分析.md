# 深入浅出Java中高效的ConcurrentLinkedQueue队列底层实现与源码分析

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/658f85faf1ea4e85a5dd3bbcdc9dbb11~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=7019&h=4963&s=1255864&e=png&b=ffffff)

* * *

  多线程开发已成为现代软件开发的基础。在多线程开发中，线程之间的通信和数据同步是非常重要的，而队列是实现线程间通信和数据同步的重要工具。本文将介绍Java中高效的ConcurrentLinkedQueue队列的底层实现和源码分析。

  本文将介绍ConcurrentLinkedQueue队列的底层实现和源码分析，包括应用场景案例、优缺点分析、类代码方法介绍和测试用例。通过阅读本文，读者可以了解Java中高效的ConcurrentLinkedQueue队列的原理和使用方法。

概述
--

  ConcurrentLinkedQueue是Java并发包中的一个线程安全的无界队列实现，使用CAS（Compare And Swap）算法实现线程安全。ConcurrentLinkedQueue是一个基于链表结构的队列，它可以保证在多线程环境下的高效性和安全性。在多线程并发的场景中，ConcurrentLinkedQueue是一个非常好的选择。

源代码解析
-----

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable {
        
    
     * Head of linked list.
     * Invariant: head.item == null
     */
    private transient volatile Node<E> head;
    
    
     * Tail of linked list.
     * Invariant: last.next == null
     */
    private transient volatile Node<E> tail;
    
    public ConcurrentLinkedQueue() {
        head = tail = new Node<E>(null);
    }
    
    private void updateHead(Node<E> h, Node<E> p) {
        if (h != p && casHead(h, p))
            h.lazySetNext(p);
    }

    private void updateTail(Node<E> t, Node<E> p) {
        if (t != p && casTail(t, p))
            t.lazySetNext(p);
    }

    public boolean offer(E e) {
        checkNotNull(e);
        final Node<E> newNode = new Node<E>(e);

        for (Node<E> t = tail, p = t;;) {
            Node<E> q = p.next;
            if (q == null) {
                if (p.casNext(null, newNode)) {
                    if (p != t)
                        updateTail(t, newNode);
                    return true;
                }
            } else if (p == q) {
                p = (t != (t = tail)) ? t : head;
            } else {
                p = (p != t && t != (t = tail)) ? t : q;
            }
        }
    }

    public E poll() {
        restartFromHead:
        for (;;) {
            for (Node<E> h = head, p = h, q;;) {
                E item = p.item;
                if (item != null && p.casItem(item, null)) {
                    if (p != h) 
                        updateHead(h, ((q = p.next) != null) ? q : p);
                    return item;
                } else if ((q = p.next) == null) {
                    updateHead(h, p);
                    return null;
                } else if (p == q) {
                    continue restartFromHead;
                } else {
                    p = q;
                }
            }
        }
    }

    private static class Node<E> {
        volatile E item;
        volatile Node<E> next;
        Node(E item) {
            
            this.item = item;
        }

        boolean casItem(E cmp, E val) {
            return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
        }

        void lazySetNext(Node<E> val) {
            UNSAFE.putOrderedObject(this, nextOffset, val);
        }

        boolean casNext(Node<E> cmp, Node<E> val) {
            return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
        }

        private static final sun.misc.Unsafe UNSAFE;
        private static final long itemOffset;
        private static final long nextOffset;
        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();
                Class<?> k = Node.class;
                itemOffset = UNSAFE.objectFieldOffset
                    (k.getDeclaredField("item"));
                nextOffset = UNSAFE.objectFieldOffset
                    (k.getDeclaredField("next"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }
    }
}

```

  ConcurrentLinkedQueue的底层实现是基于链表结构的。   它有两个重要的属性：head和tail。其中head是链表的头节点，tail是链表的尾节点。当ConcurrentLinkedQueue中没有元素的时候，head和tail是同一个节点。当ConcurrentLinkedQueue中有一个元素的时候，head是链表的头节点，tail是链表的尾节点。当ConcurrentLinkedQueue中有多个元素的时候，tail指向的是最后一个元素所在的节点。

**代码分析**

  这段代码实现了一个线程安全的队列 ConcurrentLinkedQueue，它是基于链表实现的。它包含两个成员变量 head 和 tail，分别表示链表的头和尾。当队列为空时，head 和 tail 指向同一个节点，该节点的 item 域为 null。

  offer 方法用于在队列的尾部添加新元素，它首先创建一个新的节点 newNode，然后使用一个 for 循环，不断尝试将 newNode 添加到队列的尾部。循环中的变量 t 表示当前节点的 tail，p 表示要添加 newNode 的前一个节点。首先，通过 p 节点的 next 指针找到 p 的后继节点 q，如果 q 为 null，则说明 p 是队列的最后一个节点，此时可以通过 CAS 操作将 newNode 添加到 p 的 next 指针上。如果 CAS 操作成功，则更新 tail 指针。否则，说明有另一个线程已经修改了 p 的 next 指针，那么当前线程需要重新获取节点的 tail 和 p。如果 p 和 t 不相等，则更新 t 的指针并重新获取节点的 tail 和 p。

  poll 方法用于从队列的头部移除一个元素并返回它。它首先通过一个无限循环从 head 节点开始遍历链表，尝试找到第一个不为 null 的节点 p，然后使用 CAS 操作将 p 的 item 值设置为 null，表示该节点已经被移除。如果 CAS 操作成功，则判断 p 是否为队列的头节点 h，如果不是，则说明队列中间有一个或多个节点已经被移除，需要将这些节点也移除并更新 head 指针。如果 p 是队列的头节点，则直接返回 p 的 item 值。如果 p 的 next 指针为 null，则说明队列已经为空，直接返回 null。如果 p 和 p 的后继节点相同，则说明队列正在被修改，需要重新从 head 节点开始遍历链表。

  Node 类是链表节点的实现。它包含一个 item 值和一个 next 指针，分别表示节点的元素和后继节点。该类使用了 CAS 操作来保证多线程情况下链表节点的修改是安全的。它还包含了一些静态变量和代码块，用于获取 item 和 next 字段的偏移量，以及获取 UNSAFE 对象。这些变量和方法通常不需要我们关注，是为了实现 CAS 操作而引入的。

  如下是部分源码截图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e359f6ebe4704142a0d111086cb00d80~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1211&h=936&s=110348&e=png&b=2c2c2c)

### offer方法

offer方法用于添加元素到队列中。它采用了一个基于自旋锁的算法。

*   首先，通过checkNotNull(e)方法检查元素是否为null。
*   然后，通过尾节点tail获取当前的节点p。
*   如果当前节点p的下一个节点q是null，说明当前节点是链表的尾节点。
*   如果当前节点p的下一个节点q不是null，则说明当前节点不是链表的尾节点，需要重新找到尾节点。
*   如果节点p的下一个节点q是null，则尝试使用CAS算法将新节点添加到链表中。
*   如果CAS操作成功，更新尾节点tail的指针。
*   如果CAS操作失败，则重新回到第2步。

### poll方法

poll方法用于从队列中弹出一个元素。它采用了一个基于自旋锁的算法。

*   首先，获取链表的头节点head和当前节点p。
*   如果p不是null，且p的item不是null，说明当前节点p是要被弹出的节点。
*   如果CAS操作成功，更新头节点的指针。如果当前节点p不是头节点，则需要调用updateHead方法更新头节点的指针。
*   如果当前节点p是头节点，则直接更新头节点的指针。
*   如果p的下一个节点q是null，则说明队列已经为空，直接返回null。
*   如果p的下一个节点q就是头节点head，则说明其他线程正在修改链表，需要重新从头节点开始操作。
*   如果p的下一个节点q既不是null，也不是头节点head，则说明其他线程正在修改链表，需要直接跳到节点q继续操作。

应用场景案例
------

  ConcurrentLinkedQueue适用于多线程并发的场景中，例如生产者消费者模式，线程池等。

  在生产者消费者模式中，ConcurrentLinkedQueue可以作为任务队列使用。生产者线程向队列中添加任务，消费者线程从队列中取出任务并执行。

  在线程池中，ConcurrentLinkedQueue可以作为任务队列使用。线程池中的线程从队列中取出任务并执行。

优缺点分析
-----

### 优点

*   线程安全：ConcurrentLinkedQueue是线程安全的，可以在多线程并发的环境中使用。
*   高效性：ConcurrentLinkedQueue采用了基于链表结构和CAS算法的实现方式，使其在多线程并发的场景中具有较高的效率。
*   无界队列：ConcurrentLinkedQueue是一个无界队列，可以动态地添加元素，不需要事先确定队列的大小。

### 缺点

*   不支持阻塞操作：ConcurrentLinkedQueue不支持阻塞操作，无法在队列为空时等待元素。

类代码方法介绍
-------

### Node类

Node类是ConcurrentLinkedQueue的内部类，表示链表中的一个节点。它具有以下属性：

*   item：表示节点的元素。
*   next：表示节点的下一个节点。

Node类定义了以下方法：

*   casItem(E cmp, E val)：比较并交换节点的元素。
*   lazySetNext(Node val)：无锁操作设置节点的下一个节点。
*   casNext(Node cmp, Node val)：比- compare And Swap节点的下一个节点。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2db360267364b9881f26b9a97ed0c3d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1089&h=561&s=53193&e=png&b=2b2b2b)

### ConcurrentLinkedQueue类

ConcurrentLinkedQueue类是ConcurrentLinkedQueue的主要实现类，它继承自AbstractQueue类和实现Queue、Serializable接口。它具有以下属性：

*   head：表示链表的头节点。
*   tail：表示链表的尾节点。

ConcurrentLinkedQueue类定义了以下方法：

*   offer(E e)：将元素添加到队列中。如果添加成功，则返回true，否则返回false。
*   poll()：从队列中弹出一个元素。如果队列为空，则返回null。
*   updateHead(Node h, Node p)：更新头节点指针。
*   updateTail(Node t, Node p)：更新尾节点指针。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13c5c708e2bb4026951e666d9cf0897c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1084&h=757&s=61071&e=png&b=2c2c2c)

测试用例
----

为了验证ConcurrentLinkedQueue的功能和性能，我们可以编写如下测试用例：

### 测试代码演示

```java
package com.demo.javase.day69;

import java.util.concurrent.ConcurrentLinkedQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;


 * @Author bug菌
 * @Date 2023-11-06 16:00
 */
public class ConcurrentLinkedQueueTest {

    public static void main(String[] args) throws InterruptedException {
        final int THREAD_COUNT = 1000;
        final int ELEMENT_COUNT = 100;
        ExecutorService executorService = Executors.newFixedThreadPool(THREAD_COUNT);

        ConcurrentLinkedQueue<Integer> queue = new ConcurrentLinkedQueue<>();

        
        for (int i = 0; i < ELEMENT_COUNT; i++) {
            executorService.execute(() -> {
                for (int j = 0; j < ELEMENT_COUNT; j++) {
                    queue.offer(j);
                }
            });
        }

        executorService.shutdown();
        executorService.awaitTermination(10, TimeUnit.SECONDS);

        assert ELEMENT_COUNT * ELEMENT_COUNT == queue.size();

        
        executorService = Executors.newFixedThreadPool(THREAD_COUNT);
        for (int i = 0; i < ELEMENT_COUNT; i++) {
            queue.offer(i);
        }

        for (int i = 0; i < THREAD_COUNT; i++) {
            executorService.execute(() -> {
                for (int j = 0; j < ELEMENT_COUNT; j++) {
                    Integer element = queue.poll();
                    assert element != null;
                }
            });
        }

        executorService.shutdown();
        executorService.awaitTermination(10, TimeUnit.SECONDS);

        System.out.println("queue.size() = " + queue.size()); 
    }
}

```

  测试用例中，testAdd方法模拟了1000个线程同时向队列中添加100个元素的场景，并校验队列中元素的数量是否正确。testPoll方法模拟了1000个线程同时从队列中取出100个元素的场景，并校验队列中元素的数量是否正确。通过执行这些测试用例，可以验证ConcurrentLinkedQueue的功能和性能。

### 测试结果

  根据如上测试用例，本地测试结果如下，仅供参考，你们也可以自行修改测试用例或者添加更多的测试数据或测试方法，进行熟练学习以此加深理解。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73783b1ef72b4371b40f20ecca49c2ee~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1157&h=835&s=72154&e=png&b=2c2c2c)

### 测试代码分析

  根据如上测试用例，在此我给大家进行深入详细的解读一下测试代码，以便于更多的同学能够理解并加深印象。

  该代码为测试并发队列 ConcurrentLinkedQueue 的使用，主要包含两个测试方法：

1.  testAdd：测试并发添加元素。开启多个线程向队列中添加元素；
2.  testPoll：测试并发取出元素。先将若干元素添加到队列中，然后开启多个线程同时取出元素。

  在测试添加元素的过程中，多个线程同时向队列中添加元素，由于 ConcurrentLinkedQueue 内部采用 CAS 算法保证并发的安全性，因此不需要额外的加锁操作，可以保证多线程安全的同时高效地添加元素。

  在测试取出元素的过程中，多个线程同时从队列中取出元素，同样由于 ConcurrentLinkedQueue 内部采用 CAS 算法保证并发的安全性，因此不需要额外的加锁操作，可以保证多线程安全地同时取出元素。

小结
--

  本文主要介绍了Java中高效的ConcurrentLinkedQueue队列的底层实现和源码分析，包括应用场景案例、优缺点分析、类代码方法介绍和测试用例。ConcurrentLinkedQueue是基于链表结构的队列，使用CAS算法实现线程安全，能够保证多线程并发环境下的高效性和安全性。它适用于多线程并发场景中，如生产者消费者模式，线程池等。本文还提供了测试用例，验证了ConcurrentLinkedQueue的功能和性能。

  本文介绍了Java并发包中的ConcurrentLinkedQueue队列，包括其基于链表的底层实现和源码分析，应用场景案例，优缺点分析，以及类代码方法介绍和测试用例。ConcurrentLinkedQueue是一个线程安全的无界队列实现，在多线程并发的场景中具有高效性和安全性。它适用于生产者消费者模式和线程池等多线程并发的场景中。ConcurrentLinkedQueue的优点是线程安全、高效性、无界队列，不足之处在于不支持阻塞操作。本文给出了测试用例，验证了ConcurrentLinkedQueue的功能和性能。
