# 日期时间类(Date、DateFormat、Calendar)_InfoQ写作社区
🏨Date 类
--------

### 概述

`java.util.Date`类 表示特定的瞬间，精确到毫秒。继续查阅 Date 类的描述，发现 Date 拥有多个构造函数，只是部分已经过时，但是其中有未过时的构造函数可以把毫秒值转成日期对象。

*   `public Date()`：分配 Date 对象并初始化此对象，以表示分配它的时间（精确到毫秒）。
    
*   `public Date(long date)`：分配 Date 对象并初始化此对象，以表示自从标准基准时间（称为“历元（epoch）”，即 1970 年 1 月 1 日 00:00:00 GMT）以来的指定毫秒数。
    

> 💡tips: 由于我们处于东八区，所以我们的基准时间为 1970 年 1 月 1 日 8 时 0 分 0 秒。

简单来说：使用无参构造，可以自动设置当前系统时间的毫秒时刻；指定 long 类型的构造参数，可以自定义毫秒时刻。例如：

```
import java.util.Date;

public class Demo01Date {
    public static void main(String[] args) {
        
        System.out.println(new Date()); 
        
        System.out.println(new Date(0L)); 
    }
}
```

> 💡tips:在使用 println 方法时，会自动调用 Date 类中的 toString 方法。Date 类对 Object 类中的 toString 方法进行了覆盖重写，所以结果为指定格式的字符串。

### 常用方法

Date 类中的多数方法已经过时，常用的方法有：

*   `public long getTime()` 把日期对象转换成对应的时间毫秒值。
    

🏪DateFormat 类
--------------

`java.text.DateFormat` 是日期/时间格式化子类的抽象类，我们通过这个类可以帮我们完成日期和文本之间的转换,也就是可以在 Date 对象与 String 对象之间进行来回转换。

*   **格式化**：按照指定的格式，从 Date 对象转换为 String 对象。
    
*   **解析**：按照指定的格式，从 String 对象转换为 Date 对象。
    

### 构造方法

由于 DateFormat 为抽象类，不能直接使用，所以需要常用的子类`java.text.SimpleDateFormat`。这个类需要一个模式（格式）来指定格式化或解析的标准。构造方法为：

*   `public SimpleDateFormat(String pattern)`：用给定的模式和默认语言环境的日期格式符号构造 SimpleDateFormat。
    

参数 pattern 是一个字符串，代表日期时间的自定义格式。

### 格式规则

常用的格式规则为：

