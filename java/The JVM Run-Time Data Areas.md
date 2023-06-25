# The JVM Run-Time Data Areas
  

1\. Overview[](#overview)
-------------------------

**The [Java Virtual Machine (JVM)](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm) is an abstract computing machine that enables a computer to run Java programs.** The JVM is responsible for executing the instructions contained in the compiled Java code. To do so, it requires a certain amount of memory to store the data and instructions it needs to operate. This memory is divided into various areas.

In this tutorial, we'll discuss the different types of runtime data areas and their purpose. Every JVM implementation must follow [the specifications](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5) explained here.

The JVM has several shared data areas that are shared amongst all threads running in the JVM. **As a consequence, various threads can simultaneously access any of these areas.**

### 2.1. Heap[](#1-heap)

**The [Heap](https://www.baeldung.com/java-stack-heap#heap-space-in-java) is the runtime data area where all Java objects are stored.** Thus, whenever we create a new class instance or array, the JVM will find some available memory in the Heap and assign it to the object. The Heap's creation occurs at the JVM start-up, and its destruction happens at the exit.

As per the specification, an automatic management tool must handle the storage of the objects: this tool is known as the [garbage collector](https://www.baeldung.com/java-destructor#garbage-collection).


There is no constraint about the Heap size in the JVM specifications. Memory handling is also left over to the JVM implementations. Nevertheless, if the garbage collector cannot reclaim enough free space to create a new object, the JVM throws an _OutOfMemory_ error.

### 2.2. Method Area[](#2-method-area)

**The Method Area is a shared data area in the JVM that stores class and interface definitions.** It is created when the JVM starts, and it is destroyed only when the JVM exits.

Concretely, the [class loader](https://www.baeldung.com/java-classloaders) loads the bytecode of the class and passes it to the JVM. The JVM then creates an internal representation of the class, which is used to create objects and invoke methods at runtime. This internal representation gathers information about the fields, methods, and constructors of the classes and interfaces.

Additionally, let's point out that the Method Area is a logical concept. As a result, it may be part of the Heap in a concrete JVM implementation.

Once more, the JVM specifications don't define the size of the Method Area, nor does it define the way the JVM handles memory blocks.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_2)

AD

If the available space in the Method Area is not enough to load a new class, the JVM throws an [_OutOfMemory_](https://www.baeldung.com/java-permgen-space-error) error.

### 2.3. Run-Time Constant Pool[](#3-run-time-constant-pool)

**The [Run-Time Constant Pool](https://www.baeldung.com/jvm-constant-pool) is an area within the Method Area that contains symbolic references to the class and interface names, field names, and method names.**

The JVM takes advantage of the creation of the representation for a class or interface in the Method Area to simultaneously create the Run-Time Constant Pool for this class.

When creating a Runtime Constant Pool, if the JVM needs more memory than is available in the Method Area, an _OutOfMemory_ error is thrown.

3\. Per-thread Data Areas[](#per-thread-data-areas)
---------------------------------------------------

In addition to the shared runtime data areas, the JVM also uses per-thread data areas to store specific data for each thread. **The JVM indeed supports many threads executions at the same time.**

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_3)

AD

### 3.1. PC Register[](#1-pc-register)

Each JVM thread has its [PC (program counter) register](https://www.baeldung.com/cs/os-program-counter-vs-instruction-register). Each thread executes the code of a single method at any given time. The behavior of the PC depends on the nature of the method:

*   **For a non-native method, the PC register stores the address of the current instruction being executed**
*   For a native method, the PC register has an undefined value

Lastly, let's note that the PC register lifecycle is the same as the one of its underlying thread.

### 3.2. JVM Stack[](#2-jvm-stack)

Similarly, each JVM thread has its private [Stack](https://www.baeldung.com/java-stack-heap#stack-memory-in-java). The JVM Stack is a data structure that stores method invocation information. **Each method call triggers the creation of a new frame on the stack to store the method's local variables and the return address.** Those frames can be stored in the Heap.

Thanks to the JVM Stack, the JVM can keep track of the execution of a program and log the [stack trace](https://www.baeldung.com/java-get-current-stack-trace) on demand.

Once again, the JVM specification lets the JVM implementations decide how they want to handle the JVM Stack's size and memory allocation.

A memory allocation error on the JVM Stack entails a _[StackOverflow](https://www.baeldung.com/java-stack-overflow-error)_ error. However, if a JVM implementation allows the dynamic extension of its JVM Stack's size, and if a memory error occurs during the extension, the JVM has to throw an _OutOfMemory_ error.

### 3.3. Native Method Stack[](#3-native-method-stack)

A [native method](https://www.baeldung.com/java-native#native-methods) is a method written in another programming language than Java. These methods aren't compiled to bytecode, hence the need for a different memory area.

**The Native Method Stack is very similar to the JVM Stack but is only dedicated to native methods.**


The purpose of the Native Method Stack is to keep track of the execution of a native method.

The JVM implementation can decide on its own the size of the Native Method Stack and how it handles memory blocks.

As for the JVM Stack, a memory allocation error on the Native Method Stack leads to a _StackOverflow_ error. On the other hand, a failed attempt to increase the Native Method Stack's size leads to an _OutOfMemory_ error.

To conclude, let's note that the specification highlights that a JVM implementation could decide not to support native method calls: such an implementation wouldn't need to implement a Native Method Stack.

4\. Conclusion[](#conclusion)
-----------------------------

In this tutorial, we've elaborated on the different types of runtime data areas and their purpose. These areas are crucial for the proper functioning of the JVM. Understanding them can help optimize the performance of Java applications.