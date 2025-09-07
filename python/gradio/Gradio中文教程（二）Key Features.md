# Gradio中文教程（二）Key Features
[老刘：Gradio中文教程-目录](https://zhuanlan.zhihu.com/p/682198989)
----------------------------------------------------------

关键特性
----

让我们来看看Gradio的一些关键特性。本指南旨在为您构建演示时应该了解的各种事项提供一个高层次的概述。在适当的地方，我们会链接到特定主题的更详细的指组件。

队列

[流式输出](https://zhida.zhihu.com/search?content_id=242926998&content_type=Article&match_order=1&q=%E6%B5%81%E5%BC%8F%E8%BE%93%E5%87%BA&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ3MDksInEiOiLmtYHlvI_ovpPlh7oiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNDI5MjY5OTgsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.jVBNDziF_cpCjAciYNJuWSCbNUY7LVJWP0Abmy3F4i0&zhida_source=entity)

[流式输入](https://zhida.zhihu.com/search?content_id=242926998&content_type=Article&match_order=1&q=%E6%B5%81%E5%BC%8F%E8%BE%93%E5%85%A5&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ3MDksInEiOiLmtYHlvI_ovpPlhaUiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNDI5MjY5OTgsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.Je-csTXCM83qlnf_NDaP2lRUpKu-r_fyUsBwEt8Nls0&zhida_source=entity)

[警告模态框](https://zhida.zhihu.com/search?content_id=242926998&content_type=Article&match_order=1&q=%E8%AD%A6%E5%91%8A%E6%A8%A1%E6%80%81%E6%A1%86&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ3MDksInEiOiLorablkYrmqKHmgIHmoYYiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNDI5MjY5OTgsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.Johq1oPWevmEZIAGILq0K6yYM27NtxKeR5lSuRgXpjY&zhida_source=entity)

样式

[进度条](https://zhida.zhihu.com/search?content_id=242926998&content_type=Article&match_order=1&q=%E8%BF%9B%E5%BA%A6%E6%9D%A1&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ3MDksInEiOiLov5vluqbmnaEiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNDI5MjY5OTgsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.3ulqInwHQ46kbCXl0thlBm7DFVxVV4K_CeSXsyUnnlI&zhida_source=entity)

[批量函数](https://zhida.zhihu.com/search?content_id=242926998&content_type=Article&match_order=1&q=%E6%89%B9%E9%87%8F%E5%87%BD%E6%95%B0&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ3MDksInEiOiLmibnph4_lh73mlbAiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNDI5MjY5OTgsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.o_f3VC11o5hPN3RBY84_KJd1t4e9tf-i62MXYyK6EtM&zhida_source=entity)

### 组件

Gradio包含超过30个预构建的组件（以及许多用户构建的自定义组件），这些组件可以用一行代码作为您演示的输入或输出。这些组件对应于机器学习和数据科学中的常见数据类型，例如，gr.Image组件设计用于处理输入或输出图像，gr.Label组件显示分类标签和概率，gr.Plot组件显示各种类型的图表，等等。

每个组件都包含各种构造函数属性，这些属性控制组件的属性。例如，您可以使用gr.Textbox的构造函数中的lines参数（接受一个正整数）来控制文本框中的行数。或者，您可以使用gr.Image组件的sources参数（接受一个列表，如\["webcam", "upload"\]）来控制用户提供图像的方式。

**静态和交互式组件**

每个组件都有一个设计用于显示数据的静态版本，大多数组件还有一个设计用于让用户输入或修改数据的交互式版本。通常，您不需要考虑这种区别，因为当您构建Gradio演示时，Gradio会自动根据组件是作为输入还是输出来确定组件应该是静态的还是交互式的。但是，您可以使用每个组件都支持的interactive参数手动设置这一点。

**预处理和后处理**

当组件用作输入时，Gradio会自动处理将数据从用户浏览器发送的类型（如上传的图像）转换为您的函数可以接受的格式（如numpy数组）所需的预处理。

类似地，当组件用作输出时，Gradio会自动处理将数据从您的函数返回的格式（如图像路径列表）转换为用户浏览器可以显示的格式（图像画廊）所需的后处理。

考虑一个具有三个输入组件（gr.Textbox、gr.Number和gr.Image）和两个输出（[gr.Number和gr.Gallery](https://link.zhihu.com/?target=http%3A//gr.xn--numbergr-4c2n.gallery/)）的示例演示，这些组件作为图像到图像生成模型的UI。下面是我们预处理将数据从浏览器发送到模型以及我们的后处理将从模型接收的数据所需的图示。

在这张图片中，以下预处理步骤将数据从浏览器发送到您的函数：

*   文本框中的文本转换为Python str（本质上没有预处理）
*   数字输入中的数字转换为Python float（本质上没有预处理）
*   最重要的是，用户提供的图像被转换为图像中RGB值的numpy.array表示

图像被转换为NumPy数组，因为它们是机器学习工作流程的常见格式。您可以使用构造组件时组件的参数来控制预处理。例如，如果您使用以下参数实例化Image组件，它将预处理图像为PIL格式：

```python3
img = gr.Image(type="pil")
```

后处理甚至更简单！Gradio自动识别返回数据的格式（例如，用户的函数是否为gr.Image组件返回一个numpy数组或一个str文件路径？）并将其适当地后处理为浏览器可以显示的格式。

所以在上面的图片中，以下后处理步骤将数据从用户的函数发送到浏览器：

*   float显示为数字并直接显示给用户
*   字符串文件路径列表（list\[str\]）被解释为图像文件路径列表，并在浏览器中显示为画廊

查看文档以查看每个Gradio组件的所有参数。

### 队列

每个Gradio应用程序都带有一个内置的队列系统，可以扩展到数千个并发用户。您可以通过使用queue()方法来配置队列，该方法由gr.Interface、gr.Blocks和gr.ChatInterface类支持。

例如，您可以通过设置queue()的default\_concurrency\_limit参数来控制单个时间处理的请求数量，例如：

```text
demo = gr.Interface(...).queue(default_concurrency_limit=5)
demo.launch()
```

这限制了单个时间处理此事件监听器的请求数量为5。默认情况下，default\_concurrency\_limit实际上设置为1，这意味着当许多用户使用您的应用程序时，只有一个用户的请求将在一个时间处理。这是因为许多机器学习函数消耗大量内存，因此只适合单个用户在一个时间使用演示。但是，您可以轻松地在演示中更改此参数。

有关配置队列参数的更多详细信息，请参阅队列的文档。

### 流式输出

在某些情况下，您可能希望流式传输一系列输出而不是一次性显示单个输出。例如，您可能有一个图像生成模型，并且希望在每一步显示生成的图像，直到最终图像。或者您可能有一个聊天机器人，它一次流式传输一个令牌的响应，而不是一次性返回所有响应。

在这种情况下，您可以将生成器函数提供给Gradio，而不是常规函数。在Python中创建生成器非常简单：一个函数应该yield一系列值，而不是单个return值。通常，yield语句放在某种循环中。以下是一个简单的生成器示例，它简单地计数到给定数字：

```python3
def my_generator(x):
    for i in range(x):
        yield i
```

您将生成器提供给Gradio的方式与提供常规函数的方式相同。例如，这里有一个（假的）图像生成模型，它在输出图像之前使用gr.Interface类生成几个步骤的噪声：

```text
import gradio as gr
import numpy as np
import time

def fake_diffusion(steps):
    rng = np.random.default_rng()
    for i in range(steps):
        time.sleep(1)
        image = rng.random(size=(600, 600, 3))
        yield image
    image = np.ones((1000,1000,3), np.uint8)
    image[:] = [255, 124, 0]
    yield image


demo = gr.Interface(fake_diffusion,
                    inputs=gr.Slider(1, 10, 3, step=1),
                    outputs="image")

demo.launch()

```

请注意，我们在迭代器中添加了一个time.sleep(1)来在步骤之间创建一个人工暂停，以便您能够观察迭代器的步骤（在真实的图像生成模型中，这可能不是必需的）。

### 流式输入

类似地，Gradio可以处理流式输入，例如实时音频流可以实时转录为文本，或者图像生成模型在用户在文本框中键入字母时重新运行。这在我们的构建反应式界面的指南中有更详细的介绍。

### 警告模态框

您可能希望向用户发出警告。为此，请引发一个gr.Error("custom message")来显示错误消息。您还可以发出gr.Warning("message")[和gr.Info](https://link.zhihu.com/?target=http%3A//xn--gr-iy6c.info/)("message")，只需将它们作为函数中的独立行，这将立即显示模态框，同时继续执行您的函数。需要启用队列才能使此功能正常工作。

请注意下面的gr.Error是如何被引发的，而gr.Warning和[http://gr.Info](https://link.zhihu.com/?target=http%3A//gr.Info)是单行。

```text
def start_process(name):
    gr.Info("Starting process")
    if name is None:
        gr.Warning("Name is empty")
    ...
    if success == False:
        raise gr.Error("Process failed")
```

### 样式

Gradio主题是自定义应用程序外观和感觉的最简单方法。您可以从各种主题中选择，或者创建自己的主题。为此，请将theme=关键字参数传递给Interface构造函数。例如：

```text
demo = gr.Interface(..., theme=gr.themes.Monochrome())
```

Gradio附带了一组预构建的主题，您可以从gr.themes.\*加载。您可以扩展这些主题或从头开始创建自己的主题 - 有关详细信息，请参阅主题指南。

为了获得额外的样式能力，您可以将任何CSS（以及自定义JavaScript）传递给您的Gradio应用程序。这在我们的自定义JS和CSS指南中有更详细的讨论。

### 进度条

Gradio支持创建自定义进度条的能力，以便您可以自定义和控制向用户显示的进度更新。为了启用此功能，只需将具有默认值的gr.Progress实例的参数添加到您的方法中。然后，您可以通过直接调用此实例并传递0到1之间的浮点数，或者使用Progress实例的tqdm()方法来跟踪可迭代对象的进度，如下所示。

```text
import gradio as gr
import time

def slowly_reverse(word, progress=gr.Progress()):
    progress(0, desc="Starting")
    time.sleep(1)
    progress(0.05)
    new_string = ""
    for letter in progress.tqdm(word, desc="Reversing"):
        time.sleep(0.25)
        new_string = letter + new_string
    return new_string

demo = gr.Interface(slowly_reverse, gr.Text(), gr.Text())

demo.launch()

```

如果您使用tqdm库，您甚至可以通过将默认参数设置为gr.Progress(track\_tqdm=True)来自动从函数中已存在的任何tqdm.tqdm报告进度更新！

### 批量函数

Gradio支持传递批量函数。批量函数只是接受输入列表并返回预测列表的函数。

例如，这里是一个批量函数，它接受两个输入列表（一个单词列表和一个整数列表），并返回一个修剪后的单词列表作为输出：

```text
import time

def trim_words(words, lens):
    trimmed_words = []
    time.sleep(5)
    for w, l in zip(words, lens):
        trimmed_words.append(w[:int(l)])
    return [trimmed_words]
```

使用批量函数的优点是，如果您启用队列，Gradio服务器可以自动批量处理传入请求并在并行处理它们，潜在地加快您的演示速度。以下是Gradio代码的样子（注意batch=True和max\_batch\_size=16）

使用gr.Interface类：

```text
demo = gr.Interface(
    fn=trim_words, 
    inputs=["textbox", "number"], 
    outputs=["output"],
    batch=True, 
    max_batch_size=16
)

demo.launch()
```

使用gr.Blocks类：

```text
import gradio as gr

with gr.Blocks() as demo:
    with gr.Row():
        word = gr.Textbox(label="word")
        leng = gr.Number(label="leng")
        output = gr.Textbox(label="Output")
    with gr.Row():
        run = gr.Button()

    event = run.click(trim_words, [word, leng], output, batch=True, max_batch_size=16)

demo.launch()
```

在上面的示例中，16个请求可以并行处理（总推理时间为5秒），而不是每个请求单独处理（总推理时间为80秒）。许多[Hugging Face transformers](https://zhida.zhihu.com/search?content_id=242926998&content_type=Article&match_order=1&q=Hugging+Face+transformers&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTc0NTQ3MDksInEiOiJIdWdnaW5nIEZhY2UgdHJhbnNmb3JtZXJzIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjQyOTI2OTk4LCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.ts-Ax9y6aROk1fk_UkXyy6r1z0FIFl9dINR-np8Go5o&zhida_source=entity)和diffusers模型与Gradio的批量模式非常自然地工作：这里有一个使用diffusers生成批量图像的示例演示。
