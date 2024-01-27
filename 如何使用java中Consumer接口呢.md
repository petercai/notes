# 如何使用java中Consumer接口呢
Java 8 引入了 `java.util.function` 包，其中包含了一些常用的函数式接口，如 `Consumer`、`Function`、`Predicate` 等。

`Consumer` 接口是其中一个函数式接口，用于表示接受一个输入参数并执行某种操作的操作者。它包含一个抽象方法 `accept(T t)`，该方法接受一个泛型类型的参数 `T`，并且没有返回值。 源码：

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Consumer<T> {

    
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    
     * Returns a composed {@code Consumer} that performs, in sequence, this
     * operation followed by the {@code after} operation. If performing either
     * operation throws an exception, it is relayed to the caller of the
     * composed operation.  If performing this operation throws an exception,
     * the {@code after} operation will not be performed.
     *
     * @param after the operation to perform after this operation
     * @return a composed {@code Consumer} that performs in sequence this
     * operation followed by the {@code after} operation
     * @throws NullPointerException if {@code after} is null
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}

```

`@FunctionalInterface`是一个标记注解，该注解并非强制性的，但它可以帮助编译器检查是否符合函数式接口的定义。

以下是如何在 Java 中使用 `Consumer` 接口的一些建议：

### 1\. 基本用法：

```java
import java.util.function.Consumer;

public class ConsumerExample {

    public static void main(String[] args) {
        
        Consumer<String> myConsumer = s -> System.out.println(s);

        
        myConsumer.accept("Hello, Consumer!");
    }
}

```

### 2\. 组合多个 Consumer：

你可以使用 `andThen` 方法将多个 `Consumer` 组合在一起，形成一个串行执行的操作链。

```java
import java.util.function.Consumer;

public class ConsumerExample {

    public static void main(String[] args) {
        
        Consumer<String> printUpperCase = s -> System.out.println(s.toUpperCase());
        Consumer<String> printLength = s -> System.out.println("Length: " + s.length());

        
        Consumer<String> combinedConsumer = printUpperCase.andThen(printLength);

        
        combinedConsumer.accept("Hello, Consumer!");
    }
}

```

### 3\. 使用方法引用：

你可以使用方法引用来引用已有的方法，从而创建 `Consumer` 的实例。

```java
import java.util.function.Consumer;

public class ConsumerExample {

    public static void printUpperCase(String s) {
        System.out.println(s.toUpperCase());
    }

    public static void main(String[] args) {
        
        Consumer<String> myConsumer = ConsumerExample::printUpperCase;

        
        myConsumer.accept("Hello, Consumer!");
    }
}

```

这里多说一句，方法引用本质上是lambda表达式的变体,lambda表达式本质上是匿名内部类的变体，Consumer能接收lambda表达式作为实例化，也可以用方法引用来作为实例化，后续再产出一篇文章来讲这个，插眼记录。[下一篇](https://juejin.cn/post/7308434313675898917 "https://juejin.cn/post/7308434313675898917")

### 4\. 在集合操作中使用：

`Consumer` 接口在 Java 8 引入的函数式编程特性中，经常与集合框架的方法结合使用，例如 `forEach` 方法：

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;

public class ConsumerExample {

    public static void main(String[] args) {
        List<String> words = Arrays.asList("Java", "is", "fun");

        
        Consumer<String> printUpperCase = s -> System.out.println(s.toUpperCase());
        words.forEach(printUpperCase);
    }
}

```

这些示例展示了如何使用 `Consumer` 接口执行各种操作，包括基本用法、组合、方法引用以及集合操作中的应用。