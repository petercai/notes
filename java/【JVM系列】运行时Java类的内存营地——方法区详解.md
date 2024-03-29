# 【JVM系列】运行时Java类的内存营地——方法区详解
【JVM系列】运行时Java类的内存营地——方法区详解
---------------------------

### 什么是方法区？

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9da63f8469934215934c84898409409f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=875&h=290&s=30669&e=png&a=1&b=faf7e8)

在HotSpot JVM中，方法区是一个逻辑上的概念，也被称为非堆（Non-Heap），一般用来存储类加载信息、static变量、JIT实时编译缓存的代码、常量池（Constants Pool）等。不同版本的Java其方法区的实现方式不同，在JDK 8之前，采用的是“永久代”来实现方法区，而在JDK 8之后则是采用MetaSpace的方式来实现。总的来说，方法区的基本特点如下：

*   **共享性：**  方法区与Java堆一样，是各个线程共享的内存区域。
*   **创建和内存空间：**  方法区在JVM启动时被创建，实际的物理内存空间和Java堆一样，可以是不连续的。
*   **大小和可扩展性：**  方法区的大小，就像堆空间一样，可以选择固定大小或可扩展。
*   **溢出问题：**  方法区的大小决定了系统能够保存多少个类。如果系统定义了太多的类，导致方法区溢出，虚拟机将抛出内存溢出错误，例如 `java.lang.OutOfMemoryError: PermGen space` 或 `java.lang.OutOfMemoryError: Metaspace`。
*   **释放：**  关闭JVM将释放方法区的内存空间。

方法区是用于存储类信息、常量池、静态变量等数据的区域，对于Java虚拟机的正常运行和类加载等都具有重要作用。

### 方法区的前世今生

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9de9b0f45b154869a754964f3c01c4e3~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=873&h=419&s=38875&e=png&a=1&b=fcf9ea)

在JDK 6中，方法区的实现和JVM的定义几乎完全一样，其实现方式是永久代，是完全独立于Heap的一块儿内存区域，其中类信息、静态变量、运行时常量池、static变量以及字符串常量等内容都是存储在方法区的。但是，由于StringTable存在于方法区，当Java程序中大量生成较大的字符串对象的时候，就可能造成方法区OOM。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e86ff1a178641ccb1649ea5044c0dc3~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=873&h=419&s=38380&e=png&a=1&b=fbf8e9)

在JDK 7中，为了避免方法区频繁溢出，将StringTable和静态变量这两个容易造成方法区溢出的部分移动到了堆内存当中去。jdk7 中将 StringTable 放到了堆空间中。因为永久代的回收率很低，在 full gc 的时候才会触发。 而 full gc 是老年代的空间不足，永久代不足时才会触发。这就导致 StringTable 回收率不高。而我们开发中会有大量的字符串被创建，回收率低，导致永久代内存不足。放到堆里，能及时回收内存。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab9ae7a9e2984c1298c570bdb50b1ab0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=873&h=446&s=47605&e=png&a=1&b=faf6ea)

在JDK 8之后就没有永久代的概念了，而且方法区也完全成为了逻辑概念。方法区在JDK 8之后的对应实现应该是MetaSpace，MetaSpace是由Native内存实现的由操作系统直接管理。类型信息、字段、方法、常量保存在本地内存MetaSpace当中，但字符串常量池、静态变量仍在堆中。这样以来MetaSpace 的空间上限和操作系统相关，当大量加载类的时候不用担心方法区溢出。

#### 永久代（Permanent Generation）和元空间（Metaspace）区别？

永久代是Java虚拟机堆内存的一部分，主要用于存储类的元数据信息。包括类的结构、方法、字段、注解等。在Java 8及之前的版本，永久代有一定的固定大小，当存储的类信息较多时容易导致永久代溢出。此时可能会出现`OutOfMemoryError: PermGen space`错误。

在Java 8及之后的版本，永久代被元空间所取代。元空间不再与Java堆关联，而是使用**本地内存（native memory）**来存储类的元数据信息。这样，就不再存在永久代的大小限制问题。元空间的大小受限于本地内存的大小，可以动态地扩展，减轻了类元数据区溢出的问题。更多的区别和特点如下：

