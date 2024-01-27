# 从源码角度解析ArrayList.subList的几个坑

**「规则1:」**![](https://mmbiz.qpic.cn/mmbiz_png/hC3oNAJqSRwSnXvQXV0BGiaowYBLEAU51XHI6pGPab6jKyR4SOmqP2EFpDEn2PJU6A3G0EJokUENhTuFr39gicTw/640?wx_fmt=png)

**「规则2:」**![](https://mmbiz.qpic.cn/mmbiz_png/hC3oNAJqSRwSnXvQXV0BGiaowYBLEAU51jyAM6UKMeS9GwvM0AwojBcx9PdYuoOYpYs4agppm7bCK7IYiaLZ3q8w/640?wx_fmt=png)

类的继承体系。

![](https://mmbiz.qpic.cn/mmbiz_png/hC3oNAJqSRwSnXvQXV0BGiaowYBLEAU51cNxxl2PqufT0EBPtPotR0Ra71iajsXibmeckuKJjtFstAwuFicgzic8OVw/640?wx_fmt=png)
可以看到 SubList 和 ArrayList 的继承体系非常类似，都实现了 RandomAccess 接口 继承自 AbstarctList。

但是SubList 和 ArrayList 并没有继承关系，因此 ArrayList 的 SubList 并不能强转为 ArrayList 。

**「下面我们写一个简单的测试代码片段来验证转换异常问题：」**

`public static void main(String[] args) {  
    List<Integer> integerList = new ArrayList<>();  
    integerList.add(0);  
    integerList.add(1);  
    integerList.add(2);  
    List<Integer> subList = integerList.subList(0, 1);

    ArrayList<Integer> castList = (ArrayList<Integer>) subList;  
  }

`

输出：

`Exception in thread "main" java.lang.ClassCastException: java.util.ArrayList$SubList cannot be cast to java.util.ArrayList  
`

从上面的结果也可以清晰地看出，subList 并不是 ArrayList 类型的实例，不能强转为 ArrayList 。

**「我们再来写一个代码片段：」**

`public static void main(String[] args) {  
    List<String> stringList = new ArrayList<>();  
    stringList.add("月");  
    stringList.add("伴");  
    stringList.add("小");  
    stringList.add("飞");  
    stringList.add("鱼");  
    stringList.add("哈");  
    stringList.add("哈");

    List<String> subList = stringList.subList(2, 4);  
    System.out.println("子列表：" + subList.toString());  
    System.out.println("子列表长度：" + subList.size());

    subList.set(1, "周星星");  
    System.out.println("子列表：" + subList.toString());  
    System.out.println("原始列表：" + stringList.toString());  
  }

`

输出：

`子列表：[小, 飞]  
子列表长度：2  
子列表：[小, 周星星]  
原始列表：[月, 伴, 小, 周星星, 鱼, 哈, 哈]  
`

可以观察到，对子列表的修改最终对原始列表产生了影响。

**「那么为啥修改子序列的索引为 1 的值影响的是原始列表的第 4 个元素呢？」**


源码分析
----

![](https://mmbiz.qpic.cn/mmbiz_png/hC3oNAJqSRwSnXvQXV0BGiaowYBLEAU51r6AAhgZL651PCwc2P3obW42gvulIicBGLT2NYukibKkvNpxads83Hh1A/640?wx_fmt=png)

可以看到SubList是ArrayList的一个内部类

这个 SubList 中的 parent 字段就是原始的 List。我们 可以认为 SubList 是原始 List 的视图

看看`java.util.ArrayList#subList` 源码：![](https://mmbiz.qpic.cn/mmbiz_png/hC3oNAJqSRwSnXvQXV0BGiaowYBLEAU51b5SLnH1GhicdRypa66kszqIfFOnv64hBGukUEEicy6qWd6fibngZfdttg/640?wx_fmt=png)
这里在构造子列表对象的时候传入了 this，创建了SubList类

同时从上面注释中我们可以学到以下知识点：

*   该方法返回本列表中 fromIndex （包含）和 toIndex （不包含）之间的元素视图。如果两个索引相等会返回一个空列表。
    
*   如果需要对 list 的某个范围的元素进行操作，可以用 subList，如：`list.subList(from, to).clear();`
    
*   任何对子列表的操作最终都会反映到原列表中。
    

我们再来看下SubList的构造函数

`SubList(AbstractList<E> parent,  
                int offset, int fromIndex, int toIndex) {  
            this.parent = parent;  
            this.parentOffset = fromIndex;  
            this.offset = offset + fromIndex;  
            this.size = toIndex - fromIndex;  
            this.modCount = ArrayList.this.modCount;  
        }  
`

通过子列表的构造函数我们知道，这里的偏移量 ( offset ) 的值为 fromIndex 参数，因为参数传的offset等于0

接下来我们再看下函数 `java.util.ArrayList.SubList#set` 源码：

`public E set(int index, E e) {  
    rangeCheck(index);  
    checkForComodification();  
    E oldValue = ArrayList.this.elementData(offset + index);  
    ArrayList.this.elementData[offset + index] = e;  
    return oldValue;  
}  
`

可以看到替换值的时候，获取索引是通过 offset + index 计算得来的。

这里的 `java.util.ArrayList#elementData` 即为原始列表存储元素的数组。

因此上面提到的：

> ❝
> 为啥子序列的索引为 1 的值影响的是原始列表的第 4 个元素呢？
> ❞

这个问题就解答了


另外在 SubList 的构造函数中，我们发现会将 ArrayList 的 modCount 赋值给 SubList 的 modCount 。

**「所以这个就引出了另外一个问题」**

我们先看 `java.util.ArrayList#add(E)`的源码：

`public boolean add(E e) {  
        ensureCapacityInternal(size + 1);  // Increments modCount!!  
        elementData[size++] = e;  
        return true;  
    }  
`

`private void ensureExplicitCapacity(int minCapacity) {  
        modCount++;

        // overflow-conscious code  
        if (minCapacity - elementData.length > 0)  
            grow(minCapacity);  
    }

`

可以发现新增元素和删除元素，都会对 modCount 进行修改。

我们来看下 SubList 的 核心的函数 `java.util.ArrayList.SubList#get`

`public E get(int index) {  
            rangeCheck(index);  
            checkForComodification();  
            return ArrayList.this.elementData(offset + index);  
        }  
`

会进行修改检查：

`private void checkForComodification() {  
            if (ArrayList.this.modCount != this.modCount)  
                throw new ConcurrentModificationException();  
        }  
`

我们再来看下 SubList 的 核心的函数 `java.util.ArrayList.SubList#add`

`public void add(int index, E e) {  
            rangeCheckForAdd(index);  
            checkForComodification();  
            parent.add(parentOffset + index, e);  
            this.modCount = parent.modCount;  
            this.size++;  
        }  
`

也会调用checkForComodification进行检查

从上面的 SubList 的构造函数我们可以知道，SubList 复制了 ArrayList 的 modCount，因此对原函数的新增或删除都会导致 ArrayList 的 modCount 的变化。

而子列表的遍历、增加、删除时又会检查创建 SubList 时的 modCount 是否一致，显然此时两者会不一致，导致抛出 ConcurrentModificationException (并发修改异常)。



**「既然 SubList 相当于原始 List 的视图，那么避免相互影响的修复方式有两种：」**

一种是，不直接使用 subList 方法返回的 SubList，而是重新使用 new ArrayList，在构 造方法传入 SubList，来构建一个独立的 ArrayList；

`List<Integer> subList = new ArrayList<>(list.subList(1, 4));  
`

另一种是，对于 Java 8 使用 Stream 的 skip 和 limit API 来跳过流中的元素，以及限制 流中元素的个数，同样可以达到 SubList 切片的目的。

`List<Integer> subList = list.stream().skip(1).limit(3).collect(Collectors.toList());  
`

本文通过类图分析、源码分析以及的方式对 ArrayList 的 SubList 问题进行分析

**「要点：」**

*   ArrayList 内部类 SubList 和 ArrayList 没有继承关系，因此无法将其强转为 ArrayList 。
    
*   subList()返回的是 ArrayList 的内部类 SubList，并不是 ArrayList 本身，而是 ArrayList 的一个视 图，对于 SubList 的所有操作最终会反映到原列表上
    
*   ArrayList 的 SubList 构造时传入 ArrayList 的 modCount，因此对原列表的修改将会导致子列表的遍历、增加、删除产生 ConcurrentModificationException 异常。
    
