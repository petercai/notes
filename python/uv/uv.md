
# update packages


To **check if a package has an update available** using [`uv`](https://github.com/astral-sh/uv), you can use:

---

## ✅ `uv pip list --outdated`

```bash
uv pip list --outdated
```

This command shows all installed packages that have newer versions available, just like the classic `pip list --outdated`.

### 📋 Output looks like:

```
Package    Version   Latest    Type
---------- --------- --------- ------
fastapi    0.95.1    0.110.0   wheel
httpx      0.23.3    0.27.0    wheel
```

---

## ✅ To check a **specific package**:

```bash
uv pip show <package>
```

Example:

```bash
uv pip show numpy
```

This shows metadata, including installed version — but not the latest. To compare, you'll need to run `uv pip list --outdated`.

---

## ✅ To update a package:

```bash
uv pip install -U <package>
```

Example:

```bash
uv pip install -U fastapi
```

---

## 🧠 Bonus: Clean install all updates

To upgrade everything at once:

```bash
uv pip install -U $(uv pip list --outdated --format freeze | cut -d '=' -f1)
```

(Or just update the ones you want.)

---

## 🔚 Summary

|Task|Command|
|---|---|
|Check outdated packages|`uv pip list --outdated`|
|Show installed package info|`uv pip show <pkg>`|
|Upgrade a package|`uv pip install -U <pkg>`|
|Upgrade all packages|use shell command above|

Let me know if you're using `pyproject.toml` — I can also show how to sync upgrades back to it.
# add pytest

To add `pytest` as a **dev dependency** using [`uv`](https://github.com/astral-sh/uv), you can use the `uv add` command with the `--dev` flag like this:

```bash
uv add --dev pytest
```

This command:

- Installs `pytest` into your current Python environment (assuming it's managed by `uv`).
    
- Adds `pytest` under the `[tool.poetry.group.dev.dependencies]` section of your `pyproject.toml`.
    

### Example

If your `pyproject.toml` was originally:

```toml
[tool.poetry.dependencies]
python = "^3.11"
```

After running `uv add --dev pytest`, it becomes:

```toml
[tool.poetry.dependencies]
python = "^3.11"

[tool.poetry.group.dev.dependencies]
pytest = "^8.2.1"  # or whatever latest version at the time
```

Let me know if you’re using a different setup like `hatch`, `pdm`, or just plain `pyproject.toml` without Poetry — `uv` can support those too.

# use in team

Using **uv** to manage Python packages in a team can significantly streamline your workflow. Here are the key steps and features to get you started:

### **1. Installing uv**

- **Standalone Installer**: Download and run the installer from the official uv website.
- **PyPI**: Use the command `pip install uv` to install uv from the Python Package Index.

 ### **2. Creating and Managing Projects**

- **Initialize a Project**: Use `uv init` to create a new Python project. This sets up a `pyproject.toml` file and a virtual environment.
- **Run Scripts**: Use `uv run <script>` to execute your project's entry-point script.

### **3. Dependency Management**

- **Add Dependencies**: Use `uv add <package>` to add a new dependency to your project.
- **Install Dependencies**: Use `uv install` to install all dependencies listed in your `pyproject.toml` file.
- **Upgrade/Remove Dependencies**: Use `uv upgrade <package>` and `uv remove <package>` to manage existing dependencies.

### **4. Virtual Environments**

- **Automatic Management**: uv automatically creates and manages virtual environments for your projects, ensuring clean and isolated dependencies.

### **5. Building and Publishing Packages**

- **Build**: Use `uv build` to create a distribution package for your project.
- **Publish**: Use `uv publish` to upload your package to PyPI or another package repository.

### **6. Team Collaboration**

- **Lock and Sync Environments**: Use `uv lock` to create a lock file that ensures consistent environments across different machines. Team members can use `uv sync` to replicate the environment.

### **7. Additional Features**

- **Fast Dependency Resolution**: uv is designed to be significantly faster than traditional tools like pip, thanks to its Rust implementation [[1]](https://realpython.com/python-uv/) [[2]](https://www.datacamp.com/tutorial/python-uv).
- **Compatibility**: uv works seamlessly with existing tools and workflows, making it easy to integrate into your team's processes [[2]](https://www.datacamp.com/tutorial/python-uv).

By leveraging these features, your team can manage Python projects more efficiently, ensuring smooth development and deployment processes.

Would you like more details on any specific feature or step?

  
References

[1] [Managing Python Projects With uv: An All-in-One Solution](https://realpython.com/python-uv/)

[2] [Python UV: The Ultimate Guide to the Fastest Python Package Manager](https://www.datacamp.com/tutorial/python-uv)

# requirments.txt -> project.toml


To convert a Python project with a `requirements.txt` file to a `uv` project with a `pyproject.toml`, follow these steps:

1. **Initialize the `uv` project**:

```sh
   uv init
```

2. **Delete unnecessary files**: Remove any files created by `uv` that you don't need, such as `hello.py`.
    
3. **Modify `pyproject.toml`**: Update the `pyproject.toml` file with information from your `setup.py` (e.g., description, authors).
    
4. **Add dependencies from `requirements.txt`**:
    

```sh
   uv add -r requirements.txt
```

5. **Remove `setup.py`**: Delete the `setup.py` file as it's no longer needed.
    
6. **Install the project**:
    

```sh
   uv pip install -e .
```

7. **Update Makefile**: Modify your Makefile to use `uv` commands instead of `pip`.

```makefile
   install_library:
       uv sync
       uv pip install -e .

   run_a_script:
       uv run python ./my_lib/my_script.py

   launch_test:
       uv run pytest -n auto --cov-report=xml
```

8. **Update CI configuration**: If you use CI (e.g., GitHub Actions), update your CI scripts to use `uv`.

```yaml
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - name: Install uv
           uses: astral-sh/setup-uv@v3
         - name: Set up Python environment
           run: |
             make install_library
         - name: Test with pytest
           run: |
             make launch_test
```

9. **Deploy on PyPI**: Use `uv` to build and publish your project.

```sh
   uv build
   uv publish
```

For a more detailed guide, you can refer to the [Stack Overflow discussion](https://stackoverflow.com/questions/79154674/how-to-migrate-from-a-simple-python-project-requirements-txt-setup-py-setupto) [[1]](https://stackoverflow.com/questions/79154674/how-to-migrate-from-a-simple-python-project-requirements-txt-setup-py-setupto) or the [uv documentation](https://docs.astral.sh/uv/guides/projects/) [[2]](https://docs.astral.sh/uv/guides/projects/).

Would you like more information on any of these steps?

  
References

[1] [How to migrate from a simple Python project : requirements.txt setup.py ...](https://stackoverflow.com/questions/79154674/how-to-migrate-from-a-simple-python-project-requirements-txt-setup-py-setupto)

[2] [Working on projects | uv - Astral](https://docs.astral.sh/uv/guides/projects/)