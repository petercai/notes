# 安装Jupyter Notebook

目前Jupyter有两个版本，Jupyter Lab和Jupyter Notebook，都是基于Web的交互式开发环境，其中Jupyter Lab为最新一代的产品。推荐安装Jupyter Notebook，因为它相对比较稳定、简洁，而且完全可以满足我们的需要。

```
# 安装Jupyter Notebook，使用清华大学下载源加快下载速度
pip install jupyter 
jupyter notebook


# 安装 Jupyter Lab 的命令如下
pip install jupyterlab
jupyter lab
```

一些快捷键，尽量使用这些快捷键来操作以提高效率。
Tab：代码提示（输入部分代码后按Tab键）。
Shift+Enter：执行本行并定位到新增的行。
Shift+Tab（1～3次）：查看函数方法说明（光标在函数上按住Shift再按Tab键一到三次）。
D D：双击D键删除本行。
A / B：向上/向下增加一行。
M / R：Markdown / 代码模式。

# PyGWalker

## filters
在 **PygWalker** 中，`filters` 是用于设置数据过滤条件的一个功能，能够帮助你快速对数据进行筛选和分析。下面是一个关于如何使用 `filters` 的简单说明和示例。

---

### **什么是 filters**
`filters` 用于指定数据的初始过滤条件，当你在 Jupyter Notebook 或其他支持 PygWalker 的环境中加载数据时，数据会根据 `filters` 中的条件进行筛选并显示。

---

### **如何设置 filters**

#### 基本语法
当你调用 `pygwalker.walk` 函数时，可以通过 `filters` 参数传入一个包含过滤条件的字典。格式如下：
```python
filters = {
    "column_name": {
        "type": "range" or "value",
        "value": [...],  # 筛选值，取决于类型
    }
}
```

#### 例子

假设我们有一个包含以下数据的 Pandas DataFrame：

```python
import pandas as pd
import pygwalker as pyg

# 示例数据
data = pd.DataFrame({
    "Category": ["A", "B", "A", "C", "B"],
    "Value": [10, 20, 15, 25, 30],
    "Date": pd.date_range("2023-01-01", periods=5),
})

# 设置 filters 只显示 Category 为 "A" 的数据
pyg.walk(data, filters={
    "Category": {
        "type": "value",
        "value": ["A"]
    }
})
```

此代码会初始化 PygWalker，只显示 `Category` 列为 `A` 的数据。

---

#### 使用 range 类型的 filters
如果希望对数值或日期列设置范围过滤，可以使用 `range` 类型：

```python
pyg.walk(data, filters={
    "Value": {
        "type": "range",
        "value": [10, 20]  # 只显示 Value 在 10 到 20 之间的数据
    }
})
```

---

#### 动态交互过滤
即使没有预设 `filters`，PygWalker 也允许用户在界面中通过拖拽或点选来动态添加和修改过滤条件。这种方式适合探索数据。

---

### **常见问题**
1. **支持的列类型**
   - `filters` 支持对分类列、数值列和日期列进行筛选。
2. **多个条件的组合**
   - 暂时不支持直接通过 `filters` 参数定义多个列条件的逻辑组合（如 AND/OR），但你可以先在 Pandas 中对 DataFrame 进行处理。

---

#### **总结**
`filters` 是 PygWalker 中一个强大的数据预处理功能，通过设定初始条件，可以快速聚焦于感兴趣的子集，同时保留动态探索的能力。

## color
在 **PygWalker** 的 **UI 界面**中，`filters` 面板中的 **color** 功能主要用于数据可视化时，通过颜色来直观区分不同类别或数值范围的数据。以下是关于 **color** 功能的详细说明：

---

### **color 的作用**
- **用于区分数据类别**：通过为不同的分类值分配颜色，可以清晰地区分数据组。
- **反映数值大小**：对于数值型数据，`color` 功能可以生成颜色渐变，用以表示数据的数值高低。

---

### **如何在 UI 中使用 color**