|  | 永久代 | 元空间 |
| --- | --- | --- |
| **存储位置** | 在Java堆内存中 | 使用本地内存（Native Memory） |
| **大小限制** | 固定大小，容易溢出 | 动态扩展，受限于本地内存大小 |
| **回收机制** | 使用垃圾回收器进行垃圾回收 | 不再有垃圾回收，类的元数据信息会在类加载器被卸载时被释放，同时可以设置Metaspace的大小限制 |
| **OutOfMemoryError** | 可能会出现OutOfMemoryError: PermGen space | 由于不再有固定大小的限制，OutOfMemoryError可能发生在本地内存用尽的情况下 |

总体而言，元空间的引入解决了永久代存在的一些限制和问题，提高了对类元数据信息的管理效率和灵活性。在Java 8及之后的版本，开发者不再需要过于关注永久代的大小调优，而是需要关注本地内存的使用情况。

### **设置方法区大小**

#### JDK 7 及之前的实现

在JDK 7及之前，方法区的大小可以通过 `-XX:PermSize` 参数进行设置，用于指定永久代的初始分配空间。通过 `-XX:MaxPermSize` 参数设定永久代的最大可分配空间。当JVM加载的类信息容量超过了这个值，就会抛出 `OutOfMemoryError: PermGen space` 异常，如下表：

| 配置参数 | 描述 | 默认值 |
| --- | --- | --- |
| `-XX:PermSize` | 永久代的初始分配空间大小 | 20.75MB |
| `-XX:MaxPermSize` | 永久代的最大可分配空间大小 | 32位机: 64MB, 64位机: 82MB |

#### JDK 8 及以后的实现

在JDK 8及以后，方法区的永久代被元空间（Metaspace）所取代，可以通过 `-XX:MetaspaceSize` 和 `-XX:MaxMetaspaceSize` 参数来设置元空间的大小。默认值依赖于平台， `-XX:MaxMetaspaceSize` 的值为-1，则表示没有限制。与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有可用的系统内存。当元数据大小超过可用内存时，虚拟机会抛出 `OutOfMemoryError: Metaspace` 异常。

| 配置参数 | 描述 | 默认值 |
| --- | --- | --- |
| `-XX:MetaspaceSize` | 元空间的初始大小 | 平台依赖，例如Windows下为21MB |
| `-XX:MaxMetaspaceSize` | 元空间的最大可分配空间大小 | 默认值-1，表示没有限制 |

*   `-XX:MetaspaceSize`: 用于设置初始的元空间大小。对于64位的服务器JVM，默认值为21MB。这是初始的高水位线，一旦达到这个水位线，Full GC将被触发，卸载不再存活的类，然后重置高水位线。新的高水位线取决于GC释放了多少空间。如果释放的空间不足，适当提高该值，不超过`MaxMetaspaceSize`。如果释放的空间过多，则适当降低该值。
*   如果初始化的高水位线设置过低，会导致高水位线调整多次，观察垃圾回收器的日志可以看到Full GC的多次调用。为了避免频繁的GC，建议将 `-XX:MetaspaceSize` 设置为一个相对较高的值。

### 方法区的内部结构

方法区在 Java 虚拟机内存中扮演了类的元数据存储和管理的角色。它不仅保存了类的结构信息，还包括了支持类运行时行为的相关数据。方法区的设计使得虚拟机能够高效地加载、存储和管理类信息，保证类的正确性和一致性。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21068fcd7ad64d7281c1861e63a9d69d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1371&h=631&s=72869&e=png&a=1&b=fbf7e8)

方法区主要用于存储已被虚拟机加载的类信息、常量、静态变量以及即时编译器编译后的代码缓存等，如下：

#### 类型信息

对于每个加载的类型（类class、接口interface、枚举enum、注解annotation），JVM 方法区中存储以下类型信息：

1.  **类型的完整有效名称：**  例如，对于类`java.lang.String`，存储的是完整的包名和类名。
2.  **类型直接父类的完整有效名：**  对于接口或是`java.lang.Object`，它们没有直接的父类。
3.  **类型的修饰符：**  存储类型的修饰符，可能包括`public`、`abstract`、`final`等的某个子集，用于描述该类型的访问权限、抽象性质和是否可以被继承等信息。
4.  **类型直接接口的一个有序列表：**  存储类型实现的接口列表，有序表示接口的顺序，这对于多接口实现的情况很重要。

