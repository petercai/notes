# Python进阶(二十九)Python时间&日期&时间戳处理_Python_No Silver Bullet_InfoQ写作社区
一、将字符串的时间转换为时间戳
---------------

方法:

```
import time

a = "2013-10-10 23:40:00"
timeArray = time.strptime(a, "%Y-%m-%d %H:%M:%S")

timeStamp = int(time.mktime(timeArray))

print(timeStamp)
```

二、字符串格式更改
---------

如 a = "2013-10-10 23:40:00",想改为 a = "2013/10/10 23:40:00"

方法:先转换为时间数组,然后转换为其他格式

```
a = "2013-10-10 23:40:00"
timeArray = time.strptime(a, "%Y-%m-%d %H:%M:%S")
otherStyleTime = time.strftime("%Y/%m/%d %H:%M:%S", timeArray)

print(otherStyleTime)
```

三、时间戳转换为指定格式日期
--------------

### 3.1 方法一

利用`localtime()`转换为时间数组,然后格式化为需要的格式,如

```
timeStamp = 1381419600
timeArray = time.localtime(timeStamp)
otherStyleTime=time.strftime("%Y-%m-%d %H:%M:%S", timeArray)

print(otherStyleTime)
```

### 3.2 方法二

```
import datetime
timeStamp = 1381419600
dateArray = datetime.datetime.utcfromtimestamp(timeStamp)
otherStyleTime = dateArray.strftime("%Y-%m-%d %H:%M:%S")

print(otherStyleTime)
```

注意：使用此方法时必须先设置好时区，否则有时差。

四、获取当前时间并转换为指定日期格式
------------------

### 4.1 方法一

```
import time
timeStamp = 1381419600

now = int(time.time())

timeArray = time.localtime(timeStamp)
otherStyleTime = time.strftime("%Y-%m-%d %H:%M:%S", timeArray)

print(otherStyleTime)
```

### 4.2 方法二

```
import datetime

now = datetime.datetime.now()

otherStyleTime = now.strftime("%Y-%m-%d %H:%M:%S")

print(otherStyleTime)
```

五、获得三天前的时间
----------

```
import time
import datetime

threeDayAgo=(datetime.datetime.now() - datetime.timedelta(days = 3))



otherStyleTime = threeDayAgo.strftime("%Y-%m-%d %H:%M:%S")

print(otherStyleTime)
```

注:`timedelta()`的参数有:`days,hours,seconds,microseconds`  

六、给定时间戳,计算该时间的几天前时间
-------------------

```
import datetime
import time
timeStamp = 1381419600

dateArray = datetime.datetime.utcfromtimestamp(timeStamp)
threeDayAgo = dateArray - datetime.timedelta(days = 3)

print(threeDayAgo)
```

七、给定日期字符串，直接转换为 datetime 对象
---------------------------

```
dateStr = '2013-10-10 23:40:00'
datetimeObj=datetime.datetime.strptime(dateStr, "%Y-%m-%d %H:%M:%S")
```

注：将字符串日期转换为`datetime`后可以很高效的进行统计操作，因为转换为`datetime`后，可以通过`datetime.timedelta()`方法来前后移动时间，效率很高，而且可读性很强。

八、计算两个 datetime 之间的差距
---------------------

```
a = datetime.datetime(2014,12,4,1,59,59)
b = datetime.datetime(2014,12,4,3,59,59)
diffSeconds = (b-a).total_seconds()

print(diffSeconds)
```

注：time.strftime，time.strptime，datetime.timedelta

九、Python time strftime()方法
--------------------------

Python time `strftime()` 函数接收以时间元组，并返回以可读字符串表示的当地时间，格式由参数 format 决定。

```
time.strftime(format[, t])
```

*   `format` -- 格式字符串。
    
*   `t` -- 可选的参数 t 是一个 struct\_time 对象。
    

返回以可读字符串表示的当地时间。

以下实例展示了 `strftime()` 函数的使用方法：

```
import time

t = (2009, 2, 17, 17, 3, 38, 1, 48, 0)
t = time.mktime(t)
print time.strftime("%b %d %Y %H:%M:%S", time.gmtime(t))
```

以上实例输出结果为：Feb 17 2009 09:03:38

十、Python time strptime()方法
--------------------------

Python time `strptime()` 函数根据指定的格式把一个时间字符串解析为时间元组。

```
time.strptime(string[, format])
```

*   `string` -- 时间字符串。
    
*   `format` -- 格式化字符串。
    

返回 struct\_time 对象。

以下实例展示了 `strptime()` 函数的使用方法：

```
import time

struct_time = time.strptime("30 Nov 00", "%d %b %y")
print "returned tuple: %s " % struct_time
```

以上实例输出结果为：returned tuple: (2000, 11, 30, 0, 0, 0, 3, 335, -1)

十一、datetime.timedelta
---------------------

`datetime.timedelta`对象代表两个时间之间的的时间差，两个`date`或`datetime`对象相减时可以返回一个`timedelta`对象。

构造函数：

```
class datetime.timedelta([days[, seconds[, microseconds[, milliseconds[, minutes[, hours[, weeks]]]]]]])
```

所有参数可选，且默认都是 0，参数的值可以是整数，浮点数，正数或负数。