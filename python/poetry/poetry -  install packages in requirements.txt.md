

# how to install all package in requirements.txt in poetry

To install all packages listed in a `requirements.txt` file using Poetry, there is no direct built-in Poetry command to import and install from `requirements.txt` automatically. However, you can achieve this in a few ways:

## Recommended Approaches

### 1. Manually Add Dependencies from requirements.txt to Poetry

Poetry manages dependencies via the `pyproject.toml` file, so the standard way is to add dependencies manually using:

```bash
poetry add package_name
```

for each package listed in `requirements.txt`. This ensures Poetry properly resolves and locks dependencies. But this is tedious for many packages[^1][^3].

### 2. Use a Helper Tool: `poetry-add-requirements.txt`

There is a third-party tool called `poetry-add-requirements.txt` designed specifically to add dependencies from a `requirements.txt` file into a Poetry project automatically:

- Install it via pipx (recommended):

```bash
pipx install poetry-add-requirements.txt
```

- Then run it to add dependencies from your `requirements.txt`:

```bash
poetry-add-requirements.txt requirements.txt
```

- It supports options like `--dev` to add dev dependencies, ignoring version requirements, and dry-run mode.

This tool reads the `requirements.txt` file, normalizes the dependencies, and runs `poetry add` commands for you, automating the migration process[^5].

### 3. Alternative: Convert requirements.txt to pyproject.toml Dependencies Manually

Some users prefer to manually convert the dependencies from `requirements.txt` into the `[tool.poetry.dependencies]` section of `pyproject.toml`, then run:

```bash
poetry install
```

to install all dependencies. This method is more controlled but manual[^1].

## Summary of Commands

| Task | Command/example |
| :-- | :-- |
| Add a single package to Poetry | `poetry add package_name` |
| Install all dependencies (from lock file) | `poetry install` |
| Add all packages from requirements.txt (using helper tool) | `poetry-add-requirements.txt requirements.txt` |

## Additional Notes

- Poetry does not natively parse `requirements.txt` files because it uses `pyproject.toml` and `poetry.lock` for dependency management and locking[^1][^2][^3].
- Using a tool like `poetry-add-requirements.txt` is the easiest way to automate importing dependencies from `requirements.txt`.
- After adding dependencies, run `poetry install` to install them in the virtual environment.

---

**In brief:** To install all packages from a `requirements.txt` in Poetry, either manually add them with `poetry add` or use the third-party tool `poetry-add-requirements.txt` to automate adding all dependencies, then run `poetry install` to install them[^1][^3][^5].

<div style="text-align: center">⁂</div>

[^1]: https://stackoverflow.com/questions/62764148/how-to-import-an-existing-requirements-txt-into-a-poetry-project

[^2]: https://github.com/python-poetry/poetry/issues/8837

[^3]: https://python-poetry.org/docs/cli/

[^4]: https://timonweb.com/python/how-to-import-requirementstxt-into-poetry/

[^5]: https://pypi.org/project/poetry-add-requirements.txt/

[^6]: https://www.kdnuggets.com/2023/07/forget-pip-conda-requirementstxt-poetry-instead-thank-later.html

[^7]: https://www.linkedin.com/pulse/time-ditch-requirementstxt-now-start-using-poetry-python-kulibaba-flo3c

[^8]: https://www.reddit.com/r/learnpython/comments/xjyz13/why_use_poetry_instead_of_requirementstxt_files/

[^9]: https://github.com/orgs/python-poetry/discussions/7199

