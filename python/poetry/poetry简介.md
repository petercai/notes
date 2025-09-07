


# poetry
## poetry的工作流程是什么样的

Poetry的工作流程主要涉及以下几个步骤：

### 1. **创建新项目**
   - 使用命令`poetry new 项目名称`创建一个新项目。这个命令会生成一个基本的项目结构，包括`pyproject.toml`文件、`README.md`文件、`src`目录和`tests`目录[1][4]。

### 2. **为现有项目添加Poetry**
   - 如果你有一个现有的Python项目，可以使用`poetry init`命令来为其添加Poetry支持。这个命令会引导你创建一个`pyproject.toml`文件来管理项目依赖[1][10]。

### 3. **依赖管理**
   - **添加依赖**：使用`poetry add 包名`命令来添加新的依赖项。Poetry会自动更新`pyproject.toml`文件并安装依赖[5]。
   - **安装依赖**：运行`poetry install`命令来安装项目所有依赖项。这个命令会根据`pyproject.toml`文件安装依赖，并生成`poetry.lock`文件来锁定依赖版本[1][5]。
   - **更新依赖**：使用`poetry update`命令来更新项目的依赖项[4]。

### 4. **虚拟环境管理**
   - Poetry会自动创建和管理虚拟环境，以确保项目依赖与全局Python环境隔离[9][10]。
   - 可以通过`poetry config`命令配置虚拟环境的设置，例如指定虚拟环境的位置[10]。

### 5. **运行命令**
   - 使用`poetry run 命令`来在项目的虚拟环境中执行命令。例如，`poetry run python 脚本.py`可以在虚拟环境中运行Python脚本[1][3]。

### 6. **打包和发布**
   - Poetry支持将项目打包为Python包，并发布到PyPI[7]。

### 7. **版本控制**
   - `poetry.lock`文件用于锁定依赖版本，确保项目在不同环境下的一致性。建议将这个文件纳入版本控制系统中[1][2]。

总的来说，Poetry的工作流程旨在简化Python项目的依赖管理、虚拟环境管理和打包发布过程。

Citations:
[1] https://yanxi123.com/post/2023/12/python-poetry/
[2] https://bbs.huaweicloud.com/blogs/317789
[3] https://notes.zhengxinonly.com/environment/use-poetry.html
[4] http://lxz9.com/2022/03/20/poetry/
[5] https://www.cnblogs.com/xingxia/p/17955202/python_poetry
[6] https://juejin.cn/post/7169424059024244749
[7] https://muzing.top/posts/3fa905f9/
[8] https://blog.csdn.net/AudiA6LV6/article/details/126973274
[9] https://github.com/AH-Toby/poetry-cn/blob/main/doc/8.%E7%AE%A1%E7%90%86%E7%8E%AF%E5%A2%83.md
[10] https://blog.csdn.net/weixin_51290195/article/details/123193851


## Poetry如何创建和管理虚拟环境


Poetry创建和管理虚拟环境的过程如下：

### 1. **创建虚拟环境**

Poetry可以通过以下方式创建虚拟环境：

- **使用`poetry env use`命令**：指定Python版本来创建虚拟环境。例如：
  ```bash
  poetry env use python
  ```
  这会使用当前命令行下激活的Python解释器创建一个虚拟环境[1][2]。

- **使用`poetry install`命令**：如果配置文件中设置了`virtualenvs.create=true`，执行`poetry install`时会检查是否有虚拟环境，否则会自动创建一个[5]。

### 2. **配置虚拟环境**

- **虚拟环境存放目录**：可以通过`poetry config virtualenvs.path`命令来指定虚拟环境的存放目录。例如：
  ```bash
  poetry config virtualenvs.path D:\WorkSpace\Venv
  ```
  这会将虚拟环境存放在指定的目录下[3][4]。

- **在项目目录下创建虚拟环境**：可以通过设置`virtualenvs.in-project=true`来在项目目录下创建虚拟环境。例如：
  ```bash
  poetry config virtualenvs.in-project true
  ```
  这样可以方便管理和查看虚拟环境[2]。

### 3. **激活虚拟环境**

- **使用`poetry install`命令**：在项目根目录下运行`poetry install`会安装所有依赖项，并激活虚拟环境[3][4]。

- **使用`poetry shell`命令**：可以通过`poetry shell`命令进入一个shell会话，其中已经激活了Poetry的虚拟环境[4]。

