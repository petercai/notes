# Python教程 - 内置数据计算函数

常用操作方法
------

| 方法 | 参数 | 说明 |
| --- | --- | --- |
| int() | x | 将 x 转换为整数。 |
| float() | x | 将 x 转换为浮点数。 |
| abs() | x | 返回 x 的绝对值。 |
| divmod() | x, y | 返回 x 除以 y 的商和余数。 |
| max() | iter | 返回可迭代对象 iter 中的最大值。 |
| min() | iter | 返回可迭代对象 iter 中的最小值。 |
| pow() | x, y, z | 返回“x 的 y 次方”或“x 的 y 次方除以 z 的余数”。 |
| round() | x, y | 返回四舍五入后的 x，y 表示四舍五入的小数点位数。 |
| sum() | iter, y | 返回序列或元组的数值与 y 的总和。 |
| complex | x, y | 返回“x + yj”的复数形式。 |
| bool() | x | 将参数 x 转换为布尔值 False 或 True。 |

int(x, base)
------------

int(x)可以将数字或字符串参数转换为整数类型。如果数字有小数点，会无条件舍去小数点的数字。如果 x 为空，则返回 0；如果是 True 和 False 则转换为 1 和 0。

```scss
a = int('123')
b = int(123.999)
c = int()
d = int(True)
e = int(False)
print(a)    # 123
print(b)    # 123
print(c)    # 0
print(d)    # 1
print(e)    # 0


```

int(x, base)中的 base 参数默认为 10 进制，设置为 base=8 为八进制整数，base=2 为二进制整数，base=16 则是十六进制整数。

```ini
a = int('123')
b = int('123', base=8)
c = int('123', base=16)
print(a)    
print(b)    
print(c)    


```

float(x)
--------

float(x)可以将整数或字符串参数转换为浮点数类型（带有小数点的数字）。如果数字为整数，会自动在后方加上“.0”。如果 x 为空，则返回 0.0；如果是 True 和 False 则转换为 1.0 和 0.0。

```scss
a = float('123')
b = float(123)
c = float()
d = int(True)
e = int(False)
print(a)    # 123.0
print(b)    # 123.0
print(c)    # 0.0
print(d)    # 1.0
print(e)    # 0.0


```

在 Python 中表示“无穷大”或“NaN（非数字）”的数值，可以使用“float('inf')”或“float('nan')”。

```bash
a = float('inf')
b = float('-inf')
c = float('nan')
print(a)   
print(b)   
print(c)   


```

abs(x)
------

abs(x)可以返回 x 的“绝对值”。

```scss
a = abs(-123)
print(a)   # 123


```

divmod(x, y)
------------

divmod(x, y)会返回一个内容为“(x//y, x%y)的元组”（x 除以 y 的商和余数）。

```scss
a = divmod(5,3)
b = divmod(9,2)
print(a)   # (1, 2)
print(b)   # (4, 1)


```

max(iter)
---------

max(iter)会返回一个可迭代对象（字符串、元组、列表）中的“最大值”。

```scss
a = max('100200300')
b = max([100,200,300])
c = max((100,200,300))
print(a)   # 3 ( 因为字符串拆开后只有 0 1 2 3 )
print(b)   # 300
print(c)   # 300


```

min(iter)
---------

min(iter)会返回一个可迭代对象（字符串、元组、列表）中的“最小值”。

```scss
a = min('100200300')
b = min([100,200,300])
c = min((100,200,300))
print(a)   # 0 ( 因为字符串拆开后只有 0 1 2 3 )
print(b)   # 100
print(c)   # 100


```

pow(x, y, z)
------------

使用 pow(x, y)会返回“x 的 y 次方”，使用 pow(x, y, z)会返回“x 的 y 次方除以 z 的余数”。

```scss
a = pow(2, 3)
b = pow(2, 3, 3)
print(a)    # 8 ( 2 的 3 次方 )
print(b)    # 2 ( 8 除以 3 的余数 )


```

round(x, y)
-----------

round(x, y)会返回“四舍五入后的 x，y 表示四舍五入的小数点位数”，如果不指定 y 则表示四舍五入后的结果为整数。

```scss
a = round(3.14159)
b = round(3.14159, 3)
print(a)    # 3
print(b)    # 3.142


```

需要注意的是，当遇到 0.5 的数值时容易出现问题，因为 0.5 的数字其实是 0.44444444X 或 0.5000000X 之类的数字，这时使用 round()会发生预期外的情况。

```python
print(round(1.5))   
print(round(2.5))   
print(round(3.5))   
print(round(4.5))   
print(round(5.5))   


```

sum(iter, y)
------------

sum(iter, y)会返回“序列或元组的数值与 y 的总和”，如果不指定 y 则 y 为 0。

```scss
a = sum([1,2,3,4])
b = sum((1,2,3,4))
c = sum((1,2,3,4), 5)
print(a)    # 10
print(b)    # 10
print(c)    # 15  ( 最后再加上 5 )


```

complex(x, y)
-------------

complex(x, y)会返回“x + yj”的复数形式，如果 x 是数字，需要加上数字 y，如果 x 是字符串，则不需要 y。

```scss
a = complex(1, 2)
print(a)   # (1+2j)


```

bool(x)
-------

bool(x)可以将参数 x 转换成布尔值 False 或 True，如果内容是 0 或空值就会是 False，反之其他内容就会是 True。

```scss
a = bool(1)
b = bool(0)
c = bool()
d = bool(999)
e = bool('hello')
f = bool([0])
g = bool([])
print(a)  # True
print(b)  # False
print(c)  # False
print(d)  # True
print(e)  # True
print(f)  # True (因为列表有值)
print(g)  # False (因为列表为空)


```