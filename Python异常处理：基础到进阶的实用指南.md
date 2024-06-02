# Python异常处理：基础到进阶的实用指南 

什么是异常
-----

> 在`Python`中，异常是指在程序执行过程中出现的错误或异常情况。

当`Python`解释器无法执行代码时，它会引发异常，这可能是由于语法错误、运行时错误或逻辑错误等原因引起的。

我们经常会碰到诸如以下的异常：

### SyntaxError异常

`Python`语言拥有自己的语法格式和规则。如果我们未能遵守这些规则，就会导致异常的出现。

```shell
In [52]: pint("Hello, World!")
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
Input In [52], in <cell line: 1>()
----> 1 pint("Hello, World!")

NameError: name 'pint' is not defined

```

正确语法为`print("Hello, World!")`

### NameError异常

`NameError`异常通常在代码中引用了未声明的变量或者不存在的函数或方法时触发。

```python
In [53]: print(my_variable)  
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
Input In [53], in <cell line: 1>()
----> 1 print(my_variable)

NameError: name 'my_variable' is not defined

```

### KeyError异常

`KeyError`异常通常在尝试访问字典中不存在的键时触发。

```python
In [54]: my_dict = {'a': 1, 'b': 2}
    ...: print(my_dict['c'])  
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
Input In [54], in <cell line: 2>()
      1 my_dict = {'a': 1, 'b': 2}
----> 2 print(my_dict['c'])

KeyError: 'c'

```

### FileNotFoundError（文件未找到错误）

`FileNotFoundError`异常通常在尝试打开或操作一个并不存在的文件时触发。

```shell
In [55]: file = open('non_existent_file.txt', 'r')
---------------------------------------------------------------------------
FileNotFoundError                         Traceback (most recent call last)
Input In [55], in <cell line: 1>()
----> 1 file = open('non_existent_file.txt', 'r')

FileNotFoundError: [Errno 2] No such file or directory: 'non_existent_file.txt'

```

异常可以分为两种类型：

1.  **语法错误（Syntax Errors）**：这类错误也称为解析错误，是由于代码不符合Python语法规则而引起的。比如，拼写错误、缺少括号、不正确的缩进等。
    
2.  **运行时错误（Runtime Errors）**：这类错误在程序运行时发生，通常是由于程序的逻辑错误或外部环境引起的。常见的运行时错误包括`除以零（ZeroDivisionError）`、`访问不存在的索引（IndexError）`、`使用未定义变量（NameError）`等。
    

> 异常处理机制允许我们捕获并处理这些异常，以避免程序崩溃并提供错误处理和恢复的机会。

在`Python`中，我们可以使用`try-except`语句来捕获并处理异常。`try`语句块用于执行可能会引发异常的代码，而`except`语句块用于处理异常情况。例如：

```python
try:
    
    result = 10 / 0
except ZeroDivisionError:
    
    print("除数不能为零！")

```

除了使用`try-except`语句来处理异常，还可以使用其他相关的结构和关键字，如`try-except-else`、`try-except-finally`等，来更灵活地处理异常情况。

异常的分类2
------

在`Python`中，异常可以进一步分为`内置异常（Built-in Exceptions）`和`自定义异常（Custom Exceptions）`。

1.  **内置异常（Built-in Exceptions）**：`Python`提供了一系列内置的异常类来表示不同类型的错误或异常情况。这些异常类都是内置在`Python`解释器中的，开发者可以直接使用它们来处理程序执行过程中可能出现的各种错误。常见的内置异常包括：

