# Python With 关键字和语句
在 Python 文件处理的时候，我们通常会看到使用 with 关键字的语句。

Python 使用 with 语句能够让代码更加可读，并且能够更好的处理异常。

文件处理异常
------

我们都知道在 Java 处理程序的时候通常会出现异常，这个很容易理解。

在 Window 的时候，我们对文件进行拷贝或者剪贴的时候，我们可能会遇到比如说文件不存在，目录不存在，文件重名，或者文件被其他进程占用无法删除的情况。

这种情况都是文件处理异常，在 Java 中我们通常会使用 try catch 来捕获异常，然后对异常进行处理。

这样的目的是为了避免程序被挂起或者其他影响继续执行的情况。

Python 文件处理
-----------

如果我们不使用 with 语句，我们通常会使用下面的语法来对文件进行操作。

```python

  

file = open('file_path', 'w') 
file.write('hello world !') 
file.close() 
  

file = open('file_path', 'w') 
try: 
    file.write('hello world') 
finally: 
    file.close() 



with open('file_path', 'w') as file: 
    file.write('hello world !') 

```

在 2 个例子的时候，我们会发现写法和 Java 是差不多的。

通常计算机对文件的操作是 3 步做的：

1.  打开文件
2.  操作文件
3.  关闭文件

在上面的 3 步，都有可能出现异常。很多时候会没有办法关闭文件的，或者很多程序员会忘记关闭文件。

使用 with 语句，将会默认为你关闭文件。

With 语句解释
---------

with 语句实际上就是上下文管理。在上下文管理中，包括有 **enter**() 和 **exit**()

这将会允许我们常用的 try…except…finally 使用通过封装的方式来对资源进行释放。

with 语句没有捕获异常的功能，可以将 with 理解为一个静音的 try…except…finally，能够帮助 Python 程序在出现异常的时候也能够正常的退出而不会挂起。

with 可以应用在支持上下文的对象中，文件操作。

因计算机会对文件进行大量的操作，因此 with 语句会被广泛应用到文件操作上。

Git 源代码
-------

针对 with 操作的测试程序，请访问 Github 上面我们的存储的源代码。

访问地址为： [python-tutorials/FileWith.py at master · cwiki-us-docs/python-tutorials · GitHub](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fcwiki-us-docs%2Fpython-tutorials%2Fblob%2Fmaster%2Ftests%2FFileWith.py "https://github.com/cwiki-us-docs/python-tutorials/blob/master/tests/FileWith.py")

[![](_assets/5cf46981e05a44fbbe487d8771e2393b~tplv-k3u1fbpfcp-zoom-in-crop-mark!1512!0!0!0.awebp.webp)
](https://link.juejin.cn/?target=https%3A%2F%2Fcdn.ossez.com%2Fdiscourse-uploads%2Foriginal%2F2X%2F7%2F7a264870c897b776b47d7ac079ec235c69194067.png "https://cdn.ossez.com/discourse-uploads/original/2X/7/7a264870c897b776b47d7ac079ec235c69194067.png")

我们尝试不在这里讨论过多过于复杂内容，我们想尽量简化操作。

针对 with 的使用只需要知道是 try…except…finally 的简单化版本，多用于文件操作。

[www.ossez.com/t/python-wi…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.ossez.com%2Ft%2Fpython-with%2F13387 "https://www.ossez.com/t/python-with/13387")