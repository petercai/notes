# 在Java中灵活使用迭代器iterator，高效完成各类数据遍历

迭代器是Java集合框架中的一种重要的设计模式，它提供了一种顺序访问集合中的元素的方法，而且不需要暴露集合内部的细节。

本文将会介绍Java中的迭代器用法，包括它的使用方法、应用场景、优缺点分析等方面。

简介
--

在Java中，迭代器的实现是通过实现java.util.Iterator接口来实现的。Iterator接口提供了三个主要的方法：hasNext()，next()和remove()。

*   hasNext()：检查集合中是否还有下一个元素。
*   next()：返回集合中的下一个元素。
*   remove()：从集合中移除上一个被next()方法返回的元素。

通过使用Iterator接口提供的这些方法，我们可以很方便地遍历集合中的各个元素。

源代码解析
-----

下面是一个使用Iterator遍历ArrayList的示例代码：

```java
package com.example.javase.se.classes;

import java.util.ArrayList;
import java.util.Iterator;


 * @Author ms
 * @Date 2023-11-05 18:50
 */
public class ArrayListExample {

    public static void main(String[] args) {
        
        ArrayList<String> arrayList = new ArrayList<>();
        arrayList.add("apple");
        arrayList.add("banana");
        arrayList.add("orange");

        
        Iterator<String> iterator = arrayList.iterator();
        while (iterator.hasNext()) {
            String element = iterator.next();
            System.out.println(element);
        }
    }
}

```

上述代码中，我们首先创建了一个ArrayList对象，并添加了三个元素。然后，我们使用iterator()方法获取ArrayList的迭代器，最后使用while循环遍历集合中的所有元素。具体实现步骤如下：

该程序定义了一个名为ArrayListExample的类，其中包含了一个静态的main()方法用于执行程序。程序导入了java.util包中的ArrayList和Iterator类。

在main()方法中，程序创建了一个ArrayList对象，并向这个对象添加了三个字符串元素。接着，程序使用迭代器Iterator遍历了ArrayList，并打印了各个元素的值。

因此，最终输出结果为：

> apple
> 
> banana
> 
> orange

应用场景案例
------

迭代器最基本的用途就是遍历集合中的元素。除了遍历，它还有其它的使用场景，例如在特定条件下，删除集合中的元素等。

下面是一个使用迭代器删除ArrayList中指定元素的示例代码：

```java
import java.util.ArrayList;
import java.util.Iterator;

public class ArrayListExample {
    public static void main(String[] args) {
        
        ArrayList<String> arrayList = new ArrayList<>();
        arrayList.add("apple");
        arrayList.add("banana");
        arrayList.add("orange");
        arrayList.add("pear");

        
        Iterator<String> iterator = arrayList.iterator();
        while (iterator.hasNext()) {
            String element = iterator.next();
            if (element.equals("banana")) {
                iterator.remove();
            }
        }

        
        System.out.println(arrayList);
    }
}

```

上述代码中，我们创建了一个包含四个元素的ArrayList对象，然后使用迭代器遍历集合中的所有元素，如果元素的值等于"banana"，则调用Iterator的remove()方法删除该元素。最后，我们输出剩余的元素。

### 测试结果

  根据如上测试用例，本地测试结果如下，仅供参考，你们也可以自行修改测试用例或者添加更多的测试数据或测试方法，进行熟练学习以此加深理解。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/830abb7d433e4b6b90cfea76bda32144~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1135&h=821&s=66746&e=png&b=2c2c2c)

### 测试代码分析

  这是一个Java的类文件，主要演示了如何使用ArrayList和Iterator来删除ArrayList中指定的元素。具体分析如下：

1.  首先导入了java.util包中的ArrayList和Iterator类。
    
2.  在main函数中创建了一个ArrayList对象arrayList，并向其中添加了四个字符串元素。
    
3.  接着使用迭代器Iterator遍历ArrayList中的元素。在遍历的过程中，通过if语句判断当前元素是否为“banana”，如果是，则使用iterator的remove()方法将该元素从ArrayList中删除。
    
4.  最后输出ArrayList中剩余的元素。
    

通过这个例子，可以看出使用迭代器Iterator可以方便地删除ArrayList中指定的元素。

优缺点分析
-----

使用迭代器遍历集合的优点在于，它可以避免我们在遍历集合时，使用传统的for循环方式造成的角标越界等问题。此外，迭代器使得代码更易于阅读和理解。

然而，使用迭代器遍历大型的集合时，可能会影响性能。此时，使用传统的for循环方式会更加高效。

类代码方法介绍
-------

在上述示例代码中，我们使用了如下方法：

*   ArrayList.add()：向ArrayList中添加元素。
*   Iterator.hasNext()：检查集合中是否还有下一个元素。
*   Iterator.next()：返回集合中的下一个元素。
*   Iterator.remove()：从集合中移除上一个被next()方法返回的元素。

测试用例
----

### 测试代码演示

为了保证迭代器的正确性，我们需要对其进行单元测试。下面是一个简单的单元测试示例：

```java
public class ArrayListTest {
    public static void main(String[] args) {
        ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            arrayList.add(i);
        }

        Iterator<Integer> iterator = arrayList.iterator();
        int i = 0;
        while (iterator.hasNext()) {
            int element = iterator.next();
            if ((Integer)i++ != (Integer)element) {
                System.out.println("test failed");
                return;
            }
        }
        System.out.println("test passed");
    }
}

```

上述代码中，我们创建了一个包含10个元素的ArrayList对象，并使用迭代器遍历集合中的所有元素。为了保证迭代器的正确性，我们使用了JUnit框架进行单元测试，并对每个元素进行了断言验证。

### 测试结果

  根据如上测试用例，本地测试结果如下，仅供参考，你们也可以自行修改测试用例或者添加更多的测试数据或测试方法，进行熟练学习以此加深理解。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c213509efb274cd2926642245ba1f08e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1010&h=689&s=54598&e=png&b=2c2c2c)



全文小结
迭代器是Java开发中非常常见的一种设计模式，它不仅可以用于遍历集合中的元素，还可以用于在特定条件下删除集合中的元素等。
