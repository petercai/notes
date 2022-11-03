# Java基础（八）| 常用API与StringBuilder详解_timerring的博客-CSDN博客
> ⭐本专栏旨在对JAVA的基础语法及知识点进行全面且详细的讲解，完成从0到1的java学习，面向零基础及入门的学习者，通过专栏的学习可以熟练掌握JAVA编程，同时为后续的框架学习，进阶开发的代码能力打下坚实的基础。
> 
> 🔥本文已收录于JAVA基础系列专栏： [Java基础教程](https://blog.csdn.net/m0_52316372/category_12008132.html) 免费订阅，持续更新。

![](https://img-blog.csdnimg.cn/img_convert/65b5ee36b0f611646354ff15c4826a82.jpeg#pic_center)
  

1.API
-----

### 1.1API概述

*   什么是API
    
    ​ API (Application Programming Interface) ：应用程序编程接口
    
*   java中的API
    
    ​ 指的就是 JDK 中提供的各种功能的 Java类，这些类将底层的实现封装了起来，我们不需要关心这些类是如何实现的，只需要学习这些类如何使用即可，我们可以通过帮助文档来学习这些API如何使用。
    

### 1.2如何使用API帮助文档

*   打开帮助文档

![](https://img-blog.csdnimg.cn/img_convert/28744f870236c12c97a94aff4f7413d4.png)

*   找到索引选项卡中的输入框

![](https://img-blog.csdnimg.cn/img_convert/b94b3df34cebca36074620411d2b11d4.png)

*   在输入框中输入Random

![](https://img-blog.csdnimg.cn/img_convert/cbf4618f69f26d1d671f6507033e2281.png)

*   看类在哪个包下

![](https://img-blog.csdnimg.cn/img_convert/449640ea1d469193c3402397f2c1f8ed.png)

*   看类的描述

![](https://img-blog.csdnimg.cn/img_convert/80d5f314246339d28f6d2882b7711edf.png)

*   看构造方法

![](https://img-blog.csdnimg.cn/img_convert/64d7db32916f88768bdbf9a4faf69904.png)

*   看成员方法

![](https://img-blog.csdnimg.cn/img_convert/5ba475c31f7e7e9f2d622d3c21cc10de.png)

2.String类
---------

### 2.1String类概述

String 类代表字符串，Java 程序中的所有字符串文字（例如“abc”）都被实现为此类的实例。也就是说，Java 程序中所有的双引号字符串，都是 String 类的对象。String 类在 java.lang 包下，所以使用的时候不需要导包！

### 2.2String类的特点

*   字符串不可变，它们的值在创建后不能被更改
*   **虽然 String 的值是不可变的，但是它们可以被共享**
*   字符串效果上相当于字符数组( char\[\] )，但是**底层原理是字节数组(byte\[\])**

### 2.3String类的构造方法

*   常用的构造方法
    
    | 方法名 | 说明 |
    | --- | --- |
    | public String() | 创建一个空白字符串对象，不含有任何内容 |
    | public String(char\[\] chs) | 根据字符数组的内容，来创建字符串对象 |
    | public String(byte\[\] bys) | 根据字节数组的内容，来创建字符串对象 |
    | String s = “abc”; | 直接赋值的方式创建字符串对象，内容就是abc |
    
*   示例代码
    
    ```
    `public class StringDemo01 {
        public static void main(String[] args) {
            
            String s1 = new String();
            System.out.println("s1:" + s1);
    
            
            char[] chs = {'a', 'b', 'c'};
            String s2 = new String(chs);
            System.out.println("s2:" + s2);
    
            
            byte[] bys = {97, 98, 99};
            String s3 = new String(bys);
            System.out.println("s3:" + s3);
            
    
            
            String s4 = "abc";
            System.out.println("s4:" + s4);
        }
    }` 
    
    ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)
    
    *   1
    *   2
    *   3
    *   4
    *   5
    *   6
    *   7
    *   8
    *   9
    *   10
    *   11
    *   12
    *   13
    *   14
    *   15
    *   16
    *   17
    *   18
    *   19
    *   20
    *   21
    *   22
    
    
    ```
    

### 2.4创建字符串对象两种方式的区别

*   通过构造方法创建
    
    ​ 通过 new 创建的字符串对象，**每一次 new 都会申请一个内存空间，虽然内容相同，但是地址值不同**
    
    ```java
    char[]chs = { 'a', 'b', 'c'};
    string s1 = new string(chs);
    string s2= new string (chs);
    
    ```
    
    **上面的代码中，JVM会首先创建一个字符数组，然后每一次new的时候都会有一个新的地址，只不过s1和s2参考的字符串内容是相同的。** 
    
*   直接赋值方式创建
    
    ​ 以“”方式给出的字符串，只要字符序列相同(顺序和大小写)，无论在程序代码中出现几次，**JVM 都只会建立一个 String 对象，并在字符串池中维护**。
    
    ```java
    string s3 = "abc" ;
    string s4 = "abc" ;
    
    ```
    
    在上面的代码中，针对第一行代码，JVM会建立一个String对象放在字符串池中，并给s3参考;
    
    第二行则让s4直接参考字符串池中的String对象，也就是说它们本质上是同一个对象
    

### 2.5字符串的比较

#### 2.5.1==号的作用

*   **比较基本数据类型：比较的是具体的值**
*   **比较引用数据类型：比较的是对象地址值**

**如果想要比较引用数据类型的内容（比内容），这是需要使用equals比较两个字符串内容是否相同、区分大小写。** 

#### 2.5.2equals方法的作用

*   方法介绍
    
    ```java
    public boolean equals(String s)     
    
    ```
    
*   示例代码
    
    ```
    `public class StringDemo02 {
        public static void main(String[] args) {
            
            char[] chs = {'a', 'b', 'c'};
            String s1 = new String(chs);
            String s2 = new String(chs);
    
            
            String s3 = "abc";
            String s4 = "abc";
    
            
            System.out.println(s1 == s2);
            System.out.println(s1 == s3);
            System.out.println(s3 == s4);
            System.out.println("--------");
    
            
            System.out.println(s1.equals(s2));
            System.out.println(s1.equals(s3));
            System.out.println(s3.equals(s4));
        }
    }` 
    
    ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)
    
    *   1
    *   2
    *   3
    *   4
    *   5
    *   6
    *   7
    *   8
    *   9
    *   10
    *   11
    *   12
    *   13
    *   14
    *   15
    *   16
    *   17
    *   18
    *   19
    *   20
    *   21
    *   22
    *   23
    
    
    ```
    

### 2.6用户登录案例

#### 2.6.1案例需求

​ 已知用户名和密码，请用程序实现模拟用户登录。总共给三次机会，登录之后，给出相应的提示

#### 2.6.2代码实现

```
 `public class StringTest01 {
    public static void main(String[] args) {
        
        String username = "itheima";
        String psd = "timerring";

        
        for(int i=0; i<3; i++) {

            
            Scanner sc = new Scanner(System.in);

            System.out.println("请输入用户名：");
            String name = sc.nextLine();

            System.out.println("请输入密码：");
            String pwd = sc.nextLine();

            
            if (name.equals(username) && pwd.equals(password)) {
                System.out.println("登录成功");
                break;
            } else {
                if(2-i == 0) {
                    System.out.println("你的账户被锁定，请与管理员联系");
                } else {
                    
                    
                    System.out.println("登录失败，你还有" + (2 - i) + "次机会");
                }
            }
        }
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41


```

### 2.7遍历字符串案例

#### 2.7.1案例需求

键盘录入一个字符串，使用程序实现在控制台遍历该字符串

#### 2.7.2代码实现

**public char charAt(int index)：返回指定索引处的char值**

**public int length()：返回此字符串的长度**

**数组的长度：数组名.length**

**字符串的长度：字符串对象.length()**

```
 `public class StringTest02 {
    public static void main(String[] args) {
        
        Scanner sc = new Scanner(System.in);

        System.out.println("请输入一个字符串：");
        String line = sc.nextLine();

        for(int i=0; i<line.length(); i++) {
            System.out.println(line.charAt(i));
        }
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24


```

### 2.8统计字符次数案例

#### 2.8.1案例需求

​ 键盘录入一个字符串，统计该字符串中大写字母字符，小写字母字符，数字字符出现的次数(不考虑其他字符)

#### 2.8.2代码实现

```
 `public class StringTest03 {
    public static void main(String[] args) {
        
        Scanner sc = new Scanner(System.in);

        System.out.println("请输入一个字符串：");
        String line = sc.nextLine();

        
        int bigCount = 0;
        int smallCount = 0;
        int numberCount = 0;

        
        for(int i=0; i<line.length(); i++) {
            char ch = line.charAt(i);

            
            if(ch>='A' && ch<='Z') {
                bigCount++;
            } else if(ch>='a' && ch<='z') {
                smallCount++;
            } else if(ch>='0' && ch<='9') {
                numberCount++;
            }
        }

        
        System.out.println("大写字母：" + bigCount + "个");
        System.out.println("小写字母：" + smallCount + "个");
        System.out.println("数字：" + numberCount + "个");
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45


```

### 2.9字符串拼接案例

#### 2.9.1案例需求

​ 定义一个方法，把 int 数组中的数据按照指定的格式拼接成一个字符串返回，调用该方法，

​ 并在控制台输出结果。例如，数组为 int\[\] arr = {1,2,3}; ，执行方法后的输出结果为：\[1, 2, 3\]

#### 2.9.2代码实现

```
 `public class StringTest04 {
    public static void main(String[] args) {
        
        int[] arr = {1, 2, 3};

        
        String s = arrayToString(arr);

        
        System.out.println("s:" + s);
    }

    
    
    public static String arrayToString(int[] arr) {
        
        String s = "";

        s += "[";

        for(int i=0; i<arr.length; i++) {
            if(i==arr.length-1) {
                s += arr[i];
            } else {
                s += arr[i];
                s += ", ";
            }
        }

        s += "]";

        return s;
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47


```

### 2.10字符串反转案例

#### 2.10.1案例需求

​ 定义一个方法，实现字符串反转。键盘录入一个字符串，调用该方法后，在控制台输出结果

​ 例如，键盘录入 abc，输出结果 cba

#### 2.10.2代码实现

```
 `public class StringTest05 {
    public static void main(String[] args) {
        
        Scanner sc = new Scanner(System.in);

        System.out.println("请输入一个字符串：");
        String line = sc.nextLine();

        
        String s = reverse(line);

        
        System.out.println("s:" + s);
    }

    
    
    public static String reverse(String s) {
        
        String ss = "";

        for(int i=s.length()-1; i>=0; i--) {
            ss += s.charAt(i);
        }

        return ss;
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40


```

### 2.11帮助文档查看String常用方法

| 方法名 | 说明 |
| --- | --- |
| public boolean equals(Object anObject) | 比较字符串的内容，严格区分大小写(用户名和密码) |
| public char charAt(int index) | 返回指定索引处的 char 值 |
| public int length() | 返回此字符串的长度 |

3.StringBuilder类
----------------

### 3.1StringBuilder类概述

​ StringBuilder 是一个可变的字符串类，我们可以把它看成是一个容器，这里的可变指的是 StringBuilder 对象中的内容是可变的

### 3.2StringBuilder类和String类的区别

*   **String类：内容是不可变的**
*   **StringBuilder类：内容是可变的**

### 3.3StringBuilder类的构造方法

*   常用的构造方法
    
    | 方法名 | 说明 |
    | --- | --- |
    | public StringBuilder() | 创建一个空白可变字符串对象，不含有任何内容 |
    | public StringBuilder(String str) | 根据字符串的内容，来创建可变字符串对象 |
    
*   示例代码
    

```java
public class StringBuilderDemo01 {
    public static void main(String[] args) {
        
        StringBuilder sb = new StringBuilder();
        System.out.println("sb:" + sb);
        System.out.println("sb.length():" + sb.length());

        
        StringBuilder sb2 = new StringBuilder("hello");
        System.out.println("sb2:" + sb2);
        System.out.println("sb2.length():" + sb2.length());
    }
}

```

### 3.4StringBuilder类添加和反转方法

*   添加和反转方法
    
    | 方法名 | 说明 |
    | --- | --- |
    | public StringBuilder append(任意类型) | 添加数据，并返回**对象本身** |
    | public StringBuilder reverse() | 返回相反的字符序列 |
    
*   示例代码
    

```
`public class StringBuilderDemo01 {
    public static void main(String[] args) {
        
        StringBuilder sb = new StringBuilder();

        











        
        sb.append("hello").append("world").append("java").append(100);

        System.out.println("sb:" + sb);

        
        sb.reverse();
        System.out.println("sb:" + sb);
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27


```

### 3.5StringBuilder和String相互转换

*   **StringBuilder转换为String**
    
    ​ **public String toString()：通过 toString() 就可以实现把 StringBuilder 转换为 String**
    
*   **String转换为StringBuilder**
    
    ​ **public StringBuilder(String s)：通过构造方法就可以实现把 String 转换为 StringBuilder**
    
*   示例代码
    

```
`public class StringBuilderDemo02 {
    public static void main(String[] args) {
        

        
        String s = "hello";

        

        
        StringBuilder sb = new StringBuilder(s);

        System.out.println(sb);
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25


```

### 3.6字符串拼接升级版案例

#### 3.6.1案例需求

​ 定义一个方法，把 int 数组中的数据按照指定的格式拼接成一个字符串返回，调用该方法，

​ 并在控制台输出结果。例如，数组为int\[\] arr = {1,2,3}; ，执行方法后的输出结果为：\[1, 2, 3\]

#### 3.6.2代码实现

```
 `public class StringBuilderTest01 {
    public static void main(String[] args) {
        
        int[] arr = {1, 2, 3};

        
        String s = arrayToString(arr);

        
        System.out.println("s:" + s);

    }

    
    
    public static String arrayToString(int[] arr) {
        
        StringBuilder sb = new StringBuilder();

        sb.append("[");

        for(int i=0; i<arr.length; i++) {
            if(i == arr.length-1) {
                sb.append(arr[i]);
            } else {
                sb.append(arr[i]).append(", ");
            }
        }

        sb.append("]");

        String s = sb.toString();

        return  s;
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48
*   49


```

### 3.7字符串反转升级版案例

#### 3.7.1案例需求

​ 定义一个方法，实现字符串反转。键盘录入一个字符串，调用该方法后，在控制台输出结果

​ 例如，键盘录入abc，输出结果 cba

#### 3.7.2代码实现

```
 `public class StringBuilderTest02 {
    public static void main(String[] args) {
        
        Scanner sc = new Scanner(System.in);

        System.out.println("请输入一个字符串：");
        String line = sc.nextLine();

        
        String s = myReverse(line);

        
        System.out.println("s:" + s);
    }

    
    
    public static String myReverse(String s) {
        
        





       return new StringBuilder(s).reverse().toString();
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40


```

### 3.8帮助文档查看StringBuilder常用方法

| 方法名 | 说明 |
| --- | --- |
| public StringBuilder append (任意类型) | 添加数据，并返回对象本身 |
| public StringBuilder reverse() | 返回相反的字符序列 |
| public int length() | 返回长度，实际存储值 |
| public String toString() | 通过toString()就可以实现把StringBuilder转换为String |