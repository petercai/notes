# 异常的基本概念和Java中的体系_InfoQ写作社区
🥞异常概念
------

异常，就是不正常的意思。在生活中:医生说,你的身体某个部位有异常,该部位和正常相比有点不同,该部位的功能将受影响.在程序中的意思就是：

*   **异常** ：指的是程序在执行过程中，出现的非正常的情况，最终会导致 JVM 的非正常停止。
    

在 Java 等面向对象的编程语言中，异常本身是一个类，产生异常就是创建异常对象并抛出了一个异常对象。Java 处理异常的方式是中断处理。

> 异常指的并不是语法错误,语法错了,编译不通过,不会产生字节码文件,根本不能运行.

🥪异常体系
------

异常机制其实是帮助我们**找到**程序中的问题，异常的根类是`java.lang.Throwable`，其下有两个子类：`java.lang.Error`与`java.lang.Exception`，平常所说的异常指`java.lang.Exception`。

![](https://static001.geekbang.org/infoq/c0/c09a28ba5e8197e2877ea7ac45df04a3.png)

**Throwable 体系：** 

*   **Error**:严重错误 Error，无法通过处理的错误，只能事先避免，好比绝症。
    
*   **Exception**:表示异常，异常产生后程序员可以通过代码的方式纠正，使程序继续运行，是必须要处理的。好比感冒、阑尾炎。
    

**Throwable 中的常用方法：** 

*   `public void printStackTrace()`:打印异常的详细信息。_包含了异常的类型,异常的原因,还包括异常出现的位置,在开发和调试阶段,都得使用 printStackTrace。_
    
*   `public String getMessage()`:获取发生异常的原因。_提示给用户的时候,就提示错误原因。_
    
*   `public String toString()`:获取异常的类型和异常描述信息(不用)。
    

_**出现异常,不要紧张,把异常的简单类名,拷贝到 API 中去查。** _

🍿异常分类
------

我们平常说的异常就是指 Exception，因为这类异常一旦出现，我们就要对代码进行更正，修复程序。**异常(Exception)的分类**:根据在编译时期还是运行时期去检查异常?

*   **编译时期异常**:checked 异常。在编译时期,就会检查,如果没有处理异常,则编译失败。(如日期格式化异常)
    
*   **运行时期异常**:runtime 异常。在运行时期,检查异常.在编译时期,运行异常不会编译器检测(不报错)。(如数学异常)
    

![](https://static001.geekbang.org/infoq/59/5976c61f20dd04f893e4f355b1af5b25.png)

🍝异常的产生过程解析
-----------

先运行下面的程序，程序会产生一个数组索引越界异常 ArrayIndexOfBoundsException。我们通过图解来解析下异常产生的过程。工具类

```
public class ArrayTools {
    
    public static int getElement(int[] arr, int index) {
        int element = arr[index];
        return element;
    }
}
```

测试类

```
public class ExceptionDemo {
    public static void main(String[] args) {
        int[] arr = { 34, 12, 67 };
        intnum = ArrayTools.getElement(arr, 4)
        System.out.println("num=" + num);
        System.out.println("over");
    }
}
```

上述程序执行过程图解：

![](https://static001.geekbang.org/infoq/a9/a9c705199ae91a807784988ec2cd2336.png)

> 本文内容到此结束了，
> 
> 如有收获欢迎点赞👍收藏💖关注✔️，您的鼓励是我最大的动力。
> 
> 如有错误❌疑问💬欢迎各位大佬指出。
> 
> **主页**：[共饮一杯无的博客汇总👨‍💻](https://www.infoq.cn/u/zhanjq/publish)  
> 
> **保持热爱，奔赴下一场山海**。🏃🏃🏃