`jieba` (结巴分词) is one of the most popular **Chinese text segmentation** libraries in Python.

Because written Chinese doesn’t use spaces between words, you need a tokenizer like `jieba` to split sentences into meaningful words — just like English spaces.

---

## 🧠 What Jieba Does

**Jieba = Chinese word segmentation + POS tagging + keyword extraction**

It can:

1. **Split** a sentence into words (`分词`)
    
2. **Tag** each word with its part of speech (e.g. noun, verb)
    
3. **Extract** top keywords using TF-IDF or TextRank
    
4. **Support custom dictionaries**
    

---

## ⚙️ Installation

```bash
pip install jieba
```

---

## 🧩 Basic Usage

### 1. **Accurate Mode** (default)

Splits text as precisely as possible.

```python
import jieba

text = "我来到北京清华大学"
words = jieba.cut(text)
print("/".join(words))
# Output: 我/来到/北京/清华大学
```

---

### 2. **Full Mode**

Cuts all possible words — faster but less accurate.

```python
jieba.cut(text, cut_all=True)
# Output: 我/来到/北京/清华/清华大学/华大/大学
```

---

### 3. **Search Engine Mode**

For indexing — gives finer granularity.

```python
jieba.cut_for_search(text)
# Output: 我/来到/北京/清华/华大/大学/清华大学
```

---

## 🏷️ Part-of-Speech Tagging

```python
import jieba.posseg as pseg

words = pseg.cut("我爱自然语言处理")
for w, flag in words:
    print(f"{w} ({flag})")

# Example:
# 我 (r)
# 爱 (v)
# 自然语言 (n)
# 处理 (v)
```

- `r` = pronoun
    
- `v` = verb
    
- `n` = noun
    

---

## 🔍 Keyword Extraction

### TF-IDF

```python
import jieba.analyse

text = "小明硕士毕业于中国科学院计算所，后在日本京都大学深造"
keywords = jieba.analyse.extract_tags(text, topK=3)
print(keywords)
# ['京都大学', '中国科学院', '计算所']
```

### TextRank

```python
jieba.analyse.textrank(text, topK=3)
```

---

## 🧰 Custom Dictionary

You can add your own domain-specific words:

```python
jieba.add_word("自然语言处理")
jieba.add_word("大模型")
```

Or load a file:

```python
jieba.load_userdict("mydict.txt")
# each line: word freq POS_tag
# e.g. 自然语言处理 10 n
```

---

## 🧠 Summary Table

|Feature|Function|
|---|---|
|Word segmentation|`jieba.cut(text)`|
|Full mode|`jieba.cut(text, cut_all=True)`|
|Search mode|`jieba.cut_for_search(text)`|
|POS tagging|`pseg.cut(text)`|
|Keyword extraction|`jieba.analyse.extract_tags(text)`|
|Custom dictionary|`jieba.add_word()` / `jieba.load_userdict()`|

---

Would you like me to show how `jieba` can integrate with `BeautifulSoup` — e.g. to extract and segment **Chinese text from HTML pages** automatically?