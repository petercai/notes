# Python快速进修指南：文件操作
Python提供的文件操作相对于Java来说，确实简单方便许多。不仅操作简单，代码可读性也相对较高。然而，我们需要注意的不仅仅是文件操作的简单性，还有文件操作的各种模式。在Java中，我们并不经常使用像Python中那样的操作模式。

另外，我们还需要注意文件指针的移动。无论是Java还是Python，文件都可以看作是IO流，流到哪里就算是哪里。除非重新对文件进行操作，否则想要回到文件开头，只能通过移动指针来实现。因此，在进行文件操作时，我们需要谨慎考虑文件指针的位置。

首先，我们需要使用open()函数来打开文件，并指定文件名和打开模式。常用的打开模式有多种选项，对于我们有经验的来说，r、w、a基本都能猜到他们所代表的英文意思。

*   r：只读模式，从文件中读取数据（默认模式）。
*   w：写入模式，首先清空文件内容，然后写入数据。
*   a：追加模式，将数据写入文件末尾。
*   b：二进制模式，用于处理二进制数据，也就是图片和视频文件了。你可以将"b"理解为"binary"的缩写
*   t：文本模式（默认模式），用于处理文本文件。你可以将"t"理解为"text"的缩写

```python

file = open("example.txt", "rt")


content = file.read()
print(content)


file.close()

```

除了示例中使用的rt模式，还有其他常用的模式，就是r、w和b、t的字母组合了：

*   wt：以文本模式写入文件。如果文件不存在，则创建一个新文件；如果文件已存在，则清空文件内容。
*   rb：以二进制模式读取文件。
*   wb：以二进制模式写入文件。

我们上面的写法是最基础的，为了保证文件操作的正确性和资源的释放，我们需要手动关闭文件。在Java中，可以使用try-with-resource语法来自动关闭流，而在Python中，我们也可以使用with语句来实现类似的功能，自动关闭文件，如下所示：

```python
with open("filename.txt", "r") as file:
    content = file.read()
    print(content)

```

当你在写入文件后，想要回到文件开头以便读取文件内容时，可以使用seek(0)将指针移动到文件的开头位置。以下是一个示例：

```python
with open("file.txt", "a+") as file:
    file.write("This is a new line.")
    file.seek(0)
    content = file.read()
    print(content)

```

使用`seek(0)`将指针移动到文件的开头位置。最后，我们使用`read()`函数读取整个文件的内容，并将其打印出来。指令后面的`+`号可以表示以读写方式打开文件。

使用with open()语句可以更简洁地管理文件的打开和关闭，下面是使用with open()语句进行文件交换、删除源文件和重命名临时文件的示例代码：

```python
import os


source_file = "path/to/source_file.txt"


temp_file = "path/to/temp_file.txt"


with open(source_file, "rt") as file, open(temp_file, "wt") as temp:
    content = file.read()
    temp.write(content)


os.remove(source_file)


os.rename(temp_file, source_file)

```

这次我们第一次使用了import语句，这个语句的作用是导入包。通过导入包，我们可以直接使用写好的逻辑，而不需要自己去编写。Python之所以能够如此简洁，离不开各种强大的包的支持。实际上，文件交换部分的代码也可以利用包来实现，因为已经有其他人写好了相关的功能，就像我们需要实现列表功能时可以直接使用ArrayList一样。市面上已经有很多优秀的轮子可供使用，只需要直接拿来用，千万不要重复造轮子~~

Python提供的文件操作相对于Java来说，更简单方便。不仅操作简单，代码可读性也更高。不过，我们还需要注意文件操作的各种模式和文件指针的移动。虽然文件操作只有几种方式，但我不会给出示例，避免浪费大家的时间和精力。