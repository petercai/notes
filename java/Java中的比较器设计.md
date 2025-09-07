# Java中的比较器设计
### 一、引言

开始讲解本文的知识点之前，请看下面的一段代码，猜测方法的输出

```java
@Data
class Address {
    private String provinceCode;
    private String cityCode;
    private String areaCode;
    public Address(String provinceCode, String cityCode, String areaCode) {
        this.provinceCode = provinceCode;
        this.cityCode = cityCode;
        this.areaCode = areaCode;
    }
}

@Data
class Student {
    private String name;
    private Integer age;
    private Integer examScore;
    private Address address;

    public Student(String name, Integer age, Integer examScore, Address address) {
        this.name = name;
        this.age = age;
        this.examScore = examScore;
        this.address = address;
    }
}

private static List<Student> mockStudentList(){
    List<Student> res = new ArrayList<>();
    res.add(new Student("笨笨", 16, 90, new Address("Anhui", "Hefei", "001")));
    res.add(new Student("李四", 17, 95, new Address("Anhui", "Hefei", "001")));
    res.add(new Student("王五", 20, 95, new Address("ShangHai", "PuDong", "100010")));
    res.add(new Student("张三", 21, 80, new Address("ShangHai", "PuDong", "100010")));
    res.add(new Student("康康", 21, 72, new Address("ShangHai", "PuDong", "100010")));
    return res;
}
public static void main(String[] args){
    List<Student> students = mockStudentList();
    
    List<Student> collect = students.stream().sorted().collect(Collectors.toList());
    
    
    
    System.out.println(collect);
}  

```

正确答案是排序会异常，无论是使用上面代码里面的哪种方式去排序都会出现异常，其实这个问题的答案本身很好猜因为既然要对对象列表进行排序，最起码代码得知道具体的排序规则；

对于上面的代码如果要改正有下面两种办法，例如我们就想简单的按照分数排序：

方法1：去修改一下Student类的实现，去实现 Comparable 接口

```java
@Data
public class Student implements Comparable<Student> {
    private String name;
    private Integer age;
    private Integer examScore;
    private Address address;

    public Student(String name, Integer age, Integer examScore, Address address) {
        this.name = name;
        this.age = age;
        this.examScore = examScore;
        this.address = address;
    }

    @Override
    public int compareTo(Student o) {
        return Integer.compare(this.examScore, o.examScore);
    }
}

```

方法2：去修改主方法中的排序逻辑，明确指定排序规则：

```java
List<Student> students = mockStudentList();
List<Student> collect = students.stream().sorted(Comparator.comparing(Student::getExamScore)).collect(Collectors.toList());

System.out.println(collect);

```

上面的两种方法对应的则是本文重点分析的两个接口 一个是Comparable 一个是Comparator

### 二、Comparable

源码定义：

```java
public interface Comparable<T> {
    public int compareTo(T o);
}

```

这个接口如果实现了就表明当前的类应该是具备一种“可比较的能力”，例如Integer、Double、BigDecimal类的实现，翻看源码，发现都实现了这个接口。因为这些类型一般表达数值的含义，而数字一般是可以比较的。

此外我们发现String类型也是实现了这个接口的，所以说字符串是可以直接排序的原因也是如此

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
        
}

```

关于Comparable接口的要点其实不多 ，在本文开始的例子中，仔细分析源码会发现，进入到Java优化过的排序算法“TimSort ”的逻辑以后，排序的元素实际上是要转为Comparable类型的，如果参与排序的对象本身没实现这个接口，自然上转型就会失败。

所以我们如果想直接使用SteamAPI的sorted方法或者是Arrays.sort(）方法的时候都需要确保对象确实是“可比较的”

但是很明显存在对象需要进行比较 但是又不希望去实现Comparable接口的场合，所以Java提供了另一个接口Comparator

### 三、Comparator

对于Comparator接口则需要好好研究下，首先我们检查下源码，我们就会发现这个比较接口并不像Comparable那么简单，尤其是在Java8引入Default方法以后这个接口提供了大量默认方法：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/4e409971e5554715817347c2959d47d7~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6Ieq54S25ZC45rCU5Y-R5Yqo5py6:q75.awebp?rk3s=f64ab15b&x-expires=1728555261&x-signature=NGLgyuQ4OlMlE0HqncgB5SyzjdQ%3D)

通过这些方法就可以实现大量复杂的排序逻辑

比如现在有个需求是要求对本文的Student对象**先按照分数排序从高到底排序，相同分数的按照省份编码排序，同时需要将没有得分的学生排到最后**，这时候代码就可以这样写：

```java
List<Student> students = mockStudentList();
List<Student> collect = students.stream()
        .sorted(Comparator.comparing(Student::getExamScore, Comparator.nullsFirst(Comparator.naturalOrder()))
                .reversed()
                .thenComparing(Student::getAddress, Comparator.comparing(Address::getProvinceCode)))
        .collect(Collectors.toList());
System.out.println(collect);

```

首先sorted方法需要接收一个Comparator类型的参数，所以后面的这一连串的链式调用就是为了构造一种对象之间的比较规则；

#### 3.1 Comparator.comparing 方法

Comparator.comparing在源码中存在两种实现一种是两个参数的一种是一个参数的

先分析一个参数的

```java
public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
        Function<? super T, ? extends U> keyExtractor)
{
    Objects.requireNonNull(keyExtractor);
    return (Comparator<T> & Serializable)
        (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
}

```

keyExtractor用于提取出进行比较的属性，然后由于泛型的类型限制提取出来的类型必须实现Comparable接口所以随后直接使用compareTo方法构造了一个Compartor的Lambda表达式返回。

```java
public static <T, U> Comparator<T> comparing(
        Function<? super T, ? extends U> keyExtractor,
        Comparator<? super U> keyComparator)
{
    Objects.requireNonNull(keyExtractor);
    Objects.requireNonNull(keyComparator);
    return (Comparator<T> & Serializable)
        (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),
                                          keyExtractor.apply(c2));
}

