Here are the **best tools outside `uv`** for showing Python dependency trees with _full version info_, _specifier constraints_, and _visualization_.  
I’ll split them into CLI tools vs. GUI/graph tools.

---

# ✅ **1. pipdeptree (the gold standard) — CLI**

`pipdeptree` is the most complete dependency-tree inspector and is far more detailed than `uv tree`.

### Install:

```bash
pip install pipdeptree
```

### Show full tree with versions:

```bash
pipdeptree
```

### Show _dependency specifiers_ (not only installed versions):

```bash
pipdeptree --freeze
```

Example output:

```
requests==2.31.0
  - certifi [required: >=2017.4.17, installed: 2024.7.4]
  - urllib3 [required: >=1.21.1,<3, installed: 2.2.0]
```

### Show conflicts:

```bash
pipdeptree --warn fail
```

### Display reverse dependencies:

```bash
pipdeptree -r
```

### Export as JSON:

```bash
pipdeptree --json > tree.json
```

---

# ✅ **2. pip check (conflicts only) — CLI**

Good for quick health checks:

```bash
pip check
```

This doesn’t show a tree, but it finds broken dependencies.

---

# ✅ **3. pip-licenses — show versions + license info**

If you also want to know which licenses each dependency uses:

```bash
pip install pip-licenses
pip-licenses
```

---

# 🧩 **4. GraphViz visual dependency graph (very useful)**

You can turn a dependency tree into a **real graph diagram**:

### Using pipdeptree → graphviz:

```bash
pipdeptree --graph-output dot > graph.dot
dot -Tpng graph.dot -o deps.png
```

Output:

- `deps.png` shows a full dependency graph with arrows between packages.
    

---

# 🌐 **5. pipgrip — resolver + tree**

`pipgrip` uses pip’s resolver and prints dependency tree _before installing_.

### Install:

```bash
pip install pipgrip
```

### Show tree:

```bash
pipgrip fastapi
```

This is better when you want to inspect a package **before** installing anything.

---

# 🐍 **6. pip-audit — security + tree**

```bash
pip install pip-audit
pip-audit
```

It doesn’t show a tree, but it analyzes all dependencies and versions for vulnerabilities.

---

# 🧭 Summary Table

|Tool|Tree|Shows Versions|Shows Specifiers|Graph Output|Notes|
|---|---|---|---|---|---|
|**uv tree**|✔|✔|❌ (limited)|❌|Fast but incomplete|
|**pipdeptree**|✔✔|✔✔|✔✔|✔ (dot/GraphViz)|Best overall|
|**pipgrip**|✔|✔|✔|❌|Great for pre-install inspection|
|**pip check**|❌|✔|❌|❌|Only reports issues|
|**pip-licenses**|❌|✔|❌|❌|License audit|
|**pip-audit**|❌|✔|❌|❌|Security audit|

---

# 🎉 Recommended Setup (Works Great With uv)

Even if you use `uv` to manage environments:

```bash
uv pip install pipdeptree graphviz
pipdeptree --graph-output png > deps.png
```

---

If you want, I can also show you:

- ✔ how to integrate dependency tree visualization into VSCode
    
- ✔ generate interactive HTML dependency graphs
    
- ✔ compare `uv tree` with `pipdeptree` line-by-line
    

Just tell me!