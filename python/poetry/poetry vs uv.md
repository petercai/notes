

# compare them in same table side by side

Here is a **side-by-side comparison table** of common **Poetry** and **uv** commands, highlighting their equivalents and key differences based on the latest information from 2025:


| Task / Feature                 | Poetry Command                                         | uv Command                                                                             | Notes / Differences                                                                                     |
| :----------------------------- | :----------------------------------------------------- | :------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------ |
| **Initialize Project**         | `poetry init`                                          | `uv init`                                                                              | Both create a basic `pyproject.toml`. Poetry’s is more interactive by default.                          |
| **Create New Project**         | `poetry new <name>`                                    | (No direct equivalent; use `uv init` + manual setup)                                   | Poetry scaffolds project structure; uv focuses on environment/dependency management.                    |
| **Add Dependency**             | `poetry add <package>`                                 | `uv add <package>`                                                                     | Both add dependency to `pyproject.toml` and install it.                                                 |
| **Add Dev Dependency**         | `poetry add <package> --group dev`                     | `uv add <package> --group dev`                                                         | Both add to developer dependency group.                                                                 |
| **Remove Dependency**          | `poetry remove <package>`                              | `uv remove <package>`                                                                  | Both uninstall and remove from project files.                                                           |
| **Install Dependencies**       | `poetry install`                                       | `uv sync --frozen`                                                                     | `uv sync --frozen` installs exactly from lock file; Poetry’s `install` also sets up venv automatically. |
| **Install Without Dev**        | `poetry install --without dev` (or `--no-dev` pre-1.5) | `uv sync --no-dev`                                                                     | Both skip development dependencies.                                                                     |
| **Update Lockfile**            | `poetry lock`                                          | `uv lock`                                                                              | Both regenerate lock files based on `pyproject.toml` without installing.                                |
| **Update All Dependencies**    | `poetry update`                                        | `uv sync -U`                                                                           | Updates lock file and installs latest allowed versions; `-U` important for git dependencies.            |
| **Update Specific Dependency** | `poetry update <package>`                              | `uv --upgrade-package <package> -o requirements.lock && uv pip sync requirements.lock` | More manual with uv; Poetry automates update and install in one command.                                |
| **Run Command in Environment** | `poetry run <command>`                                 | `uv run <command>`                                                                     | Both run commands inside the project’s virtual environment.                                             |
| **Show Virtualenv Path**       | `poetry env info --path`                               | `uv venv` (when creating/activating)                                                   | Poetry manages multiple named envs; uv typically uses `.venv` folder and manual management.             |
| **Remove Virtualenv**          | `poetry env remove <python_version>` or path           | Manually delete `.venv` folder                                                         | uv does not manage named envs; deletion is manual.                                                      |
| **Show Dependency Tree**       | `poetry show --tree`                                   | `uv pip tree`                                                                          | Both show package dependency trees.                                                                     |
| **Show Installed Packages**    | `poetry show`                                          | `uv pip list`                                                                          | Lists installed packages; `uv pip freeze` similar to `pip freeze`.                                      |
| **Build Package**              | `poetry build`                                         | `python -m build`                                                                      | uv does not have a dedicated build command; relies on standard Python build tools.                      |
| **Publish Package**            | `poetry publish`                                       | `twine upload dist/*`                                                                  | uv relies on external tools like `twine` for publishing.                                                |
| **Check Project Metadata**     | `poetry check`                                         | (No direct equivalent)                                                                 | uv lacks built-in metadata validation; external linters needed.                                         |
| **Activate Virtualenv Shell**  | `poetry shell`                                         | `source .venv/bin/activate` (Linux/macOS) or `.venv\Scripts\activate` (Windows)        | uv does not provide a dedicated shell command; uses standard venv activation scripts.                   |
| **Manage Python Version**      | No direct management; use `pyenv` externally           | `uv python install <version>`, `uv python pin <version>`                               | uv replaces pyenv by automatically installing and pinning Python versions via `.python-version` file.   |
| **Tool Management**            | Limited (via `poetry run`)                             | `uv tool install/upgrade/uninstall/list/run`                                           | uv provides explicit tool environment management similar to `pipx`.                                     |
| **Cache Management**           | Not explicit                                           | `uv cache clean`, `uv cache dir`                                                       | uv exposes cache management commands.                                                                   |
| **Self Update**                | Not available                                          | `uv self update`                                                                       | uv can update itself via CLI.                                                                           |

### Key Differences Summary

- **Python Version Management:** uv uniquely replaces `pyenv` by automatically installing and using Python versions specified in `.python-version`, enabling fast, reproducible Python environments. Poetry requires external tools like `pyenv` for this.
- **Speed and Resource Usage:** uv is implemented in Rust, offering faster performance and lower memory usage compared to Poetry’s Python implementation.
- **Tool Environment:** uv supports dedicated management of CLI tools (linters, formatters) isolated per project, whereas Poetry runs such tools via `poetry run`.
- **Virtual Environment Handling:** Poetry automatically manages virtual environments and provides commands like `poetry shell`; uv uses standard venv activation and manual management.
- **Build and Publish:** Poetry integrates build and publish commands; uv relies on external standard Python tools like `build` and `twine`.
- **CLI Complexity:** Poetry’s CLI is simpler and more opinionated; uv’s CLI is more complex with many subcommands but offers more granular control.

This table and summary are based on recent expert comparisons and official documentation as of mid-2025[^1][^2][^5].

If you want, I can also provide a quick command mapping script or migration tips from Poetry to uv.

<div style="text-align: center">⁂</div>

[^1]: https://www.loopwerk.io/articles/2024/python-poetry-vs-uv/

[^2]: https://www.datacamp.com/tutorial/python-uv

[^3]: https://stackoverflow.com/questions/79118841/how-can-i-migrate-from-poetry-to-uv-package-manager

[^4]: https://docs.astral.sh/uv/reference/cli/

[^5]: https://ricketyrick.github.io/uv_cheat_sheet/poetry_vs_uv.html

[^6]: https://github.com/astral-sh/uv/issues/7704

[^7]: https://pydevtools.com/handbook/how-to/how-to-migrate-from-poetry-to-uv/

[^8]: https://www.reddit.com/r/Python/comments/1jd7nhh/uv_starting_to_overtake_poetry_in_package_download/

