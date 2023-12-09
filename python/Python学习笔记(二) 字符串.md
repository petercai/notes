# Python学习笔记(二) 字符串 
这篇文章将会介绍 Python 的字符串，主要讲解字符串的常用方法和格式设置，字符串也是一种序列类型

* * *

### [](https://link.juejin.cn/?target=)1、通用序列操作

（1）创建字符串

Python 允许我们使用单引号 `''` 或双引号 `""` 创建字符串，只要左右两边的引号保持一致就可以

```python
>>> a = 'Hello Wrold'
>>> b = "Hello Python"

```

当然，我们也可以使用内置函数 `str()` 来创建字符串或将其他类型转化为字符串

```python
>>> li = ['I', 'Love', 'Python']
>>> c = str(li)      
>>> 
>>> 
>>> d = ' '.join(li) 

```

*   转义字符：在某些字符前加上 `\`，能表达特殊的含义，例如 `\n` 表示换行，`\t` 表示缩进

```python
>>> string = 'Hello\nMy\tfriend'
>>> print(string)



```

*   原始字符串：在引号前面加上 `r`，表示不对字符串的内容进行转义

```python
>>> string = r'Hello\nMy\tfriend'
>>> print(string)


```

*   长字符串：用三引号代替普通引号，表示保留文本的原格式，适用于篇幅较长的文段

```python
>>> string = '''To see a world in a grain of sand,
And a heaven in a wild flower,
Hold infinity in the palm of your hand,
And eternity in an hour.'''
>>> print(string)





```

（2）索引与切片

与列表相似，`str` 类型可以使用 `[]` 对其中某一个字符进行索引，或对其中某一段字符串进行切片

通过切片得到的字符串，是原字符串的一份副本，而非原字符串本身，所以切片操作并不会影响原字符串

```python
>>> string = 'Never give up, Never lose hope.'
>>> string[13]  
>>> string[:13] 

```

（3）序列操作符

*   可以使用 `+` 拼接字符串

```python
>>> base = 'D:\\'
>>> path = 'Document'
>>> base + path 

```

*   可以使用 `*` 重复字符串

```python
>>> item = 'wh'
>>> item * 5 

```

（4）不可变序列

字符串其实是不可变序列，一旦创建不可修改，但有的同学可能会问，那下面这个字符串拼接是怎么回事呢

```python
>>> string = '123'
>>> string += '45'
>>> string 

```

这看起来的确像是我们修改了 `123`，将它变成了 `12345`，可实际上并不是这样的

事实上，我们没有修改字符串 `123`，而是重新创建一个新的字符串 `12345`，并改变变量 `string` 的指向

请大家看下面例子，可以发现 `string` 两次输出的 `id` 是不一样的

```python
>>> string = '123'
>>> id(string)

>>> string += '45'
>>> id(string)


```

### [](https://link.juejin.cn/?target=)2、字符串方法

字符串是 Python 中最为常用的变量类型之一，有许多内置的方法，下面我们将会逐一进行讲解

建议大家亲自动手敲一遍示例代码，理解各个方法的使用场景和限制

不过吧，就算记不住所有的用法也是正常的，毕竟字符串的方法多而繁杂，在实际使用的时候查查手册就好

*   `capitalize()` ：将字符串的第一个字符转换为大写字母
*   `title()`：将所有单词的第一个字符转换为大写字母
*   `upper()`：将字符串的所有字符转化为大写字母
*   `lower()`：将字符串的所有字符转化为小写字母
*   `swapcase()` ：将字符串的所有字符大小写互换

```python
>>> string = 'while There is life There is hope.'

>>> string.capitalize()

>>> string.title()

>>> string.upper()

>>> string.lower()

>>> string.swapcase()


```

*   `center(width[, fillchar])`：将字符串居中，并用 `fillchar`（默认为空格）填充至 `width`
*   `ljust(width[, fillchar])`  ：将字符串靠左，并用 `fillchar`（默认为空格）填充至 `width`
*   `rjust(width[, fillchar])`  ：将字符串靠右，并用 `fillchar`（默认为空格）填充至 `width`

**注意**：如果 `width` 小于字符串宽度则直接返回字符串，不会截断输出

```python
>>> title = 'Title Here'
>>> title.center(50, '-')

>>> lpara = 'Para Start Here'
>>> lpara.ljust(50, '-')

>>> rpara = 'Para End Here'
>>> rpara.rjust(50, '-')


```

*   `strip([char])`  ：去掉字符串前后的 `char`（默认为空格或换行）
*   `lstrip([char])`：去掉字符串前的 `char`（默认为空格或换行）
*   `rstrip([char])`：去掉字符串后的 `char`（默认为空格或换行）

**注意**：该方法只能处理字符串开头或结尾的字符，不能用于处理中间部分的字符

```python
>>> string = '000000100010'

>>> string.strip('0')

>>> string.lstrip('0')

>>> string.rstrip('0')


```

*   `split([sep [,maxsplit]])`：按照 `sep`（默认空格或换行）对字符串进行分割  
    `maxsplit` 用于指定最大分割次数，若大于指定次数则不再分割，默认为 `-1`，表示全部分割
*   `splitlines([keepends])`：按照换行符（`\r`、`\n`、`\r\n`）对字符串进行分割  
    `keepends` 默认为 `False`，表示分割出来的子串不保留换行符，若为 `True`，则保留换行符

```python
>>> string = 'do what you say \n say what you do'

>>> string.split()

>>> string.splitlines()