```python
-- SystemExit  
-- KeyboardInterrupt  
-- GeneratorExit  
     -- StopIteration  
     -- StopAsyncIteration  
     -- ArithmeticError  
     |    -- FloatingPointError  
     |    -- OverflowError  
     |    -- ZeroDivisionError  
     -- AssertionError  
     -- AttributeError  
     -- BufferError  
     -- EOFError  
     -- ImportError  
     |    -- ModuleNotFoundError  
     -- LookupError  
     |    -- IndexError  
     |    -- KeyError  
     -- MemoryError  
     -- NameError  
     |    -- UnboundLocalError  
     -- OSError  
     |    -- BlockingIOError  
     |    -- ChildProcessError  
     |    -- ConnectionError  
     |    |    -- BrokenPipeError  
     |    |    -- ConnectionAbortedError  
     |    |    -- ConnectionRefusedError  
     |    |    -- ConnectionResetError    
     |    -- FileExistsError  
     |    -- FileNotFoundError  
     |    -- InterruptedError  
     |    -- IsADirectoryError  
     |    -- NotADirectoryError  
     |    -- PermissionError  
     |    -- ProcessLookupError  
     |    -- TimeoutError  
     -- ReferenceError  
     -- RuntimeError  
     |    -- NotImplementedError  
     |    -- RecursionError  
     -- SyntaxError  
     |    -- IndentationError  
     |         -- TabError  
     -- SystemError  
     -- TypeError  
     -- ValueError  
     |    -- UnicodeError  
     |         -- UnicodeDecodeError  
     |         -- UnicodeEncodeError  
     |         -- UnicodeTranslateError  
     -- Warning  
          -- DeprecationWarning  
          -- PendingDeprecationWarning  
          -- RuntimeWarning  
          -- SyntaxWarning  
          -- UserWarning  
          -- FutureWarning  
          -- ImportWarning  
          -- UnicodeWarning  
          -- BytesWarning  
          -- ResourceWarning  
异常捕获与处理

```

> 完整的内置异常列表可以在Python官方文档中找到。 [docs.python.org/3/library/e…](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.python.org%2F3%2Flibrary%2Fexceptions.html%23bltin-exceptions "https://docs.python.org/3/library/exceptions.html#bltin-exceptions")

2.  **自定义异常（Custom Exceptions）**：除了使用`Python`提供的内置异常类外，开发者还可以根据自己的需求定义自己的异常类。通过定义自定义异常类，开发者可以更好地组织和管理程序中可能出现的特定类型的错误。自定义异常类通常继承自`Python`内置的`Exception`类或其子类。例如：

```python
class CustomException(Exception):
    def __init__(self, message):
        self.message = message


def example_function(x):
    if x < 0:
        raise CustomException("输入不能为负数！")


try:
    example_function(-5)
except CustomException as e:
    print("捕获到自定义异常：", e.message)


```

这样就定义了一个名为`CustomException`的自定义异常类。

我们可以在程序中使用这个自定义异常来表示特定类型的错误，并根据需要在`try-except`语句中捕获和处理这些异常。

自定义异常的主要优势在于它能够使程序结构更清晰，能够更准确地表示程序中可能发生的异常情况，并能够根据具体情况提供更详细的错误信息。

异常处理的层次结构
---------

与`python`异常相关的关键字主要有：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ed4452d45f94229a421cc0583ae315f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1048&h=425&s=40469&e=png&b=faf4e0)

异常处理的层次结构如下：

```python
try:
    
    result = 1 / 0
except ZeroDivisionError as e:
    
    print(f"Error: {e}")
except Exception as e:
    
    print(f"Unexpected error: {e}")
else:
    
    print("No exceptions occurred.")
finally:
    
    print("Finally block.")

```

*   一般情况下，如果没有异常抛出以及错误，代码会执行的顺序是：`try中代码块 =》else中代码块 =》finally中的代码块`。
*   如果try中抛出了异常或者错误，此时执行的顺序是：`try中的代码块 =》except捕获的异常代码块 =》finally中的代码块`。

> `finally`块中的代码无论是否引发异常`都会执行`。`通常用于确保资源的释放或清理操作。`

### 如何捕获多个异常？

> 在`Python`中，我们可以使用多个`except`分别捕获不同类型的异常。

```python
try:
    
    file = open('filename.txt', 'r')
    result = 10 / 0  
except FileNotFoundError as e:
    print(f"文件未找到错误: {e}")
except ZeroDivisionError as e:
    print(f"除以零错误: {e}")

```

在这个例子中，我们使用了两个`except`块，分别来捕获`FileNotFoundError`和`ZeroDivisionError`这两种不同类型的异常。当`try`块中的代码抛出任一种异常时，对应类型的`except`块就会捕获并处理该异常。

> 另一种方法是通过`元组`的方式一次性捕获多个异常类型。

```python
try:
    
    file = open('filename.txt', 'r')
    result = 10 / 0  
except (FileNotFoundError, ZeroDivisionError) as e:
    print(f"发生错误: {e}")

```

