# String、StringBuilder、StringBuffer区别
*   我们在开发中经常用到String对象【**一个项目中各个地方都有它的身影，你和同事所写的代码，所使用的框架、库、工具包基本都会用到String**】。作为使用率最高的对象，java对String做了不少优化，主要分为两方面，一个是使用方式的优化，一个是性能优化
    
*   使用方式的优化 体现在：虽然String是对象，但可以不用new来创建，可以直接用""来创建String对象，这种直接声明的方式称为**字面量**
    
*   字面量：字面值。在java中“数据”被称为“字面值”，如18（整型）、8.12（浮点型）、true/false（布尔型）、'a'/'中'（字符型）、"a"/"zhangsan"（字符串型）、等都是 字面量。共有5种字面量——整数型字面量、浮点型字面量、布尔型字面量、字符型字面量、字符串型字面量
    

```java
String name = "zhangsan"; 
int a = 18;               
float grade = 88.8;       

```

**几个专业名词**

> **Java内存模型**：分为栈内存、堆内存、方法区（含字符串常量池）
> 
> **栈、堆**：java将内存分为两种：栈内存、堆内存，区别如下
> 
> *   栈中主要存放一些基本数据类型的变量和对象引用；堆中存放由new创建的对象和数组
> *   栈的存储速度比较快，仅次于寄存器，栈数据可以共享；堆中的对象不可共享
> *   栈中的数据大小和生命周期必须是确定的，当没有引用指向数据时 java会自动释放掉为该变量分配的内存空间；堆中的数据大小和生命周期不需要确定，堆中对象由jvm的垃圾回收器负责回收
> 
> **==、equals**：见2.3 String的比较

**java内存模型**（JDK7之前、JDK7之后）

JDK7之前

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28054638207d469caf09874fef78f484~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=948&h=466&s=134520&e=png&b=fdfaf9)

JDK7之后

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34d11b4007eb4115812d99bbabe40dc5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=950&h=465&s=126153&e=png&b=fdfaf9)

| 类型 | 特点 | 适用场景 |
| --- | --- | --- |
| String | 不可变，线程安全；执行速度最慢 | 操作少量数据或不需要操作数据 |
| StringBuilder | 可变，线程不安全；执行速度最快 | 需要频繁操作数据、且**不用考虑**线程安全 |
| StringBuffer | 可变，线程安全；性能较低 | 需要频繁操作数据、且需要考虑线程安全 |

2.1 String类特点
-------------

String是Java中的一个内置类，Immutable不可变，即一旦创建String对象，它的值就不能被更改。**对String对象的replace、subString、toLowerCase等操作都会返回一个新String对象**，故每次操作String时 性能较低、浪费内存空间

String类实现了Serializable接口和Comparable接口，因此可以进行序列化、在网络上传输和比较操作

2.2 创建String对象、String实例化
------------------------

### 2.2.1 实例化方法

有两种方式创建String对象：字面量赋值、new关键字

*   使用字符串字面值创建String对象，如String str = "abc"：java中有个字符串常量池，**当创建一个字面量字符串时，JVM首先检查字符串常量池中是否已经存在该字符串，如果存在 则直接返回字符串对象的引用，否则就创建一个新的字符串对象并放入字符串常量池中，最终将该对象的引用赋值给变量str**。引用str指向常量池中字符串"abc"的地址，是在常量池中拿值；【字符串常量池中不会存储相同内容的字符串】

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1902f505f3db4e84bb4b89c97a0d2975~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=602&h=265&s=31287&e=png&b=fcfcfc)

*   使用构造函数构建String对象，如String str = new String("abc")：通过new关键字创建字符串对象，**会先检查字符串常量池中是否有相同的字符串，如果有 则拷贝一份放到堆中，然后返回堆中地址；如果没有 就先在字符串常量池中创建"abc"这个字符串，而后再复制一份放到堆中 并把堆地址返回给str**。即最终字符串常量池和堆内存都会有这个对象，最后返回的是堆内存的对象引用

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2247c2af9184ad08d3fbc67544b832c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=606&h=293&s=31458&e=png&b=fcfcfc)

