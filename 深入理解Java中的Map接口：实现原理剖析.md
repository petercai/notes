# 深入理解Java中的Map接口：实现原理剖析

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a09868eb820646abbc9d7b7bae2c0482~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=7019&h=4963&s=1255864&e=png&b=ffffff)

* * *
  在Java的众多数据结构中，Map接口是一个非常常用的接口，它允许我们通过键值对的方式来存储和访问数据。在实际的开发中，我们可能会使用到多种不同的Map实现类，如HashMap、TreeMap、LinkedHashMap等，但是对于Map接口的底层实现原理，我们是否真正了解呢？本文将从源代码的角度出发，深入剖析Java中Map接口的实现原理，帮助我们更好地理解和使用它。

  本文主要介绍了Java中Map接口的实现原理，包括基于散列表的HashMap、基于红黑树的TreeMap和基于链表的LinkedHashMap三种实现方式。针对每种实现方式，本文都详细介绍了其底层数据结构、存储方式和常用操作的实现方式。通过深入分析这些实现方式的优缺点和应用场景，使读者能够在实际的开发中选择合适的Map实现类，从而达到更好的性能和效果。

概述
--

  Map接口是Java中一个非常常用的接口，它代表了一组键值对的映射关系。Map中定义了一系列键值对的操作方法，如put、get、remove等，以及一些集合操作方法，如size、isEmpty等。在Java中，Map接口的实现有许多种，其中包括基于散列表的HashMap、基于红黑树的TreeMap和基于链表的LinkedHashMap等。针对不同的应用场景和需求，我们可以选择不同的Map实现类。

### HashMap

  HashMap是Java中最常用的Map实现类之一，也是效率最高的。它基于散列表实现，通过哈希算法将键映射到哈希表中的位置，从而实现键值对的存储和查找。HashMap中每个键值对存储在一个Entry对象中，该对象包含键、值和指向下一个Entry对象的指针。

#### 底层数据结构

  HashMap底层的数据结构是一个数组，该数组的每个元素都是一个链表。每个链表存储了hashCode相同的键值对集合。当键值对被加入HashMap时，它们的键通过hashCode()方法计算出一个哈希值，根据该哈希值找到对应的链表，并将该键值对存储在链表中。

  在进行查询时，Java会先通过hashCode()方法计算出该键的哈希值，然后根据哈希值找到相应的链表，最后在链表中进行查找，找到对应的节点即可。

#### 常用操作的实现

##### put操作

  当我们向HashMap中加入一个键值对时，首先会通过hashCode()方法计算键的哈希值，然后将该键值对存储在对应的链表中。如果该链表中已经存在相同的键，则会更新该键对应的值。

```java
public V put(K key, V value) {
    if (key == null) {
        return putForNullKey(value);
    }
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}

```

**代码分析**

  此代码为 Java 中 HashMap 类的 put 方法的实现。作用是将指定的键和值添加到 HashMap 中，并返回上一次该键对应的值。

  首先判断传入的键是否为 null，如果是，则调用 putForNullKey 方法进行处理。

  如果键不为 null，则计算哈希值，然后通过调用 indexFor 方法计算该键值对在数组中的索引位置。接着，遍历该索引位置处的链表，查找是否已经存在该键值对。如果存在，则更新该键值对的值，返回旧的值。否则，将新的键值对添加到该链表的末尾，返回 null。

  如果添加操作后，HashMap 中的键值对数目超过了负载因子乘以当前数组的长度，则进行 rehash 操作，即将数组大小扩大一倍，并将旧的键值对重新分桶到新数组中。

  在添加键值对时，会调用 addEntry 方法，该方法会将新的键值对添加到链表的末尾，并更新 size 和 modCount 次数。

  如下是部分源码截图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f780674fccfc40b5b69efc4e5d7f7f5c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1224&h=935&s=118540&e=png&b=2b2b2b)

##### get操作

  当我们从HashMap中获取一个键对应的值时，首先会通过hashCode()方法计算该键的哈希值，然后在对应的链表中查找节点。如果找到了该节点，则返回该节点的值。

