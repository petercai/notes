# Java核心技术第二卷》第一章流笔记
> Java的Stream API和集合（Collection）都可以用于操作数据，但它们有一些关键的区别，使得在某些情况下，使用Stream会比使用集合更有优势：
> 
> *   并行处理：Stream API支持并行处理，这意味着你可以利用多核处理器的优势来处理大量数据。这对于处理大数据集时可以大大提高性能。
> *   函数式编程：Stream API是函数式编程的一部分，它允许你使用声明性代码来处理数据。这可以使你的代码更简洁，更易读，更易于维护。
> *   惰性执行：Stream API的许多操作是惰性的，这意味着它们只有在需要结果时才会执行。这可以帮助你优化性能，因为你可以避免对数据进行不必要的处理。
> *   链式操作：Stream API支持链式操作，这意味着你可以将多个操作链接在一起，形成一个操作管道。这可以使你的代码更简洁，更易读。
> *   无副作用：Stream操作不会修改其源，因此它们没有副作用。这使得你的代码更易于理解和测试。
> *   优化的特定操作：Stream API提供了一些优化的操作，如reduce，collect，flatMap等，这些在集合API中不可用或使用起来不方便

1.1 从迭代到流
---------

迭代方式进行偶数统计

```java
public static void 迭代遍历() {
    int count = 0;
    for(var it:list){
        if(it % 2 == 0){
            count++;
        }
    }
    System.out.println(count);
}

```

等价于流的如下写法

```java
public static void 流遍历() {
    long count = list.stream().filter(it -> it % 2 == 0).count();
    System.out.println(count);
}

```

流还有并行的处理方式

```java
public static void 并行流遍历2() {
    long count = list.parallelStream().filter(it -> it % 2 == 0).count();
    System.out.println(count);
}

```

流遵循了`做什么而非怎么做`。我们并没有指定它的执行顺序或者在哪个线程中执行。

流和集合表面看起来很像，但是她们存在显著的差距。

1.  `流并不存储元素`。元素可能储存在底层的集合中，或者按需生成。
2.  `流的操作不会修改数据源。`例如filter不会从流移除元素，而是会生成一个新的流。
3.  `流的操作是尽可能惰性执行的。`这意味这直至需要其结果是，操作才会执行。例如我们只想查找前五个长单词，那么filter在第五个单词之后就会停止过滤。这意味着我们可以操纵无限流

在上述流的示例中。我们建立一个包含三个阶段的操作管道。

1.  创建一个流
2.  指定将初始流转换为其它流的中间操作，可能有多个步骤
3.  应用终止操作，从而产生结果。这个操作会强制执行之前的惰性操作，之后流将不可用

1.2 流的创建
--------

以下是流的一些创建方法

```java
private static void 创建流(){
    var list = List.of(1, 2, 3, 4, 5);
    
    var stream = Stream.of(1, 2, 3, 4, 5);
    
    var stream1 = Stream.of(new int[]{1, 2, 3, 4, 5});
    
    var stream3 = list.stream();
    var stream4 = list.parallelStream();
    
    var stream5 = Stream.generate(Math::random);
}

```

如果想产生0,2,4,6,8……这样的集合，可以使用`iterate`方法

```java
private static void iterate创建等差数列(){
    
    Stream.iterate(0, n ->n.compareTo(10) < 0,n -> n + 2)
        .forEach(System.out::println);
}

```

第二个参数一旦拒绝了某次迭代产生的值，这个流就结束。

`Stream.ofNullable`方法会用一个对象创建一个非常短的流。

若该对象为`null`,则流的长度为0。

否则流的长度为1,其中仅包含该对象。

```java
private static void ofNullable方法(){
    Object obj = new Object();
    
    Stream<Object> stream1 = Stream.ofNullable(obj);
    stream1.forEach(System.out::println);
    
    obj = null;
    Stream<Object> stream2 = Stream.ofNullable(obj);
    stream2.forEach(System.out::println);
}

```

打印结果为

```java
java.lang.Object@7ba4f24f

```

仅仅打印了一次内容。

`toList`方法可以让其最终结果转为List

```java
private static void toList方法(){
    
    Stream<Object> stream = Stream.of(1,2,3,4,5);
    List<Object> list = stream.toList();
    System.out.println(list);
}

```

1.3 filter、map和flatMap方法
------------------------

