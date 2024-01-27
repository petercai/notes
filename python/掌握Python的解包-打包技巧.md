# 掌握Python的解包技巧：`*`和`**`的最全用法

Python中的`*`和`**`是两个强大的符号，它们具有多种用途，包括解包参数、扩展序列、字典和集合操作等。

本文介绍这两个符号的各种用法，并提供详细的示例代码，帮助更好地理解它们的功能。

1\. 解包参数
--------

### 1.1 解包位置参数

在函数定义中，`*`可以用来解包位置参数。这使得函数可以接受不定数量的位置参数，将它们打包成一个元组。

```python
def add(*args):
    result = 0
    for num in args:
        result += num
    return result

print(add(1, 2, 3))  

```

### 1.2 解包关键字参数

`**`用于解包关键字参数，将它们打包成一个字典。

```python
def person_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

person_info(name="Alice", age=30, country="USA")





```

2\. 扩展序列
--------

### 2.1 扩展列表

`*`可以用于扩展列表，将一个列表中的元素拆分后传递给另一个列表。

```python
list1 = [1, 2, 3]
list2 = [4, 5, 6]
list1.extend(list2)
print(list1)  


list1 = [1, 2, 3]
list2 = [4, 5, 6]
list1 = [*list1, *list2]
print(list1)  

```

### 2.2 扩展字典

`**`可以用于扩展字典，将一个字典中的键值对拆分后传递给另一个字典。

```python
dict1 = {"name": "Alice", "age": 30}
dict2 = {"country": "USA"}
dict1.update(dict2)
print(dict1)



dict1 = {"name": "Alice", "age": 30}
dict2 = {"country": "USA"}
dict1 = {**dict1, **dict2}
print(dict1)


```

3\. 函数参数中的`*`和`**`
------------------

### 3.1 函数参数中的`*`

`*`可以用于将位置参数和关键字参数分隔开，从而指定只接受关键字参数。

```python
def greet(name, *, message="Hello"):
    print(f"{message}, {name}!")

greet("Alice")  

```

### 3.2 函数参数中的`**`

`**`可以用于接收任意数量的关键字参数，这些参数将被打包成一个字典。

```python
def person_info(name, age, **kwargs):
    print(f"Name: {name}")
    print(f"Age: {age}")
    print("Other Info:")
    for key, value in kwargs.items():
        print(f"{key}: {value}")

person_info(name="Alice", age=30, country="USA", job="Engineer")







```

4\. 解包操作
--------

### 4.1 解包元组

`*`用于解包元组中的元素。

```python
fruits = ("apple", "banana", "cherry")
a, b, c = fruits
print(a, b, c)  

```

### 4.2 解包字典

`**`用于解包字典中的键值对。

```python
info = {"name": "Alice", "age": 30}
person_info(**info)  

字参数




```

5\. 打包参数
--------

### 5.1 打包位置参数

`*`也可用于打包位置参数，将多个参数打包成一个元组。

```python
def multiply(*args):
    result = 1
    for num in args:
        result *= num
    return result

numbers = (2, 3, 4)
print(multiply(*numbers))  

```

### 5.2 打包关键字参数

`**`用于打包关键字参数，将多个关键字参数打包成一个字典。

```python
def print_colors(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

colors = {"color1": "red", "color2": "blue", "color3": "green"}
print_colors(**colors)  





```

6\. 高级应用
--------

### 6.1 使用`*`和`**`接受不定数量的参数

```python
def advanced_example(*args, **kwargs):
    for arg in args:
        print(arg)
    for key, value in kwargs.items():
        print(f"{key}: {value}")

advanced_example(1, 2, 3, name="Alice", age=30)







```

### 6.2 函数签名和参数传递

`*`和`**`的使用对于构建通用函数和接收不定数量参数的函数非常有用。通过合理使用这些功能，您可以增强函数的灵活性和可重用性。

7\. 总结
------

`*`和`**`是Python中非常有用的符号，它们用于解包和打包参数，扩展序列和字典，以及在函数参数中接受不定数量的参数。这些功能使Python的函数更加灵活，并有助于编写更通用的代码。

* * *

[更多学习内容](https://link.juejin.cn/?target=http%3A%2F%2Fipengtao.com%2F "http://ipengtao.com/")

![](_assets/326fd2f36433408b944365d881ec80e3~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)