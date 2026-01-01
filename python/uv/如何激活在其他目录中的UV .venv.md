<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# 如何激活在其他目录中的UV .venv?

用完整路径直接 `source`（Linux/Mac）或调用（Windows）uv 创建的 `.venv` 中的激活脚本，就能从任意目录激活。uv 本身没有专用的 `activate` 子命令。[^1][^2][^3]

## Linux/Mac 激活

假设 `.venv` 在 `/path/to/project/.venv`，从其他目录运行：

```bash
source /path/to/project/.venv/bin/activate
```

激活后提示符会显示 `(.venv)`，Python 路径自动切换到虚拟环境。[^3][^1]

## Windows 激活

```cmd
# CMD
call C:\path\to\project\.venv\Scripts\activate.bat

# PowerShell  
C:\path\to\project\.venv\Scripts\Activate.ps1
```


## 常用技巧

- **用 `uv run` 代替激活**（推荐，无需激活环境）：

```bash
cd /other/dir
uv run --from /path/to/project python your_script.py
```

临时使用指定项目的 uv 环境运行命令。[^4]
- **设置别名**（永久方便）：

```bash
alias actproj='source /path/to/project/.venv/bin/activate'
actproj  # 随时激活
```

- **uv 会自动发现项目**：如果在项目子目录，`uv run python script.py` 会自动找上级 `.venv`。[^4]

激活后用 `deactivate` 退出，路径自动恢复。[^1]
<span style="display:none">[^10][^5][^6][^7][^8][^9]</span>

<div align="center">⁂</div>

[^1]: https://discuss.python.org/t/from-what-directory-can-you-activate-a-virtual-environment/24698

[^2]: https://github.com/astral-sh/uv/issues/8106

[^3]: https://everythingpython.substack.com/p/virtual-environments-using-uv

[^4]: https://github.com/astral-sh/uv/issues/10543

[^5]: https://github.com/astral-sh/uv/issues/1910

[^6]: https://stackoverflow.com/questions/65390129/venv-activate-doesnt-not-change-my-python-path

[^7]: https://stackoverflow.com/questions/32407365/can-i-move-a-virtualenv

[^8]: https://github.com/astral-sh/uv/issues/1495

[^9]: https://github.com/astral-sh/uv/issues/1598

[^10]: https://www.reddit.com/r/learnpython/comments/1ida805/for_anyone_else_struggling_to_set_up_a_virtual/

