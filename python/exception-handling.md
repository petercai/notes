# show Exception
Here are several ways to print only the current file and line number without the full stack trace:

## Option 1: Using `sys.exc_info()` (Recommended)

```python
import sys
import traceback

try:
    # some code that raises exception
    raise ValueError("Something went wrong")
except Exception as e:
    exc_type, exc_obj, exc_tb = sys.exc_info()
    fname = exc_tb.tb_frame.f_code.co_filename
    lineno = exc_tb.tb_lineno
    print(f"Error in {fname}:{lineno} - {e}")
    # Output: Error in /path/to/file.py:5 - Something went wrong
```

## Option 2: Using `traceback.extract_tb()`

```python
import sys
import traceback

try:
    raise ValueError("Something went wrong")
except Exception as e:
    exc_type, exc_value, exc_tb = sys.exc_info()
    # Get the last frame (where exception occurred)
    tb_info = traceback.extract_tb(exc_tb)[-1]
    print(f"Error in {tb_info.filename}:{tb_info.lineno} - {e}")
```

## Option 3: Simple one-liner with `__traceback__`

```python
try:
    raise ValueError("Something went wrong")
except Exception as e:
    tb = e.__traceback__
    print(f"Error at line {tb.tb_lineno}: {e}")
    # Output: Error at line 2: Something went wrong
```

## Option 4: Get just filename (not full path)

```python
import os
import sys

try:
    raise ValueError("Something went wrong")
except Exception as e:
    exc_type, exc_obj, exc_tb = sys.exc_info()
    fname = os.path.basename(exc_tb.tb_frame.f_code.co_filename)
    lineno = exc_tb.tb_lineno
    print(f"Error in {fname}:{lineno} - {e}")
    # Output: Error in script.py:5 - Something went wrong
```

## Option 5: Helper function for reuse

```python
import sys
import os

def get_exception_info(e):
    """Returns formatted error info with file and line number"""
    exc_type, exc_obj, exc_tb = sys.exc_info()
    fname = os.path.basename(exc_tb.tb_frame.f_code.co_filename)
    lineno = exc_tb.tb_lineno
    return f"{fname}:{lineno} - {type(e).__name__}: {e}"

# Usage
try:
    raise ValueError("Something went wrong")
except Exception as e:
    print(f"Error: {get_exception_info(e)}")
    # Output: Error: script.py:10 - ValueError: Something went wrong
```

## Option 6: Get function name too

```python
import sys
import os

try:
    def my_function():
        raise ValueError("Something went wrong")
    
    my_function()
except Exception as e:
    exc_type, exc_obj, exc_tb = sys.exc_info()
    fname = os.path.basename(exc_tb.tb_frame.f_code.co_filename)
    func_name = exc_tb.tb_frame.f_code.co_name
    lineno = exc_tb.tb_lineno
    print(f"Error in {fname}:{lineno} in {func_name}() - {e}")
    # Output: Error in script.py:3 in my_function() - Something went wrong
```

## Comparison Table

| Method | Output Example | Use Case |
|--------|---------------|----------|
| `sys.exc_info()` | `Error in /full/path/file.py:42 - ValueError: msg` | Full path needed |
| `os.path.basename()` | `Error in file.py:42 - ValueError: msg` | Just filename |
| `e.__traceback__` | `Error at line 42: ValueError: msg` | Simplest, line only |
| `traceback.extract_tb()` | `Error in file.py:42 - ValueError: msg` | More detail available |

## Complete Example with Logging

```python
import sys
import os
import logging

logging.basicConfig(level=logging.ERROR)
logger = logging.getLogger(__name__)

def process_data(data):
    try:
        # Your code here
        result = 1 / 0
        return result
    except Exception as e:
        exc_type, exc_obj, exc_tb = sys.exc_info()
        fname = os.path.basename(exc_tb.tb_frame.f_code.co_filename)
        lineno = exc_tb.tb_lineno
        logger.error(f"{fname}:{lineno} - {type(e).__name__}: {e}")
        return None

process_data([1, 2, 3])
# Output: ERROR:__main__:script.py:11 - ZeroDivisionError: division by zero
```

## Best Practice Recommendation

For most cases, I recommend **Option 1** or **Option 5**:

```python
import sys
import os

try:
    # your code
    raise ValueError("Something went wrong")
except Exception as e:
    exc_type, exc_obj, exc_tb = sys.exc_info()
    fname = os.path.basename(exc_tb.tb_frame.f_code.co_filename)
    lineno = exc_tb.tb_lineno
    print(f"Error in {fname}:{lineno} - {type(e).__name__}: {e}")
```

