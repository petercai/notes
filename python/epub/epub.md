# # spine



好问题，这正是 **ebooklib / EPUB 结构**里最容易让人迷糊、但又**极其核心**的概念之一 👍  
我用「**直觉版 → 规范版 → ebooklib 里的表现 → 常见坑**」四层来讲。

---

## 一句话先给结论

- **spine**：  
  👉 **EPUB 阅读顺序的“目录骨架”**（只定义“按什么顺序读”）

- **spine documents**：  
  👉 **被 spine 引用的那些具体内容文档**（真正的 XHTML 章节文件）

> spine = 顺序  
> spine documents = 内容

---

## 1️⃣ 什么是 spine？（直觉理解）

你可以把 EPUB 想成一本**散装文件合集**：

- 封面是一个文件

- 每一章是一个 XHTML 文件

- 目录是一个文件

- CSS、图片是另外的文件

👉 **spine 就是告诉阅读器：**

> “这些章节，按这个顺序读。”

⚠️ 重要：  
**spine 不等于目录（TOC）**

| 项目        | 作用        |
| --------- | --------- |
| spine     | 阅读顺序（机器用） |
| TOC / nav | 章节导航（人用）  |

---

## 2️⃣ spine 在 EPUB 规范里是什么？

在 EPUB 里，spine 定义在 **OPF（package.opf）** 文件中。

### 标准结构（简化版）

```xml
<package>
  <manifest>
    <item id="chap1" href="chapter1.xhtml" media-type="application/xhtml+xml"/>
    <item id="chap2" href="chapter2.xhtml" media-type="application/xhtml+xml"/>
  </manifest>

  <spine>
    <itemref idref="chap1"/>
    <itemref idref="chap2"/>
  </spine>
</package>
```

### 这里发生了什么？

- **manifest**：  
  👉 “我有哪些文件？”

- **spine**：  
  👉 “这些文件中，哪些参与正文？顺序是？”

所以：

> **spine = manifest 的一个有序子集**

---

## 3️⃣ 什么是 spine documents？

**spine documents 不是 EPUB 里的一个官方术语**，  
而是开发者（包括 ebooklib 文档 / 社区）里常用的说法。

👉 它指的是：

> **所有被 spine 引用的 XHTML 文档**

也就是：

```xml
<itemref idref="chap1"/>
<itemref idref="chap2"/>
```

对应的：

```text
chapter1.xhtml
chapter2.xhtml
```

### 换句话说

- spine 是 **ID 列表**

- spine documents 是 **ID 对应的内容文件**

---

## 4️⃣ 在 ebooklib 里怎么体现？

在 **ebooklib (Python)** 中，这套映射非常直接。

### 1. spine 本身

```python
book.spine
```

通常长这样：

```python
['nav', chap1, chap2, chap3]
```

注意：

- `'nav'` 是特殊值（EPUB3 导航文档）

- 其余是 `EpubHtml` 对象

---

### 2. spine documents（你真正关心的内容）

常见写法：

```python
spine_docs = [
    item for item in book.spine
    if item != 'nav'
]
```

或者更严谨一点：

```python
from ebooklib import ITEM_DOCUMENT

spine_docs = [
    item for item in book.get_items_of_type(ITEM_DOCUMENT)
    if item in book.spine
]
```

👉 **这些就是 spine documents**

---

## 5️⃣ spine vs TOC（非常容易搞混）

这是很多人踩坑的地方，尤其是你在做 **TTS / audiobook** 时。

| 项目      | spine   | TOC   |
| ------- | ------- | ----- |
| 作用      | 阅读顺序    | 导航    |
| 是否必须    | ✅ 必须    | ❌ 可选  |
| 是否线性    | ✅ 强制线性  | ❌ 树结构 |
| 是否可靠    | ✅ 非常可靠  | ❌ 经常乱 |
| TTS 用哪个 | ✅ spine | ❌ TOC |

💡 **做 audiobook / TTS：**

> **永远用 spine，不要信 TOC**

---

## 6️⃣ 实战视角（结合你在做的事）

结合你之前在做的：

- EPUB → 文本抽取

- EPUB → audiobook

- 章节切分 / 顺序保证

### 正确 pipeline 应该是：

```
EPUB
 └─ manifest（有哪些文件）
     └─ spine（读的顺序）
         └─ spine documents（章节 XHTML）
             └─ HTML → text
                 └─ 分句 / TN
                     └─ TTS
```

⚠️ 如果你不用 spine，而是：

- 按文件名排序 ❌

- 按 TOC 顺序 ❌

- 按 get_items() 顺序 ❌

都会在**某些 EPUB 上翻车**

---

## 7️⃣ 常见坑总结（血泪经验）

### ❌ 坑 1：以为 TOC 就是正文顺序

→ 实际：TOC 可以乱、可以缺、可以不全



### ❌ 坑 2：把所有 XHTML 都当正文

→ 实际：有封面页、版权页、广告页

### ❌ 坑 3：忽略 linear="no"

```xml
<itemref idref="ads" linear="no"/>
```

ebooklib 默认会给你，但你做 TTS 时通常要跳过。

---

## 8️⃣ 用一句“工程师级定义”收尾

> **spine 是 EPUB 的“可阅读路径定义”，  
> spine documents 是这条路径上真正承载内容的文档节点。**

如果你愿意，下一步我可以帮你：

- 写一个 **“ebooklib → 干净正文抽取器”**

- 或者专门为 **中文小说 EPUB → audiobook** 做 spine-aware pipeline  
  （你前面聊 CosyVoice / XTTS，这一步非常关键）







