# Python快速进修指南：函数基础
话不多说，今天我们要介绍的是函数。本系列文章追求短而精，今天我们将重点讨论函数以及与Java方法的区别。与Java方法不同，函数不需要像Java方法一样讲究修饰符等其他特性，它只需要使用"def"关键字进行声明。另外，函数的参数也与Java方法有所不同，Java方法中不存在默认参数的概念，而在Python中，函数参数是可以有默认值的，并且可以通过传递关键字参数的方式来指定参数顺序。

此外，Python函数还具有可变参数的特性，不同于Java中的实现方式，Python使用星号符号（*）来实现可变参数。请注意这种写法的使用方式。更为强大的是，Python还提供了双星号符号（**）的写法，下面我们将详细讨论这一特性。

最后，我们来谈谈返回值。与Java不同的是，Python函数可以返回多个值，而Java中需要将这些值封装成对象。Python的这种设计让我们能够更加方便地处理返回值。另外，Python还提供了一些内置函数，但如果你想使用Java的内置方法，很抱歉，你需要直接使用对象来调用这些方法。好了，接下来让我们简单了解一下函数的一些其他特性吧。

在Python中，可以使用关键字`def`来声明函数。函数声明的基本语法如下：

```python
def 函数名(参数1, 参数2, ...):
    
    
    return 返回值

```

*   def关键字用于定义函数。
*   函数名是你给函数起的名字，应该具有描述性。
*   参数是可选的，你可以在括号内指定函数需要接收的输入参数。如果没有参数，括号仍然是必须的，但可以留空。
*   函数体是函数的具体实现，包含一系列的语句和逻辑。
*   return语句用于指定函数的返回值。可以选择省略return语句，这样函数将不会返回任何值。

以下是一个简单的示例：

```python
def greet(name):
    print("Hello, " + name + "!")

greet("xiaoyu")  

```

默认参数
----

默认参数（Default arguments）：函数定义时可以为参数指定默认值，这样在函数调用时如果没有传递对应参数的值，将使用默认值。

```python
def power(x, n=2):
    return x ** n

result1 = power(2)  
result2 = power(2, 3)  
print(result1)  
print(result2)  


```

可变参数
----

可变参数（Variable arguments）：。与Java的...使用类似，有时候我们无法确定调用函数时会传递多少个参数，这时可以使用可变参数来接收不定数量的参数。在函数定义时，在参数前面加上一个星号*，这样传递的参数将被打包成一个元组

```python
def add(*numbers):
    result = 0
    for num in numbers:
        result += num
    return result

sum1 = add(1, 2, 3)  
sum2 = add(1, 2, 3, 4, 5)  
print(sum1)  
print(sum2)  

```

关键字参数
-----

关键字参数（Keyword arguments）：当函数调用时，可以使用关键字参数来指定参数的名称和对应的值，这样参数的顺序可以任意。在函数定义时，在参数前面加上两个星号**，这样传递的参数将被打包成一个字典。

```python
def person_info(**info):
    for key, value in info.items():
        print(key + ": " + value)

person_info(name="Alice", age="25", city="New York")  


```

以上我们之讲解了在Java中不常见的，常规用法就不讲解了，浪费时间。

有时候，Python中我们还可以在函数中返回多个值。实际上，Python中的多个返回值是以元组的形式返回的。我们可以通过解包操作将返回的元组拆分为多个变量。而Java中需要将这些值封装成对象

下面是一个示例，演示了函数如何返回多个值：

```python
def calculate(a, b):
    sum = a + b
    difference = a - b
    return sum, difference

result1, result2 = calculate(8, 3)
print(result1)  
print(result2)  

```

除了这一个我还没看到有啥别的大区别，Java同学注意一下！

我举一些不好理解的例子吧，像min、max、sum这种数值操作我就不列举了，我们看下range、zip、all、any吧。这些你遇见了直接百度就可以明白的，无所谓记住什么的，写多了就记住了。

range函数
-------

range(start, stop, step)：range函数用于生成一个整数序列，可以用来遍历数字范围。它接受三个参数：起始值（可选，默认为0），结束值（必选），步长（可选，默认为1）。返回的对象是一个可迭代的序列。

