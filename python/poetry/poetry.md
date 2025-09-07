## install poetry in windows

To install **Poetry** for Python on Windows, follow these steps:

### 1. Install via the Official Installer (Recommended)
Run the following command in **PowerShell** (open as Administrator for a system-wide install):

```powershell
(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | py -
```

### 2. Verify Installation
After installation, restart your terminal and check if Poetry is installed:

```powershell
poetry --version
```

### 3. Add Poetry to PATH (if needed)
If the `poetry` command is not found, add it to your system's `PATH`. The default location is:

```
%APPDATA%\Python\Scripts
```

You can manually add this to your `PATH` environment variable.

This will download and install the latest version of Poetry,
a dependency and package manager for Python.

It will add the `poetry` command to Poetry's bin directory, located at:

C:\Users\caipe\AppData\Roaming\Python\Scripts

You can uninstall at any time by executing this script with the --uninstall option,
and these changes will be reverted.

Installing Poetry (2.1.1)
Installing Poetry (2.1.1): Creating environment
Installing Poetry (2.1.1): Installing Poetry
Installing Poetry (2.1.1): Creating script
Installing Poetry (2.1.1): Done

Poetry (2.1.1) is installed now. Great!

To get started you need Poetry's bin directory (C:\Users\caipe\AppData\Roaming\Python\Scripts) in your `PATH`
environment variable.

You can choose and execute one of the following commands in PowerShell:

A. Append the bin directory to your user environment variable `PATH`:

```
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", "User") + ";C:\Users\caipe\AppData\Roaming\Python\Scripts", "User")
```

B. Try to append the bin directory to PATH every when you run PowerShell (>=6 recommended):

```
echo 'if (-not (Get-Command poetry -ErrorAction Ignore)) { $env:Path += ";C:\Users\caipe\AppData\Roaming\Python\Scripts" }' | Out-File -Append $PROFILE
```

Alternatively, you can call Poetry explicitly with `C:\Users\caipe\AppData\Roaming\Python\Scripts\poetry`.

You can test that everything is set up by executing:

`poetry --version`

### 4. Enable Shell Completion (Optional)
For **PowerShell**, enable tab completion with:

```powershell
poetry completions powershell | Out-String | Invoke-Expression
```




## poetry structure
Understanding the Challenge

The core challenge is that a single pyproject.toml file only directly manages dependencies for the project where it's located. Sub-packages often have their own pyproject.toml files, and you want to install all of them in editable mode simultaneously, treating the root as a collective.

Solution: Using path Dependencies and develop

Poetry handles this situation with a combination of path dependencies and potentially the develop extra. Here's the process:

#### Root pyproject.toml:

In the my-project/pyproject.toml, you'll need to list each sub-package as a path dependency. This tells Poetry where to find these packages locally.
You can also add the develop extra which will install all the other sub-packages in editable mode, this will avoid adding each sub-package to the project dependencies.

``` 
my-project/pyproject.toml
[tool.poetry]
name = "my-project"
version = "0.1.0"
description = ""
authors = ["Your Name <you@example.com>"]

[tool.poetry.dependencies]
python = "^3.9"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4.4"

[tool.poetry.extras]
develop = ["package-a", "package-b"]

[tool.poetry.group.develop]
optional = true

[tool.poetry.dependencies]
package-a = { path = "src/package_a", develop = true }
package-b = { path = "src/package_b", develop = true }

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

## insall all packages in develop mode
poetry install --all-extras
poetry install --extras="develop"
Explanation:
poetry install: This installs the root project's dependencies.
--extras="develop": this ensures that the packages added in the develop extra are also installed.
Poetry will recognize the path dependencies and install package_a and package_b in editable mode, linking them to the source code in the src/ directory.
All other packages will be installed as well, in normal mode.


## poetry env

To check **Poetry's virtual environments**, use the following commands:

### 1. **List All Poetry Virtual Environments**
Run:
```sh
poetry env list
```
This will show all virtual environments managed by Poetry.

### 2. **Show Full Path of the Active Virtual Environment**
```sh
poetry env info
```
This will display details like:
- Path to the virtual environment
- Python version used
- Whether the environment is currently activated

### 3. **Get Only the Virtual Environment Path**
```sh
poetry env info --path
```
This is useful if you need the exact location for manual activation.

### 4. **Manually Activate a Poetry Virtual Environment**
If you want to manually activate a specific environment, use:
```sh
source $(poetry env info --path)/bin/activate  # macOS/Linux
```
```powershell
. (poetry env info --path)\Scripts\activate  # Windows (PowerShell)
```

### 5. **Remove a Specific Virtual Environment**
If needed, remove an environment using:
```sh
poetry env remove python
```
or specify the full path:
```sh
poetry env remove <env-path>
```

## install all Poetry-managed packages in a pip-based environment

To install all **Poetry-managed** packages in a **pip-based environment** (e.g., virtualenv, Conda, system Python), follow these steps:

---

### **1. Export Poetry Dependencies to `requirements.txt`**
Run:
```sh
poetry export -f requirements.txt --output requirements.txt --without-hashes
```
This creates a `requirements.txt` file with all dependencies.

---

### **2. Create and Activate a pip Virtual Environment**
If you don’t have a virtual environment yet, create one:

```sh
python -m venv myenv
source myenv/bin/activate  # macOS/Linux
myenv\Scripts\activate  # Windows
```

For **Conda** users:
```sh
conda create -n myenv python=3.9  # Replace with your version
conda activate myenv
```

---

### **3. Install Dependencies Using pip**
Now, install the dependencies using `pip`:

```sh
pip install -r requirements.txt
```

---

### **4. Verify Installation**
Check installed packages:
```sh
pip list
```

Now, your **pip environment** has all the **Poetry-managed** packages! 🚀