```java
public V get(Object key) {
    if (key == null) {
        return getForNullKey();
    }
    int hash = hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            return e.value;
        }
    }
    return null;
}

```

**代码分析**

  如下是部分源码截图：

##### remove操作

  当我们从HashMap中移除一个键值对时，首先会通过hashCode()方法计算该键的哈希值，然后在对应的链表中查找节点。如果找到了该节点，则从链表中移除该节点。

```java
public V remove(Object key) {
    Entry<K,V> e = removeEntryForKey(key);
    return (e == null ? null : e.value);
}

final Entry<K,V> removeEntryForKey(Object key) {
    int hash = (key == null) ? 0 : hash(key.hashCode());
    int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;

    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;
        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
            if (prev == e) {
                table[i] = next;
            } else {
                prev.next = next;
            }
            e.recordRemoval(this);
            return e;
        }
        prev = e;
        e = next;
    }

    return e;
}

```

**代码分析**

  这段代码实现了从HashMap中删除指定key对应的Entry，并返回其对应的value。具体实现过程如下：

  首先根据要删除的key计算出它的hash值，然后根据hash值找到相应的桶，对应的Entry链表的头节点为table\[i\]。

  然后遍历这个Entry链表，从头节点开始，逐个节点进行比较，找到要删除的节点。

  如果要删除的节点是链表的头节点，直接将table\[i\]指向下一个节点；否则，将前一个节点的next指向当前节点的下一个节点。

  最后，将要删除的节点进行清理操作，并返回它对应的value值。如果找不到要删除的节点，则返回null。

  如下是部分源码截图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f5b92a5c3b3491ba5fde67dbf860116~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1141&h=180&s=18914&e=png&b=2c2c2c)

### TreeMap

  TreeMap是一种基于红黑树实现的Map，它能够对键进行排序，因此适用于需要按照键值进行排序的场景。TreeMap中每个键值对存储在一个节点中，该节点包含键、值、左子节点和右子节点等信息。

#### 底层数据结构

  TreeMap底层的数据结构是一棵红黑树，每个节点都包含一个键值对。红黑树是一种自平衡二叉搜索树，它能够保持树的高度平衡，因此能够保证查询、插入和删除操作都能够在O(log n)的时间复杂度内完成。

  在进行查询时，Java会从根节点开始寻找，如果键比当前节点的键小，则向左子树查找，如果键比当前节点的键大，则向右子树查找。最终查找到对应的节点即可。

#### 常用操作的实现

##### put操作

  当我们向TreeMap中加入一个键值对时，首先会寻找该键对应的节点。如果找到了该节点，则更新该节点对应的值，否则将该节点插入到树中。

```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        compare(key, key); 
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0) {
                t = t.left;
            } else if (cmp > 0) {
                t = t.right;
            } else {
                return t.setValue(value);
            }
        } while (t != null);
    } else {
        if (key == null) {
            throw new NullPointerException();
        }
        Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0) {
                t = t.left;
            } else if (cmp > 0) {
                t = t.right;
            } else {
                return t.setValue(value);
            }
        } while (t != null);
    }
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0) {
        parent.left = e;
    } else {
        parent.right = e;
    }
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}

```

  在这里，我们首先检查树是否为空，如果是，则创建一个新节点作为根节点。如果树不为空，则在树中寻找适当的位置来插入新的键值对，如果该键已经存在于树中，则更新相应的值。

  寻找适当的位置使用了与`getEntry()`方法类似的算法，但是在该算法中，一旦找到了节点，则返回该节点。这个版本中，我们不仅需要找到该节点，还需要记录该节点的父节点，以便插入新节点。

  当找到一个不存在该键的节点时，我们可以创建一个新节点并将其插入到树中。为了插入新的节点，我们需要保持树的有序性质并平衡树的高度。具体地说，我们需要执行以下步骤：

1.创建一个新的节点`e`来保存键值对，并将其父节点设置为`parent`。

2.将`e`插入到树中，将其置于`parent`的左子树或右子树中，具体取决于`cmp`的值。

3.调用`fixAfterInsertion(e)`来调整树的高度并保持树的平衡。

