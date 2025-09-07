# JDK8到JDK21的演变史
前言
--

​ 事物总是在不断地演进、变化中得以生存、升华或者消亡的。不是在沉默中爆发就是在沉默中死去，java在前些天发布了最新版的JDK21。这次版本可谓万众瞩目，也预示着它在不断地升华后必将是更加辉煌的未来。它没有被时间的滚滚长河所吞没，反而顺势而为，努力调整自己，吸取百家所长，不断完善。使它老树生花，迸发出更加勃勃的生机【ok，跩文到此为止】。

​ 由于springboot3和spring6都需要最低版本要求是JDK17这个客观现实摆在面前。倒逼着我们不得不跟进JDK的变化，因为spring官方已经发文，springboot2.x支持到11月份就结束了。意味着2.7.x就是spring2的绝唱版本了。如果有bug，到那时再想着升级可能就有点晚了。未雨绸缪，本文总结JDK8到JDK21重要新特性，不但有语法特性还包括JVM的演化过程、工作原理等相关介绍。

​ 每个版本的新特性在openJDK官网上都有，比如jdk17的新特性：[openjdk.org/projects/jd…](https://link.juejin.cn/?target=https%3A%2F%2Fopenjdk.org%2Fprojects%2Fjdk%2F17%2F%25EF%25BC%258C%25E4%25BB%258B%25E7%25BB%258D%25E7%259A%2584%25E6%2597%25B6%25E5%2580%2599%25E6%2596%25B0%25E7%2589%25B9%25E6%2580%25A7%25E5%25B0%25BD%25E9%2587%258F%25E4%25BB%258E%25E5%25AF%25B9%25E7%25BC%2596%25E7%25A8%258B%25E6%259C%2589%25E5%25BD%25B1%25E5%2593%258D%25E7%259A%2584%25E8%25A7%2592%25E5%25BA%25A6%25E5%2587%25BA%25E5%258F%2591%25E3%2580%2582 "https://openjdk.org/projects/jdk/17/%EF%BC%8C%E4%BB%8B%E7%BB%8D%E7%9A%84%E6%97%B6%E5%80%99%E6%96%B0%E7%89%B9%E6%80%A7%E5%B0%BD%E9%87%8F%E4%BB%8E%E5%AF%B9%E7%BC%96%E7%A8%8B%E6%9C%89%E5%BD%B1%E5%93%8D%E7%9A%84%E8%A7%92%E5%BA%A6%E5%87%BA%E5%8F%91%E3%80%82")

​ 我这里斗胆预测，JDK21会成为下一个类似JDK8的经典版本。

JDK版本线路图
--------

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/1ad63236647b4fa390779a93365962df~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=F47Pmcqy9mWJTB458nlSW0%2FlXdY%3D)

官网地址：[www.oracle.com/java/techno…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.oracle.com%2Fjava%2Ftechnologies%2Fjava-se-support-roadmap.html "https://www.oracle.com/java/technologies/java-se-support-roadmap.html")

上面的图片也是从官网截图的，从中可以看出LTS(长期支持版本)有：JDK8、JDK11、JDK17、JDK21和未来的JDK25这几个版本。其中JDK21至少有8年的支持，估计后续还会延期的。

**oracle在2021年更改发布Oracle JDK 免费条款和条件 (NFTC) 许可，其可以免费用于生产用途 。此次发布的更新推翻2018年Oracle JDK对生产使用收费的决定，并且不会影响 Oracle OpenJDK 的发行版。修改过的NFTC许可适用于 Oracle JDK 17 版及其以后得版本。可以理解为JDK17以后OpenJDK和Oracle JDK是一样的了**

> 协议改签的原因估计是，oralce对jdk版本的收费，导致java社区强烈不满和混乱，后续调查表明，Oracle发行的JDK版本不再是最受欢迎的java发行版。比如：AdoptOpenJDK、亚马逊jdk、Azul的jdk，甚至国内阿里、腾讯、华为等都有jdk发行版。

java8（LTS）
----------

### java8新特性

#### 1.Lambda 表达式

“λ”是一种数学符号,表示着一个函数或者表达式的参数，在java世界中表现为：**函数作为参数传递进方法中**。

Java8中引入了一个新的操作符“->”该操作符称为箭头操作符或Lambda 操作符。

*   箭头左侧 ：Lambda 表达式参数列表
*   箭头右侧：Lambda 表达式中所需要执行的功能，即Lambda 体

示例：

```java

() ->System.out.println("Hello")

() -> 5  
  

x -> 2 * x  
  

(x, y) -> x – y  
  

(int x, int y) -> x + y  
  

(String s) -> System.out.print(s)

```

#### 2.函数式接口

函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。函数式接口可以被隐式转换为 lambda 表达式。我们也可以自定义函数式接口：通过`@FunctionalInterface` 注解定义：

```java
@FunctionalInterface
public interface HelloService {
    void sayHello(String userName);
}

```

那么就可以使用Lambda表达式来表示该接口的一个实现：

```java
public class HelloServiceTest {
    public static void main(String[] args) {
        HelloService helloService = userName -> System.out.println("Hello " + userName);
        helloService.sayHello("laoma");
    }
}

```

java.util.function 中包含了很多类，用来支持 Java的 函数式编程，该包中常用的有：

```java
1. Consumer<T> 	: 消费型接口    方法：void accep(T t)
2. Supplier<T> 	: 供给型接口    方法: T get();
3. Function<T,R>: 函数型接口    方法: R apply(T t)
4. Predicate<T> : 断言型接口    方法: boolean test（T t）

```

#### 3.方法引用

方法引用通过方法的名字来指向一个方法。方法引用可以使语言的构造更紧凑简洁，减少冗余代码。方法引用使用一对冒号 **::**。

*   \*\*构造器引用：\*\*它的语法是Class::new，或者更一般的Class< T >::new实例如下：
    
    ```java
    final Car car = Car.create( Car::new ); final List< Car > cars = Arrays.asList( car );
    
    ```
    
*   \*\*静态方法引用：\*\*它的语法是Class::static\_method，实例如下：
    
    ```java
    cars.forEach( Car::collide );
    
    ```
    
*   \*\*特定类的任意对象的方法引用：\*\*它的语法是Class::method实例如下：
    
    ```java
    cars.forEach( Car::repair );
    
    ```
    
*   \*\*特定对象的方法引用：\*\*它的语法是instance::method实例如下：
    
    ```java
    final Car police = Car.create( Car::new ); cars.forEach( police::follow );
    
    ```
    

#### 4.接口的默认方法和静态方法

Java 8 中的接口允许定义默认方法。默认方法可以在不破坏已有代码的情况下向接口添加新的方法。

```java
public interface DefaultInterface {
    void sayHello();
    
    default void sayHelloLaoma(){
        System.out.println("hello 老马");
    }
    
    static void sayHelloCuidakai() {
        System.out.println("hello 老崔");
    }
}

public class DefaultInerfaceA implements DefaultInterface{
    @Override
    public void sayHello() {
        System.out.println("Hello A");
    }
}

public class DefaultInterfaceTest {
    public static void main(String[] args) {
        DefaultInerfaceA defaultInerfaceA = new DefaultInerfaceA();
        defaultInerfaceA.sayHello();
        defaultInerfaceA.sayHelloLaoma();
        DefaultInterface.sayHelloCuidakai();
    }
}

```

接口中有三个方法，实现类可以选择不实现sayHelloLaoma。从而实现不破坏已有代码的情况下向接口添加新的方法。默认方法可以被实现类重写，而静态方法不能被重写。默认方法可以通过接口实例和实现类实例调用，而静态方法只能通过接口名称调用。

#### 5.Stream API

JDK8引入了Stream API（流式API），它是一个用于处理集合和数组数据的功能强大的工具。Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

*   集合生成流的方式：
    
    *   集合.stream() - 为集合创建串行流
        
    *   集合.parallelStream() - 为集合创建并行流
        
*   流迭代器：
    
    ```Java
    
    List<String> names = Arrays.asList("老马", "老崔", "小孟", "小猪");
    names.forEach(System.out::println);
    
    
    List<String> collect = names.stream().map(name -> name.substring(0, 1)).distinct().collect(Collectors.toList());
    collect.forEach(System.out::println);
    
    
    long oldCount = names.stream().filter(name -> name.substring(0, 1).equals("老")).count();
    System.out.println("列表中出现\"老\"开头的名字有: " + oldCount);
    
    
    names.stream().limit(2).forEach(System.out::println);
    
    
    Collator collator = Collator.getInstance(Locale.CHINA);
    names.stream().sorted(collator::compare).forEach(System.out::println);
    
    ```
    
*   创建并行流的方式
    
    ```Java
    
    names.stream().parallel();
    
    
    names.parallelStream();
    
    ```
    

#### 6.Optional类

java8引入Optional类的目的是为了尽量避免空指针异常。真正使用 `Optional` 的时候，要尽可能避免以前 `obj != null` 这种判断思维，避免使用 `Optional.isPresent()` 判断，才能真正掌握 `Optional` 的用法。

