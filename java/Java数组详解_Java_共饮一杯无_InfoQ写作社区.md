# Java数组详解_Java_共饮一杯无_InfoQ写作社区
数组定义和访问
-------

### 容器概念

容器：是将多个数据存储到一起，每个数据称为该容器的元素。

### 数组概念

数组概念： 数组就是存储数据长度固定的容器，保证多个数据的数据类型要一致。

### 数组的定义

**方式一**格式：

> 数组存储的数据类型\[\] 数组名字 = new 数组存储的数据类型\[长度\];

数组定义格式详解：

*   数组存储的数据类型： 创建的数组容器可以存储什么数据类型。
    
*   \[\] : 表示数组。
    
*   数组名字：为定义的数组起个变量名，满足标识符规范，可以使用名字操作数组。
    
*   new：关键字，创建数组使用的关键字。
    
*   数组存储的数据类型： 创建的数组容器可以存储什么数据类型。
    
*   \[长度\]：数组的长度，表示数组容器中可以存储多少个元素。
    
*   注意：数组有定长特性，长度一旦指定，不可更改。
    
*   和水杯道理相同，买了一个 2 升的水杯，总容量就是 2 升，不能多也不能少。
    

按例：定义可以存储 3 个整数的数组容器：

**方式二**格式：

> 数据类型\[\] 数组名 = new 数据类型\[\]{元素 1,元素 2,元素 3...};

案例：定义存储 1，2，3，4，5 整数的数组容器。

```
int[] arr = new int[]{1,2,3,4,5};
```

**方式三**格式：

> 数据类型\[\] 数组名 = {元素 1,元素 2,元素 3...};

案例：定义存储 1，2，3，4，5 整数的数组容器。

### 数组的访问

索引： 每一个存储到数组的元素，都会自动的拥有一个编号，从 0 开始，这个自动编号称为数组索引(index)，可以通过数组的索引访问到数组中的元素。格式：

> 数组名\[索引\]

数组的长度属性： 每个数组都具有长度，而且是固定的，Java 中赋予了数组的一个属性，可以获取到数组的长度，语句为： 数组名.length ，属性 length 的执行结果是数组的长度，int 类型结果。由次可以推断出，数组的最大索引值为数组名.length-1 。

```
public static void main(String[] args) {
    int[] arr = new int[]{1,2,3,4,5};
    
    System.out.println(arr.length);
}
```

索引访问数组中的元素：

*   数组名\[索引\]=数值，为数组中的元素赋值
    
*   变量=数组名\[索引\]，获取出数组中的元素
    

数组原理内存图
-------

### 内存概述

内存是计算机中的重要原件，临时存储区域，作用是运行程序。我们编写的程序是存放在硬盘中的，在硬盘中的程序是不会运行的，必须放进内存中才能运行，运行完毕后会清空内存。Java 虚拟机要运行程序，必须要对内存进行空间的分配和管理。

### Java 虚拟机的内存划分

为了提高运算效率，就对空间进行了不同区域的划分，因为每一片区域都有特定的处理数据方式和内存管理方式。JVM 的内存划分：

### 数组在内存中的存储

#### 一个数组内存图

```
public static void main(String[] args) {
    int[] arr = new int[3];
    System.out.println(arr);
}
```