在这个例子中，我们将多个异常类型放在一个元组中作为一个捕获异常的参数，这样就可以在一个`except`块中捕获多种类型的异常。

无论是逐个指定`except`块，还是使用一个元组指定多个异常类型，都可以帮助我们清晰地捕获并处理不同类型的异常。

异常的抛出
-----

有些时候，我们可能需要主动抛出异常，比如：

*   参数不符合我们设定的校验规则。
*   捕获了异常，但是需要`wrap`一层更详细的报错或提示信息后继续抛出以方便上层处理。

在`Python`中，要手动抛出异常，可以使用`raise语句`。

```python
def divide(x, y):
    if y == 0:
        raise ZeroDivisionError("除数不能为 0")
    return x / y

try:
    result = divide(10, 0)
    print(result)
except ZeroDivisionError as e:
    print(f"发生除以零错误：{e}")

```

在这个例子中，`divide`函数用于计算两个数相除，如果除数为`0`，则抛出一个`ZeroDivisionError`异常。在`try`块中调用了`divide`函数，并在`except`块中捕获了`ZeroDivisionError`异常。当除数为`0`时，`divide`函数会抛出异常，并且在`except`块中的代码会被执行。

通过`raise`语句，我们可以手动引发特定类型的异常，将其传播到调用栈中，并在需要的地方进行捕获和处理。

自定义异常
-----

> 自定义异常通常被用于特定的情况或者错误类型，以便能够更清晰地识别和处理特定类型的问题。

以下是一个简单的自定义异常的示例：

```python
class CustomException(Exception):
    def __init__(self, message="这是一个自定义异常"):
        self.message = message
        super().__init__(self.message)

```

在这个例子中，我们创建了一个名为`CustomException`的自定义异常类，它继承自`Python`的内置异常类`Exception`。我们定义了它的`__init__`方法来初始化异常的消息，并调用了父类的`__init__`方法以设置异常消息。

接下来，我们可以使用这个自定义异常类来抛出异常，并在需要捕获它的地方进行处理：

```python
def some_function(x):
    if x < 0:
        raise CustomException("输入不能为负数")

try:
    some_function(-1)
except CustomException as e:
    print(f"捕获到了自定义异常：{e}")

```

在这个示例中，`some_function`函数根据输入的值是否小于`0`来抛出我们定义的`CustomException`自定义异常。在`try...except`块中，我们捕获并处理了这个自定义异常。

> 通过创建自定义异常类，您可以更好地组织和标识程序中特定类型的错误，从而实现更清晰、更易于维护的异常处理机制。

异常处理(函数)：碰到return我该如何
---------------------

> *   如果在`函数`内部做异常处理，此时需要考虑`return`。
> *   带有`finally`子句的`try`语句内使用`return`语句时，`finally`子句始终在`return`声明。这确保了`finally`子句中的代码始终运行。

### a. 无return时

```python
from loguru import logger


def example_function(raise_error=True):
    try:
        logger.debug("在try语句块中")
        if raise_error:
            raise ValueError("主动抛出一个异常...")
        
    except:
        logger.error("在except语句块中")
        
    else:
        logger.debug("在else语句块中")
    finally:
        logger.debug("在finally语句块中")
        


example_function()
example_function(raise_error=False)

```

这个比较清晰，也比较容易理解。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1eacdd3236ae434985a6e23c3bdc4423~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1737&h=152&s=49919&e=png&b=282c34)

我们看到：

*   无异常时，会执行`try`里面的逻辑，然后是`else`里面的逻辑，最后是`finally`中的逻辑。
*   异常时，会执行`try`里面异常前的逻辑，然后是`finally`中的逻辑，不会执行`else`中的代码。

### b. 仅finally里面无return，无异常或有异常时

```python
from loguru import logger


def example_function(raise_error=True):
    try:
        logger.debug("在try语句块中")
        if raise_error:
            raise IndexError("索引错误")
        return 1
    except:
        logger.debug("在except语句块中")
        return 2
    else:
        logger.debug("在else语句块中")
        return 2.5

    finally:
        logger.debug("在finally语句块中")
        


result = example_function()
logger.debug(f"result: {result}")

result2 = example_function(raise_error=False)  
logger.debug(f"result2: {result2}")

```

