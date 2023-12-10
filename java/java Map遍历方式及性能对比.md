# java Map遍历方式及性能对比
java中Map数据结构的遍历有哪些方式？各种使用方式的区别和性能如何呐？

1.1.entrySet遍历
--------------

entrySet遍历是最常用的一种Map遍历方式，一般在Map的键和值都需要时使用此遍历方式，使用方法分两个步骤，如下：

1.直接调用Map对象的entrySet方法，获取Entry对象。

2.从Entry对象的getKey()、getValue()方法获取key和value。

```less
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
    System.out.println("key = " + entry.getKey() + ", value = " + entry.getValue());
}

```

1.2.直接获取Map对象中的keys或者values
---------------------------

如果只使用Map对象中的keys或者values，使用该方法比较直观，keySet()方法获取Map中的所有key，通过values()方法获取所有的value。代码如下：

```go

for (Integer key : map.keySet()) {
    System.out.println("key = " + key);
}


for (Integer value : map.values()) {
    System.out.println("key = " + value);
}

```

1.3.使用Iterator
--------------

使用Iterator遍历，如果在Map遍历过程中插入或者删除节点的时候，使用该方法会比较友好。

```vbnet
Iterator<Map.Entry<Integer, Integer>> it = map.entrySet().iterator();
while (it.hasNext()) {
    Map.Entry<Integer, Integer> entry = it.next();
    System.out.println("key = " + entry.getKey() + ", value = " + entry.getValue());
}

```

1.4.java8 Lambda
----------------

java8 Lambda方式，语法看起来更简洁。

```arduino
public class Main {


    public static void main(String[] args) {

   
        
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        for (int i=0;i<100000;i++) {
            map.put(i,i);
        }

        Main main = new Main();
        long time1 = System.currentTimeMillis();
        main.loop1(map);

        long time2 = System.currentTimeMillis();
        main.loop2(map);

        long time3 = System.currentTimeMillis();
        main.loop3(map);

        long time4 = System.currentTimeMillis();
        main.loop4(map);

        long time5 = System.currentTimeMillis();

        System.out.println("time1:" + (time2-time1));
        System.out.println("time2:" + (time3-time2));
        System.out.println("time3:" + (time4-time3));
        System.out.println("time4:" + (time5-time4));
    }

    
    void loop1(Map<Integer, Integer> map) {
        for (Map.Entry<Integer, Integer> entry : map.entrySet()) {


            System.out.println("key = " + entry.getKey() + ", value = " + entry.getValue());
        }
    }

    
    void loop2(Map<Integer, Integer> map) {
        
        for (Integer key : map.keySet()) {
            System.out.println("key = " + key);
        }

        
        for (Integer value : map.values()) {
            System.out.println("key = " + value);
        }
    }

    
    void loop3(Map<Integer, Integer> map) {
        Iterator<Map.Entry<Integer, Integer>> it = map.entrySet().iterator();
        while (it.hasNext()) {
            Map.Entry<Integer, Integer> entry = it.next();
            System.out.println("key = " + entry.getKey() + ", value = " + entry.getValue());
        }
    }

    
    
    void loop4(Map<Integer, Integer> map) {
        map.forEach((key, value) -> {
            System.out.println("key = " + key + ", value = " + value);
        });
    }
}

```

10000次性能对比结果如下：

第1次
---

```makefile
loop1:590
loop2:1103
loop3:396
loop4:342

```

第2次
---

```makefile
loop1:677
loop2:1121
loop3:538
loop4:357

```

第3次
---

```makefile
loop1:823
loop2:1147
loop3:442
loop4:391

```

第4次
---

```makefile
loop1:714
loop2:1126
loop3:450
loop4:322

```

第5次
---

```makefile
loop1:639
loop2:1069
loop3:420
loop4:367

```

由此可见：使用java8 lamda表达式，对Map对象进行遍历，性能最高，单独获取keys、values由于两次遍历，可能导致性能较低，以后代码生涯中可以多使用Map的lamda表达式，代码更简洁，性能更高效.