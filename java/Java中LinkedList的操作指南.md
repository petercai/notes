# Java中LinkedList的操作指南 
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1f590f703eb494eb6e158fdd9de828c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=7019&h=4963&s=1255864&e=png&b=ffffff)

* * *

  在Java开发中，LinkedList是一个非常常见的数据结构，常用于实现栈、队列等数据结构。本文将从Java中LinkedList的基本概念和操作开始，逐步深入，介绍Linkedlist的源代码解析、应用场景案例、优缺点分析以及类代码方法介绍等内容，最后给出测试用例和全文小结。

  本文将介绍Java中LinkedList的基础知识，包括数据结构定义、基本操作、源代码解析等；随后将介绍LinkedList的应用场景案例、优缺点分析以及类代码方法介绍等内容。本文将帮助读者全面了解LinkedList，在实际开发中灵活运用，提升代码效率和质量。

概述
--

  LinkedList属于Java中的集合，是一种线性结构，可以存储不同类型的元素，并且可以动态改变元素数量。LinkedList采用链表的数据结构实现，它的每个节点都保存了下一个节点的内存地址，因此可以实现动态添加、删除和查找等操作。在实际开发中，LinkedList被广泛应用于栈、队列等数据结构的实现；同时也可以用于缓存、列表等场景中。

源代码解析
-----

  LinkedList是Java中的一个双向链表实现的集合类，它实现了List和Deque接口，提供了插入、删除、查找等操作方法。接下来我们来分析一下LinkedList的源码。

### 1\. 定义

LinkedList的源码位于java.util包下，其定义如下：

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable {
        
    transient int size = 0;
    
    transient Node<E> first;
    
    transient Node<E> last;

    
}

```

  可以看到LinkedList定义了三个成员变量：size表示链表的大小，first表示第一个节点，last表示最后一个节点，这三个变量都是transient类型的，表示不会被序列化。由于LinkedList实现了List接口，因此还需要实现List接口中的抽象方法，这里就不一一列出了。

### 2\. 节点类Node

  LinkedList中的元素都被封装成Node对象，其定义如下：

```java
private static class Node<E> {
    
    E item;
    
    Node<E> next;
    
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}

```

  每个节点包含三个属性：item表示节点存储的元素，next表示下一个节点，prev表示上一个节点。节点类使用了private修饰符，表示只能在LinkedList内部访问。

### 3\. add方法

  add方法是LinkedList中最基本的方法之一，用于在链表尾部添加一个元素，其源码如下：

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
}

```

  可以看到，linkLast方法是用于在链表尾部插入节点的，它首先得到链表最后一个节点l，然后创建一个新的节点newNode，并将之前的last节点的next指向newNode。如果链表为空，newNode就成为了第一个节点，否则就将newNode连接在l之后。最后，链表的长度加1。

### 4\. get方法

  get方法用于获取链表中指定位置的元素，其源码如下：

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}

Node<E> node(int index) {
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

```

  可以看到，get方法首先调用了checkElementIndex方法，用于检查index是否越界。然后调用node方法，获取指定位置的节点。node方法根据index的值，选择从头部或尾部开始遍历链表，找到目标节点并返回其元素值。

### 5\. remove方法

  remove方法用于从链表中删除指定位置的元素，其源码如下：

```java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}

E unlink(Node<E> x) {
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    return element;
}

```

  可以看到，remove方法首先调用了checkElementIndex方法，用于检查index是否越界。然后调用unlink方法，用于删除指定节点。unlink方法首先保存了目标节点的前一个节点和后一个节点，然后将它们连接起来，去除目标节点的引用，最后返回目标节点的元素值。

  以上是LinkedList的源码分析，通过了解LinkedList的实现原理，我们可以更好地利用它来实现一些需求。

  如下是部分源码截图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7adf92f6c7c14930887e70bf4d432258~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1255&h=946&s=99671&e=png&b=2b2b2b)

### 存储结构

  LinkedList采用链表的数据结构实现，将每个元素封装成一个Node节点，每个节点都有两个属性：元素值和指向下一个节点的指针。因此，每个节点在内存中都是相互独立的，可以独立增删，设计上也更灵活。

### 基本操作

  Linkedlist提供了一系列基本操作，包括添加元素、删除元素、查找元素、获取元素等。其中较为常见的操作有以下几种：

*   add(E e)：在LinkedList的末尾添加一个元素。
*   addFirst(E e)：在LinkedList的开头添加一个元素。
*   addLast(E e)：在LinkedList的末尾添加一个元素。
*   remove()：删除LinkedList中的第一个元素。
*   remove(Object o)：删除LinkedList中指定的元素。
*   removeFirst()：删除LinkedList中的第一个元素。
*   removeLast()：删除LinkedList中的最后一个元素。
*   size()：获取LinkedList的元素数量。
*   get(int index)：根据下标获取LinkedList中指定的元素。
*   set(int index, E element)：替换LinkedList中指定下标的元素。

  如下是部分源码截图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42414ebe8fe14e43831c9ed3e8a54d7e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1247&h=676&s=80858&e=png&b=2c2c2c)

应用场景案例
------

LinkedList的应用场景非常丰富，主要包括以下几种：

*   实现栈和队列：LinkedList可以作为栈和队列的底层数据结构，实现入栈、出栈、入队、出队等功能。
*   缓存：如果需要存储大量的数据，并且需要快速访问最近使用的数据，可以使用LinkedList实现缓存，将最近访问的数据放在LinkedList的头部，当缓存已满时，将最久未使用的数据删除。
*   列表：LinkedList可以用来存储和操作列表数据，如添加、删除和移动元素等。
*   循环链表：LinkedList可以实现循环链表，即最后一个节点指向第一个节点，可以实现循环遍历和处理操作。

优缺点分析
-----

LinkedList的优点如下：

*   可以动态添加、删除元素，在元素数量未知或者动态变化的情况下使用更为灵活。
*   添加、删除元素时，不需要移动其他元素，操作效率较高。
*   可以存储不同类型的元素，具有较高的灵活性。

LinkedList的缺点如下：

*   查找、访问LinkedList中的元素时，需要遍历LinkedList，效率较低。
*   需要额外的内存空间来存储节点的指针信息。

类代码方法介绍
-------

LinkedList类的主要方法如下：

```java
public boolean add(E e);              
public void add(int index, E element);
public void addFirst(E e);           
public void addLast(E e);            
public void clear();                 
public Object clone();               
public boolean contains(Object o);   
public E get(int index);             
public E getFirst();                 
public E getLast();                  
public int indexOf(Object o);        
public boolean isEmpty();            
public Iterator<E> iterator();       
public int lastIndexOf(Object o);    
public ListIterator<E> listIterator();
public boolean remove(Object o);     
public E remove(int index);          
public E removeFirst();              
public boolean removeFirstOccurrence(Object o);
public E removeLast();               
public boolean removeLastOccurrence(Object o);
public int size();                   
public Object[] toArray();           

