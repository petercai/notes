# overview

# TQDM 详细介绍

TQDM 是 Python 中一个非常流行的进度条库，它的名字来源于阿拉伯语 "taqaddum"（تقدّم），意思是"进步"。它可以让你在循环或任何可迭代对象中轻松添加进度条，让长时间运行的任务变得可视化。

## 基本特点

- 使用简单，只需一行代码就能添加进度条
- 开销很小，对程序性能影响极小
- 支持多种使用场景：循环、文件读取、多进程等
- 显示丰富的信息：已完成数量、总数、百分比、速度、剩余时间等
- 同时支持命令行和 Jupyter Notebook

## 安装方法

```bash
pip install tqdm
```

## 基本使用

### 1. 最简单的用法

```python
from tqdm import tqdm
import time

# 在普通循环中使用
for i in tqdm(range(100)):
    time.sleep(0.01)  # 模拟耗时操作
```

### 2. 手动控制进度

```python
from tqdm import tqdm

# 创建进度条对象
pbar = tqdm(total=100)

for i in range(10):
    # 执行一些操作
    time.sleep(0.1)
    # 手动更新进度（每次更新10）
    pbar.update(10)

pbar.close()
```

### 3. 使用 with 语句（推荐）

```python
from tqdm import tqdm

with tqdm(total=100) as pbar:
    for i in range(100):
        time.sleep(0.01)
        pbar.update(1)
```

### 4. 遍历列表

```python
items = ['apple', 'banana', 'orange', 'grape']

for item in tqdm(items, desc="处理水果"):
    # 处理每个项目
    time.sleep(0.5)
```

## 常用参数

```python
from tqdm import tqdm

for i in tqdm(
    range(100),
    desc="任务进度",        # 进度条前的描述文字
    total=100,            # 总迭代次数
    ncols=80,             # 进度条宽度
    unit="个",            # 单位
    unit_scale=True,      # 自动调整单位（如 K, M）
    colour="green",       # 进度条颜色
    leave=True,           # 完成后是否保留进度条
    position=0,           # 多进度条时的位置
    ascii=False           # 是否使用 ASCII 字符
):
    time.sleep(0.01)
```

## 实用示例

### 文件读取

```python
with open('large_file.txt', 'r') as f:
    # 获取文件总行数
    total_lines = sum(1 for _ in f)
    f.seek(0)  # 重置文件指针
    
    # 带进度条读取
    for line in tqdm(f, total=total_lines, desc="读取文件"):
        # 处理每一行
        pass
```

### 数据处理

```python
import pandas as pd
from tqdm import tqdm

# 为 pandas 启用进度条
tqdm.pandas(desc="处理数据")

df = pd.DataFrame({'A': range(10000)})
# 使用 progress_apply 替代 apply
df['B'] = df['A'].progress_apply(lambda x: x ** 2)
```

### 嵌套循环

```python
from tqdm import tqdm

for i in tqdm(range(5), desc="外层循环"):
    for j in tqdm(range(100), desc="内层循环", leave=False):
        time.sleep(0.01)
```

### 手动设置描述和后缀信息

```python
from tqdm import tqdm

pbar = tqdm(range(100))
for i in pbar:
    # 动态更新描述
    pbar.set_description(f"处理 {i}")
    # 添加后缀信息
    pbar.set_postfix(loss=0.5, acc=0.95)
    time.sleep(0.01)
```

## Jupyter Notebook 中使用

```python
from tqdm.notebook import tqdm

# 在 Notebook 中会显示更美观的进度条
for i in tqdm(range(100)):
    time.sleep(0.01)
```

## 进阶技巧

### 多进程进度条

```python
from tqdm import tqdm
from multiprocessing import Pool

def process_item(item):
    time.sleep(0.1)
    return item * 2

if __name__ == '__main__':
    items = range(100)
    with Pool(4) as pool:
        results = list(tqdm(
            pool.imap(process_item, items),
            total=len(items)
        ))
```

### 自定义格式

