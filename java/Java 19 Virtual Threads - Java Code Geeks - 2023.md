# Java 19: Virtual Threads - Java Code Geeks - 2023
in [Core Java](https://www.javacodegeeks.com/category/java/core-java) January 4th, 2023 752 Views

Java 19 introduces [Virtual Threads](https://bugs.openjdk.org/browse/JDK-8277131), which are lightweight threads designed to improve application throughput. This is a preview language feature so must be enabled using `--enable-preview`.

As you know, the JDK implements platform threads (`java.lang.Thread`) as thin wrappers around operating system (OS) threads. OS threads are expensive to create, and the number of threads is limited to the number of OS threads that the underlying hardware can support. A platform thread captures the OS thread for the code’s entire lifetime.

On the other hand, a virtual thread is an instance of `java.lang.Thread` that is not tied to a particular OS thread and does not capture the OS thread for the code’s entire lifetime. A virtual thread consumes an OS thread only while it performs calculations on the CPU. This means that several virtual threads can run their Java code on the same OS thread, effectively sharing it. When code running in a virtual thread calls a blocking I/O operation, the JVM performs a non-blocking OS call and automatically suspends the virtual thread until it can be resumed later.

Virtual threads help to improve the throughput of thread-per-request style server applications in particular because such applications consist of a great number of concurrent tasks that are not CPU bound and spend much of their time waiting.

Here is an example which creates 10,000 virtual threads; however, the JDK runs the code on perhaps only one OS thread. If we were using 10,000 platform threads (and thus 10,000 OS threads) instead, the program might crash, depending on the hardware available. Virtual threads are cheap and plentiful, and there is no need to pool them.

```

```

The [`java.lang.Thread`](https://docs.oracle.com/en/java/javase/19/docs/api/java.base/java/lang/Thread.html) class has been updated with new methods to create virtual and platform threads, such as:

```

```

Published on Java Code Geeks with permission by Fahd Shariff, partner at our [JCG program](https://www.javacodegeeks.com/join-us/jcg/). See the original article here: [Java 19: Virtual Threads](http://fahdshariff.blogspot.com/2022/12/java-19-virtual-threads.html)

Opinions expressed by Java Code Geeks contributors are their own.