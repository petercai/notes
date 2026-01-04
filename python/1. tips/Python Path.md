I'll explain Python's `Path` object methods from `pathlib` in detail, including `stem()`, `name()`, and many others.

## Import and Basic Setup

```python
from pathlib import Path

# Example paths to work with
file_path = Path("/home/user/documents/report.2023.pdf")
dir_path = Path("/home/user/documents")
```

## Core Path Properties and Methods

### 1. `name` - Full filename with extension

```python
from pathlib import Path

path = Path("/home/user/documents/report.2023.pdf")
print(path.name)
# Output: 'report.2023.pdf'

path = Path("/home/user/documents")
print(path.name)
# Output: 'documents'

path = Path("/home/user/documents/")  # Trailing slash ignored
print(path.name)
# Output: 'documents'
```

### 2. `stem` - Filename WITHOUT extension

```python
from pathlib import Path

path = Path("/home/user/documents/report.2023.pdf")
print(path.stem)
# Output: 'report.2023'  ← Removes only the LAST extension

path = Path("archive.tar.gz")
print(path.stem)
# Output: 'archive.tar'  ← Only removes .gz

path = Path("README")  # No extension
print(path.stem)
# Output: 'README'
```

**Important:** `stem` only removes the **last** extension, not all of them!

### 3. `suffix` - File extension (last one)

```python
from pathlib import Path

path = Path("/home/user/documents/report.pdf")
print(path.suffix)
# Output: '.pdf'  ← Includes the dot

path = Path("archive.tar.gz")
print(path.suffix)
# Output: '.gz'  ← Only the LAST extension

path = Path("README")  # No extension
print(path.suffix)
# Output: ''  ← Empty string

path = Path(".gitignore")  # Starts with dot
print(path.suffix)
# Output: ''  ← Not considered an extension
```

### 4. `suffixes` - All file extensions (as list)

```python
from pathlib import Path

path = Path("archive.tar.gz")
print(path.suffixes)
# Output: ['.tar', '.gz']

path = Path("report.backup.2023.pdf")
print(path.suffixes)
# Output: ['.backup', '.2023', '.pdf']

path = Path("README")
print(path.suffixes)
# Output: []

path = Path("file.txt")
print(path.suffixes)
# Output: ['.txt']
```

### 5. `parent` - Parent directory

```python
from pathlib import Path

path = Path("/home/user/documents/report.pdf")
print(path.parent)
# Output: /home/user/documents

print(path.parent.parent)
# Output: /home/user

print(path.parent.parent.parent)
# Output: /home

print(path.parent.parent.parent.parent)
# Output: /

# Can't go beyond root
print(path.parent.parent.parent.parent.parent)
# Output: /
```

### 6. `parents` - All parent directories (sequence)

```python
from pathlib import Path

path = Path("/home/user/documents/report.pdf")

# Access by index
print(path.parents[0])  # Output: /home/user/documents
print(path.parents[1])  # Output: /home/user
print(path.parents[2])  # Output: /home
print(path.parents[3])  # Output: /

# Iterate through all parents
for parent in path.parents:
    print(parent)
# Output:
# /home/user/documents
# /home/user
# /home
# /
```

### 7. `parts` - All path components as tuple

```python
from pathlib import Path

path = Path("/home/user/documents/report.pdf")
print(path.parts)
# Output: ('/', 'home', 'user', 'documents', 'report.pdf')

# On Windows:
path = Path("C:/Users/John/Desktop/file.txt")
print(path.parts)
# Output: ('C:\\', 'Users', 'John', 'Desktop', 'file.txt')

# Iterate through parts
for part in path.parts:
    print(part)
```

### 8. `anchor` - Root part of path

```python
from pathlib import Path

# Unix/Linux/Mac
path = Path("/home/user/documents/report.pdf")
print(path.anchor)
# Output: '/'

# Windows
path = Path("C:/Users/John/Desktop/file.txt")
print(path.anchor)
# Output: 'C:\\'

# Relative path (no anchor)
path = Path("documents/report.pdf")
print(path.anchor)
# Output: ''
```

## Comparison Table