```python
for i in range(1, 10, 2):
    print(i)


```

zip函数
-----

zip(*iterables)：zip函数用于将多个可迭代对象进行配对。它接受任意个可迭代对象作为参数，并返回一个元组的迭代器，其中每个元组由输入迭代器中对应位置的元素组成。当输入的可迭代对象长度不一致时，zip函数会以最短的长度为准，超出部分将被忽略。

```python
names = ["Alice", "Bob", "xiaoyu"]
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(name, age)





```

all函数
-----

all(iterable)：all函数用于检查可迭代对象中的所有元素是否为真。如果可迭代对象中所有元素都为真，返回True；否则返回False。如果可迭代对象为空，则返回True。

```python
numbers = [1, 2, 3, 4, 5]
print(all(numbers))  

numbers = [0, 1, 2, 3, 4, 5]
print(all(numbers))  

numbers = []
print(all(numbers))  


```

any函数
-----

any(iterable)：any函数用于检查可迭代对象中的任何一个元素是否为真。如果可迭代对象中任何一个元素为真，返回True；否则返回False。如果可迭代对象为空，则返回False。

```python
numbers = [0, 0, 0, 1]
print(any(numbers))  

numbers = [0, 0, 0]
print(any(numbers))  

numbers = []
print(any(numbers))  


```

本文介绍了函数的基本概念和与Java方法的区别。在Python中，函数使用"def"关键字进行声明，不需要像Java方法一样讲究修饰符等其他特性。函数的参数可以有默认值，并且可以通过传递关键字参数的方式来指定参数顺序。Python函数还具有可变参数和关键字参数的特性，可以接收不定数量的参数，并且参数的顺序可以任意。与Java不同的是，Python函数可以返回多个值，而Java需要将多个值封装成对象。此外，Python还提供了一些内置函数，如range、zip、all、any等。

# Python快速进修指南：函数进阶 
在上一篇文章中，我们讲解了函数最基础常见的用法，今天我想在这里简单地谈一下函数的其他用法。尽管这些用法可能不是非常常见，但我认为它们仍然值得介绍。因此，我将单独为它们开设一个章节，并探讨匿名函数和装饰器函数这两种特殊的用法。

在Python中，匿名函数也被称为lambda函数，它是一种没有名称的函数。但是与Java的lambda表达式相比，它们有一些区别。匿名函数通常用于在代码中定义简单的功能，并且可以在不需要额外定义函数的情况下使用。主要就是省事~~

匿名函数的语法如下：

```python
lambda arguments: expression

```

其中，arguments是函数的参数，expression是函数的返回值。如果在expression中没有使用print这样的打印函数，通常情况下函数会返回expression的值，这意味着函数中包含了return语句。

```python

print((lambda x, y: x + y)(2, 3))


add = lambda x, y: x + y
print(add(2, 3))

```

在Java中，并没有直接对应的匿名函数的概念，但可以通过定义接口或使用Lambda表达式来实现类似的功能。

在Python中，装饰器是一种特殊的函数，它可以接受一个函数作为参数，并返回一个新的函数。装饰器函数通常用于在不改变原函数代码的情况下，对函数进行扩展或修改。而在Java中，装饰器函数的概念可以通过使用注解来实现。通过在方法前添加特定的注解，可以实现对方法的装饰。

简单来说，装饰器是一种用于修改其他函数行为的函数。它们允许在不修改原始函数定义的情况下，对其进行扩展、修改或包装。

装饰器函数的语法如下：

```python
def log_decorator(original_function):
    def wrapper_function(*args, **kwargs):
        print(f"Calling {original_function.__name__} function")
        result = original_function(*args, **kwargs)
        print(f"{original_function.__name__} function finished")
        return result
    return wrapper_function

@log_decorator
def add(x, y):
    return x + y

print(add(2, 3))





```

这里我们定义了一个装饰器函数log\_decorator，它在调用原始函数之前和之后打印了一些信息。通过在add函数上使用@log\_decorator，我们将add函数传递给log\_decorator函数进行装饰。当调用add函数时，实际上是调用了被装饰后的wrapper\_function函数。

