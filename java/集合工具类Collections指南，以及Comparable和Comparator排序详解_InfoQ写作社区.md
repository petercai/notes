# 集合工具类Collections指南，以及Comparable和Comparator排序详解_InfoQ写作社区
![](_assets/c8e160eb1076b5c897ad117919b49697.jpeg.jpg)

🍟常用功能
------

*   `java.utils.Collections`是集合工具类，用来对集合进行操作。部分方法如下：
    
*   `public static <T> boolean addAll(Collection<T> c, T... elements)`:往集合中添加一些元素。
    
*   `public static void shuffle(List<?> list) 打乱顺序`:打乱集合顺序。
    
*   `public static <T> void sort(List<T> list)`:将集合中元素按照默认规则排序。
    
*   `public static <T> void sort(List<T> list，Comparator<? super T> )`:将集合中元素按照指定规则排序。
    

代码演示：

```
public class CollectionsDemo {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        
        
        
        
        
        
        Collections.addAll(list, 5, 222, 1，2);
        System.out.println(list);
        
        Collections.sort(list);
        System.out.println(list);
    }
}
结果：
[5, 222, 1, 2]
[1, 2, 5, 222]
```

代码演示之后 ，发现我们的集合按照顺序进行了排列，可是这样的顺序是采用默认的顺序，如果想要指定顺序那该怎么办呢？我们发现还有个方法没有讲，`public static <T> void sort(List<T> list，Comparator<? super T> )`:将集合中元素按照指定规则排序。接下来讲解一下指定规则的排列。

🍕Comparator 比较器
----------------

我们还是先研究这个方法`public static <T> void sort(List<T> list)`:将集合中元素按照默认规则排序。不过这次存储的是字符串类型。

```
public class CollectionsDemo2 {
    public static void main(String[] args) {
        ArrayList<String>  list = new ArrayList<String>();
        list.add("cba");
        list.add("aba");
        list.add("sba");
        list.add("nba");
        
        Collections.sort(list);
        System.out.println(list);
    }
}
```

结果：

我们使用的是默认的规则完成字符串的排序，那么默认规则是怎么定义出来的呢？说到排序了，简单的说就是两个对象之间比较大小，那么在 JAVA 中提供了两种比较实现的方式，一种是比较死板的采用`java.lang.Comparable`接口去实现，一种是灵活的当我需要做排序的时候在去选择的`java.util.Comparator`接口完成。那么我们采用的`public static <T> void sort(List<T> list)`这个方法完成的排序，实际上要求了被排序的类型需要实现 Comparable 接口完成比较的功能，在 String 类型上如下：

```
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
```

String 类实现了这个接口，并完成了比较规则的定义，但是这样就把这种规则写死了，那比如我想要字符串按照第一个字符降序排列，那么这样就要修改 String 的源代码，这是不可能的了，那么这个时候我们可以使用`public static <T> void sort(List<T> list,Comparator<? super T> )`方法灵活的完成，这个里面就涉及到了 Comparator 这个接口，位于位于 java.util 包下，排序是 comparator 能实现的功能之一,该接口代表一个比较器，比较器具有可比性！顾名思义就是做排序的，通俗地讲需要比较两个对象谁排在前谁排在后，那么比较的方法就是：

*   `public int compare(String o1, String o2)`：比较其两个参数的顺序。
    

> 两个对象比较的结果有三种：大于，等于，小于。如果要按照升序排序，则 o1 小于 o2，返回（负数），相等返回 0，01 大于 02 返回（正数）如果要按照降序排序则 o1 小于 o2，返回（正数），相等返回 0，01 大于 02 返回（负数）

操作如下:

```
public class CollectionsDemo3 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<String>();
        list.add("cba");
        list.add("aba");
        list.add("sba");
        list.add("nba");
        
        Collections.sort(list, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o2.charAt(0) - o1.charAt(0);
            }
        });
        System.out.println(list);
    }
}
```

结果如下：

🌭Comparable 和 Comparator 两个接口的区别
---------------------------------

**Comparable**：强行对实现它的每个类的对象进行整体排序。这种排序被称为类的自然排序，类的 compareTo 方法被称为它的自然比较方法。只能在类中实现 compareTo()一次，不能经常修改类的代码实现自己想要的排序。实现此接口的对象列表（和数组）可以通过 Collections.sort（和 Arrays.sort）进行自动排序，对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。

\*\*Comparator：\*\*强行对某个对象进行整体排序。可以将 Comparator 传递给 sort 方法（如 Collections.sort 或 Arrays.sort），从而允许在排序顺序上实现精确控制。还可以使用 Comparator 来控制某些数据结构（如有序 set 或有序映射）的顺序，或者为那些没有自然顺序的对象 collection 提供排序。

🥪练习
----

创建一个学生类，存储到 ArrayList 集合中完成指定排序操作。Student 初始类

```
public class Student{
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
    public String toString() {
        return "Student{" +
               "name='" + name + '\'' +
               ", age=" + age +
               '}';
    }
}
```

测试类：

```
public class Demo {

    public static void main(String[] args) {
        
        ArrayList<Student> list = new ArrayList<Student>();

        list.add(new Student("rose",18));
        list.add(new Student("jack",16));
        list.add(new Student("abc",16));
        list.add(new Student("ace",17));
        list.add(new Student("mark",16));


        
          让学生 按照年龄排序 升序
         */



        for (Student student : list) {
            System.out.println(student);
        }


    }
}
```

发现，当我们调用 Collections.sort()方法的时候 程序报错了。原因：如果想要集合中的元素完成排序，那么必须要实现比较器 Comparable 接口。于是我们就完成了 Student 类的一个实现，如下：

```
public class Student implements Comparable<Student>{
    ....
    @Override
    public int compareTo(Student o) {
        return this.age-o.age;
    }
}
```

再次测试，代码就 OK 了效果如下：

```
Student{name='jack', age=16}
Student{name='abc', age=16}
Student{name='mark', age=16}
Student{name='ace', age=17}
Student{name='rose', age=18}
```

🌮扩展
----

如果在使用的时候，想要独立的定义规则去使用 可以采用 Collections.sort(List list,Comparetor c)方式，自己定义规则：

```
Collections.sort(list, new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
        return o2.getAge()-o1.getAge();
    }
});
```

效果：

```
Student{name='rose', age=18}
Student{name='ace', age=17}
Student{name='jack', age=16}
Student{name='abc', age=16}
Student{name='mark', age=16}
```

如果想要规则更多一些，可以参考下面代码：

```
Collections.sort(list, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                
                int result = o2.getAge()-o1.getAge();

                if(result==0){
                    result = o1.getName().charAt(0)-o2.getName().charAt(0);
                }

                return result;
            }
        });
```

效果如下：

```
Student{name='rose', age=18}
Student{name='ace', age=17}
Student{name='abc', age=16}
Student{name='jack', age=16}
Student{name='mark', age=16}
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