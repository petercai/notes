

# how to use poetry. lock file?

The `poetry.lock` file is a key part of Poetry’s dependency management system that ensures **reproducible and deterministic installs** of your project’s dependencies.

## What is `poetry.lock`?

- It records the **exact versions** of all your direct and transitive dependencies, including hashes for package integrity verification.
- This guarantees that everyone working on the project, your CI system, and production environments use the **same dependency versions**, preventing “works on my machine” problems.
- It is **automatically generated and updated** by Poetry when you add or update dependencies.


## How to use the `poetry.lock` file?

### 1. Generating or updating the lock file

- When you run:

```bash
poetry lock
```

Poetry resolves dependencies specified in `pyproject.toml` and writes their exact versions into `poetry.lock`.
- Note: `poetry lock` **does not install** packages; it only updates the lock file.
- To update dependencies and the lock file simultaneously, use:

```bash
poetry update
```

which updates packages to the latest versions allowed by your version constraints and updates `poetry.lock`.


### 2. Installing dependencies using the lock file

- When you run:

```bash
poetry install
```

Poetry first checks if `poetry.lock` exists. If it does, Poetry installs the **exact versions** specified in the lock file, ensuring consistency.
- If `poetry.lock` is missing, Poetry resolves dependencies from `pyproject.toml` and creates a new lock file.


### 3. Version control and collaboration

- **Commit `poetry.lock` to your version control system (e.g., Git).**
- This ensures that all collaborators and deployment environments install the same dependency versions.
- It also improves install speed since Poetry can skip dependency resolution if the lock file is present.


### 4. When to update the lock file

- Update the lock file when you want to upgrade dependencies to newer versions within your specified constraints:

```bash
poetry update
```

- Avoid manually editing the `poetry.lock` file, as it contains hashes and metadata that must remain consistent.


## Summary

| Command | Effect |
| :-- | :-- |
| `poetry lock` | Resolves dependencies and writes `poetry.lock` (no install) |
| `poetry install` | Installs dependencies exactly as in `poetry.lock` (or creates it if missing) |
| `poetry update` | Updates dependencies to latest allowed versions and updates `poetry.lock` |
| Commit `poetry.lock` | Ensures reproducible installs across environments |

## Why use `poetry.lock`?

- Guarantees **deterministic builds** by locking exact dependency versions.
- Prevents subtle bugs caused by dependency version drift.
- Speeds up installs by skipping repeated dependency resolution.
- Facilitates collaboration and deployment consistency.

In brief, **the `poetry.lock` file locks your project’s dependencies to exact versions and should be committed to version control**. Use `poetry install` to install dependencies exactly as locked, and `poetry update` to refresh the lock file with newer versions when desired. Avoid manual edits to the lock file to maintain integrity.

<div style="text-align: center">⁂</div>

[^1]: https://stackoverflow.com/questions/65883059/what-is-the-difference-between-poetry-lock-and-poetry-update-lock

[^2]: https://realpython.com/dependency-management-python-poetry/

[^3]: https://python-poetry.org/docs/basic-usage/

[^4]: https://python-poetry.org/docs/cli/

[^5]: https://app.studyraid.com/en/read/15003/518579/locking-dependencies-with-poetrylock

[^6]: https://fig.io/manual/poetry/lock

[^7]: https://github.com/orgs/python-poetry/discussions/7847

[^8]: https://safjan.com/python-project-with-Poetry-add-poetry-lock-to-the-git-repo-or-ignore-it/