```java
public static void main(String[] args) {
    String message = "Hello, 小猪";
    
    Optional<String> emptyOptional = Optional.empty();
    
    
    
    Optional<Integer> allowNullOptional = Optional.ofNullable(null);
    
    Optional<String> messageOptional = Optional.ofNullable(message);
    if(messageOptional.isPresent()){
        System.out.println("message 不为null");
    }else {
        System.out.println("message 为null");
    }
    
    String defaultVal = emptyOptional.orElse("老崔");
    System.out.println(defaultVal);

    
    String throwVal = emptyOptional.orElseThrow(RuntimeException::new);
    System.out.println(throwVal);
}

```

#### 7.Date-Time API

是对java.util.Date强有力的补充，解决了 Date 类的大部分痛点：

*   非线程安全
*   时区处理麻烦
*   各种格式化和时间计算繁琐
*   设计有缺陷，Date类同时包含日期和时间；还有一个java.sql.Date容易混淆

```java

LocalDate date = LocalDate.of(2023, 9, 22);
System.out.println(date);


LocalDate today = LocalDate.now();
System.out.println(today);


System.out.println(today.plusDays(2));

```

> 特别说明：
> 
> 虽然比以前好用了很多，但是在作为和数据库映射的实体类时，如果是用LocalDateTime的话，是有可能遇到无法插入、更新等情况的。这个与底层的jdbc有关系，需要JDBC driver支持JDBC 4.2 API。如果你用的是Mybatis Plus，jdbc选择的是ojdbc6的话，那么大概率是会报“无效的列类型”错误的。
> 
> 所以这里还是建议数据库的实体类中的日期时间还是用最原始的java.util.Date做映射。

#### 8.更多讲解和示例【用火狐浏览器】

[note.youdao.com/s/P7I1dCA8](https://link.juejin.cn/?target=https%3A%2F%2Fnote.youdao.com%2Fs%2FP7I1dCA8 "https://note.youdao.com/s/P7I1dCA8")

### java8虚拟机

JVM一个重要的任务就是对无用的对象进行回收。这里就牵扯到两个层面的问题：

*   什么样的对象是可以被回收
*   通过什么样的算法进行回收

#### 1.什么样的对象是可以被回收

目前有两种算法判断对象是否可回收：

*   引用计数器算法
    
    每当对象被引用一次计数器加1，对象失去引用计数器减1，计数器为0是就可以判断对象死亡了。这种算法简单高效，但是对于循环引用或其他复杂情况，需要更多额外的开销，因此Java几乎不使用该算法。
    
*   可达性分析算法
    
    所谓可达性分析是指\*\*，\*\*顺着GCRoots根一直向下搜索，整个搜索的过程就构成了一条“引用链”，只要在引用链上的对象叫做可达，在引用链之外的叫不可达，不可达的对象就可以判断为可回收的对象。 哪些对象可作为GCRoots对象呢？ 包括如下：
    
    *   虚拟机栈帧上本地变量表中的引用对象（方法参数、局部变量、临时变量）
    *   方法区中的静态属性引用类型对象、常量引用对象
    *   本地方法栈中的引用对象（Native方法的引用对象）
    *   Java虚拟机内部的引用对象，如异常对象、系统类加载器等
    *   被同步锁（synchronize）持有的对象
    *   Java虚拟机内部的注册回调、本地缓存等
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/fbf7ecf7d92046a6b2700d5e8b225a72~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=2TYfIuCSCEyXaCh5Ix0%2BquR7T4s%3D)
    

#### 2.通过什么样的算法进行回收

如果说可达性分析算法解决的是对象是否能被回收的问题，那么垃圾回收算法要解决的就是怎样去回收这些“垃圾”对象的问题。垃圾回收算法大致分为三种：**标记-清除算法、标记-复制算法、标记-整理算法**

*   标记-清除算法
    
    顾名思义该算法分两个阶段：首先标记出所有需要被回收的对象，然后对标记的对象进行统一清除，清空对象所占用的内存区域，下图展示了回收前与回收后内存区域的对比：
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/f637d225dabd4c6c98f02869d6938261~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=j6sGsqeVgLReuciltaBE1LxIfA8%3D)
    
    > 标记-清除算法缺点：
    > 
    > 1.效率问题 (如果需要标记的对象太多，效率不高)
    > 
    > 2.空间问题（标记清除后会产生大量不连续的碎片，当有大对象要分配而找不到满足大小的空间时，要触发下一次垃圾收集）
    
*   标记-复制算法
    
    标记-清除算法执行效率与内存碎片的缺点，又出现了一种标记-复制算法。其核心思想是将内存分为大小相同的两个区域：运行区和预留区，所有新对象都在运行区，当运行区内存不够时，将运行区中存货的对象都复制到预留区，然后再清空整个运行区的内存。此时两个区域的角色发生变化。存活对象类似羽毛球一样，被运行区和预留区拍来拍去。下图展示了回收前与回收后内存区域的对比：
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/2e5ea8d3da1a4659bcd682304676ca8d~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=cKVfvUsGnKPrLcfxYCrZTpNKuL8%3D)
    
    > 标记-复制算法缺点：
    > 
    > 1.有一个很明显的缺点就是:造成空间浪费,只有一半的空间是可用的
    
*   标记-整理算法
    
    针对标记-复制算法浪费一半内存的情况，又有一个新的算法应运而生：“标记-整理算法”。标记过程仍然与上面两种算法一样，但后续步骤是让所有存活的对象向一端移动，然后直接清理掉存活对象边界以外的内存。
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/e1ffa47c00e0440995f6970d367dabdc~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=tx6zmTx42xSK7ITxc2IhVTFbicM%3D)
    
    > 标记整理算法解决了内存碎片问题，也不存在空间浪费的问题。但是效率上不如标记-复制算法的。
    > 
    > 比如：当内存中存活对象多，并且都是一些微小对象，而垃圾对象少时，要移动大量的存活对象才能换取少量的内存空间。
    > 
    > 不同的垃圾回收算法都有各自的优缺点，要扬长避短，把他们应用到不同的场景中。
    

#### 3.分代垃圾回收

当前虚拟机的垃圾收集都采用分代收集算法，这个思想是根据对象存活周期的不同将内存分为几块。一般将java堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。 比如在新生代中，每次收集都会有大量对象(近99%)死去，所以可以选择**标记-复制算法**，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择\*\*“标记-清除”或“标记-整理”算法\*\*进行垃圾收集。注意，“标记-清除”或“标记-整理”算法会比复制算法慢10倍以上。

> java8默认新生代采用标记-复制算法；老年代使用标记-整理算法

#### 4.垃圾回收器

如果说垃圾回收算法是方法论的话，那么垃圾回收器就是具体实现了。这里介绍几种在java8时代常见的垃圾回收期，包括出现的G1回收器。

*   Serial（串行）收集器
    
    它是一个单线程垃圾收集器。**既可以用于新生代也可以使用在老年代**。它的 “单线程” 的意义不仅仅意味着它只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作的时候必须暂停其他所有的工作线程（ “Stop The World” ）,直到它收集结束。**Serial收集器新生代采用标记-复制算法，老年代采用标记-整理算法**。gc示意图：
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/847dc9d5428c49848f0589243391a881~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=qtGiiGlJVbQ2vGmaWRSY63ozCV4%3D)
    
    Serial Old收集器是Serial收集器的老年代版本，它同样是一个单线程收集器。要想使用serial作为JDK8的垃圾回收期需要设置参数开启：
    
    ```java
    -XX:+UseSerialGC -XX:+UseSerialOldGC
    
    ```
    
*   Parallel（并行）收集器
    
    Parallel收集器其实就是Serial收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和Serial收集器类似。默认的收集线程数跟cpu核数相同，当然也可以用参数(- XX:ParallelGCThreads)指定收集线程数，但是一般不推荐修改。gc示意图：
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/749789ef56f140548c7d7fb0898a8515~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=YR3Pcmj9ZUpHVBA1Tw%2FqJMvwepQ%3D)
    
    *   Parallel Scavenge收集器（JDK8默认的新生代垃圾收集器）
        
        新生代垃圾收集器。采用标记-复制算法。关注点是吞吐量（高效的利用CPU），所谓吞吐量就是CPU中用于运行用户代码的时间与CPU总消耗时间的比值
        
    *   Parallel Old收集器（JDK8默认的老年代垃圾收集器）
        
        老年代垃圾收集器。采用标记-整理算法。
        
    
    开启参数：
    
    ```java
    -XX:+UseParallelGC -XX:+UseParallelOldGC
    
    ```
    
*   ParNew收集器
    
    ParNew收集器是JVM中的一种新生代垃圾收集器，它的作用是在**新生代进行并行垃圾回收**。 它和Parallel收集器很类似，区别主要在于它可以和CMS收集器配合使用。采用**标记-复制算法**。gc示意图：
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/98b6516688a042949e79c7090cf3bbed~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=yC%2FlgqThSOkEIphGkuhlfOmARls%3D)
    
    开启参数：
    
    ```java
    -XX:+UseParNewGC
    
    ```
    