### 4. **管理虚拟环境**

- **查看环境信息**：可以使用`poetry env info`命令来查看当前激活的环境信息[4]。

- **删除虚拟环境**：如果需要删除虚拟环境，可以手动删除对应的虚拟环境目录。

总的来说，Poetry提供了一个方便的方式来创建和管理虚拟环境，确保项目依赖与系统环境隔离。

Citations:
[1] https://notes.zhengxinonly.com/environment/use-poetry.html
[2] https://cloud.tencent.com/developer/article/2174333
[3] https://blog.csdn.net/2201_76124692/article/details/139435109
[4] https://www.cnblogs.com/jingzhexingchen/p/18224117
[5] https://blog.csdn.net/qq_40851623/article/details/135769991
[6] https://juejin.cn/post/7169424059024244749
[7] https://python.iswbm.com/c12/c12_05.html
[8] https://www.51cto.com/article/677772.html





# pip
## 如何有效生成当前项目的requirements.txt文件

是的，有专门的Python包可以帮助生成`requirements.txt`文件。其中最常用的包是**pipreqs**。

### pipreqs

**pipreqs**是一个第三方库，可以扫描你的项目目录，自动识别出项目中导入的库，并生成一个包含这些库的`requirements.txt`文件。它的优点在于只会包含项目中实际使用到的依赖包，而不是环境中所有的包。

#### 安装pipreqs

```bash
pip install pipreqs
```

#### 使用pipreqs

在项目根目录下运行以下命令来生成`requirements.txt`文件：

```bash
pipreqs . --encoding=utf8 --force
```

这个命令会覆盖已存在的`requirements.txt`文件，并使用UTF-8编码以避免编码错误。

### 其他工具

除了pipreqs外，还有其他工具可以帮助生成`requirements.txt`文件，例如**pigar**。但是，pipreqs是最常用的和推荐的工具。

### 总结

如果你需要生成一个精确的`requirements.txt`文件，包含项目中实际使用到的依赖包及其版本，使用pipreqs是一个不错的选择。它可以帮助你避免将不必要的包包含在`requirements.txt`文件中，并确保项目在不同环境下的依赖一致性。

Citations:
[1] https://www.cnblogs.com/liuhuilin/p/16129932.html
[2] https://www.cnblogs.com/lvjinfeng/articles/16333180.html
[3] https://blog.csdn.net/watson2017/article/details/139360898
[4] https://www.cnblogs.com/shengshengwang/p/17939493
[5] https://learnku.com/articles/47470
[6] https://visionguide.readthedocs.io/zh-cn/latest/python/tool/requirements/
[7] https://bertyblog.link/archives/python%20%E7%94%9F%E6%88%90%E7%89%B9%E5%AE%9A%E4%BE%9D%E8%B5%96%E9%A1%B9%E7%9A%84%20requirements.txt%20%E6%96%87%E4%BB%B6
[8] https://www.jianshu.com/p/acb3205aea54

## how to uninstall a python package with all its deps

To uninstall a Python package along with all its dependencies, follow these steps:

### **1. Using `pip-autoremove` (Recommended)**
`pip-autoremove` is a tool that removes a package and its unused dependencies.

#### **Install `pip-autoremove`**
```sh
pip install pip-autoremove
```

#### **Uninstall a package and its dependencies**
```sh
pip-autoremove <package_name> -y
```
For example:
```sh
pip-autoremove requests -y
```
This will remove `requests` and any dependencies that are not used by other installed packages.

---

### **2. Using `pip` and `pipdeptree`**
If you want more control, use `pipdeptree` to check dependencies before removing.

#### **Install `pipdeptree`**
```sh
pip install pipdeptree
```

#### **Check dependencies of a package**
```sh
pipdeptree -p <package_name>
```
For example:
```sh
pipdeptree -p openbb
```

#### **Uninstall the package and its dependencies manually**
After listing the dependencies, uninstall them:
```sh
pip uninstall <package_name> <dependency1> <dependency2> ...
```
For example:
```sh
pip uninstall requests urllib3 chardet idna -y
```

---

### **3. Uninstall All Unused Dependencies (`pip-autoremove --list-deps`)**
To find and remove all unused dependencies:
```sh
pipautoremove --list-deps | xargs pip uninstall -y 
```
