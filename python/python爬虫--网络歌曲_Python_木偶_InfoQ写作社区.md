# python爬虫--网络歌曲_Python_木偶_InfoQ写作社区
**前言:想听音乐又不想去搜索歌曲,面对那么多音乐却不知道听哪个,今天本博主教你用 python 爬虫,轻松爬取热歌榜,一起来深究深究:**

![](_assets/477b5c4d5cdf358cda4bf0116389dc3c.png)

**在进行数据爬取时,我们需要对网页进行分析,对网页内容需要进行解析,这时候我们需要几个必备的包:**

1.  `requests`:请求网页,接受网页数据
    
2.  `etree`:解析网页
    

导入格式如下:

**如果包没有下载,可以使用如下命令分别去下载这两个库(可以在**`**DOS**`**命令窗口输入以下命令来下载):**

![](_assets/b8bdf5ad0ab2872ae16b2638e712dfec.png)

1.网络状态码:
--------

**一般网络状态有以下几种:**

2.目标网址获取:
---------

**首先我们要获取的是热歌榜的歌曲,那么我们首先得知道这个网址:**

3.获取头信息:
--------

**一般在进行爬虫时,为了防止反爬机制,我们需要将我们伪装成浏览器身份去进行访问网页,那么怎么进行伪装呢?就是修改我们的访问身份:**

1.  进入网页后按`F12`;
    
2.  然后选择`NetWork`;
    
3.  按`F5`进行页面刷新;
    
4.  点击`music.163.com`;
    
5.  选择`最底下的`进行复制;
    

4.进行代码访问:
---------

```
import requests
from lxml import etree
import os
url='https://music.163.com/discover/toplist?id=3778678'
head={
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.92 Safari/537.36'
}
down_url='https://music.163.com/song/media/outer/url?id='
respone=requests.get(url,headers=head)
print(respone)
```

**下图所示结果就是访问正常了,下面我们来解析网页...**

![](_assets/115ec3cbfe0d14310181fb6cfc64c13e.png)

**解析网页一般用到的是:**

```
html=etree.HTML(respone.text)
```

![](_assets/ce6f525b83aa81eac89f7e56372fc56d.png)

**在我们进行检查时,发现歌曲的主要信息在这里,那么格式我们可以这样写:**

```
id_list=html.xpath('//a[contains(@href,"song?")]')
```

`a标签中包含href中song?及以后的数据`**获取到的信息如下:**

![](_assets/9e7f5691c99f89bef634544757a7860d.png)

**是不是感觉很奇怪和想象的不太一样,那是我们获取到的地址,那么接下来我们进行进一步的提取:**

```
for id in id_list:
    href=id.xpath('./@href')[0]
    print(href)
```

**对获取到的地址进行遍历,取到其 href 中的值:**

![](_assets/53194f40ae78533ede346f717dc4d56e.png)

**我们主要的是 id 后面的那串数字,那么这时候我们对其进行分割,取到我们想要的数据:**

```
music_id=href.split('=')[1]
print(music_id)
```

![](_assets/ddfb0098e909fe1701855640f45e9aad.png)

```
if "$" not in music_id:
  music_name=id.xpath('./text()')[0]
  print(music_name)
```

**也许会有小伙伴比较疑问为什么要用**`**if**`**这个判断条件,原因如下:**

![](_assets/f758f0c3d30cf4f8883a69184aa18935.png)

**这是因为在进行**`**id**`**获取时,有的**`**id**`**是没有的,字符串里**`**包含$**`**,如果我们不进行处理,那么就会如下错误:**

![](_assets/b9051b90cf3bc174c9727854d3c429d3.png)

`索引错误`,**因为里面**`**没有这些包含$的id**`**,所以就会报错,名字获取到了那么我们来对歌曲进行下载并存储.**

1.外链:
-----

**每个歌曲都有自己的下载链接,那么我们采用这个链接并且配合我们获取到的 id 就可以进行下载啦:**

```
down_url='https://music.163.com/song/media/outer/url?id='
```

2.拼接链接:
-------

```
music_url=down_url + music_id
music=requests.get(url=music_url,headers=head)
print(music_url)
```

**这样获取到的就是我们可以下载的链接了,运行结果:**

![](_assets/f11386db991d534b584c206d9ae4945b.png)

![](_assets/51f0a9f17aa8b8664dacb68199abc3e2.png)

3.进行存储:
-------

```
if not os.path.exists(r'C:\Users\jcgg-99977\Desktop\music'):
            os.mkdir(r'C:\Users\jcgg-99977\Desktop\music')
        else:
            with open(r'C:\Users\jcgg-99977\Desktop\music/%s.mp3' % music_name, "wb") as f:
                print("正在下载歌曲 《%s》 ..." % music_name)
                f.write(music.content)
```

**解释代码:**

1.  先`查找`文件夹,如果`不存在则进行创建`;
    
2.  创建之后打开文件夹,将获取到的文件以刚才`获取到的名字进行命名`,然后`后缀名改为.mp3`格式;
    
3.  以`二进制形式进行写入`数据文件;
    

```
import requests
from lxml import etree
import os   
url='https://music.163.com/discover/toplist?id=3778678'
head={
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.92 Safari/537.36'
}
down_url='https://music.163.com/song/media/outer/url?id='
respone=requests.get(url,headers=head)

html=etree.HTML(respone.text)
id_list=html.xpath('//a[contains(@href,"song?")]')

for id in id_list:
    href=id.xpath('./@href')[0]
    
    music_id=href.split('=')[1]
    
    if "$" not in music_id:
        music_name=id.xpath('./text()')[0]
        
        music_url=down_url + music_id
        music=requests.get(url=music_url,headers=head)
        
        if not os.path.exists(r'C:\Users\jcgg-99977\Desktop\music'):
            os.mkdir(r'C:\Users\jcgg-99977\Desktop\music')
        else:
            with open(r'C:\Users\jcgg-99977\Desktop\music/%s.mp3' % music_name, "wb") as f:
                print("正在下载歌曲 《%s》 ..." % music_name)
                f.write(music.content)
```

* * *

**注解:由于本博主网速原因就不进行演示了,但是得注意一点:**

```
r'C:\Users\jcgg-99977\Desktop\music/%s.mp3'
```

1.  `r`:防止位置中的有些字符被转义;
    
2.  这是我的桌面地址,如果复制粘贴肯定不行,得`改成自己的用户名`哦!
    

运行结果应该是这样的:

![](_assets/07b71d20b949159bc3f1cf01556a4330.png)

* * *

* * *

**若有不足,请多多提出意见,虚心学习!**