```python
from tqdm import tqdm

bar_format = '{desc}: {percentage:3.0f}%|{bar}| {n_fmt}/{total_fmt} [{elapsed}<{remaining}]'

for i in tqdm(range(100), bar_format=bar_format):
    time.sleep(0.01)
```

## 总结

TQDM 是一个功能强大且易用的进度条库，特别适合：
- 数据处理和分析脚本
- 机器学习训练过程
- 大文件读写
- 批量操作
- 任何需要可视化进度的场景

它的最大优势就是使用简单，只需在原有的迭代器外包裹一层 `tqdm()` 就能立即看到进度，这让调试和监控长时间运行的程序变得非常方便。


# integration with Gradio.Progress

# TQDM 与 Gradio Progress 的集成

是的，tqdm 可以与 Gradio 的 Progress 组件集成！Gradio 提供了专门的进度跟踪机制，可以在 Web 界面上显示任务进度。

## Gradio Progress 基本用法

### 1. 最简单的方式

```python
import gradio as gr
import time

def process_data(progress=gr.Progress()):
    # 自动跟踪迭代进度
    for i in progress.tqdm(range(100)):
        time.sleep(0.01)
    return "处理完成！"

demo = gr.Interface(
    fn=process_data,
    inputs=[],
    outputs="text"
)

demo.launch()
```

### 2. 手动更新进度

```python
import gradio as gr
import time

def process_with_manual_progress(progress=gr.Progress()):
    progress(0, desc="开始处理...")
    
    # 第一阶段
    for i in range(50):
        time.sleep(0.02)
        progress((i + 1) / 100, desc="第一阶段处理中...")
    
    # 第二阶段
    for i in range(50, 100):
        time.sleep(0.02)
        progress((i + 1) / 100, desc="第二阶段处理中...")
    
    return "全部完成！"

demo = gr.Interface(
    fn=process_with_manual_progress,
    inputs=[],
    outputs="text"
)

demo.launch()
```

## TQDM 与 Gradio 集成方式

### 方法 1: 使用 progress.tqdm()（推荐）

这是最简单的方式，直接使用 Gradio 的 `progress.tqdm()` 方法：

```python
import gradio as gr
import time

def process_items(items, progress=gr.Progress()):
    results = []
    
    # 使用 progress.tqdm() 包装迭代器
    for item in progress.tqdm(items, desc="处理数据"):
        # 模拟处理
        time.sleep(0.1)
        results.append(item * 2)
    
    return f"处理了 {len(results)} 个项目"

demo = gr.Interface(
    fn=process_items,
    inputs=gr.Textbox(value="1,2,3,4,5", label="输入数字（逗号分隔）"),
    outputs="text"
)

demo.launch()
```

### 方法 2: 结合原生 tqdm 使用

```python
import gradio as gr
from tqdm import tqdm
import time

def process_with_tqdm(num_items, progress=gr.Progress()):
    results = []
    
    # 使用 tqdm 但通过回调更新 Gradio progress
    progress(0, desc="初始化...")
    
    pbar = tqdm(range(num_items), desc="处理中")
    for i in pbar:
        time.sleep(0.1)
        results.append(i ** 2)
        
        # 同步更新 Gradio 进度
        progress((i + 1) / num_items, desc=f"已处理 {i + 1}/{num_items}")
    
    return f"完成！结果: {results[:10]}..."  # 只显示前10个

demo = gr.Interface(
    fn=process_with_tqdm,
    inputs=gr.Slider(10, 100, value=50, label="处理项目数"),
    outputs="text"
)

demo.launch()
```

### 方法 3: 多阶段进度跟踪

```python
import gradio as gr
import time

def multi_stage_process(data, progress=gr.Progress()):
    # 阶段1: 数据加载
    progress(0, desc="阶段1: 加载数据...")
    for i in progress.tqdm(range(30), desc="加载中"):
        time.sleep(0.02)
    
    # 阶段2: 数据处理
    progress(0.3, desc="阶段2: 处理数据...")
    for i in progress.tqdm(range(40), desc="处理中"):
        time.sleep(0.02)
    
    # 阶段3: 保存结果
    progress(0.7, desc="阶段3: 保存结果...")
    for i in progress.tqdm(range(30), desc="保存中"):
        time.sleep(0.02)
    
    return "所有阶段完成！"

demo = gr.Interface(
    fn=multi_stage_process,
    inputs=gr.Textbox(label="输入数据"),
    outputs="text"
)

demo.launch()
```

