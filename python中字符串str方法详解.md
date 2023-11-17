# python中字符串str方法详解
### 一、介绍

字符串是Python中常用的数据类型之一，用于表示文本数据。可以使用单引号、双引号或三引号来创建字符串。例如：

```python

str1 = 'Hello, World!'


str2 = "Hello, World!"


str3 = '''Hello,
World!'''


```

字符串可以包含字母、数字、空格、标点符号等字符。Python中的字符串是不可变的，也就是说，一旦创建了一个字符串，就不能对其进行修改。但可以通过字符串方法来处理字符串，例如截取子串、连接字符串、替换字符串等。

### 二、常用的字符串内置函数（字符串方法）

#### 1.capitalize()方法

capitalize()方法是Python中用于将字符串的第一个字符大写的方法。它会返回一个新的字符串，该字符串的第一个字符被转换为大写，其余字符不变。

```python

str.capitalize()


str_data = "hello world"
str_capitalized = str_data.capitalize()
print(str_capitalized)


Hello world

```

注意：capitalize()方法不会修改原始字符串，而是返回一个新的字符串。如果你想在原始字符串上进行更改，你需要将capitalize()的结果赋值回原始变量，例如：

```python
str_data1 = "hello world"
str_data1 = str_data1.capitalize()
print(str_data1)  

Hello world

```

#### 2.case fold()方法

casefold()是用于字符串大小写不敏感比较的字符串方法,它将字符串中的所有字符转换为小写，返回一个合适的无大小写比较版本的字符串并返回一个新的字符串。与lower()方法不同，casefold()方法更加严格，可以将一些特殊字符（例如语音符号）转换为小写。

```python

stra.casefold()


str_data = "Hello World"
str_casefold = str_data.casefold()
print(str_casefold)


hello world

```

#### 3.center()方法

center() 方法返回一个指定的宽度 width 居中的字符串，fillchar 为填充的字符，默认为空格。如果指定的宽度小于原始字符串的长度，则返回原始字符串。

```python

str.center(width[, fillchar])


str_data = "hello world"
str_center = str_data.center(30, "-")
print(str_center)


str_data = "hello world"
print(str_data.center(30))


---------hello world----------
         hello world          

```

#### 4.count()方法

count() 方法用于统计字符串里某个字符出现的次数。可选参数为在字符串搜索的开始与结束位置。

```python

str.count(sub, start= 0,end=len(string))


str_data = "hello world"
print(str_data.count("l"))


3


str_data1 = "one,two,three,four,five"
count = str_data1.count("t", 0, 10)
print(count)

2

```

注意：count()方法是区分大小写的，因此在计算子字符串出现次数时，大小写会被视为不同的字符。如果你想进行大小写不敏感的计数，可以使用casefold()或lower()方法将字符串转换为小写，然后再调用count()方法。

#### 5.encode()方法

用于将字符串编码为指定的字节序列。它接受一个字符串参数，表示要使用的编码格式，并返回一个字节对象。errors参数可以指定不同的错误处理方案。

```python

str.encode(encoding='UTF-8',errors='strict')


str_data = "hello world"
print(str_data.encode("GBK"))

b'hello world'

```

注意:encode()方法只能用于字符串到字节的编码转换，不能用于其他类型的数据。同时，需要使用相同的编码格式来解码字节序列以获取原始字符串例如，可以使用decode()方法将字节序列解码为字符串：

```python
str_data1 = b'hello world'
print(str_data1.decode("GBK"))


hello world

```

**将字符串转换为字节：** 

```python
str_data = "hello"
bytes_data = bytes(str_data, 'utf-8')  
print(bytes_data)

bytes_data = str_data.encode('utf-8')  
print(bytes)


```

**将字节转换为字符串：** 

```python
b = b'hello'
s = b.decode('utf-8')  
print(s)


```

#### 6.endswith()方法

endswith() 方法用于判断字符串是否以指定后缀结尾，如果以指定后缀结尾返回 True，否则返回 False。可选参数 "start" 与 "end" 为检索字符串的开始与结束位置。

```python

str.endswith(suffix[, start[, end]])





str_data = "hello world"
print(str_data.endswith("world"))
print(str_data.endswith("2"))

True
False


str_data = "Hello World"
print(str_data.lower().endswith("world"))

True

```

#### 7.expandtabs()方法