This gives you:
- ✅ Filename (without full path)
- ✅ Line number
- ✅ Exception type
- ✅ Exception message
- ✅ No full stack trace

# show the location when raise vs. catch

**Excellent question!** `sys.exc_info()` returns information about **where the exception was originally raised**, NOT where `print_error()` is called from.

## Demonstration

```python
import sys
import os

class ErrorHandler:
    @staticmethod
    def print_error(e: Exception):
        exc_type, exc_obj, exc_tb = sys.exc_info()
        fname = os.path.basename(exc_tb.tb_frame.f_code.co_filename)
        lineno = exc_tb.tb_lineno
        print(f"Error in {fname}:{lineno} - {e}")

def function_a():
    return 1 / 0  # Line 10 - Exception raised HERE

def function_b():
    try:
        function_a()  # Line 14
    except Exception as e:
        ErrorHandler.print_error(e)  # Line 16 - Called from here

function_b()

# Output: Error in script.py:10 - division by zero
#         ^^^^^^^^^^^^^^^^^^^^
#         Points to where exception was RAISED (line 10)
#         NOT where print_error was called (line 16)
```

## Visual Explanation

```python
# Line 5: class ErrorHandler definition
# Line 10: def function_a(): return 1/0  ← Exception RAISED here
# Line 14: function_a()                   ← Exception propagates through
# Line 16: ErrorHandler.print_error(e)   ← sys.exc_info() called here

# sys.exc_info() returns info about line 10 (where exception was raised)
```

## More Complex Example - Multiple Call Levels

```python
import sys
import os

class ErrorHandler:
    @staticmethod
    def print_error(e: Exception):
        exc_type, exc_obj, exc_tb = sys.exc_info()
        fname = os.path.basename(exc_tb.tb_frame.f_code.co_filename)
        lineno = exc_tb.tb_lineno
        print(f"Error in {fname}:{lineno} - {e}")

def level_3():
    data = [1, 2, 3]
    return data[10]  # Line 12 - IndexError raised HERE

def level_2():
    return level_3()  # Line 15

def level_1():
    try:
        level_2()  # Line 19
    except Exception as e:
        ErrorHandler.print_error(e)  # Line 21

level_1()

# Output: Error in script.py:12 - list index out of range
#         ^^^^^^^^^^^^^^^^^^^^
#         Points to line 12 where exception was RAISED
#         NOT line 21 where print_error was called
```

## What if You Want the Catch Location Instead?

If you need to know where the exception was **caught** (not raised), you can use `inspect`:

```python
import sys
import os
import inspect

class ErrorHandler:
    @staticmethod
    def print_error_with_catch_location(e: Exception):
        # Get where exception was raised
        exc_type, exc_obj, exc_tb = sys.exc_info()
        if exc_tb:
            raised_fname = os.path.basename(exc_tb.tb_frame.f_code.co_filename)
            raised_lineno = exc_tb.tb_lineno
        else:
            raised_fname = "unknown"
            raised_lineno = "unknown"
        
        # Get where this method was called from (catch location)
        frame = inspect.currentframe()
        caller_frame = frame.f_back
        catch_fname = os.path.basename(caller_frame.f_code.co_filename)
        catch_lineno = caller_frame.f_lineno
        
        print(f"Exception raised at {raised_fname}:{raised_lineno}")
        print(f"Caught and handled at {catch_fname}:{catch_lineno}")
        print(f"Error: {e}")

def problematic_function():
    return 1 / 0  # Line 25

def handler_function():
    try:
        problematic_function()
    except Exception as e:
        ErrorHandler.print_error_with_catch_location(e)  # Line 31

handler_function()

# Output:
# Exception raised at script.py:25
# Caught and handled at script.py:31
# Error: division by zero
```

## Full Stack Trace Example

To see the entire call stack from where it was raised to where it was caught:

```python
import sys
import os
import traceback

class ErrorHandler:
    @staticmethod
    def print_full_trace(e: Exception):
        """Show where exception was raised (first) and full call stack"""
        exc_type, exc_obj, exc_tb = sys.exc_info()
        
        # Get the original location (where exception was raised)
        if exc_tb:
            fname = os.path.basename(exc_tb.tb_frame.f_code.co_filename)
            lineno = exc_tb.tb_lineno
            print(f"Exception raised at: {fname}:{lineno}")
        
        # Show full stack trace
        print("\nFull stack trace:")
        tb_lines = traceback.format_tb(exc_tb)
        for line in tb_lines:
            print(line.rstrip())
        
        print(f"\nError: {type(e).__name__}: {e}")
    
    @staticmethod
    def print_error(e: Exception):
        """Simple version - just where it was raised"""
        exc_type, exc_obj, exc_tb = sys.exc_info()
        fname = os.path.basename(exc_tb.tb_frame.f_code.co_filename)
        lineno = exc_tb.tb_lineno
        print(f"Error in {fname}:{lineno} - {e}")

def deep_function():
    return 1 / 0  # Line 32

def middle_function():
    deep_function()  # Line 35

def top_function():
    try:
        middle_function()  # Line 39
    except Exception as e:
        print("=== Simple version ===")
        ErrorHandler.print_error(e)
        
        print("\n=== Full trace version ===")
        ErrorHandler.print_full_trace(e)

top_function()

# Output:
# === Simple version ===
# Error in script.py:32 - division by zero
#
# === Full trace version ===
# Exception raised at: script.py:32
#
# Full stack trace:
#   File "script.py", line 39, in top_function
#     middle_function()
#   File "script.py", line 35, in middle_function
#     deep_function()
#   File "script.py", line 32, in deep_function
#     return 1 / 0
#
# Error: ZeroDivisionError: division by zero
```

## Key Takeaways

1. **`sys.exc_info()`** returns info about where the exception was **raised** (original source)
2. **NOT** where `print_error()` is called from (catch location)
3. This is usually what you want - to know the root cause location
4. If you need catch location, use `inspect.currentframe().f_back`
5. For full stack trace, use `traceback.format_tb()`

```python
# Summary:
try:
    x = 1 / 0  # Line A - Exception RAISED here ←─┐
except Exception as e:                        #   │
    ErrorHandler.print_error(e)  # Line B     #   │
#                                             #   │
# sys.exc_info() points to Line A ────────────────┘
# NOT Line B
```

This behavior is correct and useful because you typically want to know **where the problem originated**, not where you're handling it.


# Error vs. Exception

## Key Differences Between Error and Exception in Python

In Python, **Error** and **Exception** are related but have important distinctions:

## 1. Hierarchy

```python
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
├── Exception                    # ← Most exceptions inherit from here
│   ├── StopIteration
│   ├── ArithmeticError
│   │   ├── ZeroDivisionError
│   │   ├── OverflowError
│   │   └── FloatingPointError
│   ├── AttributeError
│   ├── ImportError
│   ├── LookupError
│   │   ├── IndexError
│   │   └── KeyError
│   ├── NameError
│   ├── OSError
│   │   ├── FileNotFoundError
│   │   └── PermissionError
│   ├── RuntimeError
│   ├── TypeError
│   ├── ValueError
│   └── ...
```

## 2. "Error" is Just a Naming Convention

```python
# These are all Exceptions (inherit from Exception class)
try:
    x = 1 / 0
except ZeroDivisionError as e:  # Named "Error" but is an Exception
    print(type(e))  # <class 'ZeroDivisionError'>
    print(isinstance(e, Exception))  # True

try:
    my_list = [1, 2, 3]
    print(my_list[10])
except IndexError as e:  # Named "Error" but is an Exception
    print(isinstance(e, Exception))  # True
```

**The "Error" suffix is just a naming convention** - these are all Exception subclasses!

## 3. When to Catch `Exception` vs Specific Errors

### ❌ Bad Practice - Too Broad
```python
try:
    result = int(user_input)
    file = open('data.txt')
except Exception as e:  # Catches EVERYTHING including unexpected errors
    print("Something went wrong")
```

### ✅ Good Practice - Specific
```python
try:
    result = int(user_input)
    file = open('data.txt')
except ValueError as e:  # Specific to conversion errors
    print(f"Invalid input: {e}")
except FileNotFoundError as e:  # Specific to file errors
    print(f"File not found: {e}")
except Exception as e:  # Catch any other unexpected exceptions
    print(f"Unexpected error: {e}")
```

## 4. Exception Types You Should Know