4.增加树的大小，并返回`null`。

### LinkedHashMap

  LinkedHashMap是一种基于双向链表+散列表的Map，它与HashMap的区别在于，LinkedHashMap会维护一个双向链表，该链表按照插入顺序维护了所有键值对的顺序。因此，适用于需要按照插入顺序访问元素的场景。LinkedHashMap中每个键值对存储在一个Entry对象中，该对象包含键、值、指向前一个Entry对象的指针和指向后一个Entry对象的指针。

#### 底层数据结构

  LinkedHashMap底层同时使用了散列表和双向链表。散列表保证了键值对的查找性能，而双向链表则保证了键值对的顺序性。LinkedHashMap中的每个节点都是一个Entry对象，该对象继承自HashMap中的Entry对象，并增加了前驱指针和后继指针。

  在进行查询时，Java会先通过hashCode()方法计算该键的哈希值，然后在散列表中查找对应的节点。如果找到了该节点，则返回该节点的值。同时，由于LinkedHashMap还维护了双向链表，因此可以按照插入顺序遍历所有键值对。

#### 常用操作的实现

  LinkedHashMap的常用操作与HashMap类似，不同之处在于，LinkedHashMap需要维护一个双向链表，因此在进行put、remove操作时，需要同时更新链表。

##### put操作

  当我们向LinkedHashMap中加入一个键值对时，首先会通过hashCode()方法计算键的哈希值，然后将该键值对存储在对应的链表中。如果该链表中已经存在相同的键，则会更新该键对应的值。同时，我们还需要在链表中更新该键值对的顺序，保证链表的顺序和键值对的插入顺序一致。

```java
public V put(K key, V value) {
    Node<K,V>[] tab;
    Node<K,V> p;
    int n, i;
    if ((tab = table) == null || (n = tab.length) == 0) {
        n = (tab = resize()).length;
    }
    if ((p = tab[i = (n - 1) & hash(key)]) == null) {
        tab[i] = newNode(key, value, null);
    } else {
        Node<K,V> e;
        K k;
        if (p.hash == hash(key) &&
            ((k = p.key) == key || (key != null && key.equals(k)))) {
            e = p;
        } else if (p instanceof TreeNode) {
            e = ((TreeNode<K,V>) p).putTreeVal(this, tab, hash(key), key, value);
        } else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) {
                        treeifyBin(tab, hash(key));
                    }
                    break;
                }
                if (e.hash == hash(key) &&
                    ((k = e.key) == key || (key != null && key.equals(k)))) {
                    break;
                }
                p = e;
            }
        }
        if (e != null) {
            V oldValue = e.value;
            e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold) {
        resize();
    }
    afterNodeInsertion(true);
    return null;
}

```

**代码分析：** 

  这段代码实现了 `Map` 接口中的 `put` 方法，用于将指定的 key-value 对添加到 Map 中。以下是代码实现的步骤：

1.  首先判断当前的哈希表 `table` 是否为空，或者长度为 0。如果是，则调用 `resize()` 方法将哈希表大小扩大为原来的两倍。
    
2.  根据 key 的哈希值计算出它在哈希表中的下标位置，使用 `&(n-1)` 可以保证下标位置在哈希表的范围内。
    
3.  如果哈希表 `table` 中该下标位置 `i` 没有其他元素，则直接将新元素插入到该位置。
    
4.  如果该位置已经存在一个元素，则需要遍历该位置处的链表或红黑树，查找是否有和 `key` 相同的元素。
    
5.  如果当前位置是红黑树，则调用 `putTreeVal()` 方法在红黑树中插入元素。
    
6.  如果当前位置是链表，则遍历链表查找是否有和 `key` 相同的元素，如果找到了，则将该元素的值更新为新的值。如果没有找到，则将新元素插入链表的最后面。如果链表长度达到阈值（默认为 8），则将链表转换为红黑树。
    
7.  如果添加元素后哈希表大小超过了阈值，则调用 `resize()` 方法将哈希表大小扩大为原来的两倍。
    
