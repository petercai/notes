# Python3 标准库概览 
Python 标准库非常庞大，所提供的组件涉及范围十分广泛，使用标准库我们可以让您轻松地完成各种任务。

以下是一些 Python3 标准库中的模块：

*   os 模块：os 模块提供了许多与操作系统交互的函数，例如创建、移动和删除文件和目录，以及访问环境变量等。
*   sys 模块：sys 模块提供了与 Python 解释器和系统相关的功能，例如解释器的版本和路径，以及与 stdin、stdout 和 stderr 相关的信息。
*   time 模块：time 模块提供了处理时间的函数，例如获取当前时间、格式化日期和时间、计时等。
*   datetime 模块：datetime 模块提供了更高级的日期和时间处理函数，例如处理时区、计算时间差、计算日期差等。
*   random 模块：random 模块提供了生成随机数的函数，例如生成随机整数、浮点数、序列等。
*   math 模块：math 模块提供了数学函数，例如三角函数、对数函数、指数函数、常数等。
*   re 模块：re 模块提供了正则表达式处理函数，可以用于文本搜索、替换、分割等。
*   json 模块：json 模块提供了 JSON 编码和解码函数，可以将 Python 对象转换为 JSON 格式，并从 JSON 格式中解析出 Python 对象。
*   urllib 模块：urllib 模块提供了访问网页和处理 URL 的功能，包括下载文件、发送 POST 请求、处理 cookies 等。

操作系统接口
------

os模块提供了不少与操作系统相关联的函数。

```python
>>> import os
>>> os.getcwd()      
'C:\Python34'
>>> os.chdir('/server/accesslogs')   
>>> os.system('mkdir today')   
0

```

建议使用 "import os" 风格而非 "from os import *"。这样可以保证随操作系统不同而有所变化的 os.open() 不会覆盖内置函数 open()。

在使用 os 这样的大型模块时内置的 dir() 和 help() 函数非常有用:

```python
>>> import os
>>> dir(os)
<returns a list of all module functions>
>>> help(os)
<returns an extensive manual page created from the module's docstrings>


```

针对日常的文件和目录管理任务，:mod:shutil 模块提供了一个易于使用的高级接口:

```python
>>> import shutil
>>> shutil.copyfile('data.db', 'archive.db')
>>> shutil.move('/build/executables', 'installdir')

```

* * *

文件通配符
-----

glob模块提供了一个函数用于从目录通配符搜索中生成文件列表:

```python
>>> import glob
>>> glob.glob('*.py')
['primes.py', 'random.py', 'quote.py']

```

* * *

命令行参数
-----

通用工具脚本经常调用命令行参数。这些命令行参数以链表形式存储于 sys 模块的 argv 变量。例如在命令行中执行 "python demo.py one two three" 后可以得到以下输出结果:

```python
>>> import sys
>>> print(sys.argv)
['demo.py', 'one', 'two', 'three']

```

* * *

错误输出重定向和程序终止
------------

sys 还有 stdin，stdout 和 stderr 属性，即使在 stdout 被重定向时，后者也可以用于显示警告和错误信息。

```lua
>>> sys.stderr.write('Warning, log file not found starting a new one\n')
Warning, log file not found starting a new one

```

大多脚本的定向终止都使用 "sys.exit()"。

* * *

字符串正则匹配
-------

re模块为高级字符串处理提供了正则表达式工具。对于复杂的匹配和处理，正则表达式提供了简洁、优化的解决方案:

```python
>>> import re
>>> re.findall(r'\bf[a-z]*', 'which foot or hand fell fastest')
['foot', 'fell', 'fastest']
>>> re.sub(r'(\b[a-z]+) \1', r'\1', 'cat in the the hat')
'cat in the hat'

```

如果只需要简单的功能，应该首先考虑字符串方法，因为它们非常简单，易于阅读和调试:

```rust
>>> 'tea for too'.replace('too', 'two')
'tea for two'

```

* * *

数学
--

math模块为浮点运算提供了对底层C函数库的访问:

```lua
>>> import math
>>> math.cos(math.pi / 4)
0.70710678118654757
>>> math.log(1024, 2)
10.0

```

random提供了生成随机数的工具。

```python
>>> import random
>>> random.choice(['apple', 'pear', 'banana'])
'apple'
>>> random.sample(range(100), 10)   
[30, 83, 16, 4, 8, 81, 41, 50, 18, 33]
>>> random.random()    
0.17970987693706186
>>> random.randrange(6)    
4

```

