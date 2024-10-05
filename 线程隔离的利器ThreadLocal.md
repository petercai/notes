# 线程隔离的利器ThreadLocal
  在多线程环境下，多个线程共享相同的数据时，容易出现数据竞争和一致性问题。为了解决这些问题，Java 提供了多种方式，如同步锁、线程池等。

#### 什么是ThreadLocal

  **ThreadLocal** 是一种非常特殊且高效的机制，ThreadLocal是一个在多线程中为每一个线程创建单独的变量副本的类。**当使用ThreadLocal来维护变量时, ThreadLocal会为每个线程创建单独的变量副本, 避免因多线程操作共享变量而导致的数据不一致的情况**。

#### ThreadLocal基本用法

```java
public class ThreadLocalExample {
    private static ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 1);

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            threadLocal.set(100);
            System.out.println("Thread 1: " + threadLocal.get()); 
        });

        Thread t2 = new Thread(() -> {
            threadLocal.set(200);
            System.out.println("Thread 2: " + threadLocal.get()); 
        });

        t1.start();
        t2.start();
        
        System.out.println("main : " + threadLocal.get()); 
    }
}

```

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/5879cf6247aa4dfabc59f46b8c036faf~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5LiN54ix5oC757uT55qE6bqm56mX:q75.awebp?rk3s=f64ab15b&x-expires=1728522715&x-signature=6FpenHPeNsiwWWNeMNfi7RciDQU%3D)

  可以看到**每个线程都有自己独立的 ThreadLocal值，互不干扰。** 

ThreadLocal 类中包含几个常用的方法，用来操作线程局部变量：

*   **`set(T value)`** : 设置当前线程的本地变量值。
*   **`get()`** : 获取当前线程的本地变量值。
*   **`remove()`** : 删除当前线程的本地变量，避免内存泄漏。
*   **`withInitial(Supplier<? extends T> supplier)`** : 初始化 ThreadLocal 值。

#### ThreadLocal 的实现原理

##### 如何实现线程隔离

  Thread对象中的一个ThreadLocalMap类型的变量threadLocals, 负责存储当前线程相关的ThreadLocal对象。

```java
ThreadLocal.ThreadLocalMap threadLocals = null;

```

  **ThreadLocal 是怎样为线程分配变量副本？**

```scss
public void set(T value) {
    
    Thread t = Thread.currentThread();
    
    ThreadLocalMap map = getMap(t);
    if (map != null)
    
        map.set(this, value);
    else
     
        createMap(t, value);
}

```

  **当我们调用 `ThreadLocal` 的 `set()` 方法时，实际上是把值存储在当前线程的 `ThreadLocalMap` 中。而 `get()` 方法则从当前线程的 `ThreadLocalMap` 中获取对应的值。** 

  **ThreadLocal实现变量的多线程隔离，其实就是用了Map的数据结构来给当前线程当缓存, 要使用的时候就从本线程的threadLocals对象中获取就可以了, key就是当前线程。** 

#### ThreadLocal造成内存泄露

  如果用线程池来操作ThreadLocal 对象确实会造成内存泄露, 因为对于线程池里面不会销毁的线程, 里面总会**存在着<ThreadLocal, LocalVariable>的强引用**,而ThreadLocalMap 对于 Key 虽然是弱引用, 但是**强引用不会释放, 弱引用当然也会一直有值**, 同时创建的LocalVariable对象也不会释放, 就造成了内存泄露。

*   **强引用**：我们平时new了一个对象就是强引用，例如` Object obj = new Object();`即使在内存不足的情况下，JVM宁愿抛出OutOfMemory错误也不会回收这种对象。
    
*   **弱引用**：弱引用的对象拥有更短暂的生命周期。如果一个对象只有弱引用存在了，则下次GC**将会回收掉该对象**（不管当前内存空间足够与否）
    

  所以, 为了避免出现内存泄露的情况, **在调用ThreadLocal的get(),set(),remove()的时候都会清除线程ThreadLocalMap里所有key为null的value。** 

#### 总结

  **ThreadLocal 的作用是提供线程内的局部变量，其生命周期跟Thread一样长。在我们使用ThreadLocal时为了避免出现内存泄露的情况,在分配了ThreadLocal后 ，需调用ThreadLocal的get(),set(),remove()的方法。**