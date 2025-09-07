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

## Filters
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

## Color

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


## Capacity

在 **PygWalker** 的 UI 中，`filters` 面板中的 **capacity** 是用于调整数据点在可视化图表中的透明度的功能。通过设置透明度，可以直观地表达某些维度或数值的相对权重或显著性。

---

### **capacity 的作用**
1. **突出重点数据**：使用透明度来强调某些数据点或淡化其他数据点。
2. **处理数据密集型可视化**：对于包含大量数据点的图表（例如散点图），透明度可以帮助减少视觉混乱，揭示趋势或分布。
3. **动态筛选或探索**：结合其他筛选条件，通过透明度可以更灵活地观察数据子集。

---

### **如何在 UI 中使用 capacity**

#### **1. 添加 capacity 变量**
在 PygWalker 的界面中，你可以将一个字段（列）拖动到 **"Capacity"** 区域：

1. 在数据字段列表中选择一个字段。
2. 将该字段拖动到 **"Capacity"** 区域。
3. 图表会自动更新，将数据点的透明度与字段的值进行绑定。

---

#### **2. 支持的字段类型**
- **数值型字段（Numerical Columns）**
  - 根据数值大小调整透明度。例如，较大的值可以设置为更不透明（更显眼），较小的值更透明。
- **分类型字段（Categorical Columns）**
  - 不同类别可以被分配不同的透明度值（但一般使用数值型字段更为直观）。

---

#### **3. 调整透明度范围**
- **默认行为**：PygWalker 会自动将数值映射到透明度的一个范围（通常是 0% 到 100%）。
- **自定义范围**：
  - 点击图表中与 `Capacity` 相关的设置按钮。
  - 手动调整透明度的最小值和最大值范围。
  - 例如，设置透明度范围为 20% 到 80%，避免完全透明或完全不透明的情况。

---

#### **4. 示例**

#### **数值型字段示例**
假设有以下数据：
| Category | Value |
|----------|-------|
| A        | 10    |
| B        | 20    |
| A        | 15    |
| C        | 25    |

- 将 `Value` 字段拖动到 **Capacity**。
- 结果：`Value` 越高的数据点越不透明（更突出），`Value` 越低的数据点越透明（更淡）。

#### **分类型字段示例**
假设有以下数据：
| Category | Group |
|----------|-------|
| A        | X     |
| B        | Y     |
| A        | X     |
| C        | Z     |

- 将 `Group` 字段拖动到 **Capacity**。
- 结果：不同组的数据点会被赋予不同的透明度值。

---

### **最佳实践**
1. **结合 Color 使用**：
   - 同时使用 `Capacity` 和 `Color`，可以通过颜色和透明度的组合，更有效地传达复杂信息。
2. **避免过度使用**：
   - 如果透明度过低，可能导致数据点难以看清，建议保持一定的最低透明度。
3. **适合大规模数据**：
   - 在散点图、热力图等图表中，透明度功能特别有用，能有效缓解点重叠的问题。

---

**总结**：在 PygWalker 的 UI 中，`Capacity` 是一个强大的工具，用于通过透明度传达数据的重要性或趋势。它适合强调数据特性，尤其在处理数据点密集的图表时，非常实用。


## Size

在 **PygWalker** 的 UI 中，`filters` 面板中的 **size** 功能用于控制数据点的大小，以此来表达数据中的某些数值或类别差异。通过调整数据点的大小，可以使得可视化更具信息性，帮助用户直观地看到数据的相对差异或权重。

---

### **size 的作用**
1. **表达数值差异**：通过数据点大小的差异来反映某些数值的大小，如销售量、数量、频率等。
2. **对比不同类别**：对于分类数据，可以根据某个数值字段的大小来突出不同类别的相对重要性。
3. **优化视觉效果**：对于数据密集的图表，通过调整大小，可以避免重叠和视觉混乱。

---

### **如何在 UI 中使用 size**

#### **1. 添加 size 变量**
在 PygWalker 的界面中，你可以将一个字段（列）拖动到 **"Size"** 区域：