expandtabs() 方法把字符串中的 tab 符号 (**\\t)** 转为空格，tab 符号 **\\t** 默认的空格数是 8，在第 0、8、16...等处给出制表符位置，如果当前位置到开始位置或上一个制表符位置的字符数不足 8 的倍数则以空格代替.

```python

str.expandtabs(tabsize=8)



str_data = "Hello\t\tWorld!"
print(str_data.expandtabs(4))


Hello       World!

```

#### 8.find()方法

find() 方法检测字符串中是否包含子字符串 str ，如果指定 beg（开始） 和 end（结束） 范围，则检查是否包含在指定范围内，如果指定范围内如果包含指定索引值，返回的是索引值在字符串中的起始位置。如果不包含索引值，返回-1。

```python

str.find(str, beg=0, end=len(string))


str_data = "hello World"
index = str_data.find("World")
print(index)


6

```

find()方法区分大小写。如果需要忽略大小写，可以在调用方法时将字符串转换为小写或大写，例如：

```python
str_data = "hello World"
index = str_data.lower().find("world")
print(index)


6

```

如果要查找子字符串的最后一次出现位置，可以使用rfind()方法，它与find()方法类似，但从字符串的末尾开 始搜索。

#### 9.format()方法

format()是一个字符串方法，用于将字符串格式化为指定的格式。它接受一个或多个参数，并使用这些参数来替换字符串中的占位符。format后面传参只能是一个序列或者字典

```python

name = "Alice"
age = 30
message = "My name is {} and I am {} years old".format(name, age)
print(message)


My name is Alice and I am 30 years old

```

在这个示例中，变量name被赋值为"Alice"，变量age被赋值为30。然后使用format()方法来格式化字符串，其中占位符{}用于表示要插入变量的位置。在调用format()方法时，指定要插入的变量。最后，打印message的结果为"My name is Alice and I am 30 years old"。 **在format()方法中，还可以使用具有特定格式的占位符来格式化字符串。例如：** 

```python
price = 12.3456
message = "The price is ${:.3f}".format(price)
print(message)


The price is $12.346

```

在这个示例中，变量price被赋值为12.3456。然后使用format()方法来格式化字符串，其中占位符{:.3f}用于表示要插入变量的位置，并将变量格式化为3位小数的浮点数。最后，打印message的结果为"The price is $12.35"。

#### 10\. format_map()方法

用于将字符串格式化为指定的格式。与format()方法类似format\_map()方法也接受一个或多个参数，并使用这些参数来替换字符串中的占位符。 不同之处在于，format\_map()方法接受一个字典参数，该字典包含要替换的占位符和相应的值。字典中的键表示要替换的占位符，而值表示要插入到占位符的值。

```python

person = {"name": "Alice", "age": 30}
message = "My name is {name} and I am {age} years old".format_map(person)
print(message)


My name is Alice and I am 30 years old

```

在这个示例中，创建一个字典person，其中包含两个键值对，即"name"和"age"。然后使用format_map()方法来格式化字符串，其中占位符{name}和{age}用于表示要插入变量的位置，并且这些占位符的值从person字典中获取。最后，打印message的结果为"My name is Alice and I am 30 years old"。

#### 11.index()方法

index() 方法检测字符串中是否包含子字符串 str ，如果指定 beg（开始） 和 end（结束） 范围，则检查是否包含在指定范围内，该方法与 python find()方法一样，只不过如果str不在 string中会报一个异常。

```python

str.index(sub[, start[, end]])


str_data = "hello world"
print(str_data.index("o"))  
print(str_data.index("o", 5))  
print(str_data.index("o", 5, 7))  


4
7
ValueError: substring not found

```

在这个示例中，定义一个字符串str_data，其值为"hello world"。然后使用index()方法查找字符串中的子字符串"o"。第一个index()方法返回的结果为4，表示第一个"o"的索引位置是4。第二个index()方法从位置5开始查找，返回的结果为7，表示第二个"o"的索引位置是7。第三个index()方法指定了查找范围为位置5和位置7之间，由于在这个范围内没有找到"o"，因此引发了ValueError异常。 注意:如果要查找的子字符串不在字符串中，index()方法将引发ValueError异常。如果不想引发异常，可以使用find()方法，它与index()方法类似，但在找不到子字符串时返回-1。

#### 12.isalnum()方法

