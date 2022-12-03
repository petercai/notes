# Java重点 | 泛型_Java_几分醉意._InfoQ写作社区
> **泛型是一种<font color="red">未知的数据类型</font>，当我们不知道使用什么数据类型的时候，可以使用泛型，泛型也可以看做是一个变量，用来接收数据类型。E e：Element 元素 T t：Type 类型**

![](https://static001.geekbang.org/infoq/72/7238353a98ad85ec222e841ece34b4a4.png)

✨使用泛型的好处与弊端
-----------

> **众所周知，凡是都有两面性，首先我们来谈谈==不使用泛型的好处与坏处。==**

<font color="red">创建集合对象，不使用泛型</font>`好处：`集合不使用泛型，默认的类型是 Object 类型，可以存储任意类型数据。`坏处：`不安全，会引发异常。

<font color="red">举例说明</font>

```
private static void show01() {
        ArrayList list = new ArrayList();
        list.add("abc");
        list.add(3);
        
        
        Iterator it = list.iterator();
        
        while (it.hasNext()){
            
            Object obj = it.next();
            System.out.println(obj);
        }  

        
        



    }
```

> **那么我们接下来在来谈谈==使用泛型的好处与坏处。==**

<font color="red"></font><font color="red">创建集合对象，使用泛型</font>

```
好处：
            1.避免了类型转换的麻烦，存储的是什么类型，取出的就是什么类型
            2.把运行期异常（代码运行之后会抛出的异常），提升到了编译器（写代码的时候会报错）
        弊端：
            泛型是什么类型，只能存储什么类型的数据
```

<font color="red">举例说明</font>

```
private static void show02() {
        ArrayList<String> list = new ArrayList<>();
        list.add("abc");

        
        Iterator<String> it = list.iterator();
        while (it.hasNext()){
            String next = it.next();
            System.out.println(next+"->"+next.length()); 
        }
    }
```

✨定义和使用含有泛型的类
------------

> **定义一个含有泛型的类，模拟 ArrayList 集合泛型是一个<font color="red">未知的数据类型</font>，当我们不确定什么什么数据类型的时候，可以使用泛型泛型可以接收任意的数据类型，可以使用 Integer,String,Student...创建对象的时候确定泛型的数据类型**

<font color="red">好处 ：类型不写死，创建对象泛型是什么类型，类中泛型就是什么类型</font>

\==首先定义一个含有泛型的类==

```
public class 含有泛型的类 <E> {
    private E name;

    public E getName() {
        return name;
    }

    public void setName(E name) {
        this.name = name;
    }
    
}
```

\==接着就可以在主方法中使用它了==

```
public class 主方法 {
    public static void main(String[] args) {
        
        含有泛型的类<Object> aa = new 含有泛型的类<>();
        aa.setName("字符串");
        Object name = aa.getName();
        System.out.println(name);

        
        
        含有泛型的类<Integer> aa1 = new 含有泛型的类();
        aa1.setName(21);
        Integer name1 = aa1.getName();
        System.out.println(name1);

        
        含有泛型的类<String> aa2 = new 含有泛型的类();
        aa2.setName("字符串");
        String name2 = aa2.getName();
        System.out.println(name2);
    }
}
```

✨定义和使用含有泛型的方法
-------------

**定义含有泛型的方法：泛型定义在方法的修饰符和返回值类型之间**<font color="red">**格式：修饰符<泛型> 返回值类型 方法名（参数列表（使用泛型））{方法体；}**</font>

**含有泛型的方法，在调用方法的时候确定泛型的数据类型，传递什么类型的参数，泛型就是什么类型。** 

<font color="red">举例说明</font>

**定义含有泛型的方法**

```
public class 泛型方法 {

    
    
    public <M> void method01(M m){
        System.out.println(m);
    }

    
    public static <S> void method02(S s){
        System.out.println(s);
    }
}
```

**使用含有泛型的方法**

```
public class 主方法 {
    public static void main(String[] args) {
        
        泛型方法 aa = new 泛型方法();

        
            调用含有泛型的方法method01
            传递什么类型，泛型就是什么类型
         */
        aa.method01(1);
        aa.method01("爱睡觉");
        aa.method01(3.3);
        aa.method01(true);

        aa.method02("静态方法，不建议创建对象使用");
        
        泛型方法.method02(33); 
    }
}
```

✨定义和使用含有泛型的接口
-------------

**含有泛型的接口，第一种使用方式：<font color="red">定义接口的实现类，实现接口，指定接口的泛型。例如：</font>**

```
Scanner类实现了Iterator接口，并指定接口的泛型为String，所以重写的next方法默认就是String
    public final class Scanner implements Iterator<String>{
        public String next(){}
    }
```

**含有泛型接口第二种使用方式：<font color="red">接口使用什么泛型，实现类就使用什么泛型</font>，实现类跟着接口走，就相当于定义了一个含有泛型的类，创建对象的时候确定泛型的类型。** 

\==那么接下来我们就举例演示一下==

<font color="red">首先定义含有泛型的接口</font>

```
public interface 接口 <I>{
    public abstract void method(I i);
}
```

<font color="red">第一种使用方式</font>

```
public class 实现类1 implements 接口<String>{ 
    @Override
    public void method(String s) {
        System.out.println(s);
    }
}
```

<font color="red">第二种使用方式</font>

```
public class 实现类2<I> implements 接口<I>{ 
    @Override
    public void method(I i) {
        System.out.println(i);
    }
}
```

<font color="red">测试含有泛型的接口</font>

```
public class 主方法 {
    public static void main(String[] args) {
        
        实现类1 aa =new 实现类1();
        aa.method("字符串");

        
        实现类2<Integer> bb =new 实现类2();
        bb.method(22);

        实现类2<Double> cc =new 实现类2();
        cc.method(2.2);

        
    }
}
```

✨泛型的通配符
-------

![](https://static001.geekbang.org/infoq/c8/c801692ae4fd1d8d001ae237607dbd11.png)

![](https://static001.geekbang.org/infoq/88/8833288a1b50302a4a3c1200f103d4a8.png)

<font color="red">举例说明</font>

```
public class 泛型的通配符 {
    public static void main(String[] args) {
        ArrayList<Integer> list1 = new ArrayList<>();
        list1.add(1);
        list1.add(2);

        ArrayList<String> list2 = new ArrayList<>();
        list2.add("a");
        list2.add("b");

        aa(list1);
        aa(list2);


    }

    
            第一一个方法，能遍历所有类型的ArrayList集合
            这时候我们不知道ArrayList集合使用什么数据类型，可以使用泛型通配符“？” 来接收数据类型
            注意：
                泛型没有继承概念的 里面不能写Object
         */
    public static void aa(ArrayList<?> list){ 
        
        Iterator<?> it = list.iterator();
        while (it.hasNext()){
            Object obj = it.next(); 
            System.out.println(obj);
        }
    }
}
```

✨通配符的高级使用--受限泛型
---------------

![](https://static001.geekbang.org/infoq/cd/cd2033a628b3ab335a34c9914fc0e10e.png)

<font color="red">泛型的上限限定：？ extends E 代表使用的泛型只能是 E 类型的子类/本身

泛型的下限定：？ super E 代表使用的泛型只能是 E 类型的父类/本身</font>

✨斗地主小案例
-------

<font color="rede">按照斗地主的规则，完成洗牌发牌动作。</font>**具体规则： 使用 54 张牌打乱顺序，三个玩家参与游戏，三人交替摸排，每人 17 张牌，最后留三张作底牌**<font color="red">代码实战</font>

```
public class 斗地主案例 {
    public static void main(String[] args) {
        
        
        ArrayList<String> list = new ArrayList();
        
        String [] aa = {"♠","♥","♣","♦"};
        
        String [] bb = {"2","A","K","Q","J","10","9","8","7","6","5","4","3"};
        
        list.add("大王");
        list.add("小王");
        
        for (String s : aa) {
            for (String s1 : bb) {

                list.add(s+s1);
            }
        }


        
        
        
        
        Collections.shuffle(list);

        
        
        ArrayList<String> aa1 = new ArrayList<>(); 
        ArrayList<String> aa2 = new ArrayList<>();
        ArrayList<String> aa3 = new ArrayList<>();
        ArrayList<String> aa4 = new ArrayList<>();
        
        for (int i = 0; i < list.size(); i++) {
            
            String s = list.get(i);
            
            if (i>=51){
                
                aa4.add(s);
            }else if (i%3==0){
                
                aa1.add(s);
            }else if (i%3==1){
                
                aa2.add(s);
            }else if (i%3==2){
                
                aa3.add(s);
            }
        }

        
        System.out.println("张三"+aa1);
        System.out.println("李四"+aa2);
        System.out.println("麻子"+aa3);
        System.out.println("底牌"+aa4);

    }
}
```

<font color="red">输出结果</font>

![](https://static001.geekbang.org/infoq/3a/3a438fce92a276e74716b82f7f8e314d.png)