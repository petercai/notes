# HashSet的底层实现原理解析
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff31f22af30b4d07a2e2c5eb542f3210~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=7019&h=4963&s=1255864&e=png&b=ffffff)

* * *

  在Java中，我们常常使用HashSet来存储一组不重复的对象，但是在使用它的时候，我们可能并没有意识到它的底层实现原理。本篇文章将会深入分析HashSet的底层实现原理，以便更好地理解它的使用和优化。

  本篇文章将会深入分析Java中HashSet的底层实现原理，包括HashSet的源代码解析，应用场景案例，优缺点分析，类代码方法介绍，以及测试用例和全文小结。

概述
--

  HashSet是Java中一个常用的集合类，它继承了AbstractSet类，并且实现了Set接口。与List不同的是，HashSet中的元素是无序的，并且不允许有重复的元素。在使用HashSet时，我们通常会使用add()方法来向其中添加元素，并且使用contains()方法来判断元素是否存在于集合中。

  HashSet的底层实现原理是基于HashMap实现的。HashSet中的元素在底层都是存储在HashMap的key上的，而value则是一个"PRESENT"常量，它并没有实际的作用，只是用于填充HashMap的value值。因此，HashSet中的元素都是唯一的，并且无序。

源代码解析
-----

下面是HashSet的源代码：

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    
    private static final Object PRESENT = new Object();

    
    public HashSet() {
        map = new HashMap<>();
    }

    
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }

    
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }

    
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

    
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    
    public void clear() {
        map.clear();
    }

    
    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    
    public boolean isEmpty() {
        return map.isEmpty();
    }

    
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }

    
    public int size() {
        return map.size();
    }

    
    public Object[] toArray() {
        return map.keySet().toArray();
    }

    
    public <T> T[] toArray(T[] a) {
        return map.keySet().toArray(a);
    }

    
    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }

    
    public Object clone() {
        try {
            HashSet<E> newSet = (HashSet<E>) super.clone();
            newSet.map = (HashMap<E,Object>) map.clone();
            return newSet;
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }

    
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        s.defaultWriteObject();
        s.writeInt(map.capacity());
        s.writeFloat(map.loadFactor());
        s.writeInt(map.size());
        for (E e : map.keySet())
            s.writeObject(e);
    }

    
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();
        int capacity = s.readInt();
        if (capacity < 0) {
            throw new InvalidObjectException("Illegal capacity: " +
                                             capacity);
        }
        float loadFactor = s.readFloat();
        if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
            throw new InvalidObjectException("Illegal load factor: " +
                                             loadFactor);
        }
        int size = s.readInt();
        if (size < 0) {
            throw new InvalidObjectException("Illegal size: " +
                                             size);
        }
        map = (((HashSet<?>)this) instanceof LinkedHashSet ?
               new LinkedHashMap<>(capacity, loadFactor) :
               new HashMap<>(capacity, loadFactor));
        for (int i=0; i<size; i++) {
            E e = (E) s.readObject();
            map.put(e, PRESENT);
        }
    }
}