```

*   `partition(sep)`  ：从左到右寻找第一次出现的 `sep` 对字符串进行分隔，返回一个三元元组  
    三元元组的元素，第一个为分隔符左边的子串，第二个为分隔符本身，第三个为分隔符右边的子串
*   `rpartition(sep)`：从右到左寻找第一次出现的 `sep` 对字符串进行分隔，返回一个三元元组  
    三元元组的元素，第一个为分隔符左边的子串，第二个为分隔符本身，第三个为分隔符右边的子串

```python
>>> string = 'do what you say \n say what you do'

>>> string.partition('what')

>>> string.rpartition('what')


```

*   `find(sub [,start [,end]])`  ：从左到右检查 `sub` 是否包含在字符串从 `start` 到 `end` 范围中  
    返回第一个符合要求的子串的索引值，若没有找到则返回 -1
*   `rfind(sub [,start [,end]])`：从右到左检查 `sub` 是否包含在字符串从 `start` 到 `end` 范围中  
    返回第一个符合要求的子串的索引值，若没有找到则返回 -1

```python
>>> string = 'abcabcdabcde'

>>> string.find('cd')

>>> string.rfind('cd')


```

*   `beginswith(sub [,start [,end]])`：检查字符串从 `start` 到 `end` 范围中是否以 `sub` 开始
*   `endswith(sub [,start [,end]])`    ：检查字符串从 `start` 到 `end` 范围中是否以 `sub` 结束

```python
>>> string = 'Whatever happens, happens for a reason.'

>>> string.startswith('What')

>>> string.endswith('ending')


```

*   `join(sequence)`：将字符串插入到 `sequence` 每两两相邻的字符中

```python
>>> string = 'LOVE'
>>> string.join('python')


```

*   `count(sub [,start [, end]])`：寻找字符串中从 `start` 到 `end` 之间 `sub` 出现的次数

```python
>>> string = 'Hello World'
>>> string.count('o')


```

*   `replace(old, new[, max])`：将字符串中 `old` 子串转换为 `new` 子串，最大替换次数为 `max`

```python
>>> string = 'Hello Alice, Hello Bob.'
>>> string.replace('Hello', 'Goodbye')


```

*   `translate(table)`：根据 `table` 制定的规则替换字符

```python
>>> string = 'This is an example'
>>> table = str.maketrans('aeiou', '12345')
>>> string.translate(table)
'Th3s 3s 1n 2x1mpl2'

```

### [](https://link.juejin.cn/?target=)3、格式设置

（1）格式字符串

对于字符串的格式设置，在 Python 的早期解决方案中，主要使用类似 C 语言中的经典函数 `printf`

在格式字符串中使用转换说明符表示待插入值的位置、类型和格式，在格式字符串后写出待插入的值

```python
>>> string = 'Hello %s. Hello %s.' % ('Alice', 'Bob')
>>> string


```

上述格式字符串中的 `%s` 称为转换说明符，`s` 表示待插入的值是字符串类型，常用的格式说明符包括

| 转换说明符 | 说明 |
| --- | --- |
| `%c` | 字符 |
| `%s` | 字符串 |
| `%d` | 使用十进制表示的整数 |
| `%e` | 使用科学记数法表示的小数 |
| `%f` | 使用定点表示法表示的小数 |
| `%g` | 根据数的大小决定确定使用科学记数法还是定点表示法 |```python
>>> string = '%s is %d years old' % ('Helen', 18)
>>> string


```

（2）字符串方法

后来 Python 采用一种新的解决方案，那就是字符串方法 `format()`

在格式字符串中用花括号表示待插入值的位置、索引名称和格式，在 `format` 方法参数中写出待插入的值

```python

>>> string = 'Hello {}. Hello {}.'.format('Alice', 'Bob')
>>> string



>>> string = '{3} {0} {2} {1} {3} {0}'.format('be', 'not', 'or', 'to')
>>> string



>>> string = '{name} is {age} years old'.format(name = 'Helen', age = 18)
>>> string


```

① 转换标志：跟在感叹号后的单字符，表示用对应的格式转换给定的值

| 转换标志 | 含义 |
| --- | --- |
| `r` | 创建给定值的原始字符串表示（repr） |
| `s` | 创建给定值的普通字符串版本（str） |
| `a` | 创建给定值的 ASCII 字符表示（ascii） |```python
>>> print('{pi!r}\n{pi!s}\n{pi!a}'.format(pi = 'π'))




```

② 格式说明符：跟在冒号后的表达式，用于详细指定字符串的格式

| 类型说明符 | 说明 |
| --- | --- |
| `d` | 将整数表示为十进制数（这是整数默认使用的说明符） |
| `b` | 将整数表示为二进制数 |
| `o` | 将整数表示为八进制数 |
| `x` | 将整数表示为十六进制数 |
| `g` | 自动在定点表示法和科学记数法之间做出选择（这是小数默认使用的说明符） |
| `e` | 将小数表示为科学记数法 |
| `f` | 将小数表示为定点表示法 |
| `s` | 保持字符串的格式不变（这是字符串默认使用的说明符） |
| `c` | 将字符表示为 Unicode 码点 |```python
>>> 
>>> print('in decimal: {0:d}\nin binary : {0:b}'.format(10))


>>> print('in fixed-point notation: {0:f}\nin scientific  notation: {0:e}'.format(0.25))



>>> 
>>> print("{0:.2f}".format(1/3))


>>> 
>>> print("{0:10.2f}".format(1/4))


>>> 
>>> print("{0:<10.2f}\n{0:^10.2f}\n{0:>10.2f}".format(1/6))




>>> 
>>> print("{0:*<10.2f}\n{0:*^10.2f}\n{0:*>10.2f}".format(1/7))




```