在这篇文章中，我们介绍了函数的两种不常用的特殊用法：匿名函数和装饰器函数。匿名函数是一种没有名称的函数，通常用于定义简单的功能。我们可以使用lambda关键字来创建匿名函数，并在需要时直接调用它们。装饰器函数是一种特殊的函数，可以接受一个函数作为参数，并返回一个新的函数。装饰器函数通常用于在不改变原函数代码的情况下，对函数进行扩展或修改。通过使用装饰器，我们可以在函数调用前后执行额外的操作。这些特殊用法可以帮助我们更灵活地使用函数，并使代码更加简洁和可读。

# Python快速进修指南：异常捕获
在之前的学习中，我们已经讲解了函数和控制流等基本概念。然而，在接触实际业务时，你会发现异常捕获也是必不可少的一部分，因为在Java编程中，异常处理是不可或缺的。Python的异常捕获与Java的异常捕获原理是相同的，只是在写法上有一些区别。它们的目的都是为了处理程序在执行过程中出现错误的机制。通过捕获异常，我们可以在遇到错误时进行适当的处理，而不是直接终止程序的执行。

在接下来的内容中，我将介绍一些常见的异常情况，以及万能异常捕获（在工作中常常使用，即无论什么错误都直接抛出一个通用异常），还有为了处理业务逻辑而自定义的异常类。

需要注意的是，在Python中，else块和finally块是可选的。你可以选择将它们完全写在try语句块里，就像在Java中一样。关于这一点，我就不再详细解释了。

```python
try:
    
except ExceptionType1:
    
except ExceptionType2:
    
else:
    
finally:
    

```

常见异常
----

就举一个异常例子吧，不多说占用精力了，自己有时间现查询百度都行。举例来说，当我们尝试将一个非整数的字符串转换为整数时，会触发ValueError异常。下面是一个处理ValueError异常的示例代码：

```python
try:
    num = int(input("请输入一个整数: "))
    print("你输入的整数是:", num)
except ValueError:
    print("无效的输入，请输入一个整数")

```

其他常见异常：

*   TypeError：类型错误，当一个操作或函数应用于不适当类型的对象时抛出。
*   IndexError：索引错误，当尝试访问一个不存在的索引时抛出。
*   KeyError：键错误，当尝试访问字典中不存在的键时抛出。
*   FileNotFoundError：文件未找到错误，当试图打开一个不存在的文件时抛出。
*   ZeroDivisionError：零除错误，当尝试除以零时抛出。

万能异常捕获
------

我觉得使用万能异常捕获也是一种优化语句的方法。就像在Java中一样，直接捕获Exception异常可以处理所有可能的异常情况，这种做法也很容易记住。不过需要注意的是，虽然这种方式可以简化代码，但有时候会隐藏潜在的问题，因此在实际使用时还是需要谨慎考虑。

```python
try:
    
except Exception as e:
    

```

自定义异常
-----

写自定义异常时，你会发现跟Java一样的思路，这就是为什么从Java转向Python的过程非常简单。你已经具备了各种能力，只需要用另一种语法重新实现一次即可。事实上，所有的编程语言都有相似之处，包括前端的Vue、React等框架也是如此。这意味着你可以在不同的语言中迅速适应和转换，因为它们之间存在共通的原理和概念。所以，只要你理解了一种编程语言，学习和掌握其他语言就会变得更加容易。

```python
class MyException(Exception):
    def __init__(self, message):
        self.message = message

    def __str__(self):
        return self.message

try:
    
    raise MyException("This is a custom exception.")
except MyException as e:
    
    print(e)

```

直接抛出Exception确实是一种简洁的写法。如果时间紧迫或者只是临时测试代码，这样做可以省去定义自定义异常的步骤。不过，需要注意的是，直接抛出Exception会导致代码的可读性和可维护性降低。定义自定义异常可以更好地表达代码的意图，并且提供了更好的错误信息和异常处理方式。所以，在实际项目中，我建议还是尽可能使用自定义异常来提高代码的可读性和可维护性。

