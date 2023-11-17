# Python字符串格式化方法：str.format()详解
Python是一门强大的编程语言，拥有丰富的字符串操作方法。其中，字符串的格式化是一个非常重要的功能，用于创建包含变量值的字符串。Python提供了多种格式化字符串的方式，其中`str.format()`方法是一种强大且灵活的选项。

本文学习`str.format()`方法，提供详细的介绍和示例代码。

目录
--

1.  什么是字符串格式化？
2.  基本的`str.format()`用法
3.  位置参数与关键字参数
4.  格式控制
5.  字符串对齐
6.  数字格式化
7.  日期和时间格式化
8.  自定义格式化函数
9.  实际应用示例
10.  总结

1\. 什么是字符串格式化？
--------------

字符串格式化是将变量值插入到字符串中的占位符位置的过程。这使得能够创建动态的文本，其中一些部分可能需要根据不同情况进行替换。

`str.format()`方法是Python中用于进行字符串格式化的功能之一，它使用一种非常直观的方式来定义占位符并填充其值。

2\. 基本的`str.format()`用法
-----------------------

`str.format()`方法的基本用法是在字符串中插入花括号`{}`作为占位符，然后使用`format()`方法提供要填充到占位符的值。

下面是一个简单的示例：

```python
name = "Alice"
age = 30
message = "My name is {} and I am {} years old.".format(name, age)
print(message)

```

在这个示例中，创建了一个字符串`message`，其中包含两个占位符`{}`。然后，使用`format()`方法传递`name`和`age`变量的值，将它们填充到占位符中。

3\. 位置参数与关键字参数
--------------

`str.format()`方法可以使用位置参数或关键字参数来填充占位符。位置参数是按顺序传递的，而关键字参数使用占位符名称来匹配值。

下面是一个示例：

### 3.1 位置参数

```python
message = "My name is {0} and I am {1} years old.".format(name, age)

```

### 3.2 关键字参数

```python
message = "My name is {name} and I am {age} years old.".format(name=name, age=age)

```

使用关键字参数可以让代码更具可读性，因为可以清晰地指定每个占位符的含义。

4\. 格式控制
--------

`str.format()`方法允许您控制输出的格式，包括小数位数、对齐和填充字符等。

下面是一些示例：

### 4.1 小数位数

```python
price = 49.95
formatted_price = "The price is {:.2f} dollars.".format(price)

```

### 4.2 对齐与填充

```python

left_aligned = "{:<10}".format("Hello")

right_aligned = "{:>10}".format("World")

center_aligned = "{:^10}".format("Python")

filled = "{:*^10}".format("Text")

```

5\. 字符串对齐
---------

`str.format()`方法对字符串进行对齐，包括左对齐、右对齐和居中对齐。这对于创建漂亮的表格和报告非常有用。

### 5.1 左对齐

```python
text = "Python"
left_aligned = "{:<10}".format(text)

```

### 5.2 右对齐

```python
text = "Python"
right_aligned = "{:>10}".format(text)

```

### 5.3 居中对齐

```python
text = "Python"
center_aligned = "{:^10}".format(text)

```

6\. 数字格式化
---------

`str.format()`方法可以用于格式化数字，包括指定小数位数、千位分隔符和其他数字格式。

下面是一些示例：

### 6.1 指定小数位数

```python
value = 123.456789
formatted_value = "The value is {:.2f}".format(value)

```

### 6.2 千位分隔符

```python
number = 1234567
formatted_number = "Formatted number: {:,}".format(number)

```

### 6.3 百分比格式

```python
percentage = 0.25
formatted_percentage = "Formatted percentage: {:.2%}".format(percentage)

```

7\. 日期和时间格式化
------------

`str.format()`方法还可以用于格式化日期和时间。Python提供了`datetime`模块，可以处理日期和时间。

```python
from datetime import datetime

now = datetime.now()
formatted_date = "Current date and time: {:%Y-%m-%d %H:%M:%S}".format(now)

```

8\. 自定义格式化函数
------------

除了内置的格式化选项，还可以使用自定义的格式化函数。更灵活地控制输出的格式。

```python
def custom_format(value):
    
    return f"Custom format: {value}"

data = "example"
formatted_data = custom_format(data)

```

9\. 实际应用示例
----------

通过一个实际应用示例来演示`str.format()`方法的强大功能。

假设我们正在构建一个简单的购物清单：

```python
items = [
    {"item": "Apple", "price": 0.5},
    {"item": "Banana", "price": 0.25},
    {"item": "Orange", "price": 0.75}
]

total = sum(item["price"] for item in items)


print("Shopping List:")
for item in items:
    print("{item:<10} ${price:.2f}".format(**item))
print("-" * 20)
print("Total: ${:.2f}".format(total))

```

这段代码将输出一个漂亮的购物清单，包括项目、价格和总计。

10\. 总结
-------

Python的`str.format()`方法是一个强大而灵活的字符串格式化工具，可以轻松地创建动态文本并控制输出格式。该方法支持位置参数和关键字参数，可以格式化文本、数字、日期和时间等多种数据类型。可以使用它来对齐文本、指定小数位数、添加千位分隔符，甚至自定义格式化函数。这使得str.format()在各种应用场景中都非常有用，包括生成报告、构建清单、格式化日志等。

通过掌握`str.format()`方法，可以编写更具可读性和可维护性的代码，同时为输出的内容提供精确的控制。这有助于确保Python程序能够满足各种文本处理需求，不论是简单的格式化还是复杂的数据报告生成。

* * *