**只要使用new方法，不管字符串常量池中是否存在"abc"，都会在堆中创建新的对象（注意 和字符串常量池中的"abc"相区分）**，方式一的效率高于方式二

```java
String str1 = "ab";
String str2 = new String("cd");
String str3 = new String("ab");

String x = new String("abc");
String y = new String("abc");
System.out.println(x == y);

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7df21d91945e4505a6806e2814768b07~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=636&h=287&s=37534&e=png&b=fdfdfd)

### 2.2.2 String str1="abc"和String str2=new String("abc")区别

经过上文讲解，我们就知道两者区别在于 创建对象个数不同

*   String str="abc"创建了几个对象？ **0个 或 1个**。如果字符串常量池中没有"abc"，则在常量池中创建"abc" 并让str引用指向该对象（1个）；如果字符串常量池中有"abc"，则一个都不创建 直接返回地址值给str（0个）
*   String str=new String("abc")创建了几个对象？ **1个 或 2个**。如果字符串常量池中没有"abc"，则在字符串常量池和堆内存中各创建一个对象，返回堆地址（2个）；如果常量池中有"abc"，则只在堆中创建对象并返回地址值给str。【new相当于在堆中新建了value值，每new一个对象就会在堆中新建，地址值也因此不同，堆中的value存储着指向常量池的引用地址】

由于new关键字会在堆中开辟空间，因此开发中一般不建议使用，直接用字面量形式赋值即可。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d80057880ade4e15ad21bfd615c1e135~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=975&h=444&s=277475&e=png&b=f6efe9)

2.3 String的比较
-------------

**基本数据类型 ==比较的是数据值；引用类型 "==" 比较的是地址是否相同**。

String作为引用类型，可通过 == 和 equals 来进行比较。

查看Object源码 发现equals也是调用"==" 判断地址是否相同。即在根类Object中，== 和 equals 时等价的，子类可重写equals方法，若未重写 则默认"equals"和"=="等价

```java
public class Object{
    
    public boolean equals(Object obj) {
        return (this == obj);
    }
    
}

```

而String中的equals方法是被重写过的，当String调用equals时 会比较字符串内容是否相同。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
	public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
}

```

上述也是面试常见的一个问题："==" 和 "equals" 的区别。很多人会回答 引用类型"==" 比较地址、"equals"比较值，这是错误的说法。**Object的equals方法比较的是对象的内存地址、而String的equals方法比较的是对象的值**。String的equals比较值 是因为String类重写了equals方法；类可重写equals方法，如果类没有重写equals方法，会使用根类Object的equals方法，此时equals和==等价

2.4 String类的“加法”
----------------

C++中是允许开发者重载操作符的，但是Java中并不允许，官方仅提供了对于String类的 "+" 和 "+=" 两个特殊重载，"+" 将两个字符串连接生成一个新的对象，即在内存中新开辟了一块空间。

为什么String对象能使用 "+" 操作符？毕竟String不是Java8大基本数据类型和对应的装箱类型，而是引用类型，它能够使用 "+" 是因为官方做了处理。

```java
@Test
public void testString() {
    String str1 = "ab";
    String str2 = "cd";
    String str3 = "ab" + "cd";
    String str4 = "abcd";
    System.out.println("str3==str4：" + (str3==str4));  
    
    String str5 = str1 + "cd";
    System.out.println("str5==str4：" + (str5==str4));  
    
    String str6 = str1 + str2;
    System.out.println("str6==str4：" + (str6==str4));  
    str5 = str5.intern();  
    System.out.println("str5==str4：" + (str5==str4));  
}

```

*   当使用"+"对两个字符串常量拼接时，该结果在编译期就确定了，由于"+"两边都是常量、因此会直接连接放入常量池，注意不会将"+"两边的常量放入常量池
*   当"+"两边有变量时，其结果无法在编译期间确定，只能在运行时知道，并不会像上面一样连接放入常量池，反而将结果放在堆中。怎么将运行时动态的变量放入常量池呢？可使用String提供的native的intern()方法，将调用它的对象尝试放入常量池，如果常量池已有该字符串 就返回指向常量池中的引用，如果没有就放入常量池 并返回指向常量池中的引用