在本篇文章中，我们总结了Python中的异常捕获的重要性以及如何进行优化。异常捕获是一种处理程序在执行过程中出现错误的机制，对于程序的稳定性和可靠性至关重要。我们详细学习了Python中的基本异常捕获语法，包括try、except、else和finally块，并举例了常见的异常类型，总之，阅读本文只需5分钟，你就可以轻松掌握Python异常捕获的技巧，为自己的编程之路增添一份宝贵的经验。

# Python快速进修指南：文件操作
Python提供的文件操作相对于Java来说，确实简单方便许多。不仅操作简单，代码可读性也相对较高。然而，我们需要注意的不仅仅是文件操作的简单性，还有文件操作的各种模式。在Java中，我们并不经常使用像Python中那样的操作模式。

另外，我们还需要注意文件指针的移动。无论是Java还是Python，文件都可以看作是IO流，流到哪里就算是哪里。除非重新对文件进行操作，否则想要回到文件开头，只能通过移动指针来实现。因此，在进行文件操作时，我们需要谨慎考虑文件指针的位置。

首先，我们需要使用open()函数来打开文件，并指定文件名和打开模式。常用的打开模式有多种选项，对于我们有经验的来说，r、w、a基本都能猜到他们所代表的英文意思。

*   r：只读模式，从文件中读取数据（默认模式）。
*   w：写入模式，首先清空文件内容，然后写入数据。
*   a：追加模式，将数据写入文件末尾。
*   b：二进制模式，用于处理二进制数据，也就是图片和视频文件了。你可以将"b"理解为"binary"的缩写
*   t：文本模式（默认模式），用于处理文本文件。你可以将"t"理解为"text"的缩写

```python

file = open("example.txt", "rt")


content = file.read()
print(content)


file.close()

```

除了示例中使用的rt模式，还有其他常用的模式，就是r、w和b、t的字母组合了：

*   wt：以文本模式写入文件。如果文件不存在，则创建一个新文件；如果文件已存在，则清空文件内容。
*   rb：以二进制模式读取文件。
*   wb：以二进制模式写入文件。

我们上面的写法是最基础的，为了保证文件操作的正确性和资源的释放，我们需要手动关闭文件。在Java中，可以使用try-with-resource语法来自动关闭流，而在Python中，我们也可以使用with语句来实现类似的功能，自动关闭文件，如下所示：

```python
with open("filename.txt", "r") as file:
    content = file.read()
    print(content)

```

当你在写入文件后，想要回到文件开头以便读取文件内容时，可以使用seek(0)将指针移动到文件的开头位置。以下是一个示例：

```python
with open("file.txt", "a+") as file:
    file.write("This is a new line.")
    file.seek(0)
    content = file.read()
    print(content)

```

使用`seek(0)`将指针移动到文件的开头位置。最后，我们使用`read()`函数读取整个文件的内容，并将其打印出来。指令后面的`+`号可以表示以读写方式打开文件。

使用with open()语句可以更简洁地管理文件的打开和关闭，下面是使用with open()语句进行文件交换、删除源文件和重命名临时文件的示例代码：

```python
import os


source_file = "path/to/source_file.txt"


temp_file = "path/to/temp_file.txt"


with open(source_file, "rt") as file, open(temp_file, "wt") as temp:
    content = file.read()
    temp.write(content)


os.remove(source_file)


os.rename(temp_file, source_file)

```

这次我们第一次使用了import语句，这个语句的作用是导入包。通过导入包，我们可以直接使用写好的逻辑，而不需要自己去编写。Python之所以能够如此简洁，离不开各种强大的包的支持。实际上，文件交换部分的代码也可以利用包来实现，因为已经有其他人写好了相关的功能，就像我们需要实现列表功能时可以直接使用ArrayList一样。市面上已经有很多优秀的轮子可供使用，只需要直接拿来用，千万不要重复造轮子~~

Python提供的文件操作相对于Java来说，更简单方便。不仅操作简单，代码可读性也更高。不过，我们还需要注意文件操作的各种模式和文件指针的移动。虽然文件操作只有几种方式，但我不会给出示例，避免浪费大家的时间和精力。