isalnum() 方法用于检查字符串是否只包含字母或数字字符。如果是，则返回True，否则返回False。

```python

str.isalnum()


str_data = "Hello123"
str_data2 = "Hello 123"
str_data3 = "Hello!"
print(str_data.isalnum(),str_data2.isalnum(),str_data3.isalnum())  


True
False 
False

```

注意:isalnum()方法不会检查空格、标点符号和其他非字母或数字字符，仅检查字符串是否仅包含字母或数字字符。如果字符串中包含任何非字母或数字字符，则isalnum()方法将返回False

#### 13.isalpha()：方法

Pisalpha() 方法检测字符串是否只由字母或文字组成。

```python

str.isalpha()


str_data = "Hello"
str_data1 = "hello123"
print(str_data.isalpha(),str_data1.isalpha())
	

True
False

```

#### 14.isascii()方法

它用于检查字符串中的所有字符是否都属于 ASCII 字符集。如果字符串中的所有字符都是 ASCII 字符，那么 isascii() 方法将返回 True。如果字符串中包含至少一个非 ASCII 字符，则该方法将返回 False。

```python

str.isascii()


str_data = "Hello, World!"
print(str_data.isascii())

str_data1 = "你好，世界！"
print(str_data1.isascii())


True
False

```

#### 15.isdecimal（）方法

它用于检查字符串中的所有字符是否都是十进制数字。如果字符串中包含其他字符，比如字母、标点符号或空格，该方法都会返回 False。如果需要检查字符串中的所有字符是否都是数字，可以使用 isdigit() 方法。

```python

str.isdecimal()


string1 = "12345"
string2 = "12.345"
string3 = "①②③④⑤"
print(string1.isdecimal())  
print(string2.isdecimal()) 
print(string3.isdecimal())  


True
False
False

```

在上面的示例中，string1 只包含十进制数字，因此 isdecimal() 方法返回 True。string2 中包含小数点，所以该方法返回 False。而 string3 中包含全角数字，但它们仍然是十进制数字，因此该方法也返回 True。

#### 16.isdigit()方法

```scss
isdigit() 方法检测字符串是否只由数字组成。如果字符串中的所有字符都是数字字符，则 isdigit() 方法将返回 True。否则，它将返回 False。

```

```python

str.isdigit()


str_data = "12345"
print(str_data.isdigit())
str_data1 = "123.45"
print(str_data1.isdigit())

True
False

```

isdigit() 方法只对正整数有效，负数及小数均返回不正确。可以使用以下函数来解决：

```python

def is_number(s):
    try:  
        float(s)
        return True
    except ValueError:  
        pass  
    try:
        import unicodedata  
        unicodedata.numeric(s)  
        return True
    except (TypeError, ValueError):
        pass
        return False


print(is_number(1))
print(is_number(1.0))
print(is_number(0))
print(is_number(-2))
print(is_number(-2.0))
print(is_number("abc"))


True
True
True
True
True
False

```

注意:isdigit() 方法与 isdecimal() 方法不同，它可以识别 Unicode 数字字符，而不仅仅局限于十进制数字字符。因此，对于包含其他类型数字字符的字符串，例如罗马数字、中文数字等，isdigit() 方法可能会返回 True

#### 17.isidentifier（）方法

用于检查字符串是否是合法的 Python 标识符。如果字符串是合法的 Python 标识符，则 isidentifier() 方法将返回 True。否则，它将返回 False。 标识符是用于标识变量、函数、类、模块等 Python 对象的名称。合法的 Python 标识符必须符合以下规则：

1.  只能包含字母、数字和下划线（_）,不能包含空格、@、% 以及 $ 等特殊字符。
2.  第一个字符必须是字母或下划线,不能以数字开头。
3.  区分大小写。

```python




str_data1 = "hello_world"
str_data2 = "2hello world"
str_data3 = "_helloworld"
str_data4 = "*hello"
print(str_data1.isidentifier(),str_data2.isidentifier(),str_data3.isidentifier(),str_data4.isidentifier())


True 
False 
True 
False

```

注意：isidentifier() 方法在 Python 3.0 之前的版本中不存在。如果需要在旧版本的 Python 中检查标识符的合法性，可以使用 isalnum() 方法和其他字符串处理函数来实现类似的功能。

#### 18.islower() 方法

