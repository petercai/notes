# Set集合和其之类HashSet、LinkedHashSet_Java_共饮一杯无_InfoQ写作社区
`java.util.Set`接口和`java.util.List`接口一样，同样继承自`Collection`接口，它与`Collection`接口中的方法基本一致，并没有对`Collection`接口进行功能上的扩充，只是比`Collection`接口更加严格了。与`List`接口不同的是，`Set`接口中元素无序，并且都会以某种规则保证存入的元素不出现重复。`Set`集合有多个子类，这里我们介绍其中的`java.util.HashSet`、`java.util.LinkedHashSet`这两个集合。

> 💡tips:Set 集合取出元素的方式可以采用：迭代器、增强 for。

🥩HashSet 集合介绍
--------------

`java.util.HashSet`是`Set`接口的一个实现类，它所存储的元素是不可重复的，并且元素都是无序的(即存取顺序不一致)。`java.util.HashSet`底层的实现其实是一个`java.util.HashMap`支持。

`HashSet`是根据对象的哈希值来确定元素在集合中的存储位置，因此具有良好的存取和查找性能。保证元素唯一性的方式依赖于：`hashCode`与`equals`方法。

我们先来使用一下 Set 集合存储，看下现象，再进行原理的讲解:

```
public class HashSetDemo {
    public static void main(String[] args) {
        
        HashSet<String>  set = new HashSet<String>();

        
        set.add(new String("cba"));
        set.add("abc");
        set.add("bac"); 
        set.add("cba");  
        
        for (String name : set) {
            System.out.println(name);
        }
    }
}
```

输出结果如下，说明集合中不能存储重复元素：

> 💡tips:根据结果我们发现字符串"cba"只存储了一个，也就是说重复的元素 set 集合不存储。

🥓HashSet 集合存储数据的结构（哈希表）
------------------------

### 🍔什么是哈希表呢？

在 **JDK1.8** 之前，哈希表底层采用数组+链表实现，即使用链表处理冲突，同一 hash 值的链表都存储在一个链表里。但是当位于一个桶中的元素较多，即 hash 值相等的元素较多时，通过 key 值依次查找的效率较低。而 JDK1.8 中，哈希表存储采用数组+链表+红黑树实现，当链表长度超过阈值（8）时，将链表转换为红黑树，这样大大减少了查找时间。

简单的来说，哈希表是由数组+链表+红黑树（JDK1.8 增加了红黑树部分）实现的，如下图所示。

![](_assets/e9ebae560ca0725166f59ede7c2545ba.png)

看到这张图就有人要问了，这个是怎么存储的呢？

为了方便大家的理解我们结合一个存储流程图来说明一下：

![](_assets/392bb0c9e681748cf5be1807ab01ce06.png)

总而言之，**JDK1.8** 引入红黑树大程度优化了 HashMap 的性能，那么对于我们来讲保证 HashSet 集合元素的唯一，其实就是根据对象的 hashCode 和 equals 方法来决定的。如果我们往集合中存放自定义的对象，那么保证其唯一，就必须复写 hashCode 和 equals 方法建立属于当前对象的比较方式。

🍟HashSet 存储自定义类型元素
-------------------

给 HashSet 中存放自定义类型元素时，需要重写对象中的 hashCode 和 equals 方法，建立自己的比较方式，才能保证 HashSet 集合中的对象唯一。

创建自定义 Student 类

```
public class Student {
    private String name;
    private int age;

    public Student() {
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;
        Student student = (Student) o;
        return age == student.age &&
               Objects.equals(name, student.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

```
public class HashSetDemo2 {
    public static void main(String[] args) {
        
        HashSet<Student> stuSet = new HashSet<Student>();
        
        Student stu = new Student("张三", 43);
        stuSet.add(stu);
        stuSet.add(new Student("李四", 66));
        stuSet.add(new Student("张三", 43));
        stuSet.add(new Student("王五", 23));
        stuSet.add(stu);

        for (Student stu2 : stuSet) {
            System.out.println(stu2);
        }
    }
}
执行结果：
Student [name=李四, age=66]
Student [name=张三, age=43]
Student [name=王五, age=23]
```

🍕LinkedHashSet
---------------

我们知道 HashSet 保证元素唯一，可是元素存放进去是没有顺序的，那么我们要保证有序，怎么办呢？

在 HashSet 下面有一个子类`java.util.LinkedHashSet`，它是链表和哈希表组合的一个数据存储结构。

演示代码如下:

```
public class LinkedHashSetDemo {
  public static void main(String[] args) {
    Set<String> set = new LinkedHashSet<String>();
    set.add("bbb");
    set.add("aaa");
    set.add("abc");
    set.add("bbc");
        Iterator<String> it = set.iterator();
    while (it.hasNext()) {
      System.out.println(it.next());
    }
  }
}
结果：
  bbb
  aaa
  abc
  bbc
```

> 本文内容到此结束了，
> 
> 如有收获欢迎点赞👍收藏💖关注✔️，您的鼓励是我最大的动力。
> 
> 如有错误❌疑问💬欢迎各位大佬指出。
> 
> **主页**：[共饮一杯无的博客汇总👨‍💻](https://www.infoq.cn/u/zhanjq/publish)  
> 
> **保持热爱，奔赴下一场山海**。🏃🏃🏃