这些信息使得虚拟机能够准确地了解每个加载类型的结构和关系，包括其继承层次、实现接口、访问权限等。这对于类的加载、链接和运行时行为都具有重要作用。

#### **运行时常量池 vs 常量池（Constant Pool）**

常量池包含了编译期生成的各种字面量和符号引用，包括字符串、数字、类和方法的符号引用等。在运行时，这些常量供虚拟机使用。

*   **方法区中包含了运行时常量池：**  运行时常量池是方法区的一部分，用于存储编译期生成的各种字面量和符号引用。在类加载后，字节码文件中的常量池会被加载到运行时常量池中，供运行时使用。运行时常量池是在程序运行期间动态生成和修改的，可以包含字符串、数字、类和方法的符号引用等信息。它在方法区中的位置使得虚拟机能够更加灵活地管理和使用常量池中的信息。
    
*   **字节码文件中包含了常量池：**  字节码文件的常量池是在编译期生成的，包含了类中的常量、类名、字段名、方法名等信息。它是一种静态的、不可变的常量池，保存了在编译时确定的字面量和符号引用。字节码文件中的常量池在类加载时被加载到方法区的运行时常量池中，供虚拟机执行时使用。
    

总体而言，字节码文件中的常量池是类的静态结构的一部分，而运行时常量池是类在运行时动态生成的。运行时常量池的动态性使得它可以包含更多在运行期间生成的信息，而字节码文件中的常量池则提供了类的静态元数据。这两者的结合使得虚拟机能够更好地支持动态语言特性和类的动态变化。

#### static变量

##### non-final 的static变量

静态变量（static）和类紧密关联在一起，随着类的加载而加载，它们成为类数据在逻辑上的一部分。non-final 的static变量被类的所有实例共享，即使没有类实例时，也可以通过类名直接访问。

non-final的static变量的特点在于其值可以在程序运行过程中被修改，因此它们可以被看作是在类级别共享的可变状态。这使得它们适用于存储在整个类生命周期中需要共享的数据，例如计数器、配置信息等。类加载时分配内存空间用于存储这些静态变量，并且它们在整个应用程序的生命周期内保持不变，直到程序结束或类被卸载。

non-final的static变量的修改会影响到所有实例，因为它们在逻辑上属于整个类，而不是某个特定的实例。这也意味着不同实例之间共享相同的non-final的static变量值。

##### **全局常量：static final**

被声明为 final 的类变量在编译时会被分配，每个全局常量在编译的时候就会被确定了。这意味着在程序运行时，这些全局常量的值是不可修改的，它们被视为一种稳定的、不可变的数据。

使用 `static final` 声明的变量通常用于表示在整个类范围内都保持不变的常量值。这些常量可以在类加载时被初始化，且一经初始化就无法被修改。由于它们是静态的，可以通过类名直接访问，而不需要创建类的实例。

全局常量的命名通常采用大写字母和下划线的命名规范，以便清晰地区分它们与普通变量。例如：

```java
public class Constants {
    public static final int MAX_VALUE = 100;
    public static final String DEFAULT_NAME = "John Doe";
}

```

在上述例子中，`MAX_VALUE` 和 `DEFAULT_NAME` 都是全局常量，它们的值在编译时确定，并且在运行时不可修改。

#### **即时编译器（JIT）编译后的代码缓存：** 

方法区在 Java 虚拟机中不仅存储类的元数据信息，还承担了即时编译器（JIT）生成的本地机器代码的存储任务。即时编译器是一种优化技术，它将部分热点代码（经常执行的代码路径）编译成本地机器代码，以提高程序的执行性能。

具体而言，方法区中存储了以下内容：

1.  **本地机器代码：**  JIT 编译器将部分 Java 字节码编译成本地机器代码，这样可以更高效地执行。这些本地机器代码通常涉及程序的热点部分，即频繁执行的代码段。
2.  **代码缓存：**  编译后的本地机器代码被缓存，以避免重复编译相同的代码。缓存机制有助于减少重复工作，提高程序的运行效率。当相同的代码段再次执行时，虚拟机可以直接使用缓存中的本地机器代码，而不必重新进行编译。