`filter`的引元是一个`Predicate<T>`类型的对象，即从T映射到boolean值的函数。

我们可以用它将一个流的内容过滤到另一个流

```java
private static void filter方法(){
    var list = List.of(1, 2, 3, 4, 5);
    
    list.stream().filter(it -> it % 2 == 0).forEach(System.out::println);
}

```

`map`方法可以让我们按照某种方式转换流中的值

```java
private static void map方法(){
    var list = List.of(1, 2, 3, 4, 5);
    
    list.stream().map(it -> it * 2).forEach(System.out::println);
}

```

`flatMap`可以让其摊平多个流。假设我们手中有一个有多个列表的列表

那么我们想要将其转换为一个流，便可以使用`flatMap`

```java
private static void flatMap方法(){
    var list = List.of(List.of(1, 2), List.of(3, 4), List.of(5, 6));
    
    list.stream().flatMap(List::stream).forEach(System.out::println);
}

```

假设我们有这样一个映射函数，它返回的是一个任意的结果或多个结果。

考虑如下示例，`codePoints`方法会产生一个字符串中的所有`编码点`。

例如`codePoints("Hello 🌏")`返回流由每个字符构成，但是🌏由两个char值构成。

所以我们要采用不同的方式处理它们。

让方法返回`Stream<String>`对象

```java
private static Stream<String> codePoints示例1(String str){
    return str.codePoints().mapToObj(it -> new String(new int[]{it}, 0, 1));
}

```

首先我们使用`codePoints()`获取由整数编码点构成的流，然后让他们根据数组中的元素转换为字符串。

当使用`flatMap`时候，需要提供一个方法，它会为每一个流元素产生一个新的流。

这会显得很冗长，而且效率低下。

`mapMulti`方法提供了另一种选择。

`mapMulti`方法接受一个`BiConsumer`，这个`BiConsumer`接受两个参数：流中的`当前元素`和一个`Consumer`。你可以使用这个Consumer来提交你想要添加到结果流中的元素。

`flatMap`方法中的例子修改后如下。

```java
private static void mapMulti方法() {
    var list = List.of(List.of(1, 2), List.of(3, 4), List.of(5, 6));
    
    list.stream()
        .mapMulti((it, consumer) -> it.forEach(e -> consumer.accept(e * 2)))
        .forEach(System.out::println);
}

```

1.4 抽取子流和组合流
------------

调用`stream.limit(n)`会返回一个新的流，它在n个元素后结束，这对于裁剪无限流很有用。

```java
private static void limit方法() {
    Stream.generate(Math::random)
        .limit(100)
        .forEach(s -> System.out.print(String.format("%.2f ", s) + " "));
}

```

`skip(n)`方法正好相反，它会跳过前n个元素。

```java
private static void skip方法() {
    var list = List.of(2,3,4,5,6);
    list.stream().skip(2).forEach(it -> System.out.print(it + " "));
}

```

`takeWhile`会获取第一个谓词为真的元素及之后的元素组成的流。

```java
private static void takeWhile方法(){
    var list = List.of(2,3,4,5,6);
    list.stream().takeWhile(it -> it < 5).forEach(it -> System.out.print(it + " "));
    System.out.println();
}

```

打印内容为

> `2 3 4`

`dropWhile`会获取第一个谓词为假的元素及之后的元素组成的流。

```java
private static void dropWhile方法(){
    var list = List.of(2,3,4,5,6);
    list.stream().dropWhile(it -> it > 5).forEach(it -> System.out.print(it + " "));
    System.out.println();
}

```

打印内容为

> `2 3 4 5 6`

`concat`可以组合两个流

```java
private static void concat方法(){
    var list = List.of(2,3,4,5,6);
    Stream<Integer> stream1 = list.stream().dropWhile(it -> it > 5);
    Stream<Integer> stream2 = list.stream().takeWhile(it -> it < 5);
    Stream.concat(stream1, stream2).forEach(it -> System.out.print(it + " "));
    System.out.println();
}

```

打印内容为

> `2 3 4 5 6 2 3 4`

1.5 其他的流转换
----------

`distinct`会返回一个去除重复元素的流

```java
private static void distinct方法(){
    
    var list = List.of(1, 2, 3, 4, 5, 1, 2, 3, 4, 5);
    list.stream().distinct().forEach(it -> System.out.print(it + " "));
    System.out.println();
}


```

