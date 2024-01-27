# 探索Java8：(二)Function接口的使用
Java8 添加了一个新的特性Function，顾名思义这一定是一个函数式的操作。我们知道Java8的最大特性就是函数式接口。所有标注了`@FunctionalInterface`注解的接口都是函数式接口，具体来说，所有标注了该注解的接口都将能用在lambda表达式上。

标注了`@FunctionalInterface`的接口有很多，但此篇我们主要讲Function，了解了Function其他的操作也就很容易理解了。

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    
     * @return a composed function that first applies the {@code before}
     * function and then applies this function
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    
     * @return a composed function that first applies this function and then
     * applies the {@code after} function
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
}

```

为了方便地阅读源码，我们需要了解一些泛型的知识，如果你对泛型已经很熟悉了，那你可以跳过这段 。

泛型是JDK1.5引入的特性，通过泛型编程可以使编写的代码被很多不同的类型所共享，这可以很好的提高代码的重用性。因为本篇重点不是介绍泛型，所以我们只关注上述Function源码需要用到的泛型含义。

### 1\. 泛型类

泛型类使用`<T>`来表示该类为泛型类，其内部成员变量和函数的返回值都可以为泛型`<T>` ，Function源码的标识为`<T,R>`，也就是两个泛型参数，此处不再赘述，具体泛型类可以看网上的文章。

### 2\. 泛型方法和通配符

在方法修饰符的后面加一个`<T>`表明该方法为泛型方法，如Function 的源码里的compose方法的`<V>`。通配符也很好理解，还是compose的例子，我们可以看到compose的参数为一个Function类型，其中Functin的参数指定了其第一个参数必须是V的父类，第二个参数必须继承T，也就是T的子类。

源码解析
----

### 1.apply

讲完了上面这些就可以开始研究源码了。

首先我们已经知道了Function是一个泛型类，其中定义了两个泛型参数T和R，在Function中，T代表输入参数，R代表返回的结果。也许你很好奇，为什么跟别的java源码不一样，Function 的源码中并没有具体的逻辑呢？

其实这很容易理解，Function 就是一个函数，其作用类似于数学中函数的定义 ，（x,y）跟<T,R>的作用几乎一致。

y=f(x)y=f(x)

所以Function中没有具体的操作，具体的操作需要我们去为它指定，因此apply具体返回的结果取决于传入的lambda表达式。

```java
 R apply(T t);

```

举个例子：

```java
public void test(){
    Function<Integer,Integer> test=i->i+1;
    test.apply(5);
}


```

我们用lambda表达式定义了一个行为使得i自增1，我们使用参数5执行apply，最后返回6。这跟我们以前看待Java的眼光已经不同了，在函数式编程之前我们定义一组操作首先想到的是定义一个方法，然后指定传入参数，返回我们需要的结果。函数式编程的思想是先不去考虑具体的行为，而是先去考虑参数，具体的方法我们可以后续再设置。

再举个例子：

```java
public void test(){
    Function<Integer,Integer> test1=i->i+1;
    Function<Integer,Integer> test2=i->i*i;
    System.out.println(calculate(test1,5));
    System.out.println(calculate(test2,5));
}
public static Integer calculate(Function<Integer,Integer> test,Integer number){
    return test.apply(number);
}



```

我们通过传入不同的Function，实现了在同一个方法中实现不同的操作。在实际开发中这样可以大大减少很多重复的代码，比如我在实际项目中有个新增用户的功能，但是用户分为VIP和普通用户，且有两种不同的新增逻辑。那么此时我们就可以先写两种不同的逻辑。除此之外，这样还让逻辑与数据分离开来，我们可以实现**逻辑的复用**。

当然实际开发中的逻辑可能很复杂，比如两个方法F1,F2都需要两个个逻辑AB，但是F1需要A->B，F2方法需要B->A。这样的我们用刚才的方法也可以实现，源码如下：

```java
public void test(){
    Function<Integer,Integer> A=i->i+1;
    Function<Integer,Integer> B=i->i*i;
    System.out.println("F1:"+B.apply(A.apply(5)));
    System.out.println("F2:"+A.apply(B.apply(5)));
}




```

也很简单呢，但是这还不够复杂，假如我们F1,F2需要四个逻辑ABCD，那我们还这样写就会变得很麻烦了。

### 2.compose和andThen

compose和andThen可以解决我们的问题。先看compose的源码

```java
  default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

```

compose接收一个Function参数，返回时先用传入的逻辑执行apply，然后使用当前Function的apply。

```java
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

```

andThen跟compose正相反，先执行当前的逻辑，再执行传入的逻辑。

这样说可能不够直观，我可以换个说法给你看看

compose等价于B.apply(A.apply(5))，而andThen等价于A.apply(B.apply(5))。

```java
public void test(){
    Function<Integer,Integer> A=i->i+1;
    Function<Integer,Integer> B=i->i*i;
    System.out.println("F1:"+B.apply(A.apply(5)));
    System.out.println("F1:"+B.compose(A).apply(5));
    System.out.println("F2:"+A.apply(B.apply(5)));
    System.out.println("F2:"+B.andThen(A).apply(5));
}





```

我们可以看到上述两个方法的返回值都是一个Function，这样我们就可以使用建造者模式的操作来使用。

```java
B.compose(A).cpmpose(A).andThen(A).apply(5);

```

