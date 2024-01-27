# Python aiohttp 使用指南
aiohttp 就是 **[Python](https://link.juejin.cn/?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Fparsing-json-data-with-python%2F "https://apifox.com/apiskills/parsing-json-data-with-python/")** 中一款优秀的异步 Web 框架，它能够帮助我们构建高效的异步 Web 应用和异步 HTTP 客户端。在本文中，我们将深入探讨 aiohttp 是什么以及如何使用它，通过简单易懂的案例带领你理解异步编程，以及如何处理异步请求和异步 HTTP 客户端。

![](_assets/cee5bb8c7675462ab75d963564f90563~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

什么是 aiohttp？
------------

aiohttp 是一个基于异步 I/O 的 Web 框架，专注于提供高性能、低开销的异步 Web 服务。它允许我们同时处理大量并发请求，而不会阻塞程序执行。aiohttp 使用 Python 的 async/await 语法来实现异步编程，这使得编写异步代码更加直观和简洁。

异步编程简介
------

在传统的同步编程中，每个任务都是按照顺序依次执行的。如果一个任务需要等待一些耗时的操作（如网络请求或文件读取），那么整个程序将会被阻塞，导致其他任务无法执行。而异步编程则可以在等待某些任务的同时，继续执行其他任务，从而充分利用系统资源，提高程序的性能。

在 Python 中，我们可以使用 **[async](https://link.juejin.cn/?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Fhow-to-use-spring-boot-async%2F "https://apifox.com/apiskills/how-to-use-spring-boot-async/")**/await 来声明异步函数和执行异步操作。异步函数将返回一个协程对象，它可以在事件循环中被调度执行。

1.  **协程 (Coroutine)：**  协程是一种可以暂停执行并在稍后恢复的函数，它允许我们在函数内部进行状态保存。在异步编程中，协程非常有用，因为它们允许我们在等待I/O操作的同时，执行其他任务。
2.  **事件循环 (Event Loop)：**  事件循环是异步编程的核心，它是一个循环结构，负责不断地监听并处理事件。在 Python 中，可以使用 `asyncio` 模块提供的事件循环来驱动异步协程的执行。

aiohttp 框架介绍
------------

### 官方地址

aiohttp 的官方地址是：[docs.aiohttp.org/](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.aiohttp.org%2F "https://docs.aiohttp.org/")

### 背景

aiohttp 最初由 Andrew Svetlov 开发，旨在提供一个快速、灵活且易于使用的异步 Web 框架。特点和优势：

*   异步 Web 服务器：aiohttp 可以作为异步 Web 服务器，处理并发请求，支持 **[WebSocket](https://link.juejin.cn/?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Fwebsocket-principle-explained%2F "https://apifox.com/apiskills/websocket-principle-explained/")** 和 HTTP/2 协议。
*   异步 HTTP 客户端：aiohttp 提供了一个高性能的异步 HTTP 客户端，用于向其他服务器发出异步请求。
*   轻量级：aiohttp 的设计简单，代码量较少，易于理解和维护。
*   内置WebSocket支持：aiohttp 内置了对 WebSocket 的支持，可以轻松地实现实时双向通信。
*   兼容性：aiohttp 兼容 Python 3.5+ 版本，并且可以与其他异步库和框架无缝集成。

怎么使用 aiohttp？
-------------

### 安装 aiohttp

在开始使用 aiohttp 之前，我们需要先安装它。可以使用 pip 进行安装：

```null
pip install aiohttp

```

### 实践案例 \- 异步 HTTP 服务器

aiohttp 提供了两个主要模块：`aiohttp.web`和`aiohttp.client`。前者用于构建Web应用程序，后者用于创建异步 **[HTTP](https://link.juejin.cn/?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Fthe-5-pillars-of-every-http-request%2F "https://apifox.com/apiskills/the-5-pillars-of-every-http-request/")** 客户端。

首先，我们来看看`aiohttp.web`模块，它是我们构建 Web 应用程序的关键。以下是一个简单的异步 HTTP 服务器：

```scss
import asyncio
from aiohttp import web

async def handle(request):
    return web.Response(text="Hello, aiohttp!")

app = web.Application()
app.router.add_get('/', handle)

# 手动启动事件循环
async def start_app():
    runner = web.AppRunner(app)
    await runner.setup()
    site = web.TCPSite(runner, 'localhost', 8080)
    await site.start()

if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    try:
        loop.run_until_complete(start_app())
        loop.run_forever()
    except KeyboardInterrupt:
        pass
    finally:
        loop.run_until_complete(loop.shutdown_asyncgens())
        loop.close()


```

在你的 IDE 编辑器中运行上面的代码，你将会看到 aiohttp 服务器已经在本地运行，并监听在默认端口上。当你在浏览器中打开 `http://localhost:8080`，将会看到 "Hello, aiohttp!" 的响应。

![](_assets/10e149e36e4a43998d6235f436669d2b~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### 调试 aiohttp 接口

Apifox = Postman + Swagger + Mock + JMeter，Apifox 支持调试 http(s)、WebSocket、Socket、**[gRPC](https://link.juejin.cn/?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Fintroduction-to-grpc%2F "https://apifox.com/apiskills/introduction-to-grpc/")**、**[Dubbo](https://link.juejin.cn/?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Fdubbo-framework%2F "https://apifox.com/apiskills/dubbo-framework/")** 等协议的接口，在后端人员写完服务接口时，测试阶段可以通过 Apifox 来校验接口的正确性，图形化界面极大的方便了项目的上线效率。

在本文的例子中，就可以通过 Apifox 来测试接口。新建一个项目后，在项目中选择 **“调试模式”** ，填写请求地址后即可快速发送请求，并获得响应结果，上文的实践案例如图所示：

![](_assets/87e30ae07c0d47c5a3d5f7287004f55c~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### 处理异步请求 \- [POST](https://link.juejin.cn/?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Fthe-difference-between-put-and-post%2F "https://apifox.com/apiskills/the-difference-between-put-and-post/") 请求示例

下面我们来演示如何处理异步的 POST 请求：

```python
from aiohttp import web

async def handle(request):
    data = await request.post()
    name = data.get('name')
    return web.Response(text=f"Hello, {name}!")

app = web.Application()
app.router.add_post('/', handle)

web.run_app(app)

```

这个例子中，我们定义了一个 POST 请求的处理函数 `handle`。当收到 POST 请求时，我们从请求中获取名为 `name` 的参数，并返回对应的欢迎信息。

### 处理异步 [HTTP](https://link.juejin.cn/?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Fthe-5-pillars-of-every-http-request%2F "https://apifox.com/apiskills/the-5-pillars-of-every-http-request/") 客户端

除了作为异步 Web 服务器，aiohttp 还提供了一个异步 HTTP 客户端，用于向其他服务器发起异步请求。下面是一个简单的例子：

```python
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    url = 'https://api.example.com/data'
    result = await fetch(url)
    print(result)

asyncio.run(main())

```

上述代码中，我们定义了一个 `fetch` 函数，它使用 aiohttp 的 `ClientSession` 发起一个异步的 GET 请求，并返回响应内容。在 `main` 函数中，我们调用 `fetch` 函数并输出结果。

提示、技巧和注意事项
----------

1.  异步编程需要理解协程、事件循环和异步上下文管理器等概念，熟悉 Python 的 async/await 语法。
2.  使用 aiohttp 时，要注意处理异常和错误，确保程序的稳定性。
3.  在编写异步代码时，要充分利用 asyncio 提供的工具和函数，例如 `asyncio.gather()` 可以同时运行多个协程。
4.  在大规模应用中，可以考虑使用性能分析工具来检测和解决性能瓶颈问题。

总结
--

本文介绍了 aiohttp 是什么以及如何使用它来进行异步编程。我们了解了 aiohttp 的特点和优势，并通过简单的案例展示了如何构建异步 HTTP 服务器和异步 HTTP 客户端。希望本文能够帮助你快速入门 aiohttp，并在异步编程中发挥其优势。

_**知识扩展：** _

*   _**[Sanic 是什么？怎么使用？一文带你快速上手 Sanic](https://link.juejin.cn/?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Fwhat-is-sanic%2F "https://apifox.com/apiskills/what-is-sanic/")**_
*   _**[FastAPI 是什么？怎么使用？](https://link.juejin.cn/?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Ffastapi-get-start%2F "https://apifox.com/apiskills/fastapi-get-start/")**_

_**参考链接：** _

*   _aiohttp 官方文档：[docs.aiohttp.org/](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.aiohttp.org%2F "https://docs.aiohttp.org/")_
*   _Python asyncio 文档：[docs.python.org/3/library/a…](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.python.org%2F3%2Flibrary%2Fasyncio.html "https://docs.python.org/3/library/asyncio.html")_