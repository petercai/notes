# Java中的这些String特性可能需要了解下 
String类具有以下特性：

1.  **不可变性（Immutable）**：String对象一旦创建就不能被修改。任何对String对象的操作都会返回一个新的String对象，原始对象保持不变。
2.  **字符串表（String Table）**：StringTable表是一种存储字符串常量的内存区域，它可以提高字符串的重用率和性能。在创建字符串时，如果字符串已经存在于池中，则返回池中的字符串对象，否则会创建一个新的字符串对象并放入池中。
3.  **值传递**：在Java中，String对象是通过值传递的方式传递的。这意味着当将一个字符串传递给方法或赋值给另一个变量时，实际上传递的是字符串的副本而不是原始字符串对象。

下文将详细说明这些特性。

本文基于JDK17说明。

不可变性（Immutable）
---------------

String的不可变性指的是一旦创建了String对象，它的值就不能被修改。

这意味着在任何对String对象进行操作时，都会返回一个新的String对象，而原始对象的值保持不变。

这种特性有助于保护数据的一致性，并且在多线程环境下也更加安全。

下面是一个示例来说明String的不可变性：

```java
public class ImmutableStringExample {
    public static void main(String[] args) {
        String original = "Hello";
        String modified = original.concat(", World!");
        
        System.out.println("Original string: " + original);
        System.out.println("Modified string: " + modified);
    }
}

```

输出结果为：

```shell
Original string: Hello
Modified string: Hello, World!

```

在这个例子中，虽然使用了 `concat` 方法对原始字符串进行了修改，但是原始字符串 `original` 的值并没有改变。相反，`concat` 方法返回了一个新的字符串对象，其中包含了修改后的值。

其中`concat`函数的主要代码如下：

```java
@ForceInline
static String simpleConcat(Object first, Object second) {
    String s1 = stringOf(first);
    String s2 = stringOf(second);
    if (s1.isEmpty()) {
        
        return new String(s2);
    }
    if (s2.isEmpty()) {
        
        return new String(s1);
    }
    
    
    long indexCoder = mix(initialCoder(), s1);
    indexCoder = mix(indexCoder, s2);
    byte[] buf = newArray(indexCoder);
    
    
    indexCoder = prepend(indexCoder, buf, s2);
    indexCoder = prepend(indexCoder, buf, s1);
    
    return newString(buf, indexCoder);
}

```

String的不可变性对于设计具有很多优点。

*   提供了一种简单且安全的数据结构，因为任何时候都可以确信一个String对象的值不会在不经意间被改变。
*   由于String是不可变的，所以它们可以被安全地共享，而不必担心在共享的过程中被修改。
*   String的不可变性也有助于提高字符串操作的性能，因为它可以避免频繁的复制和重建字符串对象。

String的不可变性使得它在Java中成为一种简单、安全且高效的数据结构。

不可变性怎么保证的
---------

String 的不可变性是通过类的设计、内部实现和方法设计来保证的，这种不可变性使得 String 对象在多线程环境下更加安全，并且可以被方便地共享和重用。

String 的不可变性是通过以下几种方式来保证的：

*   **String 类的设计**：String 类被设计为 final 类，这意味着它不能被继承，也就是说无法创建 String 的子类来修改其行为。这样就防止了通过继承来修改 String 类的方法来改变其不可变性。
*   **String 对象的内部实现**：String 对象内部使用字节数组 `byte[]` 来存储字符串的值，而且这个字节数组是被声明为 `final` 的，即不可修改。一旦一个 String 对象被创建，它的值就会被存储在这个字节数组中，而且这个值是无法被修改的。也没对外提供get、set方法。
*   **String 类的方法**：String 类中的方法都被设计成不会修改原始对象的值，而是返回一个新的 String 对象，其中包含了修改后的值。比如，对于字符串连接操作 `concat()`、子串提取 `substring()`、大小写转换 `toUpperCase()` 和 `toLowerCase()` 等方法，都会返回一个新的 String 对象，而不会修改原始字符串。

如下是String对象的部分源码，可以看到value和对象都被final修饰。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence,
               Constable, ConstantDesc {
    @Stable
    private final byte[] value;
    
}