# Python快速进修指南：迭代器（Iterator）与生成器 
这一篇内容可能相对较少，但是迭代器在Java中是有用处的。因此，我想介绍一下Python中迭代器的使用方法。除了写法简单之外，Python的迭代器还有一个最大的不同之处，就是无法直接判断是否还有下一个元素。我们只能通过捕获异常或使用for循环来退出迭代，这点让我感到十分惊讶。

可迭代对象是指那些可以通过for循环进行遍历的对象。在Python中，可迭代对象通常是容器类型，例如列表、元组、字典和集合，同时也包括字符串和文件对象等。要获取一个迭代器，我们可以使用内置函数iter()。

你可能会问，如何判断一个变量是否是可迭代对象呢？不用担心，不需要死记硬背。只要这个变量具有\_\_iter\_\_()方法，那么它就是可迭代对象。这与Java中的情况相似，Java也是通过实现一个接口来拥有迭代器的能力。

所以，不用担心，你只需要记住这个简单的规则，就可以轻松判断一个变量是否是可迭代对象了。

以下示例参考：

```python
my_list = [1, 2, 3, 4, 5]
for item in my_list:
    print(item)

```

代器是用于遍历可迭代对象的工具。它有两个基本方法，即\_\_iter\_\_()和\_\_next\_\_()。`__iter__()`方法返回迭代器对象本身，而\_\_next\_\_()方法用于返回容器中的下一个元素。然而，当迭代器没有更多的元素可供遍历时，`__next__()`方法会引发StopIteration异常。

下面是一个示例，演示了如何使用迭代器遍历可迭代对象：

```python
my_dict = {"name": "xiaoyu", "age": 18, "country": "China"}
for key in iter(my_dict):
    print(key)

```

正常情况下，我们通常会使用for循环来遍历可迭代对象。但是如果你选择使用while循环，需要注意处理StopIteration异常，这就很尴尬。以下是一个示例代码供参考：

```python
my_dict = {"name": "xiaoyu", "age": 18, "country": "China"}
iterator = iter(my_dict)
while True:
    try:
        key = next(iterator)
        print(key)
    except StopIteration:
        break

```

我认为这个与Java不同，有着独特的特点。生成器是一种特殊的迭代器，它利用函数和yield语句逐步生成数据。生成器可以使用yield语句逐个生成元素，而无需一次性生成所有元素。这种方式既可以节省内存空间，又可以延迟计算。

为了更好地理解，以下是一个示例：

```python
def my_generator():
    yield 1
    
    yield 2
    
    yield 3

gen = my_generator()
print(next(gen))  
print(next(gen))  

```

return语句的优势在于它可以一次性返回一个值，而且立即执行。而相比之下，它的对手——yield关键字更加强大，它不仅可以多次返回值，还能够延迟计算其中的业务逻辑。

通过本文，我们了解了Python中迭代器的使用方法。Python的迭代器与Java有些不同，无法直接判断是否还有下一个元素，需要通过捕获异常或使用for循环来退出迭代。可迭代对象是指可以通过for循环进行遍历的对象，可以使用内置函数iter()获取迭代器。判断一个变量是否是可迭代对象，只需要判断是否具有\_\_iter\_\_()方法。迭代器是用于遍历可迭代对象的工具，具有\_\_iter\_\_()和\_\_next\_\_()方法，当没有更多元素可供遍历时，`__next__()`方法会引发StopIteration异常。生成器是一种特殊的迭代器，利用函数和yield语句逐步生成数据，可以节省内存空间，并且延迟计算。

# Python快速进修指南：面向对象
首先，让我来介绍一下今天的主题。今天我们将讨论封装、反射以及单例模式。除此之外，我们不再深入其他内容。关于封装功能，Python与Java大致相同，但写法略有不同，因为Python没有修饰符。而对于反射来说，我认为它比Java简单得多，不需要频繁地获取方法和属性，而是有专门的方法来实现。最后，我们将简单地实现一下单例模式，这样面向对象章节就告一段落了。

