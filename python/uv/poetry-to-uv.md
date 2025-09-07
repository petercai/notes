
# Covert Poetry to UV

To convert a `pyproject.toml` file created by **Poetry** (which uses `[tool.poetry]`) to one compatible with **PEP 621** used by `uv`, you need to:

---

### ✅ 1. **Manually Convert `[tool.poetry]` to `[project]`**

`uv` expects `pyproject.toml` to follow [PEP 621](https://peps.python.org/pep-0621/), which defines the `[project]` table.

Here's an example conversion:

#### Original Poetry `pyproject.toml`:

```toml
[tool.poetry]
name = "myproject"
version = "0.1.0"
description = "My awesome project"
authors = ["Peter Cai <peter@example.com>"]
license = "MIT"

[tool.poetry.dependencies]
python = "^3.10"
requests = "^2.28"
numpy = "^1.24"

[tool.poetry.dev-dependencies]
pytest = "^7.2"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

---

#### ✅ Converted to PEP 621 (`uv` compatible):

```toml
[project]
name = "myproject"
version = "0.1.0"
description = "My awesome project"
authors = [
  { name = "Peter Cai", email = "peter@example.com" }
]
license = { text = "MIT" }
dependencies = [
  "requests>=2.28,<3.0",
  "numpy>=1.24,<2.0"
]

[project.optional-dependencies]
dev = [
  "pytest>=7.2,<8.0"
]

[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

> Note: `uv` does **not** install `dev-dependencies` unless you explicitly use optional dependencies (e.g. `pip install .[dev]`)

---

### ✅ 2. **Install with `uv`**

Once the file is converted:

```bash
uv pip install .
uv pip install .[dev]  # if you want dev dependencies too
```

---

### 🧩 Optional: Automate the Conversion

There is **no official tool** to automate the conversion from `[tool.poetry]` to `[project]`, but a few options:

- Use [deptry](https://github.com/fpgmaas/deptry) or [poetry-export-plugin](https://github.com/Cado-Labs/poetry-plugin-export) to generate `requirements.txt`
    
- Use tools like `tomlkit` in a Python script to convert sections programmatically.
    

---

Would you like a script that converts the file for you?