这个我们可能第一感觉是会直接报错，其实不然。不过`pycharm`中确实飘红了~

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc0ed7afcfa843ce8bdb445fe9f97702~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1524&h=1354&s=190251&e=png&b=3f4757)

然后我们执行，我们看到这种情况下`else`中的逻辑如意想的一样，并不会执行：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5eff1175165435d9a3ca6f97142ac16~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1470&h=274&s=75366&e=png&b=282c34)

> 这个else真的好“鸡肋”啊。

### c. 都有return，无异常或有异常时

```python
from loguru import logger


def example_function(raise_error=True):
    try:
        logger.debug("在try语句块中")
        if raise_error:
            raise IndexError("索引错误")
        return 1
    except:
        logger.debug("在except语句块中")
        return 2
    else:
        logger.debug("在else语句块中")
        return 2.5

    finally:
        logger.debug("在finally语句块中")
        return 3


result = example_function()
logger.debug(f"result: {result}")

result2 = example_function(raise_error=False)  
logger.debug(f"result2: {result2}")

```

可以看到，无论是否异常，也都不会执行`else`中的逻辑~

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e32b1abaa514499981847af681bf8b8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1510&h=289&s=78025&e=png&b=282c34)

*   无异常时，因为`try`中`return`了，所以不会执行`else`子句；
*   有异常时，因为`try`中`raise`了异常，所以不会执行`else`子

有异常或无异常时，分别走完`try`或`except`中的逻辑，最终都会执行`finally`中的逻辑，并且因为`finally`中存在`return`，返回值为`finally`的`return`结果。

### d. 只有else中有return时

```python
from loguru import logger


def example_function(raise_error=True):
    try:
        logger.debug("在try语句块中")
        if raise_error:
            raise IndexError("索引错误")
        
    except:
        logger.debug("在except语句块中")
        
    else:
        logger.debug("在else语句块中")
        return 2.5

    finally:
        logger.debug("在finally语句块中")
        


result = example_function()
logger.debug(f"result: {result}")

result2 = example_function(raise_error=False) 
logger.debug(f"result2: {result2}")

```

*   异常的情况下，执行了`try`未异常代码 =\> `except` =\> `finally`的逻辑。
*   无异常的情况下，执行了`try` =\> `else` =\> `finally`的逻辑，并执行了`else`中的`return`。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77fc9078c02640479e60b90e6f0cc5c9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1481&h=299&s=87815&e=png&b=282c34)

### e. 只有else和finally中有return时

```python




from loguru import logger


def example_function(raise_error=True):
    try:
        logger.debug("在try语句块中")
        if raise_error:
            raise IndexError("索引错误")
        
    except:
        logger.debug("在except语句块中")
        
    else:
        logger.debug("在else语句块中")
        return 2.5

    finally:
        logger.debug("在finally语句块中")
        return 3


result = example_function()
logger.debug(f"result: {result}")

result2 = example_function(raise_error=False)  
logger.debug(f"result2: {result2}")


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fbd11121fcd47bcab7525bd028d0342~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1430&h=293&s=89003&e=png&b=282c34)

> 警告~警告~：请尽量避免在`finally`中`return`!

异常处理的最佳实践
---------

### 只做精准捕获异常

> *   始终只捕获那些潜在抛出异常的特定代码块。
> *   在捕获异常时，力求精确匹配具体的异常类型，避免笼统地使用通用的`Exception`。

这样可以防止隐藏其他问题，并使代码更具可读性。

```python
file = None
try:
    file = open('hello.py', 'rb')
    content = file.read()
    
except FileNotFoundError:
    print("文件未找到异常！")
except PermissionError:
    print("权限异常！")
except Exception as e:
    print("发生了一个异常:", e)
finally:
    if file is not None:
        file.close()

```

### 异常信息的记录

在处理异常时，最好记录异常的信息以便后续排查问题。可以使用标准库中的`logging`、`traceback`模块或第三方库如：`loguru`。

```python
import logging
import traceback
from loguru import logger

try:
    
    1 / 0
except Exception as e:
    traceback.print_exc()
    logging.exception("发生了异常：%s", e)
    logger.error(f"发生了异常：{e}")
    logger.exception(f"发生了异常：{e}")

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dd4aca358ab42c0aa12105b4f795c70~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1120&h=403&s=59845&e=png&b=282c34)

### 慎用或不用else，避免在finally里return