*   CMS收集器
    
    CMS（Concurrent Mark Sweep）收集器是一种老年代垃圾收集器，它的作用是在老年代进行并发垃圾回收。 采用的是标记-清除算法，通过并发的方式进行垃圾回收，CMS收集器的主要目标是减少垃圾回收的停顿时间，以提高应用程序的响应性能。gc示意图：
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/4b6753935b46473c85391434109ffe34~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=hKzPC35m2ib6GtDicWnkU6a5zHg%3D)
    
    > 步骤说明：
    > 
    > 1.初始标记： 暂停所有的其他线程(STW)，并记录下GC Roots直接能引用的对象
    > 
    > 2.并发标记： 并发标记阶段就是从GC Roots的直接关联对象开始遍历整个对象图的过程， 这个过程耗时较长但是不需要停顿用户线程， 可以与垃圾收集线程一起并发运行。因为用户程序继续运行，可能会有导致已经标记过的对象状态发生改变。（所以会出现漏标或者多标的情况）
    > 
    > 3.重新标记： 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短。主要用到三色标记里的增量更新算法(见下面详解)做重新标记。
    > 
    > 4.并发清理： 开启用户线程，同时GC线程开始对未标记的区域做清扫。这个阶段如果有新增垃圾对象会被标记为黑色不做任何处理
    > 
    > 5.并发重置：重置本次GC过程中的标记数据
    
    CMS核心参数：
    
    ```java
    -XX:+UseConcMarkSweepGC：启用cms
    -XX:ConcGCThreads：并发的GC线程数
    -XX:+UseCMSCompactAtFullCollection：FullGC之后做压缩整理（减少碎片）
    -XX:CMSFullGCsBeforeCompaction：多少次FullGC之后压缩一次，默认是0，代表每次FullGC后都会压缩一 次
    -XX:CMSInitiatingOccupancyFraction: 当老年代使用达到该比例时会触发FullGC（默认是92，这是百分比）
    -XX:+UseCMSInitiatingOccupancyOnly：只使用设定的回收阈值(-XX:CMSInitiatingOccupancyFraction设定的值)，如果不指定，JVM仅在第一次使用设定值，后续则会自动调整
    -XX:+CMSScavengeBeforeRemark：在CMS GC前启动一次Minor GC，目的在于减少老年代对年轻代的引用，降低CMS GC的标记阶段时的开销，一般CMS的GC耗时80%都在标记阶段
    -XX:+CMSParallellnitialMarkEnabled：表示在初始标记的时候多线程执行，缩短STW
    -XX:+CMSParallelRemarkEnabled：在重新标记的时候多线程执行，缩短STW;
    
    ```
    
*   G1收集器
    
    Garbage First(简称G1)收集器是垃圾收集器技术发展史上里程碑式的成果，它摒弃了传统垃圾收集器的严格的内存划分，而是采用局部回收的设计思路和基于Region的内存布局形式。G1的内存划分示意图：
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/fac513600e504bf6a6823b9fd66747a6~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=TbIJTvHufL3m%2FUd0ov8YwmAnkmM%3D)
    
    *   从上面图上可以看到，G1垃圾收集器也是基于分代收集理论设计的，但是它的堆内存的布局与其他垃圾收集器的布局有很明显的区别，G1收集器不再按照固定大小以及固定数量的分代区域划分，而是把JAVA堆划分为2048个大小相等的独立的Region，取值范围为1-32MB，且必须为2的N次幂。一般Region大小等于堆大小除以2048，比如堆大小为4096M，则Region大小为2M，当然也可以用参数"-XX:G1HeapRegionSize"手动指定Region大小，但是推荐默认的计算方式。
    *   虽然G1仍然保留新生代和老年代的概念，但新生代和老年代不再是固定的了，而是一系列区域（不需要连续，逻辑连续即可）的动态集合。由于G1这种基于Region回收的方式，可以预测停顿时间。G1会根据每个Region里面垃圾“价值”的大小，在后台维护一个优先级列表，每次根据用户设定的允许收集停顿的时间（-XX:MaxGCPauseMillis,默认为200毫秒）优先处理价值收益最大的Region。
    *   一个Region可能之前是年轻代，如果Region进行了垃圾回收，之后可能又会变成老年代，也就是说Region的区域功能可能会动态变化。
    *   G1垃圾收集器对于对象什么时候会转移到老年代跟之前讲过的原则一样，唯一不同的是对大对象的处理，G1有专门分配大对象的Region叫Humongous区，而不是让大对象直接进入老年代的Region中。在G1中，大对象的判定规则就是一个大对象超过了一个Region大小的50%，比如按照上面算的，每个Region是2M，只要一个大对象超过了1M，就会被放 入Humongous中，而且一个大对象如果太大，可能会横跨多个Region来存放。
    *   Humongous区专门存放短期巨型对象，不用直接进老年代，可以节约老年代的空间，避免因为老年代空间不够的GC开销。
    *   Full GC的时候除了收集年轻代和老年代之外，也会将Humongous区一并回收。
    
    G1收集器的gc示意图：
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/f233fdfcd394415c984e562ac449e4d0~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=3n%2BEvUhBI5c8fpQxJXg1ahEvl%2BU%3D)
    
    *   初始标记：仅仅只是标记一下GC Roots能直接关联到的对象，并且修改TAMS指针的值，让下一阶段用户线程并发运行时，能够在Region上正确的分配对象。这个阶段需要STW，耗时很短，而且是借用MinorGC（上一轮垃圾回收时触发GC）时候同步完成的。
        
    *   并发标记：从GC Roots 开始对堆中的对象进行可达性分析，递归扫描整个堆里的对象，这个过程耗时较长，但是是与用户线程并发执行的。对象扫描完之后还需要重新处理STAB记录下的在并发时有引用变动的对象。
        
    *   最终标记：这个阶段也需要STW，用于处理并发阶段结束后仍然遗留下来的最后少量的STAB记录。
        
    *   筛选回收：负责更新Region的统计数据，对各个Region的回收价值和成本排序，根据用户期望的停顿时间来执行回收计划，然后把决定回收的Region里的存活对象复制到空的Region，然后清空旧Region的空间。由于涉及到对象的移动，所以这个阶段也是需要STW的。
        
    
    > 可以看出，除了并发标记，其他阶段都是需要STW的，G1收集器不单单是追求低延迟的收集器，也衡量了吞吐量，所以在延迟和吞吐量之间做了一个权衡。
    
    G1的优缺点：
    
    *   优势：
        
        *   因为CMS是基于标记-清除的算法实现的，所以CMS会有空间碎片化的问题。而在G1收集器上是不存在的，G1从整体上来看是基于标记-整理算法实现，从Region之间又是基于标记-复制算法实现的。
            
        *   由于G1不会产生空间碎片，可以为对象的分配提供更规整的内存。此外还避免了由于分配大对象时找不到连续的内存空间，而不得不提前触发下一次垃圾回收。
            
    *   不足：
        
        *   由于跨Region引用等大量双向卡表的存在，G1收集器比CMS（只需要处理老年代到新生代的引用）占用更多的内存。
        *   CMS收集器使用写后屏障来更新维护卡表，而G1收集器除了使用写后屏障维护卡表，为了实现SATB的算法，还需要使用写前屏障来跟踪并发时指针变化情况。所以G1收集器会增加程序运行时的额外负载。
    
    G1的核心参数：
    
    ```java
    -XX:+UseG1GC  手动指定使用G1收集器执行内存回收任务（JDK9后不用设置，默认就是G1）。
    -XX:G1HeapRegionSize  设置每个Region的大小。值是2的幂,范围是1MB到32MB之间,目标是根据最小的Java堆大小划分出约2048个区域。默认是堆内存的1/2000。
    -XX:MaxGCPauseMillis  设置期望达到的最大GC停顿时间指标(JVM会尽力实现,但不保证达到)。默认值是200ms（如果这个值设置很小,如20ms,那么它收集的region会少,这样长时间后,堆内存会满。产生FullGC,FullGC会出现STW,反而影响用户体验)。
    -XX:G1NewSizePercent  新生代的最小值默认是5%，此参数在实验阶段，如果想使用加-XX:+UnlockExperimentalVMOptions参数。
    -XX:G1MaxNewSizePercent 新生代的最大值，默认值是60%，此参数在实验阶段，如果想使用加-XX:+UnlockExperimentalVMOptions参数。
    -XX:ParallelGCThreads 设置STW时GC线程数的值。最多设置为8(垃圾回收线程)。
    -XX:ConcGCThreads 设置并发标记的线程数。将n设置为并行垃圾回收线程数(ParallelGCThreads)的1/4左右。
    -XX:InitiatingHeapOccupancyPercent  设置触发并发GC周期的Java堆占用率阈值。超过此值,就触发GC。默认值是45%。
    
    ```
    

