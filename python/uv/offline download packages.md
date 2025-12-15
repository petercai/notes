# pyproject.toml -> requirments.txt

To generate a `requirements.txt` file from a `pyproject.toml` using `uv`, you can run:

```Shell
uv pip compile pyproject.toml --output-file requirements.txt
```

### 🔍 What this does:

- Reads dependencies from `pyproject.toml`
- Resolves and pins all versions
- Outputs a `requirements.txt` file with exact versions

### ✅ Requirements:

- Your `pyproject.toml` must define dependencies under `[project]` or `[tool.poetry.dependencies]` (depending on the build system).
- `uv` must be installed and up to date.

Would you like help checking your `pyproject.toml` structure or converting it from Poetry or another format?
# download all deps
You're absolutely right — as of **`uv` version 0.7.20**, the `pip download` subcommand is **not supported**. That’s why you're seeing the error:

```
error: unrecognized subcommand 'download'
```

### ✅ Alternative Approach with `uv 0.7.20`

To **vendor dependencies** (i.e. download all `.whl` files into a folder), `uv` uses the `sync` command with the `--download-only` flag:

```Shell
uv sync --download-only --platform manylinux_x86_64 --dest vendor
```

### 🔍 Explanation:

- `--download-only`: Downloads dependencies without installing them.
- `--platform manylinux_x86_64`: Targets the desired platform.
- `--dest vendor`: Specifies the folder to store the downloaded `.whl` files.
- It uses your `pyproject.toml` and `uv.lock` (if present) to resolve dependencies.

If you don’t have a `uv.lock` file yet, you can generate one with:

```Shell
uv pip compile pyproject.toml
```

Would you like help generating the lock file or customizing the vendoring process for a specific environment?# install deps from local folder

To have `uv` install from `.whl` files **first**, and fall back to `.tar.gz` **only if the `.whl` fails**, you can use a shell command that tries installing `.whl` files first, and then installs any remaining `.tar.gz` files.

Here’s a **bash script** approach that works on Linux/macOS (or Windows with WSL or Git Bash):

```Shell
#!/bin/bash

# First, try installing all .whl files
for whl in path/to/folder/*.whl; do
    uv pip install "$whl" || echo "Failed to install $whl"
done

# Then, install .tar.gz files (assuming the .whl failed or wasn't available)
for tar in path/to/folder/*.tar.gz; do
    uv pip install "$tar"
done
```

### Explanation:

- `uv pip install "$whl"` tries to install each `.whl` file.
- If it fails, it moves on (thanks to `|| echo ...`).
- Then it installs all `.tar.gz` files, which are source distributions.

### On Windows (PowerShell):

```Powershell
# Install .whl files
Get-ChildItem "path\to\folder\*.whl" | ForEach-Object {
    uv pip install $_.FullName
}

# Install .tar.gz files
Get-ChildItem "path\to\folder\*.tar.gz" | ForEach-Object {
    uv pip install $_.FullName
}
```

Would you like me to generate a ready-to-run script file for your system?