8.  调用 `afterNodeAccess()` 和 `afterNodeInsertion()` 方法，完成插入操作后的收尾工作。
    
9.  最后返回旧值，如果不存在旧值，则返回 null。
    

##### remove操作

  当我们从LinkedHashMap中移除一个键值对时，首先会通过hashCode()方法计算该键的哈希值，然后在对应的链表中查找节点。如果找到了该节点，则从链表中移除该节点。同时，我们还需要在链表中更新其他节点的前驱和后继指针，保证链表的顺序。

```java
public V remove(Object key) {
    Node<K,V>[] tab;
    Node<K,V> p;
    int n, hash;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[(hash = hash(key)) & (n-1)]) != null) {
        Node<K,V> node = null, e;
        K k;
        V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k)))) {
            node = p;
        } else if ((e = p.next) != null) {
            if (p instanceof TreeNode) {
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            } else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (v = node.value) != null) {
            if (node instanceof TreeNode) {
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, false);
            } else if (node == p) {
                tab[hash & (n-1)] = node.next;
            } else {
                p.next = node.next;
            }
            --size;
            afterNodeRemoval(node);
            return v;
        }
    }
    return null;
}

```

**代码分析：** 

  这段代码是 HashMap 中的 remove 方法的实现。它接受一个键对象作为参数，返回其对应的值对象，并将其从 HashMap 中移除。

  首先，该方法会获取数组 table 和其长度 n。然后，根据提供的键对象计算出其哈希值 hash，并取出在 table 数组中该键所对应的节点 p。如果该节点不为空，那么就需要进一步查找是否存在该键的节点，如果存在则将其移除。如果 p 节点正好是该键所对应的节点，那么将该节点赋值给 node。如果 p 节点不是该键所对应的节点，那么需要遍历该节点的链表，查找是否存在该键对应的节点。如果该节点为红黑树节点，则使用红黑树的查找方式进行查找；否则，使用链表的方式进行查找。最终如果找到了该键所对应的节点，则将其赋值给 node 变量。

  如果 node 不为空且其值不为空，则说明找到了该键对应的节点。如果该节点为红黑树节点，则调用 removeTreeNode 方法将其从红黑树移除；否则，如果该节点正好为 p 节点，则直接将其从链表中移除；否则，在链表中将其前一个节点的 next 属性指向该节点的下一个节点。

  最后，将 HashMap 的元素个数减一，并调用 afterNodeRemoval 方法。如果找不到该键所对应的节点，则返回 null。

  总之，该方法的作用就是移除 HashMap 中指定键所对应的节点。

##### 遍历操作

  由于LinkedHashMap维护了一个双向链表，因此可以通过遍历链表来实现按照插入顺序遍历所有键值对的功能。具体实现时，可以使用Iterator或者foreach循环遍历链表。

```java
public Set<Entry<K,V>> entrySet() {
    Set<Entry<K,V>> es;
    return (es = entrySet) == null ? (entrySet = new LinkedEntrySet()) : es;
}

private final class LinkedEntrySet extends AbstractSet<Entry<K,V>> {
    public final int size()                 { return size; }
    public final void clear()               { LinkedHashMap.this.clear(); }
    public final Iterator<Entry<K,V>> iterator() {
        return new LinkedEntryIterator();
    }
    public final boolean contains(Object o) {
        if (!(o instanceof Map.Entry)) {
            return false;
        }
        Map.Entry<?,?> e = (Map.Entry<?,?>) o;
        Object key = e.getKey();
        Node<K,V> node;
        if ((node = getNode(hash(key), key)) == null) {
            return false;
        }
        return node.equals(e);
    }
    public final boolean remove(Object o) {
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Object value = e.getValue();
            return removeNode(hash(key), key, value, true, true) != null;
        }
        return false;
    }
    public final Spliterator<Entry<K,V>> spliterator() {
        return Spliterators.spliterator(this, Spliterator.SIZED | Spliterator.ORDERED | Spliterator.CONCURRENT);
    }
}

private final class LinkedEntryIterator extends HashMapEntryIterator<Entry<K,V>> {
    LinkedEntryIterator() { super(); }
    public Entry<K,V> next() { return nextNode(); }
    public void remove() { LinkedHashMap.this.removeNode(lastReturned); }
}

```

  如上代码实现了 LinkedHashMap 的 entrySet() 方法，返回一个 Set 集合，其中包含了映射表中的所有键值对（即 Map.Entry 对象）。LinkedEntrySet 是 LinkedHashMap 内部类，继承了 AbstractSet 类。其中，size() 方法返回键值对数量；clear() 方法清空整个映射表；iterator() 方法返回迭代器对象，用于遍历 Set 集合中的键值对；contains() 方法判断 Set 集合中是否包含指定的键值对；remove() 方法移除指定的键值对；spliterator() 方法返回 Spliterator 对象，用于并行遍历 Set 集合中的键值对。

  LinkedEntryIterator 是另一个内部类，继承了 HashMapEntryIterator 类，并实现了 next() 和 remove() 方法。next() 方法返回下一个 Map.Entry 对象；remove() 方法移除最后一个返回的 Map.Entry 对象。