#### **1. 添加 color 变量**
在 PygWalker 的界面中，你可以将一个字段（列）拖动到 **"Color"** 区域：

1. 打开 PygWalker UI 界面。
2. 在数据字段列表中选择你感兴趣的列（例如 "Category" 或 "Value"）。
3. 将该列拖动到 **"Color"** 区域，图表会自动更新，应用颜色映射。

---

#### **2. 支持的字段类型**
- **分类字段（Categorical Columns）**
  - 每个类别会被分配不同的颜色。例如，列 `"Category"` 的值为 `A, B, C`，则会有三种不同的颜色。
  - 常用于条形图、饼图、散点图等。
- **数值字段（Numerical Columns）**
  - 使用颜色梯度来表示数值的大小。例如，数值从 `10` 到 `100`，颜色会从浅到深或从冷到暖渐变。
  - 常用于散点图、热力图等。

---

#### **3. 配置和调整颜色映射**
- **更改配色方案**
  - 在界面中点击 **Color** 图例旁的设置按钮。
  - 选择预设的配色方案（如 "Sequential"、"Diverging" 或 "Categorical"）。
- **手动分配颜色**
  - 对于分类变量，你可以手动为每个类别选择颜色。
- **调整颜色范围**
  - 对于数值变量，可以拖动颜色范围的最小值和最大值滑块，设置梯度的区间。

---

#### **4. 示例**

#### **分类字段示例**
假设有以下数据：
| Category | Value |
|----------|-------|
| A        | 10    |
| B        | 20    |
| A        | 15    |
| C        | 25    |

- 拖动 `Category` 到 **Color** 区域。
- 结果：每个类别 `A, B, C` 会显示不同颜色。

#### **数值字段示例**
假设有以下数据：
| Date       | Value |
|------------|-------|
| 2023-01-01 | 10    |
| 2023-01-02 | 20    |
| 2023-01-03 | 15    |
| 2023-01-04 | 25    |

- 拖动 `Value` 到 **Color** 区域。
- 结果：颜色梯度表示 `Value` 的大小，10 为浅色，25 为深色。

---

#### **5. 动态交互**
在应用了 color 后：
- 可以在图表中点击某个颜色块，从而对数据进行进一步的筛选。
- 颜色图例支持动态调整范围（尤其对数值型字段非常有用）。

---

**总结**：PygWalker 的 `color` 功能是一个非常直观的工具，既适合分类数据的分组展示，也适合数值数据的梯度分析，帮助用户更快洞察数据特征。




# Pandas快速入门
## 使用Pandas过程中会用到一些专门的库:
Excel处理相关包：xlrd、openpyxl、XlsxWriter
解析网页包：Requests、lxml、html5lib、Beautiful Soup 4
可视化包：Matplotlib、seaborn、Plotly、Bokeh
计算包：SciPy、statsmodels
```
pip install pandas xlrd openpyxl xlsxwriter requests lxml html5lib BeautifulSoup4 matplotlib seaborn plotly bokeh scipy statsmodels
```

## 读取数据

```
# 以下两种效果一样，如果是网址，它会自动将数据下载到内存
df = pd.read_excel('https://www.gairuo.com/file/data/dataset/team.xlsx')
df = pd.read_excel('team.xlsx') # 文件在notebook文件同一目录下
# 如果是CSV，使用pd.read_csv()，还支持很多类型的数据读取
```

## 查看数据

```
df.head() # 查看前5条，括号里可以写明你想看的条数
df.tail() # 查看尾部5条
df.sample(5) # 随机查看5条
```

## 验证数据

```
df.shape # (100, 6) 查看行数和列数
df.info() # 查看索引、数据类型和内存信息
df.describe() # 查看数值型列的汇总统计
df.dtypes # 查看各字段类型
df.axes # 显示数据行和列名
df.columns # 列名
```


## 建立索引

```
df.set_index('name', inplace=True) # 建立索引并生效

```

## 数据选取

