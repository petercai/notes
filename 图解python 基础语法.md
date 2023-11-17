# 图解python  基础语法
* * *

本系列教程讲展开讲解Python编程语言，在大家学习具体的语法知识之前，我们先了解一些Python的基础知识作为铺垫。

本篇将对 Python 进行简要的介绍，通过阅读本篇内容您将了解到：

*   Python 编程模式
*   Python 标识符与关键字
*   Python 缩进与代码块
*   Python 简单输入输出
*   Python 基本代码结构

（1）交互式编程
--------

交互式编程不需要创建脚本文件，是通过Python解释器的交互模式进来编写代码。 你只需要在命令行中输入 Python 命令即可启动交互式编程,提示窗口如下：

```shell
$ python
Python 3.9.5 (default, May  4 2021, 03:33:11)
[Clang 12.0.0 (clang-1200.0.32.29)] on darwin
Type "help", "copyright", "credits" **or** "license" **for** more information.
>>>

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec28aea80fce4e6ebba1c571480976ec~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

在 python 提示符中输入以下文本信息，然后按 Enter 键查看运行效果：

```python
>>> print("Hello, ShowMeAI, this is Python!")

```

在我当前使用的 Python3.9.5 版本中,以上实例输出结果如下：

```kotlin
Hello, ShowMeAI, this is Python!

```

大家也可以使用[上一节](https://link.juejin.cn/?target=http%3A%2F%2Fshowmeai.tech%2Farticle-detail%2F65 "http://showmeai.tech/article-detail/65")提到的Anaconda环境下的Jupyter Notebook进行交互式Python编程，启动Jupyter Notebook并新建Notebook如下，就可以在cell中进行代码编写和交互了。

（2）脚本式编程
--------

如果我们需要完成的任务较为复杂，我们可以把中间处理过程组织梳理成python脚本，然后通过脚本参数调用解释器开始执行脚本，直到脚本执行完毕。当脚本执行完成后，解释器不再有效。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/538ce418f36e431abd589ad1b098ddeb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

让我们写一个简单的 Python 脚本程序。所有 Python 文件将以 **.py** 为扩展名。将以下的源代码拷贝至 test.py 文件中。

```python
print("Hello, ShowMeAI, this is Python!")

```

使用以下命令运行程序：

```ruby
$ python test.py

```

输出结果：

```kotlin
Hello, ShowMeAI, this is Python!

```

标识符是允许作为变量（函数、类等）名称的有效字符串。其中，有一部分是关键字（语言本身保留的标识符），它是不能做它用的标识符的，否则会引起语法错误（SyntaxError 异常）。Python 还有称为 built-in 标识符集合，虽然它们不是保留字，但是不推荐使用这些特别的名字。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a211b950e9ad43dc8a5686515b3f8cf9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

Python 是动态类型语言， 也就是说不需要预先声明变量的类型。 变量的类型和值在赋值那一刻被初始化。变量赋值通过等号来执行。

Python的有效标识符由大小写字母、下划线和数字组成。数字不能作为第一个字符，标识符的长度不限，Python标识符是大小写敏感的。

在编程语言中，常见的变量命名方式有两种：

*   驼峰体：
    
    *   DateOfBirth
    *   AgeOfBoy
    *   ShowMeAI
*   下划线：
    
    *   date\_of\_birth
    *   age\_of\_boy
    *   show\_me\_ai

下面的列表显示了在Python中的保留字。这些保留字不能用作常数或变数，或任何其他标识符名称。

所有 Python 的关键字只包含小写字母。

| and | exec | not |
| --- | --- | --- |
| assert | finally | or |
| break | for | pass |
| class | from | print |
| continue | global | raise |
| def | if | return |
| del | import | try |
| elif | in | while |
| else | is | with |
| except | lambda | yield |

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3137556d3ac24163b72e09216fb3fef6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

学习 Python 与其他语言最大的区别就是，Python 的代码块不使用大括号 **{}** 来控制类，函数以及其他逻辑判断。python 最具特色的就是用缩进来写模块。

缩进可使用tab或空格等，空白数量是可变的，但是所有代码块语句必须包含相同的缩进空白数量。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/530b14fa718d4db087dc15834dc470e8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

以下实例缩进为四个空格:

```python
if True:
    print("True")
