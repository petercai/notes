# python 标准库：random
> 在数据分析，数据清洗，数据集处理中，除了使用，我们熟悉的 `numpy.random` 模块来生成随机数，或者随机采样，事实上，python 标准库也提供了 random 模块，如果不想，仅仅因为使用随机数，而单独导入 `numpy` 时，标准库提供的 `random` 模块，不失为一种，轻量级替代方案，并且两者使用起来几乎一样。

1\. 导入模块
--------

random 是 python 标准库模块，随 python 一起安装，无需单独安装，可直接导入。

```python
import random

```

2\. 实用方法
--------

### 2.1 random()

`random()`生成一个位于半开放区间 \[0, 1) 的浮点数，几乎所有`random`模块的方法的实现，都依赖于 `random()`。

```python
random.random()

```

```apache
0.9111245252327139

```

### 2.2 randint(a, b)

返回随机整数 N 满足 a <= N <= b。相当于 randrange(a, b+1)。

```python
random.randint(0, 10)

```

```null
8

```

### 2.3 randrange(start, stop\[, step\])

从 `range(start, stop, step)` 返回一个随机选择的元素。 这相当于 `choice(range(start, stop, step))` ，但实际上并没有构建一个 range 对象。

```python
random.randrange(0, 10, 2)

```

```null
2

```

### 2.4 choice(seq)

从非空序列 seq 返回一个随机元素。 如果 seq 为空，则引发 IndexError。

```python
random.choice([0, 1, 2, 3, 4, 5])

```

```null
2

```

### 2.5 choices(population, weights=None, *, cum_weights=None, k=1)

从_population_中选择替换，返回大小为 k 的元素列表。 如果 population 为空，则引发 IndexError。

如果指定了 weight 序列，则根据相对权重进行选择。 或者，如果给出 cum_weights 序列，则根据累积权重（可能使用 itertools.accumulate() 计算）进行选择。 例如，相对权重`[10, 5, 30, 5]`相当于累积权重`[10, 15, 45, 50]`。 在内部，相对权重在进行选择之前会转换为累积权重，因此提供累积权重可以节省工作量。

```python
random.choices([0, 1, 2, 3, 4, 5], k=2)

```

```angelscript
[0, 2]

```

```python
random.choices([0, 1, 2, 3, 4, 5], cum_weights=[10, 20, 30, 40, 50, 60], k=3)

```

```angelscript
[4, 4, 4]

```

### 2.6 shuffle(x\[, random\])

将序列 x 随机打乱位置。

可选参数 random 是一个0参数函数，在 \[0.0, 1.0) 中返回随机浮点数；默认情况下，这是函数 random() 。

要改变一个不可变的序列并返回一个新的打乱列表，请使用`sample(x, k=len(x))`。

```python
a = [0, 1, 2, 3, 4, 5]
random.shuffle(a)
a

```

```accesslog
[5, 2, 3, 0, 1, 4]

```

### 2.7 sample(population, k)

返回从总体序列或集合中选择的唯一元素的 k 长度列表。 用于无重复的随机抽样。

```python
random.sample([1, 2, 3, 4, 5, 6], k=2)

```

```angelscript
[1, 3]

```

### 2.8 gauss(mu, sigma)

高斯分布。 mu 是平均值，sigma 是标准差。

```python
[random.gauss(0, 1) for i in range(10)]

```

```subunit
[1.232295558291998,
 -0.23589397085653746,
 -1.4190307151921895,
 0.18999908858301912,
 0.780671045104774,
 0.041722424850158674,
 0.7392269754813698,
 1.4612049925568829,
 0.09647538110312114,
 -0.32525720572670025]

```

3\. 源码简要
--------

以下为 python 官方 github 上，random 模块的部分源码，帮助了解 random 模块的基本结构，以及本文介绍的实用方法的源码申明。

random.py:

```python
class Random(_random.Random):
    def seed(self, a=None, version=2):
        pass
    def randrange(self, start, stop=None, step=1, _int=int):
        pass
    def randint(self, a, b):
        pass
    def choice(self, seq):
        pass
    def shuffle(self, x, random=None):
        pass
    def sample(self, population, k):
        pass
    def choices(self, population, weights=None, *, cum_weights=None, k=1):
        pass
    def gauss(self, mu, sigma):
        pass
    ...

_inst = Random()
seed = _inst.seed
random = _inst.random
randrange = _inst.randrange
randint = _inst.randint
choice = _inst.choice
shuffle = _inst.shuffle
sample = _inst.sample
choices = _inst.choices
gauss = _inst.gauss

...

```

参考
--

*   \[1\] [python docs: random --- 生成伪随机数](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.python.org%2Fzh-cn%2F3%2Flibrary%2Frandom.html "https://docs.python.org/zh-cn/3/library/random.html")
*   \[2\] [github: python/cpython/Lib/random.py](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fpython%2Fcpython%2Fblob%2F3.7%2FLib%2Frandom.py "https://github.com/python/cpython/blob/3.7/Lib/random.py")

* * *
