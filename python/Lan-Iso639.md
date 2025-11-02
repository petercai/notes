# 1 python-iso639


---

### 🧩 What is **`python-iso639`**?

`iso639` (or sometimes `iso-639` / `py-iso639`) is a **Python library** for working with **ISO 639 language codes** — the international standard for identifying languages.

It helps you **convert between**:

- 2-letter codes (ISO 639-1, e.g. `"zh"`, `"en"`)
    
- 3-letter codes (ISO 639-2/3, e.g. `"zho"` or `"chi"`, `"eng"`)
    
- Full language names (e.g. `"Chinese"`, `"English"`)
    

---

### 🔧 Installation

```bash
pip install iso639
```

---

### 📘 Basic Usage

```python
from iso639 import Lang

# Lookup by ISO code
lang = Lang('zh')
print(lang.name)       # -> 'Chinese'
print(lang.pt1)        # -> 'zh'
print(lang.pt2b)       # -> 'chi'
print(lang.pt2t)       # -> 'zho'
print(lang.pt3)        # -> 'zho'
```

---

### 🇨🇳 Getting Correct Chinese Info

The ISO standard defines **“Chinese”** as a single language with variants, even though in real life you might want **Simplified Chinese**, **Traditional Chinese**, **Mandarin**, or **Cantonese**.

`iso639` only covers **language codes**, not **regional variants**.  
So:

|Variant|ISO 639 code|BCP-47 / locale code|
|---|---|---|
|Generic Chinese|`zh`|`zh`|
|Simplified Chinese (Mainland)|`zh`|`zh-Hans`|
|Traditional Chinese (Taiwan/HK)|`zh`|`zh-Hant`|
|Mandarin Chinese|`cmn`|`cmn`|
|Cantonese|`yue`|`yue`|

---

### Example: Handle Both ISO & Locale

If you want more **accurate info** (e.g. distinguish zh-Hans vs zh-Hant), combine `iso639` with `babel`:

```python
from babel import Locale
from iso639 import Lang

# ISO 639 info
print(Lang('zh').name)  # 'Chinese'

# Locale info
loc = Locale.parse('zh_Hant')
print(loc.get_display_name('en'))  # 'Chinese (Traditional Han)'

loc2 = Locale.parse('zh_Hans')
print(loc2.get_display_name('en'))  # 'Chinese (Simplified Han)'
```

---

### ✅ Summary

|Task|Library|Example|
|---|---|---|
|Get language name from ISO code|`iso639`|`Lang('zh').name → 'Chinese'`|
|Get code variants (2/3 letters)|`iso639`|`Lang('zh').pt3 → 'zho'`|
|Get regional/variant info|`babel`|`Locale.parse('zh_Hant')`|

---

# 2. Data Structure


## 🧩 1. Data Structure of `python-iso639`

When you use:

```python
from iso639 import Lang
lang = Lang("zh")
```

`Lang("zh")` returns an **object** that holds standardized ISO 639 info.  
It’s basically a **named tuple / dataclass-like object** with these fields:

|Attribute|Meaning|Example (`zh`)|
|---|---|---|
|`name`|English name of the language|`'Chinese'`|
|`pt1`|ISO 639-1 (2-letter code)|`'zh'`|
|`pt2b`|ISO 639-2/B (bibliographic code)|`'chi'`|
|`pt2t`|ISO 639-2/T (terminologic code)|`'zho'`|
|`pt3`|ISO 639-3 (3-letter code)|`'zho'`|
|`scope`|Scope (I = individual, M = macrolanguage)|`'M'`|
|`type`|Type (L = living, A = ancient, etc.)|`'L'`|
|`inverted_name`|“Lastname, Firstname” style|`'Chinese'`|

So internally it’s like:

```python
Lang(
    name='Chinese',
    pt1='zh',
    pt2b='chi',
    pt2t='zho',
    pt3='zho',
    scope='M',
    type='L',
    inverted_name='Chinese'
)
```

---

## ⚙️ 2. API Overview

Here are the main entry points (from [`iso639`](https://pypi.org/project/iso639/) library):

### ✅ Create a Lang object

```python
from iso639 import Lang

Lang('zh')        # lookup by ISO code
Lang('chinese')   # lookup by name (case-insensitive)
```

### ✅ Access attributes

```python
lang = Lang('zh')
lang.name     # 'Chinese'
lang.pt1      # 'zh'
lang.pt2t     # 'zho'
lang.pt3      # 'zho'
```

### ✅ Reverse lookup (get code from name)

```python
Lang('Chinese').pt1   # 'zh'
Lang('English').pt1   # 'en'
```

### ✅ Error handling

```python
from iso639 import LangNotFoundError

try:
    Lang('xx')
except LangNotFoundError:
    print("Unknown language")
```

### ✅ Search or list all languages

The library includes a data table internally (`iso639.languages`) if you want to inspect all.

```python
from iso639 import languages
print(len(languages))   # total number of entries
print(languages[0])     # show one entry
```

Each entry is just a Python dict like:

```python
{
  'name': 'Chinese',
  'inverted_name': 'Chinese',
  'part1': 'zh',
  'part2b': 'chi',
  'part2t': 'zho',
  'part3': 'zho',
  'scope': 'M',
  'type': 'L'
}
```

---

## 📘 Quick Summary

|Goal|Method|Example|
|---|---|---|
|Get language info from code|`Lang('zh')`|`.name → 'Chinese'`|
|Get ISO 639 codes|`.pt1`, `.pt2b`, `.pt2t`, `.pt3`|`'zh'`, `'chi'`, `'zho'`|
|Lookup by name|`Lang('Chinese')`|`.pt1 → 'zh'`|
|Handle errors|`LangNotFoundError`|—|
|Inspect all data|`iso639.languages`|list of dicts|

---

If you want, I can show you how to **filter for all Chinese-related languages** (e.g. Mandarin, Cantonese, Hakka) using this data — would you like that?