1. 打开 PygWalker UI 界面。
2. 在数据字段列表中选择你想要表示大小差异的字段（一般是数值型字段）。
3. 将该字段拖动到 **"Size"** 区域，图表会自动更新，根据字段的值来设置数据点的大小。

---

#### **2. 支持的字段类型**
- **数值型字段（Numerical Columns）**
  - 最常用于此功能的字段类型。数值越大，数据点的大小越大；数值越小，数据点的大小越小。
- **分类型字段（Categorical Columns）**
  - 也可以使用分类字段，但通常是为数值型字段设置大小效果更直观。

---

#### **3. 调整大小范围**
- **默认行为**：PygWalker 会根据字段值的范围自动映射数据点的大小。
- **手动调整**：
  - 在图表中点击与 `Size` 相关的设置按钮。
  - 可以自定义大小范围的最大和最小值。例如，设置数据点大小范围为 5 到 30，以避免过大或过小的数据点。

---

#### **4. 示例**

#### **数值型字段示例**
假设有以下数据：
| Category | Sales |
|----------|-------|
| A        | 100   |
| B        | 200   |
| A        | 150   |
| C        | 300   |

- 将 `Sales` 字段拖动到 **Size**。
- 结果：`Sales` 值越大的数据点显示越大，反映出销售量的差异。

#### **分类型字段示例**
假设有以下数据：
| Category | Group |
|----------|-------|
| A        | X     |
| B        | Y     |
| A        | X     |
| C        | Z     |

- 将 `Group` 字段拖动到 **Size**。
- 结果：不同组的数据点将按分类大小显示，但通常不如数值字段直观。

---

### **最佳实践**
1. **适用于散点图和气泡图**：
   - 在这些图表中，通过数据点大小可以更直观地传达数量或比例信息。
2. **结合 Color 和 Capacity 使用**：
   - 使用 `Size` 配合 `Color` 或 `Capacity`，可以增加维度，帮助用户更好地分析数据。
3. **避免过度使用**：
   - 若数据点大小差异过大，可能会影响可读性。建议限制最大和最小值，保持视觉平衡。

---

**总结**：在 PygWalker 的 UI 中，`Size` 功能是可视化中的一个重要工具，用于通过数据点大小表达数值差异或类别权重。结合其他功能使用，可以更全面地展示和分析数据特征。

## Details

在 **PygWalker** 的 UI 中，`filters` 面板中的 **details** 功能用于为图表添加一个额外的维度以分组或细化显示数据。通过 `details`，可以将数据按照某一字段进行更详细的划分，而不会直接影响图表的主轴（如 X 轴或 Y 轴）。它常用于提供更丰富的上下文信息或多层次的分析视图。

---

### **details 的作用**
1. **数据分组**：为每个数据点或图表元素提供额外的分组信息。
2. **细化分析**：在图表中区分不同的子集或分层信息。
3. **补充细节**：显示更多维度的数据，而不破坏主要的可视化布局。

---

### **如何在 UI 中使用 details**

#### **1. 添加 details 字段**
在 PygWalker 的界面中：
1. 在数据字段列表中选择一个字段。
2. 将字段拖动到 **"Details"** 区域。
3. 图表会根据添加的字段自动更新，显示额外的分组或详细信息。

---

#### **2. 支持的字段类型**
- **分类字段（Categorical Columns）**：
  - 常用于分组数据。例如，可以按产品类别、地区等划分数据。
- **数值字段（Numerical Columns）**：
  - 在某些图表类型中，可以为数据点添加额外的数值信息。
- **日期字段（Date Columns）**：
  - 可以按日期或时间段细分数据。

---

#### **3. 示例**

##### **示例 1：分组散点图**
假设有以下数据：
| Category | Sales | Region |
|----------|-------|--------|
| A        | 100   | East   |
| B        | 200   | West   |
| A        | 150   | East   |
| C        | 300   | South  |

**操作**：
- 将 `Region` 字段拖动到 **"Details"** 区域。

**结果**：
- 散点图中每个点除了表示 `Sales` 和 `Category` 的信息外，还会显示该点属于哪个 `Region`。

---

