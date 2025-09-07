# Gradio中文教程（一）Quickstart - 知乎
[老刘：Gradio中文教程-目录](https://zhuanlan.zhihu.com/p/682198989)
----------------------------------------------------------

### 快速入门

Gradio 是一个开源的[Python](https://zhida.zhihu.com/search?content_id=242808333&content_type=Article&match_order=1&q=Python&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ2MDksInEiOiJQeXRob24iLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNDI4MDgzMzMsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.mSHpWToohDSDNHjDvpF-8sptS0EVxCgJ7Io2evctoVM&zhida_source=entity)包，它允许你快速构建一个演示或Web应用程序，用于你的机器学习模型、[API](https://zhida.zhihu.com/search?content_id=242808333&content_type=Article&match_order=1&q=API&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ2MDksInEiOiJBUEkiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNDI4MDgzMzMsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.tao01SUsXqgSFKuhHpp4GiQHO8h_wawwuqicIo5FoXg&zhida_source=entity)，或者任何任意Python函数。然后你可以使用 Gradio内置的分享功能，在几秒钟内就共享出一个链接到你的演示或Web应用程序。无需JavaScript、CSS、或Web托管经验！

### 安装  

前置条件：Gradio 需要 Python 3.8 或以上版本。

我们建议使用 pip 安装 Gradio，pip 包是默认包含在 Python 中的。在你的终端或命令提示窗口运行以下命令：

### 创建你的第一个演示

你可以使用你最喜欢的代码编辑器（如 Visual Studio Code、PyCharm 等）、[Jupyter Notebook](https://zhida.zhihu.com/search?content_id=242808333&content_type=Article&match_order=1&q=Jupyter+Notebook&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ2MDksInEiOiJKdXB5dGVyIE5vdGVib29rIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjQyODA4MzMzLCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.rrd7v5nBS5fkjriJQZoHEOpgMpZoS0muKhO6EoKIHK0&zhida_source=entity)、Google Colab，或任何支持 Python 的环境来运行 Gradio。现在让我们开始编写你的第一个 Gradio 应用：

```python3
import gradio as gr

def greet(name, intensity):
    return "Hello, " + name + "!" * int(intensity)

demo = gr.Interface(
    fn=greet,
    inputs=["text", "slider"],
    outputs=["text"],
)

demo.launch()

```

现在，运行你的代码。如果你已经用Python编写了名为app.py的文件中的Python代码，那么你就会在终端中使用python app.py命令来运行Python代码。

**理解 Interface 类**

你将注意到，在创建第一个演示时，你已经实例化了gr.Interface类。Interface类是为机器学习模型设计的，可以接受一个或多个输入，并返回一个或多个输出。

Interface 类有三个核心参数：

*   fn：要包装用户界面（UI）的函数
*   inputs：用于输入的 Gradio 组件（组件数量应与函数参数数量匹配）
*   outputs：用于输出的 Gradio 组件（组件数量应与函数返回值数量匹配

fn 参数非常灵活—你可以传递任何 Python 函数，你想要用一个 UI包装。在上面的例子中，我们看到了一个相对简单的函数，但是这个函数可以是任何东西从一个音乐生成器到一个税收计算器到预训练机器学习模型的预测函数。

\`inputs\` 和 \`outputs\` 参数接受一个或多个 Gradio 组件。Gradio 集成了超过30个预先设计好的组件，专为机器学习应用服务，比如 \`gr.Textbox()\`、\`gr.Image()\` 和 \`gr.HTML()\` 等。随着你的需求，你可以根据需要选择和组合这些组件来创建功能丰富的 UI。

如果你的函数接受多个参数，就像上面的情况那样，请将输入组件列表传递给inputs，每个输入组件对应于函数中的一个参数，并按顺序排列。同样，如果你的函数返回多个值：简单地传递输出组件列表到outputs。这项灵活性使得Interface类成为创建演示的非常强大的方式。

### 分享你的演示

Gradio 简化了在云端共享机器学习演示的过程，无需为服务器托管烦恼。只需在调用 \`launch()\` 时设置 \`share=True\` 参数，即可生成一个公开可访问的 URL。

```python3
import gradio as gr

def greet(name):
    return "Hello " + name + "!"

demo = gr.Interface(fn=greet, inputs="textbox", outputs="textbox")
    
demo.launch(share=True)  # Share your demo with just 1 extra parameter  
```

当你运行这段代码时，将为您的演示生成一个公共URL，这个过程将在几秒钟内完成，类似于这样的URL：

[https://a23dsf231adb.gradio.live](https://link.zhihu.com/?target=https%3A//a23dsf231adb.gradio.live)

现在，世界各地的任何人都可以尝试你的Gradio演示，通过他们的浏览器，而机器学习模型和所有计算继续运行本地在你的计算机上。要了解更多关于分享您的演示，请阅读我们的专用指南，关于分享您的Gradio应用程序。

### Gradio概述

到目前为止，我们已经讨论了Interface类，这是一个高级别的类，让我们快速构建演示使用Gradio。但是，Gradio还能做些什么？

### **Gradio chat机器人**

Gradio还包括一个高级别的类，[gr. ChatInterface](https://zhida.zhihu.com/search?content_id=242808333&content_type=Article&match_order=1&q=gr.+ChatInterface&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ2MDksInEiOiJnci4gQ2hhdEludGVyZmFjZSIsInpoaWRhX3NvdXJjZSI6ImVudGl0eSIsImNvbnRlbnRfaWQiOjI0MjgwODMzMywiY29udGVudF90eXBlIjoiQXJ0aWNsZSIsIm1hdGNoX29yZGVyIjoxLCJ6ZF90b2tlbiI6bnVsbH0.J-uFlrouhw_pOnaRHud12rMNhcIvsdOgeFsR-1o0lBs&zhida_source=entity)，这个类是专门为创建Chatbot UIs而设计。与Interface类似，你提供一个函数，然后Gradio就生成了一个完全工作的Chatbot UI。如果你对创建chatbot感兴趣，可以直接跳转到我们的关于gr. ChatInterface的指南。

### **自定义演示[gr. Blocks](https://zhida.zhihu.com/search?content_id=242808333&content_type=Article&match_order=1&q=gr.+Blocks&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ2MDksInEiOiJnci4gQmxvY2tzIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjQyODA4MzMzLCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.eC0Gb4hz34lVVR1IS_h1LWV78yaqhRJfRvzBSemEnGI&zhida_source=entity)**

Gradio还提供了一种低级别的approach来设计web应用程序，使用gr. Blocks类，这个类允许你实现更加灵活的布局和数据流。 Blocks允许你控制组件在页面中的位置、处理复杂的数据流（例如输出可以作为其他函数的输入）和根据用户交互更新组件的属性/可见性——所有这些都可以在Python中实现。

使用gr. Blocks()你可以构建非常自定义且复杂的应用程序。例如，流行的图像生成Auto1111 Web UI就是使用Gradio Blocks构建的。我们在我们的关于Blocks的系列指南中更深入地探讨了gr. Blocks。

### **GradioPython & JavaScript生态系统**

  
那是Gradio核心 Python 库的概要，但是Gradio实际上是一个Python和JavaScript库的整个生态系统，允许你在Python或JavaScript中构建机器学习应用程序或编程查询它们。下面是Gradio生态系统中的其他相关部分：

Gradio Python 客户端（gradio\_client）：在 Python 中编程查询任何 Gradio 应用程序。  
Gradio JavaScript 客户端（@gradio/client）：在 JavaScript 中编程查询任何 Gradio 应用程序。  
Gradio-Lite（@gradio/lite）：使用 Pyodide 在浏览器中完全运行 Gradio 应用程序（不需要服务器！），以 Python 写作。  