输出结果

> `1 2 3 4 5`

对于流的排序，有多种`sorted`方法的变体可用。

其中一种用于操作`Compartable`元素的流，而另一种可以接受`Comparator`

```java
private static void sorted方法(){
    var list = List.of("233","123","456","78239","123","Wwh","Wwh","Tom","LiHUa","LOLOLOLO");
    list.stream().sorted(Comparator.comparingInt(String::length)).forEach(it -> System.out.print(it + " "));
    System.out.println();
    list.stream().sorted(String::compareTo).forEach(it -> System.out.print(it + " "));
    System.out.println();
}

```

输出结果为

> `233 123 456 123 Wwh Wwh Tom 78239 LiHUa LOLOLOLO 123 123 233 456 78239 LOLOLOLO LiHUa Tom Wwh Wwh`

`peek`会产生另一个流，它的元素与原来的流中的元素相同，但是每一次获取一个元素时，都会调用一个函数，方便调试

```java
private static void peek方法(){
    
    Stream.iterate(1, it -> it + 1)
        .peek(it -> System.out.print(it + " "))
        .limit(10)
        .count();
    System.out.println();
}

```

由于peek是惰性方法，所以方法最后需要添加终端操作否则它不会执行。

输出结果为

> 1 2 3 4 5 6 7 8 9 10

1.6 简单约简
--------

约简(reduction)是一种终结操作，它们会将流约简为可以在程序中使用的非流值。

上面我们可以见到其中一种约简，`count`。

其他简单约简还有`max`和`min`。

这两种方式的返回值是`Optional<T>`，它要么在其中包装了答案，要么表示没有任何值。

以下是这两种方式的示例

```java
private static void max和min(){
    var list = List.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9);
    Optional<Integer> max = list.stream().max(Integer::compareTo);
    System.out.println(max.orElse(0));
    Optional<Integer> min = list.stream().min(Integer::compareTo);
    System.out.println(min.orElse(0));
}

```

打印结果为：

> 9
> 
> 0

`findFirst`返回的是非空集合中的第一个值。如果想找到第一个符合条件的值，可以使用`filter`和`findFirst`搭配使用

```java
private static void findFirst方法(){
    var list = List.of("wwh", "Tom", "LiHua", "Wwh", "Wwh", "Tom", "Tom", "Tom","MC");
    Optional<String> first = list.stream().filter(it -> it.length() > 3).findFirst();
    System.out.println(first.orElse("Not Found"));
}

```

打印结果为：

> LiHua

如果不强调第一个匹配，而是使用任意匹配都可以，那么可以使用`findAny`方法。

这个方法在并行流处理时很有，因为流可以报告任何它找到的匹配而不是被限制为必须报告第一个匹配。

```java
private static void findAny方法(){
    var list = List.of("wwh", "Tom", "LiHua", "Wwh", "Wwh", "Tom", "Tom", "Tom","MC");
    Optional<String> any = list.stream().parallel().filter(it -> it.length() < 3).findAny();
    System.out.println(any.orElse("Not Found"));
}

```

如果想知道是否匹配可以使用`anyMatch`,这个方法会接受一个谓词引元，因此不需要使用`filter`

```java
private static void anyMatch方法(){
    var array = Stream.generate(() -> random.nextInt(100))
        .limit(100000000).toArray(Integer[]::new);
    LocalDateTime start = LocalDateTime.now();
    boolean b = Arrays.stream(array).anyMatch(it -> it.equals(99999999));
    LocalDateTime end = LocalDateTime.now();
    System.out.println("99999999 " + (b ? "存在于数组中" : "不存在于数组中") +
                       " 耗时：" + ChronoUnit.MILLIS.between(start, end) + "ms");
    start = LocalDateTime.now();
    b = Arrays.stream(array).parallel().anyMatch(it -> it.equals(99999999));
    end = LocalDateTime.now();
    System.out.println("99999999 " + (b ? "存在于数组中" : "不存在于数组中") +
                       " 耗时：" + ChronoUnit.MILLIS.between(start, end) + "ms");
}

```

`anyMatch`配合并行流大大提高了运行的速度，同样的要求，并行流可以达到三倍以上的速度。

> 99999999 不存在于数组中 耗时：289ms 99999999 不存在于数组中 耗时：56ms