在一些情况下，`else`可以被完全避免，通过在`try`块中放置需要运行成功的代码，以避免引入不必要的逻辑复杂度。

在`finally`中使用`return`可能使代码变得难以维护。由于`finally`中的代码总是会执行，因此使用`return`可能会导致不符合预期的行为，降低代码的可维护性。

### 异常链的传递(raise...from)

> 异常链传递指的是当一个异常捕获并处理后，可以选择重新引发（re-raise）同一个异常或者另一个异常，并且保留原始异常的信息。

这种做法可以保留异常的上下文信息，使得调试和排查问题更加方便。

在`Python`中，可以通过在异常处理程序中使用`raise ... from`语句来实现异常链的传递。例如，可以在捕获到某个异常后，重新引发同一个异常或者包装成新的异常并引发。

以下是一个示例，演示了异常链的传递：

```python
try:
    
    x = 10 / 0
except ZeroDivisionError as original_exception:
    
    
    raise ValueError("除数不能为零") from original_exception

```

在上面的代码中，原始的`ZeroDivisionError`异常被捕获，并且使用`raise ... from`语句重新引发成`ValueError`异常，同时保留了原始异常的信息。这样做的好处是，即使异常被重新引发了，我们仍然可以通过异常链追踪到原始异常的信息，从而更好地理解异常的发生原因。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce7946f12bc7421087dbea57336b34bc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1008&h=268&s=31976&e=png&b=282c34)

### 自定义异常（主动抛出异常raise）

> 在`Python`中，自定义异常类通常是通过继承内置的`Exception`类或其子类来创建的。

通过自定义异常类，我们可以为特定的错误情况提供更具有描述性和语义化的异常类型，从而更好地组织和处理异常情况。

```python
class CustomException(Exception):
    def __init__(self, message):
        self.message = message

    def __str__(self):
        return self.message

def divide(a, b):
    if b == 0:
        raise CustomException("除数不能为零")
    return a / b


try:
    result = divide(10, 0)
except CustomException as e:
    print("捕获到自定义异常：", e)

```

在上面的示例中，我们创建了一个名为`CustomException`的自定义异常类，它继承自内置的`Exception`类。在异常类中，我们定义了`__init__`方法来初始化异常对象，并传入一个错误信息。另外，我们还重写了`__str__`方法，以便在打印异常对象时输出错误信息。

在`divide`函数中，我们检查除数是否为零，如果除数为零，则抛出自定义的`CustomException`异常，并传入相应的错误信息。在异常处理块中，我们捕获并处理自定义异常，并打印出异常信息。

通过自定义异常，我们可以更好地组织和处理程序中可能出现的特定错误情况，提高了代码的可读性和可维护性。同时，它也使得程序的错误处理更加灵活和精确。

### 借助上下文管理器处理异常

> 异常处理不应该喧宾夺主!

上下文管理器是一种与`with`语句协同工作的特殊`Python`对象，它能够极大地简化异常处理流程，提升代码的优雅性和可读性。

我们来看一下`python工匠`里的例子：

```python
def upload_avatar(request):
    """用户上传新头像"""
    try:
        avatar_file = request.FILES['avatar']
    except KeyError:
        raise error_codes.AVATAR_FILE_NOT_PROVIDED

    try:
       resized_avatar_file = resize_avatar(avatar_file)
    except FileTooLargeError as e:
        raise error_codes.AVATAR_FILE_TOO_LARGE
    except ResizeAvatarError as e:
        raise error_codes.AVATAR_FILE_INVALID

    try:
        request.user.avatar = resized_avatar_file
        request.user.save()
    except Exception:
        raise error_codes.INTERNAL_SERVER_ERROR
    return HttpResponse({})

```

类似代码在座的各位肯定没少写~

这是一个处理用户上传头像的视图函数。这个函数内做了三件事情，并且针对每件事都做了异常捕获。如果做某件事时发生了异常，就返回对用户友好的错误到前端。

这样处理无可厚非，只是大量的`try-except`在主处理流程里使得我们一时间很难提炼出代码的核心逻辑，也不够`pythonic`。

利用上下文管理器就能够使得我们的代码更加清晰可读，具体操作是：