当进行字符串拼接时 String str = "ab"+"cd"，其底层会创建StringBuilder对象并调用StringBuilder的append方法追加字符串，而后使用toString()方法将对象转为String

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89f1cbea87a54d43ae72e880a2bce3e0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=918&h=521&s=347371&e=png&b=f4ede9)

**字符串拼接操作的总结**

*   String str1 ="ab"+"cd"：常量 与 常量 的拼接结果在 常量池，原理是 编译期 优化；常量池 中不会存在相同内容的常量；
*   String str2 = str1+"ef", String str6 = str1+str2：只要其中一个是变量，结果就在堆中。
*   变量拼接的原理 是StringBuilder 。
*   如果拼接的结果调用 intern() 方法，则主动将常量池中 还没有的字符串对象放入池中，并返回地址。

2.5 String与基本数据类型转换
-------------------

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5e37988a5174cdcb46263c5422c4f62~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=910&h=452&s=232333&e=png&b=fafafa)

UTF-8用三个二进制数据存储汉字，gbk用两个二进制数据存储汉字

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abf7072292134ec69954a4728e37f883~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=883&h=409&s=250551&e=png&b=fcfbfb)

2.6 常用方法
--------

length()、isEmpty()、charAt(int index)、substring(int beginIndex\[, int endIndex\])、equals(Object obj)、equalsIgnoreCase(String anotherString)、indexOf(String str)、toUpperCase()、toLowerCase()

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e31e82c4a344148a369a7baefa13677~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=933&h=499&s=712881&e=png&b=b2c9e3)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2b3768e86d24f9cb9298dbc29f535b4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=935&h=498&s=660258&e=png&b=b4cae3)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c1e539093394e9c8ce2f33f965a37ce~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=989&h=485&s=696659&e=png&b=b2c7e2)

按照常规方式创建对象时，即使对象属性安全一致、也会创建多个对象，资源浪费 ——》希望资源复用，声明静态常量 public static final xx，但是如果要复用一万个无规律的对象呢 ——》java提出 对象池 的概念，对象池就像一个缓存，池中存放资源

```java
User u1 = new User("zhangsan", 10);
User u2 = new User("zhangsan", 10);
Sout(u1==u2); 

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81f45dafd8404c958bcb956e110df886~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1024&h=428&s=193040&e=png&b=fcfbfb)

**JVM提供了字符串常量池**，位于堆中

String不可变，指的是 对象本身的属性或数据不会改变，即**一个对象创建后，其值不能改变（但可以修改指向），任何修改操作都会创建一个新的对象**。将变量重新赋值 是创建了一个新对象、然后将新对象的引用赋值给了变量，之前的对象是没有受到影响的

```java
public static void main(String[] args) {
    String str1 = "abc";
    str1 = "xyz";
    System.out.println(str1);   
}

