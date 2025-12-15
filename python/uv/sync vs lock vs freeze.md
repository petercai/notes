
# sync vs freeze
 **`uv sync`** and **`uv pip freeze`** are related but serve different purposes—they’re not strictly a “pair” like `git pull` and `git push`, but they complement each other in a workflow.

---

### ✅ **What `uv sync` does**

- **Purpose:** Synchronizes your environment with the dependencies declared in your `pyproject.toml` (and optionally `uv.lock`).
- **Behavior:**
    - Installs all packages listed in `[project.dependencies]` and `[tool.uv]` sections.
    - Resolves versions according to constraints and markers.
    - Updates the lock file (`uv.lock`) if needed.
    - Ensures your virtual environment matches the declared project state.
- **Analogy:** Like `npm install` or `poetry install`—it enforces the spec from your project file.

---

### ✅ **What `uv pip freeze` does**

- **Purpose:** Shows the **actual installed packages and versions** in your current environment.
- **Behavior:**
    - Lists everything in `site-packages`, including transitive dependencies.
    - Useful for debugging, generating a `requirements.txt`, or verifying what’s really installed.
- **Analogy:** Like `pip freeze` or `npm list`—it reflects the real state, not the declared spec.

---

### 🔍 Are they a pair?

- **Not exactly a pair**, but they’re complementary:
    - `uv sync` → **brings your environment in line with your declared dependencies**.
    - `uv pip freeze` → **shows what’s actually installed after syncing**.
- Common workflow:
    1. Edit `pyproject.toml` (e.g., bump `torch` to 2.8.0).
    2. Run `uv sync` to apply changes.
    3. Run `uv pip freeze` to confirm the environment matches expectations.

---

### ✅ When to use each

- Use **`uv sync`** whenever you change your dependency spec or want to recreate the environment.
- Use **`uv pip freeze`** for:
    - Debugging version conflicts.
    - Checking transitive dependencies.
    - Exporting a snapshot for reproducibility.

---

👉 Do you want me to give you a **recommended workflow for upgrading Torch with uv**, including how to lock and verify dependencies? Or a **quick comparison table of uv vs pip commands**?


# lock

Yes! **`uv` does support a lock file**, similar to `poetry.lock` or `pip-tools requirements.lock`.

### ✅ What is `uv.lock`?

- It’s a **deterministic snapshot** of all resolved dependencies (including transitive ones) for your project.
- Ensures **reproducible installs** across machines and CI/CD.
- Generated automatically when you run `uv sync` (or explicitly with `uv lock`).

---

### ✅ How it works

- Your **`pyproject.toml`** declares _constraints_ (e.g., `torch==2.8.0`).
- `uv lock` resolves those constraints into exact versions and writes them to `uv.lock`.
- `uv sync` then installs **exact versions from `uv.lock`** into your environment.

---

### ✅ Commands

- **Generate or update lock file:** uv lock
- **Sync environment from lock file:** uv sync
- **Install from lock file only (ignore pyproject):**
    
    uv sync --locked
    
- **Check differences between lock and environment:** uv pip freeze

---

### ✅ Workflow example for upgrading Torch

1. Edit `pyproject.toml` → bump `torch` to `2.8.0`.
2. Run:
    
    uv lock
    
    uv sync
    
3. Verify: uv pip freeze | grep torch

---

**`uv lock` does NOT read your `.venv` and sync from it**. Instead:

- **`uv lock` resolves dependencies from your `pyproject.toml` (and any constraints) and writes them into `uv.lock`.**
- It does **not** inspect what’s currently installed in your virtual environment.
- So if your `.venv` has newer versions installed via `uv pip install -u` or manual upgrades, those changes will **not automatically propagate to `uv.lock`** unless you also update `pyproject.toml` first.

---

### ✅ How the workflow works

- **`uv sync`**: Installs packages based on `uv.lock` (or `pyproject.toml` if no lock exists).
- **`uv lock`**: Creates or updates `uv.lock` based on the declared spec in `pyproject.toml`.
- **`.venv` state**: Is only influenced by `uv sync` or `uv pip install`; it does not feed back into the lock file.

---

### ✅ If you want `uv.lock` to reflect your current `.venv`

You have two options:

#### **Option 1 — Update `pyproject.toml` first**

- Manually bump versions in `[project.dependencies]` to match what’s in `.venv`.
- Then run: uv lock This regenerates `uv.lock` with those versions.

#### **Option 2 — Export from `.venv` and update spec**

- Use: uv pip freeze > requirements.txt
- Compare with `pyproject.toml` and update accordingly.
- Then run `uv lock`.

_(There’s no built-in “reverse sync” from `.venv` → `pyproject.toml` yet.)_

---

### ✅ Best practice

- Treat `pyproject.toml` as the **source of truth** for declared dependencies.
- Use `uv.lock` for **exact reproducibility**.
- If you upgrade via `uv pip install -u`, **immediately update `pyproject.toml`** and regenerate the lock file.

---

👉 Do you want me to give you a **script that automatically parses `uv pip freeze` and updates your `pyproject.toml` dependencies**? Or a **recommended workflow diagram for uv in multi-project editable installs**?

# sync pyproject.toml

Right now, your **`.venv`** reflects the actual installed versions (after `uv pip install -e` or `-u`), but your **`pyproject.toml`** still has the old pins. To sync them back, you need to **export the environment state and update your project spec**.

---

### ✅ Why this happens

- `uv pip install -e` or `uv pip install -u` modifies the environment but **does not automatically rewrite `pyproject.toml`**.
- `pyproject.toml` remains as originally declared, so it’s now out of sync with your actual environment.

---

### ✅ How to update `pyproject.toml` to match `.venv`