封装是指将数据和方法封装在一个类中。在Python中，我们可以通过属性和方法来实现封装。属性可以通过getter和setter方法来访问和修改，而方法可以在类的内部进行访问和使用。然而，与Java不同的是，虽然方法在Python中是可以调用的，但Java不允许。另外，属性也有一些区别，如果属性以双下划线开头，并且没有声明属性，将无法直接访问。除非你动态赋值，那么将失去封装的作用。

使用双下划线开头的属性是私有属性，下面是一个简单的示例代码：

```python
class Person:
    def __init__(self, name, age):
        self.__name = name    
         
    def get_name(self):
        return self.__name

    def set_name(self, name):
        self.__name = name

person = Person("xiaoyu")
print(person.get_name())    

```

我们都是学习Java的，所以对于getter和setter方法的使用应该是基本常识了。记住在Python中，我们使用双下划线来定义私有属性，但实际上这只是一种约定，Python并没有真正的私有属性概念。我们可以通过一些特殊的方式来访问和修改私有属性，但这违背了封装的原则，不建议直接这样做。

反射是一种强大的编程技术，它使得在运行时可以动态地获取和修改对象的属性和方法。在Python中，我们可以利用内置的getattr()、setattr()和hasattr()等函数来实现反射的功能。通过反射，我们可以在运行时根据需要获取或修改对象的属性和方法，从而实现更灵活和动态的编程。不过，我还是有原则的，毕竟Java作为一种商业生态体系成熟的编程语言，在各个领域都有着强大的应用和支持，这是其他语言所无法比拟的。

下面是一个简单的示例代码：

```python
class MyClass:
    def __init__(self, name):
        self.name = name

    def hello(self):
        print("Hello, {}!".format(self.name))

    def dance(self):
        print("dance, {}!".format(self.name))

    def cmd(self):
        method_name = input("====>")
        if hasattr(obj, method_name):
            method = getattr(obj, method_name)
            method()  


obj = MyClass("xiaoyu")
obj.cmd()

```

这样就可以获取到方法然后去实现反射了，我就不演示setattr了，自行演示吧。