```

再次给s赋值时，并不是对原来堆中实例对象进行重新赋值，而是生成一个新的实例对象，并且指向“def”这个字符串，s则指向最新生成的实例对象，之前的实例对象仍然存在，如果没有被再次引用，则会被垃圾回收。

在谈String不可变之前，我们先了解一下Java中的不可变对象

3.1 可变类型与不可变类型
--------------

1）改变一个变量、改变一个变量的值 有何区别

*   改变一个变量：将该变量指向另一个存储空间 ——修改指向
*   改变一个变量的值：将该变量当前指向的存储空间中写入一个新的值 ——修改存储内容

**2）可变类型与不可变类型**

*   不可变数据类型（immutable）：**一旦被创建，其值不能改变（但可以修改指向），任何修改操作都会创建一个新的对象**。如果想要改变数据的内容，只能改变对地址的引用，即指向其他地址。 不可变类型是线程安全的；但在频繁修改值的时候往往要开辟很多另外的地址，造成大量空间冗余
*   可变数据类型（mutable）：拥有方法可以修改自己的值/引用。 方便值的频繁修改，但程序安全性无法得到保障

> 一般情况下，java中基本数据类型（int/long/float等）、包装类、String、BigInteger、BigDecimal、Enum枚举类型、LocalDate/LocalTime/LocalDateTime等日期时间类均为不可变类型；
> 
> 而StringBuilder、StringBuffer、ArrayList、List、Set、Map均为可变类型
> 
> ```java
> String name = new String("Jenny");
> Integer age = new Integer(18);
> 
> 
> ```
> 
> 我们可自定义 不可变类。需遵守如下规则：类声明为final，以防止被继承；将类的所有成员变量被声明为private final，以确保字段在对象创建后不可被修改；仅为成员变量提供getter方法，不提供改变成员变量的方法；不提供字段的公共方法，或者在提供访问方法时，返回字段的拷贝而不是直接返回字段本身
> 
> *   对于基本类型来说，不可变 指 变量当中的数据不可改变；
> *   对于引用类型来说，不可变 指 对象本身的属性或数据不会改变，即一个对象重新创建后，不能对该对象进行任何更改。将变量重新赋值 是创建了一个新对象、然后将新对象的引用赋值给了变量，之前的对象是没有受到影响的。

**3）引用的不可变**

不可变类型的值已经不可再改变，但可以修改指向。如果我们想进一步让变量恒定成常量，则必须实现变量的引用不可变，即指向地址的指针不可变，我们只需要用final关键字修饰变量即可。final 限定变量的引用指向一个地址，不可修改

final限制了变量的引用不可更改，但没有限制 引用指向的地址中的数据的修改

*   final + 基本数据类型/不可变的引用数据类型：如final int —— 不可修改该变量
*   final + 可变的引用数据类型：final List——只限制了引用不可更改，但并未限制 引用指向的地址中的数据的修改

3.2 String为什么不可变
----------------

查看源码，String底层使用的 private final char value\[\]

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    
    private final char value[];
    
}    

```

*   因为final关键字？ 不对。被final修饰只能表示 它不可指向新的数组，又不能代表数组本身的数据不可被修改
*   真正不可变的原因时因为**private关键字**、并且String没有暴露和提供任何修改字符数组的方法，一些字符串操作都是返回新的String对象 不会影响原来数据。获取其底层字符数组时 都是复制一个新数组进行返回。并且String类型被final修饰，代表其不可被继承，从而杜绝子类覆盖父类行为的可能

**String不可变的优缺点**——保证字符串常量池发挥作用、哈希码不变 可放心使用与hash相关的对象 HashMap HashSet、不可被对象都是线程安全的

为解决String操作频繁产生临时对象的问题，Java提出StringBuilder与StringBuffer，两者都可变，即append、delete、replace操作 返回的都是源对象。

两者没有本质区别，两个类的构造器、append/delete/replace/toString等方法也基本相同，主要不同点在:

*   StringBuilder线程不安全、执行效率高
*   StringBuffer线程安全、执行效率低。除非有线程安全的需要，不然还是推荐使用StringBuilder

**StringBuilder、StringBuffer的“可变”细节**：

*   append、insert、replace、delete方法本质上都是调用System.arraycopy()方法，返回源字符串
*   substring(int)、substring(int,int)都是通过new String(value, start, end - start)创建一个新字符串。故在执行substring操作时，String、StringBuilder、StringBuffer基本没区别

4.1 底层原理
--------

查看String、StringBuilder、StringBuffer源码

```java

public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    private final char value[];
    ...
}



public final class StringBuilder
    extends AbstractStringBuilder
    implements Serializable, CharSequence
{
    @Override
    public StringBuilder append(Object obj) {}
    ...
}



public final class StringBuffer
    extends AbstractStringBuilder
    implements Serializable, CharSequence
{
    
    @Override
    public synchronized StringBuffer append(Object obj){...}
    ...
}

```

*   StringBuilder、StringBuffer都继承自AbstractStringBuilder，
*   查看AbstractStringBuilder源码，发现其底层利用可修改的char数组（JDK 9以后是byte数组），该类里面也包含了append、delete、replace、toString基本操作。
*   StringBuilder、StringBuffer继承AbstractStringBuilder，覆盖实现基本方法，区别仅在于最终的方法是否加了synchronized关键字。StringBuffer类中的增删改方法都添加了synchronized，每次操作字符串时都会加锁，所以线程安全、但是性能低

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76e9920650134ee6b0ee0948730891e8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=517&h=204&s=29171&e=png&b=2f323b)

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    public AbstractStringBuilder append(Object obj) {
        return append(String.valueOf(obj));
    }
    ...
}