> 📌备注：更详细的格式规则，可以参考[SimpleDateFormat类的API文档](https://xie.infoq.cn/link?target=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2F8%2Fdocs%2Fapi%2Fjava%2Ftext%2FSimpleDateFormat.html)。

创建 SimpleDateFormat 对象的代码如：

```
import java.text.DateFormat;
import java.text.SimpleDateFormat;

public class Demo02SimpleDateFormat {
    public static void main(String[] args) {
        
        DateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    }    
}
```

### 常用方法

DateFormat 类的常用方法有：

*   `public String format(Date date)`：将 Date 对象格式化为字符串。
    
*   `public Date parse(String source)`：将字符串解析为 Date 对象。
    

#### format 方法

使用 format 方法的代码为：

```
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;

 把Date对象转换成String
*/
public class Demo03DateFormatMethod {
    public static void main(String[] args) {
        Date date = new Date();
        
        DateFormat df = new SimpleDateFormat("yyyy年MM月dd日");
        String str = df.format(date);
        System.out.println(str); 
    }
}
```

#### parse 方法

使用 parse 方法的代码为：

```
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

 把String转换成Date对象
*/
public class Demo04DateFormatMethod {
    public static void main(String[] args) throws ParseException {
        DateFormat df = new SimpleDateFormat("yyyy年MM月dd日");
        String str = "2021年12月11日";
        Date date = df.parse(str);
        System.out.println(date); 
    }
}
```

🚂练习
----

请使用日期时间相关的 API，计算出一个人已经出生了多少天。**思路：** 

1.  获取当前时间对应的毫秒值
    
2.  获取自己出生日期对应的毫秒值
    
3.  两个时间相减（当前时间– 出生日期）
    

**代码实现：** 

```
public static void function() throws Exception {
  System.out.println("请输入出生日期 格式 yyyy-MM-dd");
  
  String birthdayString = new Scanner(System.in).next();
  
  
  SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
  
  Date birthdayDate = sdf.parse(birthdayString);  
  
  Date todayDate = new Date();  
  
  long birthdaySecond = birthdayDate.getTime();
  long todaySecond = todayDate.getTime();
  long secone = todaySecond-birthdaySecond;  
  if (secone < 0){
    System.out.println("还没出生呢");
  } else {
    System.out.println("出生的天数："+secone/1000/60/60/24);
  }
}
```

🏰Calendar 类
------------

### 概念

`java.util.Calendar`是日历类，在 Date 后出现，替换掉了许多 Date 的方法。该类将所有可能用到的时间信息封装为静态成员变量，方便获取。日历类就是方便获取各个时间属性的。

### 获取方式

Calendar 为抽象类，由于语言敏感性，Calendar 类在创建对象时并非直接创建，而是通过静态方法创建，返回子类对象，如下：Calendar 静态方法

*   `public static Calendar getInstance()`：使用默认时区和语言环境获得一个日历
    

例如：

```
import java.util.Calendar;

public class Demo06CalendarInit {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
    }    
}
```

### 常用方法

根据 Calendar 类的 API 文档，常用方法有：

*   `public int get(int field)`：返回给定日历字段的值。
    
*   `public void set(int field, int value)`：将给定的日历字段设置为给定值。
    
*   `public abstract void add(int field, int amount)`：根据日历的规则，为给定的日历字段添加或减去指定的时间量。
    
*   `public Date getTime()`：返回一个表示此 Calendar 时间值（从历元到现在的毫秒偏移量）的 Date 对象。
    

Calendar 类中提供很多成员常量，代表给定的日历字段：

#### get/set 方法

get 方法用来获取指定字段的值，set 方法用来设置指定字段的值，代码使用演示：

```
import java.util.Calendar;

public class CalendarUtil {
    public static void main(String[] args) {
        
        Calendar cal = Calendar.getInstance();
        
        int year = cal.get(Calendar.YEAR);
        
        int month = cal.get(Calendar.MONTH) + 1;
        
        int dayOfMonth = cal.get(Calendar.DAY_OF_MONTH);
        System.out.print(year + "年" + month + "月" + dayOfMonth + "日");
    }    
}
```

```
import java.util.Calendar;

public class Demo07CalendarMethod {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        cal.set(Calendar.YEAR, 2020);
        System.out.print(year + "年" + month + "月" + dayOfMonth + "日"); 
    }
}
```

#### add 方法

add 方法可以对指定日历字段的值进行加减操作，如果第二个参数为正数则加上偏移量，如果为负数则减去偏移量。代码如：

```
import java.util.Calendar;

public class Demo08CalendarMethod {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        System.out.print(year + "年" + month + "月" + dayOfMonth + "日"); 
        
        cal.add(Calendar.DAY_OF_MONTH, 2); 
        cal.add(Calendar.YEAR, -3); 
        System.out.print(year + "年" + month + "月" + dayOfMonth + "日"); 
    }
}
```

#### getTime 方法

Calendar 中的 getTime 方法并不是获取毫秒时刻，而是拿到对应的 Date 对象。

```
import java.util.Calendar;
import java.util.Date;

public class Demo09CalendarMethod {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        Date date = cal.getTime();
        System.out.println(date); 
    }
}
```

> 🚩小贴士：
> 
> 西方星期的开始为周日，中国为周一。
> 
> 在 Calendar 类中，月份的表示是以 0-11 代表 1-12 月。
> 
> 日期是有大小关系的，时间靠后，时间越大。

> 本文内容到此结束了，
> 
> 如有收获欢迎点赞👍收藏💖关注✔️，您的鼓励是我最大的动力。
> 
> 如有错误❌疑问💬欢迎各位大佬指出。
> 
> **主页**：[共饮一杯无的博客汇总👨‍💻](https://www.infoq.cn/u/zhanjq/publish)  
> 
> **保持热爱，奔赴下一场山海**。🏃🏃🏃