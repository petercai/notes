# PyInstaller 系列 - 规格文件
在本系列的 [基本用法](https://shuhari.dev/blog/2018/06/pyinstaller-introduction) 篇中我们曾说过，PyInstaller 在生成文件的同时会创建一个相应的 .spec 文件。其实，.spec 文件才是生成过程的真正核心。它本质上是一个特殊的 Python 脚本，其中记录了生成所需的指令，和包管理所使用的 setup.py 在某种程度上有些相似。当熟悉它的格式以后，你也可以按自己的意愿去修改此文件，并且某些特殊场景下修改 .spec 文件是最简便的方法。本文就讲述 spec 文件的格式、原理和一些常用使用案例。

Spec 文件生成方法
-----------

实际上，PyInstaller 生成打包文件有两种方法。第一种就是我们已经使用过的，直接指定 .py 脚本：

`pyinstaller [options] xxx.py` 

使用该方式，PyInstaller 会首先根据选项生成对应的 .spec 文件，然后执行 .spec 文件所指定的过程生成最终文件。

你也可以指定已存在的 spec 文件：

`pyinstaller [options] xxx.spec` 

指定 spec 文件的情况下通常很少再使用命令行选项，因为这些选项都可以通过 .spec 文件中对应的信息修改。我们在下一节讲述 .spec 文件的具体格式。

其实我们会发现，这两种方式本质上没什么区别，只是第二种方式省去第一步（创建spec文件）罢了。有些同学可能更喜欢命令行，其他一些同学可能喜欢手工编辑 spec 文件，这都是允许的。不过要注意的一点是，当指定 .py 文件运行时，原来的 .spec 文件内容将被覆盖，所以请尽量不要将两种方式混用（当然如果你用了版本控制的话，就尽可放心大胆地尝试了）。

Spec 文件格式
---------

.Spec 本质上是一个 Python 脚本，其基本结构是大体固定的，但根据生成模式的不同，在最后的步骤上会有所差别。具体的说，当使用单目录模式时（不清楚单目录/单文件模式的同学请参考前一片文章），对应的 spec 文件总体格式如下：

`a = Analysis(...)
pyz = PYZ(...)
exe = EXE(...)
coll = COLLECT(...)` 

若使用单文件（onefile）模式的话，则结构是这样的：

`a = Analysis(...)
pyz = PYZ(...)
exe = EXE(...)` 

它们之所以存在区别的原因是，单文件模式是将所有内容统一打包到 .exe，而单目录模式除了生成 .exe 之外，还需要拷贝其他的附属文件，所以多了一个步骤。

除此之外，还有一个较少使用的多重打包模式（Multipackage Bundle），该模式还会增加一个 MERGE 的步骤。我个人没有用过这种模式，并且据官方说在 3.0 中还有问题，因此这里略过不表。

上述各个步骤的说明如下：

*   Analysis: 分析脚本的引用关系，并将所有查找到的相关内容记录在内部结构中，供后续步骤使用；
*   PYZ: 将所有 Python 脚本模块编译为对应的 .pyd 并打包；
*   EXE: 将打包后的 Python 模块及其他文件一起生成可执行的文件结构；
*   COLLECT: 将引用到的附属文件拷贝到生成目录的对应位置。

这些步骤其实不理解也没有什么关系，因为它们是完全固定的。我们更加关心的是命令行选项对应的那些信息，以便知道哪些部分是可以由我们自己来修改的。

一个更加详细的 .spec 文件如下所示（为节省篇幅和突出重点期间，忽略了一些次要内容）:

`# -*- mode: python -*-

block_cipher = None

a = Analysis(['main.py'],
             pathex=['D:\\workspace\\python'],
             binaries=[('mylib.dll', 'lib')],
             datas=[
                 ('README.txt', '.'),
                 ('sample.png', 'img')
             ],
             hiddenimports=[],
             hookspath=[],
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher)
pyz = PYZ(a.pure, a.zipped_data,
          cipher=block_cipher)
exe = EXE(pyz,
          a.scripts,
          exclude_binaries=True,
          name='main',
          debug=False,
          strip=False,
          upx=True,
          console=True , 
          version='version.txt', 
          icon='main.ico')
coll = COLLECT(exe,
               a.binaries,
               a.zipfiles,
               a.datas,
               strip=False,
               upx=True,
               name='main')` 

其中的参数和命令行的对应关系还是比较明显的。方便起见，我们将它们列表对照一下。

| 命令行开关 | Spec 参数 | 备注 |
| --- | --- | --- |
| xx.py | Analysis 第一个参数 | 入口脚本 |
| --add-binary | Analysis.binaries | 附加二进制文件 |
| --add-data | Analysis.datas | 附加数据文件 |
| --no-upx | EXE.upx, COLLECT.upx | 使用UPX |
| -c, -w | EXE.console | 控制台/窗口 |
| --version-file | EXE.version | 版本信息 |
| --icon | EXE.icon | 程序文件图标 |

一点说明：在命令行下，数据/二进制文件参数是用路径分隔符分割的；而在 spec 文件中则是成解析后的 tuple，使用时要注意写法。

我们已经知道 .spec 本质上就是 Python 脚本文件，所以添加自己的代码也是完全可能的。比如，你想确认一下程序引用了哪些脚本，就可以在 Analysis 后面加上一句：

`print('a.scripts:', a.scripts)` 

然后在生成时就可以看到对应的输出了。

通过 Spec 文件指定解释器开关
-----------------

还有一点不太明显的是，你可以为生成的执行文件指定一些特定于 Python 解释器的开关。大部分开关可能很少用到，不过其中有一个值得了解一下，那就是 `-v`。指定 `-v` 对生成过程并无影响，但执行文件在运行时会输出所有导入模块的信息，这些信息对排查类似 "module not found" 一类的错误是很有帮助的。要知道完整的开关项请参考用户手册。

解释器开关的写法有点特殊。每个选项是一个三元的 tuple, 通常只使用第一个，后面两项为固定的 None 和 'OPTION'。比如：

`options = [('v', None, 'OPTION')]

exe = EXE(pyz,
          a.scripts,
          options,
          exclude_binaries=True,
          ...)` 

当你再生成文件并运行时，就会看到大量输出信息，类似下面：

`import _frozen_importlib # frozen
import _imp # builtin
import sys # builtin
import '_warnings' # <class '_frozen_importlib.BuiltinImporter'>
...` 

这个列表通常很长，看的时候要有点耐心。当然，最后发布给用户之前别忘了去掉。

其他技巧
----

.Spec 本质上是 Python 脚本文件，因此你可以在其中应用任何你所知道的 Python 代码技巧。比如，如果数据文件很多导致 Analysis 太长的话，那么你可以把它提取为单独的变量：

`data_files = [(...)]
a = Analysis(...,
             datas=data_files,
             ...)` 

另一个小技巧是，可以为数据/二进制文件指定通配符，从而匹配同一类型的多个文件。

`a = Analysis(...,
             datas=[('media/*.mp3', 'media')],
             ...)` 

Spec 文件还有一些具体的技术细节我们就不再这里详述的。有兴趣的同学可以自行阅读 PyInstaller 官方文档中的 Using Spec Files 一节。

在下一篇文章中，我们将介绍 PyInstaller 的 Hook 机制。

PyInstaller 系列索引
----------------

本文系列介绍 PyInstaller 的使用知识和一些技术性的内幕。

*   [PyInstaller 系列 - 索引](https://shuhari.dev/blog/2018/6/pyinstaller-index)
*   [PyInstaller 系列 - 介绍](https://shuhari.dev/blog/2018/6/pyinstaller-introduction)
*   [PyInstaller 系列 - 单目录和单文件模式](https://shuhari.dev/blog/2018/6/pyinstaller-onedir-and-onefile)
*   PyInstaller 系列 - 规格文件
*   [PyInstaller 系列 - Hook 机制](https://shuhari.dev/blog/2018/6/pyinstaller-hook)