# Python高阶语法---函数_Python_木偶_InfoQ写作社区
**所谓函数就是一个代码块,函数是组织好的，可重复使用的，用来实现单一，或相关联功能的代码段。** 

1.函数的语法:
--------

2.函数规则:
-------

*   **函数代码块以 def 关键词开头，后接函数标识符名称(函数名)和圆括号 ();**
    
*   **任何传入参数和自变量必须放在圆括号中间，圆括号之间可以用于定义参数(传的参数一般为形参);**
    
*   **函数的第一行语句可以选择性地使用文档字符串—用于存放函数文档说明;**
    
*   **函数内容以冒号开始，并且注意行缩进;**
    
*   **return \[表达式\] 结束函数，选择性地返回一个值给调用方。不带表达式的 return 相当于返回 None。** 
    

3.函数文档说明:
---------

*   **函数文档说明是为了客观性的知道函数的各个部分是什么,具体是干什么的**
    

4.举例:
-----

```
def today(day):
    """
    这是用于测试的
    :param day: 输出内容
    :return: 中断程序
    """
    print("今天是:{0}".format(day))
    return
    print("您好世界")
today("周四")
```

**像这种,就不会继续执行**`**return**`**后的代码了,程序直接跳出;函数文档说明内容,解释了函数的作用**

![](https://static001.geekbang.org/infoq/f2/f2bfa3495fa0108294602f0a022212f1.png)

![](https://static001.geekbang.org/infoq/fd/fd351f63bd5fa85f45f107e7e07a811f.png)

* * *

**因为后面三个函数都离不开匿名函数,所以我们先来看看匿名函数。** 

*   ambda 只是一个表达式，但是函数体比 def 简单很多。
    
*   lambda 的主体是一个表达式，而不是一个代码块。仅仅能在 lambda 表达式中封装有限的逻辑进去。
    
*   lambda 函数拥有自己的命名空间，且不能访问自己参数列表之外或全局命名空间里的参数
    

1.匿名函数定义:
---------

**匿名函数:不需要显示的制定函数名,格式:**`**函数名=lambda 参数:返回值**`  

```
sum=lambda a,b:a+b
print("两数之和为:%d"%sum(10,20))
```

2.匿名函数作用:
---------

*   可实现标准函数的功能
    
*   可作为一个函数的参数进行传递
    

3.匿名函数的特点:
----------

*   本质上是一个函数表达式;
    
*   匿名函数只能写一行代码;
    
*   使用关键字`lambda`  
    
*   若有返回值,则不需要 return 关键字;
    
*   函数执行完成,这个匿名函数就被释放
    

4.匿名函数注意:
---------

*   lambda 中不可以使用 if、while、for 语句
    
*   lambda 中返回结果不需使用 return 返回值
    
*   lambda 主体是一个表达式不是代码块
    

5.举例(列表排序):
-----------

```
my_list=[
    [11,2,100],
    [20,5,10],
    [18,3,-11]
]
my_list.sort(key=lambda a:a[1])
print(my_list)
```

![](https://static001.geekbang.org/infoq/06/06f3a2da9a799d299a04c955b623e850.png)

* * *

**map() 会根据提供的函数对指定序列做映射**

![](https://static001.geekbang.org/infoq/eb/eb0bd7cddfbf3eccadc093cfb65cc994.png)

1.map 语法:
---------

**格式:**`**map(方法,参数序列)**`  

2.map 使用方法:
-----------

```
my_list=["smith","tom"]

print(list(map(lambda x:x.capitalize(),my_list )))
```

![](https://static001.geekbang.org/infoq/b7/b765a93754c15c61636c33f071a72419.png)

**reduce:对参数序列元素进行累计**

![](https://static001.geekbang.org/infoq/72/72e76a773fa4bf01cc6407ed00e0815e.png)

1.reduce 语法:
------------

**格式:**`**reduce(function, iterable)**`**,fuction:函数有两个参数,iterable:可迭代对象**

2.reduce 使用方法:
--------------

**在使用次函数时,务必注意首先需要进行导包操作:**`**from functools import reduce**`**,否则无法使用**

```
def add(x, y) :            
    return x + y
print(reduce(add,[1,2,3,4,5]))
```

![](https://static001.geekbang.org/infoq/15/15220f4cfbb6cf4da977f3e6b69093bf.png)

3.注意:
-----

*   reduce 把一个函数作用在一个序列\[x1, x2, x3, ...\]上，这个函数必须接收`两个`参数，
    
*   reduce 把结果继续和序列的下一个元素做累积计算，其效果就是：`reduce(func,[1,2,3])` 等同于 `func(func(1,2),3)`  
    

`**filter()**`**也接收一个函数和一个序列。和**`**map()**`**不同的是，**`**filter()**`**把传入的函数依次作用于每个元素，然后根据返回值是 True 还是 False 决定保留还是丢弃该元素。对于序列中的元素进行筛选，最终获取符合条件的序列,然后进行输出或者进行操作**

![](https://static001.geekbang.org/infoq/99/9969c0586a1da481c880fb16537ee3c3.png)

1.filter 语法:
------------

**filter():两个参数一个为函数,一个为序列,格式:**`**filter(function, iterable)**`**fuction:函数,iterable:可迭代对象**

2.filter 用法:
------------

20 个数中找到能被 3 整除的数

```
my_list2=[i for i in range(20)]
print(list(filter(lambda x:x%3==0,my_list2)))
```

![](https://static001.geekbang.org/infoq/d4/d4a04ba11f308178aeba00e695bbac3c.png)

**sort 函数适用于排序,详情请戳**[**列表之sort**](https://xie.infoq.cn/link?target=https%3A%2F%2Feditor.csdn.net%2Fmd%2F%3FarticleId%3D107356498)  

```
"""
map
"""
my_list=["smith","tom"]

print(list(map(lambda x:x.capitalize(),my_list )))

"""
reduce函数
reduce把一个函数作用在一个序列[x1, x2, x3, ...]上，这个函数必须接收两个参数，
reduce把结果继续和序列的下一个元素做累积计算，其效果就是：
reduce(func,[1,2,3]) 等同于 func(func(1,2),3)
"""
from functools import reduce
my_list1=[2,5,6,7]
print(reduce(lambda x,y:x+y,my_list1))
def add(x, y) :            
    return x + y
print(reduce(add,[1,2,3,4,5]))
"""
filter函数
filter()也接收一个函数和一个序列。和map()不同的是，filter()把传入的函数依次作用于每个元素，
然后根据返回值是True还是False决定保留还是丢弃该元素。
对于序列中的元素进行筛选，最终获取符合条件的序列
"""
my_list2=[i for i in range(20)]
print(list(filter(lambda x:x%3==0,my_list2)))
```

![](https://static001.geekbang.org/infoq/96/962413ceec7a5a7a60b3a3427f24ea95.png)