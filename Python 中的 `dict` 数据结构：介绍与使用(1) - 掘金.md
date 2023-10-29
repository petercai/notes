# Python 中的 `dict` 数据结构：介绍与使用(1) - 掘金
     

简介
--

`dict`（字典）是 Python 中用于存储键值对（key-value pairs）的数据结构。字典在 Python 中是可变的，即你可以在创建后添加、删除或更改其元素。

创建字典
----

创建字典的最简单的方法是使用花括号 `{}`。

```python
my_dict = {'name': 'Alice', 'age': 30, 'email': 'alice@email.com'}

```

也可以使用内置的 `dict()` 函数来创建字典。

```python
another_dict = dict(name='Bob', age=40, email='bob@email.com')

```

常用操作
----

### 添加/更新元素

*   使用键值对直接添加或更新。
    
    ```python
    my_dict['address'] = '123 Main St'
    
    ```
    

### 删除元素

*   使用 `pop()` 方法删除并返回指定键的值。
    
    ```python
    age = my_dict.pop('age')
    
    ```
    
*   使用 `del` 语句删除指定键。
    
    ```python
    del my_dict['email']
    
    ```
    
*   使用 `clear()` 方法删除所有元素。
    
    ```python
    my_dict.clear()
    
    ```
    

### 其他方法

*   `keys()`: 返回字典中的所有键。
    
    ```python
    keys = my_dict.keys()
    
    ```
    
*   `values()`: 返回字典中的所有值。
    
    ```python
    values = my_dict.values()
    
    ```
    
*   `items()`: 返回字典中的所有键值对。
    
    ```python
    items = my_dict.items()
    
    ```
    
*   `get()`: 获取指定键的值，可设置默认值。
    
    ```python
    age = my_dict.get('age', 0)
    
    ```
    
*   `update()`: 更新字典中的键值对。
    
    ```python
    my_dict.update({'age': 35, 'phone': '123-456-7890'})
    
    ```
    

字典推导式
-----

字典推导式提供了一种简洁的方式来创建字典。

```python
squares = {x: x*x for x in range(10)}

```

总结
--

Python 的 `dict` 数据结构提供了非常灵活和高效的方式来存储和操作键值对。掌握字典及其方法是成为 Python 高手的关键之一。