##### **示例 2：堆叠柱状图**
假设有以下数据：
| Product | Revenue | Year |
|---------|---------|------|
| A       | 1000    | 2021 |
| B       | 1500    | 2021 |
| A       | 1200    | 2022 |
| B       | 1800    | 2022 |

**操作**：
- 将 `Year` 字段拖动到 **"Details"** 区域。

**结果**：
- 柱状图显示每个 `Product` 的 `Revenue`，并按 `Year` 细分。

---

#### **4. 配合其他功能使用**
- **Color 和 Details**：
  - 将一个字段拖到 `Color`，另一个字段拖到 `Details`，可以通过颜色和分组的组合展示多维信息。
- **Size 和 Details**：
  - 将数值字段拖到 `Size`，分类字段拖到 `Details`，数据点不仅大小不同，还会显示分组信息。

---

#### **5. 使用场景**
- **多维度分析**：
  在主图表中无法直接表达的数据维度可以通过 `details` 添加，避免混乱。
- **分层聚合**：
  对数据进行层次化分析，比如同时按国家、城市进行划分。
- **补充上下文信息**：
  为图表提供更丰富的注解或细节。

---

**总结**：  
`details` 是一个强大的辅助功能，用于在不改变主图表结构的情况下，添加更丰富的分组和细节信息。它适合在多维数据中挖掘更多模式，同时保持图表的整洁和清晰度。

## Text

在 **PygWalker** 的 UI 中，`filters` 面板中的 **text** 功能用于对文本类型的数据字段进行筛选或细化分析。通过 `text`，你可以指定包含、开头、结尾或完全匹配的文本字符串，以过滤数据或细化图表中的显示内容。

---

### **text 的作用**
1. **快速搜索特定文本**：根据指定的关键字过滤数据。
2. **灵活的文本匹配规则**：支持部分匹配、前缀匹配、后缀匹配等方式。
3. **动态探索数据**：结合其他筛选条件，更精准地查看子集数据。

---

### **如何在 UI 中使用 text**

#### **1. 添加 text 字段到筛选器**
1. 在 PygWalker UI 界面中，找到你的文本类型字段（如 `Name` 或 `Category`）。
2. 将该字段拖动到 `filters` 面板中的 **text** 区域。
3. 在弹出的文本输入框中输入筛选条件。

---

#### **2. 常见的筛选方式**
在 **text** 筛选框中，可以使用以下几种方式来筛选数据：

##### **1. 包含关键字**
输入关键字，过滤包含该关键字的行：
- **示例**：输入 `Apple`，筛选出所有包含 `Apple` 的记录（如 `Apple Inc.` 或 `Green Apple`）。

##### **2. 匹配前缀**
使用前缀匹配，显示以指定文本开头的数据：
- **示例**：输入 `App`，筛选出 `Apple` 和 `Application`。

##### **3. 匹配后缀**
使用后缀匹配，显示以指定文本结尾的数据：
- **示例**：输入 `Inc.`，筛选出 `Apple Inc.` 和 `Google Inc.`。

##### **4. 完全匹配**
使用等值匹配，只显示完全相等的数据：
- **示例**：输入 `Apple`，只会显示字段值为 `Apple` 的行。

---

### **3. 示例场景**

#### 示例 1：筛选产品名称
假设有以下数据：
| Product       | Sales |
|---------------|-------|
| Apple         | 100   |
| Banana        | 200   |
| Green Apple   | 150   |
| Apple Inc.    | 300   |

1. 将 `Product` 字段拖动到 **text** 筛选区域。
2. 输入 `Apple`。
3. 结果：仅显示包含 `Apple` 的记录（`Apple`、`Green Apple`、`Apple Inc.`）。

---

#### 示例 2：筛选分类字段
假设有以下数据：
| Category  | Quantity |
|-----------|----------|
| Fruits    | 500      |
| Vegetables| 300      |
| Beverages | 200      |

1. 将 `Category` 字段拖动到 **text** 筛选区域。
2. 输入 `Veg`。
3. 结果：仅显示 `Vegetables`。

---

