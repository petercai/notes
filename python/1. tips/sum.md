# sum

这是一个非常巧妙的 Python 技巧！让我详细解释一下为什么使用 `sum(1 for ...)`。

## 为什么用 `sum(1 for ...)`？

这是一个**高效计数**的惯用写法，用来统计满足条件的元素数量。

### 分解理解

```python
# 原代码
sentences_count = sum(1 for row in sentences_of_chapter if row.strip() not in TTS_SML.values())
```

这个表达式可以分解为：

1. **生成器表达式**: `(1 for row in sentences_of_chapter if row.strip() not in TTS_SML.values())`
2. **对每个满足条件的元素生成数字 1**
3. **sum() 把所有的 1 加起来**，得到总数

### 等价的代码

```python
# 方法1: 使用 sum(1 for ...)（最优雅）
sentences_count = sum(1 for row in sentences_of_chapter 
                      if row.strip() not in TTS_SML.values())

# 方法2: 传统 for 循环
sentences_count = 0
for row in sentences_of_chapter:
    if row.strip() not in TTS_SML.values():
        sentences_count += 1

# 方法3: 使用列表推导式（不推荐，占内存）
sentences_count = len([row for row in sentences_of_chapter 
                       if row.strip() not in TTS_SML.values()])

# 方法4: 使用 filter
sentences_count = len(list(filter(
    lambda row: row.strip() not in TTS_SML.values(), 
    sentences_of_chapter
)))
```

## 为什么 `sum(1 for ...)` 是最佳选择？

### 1. **内存效率高**

```python
# ❌ 占用内存 - 创建了完整的列表
count = len([1 for row in data if condition])

# ✅ 不占内存 - 使用生成器，逐个产生
count = sum(1 for row in data if condition)
```

生成器表达式 `(1 for ...)` 是**惰性求值**的，每次只产生一个 1，不会创建完整的列表。

### 2. **代码简洁优雅**

一行代码就完成了过滤和计数，可读性强。

### 3. **性能好**

```python
import time

data = range(1000000)

# 测试 sum(1 for ...)
start = time.time()
count1 = sum(1 for x in data if x % 2 == 0)
print(f"sum(1 for ...): {time.time() - start:.4f}秒")

# 测试 len([...])
start = time.time()
count2 = len([x for x in data if x % 2 == 0])
print(f"len([...]): {time.time() - start:.4f}秒")
```

生成器方式通常更快，因为不需要构建中间列表。

## 工作原理详解

```python
# 假设有这样的数据
sentences_of_chapter = ["句子1", "  ", "句子2", "SML标记", "句子3"]
TTS_SML = {"pause": "SML标记", "speed": "另一个标记"}

# 生成器的工作过程：
# 第1次迭代: "句子1".strip() not in ["SML标记", "另一个标记"] → True → 产生 1
# 第2次迭代: "  ".strip() not in ["SML标记", "另一个标记"] → True → 产生 1
# 第3次迭代: "句子2".strip() not in ["SML标记", "另一个标记"] → True → 产生 1
# 第4次迭代: "SML标记".strip() not in ["SML标记", "另一个标记"] → False → 跳过
# 第5次迭代: "句子3".strip() not in ["SML标记", "另一个标记"] → True → 产生 1

# sum() 计算: 1 + 1 + 1 + 1 = 4
sentences_count = 4
```

## 其他使用场景

```python
# 统计偶数个数
even_count = sum(1 for x in range(100) if x % 2 == 0)

# 统计字符串中大写字母个数
uppercase_count = sum(1 for c in "Hello World" if c.isupper())

# 统计列表中大于 10 的元素个数
large_nums = sum(1 for n in numbers if n > 10)

# 统计文件行数
line_count = sum(1 for line in open('file.txt'))

# 统计满足多个条件的元素
count = sum(1 for item in items 
            if item.price > 100 and item.stock > 0)
```

## 什么时候不用 `sum(1 for ...)`？

```python
# 如果只是统计总数，直接用 len()
total = len(my_list)  # ✅ 简单直接

# 如果需要保留元素，用列表推导
filtered = [x for x in data if x > 10]  # ✅ 需要实际的元素
count = len(filtered)

# 如果需要元素本身，不要用 sum(1 for ...)
# ❌ 错误
items = sum(1 for x in data if x > 10)  # 这只给你数量，不是元素

# ✅ 正确
items = [x for x in data if x > 10]  # 这给你实际的元素
```

## 总结

`sum(1 for ...)` 是 Python 中统计满足条件的元素数量的**标准惯用法**：
- ✅ 内存效率高（生成器）
- ✅ 代码简洁优雅
- ✅ 性能好
- ✅ Pythonic（符合 Python 风格）

这就是为什么你会在很多 Python 代码中看到这种写法！