相似的还有`allMatch`和`noneMatch`，它们分别在所有元素和没有元素匹配谓词的情况下返回`true`

1.7 Optional类型
--------------

`Optional<T>`对象是一种包装器对象，要么包装了类型`T`的对象，要么没有包装任何对象。

第一种情况下，我们称值是存在的。

`Optional<T>`类型被当作一种更安全的方式来替代`T`的引用，这种引用要么引用某个对象，要么为`null`。

### 1.7.1 获取Optional值

有效地使用`Optional`关键字是要使用这样的方法：它的值不存在的情况下会产生一个可替代物，只有当值存在时才会使用这个值。

通常我们希望在没有任何匹配时，我们能用某种默认值来替代。

```java
var optional = Optional.empty(); 
String result = (String) optional.orElse("Tom"); 

```

或者我们也可以调用代码计算默认值

```java
result = (String) optional.orElseGet(() -> "Wwh");

```

或者可以在没有值的时候抛出异常

```java
try {
    result = (String) optional.orElseThrow(() -> new IllegalArgumentException("值为空"));
} catch (IllegalArgumentException e) {
    System.out.println(e.getMessage());
}

```

### 1.7.2 消费Optional值

上一节中，我们看到了在不存在任何值的情况下产生相应的替代物。另一条使用可选值的策略是只有在其存在的情况下才消费该值。可以使用`ifPresent`方法

```java
var optional = Optional.of(666);
optional.ifPresent(it -> System.out.println(it * 2 % 233));

```

如果想要在可选值存在时执行一种操作，然后在值不存在时执行另一种操作，可以使用`ifPresentOrElse`方法

```java
optional.ifPresentOrElse(
    it -> System.out.println(it * 2 % 233), 
    () -> System.out.println("值为空")
);

```

### 1.7.3 管道化Optional值

在编程中，管道化通常指的是将多个操作链接在一起，形成一个操作序列，其中每个操作的输出都是下一个操作的输入。这种方式可以使代码更简洁，更易读，也更易于维护。

上一节中我们知道了如何从`Optional`对象中获取值。另一种策略是保持`Optional`完整，使用`map`方法转换`Optional`内部的值。

```java
var optional = Optional.of(666);
var result = optional.map(it -> it * 2 % 233); 

optional = Optional.empty();
result = optional.map(it -> it * 2 % 233); 

```

相似的，可以使用`filter`方法来处理那些在转换它之前或之后满足某种特定属性的`Optional`值。

注意，`Optional`中的`filter`方法与`Stream`中类似，但是它的尺寸只有1或0

```java
var opt = Optional.of("666www");
var res = opt.filter(it -> it.length() > 10); 

```

也可以用`or`方法将空的`Optional`对象替换为一个可替代的`Optional`。

这个可替代值将以惰性计算。

```java
opt = Optional.of("666www"); 

```

### 1.7.4 Optional类型正确用法的提示

*   Optional类型的变量应该永远不为null
*   不要在集合和映射中使用Optional对象

### 1.7.5 创建Optional的值

```java
var optional = Optional.of("Wwh"); 

optional = Optional.ofNullable(null); 

optional = Optional.empty(); 

```

### 1.7.6 用flatMap构建Optional值的函数

假设你有一个可以产生`Optional<T>`对象的方法f，并且目标类型`T`具有一个可以产生`Optional<U>`的对象的方法`g`。

如果它们都是普通的方法，那么可以调用`s.f().g()`将它们组合起来，但这种组合无法工作，因为`s.f()`类型为`Optional<T>`，而不是`T`。

因此需要调用

> Optional result = s.f().flatMap(T::g)

下面是个将双精度浮点数平方的例子

```java
public static void main(String[] args) {
    Optional<Double> result = f(4.0).flatMap(x -> Optional.of(x * x));
    System.out.println(result);
}

private static Optional<Double> f(Double t){
    return Optional.of(t);
}

```

如果有更多的方法，都可以用`flatMap`连接起来，进而构建由这些步骤构建的管道，只有所有步骤都成功，该管道才会成功。

### 1.7.7 将Optional转换为流

`stream`方法会将`Optional<T>`对象转换为一个具有0个或1个元素的`Stream<T>`对象

这会使返回`Optional`结果的方法变得很有用。

假设我们有一个用户ID流和如下方法

```java
private static Optional<User> lookup(String id){
    return Optional.of(new User(id));
}

```

