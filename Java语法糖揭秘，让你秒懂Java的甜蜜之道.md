# Java语法糖揭秘，让你秒懂Java的甜蜜之道
### 简介

语法糖（Syntactic Sugar），也称糖衣语法，是由英国计算机学家 Peter.J.Landin 发明的一个术语，指在计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用。简而言之，语法糖让程序更加简洁，非常利于操作。事实上听名字也能想到，加在语法中的糖让语法变得更”甜“。

### 糖1：switch

switch对于char, byte, short, int类型是本身就支持的，但其实它们都是转换成了整型，最后支持的其实是整型。但由于Java语法糖的出现，switch也支持String和enum类型了

原代码

```java
public class SugarTest {
    public static void main(String[] args) {
        String str = "java";
        switch (str) {
            case "java":
                System.out.println("1");
                break;
            case "javac":
                System.out.println("2");
                break;
            default:
                System.out.println("default");
        }
    }
}

```

编译后代码

```java
public class SugarTest {
    public SugarTest() {
    }

    public static void main(String[] args) {
        String str = "java";
        byte var3 = -1;
        switch(str.hashCode()) {
        case 3254818:
            if (str.equals("java")) {
                var3 = 0;
            }
            break;
        case 100899457:
            if (str.equals("javac")) {
                var3 = 1;
            }
        }

        switch(var3) {
        case 0:
            System.out.println("1");
            break;
        case 1:
            System.out.println("2");
            break;
        default:
            System.out.println("default");
        }

    }
}

```

可以看出，switch对于字符串类是比较字符串的hashcode方法以及通过equals方法确定是哪个字符串的，由于hashcode有一定概率会出现hash碰撞，用hashcode比较可能不太安全，所以仍需要进一步通过equals比较

### 糖2：自动拆装箱

