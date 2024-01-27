# 如何正确遍历删除List中的元素
删除List中元素这个场景很场景，很多人可能直接在循环中直接去删除元素，这样做对吗？我们就来聊聊。

### for循环索引删除

删除长度为4的字符串元素。

```csharp
    List<String> list = new ArrayList<String>();
    list.add("AA");
    list.add("BBB");
    list.add("CCCC");
    list.add("DDDD");
    list.add("EEE");
​
    for (int i = 0; i < list.size(); i++) {
        if (list.get(i).length() == 4) {
            list.remove(i);
        }
    }
    System.out.println(list);
}

```

实际上输出结果：

```csharp
[AA, BBB, DDDD, EEE]

```

DDDD 竟然没有删掉！

原因是：删除某个元素后，list的大小size发生了变化，而list的索引也在变化，索引为`i`的元素删除后，后边元素的索引自动向前补位，即原来索引为`i+1`的元素，变为了索引为`i`的元素，但是下一次循环取的索引是`i+1`，此时你以为取到的是原来索引为`i+1`的元素，其实取到是原来索引为`i+2`的元素，所以会导致你在遍历的时候漏掉某些元素。

比如当你删除第1个元素后，继续根据索引访问第2个元素时，因为删除的关系后面的元素都往前移动了一位，所以实际访问的是第3个元素。不会报出异常，只会出现漏删的情况。

### foreach循环删除元素

```scss
for (String s : list) {
        if (s.length() == 4) {
            list.remove(s);
​
        }
    }
    System.out.println(list);

```

如果没有break，会报错：

> java.util.ConcurrentModificationException at java.util.ArrayListItr.checkForComodification(ArrayList.java:911)atjava.util.ArrayListItr.checkForComodification(ArrayList.java:911) at java.util.ArrayListItr.next(ArrayList.java:861) at com.demo.ApplicationTest.testDel(ApplicationTest.java:64) at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) at java.lang.reflect.Method.invoke(Method.java:498) at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50) at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12) at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47) at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)

报ConcurrentModificationException错误的原因：

看一下JDK源码中ArrayList的remove源码是怎么实现的：

```ini
public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0
                if (elementData[index] == null) {
                    fastRemove(index)
                    return true
                }
        } else {
            for (int index = 0
                if (o.equals(elementData[index])) {
                    fastRemove(index)
                    return true
                }
        }
        return false
    }

```

一般情况下程序会最终调用fastRemove方法：

```arduino
private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; 
    }

```

在fastRemove方法中，可以看到第2行把modCount变量的值加一，但在ArrayList返回的迭代器会做迭代器内部的修改次数检查：

```arduino
final void checkForComodification() {
     if (modCount != expectedModCount)
             throw new ConcurrentModificationException();
     }

```

而foreach写法是对实际的Iterable、hasNext、next方法的简写，因为上面的remove(Object)方法修改了modCount的值，所以才会报出并发修改异常。

**阿里开发手册也明确说明禁止使用`foreach`删除、增加List元素。** 

### 迭代器Iterator删除元素

```vbnet
    Iterator<String> iterator = list.iterator();
    while(iterator.hasNext()){
        if(iterator.next().length()==4){
            iterator.remove();
        }
    }
    System.out.println(list);

```

> \[AA, BBB, EEE\]

这种方式可以正常的循环及删除。但要注意的是，**使用iterator的remove方法**，而不是List的remove方法,如果用list的remove方法同样会报上面提到的ConcurrentModificationException错误。

### 总结

无论什么场景，都不要对List使用for循环的同时，删除List集合元素，要使用迭代器删除元素。