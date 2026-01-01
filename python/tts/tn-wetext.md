# wetext vs. WeTextProcessing

好，下面用**中文**把这两个 **TN（Text Normalization / 文本正则化）项目**讲清楚，我会按**定位 → 技术路线 → 能力差异 → 选型建议**来对比，偏向你做 **TTS / 有声书 / 推理系统集成** 的真实需求。

---

# 一句话结论（先给你答案）

> **WeTextProcessing 是“完整、重量级、规则驱动的工业级 TN/ITN 系统”；
> wetext 是“从 WeTextProcessing 拆出来的、轻量、好装、够用的 Python 运行时版本”。**

如果你**要效果和覆盖面** → 选 **WeTextProcessing**
如果你**要易集成、少依赖、推理快** → 选 **wetext**

---

## 一、项目定位对比

### 1️⃣ WeTextProcessing（官方完整版）

* **项目地址**
  [https://github.com/wenet-e2e/WeTextProcessing](https://github.com/wenet-e2e/WeTextProcessing)

* **出身**
  
  * WeNet 官方文本处理模块
  * 面向 **ASR / TTS / 工业级语音系统**

* **目标**
  
  * 提供 **高可控、可解释、规则完备** 的
    
    * TN（文本 → 可读文本）
    * ITN（ASR 输出 → 规范文本）

👉 本质：**规则驱动的“文本编译器”**

---

### 2️⃣ wetext（轻量 Python 版）

* **项目地址**
  [https://github.com/pengzhendong/wetext](https://github.com/pengzhendong/wetext)

* **出身**
  
  * 社区作者基于 WeTextProcessing 思想做的 **简化实现**

* **目标**
  
  * **不依赖 pynini**
  * 用 **纯 Python** 提供 TN / ITN 能力
  * 更适合直接集成到推理 / 应用层

👉 本质：**WeTextProcessing 的“好用运行时子集”**

---

## 二、核心技术路线差异（⚠️ 非常重要）

| 维度   | WeTextProcessing | wetext         |
| ---- | ---------------- | -------------- |
| 核心机制 | **FST（有限状态机）**   | Python 规则 / 逻辑 |
| 依赖   | **pynini（重）**    | 无 pynini（轻）    |
| 规则表达 | 形式语言 / 组合式       | 代码 + 正则        |
| 可解释性 | ⭐⭐⭐⭐⭐            | ⭐⭐⭐            |
| 可维护性 | 偏工程 / 偏学术        | 偏应用            |

### 为什么 pynini 是关键分水岭？

* **WeTextProcessing**
  
  * 使用 FST
  
  * 能精确描述：
    
    * 数字
    * 日期
    * 金额
    * 单位
    * 中英文混排
    * 歧义消解

* **wetext**
  
  * 放弃 FST
  * 牺牲一部分 **极端复杂场景**
  * 换来：**安装简单、部署稳定**

---

## 三、TN / ITN 能力对比（贴近 TTS）

### 1️⃣ 中文数字 & 日期

| 场景           | WeTextProcessing | wetext |
| ------------ | ---------------- | ------ |
| `123`        | 一百二十三            | 一百二十三  |
| `2025-10-11` | 二零二五年十月十一日       | ✅      |
| `3:45`       | 三点四十五分           | ✅      |
| `1.2万`       | 一万二千             | ⚠️部分情况 |

👉 **常规 TTS 输入：两者都 OK**

---

### 2️⃣ 复杂歧义 & 工业级场景

| 场景      | WeTextProcessing | wetext |
| ------- | ---------------- | ------ |
| 金融金额    | 非常强              | 一般     |
| 单位组合    | 非常强              | 有限     |
| 中英混排    | 强                | 中      |
| ASR ITN | 完整               | 有，但不全  |

👉 **ASR → 文本还原**：
**WeTextProcessing 明显更强**

---

## 四、工程 & 集成体验（你会真正关心的）

### 安装体验

* **WeTextProcessing**
  
  ```bash
  pip install pynini   # 很可能失败
  ```
  
  * Linux 还好
  * macOS / Windows / ARM 经常踩雷

* **wetext**
  
  ```bash
  pip install wetext
  ```
  
  * 干净、直接、稳定

---

### Python 调用体验

#### WeTextProcessing（偏重）

```python
from tn.chinese.normalizer import Normalizer

normalizer = Normalizer()
normalizer.normalize("2025年10月11日")
```

#### wetext（偏轻）

```python
from wetext import normalize

normalize("2025年10月11日")
```

👉 **做 TTS 推理 / audiobook pipeline：wetext 明显舒服**

---

## 五、怎么选？（结合你之前的上下文）

结合你前面问过的：

* TTS
* 有声书
* CosyVoice / IndexTTS
* Python 推理环境
* 不想被依赖折腾

### ✅ 强烈建议你的默认方案

> **wetext 作为主 TN 引擎**

原因：

* 你输入文本是“人写的”，不是 ASR 噪声
* 90% audiobook 场景用不到 FST 的极限能力
* 推理稳定性 > 规则完备性

---

### 🧠 什么时候一定要 WeTextProcessing？

只有在这些情况：

* 做 **ASR → 文本还原**
* 做 **金融 / 法律播报**
* 需要 **极端精细的数字/单位歧义处理**
* 能接受复杂依赖

---

## 六、一句工程级总结（给架构用）

> **WeTextProcessing 是“规则引擎”，
> wetext 是“工程可用的 TN SDK”。**

如果你愿意，我可以下一步帮你：

* ✅ 设计 **“有声书专用 TN 层”**（只保留必要规则）
* ✅ 对比 **wetext vs paddlespeech TN** 的集成成本
* ✅ 写一个 **TN Adapter 接口**，以后随时可切换引擎

你直接说下一步要“偏工程”还是“偏算法”。

# Pynini

Pynini 可以把它理解成：**用 Python 写“规则语法”，然后把这些规则编译成（加权）有限状态机/有限状态变换器（WFST）去跑**的一套库。它在语音/文本领域最常见的用途就是做 **Text Normalization（TN）/ Inverse TN（ITN）** 这类“规则型”文本处理。([openfst.org][1])

---

## Pynini 是什么（核心定义）

* **形式化工具**：Pynini 是一个 Python 扩展库，用来**编译、优化、应用**一组文本处理规则。([PyPI][2])
* **编译目标**：把你写的字符串规则、正则表达式、上下文相关改写规则（rewrite rules）等，编译成 **加权有限状态变换器（Weighted Finite-State Transducers / WFST）**。([openfst.org][1])
* **底层依赖**：它基于 **OpenFst**（一个 C++ 的 FST/WFST 工具库）以及其 Python 扩展来做实际的图结构运算与应用。([openfst.org][1])

---

## 为啥它适合做 TN / ITN

TN/ITN 的本质是“把非标准写法转换成可读/可念的标准形式”，例如：

* TN：`"123"` → `"一百二十三"` / `"one hundred twenty three"`
* ITN：`"one hundred twenty three dollars"` → `"$123"`

这类任务有两个特点：

1. **规则多、组合复杂**：日期、金额、单位、电话号码、分数、序号、网络地址……
2. **需要可控 + 可解释**：你想要“按确定规则”输出，而不是模型随机猜。

Pynini 用 WFST 的方式做这事的优势是：规则是“图”，可以组合、求并/交、最短路径、权重选择、确定化/最小化等，**速度快、确定性强、可解释**。官方也明确提到它用于将“rewrite rules”等编译为 WFST，并提供通用优化例程。([Google Research][3])

---

## Pynini 能写什么规则（你会看到的典型能力）

Pynini 常见能力（也是很多 TN 系统的基础构件）：

* **字符串/正则 → FST**：把模式编成“接受器/变换器”
* **上下文相关改写**：类似“在某些上下文里把 A 改写成 B”的规则（例如数字读法、连读、单位组合）([openfst.org][1])
* **加权**：给不同候选输出不同代价，用来在歧义时选更合理的那个（比如 `“3/4”` 是“三分之四”还是“3除以4”）

---

## 为什么很多人觉得它“安装很麻烦”

因为它不是纯 Python 库，它依赖 OpenFst 这套 C++ 工具链（以及编译环境）。所以在某些系统（尤其是没有现成 wheel、或平台比较特殊）上安装会遇到编译/依赖问题。PyPI 上也把它定位为 Python 扩展模块，并说明使用 OpenFst 作为后端。([PyPI][2])
另外也有人专门做“构建 pynini wheels”的仓库来缓解安装问题，这也侧面说明了它的打包/发布在不同平台上可能不那么顺滑。([GitHub][4])

---

## 跟你前面问的 WeTextProcessing / wetext 的关系（一句话串起来）

* **WeTextProcessing**：典型的“基于 Pynini/WFST 的规则 TN/ITN 工程”
* **wetext（pengzhendong/wetext）**：更偏“封装/轻量运行时/减少依赖”的路线（你之前就关注到它更容易装）

如果你告诉我你部署环境（Windows/Oracle Linux aarch64/是否能 conda、是否能编译），我可以按你的环境给出：**安装 Pynini 最稳的路线**（conda-forge / pip wheel / 源码编译）以及是否建议直接用 wetext 这种“免 Pynini”方案。

[1]: https://www.openfst.org/twiki/bin/view/GRM/Pynini?utm_source=chatgpt.com "Pynini < GRM < TWiki"
[2]: https://pypi.org/project/pynini/?utm_source=chatgpt.com "pynini"
[3]: https://research.google/pubs/pynini-a-python-library-for-weighted-finite-state-grammar-compilation/?utm_source=chatgpt.com "A Python library for weighted finite-state grammar compilation"
[4]: https://github.com/ServiceNow/build-pynini-wheels/blob/master/README.md?utm_source=chatgpt.com "build-pynini-wheels/README.md at master"

# ttsfrd

下面用中文把两件事讲清楚：**Pynini 是什么**、以及 **CosyVoice 里常说的 ttsfrd 是什么、谁家的**。

---

## 1) Pynini 是什么？（详细解释）

**Pynini** 是一个用于“有限状态”文本处理的 **Python 工具库**，核心用途是把你写的各种**规则/语法**（例如正则、重写规则、字符串模式等）**编译成加权有限状态转换器**（WFST：Weighted Finite-State Transducer），然后高效地做匹配、转换和规范化。([OpenFST][1])

### 它解决什么问题？

在语音/TTS/ASR 的前端里，经常需要做 **TN/ITN**：

* **TN（Text Normalization）**：把书面形式变成“可读形式”
  例：`2025-12-27` → “二零二五年十二月二十七日”，`3:45` → “三点四十五分”
* **ITN（Inverse Text Normalization）**：反过来把口语/识别文本还原成书面形式
  例：“三点四十五分” → `3:45`

这类任务如果靠纯正则很难覆盖所有边界情况（单位、货币、日期、序数、英文混排等），而 FST/WFST 擅长做**可组合、可控、可解释、可调权重**的规则系统。

### 它和 OpenFst / OpenGrm 的关系

* Pynini 依赖 **OpenFst**（一个 C++ 的有限状态库），Pynini 本质上是“在 Python 里使用 OpenFst 并提供更方便的语法编译能力”。([OpenFST][1])
* 它属于 **OpenGrm** 生态的一部分（同生态里还有 Thrax 等）。([OpenGrm][2])

### 为什么很多人觉得它“安装麻烦”？

因为它涉及到 OpenFst 这类底层依赖，很多平台（尤其 Windows/macOS）经常需要用 **conda-forge** 之类方式来装，直接 pip 可能踩坑（编译/依赖版本）。([Anaconda][3])

---

## 2) CosyVoice 里的 ttsfrd 是什么？谁家的产品？

在 CosyVoice 里，**ttsfrd** 指的是一个可选的“更强的文本前端资源 + 二进制运行库（wheel）”，主要用来提升 **文本规范化（TN）** 等前端处理效果。

### 它在 CosyVoice 里扮演什么角色？

CosyVoice 的说明很明确：

* **可选安装 ttsfrd**：安装后“文本规范化表现更好”
* **不装也能跑**：不安装时会 **fallback 到 WeTextProcessing / wetext** 做 TN（不同版本文档里写法略有差异，但意思一致：不装 ttsfrd 就用 WeTextProcessing/wetext 兜底）。([PyPI][4])

并且 ttsfrd 通常需要：

* 解压 `resource.zip`
* 安装一个平台相关的 wheel（例如 `ttsfrd-...-linux_x86_64.whl`）([PyPI][4])
  这也解释了为什么很多人遇到“平台不支持 wheel / 没有对应 Python 版本”的问题。([GitHub][5])

### 谁家的？

从公开信息看，ttsfrd 与 **ModelScope / iic / KAN-TTS** 这一套生态高度关联：

* ModelScope 上有一个模型条目 **“KANTTS语音合成前端”**，明确写它是 **KANTTS ttsfrd 前端框架依赖的资源模型**。([modelscope.cn][6])
* CosyVoice 的 ttsfrd 资源也经常通过 ModelScope 的 `iic/CosyVoice-ttsfrd` 分发。([modelscope.cn][7])
* CosyVoice 代码里也能看到 **Alibaba Inc** 的版权信息。([GitHub][8])

所以更稳妥的表述是：**ttsfrd 是阿里/ModelScope（iic）相关生态里使用的一个 TTS 前端资源与运行库组件**，在 CosyVoice 中作为可选增强前端使用。([modelscope.cn][6])

### 你什么时候需要它？

* 你在做 **有声书/长文本**，对数字、日期、单位、符号、混合中英的读法一致性要求很高 → 值得装
* 你在 **非 Linux x86_64**、或 Python 版本不匹配 → 很可能装不上，那么就用 **WeTextProcessing/wetext** 兜底（CosyVoice 也支持）。([PyPI][4])

---

简短结论先给你 👇

> **ttsfrd 不是只属于 CosyVoice 的“内部模块”，而是一个**
> **独立存在、但不开源、以二进制 + 资源包形式分发的 TTS 前端产品级组件**。
> CosyVoice 只是“集成并使用了它”。

下面我把这个事**彻底说清楚**，不留模糊地带。

---

## 一、一句话定位 ttsfrd

**ttsfrd = 商用级 TTS Frontend Runtime（前端运行时）**

它本质是一个：

* **文本前端引擎**
* 覆盖：

  * Text Normalization（TN）
  * 分词 / 规则处理
  * 发音与读法决策
* **以二进制 wheel + 资源包形式发布**
* **不开源**

---

## 二、ttsfrd 是否“独立存在”？——是的 ✅

### 结论非常明确：

> **ttsfrd 是独立存在的产品级组件**
> ❌ 不是 CosyVoice 专属
> ❌ 不是 CosyVoice 内部代码
> ✅ CosyVoice 只是“调用者之一”

### 关键证据（逻辑层面）

1. **它有独立的 wheel**

   ```text
   ttsfrd-*-linux_x86_64.whl
   ```

   * 这意味着：

     * 有稳定 API
     * 有明确 ABI
     * 可以被多个项目复用

2. **它有独立的 resource.zip**

   * 前端规则、词典、模型等
   * 与 CosyVoice 模型本身无关

3. **CosyVoice 的行为**

   * 文档明确写：

     * *“如果安装了 ttsfrd，则优先使用”*
     * *“否则 fallback 到 wetext / WeTextProcessing”*

👉 这在架构上已经说明：
**CosyVoice ≠ ttsfrd 的宿主，而是 ttsfrd 的“可选使用方”**

---

## 三、那 ttsfrd 到底是谁家的？

### 更准确的说法是：

> **ttsfrd 来自阿里 / ModelScope / iic / KAN-TTS 这一条产品线**

你可以把它理解为：

* 阿里内部长期使用的 **工业级 TTS 前端**
* 被：

  * KAN-TTS
  * CosyVoice
  * ModelScope 上的多个 TTS 项目
    复用

它不是“开源项目”，而是：

* **对外发布二进制**
* **对内长期演进的核心组件**

这也是为什么：

* 你**看不到源码**
* 但它**效果明显好于 wetext**
* 却**安装受平台限制**

---

## 四、ttsfrd vs wetext / WeTextProcessing（本质区别）

| 维度     | ttsfrd | wetext | WeTextProcessing |
| ------ | ------ | ------ | ---------------- |
| 是否独立产品 | ✅      | ❌（社区库） | ❌（研究/工程库）        |
| 是否开源   | ❌      | ✅      | ✅                |
| 工业成熟度  | ⭐⭐⭐⭐⭐  | ⭐⭐⭐    | ⭐⭐⭐⭐             |
| TN 覆盖度 | 非常完整   | 中等     | 非常完整             |
| 可部署性   | 平台受限   | 非常好    | 一般               |
| 目标用户   | 商业 TTS | 开发者    | 算法/研究            |

**一句话：**

* **ttsfrd = 商用品质**
* **wetext = 工程友好**
* **WeTextProcessing = 规则之王**

---

## 五、为什么 CosyVoice 不“强制绑定” ttsfrd？

这是一个**非常成熟的架构决策**：

1. **ttsfrd 不开源**

   * 强制依赖会限制 CosyVoice 的传播

2. **ttsfrd wheel 平台有限**

   * ARM / Windows / Python 版本经常不匹配

3. **CosyVoice 的定位**

   * 模型是核心
   * 前端可插拔

👉 所以 CosyVoice 的策略是：

```text
if ttsfrd 可用:
    用工业级前端
else:
    用开源 TN 方案
```

这是**非常专业的工程做法**。

---

## 六、给你一个“不会踩坑”的认知模型

你以后可以这样理解：

> * **CosyVoice**：TTS 模型系统
> * **ttsfrd**：可选的、闭源、工业级文本前端
> * **wetext / WeTextProcessing**：开源替代方案

**它们是“组合关系”，不是“隶属关系”**。

---

## 七、给你一个实用建议（结合你之前的目标）

你之前明确说过：

* 有声书
* 推理环境复杂
* Oracle Linux / ARM
* 不想被依赖拖死

### 那我给你一个非常清晰的建议：

> **不要“依赖” ttsfrd**
> **把它当成“有就用、没有也不影响架构”的可选加速器**

架构上建议你：

```text
TN Adapter Interface
 ├─ ttsfrd (optional)
 ├─ wetext (default)
 └─ 自定义规则（补丁）
```

这样你未来：

* 上云
* 换平台
* 商用 / 开源切换
  都不会被锁死。

---

如果你愿意，我下一步可以帮你做一件**非常值钱的事**👇
👉 **给你设计一个“TTS 前端解耦架构图 + Python 接口”**，
让你随时切换 `ttsfrd / wetext / 自定义 TN`，而不用动模型代码。

只要你一句话：**“画架构”** 或 **“给代码”**。


