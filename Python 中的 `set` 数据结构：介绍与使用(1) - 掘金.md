# Python 中的 `set` 数据结构：介绍与使用(1) - 掘金
     

简介
--

`set` 是 Python 中用于存储无序且不重复元素的数据结构。`set` 是可变的，但其中存储的元素必须是不可变类型。

创建集合
----

创建集合最简单的方法是使用花括号 `{}`。

```python
my_set = {1, 2, 3, 4, 5}

```

也可以使用内置的 `set()` 函数来创建集合。

```python
another_set = set([6, 7, 8, 9, 10])

```

常用操作
----

### 添加元素

*   `add()`: 向集合添加一个元素。
    
    ```python
    my_set.add(6)
    
    ```
    
*   `update()`: 向集合添加多个元素。
    
    ```python
    my_set.update([7, 8, 9])
    
    ```
    

### 删除元素

*   `remove()`: 删除集合中的指定元素。
    
    ```python
    my_set.remove(1)
    
    ```
    
*   `discard()`: 删除集合中的指定元素，如果元素不存在，不会引发错误。
    
    ```python
    my_set.discard(10)
    
    ```
    
*   `pop()`: 删除并返回集合中的一个随机元素。
    
    ```python
    item = my_set.pop()
    
    ```
    
*   `clear()`: 删除集合中的所有元素。
    
    ```python
    my_set.clear()
    
    ```
    

### 集合运算

*   `union()`: 返回两个集合的并集。
    
    ```python
    union_set = my_set.union(another_set)
    
    ```
    
*   `intersection()`: 返回两个集合的交集。
    
    ```python
    intersection_set = my_set.intersection(another_set)
    
    ```
    
*   `difference()`: 返回两个集合的差集。
    
    ```python
    difference_set = my_set.difference(another_set)
    
    ```
    
*   `symmetric_difference()`: 返回两个集合的对称差集。
    
    ```python
    symmetric_difference_set = my_set.symmetric_difference(another_set)
    
    ```
    

集合推导式
-----

集合推导式提供了一种简洁的创建集合的方式。

```python
squares = {x*x for x in range(10)}

```

总结
--

Python 的 `set` 数据结构提供了一种灵活且高效的方式来存储和操作无序且不重复的元素集。它提供了多种内置方法，以方便地进行集合运算和元素操作。
