

# create poetry new env

To create a new virtual environment for a project using **Poetry**, you typically follow these steps:

## Creating a New Poetry Project (which creates a new env)

If you want to create a **new project** along with its own virtual environment, use:

```bash
poetry new my-project
cd my-project
```

Then, when you run any Poetry command that requires the environment (like `poetry install`), Poetry will automatically create a new virtual environment for the project.

---

## Creating a New Virtual Environment for an Existing Project

If you already have a project and want to create a new Poetry-managed virtual environment, do:

```bash
poetry install
```

This will:

- Create a new virtual environment if one does not exist.
- Install dependencies from `pyproject.toml`.

---

## Manually Creating or Switching to a New Environment

If you want to explicitly create or switch to a virtual environment with a specific Python version, use:

```bash
poetry env use python3.9
```

Replace `python3.9` with the desired Python interpreter version or full path to the Python executable. This will create a new virtual environment with that Python version if it doesn't exist.

---

## Check Virtual Environment Info

To see the path of the current Poetry virtual environment:

```bash
poetry env info --path
```


---

## Summary

| Command                    | Purpose                                            |
| :------------------------- | :------------------------------------------------- |
| `poetry new my-project`    | Create a new project with default setup            |
| `poetry install`           | Create virtual environment \& install dependencies |
| `poetry env use python3.x` | Create/select environment with specific Python     |
| `poetry env info --path`   | Show current virtual environment path              |


---

If you want to **force** Poetry to recreate the environment (e.g., after deleting the `.venv`), simply run `poetry install` again and it will create a fresh environment.

---

If you want me to provide a step-by-step example or help with a specific Python version, just ask!

