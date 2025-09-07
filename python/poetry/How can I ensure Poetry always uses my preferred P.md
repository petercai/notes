

# How can I ensure Poetry always uses my preferred Python version

To ensure Poetry always uses your preferred Python version for a project, you can explicitly specify which Python interpreter Poetry should use when creating or managing the virtual environment. Here are the key methods to achieve this:

## 1. Use `poetry env use` to specify the Python version or interpreter

Poetry provides the `env use` command to tell it exactly which Python executable to use for the current project:

- You can specify a full path to the Python interpreter:

```
poetry env use /full/path/to/python
```

- Or specify a Python version if it is in your system `PATH`:

```
poetry env use python3.9
```

- You can also just specify the minor version:

```
poetry env use 3.9
```


This command creates a new virtual environment using the specified Python version and associates it with your project. If you want to revert to the default system Python behavior, you can run:

```
poetry env use system
```


## 2. Use Python version managers like `pyenv` to set the Python version in your shell

If you use tools like `pyenv` to manage multiple Python versions, set the desired Python version locally or globally before running Poetry commands:

```bash
pyenv install 3.9.8          # Install desired Python version if not installed
pyenv local 3.9.8            # Set Python 3.9.8 for the current project directory
poetry install               # Poetry will detect and use this Python version
```

Poetry respects the Python version active in your shell environment and will use it to create the virtual environment if compatible with your project’s Python version constraints.

## 3. Specify the Python version range in your `pyproject.toml`

Make sure your `pyproject.toml` file specifies the Python version range your project supports, for example:

```toml
[tool.poetry.dependencies]
python = ">=3.8,<3.12"
```

Poetry uses this to verify compatibility with the Python interpreter it selects. If the interpreter does not satisfy this range, Poetry will prompt you to switch or explicitly specify a compatible Python version.

## 4. Managing existing virtual environments

If you want to switch an existing virtual environment to a different Python version, you may need to remove the old environment and create a new one with the preferred Python interpreter:

```bash
poetry env remove <env-name-or-python-version>
poetry env use 3.9
poetry install
```

You can list existing environments with:

```bash
poetry env list
```


## Summary

| Method | Description |
| :-- | :-- |
| `poetry env use <python>` | Explicitly tell Poetry which Python interpreter/version to use for the project environment |
| Use `pyenv` or similar tools | Set the Python version in your shell before running Poetry, so Poetry picks it up automatically |
| Specify Python version in `pyproject.toml` | Define the compatible Python version range for your project |
| Remove and recreate virtualenv | To switch Python versions, remove old env and create a new one with the preferred Python |

By combining these approaches, you can ensure Poetry consistently uses your preferred Python version for your projects.

---

**References:**

- Poetry’s official docs on managing environments and using `env use` command[^1].
- Guidance on using `pyenv` with Poetry to control Python versions[^1][^2].
- Tutorials on specifying Python versions and managing virtual environments with Poetry[^5].

<div style="text-align: center">⁂</div>

[^1]: https://python-poetry.org/docs/managing-environments/

[^2]: https://stackoverflow.com/questions/59810276/why-is-my-poetry-virtualenv-using-the-system-python-instead-of-the-pyenv-python

[^3]: https://github.com/python-poetry/poetry/issues/655

[^4]: https://github.com/python-poetry/poetry/issues/7158

[^5]: https://www.datacamp.com/tutorial/python-poetry

[^6]: https://realpython.com/dependency-management-python-poetry/

[^7]: https://python-poetry.org/docs/

[^8]: https://endjin.com/blog/2023/03/how-to-setup-python-pyenv-poetry-on-windows.html

[^9]: https://community.render.com/t/poetry-doenst-use-chosen-python-version/5796

[^10]: https://forum.level1techs.com/t/python-poetry-review/206892

