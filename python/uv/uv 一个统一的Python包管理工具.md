# uv: 一个统一的Python包管理工具
`uv`是由Astral公司开发的一个极其快速的Python包管理器,完全用`Rust`编写。它最初在2月份发布,作为`pip`工作流的替代品。现在,`uv`已经扩展成为一个端到端的解决方案,可以管理Python项目、命令行工具、单文件脚本,甚至Python本身。可以说,[uv](https://so.csdn.net/so/search?q=uv&spm=1001.2101.3001.7020)就像是Python界的Cargo:一个快速、可靠、易用的统一接口。

### 主要特性

uv具有以下主要功能:

**端到端项目管理:**  
通过`uv run`、`uv lock`和`uv sync`,uv可以基于标准元数据生成跨平台的锁文件,并从中安装依赖,类似Poetry、PDM和Rye等工具,但性能更高。

**工具管理:**  
通过`uv tool install`和`uv tool run`(别名`uvx`),uv可以在隔离的虚拟环境中安装命令行工具,并无需显式安装即可执行一次性命令(如`uvx ruff check`),类似`pipx`但速度更快。

**Python安装:**  
通过`uv python install`,uv可以自动下载安装Python,类似`pyenv`但更高效。

**脚本执行:**  
uv支持基于PEP 723的内联元数据的单文件Python脚本。只需`uv run`即可执行独立的Python脚本。

以上所有功能都基于uv极快的跨平台依赖解析器。下图展示了在有无缓存的情况下,uv解析Transformers项目全部可选依赖的速度:  
![](https://i-blog.csdnimg.cn/direct/99615a2e3f4a4be091f5553c31d3cb2c.png)

### 使用示例

下面通过一些示例代码展示uv的基本用法。

**安装uv:**

```
`$ curl -LsSf https://astral.sh/uv/install.sh | sh
$ pip install uv 
$ pipx install uv` 

*   1
*   2
*   3


```

**管理Python项目:**

初始化项目并添加fastapi依赖:

```
`$ uv init && uv add "fastapi>=0.112"` 

*   1


```

会生成如下`pyproject.toml`:

```
`[project]
name = "hello-world"  
version = "0.1.0"
readme = "README.md"
dependencies = ["fastapi>=0.112"]` 

*   1
*   2
*   3
*   4
*   5


```

然后uv会基于项目依赖创建一个锁文件,保证项目环境在不同机器上的一致性。

通过`uv run`可以直接在项目环境中执行命令,无需手动激活虚拟环境

**管理命令行工具:**

通过`uv tool install`可以把命令行工具(如Ruff)安装到隔离的虚拟环境中,通过`uvx`可以直接执行命令而无需安装:

**执行单文件脚本:**

考虑如下`main.py`,它依赖`requests`和`rich`,但没有包含任何元数据:

```
`import requests
from rich.pretty import pprint

resp = requests.get("https://peps.python.org/api/peps.json") 
data = resp.json()
pprint([(k, v["title"]) for k, v in data.items()][:10])` 

*   1
*   2
*   3
*   4
*   5
*   6


```

通过`uv add`可以自动将依赖声明嵌入到脚本中:

```
`$ uv add --script main.py "requests<3" "rich"` 

*   1


```

然后通过`uv run main.py`即可在隔离环境中执行脚本并自动安装依赖

### 总结

uv是一个功能强大、简单易用的Python包管理工具,集成了端到端的解决方案。它在性能和可靠性上都有突出的优势。

无论是小型脚本还是大型项目,从初学者到专家,uv都能很好地满足Python开发的各种需求。Astral公司希望通过uv来降低Python开发的复杂度,提高生产力。

如果你在内部使用uv,并有兴趣在Python打包方面与Astral合作,欢迎联系他们,共同应对未来的挑战。