```

两个参数的方法则是直接用第二个Comparator参数构造了一个Comparator Lambda表达式返回。也就是说两个参数的实现就是用第一个函数提取出要比较的部分，然后用第二个参数的比较器进行比较，就比如我们前面写的代码片段中：

```java
thenComparing(Student::getAddress, Comparator.comparing(Address::getProvinceCode))

```

第一个参数是用于提取出Student实例中Address属性，但是这个属性依然是个对象，对于代码而言还是不知道怎么对这个对象比较大小，所以提供了第二个参数告诉程序怎么比较。

#### 3.2 Comparator.nullsFirst 方法

在日常代码开发中，如果我们要对对象列表按照某个字段值进行排序但是这个字段值有可能为空，这时候如果我们直接排序是否会有问题呢？

就比如我们本文开始的例子，假如有学生的得分是null 那么按照下面的排序方法 你将收获熟悉的NPE异常：

```java
private static List<Student> mockStudentList() {
    List<Student> res = new ArrayList<>();
    res.add(new Student("笨笨", 16, 90, new Address("Anhui", "Hefei", "001")));
    res.add(new Student("李四", 17, 95, new Address("Anhui", "Hefei", "001")));
    res.add(new Student("王五", 20, 95, new Address("ShangHai", "PuDong", "002")));
    res.add(new Student("张三", 21, 80, new Address("ShangHai", "PuDong", "100010")));
    
    res.add(new Student("康康", 21, null, new Address("ShangHai", "PuDong", "100010")));
    return res;
}

private static void demo1() {
    List<Student> students = mockStudentList();
    List<Student> collect = students.stream().sorted(Comparator.comparing(Student::getExamScore)).collect(Collectors.toList());
    System.out.println(collect);
}

```

那么有没有办法避免呢 ？ 当然有 只要我们在代码中显示声明，排序的时候把null值排在前面（nullsFirst）或者后面（nullsLast）就可以：

```java
List<Student> collect = students.stream()
        .sorted(Comparator.comparing(Student::getExamScore, 
                                     Comparator.nullsLast(Comparator.naturalOrder())))
        .collect(Collectors.toList());

```

> PS：其实在Java8有这套解决方案以前 在Spring里面也有一个org.springframework.util.comparator.Comparators 类似的nullsLow 和nullsHigh来解决这种问题

有的读者可能会对 Comparator.nullsLast(Comparator.naturalOrder())) 这种写法有点疑问 为什么nulllast还需要再接收一个Comparator 参数呢？这时候就需要去看看这个Comparator.nullsLast实现了。

**在Comparator接口的同包下有个Comparators类（这个类没有用public修饰），这个类支撑了Comparator接口大量Default方法的功能实现**。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/6882cc6dae23430aa4cffce89cc1692b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6Ieq54S25ZC45rCU5Y-R5Yqo5py6:q75.awebp?rk3s=f64ab15b&x-expires=1728555261&x-signature=C%2BcOGWnaUuovqQUob4EpfLVqnBw%3D)

在Comparators下面定义了NullComparator，这个类就是实现null值排序的关键 我们关注下关键代码：

```java
 final static class NullComparator<T> implements Comparator<T>, Serializable {
    private final boolean nullFirst;   
    private final Comparator<T> real;

    NullComparator(boolean nullFirst, Comparator<? super T> real) {
        this.nullFirst = nullFirst;
        this.real = (Comparator<T>) real;
    }

    @Override
    public int compare(T a, T b) {
        if (a == null) {
            return (b == null) ? 0 : (nullFirst ? -1 : 1);
        } else if (b == null) {
            return nullFirst ? 1: -1;
        } else {
            
            return (real == null) ? 0 : real.compare(a, b);
        }
    }
 }    

```

代码写的非常清晰 ，我们传递进去的Comparator.naturalOrder() 比较器就是解决比较都不是null值的情况下如何进行比较的问题。

#### 3.2 Comparator.naturalOrder() 方法

这个所谓的自然排序方法的实现我们也可以深究一下怎么实现

```java

public static <T extends Comparable<? super T>> Comparator<T> naturalOrder() {
    return (Comparator<T>) Comparators.NaturalOrderComparator.INSTANCE;
}



enum NaturalOrderComparator implements Comparator<Comparable<Object>> {
    INSTANCE;

    @Override
    public int compare(Comparable<Object> c1, Comparable<Object> c2) {
        return c1.compareTo(c2);
    }

    @Override
    public Comparator<Comparable<Object>> reversed() {
        return Comparator.reverseOrder();
    }
}

```

从上面的代码可以看出所谓的自然排序就是调用实现了Comparable 实例的compareTo方法而已；（ps：**这种用枚举来实现接口的写法，是不是还挺少见的**）

### 四、总结

下面总结下Comparable、Comparator、以及Comparators；

首先前面两个是接口最后一个则是为Comparator接口提供了一些额外能力的一个支撑类

如果我们定义的类具备比较能力，就可以实现Comparable接口，复写compareTo方法，如果我们比较的对象没有实现这个接口但是仍然需要排序就需要通过Comparator接口提供的一系列方法去定义排序规则。

此外"枚举实现接口" 和 “接口定义default方法搭配一个同包权限的支撑类"的设计思路也可以在日常开发中进行学习和借鉴