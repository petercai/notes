
## Code Breakdown

```python
from iso639 import languages
```
This imports the `languages` object directly from `iso639` (different from `Iso639()` class).

```python
if len(args["language"]) == 2:
```
Checks if the language code is 2 characters long (ISO 639-1 format like 'en', 'fr').

```python
lang_array = languages.get(part1=args["language"])
```
Gets language info using the 2-letter code. Note: it's `part1` not `alpha2` when using the `languages` object directly.

```python
if lang_array:
    args["language"] = lang_array.part3
    args["language_iso1"] = lang_array.part1
```
If a language is found, converts the 2-letter code to 3-letter code and stores both.

## Will get() return None or empty array?

**Answer: `languages.get()` returns `None` when no match is found, NOT an empty array.**

Here's proof:

```python
from iso639 import languages

# Valid code
result = languages.get(part1='en')
print(type(result))  # <class 'iso639.language.Language'>
print(result)        # Language object with attributes

# Invalid code
result = languages.get(part1='xx')
print(result)        # None
print(type(result))  # <class 'NoneType'>
```

## Complete Example

```python
from iso639 import languages

args = {"language": "en"}

if len(args["language"]) == 2:
    lang_array = languages.get(part1=args["language"])
    if lang_array:
        args["language"] = lang_array.part3        # 'eng'
        args["language_iso1"] = lang_array.part1   # 'en'
        print(f"Converted: {args}")
        # Output: Converted: {'language': 'eng', 'language_iso1': 'en'}
    else:
        print("Language not found")
```

## Key Differences: `languages.get()` vs `Iso639().get()`

### Using `languages` object directly:
```python
from iso639 import languages

lang = languages.get(part1='en')  # Returns None if not found
if lang:
    print(lang.part3)  # 'eng'
    print(lang.part1)  # 'en'
    print(lang.name)   # 'English'
```

### Using `Iso639()` class:
```python
from iso639 import Iso639

iso = Iso639()
try:
    lang = iso.get(alpha2='en')  # Raises KeyError if not found
    print(lang.alpha3)  # 'eng'
except KeyError:
    print("Not found")
```

## Attributes Comparison

| `languages` object | `Iso639()` class |
|-------------------|------------------|
| `part1` | `alpha2` |
| `part3` | `alpha3` |
| `part2b` | `bibliographic` |
| `name` | `name` |

## Better Error Handling for Your Code

```python
from iso639 import languages

args = {"language": "en"}

if len(args["language"]) == 2:
    lang_array = languages.get(part1=args["language"])
    
    if lang_array:
        # Successfully found the language
        args["language"] = lang_array.part3
        args["language_iso1"] = lang_array.part1
        print(f"Converted: ISO-639-1 '{args['language_iso1']}' -> ISO-639-3 '{args['language']}'")
    else:
        # Language not found - lang_array is None
        print(f"Warning: Invalid language code '{args['language']}'")
        # You might want to keep the original or set a default
```

## Test Cases

```python
from iso639 import languages

def test_language_conversion(code):
    args = {"language": code}
    
    if len(args["language"]) == 2:
        lang_array = languages.get(part1=args["language"])
        
        if lang_array:
            args["language"] = lang_array.part3
            args["language_iso1"] = lang_array.part1
            return args
        else:
            return None
    return args

# Valid codes
print(test_language_conversion('en'))  # {'language': 'eng', 'language_iso1': 'en'}
print(test_language_conversion('fr'))  # {'language': 'fra', 'language_iso1': 'fr'}
print(test_language_conversion('de'))  # {'language': 'deu', 'language_iso1': 'de'}

# Invalid code
print(test_language_conversion('xx'))  # None

# Already 3-letter code (skips conversion)
print(test_language_conversion('eng'))  # {'language': 'eng'}
```

## Summary

- **Returns**: `None` when language not found, **NOT an empty array**
- **Usage**: `if lang_array:` is correct way to check
- **Purpose**: Converts ISO 639-1 (2-letter) to ISO 639-3 (3-letter) codes
- **Behavior**: Silent failure (returns `None`) instead of raising exception