```

值传递
---

在Java中，String对象的传递是通过值传递（pass by value）进行的。

这意味着在将String对象传递给方法或赋值给另一个变量时，传递的是对象的副本而不是对象本身。

当你将一个String对象传递给方法时，实际上传递的是对象的引用的副本，而不是对象本身。这意味着方法内部的操作不会影响原始的String对象，因为它们操作的是副本。

下面是一个示例来说明String的值传递：

```java
public class StringValuePassingExample {
    public static void main(String[] args) {
        String original = "Hello";
        modifyString(original);
        System.out.println("Original string after method call: " + original);
    }

    public static void modifyString(String str) {
        str = str + ", World!";
        System.out.println("Modified string inside method: " + str);
    }
}

```

输出结果为：

```shell
Modified string inside method: Hello, World!
Original string after method call: Hello

```

在这个例子中，虽然在 `modifyString` 方法内部对 `str` 进行了修改，但原始的 `original` 字符串并没有受到影响。这是因为在方法调用时，传递的是 `original` 字符串的副本，而不是原始对象本身。

因此，在方法内部对 `str` 的任何修改都不会影响原始的 `original` 字符串。

字符串存储在StringTable
-----------------

StringTable是一种特殊的内存区域，用于存储字符串常量。

当创建字符串时，如果该字符串已经存在于StringTable中，则直接返回对该字符串的引用，而不会创建新的字符串对象；如果该字符串不在StringTable中，则会创建一个新的字符串对象，并将其添加到StringTable中。

如果字符串是动态创建的，比如通过new、concat、substring、toUpperCase动态创建的会放到堆内存中。

StringTable、字符串、堆的示意图如下所示：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d6cf2f8f0a143e0aa16fd9b971bdba0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=521&h=582&s=43211&e=jpg&b=fdfdfd)

StringTable的设计有几个主要原因：

*   **节省内存空间**：由于字符串常常是应用程序中使用的不可变的常量，因此可以被多个字符串引用。通过StringTable，可以确保相同的字符串常量在内存中只有一份拷贝，从而节省内存空间。
*   **提高性能**：由于字符串常量在内存中只有一份拷贝，所以可以通过比较字符串的引用地址来比较字符串的内容，而不必比较字符串的实际内容，这样可以提高比较字符串的效率。
*   **保证字符串的唯一性**：通过StringTable，可以确保在应用程序中使用的字符串常量是唯一的，这有助于减少由于字符串拼接等操作而引起的错误。
*   **方便字符串的共享和重用**：由于StringTable中的字符串常量是唯一的，因此可以方便地共享和重用字符串常量，从而提高应用程序的性能和效率。

字符串比拼
-----

```java
import java.util.HashMap;

public class StringTableDemo {
    public static void main(String[] args) {
        String str1 = "abc";
        String str2 = new String("abc");
        System.out.println(str1 == str2);
        String str3 = new String("abc");
        System.out.println(str3 == str2);
        String str4 = "a" + "b";
        System.out.println(str4 == "ab");
        String s1 = "a";
        String s2 = "b";
        String str6 = s1 + s2;
        System.out.println(str6 == "ab");
        String str7 = "abc".substring(0, 2);
        System.out.println(str7 == "ab");
        String str8 = "abc".toUpperCase();

        System.out.println(str8 == "ABC");
        String s5 = "a";
        String s6 = "abc";
        String s7 = s5 + "bc";
        System.out.println(s6 == s7.intern());
    }
}

```

**通过以上的例子可以总结出以下规律：** 

*   单独使用`""`引号创建的字符串都是常量，编译期就已经确定存储到StringPool中。
*   使用new String("")创建的对象会存储到heap中，是运行期新创建的。
*   使用只包含常量的字符串连接符如"aa"+"bb"创建的也是常量，编译期就能确定已经存储到StringPool中。
*   使用包含变量的字符串连接如"aa"+s创建的对象是运行期才创建的，存储到heap中。
*   运行期调用String的`intern()`方法可以向String Pool中动态添加对象。

