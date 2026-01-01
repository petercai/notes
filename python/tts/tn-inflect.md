

**Python 的 `inflect` 是一个「英文词形变化 + 数字语言化」库**，  
主要用来把**“机器友好的文本”变成“人类/语音友好的英文”**。

> 👉 它做的是 **英文 inflection（屈折变化）**，不是语法分析，也不是翻译。

---

## 它主要干什么？（三大核心能力）

### 1️⃣ 数字 → 英文读法（TTS 核心用途）

```python
import inflect
p = inflect.engine()

p.number_to_words(123)
# 'one hundred and twenty-three'

p.number_to_words(2025)
# 'two thousand and twenty-five'
```

这在 **TTS / audiobook / ASR / 语音助手**里是刚需。

如果你直接把 `123` 喂给英文 TTS：

- ❌ 有的会念成 “one two three”

- ❌ 有的直接报错

- ❌ 有的节奏很怪

👉 **inflect 就是英文 TN（Text Normalization）的一部分**

---

### 2️⃣ 单复数、量词、冠词自动处理

```python
p.plural("book")
# 'books'

p.plural("child")
# 'children'

p.singular_noun("mice")
# 'mouse'
```

甚至还能帮你处理 **a / an**：

```python
p.a("apple")
# 'an apple'

p.a("banana")
# 'a banana'
```

对比你自己写规则：

- 不规则复数一堆

- 发音规则一堆

- 特例更多

👉 inflect 已经把**英文坑全踩过一遍**

---

### 3️⃣ 序数、数量表达（第几、多少个）

```python
p.ordinal(3)
# '3rd'

p.number_to_words(3, ordinal=True)
# 'third'

p.no("apple", 5)
# 'five apples'

p.no("child", 2)
# 'two children'
```

非常适合：

- 报告生成

- 自动播报

- UI / 可读文本

- audiobook narration

---

## inflect ≠ num2words（关键区别）

你前面提到你在用 `num2words`，这点非常关键。

| 功能      | inflect | num2words |
| ------- | ------- | --------- |
| 数字 → 英文 | ✅       | ✅         |
| 单复数处理   | ✅       | ❌         |
| a / an  | ✅       | ❌         |
| 量词智能    | ✅       | ❌         |
| 英文特化    | ⭐⭐⭐⭐    | ⭐⭐        |
| 多语言     | ❌（只英文）  | ✅         |

👉 **结论**：

- **纯英文 + TTS → inflect 更专业**

- **多语言数字 → num2words**

在英文 audiobook pipeline 里，**inflect 是更“工程级”的选择**。

---

## 和你熟悉的 TTS 场景怎么对应？

### 在 TTS 里的角色定位

```
Raw Text
  ↓
Text Normalization (TN)
  ├─ inflect (numbers, plural, ordinal)
  ├─ date / time
  ├─ abbreviation
  ↓
Clean Spoken Text
  ↓
TTS Model
```

例如：

```text
Chapter 3, published in 2025.
```

经 TN 处理后：

```text
Chapter three, published in two thousand and twenty-five.
```

👉 这一步 **TTS 模型本身通常不做**，或者做得不完整。

---

## 它解决什么问题，又解决不了什么？

### ✅ 擅长的

- 英文数字、数量

- 单复数、词形变化

- 让文本“听起来像人说的”

### ❌ 不擅长 / 不做

- 中文（完全不支持）

- 日期 / 时间的复杂格式（要你自己拼）

- 语义理解

- 句法分析

👉 所以它**不是 NLP 全家桶，只是 TN 里的一块砖**

---

## 一个非常典型的使用方式（工程级）

```python
import inflect
p = inflect.engine()

def normalize_en(text):
    text = re.sub(r"\d+", lambda m: p.number_to_words(m.group()), text)
    return text
```

然后再接：

- 缩写展开

- 单位处理（km → kilometers）

- 标点断句

---

## 和你现在做的事情的直接关联（重点）

你现在在：

- TTS（XTTS / CosyVoice）

- audiobook

- 中英混合

- 自己做 TN pipeline

👉 **inflect = 英文 TN 的“底座库”**  
👉 **中文那部分才需要 wetext / paddlespeech / 自写规则**

如果你愿意，下一步我可以直接帮你：

- 🔹 写一个 **“中英混合 TN pipeline（inflect + 中文 TN）”**

- 🔹 对比 **inflect vs wetext 在 CosyVoice 里的角色**

- 🔹 给你一个 **audiobook 级英文 TN 模板**

你更想往哪一步走？