java9
-----

### java9新特性

#### 1.模块化

这个特性是java9中最大个一个新特性。我当时的感觉就是借鉴了OSGI的思想：一个模块可以通过声明式的配置，把自己的接口暴露给其他模块使用。只不过java9是通过module-info.java定义模块信息和依赖关系的。模块化想实现的目标：

*   主要目的在于减少内存开销
*   简化各种类库和大型应用的开发和维护
*   改进安全性、可维护性和提高性能

不过这种module-info.java方式，其实Maven通过pom.xml已经实现了。所以基本模块化这个java9中最主要的新特性，基本在实际开发项目中很少用到。

#### 2.接口私有化

java8中接口是可以有默认方法，假设有几个默认方法中，大部分的代码一致，那么最常规的操作就是把这几个方法中公共的部分抽离出一个private方法。而在java9中，增加了这个特性。

```java
public interface DemoInterface {
    void normalInterfaceMethod();

    default void defaultInterfaceMethod() { init(); }
    default void anotherDefaultInterfaceMethod() { init(); }

    
    private void init() { System.out.println("Initializing"); }
}

```

#### 3.String 存储结构优化

Java 8 及之前的版本，String 一直是用 char\[\] 存储。在 Java 9 之后，String 的实现改用 byte\[\] 数组存储字符串，节省了空间。

#### 4.try-with-resources 增强

java9以前，只能在try里面声明的变量，才能自动释放。如：

```Java
public void testTryWithJava8(){
    try(FileInputStream inputStream = new FileInputStream("aaa.html");) {
        String htmlContent = IoUtil.read(inputStream, StandardCharsets.UTF_8);
        System.out.println(htmlContent);
    }catch (IOException e) {
        System.out.println(e.getMessage());
    }
}

```

java9以后:

```Java
@Test
public void testTryWithJava9() throws FileNotFoundException {
    final FileInputStream inputStream1 = new FileInputStream("aaa.html");
    final FileInputStream inputStream2 = new FileInputStream("aaa.html");
    try (inputStream1; inputStream2) {
        String htmlContent = IoUtil.read(inputStream1, StandardCharsets.UTF_8);
        System.out.println(htmlContent);
    }catch (Exception e) {
        System.out.println(e.getMessage());
    }
}

```

#### 5.Stream和Optional增强

*   Stream 增强
    
    java9中Stream增加了新方法 ofNullable()、dropWhile()、takeWhile() 以及 iterate() 方法的重载方法。可以通过一个Predicate来指定什么时候结束迭代。
    
    ```java
    
     * 测试Stream新增takeWhile方法
     * takeWhile  从流中的头开始取元素,直到不满足条件为止
     */
    public static void testTakeWhile(){
        List<Integer> list = Arrays.asList(1, 89, 63, 45, 72, 65, 41, 65, 82, 35, 95, 100);
        
        list = list.stream().takeWhile(e -> e % 2 == 1).collect(Collectors.toList());
        
        System.out.println(list);
    }
    
    
     * 测试Stream新增dropWhile方法
     * dropWhile  从头开始删除满足条件的数据,直到遇见第一个不满足的位置,并保留剩余元素
     */
    public static void testDropWhile() {
        List<Integer> list = Arrays.asList(2, 86, 63, 45, 72, 65, 41, 65, 82, 35, 95, 100);
        
        list = list.stream().dropWhile(e -> e % 2 == 0).collect(Collectors.toList());
        
        System.out.println(list);
    }
    
    
     * 测试Stream新增ofNullable方法
     * ofNullable 允许创建Stream流时,只放入一个null
     */
    public static void testOfNullable(){
        
        Stream<Integer> stream1 = Stream.of(10, 20, 30, null);
        
        Stream<Integer> stream2 = Stream.ofNullable(null);
        
        Stream<Integer> stream3 = Stream.of(null);
    }
    
    
     * 测试Stream新增iterate方法
     * iterate指定种子数,指定条件和迭代方式来获取流
     */
    public static void testNewIterate(){
        
        Stream.generate(Math::random).limit(10).forEach(System.out::println);
        
        Stream.iterate(0,t-> t+2).limit(10).forEach(System.out::println);
        
        Stream.iterate(0,t -> t<10,t-> t+1).forEach(System.out::println);
    }
    
    ```
    
*   Optional 增强
    
    > Optional和Stream之间的结合也得到了改进,现在可以通过Optional的新方法将一个Optional对象转换为一个Stream对象(可能是空的)
    
    ```java
    
     * Optional类新增Stream方法,可以将一个Optional转换为Stream
     */
    public static void testOptionalStream(){
        List<Integer> list =new ArrayList<>();
        Collections.addAll(list,10,5,45,95,36,85,47);
        Optional<List<Integer>> optional=Optional.ofNullable(list);
    
        
        Stream<List<Integer>> stream = optional.stream();
        
        stream.flatMap(x->x.stream()).forEach(System.out::println);
    }
    
    ```
    

#### 6.快速创建不可变集合

增加了List.of()、Set.of()、Map.of() 和 Map.ofEntries()等工厂方法来创建不可变集合。使用 of() 创建的集合为不可变集合，不能进行添加、删除、替换、 排序等操作，不然会报异常。

#### 7.InputStream新增transferTo方法

InputStream新增transferTo方法,可以用来将数据直接传输到OutpuStream,这是在处理原始数据时非常常见的一种方法。

```java
InputStream inputStream =new FileInputStream("aaa.txt");
OutputStream outputStream=new FileOutputStream("bbb.txt");
try (inputStream; outputStream){
    
    inputStream.transferTo(outputStream);
} catch (IOException e) {
    e.printStackTrace();
}

```

### java9虚拟机

#### 1.G1成为默认垃圾回收器

在 Java 8 中，默认使用的垃圾回收器为 Parallel Scavenge（新生代）+Parallel Old（老年代）。但到了 Java 9，CMS 垃圾回收器被弃用，取而代之的是 G1（Garbage-First Garbage Collector），成为新的默认垃圾回收器。

java10
------

### java10新特性

#### 1.局部变量类型推断var

java10提供了类似javascript的var关键字来声明局部变量。

```java
var list = new ArrayList<>();
for(var i=0;i<10;i++) {
    list.add(i);
}

System.out.println(list);

```

在局部变量使用时，如下情况不适用：

*   初始值为null
*   方法引用
*   Lambda表达式
*   数组的静态初始化

#### 2.新增不可变集合方法

List，Set，Map 提供了静态方法copyOf()返回入参集合的一个不可变拷贝。使用 copyOf() 创建的集合为不可变集合，不能进行添加、删除、替换、 排序等操作，不然会报 java.lang.UnsupportedOperationException 异常。

```java
var list = new ArrayList<>();
list.add(1);list.add(2);list.add(3);
var list2 = List.copyOf(list);
list2.forEach(System.out::println);

list2.add(4);

```

### java10虚拟机

#### 1.垃圾回收器接口

在早期的 JDK 结构中，组成垃圾收集器 (GC) 实现的组件分散在代码库的各个部分。 Java 10 通过引入一套纯净的垃圾收集器接口来将不同垃圾收集器的源代码分隔开。

#### 2.G1并行Full GC

从 Java9 开始 G1 就了默认的垃圾回收器，G1 是以一种低延时的垃圾回收器来设计的，旨在避免进行 Full GC,但是 Java9 的 G1 的 FullGC 依然是使用单线程去完成标记清除算法,这可能会导致垃圾回收期在无法回收内存的时候触发 Full GC。为了最大限度地减少 Full GC 造成的应用停顿的影响，从 Java10 开始，G1 的 FullGC 改为并行的标记清除算法，同时会使用与年轻代回收和混合回收相同的并行工作线程数量，从而减少了 Full GC 的发生，以带来更好的性能提升、更大的吞吐量。

java11（LTS）
-----------

### java11新特性

#### 1.字符串API增强

*   isBlank() ： 判断字符串是否为空字符串，或者trim()之后是否为空字符串
*   strip() ：去掉字符串前后的全角和半角空白字符
*   repeat(n) ：按照给定的次数，重复字符串的内容
*   lines() ：将一个 字符串以行终止符（\\n 或者 \\r）进行分割，并将其分割为Stream流

```java

System.out.println("    ".isBlank());

System.out.println(" 去掉字符串首尾空格 ".strip());

System.out.println("重复字符串".repeat(3));

System.out.println("a\nb\rc".lines().collect(Collectors.toList()));

```

输出结果为：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/04da6f3a4f724dd6900adae083c36599~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=e6Z4DfiXMqEKNPSvo2hWH%2FRgvCY%3D)

#### 2.Http Client标准化

该 API 通过 CompleteableFuture 提供了非阻塞请求和响应，在 Java 11 中可以在 java.net 包中找到这个 API。使用示例：