我们可以使用`filter`过滤无效`id`

例如下面这种方式

```java
Stream<User> users = list.stream().map(将Optional转换为流::lookup)
    .filter(Optional::isPresent)
    .map(Optional::get);

```

不过这种方式使用了不建议使用的`isPresent`和`get`方法。

我们可以使用下面的调用方式

```java
Stream<User> userStream = list.stream().map(将Optional转换为流::lookup)
                .flatMap(Optional::stream);

```

1.8 收集结果
--------

处理完流之后，可以用下列方式查看结果

`iterator`方法可以获得迭代器。

```java
private static void iterator方法(){
    Iterator<Integer> iterator = stream.iterator();
    while (iterator.hasNext()){
        System.out.print(iterator.next() + " ");
    }
    System.out.println();
}

```

可以使用`forEach`将某个函数应用于每个元素

```java
private static void forEach方法(){
    Stream<Integer> stream = list.stream();
    stream.forEach(it -> System.out.print(it * 2 + " "));
    System.out.println();
}

```

在并行流上，`forEach`会以任意顺序遍历各个元素。

如果想要按照流中顺序来处理，可以使用`forEachOrdered`方法。

不过这个方法会丧失并行处理的部分甚至全部优势。

`toArray`方法可以获得由流的元素构成的数组。

```java
private static void toArray方法(){
    Stream<Integer> stream = list.stream();
    Object[] objects = stream.toArray();
    for (Object object : objects) {
        System.out.print(object + " ");
    }
    System.out.println();
}

```

将流中元素收集到另一个目标中，可以使用`collect`方法，它会接收一个`Collector`接口的实例。

收集器是一种收集众多元素并产生单一结果的对象。

不过在`Java 16`中，它新增了`toList`等方法，可以直接转为相应对象。

但有时我们需要控制获得集的种类，那么可以使用下面的调用。

```java
TreeSet<Integer> result1 = stream.collect(Collectors.toCollection(TreeSet::new)); 

```

也可以将流转换为字符串

```java
String result2 = stream
    .map(String::valueOf)
    .collect(Collectors.joining(", ")); 

```

还可以将流约简为总和、数量、平均值等

通过调用result3中的方法获取总和、数量、平均值，最大值，最小值

```java
IntSummaryStatistics result3 = stream
    .collect(Collectors.summarizingInt(Integer::intValue));

```

1.9 收集到映射表中
-----------

假设我们有一个`Stream<Person>`

```java
class Person{
    private Long id;
    private String name;
    public Person(Long id, String name) {
        this.id = id;
        this.name = name;
    }
    public Long id() {
        return id;
    }
    public String name() {
        return name;
    }
}

```

我们想将其转换为`Map`类型，使得id和名对应

可以使用如下方式

```java
private static void Person流转映射表(){
    Stream<Person> personStream = Stream.of(new Person(1L, "张三"), new Person(2L, "李四"));
    Map<Long,String> collect = personStream.collect(Collectors.toMap(Person::id, Person::name));
}

```

如果需要值直接映射到原本的对象，可以采用如下方式

```java
private static void Person流转映射表_重复键(){
    Stream<Person> personStream = Stream.of(new Person(1L, "张三"), new Person(1L, "李四"));
    Map<Long,Person> collect = personStream
        .collect(Collectors.toMap(Person::id, Function.identity(),(oldValue, newValue) -> oldValue));
}

```

其中`Function.identity()`被用作值生成器，意味着映射表的值将是原始的`Person`对象，没有任何变化。

如果有多个元素具有相同的键，就会存在冲突，收集器将会抛出`IllegalStateException`异常。

我们可以提供第三个函数引元来覆盖这种行为。

该函数会根据已有的值和新值来解决冲突并确定键对应的值。

假设我们想要了解每个国家中的所有语言，我们需要一个`Map<String,Set<String>>`。

例如巴西有\[西班牙语, 葡萄牙语\]。

我们首先需要为每种语言存储一个单例集，当我们找到给定国家的新语言时，将其添加进集合。

```java
private static void 语言流转映射表_重复键_合并() {
    Stream<Locale> availableLocales = Stream.of(Locale.getAvailableLocales());
    Map<String, Set<String>> collect = availableLocales.collect(
        Collectors.toMap(
            Locale::getDisplayCountry,
            l -> Set.of(l.getDisplayLanguage()),
            (oldValue, newValue) -> {
                Set<String> union = new HashSet<>(oldValue);
                union.addAll(newValue);
                return union;
            }
        ));
    System.out.println(collect);
}

```

