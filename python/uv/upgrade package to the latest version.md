here’s how to **upgrade the `ebooklib` package to the latest version using `uv`** (the ultra-fast Python package manager from Astral).

---

### 🚀 Step-by-step (command line)

#### 1. Check your current version

```bash
uv pip show ebooklib
```

If it’s not installed yet, that command will tell you.

---

#### 2. Upgrade to the latest version from PyPI

```bash
uv pip install -U ebooklib
```

or equivalently:

```bash
uv pip install --upgrade ebooklib
```

✅ That will:

- Fetch the latest release of `ebooklib` from PyPI
    
- Replace your currently installed version (if older)
    
- Work inside your current virtual environment (or global environment if none)
    

---

#### 3. Verify upgrade

```bash
uv pip show ebooklib
```

Check the `Version:` field — it should now show the newest one.

---

### ⚙️ Optional tips

- To upgrade **all** outdated packages:
    
    ```bash
    uv pip list --outdated
    uv pip install -U $(uv pip list --outdated --format=freeze | cut -d= -f1)
    ```
    
- If you’re inside a project using a `pyproject.toml` (like Poetry or PDM style), you can also do:
    
    ```bash
    uv add ebooklib --upgrade
    ```
    
    That will update both the installed version **and** the dependency version in `pyproject.toml`.
    

---

Would you like me to show how to **pin the version** (e.g. `ebooklib>=0.18,<1.0`) or **revert** to an older one later?