```python
class raise_api_error:
    """captures specified exception and raise ApiErrorCode instead

    :raises: AttributeError if code_name is not valid
    """
    def __init__(self, captures, code_name):
        self.captures = captures
        self.code = getattr(error_codes, code_name)

    def __enter__(self):
        
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        
        
        
        if exc_type is None:
            return False

        if exc_type == self.captures:
            raise self.code from exc_val
        return False

```

接口的主要逻辑就变成下面这样：

```python
def upload_avatar(request):
    """用户上传新头像"""
    with raise_api_error(KeyError, 'AVATAR_FILE_NOT_PROVIDED'):
        avatar_file = request.FILES['avatar']

    with raise_api_error(ResizeAvatarError, 'AVATAR_FILE_INVALID'),\
            raise_api_error(FileTooLargeError, 'AVATAR_FILE_TOO_LARGE'):
        resized_avatar_file = resize_avatar(avatar_file)

    with raise_api_error(Exception, 'INTERNAL_SERVER_ERROR'):
        request.user.avatar = resized_avatar_file
        request.user.save()
    return HttpResponse({})

```

简单明了了不少。

* * *

`contextlib`模块也提供了一些工具，用于支持与上下文管理器相关的操作。虽然`contextlib`模块本身并不直接用于异常处理，但它提供了一些用于创建上下文管理器的实用程序，这些上下文管理器可以帮助简化异常处理。

我们使用`contextlib.contextmanager`装饰器来创建一个简单的上下文管理器，用于捕获特定类型的异常：

```python
import contextlib
from loguru import logger


@contextlib.contextmanager
def handle_file_exceptions():
    try:
        yield
    except FileNotFoundError as e:
        logger.exception(f"File not found error: {e}")
    except PermissionError as e:
        logger.exception(f"Permission error: {e}")



with handle_file_exceptions():
    with open('nonexistent_file.txt', 'r') as file:
        content = file.read()

```

在这个例子中，`handle_file_exceptions`函数使用`contextlib.contextmanager`装饰器转变为一个上下文管理器。在调用`yield`之前的代码段被看作是上下文管理器的`__enter__`方法，在`yield`之后的代码段被看作是上下文管理器的`__exit__`方法。在`__exit__`方法中，我们捕获了`FileNotFoundError`和`PermissionError`，并打印了相应的错误信息。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c95d0b52d40c4389b3b0cd12f81ff9cd~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1634&h=559&s=102728&e=png&b=282c34)

虽然`contextlib`模块的主要目的是创建上下文管理器，但它也可以与异常处理一起使用，使得在特定上下文情境中进行异常处理变得更加方便和简洁。

### 使用装饰器处理异常

> `异常处理装饰器是另一种用于简化异常处理逻辑的技术`，它可以在函数调用时自动捕获并处理异常，从而减少重复代码量，提高代码的可维护性和可读性。

```python
from loguru import logger


def handle_exception(exception_handler):
    def decorator(func):
        def wrapper(*args, **kwargs):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                return exception_handler(e, *args, **kwargs)

        return wrapper

    return decorator



def custom_exception_handler(exception, *args, **kwargs):
    
    if isinstance(exception, ZeroDivisionError):
        logger.error(f"捕获到异常：{exception}")

    return None  



@handle_exception(custom_exception_handler)
def divide(a, b):
    return a / b


result = divide(10, 1)
logger.info(f"result: {result}")

result2 = divide(10, 0)
logger.info(f"result2: {result2}")

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cb0e4ffd82c4e3591186cf1d00e2312~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1149&h=89&s=21703&e=png&b=282c34)

> 使用异常处理装饰器可以使代码更加简洁、清晰，并且能够提高代码的复用性和可维护性。  
> `注意`: 异常处理装饰器的具体实现可以根据实际需求进行定制和扩展，以满足不同场景的需求。

```python
from functools import wraps
from typing import Callable, Dict, Type
from loguru import logger


