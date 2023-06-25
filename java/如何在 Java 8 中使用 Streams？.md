# 如何在 Java 8 中使用 Streams？
Java 8 Streams 是一个非常强大的功能，它提供了一种简洁、优雅的方式来处理数据集合。通过使用 Streams，我们可以轻松地过滤、映射、排序、聚合等操作数据。本教程将介绍 Streams 的基本概念，以及如何在 Java 8 中使用 Streams。本教程还包括许多代码示例，以帮助您更好地理解 Streams 的工作方式。

![](_assets/db3132684c462e0d547d461885ce4200.png)

什么是 Streams？
------------

在 Java 中，Stream 是一个用于操作集合元素的接口。它允许我们通过管道操作（Pipeline）来处理集合元素，从而实现过滤、排序、映射、聚合等操作。Stream 并不是一个集合，而是一个类似于 Iterator 的对象，它支持在集合上进行连续的操作。Stream 不改变原始的集合，而是在每次操作后返回一个新的 Stream 对象。

如何创建 Streams？
-------------

在 Java 8 中，可以使用 Collection.stream() 或 Collection.parallelStream() 方法来创建 Stream 对象。例如：

```
List<String> list = Arrays.asList("one", "two", "three", "four", "five");

// 创建串行流
Stream<String> stream = list.stream();

// 创建并行流
Stream<String> parallelStream = list.parallelStream();
```

也可以使用 Stream.of() 方法来创建一个包含指定元素的 Stream 对象。例如：

```
Stream<String> stream = Stream.of("one", "two", "three", "four", "five");
```

还可以使用 Stream.empty() 方法来创建一个空的 Stream 对象。例如：

```
Stream<String> emptyStream = Stream.empty();
```

如何操作 Streams？
-------------

在 Java 8 中，Stream 提供了一系列方法来操作数据集合，包括中间操作和终止操作。中间操作会返回一个新的 Stream 对象，而终止操作会返回一个非 Stream 的结果。

### 中间操作

中间操作用于在 Stream 上进行连续的转换和过滤。以下是一些常见的中间操作：

*   **filter**：过滤符合条件的元素。
    
*   **map**：对元素进行转换操作。
    
*   **flatMap**：将一个 Stream 中的每个元素都转换成一个新的 Stream，然后将这些 Stream 连接起来形成一个新的 Stream。
    
*   **distinct**：去除重复元素。
    
*   **sorted**：对元素进行排序。
    
*   **peek**：对元素进行遍历操作，通常用于调试和打印日志。
    

以下是一些中间操作的示例：

```
List<String> list = Arrays.asList("one", "two", "three", "four", "five");

// 过滤长度为3的字符串
Stream<String> stream = list.stream().filter(s -> s.length() == 3);

// 转换成大写
Stream<String> stream = list.stream().map(String::toUpperCase);

// 扁平化字符串列表
Stream<String> stream = list.stream().flatMap(s -> Stream.of(s.split(""));
// 去重
Stream<String> stream = list.stream().distinct();

// 排序
Stream<String> stream = list.stream().sorted();

// 遍历输出
list.stream().peek(System.out::println).count();
```

### 终止操作

终止操作用于在 Stream 上执行最终的操作，并返回非 Stream 的结果。以下是一些常见的终止操作：

*   **forEach**：对 Stream 中的每个元素执行操作。
    
*   **count**：返回 Stream 中元素的个数。
    
*   **collect**：将 Stream 中的元素转换成其他形式。
    
*   **reduce**：将 Stream 中的元素进行聚合操作。
    
*   **min**：返回 Stream 中的最小值。
    
*   **max**：返回 Stream 中的最大值。
    

以下是一些终止操作的示例：

```
List<String> list = Arrays.asList("one", "two", "three", "four", "five");

// 遍历输出
list.stream().forEach(System.out::println);

// 计算元素个数
long count = list.stream().count();

// 将元素转换成 List
List<String> newList = list.stream().collect(Collectors.toList());

// 求和
int sum = Stream.of(1, 2, 3, 4, 5).reduce(0, (a, b) -> a + b);

// 求最小值
int min = Stream.of(1, 2, 3, 4, 5).min(Integer::compareTo).get();

// 求最大值
int max = Stream.of(1, 2, 3, 4, 5).max(Integer::compareTo).get();
```

Streams 的并行处理
-------------

在 Java 8 中，Streams 提供了并行处理的功能，可以将集合分成多个部分进行处理，从而提高处理效率。要使用并行 Streams，只需要使用 Collection.parallelStream() 方法来创建一个并行的 Stream 对象即可。

以下是一个示例：

```
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 普通流处理
long start = System.currentTimeMillis();
int sum1 = list.stream().mapToInt(Integer::intValue).sum();
long end = System.currentTimeMillis();
System.out.println("串行流处理时间：" + (end - start) + "ms");

// 并行流处理
start = System.currentTimeMillis();
int sum2 = list.parallelStream().mapToInt(Integer::intValue).sum();
end = System.currentTimeMillis();
System.out.println("并行流处理时间：" + (end - start) + "ms");

System.out.println("串行流结果：" + sum1);
System.out.println("并行流结果：" + sum2);
```

输出结果：

```
串行流处理时间：2ms
并行流处理时间：1ms
串行流结果：55
并行流结果：55
```

总结
--

Java 8 Streams 是一个非常强大的功能，它提供了一种简洁、优雅的方式来处理数据集合。本教程介绍了 Streams 的基本概念，以及如何在 Java 8 中使用 Streams。同时，本教程也包含了许多代码示例，以帮助读者更好地理解和应用 Streams。

在使用 Streams 时，需要注意以下几点：

1.  尽量避免在 Stream 中执行过多的计算，因为这会影响性能。
    
2.  在使用并行流处理时，要注意线程安全问题。
    
3.  不要试图在同一个 Stream 对象上执行多次终止操作，因为这会导致 IllegalStateException 异常。
    
4.  在使用 collect 操作时，可以使用 Collectors 工具类提供的方法，例如 toList、toSet 等，以方便地将元素转换成其他形式。
    

总的来说，Java 8 Streams 是一个非常强大、灵活的功能，它可以帮助我们更加高效地处理数据集合。如果你还没有尝试过 Streams，希望本教程能够帮助你入门，并掌握其基本用法。