#### **Option 1 — Manual update using `uv pip freeze`**

1. Get the current environment state: uv pip freeze This lists all installed packages and versions.
2. Compare with your `[project.dependencies]` in `pyproject.toml`.
3. Update the versions manually (or copy relevant lines).

---

#### **Option 2 — Use `uv export` (recommended)**

`uv` can export the current environment to a requirements file:

uv export > requirements.txt

Then you can:

- Use this as a reference to update `pyproject.toml`.
- Or keep `requirements.txt` for reproducibility alongside `pyproject.toml`.

---

#### **Option 3 — Automate sync with `uv lock`**

If you want reproducibility without editing `pyproject.toml` every time:

1. Run: uv lock This creates `uv.lock` with exact versions.
2. Commit `uv.lock` to your repo.
3. Future installs use: This ensures the environment matches the lock file, even if `pyproject.toml` has looser constraints.

---

### ✅ Best practice workflow for multi-project editable installs

- Keep **`pyproject.toml`** with **broad constraints** (e.g., `torch>=2.8,<3`).
- Use **`uv.lock`** for exact reproducibility.
- Use **`uv export`** if you need a `requirements.txt` for external tools.

---

👉 Do you want me to give you a **script that parses `uv pip freeze` and updates your `[project.dependencies]` in `pyproject.toml` automatically**? Or a **recommended template for pyproject + lock strategy for multi-source projects**?


# comparison

Great question! By default, **`uv export`** and **`uv pip freeze`** are meant for different purposes, so their output formats can differ (hashes, markers, editable paths, etc.). You can make them **comparable** by normalizing both outputs to the same simple `name==version` format and sorting them.

Below are a few practical approaches you can run in **zsh/VS Code terminal on macOS**.

---

## Option A — Quick one-liner comparison (normalize + sort)

# Normalize and sort uv pip freeze

uv pip freeze \

  | sed -E 's/;._$//' \_

  _| sed -E 's/\s+//g' \_

  _| sort \_

  _> /tmp/pip-freeze.txt_

_  

# Normalize and sort uv export (strip hashes, markers, extras)

uv export \

  | grep -v -E '^(#|--hash)' \

_

  _| sed -E 's/;._$//' \

  | sed -E 's/\s+//g' \

  | sort \

  > /tmp/uv-export.txt

  

# Diff

diff -u /tmp/pip-freeze.txt /tmp/uv-export.txt || true

**What this does**

- Removes environment markers (`; python_version ...`).
- Removes hashes and comments from `uv export`.
- Sorts both outputs alphabetically.
- Shows a unified diff for easy comparison.

---

## Option B — Prefer clean `uv export` flags (if available)

If your `uv` build supports these flags, they simplify the export:

# Try producing clean requirements-like output

uv export --no-hashes --format requirements \

  | sed -E 's/;._$//' \_

  _| sed -E 's/\s+//g' \_

  _| sort \_

  _> /tmp/uv-export.txt_

_  

uv pip freeze \

_

  _| sed -E 's/;._$//' \

  | sed -E 's/\s+//g' \

  | sort \

  > /tmp/pip-freeze.txt

  

diff -u /tmp/pip-freeze.txt /tmp/uv-export.txt || true

> If `uv export` doesn’t support `--no-hashes` or `--format requirements` on your version, just fall back to **Option A** (strip with `grep` and `sed`).

---

## Option C — Handle editable installs uniformly

Editable installs show up differently:

- `uv pip freeze` often shows `package @ file:///path`
- `uv export` may show `-e file:///path` or `-e .`

Normalize editable lines across both:

# Normalize pip freeze (turn @ file paths into -e lines)

uv pip freeze \

  | sed -E 's/@ file:\/\//-e file:\/\//g' \

  | sed -E 's/;._$//' \_

  _| sed -E 's/\s+//g' \_

  _| sort > /tmp/pip-freeze.txt_

_  

# Normalize uv export (remove hashes/comments)

uv export \

  | grep -v -E '^(#|--hash)' \

_

  _| sed -E 's/;._$//' \

  | sed -E 's/\s+//g' \

  | sort > /tmp/uv-export.txt

  

diff -u /tmp/pip-freeze.txt /tmp/uv-export.txt || true

---

## Notes and pitfalls

- **Markers & platforms**: `uv export` may include platform markers; `pip freeze` shows what’s installed for your current platform. Stripping markers (`; ...`) makes them comparable.
- **Extras**: If you have extras like `package[feature]==version`, both commands may include them. The `sed` above preserves extras but removes markers/hashes.
- **Local paths**: For local editable packages, equivalence is fuzzy—compare presence rather than exact path syntaxes.
- **Order**: Sorting ensures a stable comparison, since `pip freeze` order may differ from `uv export`.

---

## Handy zsh function (drop-in)

Add this to your shell or `.zshrc` for a reusable comparison:

uvcmp() {

  local pf="/tmp/pip-freeze.txt"

  local ue="/tmp/uv-export.txt"

  

  uv pip freeze \

    | sed -E 's/@ file:\/\//-e file:\/\//g' \

    | sed -E 's/;._$//' \_

    _| sed -E 's/\s+//g' \_

    _| sort > "$pf"_

  _uv export \

    | grep -v -E '^(#|--hash)' \_

    _| sed -E 's/;._$//' \

    | sed -E 's/\s+//g' \

    | sort > "$ue"

  

  echo "Comparing:"

  echo "  $pf"

  echo "  $ue"

  diff -u "$pf" "$ue" || true

}

Run it with:

uvcmp

---

If you’d like, share your current outputs of:

uv pip freeze | head -n 20

uv export | head -n 20

and I’ll tailor the normalization to your exact `uv` version and project setup.