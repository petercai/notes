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