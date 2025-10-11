
### 🧹 Clean cache for Astral’s `uv` (Python package manager)

`uv` caches downloaded packages and build artifacts to speed up installs.  
You can clear them manually or via command line.

#### ✅ Option 1 — Use built-in command

```bash
uv cache clean
```

That removes all cached wheels, source distributions, and metadata.

#### ✅ Option 2 — View cache size first

```bash
uv cache info
```

This shows paths and sizes — useful if you want to inspect before cleaning.

#### ✅ Option 3 — Remove cache manually

Default cache directory (platform dependent):

|OS|Path|
|---|---|
|**Linux/macOS**|`~/.cache/uv`|
|**Windows**|`%LOCALAPPDATA%\uv\Cache`|

You can safely delete those directories manually if needed.

---

### ⚙️ Optional — set custom cache directory

If you want uv to use a different cache path:

```bash
uv config set cache-dir /path/to/custom/cache
```

---

Would you like me to show how to **clean only specific parts** (e.g., old builds or a particular package’s cache)?