若我们想获取`TreeMap`，那么构造器需要第四个引元

```java
private static void 获取TreeMap(){
    Stream<Person> personStream = Stream.of(new Person(1L, "张三"), new Person(2L, "李四"));
    TreeMap<Long, Person> collect = personStream
        .collect(Collectors.toMap(Person::id, Function.identity(), (oldValue, newValue) -> oldValue, TreeMap::new));
}

```

1.10 群组和分区
----------

上一节中，我们收集了每个国家中的所有语言，但是处理有些冗长。

将具有相同特性的值群聚成组是非常常见的，我们可以使用`gropingBy`方法来支持它。

```java
private static void 语言流转映射表_重复键_合并_使用groupingBy() {
    Stream<Locale> availableLocales = Stream.of(Locale.getAvailableLocales());
    Map<String, Set<String>> collect = availableLocales.collect(
        Collectors.groupingBy(
            Locale::getDisplayCountry,
            Collectors.mapping(Locale::getDisplayLanguage, Collectors.toSet())
        ));
    System.out.println(collect);
}

```

当分类函数是一个`返回boolean值的函数`时，这种情况下`partitioningBy`更为高效。

```java
private static void 语言流转映射表_重复键_合并_使用partitioningBy() {
    Stream<Locale> availableLocales = Stream.of(Locale.getAvailableLocales());
    Map<Boolean, List<Locale>> collect = availableLocales.collect(
        Collectors.partitioningBy(l -> l.getLanguage().equals("zh")));
    System.out.println(collect.get(true));
}

```

它会返回所有地区语言编码等于`zh`的语言

1.11 下游收集器
----------

`groupingBy`方法会产生一个映射表，它的每个值都是一个列表。

如果想要处理列表，则需要提供一个`下游收集器`。

例如想获得集而不是列表，应该这么写

```java
private static void 语言流转映射表_重复键_合并_使用groupingBy() {
    Stream<Locale> availableLocales = Stream.of(Locale.getAvailableLocales());
    Map<String, Set<Locale>> collect = availableLocales.collect(
        Collectors.groupingBy(
            Locale::getCountry,
            toSet()
        ));
    System.out.println(collect);
}

```

Java提供了多种可以将收集到的元素约简为数字的收集器

*   `counting`会产生收集到的元素的个数
    
    ```java
    private static void 语言流转映射表_统计元素个数(){
        Stream<Locale> availableLocales = Stream.of(Locale.getAvailableLocales());
        Map<String, Long> collect = availableLocales.collect(
            Collectors.groupingBy(
                Locale::getCountry,
                Collectors.counting()
            ));
        System.out.println(collect);
    }
    
    ```
    
    可以对每个国家有多少locale进行计数
    
*   `summing(Int|Long|Double)`和`averging(Int|Long|Double)`，会接受一个函数作为引元，将该函数应用到下游元素上。
    
    ```java
    private static void 语言流转映射表_统计总和和平均() {
        Stream<Locale> availableLocales = Stream.of(Locale.getAvailableLocales());
        Map<String,Integer> collect = availableLocales.collect(
            Collectors.groupingBy(
                Locale::getCountry,
                summingInt(l -> l.getLanguage().length())
            ));
        System.out.println(collect);
        availableLocales = Stream.of(Locale.getAvailableLocales());
        Map<String, Double> collect1 = availableLocales.collect(
            Collectors.groupingBy(
                Locale::getCountry,
                Collectors.averagingInt(l -> l.getLanguage().length())
            ));
        System.out.println(collect1);
    }
    
    ```
    
*   `maxBy`和`minBy`接受一个比较器，分别产生下游元素中的最大值和最小值
    
    ```java
    private static void maxBy和minBy方法(){
        Stream<Student> studentStream = Stream.of(new Student("北京", "张三"), new Student("上海", "李四"));
        Map<String, Optional<Student>> collect = studentStream.
            collect(groupingBy(Student::getCity, maxBy(Comparator.comparing(Student::getName))));
        System.out.println(collect);
        studentStream = Stream.of(new Student("北京", "张三"), new Student("上海", "李四"));
        collect = studentStream.
            collect(groupingBy(Student::getCity, minBy(Comparator.comparing(Student::getName))));
        System.out.println(collect.get("北京").get().getName());
    }
    
    ```
    
