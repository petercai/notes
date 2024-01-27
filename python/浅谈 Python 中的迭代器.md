# 浅谈 Python 中的迭代器_Python_何极光_InfoQ写作社区
一、迭代器简介
-------

迭代是访问集合元素的一种方式。迭代器是一个可以记住遍历的位置的对象。迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退。

二、可迭代对象
-------

我们已经知道可以对 list、tuple、str 等类型的数据使用`for...in...`的循环语法从其中依次拿到数据进行使用，我们把这样的过程称为遍历，也叫迭代。

但是，是否所有的数据类型都可以放到`for...in...`的语句中，然后让`for...in...`每次从中取出一条数据供我们使用，即供我们迭代吗？

```
>>> for i in 100:
...     print(i)
...
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'int' object is not iterable
>>>



>>> class MyList(object):
...     def __init__(self):
...             self.container = []
...     def add(self, item):
...             self.container.append(item)
...
>>> mylist = MyList()
>>> mylist.add(1)
>>> mylist.add(2)
>>> mylist.add(3)
>>> for num in mylist:
...     print(num)
...
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'MyList' object is not iterable
>>>
```

我们自定义了一个容器类型 MyList，在将一个存放了多个数据的 MyList 对象放到`for...in...`的语句中，发现`for...in...`并不能从中依次取出一条数据返回给我们，也就说我们随便封装了一个可以存放多条数据的类型却并不能被迭代使用。

我们把可以通过`for...in...`这类语句迭代读取一条数据供我们使用的对象称之为可迭代对象（Iterable）。

三、判断一个对象是否可以迭代
--------------

我们可以使用`isinstance()`判断一个对象是否是 Iterable 对象：

```
In [50]: from collections import Iterable

In [51]: isinstance([], Iterable)
Out[51]: True

In [52]: isinstance({}, Iterable)
Out[52]: True

In [53]: isinstance('abc', Iterable)
Out[53]: True

In [54]: isinstance(mylist, Iterable)
Out[54]: False

In [55]: isinstance(100, Iterable)
Out[55]: False
```

四、可迭代对象的本质
----------

我们分析对可迭代对象进行迭代使用的过程，发现每迭代一次（即在`for...in...`中每循环一次）都会返回对象中的下一条数据，一直向后读取数据直到迭代了所有数据后结束。那么，在这个过程中就应该有一个“人”去记录每次访问到了第几条数据，以便每次迭代都可以返回下一条数据。我们把这个能帮助我们进行数据迭代的“人”称为迭代器（Iterator）。

可迭代对象的本质就是可以向我们提供一个这样的中间“人”即迭代器帮助我们对其进行迭代遍历使用。

可迭代对象通过`__iter__`方法向我们提供一个迭代器，我们在迭代一个可迭代对象的时候，实际上就是先获取该对象提供的一个迭代器，然后通过这个迭代器来依次获取对象中的每一个数据。

那么也就是说，一个具备了`__iter__`方法的对象，就是一个可迭代对象。

```
>>> class MyList(object):
...     def __init__(self):
...             self.container = []
...     def add(self, item):
...             self.container.append(item)
...     def __iter__(self):
...             """返回一个迭代器"""
...             
...             pass
...
>>> mylist = MyList()
>>> from collections import Iterable
>>> isinstance(mylist, Iterable)
True
>>>
```

五、iter()函数与 next()函数
--------------------

list、tuple 等都是可迭代对象，我们可以通过`iter()`函数获取这些可迭代对象的迭代器。然后我们可以对获取到的迭代器不断使用`next()`函数来获取下一条数据。`iter()`函数实际上就是调用了可迭代对象的`__iter__`方法。

```
>>> li = [11, 22, 33, 44, 55]
>>> li_iter = iter(li)
>>> next(li_iter)
11
>>> next(li_iter)
22
>>> next(li_iter)
33
>>> next(li_iter)
44
>>> next(li_iter)
55
>>> next(li_iter)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>>
```

注意：当我们已经迭代完最后一个数据之后，再次调用`next()`函数会抛出 StopIteration 的异常，来告诉我们所有数据都已迭代完成，不用再执行`next()`函数了。

六、判断一个对象是否是迭代器
--------------

我们可以使用`isinstance()`判断一个对象是否是 Iterator 对象：

```
In [56]: from collections import Iterator

In [57]: isinstance([], Iterator)
Out[57]: False

In [58]: isinstance(iter([]), Iterator)
Out[58]: True

In [59]: isinstance(iter("abc"), Iterator)
Out[59]: True
```

七、迭代器 Iterator
--------------

通过上面的分析，我们已经知道：迭代器是用来帮助我们记录每次迭代访问到的位置，当我们对迭代器使用`next()`函数的时候，迭代器会向我们返回它所记录位置的下一个位置的数据。实际上，在使用`next()`函数的时候，调用的就是迭代器对象的`__next__`方法（Python3 中是对象的`__next__`方法，Python2 中是对象的`next()`方法）。所以，我们要想构造一个迭代器，就要实现它的`__next__`方法。但这还不够，Python 要求迭代器本身也是可迭代的，所以我们还要为迭代器实现`__iter__`方法，而`__iter__`方法要返回一个迭代器，迭代器自身正是一个迭代器，所以迭代器的`__iter__`方法返回自身即可。

一个实现了`__iter__`方法和`__next__`方法的对象，就是迭代器。

```
class MyList(object):
    """自定义的一个可迭代对象"""
    def __init__(self):
        self.items = []

    def add(self, val):
        self.items.append(val)

    def __iter__(self):
        myiterator = MyIterator(self)
        return myiterator


class MyIterator(object):
    """自定义的供上面可迭代对象使用的一个迭代器"""
    def __init__(self, mylist):
        self.mylist = mylist
        
        self.current = 0

    def __next__(self):
        if self.current < len(self.mylist.items):
            item = self.mylist.items[self.current]
            self.current += 1
            return item
        else:
            raise StopIteration

    def __iter__(self):
        return self


if __name__ == '__main__':
    mylist = MyList()
    mylist.add(1)
    mylist.add(2)
    mylist.add(3)
    mylist.add(4)
    mylist.add(5)
    for num in mylist:
        print(num)
```

八、for...in...循环的本质
------------------

`for item in Iterable`循环的本质就是先通过`iter()`函数获取可迭代对象 Iterable 的迭代器，然后对获取到的迭代器不断调用`next()`方法来获取下一个值并将其赋值给 item，当遇到 StopIteration 的异常后循环结束。