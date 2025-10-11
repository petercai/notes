## 🧠 Goal

You have a large virtual environment (say `venv/`) already built for one project, and you want **another project** to reuse that same environment — ideally **with `uv`** managing installs and syncs efficiently.

---

## 🧩 Option 1 — Reuse via `UV_PROJECT_ENVIRONMENT` (recommended)

`uv` lets you **point a project to an existing virtual environment** using an environment variable.

### Example:

```bash
set UV_PROJECT_ENVIRONMENT=C:\path\to\shared\.venv        # Windows PowerShell
# or
export UV_PROJECT_ENVIRONMENT=/path/to/shared/.venv       # Linux/macOS
```

Now when you run in another project folder:

```bash
uv pip install -r requirements.txt
```

it will **reuse that venv** instead of creating a new one.

✅ Works with:

- `uv run`
    
- `uv pip install`
    
- `uv sync`
    
- `uv pip freeze`
    

This approach keeps all your packages cached and avoids duplication.

---

## 🧩 Option 2 — Use `uv venv` to explicitly point to existing path

You can tell `uv` to use or create a venv at a given location.

```bash
uv venv --python 3.12 /path/to/shared/.venv
```

Then in your new project:

```bash
uv run --python /path/to/shared/.venv python script.py
```

or just:

```bash
uv run -p /path/to/shared/.venv some_command
```

That uses the same environment for all runs.

---

## 🧩 Option 3 — Symlink `.venv` between projects

If you like having `.venv` inside each project folder for tooling (e.g. VSCode auto-detect):

```bash
cd new_project
rmdir .venv
mklink /D .venv "C:\path\to\shared\.venv"   # Windows
# or
ln -s /path/to/shared/.venv .venv           # Linux/macOS
```

Now all Python tools think you have a `.venv`, but it’s shared.

⚠️ Caution:  
Changes to dependencies in one project affect all linked projects.

---

## 🧩 Option 4 — Use `uv lock` + `uv sync` for fast rebuilds

If you don’t want a truly shared venv but still want speed:

```bash
uv lock
uv sync
```

This will reuse cached wheels from the first build (thanks to uv’s cache), so even if each project has its own `.venv`, installation is **instant** for already-built dependencies.

---

## 🔍 Summary

| Goal                           | Best Option                         |
| ------------------------------ | ----------------------------------- |
| Reuse one venv across projects | `UV_PROJECT_ENVIRONMENT` or symlink |
| Rebuild quickly but isolated   | `uv lock` + `uv sync`               |
| Custom path for global venv    | `uv venv /path/to/env`              |

---



---

# 🧱 Goal
here’s a **best-practice guide** for reusing a large, shared virtual environment across multiple projects with **`uv`**, while keeping projects isolated and reproducible.

✅ Save disk space & time by reusing one large `venv`  
✅ Keep each project reproducible (has its own `uv.lock`)  
✅ Work smoothly with IDEs (e.g. VS Code, PyCharm)  
✅ Still benefit from `uv`’s fast dependency management & caching

---

# ⚙️ Setup Overview

You’ll create:

```
C:\dev\
 ├── shared-env\.venv        ← your big reusable environment
 ├── projectA\
 │    ├── pyproject.toml
 │    ├── uv.lock
 │    └── .venv → symlink or env var pointing to shared-env\.venv
 └── projectB\
      ├── pyproject.toml
      ├── uv.lock
      └── .venv → symlink or env var pointing to shared-env\.venv
```

---

## 🧩 Step 1: Create a shared environment

```bash
cd C:\dev
uv venv shared-env\.venv --python 3.12
```

Install all your heavy global packages (like `torch`, `numpy`, etc.) once:

```bash
uv pip install torch numpy pandas scipy coqui-tts
```

---

## 🧩 Step 2: Reuse it in each project

Option A (recommended): **via environment variable**

In PowerShell:

```bash
setx UV_PROJECT_ENVIRONMENT "C:\dev\shared-env\.venv"
```

In bash/zsh:

```bash
export UV_PROJECT_ENVIRONMENT=/home/you/dev/shared-env/.venv
```

Now every time you run in `projectA` or `projectB`:

```bash
uv sync
uv run python main.py
```

→ They all use that **shared** venv transparently.

---

Option B: **Symlink `.venv`**

In each project folder:

```bash
rmdir .venv
mklink /D .venv "C:\dev\shared-env\.venv"
# or: ln -s /home/you/dev/shared-env/.venv .venv
```

Your IDE will automatically pick it up as the local environment.

---

## 🧩 Step 3: Keep each project isolated (logically)

Each project has its own dependency list (and `uv.lock` file):

```bash
cd C:\dev\projectA
uv init  # creates pyproject.toml
uv add fastapi uvicorn
uv lock
```

and in another project:

```bash
cd C:\dev\projectB
uv add gradio
uv lock
```

Even though they use the same physical environment, `uv lock` ensures you can reproduce each setup later — e.g. in CI/CD or production.

---

## 🧩 Step 4: Rebuild or update selectively

If you ever want to rebuild or isolate a project later:

```bash
uv sync --frozen --python 3.12 --no-cache
```

Or to rebuild fresh in a new venv:

```bash
uv venv .venv --python 3.12
uv sync
```

You’ll still get lightning-fast installs because `uv` reuses cached wheels.

---

## 🧩 Step 5: (Optional) Cache tuning for speed

You can view cache info:

```bash
uv cache info
```

Clean old builds occasionally:

```bash
uv cache clean
```

And if you want to move the cache (e.g. onto a larger SSD):

```bash
uv config set cache-dir "D:\uv-cache"
```

---

# ✅ Summary Table

|Goal|Command / Config|Notes|
|---|---|---|
|Create shared venv|`uv venv shared-env/.venv`|One-time setup|
|Use globally|`UV_PROJECT_ENVIRONMENT=/path/to/.venv`|Cleanest setup|
|IDE friendly|`.venv` symlink to shared venv|Works for VSCode, PyCharm|
|Reproducibility|`uv lock` per project|Each project isolated logically|
|Fast rebuilds|`uv sync`|Uses cache|
|Maintenance|`uv cache clean`|Free disk space|