```java
var request = HttpRequest.newBuilder()
                .uri(URI.create("https://www.mayuanfei.com"))
                .GET()
                .build();
var client = HttpClient.newHttpClient();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString(StandardCharsets.UTF_8));
System.out.println(response.body());

client.sendAsync(request, HttpResponse.BodyHandlers.ofString(StandardCharsets.UTF_8))
        .thenApply(HttpResponse::body)
        .thenAccept(System.out::println);

```

#### 3.Lambda表达式中可以使用var

在java10中就有局部变量类型推断var，但是这里存在几个限制：

*   只能用于局部变量上
*   声明时必须初始化
*   不能作为方法的参数
*   不能再Lambda表达式中使用

java11开始允许开发者在Lambda表达式中使用var进行参数声明。下例中，效果等价：

```java
Consumer<String> consumer1 = (var c) -> System.out.println(c);
Consumer<String> consumer2 = (String c) -> System.out.println(c);

```

#### 4.Optional增强

新增empty()方法判断执行的Optional对象是否为空

```java
var obj= Optional.empty();

System.out.println(obj.isEmpty());

```

#### 5.读写文件增强

对Files类增加了**writeString**和**readString**两个静态方法，可以直接把String写入文件，或者把整个文件读出为一个String。

```Java
String filePath = "/Users/mayuanfei/Desktop/bbb.txt";
Files.writeString(Path.of(filePath), "写入文件");
String fileContent = Files.readString(Path.of(filePath));
System.out.println(fileContent);

```

#### 6.单文件代码

该功能可以允许直接运行java代码。java文件会在内存中执行编译且直接执行。唯一的约束在于所有相关的类必须定义在一个java文件中。如下代码:

```Java
public class HelloWorld{
    public static void main(String[] args){
        System.out.println("Hello World!!!");
    }
}

```

执行java代码:

```shell
java HelloWorld.java
# 输出结果
Hello World!!!

```

### java11虚拟机

#### 1.新的垃圾回收器Epsilon

Epsilon垃圾回收器是一种实验性的垃圾回收器，它的设计目标是完全消除垃圾回收的暂停时间。需要注意的是，Epsilon垃圾回收器并不适用于大多数实际的生产环境，因为它会导致堆内存的快速耗尽，可能导致应用程序崩溃。它更多地用于特定的测试和调试目的。

启用参数：

```java
-xx:+UnlockExperimentalVMOptions -xx:+UseEpsilonGc

```

Java12
------

### Java12新特性

#### 1.数字格式化工具类

```Java
NumberFormat fmtShort = NumberFormat.getCompactNumberInstance(Locale.CHINESE, NumberFormat.Style.SHORT);
        String resultShort = fmtShort.format(10000000);
        System.out.println(resultShort);

        NumberFormat fmt2Short = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.SHORT);
        String result2Short = fmt2Short.format(10000000);
        System.out.println(result2Short);

        NumberFormat fmtLong = NumberFormat.getCompactNumberInstance(Locale.CHINESE, NumberFormat.Style.LONG);
        String resultLong = fmtLong.format(10000000);
        System.out.println(resultLong);

        NumberFormat fmt2Long = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.LONG);
        String result2Long = fmt2Long.format(10000000);
        System.out.println(result2Long);

```

输出结果：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/b621ca1a0dcd4c56834559875af73f1f~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=%2FR4f0I15uloWTXaWIJFqMi7RenA%3D)

#### 2.字符串增强

java11增加了两个字符串的处理方法：

*   indent() : 实现字符串缩进
*   transform() ：转变指定的字符串

```java
String name = "老马";
String nameIndent = name.indent(4);
System.out.println("缩进:" + nameIndent);
String msg = name.transform(n -> n + " 你好!");
System.out.println(msg);

```

执行结果：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/4a15a8ee2f55442b83a74e4947c033eb~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=%2BZ6ECfnbOWluY%2F%2B5ca1t58xij3I%3D)

#### 3.Files工具类增强

Java12添加了mismatch()方法来比较两个文件，如果两个文件内容相同则返回-1L；否则返回第一个不匹配字符的位置。假设两个文件内容如下：

*   bbb.txt 内容 : `中文3字节`
*   bbb2.txt内容: `中文4字节`

```Java
String file1Path = "/Users/mayuanfei/Desktop/bbb.txt";
String file2Path = "/Users/mayuanfei/Desktop/bbb2.txt";
long mismatch = Files.mismatch(Path.of(file1Path), Path.of(file2Path));
System.out.println("第一个不同字符的位置: " + mismatch);

```

输出结果：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/ba6c7bf9c5424ec4bd40e2c708b7d0bb~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=Z%2BoYLgvgrHsDP5%2FomTn1ZbDeAwU%3D)

### Java12虚拟机

#### 1.新的垃圾回收器Shenandoah GC

Shenandoah GC（Shenandoah Garbage Collector）是一种低停顿时间的垃圾回收器。Shenandoah GC的设计目标是在几乎不受应用程序停顿影响的情况下，实现较短的垃圾回收停顿时间。它采用了并发标记-并发清除（Concurrent Marking-Concurrent Clearing）的策略来实现这一目标。

Shenandoah GC的主要特点包括：

*   并发标记：Shenandoah GC在进行垃圾标记时，与应用程序线程并发执行，几乎没有停顿时间。
*   并发清除：在垃圾回收的清理阶段，Shenandoah GC也与应用程序线程并发执行，以减少垃圾回收的停顿时间。
*   压缩：Shenandoah GC采用了压缩技术，通过移动存活对象来减少内存碎片。
*   低停顿时间：Shenandoah GC的主要目标是实现低停顿时间，尽量减少应用程序的停顿。

Shenandoah GC适用于需要快速响应和低停顿时间的应用程序，特别是那些对延迟敏感的场景，如：在线事务处理、大规模互联网应用等。它在垃圾回收停顿时间方面的改进使得Java应用程序能够更好地满足实时性和性能要求。

Java13
------

### Java13新特性

#### 1.SocketAPI重构

Java 13 将 Socket API 的底层进行了重写， NioSocketImpl 是对 PlainSocketImpl 的直接替代，它使用 java.util.concurrent 包下的锁而不是同步方法。并且，在 Java 13 中是默认使用新的 Socket 实现。

#### 2.FileSystems增强

FileSystems 类是用于创建和访问文件系统的工具类。它提供了一组静态方法，用于创建不同类型的文件系统对象，例如本地文件系统、ZIP文件系统、JAR文件系统等。 java13中对该类增加了3个新方法：

*   newFileSystem(Path)
*   newFileSystem(Path, Map<String, ?>)
*   newFileSystem(Path, Map<String, ?>, ClassLoader)

```java

 * 该示例是读取ojdbc的jar包中,指定包路径下的所有文件名
 */
File jarFile = new File("/Users/mayuanfei/Desktop/ojdbc6-11.2.0.4.jar");
FileSystem fileSystem = FileSystems.newFileSystem(jarFile.toPath());
Path oracleSqlPath = fileSystem.getPath("oracle/sql");
Files.list(oracleSqlPath).forEach(path -> System.out.println(path.getFileName()));

```

执行结果：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/9d79669b1f0a4fd6bd613c7fce988a23~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=XVBCogABJqMfNxZKhSrFrWCo8H0%3D)

Java14
------

### Java14新特性

#### 1.switch 增强

之前Switch的写法：

```java
private static String getTextBefore(int number) {
    String result = "";
    switch (number) {
        case 1,2:
            result = "一或者二";
            break;
        case 3,4: 
            result = "三或者四";
            break;
        case 5,6,7:
            result = "五或者六或者七";
            break;
        default:
            result = "未知数值";
            break;
    };
    return result;
}

```

现在switch的写法

```java
private static String getText(int number) {
    return switch (number) {
        case 1,2 -> "一或者二";
        case 3,4 -> "三或者四";
        case 5,6,7 -> "五或者六或者七";
        default -> "未知数值";
    };
}

```

> 其实早在java12中就引入了switch的预览特性，但是需要增加参数来启用。而在java14中直接就能使用了。算是转正了；另外在java13中还提供了yield关键字标识一个程序块直接返回值。

### Java14虚拟机

#### 1.移除CMS垃圾收集器

移除了 CMS(Concurrent Mark Sweep) 垃圾收集器

#### 2.支持ZGC

从 Java11 引入的 ZGC 作为继 G1 过后的下一代 GC 算法，从支持 Linux 平台到 Java14 开始支持 MacOS 和 Window。至于ZGC的详细情况在java21一并介绍。

Java15
------

### Java15新特性

#### 1.隐藏类

隐藏类是为框架所设计的，隐藏类不能直接被其他类的字节码使用，只能在运行时生成类并通过反射间接使用它们。JEP371描述了隐藏类的原因：允许框架将类定义为框架不可发现的实现细节，这样它们就不能被其他类链接，也不能通过反射发现。这里举个例子，但是实际上在实际开发中基本不会用到，这里作为了解即可：

*   第一步，创建一个java类
    
    ```java
    public class JEP371HiddenClass {
        public static String hello() {
            return "hello the world!!!";
        }
    }
    
    ```
    
