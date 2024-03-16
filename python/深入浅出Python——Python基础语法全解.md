# 深入浅出Python——Python基础语法全解_Python_何极光_InfoQ写作社区
前言：

1.  Python 是一个高层次的结合了解释性、编译性、互动性和面向对象的脚本语言。
    
2.  本博文主要涉及 Python 基础入门、流程控制（条件语句、循环语句）、数据序列等知识。
    
3.  本博文若存在任何知识错误，请读者第一时间在评论区指出，方便我修正内容！
    
4.  获取更多内容，请关注博主，在主页进行阅读，感谢您的支持！
    

一、Python 简介
-----------

### 1\. 了解 Python

Python 是一种解释型（这意味着开发过程中没有了编译这个环节）、面向对象（支持面向对象的风格或代码封装在对象的编程技术）、动态数据类型的交互式（可在命令行中通过 Python 提示符及直接代码执行程序）高级程序设计语言。

![](_assets/b18e25c785124bb7af9b134107e92e67.png)

### 2\. Python 介绍

Python 是时下最流行、最火爆的编程语言，具体原因：

*   简单、易学，适应人群广泛
    
*   ![](_assets/20200630160133585.png)
    
*   免费、开源
    
*   应用领域广泛
    
*   ![](_assets/20200630160612457.png)
    

> 备注：以下知名框架均是 Python 语言开发。
> 
> 1.  Google 开源机器学习框架：TensorFlow
>     
> 2.  开源社区主推学习框架：Scikit-learn
>     
> 3.  百度开源深度学习框架：Paddle
>     

### 3\. Python 特点

1.  易于学习：Python 有相对较少的关键字，结构简单，和一个明确定义的语法，学习起来更加简单。
    
2.  易于阅读：Python 代码定义的更清晰。
    
3.  易于维护：Python 的成功在于它的源代码是相当容易维护的。
    
4.  一个广泛的标准库：Python 的最大的优势之一是丰富的库，跨平台的，在 UNIX，Windows 和 Macintosh 兼容很好。
    
5.  互动模式：互动模式的支持，您可以从终端输入执行代码并获得结果的语言，互动的测试和调试代码片断。
    
6.  可移植：基于其开放源代码的特性，Python 已经被移植（也就是使其工作）到许多平台。
    
7.  可扩展：如果你需要一段运行很快的关键代码，或者是想要编写一些不愿开放的算法，你可以使用 C 或 C++完成那部分程序，然后从你的 Python 程序中调用。
    
8.  数据库：Python 提供所有主要的商业数据库的接口。
    
9.  GUI 编程：Python 支持 GUI 可以创建和移植到许多系统调用。
    
10.  可嵌入: 你可以将 Python 嵌入到 C/C++程序，让你的程序的用户获得"脚本化"的能力。
    

### 4\. Python 发展历史

