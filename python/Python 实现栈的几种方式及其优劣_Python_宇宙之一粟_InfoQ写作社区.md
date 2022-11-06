# Python 实现栈的几种方式及其优劣_Python_宇宙之一粟_InfoQ写作社区
1 栈的概念
------

栈由一系列对象对象组织的一个集合，这些对象的增加和删除操作都遵循一个“后进先出”（Last In First Out，LIFO）的原则。

在任何时刻只能向栈中插入一个对象，但只能取得或者删除只能在栈顶进行。比如由书构成的栈，唯一露出封面的书就是顶部的那本，为了拿到其他的书，只能移除压在上面的书，如图：

![](https://static001.geekbang.org/infoq/81/81ec141a26831104e85eb0b3abb90a81.png)

### 栈的实际应用

实际上很多应用程序都会用到栈，比如：

*   网络浏览器将最近浏览的网址存放在一个栈中。每当用户访问者访问一个新网站时，这个新网站的网址就被压入栈顶。这样，每当我们在浏览器单击“后退”按钮时（或者按键盘快捷键 `CTRL+Z` ，大部分撤销快捷键），就可以弹出当前最近一次访问的网址，以回到其先前访问的浏览状态。
    
*   文本编辑器通常会提供一个“撤销”机制以取消最近的编辑操作并返回到先前状态。这个撤销操作也是通过将文本的变化状态保存在一个栈中得以实现。
    
*   一些高级语言的内存管理，JVM 的栈、Python 栈还用于内存管理、嵌套语言特性的运行时环境等
    
*   回溯（玩游戏，寻找路径，穷举搜索）
    
*   在算法中使用，如汉诺塔、树形遍历、直方图问题，也用于图算法，如拓扑排序
    

语法处理：

*   参数和局部变量的空间是用堆栈在内部创建的。
    
    编译器对大括号匹配的语法检查
    
    对递归的支持
    
    在编译器中像后缀或前缀一样的表达式
    

2 栈的抽象数据类型
----------

任何数据结构都离不开数据的保存和获得方式，如前所述，栈是元素的有序集合，添加和操作与移除都发生在其顶端（栈顶），那么它的抽象数据类型包括：

*   `Stack()` :创建一个空栈，它不需要参数，且会返回一个空栈
    
*   `push(e)`: 将一个元素 e 添加到栈 S 的栈顶，它需要一个参数 e，且无返回值
    
*   `pop()` : 将栈顶端的元素移除，它不需要参数，但会返回顶端的元素，并且修改栈的内容
    
*   `top()`: 返回栈顶端的元素，但是并不移除栈顶元素；若栈为空，这个操作会操作
    
*   `is_empty()`: 如果栈中不包含任何元素，则返回一个布尔值 `True`  
    
*   `size()`：返回栈中元素的数据。它不需要参数，且会返回一个整数。在 Python 中，可以用 `__len__` 这个特殊方法实现。
    

![](https://static001.geekbang.org/infoq/f8/f82fbe46089a410f99dba6b5c5889c84.jpeg?x-oss-process=image%2Fresize%2Cp_80%2Fauto-orient%2C1)

Python 栈的大小可能是固定的，也可能有一个动态的实现，即允许大小变化。在大小固定栈的情况下，试图向已经满的栈添加一个元素会导致栈溢出异常。同样，试图从一个已经是空的栈中移除一个元素，进行 `pop()` 操作这种情况被称为下溢。

3 用 Python 的列表实现栈
-----------------

在学习 Python 的时候，一定学过 Python 列表 `list` , 它能通过一些内置的方式实现栈的功能：

*   通过 `append` 方法用于添加一个元素到列表尾部，这种方式就能模拟 `push()` 操作
    
*   通过 `pop()` 方法用于模拟出栈操作
    
*   通过 `L[-1]` 模拟 `top()`操作
    
*   通过判断 `len(L)==0` 模拟 `isEmpty()` 操作
    
*   通过 `len()` 函数实现 `size()` 函数
    

![](https://static001.geekbang.org/infoq/e7/e7e80e3680c3e58cc0333f8f1348a98a.png)

代码如下：

```
class ArrayStack:
    """ 通过 Python 列表实现 LIFO 栈"""

    def __init__(self):
        self._data = []

    def size(self):
        """ return the number of elements in the stack"""
        return len(self._data)

    def is_empty(self):
        """ return True if the stack is empty"""
        return len(self._data) == 0

    def push(self, e):
        """ add element e to the top of the stack"""
        self._data.append(e)
    
    def pop(self):
        """ remove and return the element from the top of the stack
        """
        if self.is_empty():
            raise Exception('Stack is empty')
        return self._data.pop()

    def top(self):
        """return the top of the stack

        Raise Empty exception if the stack is empty
        """
        if self.is_empty():
            raise Exception('Stack is empty')
        return self._data[-1]  


arrayStack = ArrayStack()
arrayStack.push("Python")
arrayStack.push("Learning")
arrayStack.push("Hello")

print("Stack top element: ", arrayStack.top())
print("Stack length: ", arrayStack.size())
print("Stack popped item: %s" % arrayStack.pop())
print("Stack is empty?", arrayStack.is_empty())

arrayStack.pop()
arrayStack.pop()
print("Stack is empty?", arrayStack.is_empty())
```

运行该程序，结果：

```
Stack top element:  Hello
Stack length:  3
Stack popped item: Hello
Stack is empty? False
Stack is empty? True
```

除了将列表的队尾作为栈顶，也可以通过将列表的头部作为栈的顶端。不过在这种情况下，便无法直接使用 `pop()` 方法和 `append()`方法，但是可以通过 `pop()` 和 `insert()` 方法显式地访问下标为 0 的元素，即列表的第一个元素，代码如下：

```
class ArrayStack:
    """ 通过 Python 列表实现 LIFO 栈"""

    def __init__(self):
        self._data = []

    def size(self):
        """ return the number of elements in the stack"""
        return len(self._data)

    def is_empty(self):
        """ return True if the stack is empty"""
        return len(self._data) == 0

    def push(self, e):
        """ add element e to the top of the stack"""
        self._data.insert(0, e)
    
    def pop(self):
        """ remove and return the element from the top of the stack
        """
        if self.is_empty():
            raise Exception('Stack is empty')
        return self._data.pop(0)

    def top(self):
        """return the top of the stack

        Raise Empty exception if the stack is empty
        """
        if self.is_empty():
            raise Exception('Stack is empty')
        return self._data[0]
```

虽然我们改变了抽象数据类型的实现，却保留了其逻辑特征，这种能力体现了抽象思想。不管，虽然两种方法都实现了栈，但两者的性能方法有差异：

*   `append()` 和 `pop()` 方法的时间复杂度都是 _**O(1)**，_常数级别操作
    
*   第二种实现的性能则受制于栈中的元素个数，这是因为 `insert(0)` 和 `pop(0)` 的时间复杂度都是 _O(n)，元素越多就越慢。_
    

4 用 collections.deque 实现栈
-------------------------

在 Python 中，`collections` 模块有一个双端队列数据结构 [deque](https://xie.infoq.cn/link?target=https%3A%2F%2Fdocs.python.org%2F3%2Flibrary%2Fcollections.html%23collections.deque)，这个数据结构同样实现了 `append()` 和 `pop()` 方法：

```
>>> from collections import deque
>>> myStack = deque()
>>> myStack.append('Apple')
>>> myStack.append('Banana')
>>> myStack.append('Orange')
>>> 
>>> myStack
deque(['Apple', 'Banana', 'Orange'])
>>> myStack.pop()
'Orange'
>>> myStack.pop()
'Banana'
>>>
>>> len(myStack)
1
>>> myStack[0]
'Apple'
>>> myStack.pop()
'Apple'

>>>
>>> myStack.pop()
Traceback (most recent call last):
  File "<pyshell#13>", line 1, in <module>
    myStack.pop()
IndexError: pop from an empty deque
>>>
```

### 为什么有了 list 还需要 deque？

可能你可以看到 deque 和列表 list 对元素的操作差不多，那么为什么 Python 中有列表还增加了 deque 这一个数据结构呢？

那是因为，Python 中的列表建立在连续的内存块中，意味着列表的元素是紧挨着存储的。

![](https://files.realpython.com/media/stack_list_memory_usage.bf57ab8fa608.png)

这对一些操作来说非常有效，比如对列表进行索引。获取 `myList[3]` 的速度很快，因为 Python 确切地知道在内存中寻找它的位置。这种内存布局也允许切片在列表上很好地工作。

毗连的内存布局是 list 可能需要花费更多时间来 `.append()` 一些对象。如果连续的内存块已经满了，那么它将需要获得另一个内存块，先将整体 copy 过去，这个动作可能比一般的 `.append()` 操作花费更多的时间。

![](https://files.realpython.com/media/stack_list_memory_push.8ad17a48f40a.png)

而双端队列 `deque` 是建立在一个双链表的基础上。在一个链接列表结构中，每个条目都存储在它自己的内存块中，并有一个对列表中下一个条目的引用。

双链表也是如此，只是每个条目都有对列表中前一个和后一个条目的引用。这使得你可以很容易地在列表的两端添加节点。

在一个链接列表结构中添加一个新的条目，只需要设置新条目的引用指向当前堆栈的顶部，然后将堆栈的顶部指向新条目。

![](https://static001.geekbang.org/infoq/70/70540efbaabb328eaa3cd8d975cebd92.png)

然而，这种在栈上不断增加和删除条目的时间是有代价的。获取 `myDeque[3]` 的速度要比列表慢，因为 Python 需要走过列表的每个节点来获取第三个元素。

幸运的是，你很少想在栈上做随机索引元素或进行列表切片操作。栈上的大多数操作都是 `push` 或 `pop`。

如果你的代码不使用线程，常数时间的 `.append()` 和 `.pop()` 操作使 deque 成为实现 Python 栈的一个更好的选择。

5 用 queue.LifoQueue 实现栈
-----------------------

Python 栈在多线程程序中也很有用，我们已经学习了 `list` 和 `deque` 两种方式。对于任何可以被多个线程访问的数据结构，在多线程编程中，我们不应该使用 `list`，因为列表不是线程安全的。deque 的 `.append()` 和 `.pop()` 方法是原子性的，意味着它们不会被不同的线程干扰。

因此，虽然使用 deque 可以建立一个线程安全的 Python 堆栈，但这样做会使你自己在将来被人误用，造成竞态条件。

好吧，如果你是多线程编程，你不能用 `list` 来做堆栈，你可能也不想用 `deque` 来做堆栈，那么你如何为一个线程程序建立一个 Python 堆栈？

答案就在 `queue` 模块中：[queue.LifoQueue](https://xie.infoq.cn/link?target=https%3A%2F%2Fdocs.python.org%2F3%2Flibrary%2Fqueue.html)。还记得你是如何学习到栈是按照后进先出（LIFO）的原则运行的吗？嗯，这就是 LifoQueue 的 "Lifo "部分所代表的含义。

虽然 list 和 deque 的接口相似，但 LifoQueue 使用 `.put()` 和 `.get()` 来从栈中添加和删除数据。

```
>>> from queue import LifoQueue
>>> stack = LifoQueue()
>>> stack.put('H')
>>> stack.put('E')
>>> stack.put('L')
>>> stack.put('L')
>>> stack.put('O')
>>> stack
<queue.LifoQueue object at 0x00000123159F7310>
>>> 
>>> stack.get()
'O'
>>> stack.get()
'L'
>>> stack.empty()
False
>>> stack.qsize()
3
>>> stack.get()
'L'
>>> stack.get()
'E'
>>> stack.qsize()
1
>>> stack.get()
'H'
>>> stack.get_nowait()
Traceback (most recent call last):
  File "<pyshell#31>", line 1, in <module>
    stack.get_nowait()
_queue.Empty
>>> 

>>> stack.put('Apple')
>>> stack.get_nowait()
'Apple'
```

与 deque 不同，LifoQueue 被设计为完全线程安全的。它的所有方法都可以在线程环境中安全使用。它还为其操作添加了可选的超时功能，这在线程程序中经常是一个必须的功能。

然而，这种完全的线程安全是有代价的。为了实现这种线程安全，LifoQueue 必须在每个操作上做一些额外的工作，这意味着它将花费更长的时间。

通常情况下，这种轻微的减速对你的整体程序速度并不重要，但如果你已经测量了你的性能，并发现你的堆栈操作是瓶颈，那么小心地切换到 deque 可能是值得做的。

6 选择哪一种实现作为栈
------------

一般来说，如果你不使用多线程，你应该使用 `deque`。如果你使用多线程，那么你应该使用 `LifoQueue`，除非你已经测量了你的性能，发现 `push` 和 `pop` 的速度的小幅提升会带来足够的差异，以保证维护风险。

你可以对列表可能很熟悉，但需要谨慎使用它，因为它有可能存在内存重新分配的问题。`deque` 和 `list` 的接口是相同的，而且 `deque` 没有线程不安全问题。

7 总结
----

本文介绍了栈这一数据结构，并介绍了在现实生活中的程序中如何使用它的情况。在文章的中，介绍了 Python 中实现栈的三种不同方式，知道了 `deque` 对于非多线程程序是一个更好的选择，如果你要在多线程编程环境中使用栈的话，可以使用 `LifoQueue`。

希望本文能对你有所帮助，如果喜欢本文，可以点个关注。

> 这里是宇宙之一粟，下一篇文章见！
> 
> 宇宙古今无有穷期，一生不过须臾，当思奋争。

参考链接：

*   [数据结构](https://xie.infoq.cn/link?target=https%3A%2F%2Fdocs.python.org%2Fzh-cn%2F3%2Ftutorial%2Fdatastructures.html)  
    
*   [How to Implement a Python Stack](https://xie.infoq.cn/link?target=https%3A%2F%2Frealpython.com%2Fhow-to-implement-python-stack%2F)  
    
*   Python 数据结构与算法分析（第 2 版），3.3 栈