以上方法执行，输出的结果是\[I@5f461635，这个是什么呢？是数组在内存中的地址。new 出来的内容，都是在堆内存中存储的，而方法中的变量 arr 保存的是数组的地址。输出 arr\[0\]，就会输出 arr 保存的内存地址中数组中 0 索引上的元素

![](https://static001.geekbang.org/infoq/af/afd77c929efe3d059abdb6a6ce5d9bde.png)

#### 两个数组内存图

```
public static void main(String[] args) {
    int[] arr = new int[3];
    int[] arr2 = new int[2];
    System.out.println(arr);
    System.out.println(arr2);
}
```

![](https://static001.geekbang.org/infoq/1a/1a8ba45ca7dfdba35205f31e2058e946.png)

数组的常见操作
-------

### 数组越界异常

```
public static void main(String[] args) {
        int[] array = {15, 25, 35};
        System.out.println(array[0]); 
        System.out.println(array[1]); 
        System.out.println(array[2]); 

        
        
        System.out.println(array[3]);
    }
```

创建数组，赋值 3 个元素，数组的索引就是 15，25，35，没有 3 索引，因此我们不能访问数组中不存在的索引，程序运行后，将会抛出 **ArrayIndexOutOfBoundsException** 数组越界异常。在开发中，数组的越界异常是不能出现的，一旦出现了，就必须要修改我们编写的代码。

```
15
25
35
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 3
  at com.zjq.javabase.base05.demo03.Demo01ArrayIndex.main(Demo01ArrayIndex.java:25)
```

### 数组空指针异常

观察一下代码，运行后会出现什么结果。

```
public static void main(String[] args) {
        int[] array = null;

        System.out.println(array[0]);
    }
```

array= null 这行代码，意味着变量 arr 将不会在保存数组的内存地址，也就不允许再操作数组了，因此运行的时候会抛出 NullPointerException 空指针异常。在开发中，数组的越界异常是不能出现的，一旦出现了，就必须要修改我们编写的代码。

```
Exception in thread "main" java.lang.NullPointerException
  at com.zjq.javabase.base05.demo03.Demo02ArrayNull.main(Demo02ArrayNull.java:20)
```

### 数组遍历

数组遍历： 就是将数组中的每个元素分别获取出来，就是遍历。遍历也是数组操作中的基石。

```
public static void main(String[] args) {
    int[] arr = { 1, 2, 3, 4, 5 };
    System.out.println(arr[0]);
    System.out.println(arr[1]);
    System.out.println(arr[2]);
    System.out.println(arr[3]);
    System.out.println(arr[4]);
}
```

以上代码是可以将数组中每个元素全部遍历出来，但是如果数组元素非常多，这种写法肯定不行，因此我们需要改造成循环的写法。数组的索引是 0 到 lenght-1 ，可以作为循环的条件出现。

```
public static void main(String[] args) {
    int[] arr = { 1, 2, 3, 4, 5 };
    for (int i = 0; i < arr.length; i++) {
        System.out.println(arr[i]);
    }
}
```

### 数组获取最大值元素

最大值获取：从数组的所有元素中找出最大值。**实现思路**：

*   定义变量，保存数组 0 索引上的元素
    
*   遍历数组，获取出数组中的每个元素
    
*   将遍历到的元素和保存数组 0 索引上值的变量进行比较
    
*   如果数组元素的值大于了变量的值，变量记录住新的值
    
*   数组循环遍历结束，变量保存的就是数组中的最大值
    

代码实现如下：

```
public static void main(String[] args) {
        int[] array = { 5, 15, 30, 20, 10000, 30, 35 };

        int max = array[0]; 
        for (int i = 1; i < array.length; i++) {
            
            if (array[i] > max) {
                max = array[i];
            }
        }
        
        System.out.println("最大值：" + max);
    }
```

### 数组反转

**数组的反转**： 数组中的元素颠倒顺序，例如原始数组为 1,2,3,4,5，反转后的数组为 5,4,3,2,1 **实现思想**：数组最远端的元素互换位置。

*   实现反转，就需要将数组最远端元素位置交换
    
*   定义两个变量，保存数组的最小索引和最大索引
    
*   两个索引上的元素交换位置
    
*   最小索引++，最大索引--，再次交换位置
    
*   最小索引超过了最大索引，数组反转操作结束
    

代码实现：

```
public static void main(String[] args) {
    int[] arr = { 1, 2, 3, 4, 5 };
    
    循环中定义变量min=0最小索引
    max=arr.length‐1最大索引
    min++,max‐‐
    */
    for (int min = 0, max = arr.length ‐ 1; min <= max; min++, max‐‐) {
        
        int temp = arr[min];
        arr[min] = arr[max];
        arr[max] = temp;
    }
    
    for (int i = 0; i < arr.length; i++) {
        System.out.println(arr[i]);
    }
}
```

数组作为方法参数和返回值
------------

### 数组作为方法参数

以前的方法中我们学习了方法的参数和返回值，但是使用的都是基本数据类型。那么作为引用类型的数组能否作为方法的参数进行传递呢，当然是可以的。数组作为方法参数传递，传递的参数是数组内存的地址。

```
public static void main(String[] args) {
    int[] arr = { 1, 3, 5, 7, 9 };
    
    printArray(arr);
}

创建方法，方法接收数组类型的参数
进行数组的遍历
*/
public static void printArray(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        System.out.println(arr[i]);
    }
}
```

### 数组作为方法返回值

数组作为方法的返回值，返回的是数组的内存地址。

```
public static void main(String[] args) {
    
    
    int[] arr = getArray();
    for (int i = 0; i < arr.length; i++) {
        System.out.println(arr[i]);
    }
}

    创建方法，返回值是数组类型
    return返回数组的地址
*/
public static int[] getArray() {
    int[] arr = { 1, 3, 5, 7, 9 };
    
    return arr;
}
```

> 本文内容到此结束了，
> 
> 如有收获欢迎点赞👍收藏💖关注✔️，您的鼓励是我最大的动力
> 
> 如有错误❌疑问💬欢迎各位指出。
> 
> **主页**：[共饮一杯无的博客汇总👨‍💻](https://www.infoq.cn/u/zhanjq/publish)  
> 
> **保持热爱，奔赴下一场山海**。🏃🏃🏃