# Python 教程 - 集合
集合（set）可以看作是“只有键，没有值”的字典。在一个集合中，所有的键都是唯一的，因为集合不会包含重复的数据。这个特性使得集合经常用来去除重复的元素，或者判断元素之间是否有交集、并集或差集等关联性。

创建集合
----

集合由数字、字符串或布尔值组成，同一个集合中的元素可以是不同的类型。创建集合有两种方式：

### set()

使用 `set()` 可以创建空集合，或者将列表、元组、字符串或字典转换成集合。使用方法为 `set(要变成集合的元素)`。

> 如果创建时出现重复的元素，只会保留一个。如果是字典，只会保留键。如果是布尔值，True 等同于 1，False 等同于 0。

```dart
a = set()
b = set([1,2,3,4,5,1,2,3,4,5])
c = set({'x':1,'y':2,'z':3})
d = set('hello')
print(a)   # set()
print(b)   # {1, 2, 3, 4, 5}  只留下不重复的部分
print(c)   # {'x', 'y', 'z'}  如果是字典，只保留键
print(d)   # {'l', 'o', 'h', 'e'}  只留下不重复的部分


```

### 大括号 {}

如果不是空集合，可以使用 `{元素}` 创建集合（单纯写大括号，会变成“空字典”）。

```css
a = {0,1,2,3,'a','b',False}
print(a)  # {0, 1, 2, 3, 'a', 'b'}  False 等同于 0，所以只保留 0


```

### 添加元素

使用 `集合.add(元素)` 可以将某个元素加入到集合中。下面的代码会将 `x` 和 `y` 两个字符串加入到 `a` 集合。

```csharp
a = {0,1,2,3,4,5}
a.add('x')
a.add('y')
print(a)   # {0, 1, 2, 3, 4, 5, 'x', 'y'}


```

移除元素
----

有两种方法可以移除集合里的某个元素：

### remove()

使用 `集合.remove(元素)` 可以将指定的元素移除，但如果该元素不存在，就会执行错误。

```lua
a = {0,1,2,3,'x','y','z'}
a.remove('x')
print(a)   # {0, 1, 2, 3, 'y', 'z'}


```

### discard()

如果不希望在移除元素时发生执行错误的情况，可以使用 `集合.discard(元素)` 移除指定元素。

```bash
a = {0,1,2,3,'x','y','z'}
a.discard('x')
a.discard('a')   
print(a)         


```

集合运算
----

### 交集、并集、差集、对称差集

集合有四种运算类型，分别是“交集、并集、差集、对称差集”。通过下图可以了解四种运算类型。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0bb3fb60b0a49feae9a40374f694592~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=983&s=54518&e=jpg&b=fefefe)

使用集合运算有两种方法，一种是使用特定的方法，另一种是使用“符号”（集合运算符）。

| 集合 | 方法 | 运算符 |
| --- | --- | --- |
| 交集 | a.intersection(b) | a&b |
| 并集 | a.union(b) | a｜b |
| 差集 | a.difference(b) | a-b |
| 对称差集 | a.symmetricdifference(b) | a^b |

下面的代码对 `a` 和 `b` 进行集合运算。

```bash
a = {1,2,3,4,5}
b = {3,4,5,6,7}


print(a.intersection(b))   
print(a&b)                 

print(a.union(b))          
print(a|b)                 

print(a.difference(b))     
print(a-b)                 

print(a.symmetricdifference(b))  
print(a^b)                        


```

### 子集合、超集合

假设有 A、B 两个集合，超集合和子集合的关系可以参考下图：

| 集合 | 说明 |
| --- | --- |
| 超集合 | A 完全包含 B，A 和 B 所包含的元素可能完全相同 |
| 真超集合 | A 完全包含 B，且具有 B 没有的的元素 |
| 子集合 | B 完全被 A 包含，A 和 B 所包含的元素可能完全相同 |
| 真子集合 | B 完全被 A 包含，且 A 具有 B 没有的的元素 |

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39a5ebcf9c3a483bbab69c7e7c27b8f2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=613&s=59263&e=jpg&b=fefefe)

下面的代码中有四个集合，使用「大于、小于、等于」可以表示彼此的关系，从打印出的结果可以看到各个集合之间的关系。

```css
a = {1,2,3,4,5,6,7}
b = {3,4,5,6,7}
c = {1,2,3,4,5,6,7}
d = {6,7,8,9}

print(a<=a)   # True 自己是自己的子集合
print(b<=a)   # True b 是 a 的子集合
print(b<a)    # True b 也是 a 的真子集合 ( 因为没有等于，完全包含 )
print(c<=a)   # True c 是 a 的子集合
print(a<=c)   # True a 也是 c 的子集合
print(d<a)    # False d 和 a 没有子集合或超集合关系


```

此外，使用「`b.issubset(a)`」方法可以检查 b 是否为 a 的子集合、「`a.issuperset(b)`」方法可以检查 a 是否为 b 的超集合。

```css
a = {1,2,3,4,5,6,7}
b = {3,4,5,6,7}
c = {1,2,3,4,5,6,7}
d = {6,7,8,9}

print(b.issubset(a))     # True b 是 a 的子集合
print(a.issuperset(b))   # True a 是 b 的超集合
print(c.issubset(a))     # True c 是 a 的子集合
print(d.issubset(a))     # Fasle d 不是 a 的子集合


```

获取集合长度
------

使用 `len(集合)` 可以返回某个集合的长度（有多少个元素）。

```ini
a = {1,2,3,4,5,6,7}
print(len(a))   


```

检查元素是否存在
--------

使用 `元素 in 集合` 可以检查集合中是否存在某个元素，如果存在则返回 True，不存在则返回 False（使用 `元素 not in 集合` 可以判断不存在）。

```bash
a = {'a','b','c','d',1,2,3}
print('b' in a)    
print(2 in a)      
print(99 in a)     


```