自动拆装箱实际上是通过包装类的valueOf方法和xxxValue方法实现的，具体见我的一篇文章[解锁Java自动拆装箱的神秘面纱【字节码级别深度解析】 - 掘金 (juejin.cn)](https://juejin.cn/post/7303798788718657573 "https://juejin.cn/post/7303798788718657573")专门介绍

### 糖3：for-each语法

增强for循环可谓是广泛使用，省去了去定义一个自增变量

```java
public class SugarTest {
    public static void main(String[] args) {
        String[] strs = new String[]{"java", "python", "c++"};
        for (String str : strs) {
            System.out.println(str);
        }

        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            list.add(i);
        }

        for (Integer num : list) {
            System.out.println(num);
        }
    }
}

```

编译后代码

```java
public class SugarTest {
    public SugarTest() {
    }

    public static void main(String[] args) {
        String[] strs = new String[]{"java", "python", "c++"};
        String[] var2 = strs;
        int i = strs.length;

        for(int var4 = 0; var4 < i; ++var4) {
            String str = var2[var4];
            System.out.println(str);
        }

        List list = new ArrayList();

        for(i = 0; i < 3; ++i) {
            list.add(i);
        }

        Iterator var7 = list.iterator();

        while(var7.hasNext()) {
            Integer num = (Integer)var7.next();
            System.out.println(num);
        }

    }
}

```

可以看出，一般的数组使用for-each背后的实现仅仅是把它变为普通fot循环而已，对于集合类使用for-each循环背后的实现是使用了Iterator遍历的

由于使用了迭代器Iterator，所以在数组遍历时不能修改数组（比如调用remove方法），否则根据fail-fast机制会抛出java.util.ConcurrentModificationException异常

### 糖4：方法变长参数

Java是允许将一种类型的任意数量的值作为参数使用的

```java
public class SugarTest {
    public static void main(String[] args) {
        test("java", "python", "c++");
    }

    public static void test(String... strs) {
        for (String str : strs) {
            System.out.println(str);
        }
    }
}

```

编译后的代码

```java
public class SugarTest {
    public SugarTest() {
    }

    public static void main(String[] args) {
        test("java", "python", "c++");
    }

    public static void test(String... strs) {
        String[] var1 = strs;
        int var2 = strs.length;

        for(int var3 = 0; var3 < var2; ++var3) {
            String str = var1[var3];
            System.out.println(str);
        }

    }
}

```

事实上就是将参数转变为了对应的数组

### 糖5：if条件编译

条件编译就是在编译期间编译器会选择有效的代码编译，不满足条件的代码称为无用代码，会被直接丢弃

```java
public class SugarTest {
    private static final boolean DEBUG = true;

    public static void main(String[] args) {
        if (DEBUG) {
            System.out.println("DEBUG模式");
        } else {
            System.out.println("非DEBUG模式");
        }
    }
}

```

编译后的代码

```java
public class SugarTest {
    private static final boolean DEBUG = true;

    public SugarTest() {
    }

    public static void main(String[] args) {
        System.out.println("DEBUG模式");
    }
}

```

上面的代码中编译器在编译时已经能确定只用执行DEBUG为true的代码，那么编译器只需要保存满足条件的一行代码即可，不需要全部保存

##### 谈一谈条件编译的作用

有人会问了，既然DEBUG已知为true了，还去写个if多此一举干嘛，这其实是为了后期便于维护，就像上述代码一样，DEBUG模式可能是一段代码，非DEBUG模式可能是另外一段代码，开发者只需要改变常量DEBUG的值即可做到两种模式下的自由切换，岂不乐哉？这样配合编译器的条件编译机制又能使编译后的代码更为简洁，简直perfect！（通常这个常量可通过配置文件配置更为简便）

### 糖6：泛型

泛型可谓是个好东西了，在没有泛型时我们返回的方法值、参数等都在编写代码时就必须确定它的类型，这为我们带来了诸多不便，我们往往需要将这些参数和返回值设置为Object类型然后强转，但基于开发者的强转是极易出现问题的。而有了泛型后这些问题便都不是问题了

```java
public class SugarTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("111");
    }
}

```

编译后的代码

```java
public class SugarTest {
    public static void main(String[] args) {
        List list = new ArrayList();
        list.add("111");
    }
}

```

可以看出，泛型在编译期间便直接被扔掉了。泛型的实现机制是通过编译器的一种被称为类型擦除（type erasure）的处理实现的，也就是编译器不认识List这个类，它只认识并执行List这个类

类型擦除主要两个过程

*   将所有使用到泛型参数的地方都用其最顶级的层次（顶层父类）替换掉
*   将所有泛型擦除，即将尖括号<>删除

如下这个例子

```java
public class SugarTest {
    public static void main(String[] args) {
        String s = "1234";
        test(s);
    }

    public static <T extends Object> void test(T arg) {
        List<T> list = new ArrayList<>();
        list.add(arg);
    }
}

```

编译后的代码将所有泛型替换为Object型

```java
public class SugarTest {
    public static void main(String[] args) {
        String s = "1234";
        test((Object)s);
    }

    public static void test(Object arg) {
        List list = new ArrayList();
        list.add(arg);
    }
}

```

### 糖7：内部类

```java
class OutClass {
    class Node {
        public Integer num;
    }
}

public class SugarTest {
    public static void main(String[] args) {
        OutClass outClass = new OutClass();
        OutClass.Node node = outClass.new Node();
        node.num = 10;
    }
}

```

编译后事实上会产生两个后缀为.class的字节码文件，OutClass.class和OutClass$Node.class文件，由此可以看出内部类并不是真正套在一个类的内部，而是分成两个类编译

再看看静态内部类实际上是一样的

```java
class OutClass {
    static class Node {
        public Integer num;
    }
}

public class SugarTest {
    public static void main(String[] args) {
        OutClass.Node node = new OutClass.Node();
        node.num = 10;
    }
}

```

从上我们也能看出内部类和静态内部类的一些区别，内部类必须通过它的外部类的实例去new一个内部类，而静态内部类可以直接new一个内部类（但是要求内部类存在时外部类必定也存在）

### 糖8：assert断言

assert关键字是后来引入的，为了避免和老版本的Java代码引入assert关键字而出错，默认Java是不开启断言检查的，也就是说assert等于无用，如果要开启断言检查使用选项-ea或-enableassertions开启

【即java -ea SugarTest运行】

```java
public class SugarTest {
    public static void main(String[] args) {
        Integer num = 1;
        assert num != null : "Error!";
        System.out.println(num);
    }
}

```

编译后的代码

```java
public class SugarTest {
    public SugarTest() {
    }
    
    public static void main(String[] args) {
        Integer num = 1;
        if(!$assertionsDisabled && num == null) {
            throw new AssertionError("Error!");
        } else {
         	System.out.println(num);
            return;
        }
    }
    
    static final boolean $assertionsDisabled = !src/SugarTest.desiredAssertionStatus();
}

```

断言首先会检查$assertionsDisabled是否为true，如果为true说明断言被禁用了，直接执行else的内容，如果设置了开启断言则会判断断言后的表达式是否正确，不正确会抛出异常

### 糖9：数值字面量

允许在数字间插入任意数量的下划线，对数字的值不会产生任何影响，主要是方便阅读

【比如1\_0000\_0000可以直接看出是1亿】

```java
public class SugarTest {
    public static void main(String[] args) {
        Integer num = 1_0000_0000;
        System.out.println(num);
    }
}

```

编译后的代码

```java
public class SugarTest {
    public SugarTest() {
    }

    public static void main(String[] args) {
        Integer num = 100000000;
        System.out.println(num);
    }
}

```

### 糖10：Enum枚举类

枚举类让我们可以更方便的使用和定义一些常量，但在代码中实际上enum就是个关键字，看不出来枚举类到底是个什么神仙玩意，也看不出它是个什么对象

```java
public enum EnumTest {
    JAVA, PYTHON, JS;
}

```

编译后的代码

```java
public final class EnumTest extends Enum {

    public static final EnumTest JAVA;
    public static final EnumTest PYTHON;
    public static final EnumTest JS;
    private static final EnumTest $VALUES[];

    static {
        JAVA = new EnumTest("JAVA", 0);
        PYTHON = new EnumTest("PYTHON", 1);
        JS = new EnumTest("JS", 2);
        $VALUES = (new EnumTest[]{
                JAVA, PYTHON, JS
        });
    }
    private EnumTest(String s, int i) {
        super(s, i);
    }

    public static EnumTest[] values() {
        return (EnumTest[]) $VALUES.clone();
    }

    public static EnumTest valueOf(String name) {
        return (EnumTest) Enum.valueOf(EnumTest, name);
    }
}

```

可以看出，每一个枚举类中的一个常量都是一个EnumTest对象，这个对象有两个属性，一个是常量的名字，另一个是从0开始的数值字面量

所有变量定义为final连类也定义为final，说明枚举类的变量不能被修改该类也不能被继承，values方法返回的是数组的深拷贝，所以即使通过该方法获得了数组也无法修改数组中的值

枚举类的构造器是私有化的，所以用户定义的每一个变量返回给用户时都是同一个EnumTest对象，如果只在枚举类定义一个常量，那么这个枚举类就是个单例模式了

### 糖11：try-with-resource异常处理

相信大家都写过try-catch-finally代码，每次new一个流时一般都要在finally中将这个流close，如果流比较多，代码就会显得臃肿，那么这个语法糖就能让代码及其简洁

```java
public class SugarTest {
    public static void main(String[] args) {
        try (BufferedReader reader = new BufferedReader(
            							new FileReader("c:\\file\\test.txt"));
             BufferedWriter writer = new BufferedWriter(new PrintWriter(System.out))) {
            String str;
            while ((str = reader.readLine()) != null) {
                System.out.println(str);
            }
            writer.write("读取文件完毕");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

编译后代码

```java
public class SugarTest {
    public SugarTest() {
    }

    public static void main(String[] args) {
        try {
            BufferedReader reader = new BufferedReader(new FileReader("c:\\file\\test.txt"));
            Throwable var2 = null;

            try {
                BufferedWriter writer = new BufferedWriter(new PrintWriter(System.out));
                Throwable var4 = null;

                try {
                    String str;
                    while((str = reader.readLine()) != null) {
                        System.out.println(str);
                    }

                    writer.write("读取文件完毕");
                } catch (Throwable var29) {
                    var4 = var29;
                    throw var29;
                } finally {
                    if (writer != null) {
                        if (var4 != null) {
                            try {
                                writer.close();
                            } catch (Throwable var28) {
                                var4.addSuppressed(var28);
                            }
                        } else {
                            writer.close();
                        }
                    }

                }
            } catch (Throwable var31) {
                var2 = var31;
                throw var31;
            } finally {
                if (reader != null) {
                    if (var2 != null) {
                        try {
                            reader.close();
                        } catch (Throwable var27) {
                            var2.addSuppressed(var27);
                        }
                    } else {
                        reader.close();
                    }
                }

            }
        } catch (IOException var33) {
            var33.printStackTrace();
        }

    }
}

```

事实上这么简洁的代码最后编译器也只是把它转换成了try-catch-finally而已，代码故意采取两个流，目的是让读者清楚编译器会先对writer释放流再对reader释放流，也就是先写的流后释放

### 糖12：Lambda表达式

```java
public class SugarTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("JAVA");
        list.forEach((elem) -> System.out.println(elem));
    }
}

```

编译后的代码

```java
public class Lambda {
    public static void main(String[] arrstring) {
        List list = new ArrayList();
        list.add("JAVA");
        
        list.forEach((Consumer<String>)LambdaMetafactory.metafactory(null, null, null, (Ljava/lang/Object;)V, lambda$main$0(java.lang.String ), (Ljava/lang/String;)V)());
    }
    
    private static void lambda$main$0(String s) {
        System.out.println(s);
    }

}

```

可以看出编译后端Lambda语法是新增了一个方法，方法中是匿名内部类的方法体，然后调用LambdaMetafactory.metafactory这个方法执行