通过将热点代码编译成本地机器代码，并进行缓存，JIT 编译器在运行时动态地优化程序的性能。这种动态编译和优化的机制使得虚拟机能够更好地适应不同的程序运行情境，提高程序的执行效率。方法区的设计使得它能够承担这一重要角色，为 Java 程序的性能提供支持。

#### 域（Field)信息

JVM 在方法区中保存类型的所有域的相关信息以及域的声明顺序。域的相关信息包括：

1.  **域名称：**  表示字段的名字，例如，对于类中的属性，如 `int age;` 中的 "age"。
2.  **域类型：**  表示字段的数据类型，例如，对于类中的属性，如 `int age;` 中的 "int"。
3.  **域修饰符：**  表示字段的修饰符，可能包括以下某个子集：
    *   `public`: 表示该字段对所有类可见。
    *   `private`: 表示该字段仅对声明它的类可见。
    *   `protected`: 表示该字段对其自雷和同一包内的类可见。
    *   `static`: 表示该字段为静态字段，属于类而非实例。
    *   `final`: 表示该字段为常量，不能被修改。
    *   `volatile`: 表示该字段可能会被多个线程同时访问，因此需要进行同步处理。
    *   `transient`: 表示该字段不会被序列化。

这些信息使得虚拟机能够了解每个字段的名称、类型、访问权限、是否为静态字段以及其他修饰信息，从而能够正确地处理字段的访问和操作。这对于类的实例化、访问字段以及字段的初始化等操作都具有重要作用。

#### 方法（Method) 信息

JVM 必须保存所有方法的以下信息，与域信息类似，包括声明顺序：

1.  **方法名称：**  表示方法的名字，例如，对于一个方法，如 `public void doSomething()` 中的 "doSomething"。
2.  **方法的返回类型：**  表示方法返回值的数据类型，例如，对于一个方法，如 `public int calculate()` 中的 "int"。
3.  **方法参数的数量和类型（按顺序）：**  表示方法接收的参数的数量和类型，例如，对于一个方法，如 `public void setName(String name, int age)` 中的 "(String, int)"。
4.  **方法的修饰符：**  表示方法的修饰符，可能包括以下某个子集：
    *   `public`: 表示该方法对所有类可见。
    *   `private`: 表示该方法仅对声明它的类可见。
    *   `protected`: 表示该方法对其自雷和同一包内的类可见。
    *   `static`: 表示该方法为静态方法，属于类而非实例。
    *   `final`: 表示该方法不能被子类覆写。
    *   `synchronized`: 表示该方法在多线程环境中需要同步。
    *   `native`: 表示该方法用其他语言（如C）实现。
    *   `abstract`: 表示该方法没有实现，需要在子类中被覆写。
5.  **方法的字节码（bytecodes)、操作数栈、局部变量表及大小：**  这些信息用于存储方法的具体实现，包括方法体的字节码指令、操作数栈的状态、局部变量表的内容及其大小。这些信息对于虚拟机执行方法体的字节码指令非常关键。
6.  **异常表：**  用于存储方法中可能抛出的异常的处理信息，包括每个异常处理块的开始位置、结束位置、代码处理在程序计数器中的偏移地址以及被捕获的异常类的常量池索引。这些信息帮助虚拟机在出现异常时进行适当的处理。

这些方法信息使得虚拟机能够正确地加载、解析和执行方法，处理方法的调用、返回值、异常处理等操作。

### **字符串常量、静态变量数据存放区域变化**

在 Java 6 中，所有常量池数据都被存放在永久代中。永久代是 Java 虚拟机内存区域的一部分，用于存储类的元数据、常量池等信息。这个阶段，永久代就是方法区的实现方案。

然而，随着 Java 7 的引入，HotSpot 虚拟机对永久代的管理进行了调整。具体而言，HotSpot 将字符串常量和静态变量数据从永久代迁移到了堆中。这个变化的目的是为了缓解永久代的内存压力，因为永久代在一些场景下容易发生内存溢出。

在 Java 8 中，这个变化并没有被保留了下来，字符串常量和静态变量数据仍然存放在堆中，而不再是永久代。这意味着在 Java 8 中，虽然常量池在 JVM 规范中定义为方法区的一部分，但实际上部分常量池的内容是保存在堆中的。