### **4. 使用场景**
- **搜索特定记录**：在大量数据中快速找到匹配条件的子集。
- **动态探索**：与其他筛选器（如日期或数值）结合，筛选更细化的结果。
- **文本清理检查**：用于查找包含某些字符或模式的异常数据。

---

### **5. 注意事项**
- **大小写敏感性**：一般情况下，筛选是大小写不敏感的。如果需要区分大小写，需借助 PygWalker 其他高级功能。
- **性能问题**：对于非常大的数据集，频繁的文本筛选可能会增加加载时间。
- **与其他 filters 配合使用**：`text` 筛选器可以和其他筛选器（如 `numeric`、`category` 等）组合使用，实现多维度数据探索。

---

**总结**：  
PygWalker 中的 `text` 功能是一个强大的文本数据筛选工具，适用于任何涉及文本数据的分析任务。无论是搜索特定关键字、查找模式，还是进行灵活的部分匹配，`text` 功能都能提供直观且强大的支持。


# Pandas快速入门

## 从数据库中读取数据到dataframe

在 **Pandas** 中，可以通过使用 `read_sql` 或 `read_sql_query` 方法，从数据库中读取数据到 DataFrame。以下是完整的步骤和常见用法：

---

### **1. 安装必要的库**
首先，确保安装了以下库：
- **pandas**：用于数据操作。
- **SQLAlchemy** 或 **sqlite3**：用于连接数据库。

安装命令：
```bash
pip install pandas sqlalchemy
```

如果需要特定数据库的支持（如 MySQL、PostgreSQL 等），需要安装对应的驱动程序：
- MySQL：`pip install pymysql`
- PostgreSQL：`pip install psycopg2`

---

### **2. 数据库连接**
Pandas 支持多种数据库连接方式。以下是一些常见的数据库连接示例：

#### SQLite
```python
import sqlite3
connection = sqlite3.connect("example.db")
```

#### MySQL
```python
from sqlalchemy import create_engine
engine = create_engine("mysql+pymysql://username:password@host:port/database")
```

#### PostgreSQL
```python
from sqlalchemy import create_engine
engine = create_engine("postgresql+psycopg2://username:password@host:port/database")
```

---

### **3. 从数据库读取数据到 DataFrame**

#### 使用 SQL 查询读取数据
```python
import pandas as pd

# 使用 SQL 查询
query = "SELECT * FROM table_name"
df = pd.read_sql_query(query, con=engine)
```

#### 直接读取整个表
```python
df = pd.read_sql_table("table_name", con=engine)
```

**注意**：`read_sql_table` 需要 SQLAlchemy 的连接对象。

---

### **4. 示例**

#### 示例：SQLite 数据库
```python
import sqlite3
import pandas as pd

# 创建数据库连接
connection = sqlite3.connect("example.db")

# 使用 SQL 查询读取数据
query = "SELECT * FROM users"
df = pd.read_sql_query(query, con=connection)

print(df.head())
```

#### 示例：MySQL 数据库
```python
from sqlalchemy import create_engine
import pandas as pd

# 创建 MySQL 数据库连接
engine = create_engine("mysql+pymysql://root:password@localhost:3306/test_db")

# 读取数据到 DataFrame
query = "SELECT id, name, age FROM users WHERE age > 18"
df = pd.read_sql_query(query, con=engine)

print(df.head())
```

---

### **5. 参数调整**
- **指定列类型**：
  ```python
  df = pd.read_sql_query(query, con=engine, dtype={"id": int, "name": str})
  ```
- **分页读取**（避免一次性读取大表）：
  ```python
  chunks = pd.read_sql_query(query, con=engine, chunksize=1000)
  for chunk in chunks:
      print(chunk.head())
  ```

---

### **6. 写入数据库**
如果需要将 DataFrame 写入数据库，可以使用 `to_sql` 方法：
```python
df.to_sql("new_table", con=engine, if_exists="replace", index=False)
```

---

**总结**：通过 Pandas 和数据库的结合，可以方便地将数据库中的数据导入到 DataFrame 中进行分析。选择适合的连接方式和驱动程序，根据需求灵活使用 `read_sql_query` 或 `read_sql_table` 方法即可。


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