检测字符串是否由小写字母组成。如果字符串中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是小写，则返回 True，否则返回 False

```python

str.islower()


str_data1 = "HELLO WORLD"
str_data2 = "hello world"
print(str_data1.islower(), str_data2.islower())


False True

```

注意：islower() 方法只针对字母字符进行判断，它不会检查字符串中是否包含其他类型的字符。如果字符串中既包含小写字母，又包含大写字母或其他类型的字符，islower() 方法也会返回 False。

#### 19.isnumeric()方法

isnumeric() 方法检测字符串是否只由数字组成。这种方法是只针对unicode对象。如果字符串中的所有字符都是数字字符，则 isnumeric() 方法将返回 True。否则，它将返回 False。

```python

str.isnumeric()


str_data = "123"
str_data1 = "123abc"
str_data2 = "X"
print(str_data.isnumeric())
print(str_data1.isnumeric())
print(str_data2.isnumeric())


True
False
False

```

#### 20.isprintable()方法

用于检查字符串中的所有字符是否都是可打印字符。可打印字符指的是可以显示在屏幕上的字符，包括字母、数字、符号和空格等，但不包括控制字符和不可打印字符，如制表符、换行符、回车符等。 如果字符串中的所有字符都是可打印字符，则 isprintable() 方法将返回 True。否则，它将返回 False。

```python

str.isprintable


str_data1 = "hello world"
str_data2 = "hello\nworld"
str_data3 = "hello\tworld"
print(str_data1.isprintable())
print(str_data2.isprintable())
print(str_data3.isprintable())


True
False
False

```

注意：isprintable() 方法只针对字符串中的单个字符进行判断，而不是针对整个字符串。如果字符串中包含多个字符，其中只要有一个字符不是可打印字符，该方法就会返回 False。

#### 21.isspace()方法

isspace() 方法检测字符串是否只由空格组成。空格字符包括空格、制表符、换行符、回车符等，它们都是 ASCII 码表中的控制字符。如果字符串中的所有字符都是空格字符，则 isspace() 方法将返回 True。否则，它将返回 False。

```python

str.isspace()


str_data = " "
str_data2 = "hello world"
str_data3 = "hello\tworld"
print(str_data.isspace(), str_data2.isspace(), str_data3.isspace())


True False False

```

isspace() 方法只针对字符串中的单个字符进行判断，而不是针对整个字符串。如果字符串中包含多个字符，其中只要有一个字符不是空格字符，该方法就会返回 False。

#### 22.istitle（）方法

用于检查字符串中的所有单词的首字母是否都是大写字母，并且其余字母都是小写字母或数字。istitle() 方法将返回 True。否则，它将返回 False。

```python

str.istitle()


str_data = "Hello World"
str_data1 = "Hello world"
str_data2 = "hello World"
str_data4 = "123Hello world"
print(str_data.istitle(),str_data1.istitle(),str_data2.istitle(),str_data3.istitle(),str_data2.istitle())


True False False False False

```

注意：istitle() 方法只针对字符串中的单词进行判断，而不是针对整个字符串。如果字符串中包含多个单词，其中只要有一个单词不符合上述条件，该方法就会返回 False。 istitle() 方法是 Python 中用于检查字符串中的所有单词的首字母是否都是大写字母，并且其余字母都是小写字母或数字的简单而实用的方法。

#### 23.isupper()方法

用于检查字符串中的所有字符是否都是大写字母。如果字符串中的所有字符都是大写字母，则 isupper() 方法将返回 True。否则，它将返回 False。

```python

str.isupper()


str_data = "HELLO WORLD"
str_data2 = "Hello World"
str_data3 = "1234"
print(str_data.isupper())
print(str_data2.isupper())
print(str_data3.isupper())


True
False
False

```

注意：isupper() 方法只针对字符串中的单个字符进行判断，而不是针对整个字符串。如果字符串中包含多个字符，其中只要有一个字符不是大写字母，该方法就会返回 False。（纯大写字母字符串。）

#### 24.join()方法

join() 方法用于将序列中的元素以指定的字符连接生成一个新的字符串。返回通过指定字符连接序列中元素后生成的新字符串。

```python

str = separator.join(iterable)



words = ["hello", "world", "python"]
separator = " "
result = separator.join(words)
print(result)


hello world python

```

join() 方法还可以用于将多个字符串拼接成一个字符串，例如：