总体来说，Java 虚拟机规范将常量池归类为方法区的一部分，但HotSpot 在实际实现中对部分常量池内容进行了迁移，将其保存在堆中。这一调整旨在提高对于字符串常量和静态变量数据的内存管理效率。

### 方法区OOM（Java 8之后）

在Java 8及之后的版本，永久代（PermGen）被元空间（Metaspace）所取代。因此，发生在Java 8及之后版本的方法区OOM通常指的是Metaspace的OOM。

Metaspace是一块与堆不同的本地内存区域，它用于存储类的元数据信息，包括类的结构、方法、字段等信息。Metaspace的OOM与永久代的OOM有一些相似之处，但也有一些区别。

常见原因导致Metaspace的OOM：

1.  **动态生成类过多：**  如果应用程序在运行时动态生成大量的类，例如使用字节码操作库或动态代理，可能导致Metaspace内存不足。
2.  **持续加载大量不同的类：**  如果应用程序持续加载大量不同的类，Metaspace可能会被耗尽。
3.  **缺少Metaspace的垃圾回收：**  在Java 8及之后的版本，Metaspace不再有垃圾回收的概念，因此如果动态生成的类或加载的类不再使用，Metaspace也不会自动释放相关内存。这可能导致Metaspace逐渐增长，最终耗尽内存。

解决Metaspace的OOM：

1.  **调整Metaspace大小：**  可以通过JVM参数（如`-XX:MaxMetaspaceSize`）来调整Metaspace的大小。增大Metaspace的内存限制可能会缓解OOM，但需要确保整个系统内存足够。
2.  **优化类加载和卸载：**  确保应用程序中只加载必要的类，避免不必要的类加载。使用工具监控类加载和卸载的情况，查看是否有无用的类没有被卸载。
3.  **检查动态生成类的逻辑：**  如果应用程序中有动态生成类的逻辑，确保生成的类数量有限，并且可以被及时卸载。
4.  **监控Metaspace使用情况：**  使用JVM的监控工具，如VisualVM或JConsole，来监控Metaspace的使用情况，及时发现潜在的OOM问题。

总的来说，Metaspace的OOM问题需要特别关注动态生成类的逻辑和类加载的行为，以及及时调整Metaspace的内存大小。

#### 方法区的垃圾回收

Java 8及之后的版本使用元空间（Metaspace）替代了永久代，Metaspace是在堆之外的本地内存。其空间的大小默认不受限制，可以根据系统的实际内存情况动态扩展。因此，不再存在永久代溢出的问题，但仍然需要关注系统内存的使用情况。

MetaSpace的垃圾回收主要发生在类的卸载时，一般由JVM自动完成。当一个类不再被引用，而且它的加载器也不再存活，MetaSpace会触发对该类的垃圾回收。MetaSpace的垃圾回收主要涉及到不再使用的类的卸载。判定一个类型是否属于“不再被使用的类”的条件则比较苛刻，需要同时满足以下三个条件：

1.  该类的所有实例都已经被回收，即Java堆中不存在该类及其任何派生子类的实例。
2.  加载该类的类加载器已经被回收，这个条件通常难以达成，除非是一些特殊场景如OSGI、JSP的重加载等。
3.  该类对应的java.lang.Class对象没有在任何地方被引用，无法通过反射访问该类的方法。

Java虚拟机被允许对满足上述三个条件的无用类进行回收，这里强调的是“被允许”，而不是像对象一样没有引用就必然回收。

总结
--

方法区是Java运行时内存区域非常重要的一个区域，这个区域主要用来存储一些类加载信息、static变量、运行时常量池和JIT实时编译的缓存代码等。

实际上，方法区到现在已经是一个逻辑概念了，在JDK6之前方法区的实现方式是永久带，永久带是一块独立的内存区域，几乎和逻辑上的方法区保持一致。到JDK8之后，方法区的实现方法改成了MetaSpace，MetaSpace是一块儿属于Native内存的区域，只负责存储类相关的信息、static变量和JIT实时编译的缓存代码等。像字符串常量和常量池的存储区域就被放到了Heap中，此时方法区已经完全成为了一个逻辑上的概念。

最后，对于MetaSpace来说一般很难出现OOM的情况，因为MetaSpace出现OOM意味着整个Native的内存已经被耗尽，但是仍然可以通过调整MetaSpace的相关参数来优化JVM的性能。