class TryDecorator:
    def __init__(self):
        self.exception_handlers: Dict[Type[Exception], Callable] = {}

    def try_(self, func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                handler = self.get_exception_handler(type(e))
                if handler is None:
                    raise e
                return handler(func, e, *args, **kwargs)

        return wrapper

    def except_(self, *exceptions: Type[Exception]) -> Callable:
        def decorator(handler: Callable) -> Callable:
            for exception in exceptions:
                self.exception_handlers[exception] = handler
            return handler

        return decorator

    def get_exception_handler(self, exception_type: Type[Exception]) -> Callable:
        for exception, handler in self.exception_handlers.items():
            if issubclass(exception_type, exception):
                return handler
        return lambda *args, **kwargs: None  


try_decorator = TryDecorator()


@try_decorator.try_
def my_function():
    logger.info(1 / 0)
    logger.info('hello world')


@try_decorator.try_
def my_function2():
    raise IndexError("主动抛出了IndexError")


@try_decorator.except_(ZeroDivisionError)
def handle_zero_division_error(func: Callable, exception: Exception, *args, **kwargs):
    logger.error(f"ZeroDivisionError in function {func.__name__}: {exception}")


@try_decorator.except_(IndexError)
def handle_zero_division_error(func: Callable, exception: Exception, *args, **kwargs):
    logger.error(f"IndexError in function {func.__name__}: {exception}")


if __name__ == '__main__':
    my_function()
    my_function2()

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcee53b9c81a4f849c8f712800873ff7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1624&h=122&s=24633&e=png&b=282c34)

### 让你的函数返回一些有意义的、有类型的、安全的东西！

对于`python`函数的异常处理往往令我们比较的头疼。

比如，可能你的函数的正常流程是返回一个`字符串`，异常的时候是直接`raise`抛出呢还是返回一个特定值？这是一个问题！

> 传统的异常处理机制依赖于在函数内部抛出异常，然后在调用函数的地方进行捕获和处理。  
> 这种方式可能会导致代码分散、难以维护，同时将错误处理的责任分散到多个地方。异常可能会中断程序的正常流程，使得代码复杂和难以预测。

聪明如你，可能自己封装一个`统一的结构化响应`。

> `统一响应`：谁弄（异）哭（常）的，谁就要负责哄（包装）~

不过我要告诉你的是，使用`returns`模块可以将函数的结果值进行封装，这样就可以更灵活地处理异常情况。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d17ede64aba24aaf8c2d720368ec4b12~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1820&h=889&s=165173&e=png&b=ffffff)

`returns`模块提供了`Success`和`Failure`两种类型，分别代表函数成功执行和出现错误。这就使得函数返回的结果更加明确，可以更容易地进行组合、转换和处理。

`安装`：

```shell
pip install returns

```

我们改写下`使用装饰器处理异常`部分的示例代码：

```python
from typing import Callable
from returns.result import Success, Failure, Result
from loguru import logger
from functools import wraps


class TryDecorator:
    def __init__(self):
        
        self.exception_handlers = {}

    def try_(self, func: Callable) -> Callable[..., Result]:
        @wraps(func)
        def wrapper(*args, **kwargs) -> Result:
            try:
                result = func(*args, **kwargs)
                
                if isinstance(result, Result):
                    return result
                
                return Success(result)
            except Exception as e:
                
                handler = self.get_exception_handler(type(e))
                if handler is None:
                    
                    return Failure(e)
                
                return handler(func, e, *args, **kwargs)

        return wrapper

    def except_(self, *exceptions: type) -> Callable:
        def decorator(handler: Callable) -> Callable:
            
            for exception in exceptions:
                self.exception_handlers[exception] = handler
            return handler

        return decorator

    def get_exception_handler(self, exception_type: type) -> Callable:
        
        for exception, handler in self.exception_handlers.items():
            if issubclass(exception_type, exception):
                return handler
        
        return lambda *args, **kwargs: None


try_decorator = TryDecorator()


@try_decorator.except_(ZeroDivisionError)
def handle_zero_division_error(func: Callable, exception: Exception, *args, **kwargs) -> Result:
    
    logger.error(f"ZeroDivisionError in function {func.__name__}: {exception}")
    return Failure(exception)


@try_decorator.except_(IndexError)
def handle_index_error(func: Callable, exception: Exception, *args, **kwargs) -> Result:
    
    logger.error(f"IndexError in function {func.__name__}: {exception}")
    return Failure(exception)


@try_decorator.try_
def my_function(a, b):
    return Success(a / b)


@try_decorator.try_
def my_function2():
    raise IndexError("主动抛出了IndexError")


if __name__ == '__main__':
    
    
    result1 = my_function(10, 1)
    if isinstance(result1, Success):
        logger.info(f"没有异常：{result1}， result1: {result1.unwrap()}")

    
    result2 = my_function(10, 0)
    if isinstance(result2, Failure):
        logger.info(f"发生异常：{result2}， result2: {result2.failure()}")

    result3 = my_function2()
    if isinstance(result3, Failure):
        logger.error(f"Failed with: {result3.failure()}")

```

