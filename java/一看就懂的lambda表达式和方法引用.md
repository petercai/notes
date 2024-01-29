# 一看就懂的lambda表达式和方法引用
Lambda 表达式是 Java 8 引入的一项重要特性，它提供了一种简洁、清晰且灵活的语法来表示可传递的匿名函数。Lambda 表达式是一种语法糖，提供了一种更简洁的语法来实例化函数式接口（Functional Interface）。

### 基本语法：

Lambda 表达式的基本语法如下：

```java
(parameters) -> expression

```

或

```java
(parameters) -> { statements; }

```

其中：

*   `parameters` 指的是 Lambda 表达式的参数列表。
*   `->` 是 Lambda 运算符，它将参数列表与 Lambda 表达式的主体分开。
*   `expression` 是 Lambda 表达式的返回表达式。
*   `{}` 内包含了 Lambda 表达式的代码块，可以包含多个语句。

### 示例：

以下是一些 Lambda 表达式的示例：

1.  不带参数的 Lambda 表达式：
    
    ```java
    () -> System.out.println("Hello, Lambda!");
    
    ```
    
2.  带参数的 Lambda 表达式：
    
    ```java
    (int x, int y) -> x + y
    
    ```
    
3.  带参数和代码块的 Lambda 表达式：
    
    ```java
    (String s) -> {
        System.out.println(s);
        
    }
    
    ```
    

### 使用场景：

Lambda 表达式通常用于函数式接口的实例化，即接口中只有一个抽象方法的情况。例如，`Runnable`、`Callable`、`Comparator` 等接口都是函数式接口，可以通过 Lambda 表达式来实例化。

```java

Runnable myRunnable = () -> System.out.println("Hello, Lambda!");


Comparator<String> myComparator = (s1, s2) -> s1.compareTo(s2);

```

### 本质：

虽然 Lambda 表达式的语法看起来像是一个函数，但实际上它在 Java 中被实现为一个对象，是一个匿名的内部类对象。这个对象的类型是由编译器根据上下文和目标类型进行推导的。这个对象可以被赋值给相应的函数式接口。 在讲Consumer接口时，我们的一个例子。

```java
import java.util.function.Consumer;

public class ConsumerExample {

    public static void main(String[] args) {
        
        Consumer<String> myConsumer = s -> System.out.println(s);

        
        myConsumer.accept("Hello, Consumer!");
    }
}

```

首先`s -> System.out.println(s)`是lambda表达式，本质是一个只含有一个方法的匿名内部类对象。它的演化流程是,定义一个函数式接口

```java
interface MyFunctionalInterface {
    void myMethod(String s);
}

```

然后使用匿名内部类（这个内部类实现这个函数式接口，编写抽象方法的逻辑），并实例化这个对象，由于java的多态特点，父类引用可以指向子类对象，因此父类引用`MyFunctionalInterface`可以指向新的接口实例化后的对象。

```java
MyFunctionalInterface myFunction = new MyFunctionalInterface() {
    @Override
    public void myMethod(String s) {
        
        System.out.println(s);
    }
};

```

这样的写法弊端是语法冗长，代码显得不够简洁。Lambda 表达式的引入就是为了解决这个问题，让代码更加紧凑和易读。使用 Lambda 表达式，上面的代码可以被简化为：

```java
MyFunctionalInterface myFunction =（String s）-> System.out.println(s);

```

当 Lambda 表达式只有一个参数时，可以省略参数的括号。而且当上下文可以**推断**出类型时，参数的类型也可以省略。最终变为：

```java
 Consumer<String> myConsumer = s -> System.out.println(s);

```

用`java`泛型来推断`s`的类型为`String`。

`Consumer`本身就是一个函数式接口，所以此处`System.out.println(s)`就是Consumer中抽象方法`accept`的实现逻辑。示例中的`MyFunctionalInterface`其实就是`Consumer`，或者经过编译器推断，这个函数式接口就是`Consumer`。

### 方法引用：

Lambda 表达式的一个变体是方法引用，它允许你直接引用已有的方法。例如：

```java

Function<String, Integer> toInt = Integer::parseInt;


List<String> list = Arrays.asList("1", "2", "3");
list.forEach(System.out::println);

```

这里 `Integer::parseInt` 是一个静态方法引用，`System.out::println` 是一个实例方法引用。

有四种主要的方法引用的形式：

1.  **静态方法引用（Static Method Reference）：**  `ClassName::staticMethodName`
    
2.  **实例方法引用（Instance Method Reference）：** 
    
    *   对于特定对象的实例方法：`instance::instanceMethodName`
    *   对于任意对象的实例方法：`ClassName::instanceMethodName`
3.  **构造方法引用（Constructor Reference）：**  `ClassName::new`
    
4.  **数组构造方法引用（Array Constructor Reference）：**  `TypeName[]::new`
    

### Lambda 表达式的优势：

1.  **简洁性：**  Lambda 表达式可以减少冗余的代码，使代码更加简洁和易读。
    
2.  **函数式编程：**  提供了函数式编程的支持，使得在Java中更容易使用函数作为一等公民。
    
3.  **并行处理：**  可以更容易地实现并行处理，通过并行流等特性，提高程序的性能。
    

Lambda 表达式是 Java 8 引入的一项重要特性，它在编写简洁、清晰的代码和支持函数式编程方面起到了关键作用。方法引用是Lambda 表达式的变体。