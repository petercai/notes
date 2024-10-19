# Java 8函数式编程全攻略
Java 8 (又称为 jdk 1.8) 是 Java 语言开发的一个主要版本。 Oracle 公司于 2014 年 3 月 18 日发布 Java 8 ，它支持函数式编程，新的 JavaScript 引擎，新的日期 API，新的Stream API 等。

**特点和用途：** 

*   **单一抽象方法：**  函数接口只能有一个抽象方法，但可以有多个默认方法（default）或静态方法（static）。
*   **Lambda 表达式：**  可以使用函数接口创建 Lambda 表达式，从而简洁地表示匿名函数，例如在集合操作、线程处理等场景中。
*   **方法引用：**  可以通过函数接口的类型来引用一个已存在的方法，使代码更简洁和可读性更高。

> **肖哥弹架构** 跟大家“弹弹” 函数式，需要代码关注
> 
> 欢迎 点赞，点赞，点赞。
> 
> 关注公号**Solomon肖哥弹架构**获取更多精彩内容

历史热点文章
------

*   [依赖倒置原则：支付网关设计应用案例](https://link.juejin.cn/?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU2OTcxODA1NQ%3D%3D%26mid%3D2247484123%26idx%3D1%26sn%3Dc2c78ebed194d356cac3290d157d8ce6%26chksm%3Dfcfb2693cb8caf85a308441d73801f06501d88b81076460c50e83f1e22ac26922cfe894d5f38%26scene%3D21%23wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzU2OTcxODA1NQ==&mid=2247484123&idx=1&sn=c2c78ebed194d356cac3290d157d8ce6&chksm=fcfb2693cb8caf85a308441d73801f06501d88b81076460c50e83f1e22ac26922cfe894d5f38&scene=21#wechat_redirect")
*   [Holder模式（Holder Pattern）：公司员工权限管理系统实战案例分析](https://link.juejin.cn/?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU2OTcxODA1NQ%3D%3D%26mid%3D2247484084%26idx%3D1%26sn%3D8f69089bb88bddd749ef75dfceb301ae%26chksm%3Dfcfb26fccb8cafeae28bf17ccb869af81a67e9af72761b663b7b446f94003027c04c60a5d100%26scene%3D21%23wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzU2OTcxODA1NQ==&mid=2247484084&idx=1&sn=8f69089bb88bddd749ef75dfceb301ae&chksm=fcfb26fccb8cafeae28bf17ccb869af81a67e9af72761b663b7b446f94003027c04c60a5d100&scene=21#wechat_redirect")
*   [一个项目代码讲清楚DO/PO/BO/AO/E/DTO/DAO/ POJO/VO](https://link.juejin.cn/?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU2OTcxODA1NQ%3D%3D%26mid%3D2247484077%26idx%3D1%26sn%3D032d485e3a84565354e422e7476ff3bc%26chksm%3Dfcfb26e5cb8caff397ae7a25a893bb4d3271c300bf9611ea54a24af68937b6372a45eb4cfdc3%26scene%3D21%23wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzU2OTcxODA1NQ==&mid=2247484077&idx=1&sn=032d485e3a84565354e422e7476ff3bc&chksm=fcfb26e5cb8caff397ae7a25a893bb4d3271c300bf9611ea54a24af68937b6372a45eb4cfdc3&scene=21#wechat_redirect")
*   [写代码总被Dis：5个项目案例带你掌握SOLID技巧,代码有架构风格](https://link.juejin.cn/?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU2OTcxODA1NQ%3D%3D%26mid%3D2247484134%26idx%3D1%26sn%3D8ca92f4dcac17adea8417d795dfca449%26chksm%3Dfcfb26aecb8cafb8aa7d95931958b9077c2ddc0930d584eeb12edc2f70828af3853b6a6cf8d6%26scene%3D21%23wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzU2OTcxODA1NQ==&mid=2247484134&idx=1&sn=8ca92f4dcac17adea8417d795dfca449&chksm=fcfb26aecb8cafb8aa7d95931958b9077c2ddc0930d584eeb12edc2f70828af3853b6a6cf8d6&scene=21#wechat_redirect")
*   [里氏替换原则在金融交易系统中的实践，再不懂你咬我](https://link.juejin.cn/?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU2OTcxODA1NQ%3D%3D%26mid%3D2247484097%26idx%3D1%26sn%3D57f1539b7ba2c53284c2dd6550635890%26chksm%3Dfcfb2689cb8caf9fe0a333b21dd2fefefa99b719c1420f64b97a9e101e76698896f4458d5c1c%26scene%3D21%23wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzU2OTcxODA1NQ==&mid=2247484097&idx=1&sn=57f1539b7ba2c53284c2dd6550635890&chksm=fcfb2689cb8caf9fe0a333b21dd2fefefa99b719c1420f64b97a9e101e76698896f4458d5c1c&scene=21#wechat_redirect")

### 1\. BiConsumer<T, U>

**业务数据**：员工名单和他们的工作评价。

```java
List<Employee> employees = Arrays.asList(new Employee("Alice", "Excellent"), new Employee("Bob", "Good"));

```

**函数使用**：

```java
BiConsumer<Employee, String> recordFeedback = (employee, feedback) -> 
    System.out.println(employee.getName() + ": " + feedback);
employees.forEach(record -> recordFeedback.accept(record, record.getFeedback()));

```

**输出结果**：

```makefile
描述：记录员工评价
结果值：Alice: Excellent
         Bob: Good

```

### 2\. BiFunction<T, U, R>

**业务数据**：产品列表和折扣率。

```java
List<Product> products = Arrays.asList(new Product("Laptop", 1200.00), new Product("Smartphone", 700.00));
double discountRate = 0.9; 

```

**函数使用**：

```java
BiFunction<Product, Double, Double> calculateDiscountedPrice = (product, rate) -> product.getPrice() * rate;
double totalDiscountedPrice = products.stream()
                                      .mapToDouble(product -> calculateDiscountedPrice.apply(product, discountRate))
                                      .sum();

```

**输出结果**：

```yaml
描述：计算折扣后的总价格
结果值：折扣后总价格为 1560.00

```

### 3\. BinaryOperator

**业务数据**：一系列整数，需要求和。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);


```

**函数使用**：

```java
BinaryOperator<Integer> add = (a, b) -> a + b;
Integer sum = numbers.stream().reduce(add).orElse(0);

```

**输出结果**：

```null
描述：整数列表求和
结果值：总和为 15

```

### 4\. BiPredicate<T, U>

**业务数据**：员工名单和他们的工作年限。

```java
List<Employee> employees = Arrays.asList(new Employee("Alice", 5), new Employee("Bob", 3));
int experienceYears = 3;

```

**函数使用**：

```java
BiPredicate<Employee, Integer> isExperienced = (employee, years) -> employee.getYearsOfExperience() >= years;
List<Employee> experiencedEmployees = employees.stream()
                                                  .filter(isExperienced)
                                                  .collect(Collectors.toList());

```

**输出结果**：

```css
描述：筛选经验丰富的员工
结果值：[Alice]

```

### 5\. BooleanSupplier

**业务数据**：当前系统状态。

```java
boolean systemIsUp = true; 

```

**函数使用**：

```java
BooleanSupplier checkSystemStatus = () -> systemIsUp;
boolean systemStatus = checkSystemStatus.getAsBoolean();

```

**输出结果**：

```arduino
描述：检查系统是否运行正常
结果值：系统状态为 true（运行正常）

```

### 6\. Consumer

**业务数据**：一系列客户名称。

```java
List<String> customers = Arrays.asList("Alice", "Bob", "Charlie");

```

**函数使用**：

```java
Consumer<String> greetCustomer = name -> System.out.println("Hello, " + name);
customers.forEach(greetCustomer);

```

**输出结果**：

```markdown
描述：对每个客户打招呼
结果值：Hello, Alice
         Hello, Bob
         Hello, Charlie


```

### 7\. DoubleBinaryOperator

**业务数据**：两个销售额。

```java
double salesJanuary = 10000.0;
double salesFebruary = 12000.0;

```

**函数使用**：

```java
DoubleBinaryOperator sumSales = (a, b) -> a + b;
double totalSales = sumSales.applyAsDouble(salesJanuary, salesFebruary);

```

**输出结果**：

```null
描述：计算两个月的总销售额
结果值：总销售额为 22000.0

```

### 8\. DoubleConsumer

**业务数据**：一系列员工的绩效评分。

```java
List<Double> performanceScores = Arrays.asList(4.5, 3.8, 4.2);

```

**函数使用**：

```java
DoubleConsumer recordPerformance = score -> System.out.println("Performance Score: " + score);
performanceScores.forEach(recordPerformance);

```

**输出结果**：

```yaml
描述：记录每位员工的绩效评分
结果值：Performance Score: 4.5
         Performance Score: 3.8
         Performance Score: 4.2

```

### 9\. DoublePredicate

**业务数据**：一系列测试成绩。

```java
List<Double> scores = Arrays.asList(75.5, 82.0, 90.0, 65.0);

```

**函数使用**：

```java
DoublePredicate isPassingScore = score -> score >= 70.0;
long passingCount = scores.stream()
                            .filter(isPassingScore)
                            .count();

```

**输出结果**：

```null
描述：计算及格（大于等于70分）的测试成绩数量
结果值：及格人数为 3

```

### 10\. DoubleSupplier

**业务数据**：需要生成随机数的场景。

**函数使用**：

```java
DoubleSupplier random = Math::random;
double randomValue = random.getAsDouble();

```

**输出结果**：

```arduino
描述：生成一个[0, 1)区间的随机double值
结果值：一个随机double值，例如 0.5678

```

### 11\. DoubleToIntFunction

**业务数据**：一系列商品价格。

```java
List<Double> prices = Arrays.asList(19.99, 49.99, 29.99);

```

**函数使用**：

```java
DoubleToIntFunction toIntFloor = Math::toIntExact;
List<Integer> priceFloors = prices.stream()
                                   .map(toIntFloor)
                                   .collect(Collectors.toList());

```

**输出结果**：

```css
描述：将价格四舍五入到最接近的整数
结果值：[20, 50, 30]

```

### 12\. DoubleToLongFunction

**业务数据**：一系列大数值的测量数据。

```java
List<Double> measurements = Arrays.asList(123456789.0, 987654321.0);

```

**函数使用**：

```java
DoubleToLongFunction toLong = Double::longValue;
List<Long> measurementsLong = measurements.stream()
                                           .map(toLong)
                                           .collect(Collectors.toList());

```

**输出结果**：

```arduino
描述：将double类型的测量数据转换为long类型
结果值：[123456789, 987654321]

```

### 13\. DoubleUnaryOperator

**业务数据**：一系列需要增加固定值的数字。

```java
List<Double> numbers = Arrays.asList(5.5, 10.0, 15.5);
double addend = 3.0;

```

**函数使用**：

```java
DoubleUnaryOperator addFixedValue = number -> number + addend;
List<Double> updatedNumbers = numbers.stream()
                                        .map(addFixedValue)
                                        .collect(Collectors.toList());

```

**输出结果**：

```css
描述：在每个数字上增加3.0
结果值：[8.5, 13.0, 18.5]

```

### 14\. Function<T, R>

**业务数据**：一系列员工及其职位。

```java
List<Employee> employees = Arrays.asList(new Employee("Alice", "Developer"), new Employee("Bob", "Manager"));

```

**函数使用**：

```java
Function<Employee, String> getPosition = Employee::getPosition;
List<String> positions = employees.stream()
                                    .map(getPosition)
                                    .collect(Collectors.toList());

```

**输出结果**：

```css
描述：获取所有员工的职位
结果值：["Developer", "Manager"]

```

### 15\. IntBinaryOperator

**业务数据**：两个整数，需要计算它们的位运算结果。

```java
int a = 5; 
int b = 3; 

```

**函数使用**：

```java
IntBinaryOperator bitwiseAnd = (x, y) -> x & y;
int result = bitwiseAnd.applyAsInt(a, b);

```

**输出结果**：

```scss
描述：计算两个整数的按位与
结果值：1 (0101 & 0011)

```

### 16\. IntConsumer

**业务数据**：一系列整数，需要打印它们。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

```

**函数使用**：

```java
IntConsumer printNumber = System.out::println;
numbers.forEach(printNumber);

```

**输出结果**：

```markdown
描述：打印每个整数
结果值：1
         2
         3
         4
         5


```

### 17\. IntFunction

**业务数据**：一系列整数，需要转换为它们的平方。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4);

```

**函数使用**：

```java
IntFunction<Integer> square = n -> n * n;
List<Integer> squares = numbers.stream()
                                 .map(square)
                                 .collect(Collectors.toList());

```

**输出结果**：

```css
描述：计算每个整数的平方
结果值：[1, 4, 9, 16]

```

### 18\. IntPredicate

**业务数据**：一系列年龄值。

```java
List<Integer> ages = Arrays.asList(15, 20, 25, 30, 35);

```

**函数使用**：

```java
IntPredicate isAdultAge = age -> age >= 18;
long adultCount = ages.stream()
                       .filter(isAdultAge)
                       .count();

```

**输出结果**：

```null
描述：计算成年人的数量
结果值：成年人数为 3

```

### 19\. IntSupplier

**业务数据**：需要生成一个表示当前年份的整数。

**函数使用**：

```java
IntSupplier currentYear = () -> 2024;
int thisYear = currentYear.getAsInt();

```

**输出结果**：

```null
描述：获取当前年份
结果值：2024

```

### 20\. IntToDoubleFunction

**业务数据**：一系列表示年数的整数。

```java
List<Integer> years = Arrays.asList(1990, 2000, 2010, 2020);

```

**函数使用**：

```java
IntToDoubleFunction toDouble = Integer::doubleValue;
List<Double> yearDoubles = years.stream()
                                  .map(toDouble)
                                  .collect(Collectors.toList());

```

**输出结果**：

```yaml
描述：将年份转换为double类型
结果值：[1990.0, 2000.0, 2010.0, 2020.0]

```

### 21\. IntToLongFunction

**业务数据**：一系列表示秒数的整数。

```java
List<Integer> seconds = Arrays.asList(60, 120, 180);

```

**函数使用**：

```java
IntToLongFunction toLong = Integer::toLong;
List<Long> secondsLong = seconds.stream()
                                  .map(toLong)
                                  .collect(Collectors.toList());

```

**输出结果**：

```arduino
描述：将秒数转换为long类型
结果值：[60L, 120L, 180L]

```

### 22\. IntUnaryOperator

**业务数据**：一系列整数，需要转换为它们的相反数。

```java
List<Integer> numbers = Arrays.asList(-1, 2, -3, 4);

```

**函数使用**：

```java
IntUnaryOperator negate = n -> -n;
List<Integer> negatedNumbers = numbers.stream()
                                        .map(negate)
                                        .collect(Collectors.toList());

```

**输出结果**：

```css
描述：计算每个整数的相反数
结果值：[1, -2, 3, -4]

```

### 23\. LongBinaryOperator

**业务数据**：两个长整数，需要计算它们的乘积。

```java
long a = 1234567890123456789L;
long b = 987654321098765432L;

```

**函数使用**：

```java
LongBinaryOperator multiply = (x, y) -> x * y;
long product = multiply.applyAsLong(a, b);

```

**输出结果**：

```null
描述：计算两个长整数的乘积
结果值：一个非常大的乘积值

```

### 24\. LongConsumer

**业务数据**：一系列长整数，需要打印它们。

```java
List<Long> longs = Arrays.asList(1L, 2L, 3L, 4L, 5L);

```

**函数使用**：

```java
LongConsumer printLong = System.out::println;
longs.forEach(printLong);

```

**输出结果**：

```markdown
描述：打印每个长整数
结果值：1
         2
         3
         4
         5


```

### 25\. LongFunction

**业务数据**：一系列长整数，需要转换为它们的字符串表示。

```java
List<Long> longs = Arrays.asList(1L, 2L, 3L, 4L, 5L);

```

**函数使用**：

```java
LongFunction<String> toHexString = Long::toHexString;
List<String> hexStrings = longs.stream()
                                 .map(toHexString)
                                 .collect(Collectors.toList());

```

**输出结果**：

```css
描述：将长整数转换为十六进制字符串
结果值：["1", "2", "3", "4", "5"]

```

### 26\. LongPredicate

**业务数据**：一系列长整数，需要判断它们是否为偶数。

```java
List<Long> longs = Arrays.asList(1L, 2L, 3L, 4L, 5L);

```

**函数使用**：

```java
LongPredicate isEven = number -> number % 2 == 0;
long evenCount = longs.stream()
                        .filter(isEven)
                        .count();

```

**输出结果**：

```null
描述：计算偶数的数量
结果值：偶数的数量为 2

```

### 27\. LongSupplier

**业务数据**：需要生成一个表示当前时间戳的长整数。

**函数使用**：

```java
LongSupplier timestamp = System::currentTimeMillis;
long currentTimestamp = timestamp.getAsLong();

```

**输出结果**：

```null
描述：获取当前时间戳
结果值：当前时间戳的长整数值，例如 1674869600000

```

### 28\. LongToDoubleFunction

**业务数据**：一系列长整数，需要转换为double类型。

```java
List<Long> longs = Arrays.asList(1L, 2L, 3L, 4L, 5L);

```

**函数使用**：

```java
LongToDoubleFunction toDouble = Long::doubleValue;
List<Double> doubles = longs.stream()
                              .map(toDouble)
                              .collect(Collectors.toList());

```

**输出结果**：

```scss
描述：将长整数转换为double类型
结果值：[1.0, 2.0, 3.0, 4.0, 5.0]

```

### 29\. LongToIntFunction

**业务数据**：一系列长整数，需要转换为int类型。

```java
List<Long> longs = Arrays.asList(1L, 2L, 3L, 4L, 5L);

```

**函数使用**：

```java
LongToIntFunction toInt = Long::intValue;
List<Integer> ints = longs.stream()
                            .mapToInt(toInt)
                            .boxed()
                            .collect(Collectors.toList());

```

**输出结果**：

```arduino
描述：将长整数转换为int类型
结果值：[1, 2, 3, 4, 5]

```

### 30\. LongUnaryOperator

**业务数据**：一系列长整数，需要计算它们的两倍。

```java
List<Long> longs = Arrays.asList(1L, 2L, 3L, 4L, 5L);

```

**函数使用**：

```java
LongUnaryOperator doubleIt = number -> number * 2;
List<Long> doubledLongs = longs.stream()
                                 .map(doubleIt)
                                 .collect(Collectors.toList());

```

**输出结果**：

```css
描述：计算每个长整数的两倍
结果值：[2L, 4L, 6L, 8L, 10L]

```

### 31\. ObjDoubleConsumer

**业务数据**：一系列订单和它们的销售额。

```java
List<Order> orders = Arrays.asList(new Order(101, 100.0), new Order(102, 200.0));

```

**函数使用**：

```java
ObjDoubleConsumer<Order> logSales = (order, sales) -> 
    System.out.println("Order " + order.getId() + " has sales of " + sales);
orders.forEach(order -> logSales.accept(order, order.getSales()));

```

**输出结果**：

```css
描述：记录每个订单的销售额
结果值：Order 101 has sales of 100.0
         Order 102 has sales of 200.0

```

### 32\. ObjIntConsumer

**业务数据**：一系列书籍和它们的页数。

```java
List<Book> books = Arrays.asList(new Book("Java 8", 500), new Book("Kotlin", 300));

```

**函数使用**：

```java
ObjIntConsumer<Book> logPages = (book, pages) -> 
    System.out.println(book.getTitle() + " has " + pages + " pages");
books.forEach(book -> logPages.accept(book, book.getPages()));

```

**输出结果**：

```markdown
描述：记录每本书的页数
结果值：Java 8 has 500 pages
         Kotlin has 300 pages


```

### 33\. ObjLongConsumer

**业务数据**：一系列事件和它们的时间戳。

```java
List<Event> events = Arrays.asList(new Event("Conference", 1617636800000L), new Event("Meetup", 1618223200000L));

```

**函数使用**：

```java
ObjLongConsumer<Event> logTimestamp = (event, timestamp) -> 
    System.out.println(event.getName() + " is scheduled at " + timestamp);
events.forEach(event -> logTimestamp.accept(event, event.getTimestamp()));

```

**输出结果**：

```csharp
描述：记录每个事件的时间戳
结果值：Conference is scheduled at 1617636800000
         Meetup is scheduled at 1618223200000

```

### 34\. Predicate

**业务数据**：一系列客户邮箱地址。

```java
List<String> emailAddresses = Arrays.asList("alice@example.com", "bob", "charlie@example.com");

```

**函数使用**：

```java
Predicate<String> isValidEmail = email -> email.contains("@");
long validEmailCount = emailAddresses.stream()
                                       .filter(isValidEmail)
                                       .count();

```

**输出结果**：

```null
描述：计算有效的邮箱地址数量
结果值：有效邮箱数量为 2

```

### 35\. Supplier

**业务数据**：生成一个唯一的订单ID。

**函数使用**：

```java
Supplier<Long> uniqueOrderId = () -> System.nanoTime();
long newOrderId = uniqueOrderId.get();

```

**输出结果**：

```null
描述：生成一个基于纳秒的时间戳的唯一订单ID
结果值：一个基于当前时间纳秒的唯一长整数值，例如 1617636800000000000

```

### 36\. ToDoubleBiFunction<T, U>

**业务数据**：一系列订单和它们的成本。

```java
List<Order> orders = Arrays.asList(new Order(101, 100.0, 80.0), new Order(102, 200.0, 150.0));

```

**函数使用**：

```java
ToDoubleBiFunction<Order, Double> calculateProfit = (order, taxRate) -> (order.getSales() - order.getCost()) * (1 + taxRate);
double totalProfit = orders.stream()
                           .mapToDouble(order -> calculateProfit.applyAsDouble(order, 0.05))
                           .sum();

```

**输出结果**：

```erlang
描述：计算所有订单的总利润（含5
结果值：总利润为 45.0

```

### 37\. ToDoubleFunction

**业务数据**：一系列商品价格。

```java
List<Double> prices = Arrays.asList(19.99, 29.99, 39.99);

```

**函数使用**：

```java
ToDoubleFunction<Double> getDiscountedPrice = price -> price * 0.9;
double totalDiscounted = prices.stream()
                                 .mapToDouble(getDiscountedPrice)
                                 .sum();

```

**输出结果**：

```null
描述：计算所有商品的折扣价格总和
结果值：折扣后总价格为 83.29

```

### 38\. ToIntBiFunction<T, U>

**业务数据**：一系列订单和它们的销售额。

```java
List<Order> orders = Arrays.asList(new Order(101, 100.0), new Order(102, 200.0));

```

**函数使用**：

```java
ToIntBiFunction<Order, Integer> calculateTax = (order, taxRate) -> (int)(order.getSales() * taxRate);
int totalTax = orders.stream()
                      .mapToInt(order -> calculateTax.applyAsInt(order, 7))
                      .sum();

```

**输出结果**：

```erlang
描述：计算所有订单的税额（税率7
结果值：总税额为 27

```

### 39\. ToIntFunction

**业务数据**：一系列客户的ID。

```java
List<Integer> customerIds = Arrays.asList(101, 102, 103, 104);

```

**函数使用**：

```java
ToIntFunction<Integer> increment = id -> id + 1;
int nextCustomerId = customerIds.stream()
                                   .map(increment)
                                   .max(Integer::compare)
                                   .orElse

```

**输出结果**：

```null
描述：找出最大的客户ID并加1 
结果值：下一个客户ID为 105

```

### 40\. ToLongBiFunction<T, U>

**业务数据**： 一系列交易和它们的日期。

```java
List<Transaction> transactions = Arrays.asList(new Transaction("2021-03-01", 100.0), new Transaction("2021-03-02", 200.0));

```

**函数使用**：

```java
ToLongBiFunction<Transaction, String> parseDate = (transaction, format) -> 
    LocalDate.parse(transaction.getDate(), DateTimeFormatter.ofPattern(format)).toEpochDay();
long firstTransactionDay = transactions.stream()
                                          .mapToLong(transaction -> parseDate.applyAsLong(transaction, "yyyy-MM-dd"))
                                          .findFirst()
                                          .orElse(0L);

```

**输出结果**：

```null
描述：解析第一笔交易的日期为时间戳
结果值：第一笔交易对应的时间戳为 1615099200

```

### 41\. ToLongFunction

**业务数据**： 一系列员工的工号。

```java
List<Employee> employees = Arrays.asList(new Employee("Alice", 1), new Employee("Bob", 2));

```

**函数使用**：

```java
ToLongFunction<Employee> getEmployeeId = Employee::getId;
long maxEmployeeId = employees.stream()
                                 .mapToLong(getEmployeeId)
                                 .max()
                                 .orElse(0L);

```

**输出结果**

```null
描述：找出最大的员工工号
结果值：最大工号为 2L

```

### 42\. ToUnaryOperator

**业务数据**： 一系列字符串，需要转换为大写。

```ini
java
List<String> strings = Arrays.asList("hello", "world", "java", "8")

```

**函数使用**：

```java
UnaryOperator<String> toUpperCase = String::toUpperCase;
List<String> upperCaseStrings = strings.stream()
                                          .map(toUpperCase)
                                          .collect(Collectors.toList());

```

**输出结果**：

```css
描述：将所有字符串转换为大写
结果值：["HELLO", "WORLD", "JAVA", "8"]

```