```python

str_data1 = "hello"
str_data2 = "world"
wtr_data3 = "python"
result_data = "+".join([str_data1, str_data2, str_data3])
print(result_data)


hello+world+1234

```

注意：join() 方法是 Python 中用于将一个可迭代对象中的元素连接成一个字符串的常用方法，它能够方便地将多个字符串或列表中的元素拼接在一起。

#### 25.ljust()方法

ljust() 方法返回一个原字符串左对齐,并使用空格填充至指定长度的新字符串。如果指定的长度小于原字符串的长度则返回原字符串。

```python

str.ljust(width[, fillchar])



text = "hello"
print(1111111111)
padded_text = text.ljust(10)
print(padded_text)


1111111111
hello     

data_text = "world"
padded_text = data_text.ljust(10, '*')
print(padded_text)


world*****

```

#### 26.lower()方法

```scss
lower() 方法转换字符串中所有大写字符为小写。

```

```python

string = str.lower()


text = "Hello, World!"
lower_text = text.lower()
print(lower_text)  


hello, world!

```

注意:lower() 方法不会改变原始字符串，而是返回一个新的字符串。如果原始字符串中没有大写字母，则该方法返回原始字符串。 此外，Python 还提供了 upper() 方法，用于将字符串中的所有小写字母转换为大写字母。

#### 27.lstrip()方法

用于截掉字符串左边的空格或指定字符。

```python

str.lstrip([chars])   


text = "    hello     1"
stripped_text = text.lstrip()
print(stripped_text)  


hello     1

```

lstrip() 方法不会改变原始字符串，而是返回一个新的字符串。如果原始字符串中没有需要删除的字符，则该方法返回原始字符串。 此外，Python 还提供了 rstrip() 方法，用于删除字符串结尾的指定字符，以及 strip() 方法，用于删除字符串两端的指定字符。 总之，lstrip() 方法是 Python 中用于删除字符串开头指定字符的方法，它可以方便地用于去除字符串中的空格和其他无用字符。

#### 28.maketrans()方法

是 Python 字符串对象的一个内置方法，用于创建一个翻译表（translation table），该表可以用于对字符串进行替换操作。 **注：**两个字符串的长度必须相同，为一一对应的关系。

```python

table = str.maketrans(dict, [deletechars])



text = "hello, world!"
trans_table = text.maketrans({'l': 'X', 'o': 'V'})
trans_text = text.translate(trans_table)
print(trans_text)


heXXV, wVrXd!

```

注意:maketrans() 方法不会改变原始字符串，而是返回一个翻译表。同时，translate() 方法也不会改变原始字符串，而是返回一个新的字符串。 另外，如果需要删除字符串中的某些字符，可以在调用 maketrans() 方法时指定 deletechars 参数。 总之，maketrans() 方法是 Python 中用于创建翻译表的方法，它可以方便地进行字符串替换和删除操作。

#### 29 .partition()方法

用于根据指定的分隔符将字符串分成三部分，分别为分隔符左边的子串、分隔符本身和分隔符右边的子串。如果字符串中不存在指定的分隔符，则返回原始字符串和两个空字符串的元组。并返回一个包含分割结果的元组。

```python

str.partition(str)


text = "Hello,World!"
parts = text.partition(",")
print(parts)


('Hello', ',', 'World!')


sentence = "The quick brown fox jumps over the lazy dog"
words = sentence.partition("brown ")
print(words)  

('The quick ', 'brown ', 'fox jumps over the lazy dog')


```

注意：如果指定的分隔符在字符串中出现多次，则只会将第一个匹配到的分隔符作为分隔符进行分割。

#### 30.removeprefix()方法

用于从字符串的开头移除指定的前缀，并返回修改后的字符串。

```python

new_str = str.removeprefix(prefix)


text = "Hello, World!"
new_text = text.removeprefix("Hello, ")
print(new_text) 


World!

```

注意：如果字符串中不包含指定的前缀，则方法不会进行任何操作，直接返回原始字符串。

#### 31.removesuffix()方法

用于从字符串的末尾移除指定的后缀，并返回修改后的字符串。

```python

new_str = str.removesuffix(suffix)


text = "Hello, World!"
new_text = text.removesuffix(", World!")
print(new_text)


Hello

```

#### 32.replace()方法

