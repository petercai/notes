
Good question. If by “uv” you mean the **Astral `uv`** Python tool (for dependency / venv management), you can use its built-in commands to show some form of a dependency tree. Here’s how, plus some caveats.

---

## How to Show Dependency Tree & Versions in `uv`

1. **Use `uv tree`**  
    The primary command is:
    
    ```bash
    uv tree
    ```
    
    This shows the dependency tree of your project. ([Astral Docs](https://docs.astral.sh/uv/reference/cli/?utm_source=chatgpt.com "Commands | uv"))
    
2. **Show All Platforms / “Universal” Tree**  
    If you want a platform-independent tree (i.e. show dependencies resolved for all supported Python versions / platforms), use:
    
    ```bash
    uv tree --universal
    ```
    
    This shows “resolved package versions for all Python versions and platforms” rather than just the ones in your current venv. ([Astral Docs](https://docs.astral.sh/uv/reference/cli/?utm_source=chatgpt.com "Commands | uv"))
    
3. **Limit Tree Depth (Optional)**  
    You can limit how deep the tree goes (i.e. how many levels of transitive dependencies):
    
    ```bash
    uv tree --depth N
    ```
    
    Replace `N` with the max depth you want. ([Hexmos](https://hexmos.com/freedevtools/tldr/uv/uv-tree/?utm_source=chatgpt.com "Display UV Dependencies - Analyze Project Tree | Online Free DevTools by Hexmos - uv Commands - TLDR | Online Free DevTools by Hexmos"))
    
4. **Show Outdated Versions**  
    If you want to see which packages are outdated (i.e., have newer versions available), you can do:
    
    ```bash
    uv tree --outdated
    ```
    
    This shows the latest available version for each package in the tree. ([Gist](https://gist.github.com/caiosalgado/cfc7a0c0e870ddeea328d32493be53c7?utm_source=chatgpt.com "UV Documentation – A Comprehensive Guide for UV, the Fast Python Package Installer and Virtual Environment Manager This file, uv-llms-full.txt, contains detailed documentation and usage examples for UV. UV is a Rust-powered tool designed as a faster alternative to pip/virtualenv. It features rapid dependency resolution, integrated virtual environment management, a pip-compatible interface, lockfile support for reproducibility, and Python version management, all backed by smart caching and high performance. Keywords: llms.txt uv, UV documentation, Rust Python package installer, virtual environment manager, dependency resolution. · GitHub"))
    
5. **Exclude / Include Dev Dependencies**
    
    - To omit development dependencies: `--no-dev` ([Astral Docs](https://docs.astral.sh/uv/reference/cli/?utm_source=chatgpt.com "Commands | uv"))
        
    - To only show dev dependencies: `--only-dev` ([Gist](https://gist.github.com/caiosalgado/cfc7a0c0e870ddeea328d32493be53c7?utm_source=chatgpt.com "UV Documentation – A Comprehensive Guide for UV, the Fast Python Package Installer and Virtual Environment Manager This file, uv-llms-full.txt, contains detailed documentation and usage examples for UV. UV is a Rust-powered tool designed as a faster alternative to pip/virtualenv. It features rapid dependency resolution, integrated virtual environment management, a pip-compatible interface, lockfile support for reproducibility, and Python version management, all backed by smart caching and high performance. Keywords: llms.txt uv, UV documentation, Rust Python package installer, virtual environment manager, dependency resolution. · GitHub"))
        
    - You can also filter by a specific “group” if you’re using uv’s dependency groups. ([Gist](https://gist.github.com/caiosalgado/cfc7a0c0e870ddeea328d32493be53c7?utm_source=chatgpt.com "UV Documentation – A Comprehensive Guide for UV, the Fast Python Package Installer and Virtual Environment Manager This file, uv-llms-full.txt, contains detailed documentation and usage examples for UV. UV is a Rust-powered tool designed as a faster alternative to pip/virtualenv. It features rapid dependency resolution, integrated virtual environment management, a pip-compatible interface, lockfile support for reproducibility, and Python version management, all backed by smart caching and high performance. Keywords: llms.txt uv, UV documentation, Rust Python package installer, virtual environment manager, dependency resolution. · GitHub"))
        
6. **Show Sizes**  
    If you want to see compressed wheel sizes for packages in the tree:
    
    ```bash
    uv tree --show-sizes
    ```
    
    ([Astral Docs](https://docs.astral.sh/uv/reference/cli/?utm_source=chatgpt.com "Commands | uv"))
    
7. **Invert the Tree**  
    To show _reverse dependencies_ (i.e., display which package depends on a given package):
    
    ```bash
    uv tree --invert
    ```
    
    ([Hexmos](https://hexmos.com/freedevtools/tldr/uv/uv-tree/?utm_source=chatgpt.com "Display UV Dependencies - Analyze Project Tree | Online Free DevTools by Hexmos - uv Commands - TLDR | Online Free DevTools by Hexmos"))
    

---

## Limitations / Caveats

- **Version Specifiers Not Always Shown**  
    There’s a known issue: `uv pip tree` does _not_ always display version specifiers for dependencies. ([GitHub](https://github.com/astral-sh/uv/issues/5217?utm_source=chatgpt.com "`uv pip tree` should show version specifiers for dependencies. · Issue #5217 · astral-sh/uv · GitHub"))
    
- **No Built-in `pipdeptree`-style Full Specifier Display**  
    There was a feature request for `uv` to support a `pipdeptree`-like output, but as of now, it’s not fully implemented. ([GitHub](https://github.com/astral-sh/uv/issues/3272?utm_source=chatgpt.com "Add a `pipdeptree` like output · Issue #3272 · astral-sh/uv · GitHub"))
    
- **Dependency Resolution Behavior**  
    The versions shown in the tree depend on how `uv` resolved dependencies (e.g., `--resolution` strategy). ([Astral Docs](https://docs.astral.sh/uv/reference/cli/?utm_source=chatgpt.com "Commands | uv"))
    

---

## Example

```bash
# Show a full tree of dependencies (including transitive)
uv tree

# Show platform-independent versions
uv tree --universal

# Show only up to 2 levels deep
uv tree --depth 2

# Show which deps are outdated
uv tree --outdated

# Show dev-only dependencies
uv tree --only-dev
```

---

If you like, I can suggest other tools (outside `uv`) to visualize dependency trees (with more detail) — do you want me to?