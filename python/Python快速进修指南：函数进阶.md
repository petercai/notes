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