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
![](https://img-blog.csdnimg.cn/20210105162403295.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RhbmdmcmVlemU=,size_16,color_FFFFFF,t_70)

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
![](https://img-blog.csdnimg.cn/20210106090709335.png)

加密问题
----

上面还有个变量block_cipher，主要是防止exe被反编译。

```
`block_cipher = pyi_crypto.PyiBlockCipher(key='123456789')` 

*   1


```

下面这边文章写的很详细，我也通过这位作者的方法成功进行了加密。  
[加密](https://mp.weixin.qq.com/s?__biz=MzAxMTkwODIyNA==&mid=2247507585&idx=2&sn=49927935a8103123e9138d8dc0665c1c&chksm=9bbb7b6eacccf278ee22791d05804180fddeacd2bcbad4974fb84023985a34b379c262dabc92&xtrack=1&scene=0&subscene=10000&clicktime=1602025341&enterid=1602025341&ascene=7&devicetype=android-29&version=3.0.36.2008&nettype=WIFI&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&exportkey=AaKPf3QPE7vXCst05p67Hzc%3D&pass_ticket=ZEu6DU5vbKl5LzJ%2BB0psE4ZzaQt5Ay8207BKqROAE3QaDt0iwhDRvtvgNurGX1t1&wx_header=1&platform=win).