### Recoverable Exceptions (Catch These)
```python
# ValueError - Invalid value
try:
    int("abc")
except ValueError:
    print("Cannot convert to integer")

# KeyError - Missing dictionary key
try:
    my_dict = {'a': 1}
    print(my_dict['b'])
except KeyError:
    print("Key not found")

# FileNotFoundError - File doesn't exist
try:
    open('nonexistent.txt')
except FileNotFoundError:
    print("File not found")

# IndexError - List index out of range
try:
    my_list = [1, 2, 3]
    print(my_list[10])
except IndexError:
    print("Index out of range")

# TypeError - Wrong type
try:
    "string" + 5
except TypeError:
    print("Cannot add string and integer")

# AttributeError - Attribute doesn't exist
try:
    my_str = "hello"
    my_str.nonexistent_method()
except AttributeError:
    print("Attribute not found")
```

### System Exceptions (Usually Don't Catch)
```python
# KeyboardInterrupt - User pressed Ctrl+C
try:
    while True:
        pass
except KeyboardInterrupt:
    print("Interrupted by user")  # OK to catch for cleanup

# SystemExit - sys.exit() called
try:
    import sys
    sys.exit(0)
except SystemExit:
    print("Exit called")  # Usually shouldn't catch this
```

## 5. Creating Custom Exceptions

### Use "Error" suffix for consistency
```python
# Good naming convention
class DatabaseError(Exception):
    """Raised when database operation fails"""
    pass

class ValidationError(Exception):
    """Raised when validation fails"""
    pass

class APIError(Exception):
    """Raised when API call fails"""
    def __init__(self, status_code, message):
        self.status_code = status_code
        self.message = message
        super().__init__(f"API Error {status_code}: {message}")

# Usage
try:
    if not valid_data:
        raise ValidationError("Invalid email format")
except ValidationError as e:
    print(f"Validation failed: {e}")
```

## 6. Practical Patterns

### Pattern 1: Specific to General
```python
def process_file(filename):
    try:
        with open(filename) as f:
            data = f.read()
            result = int(data)
            return result
    except FileNotFoundError:
        print(f"File {filename} not found")
        return None
    except ValueError:
        print("File content is not a valid number")
        return None
    except PermissionError:
        print("No permission to read file")
        return None
    except Exception as e:  # Catch unexpected errors
        print(f"Unexpected error: {e}")
        return None
```

### Pattern 2: Re-raising with Context
```python
def load_config(filename):
    try:
        with open(filename) as f:
            return json.load(f)
    except FileNotFoundError:
        raise ConfigurationError(f"Config file {filename} not found")
    except json.JSONDecodeError as e:
        raise ConfigurationError(f"Invalid JSON in {filename}: {e}")
```

### Pattern 3: Cleanup with finally
```python
def process_data():
    file = None
    try:
        file = open('data.txt')
        # process file
    except FileNotFoundError as e:
        print(f"Error: {e}")
    except Exception as e:
        print(f"Unexpected error: {e}")
    finally:
        if file:
            file.close()  # Always cleanup
```

## 7. What NOT to Catch

```python
# ❌ Don't catch these (they indicate serious problems)
try:
    # code
except BaseException:  # Too broad! Catches SystemExit, KeyboardInterrupt
    pass

# ❌ Don't catch bare Exception for critical code
try:
    critical_operation()
except Exception:  # Hides bugs
    pass

# ❌ Don't silence exceptions
try:
    important_operation()
except:  # Never use bare except!
    pass
```

## 8. Summary Table

| Term | What It Is | When to Use |
|------|-----------|-------------|
| `Exception` | Base class for catchable exceptions | Catch as last resort after specific exceptions |
| `*Error` classes | Specific exception types (ValueError, TypeError, etc.) | Catch these specifically when you expect them |
| `BaseException` | Root of all exceptions | Almost never catch this |
| Custom `*Error` | Your own exception classes | Create for domain-specific errors |

## Key Takeaways

1. **"Error" is just naming** - classes like `ValueError`, `TypeError` are all `Exception` subclasses
2. **Catch specific exceptions first** - use `except ValueError:` not just `except Exception:`
3. **Use "Error" suffix** for custom exceptions (e.g., `ValidationError`, `DatabaseError`)
4. **Don't catch everything** - let unexpected errors propagate so you can fix bugs
5. **Exception hierarchy matters** - catch specific exceptions before general ones

```python
# Best Practice Example
try:
    result = risky_operation()
except ValueError as e:      # Most specific
    handle_value_error(e)
except IOError as e:         # More specific
    handle_io_error(e)
except Exception as e:       # General fallback
    log_unexpected_error(e)
    raise  # Re-raise to not hide bugs
```