是不是清爽了许多，输出结果为：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8952070f3d8849d7ba271dca08d3550a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1333&h=126&s=43511&e=png&b=282c34)

*   `returns`模块促进了函数式编程风格的使用。
*   使用`returns`模块可以更容易地编写测试，因为你可以明确定义函数的行为并检查所得的结果。
*   使用返回值进行异常处理能够提供更多的灵活性和可预测性。通过返回值传递错误信息，调用者可以决定是否要处理错误，以及如何处理错误。

> 更多`returns`特性请阅读：  
> github: [github.com/dry-python/…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fdry-python%2Freturns "https://github.com/dry-python/returns")

更多场景举例
------

> 只做示例介绍，不一定遵循最佳实践。

### 基本的异常处理重试装饰器

下面是一个基本的异常处理重试装饰器的示例：

```python
import time
from functools import wraps
from typing import Callable


def retry_on_exception(max_retries: int, delay: float = 0.1):
    def decorator(func: Callable):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Exception caught: {e}. Retrying...")
                    time.sleep(delay)
            raise Exception(f"Failed after {max_retries} retries")

        return wrapper

    return decorator



@retry_on_exception(max_retries=3)
def potentially_unstable_function():
    import random
    if random.randint(1, 10) < 8:
        raise ValueError("Random error")
    else:
        return "Success"


if __name__ == "__main__":
    result = potentially_unstable_function()
    print(result)

```

在这个示例中，`retry_on_exception`是一个装饰器工厂函数，它接受最大重试次数和延迟作为参数。它返回一个装饰器，该装饰器会捕获被装饰函数可能抛出的异常，并在遇到异常时进行重试。

被装饰的函数`potentially_unstable_function`可能会抛出随机异常。我们使用`retry_on_exception`装饰器装饰它，以便在遇到异常时进行重试，最多重试三次。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b03d7355681440cb9238ee35c82cf6e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1103&h=210&s=45030&e=png&b=282c34)

你可以根据需要调整`max_retries`和`delay`参数来控制重试的次数和重试之间的延迟。

### 异常优雅重试

其实，很多轮子已经是卷中卷了，我们大可使用优雅的`retrying`模块。

```shell
pip install retrying

```

使用也很简单：

```python
from retrying import retry
import random
from loguru import logger

attempt = 0



@retry(stop_max_attempt_number=3, wait_fixed=100)  
def potentially_unstable_function():
    global attempt
    attempt += 1
    if random.randint(1, 10) < 8:
        logger.error(attempt)
        raise ValueError("Random error")
    else:
        return "Success"


if __name__ == "__main__":
    result = potentially_unstable_function()
    logger.info(result)

```

这里为了演示确实最大重试了`stop_max_attempt_number=3`，加了`attempt`，不然代码只会更少！

超过`3`次时报错如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32085d6b7d544058b984bebe926ed44b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1034&h=432&s=88724&e=png&b=282c34)

总结
--

恭喜毕业！异常处理是编写稳健、可维护Python代码的重要组成部分。以下是异常处理的关键点总结：

1.  捕获最具体的异常：为了更好地理解和处理错误，请捕获最具体的异常类型。
    
2.  不要使用单独的`except`处理所有异常：避免使用空的`except`块，因为它们会捕获所有异常，包括意外的情况，会增加调试的困难。
    
3.  谨慎使用`else`和`finally`块：`else`块用于在没有异常时执行特定代码，`finally`块用于确保资源释放或清理操作。
    
4.  记录异常信息：捕获异常后，通常应记录异常信息，以便诊断和修复问题。
    
5.  合理使用自定义异常： 当您的应用程序遇到特定异常条件时，建议创建一个自定义异常类，可以更清晰地表达和有效处理这些情况。
    
6.  避免不必要的异常处理：不应将异常处理用于预期的控制流，而应只在真正可能发生异常的地方使用它。
    
7.  使用装饰器，上下文管理器，`returns`库使你的异常处理更加的`pythonic`。
    