```

### 代码分析

  LinkedList是Java集合框架中的一种双向链表实现的列表，支持快速的增删改查操作。其常用方法包括：

1.  add(E e)：在列表末尾添加元素，返回是否添加成功。
2.  add(int index, E element)：在指定位置插入元素，返回是否插入成功。
3.  addFirst(E e)：在列表开头插入元素。
4.  addLast(E e)：在列表末尾插入元素。
5.  clear()：清空列表中的所有元素。
6.  clone()：克隆一个新的LinkedList。
7.  contains(Object o)：判断列表中是否包含指定元素。
8.  get(int index)：获取指定位置的元素。
9.  getFirst()：获取列表中的第一个元素。
10.  getLast()：获取列表中的最后一个元素。
11.  indexOf(Object o)：返回指定元素在列表中的首次出现位置的索引，若不存在则返回-1。
12.  isEmpty()：判断列表是否为空。
13.  iterator()：返回一个迭代器，用于遍历列表中的元素。
14.  lastIndexOf(Object o)：返回指定元素在列表中最后一次出现位置的索引，若不存在则返回-1。
15.  listIterator()：返回一个列表迭代器，用于遍历列表中的元素。
16.  remove(Object o)：移除列表中的指定元素，返回是否移除成功。
17.  remove(int index)：移除列表中指定位置的元素，返回被移除的元素。
18.  removeFirst()：移除列表中第一个元素。
19.  removeFirstOccurrence(Object o)：移除列表中第一次出现的指定元素，返回是否移除成功。
20.  removeLast()：移除列表中的最后一个元素。
21.  removeLastOccurrence(Object o)：移除列表中最后一次出现的指定元素，返回是否移除成功。
22.  size()：返回列表中的元素个数。
23.  toArray()：将列表转换为一个数组。

测试用例
----

### 测试代码演示

下面给出LinkedList的一个简单测试用例：

```java
package com.demo.javase.day59;

import java.util.LinkedList;


 * @Author bug菌
 * @Date 2023-11-05 23:48
 */
public class LinkedListTest {

    public static void main(String[] args) {
        LinkedList<String> linkedList = new LinkedList<String>();

        
        linkedList.add("A");
        linkedList.add("B");
        linkedList.add("C");

        
        System.out.println(linkedList);

        
        linkedList.addFirst("D");
        linkedList.addLast("E");

        
        System.out.println(linkedList);

        
        linkedList.removeFirst();
        linkedList.removeLast();

        
        System.out.println(linkedList);

        
        int size = linkedList.size();
        System.out.println("size: " + size);

        
        String element = linkedList.get(1);
        System.out.println("element: " + element);

        
        linkedList.set(1, "F");

        
        System.out.println(linkedList);
    }
}


```

### 测试结果

  根据如上测试用例，本地测试结果如下，仅供参考，你们也可以自行修改测试用例或者添加更多的测试数据或测试方法，进行熟练学习以此加深理解。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb003aa6eec84aab93daf5fa56337069~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1217&h=937&s=86596&e=png&b=2c2c2c)

### 测试代码分析

  根据如上测试用例，在此我给大家进行深入详细的解读一下测试代码，以便于更多的同学能够理解并加深印象。

这是一个使用Java中的LinkedList类进行操作的示例代码。主要实现了以下功能：

1.  创建一个空的LinkedList对象。
2.  向LinkedList中添加元素。
3.  在LinkedList的开头和末尾添加元素。
4.  删除LinkedList中的第一个和最后一个元素。
5.  获取LinkedList中的元素数量。
6.  根据下标获取LinkedList中指定的元素。
7.  替换LinkedList中指定下标的元素。

运行代码后，会输出LinkedList中的元素以及各种操作后的结果。

