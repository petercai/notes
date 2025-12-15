Here is the **official compatibility matrix** for **Numba ↔ llvmlite ↔ Python** so you can pick the **correct llvmlite version** for your environment.

This table comes from Numba’s release docs.

---

# ✅ **Numba ↔ llvmlite Compatibility Matrix (Most Important)**

|**Numba version**|**llvmlite version**|**Supported Python versions**|
|---|---|---|
|**0.60.x**|**0.43.x**|3.9–3.12|
|**0.59.x**|**0.42.x**|3.9–3.12|
|**0.58.x**|**0.41.x**|3.8–3.12|
|**0.57.x**|**0.40.x**|3.8–3.11|
|**0.56.x**|**0.39.x**|3.8–3.10|
|**0.55.x**|**0.38.x**|3.7–3.10|
|**0.54.x**|**0.37.x**|3.7–3.9|
|**0.53.x**|**0.36.x**|3.7–3.9|

---

# 🔍 **Most common stable pairings**

If you want a commonly working setup:

### ✔ Python 3.12 → best match

```text
Numba 0.60.x + llvmlite 0.43.x
```

### ✔ Python 3.11 → best match

```text
Numba 0.59.x + llvmlite 0.42.x
```

### ✔ Python 3.10 → best match

```text
Numba 0.58.x + llvmlite 0.41.x
```

---

# 🎯 **Given your request: llvmlite==0.41.1**

This version **requires**:

```
Numba 0.58.x
Python 3.8–3.12
```

So the correct matching install is:

```bash
uv add numba==0.58.1 llvmlite==0.41.1
```

or:

```bash
uv pip install numba==0.58.1 llvmlite==0.41.1
```

---

# 🧪 **Check your environment**

Run this:

```bash
uv run python - << 'EOF'
import sys
print("Python:", sys.version)
try:
    import numba, llvmlite
    print("Numba:", numba.__version__)
    print("llvmlite:", llvmlite.__version__)
except Exception as e:
    print("Error:", e)
EOF
```

---

# ❗ If you tell me:

✔ your Python version  
✔ your Numba version (if already installed)

I will give you the **exact llvmlite version** you should install for perfect compatibility.