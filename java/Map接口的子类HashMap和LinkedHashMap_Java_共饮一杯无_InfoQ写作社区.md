# Map接口的子类HashMap和LinkedHashMap_Java_共饮一杯无_InfoQ写作社区
🍜HashMap 存储自定义类型键值
-------------------

练习：每位学生（姓名，年龄）都有自己的家庭住址。那么，既然有对应关系，则将学生对象和家庭住址存储到 map 集合中。学生作为键, 家庭住址作为值。

> 注意，学生姓名相同并且年龄相同视为同一名学生。

编写学生类：

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
        return age == student.age && Objects.equals(name, student.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

编写测试类：

```
public class HashMapTest {
    public static void main(String[] args) {
        
        Map<Student,String>map = new HashMap<Student,String>();
        
        map.put(newStudent("lisi",28), "上海");
        map.put(newStudent("wangwu",22), "北京");
        map.put(newStudent("zhaoliu",24), "成都");
        map.put(newStudent("zhouqi",25), "广州");
        map.put(newStudent("wangwu",22), "南京");
        
        
        Set<Student>keySet = map.keySet();
        for(Student key: keySet){
            Stringvalue = map.get(key);
            System.out.println(key.toString()+"....."+value);
        }
    }
}
```

*   当给 HashMap 中存放自定义对象时，如果自定义对象作为 key 存在，这时要保证对象唯一，必须复写对象的 hashCode 和 equals 方法(如果忘记，请回顾 HashSet 存放自定义对象)。
    
*   如果要保证 map 中存放的 key 和取出的顺序一致，可以使用`java.util.LinkedHashMap`集合来存放。
    

🍝LinkedHashMap
---------------

我们知道 HashMap 保证成对元素唯一，并且查询速度很快，可是成对元素存放进去是没有顺序的，那么我们要保证有序，还要速度快怎么办呢？在 HashMap 下面有一个子类 LinkedHashMap，它是链表和哈希表组合的一个数据存储结构。

```
public class LinkedHashMapDemo {
    public static void main(String[] args) {
        LinkedHashMap<String, String> map = new LinkedHashMap<String, String>();
        map.put("青菜", "萝卜");
        map.put("红花", "绿叶");
        map.put("美景", "佳人");
        Set<Entry<String, String>> entrySet = map.entrySet();
        for (Entry<String, String> entry : entrySet) {
            System.out.println(entry.getKey() + "  " + entry.getValue());
        }
    }
}
```

结果:

🍠Map 集合练习
----------

**需求：** 计算一个字符串中每个字符出现次数。**分析：** 

1.  获取一个字符串对象
    
2.  创建一个 Map 集合，键代表字符，值代表次数。
    
3.  遍历字符串得到每个字符。
    
4.  判断 Map 中是否有该键。
    
5.  如果没有，第一次出现，存储次数为 1；如果有，则说明已经出现过，获取到对应的值进行++，再次存储。
    
6.  打印最终结果
    

**代码：** 

```
public class MapTest {
public static void main(String[] args) {
        
        System.out.println("请录入一个字符串:");
        String line = new Scanner(System.in).nextLine();
        
        findChar(line);
    }
    private static void findChar(String line) {
        
        HashMap<Character, Integer> map = new HashMap<Character, Integer>();
        
        for (int i = 0; i < line.length(); i++) {
            char c = line.charAt(i);
            
            if (!map.containsKey(c)) {
                
                map.put(c, 1);
            } else {
                
                Integer count = map.get(c);
                
                
                map.put(c, ++count);
            }
        }
        System.out.println(map);
    }
}
```

🍢JDK9 对集合添加的优化
---------------

通常，我们在代码中创建一个集合（例如，List 或 Set ），并直接用一些元素填充它。 实例化集合，几个 add 方法 调用，使得代码重复。

```
public class Demo01 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("abc");
        list.add("def");
        list.add("ghi");
        System.out.println(list);
    }
}
```

Java 9，添加了几种集合工厂方法,更方便创建少量元素的集合、map 实例。新的 List、Set、Map 的静态工厂方法可以更方便地创建集合的不可变实例。例子：

```
public class HelloJDK9 {  
    public static void main(String[] args) {  
        Set<String> str1=Set.of("a","b","c");  
        
        System.out.println(str1);  
        Map<String,Integer> str2=Map.of("a",1,"b",2);  
        System.out.println(str2);  
        List<String> str3=List.of("a","b");  
        System.out.println(str3);  
    }  
}
```

需要注意以下两点：

> 1.  of()方法只是 Map，List，Set 这三个接口的静态方法，其父类接口和子类实现并没有这类方法，比如    HashSet，ArrayList 等待；
>     
> 2.  返回的集合是不可变的；
>     

> 本文内容到此结束了，
> 
> 如有收获欢迎点赞👍收藏💖关注✔️，您的鼓励是我最大的动力。
> 
> 如有错误❌疑问💬欢迎各位大佬指出。
> 
> **主页**：[共饮一杯无的博客汇总👨‍💻](https://www.infoq.cn/u/zhanjq/publish)  
> 
> **保持热爱，奔赴下一场山海**。🏃🏃🏃