测试用例
----

下面是一个简单的示例，演示如何使用Map接口：

### 测试代码演示

```java
package com.demo.javase.day65;

import java.util.HashMap;
import java.util.Map;


 * @Author bug菌
 * @Date 2023-11-06 11:45
 */
public class MapTest {
    public static void main(String[] args) {
        
        Map<String, Integer> studentGrades = new HashMap<>();

        
        studentGrades.put("张三", 90);
        studentGrades.put("李四", 80);
        studentGrades.put("王五", 70);

        
        System.out.println(studentGrades.containsKey("张三"));  
        System.out.println(studentGrades.containsKey("赵六"));  

        
        System.out.println(studentGrades.get("张三"));  

        
        for (Map.Entry<String, Integer> entry : studentGrades.entrySet()) {
            System.out.println(entry.getKey() + "的成绩是：" + entry.getValue());
        }

        
        studentGrades.remove("李四");

        
        studentGrades.clear();
    }
}

```

### 测试结果

  根据如上测试用例，本地测试结果如下，仅供参考，你们也可以自行修改测试用例或者添加更多的测试数据或测试方法，进行熟练学习以此加深理解。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69e57f309888411d93281a75b30ee474~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1211&h=882&s=85605&e=png&b=2c2c2c)

### 测试代码分析

  根据如上测试用例，在此我给大家进行深入详细的解读一下测试代码，以便于更多的同学能够理解并加深印象。

  该代码演示了Map的基本用法，包括创建Map实例、向Map中添加键值对、判断是否包含某个键、获取某个键对应的值、遍历Map中所有的键值对、删除某个键值对、清空Map中所有的键值对等操作。

  具体来说，代码中首先创建了一个HashMap实例，接着使用put方法向Map中添加了三个键值对。之后使用containsKey方法判断Map中是否包含某个键，并使用get方法获取某个键对应的值。然后使用entrySet方法遍历Map中所有的键值对，并使用getKey和getValue方法分别获取键和值。接着使用remove方法删除了Map中的某个键值对，最后使用clear方法清空了Map中所有的键值对。

小结
--

  本文主要介绍了Java中Map接口的三种常见实现方式：基于散列表的HashMap、基于红黑树的TreeMap和基于链表的LinkedHashMap。针对每种实现方式，本文都详细介绍了其底层数据结构、存储方式和常用操作的实现方式。通过深入分析这些实现方式的优缺点和应用场景，我们能够更好地理解和使用Map接口，在实际的开发中选择合适的Map实现类，从而达到更好的性能和效果。

  本文介绍了Java中Map接口的三种常见实现方式：基于散列表的HashMap、基于红黑树的TreeMap和基于链表的LinkedHashMap。对于每种实现方式，本文详细介绍了其底层数据结构、存储方式和常用操作的实现方式，并分析了它们的优缺点和应用场景。对于开发人员来说，选择合适的Map实现类非常重要，能够在性能和效果方面带来很大的影响。因此，通过深入了解Map实现方式的细节和原理，能够更好地使用它们。
