# List集合和其子类ArrayList、LinkedList_Java_共饮一杯无_InfoQ写作社区
我们掌握了 Collection 接口的使用后，再来看看 Collection 接口中的子类，他们都具备那些特性呢？

接下来，我们一起学习 Collection 中的常用几个子类（`java.util.List`集合、`java.util.Set`集合）。

🍡List 接口介绍
-----------

`java.util.List`接口继承自`Collection`接口，是单列集合的一个重要分支，习惯性地会将实现了`List`接口的对象称为 List 集合。在 List 集合中允许出现重复的元素，所有的元素是以一种线性方式进行存储的，在程序中可以通过索引来访问集合中的指定元素。另外，List 集合还有一个特点就是**元素有序，即元素的存入顺序和取出顺序一致。** 

List 接口特点：

1.  它是一个元素存取有序的集合。例如，存元素的顺序是 11、22、33。那么集合中，元素的存储就是按照 11、22、33 的顺序完成的）。
    
2.  它是一个带有索引的集合，通过索引就可以精确的操作集合中的元素（与数组的索引是一个道理）。
    
3.  集合中可以有重复的元素，通过元素的 equals 方法，来比较是否为重复的元素。
    

> tips:List 接口的子类 java.util.ArrayList 类，该类中的方法都是来自 List 中定义。

🥟List 接口中常用方法
--------------

List 作为 Collection 集合的子接口，不但继承了 Collection 接口中的全部方法，而且还增加了一些根据元素索引来操作集合的特有方法，如下：

*   `public void add(int index, E element)`: 将指定的元素，添加到该集合中的指定位置上。
    
*   `public E get(int index)`:返回集合中指定位置的元素。
    
*   `public E remove(int index)`: 移除列表中指定位置的元素, 返回的是被移除的元素。
    
*   `public E set(int index, E element)`:用指定元素替换集合中指定位置的元素,返回值的更新前的元素。
    

List 集合特有的方法都是跟索引相关，我们再来复习一遍吧：

```
public class ListDemo {
    public static void main(String[] args) {
    
      List<String> list = new ArrayList<String>();
      
      
      list.add("张三");
      list.add("李四");
      list.add("王五");
      
      System.out.println(list);
      
      list.add(1,"赵六");
      
      System.out.println(list);
      
      
      System.out.println("删除索引位置为2的元素");
      System.out.println(list.remove(2));
      
      System.out.println(list);
      
      
      
      
      list.set(0, "三毛");
      System.out.println(list);
      
      
      
      
      for(int i = 0;i<list.size();i++){
        System.out.println(list.get(i));
      }
      
      for (String string : list) {
      System.out.println(string);
    }    
  }
}
```

🥠List 的子类
----------

### 🥡ArrayList 集合

`java.util.ArrayList`集合数据存储的结构是数组结构。元素增删慢，查找快，由于日常开发中使用最多的功能为查询数据、遍历数据，所以`ArrayList`是最常用的集合。许多程序员开发时非常随意地使用 ArrayList 完成任何需求，并不严谨，这种用法是不提倡的。

### 🦪LinkedList 集合

`java.util.LinkedList`集合数据存储的结构是链表结构。方便元素添加、删除的集合。

> LinkedList 是一个双向链表，那么双向链表是什么样子的呢，我们用个图了解下

![](https://static001.geekbang.org/infoq/9d/9d738bc52c658f4ae9d757f06c330b59.png)

实际开发中对一个集合元素的添加与删除经常涉及到首尾操作，而 LinkedList 提供了大量首尾操作的方法。这些方法我们作为了解即可：

*   `public void addFirst(E e)`:将指定元素插入此列表的开头。
    
*   `public void addLast(E e)`:将指定元素添加到此列表的结尾。
    
*   `public E getFirst()`:返回此列表的第一个元素。
    
*   `public E getLast()`:返回此列表的最后一个元素。
    
*   `public E removeFirst()`:移除并返回此列表的第一个元素。
    
*   `public E removeLast()`:移除并返回此列表的最后一个元素。
    
*   `public E pop()`:从此列表所表示的堆栈处弹出一个元素。
    
*   `public void push(E e)`:将元素推入此列表所表示的堆栈。
    
*   `public boolean isEmpty()`：如果列表不包含元素，则返回 true。
    

LinkedList 是 List 的子类，List 中的方法 LinkedList 都是可以使用，这里就不做详细介绍，我们只需要了解 LinkedList 的特有方法即可。在开发时，LinkedList 集合也可以作为堆栈，队列的结构使用。

方法演示：

```
public class LinkedListDemo {
    public static void main(String[] args) {
        LinkedList<String> link = new LinkedList<String>();
        
        link.addFirst("zjq");
        link.addFirst("zjq666");
        link.addFirst("zjq888");
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

> 本文内容到此结束了，
> 
> 如有收获欢迎点赞👍收藏💖关注✔️，您的鼓励是我最大的动力。
> 
> 如有错误❌疑问💬欢迎各位大佬指出。
> 
> **主页**：[共饮一杯无的博客汇总👨‍💻](https://www.infoq.cn/u/zhanjq/publish)  
> 
> **保持热爱，奔赴下一场山海**。🏃🏃🏃