*   `CollectingAndThen`收集器会在收集器后面添加一个最终步骤。例如我们想知道有多少个不同的结果，可以收集到集里再计算集的尺寸
    
    ```java
    private static void collectionAndThen方法(){
        var list = Stream.generate(Math::random).limit(30).toList();
        Map<Character, Integer> collect = list.stream().collect(groupingBy(
            s -> String.valueOf(s + 0.5).charAt(0),
            collectingAndThen(toSet(), Set::size)
        ));
        System.out.println(collect);
    }
    
    ```
    
*   `mapping`收集器做法正好相反，它会将一个函数应用于收集到的每个元素，并将结果传到下游收集器
    
    ```java
    private static void mapping方法(){
        var list = Stream.generate(Math::random).limit(30).toList();
        Map<Character, Set<Integer>> collect = list.stream().collect(
            groupingBy(
                s -> String.valueOf(s + 0.5).charAt(0),
                mapping(s -> String.valueOf(s + 0.5).length(), toSet())
            )
        );
        System.out.println(collect);
    }
    
    ```
    
*   `flatMapping`方法，可以将流摊平，并添加到指定集合中
    
    ```java
    private static void flatMapping方法(){
        List<String> words = List.of("Hello", "World");
        Map<Integer, Set<Character>> result = words.stream()
            .collect(Collectors.groupingBy(
                String::length,
                Collectors.flatMapping(
                    word -> word.chars().mapToObj(c -> (char) c),
                    Collectors.toSet()
                )
            ));
    }
    
    ```
    
*   `summarizingInt`等方法可以获取函数的总和，平均值，最大最小值，数量
    
    ```java
    private static void summarizing方法(){
        var list = Stream.generate(Math::random).limit(30).toList();
        Map<Character, DoubleSummaryStatistics> collect = list.stream().collect(groupingBy(
            s -> String.valueOf(s + 0.3).charAt(0),
            summarizingDouble(Double::valueOf)
        ));
        System.out.println(collect);
    }
    
    ```
    
*   `filtering`收集器会将过滤器应用到每个组上
    
    ```java
    private static void filtering方法(){
        var list = Stream.generate(Math::random).limit(30).toList();
        Map<Character, DoubleSummaryStatistics> collect = list.stream().collect(groupingBy(
            s -> String.valueOf(s + 0.3).charAt(0),
            filtering(s -> s>0.5,summarizingDouble(Double::valueOf))
        ));
        System.out.println(collect);
    }
    
    ```
    

1.12 约简操作
---------

`reduce方法`是一种用于从流中计算某个值的通用机制，最简单的形式是接受一个二元函数，并从前两个元素持续应用。

```java
private static void 简单reduce方法(){
    List<Integer> list = Stream.generate(() -> random.nextInt(15)).limit(30).toList();
    Optional<Integer> reduce = list.stream().reduce(Integer::sum);
    reduce.ifPresent(System.out::println);
}

```

如果流为空，会返回一个`Optional`，因为没有任何有效的结果。

如果要用并行流来约简，那么这项约简操作必须是可结合的，即组合元素时使用的顺序不会产生任何影响。

即(xopy)opz\=xop(yopz)(x~op~y)~op~z = x~op~(y~op~z)

通常我们会有一个幺元值e使得eopx\=xe~op~ x = x，作为元素的起点。

```java
private static void 指定幺元值reduce方法(){
    List<Integer> list = Stream.generate(() -> random.nextInt(15)).limit(30).toList();
    Integer result = list.stream().reduce(0,Integer::sum);
    System.out.println(result);
}

```

这种情况会返回一个值，这样我们就可以不用使用`Optional`类了。

当有时你需要对某些属性求和，例如字符串流中的所有字符串的长度，那么就不能简单地使用`reduce`,还需要提供一个函数来处理结果

```java
Long result = limit.reduce(0L,Long::sum,Long::sum);

```

这种情况下为`reduce`提供第三个参数可以合并它们的结果。

1.13 基本类型流
----------

目前为止所有流中的元素都是对象，但是将每个元素都包装到对象中是非常低效的。

有三种流可以直接存储基本类型