```

  从上面的源代码可以看出，HashSet实际上是基于HashMap实现的，也就是说HashSet本身的操作都是委托给底层的HashMap来完成的。如下是对该源码的一些分析：

  该HashSet类实现了Set接口，是一个基于HashMap实现的无序且不允许重复元素的集合。

  该类拥有一系列构造函数，可以根据不同的参数创建不同的HashSet。其中，无参构造函数默认创建一个空的HashSet，可以接收集合类型的构造函数会将传入的集合中的元素添加到当前HashSet中，可以接收初始容量和负载因子的构造函数会创建一个空的HashMap并指定初始容量和负载因子，可以接收初始容量的构造函数会创建一个空的HashMap并指定初始容量。

  该类定义了一系列方法，包括添加元素到HashSet中、将另一个集合中的元素添加到当前HashSet中、判断HashSet是否包含某个元素、从HashSet中删除某个元素、获取HashSet的大小、判断HashSet是否为空、清空HashSet等方法。

  该类还实现了Cloneable和Serializable接口，可以实现克隆和序列化。其中，克隆时会克隆一个新的HashSet并将当前HashSet中的所有元素添加到新的HashSet中，序列化时会将当前HashSet中的所有元素按顺序写到输出流中，并在反序列化时读取这些元素并添加到新的HashSet中。

  该类的内部实现使用HashMap来存储HashSet中的元素，利用HashMap不允许键重复的特性保证HashSet中元素的唯一性。同时，HashSet中的元素顺序是不确定的，因为HashMap中元素的顺序是基于哈希算法裂解的。

  此外，可以看到，HashSet中定义了一个transient的HashMap对象map，这个Map对象中的key存储了HashSet中的元素，而value则使用了一个"PRESENT"常量，它并没有实际的作用，只是用于填充Map中的value值。

  在HashSet中，添加元素的方法是add()，我们来详细解析一下这个方法的实现：

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f48399962d2f4f8395debde0b43dc1b1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1144&h=712&s=91475&e=png&b=2c2c2c)

  首先，add()方法从map中调用put()方法，并将元素e作为key，"PRESENT"常量作为value加入map中。如果put()方法返回null，则说明添加的元素在HashSet中并不存在，返回true表示添加成功；否则说明添加的元素已经存在于HashSet中，返回false表示添加失败。

  如下是部分源码截图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8b0383aff8d444da4ab34176a0ad472~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1206&h=940&s=112680&e=png&b=2b2b2b)

应用场景案例
------

HashSet的应用场景非常广泛，常见的使用场景有：

1.  去重：使用HashSet可以非常方便地去重，只需要将需要去重的对象添加到HashSet中，最后再将HashSet转换成需要的数据结构即可。
    
2.  查找：由于HashSet中的元素都是唯一的，因此我们可以使用HashSet来查找某个元素是否存在于HashSet中。
    
3.  缓存：由于HashSet具有快速查找和去重的特点，因此在缓存场景中也经常使用HashSet来存储数据。
    

优缺点分析
-----

### 优点

1.  去重：HashSet中的元素都是唯一的，这个特点非常适合去重场景。
    
2.  快速查找：由于HashSet中的元素是基于HashMap实现的，因此在查找元素时具有非常快的速度。
    
3.  高效率：HashSet的实现非常高效，支持快速的添加、删除、查找等操作。
    

### 缺点

1.  无序：由于HashSet中的元素是无序的，因此如果需要有序的元素，我们需要使用LinkedHashSet。
    
2.  线程不安全：HashSet是线程不安全的，因此在多线程的情况下，我们需要使用ConcurrentHashMap。
    

类代码方法介绍
-------

1.  add(E e)方法：向HashSet中添加元素，并返回是否添加成功。
    
2.  addAll(Collection < ? extends E> c)方法：将另一个集合中的元素添加到当前HashSet中，并返回是否添加成功。
    
3.  clear()方法：清空HashSet中的所有元素。
    
4.  contains(Object o)方法：判断HashSet中是否包含某个元素。
    
5.  isEmpty()方法：判断HashSet是否为空。
    
6.  remove(Object o)方法：从HashSet中删除某个元素，并返回是否删除成功。
    
7.  size()方法：获取HashSet的大小。
    

  如下是部分源码截图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a32f0060c8384a6d8458fb52fa54c66e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=882&h=197&s=12651&e=png&b=2b2b2b)
 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc0dbebf0abb4192afa74b2fc2de2215~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1004&h=506&s=30101&e=png&b=2b2b2b)

常用方法
----

HashSet的常用方法列举如下：

1.  iterator()方法：返回HashSet的迭代器。
    
2.  toArray()方法：返回一个包含HashSet中所有元素的Object数组。
    
3.  toArray(T\[\] a)方法：返回一个包含HashSet中所有元素的泛型数组。
    
4.  clone()方法：克隆当前HashSet。
    
5.  equals(Object o)方法：判断当前HashSet是否与另一个对象o相等。
    
6.  hashCode()方法：返回当前HashSet的哈希码。
    
7.  retainAll(Collection < ?> c)方法：保留HashSet与另一个集合c中相同的元素，删除不同的元素。
    
8.  removeAll(Collection< ?> c)方法：删除HashSet中与另一个集合c中相同的元素。
    

测试用例
----

```java
package com.demo.javase.day62;

import java.util.HashSet;
import java.util.Iterator;


 * @Author bug菌
 * @Date 2023-11-06 10:54
 */
public class HashSetTest {

    public static void main(String[] args) {
        
        HashSet<String> set = new HashSet<>();

        
        set.add("A");
        set.add("B");
        set.add("C");
        set.add("D");
        set.add("E");

        
        System.out.println(set.contains("D"));

        
        set.remove("B");

        
        System.out.println(set.size());

        
        Iterator<String> it = set.iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
        }

        
        set.clear();

        
        System.out.println(set.isEmpty());
    }
}

```

### 测试结果

  根据如上测试用例，本地测试结果如下，仅供参考，你们也可以自行修改测试用例或者添加更多的测试数据或测试方法，进行熟练学习以此加深理解。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef881da361544b2cac22e4ba176d674a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1159&h=846&s=61781&e=png&b=2b2b2b)

### 测试代码分析

  根据如上测试用例，在此我给大家进行深入详细的解读一下测试代码，以便于更多的同学能够理解并加深印象。

  此代码演示了如何使用HashSet。首先，它创建了一个空的HashSet并添加了五个元素。然后，它检查HashSet是否包含一个给定的元素“D”，并删除元素“B”。接下来，它打印HashSet的大小并遍历HashSet并打印每个元素。然后，它清空HashSet并检查HashSet是否为空。此代码演示了如何使用HashSet。首先，它创建了一个空的HashSet并添加了五个元素。然后，它检查HashSet是否包含一个给定的元素“D”，并删除元素“B”。接下来，它打印HashSet的大小并遍历HashSet并打印每个元素。然后，它清空HashSet并检查HashSet是否为空。

小结
--

  本篇文章深入分析了Java中HashSet的底层实现原理，包括源代码解析、应用场景案例、优缺点分析、类代码方法介绍和测试用例。通过学习HashSet的底层实现原理，我们能够更好地理解HashSet的使用和优化，并且能够更好地应用HashSet来解决实际问题。

  本篇文章深入分析了Java中HashSet的底层实现原理，包括源代码解析、应用场景案例、优缺点分析、类代码方法介绍和测试用例。从源代码解析可以看出HashSet是基于HashMap实现的，添加元素的方法是add()方法，它将元素作为key，"PRESENT"常量作为value加入map中，成功返回true，失败返回false。HashSet的优点包括去重、快速查找和高效率，缺点包括无序和线程不安全。常用方法包括iterator()、toArray()、clone()等，测试用例包括添加元素、判断是否包含元素、遍历、删除元素等。

