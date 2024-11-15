# PyGWalker 快速入门 
在 Jupyter Notebook 中快速开始[](#在-jupyter-notebook-中快速开始)
-----------------------------------------------------

在 Jupyter Notebook 中导入 pygwalker 和 pandas 来开始使用。

将你的数据加载成一个 dataframe，并将其传递给 pygwalker。

![](https://docs-us.oss-us-west-1.aliyuncs.com/img/pygwalker/travel-ani-0-light.gif)

> pygwalker 不仅接受 pandas dataframe，还接受 modin dataframe，甚至可以接受数据连接，比如 snowflake。

### 提升 pygwalker 的性能[](#提升-pygwalker-的性能)

有时候你的 dataframe 可能非常大，导致 pygwalker 的性能变慢。现在我们提供了一种简单的方式来提升性能，只需要添加一个额外的参数 `kernel_computation`。

> 通过设置 kernel\_computation=True，将启用由 DuckDB 提供动力的 pygwalker 的新计算引擎。

### 在 Snowflake 中使用 pygwalker[](#在-snowflake-中使用-pygwalker)

有时候你的数据可能非常庞大，你不想将其加载到本地内存中。PyGWalker 允许将其所有计算推送到远程 OLAP 服务，比如 Snowflake。

以下是使用 PyGWalker 在 Snowflake 中的代码示例。

在 Streamlit 中快速开始[](#在-streamlit-中快速开始)
---------------------------------------

PyGWalker 在本地进行数据探索时非常强大，如果能在 web 应用中运行就更好了。 基本上，有很多方式可以实现这一点：

*   使用 [Streamlit (opens in a new tab)](https://streamlit.io/) 构建一个 web 应用。

Streamlit 是一个很好的用 Python 构建数据应用的工具，特别适合那些对 web 开发不太熟悉的数据科学家。 以下是在 Streamlit 中使用 PyGWalker 的快速示例。

[(opens in a new tab)](https://user-images.githubusercontent.com/22167673/271170853-5643c3b1-6216-4ade-87f4-41c6e6893eab.png)

阅读社区文章以了解更多有关如何在 Streamlit 中使用 PyGWalker 的信息：[pygwalker streamlit api](https://docs.kanaries.net/zh/pygwalker/api-reference/streamlit)