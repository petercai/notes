

# how to add sub-folder project(s) in poetry

To add sub-folder projects or multiple Python packages within a Poetry-managed project, you have a few options depending on your project structure and goals. Poetry itself does not natively support multiple independent subprojects with separate `pyproject.toml` files inside one repository, but you can organize multiple packages or subfolders in a single Poetry project or use plugins/monorepo strategies.

## 1. Single Poetry Project with Multiple Packages in Subfolders

You can organize your project to include multiple Python packages as subfolders inside your main project, and configure Poetry to include them all in the build.

### Example structure:

```
my-project/
├── pyproject.toml
├── src/
│   ├── package_one/
│   │   └── __init__.py
│   └── package_two/
│       └── __init__.py
└── tests/
```

In your `pyproject.toml`, specify the packages to include by editing the `[tool.poetry]` section:

```toml
[tool.poetry]
name = "my-project"
version = "0.1.0"
packages = [
    { include = "package_one", from = "src" },
    { include = "package_two", from = "src" }
]
```

This tells Poetry to treat both `package_one` and `package_two` as packages inside the `src` folder and include them when building or installing the project.

### Notes:

- Each subfolder must be a valid Python package (contain `__init__.py`).
- This is suitable when the sub-packages are part of the same distribution.


## 2. Managing Multiple Independent Projects (Monorepo)

If you want truly independent subprojects (each with their own `pyproject.toml`), Poetry does not currently support managing multiple subprojects under a single root project natively.

### Workarounds:

- **Separate Poetry projects:** Each subfolder is a separate Poetry project with its own `pyproject.toml`. You can manage them independently and use tools like `tox` or custom scripts to coordinate builds/tests.
- **Poetry Multiproject Plugin:** Use community plugins like [poetry-multiproject-plugin](https://pypi.org/project/poetry-multiproject-plugin/) which add commands to help build and check multiple projects in a monorepo setup, allowing relative package includes and shared code between projects.


## 3. Namespace Packages and Shared Code

For namespace packages spanning multiple subfolders, you can configure Poetry similarly by specifying packages in `pyproject.toml` and ensuring proper `__init__.py` or `py.typed` files for typing support.

## Summary

| Method | Description | Use Case |
| :-- | :-- | :-- |
| Single Poetry project with multiple packages | Use `packages` in `pyproject.toml` to include subfolders as packages | Multiple packages in one distribution |
| Separate Poetry projects per subfolder | Each subfolder has its own `pyproject.toml`; manage independently | Truly independent projects in one repo |
| Poetry Multiproject Plugin | Plugin to support building multiple projects with shared code in a monorepo | Monorepos with shared code and builds |


---

### References

- Specifying multiple packages in `pyproject.toml` with `packages` key: [GitHub issue \#2270](https://github.com/python-poetry/poetry/issues/2270)[^2]
- Poetry official `poetry new` and package layout options: [Poetry docs](https://python-poetry.org/docs/cli/#new)[^3]
- Poetry Multiproject Plugin for monorepos: [PyPI poetry-multiproject-plugin](https://pypi.org/project/poetry-multiproject-plugin/)[^6]

---

If you want to add sub-folder projects inside a single Poetry project, the recommended way is to list them explicitly in the `packages` section of your `pyproject.toml` and ensure each subfolder is a proper Python package. For managing multiple independent projects in subfolders, consider separate Poetry projects or a monorepo plugin.

<div style="text-align: center">⁂</div>

[^1]: https://stackoverflow.com/questions/78263343/poetry-package-a-subfolder-as-a-package

[^2]: https://github.com/python-poetry/poetry/issues/2270

[^3]: https://python-poetry.org/docs/cli/

[^4]: https://github.com/python-poetry/poetry/issues/4412

[^5]: https://realpython.com/dependency-management-python-poetry/

[^6]: https://pypi.org/project/poetry-multiproject-plugin/

[^7]: https://www.youtube.com/watch?v=KVgG5QRypZI

[^8]: https://jude.hashnode.dev/simplify-your-python-projects-with-poetry

[^9]: https://python-poetry.org/docs/dependency-specification/

[^10]: https://python-poetry.org/docs/managing-dependencies/

[^11]: https://youtrack.jetbrains.com/issue/PY-57652/Support-poetry-based-environments-when-the-pyproject.toml-is-not-in-the-root-folder-e.g.-multiple-projects-in-single-repo

[^12]: https://stackoverflow.com/questions/73848440/poetry-install-multiple-projects-in-the-same-environment

[^13]: https://python-poetry.org/docs/basic-usage/

[^14]: https://github.com/python-poetry/poetry/issues/161

