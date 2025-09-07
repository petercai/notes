
# how too add or install a package using poetry

To add or install a package using Poetry, you use the `poetry add` command. This command both adds the package as a dependency in your `pyproject.toml` and installs it into your Poetry-managed virtual environment.

### How to Add or Install a Package with Poetry

```bash
poetry add <package-name>
```

- This updates your `pyproject.toml` file to include the package under `[tool.poetry.dependencies]`.
- It also updates the `poetry.lock` file to lock the exact package version.
- The package is installed immediately in your virtual environment.


### Example

To add the package `requests`:

```bash
poetry add requests
```


### Additional Notes

- To add a development dependency (e.g., testing tools), use the `--dev` flag:

```bash
poetry add --dev pytest
```

- After adding packages, you can run:

```bash
poetry install
```

to install all dependencies defined in your project.
- If you want to install all dependencies from the `pyproject.toml` (for example, after cloning a project), use:

```bash
poetry install
```


This command reads the lock file (or resolves dependencies if the lock file is missing) and installs the exact versions.

### Summary

| Action | Command |
| :-- | :-- |
| Add and install package | `poetry add <package>` |
| Add dev dependency | `poetry add --dev <package>` |
| Install all dependencies | `poetry install` |

This is the recommended and official way to manage packages with Poetry[^1][^2][^4].

<div style="text-align: center">⁂</div>

[^1]: https://python-poetry.org/docs/cli/

[^2]: https://python-poetry.org/docs/basic-usage/

[^3]: https://python-poetry.org/docs/

[^4]: https://stackoverflow.com/questions/66422871/how-do-i-install-python-packages-with-poetry

[^5]: https://python-poetry.org/docs/1.8/cli/

[^6]: https://www.youtube.com/watch?v=U_cPHzfdqPU

[^7]: https://realpython.com/dependency-management-python-poetry/

[^8]: https://github.com/orgs/python-poetry/discussions/1135