*   第二步，保证上面的JEP371HiddenClass被编译为class类文件
    
*   第三步，测试类
    
    ```java
    public class JEP371HiddenClassTest {
    
        
        public static String getHiddenClassBase64() throws IOException {
            String classFilePath = "JEP371HiddenClass.class";
            byte[] bytes = Files.readAllBytes(Path.of(classFilePath));
            return Base64.getEncoder().encodeToString(bytes);
        }
    
        public static void main(String[] args) throws Throwable {
            String CLASS_INFO = getHiddenClassBase64();
            System.out.println("类base64内容: " + CLASS_INFO);
            byte[] classInBytes = Base64.getDecoder().decode(CLASS_INFO);
            Class<?> proxy = MethodHandles.lookup()
                    .defineHiddenClass(classInBytes, true, 
                                       MethodHandles.Lookup.ClassOption.NESTMATE)
                    .lookupClass();
    
            
            System.out.println("类名: " + proxy.getName());
            
            for(Method method : proxy.getDeclaredMethods()) {
                System.out.println("方法名: " + method.getName());
            }
            
            MethodHandle mh = MethodHandles.lookup()
                .findStatic(proxy, "hello", MethodType.methodType(String.class));
            String result = (String) mh.invokeExact();
            System.out.println("执行hello方法结果: " + result);
    
        }
    }
    
    ```
    
    执行结果：
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/450041376a3a4c918797aa58b530b568~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=bZ%2BN%2FaPHlM8xEZohcwmLmW7SE3s%3D)
    

#### 2.文本块(text blocks)

文本块首次是在java13以预览形式发布的，经过13,14两个版本的改进，在java15中正式发布。使用示例：

```java
String htmlString = """
    <!DOCTYPE html>
    <html>
        <body>
            <h1>"Hello World!"</h1>
        </body>
    </html>
    """;
System.out.println(htmlString);


```

> 使用注意点：
> 
> *   开始的三个双引号必须单独成行
> *   仅使用空格或仅使用制表符来缩进文本块。混合空格将导致不规则缩进的结果。
> *   结尾的三个双引号位置，能决定前面的缩进

### Java15虚拟机

#### 1.禁用和废弃偏向锁

偏向锁的引入增加了 JVM 的复杂性大于其带来的性能提升。不过，你仍然可以使用 `-XX:+UseBiasedLocking` 启用偏向锁定，但它会提示 这是一个已弃用的 API。

#### 2.可用ZGC

经过多个版本的迭代，不断的完善和修复问题，ZGC 在 Java 15 已经可以正式使用了！不过，默认的垃圾回收器依然是 G1。你可以通过下面的参数启动 ZGC：

```Java
$ java -XX:+UseZGC className

```

更多ZGC介绍在Java21中。

Java16
------

### Java16新特性

#### 1.instanceof 转正

