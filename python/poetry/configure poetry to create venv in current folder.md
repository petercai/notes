

# configure poetry to create venv in current folder .ven

To configure **Poetry** to always create the virtual environment in the current folder as `.venv`, set the `virtualenvs.in-project` option to `true` using the following command:

```
poetry config virtualenvs.in-project true --local
```

This will cause Poetry to store the virtual environment in a `.venv` subdirectory of the project root each time a new environment is created or `poetry install` is run.[^1][^2][^3]

## Step-by-Step Instructions

- Run this command in your project folder:

```
poetry config virtualenvs.in-project true --local
```

This creates or updates a `poetry.toml` file in the project with the configuration.[^2][^1]
- To verify, create or install your environment:

```
poetry install
```

The `.venv` directory will appear in the current folder when Poetry sets up the environment.[^3][^1]
- Optionally, you can set it globally (applies to all projects):

```
poetry config virtualenvs.in-project true
```