```
# 查看指定列
df['Q1']
df.Q1 # 同上，如果列名符合Python变量名要求，可使用

# 选择多列
df[['team', 'Q1']] # 只看这两列，注意括号
df.loc[:, ['team', 'Q1']] # 和上一行效果一样
```

df.loc[x, y]是一个非常强大的数据选择函数，其中x代表行，y代表
列，行和列都支持条件表达式，也支持类似列表那样的切片

```
# 选择行
# 用指定索引选取
df[df.index == 'Liver'] # 指定姓名
# 用自然索引选择，类似列表的切片
df[0:3] # 取前三行
df[0:10:2] # 在前10个中每两个取一个
df.iloc[:10,:] # 前10个

# 指定行和列
df.loc['Ben', 'Q1':'Q4'] # 只看Ben的四个季度成绩
df.loc['Eorge':'Alexander', 'team':'Q4'] # 指定行区间

# 条件选择
# 单一条件
df[df.Q1 > 90] # Q1列大于90的
df[df.team == 'C'] # team列为'C'的
df[df.index == 'Oscar'] # 指定索引即原数据中的name
# 组合条件
df[(df['Q1'] > 90) & (df['team'] == 'C')] # and关系
df[df['team'] == 'C'].loc[df.Q1>90] # 多重筛选

# 排序
df.sort_values(by='Q1') # 按Q1列数据升序排列
df.sort_values(by='Q1', ascending=False) # 降序
df.sort_values(['team', 'Q1'], ascending=[True, False]) # team升序，Q1降序

# 分组聚合
df.groupby('team').sum() # 按团队分组对应列相加
df.groupby('team').mean() # 按团队分组对应列求平均
# 不同列不同的计算方法
df.groupby('team').agg({'Q1': sum, # 总和
		'Q2': 'count', # 总数
		'Q3':'mean', # 平均
		'Q4': max}) # 最大值

# 数据转换
df.groupby('team').sum().T
df.groupby('team').sum().stack()
df.groupby('team').sum().unstack()

# 增加列
df['one'] = 1 # 增加一个固定值的列
df['total'] = df.Q1 + df.Q2 + df.Q3 + df.Q4 # 增加总成绩列
# 将计算得来的结果赋值给新列
df['total'] = df.loc[:,'Q1':'Q4'].apply(lambda x:sum(x), axis=1)
df['total'] = df.sum(axis=1) # 可以把所有为数字的列相加
df['avg'] = df.total/4 # 增加平均成绩列

# 统计分析
df.mean() # 返回所有列的均值
df.mean(1) # 返回所有行的均值，下同
df.corr() # 返回列与列之间的相关系数
df.count() # 返回每一列中的非空值的个数
df.max() # 返回每一列的最大值
df.min() # 返回每一列的最小值
df.median() # 返回每一列的中位数
df.std() # 返回每一列的标准差
df.var() # 方差
s.mode() # 众数

# 导出
df.to_excel('team-done.xlsx') # 导出 Excel文件
df.to_csv('team-done.csv') # 导出 CSV文件
```



# 数据结构

## Python的数据结构

- 不可变数据：Number（数字）、String（字符串）、Tuple（元组）
- 可变数据：List（列表）、Dictionary（字典）、Set（集合）

```
# type()函数查看数据的类型
type(123) # 返回int
>>> int
a = "Hello"
type(a) # 返回str
>>> str

# 用isinstance来判断数据是不是指定的类型
isinstance(123, int) # 123是不是整型值
>>> True
isinstance('123', int)
>>> False
isinstance(True, bool)
>>> True
```

### 数字

```
x = 1 # int, 整型
y = 1.2 # float, 浮点
z = 1j # complex, 复数


a = 10
b = 21
# 数值计算
a + b # 31
a - b # -11
a * b # 210
b / a # 2.1
a ** b # 表示10的21次幂
b % a # 1 （取余）
# 地板除，相除后只保留整数部分，即向下取整
# 但如果其中一个操作数为负数，则取负无穷大方向距离结果最近的整数
9//2 # 4
9.0//2.0 # 4.0
-11//3 # -4
-11.0//3 # -4.0

```