| JDK版本 | 更新类型 | JEP | 更新内容 |
| --- | --- | --- | --- |
| Java SE 14 | preview | [JEP 305open in new window](https://link.juejin.cn/?target=https%3A%2F%2Fopenjdk.org%2Fjeps%2F305 "https://openjdk.org/jeps/305") | 首次引入 instanceof 模式匹配。 |
| Java SE 15 | Second Preview | [JEP 375open in new window](https://link.juejin.cn/?target=https%3A%2F%2Fopenjdk.org%2Fjeps%2F375 "https://openjdk.org/jeps/375") | 相比较上个版本无变化 |
| Java SE 16 | Permanent Release | [JEP 394open in new window](https://link.juejin.cn/?target=https%3A%2F%2Fopenjdk.org%2Fjeps%2F394 "https://openjdk.org/jeps/394") | 模式变量不再隐式为 final。 |

java16以前的`instanceof`模式匹配能力，允许在条件判断中使用模式变量。示例代码：

```java
final Object msg = "Hello World";
if (msg instanceof String str) {
    System.out.println(str.length());
} else {
    System.out.println("Not a string");
}

```

在上面的示例中，我们使用`instanceof`模式匹配来判断`msg`是否是`String`类型。如果是，我们将其赋值给模式变量`str`，然后可以在条件块中直接访问`str`的方法和属性。到了java16，你可以对 instanceof 中的变量值进行修改。示例代码：

```java
Object msg = "Hello World";
if (msg instanceof String str) {
    str = "Hello laoma";
    System.out.println(msg);
    System.out.println(str);
} else {
    System.out.println("Not a string");
}

```

#### 2.Records(记录类) 增强

以前定义类都用`class`关键字，但是java16开始，就多了一个`record`选择。这个记录类主要是为了提供一种更为简洁、紧凑的final类的定义方式。例如下面的代码：

```java
public record Data( int x, int y)

```

上面这句话创建了一个记录类，隐含的做了如下事情：

*   所有字段都是final类型
*   有规范的构造方法
*   与字段名相同的访问方法
*   默认生成了equals、hashcode、toString方法

等价的类如下：

```java
public class Data {
    final private int x;
    final private int y;
    public Data( int x, int y){
        this.x = x;
        this.y = y;
    }

    public boolean equals(Object o) {
        ...
    }

    public int hashCode() {
       ...
    }

    public String toString() {
        ...
    }
    
    public int x() {
        return this.x;
    }
    
    public int y() {
        return this.y;
    }
}

```

### Java16虚拟机

#### 1.ZGC增强

Java16 将 ZGC 线程栈处理从安全点转移到一个并发阶段，甚至在大堆上也允许在毫秒内暂停 GC 安全点。消除 ZGC 垃圾收集器中最后一个延迟源可以极大地提高应用程序的性能和效率。

#### 2.弹性元空间

自从引入了 Metaspace 以来，根据反馈，Metaspace 经常占用过多的堆外内存，从而导致内存浪费。弹性元空间这个特性，可将未使用的 HotSpot 类元数据（即元空间，metaspace）内存更快速地返回到操作系统，从而减少元空间的占用空间。

#### 3.打包工具

在 Java 14 中，JEP 343 引入了打包工具，命令是 jpackage。在 Java 15 中，继续孵化，现在在 Java 16 中，终于成为了正式功能

Java17（LTS）
-----------

### Java17新特性

#### 1.增强的伪随机数生成器

Java 17 为伪随机数生成器 （pseudorandom number generator，RPNG，又称为确定性随机位生成器）增加了新的接口类型和实现，使得开发者更容易在应用程序中互换使用各种 PRNG 算法。一般来说，PRNG 会依赖于一个初始值，也称为种子，来生成对应的伪随机数序列。只要种子确定了，PRNG 所生成的随机数就是完全确定的，因此其生成的随机数序列并不是真正随机的。示例代码：

```java
String algorithm = "L128X256MixRandom";
RandomGeneratorFactory<RandomGenerator> mixRandom = RandomGeneratorFactory.of(algorithm);

RandomGenerator randomGenerator = mixRandom.create(System.currentTimeMillis());

for(int i=0;i<100;i++){
    int random = randomGenerator.nextInt(10);
    System.out.println(random);
}

```

#### 2.密封类（Sealed Classes）

Sealed Classes 有两个主流的翻译：1.密封类；2.封闭类。这里采用密封类这个翻译，但是当提到封闭类也知道是一个意思即可。密封类经过java15，java16两个版本的改进，在java17中转正了。它的作用就是限制类的继承，早先我们限制类的集成可以用`final`修饰，不过这种要么可以继承，要么不能继承，很是粗犷。为了进一步增强限制能力，java17中的密封类增加了几个重要的关键字：

*   sealed ： 修饰类/接口，用来描述这个类/接口为密封类/接口
*   non-sealed ：修饰类/接口，用来描述这个类/接口为非密封类/接口
*   permits ：用在`extends`和`implements`之后，指定可以继承或者实现的类

这里举个列子，有个定时任务的继承图如下：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/fb7ac62510a54652a60e4ad85d59f30a~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=bE56B0X2KVvVaIM6IR6V%2FLb3L64%3D)

要求如下：

*   TimedTask接口：只能被DagTimedTask和CronTimedTask实现，不能有别的类实现。
*   DagTimedTask和CronTimedTask类：能被任何类继承
*   TestDagTimedTask类：不能被任何类继承
*   TestCronTimedTask：可以被任何类继承

实现上面的示意代码如下：

```java

 * 定时任务接口
 */
public sealed interface TimedTask permits CronTimedTask, DagTimedTask {
    String run(String param);
}


 * cron表达式定时任务
 * sealed修饰之后子类就必须在sealed、non-sealed、final之间选择一个
 */
public non-sealed class CronTimedTask implements TimedTask {
    @Override
    public String run(String param) {
        return null;
    }
}

 * 工作流定时任务
 * sealed修饰之后子类就必须在sealed、non-sealed、final之间选择一个
 */
public non-sealed class DagTimedTask implements TimedTask {
    @Override
    public String run(String param) {
        return null;
    }
}


 * 具体实现类-可以被继承
 */
public class TestCronTimedTask extends CronTimedTask {
}

 * 具体实现类-不可以被继承
 */
public final class TestDagTimedTask extends DagTimedTask {
}



```

#### 3.弃用Applet API

Applet API 用于编写在 Web 浏览器端运行的 Java 小程序，很多年前就已经被淘汰了，已经没有理由使用了。本次弃用是为了删除。

#### 4.删除远程方法调用激活机制

删除远程方法调用 (RMI) 激活机制，同时保留 RMI 的其余部分。RMI 激活机制已过时且不再使用。

### Java17虚拟机

#### 1.删除实验性的AOT和JIT编译器

在 Java 9 的 JEP 295open in new window ,引入了实验性的提前 (AOT) 编译器，在启动虚拟机之前将 Java 类编译为本机代码。Java 17，删除实验性的提前 (AOT) 和即时 (JIT) 编译器，因为该编译器自推出以来很少使用，维护它所需的工作量很大。保留实验性的 Java 级 JVM 编译器接口 (JVMCI)，以便开发人员可以继续使用外部构建的编译器版本进行 JIT 编译。

Java18
------

### Java18新特性

#### 1.优化Java API文档中的代码片段

java18之前：

```java
<pre>{@code
    lines of source code
}</pre>

```

java18之后，可以通过 `@snippet` 标签来做这件事情。：

```java

 * The following code shows how to use {@code Optional.isPresent}:
 * {@snippet :
 * if (v.isPresent()) {
 *     System.out.println("v: " + v.get());
 * }
 * }
 */

```

#### 2.互联网地址解析SPI

Java 18 定义了一个全新的 SPI（service-provider interface），用于主机地址和域名地址的解析，以便 `java.net.InetAddress` 可以使用平台之外的第三方解析器。

```Java
InetAddress inetAddress = InetAddress.getByName("www.163.com");
System.out.println(inetAddress.getHostAddress());

```

### Java18虚拟机

#### 1.默认字符集为UTF-8

JDK 终于将 UTF-8 设置为默认字符集。在 Java 17 及更早版本中，默认字符集是在 Java 虚拟机运行时才确定的，取决于不同的操作系统、区域设置等因素，因此存在潜在的风险。比如不同的操作系统间，出现乱码的事情。

#### 2.简易的Web服务器

Java18之后，可以使用jwebserver命令启动一个简易的静态Web服务器。但是这个服务器是不支持CGI和Servlet，只限于静态文件。

#### 3.使用方法句柄重新实现反射核心

Java 18 改进了 java.lang.reflect.Method、Constructor 的实现逻辑，使之性能更好，速度更快。这项改动不会改动相关 API ，这意味着开发中不需要改动反射相关代码，就可以体验到性能更好反射。

Java19
------

### Java19新特性

#### 1.虚拟线程（预览）

虚拟线程（Virtual Thread）是 JDK 而不是 OS 实现的轻量级线程(Lightweight Process，LWP），许多虚拟线程共享同一个操作系统线程，虚拟线程的数量可以远大于操作系统线程的数量。虚拟线程在其他多线程语言中已经被证实是十分有用的，比如 Go 中的 Goroutine、Erlang 中的进程。

虚拟线程避免了上下文切换的额外耗费，兼顾了多线程的优点，简化了高并发程序的复杂，可以有效减少编写、维护和观察高吞吐量并发应用程序的工作量。

#### 2.结构化并发(孵化)

JDK 19 引入了结构化并发，一种多线程编程方法，目的是为了通过结构化并发 API 来简化多线程编程，并不是为了取代java.util.concurrent，目前处于孵化器阶段。结构化并发将不同线程中运行的多个任务视为单个工作单元，从而简化错误处理、提高可靠性并增强可观察性。也就是说，结构化并发保留了单线程代码的可读性、可维护性和可观察性。

Java20
------

### Java20新特性

#### 1.作用域值（第一次孵化）

作用域值（Scoped Values）它可以在线程内和线程间共享不可变的数据。当然你可以使用 `ThreadLocal` 来保存当前线程变量，但是需要手动清理，开发者常常忘记，且变量不能被子线程继承；而使用 `InheritableThreadLocal` 共享信息可以被子线程继承，但是数据会拷贝多份，占用更多内存。

引入`Scoped values`，**允许在线程内和线程间共享不可变数据**，这比线程局部变量更加方便，尤其是在使用大量虚拟线程时。这提高了易用性、可理解性、健壮性以及性能。不过这是一个正在孵化的 API，未来可能会被删除。

#### 2.结构化并发(二次孵化)

通过引入用于结构化并发 API 来简化多线程编程。结构化并发将在不同线程中运行的多个任务视为单个工作单元，从而简化错误处理，提高可靠性，增强可观察性。因为是个孵化状态提案，这里不做过多研究。

Java21（LTS）
-----------

### Java21新特性

#### 1.有序集合（[Sequenced Collections](https://link.juejin.cn/?target=https%3A%2F%2Fopenjdk.org%2Fjeps%2F431 "https://openjdk.org/jeps/431")）

JDK21中为了更清楚地表达顺序集合和简化操作方法，引入了三个新接口：

*   SequencedCollection
*   SequencedMap
*   SequencedSet

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/eae46188b7ff4966824718d224933280~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=V6akXWqued6YsqaE6kj3G8oZ54U%3D)

绿色方框是新增的 3 个接口，从中可以看到已有集合类的继承关系的变化：

*   List 继承自 SequencedCollection。
*   Deque 继承自 SequencedCollection。
*   LinkedHashSet 实现了 SequencedSet 接口。
*   SortedSet 继承自 SequencedSet。
*   LinkedHashMap 实现了 SequencedMap 接口。
*   SortedMap 继承自 SequencedMap。

示例代码：

```java
SequencedMap<Integer, String> map = new LinkedHashMap<>();
map.put(1, "老马");
map.put(2, "老崔");
map.put(3, "小猪");
map.forEach((key, value) -> System.out.println(value));

```

#### 2.记录模式（[Record Patterns](https://link.juejin.cn/?target=https%3A%2F%2Fopenjdk.org%2Fjeps%2F440 "https://openjdk.org/jeps/440")）

*   Record Patterns
    
    该功能简化了数据查询，使开发人员可以更轻松地从记录类中提取数据。这里举个例子理解该特性:
    
    ```Java
    
    record Point(int x, int y) {}
    
    static void printSum(Object obj) {
        if (obj instanceof Point p) {
            int x = p.x();
            int y = p.y();
            System.out.println(x+y);
        }
    }
    
    
    static void printSum(Object obj) {
        if (obj instanceof Point(int x, int y)) {
            System.out.println(x+y);
        }
    }
    
    ```
    
*   嵌套Record Pattern
    
    ```java
    record Point(int x, int y) {}
    enum Color { RED, GREEN, BLUE }
    record ColoredPoint(Point p, Color c) {}
    record Rectangle(ColoredPoint upperLeft, ColoredPoint lowerRight) {}
    
    static void printUpperLeftColoredPoint(Rectangle r) {
        if (r instanceof Rectangle(ColoredPoint ul, ColoredPoint lr)) {
            System.out.println(ul.c());
        }
    }
    public static void main(String[] args) {
        
        ColoredPoint upperLeft = new ColoredPoint(new Point(100, 100), Color.RED);
        
        ColoredPoint lowerRight = new ColoredPoint(new Point(200, 200), Color.GREEN);
        Rectangle rectangle = new Rectangle(upperLeft, lowerRight);
        
        printUpperLeftColoredPoint(rectangle);
    }
    
    ```
    

#### 3.switch模式匹配

switch 的模式匹配可以与Record Patterns结合使用 允许在任何对象上制定 switch 语句和表达式。看一下代码例子：

```java
public static void process(Object obj) {
    switch (obj) {
        case String s -> System.out.println("String: " + s);
        case Integer i -> System.out.println("Integer: " + i);
        case Double d -> System.out.println("Double: " + d);
        default -> System.out.println("Unknown type");
    }
}

public static void main(String[] args) {
    process(1);
    process("2");
    process(3d);
    process(4L);
}

```

#### 4.虚拟线程

​ 虚拟线程不是java发明的，而是从别的语言中借鉴过来的。比如Go语言的goroutines和Erlang语言早就支持虚拟线程了。只不过叫的名字是：”协程“。其实是一个意思。虚拟线程在开发使用过程中，可以当做一个普通线程来使用，虚拟线程通过线程来调度的。**虚拟线程的出现不是为了提高运行速度；而是用来提高吞吐量（指单位时间内应用处理请求的数量）的。** 

​ 上面加粗的这句话，是虚拟线程使用时的指导思想。也就是说如果你的\*\*应用场景中阻塞操作比较多的情况用虚拟线程；而没有阻塞的情况用线程。\*\*先把这个flag立好。

*   创建虚拟线程的几种方法。

```java
static class MyJob implements Runnable {
    @Override
    public void run() {
        try {
            
            Thread.sleep(1000);
            System.out.println(System.currentTimeMillis() + ":" + Thread.currentThread() + " : " + Thread.currentThread().isVirtual());
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}

@SneakyThrows
public static void main(String[] args) {
    
    Thread thread1 = Thread.startVirtualThread(new MyJob());
    thread1.join();

    
    Thread thread2 = Thread.ofVirtual().name("myVirtualThread2").unstarted(new MyJob());
    
    thread2.start();
    thread2.join();
    
    Thread thread3 = Thread.ofVirtual().name("myVirtualThread3").start(new MyJob());
    thread3.join();

    
    ExecutorService executorService = Executors.newVirtualThreadPerTaskExecutor();
    Future<?> job1 = executorService.submit(new MyJob());
    Future<?> job2 =executorService.submit(new MyJob());
    job1.get();
    job2.get();

    
    ThreadFactory threadFactory = Thread.ofVirtual().name("virtualThreadFactory").factory();
    Thread thread = threadFactory.newThread(new MyJob());
    thread.start();
    thread.join();
}

```

*   虚拟线程要解决的问题
    
    上面给出的例子中，运行结果如下：
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/45a4230822864061ac4d424450dfcaa7~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=UWxamvBwvACVVRfLd581G1QEi68%3D)
    
    可以看到在Executors方式创建的两个任务，几乎是同时执行的，这个在传统的线程中是不可能的。这就是最上面立的那个flag中所说的。有IO阻塞的时候，尽量用这个虚线程，它可以达到了一个线程并行执行两个任务的效果。可以把虚拟线程理解为一个对象，这个虚拟线程对象有几种状态，比如是不是睡眠中，是不是运行中，而一个线程可以支持同时运行多个虚拟线程对象，当线程发现某个虚拟线程对象睡眠时，就会去运行其他的虚拟线程对象。
    
    但是如果是非阻塞的情况就另当别论了，此时因为虚拟线程有上下文切换的时间消耗，执行效率反而不如多线程来的效率更高了。
    

### Java21虚拟机

#### 1.ZGC垃圾回收器

ZGC（The Z Garbage Collector）其实早在JDK11中就存在了，同一时期的Epsilon都被淘汰了，但是ZGC坚挺的留了下来。但是在java21默认还是G1垃圾回收器。

*   ZGC的目标
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/4dc7cbb119b44737a9891bddd2070d7f~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=LFSt7bLHicrhwCA59h4pxW5lpMw%3D)
    *   支持16TB级别的堆
    *   垃圾回收最大停顿时间不超过10ms，实际在java16那个版本中就不超过1ms了
    *   为将来的GC特性奠定基础
    *   最坏情况下应用程序的吞吐量下降15%
