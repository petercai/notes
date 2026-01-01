# type check

**No, Python type hints are NOT validated at runtime by default.** They are only for static type checkers and documentation.

## Runtime Behavior

```python
def showException(e: Exception):
    print(f"Error: {e}")

# These ALL work at runtime - no validation!
showException(ValueError("test"))           # ✓ Works (is an Exception)
showException("not an exception")           # ✓ Works (wrong type, but no error!)
showException(123)                          # ✓ Works (wrong type, but no error!)
showException(None)                         # ✓ Works (wrong type, but no error!)

# Python does NOT enforce type hints at runtime
```

## How to Check Types

### Option 1: Manual isinstance() check
```python
def showException(e: Exception):
    if not isinstance(e, Exception):
        raise TypeError(f"Expected Exception, got {type(e).__name__}")
    print(f"Error: {e}")

# Now this works:
showException(ValueError("test"))  # ✓ OK

# This raises TypeError:
showException("not an exception")  # ✗ TypeError: Expected Exception, got str
```

### Option 2: Use static type checker (mypy, pyright, etc.)

```python
# file: script.py
def showException(e: Exception):
    print(f"Error: {e}")

showException("not an exception")  # Runtime: works fine
```

Run mypy:
```bash
$ mypy script.py
script.py:4: error: Argument 1 to "showException" has incompatible type "str"; expected "Exception"
```

**But the script still runs** - mypy just warns you before runtime.

### Option 3: Runtime type checking with decorators

```python
from functools import wraps
from typing import get_type_hints

def enforce_types(func):
    """Decorator to enforce type hints at runtime"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        hints = get_type_hints(func)
        
        # Check positional arguments
        for arg, (arg_name, arg_type) in zip(args, hints.items()):
            if arg_name != 'return' and not isinstance(arg, arg_type):
                raise TypeError(f"{arg_name} must be {arg_type.__name__}, got {type(arg).__name__}")
        
        return func(*args, **kwargs)
    return wrapper

@enforce_types
def showException(e: Exception):
    print(f"Error: {e}")

# Now it's validated at runtime:
showException(ValueError("test"))      # ✓ Works
showException("not an exception")      # ✗ TypeError: e must be Exception, got str
```

### Option 4: Use pydantic for validation

```python
from pydantic import validate_call, ValidationError

@validate_call
def showException(e: Exception):
    print(f"Error: {e}")

# This works:
showException(ValueError("test"))

# This raises ValidationError:
try:
    showException("not an exception")
except ValidationError as err:
    print(err)
    # ValidationError: 1 validation error for showException
    # e
    #   Input should be an instance of Exception
```

## Practical Example: Exception Handler

```python
# Without validation (type hint is just documentation)
def log_exception(e: Exception, context: str):
    print(f"[{context}] {type(e).__name__}: {e}")

# This works but might not be what you want:
log_exception("some string", "user_login")  # No error!


# With validation
def log_exception_safe(e: Exception, context: str):
    if not isinstance(e, Exception):
        # Convert to proper exception or raise error
        e = RuntimeError(str(e))
    print(f"[{context}] {type(e).__name__}: {e}")

# Now it's safe:
log_exception_safe("some string", "user_login")  # Converts to RuntimeError
log_exception_safe(ValueError("bad input"), "user_login")  # Uses as-is
```

## Comparison Table

| Method | Runtime Validation | Setup Required | Performance Impact |
|--------|-------------------|----------------|-------------------|
| Type hints only | ❌ No | None | None |
| `isinstance()` check | ✅ Yes | Manual | Minimal |
| mypy/pyright | ✅ Before runtime | Install tool | None (pre-runtime) |
| Custom decorator | ✅ Yes | Write decorator | Small |
| pydantic | ✅ Yes | Install library | Small-Medium |

## Best Practices

### For library/public API functions:
```python
def process_error(e: Exception, severity: str = "error") -> None:
    """
    Process an exception with logging.
    
    Args:
        e: Exception to process
        severity: Log severity level
        
    Raises:
        TypeError: If e is not an Exception instance
    """
    if not isinstance(e, Exception):
        raise TypeError(f"e must be an Exception, got {type(e).__name__}")
    
    # Process the exception
    print(f"[{severity.upper()}] {type(e).__name__}: {e}")
```

### For internal functions:
```python
# Just use type hints + mypy in CI/CD
def _internal_handler(e: Exception) -> str:
    return f"{type(e).__name__}: {e}"
```

### For catching exceptions with type hints:
```python
import sys
from typing import Type

def handle_specific_error(
    error_type: Type[Exception],
    handler: callable
):
    """Handle a specific exception type with a custom handler"""
    try:
        # some code
        raise ValueError("test")
    except error_type as e:
        handler(e)

# Usage:
def my_handler(e: Exception):
    print(f"Handled: {e}")

handle_specific_error(ValueError, my_handler)
```

## Key Takeaways

1. **Type hints are NOT enforced at runtime** - they're documentation and for static checkers
2. **Use `isinstance()`** if you need runtime validation
3. **Use mypy/pyright** in your development workflow for static checking
4. **Use pydantic** if you need automatic runtime validation
5. **For critical functions**, add explicit `isinstance()` checks with clear error messages

```python
# Summary: What actually happens
def func(e: Exception):  # Type hint = documentation only
    pass

func("wrong type")  # ✓ Python allows this at runtime

# What you should do:
def func_safe(e: Exception):
    if not isinstance(e, Exception):  # Manual check
        raise TypeError("e must be Exception")
    pass

func_safe("wrong type")  # ✗ Now raises TypeError
```