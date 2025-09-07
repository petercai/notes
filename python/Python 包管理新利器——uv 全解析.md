# Python 包管理新利器——uv 全解析
一、引言
----

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/76c034c8df13480e8feb096a879a56df~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgcm9nZXJvZ2Vycw==:q75.awebp?rk3s=f64ab15b&x-expires=1742518471&x-signature=ZCrYabzPtm8evlNsp9szkuEQO%2FM%3D)

在 Python 开发的世界中，包管理工具起着至关重要的作用。而 uv 作为一款新兴的 Python 包管理工具，带来了诸多重要性和创新之处。

uv 由 Astral 公司开发，完全用 Rust 编写，具有惊人的速度。它最初作为 pip 工作流的替代品推出，如今已扩展成为一个端到端的解决方案，能够管理 Python 项目、命令行工具、单文件脚本甚至 Python 本身。可以说，uv 就像是 Python 界的 Cargo，为开发者提供了一个快速、可靠、易用的统一接口。

在重要性方面，高效的包管理工具能够极大地提高开发效率。uv 通过其极快的跨平台依赖解析器，能够快速安装和解析 Python 包，节省宝贵的开发时间。对于大型项目而言，快速的依赖安装和解析可以加速项目的启动和迭代过程。对于小型脚本，uv 同样能够提供便捷的包管理功能，让开发者无需在繁琐的依赖管理上花费过多精力。

uv 作为 Python 包管理工具，在重要性和创新方面都表现出色，为 Python 开发者带来了全新的开发体验。

二、uv 是什么？
---------

uv 是由 Astral 公司开发，用 Rust 编写的快速 Python 包管理器。它最初作为 pip 工作流的替代品推出，如今已扩展成为一个端到端的解决方案，能够管理 Python 项目、命令行工具、单文件脚本甚至 Python 本身。uv 就像是 Python 界的 Cargo，为开发者提供了一个快速、可靠、易用的统一接口。

uv 具有多项突出的特性。在端到端项目管理方面，通过 `uv run`、`uv lock` 和 `uv sync` 等命令，可以基于标准元数据生成跨平台的锁文件，并从中安装依赖，性能比 Poetry、PDM 和 Rye 等工具更高。在工具管理方面，`uv tool install` 和 `uv tool run`（别名 uvx）可以在隔离的虚拟环境中安装命令行工具，并无需显式安装即可执行一次性命令，速度比 pipx 更快。此外，uv 还支持 Python 安装，通过 `uv python install` 可以自动下载安装 Python，类似 pyenv 但更高效。对于单文件脚本，uv 支持基于 PEP 723 的内联元数据，只需 `uv run` 即可执行独立的 Python 脚本。

所有这些功能都基于 uv 极快的跨平台依赖解析器。uv 在性能和可靠性上都有突出的优势，无论是小型脚本还是大型项目，从初学者到专家，都能很好地满足 Python 开发的各种需求。

三、主要特性
------

### 1\. 端到端项目管理

uv 在端到端项目管理方面表现出色。通过 `uv run`、`uv lock` 和 `uv sync`，它能够基于标准元数据生成跨平台锁文件，并从中安装依赖。与类似工具如 Poetry、PDM 和 Pipenv 相比，uv 的性能更高。这意味着在处理大型项目时，uv 可以更快地解析和安装依赖，提高开发效率。同时，它还能保证项目环境在不同机器上的一致性，为团队协作提供了便利。

### 2\. 工具管理

在工具管理方面，uv 的 `uv tool install` 和 `uvx`（uv tool run 的别名）功能强大。它们可以在隔离的虚拟环境中安装命令行工具，速度比 pipx 更快。无需显式安装即可执行一次性命令，如 uvx ruff check，为开发者提供了更加便捷的工具使用方式。

### 3\. Python 安装

`uv python install` 在 Python 安装方面有着独特的优势。它可以自动下载安装 Python，类似 pyenv 但更高效。这使得开发者在搭建开发环境时更加轻松快捷，无需繁琐的手动操作。

### 4\. 脚本执行

uv 支持单文件 Python 脚本，通过 uv run 可以执行并自动安装依赖。对于没有包含任何元数据但有依赖的脚本，如 main.py，可以通过 uv add 将依赖声明嵌入到脚本中，然后使用 uv run 在隔离环境中执行脚本，自动安装所需依赖。这种方式为快速开发和测试小型脚本提供了便利。

四、使用示例
------

### 1\. 安装 uv

uv 可以通过多种方式进行安装，以下是一些常见的安装方式：

*   使用 curl： `curl -LsSf <https://astral.sh/uv/install.sh> |sh`

*   使用 pip： `pip install uv`

*   使用 pipx：`pipx install uv`

> 更多安装方式可以查看官方文档：[docs.astral.sh/uv/getting-…](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.astral.sh%2Fuv%2Fgetting-started%2Finstallation%2F "https://docs.astral.sh/uv/getting-started/installation/")

### 2\. 管理 Python 项目

*   初始化项目：可以使用 `uv init` 命令初始化一个 Python 项目。例如，`uv init && uv add "fastapi>=0.112"`会初始化项目并添加 fastapi 依赖，生成如下 pyproject.toml 文件：

