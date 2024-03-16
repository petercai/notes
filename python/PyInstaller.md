# PyInstaller
简介[¶](#id1 "永久链接至标题")
---------------------

PyInstaller可以用来打包python应用程序，打包完的程序就可以在没有安装Python解释器的机器上运行了。PyInstaller支持Python 2.7和Python 3.3+。可以在Windows、Mac OS X和Linux上使用，但是并不是跨平台的，而是说你要是希望打包成.exe文件，需要在Windows系统上运行PyInstaller进行打包工作；打包成mac app，需要在Mac OS上使用。


使用[¶](#id3 "永久链接至标题")
---------------------

PyInstaller分析你的python程序，找到所有的依赖项。然后将依赖文件和python解释器放到一个文件夹下或者一个可执行文件中。

1.  打包成一个文件夹
    
    当使用PyInstaller打包的时候，默认生成一个文件夹，文件夹中包含所有依赖项，以及可执行文件。打包成文件夹的好处就是debug的时候可以清楚的看到依赖项有没有包含。另一个好处是更新的时候，只需要更新可执行文件就可以了。当然缺点也很明显，不方便，不易管理。
    
    那么它是如何工作的呢？PyInstaller的引导程序是一个二进制可执行程序。当用户启动你的程序的时候，PyInstaller的引导程序开始运行，首先创建一个临时的Python环境，然后通过Python解释器导入程序的依赖，当然他们都在同一个文件夹下。
    
2.  打包成一个文件
    
    我们可以用 `--onefile` 或者 `-F` 参数将所有文件打包到一个可执行文件中。
    
    $ pyinstaller -F script.py
    
    打包成一个文件相对于文件夹更容易管理。坏处运行相对比较慢。这个文件中包含了压缩的依赖文件拷贝（.so文件）。
    
    当程序运行时，PyInstaller的引导程序会新建一个临时文件夹。然后解压程序的第三方依赖文件到临时文件夹中。这也是为什么一个可执行文件比文件夹中执行的时间要长的原因。剩下的就和上面的一样了。
    
3.  spec 文件
    
    当你执行下面命令
    
    $ pyinstaller options..script.py
    
    PyInstaller首先建一个sepc(specification)文件：script.spec。这个文件的存放地址可以使用参数–specpath= 来定义，默认放在当前文件夹下。
    
    spec文件的作用是什么呢？它会告诉PyInstaller如何处理你的py文件，它会将你的py文件名字和输入的大部分参数进行编码。PyInstaller通过执行spec文件中的内容来生成app，有点像makefile。正常使用中我们是不需要管spec文件的，但是下面几种情况需要修改spec文件：
    
    *   需要打包资源文件
        
    *   需要include一些PyInstaller不知道的run-time库
        
    *   为可执行文件添加run-time 选项
        
    *   多程序打包
        
    
    可以通过下面命令生成spec文件
    
    $ pyi-makespec options script.py \[other scripts ...\]
    
    修改完spec文件，就可以通过下面命令来生成app文件了
    
    $ pyinstaller options script.spec
    
    当通过spec文件来生成app文件的时候只有下面几个参数是有用的：
    
    *   `–upx-dir=`
        
    *   `–distpath=`
        
    *   `–noconfirm=`
        
    *   `–ascii`
        
4.  spec 文件解析
    
    \# -*- mode: python ; coding: utf-8 -*-
    
    block_cipher = None
    a = Analysis(\['script.py'\],
                 pathex=\[\],
                 binaries=\[\],
                 datas=\[\],
                 hiddenimports=\[\],
                 hookspath=\[\],
                 hooksconfig={},
                 runtime_hooks=\[\],
                 excludes=\[\],
                 win\_no\_prefer_redirects=False,
                 win\_private\_assemblies=False,
                 cipher=block_cipher,
                 noarchive=False)
    pyz = PYZ(a.pure, a.zipped_data,
                 cipher=block_cipher)
    exe = EXE(pyz,
              a.scripts,
              a.binaries,
              a.zipfiles,
              a.datas,  
              \[\],
              name='script',
              debug=False,
              bootloader\_ignore\_signals=False,
              strip=False,
              upx=True,
              upx_exclude=\[\],
              runtime_tmpdir=None,
              console=True,
              disable\_windowed\_traceback=False,
              target_arch=None,
              codesign_identity=None,
              entitlements_file=None )
    coll = COLLECT(...)
    
    spec文件中主要包含4个class: `Analysis`, `PYZ`, `EXE`和`COLLECT`。
    
    *   `Analysis` 以py文件为输入，它会分析py文件的依赖模块，并生成相应的信息
        
    *   `PYZ` 是一个.pyz的压缩包，包含程序运行需要的所有依赖
        
    *   `EXE` 根据上面两项生成
        
    *   `COLLECT` 生成其他部分的输出文件夹，COLLECT也可以没有
        
5.  修改spec文件
    
    我们上面说过有时候PyInstaller自动生成的spec文件并不能满足我们的需求，最常见的情况就是我们的程序依赖我们本地的一些数据文件，这个时候就需要我们自己去编辑spec文件来添加数据文件了。 上面的spec文件解析中Analysis中的datas就是要添加到项目中的数据文件，我们可以编辑datas.
    
    a = Analysis(
        ...
        datas = \[('you/source/file/path','file\_name\_in_project'),
        ('source/file2', 'file_name2')\]
        ...
        )
    
    可以认为datas是一个List,每个元素是一个二元组。元组的第一个元素是你本地文件索引，第二个元素是拷贝到项目中之后的文件名字。除了上面那种写法，也可以将其提出来。
    
    added_files = \[...\]
    
    a = Analysis(
        ...
        datas = added_files,
        ...
        )
    
    其他的二进制文件添加方法类似。
    
6.  总结
    
    最后简单来说，我们要通过PyInstaller生成可执行的文件主要下面两步。
    
    $ pyinstaller \[option\] mypython.py
    
    option为空生成文件夹，选择onefile，生成一个文件。 如果项目有一些依赖的数据文件，上面生成的二进制文件是无法运行的，这个时候可以通过修改spec文件，让后再用pyinstaller运行spec文件。
    
    $ pyinstaller \[option\] mypython.spec
    
    当然也按上文那样先生成spec文件。
    

Pyinstaller解释[¶](#id4 "永久链接至标题")
--------------------------------

usage: pyinstaller \[-h\] \[-v\] \[-D\] \[-F\] \[--specpath DIR\] \[-n NAME\] \[--add-data <SRC;DEST or SRC:DEST>\] \[--add-binary <SRC;DEST or SRC:DEST>\] \[-p DIR\] \[--hidden-import MODULENAME\] \[--collect-submodules MODULENAME\] \[--collect-data MODULENAME\] \[--collect-binaries MODULENAME\] \[--collect-all MODULENAME\] \[--copy-metadata PACKAGENAME\] \[--recursive-copy-metadata PACKAGENAME\] \[--additional-hooks-dir HOOKSPATH\] \[--runtime-hook RUNTIME_HOOKS\] \[--exclude-module EXCLUDES\] \[--key KEY\] \[--splash IMAGE_FILE\] \[-d {all,imports,bootloader,noarchive}\] \[--python-option PYTHON_OPTION\] \[-s\] \[--noupx\] \[--upx-exclude FILE\] \[-c\] \[-w\] \[-i <FILE.ico or FILE.exe,ID or FILE.icns or "NONE">\] \[--disable-windowed-traceback\] \[--version-file FILE\] \[-m <FILE or XML>\] \[--no-embed-manifest\] \[-r RESOURCE\] \[--uac-admin\] \[--uac-uiaccess\] \[--win-private-assemblies\] \[--win-no-prefer-redirects\] \[--osx-bundle-identifier BUNDLE_IDENTIFIER\] \[--target-architecture ARCH\] \[--codesign-identity IDENTITY\] \[--osx-entitlements-file FILENAME\] \[--runtime-tmpdir PATH\] \[--bootloader-ignore-signals\] \[--distpath DIR\] \[--workpath WORKPATH\] \[-y\] \[--upx-dir UPX_DIR\] \[-a\] \[--clean\] \[--log-level LEVEL\] scriptname \[scriptname ...\]

位置参数：
  scriptname            \# 要处理的脚本文件的名称或恰好是一个 .spec 文件。 如果指定了 .spec 文件，则大多数选项都是不必要的并被忽略。

可选参数：
  -h, --help            \# 显示此帮助信息并退出
  -v, --version         \# 显示程序版本信息并退出。
  --distpath DIR        \# 捆绑应用程序的放置位置（默认：./dist）
  --workpath WORKPATH   \# 将所有临时工作文件、.log、.pyz 等放在哪里（默认：./build）
  -y, --noconfirm       \# 替换输出目录（默认：SPECPATH/dist/SPECNAME）而不要求确认
  --upx-dir UPX_DIR     \# UPX 实用程序的路径（默认：搜索执行路径）
  -a, --ascii           \# 不包括 unicode 编码支持（默认：如果可用，包括在内）
  --clean               \# 在构建之前清理 PyInstaller 缓存并删除临时文件。
  --log-level LEVEL     \# 构建时控制台消息中的详细信息量。 LEVEL 可以是 TRACE、DEBUG、INFO、WARN、ERROR、CRITICAL 之一（默认值：INFO）。

生成什么：
  -D, --onedir          \# 创建一个包含可执行文件的单文件夹包（默认）
  -F, --onefile         \# 创建一个单一文件捆绑的可执行文件。
  --specpath DIR        \# 存放生成的spec文件的文件夹（默认：当前目录）
  -n NAME, --name NAME  \# 分配给捆绑的应用程序和规范文件的名称（默认值：第一个脚本的基本名称）

What to bundle, where to search:
  --add-data <SRC;DEST or SRC:DEST>
                        \# 要添加到可执行文件的其他非二进制文件或文件夹。 路径分隔符是特定于平台的，使用 \`\`os.pathsep\`\`（在 Windows 上是 ``;`` 而在大多数 unix 系统上是 ``:``）。 此选项可以多次使用。
  --add-binary <SRC;DEST or SRC:DEST>
                        \# 要添加到可执行文件的附加二进制文件。 有关更多详细信息，请参见 --add-data 选项。 此选项可以多次使用。
  -p DIR, --paths DIR   \# 搜索导入的路径（例如使用 PYTHONPATH）。 允许多个路径，由 ``':'`` 分隔，或多次使用此选项。 等效于在规范文件中提供 \`\`pathex\`\` 参数。
  --hidden-import MODULENAME, --hiddenimport MODULENAME
                        \# 命名在脚本代码中不可见的导入。 此选项可以多次使用。
  --collect-submodules MODULENAME
                        \# 从指定的包或模块中收集所有子模块。 此选项可以多次使用。
  --collect-data MODULENAME, --collect-datas MODULENAME
                        \# 从指定的包或模块中收集所有数据。 此选项可以多次使用。
  --collect-binaries MODULENAME
                        \# 从指定的包或模块中收集所有二进制文件。 此选项可以多次使用。
  --collect-all MODULENAME
                        \# 从指定的包或模块中收集所有子模块、数据文件和二进制文件。 此选项可以多次使用。
  --copy-metadata PACKAGENAME
                        \# 复制指定包的元数据。 此选项可以多次使用。
  --recursive-copy-metadata PACKAGENAME
                        \# 复制指定包及其所有依赖项的元数据。 此选项可以多次使用。
  --additional-hooks-dir HOOKSPATH
                        \# 搜索钩子的附加路径。 此选项可以多次使用。
  --runtime-hook RUNTIME_HOOKS
                        \# 自定义运行时挂钩文件的路径。 运行时挂钩是与可执行文件捆绑在一起的代码，在任何其他代码或模块之前执行以设置运行时环境的特殊功能。 此选项可以多次使用。
  --exclude-module EXCLUDES
                        \# 将被忽略的可选模块或包（Python 名称，而不是路径名称）（就好像它没有找到一样）。 此选项可以多次使用。
  --key KEY             \# 用于加密 Python 字节码的密钥。
  --splash IMAGE_FILE   \# (EXPERIMENTAL) 将带有图像 IMAGE_FILE 的启动画面添加到应用程序。 启动画面可以在解包时显示进度更新。

[参考文档](http://legendtkl.com/2015/11/06/pyinstaller/)

# PyInstaller  文档
简介[¶](#id1 "永久链接至标题")
---------------------

PyInstaller可以用来打包python应用程序，打包完的程序就可以在没有安装Python解释器的机器上运行了。PyInstaller支持Python 2.7和Python 3.3+。可以在Windows、Mac OS X和Linux上使用，但是并不是跨平台的，而是说你要是希望打包成.exe文件，需要在Windows系统上运行PyInstaller进行打包工作；打包成mac app，需要在Mac OS上使用。

安装更新[¶](#id2 "永久链接至标题")
-----------------------

\# 安装
$ pip install pyinstaller -i https://pypi.tuna.tsinghua.edu.cn/simple
...
Successfully installed altgraph-0.17.2 macholib-1.15.2 pyinstaller-4.9 pyinstaller-hooks-contrib-2022.2

\# 更新
$ pip install --upgrade pyinstaller -i https://pypi.tuna.tsinghua.edu.cn/simple

\# Windows上运行PyInstaller还需要PyWin32或者pypiwin32，其中pypiwin32在你安装PyInstaller的时候会自动安装。

使用[¶](#id3 "永久链接至标题")
---------------------

PyInstaller分析你的python程序，找到所有的依赖项。然后将依赖文件和python解释器放到一个文件夹下或者一个可执行文件中。

1.  打包成一个文件夹
    
    当使用PyInstaller打包的时候，默认生成一个文件夹，文件夹中包含所有依赖项，以及可执行文件。打包成文件夹的好处就是debug的时候可以清楚的看到依赖项有没有包含。另一个好处是更新的时候，只需要更新可执行文件就可以了。当然缺点也很明显，不方便，不易管理。
    
    那么它是如何工作的呢？PyInstaller的引导程序是一个二进制可执行程序。当用户启动你的程序的时候，PyInstaller的引导程序开始运行，首先创建一个临时的Python环境，然后通过Python解释器导入程序的依赖，当然他们都在同一个文件夹下。
    
2.  打包成一个文件
    
    我们可以用 `--onefile` 或者 `-F` 参数将所有文件打包到一个可执行文件中。
    
    $ pyinstaller -F script.py
    
    打包成一个文件相对于文件夹更容易管理。坏处运行相对比较慢。这个文件中包含了压缩的依赖文件拷贝（.so文件）。
    
    当程序运行时，PyInstaller的引导程序会新建一个临时文件夹。然后解压程序的第三方依赖文件到临时文件夹中。这也是为什么一个可执行文件比文件夹中执行的时间要长的原因。剩下的就和上面的一样了。
    
3.  spec 文件
    
    当你执行下面命令
    
    $ pyinstaller options..script.py
    
    PyInstaller首先建一个sepc(specification)文件：script.spec。这个文件的存放地址可以使用参数–specpath= 来定义，默认放在当前文件夹下。
    
    spec文件的作用是什么呢？它会告诉PyInstaller如何处理你的py文件，它会将你的py文件名字和输入的大部分参数进行编码。PyInstaller通过执行spec文件中的内容来生成app，有点像makefile。正常使用中我们是不需要管spec文件的，但是下面几种情况需要修改spec文件：
    
    *   需要打包资源文件
        
    *   需要include一些PyInstaller不知道的run-time库
        
    *   为可执行文件添加run-time 选项
        
    *   多程序打包
        
    
    可以通过下面命令生成spec文件
    
    $ pyi-makespec options script.py \[other scripts ...\]
    
    修改完spec文件，就可以通过下面命令来生成app文件了
    
    $ pyinstaller options script.spec
    
    当通过spec文件来生成app文件的时候只有下面几个参数是有用的：
    
    *   `–upx-dir=`
        
    *   `–distpath=`
        
    *   `–noconfirm=`
        
    *   `–ascii`
        
4.  spec 文件解析
    
    \# -*- mode: python ; coding: utf-8 -*-
    
    block_cipher = None
    a = Analysis(\['script.py'\],
                 pathex=\[\],
                 binaries=\[\],
                 datas=\[\],
                 hiddenimports=\[\],
                 hookspath=\[\],
                 hooksconfig={},
                 runtime_hooks=\[\],
                 excludes=\[\],
                 win\_no\_prefer_redirects=False,
                 win\_private\_assemblies=False,
                 cipher=block_cipher,
                 noarchive=False)
    pyz = PYZ(a.pure, a.zipped_data,
                 cipher=block_cipher)
    exe = EXE(pyz,
              a.scripts,
              a.binaries,
              a.zipfiles,
              a.datas,  
              \[\],
              name='script',
              debug=False,
              bootloader\_ignore\_signals=False,
              strip=False,
              upx=True,
              upx_exclude=\[\],
              runtime_tmpdir=None,
              console=True,
              disable\_windowed\_traceback=False,
              target_arch=None,
              codesign_identity=None,
              entitlements_file=None )
    coll = COLLECT(...)
    
    spec文件中主要包含4个class: `Analysis`, `PYZ`, `EXE`和`COLLECT`。
    
    *   `Analysis` 以py文件为输入，它会分析py文件的依赖模块，并生成相应的信息
        
    *   `PYZ` 是一个.pyz的压缩包，包含程序运行需要的所有依赖
        
    *   `EXE` 根据上面两项生成
        
    *   `COLLECT` 生成其他部分的输出文件夹，COLLECT也可以没有
        
5.  修改spec文件
    
    我们上面说过有时候PyInstaller自动生成的spec文件并不能满足我们的需求，最常见的情况就是我们的程序依赖我们本地的一些数据文件，这个时候就需要我们自己去编辑spec文件来添加数据文件了。 上面的spec文件解析中Analysis中的datas就是要添加到项目中的数据文件，我们可以编辑datas.
    
    a = Analysis(
        ...
        datas = \[('you/source/file/path','file\_name\_in_project'),
        ('source/file2', 'file_name2')\]
        ...
        )
    
    可以认为datas是一个List,每个元素是一个二元组。元组的第一个元素是你本地文件索引，第二个元素是拷贝到项目中之后的文件名字。除了上面那种写法，也可以将其提出来。
    
    added_files = \[...\]
    
    a = Analysis(
        ...
        datas = added_files,
        ...
        )
    
    其他的二进制文件添加方法类似。
    
6.  总结
    
    最后简单来说，我们要通过PyInstaller生成可执行的文件主要下面两步。
    
    $ pyinstaller \[option\] mypython.py
    
    option为空生成文件夹，选择onefile，生成一个文件。 如果项目有一些依赖的数据文件，上面生成的二进制文件是无法运行的，这个时候可以通过修改spec文件，让后再用pyinstaller运行spec文件。
    
    $ pyinstaller \[option\] mypython.spec
    
    当然也按上文那样先生成spec文件。
    

Pyinstaller解释[¶](#id4 "永久链接至标题")
--------------------------------

usage: pyinstaller \[-h\] \[-v\] \[-D\] \[-F\] \[--specpath DIR\] \[-n NAME\] \[--add-data <SRC;DEST or SRC:DEST>\] \[--add-binary <SRC;DEST or SRC:DEST>\] \[-p DIR\] \[--hidden-import MODULENAME\] \[--collect-submodules MODULENAME\] \[--collect-data MODULENAME\] \[--collect-binaries MODULENAME\] \[--collect-all MODULENAME\] \[--copy-metadata PACKAGENAME\] \[--recursive-copy-metadata PACKAGENAME\] \[--additional-hooks-dir HOOKSPATH\] \[--runtime-hook RUNTIME_HOOKS\] \[--exclude-module EXCLUDES\] \[--key KEY\] \[--splash IMAGE_FILE\] \[-d {all,imports,bootloader,noarchive}\] \[--python-option PYTHON_OPTION\] \[-s\] \[--noupx\] \[--upx-exclude FILE\] \[-c\] \[-w\] \[-i <FILE.ico or FILE.exe,ID or FILE.icns or "NONE">\] \[--disable-windowed-traceback\] \[--version-file FILE\] \[-m <FILE or XML>\] \[--no-embed-manifest\] \[-r RESOURCE\] \[--uac-admin\] \[--uac-uiaccess\] \[--win-private-assemblies\] \[--win-no-prefer-redirects\] \[--osx-bundle-identifier BUNDLE_IDENTIFIER\] \[--target-architecture ARCH\] \[--codesign-identity IDENTITY\] \[--osx-entitlements-file FILENAME\] \[--runtime-tmpdir PATH\] \[--bootloader-ignore-signals\] \[--distpath DIR\] \[--workpath WORKPATH\] \[-y\] \[--upx-dir UPX_DIR\] \[-a\] \[--clean\] \[--log-level LEVEL\] scriptname \[scriptname ...\]

位置参数：
  scriptname            \# 要处理的脚本文件的名称或恰好是一个 .spec 文件。 如果指定了 .spec 文件，则大多数选项都是不必要的并被忽略。

可选参数：
  -h, --help            \# 显示此帮助信息并退出
  -v, --version         \# 显示程序版本信息并退出。
  --distpath DIR        \# 捆绑应用程序的放置位置（默认：./dist）
  --workpath WORKPATH   \# 将所有临时工作文件、.log、.pyz 等放在哪里（默认：./build）
  -y, --noconfirm       \# 替换输出目录（默认：SPECPATH/dist/SPECNAME）而不要求确认
  --upx-dir UPX_DIR     \# UPX 实用程序的路径（默认：搜索执行路径）
  -a, --ascii           \# 不包括 unicode 编码支持（默认：如果可用，包括在内）
  --clean               \# 在构建之前清理 PyInstaller 缓存并删除临时文件。
  --log-level LEVEL     \# 构建时控制台消息中的详细信息量。 LEVEL 可以是 TRACE、DEBUG、INFO、WARN、ERROR、CRITICAL 之一（默认值：INFO）。

生成什么：
  -D, --onedir          \# 创建一个包含可执行文件的单文件夹包（默认）
  -F, --onefile         \# 创建一个单一文件捆绑的可执行文件。
  --specpath DIR        \# 存放生成的spec文件的文件夹（默认：当前目录）
  -n NAME, --name NAME  \# 分配给捆绑的应用程序和规范文件的名称（默认值：第一个脚本的基本名称）

What to bundle, where to search:
  --add-data <SRC;DEST or SRC:DEST>
                        \# 要添加到可执行文件的其他非二进制文件或文件夹。 路径分隔符是特定于平台的，使用 \`\`os.pathsep\`\`（在 Windows 上是 ``;`` 而在大多数 unix 系统上是 ``:``）。 此选项可以多次使用。
  --add-binary <SRC;DEST or SRC:DEST>
                        \# 要添加到可执行文件的附加二进制文件。 有关更多详细信息，请参见 --add-data 选项。 此选项可以多次使用。
  -p DIR, --paths DIR   \# 搜索导入的路径（例如使用 PYTHONPATH）。 允许多个路径，由 ``':'`` 分隔，或多次使用此选项。 等效于在规范文件中提供 \`\`pathex\`\` 参数。
  --hidden-import MODULENAME, --hiddenimport MODULENAME
                        \# 命名在脚本代码中不可见的导入。 此选项可以多次使用。
  --collect-submodules MODULENAME
                        \# 从指定的包或模块中收集所有子模块。 此选项可以多次使用。
  --collect-data MODULENAME, --collect-datas MODULENAME
                        \# 从指定的包或模块中收集所有数据。 此选项可以多次使用。
  --collect-binaries MODULENAME
                        \# 从指定的包或模块中收集所有二进制文件。 此选项可以多次使用。
  --collect-all MODULENAME
                        \# 从指定的包或模块中收集所有子模块、数据文件和二进制文件。 此选项可以多次使用。
  --copy-metadata PACKAGENAME
                        \# 复制指定包的元数据。 此选项可以多次使用。
  --recursive-copy-metadata PACKAGENAME
                        \# 复制指定包及其所有依赖项的元数据。 此选项可以多次使用。
  --additional-hooks-dir HOOKSPATH
                        \# 搜索钩子的附加路径。 此选项可以多次使用。
  --runtime-hook RUNTIME_HOOKS
                        \# 自定义运行时挂钩文件的路径。 运行时挂钩是与可执行文件捆绑在一起的代码，在任何其他代码或模块之前执行以设置运行时环境的特殊功能。 此选项可以多次使用。
  --exclude-module EXCLUDES
                        \# 将被忽略的可选模块或包（Python 名称，而不是路径名称）（就好像它没有找到一样）。 此选项可以多次使用。
  --key KEY             \# 用于加密 Python 字节码的密钥。
  --splash IMAGE_FILE   \# (EXPERIMENTAL) 将带有图像 IMAGE_FILE 的启动画面添加到应用程序。 启动画面可以在解包时显示进度更新。

[参考文档](http://legendtkl.com/2015/11/06/pyinstaller/)

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

# Pyinstaller的Spec文件用法
前言
--

当我们用py完成一些功能，可以通过Pyinstaller将源码打包成exe来独立运行，用户使用时只需要执行这个exe文件即可，不需要在机器上再安装Python及其他包就可运行了。（理论上，后面会附加一些我自己实际运用时遇到的一些问题）

Pyinstaller打包方式一般分为 直接输入指令 和 利用spec文件进行打包。由于直接输入指令实际就是根据指令生成spec文件，再根据spec文件的内容进行打包操作，所以一下重点说明spec文件的内容，结尾处附上指令以供参考。

SPEC打包
------

第一步当然是最基础的用法，我们先创建一个main.py作为启动脚本。在控制台输入

```
`pyinstaller [-F/-D] [-w/-c] [-i xxx.ico] main.py` 

*   1


```

我们可以发现路径下多了main.spec文件。后续我们可以修改spec文件里的内容，然后输入指令来进行打包操作了

```
`pyinstaller main.spec` 

*   1


```

后续打包的参数越来越多，每次输入一大堆参数显然不如直接使用spec来的高效，所以尽量使用spec文件进行打包操作。

### SPEC内容说明

下图就是对一个main.py进行打包时，默认生成的spec文件，我们来看下每个参数表示的含义  
![](_assets/20210105162403295.png)

| 变量 | 含义 |
| --- | --- |
| a | Analysis类的实例，要求传入各种脚本用于分析程序的导入和依赖。a中内容主要包括以下四部分：scripts，即可以在命令行中输入的Python脚本；pure，程序代码文件中的纯Python模块，包括程序的代码文件本身；binaries，程序代码文件中需要的非Python模块，包括–add-binary参数指定的内容；datas，非二进制文件，包括–add-data参数指定的内容。 |
| pyz | PYZ的实例，是一个.pyz文件，包含了所有pure中的所有Python模块。 |
| exe | EXE类的实例，这个类是用来处理Analysis和PYZ的结果的，也是用来生成最后的exe可执行程序。 |
| coll | COLLECT类的实例，用于创建输出目录。在-F模式下，是没有COLLECT实例的，并且所有的脚本、模块和二进制文件都包含在了最终生成的exe文件中。 |
| block_cipher | 加密密钥 |

以上内容中 修改比较多的是a、exe的内容，coll和pyz基本没有遇到需要修改的情况。

| 参数 | 含义 |
| --- | --- |
| Analysis参数scripts | 也是第一个参数，它是一个脚本列表，可以传入多个py脚本，效果与命令行中指定多py文件相同，即py文件不止一个时，比如“pyinstaller xxx1.py xxx2.py”，pyinstaller会依次分析并执行，并把第一个py名称作为spec和dist文件下的文件夹和程序的名称 |
| Analysis参数pathex | 默认有一个spec的目录，当我们的一些模块不在这个路径下，记得把用到的模块的路径添加到这个list变量里。同命令“-p DIR/–paths DIR”. |
| Analysis参数datas | 作用是将本地文件打包时拷贝到目标路径下。datas是一个元素为元组的列表，每个元组有两个元素，都必须是字符串类型，元组的第一个元素为数据文件或文件夹，元组的第二个元素为运行时这些文件或文件夹的位置。例如：datas=\[(’./src/a.txt’, ‘./dst’)\]，表示打包时将"./src/a.txt"文件添加（copy）到相对于exe目录下的dst目录中。也可以使用通配符：datas= \[ (’/mygame/sfx/*.mp3’, ‘sfx’ ) \]，表示将/mygame/sfx/目录下的所有.mp3文件都copy到sfx文件夹中。也可以添加整个文件夹：datas= \[ (’/mygame/data’, ‘data’ ) \]，表示将/mygame/data文件夹下所有的文件都copy到data文件夹下。同命令“–add-data”。 |
| Analysis参数binaries | 添加二进制文件，也是一个列表，定义方式与datas参数一样。没具体使用过。同命令“–add-binary”。 |
| Analysis参数hiddenimports | 指定脚本中需要隐式导入的模块，比如在\_\_import\_\_、imp.find_module()、exec、eval等语句中导入的模块，这些模块PyInstaller是找不到的，需要手动指定导入，这个选项可以使用多次。同命令“–hidden-import MODULENAME/–hiddenimport MODULENAME”。 |
| Analysis参数hookspath | 指定额外hook文件（可以是py文件）的查找路径，这些文件的作用是在PyInstaller运行时改变一些Python或者其他库原有的函数或者变量的执行逻辑（并不会改变这些库本身的代码），以便能顺利的打包完成，这个选项可以使用多次。同命令“–additional-hooks-dir HOOKSPATH”。 |
| Analysis参数runtime_hooks | 指定自定义的运行时hook文件路径（可以是py文件），在打好包的exe程序中，在运行这个exe程序时，指定的hook文件会在所有代码和模块之前运行，包括main文件，以满足一些运行环境的特殊要求，这个选项可以使用多次。同命令“–runtime-hook RUNTIME_HOOKS”。 |
| Analysis参数excludes | 指定可以被忽略的可选的模块或包，因为某些模块只是PyInstaller根据自身的逻辑去查找的，这些模块对于exe程序本身并没有用到，但是在日志中还是会提示“module not found”，这种日志可以不用管，或者使用这个参数选项来指定不用导入，这个选项可以使用多次。同命令“–exclude-module EXCLUDES”。 |
| exe参数console | 设置是否显示命令行窗口，同命令-w/-c。 |
| exe参数icon | 设置程序图标，默认spec是没有的，需要手动添加，参数值就是图片路径的字符串。同命令“命令-i/–icon”。 |

一些实际应用的问题
---------

### 关于动态加载的module导入问题

有些脚本中，我们使用了\_\_import\_\_等一些动态导入库的函数时，pyinstaller打包是无法找到这些模块的，在使用时就会报"ImportError 找不到指定模块"，这个时候就需要手动导入动态加载的模块。

在以下结构中,我们\_\_import\_\_(‘A.AA’)

*   A
    *   AA
        *   AAA
        *   AAB
    *   AB
        *   ABA
        *   ABB

#### 直接在spec中声明

在"Analysis参数hiddenimports"中添加需要导入模块,hiddenimports=\[‘A.AA’\]。

#### 通过hook脚本导入

1.在spec中，使hiddenimports=\[‘A’\]，  
2.名称按照"hooks-xxx.py"的方式创建一个脚本，此时我们创建一个"hooks-A.py"， 脚本的主要内容为给"hiddenimports"这个list赋值,可以直接赋值，也可以写逻辑（完全可以看成一个py脚本）。

```
`hiddenimports = ['A','A.AA']` 

*   1


```

3.填写spec中的hookspath参数，值为"hooks-xxx.py"的路径。

完成以上3步，就可以在打包的时候带上动态加载的脚本了。

### runtime_hooks参数的使用

runtime_hooks参数可以跟hookspath一样，指定脚本，当我们运行打包的exe时，会在exe执行前，先将参数中对应的脚本先执行一次。可以用作创建exe运行的一些环境。

Pyinstaller库中有一些自带的runtime_hooks、hookspath脚本，可以去找来看以下。

### 打包后exe无法运行的问题

#### 管理员权限问题

放C盘就无法正常执行脚本，这个问题暂时没找到答案。就放D盘吧

#### 部分PC无法使用

其他PC上exe都可以正常运行，但是有一台就是不行，把控制台打开查看打印，报的是ImportError，这肯定不可能。  
后来查到exe还需要vc环境，所以就安装了个vc运行时库，问题就解决了。  
![](_assets/20210106090709335.png)

加密问题
----

上面还有个变量block_cipher，主要是防止exe被反编译。

```
`block_cipher = pyi_crypto.PyiBlockCipher(key='123456789')` 

*   1


```

下面这边文章写的很详细，我也通过这位作者的方法成功进行了加密。  
[加密](https://mp.weixin.qq.com/s?__biz=MzAxMTkwODIyNA==&mid=2247507585&idx=2&sn=49927935a8103123e9138d8dc0665c1c&chksm=9bbb7b6eacccf278ee22791d05804180fddeacd2bcbad4974fb84023985a34b379c262dabc92&xtrack=1&scene=0&subscene=10000&clicktime=1602025341&enterid=1602025341&ascene=7&devicetype=android-29&version=3.0.36.2008&nettype=WIFI&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&exportkey=AaKPf3QPE7vXCst05p67Hzc%3D&pass_ticket=ZEu6DU5vbKl5LzJ%2BB0psE4ZzaQt5Ay8207BKqROAE3QaDt0iwhDRvtvgNurGX1t1&wx_header=1&platform=win).