* * *

访问 互联网
------

有几个模块用于访问互联网以及处理网络通信协议。其中最简单的两个是用于处理从 urls 接收的数据的 urllib.request 以及用于发送电子邮件的 smtplib:

```python
>>> from urllib.request import urlopen
>>> for line in urlopen('http://tycho.usno.navy.mil/cgi-bin/timer.pl'):
...     line = line.decode('utf-8')  
...     if 'EST' in line or 'EDT' in line:  
...         print(line)

<BR>Nov. 25, 09:43:32 PM EST

>>> import smtplib
>>> server = smtplib.SMTP('localhost')
>>> server.sendmail('soothsayer@example.org', 'jcaesar@example.org',
... """To: jcaesar@example.org
... From: soothsayer@example.org
...
... Beware the Ides of March.
... """)
>>> server.quit()

```

注意第二个例子需要本地有一个在运行的邮件服务器。

* * *

日期和时间
-----

datetime 模块为日期和时间处理同时提供了简单和复杂的方法。

支持日期和时间算法的同时，实现的重点放在更有效的处理和格式化输出。

实例
--

```perl
import datetime  
  

current_datetime = datetime.datetime.now()  
print(current_datetime)  
  

current_date = datetime.date.today()  
print(current_date)  
  

formatted_datetime = current_datetime.strftime("%Y-%m-%d %H:%M:%S")  
print(formatted_datetime)  


```

输出结果为：

```yaml
2023-07-17 18:37:56.036914
2023-07-17
2023-07-17 18:37:56

```

该模块还支持时区处理:

```python
>>> 
>>> from datetime import date
>>> now = date.today()    
>>> now
datetime.date(2023, 7, 17)
>>> now.strftime("%m-%d-%y. %d %b %Y is a %A on the %d day of %B.")
'07-17-23. 17 Jul 2023 is a Monday on the 17 day of July.'

>>> 
>>> birthday = date(1964, 7, 31)
>>> age = now - birthday   
>>> age.days             
21535

```

* * *

数据压缩
----

以下模块直接支持通用的数据打包和压缩格式：zlib，gzip，bz2，zipfile，以及 tarfile。

```python
>>> import zlib
>>> s = b'witch which has which witches wrist watch'
>>> len(s)
41
>>> t = zlib.compress(s)
>>> len(t)
37
>>> zlib.decompress(t)
b'witch which has which witches wrist watch'
>>> zlib.crc32(s)
226805979

```

* * *

性能度量
----

有些用户对了解解决同一问题的不同方法之间的性能差异很感兴趣。Python 提供了一个度量工具，为这些问题提供了直接答案。

例如，使用元组封装和拆封来交换元素看起来要比使用传统的方法要诱人的多,timeit 证明了现代的方法更快一些。

```css
>>> from timeit import Timer
>>> Timer('t=a; a=b; b=t', 'a=1; b=2').timeit()
0.57535828626024577
>>> Timer('a,b = b,a', 'a=1; b=2').timeit()
0.54962537085770791

```

相对于 timeit 的细粒度，:mod:profile 和 pstats 模块提供了针对更大代码块的时间度量工具。

* * *

测试模块
----

开发高质量软件的方法之一是为每一个函数开发测试代码，并且在开发过程中经常进行测试

doctest模块提供了一个工具，扫描模块并根据程序中内嵌的文档字符串执行测试。

测试构造如同简单的将它的输出结果剪切并粘贴到文档字符串中。

通过用户提供的例子，它强化了文档，允许 doctest 模块确认代码的结果是否与文档一致:

```python
def average(values):
    """Computes the arithmetic mean of a list of numbers.

    >>> print(average([20, 30, 70]))
    40.0
    """
    return sum(values) / len(values)

import doctest
doctest.testmod()   

```

unittest模块不像 doctest模块那么容易使用，不过它可以在一个独立的文件里提供一个更全面的测试集:

```python
import unittest

class TestStatisticalFunctions(unittest.TestCase):

    def test_average(self):
        self.assertEqual(average([20, 30, 70]), 40.0)
        self.assertEqual(round(average([1, 5, 7]), 1), 4.3)
        self.assertRaises(ZeroDivisionError, average, [])
        self.assertRaises(TypeError, average, 20, 30, 70)

unittest.main() 

```