else:
    print("False")

```

以下代码将会执行错误：

```python




if True:
    print("ShowMeAI")
    print("Awesome")
else:
    print("Test")
    
  print("False")

```

执行以上代码，会出现如下错误提醒：

```arduino
  File "<stdin>", line 11
    print("False")
                  ^
IndentationError: unindent does not match any outer indentation level

```

经常遇到的对齐错误有2种：

*   **IndentationError: unindent does not match any outer indentation level**
    
    *   错误表明，你使用的缩进方式不一致，有的是 tab 键缩进，有的是空格缩进，改为一致即可
*   **IndentationError: unexpected indent**
    
    *   文件里格式不对，可能是tab和空格没对齐的问题

因此，在 Python 的代码块中必须使用相同数目的行首缩进空格数。

建议你在实际编程中，每个缩进层次使用 **单个制表符** 或 **两个空格** 或 **四个空格** , 切记不能混用

Python语句中一般以新行作为语句的结束符。

但是我们可以使用斜杠（ \\）将一行的语句分为多行显示，如下所示：

```python
total = item_one + \
        item_two + \
        item_three

```

语句中包含 \[\], {} 或 () 括号就不需要使用多行连接符。如下实例：

```python
days = ['Monday', 'Tuesday', 'Wednesday',
        'Thursday', 'Friday']

```

Python 可以使用引号( **'** )、双引号( **"** )、三引号( **'''** 或 **"""** ) 来表示字符串，引号的开始与结束必须是相同类型的。（更详细的python字符串知识参见[python字符串及操作](https://link.juejin.cn/?target=http%3A%2F%2Fshowmeai.tech%2Farticle-detail%2F76 "http://showmeai.tech/article-detail/76")）

其中三引号可以由多行组成，编写多行文本的快捷语法，常用于文档字符串，在文件的特定地点，被当做注释。

```ini
word = 'word'
sentence = "这是ShowMeAI的教程。"
paragraph = """这是包含多行的语句。
有一行包含了ShowMeAI"""

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eed4f92c4fc742729b3338ba0f46ad4a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

python中单行注释采用 **#** 开头。

```python





print("Hello, ShowMeAI, this is Python!") 

```

输出结果：

```shell
Hello, ShowMeAI, this is Python!

```

注释可以在语句或表达式行末：

```shell
name = "ShowMeAI" # 这是一个注释

```

