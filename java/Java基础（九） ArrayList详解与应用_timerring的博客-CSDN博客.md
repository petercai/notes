# Java基础（九）| ArrayList详解与应用_timerring的博客-CSDN博客
> ⭐本专栏旨在对JAVA的基础语法及知识点进行全面且详细的讲解，完成从0到1的java学习，面向零基础及入门的学习者，通过专栏的学习可以熟练掌握JAVA编程，同时为后续的框架学习，进阶开发的代码能力打下坚实的基础。
> 
> 🔥本文已收录于JAVA基础系列专栏： [Java基础教程](https://blog.csdn.net/m0_52316372/category_12008132.html) 免费订阅，持续更新。

![](_assets/0a5ba5501ba97fffe1d24728db1870b3.jpeg.jpg)
  

1.[ArrayList](https://so.csdn.net/so/search?q=ArrayList&spm=1001.2101.3001.7020)
--------------------------------------------------------------------------------

### 1.1ArrayList类概述

*   什么是[集合](https://so.csdn.net/so/search?q=%E9%9B%86%E5%90%88&spm=1001.2101.3001.7020)
    
    ​ 提供一种存储空间可变的存储模型，存储的数据容量可以发生改变
    
*   ArrayList集合的特点
    
    ​ 底层是[数组](https://so.csdn.net/so/search?q=%E6%95%B0%E7%BB%84&spm=1001.2101.3001.7020)实现的，长度可以变化
    
*   泛型的使用
    
    ​ 用于约束集合中存储元素的数据类型
    

### 1.2ArrayList类常用方法

#### 1.2.1构造方法

| 方法名 | 说明 |
| --- | --- |
| public ArrayList() | 创建一个空的集合对象 |

#### 1.2.2成员方法

E表示返回的类型是集合中元素的类型。

| 方法名 | 说明 |
| --- | --- |
| public boolean remove(Object o) | 删除指定的元素，返回删除是否成功 |
| public E remove(int index) | 删除指定索引处的元素，返回被删除的元素 |
| public E set(int index,E element) | 修改指定索引处的元素，返回被修改的元素 |
| public E get(int index) | 返回指定索引处的元素 |
| public int size() | 返回集合中的元素的个数 |
| public boolean add(E e) | 将指定的元素追加到此集合的末尾 |
| public void add(int index,E element) | 在此集合中的指定位置插入指定的元素 |

#### 1.2.3示例代码

```
`public class ArrayListDemo02 {
    public static void main(String[] args) {
        
        ArrayList<String> array = new ArrayList<String>();

        
        array.add("hello");
        array.add("world");
        array.add("java");

        



        


        


        


        

        

        



        

        
        System.out.println(array.size());

        
        System.out.println("array:" + array);
    }
}` 

![](_assets/newCodeMoreWhite.png)

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

### 1.3ArrayList存储字符串并遍历

#### 1.3.1案例需求

​ 创建一个存储字符串的集合，存储3个字符串元素，使用程序实现在控制台遍历该集合

#### 1.3.2代码实现

```
 `public class ArrayListTest01 {
    public static void main(String[] args) {
        
        ArrayList<String> array = new ArrayList<String>();

        
        array.add("刘正风");
        array.add("左冷禅");
        array.add("风清扬");

        


        
        for(int i=0; i<array.size(); i++) {
            String s = array.get(i);
            System.out.println(s);
        }
    }
}` 

![](_assets/newCodeMoreWhite.png)

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


```

### 1.4ArrayList存储学生对象并遍历

#### 1.4.1案例需求

创建一个存储学生对象的集合，存储3个学生对象，使用程序实现在控制台遍历该集合

#### 1.4.2代码实现

```
 `public class ArrayListTest02 {
    public static void main(String[] args) {
        
        ArrayList<Student> array = new ArrayList<>();

        
        Student s1 = new Student("林青霞", 30);
        Student s2 = new Student("风清扬", 33);
        Student s3 = new Student("张曼玉", 18);

        
        array.add(s1);
        array.add(s2);
        array.add(s3);

        
        for (int i = 0; i < array.size(); i++) {
            Student s = array.get(i);
            System.out.println(s.getName() + "," + s.getAge());
        }
    }
}` 

![](_assets/newCodeMoreWhite.png)

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


```

### 1.5ArrayList存储学生对象并遍历升级版

#### 1.5.1案例需求

创建一个存储学生对象的集合，存储3个学生对象，使用程序实现在控制台遍历该集合

学生的姓名和年龄来自于键盘录入

#### 1.5.2代码实现

```
 `public class ArrayListTest {
    public static void main(String[] args) {
        
        ArrayList<Student> array = new ArrayList<Student>();

        
        
        addStudent(array);
        addStudent(array);
        addStudent(array);

        
        
        for (int i = 0; i < array.size(); i++) {
            Student s = array.get(i);
            System.out.println(s.getName() + "," + s.getAge());
        }
    }

    
    public static void addStudent(ArrayList<Student> array) {
        
        Scanner sc = new Scanner(System.in);

        System.out.println("请输入学生姓名:");
        String name = sc.nextLine();

        System.out.println("请输入学生年龄:");
        String age = sc.nextLine();

        
        Student s = new Student();
        s.setName(name);
        s.setAge(age);

        
        
        array.add(s);
    }
}` 

![](_assets/newCodeMoreWhite.png)

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
*   50
*   51
*   52
*   53


```

2.学生管理系统
--------

### 2.1学生管理系统实现步骤

*   案例需求
    
    ​ 针对目前我们的所学内容，完成一个综合案例：学生管理系统！该系统主要功能如下：
    
    ​ 添加学生：通过键盘录入学生信息，添加到集合中
    
    ​ 删除学生：通过键盘录入要删除学生的学号，将该学生对象从集合中删除
    
    ​ 修改学生：通过键盘录入要修改学生的学号，将该学生对象其他信息进行修改
    
    ​ 查看学生：将集合中的学生对象信息进行展示
    
    ​ 退出系统：结束程序
    
*   实现步骤
    
    1.  定义学生类，包含以下成员变量
        
        ​ [private](https://so.csdn.net/so/search?q=private&spm=1001.2101.3001.7020) String sid // 学生id
        
        ​ private String name // 学生姓名
        
        ​ private String age // 学生年龄
        
        ​ private String address // 学生所在地
        
    2.  学生管理系统主界面的搭建步骤
        
        2.1 用输出语句完成主界面的编写  
        2.2 用[Scanner](https://so.csdn.net/so/search?q=Scanner&spm=1001.2101.3001.7020)实现键盘输入  
        2.3 用switch语句完成选择的功能  
        2.4 用循环完成功能结束后再次回到主界面
        
    3.  学生管理系统的添加学生功能实现步骤
        
        3.1 定义一个方法，接收ArrayList集合  
        3.2 方法内完成添加学生的功能  
        ​ ①键盘录入学生信息  
        ​ ②根据录入的信息创建学生对象  
        ​ ③将学生对象添加到集合中  
        ​ ④提示添加成功信息  
        3.3 在添加学生的选项里调用添加学生的方法
        
    4.  学生管理系统的查看学生功能实现步骤
        
        4.1 定义一个方法，接收ArrayList集合  
        4.2 方法内遍历集合，将学生信息进行输出  
        4.3 在查看所有学生选项里调用查看学生方法
        
    5.  学生管理系统的删除学生功能实现步骤
        
        5.1 定义一个方法，接收ArrayList集合  
        5.2 方法中接收要删除学生的学号  
        5.3 遍历集合，获取每个学生对象  
        5.4 使用学生对象的学号和录入的要删除的学号进行比较,如果相同，则将当前学生对象从集合中删除  
        5.5 在删除学生选项里调用删除学生的方法
        
    6.  学生管理系统的修改学生功能实现步骤
        
        6.1 定义一个方法，接收ArrayList集合  
        6.2 方法中接收要修改学生的学号  
        6.3 通过键盘录入学生对象所需的信息，并创建对象  
        6.4 遍历集合，获取每一个学生对象。并和录入的修改学生学号进行比较.如果相同，则使用新学生对象替换当前学生对象  
        6.5 在修改学生选项里调用修改学生的方法
        
    7.  退出系统
        
        使用System.exit(0);退出JVM
        

### 2.2学生类的定义

```
`public class Student {
    
    private String sid;
    
    private String name;
    
    private String age;
    
    private String address;

    public Student() {
    }

    public Student(String sid, String name, String age, String address) {
        this.sid = sid;
        this.name = name;
        this.age = age;
        this.address = address;
    }

    public String getSid() {
        return sid;
    }

    public void setSid(String sid) {
        this.sid = sid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}` 

![](_assets/newCodeMoreWhite.png)

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
*   50
*   51
*   52


```

### 2.3测试类的定义

```
`public class StudentManager {
    
    public static void main(String[] args) {
        
        ArrayList<Student> array = new ArrayList<Student>();

        
        while (true) {
            
            System.out.println("--------欢迎来到学生管理系统--------");
            System.out.println("1 添加学生");
            System.out.println("2 删除学生");
            System.out.println("3 修改学生");
            System.out.println("4 查看所有学生");
            System.out.println("5 退出");
            System.out.println("请输入你的选择：");

            
            Scanner sc = new Scanner(System.in);
            String line = sc.nextLine();

            
            switch (line) {
                case "1":
                    addStudent(array);
                    break;
                case "2":
                    deleteStudent(array);
                    break;
                case "3":
                    updateStudent(array);
                    break;
                case "4":
                    findAllStudent(array);
                    break;
                case "5":
                    System.out.println("谢谢使用");
                    System.exit(0); 
            }
        }
    }

    
    public static void addStudent(ArrayList<Student> array) {
        
        Scanner sc = new Scanner(System.in);

        String sid;

        while (true) {
            System.out.println("请输入学生学号：");
            sid = sc.nextLine();

            boolean flag = isUsed(array, sid);
            if (flag) {
                System.out.println("你输入的学号已经被占用，请重新输入");
            } else {
                break;
            }
        }

        System.out.println("请输入学生姓名：");
        String name = sc.nextLine();

        System.out.println("请输入学生年龄：");
        String age = sc.nextLine();

        System.out.println("请输入学生居住地：");
        String address = sc.nextLine();

        
        Student s = new Student();
        s.setSid(sid);
        s.setName(name);
        s.setAge(age);
        s.setAddress(address);

        
        array.add(s);

        
        System.out.println("添加学生成功");
    }

    
    public static boolean isUsed(ArrayList<Student> array, String sid) {
        
        boolean flag = false;

        for(int i=0; i<array.size(); i++) {
            Student s = array.get(i);
            if(s.getSid().equals(sid)) {
                flag = true;
                break;
            }
        }

        return flag;
    }

    
    public static void findAllStudent(ArrayList<Student> array) {
        
        if (array.size() == 0) {
            System.out.println("无信息，请先添加信息再查询");
            
            return;
        }

        
        
        System.out.println("学号\t\t\t姓名\t\t年龄\t\t居住地");

        
        for (int i = 0; i < array.size(); i++) {
            Student s = array.get(i);
            System.out.println(s.getSid() + "\t" + s.getName() + "\t" + s.getAge() + "岁\t\t" + s.getAddress());
        }
    }

    
    public static void deleteStudent(ArrayList<Student> array) {
        
        Scanner sc = new Scanner(System.in);

        System.out.println("请输入你要删除的学生的学号：");
        String sid = sc.nextLine();

        
        
        

        int index = -1;

        for (int i = 0; i < array.size(); i++) {
            Student s = array.get(i);
            if (s.getSid().equals(sid)) {
                index = i;
                break;
            }
        }

        if (index == -1) {
            System.out.println("该信息不存在，请重新输入");
        } else {
            array.remove(index);
            
            System.out.println("删除学生成功");
        }
    }

    
    public static void updateStudent(ArrayList<Student> array) {
        
        Scanner sc = new Scanner(System.in);

        System.out.println("请输入你要修改的学生的学号：");
        String sid = sc.nextLine();

        
        System.out.println("请输入学生新姓名：");
        String name = sc.nextLine();
        System.out.println("请输入学生新年龄：");
        String age = sc.nextLine();
        System.out.println("请输入学生新居住地：");
        String address = sc.nextLine();

        
        Student s = new Student();
        s.setSid(sid);
        s.setName(name);
        s.setAge(age);
        s.setAddress(address);

        
        for (int i = 0; i < array.size(); i++) {
            Student student = array.get(i);
            if (student.getSid().equals(sid)) {
                array.set(i, s);
            }
        }

        
        System.out.println("修改学生成功");
    }
}` 

![](_assets/newCodeMoreWhite.png)

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
*   50
*   51
*   52
*   53
*   54
*   55
*   56
*   57
*   58
*   59
*   60
*   61
*   62
*   63
*   64
*   65
*   66
*   67
*   68
*   69
*   70
*   71
*   72
*   73
*   74
*   75
*   76
*   77
*   78
*   79
*   80
*   81
*   82
*   83
*   84
*   85
*   86
*   87
*   88
*   89
*   90
*   91
*   92
*   93
*   94
*   95
*   96
*   97
*   98
*   99
*   100
*   101
*   102
*   103
*   104
*   105
*   106
*   107
*   108
*   109
*   110
*   111
*   112
*   113
*   114
*   115
*   116
*   117
*   118
*   119
*   120
*   121
*   122
*   123
*   124
*   125
*   126
*   127
*   128
*   129
*   130
*   131
*   132
*   133
*   134
*   135
*   136
*   137
*   138
*   139
*   140
*   141
*   142
*   143
*   144
*   145
*   146
*   147
*   148
*   149
*   150
*   151
*   152
*   153
*   154
*   155
*   156
*   157
*   158
*   159
*   160
*   161
*   162
*   163
*   164
*   165
*   166
*   167
*   168
*   169
*   170
*   171
*   172
*   173
*   174
*   175
*   176
*   177
*   178
*   179
*   180
*   181
*   182
*   183
*   184
*   185
*   186
*   187
*   188
*   189
*   190
*   191
*   192


```