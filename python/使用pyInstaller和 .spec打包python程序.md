# 使用pyInstaller和*.spec打包python程序
当我们用 py 完成一些功能，可以通过 Pyinstaller 将源码打包成 exe 来独立运行，用户使用时只需要执行这个 exe 文件即可，不需要在机器上再安装 Python 及其他包就可运行了。

Pyinstaller 打包方式一般分为 直接输入指令 和 利用 spec 文件进行打包。由于直接输入指令实际就是根据指令生成 spec 文件，再根据 spec 文件的内容进行打包操作，所以一下重点说明 spec 文件的内容。

这个用 pip 模块直接下载就行，直接就下载在本次需要打包的 Python 环境下（base 环境）

先使用下述命令，创建一个spec文件，x1main是主程序的x1main.py名字

```

```

spec文件格式说明：

| 

变量

 | 

含义

 |
| 

a

 | 

Analysis类的实例，要求传入各种脚本用于分析程序的导入和依赖。a中内容主要包括以下四部分：

**scripts ，即可以在命令行中输入的Python脚本，*.py文件；**

pure，程序代码文件中的纯Python模块，包括程序的代码文件本身；

binaries，程序代码文件中需要的非Python模块，包括–add-binary参数指定的内容；

**datas: 非*.py文件，其它静态文件，例如jpg，png，txt，dll文件等。** 非二进制文件，包括–add-data参数指定的内容。

 |
| 

Pyz

 | 

PYZ的实例，是一个.pyz文件，包含了所有pure中的所有Python模块。

 |
| 

Exe

 | 

EXE类的实例，这个类是用来处理Analysis和PYZ的结果的，也是用来生成最后的exe可执行程序。

一般修改：

**Name ：打包后的exe程序名字，例如：智盟科技AI.exe，就输入name=“智盟科技AI”**

**Icon ：打包后的exe程序的图标。** 

 |
| 

Coll

 | 

COLLECT类的实例，用于创建输出目录。在-F模式下，是没有COLLECT实例的，并且所有的脚本、模块和二进制文件都包含在了最终生成的exe文件中。

 |
| 

block_cipher

 | 

加密密钥

 |

实际例子：

1.如果要打包多py文件，依次加到下图第一个红框的\[ \]内，第一个是主程序的名字。

2.对于非py文件，依次加入到datas=\[ \]内，前面是文件名，后面是目标目录名称。

  ![](_assets/e4d108e485fe8cb1.png)

![](_assets/ec5eb272c4cc6ac2.png)

输入如下格式的命令即可

然后会产生两个目录，build和dist，生成的文件在dist目录中，包括新产生的exe文件、依赖的python的包、静态的程序文件。

![](_assets/db418b69f77ec190.png)

**基于python研发的智盟AI视觉识别系统** 基于深度学习技术 (DL，Deep Learning)应用，其核心是 Inteance UniReco 模型 ，通过将输入图像进行分 patch 的操作，送入到 encoder （编码器）中，将 RGB 三维彩色空间进行自动可学习的特征提取，包括纹理、结构、颜色等多达 2048 维特征。 相比于传统的图像处理算法的低维度、纯手动方式，该模型能够捕获更多潜在以及非视觉直观感知的特征用于视觉多种识别任务。

![](_assets/1ff081b4-1886a7cc7c7-0006-c155-e72-64f3f.mp4.jpg)

来自 “ ITPUB博客 ” ，链接：http://blog.itpub.net/725190/viewspace-2955439/，如需转载，请注明出处，否则将追究法律责任。