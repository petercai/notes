# Python: What are lambda expressions?
Have you ever seen the word `lambda` used in Python?

`sorted_by_values = sorted(items, key=lambda i: i[1])` 

That's called a "lambda expression" and it defines a "lambda function".

What are **lambda expressions** and **lambda functions**? And how are **lambda functions** different from other functions in Python?

Lambda expressions define functions
-----------------------------------

A lambda expression looks like this:

`>>> square = lambda n: n**2` 

This `square` variable [points](https://www.pythonmorsels.com/pointers/) to some object now. What do you think its _type_ might be?

`>>> type(square)
<class 'function'>` 

It's a function!

We could [call this function](https://www.pythonmorsels.com/calling-a-function/) just like any other function in Python:

A lambda expression is a way of making a function. And a lambda function is the object we get back from a lambda expression:

`>>> square
<function <lambda> at 0x7f7f221eaca0>` 

But wait... don't we already have a syntax for making functions in Python?

We do and you've probably seen it many times before:

`def square(n):
    return n**2` 

We can use this `def` syntax for [defining a new function](https://www.pythonmorsels.com/making-a-function/).

So why do lambda expressions exist?

Lambda expressions can be defined on the same line they're used
---------------------------------------------------------------

Let's say we'd like to sort the items in a dictionary by their values.

Python's [built-in `sorted` function](https://www.pythonmorsels.com/built-in-functions-in-python/) will sort iterable items in ascending order (using the `<` operator):

`>>> fruits = ['lemon', 'apple', 'lime', 'pear', 'watermelon', 'banana']
>>> sorted(fruits)
['apple', 'banana', 'lemon', 'lime', 'pear', 'watermelon']` 

What if we wanted items by something other than their _value_? For example, what if we wanted to sort strings by their lengths?

The `sorted` function accepts an optional `key` argument for this:

`>>> fruits = ['lemon', 'apple', 'lime', 'pear', 'watermelon', 'banana']
>>> sorted(fruits, key=len)
['lime', 'pear', 'lemon', 'apple', 'banana', 'watermelon']` 

The `key` argument can be any function (technically any [callable](https://www.pythonmorsels.com/class-function-and-callable/) will do). The `sorted` function will call that function on each item in the given iterable, sorting each item by the result of those function calls.

What if you don't already have a function to pass to `sorted`? For example, let's say we wanted to sort dictionary items by their values:

`>>> counts = {"apple": 3, "pear": 2, "banana": 4, "lime": 1}
>>> counts.items()
dict_items([('apple', 3), ('pear', 2), ('banana', 4), ('lime', 1)])` 

We could define a function that accepts each item and returns the value of that item:

`def by_value(item):
    """Return the value from a given (key, value) tuple."""
    key, value = item
    return value` 

Then we can psas that function to `sorted`:

`>>> sorted(counts.items(), key=by_value)
[('lime', 1), ('pear', 2), ('apple', 3), ('banana', 4)]` 

Note that we needed to define the function before we called `sorted` with it:

`>>> def by_value(item):
...     """Return the value from a given (key, value) tuple."""
...     key, value = item
...     return value
...
>>> sorted(counts.items(), key=by_value)
[('lime', 1), ('pear', 2), ('apple', 3), ('banana', 4)]` 

Instead of using `def` to define our function, we could have used a lambda function:

`>>> sorted(counts.items(), key=lambda item: item[1])
[('lime', 1), ('pear', 2), ('apple', 3), ('banana', 4)]` 

Unlike the `def` syntax for defining a function, you can pass a lambda function into another function **on the same line of code that you _define_ the lambda function**. So lambda expressions allows us to **write shorter code** by defining a function on the same line that we pass that function to `sorted`.

The limitations of lambda expressions
-------------------------------------

Lambda functions are [anonymous functions](https://en.wikipedia.org/wiki/Anonymous_function), meaning they have no name.

If we ask for help on a lambda function, it'll say its name is `<lambda>`:

`>>> square = lambda n: n**2
>>> help(square)
Help on function <lambda> in module __main__:

<lambda> lambda n` 

Lambda functions are also limited to a single Python expression. You can't put multiple lines of code within a lambda expression.

Due to their single expression limitation, lambda expressions have no need for `return` statements (and don't allow them). The result of calling the single expression within a lambda expression will be the return value of the lambda function.

So lambda functions can only execute a single Python expression and their return value will be the result of that single expression.

So how do lambda functions compare to traditional functions, in terms of costs and benefits? Well, it depends on how you categorize differences.

If I had to classify the pros and cons of using `lambda` over `def`, I'd say:

*   **Pro**: Can be immediately passed around (no variable needed)
*   **Pro**: Return their sole expression automatically
*   **Con/Pro**: Can only have a single expression within them
*   **Con**: Don't have a name and can't have a docstring
*   **Con**: Have a very different syntax from the usual `def` syntax

When should you avoid lambda expressions?
-----------------------------------------

[PEP 8](https://peps.python.org/pep-0008/#programming-recommendations), the official Python styleguide, says that code like this:

Should always be replaced by code like this:

`def square(n): return n**2` 

Or if you prefer, a multi-line function definition with a nice [docstring](https://www.pythonmorsels.com/docstrings/) like this:

`def square(n):
    """Return the square of the given number."""
    return n**2` 

I would take this advice a step further by arguing that **lambda expressions should _usually_ be avoided**. Giving a function a name often makes its use easier to understand and many lambda functions can be replaced by (fairly well-named) functions that already exist. See [Overusing lambda expressions in Python](https://treyhunner.com/2018/09/stop-writing-lambda-expressions/) for my opinions on avoiding lambda expressions.

Where is lambda often used?
---------------------------

Lambda expressions are often very handy when using functions or classes that involve passing functions around.

For example, you'll often see lambda expressions used with the built-in `sorted` function:

`>>> items = ["2 tomatoes", "1 avocado", "6 bananas", "4 clementines"]
>>> sorted(items, key=lambda item: int(item.split()[0]))
['1 avocado', '2 tomatoes', '4 clementines', '6 bananas']` 

Lambda expressions are also often used with the `map` and `filter` [built-in functions](https://www.pythonmorsels.com/built-in-functions-in-python/):

`>>> numbers = [2, 1, 3, 4, 7, 11, 18, 29]
>>> list(filter(lambda n: n % 2 == 1))
[1, 3, 7, 11, 29]
>>> list(map(lambda n: n**2, numbers))
[4, 1, 9, 16, 49, 121, 324, 841]` 

Though I usually recommend [avoiding Python's `map` and `filter` functions](https://www.pythonmorsels.com/map-and-filter-python/).

Pass functions around, but use lambda conservatively
----------------------------------------------------

Python's functions can be passed around because [functions are objects](https://www.pythonmorsels.com/everything-is-an-object/) in Python. If you want to define a function on the same line of code that you pass that function off to another function, you can use a lambda expression.

`>>> sorted(items, key=lambda item: int(item.split()[0]))
['1 avocado', '2 tomatoes', '4 clementines', '6 bananas']` 

Though personally, I always like to ask myself whether the code I'm writing might be clearer if I gave my function a name instead.

`>>> def by_quantity(item):
...     quantity, name = item.split()
...     return int(quantity)
...
>>> sorted(items, key=by_quantity)
['1 avocado', '2 tomatoes', '4 clementines', '6 bananas']` 

It's up to you when and where you use lambda expressions, but keep in mind that **lambda expressions** are just **a special syntax for making a function that doesn't have a name**. Anything you can do with a lambda function, you can do with a non-lambda function.