`IntStream`存储`short`、`char`、`byte`、`boolean`

`DoubleStream`存储`float`

有如下两种方式创建流

```java
private static void 基本类型流(){
    IntStream intStream = IntStream.of(1,2,3,4,5);
    intStream = Arrays.stream(new int[]{1,2,3,4,5});
}

```

与对象流相同,可以使用`generate`和`iterate`

`InStream`有`range`和`rangeClosed`方法可以生成步长为1的整数范围

```java
private static void range和rangeClosed方法(){
    IntStream range = IntStream.range(0, 100); 
    range.forEach(System.out::println);
    range = IntStream.rangeClosed(0,100); 
    range.forEach(System.out::println);
}

```

当有一个基本流时，可以使用`box`将其包装为对象流

```java
private static void boxed方法(){
    IntStream range = IntStream.rangeClosed(1,100);
    Stream<Integer> boxed = range.boxed();
}

```

可以用`mapToInt`方法再将对象流转为基本流

```java
range = boxed.mapToInt(Integer::intValue);

```

1.14 并行流
--------

流使得并行处理块操作变得很容易。

可以使用`parallelStream()`获取并行流

```java
Stream stream = l.parallelStream();

```

或者使用`parallel`方法将任意流转换为并行流

```java
stream = (Stream) stream.parallel();

```

只要终结方法执行时流处于并行状态，那么所有中间操作就都将被并行化。

当流并行操作执行时，目标是让其返回结果和顺序流执行返回的结果相同。

`重要的是`,这些操作是无状态的，并且可以以任意顺序执行。

```java
private static void 错误的并行流(){
    int[] value = new int[10];
    var list = Stream.generate(() -> random.nextInt(10)).limit(5000).toList();
    list.parallelStream().forEach(it -> {
        value[it]++;
    });
    System.out.println(Arrays.stream(value).sum()); 
}

```

在上述代码中，`forEach`在多个并发线程中运行，每个都会更新共享的数组，这是很经典的竞争情况。

它最终输出的结果每次都不一样。

最佳的解决方式是远离可修改状态

可以使用`filter`先过滤我们所需的元素，然后使用`groupingby`将其分组，最后进行统计。

```java
private static void 正确的并行流(){
    int[] value = new int[10];
    int sum = 0;
    var list = Stream.generate(() -> random.nextInt(10)).limit(5000).toList();
    Map<Integer, Long> result = list.parallelStream().collect(Collectors.groupingBy(it -> it, Collectors.counting()));
    long collect = result.values().stream().mapToLong(Long::valueOf).sum();
    System.out.println(collect); 
}

```

默认情况下，从有序数组和集合中产生的流。它们的结果是按照原来的元素顺序累积的，如果相同的操作进行两次，将会得到完全相同的结果。

```java
private static void 有序流(){
    IntStream intStream = IntStream.rangeClosed(0, 10000);
    int sum = intStream.parallel().sum();
    System.out.println(sum); 
}

```

排序并不排斥高效的并行处理。

当放弃排序操作时，有些操作可以被更有效地并行化。

通过在流上调用`Stream.unordered`方法就可以表明我们对排序不感兴趣。

例如我们有时想去除顺序流中的重复元素，`distinct`会保留所有相同元素的第一个。

这对并行化是一种阻碍。因为每个处理部分的线程在其之前的所有部分处理完之前，并不知道该丢弃哪些元素。

```java
private static void 无序加速并行处理(){
    var list = Stream.generate(() -> random.nextInt(50)).limit(1000000).sorted().toList();
    long start = System.nanoTime();
    Stream<Integer> distinct = list.parallelStream().unordered().distinct();
    long end = System.nanoTime();
    System.out.println("耗时：" + (end - start)/1000 + " 毫秒");
}

```

这种情况下，并行流比顺序流更快。

合并映射表的代价很高，但我们可以使用`Collectors.groupingByCouncurrent`方法使用了共享的并发映射表。

为了从并行化中获益，映射表中值的顺序不会与流的顺序相同。

不是所有流转换为并行流都会加速操作，要牢记以下几条：

*   并行化会导致大量开销，只有面对很大的数量集才划算
*   在底层的数据源可以被有效地分割为多个部分时，将流并行化才有意义
*   并行流使用的线程池可能会因诸如文件I/O或网络访问这样的操作被阻塞而饿死