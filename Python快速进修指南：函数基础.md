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