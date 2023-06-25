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