*   ZGC的内存布局
    
    ZGC完全抛弃了按代收集理论，它与G1一样将内存划分成各个小的区域的，但与G1有所不同的是，ZGC的各个内存区域称为页面(Page)，而且页面也不是全部大小相等。ZGC按照页面大小将页面分为三类：小页面、中页面和大页面。
    
    *   小页面：小页面容量固定为2MB，用于存放小于等于256KB的对象。
    *   中页面：中页面容量为32MB，用于存放大于等于256KB但小于4MB的对象。
    *   大页面：大页面的容量不固定，可以动态变化，主要用于存放大于等于4MB的大对象。
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/9cf8df2ef13745d2ba65c2f13035a74b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=ucf63Yt2D8WS27Br5c09GLKtk70%3D)
    
*   ZGC的工作流程
    
    ZGC工作流程大体上可以分为三个阶段： **1.标记阶段（标记存活对象） 2.对象转移阶段（转移存活对象） 3.对象重定位阶段（重定位对象指针）**
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/01604f5f49754f278188ffe731fb64db~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=D%2BE1TkXpa0MdxbrotiTT3DgGkFc%3D)
    
*   ZGC可配置参数
    
    ![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/40889ac49c97401fbb7bf79fe0c9a15a~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6ICB6amsOTUyNw==:q75.awebp?rk3s=f64ab15b&x-expires=1728541412&x-signature=CljPpO8%2FUvSqhZHPObEVusUtBqQ%3D)
    

Java8到Java21显著变化汇总
------------------

### 语言变化

#### 1.接口中可以有私有方法

```java
public interface DemoInterface {
    void normalInterfaceMethod();

    default void defaultInterfaceMethod() { init(); }
    default void anotherDefaultInterfaceMethod() { init(); }

    
    private void init() { System.out.println("Initializing"); }
}

```

#### 2.try-with-resources 增强

Java8中：

```java
public void testTryWithJava8(){
    try(FileInputStream inputStream = new FileInputStream("aaa.html");) {
        String htmlContent = IoUtil.read(inputStream, StandardCharsets.UTF_8);
        System.out.println(htmlContent);
    }catch (IOException e) {
        System.out.println(e.getMessage());
    }
}

```

java21中：

```java
public void testTryWithJava9() throws FileNotFoundException {
    final FileInputStream inputStream1 = new FileInputStream("aaa.html");
    final FileInputStream inputStream2 = new FileInputStream("aaa.html");
    try (inputStream1; inputStream2) {
        String htmlContent = IoUtil.read(inputStream1, StandardCharsets.UTF_8);
        System.out.println(htmlContent);
    }catch (Exception e) {
        System.out.println(e.getMessage());
    }
}

```

#### 3.局部变量类型推断var

```java
var list = new ArrayList<>();
for(var i=0;i<10;i++) {
    list.add(i);
}

System.out.println(list);

```

#### 4.http client标准化

```java
var request = HttpRequest.newBuilder()
                .uri(URI.create("https://www.mayuanfei.com"))
                .GET()
                .build();
var client = HttpClient.newHttpClient();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString(StandardCharsets.UTF_8));
System.out.println(response.body());

client.sendAsync(request, HttpResponse.BodyHandlers.ofString(StandardCharsets.UTF_8))
        .thenApply(HttpResponse::body)
        .thenAccept(System.out::println);

```

#### 5.读写文件增强

```java
String filePath = "/Users/mayuanfei/Desktop/bbb.txt";
Files.writeString(Path.of(filePath), "写入文件");
String fileContent = Files.readString(Path.of(filePath));
System.out.println(fileContent);

```

#### 6.Files工具类增强

```java

String file1Path = "/Users/mayuanfei/Desktop/bbb.txt";
String file2Path = "/Users/mayuanfei/Desktop/bbb2.txt";
long mismatch = Files.mismatch(Path.of(file1Path), Path.of(file2Path));
System.out.println("第一个不同字符的位置: " + mismatch);

```

#### 7.Switch增强

```java
public static void process(Object obj) {
    switch (obj) {
        case String s -> System.out.println("String: " + s);
        case Integer i -> System.out.println("Integer: " + i);
        case Double d -> System.out.println("Double: " + d);
        default -> System.out.println("Unknown type");
    }
}

```

#### 8.文本块

```java
String htmlString = """
    <!DOCTYPE html>
    <html>
        <body>
            <h1>"Hello World!"</h1>
        </body>
    </html>
    """;
System.out.println(htmlString);

```

#### 9.instanceof模式匹配

```java
final Object msg = "Hello World";
if (msg instanceof String str) {
    System.out.println(str.length());
} else {
    System.out.println("Not a string");
}

```

#### 10.记录类

```java
record Point(int x, int y) {}

static void printSum(Object obj) {
    if (obj instanceof Point(int x, int y)) {
        System.out.println(x+y);
    }
}

```

#### 11.密封类

```java

 * 定时任务接口
 */
public sealed interface TimedTask permits CronTimedTask, DagTimedTask {
    String run(String param);
}


 * cron表达式定时任务
 * sealed修饰之后子类就必须在sealed、non-sealed、final之间选择一个
 */
public non-sealed class CronTimedTask implements TimedTask {
    @Override
    public String run(String param) {
        return null;
    }
}

 * 工作流定时任务
 * sealed修饰之后子类就必须在sealed、non-sealed、final之间选择一个
 */
public non-sealed class DagTimedTask implements TimedTask {
    @Override
    public String run(String param) {
        return null;
    }
}

```

#### 12.有序集合

```java
SequencedMap<Integer, String> map = new LinkedHashMap<>();
map.put(1, "老马");
map.put(2, "老崔");
map.put(3, "小猪");
map.forEach((key, value) -> System.out.println(value));

```

#### 13.虚拟线程

```java
Thread thread1 = Thread.startVirtualThread(()->{
    try {
        Thread.sleep(1000);
        System.out.println(Thread.currentThread()+":" +Thread.currentThread().isVirtual());
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
});
thread1.join();

```

### 虚拟机变化

*   Java8的默认垃圾收集器为Parallel（并行）收集器，采用的是分代收集。
    
*   Java21默认垃圾收集器还是G1，但是推荐设置为ZGC这种局部收集的垃圾收集器。垃圾回收最大停顿时间不超过1ms。理论上只要开ZGC,设置一下最大最小堆内存即可：
    
    ```java
    -XX:+UseZGC -Xms512m -Xmx512m
    
    ```