```python
from pathlib import Path

path = Path("/home/user/docs/archive.tar.gz")

print(f"Full path:    {path}")               # /home/user/docs/archive.tar.gz
print(f"name:         {path.name}")          # archive.tar.gz
print(f"stem:         {path.stem}")          # archive.tar
print(f"suffix:       {path.suffix}")        # .gz
print(f"suffixes:     {path.suffixes}")      # ['.tar', '.gz']
print(f"parent:       {path.parent}")        # /home/user/docs
print(f"parents[0]:   {path.parents[0]}")    # /home/user/docs
print(f"parents[1]:   {path.parents[1]}")    # /home/user
print(f"parts:        {path.parts}")         # ('/', 'home', 'user', 'docs', 'archive.tar.gz')
print(f"anchor:       {path.anchor}")        # /
```

## Practical Examples

### Example 1: Remove All Extensions

```python
from pathlib import Path

def get_base_name(path):
    """Get filename without ANY extensions"""
    p = Path(path)
    name = p.name
    # Remove all suffixes
    for suffix in p.suffixes:
        name = name.replace(suffix, '', 1)
    return name

print(get_base_name("archive.tar.gz"))        # archive
print(get_base_name("report.backup.pdf"))     # report
print(get_base_name("document.txt"))          # document
```

### Example 2: Change File Extension

```python
from pathlib import Path

def change_extension(path, new_ext):
    """Change file extension"""
    p = Path(path)
    # Ensure new_ext starts with dot
    if not new_ext.startswith('.'):
        new_ext = '.' + new_ext
    
    # Replace extension
    return p.parent / (p.stem + new_ext)

original = Path("document.txt")
new_path = change_extension(original, ".pdf")
print(new_path)  # document.pdf

original = Path("archive.tar.gz")
new_path = change_extension(original, ".zip")
print(new_path)  # archive.tar.zip  ← Only changes last extension
```

### Example 3: Add Suffix Before Extension

```python
from pathlib import Path

def add_suffix_before_ext(path, suffix):
    """Add suffix before file extension"""
    p = Path(path)
    return p.parent / (p.stem + suffix + p.suffix)

original = Path("report.pdf")
new_path = add_suffix_before_ext(original, "_final")
print(new_path)  # report_final.pdf

original = Path("/home/user/image.jpg")
new_path = add_suffix_before_ext(original, "_resized")
print(new_path)  # /home/user/image_resized.jpg
```

### Example 4: Get Relative Path

```python
from pathlib import Path

base = Path("/home/user/projects")
full = Path("/home/user/projects/python/script.py")

relative = full.relative_to(base)
print(relative)  # python/script.py

# With different base
base = Path("/home/user")
relative = full.relative_to(base)
print(relative)  # projects/python/script.py
```

### Example 5: Working with Parent Directories

```python
from pathlib import Path

path = Path("/home/user/documents/2023/reports/final/report.pdf")

# Get specific ancestor
print(path.parents[0])  # /home/user/documents/2023/reports/final
print(path.parents[1])  # /home/user/documents/2023/reports
print(path.parents[2])  # /home/user/documents/2023

# Find parent containing specific directory name
def find_parent_with_name(path, name):
    """Find parent directory with specific name"""
    for parent in Path(path).parents:
        if parent.name == name:
            return parent
    return None

docs_parent = find_parent_with_name(path, "documents")
print(docs_parent)  # /home/user/documents
```

## Additional Useful Methods

### `is_absolute()` - Check if path is absolute

```python
from pathlib import Path

print(Path("/home/user/file.txt").is_absolute())  # True
print(Path("documents/file.txt").is_absolute())   # False
print(Path("../file.txt").is_absolute())          # False
```

### `is_relative_to()` - Check if path is relative to another (Python 3.9+)

```python
from pathlib import Path

path = Path("/home/user/documents/file.txt")
print(path.is_relative_to("/home/user"))      # True
print(path.is_relative_to("/home/admin"))     # False
```

### `resolve()` - Get absolute path, resolving symlinks

