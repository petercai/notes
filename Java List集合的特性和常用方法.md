# Java List集合的特性和常用方法

**\- 2023.11.14 -**

> 通过前面文章的介绍，相信大家对Java集合框架有了简单的理解，接下来说说集合中最常使用的一个集合类的父类，List 集合。那么，List到底是什么？它有哪些特性？又该如何使用呢？让我们一起来揭开List的神秘面纱。

List，顾名思义，就是列表的意思。在Java中，List是一个接口，它继承了Collection接口，表示一个有序的、可重复的元素集合。下面我们从List 接口的概念、特点和常用方法等方面来介绍List。

java.util.List 接口，继承自 Collection 接口(可以回看咱们第二篇中的框架体系），List 接口是单列集合的一个重要分支，习惯性地将实现了List 接口的对象称为List集合。 ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7520b036dafb498795d8b0fcffe4c82a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1043&h=500&s=239738&e=png&b=faea5c)

在list 集合中允许出现重复的元素，所有的元素对应一个整数型的序号记载其在容器中的位置进行存储，在程序中可以通过索引来访问集合中的指定元素。另外，List集合还是 有序的，即元素的存入和取出顺序一致。

### List 接口的特点：

*   它是一个元素存取有序的集合。例如，存元素的顺序是3,45,6。那么集合中，元素的存储就是按照3,45,6的顺序完成的）。
    
*   它是一个带有索引的集合，通过索引就可以精确的操作集合中的元素（与数组的索引是一个道理）。
    
*   可以有重复的元素，通过元素的equals方法，来比较是否为重复的元素。
    

### List接口中常用方法：

List作为Collection集合的子接口，不但继承了Collection接口中的全部方法，而且还增加了一些根据元素索引来操作集合的特有方法，如下：

*   public void add(int index, E element)：将指定的元素，添加到该集合中的指定位置上。
    
*   public E get(int index)：返回集合中指定位置的元素。
    
*   public E remove(int index)：移除列表中指定位置的元素, 返回的是被移除的元素。
    
*   public E set(int index, E element)：用指定元素替换集合中指定位置的元素,返回值的更新前的元素。
    

> 编程学习，从[云端源想](https://link.juejin.cn/?target=https%3A%2F%2Fwww.ydcode.cn%2F%3FsourceId%3D240 "https://www.ydcode.cn/?sourceId=240")开始，课程视频、在线书籍、在线编程、一对一咨询……你想要的全部学习资源这里都有，重点是统统免费！可以一试哦！

通过代码来体验一下：

```java
    public class Demo1List {
    public static void main(String[] args) {
       List<String> names = new ArrayList<>();
       
        names.add("张三");
        names.add("李四");
        names.add("王五");
        System.out.println("集合---"+names);
       
      
        names.add(1, "赵六");
        System.out.println("集合---" + names);
      
      
        String name = names.get(3);
        System.out.println("元素---" + name);
      
      
        String remove = names.remove(2);
        System.out.println("移除后的集合" + names);
    
    
    
      
      
        names.set(1, "张二狗");
        System.out.println("集合set方法---" + names);
    }
}

```

List接口有很多实现类，如ArrayList、LinkedList等，它们各自有着不同的特点和应用场景。下面分别来介绍一下常用的ArrayList 集合和LinkedList集合。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d6b8a1c5a4b42ef8360f81b79bda67f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=619&h=314&s=51378&e=png&b=f7f7f7)

ArrayList 集合
------------

通过 javaApi 帮助文档 ，可以看到 List的实现类其实挺多，在此选择比较常见的 `ArrayList` 和 `LinkedList` 简单介绍。

**ArrayList 有以下两个特点：** 

*   底层的数据结构是一个数组；
    
*   这个数组会自动扩容，看起来像一个长度可变的数组。
    
*   通过阅读源码的方式，简单分析下这两个特点的实现：
    

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ccb2a0ada874d14811b9e295df02cd2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=875&h=845&s=168667&e=png&b=292d35)

在实例化ArrayList时，调用了对象的无参构造器，在无参构造器中，首先看到变量 elementData 的定义就是一个数组类型，它存储的就是集合中的元素，其次在初始化对象时，把一个长度为0的Object\[\] 数组，赋值给了 elementData 。这就是刚刚所说的 **ArrayList 底层是一个数组**。

下面再来看自动扩容这个特点又是怎么实现的。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/938e5569e60149be8ed82b28cf252158~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=849&h=898&s=193947&e=png&b=292d35)

在向集合中添加一个元素之前，会计算集合中数组的长度是否满足，可以通过代码追踪，通过一系列方法的调用，会使用 arrays 工具类的复制方法 （根据文档，介绍复制方法）创建一个新的长度的数组，将添加的元素保存进去，这就是说的**数组可变，自动扩容**。

ArrayList的两个特点就介绍到这里了，大家有兴趣的可以去读读源码，挺有意思。

#### 重点说明：

之前讲过，数组结构的特点是元素增删慢，查找快。由于java.util.ArrayList 集合数据存储的结构是数组结构，所以它的特点也是元素增删慢，但是查询快。

由于日常开发中使用最多的功能为查询数据、遍历数据，所以ArrayList 也是最常使用的集合。

而因着这些特点呢，在日常开发中，有些开发人员就非常随意地使用ArrayList完成任何需求，这是不严谨，这种编码方式也是不提倡的。

接着来看看下面这个实现类：java.util.LinkedList 集合数据存储的结构是链表结构。方便元素添加、删除的集合。

LinkedList是一个双向链表，那么双向链表是什么样子的呢，我上篇文章说过的结构图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8448db56e8bf4da794a99a18e0627c17~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=222&s=26582&e=png&b=ffffff)

inkedList 是由链表来说实现的，并且它实现了List接口的所有方法，还增加了一些自己特有的方法。

那么之前介绍过双向链表的特点，所以LinkedList的特点就是：**元素添加，删除速度快，而查询速度慢。** 

### 常用方法

LinkedList 作为 List的实现类，List中的方法LinkedList都是可以使用，所以这些方法就不做详细介绍；而特别练习一下 linkedList 提供的特有方法，因为在实际开发中对一个集合元素的添加与删除也经常涉及到首尾操作。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd13c4a44b1043fbb334a07f2a2b0108~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=526&s=408924&e=png&b=d5f1f3)

下面看下演示代码：

```java
 public class Demo2LinkedList {
   
        public static void main(String[] args) {
            LinkedList<String> link = new LinkedList<String>();
            
            link.addFirst("abc1");
            link.addFirst("abc2");
            link.addFirst("abc3");
            System.out.println(link);
            
            System.out.println(link.getFirst());
            System.out.println(link.getLast());
            
            System.out.println(link.removeFirst());
            System.out.println(link.removeLast());
    
            while (!link.isEmpty()) { 
                System.out.println(link.pop()); 
            }
    
            System.out.println(link);
        }
    }

```

虽然List功能强大，但我们也不能滥用。在使用时，我们需要注意以下几点：

*   尽量避免频繁的插入和删除操作，因为这会影响List的性能。在这种情况下，我们可以考虑使用LinkedList。
    
*   List的大小是有限的，当元素超过List的最大容量时，会抛出OutOfMemoryError异常。因此，我们需要合理地设置List的初始容量和最大容量。
    

总的来说，Java单列集合List是一个非常强大的工具，它可以帮助我们解决很多编程问题。只要我们能够正确地使用它，就能够在编程的世界中找到无尽的乐趣。