它用于将字符串中的某个子串替换为另一个指定的子串，并返回替换后的新字符串。

```python

new_str = str.replace(old, new[, count])



text = "Hello, World!"
new_text = text.replace("Hello", "Hi")
print(new_text)  


Hi, World!

```

#### 33.rfind()方法

用于在字符串中查找指定子串最后一次出现的位置，并返回其索引。如果子串未找到，则返回 -1。

```python

index = str.rfind(sub[, start[, end]])


text = "Hello, World!"
index = text.rfind("o")
print(index)


8  


text = "Hello, World!"
index = text.rfind("W", 0, 5)
print(index)  


-1


text = "Hello,World!"
index = text.rfind("l")
print(index) 




text = "Hello, World!"
index = text.rfind("x")
print(index) 


```

#### 34.rindex()方法

查找子串在字符串中最后一次出现的位置。不同之处在于，如果子串不存在于字符串中，rindex() 方法会抛出一个 ValueError 异常，而 rfind() 方法会返回 -1。

```python

str.rindex(sub[, start[, end]])


text = "Hello, World!"
index = text.rindex("o")
print(index)  

index = text.rindex("o", 0, 4)
print(index)  

index = text.rindex("l")
print(index)  

index = text.rindex("x")  


```

#### 35.rjust()方法

```scss
rjust() 返回一个原字符串右对齐,并使用空格填充至长度 width 的新字符串。如果指定的长度小于字符串的长度则返回原字符串。

```

```python

str.rjust(width[, fillchar])


text = "Hello"
width = 10
fillchar = "-"
new_text = text.rjust(width, fillchar)
print(new_text)  

width = 5
new_text = text.rjust(width, fillchar)
print(new_text)  

```

#### 36.rpartition()方法

方法类似于 partition() 方法，只是该方法是从目标字符串的末尾也就是右边开始搜索分割符。。 如果字符串包含指定的分隔符，则返回一个3元的元组，第一个为分隔符左边的子串，第二个为分隔符本身，第三个为分隔符右边的子串。

```python

str.rpartition(str)


text = "I have an apple"
partition = text.rpartition("an")
print(partition)

partition = text.rpartition("banana")
print(partition)


('I have ', 'an', ' apple')
('', '', 'I have an apple')



```

#### 37.rsplit()方法

rsplit() 方法用于将字符串从右边开始按照指定的分隔符分割，并返回分割后的结果列表。与 split() 方法类似，但是分割是从字符串右边开始进行的。

```python

str.rsplit()


text = "apple, banana, cherry, date"
result = text.rsplit(", ")
print(result)

result = text.rsplit(", ", 2)
print(result)


['apple', 'banana', 'cherry', 'date']
['apple, banana', 'cherry', 'date']  

```

#### 38.rstrip()方法

rstrip() 删除 string 字符串末尾的指定字符，默认为空白符，包括空格、换行符、回车符、制表符。可以通过传入一个字符串或字符集合作为参数来指定要删除的字符或字符集合。如果不传入参数，则默认删除字符串末尾的空格符。

```python

str.rstrip([chars])


text = "  hello world  "
print(text.rstrip())

text = "banana#apple#cherry#"
print(text.rstrip("#"))

text = "apple**banana**cherry**"
print(text.rstrip("*"))

text = "123abcABC789"
print(text.rstrip("123456789"))


  hello world
banana
apple**banana**cherry
123abcABC


```

#### 39.split（）方法

用于将字符串按照指定的分隔符分割，并返回分割后的结果列表。默认情况下，分隔符是空格符。如果参数 num 有指定值，则分隔 num+1 个子字符串

```python

str.split(str="", num=string.count(str))


text = "apple banana cherry"
result = text.split()
print(result)

text = "apple,banana,cherry"
result = text.split(",")
print(result)

text = "apple::banana::cherry"
result = text.split("::")
print(result)


['apple', 'banana', 'cherry']
['apple', 'banana', 'cherry']
['apple', 'banana', 'cherry']

```

#### 40.splitlines()方法

用于将字符串按照换行符（\\n）分割成多个行，返回一个包含行字符串的列表。它可以处理不同类型的换行符（例如 \\n、\\r\\n、\\r 等），并且在分割时会自动删除换行符。返回的是一个列表。

