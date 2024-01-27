# Python装饰器的用途和实例 
### 说明

> 装饰器是Python中非常有用的工具，它们可以用于修改或扩展函数或类的行为，而无需修改其原始定义。装饰器通常是一个函数，它接受一个函数作为参数，并返回一个新的函数或类。下面我们将介绍一些常见的装饰器用途和示例。

1.  记录日志

装饰器可以用于记录函数的调用信息，比如函数的名称、参数和返回值等。这对于调试和性能分析非常有用。以下是一个简单的记录日志的装饰器示例：

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print(f"Calling function: {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Function {func.__name__} returned: {result}")
        return result
    return wrapper

@logger
def add(a, b):
    return a + b

result = add(2, 3)  
                    

```

2.  输入验证

装饰器可以用于验证函数的输入参数是否符合要求，如果不符合，则抛出异常或进行其他处理。以下是一个简单的输入验证装饰器示例：

```python
def validate_input(func):
    def wrapper(*args, **kwargs):
        for arg in args:
            if not isinstance(arg, int):
                raise ValueError("Invalid input: integers required")
        for value in kwargs.values():
            if not isinstance(value, int):
                raise ValueError("Invalid input: integers required")
        return func(*args, **kwargs)
    return wrapper

@validate_input
def multiply(a, b):
    return a * b

result = multiply(2, 3)  
result = multiply(2, "3")  

```

3.  缓存结果

装饰器可以用于缓存函数的计算结果，避免重复计算，提高执行效率。以下是一个简单的缓存结果装饰器示例：

```less
import functools

def memoize(func):
    cache = {}
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        key = args + tuple(sorted(kwargs.items()))
        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]
    return wrapper

@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

result = fibonacci(50)  # 返回：12586269025

```

4.  权限检查

装饰器可以用于检查用户权限，确保只有具有特定权限的用户才能调用某些函数。这在Web应用程序中特别有用，可以帮助确保用户只能访问其具有权限的资源。以下是一个简单的权限检查装饰器示例：

```python
def check_permission(func):
    def wrapper(*args, **kwargs):
        if user_has_permission():
            return func(*args, **kwargs)
        else:
            raise PermissionError("User does not have permission to access this resource")
    return wrapper

@check_permission
def delete_file(file_path):
    
    pass

```

5.  性能分析

装饰器可以用于对函数的性能进行分析，比如计算函数的执行时间、调用次数等信息。这对于优化程序性能非常有帮助，可以帮助开发人员找到程序的瓶颈所在。以下是一个简单的性能分析装饰器示例：

```python
import time
def profile_performance(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"Function {func.__name__} took {end_time - start_time} seconds to execute")
        return result
    return wrapper

@profile_performance
def heavy_computation():
    
    pass

```

### 总结

> 通过上述示例，我们可以看到装饰器的强大功能。它们可以帮助我们轻松地修改和扩展函数或类的行为，而无需对其进行直接修改。装饰器在Python中被广泛使用，因为它们使代码更加模块化、可复用和易于维护。希望本文能够帮助大家理解装饰器的用途和实例，并在实际项目中灵活运用它们。