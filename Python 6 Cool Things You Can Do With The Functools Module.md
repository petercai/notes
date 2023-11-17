# Python: 6 Cool Things You Can Do With The Functools Module
In this article let’s look at the [functools Standard Library](https://docs.python.org/3/library/functools.html) module and 6 cool things you can do with it (be warned, a lot of [decorators](https://pybit.es/articles/decorators-by-example/) are coming your way! 😍) …

1\. Cache (“memoize”) things
----------------------------

You can use the `[@cache](https://docs.python.org/3/library/functools.html#functools.cache)` decorator (formerly called `@lru_cache`) as a “simple lightweight unbounded function cache”.

The classic example is calculating a Fibonacci series where the intermediate results are cached, speeding up the calculation significantly:

```
from functools import cache

@cache
def fibonacci(n: int) -> int:
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

for i in range(40):
    print(fibonacci(i))
```

On my system this code takes 0.02s to complete. 😎

However if I comment the @cache decorator it takes 28.30s because of all the repeated calculations! 😱

Hence caching is especially useful and crucial for tasks with expensive repeat computations.

New to _caching_? Check out [our YouTube video](https://www.youtube.com/watch?v=RL2sCl_xbxA).

You can do the same for _[properties](https://www.youtube.com/watch?v=8BbngXWouzo)_ using `[@cached_property](https://docs.python.org/3/library/functools.html#functools.cached_property)`.

2\. Write less dunder methods
-----------------------------

Using the `[@total_ordering](https://docs.python.org/3/library/functools.html#functools.total_ordering)` decorator you can write the `__eq__()` dunder and one of `__lt__()`, `__le__()`, `__gt__()`, or `__ge__()`, so only two, and it will provide the other ones automatically for you. Less code, nice automation.

_As per the docs it does come with the cost of slower execution and more complex stack traces. Also this decorator makes no attempt to override methods already declared in the class or its superclasses. 🤔_

_The term “dunder” is colloquially derived from “double underscore. In the context of Python, dunder methods, also known as “magic methods” or “special methods,” are a set of predefined methods with double underscores at the beginning and end of their names (e.g., `__init__`, `__str__`). Learn more about them in my Dan Bader guest article [here](https://dbader.org/blog/python-dunder-methods) and practice with them [on our platform](https://codechalleng.es/bites/11)._

3\. Freeze functions
--------------------

`[partial()](https://docs.python.org/3/library/functools.html#functools.partial)` lets you put a basic wrapper around an existing function so that you can set a default value where there normally wouldn’t be one.

For example, if I wanted the print() function to always end with a comma instead of a newline, I could use `partial()` as follows:”

```
from functools import partial
print_no_newline = partial(print, end=', ')

for _ in range(3): print('test')
test
test
test

for _ in range(3): print_no_newline('test')
test, test, test,
```

Another example is _freezing_ the `[pow()](https://docs.python.org/3/library/functions.html#pow)` built-in to always square by fixating the `exp` argument to 2:

```
from functools import partial


square = partial(pow, exp=2)

print(square(4)) 
print(square(5)) 
```

By using `partial()`, you can simplify repetitive calls, enhance code clarity, and create reusable components with preset configurations.

There is also [`partialmethod()`](https://docs.python.org/3/library/functools.html#functools.partialmethod) which behaves like `partial()` but is designed to be used as a method definition rather than being directly callable.

4\. Use generic functions
-------------------------

With the introduction of [PEP 443](https://peps.python.org/pep-0443/), Python added support for “single-dispatch generic functions”.

These allow you to define a set of functions (variants) for one main function, where each variant handles a different type of argument.

The `[@singledispatch](https://docs.python.org/3/library/functools.html#functools.singledispatch)` decorator orchestrates this behavior, enabling the function to change its behavior based on the type of its argument.

Let’s take a look at a simple example:

```
from functools import singledispatch

@singledispatch
def process(data):
    """Default behavior for unrecognized types."""
    print(f"Received data: {data}")

@process.register(str)
def _(data):
    """Handle string objects."""
    print(f"Processing a string: {data}")

@process.register(int)
def _(data):
    """Handle integer objects."""
    print(f"Processing an integer: {data}")

@process.register(list)
def _(data):
    """Handle list objects."""
    print(f"Processing a list of length: {len(data)}")

process(42)        
process("hello")   
process([1, 2, 3]) 
process(2.5) 
```

In the example above, when we call the `process` function, the appropriate registered function is invoked based on the type of the argument passed.

For data types that do not have a registered function, the default behavior (defined under the main `@singledispatch` decorated function) is used.

Such a design can make your code more organized and clear, especially when one function needs to handle various data types differently.

Note that the repeated use of `_` as the function name is idiomatic for discarding values, see also our YouTube video: [5 use cases for underscores in Python](https://www.youtube.com/watch?v=ZPdpmiWdDm8).

And to practice, check out [Bite exercise #76](https://codechalleng.es/bites/76).

5\. Help writing better decorators
----------------------------------

When writing a decorator in Python, it’s best practice to use `[functools.wraps()](https://docs.python.org/3/library/functools.html#functools.wraps)` to not lose the docstring and other metadata of the function you are decorating:

```
from functools import wraps

def mydecorator(func):
    @wraps(func)
    def wrapped(*args, **kwargs):
        result = func(*args, **kwargs)
        return result

    return wrapped


@mydecorator
def hello(name: str):
    """Print a salute message"""
    print(f"Hello {name}")

print(hello.__doc__)  
print(hello.__annotations__)  

print(hello.__doc__)  
print(hello.__annotations__) 
```

Preserving the metadata of the decorated function like this, it becomes easier for developers to understand the purpose and usage of the function.

6\. Aggregate data or transform cumulatively
--------------------------------------------

`[functools.reduce(func, iterable)](https://docs.python.org/3/library/functools.html#functools.reduce)` is a function that accumulates results by successively applying a function to the elements of an iterable, from left to right.

_Note `reduce()` was moved into the functools module in Python 3, in Python 2 `reduce()` was a built-in function._

This can be useful in various scenarios where you want to aggregate data or transform it in a cumulative way.

Here is a an example where I use it to aggregate operator module operations on a list of numbers:

```
from functools import reduce
import operator

numbers = list(range(1, 11))  
print(operator.add(1, 2))  

print(reduce(operator.add, numbers))  
print(reduce(operator.sub, numbers))  
print(reduce(operator.mul, numbers))  
print(reduce(operator.truediv, numbers)) 
```

* * *

Conclusion
----------

In conclusion, the `functools` module in Python’s Standard Library is a treasure trove of tools, especially for those who frequently work with functions and decorators.

Whether you’re looking to optimize with caching, streamline your class comparisons, wrap functions in flexible ways, or even handle function dispatch based on argument types, `functools` has you covered.

As we’ve seen, these utilities can simplify your code, boost performance, and overall make your code more Pythonic. 🐍 😍

Next time you find yourself reaching for a function-based solution, remember to peek into `functools` — there might just be a tool waiting to make your life easier.

Keep calm and code in Python! 🐍🚀

More Python tips
----------------

We distilled 250 of these kind of tips in our Pybites Python Tips Book. Timeless, real world and practical tips that will make your code more Pythonic.