## 实际应用场景

### 示例 1: 图像批量处理

```python
import gradio as gr
import time

def batch_process_images(images, progress=gr.Progress()):
    if not images:
        return "请上传图片"
    
    processed = []
    
    for idx, img in progress.tqdm(
        enumerate(images), 
        total=len(images),
        desc="处理图片"
    ):
        # 模拟图像处理
        time.sleep(0.5)
        processed.append(img)
        
        # 可以添加更详细的信息
        progress(
            (idx + 1) / len(images), 
            desc=f"已处理 {idx + 1}/{len(images)} 张图片"
        )
    
    return f"成功处理 {len(processed)} 张图片"

demo = gr.Interface(
    fn=batch_process_images,
    inputs=gr.File(file_count="multiple", label="上传多张图片"),
    outputs="text"
)

demo.launch()
```

### 示例 2: 文件处理

```python
import gradio as gr
import time

def process_file(file, progress=gr.Progress()):
    if file is None:
        return "请上传文件"
    
    # 模拟读取文件
    progress(0, desc="读取文件...")
    time.sleep(0.5)
    
    # 模拟处理行
    total_lines = 100  # 假设有100行
    results = []
    
    for i in progress.tqdm(range(total_lines), desc="处理行"):
        time.sleep(0.02)
        results.append(f"处理第 {i+1} 行")
    
    progress(1.0, desc="完成！")
    
    return f"文件处理完成，共处理 {len(results)} 行"

demo = gr.Interface(
    fn=process_file,
    inputs=gr.File(label="上传文件"),
    outputs="text"
)

demo.launch()
```

### 示例 3: 使用 Blocks 实现更复杂的布局

```python
import gradio as gr
import time

def complex_task(text, progress=gr.Progress()):
    # 数据预处理
    progress(0, desc="步骤 1/3: 预处理...")
    for _ in progress.tqdm(range(33), desc="预处理"):
        time.sleep(0.03)
    
    # 主处理
    progress(0.33, desc="步骤 2/3: 主处理...")
    for _ in progress.tqdm(range(34), desc="主处理"):
        time.sleep(0.03)
    
    # 后处理
    progress(0.67, desc="步骤 3/3: 后处理...")
    for _ in progress.tqdm(range(33), desc="后处理"):
        time.sleep(0.03)
    
    return f"处理完成！输入内容: {text}"

with gr.Blocks() as demo:
    gr.Markdown("# 带进度条的任务处理")
    
    with gr.Row():
        with gr.Column():
            input_text = gr.Textbox(label="输入文本", value="测试数据")
            submit_btn = gr.Button("开始处理")
        
        with gr.Column():
            output_text = gr.Textbox(label="处理结果")
    
    submit_btn.click(
        fn=complex_task,
        inputs=input_text,
        outputs=output_text
    )

demo.launch()
```

## 关键要点

1. **参数传递**: 必须在函数定义中添加 `progress=gr.Progress()` 参数
2. **更新方式**: 
   - `progress.tqdm()` - 自动跟踪迭代器
   - `progress(value, desc)` - 手动更新，value 是 0-1 之间的浮点数
3. **描述文本**: 使用 `desc` 参数显示当前阶段的描述信息
4. **多阶段**: 可以多次调用 progress 来更新不同阶段

## 注意事项

- Gradio 的 progress 只在函数内部有效
- 进度值范围是 0 到 1（0% 到 100%）
- `progress.tqdm()` 会自动计算百分比，非常方便
- 在 Jupyter Notebook 中使用时，进度条会显示在 Gradio 的 Web 界面上，而不是笔记本中

这样的集成让你的 Gradio 应用更加用户友好，用户可以实时看到任务的处理进度！