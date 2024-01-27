# python基础篇: python字符串方法
> Python提供了丰富的字符串处理方法，可以方便地对字符串进行操作、处理和转换。在本文中，我们将介绍Python中常用的字符串方法。

python中字符串内置方法很多，可以通过`dir()`方式查看具体有哪些方法，下表是python字符串的全部的内置方法

| 方法名 | 描述 |
| --- | --- |
| capitalize() | 把字符串第一个字符转换为大写 |
| casefold() | 把字符串转换为小写 |
| center() | 返回一个居中对齐的字符串 |
| count() | 返回字符串中指定值出现的次数 |
| encode() | 返回字符串的编码版本 |
| endswith() | 判断字符串是否以指定的值结尾 |
| expandtabs() | 设置字符串中制表符的空格数 |
| find() | 在字符串中查找指定值并返回其位置 |
| format() | 格式化字符串中的指定值 |
| format_map() | 格式化字符串中的指定值 |
| index() | 在字符串中查找指定值并返回其位置 |
| isalnum() | 如果字符串中的所有字符都是字母或数字，则返回True |
| isalpha() | 如果字符串中的所有字符都是字母，则返回True |
| isascii() | 如果字符串中的所有字符都是ASCII字符，则返回True |
| isdecimal() | 如果字符串中的所有字符都是十进制数字，则返回True |
| isdigit() | 如果字符串中的所有字符都是数字，则返回True |
| isidentifier() | 如果字符串是一个有效的标识符，则返回True |
| islower() | 如果字符串中的所有字符都是小写，则返回True |
| isnumeric() | 如果字符串中的所有字符都是数字，则返回True |
| isprintable() | 如果字符串中的所有字符都可打印，则返回True |
| isspace() | 如果字符串中的所有字符都是空格，则返回True |
| istitle() | 如果字符串遵循标题规则，则返回True |
| isupper() | 如果字符串中的所有字符都是大写，则返回True |
| join() | 把可迭代对象中的元素合并成一个字符串 |
| ljust() | 返回字符串的左对齐版本 |
| lower() | 把字符串转换为小写 |
| lstrip() | 返回字符串的左侧去除指定字符的版本 |
| maketrans() | 返回用于转换字符的翻译表 |
| partition() | 把字符串分为三部分 |
| replace() | 返回把指定值替换为新值的字符串 |
| rfind() | 在字符串中查找指定值并返回最后出现的位置 |
| rindex() | 在字符串中查找指定值并返回最后出现的位置 |
| rjust() | 返回字符串的右对齐版本 |
| rpartition() | 把字符串分为三部分 |
| rsplit() | 在指定的分隔符处拆分字符串并返回列表 |
| rstrip() | 返回字符串的右侧去除指定字符的版本 |
| split() | 在指定的分隔符处拆分字符串并返回列表 |
| splitlines() | 在换行符处拆分字符串并返回列表 |
| startswith() | 判断字符串是否以指定的值开头 |
| strip() | 返回字符串的去除指定字符版本 |
| swapcase() | 把字符串中的大小写字母互换 |
| title() | 把字符串中的每个单词的首字母转换为大写 |
| translate() | 返回一个已翻译的字符串 |
| upper() | 在字符串开头填充指定数量的0值。 |

字符串拼接
-----

字符串拼接是指将两个或多个字符串连接在一起形成一个新的字符串。在Python中，可以使用加号（+）或字符串格式化（%）操作符进行字符串拼接，还可以使用`format`方法进行拼接。

```python

name = "John"
age = 30
message = "My name is " + name + " and I'm " + str(age) + " years old."


name = "John"
age = 30
message = "My name is %s and I'm %d years old." % (name, age)

name = "John"
age = 30
message = "My name is {} and I'm {}years old.".format(name, age)

```

字符串分割
-----

字符串分割是指将一个字符串按照指定的分隔符拆分成多个子字符串。在Python中，可以使用`split()`方法进行字符串分割。

```python

message = "My name is John and I'm 30 years old."
parts = message.split(" ")  
print(parts)  

```

其中还有`rsplit()`，该方法将一个字符串拆分为一个列表，从右边开始。如果未指定**max**，则此方法将返回与split()方法相同的结果。

字符串替换
-----

字符串替换是指将一个字符串中的某些子串替换成其他的子串。在Python中，可以使用replace()方法进行字符串替换。

```python

message = "Hello, world!"
new_message = message.replace("world", "Python")
print(new_message)  

```

字符串大小写转换
--------

字符串大小写转换是指将一个字符串中的所有字符转换成大写或小写形式。在Python中，可以使用upper()方法将字符串转换成大写形式，使用lower()方法将字符串转换成小写形式。

```pthon
# 使用upper()方法将字符串转换成大写形式
message = "Hello, world!"
new_message = message.upper()
print(new_message)  # HELLO, WORLD!

# 使用lower()方法将字符串转换成小写形式
message = "Hello, world!"
new_message = message.lower()
print(new_message)  # hello, world!

```

如果只想把字符串第一个字符变成大写的可以使用`capitalize()`,该方法是返回一个字符串，其中第一个字符为大写，其余为小写。

字符串判断
-----

字符串判断是指判断一个字符串是否符合某种条件。在Python中，可以使用一系列的判断方法来判断字符串是否符合特定的条件。

```python

message = "Hello, world!"
if message.startswith("Hello"):
    print("The message starts with 'Hello'.")


message = "Hello, world!"
if message.endswith("world!"):
    print("The message ends with 'world!'.")


message = "Hello, world!"
if message.isalpha():
    print("The message is all letters.")


```

partition()的用法
--------------

可以用于将一个字符串按照指定的分隔符进行分割，返回一个元组，包含分隔符之前的子字符串、分隔符本身和分隔符之后的子字符串。

具体来说，partition()方法的语法如下：

```python
str.partition(separator)

```

> str是要进行分割的字符串，separator是分隔符。该方法返回一个元组，元组包含三个元素，分别是分隔符之前的子字符串、分隔符本身和分隔符之后的子字符串。如果分隔符在字符串中不存在，则返回一个元组，元组包含原字符串、空字符串和空字符串。

下面是一个使用partition()方法的例子：

```python
s = "Hello, world!"
result = s.partition(",")
print(result) 

```

总结
--

Python字符串是一种非常常见的数据类型，Python为字符串类型提供了很多实用的方法来操作和处理字符串。通过使用内置函数dir()可以查看字符串对象的所有方法，并且通过调用这些方法可以完成各种字符串操作，例如字符串的大小写转换、字符串的拼接和替换、字符串的查找和分割等等。掌握这些字符串方法可以让我们更加高效地处理和操作字符串数据。
