# pdf文本和表格处理——pdfplumber的安装与简单使用_pdfplumber的debug
最新推荐文章于 2024-09-23 11:26:50 发布

![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[July M](https://blog.csdn.net/Elaine_jm "July M") ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCurrentTime2.png)
 于 2018-12-13 03:09:55 发布

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

pdf的文本和表格处理用多种方式可以实现， 本文介绍pdfplumber对文本和表格提取。这个库在[GitHub](https://so.csdn.net/so/search?q=GitHub&spm=1001.2101.3001.7020)上星300多，不过使用起来很方便， 效果也很好，可以满足对pdf中信息的提取需求。

pdfplumber简介
------------

Pdfplumber是一个可以处理pdf格式信息的库。可以查找关于每个文本字符、矩阵、和行的详细信息，也可以对表格进行提取并进行可视化调试。

pdfplumber安装
------------

安装直接采用[pip](https://edu.csdn.net/cloud/sd_summit?utm_source=glcblog&spm=1001.2101.3001.7020)即可。命令行中输入

> pip install pdfplumber

如果要进行可视化的调试，则需要安装ImageMagick。  
Pdfplumber GitHub： [https://github.com/jsvine/pdfplumber](https://github.com/jsvine/pdfplumber)  
ImageMagick地址：  
[http://docs.wand-py.org/en/latest/guide/install.html#install-imagemagick-windows](http://docs.wand-py.org/en/latest/guide/install.html#install-imagemagick-windows)  
（**官网地址没有6x， 6x地址**：[https://imagemagick.org/download/binaries/）](https://imagemagick.org/download/binaries/%EF%BC%89)

（**注意**：我在装ImageMagick，使用起来是报错了， 网上参照了[这里](https://blog.csdn.net/blmoistawinde/article/details/82051915) 了解到应该装6x版，7x版会报错。故找了6x的地址如上。）

在使用to\_image[函数](https://marketing.csdn.net/p/3127db09a98e0723b83b2914d9256174?pId=2782?utm_source=glcblog&spm=1001.2101.3001.7020)输出图片时，如果报错`DelegateException`。则安装GhostScript 32位。（**注意**，一定要下载32位版本，哪怕Windows和python的版本是64位的。）[1](#fn1)  
GhostScript: [https://www.ghostscript.com/download/gsdnld.html](https://www.ghostscript.com/download/gsdnld.html)

简单使用
----

最基本的用法如下，读取pdf中的某一页。

```
`import pdfplumber
with pdfplumber.open("path/to/file.pdf") as pdf:
    first_page = pdf.pages[0]
    print(first_page.chars[0])` 

*   1
*   2
*   3
*   4


```

pdfplumber.pdf中包含了.metadata和.pages两个属性。  
.metadata是一个包含pdf信息的字典。  
.pages是一个包含页面信息的列表。

每个pdfplumber.page的类中包含了几个主要的属性。  
.page\_number 页码  
.width 页面宽度  
.height 页面高度  
.objects/.chars/.lines/.rects 这些属性中每一个都是一个列表，每个列表都包含一个字典，每个字典用于说明页面中的对象信息， 包括直线，字符， 方格等位置信息。

### 一些常用的方法

.extract\_text() 用来提页面中的文本，将页面的所有字符对象整理为的那个字符串  
.extract\_words() 返回的是所有的单词及其相关信息  
.extract\_tables() 提取页面的表格  
.to\_image() 用于可视化调试时，返回PageImage类的一个实例

```
`import pdfplumber
import pandas as pd

with pdfplumber.open("中新科技：2015年年度报告摘要.PDF") as pdf:
    page = pdf.pages[1]   
    text = page.extract_text()
    print(text)
    table = page.extract_tables()
    for t in table:
        
        df = pd.DataFrame(t[1:], columns=t[0])
        print(df)` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12


```

显示文本：  
![](https://i-blog.csdnimg.cn/blog_migrate/3e775b700e767cc60bec80990b3343f4.png)

显示表格：  
![](https://i-blog.csdnimg.cn/blog_migrate/0e0d3dc1dba3876ff7eef202042aaf12.png)

### Visual Debugging可视化调试

Pdfplumber提供的图形debug功能，能够获取pdf页面中的表格，并对识别的信息用框框起来，并进行调整，以优化识别的情况。

具体可参照GitHub上给出的示例。  
在 [https://github.com/jsvine/pdfplumber/tree/master/examples](https://github.com/jsvine/pdfplumber/tree/master/examples)

![](https://i-blog.csdnimg.cn/blog_migrate/b521f1bfd085d438bdd85024fcdebd63.png)

* * *

1.  [https://blog.csdn.net/blmoistawinde/article/details/82051915](https://blog.csdn.net/blmoistawinde/article/details/82051915) [↩︎](#fnref1)