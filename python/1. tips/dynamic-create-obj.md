# create dynamically

在Python中，有几种方式可以动态地为对象实例创建属性并赋值：

## 1. 使用 `setattr()` 函数

```python
class MyClass:
    pass

obj = MyClass()

# 动态添加属性
setattr(obj, 'name', 'Alice')
setattr(obj, 'age', 25)
setattr(obj, 'city', 'Beijing')

# 访问属性
print(obj.name)  # Alice
print(obj.age)   # 25
```

## 2. 直接使用点号赋值

```python
class MyClass:
    pass

obj = MyClass()

# 直接添加属性
obj.name = 'Bob'
obj.age = 30
obj.email = 'bob@example.com'

print(obj.name)  # Bob
```

## 3. 在 `__init__` 中动态设置

```python
class DynamicObject:
    def __init__(self, **kwargs):
        for key, value in kwargs.items():
            setattr(self, key, value)

# 使用时传入参数
obj = DynamicObject(name='Charlie', age=35, country='China')
print(obj.name)     # Charlie
print(obj.country)  # China
```

## 4. 使用 `types.SimpleNamespace`

这是Python标准库提供的一个轻量级选项：

```python
from types import SimpleNamespace

obj = SimpleNamespace()
obj.name = 'David'
obj.age = 40

# 或者直接初始化
obj2 = SimpleNamespace(name='Eve', age=28, job='Engineer')
print(obj2.name)  # Eve
```

## 5. 批量动态添加属性

```python
class MyClass:
    pass

obj = MyClass()

# 从字典批量添加
attributes = {
    'username': 'user123',
    'score': 95,
    'level': 'advanced'
}

for key, value in attributes.items():
    setattr(obj, key, value)

print(obj.username)  # user123
print(obj.score)     # 95
```

这些方法都可以让你动态地创建对象属性，而不需要使用字典的键值访问方式。`setattr()` 是最灵活的方式，特别是当属性名称本身是动态生成的字符串时。

# read-only instance


对于创建只读属性的对象实例，**`types.SimpleNamespace` 配合 `__setattr__` 重写**或者**使用 `@dataclass(frozen=True)`** 是最合适的方法。

## 推荐方案 1: 使用 `dataclass` (frozen=True)

这是最Pythonic和简洁的方式：

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class ReadOnlyObject:
    name: str
    age: int
    city: str

# 创建实例
obj = ReadOnlyObject(name='Alice', age=25, city='Beijing')

print(obj.name)  # Alice
print(obj.age)   # 25

# 尝试修改会抛出错误
# obj.name = 'Bob'  # FrozenInstanceError
```

### 动态创建 frozen dataclass：

```python
from dataclasses import dataclass, make_dataclass

# 动态生成只读类
ReadOnlyClass = make_dataclass(
    'ReadOnlyClass',
    [('name', str), ('age', int), ('email', str)],
    frozen=True
)

obj = ReadOnlyClass(name='Charlie', age=30, email='charlie@example.com')
print(obj.name)  # Charlie

# obj.age = 35  # 报错：FrozenInstanceError
```

## 推荐方案 2: 自定义类重写 `__setattr__`

如果需要更灵活的动态属性创建：

```python
class ReadOnlyObject:
    _initialized = False
    
    def __init__(self, **kwargs):
        for key, value in kwargs.items():
            object.__setattr__(self, key, value)
        object.__setattr__(self, '_initialized', True)
    
    def __setattr__(self, key, value):
        if self._initialized:
            raise AttributeError(f"Cannot modify attribute '{key}' - object is read-only")
        object.__setattr__(self, key, value)

# 使用
obj = ReadOnlyObject(name='David', age=40, score=95)
print(obj.name)   # David
print(obj.score)  # 95

# obj.name = 'Eve'  # 报错：Cannot modify attribute
```

## 推荐方案 3: 使用 `namedtuple`

如果属性名称事先已知或可以动态生成：

```python
from collections import namedtuple

# 动态创建
Person = namedtuple('Person', ['name', 'age', 'city'])
obj = Person(name='Frank', age=35, city='Shanghai')

print(obj.name)  # Frank
print(obj.age)   # 35

# obj.name = 'George'  # 报错：AttributeError
```

### 从字典动态创建：

```python
from collections import namedtuple

data = {'name': 'Helen', 'age': 28, 'job': 'Engineer'}

# 动态生成类名和字段
ReadOnlyObj = namedtuple('ReadOnlyObj', data.keys())
obj = ReadOnlyObj(**data)

print(obj.name)  # Helen
print(obj.job)   # Engineer
```

## 比较和建议

| 方案 | 优点 | 适用场景 |
|------|------|---------|
| **dataclass(frozen=True)** | 最简洁，类型提示友好，性能好 | 属性类型已知，推荐首选 |
| **自定义__setattr__** | 最灵活，完全控制行为 | 需要自定义验证逻辑 |
| **namedtuple** | 轻量级，内存效率高 | 简单数据结构，属性较少 |

**我的推荐**：如果是现代Python项目（3.7+），优先使用 **`@dataclass(frozen=True)`**，它既简洁又强大，还支持类型提示。如果需要完全动态的属性创建，使用**自定义 `__setattr__`** 方案。