```python
from pathlib import Path

# Resolve relative path to absolute
path = Path("../documents/file.txt")
absolute = path.resolve()
print(absolute)  # /home/user/documents/file.txt

# Resolve symlinks
symlink = Path("/home/user/link_to_file")
real_path = symlink.resolve()
print(real_path)  # /actual/location/of/file
```

### `exists()`, `is_file()`, `is_dir()` - Check path type

```python
from pathlib import Path

path = Path("/home/user/documents/file.txt")

print(path.exists())   # True if exists (file or directory)
print(path.is_file())  # True if it's a file
print(path.is_dir())   # True if it's a directory
print(path.is_symlink())  # True if it's a symbolic link
```

### `with_name()` - Replace filename

```python
from pathlib import Path

path = Path("/home/user/documents/old_name.txt")
new_path = path.with_name("new_name.txt")
print(new_path)  # /home/user/documents/new_name.txt
```

### `with_stem()` - Replace stem (Python 3.9+)

```python
from pathlib import Path

path = Path("/home/user/documents/report.pdf")
new_path = path.with_stem("summary")
print(new_path)  # /home/user/documents/summary.pdf
```

### `with_suffix()` - Replace extension

```python
from pathlib import Path

path = Path("/home/user/documents/report.txt")
new_path = path.with_suffix(".pdf")
print(new_path)  # /home/user/documents/report.pdf

# Remove extension
new_path = path.with_suffix("")
print(new_path)  # /home/user/documents/report
```

### `joinpath()` or `/` operator - Join paths

```python
from pathlib import Path

base = Path("/home/user")
full = base.joinpath("documents", "file.txt")
print(full)  # /home/user/documents/file.txt

# More Pythonic way using / operator
full = base / "documents" / "file.txt"
print(full)  # /home/user/documents/file.txt
```

## Complete Reference Table

| Method/Property | Returns | Example Input | Example Output |
|----------------|---------|---------------|----------------|
| `name` | Full filename | `/home/file.txt` | `file.txt` |
| `stem` | Filename without last ext | `/home/file.tar.gz` | `file.tar` |
| `suffix` | Last extension | `/home/file.tar.gz` | `.gz` |
| `suffixes` | All extensions (list) | `/home/file.tar.gz` | `['.tar', '.gz']` |
| `parent` | Parent directory | `/home/user/file.txt` | `/home/user` |
| `parents` | All parents (sequence) | `/home/user/file.txt` | `['/home/user', '/home', '/']` |
| `parts` | All components (tuple) | `/home/user/file.txt` | `('/', 'home', 'user', 'file.txt')` |
| `anchor` | Root/drive | `/home/user/file.txt` | `/` |
| `is_absolute()` | Boolean | `../file.txt` | `False` |
| `resolve()` | Absolute path | `../file.txt` | `/home/user/file.txt` |
| `exists()` | Boolean | `/home/file.txt` | `True/False` |
| `is_file()` | Boolean | `/home/file.txt` | `True/False` |
| `is_dir()` | Boolean | `/home` | `True/False` |
| `with_name(x)` | New path | `/home/old.txt` → `"new.txt"` | `/home/new.txt` |
| `with_stem(x)` | New path | `/home/old.txt` → `"new"` | `/home/new.txt` |
| `with_suffix(x)` | New path | `/home/file.txt` → `".pdf"` | `/home/file.pdf` |

## Common Patterns

```python
from pathlib import Path

# Pattern 1: Process all files in directory
directory = Path("/home/user/documents")
for file in directory.iterdir():
    if file.is_file():
        print(f"File: {file.name}, Extension: {file.suffix}")

# Pattern 2: Find all files with specific extension
pdf_files = list(directory.glob("*.pdf"))
all_pdfs = list(directory.rglob("*.pdf"))  # Recursive

# Pattern 3: Create backup filename
def create_backup_path(original_path):
    p = Path(original_path)
    return p.parent / (p.stem + "_backup" + p.suffix)

# Pattern 4: Safe file writing
output_path = Path("/home/user/output.txt")
output_path.parent.mkdir(parents=True, exist_ok=True)
output_path.write_text("Hello, World!")
```

This comprehensive guide covers all the main `Path` methods for working with file paths in Python!