```ini
[project]
name = "hello-world"
version = "0.1.0"
readme = "README.md"
dependencies = ["fastapi>=0.112"]

```

*   添加依赖：通过 `uv add` 命令可以添加项目依赖。

*   生成锁文件：`uv` 会基于项目依赖创建一个锁文件，保证项目环境在不同机器上的一致性。

*   执行命令：通过 `uv run` 可以直接在项目环境中执行命令，无需手动激活虚拟环境。

### 3\. 管理命令行工具

*   安装：通过 `uv tool install` 可以把命令行工具（如 Ruff）安装到隔离的虚拟环境中。

*   执行一次性命令：通过 `uvx`（uv tool run 的别名）可以直接执行命令而无需安装，例如 `uvx ruff check`。

### 4\. 执行单文件脚本

*   对于没有包含任何元数据但有依赖的脚本，如 `main.py`，可以通过 \`uv add --script main.py "requests<3" "rich"将依赖声明嵌入到脚本中。

*   然后使用 `uv run main.py` 即可在隔离环境中执行脚本并自动安装依赖。

五、与其他工具对比
---------

### 1\. 与 pip 对比

uv 作为 Python 包管理工具，在性能方面比 pip 快很多，具体而言，不使用缓存时，uv 比 pip 快 8 - 10 倍；而在有热缓存的情况下，如重新创建虚拟环境或更新依赖项时，提速可达 80 - 115 倍。此外，uv 还支持更多高级功能，例如通过 `uv run`、`uv lock` 和 `uv sync` 等命令，可以基于标准元数据生成跨平台的锁文件，并从中安装依赖，而 pip 在这方面的功能相对较弱。

### 2\. 与 pip-tools、pipx、poetry、pyenv、virtualenv 对比

*   **pip-tools**：uv 在没有缓存的情况下比 pip-tools 快 8 - 10 倍，在有热缓存的情况下快 80 - 115 倍。pip-tools 的依赖关系解析功能相对较弱，而 uv 可以基于标准元数据生成跨平台的锁文件，保证项目环境的一致性。

*   **pipx**：在工具管理方面，uv 的 `uv tool install` 和 `uvx`（uv tool run 的别名）可以在隔离的虚拟环境中安装命令行工具，速度比 pipx 更快。无需显式安装即可执行一次性命令，为开发者提供了更加便捷的工具使用方式。

*   **poetry**：uv 在端到端项目管理方面与 Poetry 类似，但性能更高。Poetry 预先解析完整的依赖图，并按拓扑顺序安装依赖项，依据 pyproject.toml 管理项目内外的虚拟环境，还会生成 poetry.lock 文件确保项目的复用。然而，Poetry 的解析速度较慢，部分是因为 Python 包声明支持库的方式不一致，可能会导致支持库解析的时间较长。

*   **pyenv**：uv 在 Python 安装方面，通过 `uv python install` 可以自动下载安装 Python，类似 pyenv 但更高效。pyenv 主要用于切换 Python 版本，并不直接管理虚拟环境，而 uv 不仅可以管理 Python 版本，还能全面管理 Python 项目、命令行工具和单文件脚本。

*   **virtualenv**：uv 旨在作为 pip、pip-tools 和 virtualenv 的直接替代品，在性能和功能上都有很大的提升。uv 比 virtualenv 创建虚拟环境的速度更快，例如，uv 比 python -m venv 快 80 倍，比 virtualenv 快 7 倍，且不依赖于 Python。同时，uv 支持更多高级功能，如跨平台依赖解析、工具管理和脚本执行等。

六、总结
----

uv 作为一款强大的 Python 包管理工具，其功能强大、简单易用的特点在众多方面得到了充分体现。无论是端到端项目管理、工具管理、Python 安装还是脚本执行，uv 都展现出了卓越的性能和可靠性。

在端到端项目管理方面，uv 通过 `uv run`、`uv lock` 和 `uv sync` 等命令，能够基于标准元数据生成跨平台的锁文件，并从中安装依赖。这一功能使得大型项目的依赖安装和解析更加快速，提高了开发效率，同时也保证了项目环境在不同机器上的一致性，为团队协作提供了便利。

在工具管理方面，`uv tool install` 和 `uvx`（uv tool run 的别名）可以在隔离的虚拟环境中安装命令行工具，速度比 pipx 更快。无需显式安装即可执行一次性命令，为开发者提供了更加便捷的工具使用方式。

`uv python install` 在 Python 安装方面有着独特的优势，可以自动下载安装 Python，类似 pyenv 但更高效。这使得开发者在搭建开发环境时更加轻松快捷，无需繁琐的手动操作。

对于单文件脚本，uv 支持基于 PEP 723 的内联元数据，只需 `uv run` 即可执行独立的 Python 脚本。对于没有包含任何元数据但有依赖的脚本，也可以通过 uv add 将依赖声明嵌入到脚本中，然后使用 `uv run` 在隔离环境中执行脚本，自动安装所需依赖。

综上所述，uv 作为 Python 包管理工具，功能强大、简单易用，降低了 Python 开发的复杂度，提高了生产力。无论是小型脚本还是大型项目，从初学者到专家，uv 都能很好地满足 Python 开发的各种需求。如果你还在为 Python 包管理而烦恼，不妨试试 uv，相信它会给你带来全新的开发体验。