```python

str.splitlines([keepends])


text = "Hello\nworld\r\nthis is a\nmultiline\rstring\r"
lines = text.splitlines()
print(lines)


['Hello', 'world', 'this is a', 'multiline', 'string']

```

注意：splitlines() 方法不会将最后一行后面的换行符作为一行处理，因此在上面的示例中，最后一行的换行符没有被包含在分割结果中。如果想要保留最后一行的换行符，可以将 keepends 参数设置为 True，例如 text.splitlines(keepends=True)。

#### 41.startswith()方法

用于检查字符串是否是以指定子字符串开头，如果是则返回 True，否则返回 False。如果参数 beg 和 end 指定值，则在指定范围内检查。

```python

str.startswith(str, beg=0,end=len(string));


text = "Hello, world!"
result1 = text.startswith("Hello")
result2 = text.startswith("hello")
print(result1) 
print(result2)  


True
False

```

startswith() 方法还可以接受一个可选的起始位置参数，指定从哪个索引位置开始检查。列如：

```python
text = "Hello, world!"
result = text.startswith("world", 7)
print(result)

True

```

#### 42.strip()方法

用于去除字符串开头和结尾的空格或指定的字符。注意：该方法只能删除开头或是结尾的字符，不能删除中间部分的字符。

```python

str.strip([chars])


text = "  Hello, world!  "
result1 = text.strip()
result2 = text.strip(" !")
print(result1)
print(result2)


Hello, world!
Hello, world

```

注意：strip() 方法返回一个新的字符串，原始的字符串不会被修改。

#### 43.swapcase()方法

swapcase() 方法用于交换字符串中的大小写字母。具体来说，它会将所有小写字母转换为大写字母，将所有大写字母转换为小写字母。

```python

text = "Hello, World!"
result = text.swapcase()
print(result)


hELLO, wORLD!

```

注意：swapcase() 方法返回一个新的字符串，原始的字符串不会被修改。

#### 44.titlt() 方法

title() 方法用于将字符串中每个单词的首字母大写，并将其余字母小写化。具体来说，它将每个单词的第一个字符转换为大写字母，将该单词中其余的所有字符转换为小写字母。

```python

str.title()

text = "hello, world!"
result = text.title()
print(result)


Hello, World!

```

注意：title() 方法返回一个新的字符串，原始的字符串不会被修改。另外，该方法仅将每个单词的第一个字符转换为大写字母，如果单词中还有其他大写字母，它们将不会被转换为小写字母。

#### 45.translate()方法

translate() 方法用于根据给定的转换表将字符串中的字符进行转换。它接受一个转换表作为参数，转换表可以通过 str.maketrans() 方法创建。转换表中包含了一组字符映射关系，将一个字符映射到另一个字符。当我们使用 translate() 方法时，该方法会将字符串中的每个字符都根据转换表进行转换。

```python

str.translate(table[, deletechars]);

text = "Hello, World!"
translation_table = str.maketrans("lo", "ab")
result = text.translate(translation_table)
print(result)


Heaab, Wbrad!

```

在上面的示例中，我们定义了一个字符串 text，然后使用 str.maketrans() 方法创建了一个转换表，将字符串中的字母 l 和 o 分别转换为字母 a 和 b。接下来，我们使用 translate() 方法将字符串中的字母根据转换表进行了转换，返回的结果是 "Hebba, Wabrd!"。 注意：translate() 方法返回一个新的字符串，原始的字符串不会被修改。另外，当我们使用 translate() 方法时，如果字符串中包含转换表中没有映射关系的字符，则这些字符不会被转换，直接保留原来的字符。

#### 46.upper()方法

upper() 方法用于将字符串中的所有小写字母转换为大写字母，并返回转换后的新字符串。这个方法不会修改原始的字符串，而是返回一个新的字符串。

```python

str.upper()


text = "hello, world!"
result = text.upper()
print(result)


HELLO, WORLD!

```

#### 47.zfill()方法

zfill() 方法返回指定长度的字符串，原字符串右对齐，前面填充0。如果字符串长度已经达到或超过指定长度，则返回原始字符串。

```python

str.zfill(width)


text = "10"
result = text.zfill(5)
print(result)


00010


```

注意：如果字符串中包含正负号或其他特殊字符，zfill() 方法不会将其替换为0，只会在左侧填充0以达到指定长度。如果希望将特殊字符替换为0，需要在调用 zfill() 方法之前先将其替换为0。