python 中多行注释使用三个单引号(''')或三个双引号(""")。

```python





'''
这是多行注释，使用单引号。
这是多行注释，使用单引号。
这是多行注释，使用单引号。
'''

"""
这是多行注释，使用双引号。
这是多行注释，使用双引号。
这是多行注释，使用双引号。
"""

```

函数之间或类的方法之间用空行分隔，表示一段新的代码的开始。类和函数入口之间也用一行空行分隔，以突出函数入口的开始。

空行与代码缩进不同，空行并不是Python语法的一部分。书写时不插入空行，Python解释器运行也不会出错。但是空行的作用在于分隔两段不同功能或含义的代码，便于日后代码的维护或重构。

下面的程序执行后就会等待用户输入，按回车键后就会退出：

```python



input("按下 enter 键退出，其他任意键显示...\n")

```

以上代码中 ，**\\n** 实现换行。一旦用户按下 enter(回车) 键退出，其它键显示。

Python可以在同一行中使用多条语句，语句之间使用分号(;)分割，以下是一个简单的实例：

```python


import sys; x = 'ShowMeAI'; sys.stdout.write(x + '\n')

```

执行以上代码，输入结果为：

```shell
$ python test.py
ShowMeAI

```

python3中print默认输出是换行的，如果要实现不换行需要在变量末尾加上 「**, end=''**」。

```python



x="a"
y="b"

print(x)
print(y)
print('---------')


print(x, end='')
print(y, end='')


print(x, y, end='')

```

以上实例执行结果为：

```shell
a
b
---------
a b a b

```

缩进相同的一组语句构成一个代码块，我们称之代码组。

像if、while、def和class这样的复合语句，首行以关键字开始，以冒号( : )结束，该行之后的一行或多行代码构成代码组。

我们将首行及后面的代码组称为一个子句(clause)。

如下实例：

```r
if expression : 
   suite 
elif expression :  
   suite  
else :  
   suite 

```

* * *

本教程系列的代码可以在[ShowMeAI对应的github](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FShowMeAI-Hub%2Fpython-tutorial "https://github.com/ShowMeAI-Hub/python-tutorial")中下载，可本地python环境运行，能科学上网的宝宝也可以直接借助google colab一键运行与交互操作学习哦！

本教程系列涉及的Python速查表可以在以下地址下载获取：

*   [Python速查表](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FShowMeAI-Hub%2Fawesome-AI-cheatsheets%2Ftree%2Fmain%2FPython "https://github.com/ShowMeAI-Hub/awesome-AI-cheatsheets/tree/main/Python")

*   [Python教程—Python3文档](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.python.org%2Fzh-cn%2F3%2Ftutorial%2Findex.html "https://docs.python.org/zh-cn/3/tutorial/index.html")
*   [Python教程-廖雪峰的官方网站](https://link.juejin.cn/?target=https%3A%2F%2Fwww.liaoxuefeng.com%2Fwiki%2F1016959663602400 "https://www.liaoxuefeng.com/wiki/1016959663602400")

*   [python介绍](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F61 "http://www.showmeai.tech/article-detail/61")
*   [python安装与环境配置](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F65 "http://www.showmeai.tech/article-detail/65")
*   [python基础语法](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F66 "http://www.showmeai.tech/article-detail/66")
*   [python基础数据类型](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F67 "http://www.showmeai.tech/article-detail/67")
*   [python运算符](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F68 "http://www.showmeai.tech/article-detail/68")
*   [python条件控制与if语句](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F69 "http://www.showmeai.tech/article-detail/69")
*   [python循环语句](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F70 "http://www.showmeai.tech/article-detail/70")
*   [python while循环](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F71 "http://www.showmeai.tech/article-detail/71")
*   [python for循环](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F72 "http://www.showmeai.tech/article-detail/72")
*   [python break语句](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F73 "http://www.showmeai.tech/article-detail/73")
*   [python continue语句](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F74 "http://www.showmeai.tech/article-detail/74")
*   [python pass语句](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F75 "http://www.showmeai.tech/article-detail/75")
*   [python字符串及操作](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F76 "http://www.showmeai.tech/article-detail/76")
*   [python列表](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F77 "http://www.showmeai.tech/article-detail/77")
*   [python元组](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F78 "http://www.showmeai.tech/article-detail/78")
*   [python字典](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F79 "http://www.showmeai.tech/article-detail/79")
*   [python集合](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F80 "http://www.showmeai.tech/article-detail/80")
*   [python函数](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F81 "http://www.showmeai.tech/article-detail/81")
*   [python迭代器与生成器](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F82 "http://www.showmeai.tech/article-detail/82")
*   [python数据结构](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F83 "http://www.showmeai.tech/article-detail/83")
*   [python模块](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F84 "http://www.showmeai.tech/article-detail/84")
*   [python文件读写](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F85 "http://www.showmeai.tech/article-detail/85")
*   [python文件与目录操作](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F86 "http://www.showmeai.tech/article-detail/86")
*   [python错误与异常处理](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F87 "http://www.showmeai.tech/article-detail/87")
*   [python面向对象编程](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F88 "http://www.showmeai.tech/article-detail/88")
*   [python命名空间与作用域](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F89 "http://www.showmeai.tech/article-detail/89")
*   [python时间和日期](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Farticle-detail%2F90 "http://www.showmeai.tech/article-detail/90")

*   [图解Python编程：从入门到精通系列教程](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Ftutorials%2F56 "http://www.showmeai.tech/tutorials/56")
*   [图解数据分析：从入门到精通系列教程](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Ftutorials%2F33 "http://www.showmeai.tech/tutorials/33")
*   [图解AI数学基础：从入门到精通系列教程](https://link.juejin.cn/?target=http%3A%2F%2Fshowmeai.tech%2Ftutorials%2F83 "http://showmeai.tech/tutorials/83")
*   [图解大数据技术：从入门到精通系列教程](https://link.juejin.cn/?target=http%3A%2F%2Fwww.showmeai.tech%2Ftutorials%2F84 "http://www.showmeai.tech/tutorials/84")