![](_assets/9e6f643349ee452e962cc9198c65d55a~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

单例模式是一种常用的设计模式，它可以确保一个类只有一个实例，并且提供一个全局访问点，方便其他对象对该实例进行调用。在Python中，我们可以通过使用模块级别的变量来实现单例模式，这种方式非常简洁和高效。

下面是一个简单的示例代码，展示了如何在Python中实现单例模式：

```python
class Singleton:
    _instance = None

    @classmethod
    def get_instance(cls):
        if not cls._instance:
            cls._instance = cls()
        return cls._instance
        
s1 = Singleton.get_instance()
s2 = Singleton.get_instance()

print(s1 is s2)  

```

与Java相似，Python中也可以使用classmethod装饰器来实现方法，只是在Python中我们称之为装饰器而非注解。

另外，Python中也有一种类似于Java中常用的stream流处理for循环的高级用法，只不过在Python中这种写法是倒着的。所以人们称之为字典推导或列表推导。为了方便记忆，我一直称之为推倒。

```python
student = {
    "name": "xiaoyu",
    "age": 18
}

[print(key + ": " + str(value)) for key, value in student.items() if key == "name"]


```

在今天的课上，我们深入讨论了封装、反射和单例模式这几个重要的概念。我不想过多地赘述它们的细节，但是请大家务必记住它们的基本语法规则，因为这也是面向对象章节的结束。我希望大家能够牢牢掌握这些知识点，为未来的学习打下坚实的基础。

# Python快速进修指南：面向对象基础
当我深入学习了面向对象编程之后，我首先感受到的是代码编写的自由度大幅提升。不同于Java中严格的结构和约束，Python在面向对象的实现中展现出更加灵活和自由的特性。它使用了一些独特的关键字，如self和cls，这些不仅增强了代码的可读性，还提供了对类和实例的明确引用。正如Java，Python也依赖于对象和类的概念，允许我们通过定义类来创建和操作对象。尽管在表面上Python和Java在面向对象的实现上看似相似，但实际上，它们在细节处理上存在一些显著的差异。接下来，我们将探索这些差异，并深入了解它们在实际应用中的具体表现，以便更好地理解面向对象编程在不同语言中的独特风格和优势。

首先，你需要声明一个类。在Python中，这通常是通过使用class关键字来完成的。下面是一个简单的类声明的示例：

```python
class MyClass:
    myAttr = "类的属性"
    def __init__(self, attribute):
        self.attribute = attribute

    def my_method(self):
        return f"Value of attribute is {self.attribute}"

```

关于上面的类声明你可能发现了attribute和myAttr属性不一样，不报错吗？这就是Python的特点：动态属性赋值。在Python中，不仅可以在类的初始化方法\_\_init\_\_中直接定义新的属性，还可以在对象创建之后的任何时刻动态地添加属性，这种做法在Java中会引发错误，但在Python中却是完全合法的，反映了其动态类型的本质。下面再详细说下。

在Java中，this关键字是隐式的，用于指代当前对象的实例，而在Python中，self必须显式声明并作为方法的第一个参数传递。

返回值里的`f`在这里表示格式化，它使得在字符串中直接嵌入表达式成为可能。Python会自动进行求值并将结果转换为字符串。

创建对象
----

一旦定义了类，就可以使用该类来创建对象。这是通过简单地调用类名并传递必要的参数来完成的。例如：

```python
my_object = MyClass("Hello")
my_object.subAttr = "是子类的"
print(my_object.subAttr) 
print(my_object.my_method()) 

```

虽然在Python中，self关键字需要显式地在方法定义中指出，但其实它的作用与Java中的this关键字相似，代表着方法所属的对象实例。在调用实例方法时，Python会自动将对象实例作为第一个参数传递给self，因此在正常使用实例方法时，我们无需显式地传递这个参数。例如，在调用my\_object.my\_method()时，my\_object实例会自动作为self参数传递给my\_method。这种机制确保了方法能够访问和操作所属对象实例的数据。

如果尝试直接通过类名来调用实例方法，如MyClass.my_method()，将会引发错误。这是因为没有提供必要的实例参数，导致self没有被正确初始化。要想通过类名调用方法，方法必须是类方法或静态方法。来看下

类方法和静态方法
--------

在Python中，@classmethod和@staticmethod是两种常用的方法装饰器，它们分别用于定义类方法和静态方法。

其特点是第一个参数通常是cls，代表着类本身。这与实例方法中的self参数相似，但有一个重要的区别：cls参数指向类，而不是类的某个特定实例。类方法的一个限制是它们无法访问特定实例的属性，因为它们不与任何实例绑定。

```python
class MyClass:
    @classmethod
    def my_class_method(cls):
        
        return "这是一个类方法"

```

静态方法实际上是独立于类的实例和类本身的。静态方法不接收传统意义上的self或cls参数，这意味着它们既不能访问类的实例属性（即对象级别的数据），也不能访问类属性（即与类本身相关联的数据）。静态方法的这种特性使得它们更像是普通函数，但为了逻辑上的整洁和组织性，它们被放置在类的定义中。

```python
class MyClass:
    @staticmethod
    def my_static_method():
        return "这是一个静态方法"

```

作为一名有着Java背景的开发者，你无疑已经习惯了Java那严格的类型系统和细致的访问控制机制。转向Python，你会发现一个截然不同的编程世界。Python的面向对象编程（OOP）方式为代码组织提供了更高的自由度和灵活性，这种变化可能会给你带来新鲜感，同时也是一个挑战。需要注意的是，Python的这种灵活性可能会导致更少的编译时错误检查。由于Python是一种解释型语言，很多错误只有在运行时才会被捕捉到。

# Python快速进修指南：面向对象进阶
在上一期中，我们对Python中的对象声明进行了初步介绍。这一期，我们将深入探讨对象继承、组合以及多态这三个核心概念。不过，这里不打算赘述太多理论，因为我们都知道，Python与Java在这些方面的主要区别主要体现在语法上。例如，Python支持多重继承，这意味着一个类可以同时继承多个父类的属性和方法，而Java则只支持单一继承。另一个值得注意的差异是对super关键字的使用。在Java中，我们经常需要显式地使用super来调用父类的构造器，而在Python中，这一步骤是可选的。如果没有指定，Python会自动调用父类的构造器，这让代码看起来更加简洁。

当然，我们这里不会举出太多复杂的例子来让大家头疼。毕竟我们的目标是理解和应用这些概念，而不是准备考试。我们将通过一些简单直观的例子，帮助大家清晰地把握Python在继承、组合和多态方面的特点和优势。

Python中的继承是一种用于创建新类的机制，新类可以继承一个或多个父类的特性。在面向对象编程中，和Java一样继承提供了代码复用的强大工具。

```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        pass

class Dog(Animal):
    
        

    def speak(self):
        return "Woof!"


dog = Dog("Buddy")
print(dog.speak())  
print(dog.name)  

```

不声明\_\_init\_\_默认使用super调用父类的构造器，如果你一旦声明了\_\_init\_\_那么记得显式的调用super，否则将无效

在Python中，多继承是一个强大的特性，允许一个类同时继承多个父类。下面是一个多继承的例子，来帮助你更好地理解这一概念：

```python
class Father:
    def __init__(self):
        self.father_name = "Father's name"

    def gardening(self):
        return "I enjoy gardening"

class Mother:
    def __init__(self):
        self.mother_name = "Mother's name"

    def cooking(self):
        return "I love cooking"

class Child(Father, Mother):
    def __init__(self):
        Father.__init__(self)
        Mother.__init__(self)
        self.child_name = "Child's name"

    def activities(self):
        return f"{self.father_name} likes {self.gardening()} and {self.mother_name} likes {self.cooking()}."


child = Child()
child.father_name = "father"
child.mother_name = "mom"
print(child.activities())

```

在Python中，我们可以在程序运行过程中根据需要向对象动态地添加新的行为或数据，这种方式为处理各种复杂和不可预见的编程情况提供了极大的便利。相比之下，Java由于其静态类型的特性，对于运行时的动态变化就显得有些束手束脚。在Java中，所有的属性和方法都必须在编译时明确定义。

```python
class Printer:
    def print_document(self, content):
        return f"Printing: {content}"

class Scanner:
    def scan_document(self):
        return "Scanning a document"

class MultifunctionMachine:
    pass


mfm = MultifunctionMachine()
mfm.printer = Printer()
mfm.scanner = Scanner()


print(mfm.printer.print_document("Hello World"))  
print(mfm.scanner.scan_document())                

```

pass 这个看似简单却不可忽视的关键字，我不太确定之前是否有提及。在Python的世界里，pass 是一种特别有趣的存在。想象一下，在编写代码时，我们常常会遇到一种场景：逻辑上需要一个语句，但此刻我们可能还没有准备好具体的实现，或者我们故意想留下一个空的代码块，以备将来填充。这时，pass 就像是一个灵巧的小助手，默默地站在那里，告诉Python解释器：“嘿，我在这里，但你可以暂时忽略我。”

为了让大家更直观地理解多态，我准备了一个例子，相信通过这个实例，多态的概念会变得生动而明晰。在Python中，多态表现得非常直观，而且它与Java在这方面的处理几乎如出一辙。

```python
class Animal:
    def speak(self):
        raise NotImplementedError("Subclass must implement abstract method")

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

def animal_speak(animal):
    print(animal.speak())


my_dog = Dog()
my_cat = Cat()

animal_speak(my_dog)  
animal_speak(my_cat)  

```

在本期文章中，我们深入探讨了Python的对象继承、组合以及多态这三个核心概念。从继承的灵活性，如Python的多重继承和super关键字的使用，到组合中的动态属性添加，我们逐一解析了Python与Java在这些方面的相似之处和差异。通过具体的例子，我们展示了Python中多态的直观表现，强调了它与Java的相似性。这些讨论不仅帮助理解Python的对象模型，而且对比了Java和Python在面向对象编程方面的不同处理方式