```

4.2 Java中各种池：
-------------

池相当于一个共享资源，是对资源的整合和调配，节省存储空间，当需要的时候可以直接在池中取，用完之后再还回去。比如，如果你喝水，你可以拿杯子去水龙头接。如果很多人喝水，那就只能排队去接。现在有了池，每个人到池里舀一杯就好，省去了等着水龙头出水的时间。

**Java中常用的池都有：常量池、线程池、数据库连接池**

*   **常量池**：常量池在java用于保存编译期已确定的，已编译的class文件中的一份数据。它包括了关于类、方法、接口中的常量，也包括字符串常量。常量在编译器赋值时，会先在常量池中检测是否有这个值，若存在直接返回该值的地址值给常量，若不存在则先在常量池中创建该值，再返回该值的地址给常量。因此常量池中不可能出现相等的数据。

常量池的好处：常量池是为了避免频繁的创建和销毁对象而影响系统性能，实现了常量池中的内容由对象共享。例如字符串常量池，在编译阶段就把所有的字符串文字放到一个常量池中。节省内存空间：常量池中所有相同的字符串常量被合并，只占用一个空间。节省运行时间：比较字符串时，双等比equals()快。对于两个引用变量，只用==判断引用是否相等，也就可以判断实际值是否相等。

整数常量池 \[-128,127\]：将\[-128, 127\]之间256个整数所有的包装对象提前创建好。如果一个整数范围在\[-128, 127\]里面的整数进行包装，包装时不需要再new对象了，直接从“整数常量池”中取出来

字符串常量池

*   **线程池**：ThreadLocal。

先判断线程池中的核心线程们是否空闲，如果空闲，就把这个新的任务指派给某一个空闲线程去执行。如果没有空闲，并且当前线程池中的核心线程数还小于 corePoolSize，那就再创建一个核心线程。

如果线程池的线程数已经达到核心线程数，并且这些线程都繁忙，就把这个新来的任务放到等待队列中去。如果等待队列又满了，那么查看一下当前线程数是否到达maximumPoolSize，如果还未到达，就继续创建线程。如果已经到达了，就交给RejectedExecutionHandler来决定怎么处理这个任务。

*   **连接池**

String、StringBuilder、StringBuffer区别:

*   String为字符串常量，不可变、线程安全
*   StringBuilder、StringBuffer均为字符串变量、可变，其中StringBuilder线程不安全、StringBuffer线程安全。StringBuffer和StringBuilder可通过toString()方法转换成String类型。执行速度：StringBuilder > StringBuffer > String
*   操作**少量**的字符串 用String； **单线程**下需要操作**大量**数据 用StringBuilder；**多线程**下需要操作**大量**数据 用StringBuffer

字符串拼接操作:

*   常量 与 常量 的拼接结果在 常量池，原理是 编译期 优化；常量池 中不会存在相同内容的常量
*   只要其中一个是变量，结果就在堆中
*   变量拼接的原理 是StringBuilder
*   如果拼接的结果调用 intern() 方法，则主动将常量池中 还没有的字符串对象放入池中，并返回地址

String有两种实例化方法：字面量赋值、new关键字

String str1="abc"和String str2=new String("abc")区别：创建对象不同

*   String str="abc"创建 \*\*0个 或 1个 \*\*对象？。如果字符串常量池中没有"abc"，则在常量池中创建"abc" 并让str引用指向该对象（1个）；如果字符串常量池中有"abc"，则一个都不创建 直接返回地址值给str（0个）
*   String str=new String("abc")创建 **1个 或 2个** 对象。如果字符串常量池中没有"abc"，则在字符串常量池和堆内存中各创建一个对象，返回堆地址（2个）；如果常量池中有"abc"，则只在堆中创建对象并返回地址值给str。【new相当于在堆中新建了value值，每new一个对象就会在堆中新建，地址值也因此不同，堆中的value存储着指向常量池的引用地址】
