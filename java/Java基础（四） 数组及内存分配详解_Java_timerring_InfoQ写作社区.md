# Java基础（四）| 数组及内存分配详解_Java_timerring_InfoQ写作社区
> ⭐本专栏旨在对 JAVA 的基础语法及知识点进行全面且详细的讲解，完成从 0 到 1 的 java 学习，面向零基础及入门的学习者，通过专栏的学习可以熟练掌握 JAVA 编程，同时为后续的框架学习，进阶开发的代码能力打下坚实的基础。
> 
> 🔥本文已收录于 JAVA 基础系列专栏： [Java基础教程](https://xie.infoq.cn/link?target=https%3A%2F%2Fblog.csdn.net%2Fm0_52316372%2Fcategory_12008132.html) 免费订阅，持续更新。

![](assets/68038a53f4ffa584b73ac290fb1fe150.jpeg.jpg)

0.IDEA 开发工具
-----------

​ 参见：[IDEA中常用快捷键以及文件目录总结](https://xie.infoq.cn/link?target=https%3A%2F%2Fblog.csdn.net%2Fm0_52316372%2Farticle%2Fdetails%2F127270175)  

1.数组
----

### 1.1 什么是数组

​ 数组就是存储数据长度固定的容器，存储多个数据的数据类型要一致。

### 1.2 数组定义格式

#### 1.2.1 第一种

​ 数据类型\[\] 数组名

​ 示例：

```
int[] arr;        
double[] arr;      
char[] arr;
```

#### 1.2.2 第二种

​ 数据类型 数组名\[\]

​ 示例：

```
int arr[];
double arr[];
char arr[];
```

### 1.3 数组动态初始化

#### 1.3.1 什么是动态初始化

​ 数组动态初始化就是只给定数组的长度，由系统给出默认初始化值

#### 1.3.2 动态初始化格式

```
数据类型[] 数组名 = new 数据类型[数组长度];
```

#### 1.3.3 动态初始化格式详解

*   等号左边：
    
*   int:数组的数据类型
    
*   \[\]:代表这是一个数组
    
*   arr:代表数组的名称
    
*   等号右边：
    
*   new:为数组开辟内存空间
    
*   int:数组的数据类型
    
*   \[\]:代表这是一个数组
    
*   5:代表数组的长度
    

### 1.4 数组元素访问

#### 1.4.1 什么是索引

​ 每一个存储到数组的元素，都会自动的拥有一个编号，从 0 开始。

​ 这个自动编号称为数组索引(index)，可以通过数组的索引访问到数组中的元素。

#### 1.4.2 访问数组元素格式

#### 1.4.3 示例代码

```
public class ArrayDemo {
    public static void main(String[] args) {
        int[] arr = new int[3];

        
        System.out.println(arr); 

        
        System.out.println(arr[0]);
        System.out.println(arr[1]);
        System.out.println(arr[2]);
    }
}
```

### 1.5 内存分配

#### 1.5.1 内存概述

​ 内存是计算机中的重要原件，临时存储区域，作用是运行程序。

​ 我们编写的程序是存放在硬盘中的，在硬盘中的程序是不会运行的。

​ 必须放进内存中才能运行，运行完毕后会清空内存。

​ **Java 虚拟机要运行程序，必须要对内存进行空间的分配和管理。** 

#### 1.5.2java 中的内存分配

*   目前我们只需要记住两个内存，分别是：栈内存和堆内存
    

### 1.6 单个数组的内存图

### 1.7 多个数组的内存图

### 1.8 多个数组指向相同内存图

![](assets/53f6382d8b350f84f29a182ad48d40d5.png)

### 1.9 数组静态初始化

#### 1.9.1 什么是静态初始化

​ 在创建数组时，直接将元素确定

#### 1.9.2 静态初始化格式

*   完整版格式
    

```
数据类型[] 数组名 = new 数据类型[]{元素1,元素2,...};
```

*   简化版格式
    

```
数据类型[] 数组名 = {元素1,元素2,...};
```

#### 1.9.3 示例代码

```
public class ArrayDemo {
    public static void main(String[] args) {
        
        int[] arr = {1, 2, 3};

        
        System.out.println(arr);

        
        System.out.println(arr[0]);
        System.out.println(arr[1]);
        System.out.println(arr[2]);
    }
}
```

### 1.10 数组操作的两个常见小问题

#### 1.10.1 索引越界异常

*   出现原因
    

```
public class ArrayDemo {
      public static void main(String[] args) {
          int[] arr = new int[3];
          System.out.println(arr[3]);
      }
  }
```

数组长度为 3，索引范围是 0~2，但是我们却访问了一个 3 的索引。

程序运行后，将会抛出 ArrayIndexOutOfBoundsException 数组越界异常。在开发中，数组的越界异常是不能出现的，一旦出现了，就必须要修改我们编写的代码。

*   解决方案
    
*   将错误的索引修改为正确的索引范围即可！
    

#### 1.10.2 空指针异常

*   出现原因
    

```
public class ArrayDemo {
      public static void main(String[] args) {
          int[] arr = new int[3];
  
          
          arr = null;
          System.out.println(arr[0]);
      }
  }
```

arr = null 这行代码，意味着变量 arr 将不会在保存数组的内存地址，也就不允许再操作数组了，因此运行的时候会抛出 NullPointerException 空指针异常。在开发中，数组的越界异常是不能出现的，一旦出现了，就必须要修改我们编写的代码。

*   解决方案
    
*   给数组一个真正的堆内存空间引用即可！
    

### 1.11 数组遍历

*   数组遍历：就是将数组中的每个元素分别获取出来，就是遍历。遍历也是数组操作中的基石。
    

```
public class ArrayTest01 {
    public static void main(String[] args) {
      int[] arr = { 1, 2, 3, 4, 5 };
      System.out.println(arr[0]);
      System.out.println(arr[1]);
      System.out.println(arr[2]);
      System.out.println(arr[3]);
      System.out.println(arr[4]);
    }
  }
```

以上代码是可以将数组中每个元素全部遍历出来，但是如果数组元素非常多，这种写法肯定不行，因此我们需要改造成循环的写法。数组的索引是 0 到 lenght-1 ，可以作为循环的条件出现。

```
public class ArrayTest01 {
      public static void main(String[] args) {
          
          int[] arr = {11, 22, 33, 44, 55};
  
          
          for(int x=0; x<arr.length; x++) {
              System.out.println(arr[x]);
          }
      }
  }
```

### 1.12 数组最值

*   最大值获取：从数组的所有元素中找出最大值。
    
*   实现思路：
    
*   定义变量，保存数组 0 索引上的元素
    
*   遍历数组，获取出数组中的每个元素
    
*   将遍历到的元素和保存数组 0 索引上值的变量进行比较
    
*   如果数组元素的值大于了变量的值，变量记录住新的值
    
*   数组循环遍历结束，变量保存的就是数组中的最大值
    
*   代码实现：
    

```
public class ArrayTest02 {
      public static void main(String[] args) {
          
          int[] arr = {12, 45, 98, 73, 60};
  
          
          
          int max = arr[0];
  
          
          for(int x=1; x<arr.length; x++) {
              if(arr[x] > max) {
                  max = arr[x];
              }
          }
  
          
          System.out.println("max:" + max);
      }
  }
```