Python 发展历史：[https://baike.baidu.com/item/Python/407313?fr=aladdin](https://xie.infoq.cn/link?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FPython%2F407313%3Ffr%3Daladdin)  

### 5\. Python 版本

*   Python 2.X
    
*   Python 3.X
    

二、Python 解释器
------------

### 1\. 解释器的作用

*   Python 解释器作用：运行文件
    
*   Python 解释器种类
    
*   CPython，C 语言开发的解释器\[官方\]，应用广泛的解释器。
    
*   IPython，基于 CPython 的一种交互式解释器。
    
*   其他解释器
    
*   PyPy，基于 Python 语言开发的解释器。
    
*   Jython，运行在 Java 平台的解释器，直接把 Python 代码编译成 Java 字节码执行。
    
*   IronPython，运行在微软.Net 平台上的 Python 解释器，可以直接把 Python 代码编译成.Net 的字节码。
    

### 2\. 解释器的安装

Python 解释器的安装：[https://blog.csdn.net/weixin\_43495473/article/details/103559812](https://xie.infoq.cn/link?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_43495473%2Farticle%2Fdetails%2F103559812)  

三、PyCharm 安装与使用
---------------

### 1\. PyCharm 的作用

PyCharm 是一种 Python ==IDE==（集成开发环境），带有一整套可以帮助用户在使用 Python 语言开发时==提高其效率的工具==，内部集成的功能如下：

*   Project 管理
    
*   智能提示
    
*   语法高亮
    
*   代码跳转
    
*   调试代码
    
*   解释代码(解释器)
    
*   框架和库
    
*   ......
    

### 2\. PyCharm 安装与使用

PyCharm 的安装：[https://blog.csdn.net/weixin\_43495473/article/details/103560198](https://xie.infoq.cn/link?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_43495473%2Farticle%2Fdetails%2F103560198)  

四、注释
----

### 1\. 注释的作用

通过用自己熟悉的语言，在程序中对某些代码进行标注说明，这就是注释的作用，能够大大增强程序的可读性。

### 2\. 注释的分类及语法

注释分为两类：==单行注释== 和 ==多行注释==。

*   单行注释
    

只能注释一行内容，语法如下：

*   多行注释
    

可以注释多行内容，一般用在注释一段代码的情况， 语法如下：

```
"""
    第一行注释
    第二行注释
    第三行注释
"""

'''
    注释1
    注释2
    注释3
'''
```

> 快捷键： ==ctrl + /==

**注意：解释器不执行任何的注释内容。** 

五、变量
----

### 1\. 变量的作用

变量，简单来说，其实就是计算机内存中的一块储存空间，用来存储 CPU 需要使用的数据。 而这个储存空间需要一个名字，这个名字的统称就叫变量。

![](_assets/ccaf76eff41705a7332bb80ef9d30e3d.png)

作用：

1.  用来临时保存数据
    
2.  便于项目的后期升级维护
    
3.  节约内存
    

### 2\. 定义变量

**变量名自定义，要满足==标识符==命名规则。** 

#### 2.1 标识符

标识符命名规则是 Python 中定义各种名字的时候的统一规范，具体如下：

*   由数字、字母、下划线组成
    
*   不能数字开头
    
*   不能使用内置关键字
    
*   严格区分大小写
    

![](_assets/ecdf7bad60f930ae37da008652456da8.png)

#### 2.2 命名习惯

*   见名知义。
    
*   大驼峰：即每个单词首字母都大写，例如：`MyName`。
    
*   小驼峰：第二个（含）以后的单词首字母大写，例如：`myName`。
    
*   下划线：例如：`my_name`。
    

#### 2.3 使用变量

```
my_name = "张三"
print(my_name)

myAge = 23
print(myAge)
```

#### 2.4 认识 bug

所谓 bug，就是程序中的错误。如果程序有错误，需要程序员排查问题，纠正错误。

#### 2.5 Debug 工具

Debug 工具是 PyCharm IDE 中集成的用来调试程序的工具，在这里程序员可以查看程序的执行细节和流程或者调解 bug。

Debug 工具使用步骤：

1.  打断点
    
2.  Debug 调试
    

六、数据类型
------

**在 Python 里为了应对不同的业务需求，也把数据分为不同的类型。** 

![](_assets/0e319e7a70d8b6bb02938c7017142501.png)

> 检测数据类型的方法：`type(变量名或者数值)`  

```
a = 1
print(type(a))  

b = 1.1
print(type(b))  

c = True
print(type(c))  

d = "12345"
print(type(d))  

e = [10, 20, 30]
print(type(e))  

f = (10, 20, 30)
print(type(f))  

h = {10, 20, 30}
print(type(h))  

g = {"name": "张三", "age": 20}
print(type(g))
```

思考：

1.  为什么 Python 会提出这么多种的数据类型?
    
2.  > 有限的内存, 无限的变量, 合理的使用内存
    
3.  Python 中，程序员定义变量需要关心变量的类型吗?
    
4.  > *   Python 弱类型语言
    >     
    > *   程序员不需要关心变量的类型, 只需要把数据书写正确即可，python 会通过自动推到出您变量的类型。
    >     
    

七、输出
----

**作用：程序输出内容给用户。** 

```
print('hello Python')

age = 18
print(age)
```

### 1\. 格式化输出

> *   %06d，表示输出的整数显示位数，不足以 0 补全，超出当前位数则原样输出。
>     
> *   %.2f，表示小数点后显示的小数位数。
>     

### 2\. 内置函数 format

format()用于字符串格式化，功能非常强大，格式是 str.format()，format 函数可以接受不限个参数，位置可以不按顺序。

### 3\. f-格式化字符串

f-string 是格式化字符串的一种很好的新方法。与其他格式化方式相比，它不仅更易读，更简洁，不易出错，而且速度更快！格式为：`f'{表达式}'`。

### 4\. 体验格式化字符串

```
name = "张三"
age = 23
weight = 75.5
student_id = 1


print('我的名字是%s' % name)


print('我的学号是%04d' % student_id)


print('我的体重是%.2f公斤' % weight)


print('我的名字是%s，今年%d岁了' % (name, age))


print('我的名字是%s，明年%d岁了' % (name, age + 1))


print('我的名字是{0}, 明年{1}岁了'.format(name, age+1))


print('我的名字是{0}, 今年{1}岁了'.format("李思", 18))


print(f'我的名字是{name}, 明年{age + 1}岁了')
```

![](_assets/e737fd364706416beeedc7256025ec18.png)

### 5\. 补充知识

字符串输出的本质其实就是拼接字符串，那么我们都用`%s`完全也是可以的。很多时候，我们不用区分整型、浮点型等，直接用`%s`输出就可以了。

```
print('我的名字是%s，今年%s岁了，我的体重是%s公斤' % (name, age ,weight))
```

![](_assets/d5db840bbb72ee4fe861c05e0c9b0491.png)

### 6\. 转义字符

*   `\n`：换行。
    
*   `\t`：制表符，一个 tab 键（4 个空格）的距离。
    

### 7\. 结束符

> 在 Python 中，print()， 默认自带`end="\n"`这个换行结束符，所以导致每两个`print`直接会换行展示，用户可以按需求更改结束符。

```
print('内容', end=" ")
print('内容', end="···")
```

八、输入
----

**在 Python 中，程序接收用户输入的数据的功能即是输入。** 

![](_assets/593347c0f6d9474189447d021175d4e1.png)

### 1\. 输入的语法

### 2\. 输入的特点

*   当程序执行到`input`，等待用户输入，输入完成之后才继续向下执行。
    
*   在 Python 中，`input`接收用户输入后，一般存储到变量，方便使用。
    
*   在 Python 中，`input`会把接收到的任意用户输入的数据都当做字符串处理。
    

```
password = input('请输入您的密码：')

print(f'您输入的密码是{password}')

print(type(password))
```

![](_assets/131bc1a72da4834950b6c7be6a9e0dc0.png)

九、转换数据类型
--------

### 1\. 转换数据类型的作用

问：input()接收用户输入的数据都是字符串类型，如果用户输入 1，想得到整型该如何操作？

答：转换数据类型即可，即将字符串类型转换成整型。

### 2\. 转换数据类型的函数

### 3\. 体验转换数据类型

需求：input 接收用户输入，用户输入“1”，将这个数据 1 转换成整型。

```
num = input('请输入您的幸运数字：')


print(f"您的幸运数字是{num}")



print(type(num))


print(type(int(num)))
```

```
num1 = 1
print(float(num1))
print(type(float(num1)))


num2 = 10
print(type(str(num2)))


list1 = [10, 20, 30]
print(tuple(list1))
print(type(tuple(list1)))



t1 = (100, 200, 300)
print(list(t1))
print(type(list(t1)))


str1 = '10'
str2 = '[1, 2, 3]'
str3 = '(1000, 2000, 3000)'
print(type(eval(str1)))
print(type(eval(str2)))
print(type(eval(str3)))
```

十、运算符
-----

### 1\. 运算符的分类

*   算数运算符
    
*   赋值运算符
    
*   复合赋值运算符
    
*   比较运算符
    
*   逻辑运算符
    

### 2\. 算数运算符

> 混合运算优先级顺序：`()`高于 `**` 高于 `*` `/` `//` `%` 高于 `+` `-`  

### 3\. 赋值运算符

1.  单个变量赋值
    

2.  多个变量赋值
    

```
num1, float1, str1 = 10, 0.5, 'hello world'
print(num1)
print(float1)
print(str1)
```

3.  多变量赋相同值
    

```
a = b = 10
print(a)
print(b)
```

### 4\. 复合赋值运算符

```
a = 100
a += 1

print(a)

b = 2
b *= 3

print(b)

c = 10
c += 1 + 2

print(c)
```

### 5\. 比较运算符

**比较运算符也叫关系运算符， 通常用来判断。** 

```
a = 7
b = 5
print(a == b)  
print(a != b)  
print(a < b)   
print(a > b)   
print(a <= b)  
print(a >= b)
```

### 6\. 逻辑运算符

```
a = 1
b = 2
c = 3
print((a < b) and (b < c))  
print((a > b) and (b < c))  
print((a > b) or (b < c))   
print(not (a > b))
```

### 7\. 补充知识

数字之间的逻辑运算

```
a = 0
b = 1
c = 2


print(a and b)  
print(b and a)  
print(a and c)  
print(c and a)  
print(b and c)  
print(c and b)  


print(a or b)  
print(a or c)  
print(b or c)
```

十一、条件语句
-------

### 1\. 了解条件语句

Python 条件语句是通过一条或多条语句的执行结果（True 或者 False）来决定执行的代码块。

### 2\. if 语法

```
if 条件:
    条件成立执行的代码1
    条件成立执行的代码2
    ......
```

#### 2.1 体验 if 语句

```
if True:
    print('条件成立执行的代码1')
    print('条件成立执行的代码2')


print('我是无论条件是否成立都要执行的代码')
```

![](_assets/20200630183252892.png)

#### 2.2 简单版：网吧上网

需求分析：如果用户年龄大于等于 18 岁，即成年，输出"已经成年，可以上网"。

```
age = 20
if age >= 18:
    print('已经成年，可以上网')

print('系统关闭')
```

#### 2.3 进阶版：网吧上网

新增需求：用户可以输出自己的年龄，然后系统进行判断是否成年，成年则输出"您的年龄是'用户输入的年龄'，已经成年，可以上网"。

```
age = int(input('请输入您的年龄：'))

if age >= 18:
    print(f'您的年龄是{age},已经成年，可以上网')


print('系统关闭')
```

### 3\. if...else...

作用：条件成立执行 if 下方的代码; 条件不成立执行 else 下方的代码。

```
if 条件:
    条件成立执行的代码1
    条件成立执行的代码2
    ......
else:
    条件不成立执行的代码1
    条件不成立执行的代码2
    ......
```

> 思考：网吧上网的实例，如果成年，允许上网，如果不成年呢？是不是应该回复用户不能上网？

#### 3.1 实用版：网吧上网

```
age = int(input('请输入您的年龄：'))

if age >= 18:
    print(f'您的年龄是{age},已经成年，可以上网')
else:
    print(f'您的年龄是{age},未成年，请自行回家写作业')

print('系统关闭')
```

> 注意：如果条件成立执行了某些代码，那么其他的情况的代码将不会执行。

### 4\. 多重判断

```
if 条件1:
    条件1成立执行的代码1
    条件1成立执行的代码2
    ......
elif 条件2：
    条件2成立执行的代码1
    条件2成立执行的代码2
    ......
......
else:
    以上条件都不成立执行执行的代码
```

多重判断也可以和 else 配合使用。一般 else 放在整个 if 语句的最后，表示以上条件都不成立的时候执行的代码。

> 思考：中国合法工作年龄为 18-60 岁，即如果年龄小于 18 的情况为童工，不合法；如果年龄在 18-60 岁之间为合法工龄；大于 60 岁为法定退休年龄。

#### 4.1 实例：工龄判断

```
age = int(input('请输入您的年龄：'))
if age < 18:
    print(f'您的年龄是{age},童工一枚')
elif (age >= 18) and (age <= 60):
    print(f'您的年龄是{age},合法工龄')
elif age > 60:
    print(f'您的年龄是{age},可以退休')
```

> 拓展：`age >= 18 and age <= 60`可以化简为`18 <= age <= 60`。

### 5\. if 嵌套

```
if 条件1：
    条件1成立执行的代码
    条件1成立执行的代码
    
    if 条件2：
        条件2成立执行的代码
        条件2成立执行的代码
```

注意：条件 2 的 if 也是出于条件 1 的缩进关系内部。

> 思考：坐公交：如果有钱可以上车，没钱不能上车；上车后如果有空座，则可以坐下；如果没空座，就要站着。怎么书写程序？

#### 5.1 实例：坐公交

1.  判断是否能上车
    

```
"""
    1. 如果有钱，则可以上车
    2. 上车后，如果有空座，可以坐下
    上车后，如果没有空座，则站着等空座位
    如果没钱，不能上车
"""

money = 1
if money == 1:
    print('土豪，不差钱，顺利上车')
else:
    print('没钱，不能上车，追着公交车跑')
```

2.  判断是否能坐下
    

```
"""
    1. 如果有钱，则可以上车
    2. 上车后，如果有空座，可以坐下
    上车后，如果没有空座，则站着等空座位
    如果没钱，不能上车
"""

money = 1
seat = 0
if money == 1:
    print('土豪，不差钱，顺利上车')
    if seat == 1:
        print('有空座，可以坐下')
    else:
        print('没有空座，站等')
else:
    print('没钱，不能上车，追着公交车跑')
```

#### 5.2 if 嵌套执行流程

![](_assets/20200701094142253.png)

### 6\. 应用：猜拳游戏

需求分析：

*   参与游戏的角色
    
*   玩家
    
*   手动出拳
    
*   电脑
    
*   随机出拳
    
*   判断输赢
    
*   玩家获胜
    

*   平局
    
*   玩家出拳 和 电脑出拳相同
    
*   电脑获胜
    

随机做法：

1.  导出 random 模块
    

2.  使用 random 模块中的随机整数功能
    

#### 6.1 初始版：猜拳游戏

```
"""
提示：0-石头，1-剪刀，2-布
3. 出拳
玩家输入出拳
电脑随机出拳

4. 判断输赢
玩家获胜
平局
电脑获胜
"""


import random


computer = random.randint(0, 2)
print(computer)

player = int(input('请出拳：0-石头，1-剪刀，2-布：'))


if (player == 0 and computer == 1) or (player == 1 and computer == 2) or (player == 2 and computer == 0):
    print('玩家获胜')


elif player == computer:
    print('平局')
else:
    print('电脑获胜')
```

#### 6.2 进阶版：猜拳游戏

```
"""
石头剪刀布游戏
玩家VS电脑
站在玩家的角度，分析胜利、失败、平局
0-石头，1-剪刀，2-布
"""

import random

import sys

print("-----欢迎来到人机对战小游戏-----")
name = input("请输入您的游戏名：")
while True:
    stats = input("欢迎“%s”来到本游戏，开始游戏输入Y，退出游戏输入N，请您输入：" % name)
    if stats == "Y" or stats == "y":
        print("游戏开始……")
        print("石头输入0、剪刀输入1、布输入2")
        break
    elif stats == "N" or stats == "n":
        print("游戏结束……")
        sys.exit(0)
    else:
        print("请按照要求重新输入！")
print("-------------加载中-------------")


def Game(user, comp):
    if (user== 0 and comp== 1) or (user== 1 and comp== 2) or (user== 2 and comp== 0):
        print("机器输入%d，您赢了！" % comp)
    elif user == comp:
        print("机器输入%d，平局了！" % comp)
    else:
        print("机器输入%d，您输了！" % comp)
    res = input("重新游戏请输入X，退出游戏按任意键")
    if res == "X" or res == "x":
        return
    else:
        sys.exit(0)


while True:
    user = int(input("请您出拳，输入相应数字！"))
    if user == 0 or user == 1 or user == 2:
        comp = random.randint(0, 2)
        Game(user, comp)
    else:
        print("输入数字有误，请确认后，重新输入！")
```

### 7\. 三目运算符

**三目运算符也叫三元运算符。** 

语法如下：

快速体验：

```
a = 1
b = 2

c = a if a > b else b
print(c)
```

十二、循环简介
-------

### 1\. 循环的作用

**让代码更高效的重复执行。** 

### 2\. 循环的分类

在 Python 中，循环分为`while`和`for`两种，最终实现效果相同。

### 3\. while 的语法

```
while 条件:
    条件成立重复执行的代码1
    条件成立重复执行的代码2
    ......
```

### 4\. while 循环快速体验

需求：复现重复执行 10 次`print('Hello World')`。

分析：初始值是 0 次，终点是 10 次，重复做的事情输出“Hello World”。

```
i = 0
while i < 10:
    print('Hello World')
    i += 1

print('任务结束')
```

### 5\. while 的应用

#### 5.1 应用一：计算 1-100 累加和

分析：1-100 的累加和，即 1 + 2 + 3 + 4 +….，即前两个数字的相加结果 + 下一个数字( 前一个数字 + 1)。

```
i = 1
result = 0
while i <= 100:
    result += i
    i += 1


print(result)
```

> 注意：为了验证程序的准确性，可以先改小数值，验证结果正确后，再改成 1-100 做累加。

#### 5.2 应用二：计算 1-100 偶数累加和

分析：1-100 的偶数和，即 2 + 4 + 6 + 8....，得到偶数的方法如下：

*   偶数即是和 2 取余结果为 0 的数字，可以加入条件语句判断是否为偶数，为偶数则累加
    
*   初始值为 0 / 2 , 计数器每次累加 2
    

**方法一：条件判断和 2 取余数则累加**

```
i = 1
result = 0
while i <= 100:
    if i % 2 == 0:
        result += i
    i += 1


print(result)
```

**方法二：计数器控制**

```
i = 0
result = 0
while i <= 100:
    result += i
    i += 2


print(result)
```

### 6\. break 和 continue

**break 和 continue 是循环中满足一定条件退出循环的两种不同方式。** 

#### 6.1 理解

举例：一共吃 5 个苹果，吃完第一个，吃第二个…，这里"吃苹果"的动作是不是重复执行？

情况一：如果吃的过程中，吃完第三个吃饱了，则不需要再吃第 4 个和第五个苹果，即是吃苹果的动作停止，这里就是 break 控制循环流程，即==终止此循环==。

情况二：如果吃的过程中，吃到第三个吃出一个大虫子...,是不是这个苹果就不吃了，开始吃第四个苹果，这里就是 continue 控制循环流程，即==退出当前一次循环继而执行下一次循环代码==。

#### 6.2 break 实例

```
i = 1
while i <= 5:
    if i == 4:
        print(f'吃饱了不吃了')
        break
    print(f'吃了第{i}个苹果')
    i += 1
```

![](_assets/2020071818190989.png)

#### 6.3 continue 实例

```
i = 1
while i <= 5:
    if i == 3:
        print(f'大虫子，第{i}个不吃了')
        
        i += 1
        continue
    print(f'吃了第{i}个苹果')
    i += 1
```

![](_assets/20200718184445477.png)

### 7\. while 循环嵌套

#### 7.1 语法

```
while 条件1:
    条件1成立执行的代码
    ......
    while 条件2:
        条件2成立执行的代码
        ......
```

> 总结：所谓 while 循环嵌套，就是一个 while 里面嵌套一个 while 的写法，每个 while 和之前的基础语法是相同的。

#### 7.2 执行过程

当内部循环执行完成之后，再执行下一次外部循环的条件判断。

![](_assets/20200718185621834.png)

#### 7.3 while 循环嵌套应用

**应用一：打印星号（正方形）**

```
j = 0
while j <= 4:
    
    i = 0
    while i <= 4:
        
        print('*', end='')
        i += 1
    
    print()
    j += 1
```

![](_assets/20200718185904513.png)

**应用二：打印星号（三角形）**

```
j = 0
while j <= 4:
    
    i = 0
    
    while i <= j:
        print('*', end='')
        i += 1
    print()
    j += 1
```

![](_assets/20200718192702134.png)

**应用二：九九乘法表**

```
j = 1
while j <= 9:
    
    i = 1
    while i <= j:
        print(f'{i}*{j}={j*i}', end='\t')
        i += 1
    print()
    j += 1
```

![](_assets/20200718192817742.png)

### 8\. for 循环

#### 8.1 语法

```
for 临时变量 in 序列:
    重复执行的代码1
    重复执行的代码2
    ......
```

#### 8.2 快速体验

```
str1 = 'Hello World'
for i in str1:
    print(i)
```

![](_assets/20200718193008674.png)

#### 8.3 break

```
str1 = 'Hello World'
for i in str1:
    if i == 'e':
        print('遇到e不打印')
        break
    print(i)
```

![](_assets/20200718193130752.png)

#### 8.4 continue

```
str1 = 'Hello World'
for i in str1:
    if i == 'e':
        print('遇到e不打印')
        continue
    print(i)
```

![](_assets/20200718193216851.png)

十三、else
-------

循环可以和 else 配合使用，else 下方缩进的代码指的是==当循环正常结束之后要执行的代码==。

### 1\. while...else

#### 1.1 语法

```
while 条件:
    条件成立重复执行的代码
else:
    循环正常结束之后要执行的代码
```

#### 1.2 示例

```
i = 1
while i <= 5:
    print('Hello World')
    i += 1
else:
    print('执行完毕')
```

![](_assets/20200718193648276.png)

#### 1.3 退出循环的方式

1.  break 终止循环
    

```
i = 1
while i <= 5:
    if i == 3:
        print("提前结束")
        break
    print('Hello World')
    i += 1
else:
    print('执行完毕')
```

![](_assets/20200718193909137.png)

> 所谓 else 指的是循环正常结束之后要执行的代码，即如果是 break 终止循环的情况，else 下方缩进的代码将不执行。

2.  continue 控制循环
    

```
i = 1
while i <= 5:
    if i == 3:
        print("打断一下")
        i += 1
        continue
    print('Hello World')
    i += 1
else:
    print('执行完毕')
```

![](_assets/20200718194053236.png)

> 因为 continue 是退出当前一次循环，继续下一次循环，所以该循环在 continue 控制下是可以正常结束的，当循环结束后，则执行了 else 缩进的代码。

### 2\. for...else

#### 2.1 语法

```
for 临时变量 in 序列:
    重复执行的代码
    ...
else:
    循环正常结束之后要执行的代码
```

> 所谓 else 指的是循环正常结束之后要执行的代码，即如果是 break 终止循环的情况，else 下方缩进的代码将不执行。

#### 2.2 示例

```
str1 = 'Hello World'
for i in str1:
    print(i)
else:
    print('循环正常结束之后执行的代码')
```

![](_assets/20200718194325854.png)

#### 2.3 退出循环的方式

1.  break 终止循环
    

```
str1 = 'Hello World'
for i in str1:
    if i == 'e':
        print('遇到e不打印')
        break
    print(i)
else:
    print('循环正常结束之后执行的代码')
```

![](_assets/20200718194512861.png)

> 没有执行 else 缩进的代码。

2.  continue 控制循环
    

```
str1 = 'Hello World'
for i in str1:
    if i == 'e':
        print('遇到e不打印')
        continue
    print(i)
else:
    print('循环正常结束之后执行的代码')
```

![](_assets/20200718194628478.png)

> 因为 continue 是退出当前一次循环，继续下一次循环，所以该循环在 continue 控制下是可以正常结束的，当循环结束后，则执行了 else 缩进的代码。

十四、字符串
------

### 1\. 认识字符串

字符串是 Python 中最常用的数据类型。我们一般使用引号来创建字符串。创建字符串很简单，只要为变量分配一个值即可。

```
a = 'hello world'
b = "abcdefg"
print(type(a))
print(type(b))
```

> 注意：控制台显示结果为`<class 'str'>`， 即数据类型为 str(字符串)。

### 2\. 字符串特征

*   一对引号字符串
    

```
name1 = 'Tom'
name2 = "Rose"
```

*   三引号字符串
    

```
name3 = ''' Tom '''
name4 = """ Rose """
a = ''' i am Tom, 
        nice to meet you! '''

b = """ i am Rose, 
        nice to meet you! """
```

**注意：三引号形式的字符串支持换行。** 

> 思考：如果创建一个字符串 `I'm Tom`?

```
c = "I'm Tom"
d = 'I\'m Tom'
```

### 3\. 字符串输出

```
print('hello world')

name = 'Tom'
print('我的名字是%s' % name)
print(f'我的名字是{name}')
```

### 4\. 字符串输入

在 Python 中，使用`input()`接收用户输入。

```
name = input('请输入您的名字：')
print(f'您输入的名字是{name}')
print(type(name))

password = input('请输入您的密码：')
print(f'您输入的密码是{password}')
print(type(password))
```

![](_assets/20200718195918184.png)

### 5\. 下标

#### 5.1 概念

`“下标”`又叫`“索引”`，就是编号。比如火车座位号，座位号的作用：按照编号快速找到对应的座位。同理，下标的作用即是通过下标快速找到对应的数据。

#### 5.2 快速体验

需求：字符串`name = "abcdef"`，取到不同下标对应的数据。

*   代码
    

```
name = "abcdef"

print(name[1])
print(name[0])
print(name[2])
```

*   输出结果
    
*   ![](_assets/2020071820255627.png)
    

> 注意：下标从==0==开始。
> 
> ![](_assets/20200718202624144.png)

### 6\. 切片

切片是指对操作的对象截取其中一部分的操作。**字符串、列表、元组**都支持切片操作。

#### 6.1 语法

> 注意：
> 
> ```
> - 不包含结束位置下标对应的数据， 正负整数均可；
> 
> ```
> 
> 步长是选取间隔，正负整数均可，默认步长为 1。

#### 6.2 示例

```
name = "abcdefg"

print(name[2:5:1])  
print(name[2:5])  
print(name[:5])  
print(name[1:])  
print(name[:])  
print(name[::2])  
print(name[:-1])  
print(name[-4:-1])  
print(name[::-1])
```

### 7\. 常用操作方法

字符串的常用操作方法有查找、修改和判断三大类。

#### 7.1 查找

所谓字符串查找方法即是查找子串在字符串中的位置或出现的次数。

*   find()：检测某个子串是否包含在这个字符串中，如果在返回这个子串开始的位置下标，否则则返回-1。
    

1.  语法
    

```
字符串序列.find(子串, 开始位置下标, 结束位置下标)
```

> 注意：开始和结束位置下标可以省略，表示在整个字符串序列中查找。

2.  快速体验
    

```
mystr = "hello world and buran and list and Python"

print(mystr.find('and'))  
print(mystr.find('and', 15, 30))  
print(mystr.find('ands'))
```

*   index()：检测某个子串是否包含在这个字符串中，如果在返回这个子串开始的位置下标，否则则报异常。
    

1.  语法
    

```
字符串序列.index(子串, 开始位置下标, 结束位置下标)
```

> 注意：开始和结束位置下标可以省略，表示在整个字符串序列中查找。

2.  快速体验
    

```
mystr = "hello world and buran and list and Python"

print(mystr.index('and'))  
print(mystr.index('and', 15, 30))  
print(mystr.index('ands'))
```

*   rfind()： 和 find()功能相同，但查找方向为==右侧==开始。
    
*   rindex()：和 index()功能相同，但查找方向为==右侧==开始。
    
*   count()：返回某个子串在字符串中出现的次数。
    

1.  语法
    

```
字符串序列.count(子串, 开始位置下标, 结束位置下标)
```

> 注意：开始和结束位置下标可以省略，表示在整个字符串序列中查找。2. 快速体验

```
mystr = "hello world and buran and list and Python"

print(mystr.count('and'))  
print(mystr.count('ands'))  
print(mystr.count('and', 0, 20))
```

#### 7.2 修改

所谓修改字符串，指的就是通过函数的形式修改字符串中的数据。

*   replace()：替换字符串。
    

1.  语法
    

```
字符串序列.replace(旧子串, 新子串, 替换次数)
```

> 注意：替换次数如果查出子串出现次数，则替换次数为该子串出现次数。

2.  快速体验
    

```
mystr = "hello world and buran and list and Python"

print(mystr.replace('and', 'he'))

print(mystr.replace('and', 'he', 10))

print(mystr)
```

> 注意：数据按照是否能直接修改分为==可变类型==和==不可变类型==两种。字符串类型的数据修改的时候不能改变原有字符串，属于不能直接修改数据的类型即是不可变类型。

*   split()：按照指定字符分割字符串。
    

1.  语法
    

> 注意：num 表示的是分割字符出现的次数，即将来返回数据个数为 num+1 个。

2.  快速体验
    

```
mystr = "hello world and buran and list and Python"

print(mystr.split('and'))

print(mystr.split('and', 2))

print(mystr.split(' '))

print(mystr.split(' ', 2))
```

> 注意：如果分割字符是原有字符串中的子串，分割后则丢失该子串。

*   join()：用一个字符或子串合并字符串，即是将多个字符串合并为一个新的字符串。
    

1.  语法
    

2.  快速体验
    

```
list1 = ['hello', 'buran', 'list']
t1 = ('aa', 'b', 'cc', 'ddd')

print('_'.join(list1))

print('...'.join(t1))
```

*   capitalize()：将字符串第一个字符转换成大写。
    

```
mystr = "hello world and buran and list and Python"

print(mystr.capitalize())
```

> 注意：capitalize()函数转换后，只字符串第一个字符大写，其他的字符全都小写。

*   title()：将字符串每个单词首字母转换成大写。
    

```
mystr = "hello world and buran and list and Python"

print(mystr.title())
```

*   lower()：将字符串中大写转小写。
    

```
mystr = "hello world and buran and list and Python"

print(mystr.lower())
```

*   upper()：将字符串中小写转大写。
    

```
mystr = "hello world and buran and list and Python"

print(mystr.upper())
```

*   lstrip()：删除字符串左侧空白字符。
    
*   rstrip()：删除字符串右侧空白字符。
    
*   strip()：删除字符串两侧空白字符。
    
*   ljust()：返回一个原字符串左对齐,并使用指定字符(默认空格)填充至对应长度 的新字符串。
    
*   rjust()：返回一个原字符串右对齐,并使用指定字符(默认空格)填充至对应长度 的新字符串，语法和 ljust()相同。
    
*   center()：返回一个原字符串居中对齐,并使用指定字符(默认空格)填充至对应长度 的新字符串，语法和 ljust()相同。
    

#### 7.3 判断

所谓判断即是判断真假，返回的结果是布尔型数据类型：True 或 False。

*   startswith()：检查字符串是否是以指定子串开头，是则返回 True，否则返回 False。如果设置开始和结束位置下标，则在指定范围内检查。
    

1.  语法
    

```
字符串序列.startswith(子串, 开始位置下标, 结束位置下标)
```

2.  快速体验
    

```
mystr = "hello world and buran and list and Python"

print(mystr.startswith('hello'))

print(mystr.startswith('hello', 5, 20))
```

*   endswith()：：检查字符串是否是以指定子串结尾，是则返回 True，否则返回 False。如果设置开始和结束位置下标，则在指定范围内检查。
    

1.  语法
    

```
字符串序列.endswith(子串, 开始位置下标, 结束位置下标)
```

2.  快速体验
    

```
mystr = "hello world and buran and list and Python"

print(mystr.endswith('Python'))

print(mystr.endswith('python'))

print(mystr.endswith('Python', 2, 20))
```

*   isalpha()：如果字符串至少有一个字符并且所有字符都是字母则返回 True, 否则返回 False。
    

```
mystr1 = 'hello'
mystr2 = 'hello12345'

print(mystr1.isalpha())

print(mystr2.isalpha())
```

*   isdigit()：如果字符串只包含数字则返回 True 否则返回 False。
    

```
mystr1 = 'aaa12345'
mystr2 = '12345'

print(mystr1.isdigit())

print(mystr2.isdigit())
```

*   isalnum()：如果字符串至少有一个字符并且所有字符都是字母或数字则返 回 True,否则返回 False。
    

```
mystr1 = 'aaa12345'
mystr2 = '12345-'

print(mystr1.isalnum())

print(mystr2.isalnum())
```

*   isspace()：如果字符串中只包含空白，则返回 True，否则返回 False。
    

```
mystr1 = '1 2 3 4 5'
mystr2 = '     '

print(mystr1.isspace())

print(mystr2.isspace())
```

十五、列表
-----

### 1\. 列表的格式

```
[数据1, 数据2, 数据3, 数据4......]
```

**列表可以一次性有序存储多个不同类型的数据。** 

### 2\. 列表的常用操作

列表的作用是一次性存储多个数据，程序员可以对这些数据进行的操作有：增、删、改、查。

#### 2.1 查找

1.  下标
    

```
name_list = ['Tom', 'Lily', 'Rose']

print(name_list[0])  
print(name_list[1])  
print(name_list[2])
```

2.  函数
    

*   index()：返回指定数据所在位置的下标 。
    

```
列表序列.index(数据, 开始位置下标, 结束位置下标)
```

```
name_list = ['Tom', 'Lily', 'Rose']

print(name_list.index('Lily', 0, 2))
```

> 注意：如果查找的数据不存在则报错。

*   count()：统计指定数据在当前列表中出现的次数。
    

```
name_list = ['Tom', 'Lily', 'Rose']

print(name_list.count('Lily'))
```

*   len()：访问列表长度，即列表中数据的个数。
    

```
name_list = ['Tom', 'Lily', 'Rose']

print(len(name_list))
```

#### 2.2 判断是否存在

*   in：判断指定数据在某个列表序列，如果在返回 True，否则返回 False。
    

```
name_list = ['Tom', 'Lily', 'Rose']


print('Lily' in name_list)


print('Lilys' in name_list)
```

*   not in：判断指定数据不在某个列表序列，如果不在返回 True，否则返回 False。
    

```
name_list = ['Tom', 'Lily', 'Rose']


print('Lily' not in name_list)


print('Lilys' not in name_list)
```

#### 2.2 增加

**作用：增加指定数据到列表中。** 

*   append()：列表结尾追加数据。
    

```
name_list = ['Tom', 'Lily', 'Rose']

name_list.append('xiaoming')


print(name_list)
```

> 列表追加数据的时候，直接在原列表里面追加了指定数据，即修改了原列表，故列表为可变类型数据。

**注意点：如果 append()追加的数据是一个序列，则追加整个序列到列表。** 

```
name_list = ['Tom', 'Lily', 'Rose']

name_list.append(['xiaoming', 'xiaohong'])


print(name_list)
```

*   extend()：列表结尾追加数据，如果数据是一个序列，则将这个序列的数据逐一添加到列表。
    

单个数据：

```
name_list = ['Tom', 'Lily', 'Rose']

name_list.extend('xiaoming')


print(name_list)
```

序列数据：

```
name_list = ['Tom', 'Lily', 'Rose']

name_list.extend(['xiaoming', 'xiaohong'])


print(name_list)
```

*   insert()：指定位置新增数据。
    

```
name_list = ['Tom', 'Lily', 'Rose']

name_list.insert(1, 'xiaoming')


print(name_list)
```

#### 2.3 删除

*   del
    

删除列表：

```
name_list = ['Tom', 'Lily', 'Rose']


del name_list
print(name_list)
```

删除指定数据：

```
name_list = ['Tom', 'Lily', 'Rose']

del name_list[0]


print(name_list)
```

*   pop()：删除指定下标的数据(默认为最后一个)，并返回该数据。
    

```
name_list = ['Tom', 'Lily', 'Rose']

del_name = name_list.pop(1)


print(del_name)


print(name_list)
```

*   remove()：移除列表中某个数据的第一个匹配项。
    

```
name_list = ['Tom', 'Lily', 'Rose']

name_list.remove('Rose')


print(name_list)
```

*   clear()：清空列表
    

```
name_list = ['Tom', 'Lily', 'Rose']

name_list.clear()
print(name_list)
```

#### 2.4 修改

*   修改指定下标数据
    

```
name_list = ['Tom', 'Lily', 'Rose']

name_list[0] = 'aaa'


print(name_list)
```

*   逆置：reverse()
    

```
num_list = [1, 5, 2, 3, 6, 8]

num_list.reverse()


print(num_list)
```

*   排序：sort()
    

```
列表序列.sort( key=None, reverse=False)
```

```
num_list = [1, 5, 2, 3, 6, 8]

num_list.sort()


print(num_list)
```

> 注意：reverse 表示排序规则，**reverse = True** 降序， **reverse = False** 升序（默认）

#### 2.5 复制

*   函数：copy()
    

```
name_list = ['Tom', 'Lily', 'Rose']

name_li2 = name_list.copy()


print(name_li2)
```

### 3\. 列表的循环遍历

需求：依次打印列表中的各个数据。

#### 3.1 while

```
name_list = ['Tom', 'Lily', 'Rose']

i = 0
while i < len(name_list):
    print(name_list[i])
    i += 1
```

#### 3.2 for

```
name_list = ['Tom', 'Lily', 'Rose']

for i in name_list:
    print(i)
```

### 4\. 列表嵌套

所谓列表嵌套指的就是一个列表里面包含了其他的子列表。

应用场景：要存储班级一、二、三三个班级学生姓名，且每个班级的学生姓名在一个列表。

```
name_list = [['小明', '小红', '小绿'], ['Tom', 'Lily', 'Rose'], ['张三', '李四', '王五']]
```

> 思考： 如何查找到数据"李四"？

```
print(name_list[2])


print(name_list[2][1])
```

十六、元组
-----

**一个元组可以存储多个数据，元组内的数据是不能修改的。** 

### 1\. 定义元组

元组特点：定义元组使用==小括号==，且==逗号==隔开各个数据，数据可以是不同的数据类型。

```
t1 = (10, 20, 30)


t2 = (10,)
```

> 注意：如果定义的元组只有一个数据，那么这个数据后面也好添加逗号，否则数据类型为唯一的这个数据的数据类型。

```
t2 = (10,)
print(type(t2))  

t3 = (20)
print(type(t3))  

t4 = ('hello')
print(type(t4))
```

### 2\. 元组的常见操作

元组数据不支持修改，只支持查找，具体如下：

*   按下标查找数据
    

```
tuple1 = ('aa', 'bb', 'cc', 'bb')
print(tuple1[0])
```

*   index()：查找某个数据，如果数据存在返回对应的下标，否则报错，语法和列表、字符串的 index 方法相同。
    

```
tuple1 = ('aa', 'bb', 'cc', 'bb')
print(tuple1.index('aa'))
```

*   count()：统计某个数据在当前元组出现的次数。
    

```
tuple1 = ('aa', 'bb', 'cc', 'bb')
print(tuple1.count('bb'))
```

*   len()：统计元组中数据的个数。
    

```
tuple1 = ('aa', 'bb', 'cc', 'bb')
print(len(tuple1))
```

### 3\. 元组的注意事项

**元组内的直接数据如果修改则立即报错。** 

```
tuple1 = ('aa', 'bb', 'cc', 'bb')
tuple1[0] = 'aaa'
```

**但是如果元组里面有列表，修改列表里面的数据则是支持的，故自觉很重要。** 

```
tuple2 = (10, 20, ['aa', 'bb', 'cc'], 50, 30)
print(tuple2[2])  


tuple2[2][0] = 'aaaaa'
print(tuple2)
```

十七、字典
-----

**字典里面的数据是以“键值对”形式出现，字典数据和数据顺序没有关系，即字典不支持下标，后期无论数据如何变化，只需要按照对应的键的名字查找数据即可。** 

### 1\. 创建字典的语法

字典特点：

*   符号为==大括号==
    
*   数据为==键值对==形式出现
    
*   各个键值对之间用==逗号==隔开
    

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}


dict2 = {}

dict3 = dict()
```

> 注意：一般称冒号前面的为键(key)，简称 k；冒号后面的为值(value)，简称 v。

### 2\. 字典常见操作

#### 2.1 增

写法：==字典序列\[key\] = 值==

> 注意：如果 key 存在则修改这个 key 对应的值；如果 key 不存在则新增此键值对。

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}

dict1['name'] = 'Rose'

print(dict1)

dict1['id'] = 110


print(dict1)
```

> 注意：字典为可变类型。

#### 2.2 删

*   del() / del：删除字典或删除字典中指定键值对。
    

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}

del dict1['gender']

print(dict1)
```

*   clear()：清空字典
    

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}

dict1.clear()
print(dict1)
```

#### 2.3 改

写法：==字典序列\[key\] = 值==

> 注意：如果 key 存在则修改这个 key 对应的值 ；如果 key 不存在则新增此键值对。

#### 2.4 查

1.  key 值查找
    

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
print(dict1['name'])  
print(dict1['id'])
```

> 如果当前查找的 key 存在，则返回对应的值；否则则报错。2. get()

> 注意：如果当前查找的 key 不存在则返回第二个参数(默认值)，如果省略第二个参数，则返回 None。

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
print(dict1.get('name'))  
print(dict1.get('id', 110))  
print(dict1.get('id'))
```

3.  keys()
    

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
print(dict1.keys())
```

4.  values()
    

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
print(dict1.values())
```

5.  items()
    

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
print(dict1.items())
```

### 3\. 字典的循环遍历

#### 3.1 遍历字典的 key

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
for key in dict1.keys():
    print(key)
```

#### 3.2 遍历字典的 value

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
for value in dict1.values():
    print(value)
```

#### 3.3 遍历字典的元素

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
for item in dict1.items():
    print(item)
```

#### 3.4 遍历字典的键值对

```
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
for key, value in dict1.items():
    print(f'{key} = {value}')
```

十八、集合
-----

**集合具有确定性，无序性和去重性。** 

### 1\. 创建集合

创建集合使用`{}`或`set()`， 但是如果要创建空集合只能使用`set()`，因为`{}`用来创建空字典。

```
s1 = {10, 20, 30, 40, 50}
print(s1)

s2 = {10, 30, 20, 10, 30, 40, 30, 50}
print(s2)

s3 = set('abcdefg')
print(s3)

s4 = set()
print(type(s4))  

s5 = {}
print(type(s5))
```

> 特点：
> 
> 1.  集合可以去掉重复数据；
>     
> 2.  集合数据是无序的，故不支持下标。
>     

### 2\. 集合常见操作方法

#### 2.1 增加数据

*   add()
    

```
s1 = {10, 20}
s1.add(100)
s1.add(10)
print(s1)
```

> 因为集合有去重功能，所以，当向集合内追加的数据是当前集合已有数据的话，则不进行任何操作。

*   update(), 追加的数据是序列。
    

```
s1 = {10, 20}

s1.update([100, 200])
s1.update('abc')
print(s1)
```

#### 2.2 删除数据

*   remove()，删除集合中的指定数据，如果数据不存在则报错。
    

```
s1 = {10, 20}

s1.remove(10)
print(s1)

s1.remove(10)  
print(s1)
```

*   discard()，删除集合中的指定数据，如果数据不存在也不会报错。
    

```
s1 = {10, 20}

s1.discard(10)
print(s1)

s1.discard(10)
print(s1)
```

*   pop()，随机删除集合中的某个数据，并返回这个数据。
    

```
s1 = {10, 20, 30, 40, 50}

del_num = s1.pop()
print(del_num)
print(s1)
```

#### 2.3 查找数据

*   in：判断数据在集合序列
    
*   not in：判断数据不在集合序列
    

```
s1 = {10, 20, 30, 40, 50}

print(10 in s1)
print(10 not in s1)
```

十九、公共操作
-------

### 1\. 运算符

#### 1.1 +

```
str1 = 'aa'
str2 = 'bb'
str3 = str1 + str2
print(str3)  



list1 = [1, 2]
list2 = [10, 20]
list3 = list1 + list2
print(list3)  


t1 = (1, 2)
t2 = (10, 20)
t3 = t1 + t2
print(t3)
```

#### 1.2 \*

```
print('-' * 10)  


list1 = ['hello']
print(list1 * 4)  


t1 = ('world',)
print(t1 * 4)
```

#### 1.3 in 或 not in

```
print('a' in 'abcd')  
print('a' not in 'abcd')  


list1 = ['a', 'b', 'c', 'd']
print('a' in list1)  
print('a' not in list1)  


t1 = ('a', 'b', 'c', 'd')
print('aa' in t1)  
print('aa' not in t1)
```

### 2\. 公共方法

#### 2.1 len()

```
str1 = 'abcdefg'
print(len(str1))  


list1 = [10, 20, 30, 40]
print(len(list1))  


t1 = (10, 20, 30, 40, 50)
print(len(t1))  


s1 = {10, 20, 30}
print(len(s1))  


dict1 = {'name': 'Rose', 'age': 18}
print(len(dict1))
```

#### 2.2 del()

```
str1 = 'abcdefg'
del str1
print(str1)


list1 = [10, 20, 30, 40]
del(list1[0])
print(list1)
```

#### 2.3 max()

```
str1 = 'abcdefg'
print(max(str1))  


list1 = [10, 20, 30, 40]
print(max(list1))
```

#### 2.4 min()

```
str1 = 'abcdefg'
print(min(str1))  


list1 = [10, 20, 30, 40]
print(min(list1))
```

#### 2.5 range()

```
for i in range(1, 10, 1):
    print(i)


for i in range(1, 10, 2):
    print(i)


for i in range(10):
    print(i)
```

> 注意：range()生成的序列不包含 end 数字。

#### 2.6 enumerate()

```
enumerate(可遍历对象, start=0)
```

> 注意：start 参数用来设置遍历数据的下标的起始值，默认为 0。

```
list1 = ['a', 'b', 'c', 'd', 'e']

for i in enumerate(list1):
    print(i)

for index, char in enumerate(list1, start=1):
    print(f'下标是{index}, 对应的字符是{char}')
```

![](_assets/20200724200540723.png)

### 3\. 容器类型转换

#### 3.1 tuple()

**作用：将某个序列转换成元组。** 

```
list1 = [10, 20, 30, 40, 50, 20]
s1 = {100, 200, 300, 400, 500}

print(tuple(list1))
print(tuple(s1))
```

#### 3.2 list()

**作用：将某个序列转换成列表。** 

```
t1 = ('a', 'b', 'c', 'd', 'e')
s1 = {100, 200, 300, 400, 500}

print(list(t1))
print(list(s1))
```

#### 3.3 set()

**作用：将某个序列转换成集合。** 

```
list1 = [10, 20, 30, 40, 50, 20]
t1 = ('a', 'b', 'c', 'd', 'e')

print(set(list1))
print(set(t1))
```

> 注意：
> 
> 1.  集合可以快速完成列表去重。
>     
> 2.  集合不支持下标。
>     

二十、推导式
------

### 1\. 列表推导式

**作用：用一个表达式创建一个有规律的列表或控制一个有规律列表。** 

\==列表推导式又叫列表生成式。==

#### 1.1 快速体验

需求：创建一个 0-10 的列表。

*   while 循环实现
    

```
list1 = []


i = 0
while i < 10:
    list1.append(i)
    i += 1

print(list1)
```

*   for 循环实现
    

```
list1 = []
for i in range(10):
    list1.append(i)

print(list1)
```

*   列表推导式实现
    

```
list1 = [i for i in range(10)]
print(list1)
```

#### 1.2 带 if 的列表推导式

需求：创建 0-10 的偶数列表

*   方法一：range()步长实现
    

```
list1 = [i for i in range(0, 10, 2)]
print(list1)
```

*   方法二：if 实现
    

```
list1 = [i for i in range(10) if i % 2 == 0]
print(list1)
```

#### 1.3 多个 for 循环实现列表推导式

需求：创建列表如下：

```
[(1, 0), (1, 1), (1, 2), (2, 0), (2, 1), (2, 2)]
```

*   代码如下：
    

```
list1 = [(i, j) for i in range(1, 3) for j in range(3)]
print(list1)
```

### 2\. 字典推导式

**字典推导式作用：快速合并列表为字典或提取字典中目标数据。** 

#### 2.1 快速体验

1.  创建一个字典：字典 key 是 1-5 数字，value 是这个数字的 2 次方。
    

```
dict1 = {i: i**2 for i in range(1, 5)}
print(dict1)
```

2.  将两个列表合并为一个字典
    

```
list1 = ['name', 'age', 'gender']
list2 = ['Tom', 20, 'man']

dict1 = {list1[i]: list2[i] for i in range(len(list1))}
print(dict1)
```

3.  提取字典中目标数据
    

```
counts = {'MBP': 268, 'HP': 125, 'DELL': 201, 'Lenovo': 199, 'acer': 99}


count1 = {key: value for key, value in counts.items() if value >= 200}
print(count1)
```

### 3\. 集合推导式

需求：创建一个集合，数据为下方列表的 2 次方。

代码如下：

```
list1 = [1, 1, 2]
set1 = {i ** 2 for i in list1}
print(set1)
```

> 注意：集合有数据去重功能。

_**补充：接下来的 Python 高级语法：函数、文件操作、面向对象、模块、包、异常等知识，我将陆续更新在自己的博客之中。如果本博文的知识能帮助到您的 Python 学习，本博主实属荣幸。更多后续知识，请关注博主，方便您第一时间查看，谢谢！**_

![](_assets/f38a2fb7c53af8734007d274f8da08e5.jpeg.jpg)

# 深入浅出Python——Python高级语法之函数_Python_何极光_InfoQ写作社区
前言：本博文主要讲解 Python 函数的用法，属于 Python 高级语法。基础语法见：[https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a](https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a)，更多内容请访问博主的主页，谢谢！

一. 函数的作用
--------

函数就是将一段具有独立功能的代码块 整合到一个整体并命名，在需要的位置调用这个名称即可完成对应的需求。

> 函数在开发过程中，可以更高效的实现代码重用。

二. 函数的使用步骤
----------

### 1\. 定义函数

```
def 函数名(参数):
    代码1
    代码2
    ......
```

### 2\. 调用函数

注意：

1.  不同的需求，参数可有可无。
    
2.  在 Python 中，函数必须先定义后使用。
    

\==一定是先定义函数，后调用函数。==

三. 函数的参数作用
----------

思考：完成需求如下：一个函数完成两个数 1 和 2 的加法运算，如何书写程序？

```
def add_num1():
    result = 1 + 2
    print(result)



add_num1()
```

思考：上述 add\_num1 函数只能完成数字 1 和 2 的加法运算，如果想要这个函数变得更灵活，可以计算任何用户指定的两个数字的和，如何书写程序？

分析：用户要在调用函数的时候指定具体数字，那么在定义函数的时候就需要接收用户指定的数字。函数调用时候指定的数字和定义函数时候接收的数字即是函数的参数。

```
def add_num2(a, b):
    result = a + b
    print(result)



add_num2(10, 20)
```

**函数的参数：函数调用的时候可以传入真实数据，增大函数的使用的灵活性。** 

*   形参：函数定义时书写的参数(非真实数据)
    
*   实参：函数调用时书写的参数(真实数据)
    

四. 函数的返回值作用
-----------

例如：我们去超市购物，比如买烟，给钱之后，是不是售货员会返回给我们烟这个商品，在函数中，如果需要返回结果给用户需要使用函数返回值。

```
def buy():
    return '烟'


goods = buy()
print(goods)
```

需求：制作一个计算器，计算任意两数字之和，并保存结果。

```
def sum_num(a, b):
    return a + b



result = sum_num(1, 2)
print(result)
```

**函数的返回值**

*   作用：函数调用后，返回需要的计算结果
    
*   写法
    

五. 函数的说明文档
----------

思考：定义一个函数后，程序员如何书写程序能够快速提示这个函数的作用？

答：注释

思考：如果代码多，我们是不是需要在很多代码中找到这个函数定义的位置才能看到注释？如果想更方便的查看函数的作用怎么办？

答：函数的说明文档

> 函数的说明文档也叫函数的文档说明。

### 1\. 语法

*   定义函数的说明文档
    

```
def 函数名(参数):
    """ 说明文档的位置 """
    代码
    ......
```

*   查看函数的说明文档
    

### 2\. 快速体验

```
def sum_num(a, b):
    """ 求和函数 """
    return a + b


help(sum_num)
```

![](_assets/2d50279af7ea79c34ca880be1aaa0d8b.png)

**函数的说明文档**

*   作用：保存函数解释说明的信息
    
*   写法
    

```
def 函数名():
    """ 函数说明文档 """
```

六. 函数嵌套调用
---------

所谓函数嵌套调用指的是==一个函数里面又调用了另外一个函数==。

*   示例
    

```
def testB():
    print('---- testB start----')
    print('这里是testB函数执行的代码...(省略)...')
    print('---- testB end----')

def testA():
    print('---- testA start----')
    testB()
    print('---- testA end----')

testA()
```

*   效果
    

![](_assets/fca0adbc5a658de9d0af17023bb96a7e.png)

*   执行流程
    
*   ![](_assets/20200727075957983.png)
    

> 如果函数 A 中，调用了另外一个函数 B，那么先把函数 B 中的任务都执行完毕之后才会回到上次 函数 A 执行的位置。

七. 函数应用
-------

### 1\. 打印图形

1.  打印一条横线
    

```
def print_line():
    print('-' * 20)


print_line()
```

![](_assets/ac49995f5e410ff4776c0a5f556b00ee.png)

2.  打印多条横线
    

```
def print_line():
    print('-' * 20)


def print_lines(num):
    i = 0
    while i < num:
        print_line()
        i += 1


print_lines(5)
```

![](_assets/20200727080308284.png)

### 2\. 函数计算

1.  求三个数之和
    

```
def sum_num(a, b, c):
    return a + b + c


result = sum_num(1, 2, 3)
print(result)
```

2.  求三个数平均值
    

```
def average_num(a, b, c):
    sumResult = sum_num(a, b, c)
    return sumResult / 3

result = average_num(1, 2, 3)
print(result)
```

八. 变量作用域
--------

变量作用域指的是变量生效的范围，主要分为两类：==局部变量==和==全局变量==。

*   局部变量
    

所谓局部变量是定义在函数体内部的变量，即只在函数体内部生效。

```
def testA():
    a = 100

    print(a)


testA()  
print(a)
```

> 变量 a 是定义在`testA`函数内部的变量，在函数外部访问则立即报错。

局部变量的作用：在函数体内部，临时保存数据，即当函数调用完成后，则销毁局部变量。

全局变量：指的是在函数体内、外都能生效的变量。

> 思考：如果有一个数据，在函数 A 和函数 B 中都要使用，该怎么办？
> 
> 答：将这个数据存储在一个全局变量里面。

```
a = 100


def testA():
    print(a)  


def testB():
    print(a)  


testA()  
testB()
```

思考：`testB`函数需求修改变量 a 的值为 200，如何修改程序？

```
a = 100


def testA():
    print(a)


def testB():
    a = 200
    print(a)


testA()  
testB()  
print(f'全局变量a = {a}')
```

思考：在`testB`函数内部的`a = 200`中的变量 a 是在修改全局变量`a`吗？

答：不是。观察上述代码发现，15 行得到 a 的数据是 100，仍然是定义全局变量 a 时候的值，而没有返回

`testB`函数内部的 200。综上：`testB`函数内部的`a = 200`是定义了一个局部变量。

思考：如何在函数体内部修改全局变量？

```
a = 100


def testA():
    print(a)


def testB():
    
    global a
    a = 200
    print(a)


testA()  
testB()  
print(f'全局变量a = {a}')
```

九. 多函数程序执行流程
------------

一般在实际开发过程中，一个程序往往由多个函数（后面知识中会讲解类）组成，并且多个函数共享某些数据，如下所示：

*   共用全局变量
    

```
glo_num = 0


def test1():
    global glo_num
    
    glo_num = 100


def test2():
    
    print(glo_num)
    


test1()

test2()
```

*   返回值作为参数传递
    

```
def test1():
    return 50


def test2(num):
    print(num)



result = test1()



test2(result)
```

十. 函数的返回值
---------

思考：如果一个函数如些两个 return (如下所示)，程序如何执行？

```
def return_num():
    return 1
    return 2


result = return_num()
print(result)
```

答：只执行了第一个 return，原因是因为 return 可以退出当前函数，导致 return 下方的代码不执行。

思考：如果一个函数要有多个返回值，该如何书写代码？

```
def return_num():
    return 1, 2


result = return_num()
print(result)
```

> 注意：
> 
> 1.  `return a, b`写法，返回多个数据的时候，默认是元组类型。
>     
> 2.  return 后面可以连接列表、元组或字典，以返回多个值。
>     

十一. 函数的参数
---------

### 1\. 位置参数

位置参数：调用函数时根据函数定义的参数位置来传递参数。

```
def user_info(name, age, gender):
    print(f'您的名字是{name}, 年龄是{age}, 性别是{gender}')


user_info('TOM', 20, '男')
```

> 注意：传递和定义参数的顺序及个数必须一致。

### 2\. 关键字参数

函数调用，通过“键=值”形式加以指定。可以让函数更加清晰、容易使用，同时也清除了参数的顺序需求。

```
def user_info(name, age, gender):
    print(f'您的名字是{name}, 年龄是{age}, 性别是{gender}')


user_info('Rose', age=20, gender='女')
user_info('小明', gender='男', age=16)
```

注意：**函数调用时，如果有位置参数时，位置参数必须在关键字参数的前面，但关键字参数之间不存在先后顺序。** 

### 3\. 缺省参数

缺省参数也叫默认参数，用于定义函数，为参数提供默认值，调用函数时可不传该默认参数的值（注意：所有位置参数必须出现在默认参数前，包括函数定义和调用）。

```
def user_info(name, age, gender='男'):
    print(f'您的名字是{name}, 年龄是{age}, 性别是{gender}')


user_info('TOM', 20)
user_info('Rose', 18, '女')
```

> 注意：函数调用时，如果为缺省参数传值则修改默认参数值；否则使用这个默认值。

### 4\. 不定长参数

不定长参数也叫可变参数。用于不确定调用的时候会传递多少个参数(不传参也可以)的场景。此时，可用包裹(packing)位置参数，或者包裹关键字参数，来进行参数传递，会显得非常方便。

*   包裹位置传递
    

```
def user_info(*args):
    print(args)



user_info('TOM')

user_info('TOM', 18)
```

> 注意：传进的所有参数都会被 args 变量收集，它会根据传进参数的位置合并为一个元组(tuple)，args 是元组类型，这就是包裹位置传递。

*   包裹关键字传递
    

```
def user_info(**kwargs):
    print(kwargs)



user_info(name='TOM', age=18, id=110)
```

> 综上：无论是包裹位置传递还是包裹关键字传递，都是一个组包的过程。

十二. 拆包和交换变量值
------------

### 1\. 拆包

*   拆包：元组
    

```
def return_num():
    return 100, 200


num1, num2 = return_num()
print(num1)  
print(num2)
```

*   拆包：字典
    

```
dict1 = {'name': 'TOM', 'age': 18}
a, b = dict1


print(a)  
print(b)  

print(dict1[a])  
print(dict1[b])
```

### 2\. 交换变量值

需求：有变量`a = 10`和`b = 20`，交换两个变量的值。

*   方法一
    

借助第三变量存储数据。

```
c = 0


c = a


a = b


b = c

print(a)  
print(b)
```

*   方法二
    

```
a, b = 1, 2
a, b = b, a
print(a)  
print(b)
```

十三. 引用
------

### 1\. 了解引用

在 python 中，值是靠引用来传递来的。

**我们可以用**`**id()**`**来判断两个变量是否为同一个值的引用。**  我们可以将 id 值理解为那块内存的地址标识。

```
a = 1
b = a

print(b)  

print(id(a))  
print(id(b))  

a = 2
print(b)  

print(id(a))  
print(id(b))  



aa = [10, 20]
bb = aa

print(id(aa))  
print(id(bb))  


aa.append(30)
print(bb)  

print(id(aa))  
print(id(bb))
```

### 2\. 引用当做实参

代码如下：

```
def test1(a):
    print(a)
    print(id(a))

    a += a

    print(a)
    print(id(a))



b = 100
test1(b)


c = [11, 22]
test1(c)
```

![](_assets/20200727170928179.png)

十四. 可变和不可变类型
------------

所谓可变类型与不可变类型是指：数据能够直接进行修改，如果能直接修改那么就是可变，否则是不可变。

*   可变类型
    
*   列表
    
*   字典
    
*   集合
    
*   不可变类型
    
*   整型
    
*   浮点型
    
*   布尔
    
*   字符串
    
*   元组
    

十五. 总结
------

*   变量作用域
    
*   全局：函数体内外都能生效
    
*   局部：当前函数体内部生效
    
*   函数多返回值写法
    

*   函数的参数
    
*   位置参数
    
*   形参和实参的个数和书写顺序必须一致
    
*   关键字参数
    
*   写法： `key=value`  
    
*   特点：形参和实参的书写顺序可以不一致；关键字参数必须书写在位置参数的后面
    
*   缺省参数
    
*   缺省参数就是默认参数
    
*   写法：`key=vlaue`  
    
*   不定长位置参数
    
*   收集所有位置参数，返回一个元组
    
*   不定长关键字参数
    
*   收集所有关键字参数，返回一个字典
    

> 引用：Python 中，数据的传递都是通过引用。

\==友情链接：==

1.  [深究Python中的递归【建议收藏】](https://xie.infoq.cn/link?target=https%3A%2F%2Fluckylifes.blog.csdn.net%2Farticle%2Fdetails%2F107682376)  
    
2.  [浅谈Python匿名函数（lambda表达式）](https://xie.infoq.cn/link?target=https%3A%2F%2Fluckylifes.blog.csdn.net%2Farticle%2Fdetails%2F107697395)  
    
3.  [浅谈Python高阶函数](https://xie.infoq.cn/link?target=https%3A%2F%2Fluckylifes.blog.csdn.net%2Farticle%2Fdetails%2F107698729)  
    
4.  [Python学员管理系统【函数实现】](https://xie.infoq.cn/link?target=https%3A%2F%2Fluckylifes.blog.csdn.net%2Farticle%2Fdetails%2F107681516)  
    
5.  [深入浅出Python——Python高级语法之文件操作](https://xie.infoq.cn/link?target=https%3A%2F%2Fluckylifes.blog.csdn.net%2Farticle%2Fdetails%2F107699888)

# 深入浅出Python——Python高级语法之异常、模块、包_Python_何极光_InfoQ写作社区
前言：本博文主要讲解 Python 异常、模块、包，属于 Python 高级语法。基础语法见：[https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a](https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a)，更多内容请访问博主的主页，谢谢！

一、了解异常
------

当检测到一个错误时，解释器就无法继续执行了，反而出现了一些错误的提示，这就是所谓的"异常"。

例如：以`r`方式打开一个不存在的文件。

```
FileNotFoundError: [Errno 2] No such file or directory: 'test.txt'
```

二、异常的写法
-------

### 1\. 语法

```
try:
    可能发生错误的代码
except:
    如果出现异常执行的代码
```

### 2\. 快速体验

需求：尝试以`r`模式打开文件，如果文件不存在，则以`w`方式打开。

```
try:
    f = open('test.txt', 'r')
except:
    f = open('test.txt', 'w')
```

### 3\. 捕获指定异常

#### 3.1 语法

```
try:
    可能发生错误的代码
except 异常类型:
    如果捕获到该异常类型执行的代码
```

#### 3.2 体验

```
try:
    print(num)
except NameError:
    print('有错误')
```

> 注意：
> 
> 1.  如果尝试执行的代码的异常类型和要捕获的异常类型不一致，则无法捕获异常。
>     
> 2.  一般 try 下方只放一行尝试执行的代码。
>     

#### 3.3 捕获多个指定异常

当捕获多个异常时，可以把要捕获的异常类型的名字，放到`except`后，并使用元组的方式进行书写。

```
try:
    print(1/0)

except (NameError, ZeroDivisionError):
    print('有错误')
```

#### 3.4 捕获异常描述信息

```
try:
    print(num)
except (NameError, ZeroDivisionError) as result:
    print(result)
```

#### 3.5 捕获所有异常

`Exception`是所有程序异常类的父类。

```
try:
    print(num)
except Exception as result:
    print(result)
```

### 4\. 异常的 else

`else`表示的是如果没有异常要执行的代码。

```
try:
    print(1)
except Exception as result:
    print(result)
else:
    print('我是else，是没有异常的时候执行的代码')
```

### 5\. 异常的 finally

`finally`表示的是无论是否异常都要执行的代码，例如关闭文件。

```
try:
    f = open('test.txt', 'r')
except Exception as result:
    f = open('test.txt', 'w')
else:
    print('没有异常，真开心')
finally:
    f.close()
```

三、异常的传递
-------

需求：

​ 1. 尝试只读方式打开 test.txt 文件，如果文件存在则读取文件内容，文件不存在则提示用户即可。

​ 2. 读取内容要求：尝试循环读取内容，读取过程中如果检测到用户意外终止程序，则`except`捕获异常并提示用户。

```
import time
try:
    f = open('test.txt')
    try:
        while True:
            content = f.readline()
            if len(content) == 0:
                break
            time.sleep(2)
            print(content)
    except:
        
        
        print('意外终止了读取数据')
    finally:
        f.close()
        print('关闭文件')
except:
    print("没有这个文件")
```

四、自定义异常
-------

在 Python 中，抛出自定义异常的语法为 `raise 异常类对象`。

需求：密码长度不足，则报异常（用户输入密码，如果输入的长度不足 3 位，则报错，即抛出自定义异常，并捕获该异常）。

```
class ShortInputError(Exception):
    def __init__(self, length, min_len):
        self.length = length
        self.min_len = min_len

    
    def __str__(self):
        return f'你输入的长度是{self.length}, 不能少于{self.min_len}个字符'


def main():
    try:
        con = input('请输入密码：')
        if len(con) < 3:
            raise ShortInputError(len(con), 3)
    except Exception as result:
        print(result)
    else:
        print('密码已经输入完成')


main()
```

五、模块
----

Python 模块(Module)，是一个 Python 文件，以 .py 结尾，包含了 Python 对象定义和 Python 语句。

模块能定义函数，类和变量，模块里也能包含可执行的代码。

### 1\. 导入模块

#### 1.1 导入模块的方式

*   import 模块名
    
*   import 模块名 1, 模块名 2（不建议使用，非 PEP8 规范）
    
*   from 模块名 import 功能名
    
*   from 模块名 import \*
    
*   import 模块名 as 别名
    
*   from 模块名 import 功能名 as 别名
    

#### 1.2 导入方式详解

##### 1.2.1 import

*   语法
    

```
import 模块名
import 模块名1, 模块名2...


模块名.功能名()
```

*   体验
    

```
import math
print(math.sqrt(9))
```

##### 1.2.2 from..import..

*   语法
    

```
from 模块名 import 功能1, 功能2, 功能3...
```

*   体验
    

```
from math import sqrt
print(sqrt(9))
```

##### 1.2.3 from..import \*

*   语法
    

*   体验
    

```
from math import *
print(sqrt(9))
```

##### 1.2.4 from..import \*

*   语法
    

```
import 模块名 as 别名


from 模块名 import 功能 as 别名
```

*   体验
    

```
import time as tt

tt.sleep(2)
print('hello')


from time import sleep as sl
sl(2)
print('hello')
```

### 2\. 制作模块

在 Python 中，每个 Python 文件都可以作为一个模块，模块的名字就是文件的名字。**也就是说自定义模块名必须要符合标识符命名规则。** 

#### 2.1 定义模块

新建一个 Python 文件，命名为`my_module1.py`，并定义`testA`函数。

```
def testA(a, b):
    print(a + b)
```

#### 2.2 测试模块

在实际开中，当一个开发人员编写完一个模块后，为了让模块能够在项目中达到想要的效果，这个开发人员会自行在 py 文件中添加一些测试信息.，例如，在`my_module1.py`文件中添加测试代码。

```
def testA(a, b):
    print(a + b)


testA(1, 1)
```

此时，无论是当前文件，还是其他已经导入了该模块的文件，在运行的时候都会自动执行`testA`函数的调用。

解决办法如下：

```
def testA(a, b):
    print(a + b)


if __name__ == '__main__':
    testA(1, 1)
```

#### 2.3 调用模块

```
import my_module1
my_module1.testA(1, 1)
```

#### 2.4 调用模块

如果使用`from .. import ..`或`from .. import *`导入多个模块的时候，且模块内有同名功能。当调用这个同名功能的时候，调用到的是后面导入的模块的功能。

```
def my_test(a, b):
    print(a + b)


def my_test(a, b):
    print(a - b)
   

from my_module1 import my_test
from my_module2 import my_test


my_test(1, 1)
```

### 3\. 模块定位顺序

当导入一个模块，Python 解析器对模块位置的搜索顺序是：

1.  当前目录
    
2.  如果不在当前目录，Python 则搜索在 shell 变量 PYTHONPATH 下的每个目录。
    
3.  如果都找不到，Python 会察看默认路径。UNIX 下，默认路径一般为/usr/local/lib/python/
    

模块搜索路径存储在 system 模块的 sys.path 变量中。变量里包含当前目录，PYTHONPATH 和由安装过程决定的默认目录。

*   注意
    
*   自己的文件名不要和已有模块名重复，否则导致模块功能无法使用
    
*   `使用from 模块名 import 功能`的时候，如果功能名字重复，调用到的是最后定义或导入的功能。
    

### 4\. `__all__`  

如果一个模块文件中有`__all__`变量，当使用`from xxx import *`导入时，只能导入这个列表中的元素。

*   my\_module1 模块代码
    

```
__all__ = ['testA']


def testA():
    print('testA')


def testB():
    print('testB')
```

*   导入模块的文件代码
    

```
from my_module1 import *
testA()
testB()
```

六、包
---

包将有联系的模块组织在一起，即放到同一个文件夹下，并且在这个文件夹创建一个名字为`__init__.py` 文件，那么这个文件夹就称之为包。

### 1\. 制作包

\[New\] — \[Python Package\] — 输入包名 — \[OK\] — 新建功能模块(有联系的模块)。

注意：新建包后，包内部会自动创建`__init__.py`文件，这个文件控制着包的导入行为。

#### 1.1 快速体验

1.  新建包`mypackage`  
    
2.  新建包内模块：`my_module1` 和 `my_module2`  
    
3.  模块内代码如下
    

```
print(1)


def info_print1():
    print('my_module1')
```

```
print(2)


def info_print2():
    print('my_module2')
```

#### 1.2 导入包

##### 1.2.1 方法一

```
import my_package.my_module1

my_package.my_module1.info_print1()
```

##### 1.2.2 方法二

注意：必须在`__init__.py`文件中添加`__all__ = []`，控制允许导入的模块列表。

```
from my_package import *

my_module1.info_print1()
```

七、总结
----

### 1\. 异常

*   异常语法
    

```
try:
      可能发生异常的代码
except:
      如果出现异常执行的代码
else:
      没有异常执行的代码
finally:
      无论是否异常都要执行的代码
```

*   捕获异常
    

```
except 异常类型:
      代码

except 异常类型 as xx:
        代码
```

*   自定义异常
    

```
class 异常类类名(Exception):
      代码
    
    
    def __str__(self):
      return ...



raise 异常类名()


except Exception...
```

### 2\. 模块、包

*   导入模块方法
    

```
import 模块名

from 模块名 import 目标

from 模块名 import *
```

*   导入包
    

```
import 包名.模块名

from 包名 import *
```

*   `__all__ = []` ：允许导入的模块或功能列表

# 深入浅出Python——Python高级语法之文件操作_Python_何极光_InfoQ写作社区
前言：本博文主要讲解 Python 文件操作的写法，属于 Python 高级语法。基础语法见：[https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a](https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a)，更多内容请访问博主的主页，谢谢！

一、文件操作的作用
---------

思考：文件操作包含什么？

答：打开、关闭、读、写、复制....

思考：文件操作的的作用是什么？

答：读取内容、写入内容、备份内容......

> 总结：文件操作的作用就是==把一些内容(数据)存储存放起来，可以让程序下一次执行的时候直接使用，而不必重新制作一份，省时省力==。

二、文件的基本操作
---------

### 1\. 文件操作步骤

1.  打开文件
    
2.  读写等操作
    
3.  关闭文件
    

> 注意：可以只打开和关闭文件，不进行任何读写操作。

#### 1.1 打开

在 python，使用 open 函数，可以打开一个已经存在的文件，或者创建一个新文件，语法如下：

name：是要打开的目标文件名的字符串(可以包含文件所在的具体路径)。

mode：设置打开文件的模式(访问模式)：只读、写入、追加等。

##### 1.1.1 打开文件模式

##### 1.1.2 快速体验

```
f = open('test.txt', 'w')
```

> 注意：此时的`f`是`open`函数的文件对象。

#### 2.1 文件对象方法

##### 1.2.1 写

*   语法
    

*   体验
    

```
f = open('test.txt', 'w')


f.write('hello world')


f.close()
```

> 注意：
> 
> 1.  `w` 和`a`模式：如果文件不存在则创建该文件；如果文件存在，`w`模式先清空再写入，`a`模式直接末尾追加。
>     
> 2.  `r`模式：如果文件不存在则报错。
>     

##### 1.2.2 读

*   read()
    

> num 表示要从文件中读取的数据的长度（单位是字节），如果没有传入 num，那么就表示读取文件中所有的数据。

*   readlines()
    

readlines 可以按照行的方式把整个文件中的内容进行一次性读取，并且返回的是一个列表，其中每一行的数据为一个元素。

```
f = open('test.txt')
content = f.readlines()


print(content)


f.close()
```

*   readline()
    

readline()一次读取一行内容。

```
f = open('test.txt')

content = f.readline()
print(f'第一行：{content}')

content = f.readline()
print(f'第二行：{content}')


f.close()
```

##### 1.2.3 seek()

作用：用来移动文件指针。

语法如下：

> 起始位置：
> 
> *   0：文件开头
>     
> *   1：当前位置
>     
> *   2：文件结尾
>     

#### 3.1 关闭

三、文件备份
------

需求：用户输入当前目录下任意文件名，程序完成对该文件的备份功能(备份文件名为 xx\[备份\]后缀，例如：test\[备份\].txt)。

### 1\. 步骤

1.  接收用户输入的文件名
    
2.  规划备份文件名
    
3.  备份文件写入数据
    

### 2\. 代码实现

1.  接收用户输入目标文件名
    

```
old_name = input('请输入您要备份的文件名：')
```

2.  规划备份文件名
    
3.  2.1 提取目标文件后缀
    
4.  2.2 组织备份的文件名，xx\[备份\]后缀
    

```
index = old_name.rfind('.')






new_name = old_name[:index] + '[备份]' + old_name[index:]
```

3.  备份文件写入数据
    
4.  3.1 打开源文件 和 备份文件
    
5.  3.2 将源文件数据写入备份文件
    
6.  3.3 关闭文件
    

```
old_f = open(old_name, 'rb')
new_f = open(new_name, 'wb')


while True:
    con = old_f.read(1024)
    if len(con) == 0:
        break
    new_f.write(con)


old_f.close()
new_f.close()
```

### 3\. 思考

如果用户输入`.txt`，这是一个无效文件，程序如何更改才能限制只有有效的文件名才能备份？

答：添加条件判断即可。

```
old_name = input('请输入您要备份的文件名：')

index = old_name.rfind('.')


if index > 0:
    postfix = old_name[index:]

new_name = old_name[:index] + '[备份]' + postfix

old_f = open(old_name, 'rb')
new_f = open(new_name, 'wb')

while True:
    con = old_f.read(1024)
    if len(con) == 0:
        break
    new_f.write(con)

old_f.close()
new_f.close()
```

四、文件和文件夹的操作
-----------

在 Python 中文件和文件夹的操作要借助 os 模块里面的相关功能，具体步骤如下：

1.  导入 os 模块
    

2.  使用`os`模块相关功能
    

### 1\. 文件重命名

### 2\. 删除文件

### 3\. 创建文件夹

### 4\. 删除文件夹

### 5\. 获取当前目录

### 6\. 改变默认目录

### 7\. 获取目录列表

五、应用案例
------

需求：批量修改文件名，既可添加指定字符串，又能删除指定字符串。

*   步骤
    

1.  设置添加删除字符串的的标识
    
2.  获取指定目录的所有文件
    
3.  将原有文件名添加/删除指定字符串，构造新名字
    
4.  os.rename()重命名
    

*   代码
    

```
import os


flag = 1


dir_name = './'


file_list = os.listdir(dir_name)




for name in file_list:

    
    if flag == 1:
        new_name = 'Python-' + name
    
    elif flag == 2:
        num = len('Python-')
        new_name = name[num:]

    
    print(new_name)
    
    
    os.rename(dir_name+name, dir_name+new_name)
```

六、总结
----

*   文件操作步骤
    
*   打开
    
*   操作
    
*   读
    
*   写
    
*   seek()
    
*   关闭
    
*   主访问模式
    
*   w：写，文件不存在则新建该文件
    
*   r：读，文件不存在则报错
    
*   a：追加
    
*   文件和文件夹操作
    
*   重命名：`os.rename()`  
    
*   获取当前目录：`os.getcwd()`  
    
*   获取目录列表：`os.listdir()`

# 深入浅出Python——Python高级语法之面向对象_Python_何极光_InfoQ写作社区
前言：本博文主要讲解 Python 面向对象，属于 Python 高级语法。基础语法见：[https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a](https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a)，更多内容请访问博主的主页，谢谢！

一、理解面向对象
--------

面向对象是一种抽象化的编程思想，很多编程语言中都有的一种思想。

例如：洗衣服

思考：几种途径可以完成洗衣服？

答： 手洗 和 机洗。

手洗：找盆 - 放水 - 加洗衣粉 - 浸泡 - 搓洗 - 拧干水 - 倒水 - 漂洗 N 次 - 拧干 - 晾晒。

机洗：打开洗衣机 - 放衣服 - 加洗衣粉 - 按下开始按钮 - 晾晒。

思考：对比两种洗衣服途径，同学们发现了什么？

答：机洗更简单

思考：机洗，只需要找到一台洗衣机，加入简单操作就可以完成洗衣服的工作，而不需要关心洗衣机内部发生了什么事情。

> 总结：==面向对象就是将编程当成是一个事物，对外界来说，事物是直接使用的，不用去管他内部的情况。而编程就是设置事物能够做什么事。==

二、类和对象
------

思考：洗衣机洗衣服描述过程中，洗衣机其实就是一个事物，即对象，洗衣机对象哪来的呢？

答：洗衣机是由工厂工人制作出来。

思考：工厂工人怎么制作出的洗衣机？

答：工人根据设计师设计的功能图纸制作洗衣机。

总结：图纸 → 洗衣机 → 洗衣服。

在面向对象编程过程中，有两个重要组成部分：==类== 和 ==对象==。

\==类和对象的关系：用类去创建一个对象。==

### 1\. 理解类和对象

#### 1.1 类

类是对一系列具有相同==特征==和==行为==的事物的统称，是一个==抽象的概念==，不是真实存在的事物。

*   特征即是属性
    
*   行为即是方法
    

类比如是制造洗衣机时要用到的图纸，也就是说==类是用来创建对象==。

#### 1.2 对象

对象是类创建出来的真实存在的事物，例如：洗衣机。

> 注意：开发中，先有类，再有对象。

### 2\. 面向对象实现方法

#### 2.1 定义类

Python2 中类分为：经典类 和 新式类

*   语法
    

> 注意：类名要满足标识符命名规则，同时遵循==大驼峰命名习惯==。

*   体验
    

```
class Washer():
    def wash(self):
        print('我会洗衣服')
```

*   拓展：经典类
    

不由任意内置类型派生出的类，称之为经典类

#### 2.2 定义类

对象又名实例。

*   语法
    

*   体验
    

```
haier1 = Washer()


print(haier1)


haier1.wash()
```

> 注意：创建对象的过程也叫实例化对象。

#### 2.3 self

self 指的是调用该函数的对象。

```
class Washer():
    def wash(self):
        print('我会洗衣服')
        
        print(self)



haier1 = Washer()

print(haier1)

haier1.wash()


haier2 = Washer()

print(haier2)
```

> 注意：打印对象和 self 得到的结果是一致的，都是当前对象的内存中存储地址。

三、添加和获取对象属性
-----------

属性即是特征，比如：洗衣机的宽度、高度、重量...

对象属性既可以在类外面添加和获取，也能在类里面添加和获取。

### 1\. 类外面添加对象属性

*   语法
    

*   体验
    

```
haier1.width = 500
haier1.height = 800
```

### 2\. 类外面获取对象属性

*   语法
    

*   体验
    

```
print(f'haier1洗衣机的宽度是{haier1.width}')
print(f'haier1洗衣机的高度是{haier1.height}')
```

### 3\. 类里面获取对象属性

*   语法
    

*   体验
    

```
class Washer():
    def print_info(self):
        
        print(f'haier1洗衣机的宽度是{self.width}')
        print(f'haier1洗衣机的高度是{self.height}')


haier1 = Washer()


haier1.width = 500
haier1.height = 800

haier1.print_info()
```

四、魔法方法
------

在 Python 中，`__xx__()`的函数叫做魔法方法，指的是具有特殊功能的函数。

### 1\. `__init__()`  

#### 1.1 体验`__init__()`  

思考：洗衣机的宽度高度是与生俱来的属性，可不可以在生产过程中就赋予这些属性呢？

答：理应如此。

\==`__init__()`方法的作用：初始化对象。==

```
class Washer():
    
    
    def __init__(self):
        
        self.width = 500
        self.height = 800


    def print_info(self):
        
        print(f'洗衣机的宽度是{self.width}, 高度是{self.height}')


haier1 = Washer()
haier1.print_info()
```

> 注意：
> 
> *   `__init__()`方法，在创建一个对象时默认被调用，不需要手动调用
>     
> *   `__init__(self)`中的 self 参数，不需要开发者传递，python 解释器会自动把当前的对象引用传递过去。
>     

#### 1.2 带参数的`__init__()`  

思考：一个类可以创建多个对象，如何对不同的对象设置不同的初始化属性呢？

答：传参数。

```
class Washer():
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def print_info(self):
        print(f'洗衣机的宽度是{self.width}')
        print(f'洗衣机的高度是{self.height}')


haier1 = Washer(10, 20)
haier1.print_info()


haier2 = Washer(30, 40)
haier2.print_info()
```

### 2\. `__str__()`  

当使用 print 输出对象的时候，默认打印对象的内存地址。如果类定义了`__str__`方法，那么就会打印从在这个方法中 return 的数据。

```
class Washer():
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def __str__(self):
        return '这是海尔洗衣机的说明书'


haier1 = Washer(10, 20)

print(haier1)
```

### 3\. `__del__()`  

当删除对象时，python 解释器也会默认调用`__del__()`方法。

```
class Washer():
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def __del__(self):
        print(f'{self}对象已经被删除')


haier1 = Washer(10, 20)


del haier1
```

五、综合应用
------

### 1\. 烤地瓜

#### 1.1 需求

需求主线：

1.  被烤的时间和对应的地瓜状态：
    
2.  0-3 分钟：生的
    
3.  3-5 分钟：半生不熟
    
4.  5-8 分钟：熟的
    
5.  超过 8 分钟：烤糊了
    
6.  添加的调料：
    
7.  用户可以按自己的意愿添加调料
    

#### 1.2 步骤分析

需求涉及一个事物： 地瓜，故案例涉及一个类：地瓜类。

##### 1.2.1 定义类

*   地瓜的属性
    
*   被烤的时间
    
*   地瓜的状态
    
*   添加的调料
    
*   地瓜的方法
    
*   被烤
    
*   用户根据意愿设定每次烤地瓜的时间
    
*   判断地瓜被烤的总时间是在哪个区间，修改地瓜状态
    
*   添加调料
    
*   用户根据意愿设定添加的调料
    
*   将用户添加的调料存储
    
*   显示对象信息
    

##### 1.2.2 创建对象，调用相关实例方法

#### 1.3 代码实现

##### 1.3.1 定义类

*   地瓜属性
    
*   定义地瓜初始化属性，后期根据程序推进更新实例属性
    

```
class SweetPotato():
    def __init__(self):
        
        self.cook_time = 0
        
        self.cook_static = '生的'
        
        self.condiments = []
```

##### 1.3.2 定义烤地瓜方法

```
class SweetPotato():
    ......
    
    def cook(self, time):
        """烤地瓜的方法"""
        self.cook_time += time
        if 0 <= self.cook_time < 3:
            self.cook_static = '生的'
        elif 3 <= self.cook_time < 5:
            self.cook_static = '半生不熟'
        elif 5 <= self.cook_time < 8:
            self.cook_static = '熟了'
        elif self.cook_time >= 8:
            self.cook_static = '烤糊了'
```

##### 1.3.3 书写 str 魔法方法，用于输出对象状态

```
class SweetPotato():
        ......

    def __str__(self):
        return f'这个地瓜烤了{self.cook_time}分钟, 状态是{self.cook_static}'
```

##### 1.3.4 创建对象，测试实例属性和实例方法

```
digua1 = SweetPotato()
print(digua1)
digua1.cook(2)
print(digua1)
```

##### 1.3.5 定义添加调料方法，并调用该实例方法

```
class SweetPotato():
        ......

    def add_condiments(self, condiment):
        """添加调料"""
        self.condiments.append(condiment)
    def __str__(self):
        return f'这个地瓜烤了{self.cook_time}分钟, 状态是{self.cook_static}, 添加的调料有{self.condiments}'
      

digua1 = SweetPotato()
print(digua1)

digua1.cook(2)
digua1.add_condiments('酱油')
print(digua1)

digua1.cook(2)
digua1.add_condiments('辣椒面')
print(digua1)

digua1.cook(2)
print(digua1)

digua1.cook(2)
print(digua1)
```

#### 1.4 代码总览

```
class SweetPotato():
    def __init__(self):
        
        self.cook_time = 0
        
        self.cook_static = '生的'
        
        self.condiments = []

    def cook(self, time):
        """烤地瓜的方法"""
        self.cook_time += time
        if 0 <= self.cook_time < 3:
            self.cook_static = '生的'
        elif 3 <= self.cook_time < 5:
            self.cook_static = '半生不熟'
        elif 5 <= self.cook_time < 8:
            self.cook_static = '熟了'
        elif self.cook_time >= 8:
            self.cook_static = '烤糊了'

    def add_condiments(self, condiment):
        """添加调料"""
        self.condiments.append(condiment)

    def __str__(self):
        return f'这个地瓜烤了{self.cook_time}分钟, 状态是{self.cook_static}, 添加的调料有{self.condiments}'


digua1 = SweetPotato()
print(digua1)

digua1.cook(2)
digua1.add_condiments('酱油')
print(digua1)

digua1.cook(2)
digua1.add_condiments('辣椒面')
print(digua1)

digua1.cook(2)
print(digua1)

digua1.cook(2)
print(digua1)
```

### 2\. 搬家具

#### 2.1 需求

将小于房子剩余面积的家具摆放到房子中。

#### 2.2 步骤分析

需求涉及两个事物：房子 和 家具，故被案例涉及两个类：房子类 和 家具类。

##### 2.2.1 定义类

*   房子类
    
*   实例属性
    
*   房子地理位置
    
*   房子占地面积
    
*   房子剩余面积
    
*   房子内家具列表
    
*   实例方法
    
*   容纳家具
    
*   显示房屋信息
    
*   家具类
    
*   家具名称
    
*   家具占地面积
    

##### 2.2.2 创建对象并调用相关方法

#### 2.3 代码实现

##### 2.3.1 定义类

*   家具类
    

```
class Furniture():
    def __init__(self, name, area):
        
        self.name = name
        
        self.area = area
```

*   房子类
    

```
class Home():
    def __init__(self, address, area):
        
        self.address = address
        
        self.area = area
        
        self.free_area = area
        
        self.furniture = []

    def __str__(self):
        return f'房子坐落于{self.address}, 占地面积{self.area}, 剩余面积{self.free_area}, 家具有{self.furniture}'

    def add_furniture(self, item):
        """容纳家具"""
        if self.free_area >= item.area:
            self.furniture.append(item.name)
            
            self.free_area -= item.area
        else:
            print('家具太大，剩余面积不足，无法容纳')
```

##### 2.3.2 创建对象并调用实例属性和方法

```
bed = Furniture('双人床', 6)
jia1 = Home('北京', 1200)
print(jia1)

jia1.add_furniture(bed)
print(jia1)

sofa = Furniture('沙发', 10)
jia1.add_furniture(sofa)
print(jia1)

ball = Furniture('篮球场', 1500)
jia1.add_furniture(ball)
print(jia1)
```

六、三大特征
------

*   封装
    
*   将属性和方法书写到类的里面的操作即为封装
    
*   封装可以为属性和方法添加私有权限
    
*   继承
    
*   子类默认继承父类的所有属性和方法
    
*   子类可以重写父类属性和方法
    
*   多态
    
*   传入不同的对象，产生不同的结果
    

七、封装
----

**什么是封装：** 

1.  封装是面向对象编程的一大特点
    
2.  面向对象编程的第一步，将属性和方法封装到一个抽象的类中(为什么说是抽象的，因为类不能直接使用)
    
3.  外界使用类创建对象，然后让对象调用方法
    
4.  对象方法的细节都被封装在类的内部
    

> 总结：类里面不光有属性还有方法。这种将属性通过方法直接在类内部操作的形式叫做封装。这里的封装是把属性封装在类内部。这样对类形成了一种“黑盒”状态，我们不需要知道类内部是什么样的，只要对对象进行操作就可以。在封装时，我们也可以根据自身的需求，设置一定的访问/设置条件（权限），对数据和代码起到一定的保护作用。

八、继承
----

### 1\. 继承的概念

生活中的继承，一般指的是子女继承父辈的财产。

*   拓展 1：经典类或旧式类
    

不由任意内置类型派生出的类，称之为经典类。

*   拓展 2：新式类
    

Python 面向对象的继承指的是多个类之间的所属关系，即子类默认继承父类的所有属性和方法，具体如下：

```
class A(object):
    def __init__(self):
        self.num = 1

    def info_print(self):
        print(self.num)


class B(A):
    pass


result = B()
result.info_print()
```

> 在 Python 中，所有类默认继承 object 类，object 类是顶级类或基类；其他子类叫做派生类。

### 2\. 单继承

所谓单继承意思就是一个子类只能继承一个父类。

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')

        

class Prentice(Master):
    pass



daqiu = Prentice()

print(daqiu.kongfu)

daqiu.make_cake()
```

### 3\. 多继承

所谓多继承意思就是一个类同时继承了多个父类。

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')



class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    pass


daqiu = Prentice()
print(daqiu.kongfu)
daqiu.make_cake()
```

> 注意：当一个类有多个父类的时候，默认使用第一个父类的同名属性和方法。

### 4\. 子类重写父类同名方法和属性

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')



class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


daqiu = Prentice()
print(daqiu.kongfu)
daqiu.make_cake()


print(Prentice.__mro__)
```

> 子类和父类具有同名属性和方法，默认使用子类的同名属性和方法。

### 5\. 子类调用父类的同名方法和属性

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'

    def make_cake(self):
        
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    
    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)


daqiu = Prentice()

daqiu.make_cake()

daqiu.make_master_cake()

daqiu.make_school_cake()

daqiu.make_cake()
```

### 6\. 多层继承

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)



class Tusun(Prentice):
    pass


xiaoqiu = Tusun()

xiaoqiu.make_cake()

xiaoqiu.make_school_cake()

xiaoqiu.make_master_cake()
```

### 7\. super()调用父类方法

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(Master):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')

        
        
        

        
        super().__init__()
        super().make_cake()


class Prentice(School):
    def __init__(self):
        self.kongfu = '[独创煎饼果子技术]'

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    
    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)

    
    def make_old_cake(self):
        
        
        
        
        

        
        
        
        

        
        super().__init__()
        super().make_cake()


daqiu = Prentice()

daqiu.make_old_cake()
```

> 注意：使用 super() 可以自动查找父类。调用顺序遵循 `__mro__` 类属性的顺序。比较适合单继承使用。

### 8\. 私有权限

#### 8.1 定义私有属性和方法

在 Python 中，可以为实例属性和方法设置私有权限，即设置某个实例属性或实例方法不继承给子类。

设置私有权限的方法：在属性名和方法名 前面 加上两个下划线 `__`。

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'
        
        self.__money = 2000000

    
    def __info_print(self):
        print(self.kongfu)
        print(self.__money)

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)



class Tusun(Prentice):
    pass


daqiu = Prentice()




xiaoqiu = Tusun()
```

> 注意：私有属性和私有方法只能在类里面访问和修改。

#### 8.2 获取和修改私有属性值

在 Python 中，一般定义函数名`get_xx`用来获取私有属性，定义`set_xx`用来修改私有属性值。

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'
        self.__money = 2000000

    
    def get_money(self):
        return self.__money

    
    def set_money(self):
        self.__money = 500

    def __info_print(self):
        print(self.kongfu)
        print(self.__money)

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)



class Tusun(Prentice):
    pass


daqiu = Prentice()

xiaoqiu = Tusun()

print(xiaoqiu.get_money())

xiaoqiu.set_money()
print(xiaoqiu.get_money())
```

### 9\. 总结

*   继承的特点
    
*   子类默认拥有父类的所有属性和方法
    
*   子类重写父类同名方法和属性
    
*   子类调用父类同名方法和属性
    
*   super()方法快速调用父类方法
    
*   私有权限
    
*   不能继承给子类的属性和方法需要添加私有权限
    
*   语法
    

```
class 类名():
  
  __属性名 = 值

  
  def __函数名(self):
    代码
```

九、多态
----

### 1\. 了解多态

多态指的是一类事物有多种形态，（一个抽象类有多个子类，因而多态的概念依赖于继承）。

*   定义：多态是一种使用对象的方式，子类重写父类方法，调用不同子类对象的相同父类方法，可以产生不同的执行结果
    
*   好处：调用灵活，有了多态，更容易编写出通用的代码，做出通用的编程，以适应需求的不断变化！
    
*   实现步骤：
    
*   定义父类，并提供公共方法
    
*   定义子类，并重写父类方法
    
*   传递子类对象给调用者，可以看到不同子类执行效果不同
    

### 2\. 体验多态

```
class Dog(object):
    def work(self):  
        print('指哪打哪...')


class ArmyDog(Dog):  
    def work(self):  
        print('追击敌人...')


class DrugDog(Dog):
    def work(self):
        print('追查毒品...')


class Person(object):
    def work_with_dog(self, dog):  
        dog.work()


ad = ArmyDog()
dd = DrugDog()

daqiu = Person()
daqiu.work_with_dog(ad)
daqiu.work_with_dog(dd)
```

十、类属性和实例属性
----------

### 1\. 类属性

#### 1.1 设置和访问类属性

*   类属性就是 **类对象** 所拥有的属性，它被 **该类的所有实例对象所共有**。
    
*   类属性可以使用 **类对象** 或 **实例对象** 访问。
    

```
class Dog(object):
    tooth = 10


wangcai = Dog()
xiaohei = Dog()

print(Dog.tooth)  
print(wangcai.tooth)  
print(xiaohei.tooth)
```

> 类属性的优点
> 
> *   **记录的某项数据 始终保持一致时**，则定义类属性。
>     
> *   **实例属性** 要求 **每个对象** 为其 **单独开辟一份内存空间** 来记录数据，而 **类属性** 为全类所共有 ，**仅占用一份内存**，**更加节省内存空间**。
>     

#### 1.2 修改类属性

类属性只能通过类对象修改，不能通过实例对象修改，如果通过实例对象修改类属性，表示的是创建了一个实例属性。

```
class Dog(object):
    tooth = 10


wangcai = Dog()
xiaohei = Dog()


Dog.tooth = 12
print(Dog.tooth)  
print(wangcai.tooth)  
print(xiaohei.tooth)  


wangcai.tooth = 20
print(Dog.tooth)  
print(wangcai.tooth)  
print(xiaohei.tooth)
```

### 2\. 实例属性

```
class Dog(object):
    def __init__(self):
        self.age = 5

    def info_print(self):
        print(self.age)


wangcai = Dog()
print(wangcai.age)  

wangcai.info_print()
```

十一、类方法和静态方法
-----------

### 1\. 类方法

#### 1.1 类方法特点

需要用装饰器`@classmethod`来标识其为类方法，对于类方法，**第一个参数必须是类对象**，一般以`cls`作为第一个参数。

#### 1.2 类方法特点

*   当方法中 **需要使用类对象** (如访问私有类属性等)时，定义类方法
    
*   类方法一般和类属性配合使用
    

```
class Dog(object):
    __tooth = 10

    @classmethod
    def get_tooth(cls):
        return cls.__tooth


wangcai = Dog()
result = wangcai.get_tooth()
print(result)
```

### 2\. 静态方法

#### 2.1 静态方法特点

*   需要通过装饰器`@staticmethod`来进行修饰，**静态方法既不需要传递类对象也不需要传递实例对象（形参没有 self/cls）**。
    
*   静态方法 也能够通过 **实例对象** 和 **类对象** 去访问。
    

#### 2.2 静态方法使用场景

*   当方法中 **既不需要使用实例对象**(如实例对象，实例属性)，**也不需要使用类对象** (如类属性、类方法、创建实例等)时，定义静态方法
    
*   **取消不需要的参数传递**，有利于 **减少不必要的内存占用和性能消耗**
    

```
class Dog(object):
    @staticmethod
    def info_print():
        print('这是一个狗类，用于创建狗实例....')


wangcai = Dog()

wangcai.info_print()
Dog.info_print()
```