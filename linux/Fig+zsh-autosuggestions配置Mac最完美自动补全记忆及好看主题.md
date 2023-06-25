# Fig+zsh-autosuggestions配置Mac最完美自动补全记忆及好看主题 

> 前言：由于并不经常使用命令行，且常用的就是几个git和进入文件夹的命令，平常使用Mac自带的`terminal`满足日常需要一句足够，但是在一次偶然使用了`iterm2`后大呼真香，看着舒心还使用更加便利

本文摘抄自： [Mac OS 终端利器 iTerm2](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fxishuai%2Fp%2Fmac-iterm2.html)

[https://www.cnblogs.com/xishuai/p/mac-iterm2.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fxishuai%2Fp%2Fmac-iterm2.html)  
参考：  
[MacOS 终端工具、插件推荐](https://www.jianshu.com/p/c9040b4321ae)  
[\[Mac\] Fig ：好用的终端工具](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.macsofter.com%2F27339.html)  
[https://fig.io/](https://links.jianshu.com/go?to=https%3A%2F%2Ffig.io%2F)

###### 步骤一：安装iterm2

使用命令行安装（前提是要安装好homebrew）

```
brew install --cask iterm2 
```

或者直接到官网下载iterm软件解压后安装  
[https://iterm2.com/downloads.html](https://links.jianshu.com/go?to=https%3A%2F%2Fiterm2.com%2Fdownloads.html)

###### 步骤二：配置好看的主题

常用的主题Solarized Dark theme，直接到github官网下载到本地  
[https://github.com/altercation/solarized.git](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Faltercation%2Fsolarized.git)  
下载使用到的字体：  
[Meslo LG M Regular for Powerline.ttf](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fpowerline%2Ffonts%2Fblob%2Fmaster%2FMeslo%2520Slashed%2FMeslo%2520LG%2520M%2520Regular%2520for%2520Powerline.ttf)  
安装好字体

导入将下载好的主题  
打开`iTerm2`，，打开 `Preferences` 配置界面  
1、然后`Profiles` -\> `Colors` -\> `Color Presets` -\> `Import`，  
选择刚才解压的  
2、`solarized`->`iterm2-colors-solarized`->`Solarized Dark.itermcolors`文件  
导入成功，最后选择 Solarized Dark 主题，就可以了。  

![](_assets/1605558-1280811c421c495c.png.webp)

image.png

3、设置`solarized`用到的特殊字体`Meslo LG M Regular for Powerline`  
打开 Preferences 配置界面，  
然后`Profiles` -\> `Text`-\> `Font` -\> `Chanage Font`  
选择 Meslo LG M Regular for Powerline 字体。  

![](_assets/1605558-4d343bf48d0f0e5b.png.webp)

image.png

###### 步骤三：配置 Oh My Zsh

Oh My Zsh 是对主题的进一步扩展，地址：[https://github.com/robbyrussell/oh-my-zsh](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Frobbyrussell%2Foh-my-zsh)

一键安装：

```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" 
```

安装好之后，需要把 Zsh 设置为当前用户的默认 Shell（这样新建标签的时候才会使用 Zsh）：

然后，我们编辑vim ~/.zshrc文件，将主题配置修改为ZSH_THEME="agnoster"。  

![](_assets/1605558-a4b0f7709f8abc75.png.webp)

image.png

  
`agnoster`是比较常用的 zsh 主题之一，你可以挑选你喜欢的主题，zsh 主题列表：[https://github.com/robbyrussell/oh-my-zsh/wiki/themes](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Frobbyrussell%2Foh-my-zsh%2Fwiki%2Fthemes)

###### 步骤四：配置高亮

效果就是上面截图的那样，特殊命令和错误命令，会有高亮显示。

使用 Homebrew 安装：

```
brew install zsh-syntax-highlighting 
```

安装成功之后，编辑vim ~/.zshrc文件，在最后一行增加下面配置：

```
# 找到zsh-syntax-highlighting.zsh的位置,可能是下面的其中一个，我最细安装的就已经是第二个，前段时间安装的就是第一个，查看下具体是那个就可以
source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
# 我最新M1的Mac mini安装的位置变成了另一个地址，把上面的换一下就好了
source /opt/homebrew/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh 
```

![](_assets/1605558-cb875a9e50a7ce59.png.webp)

image.png

完成后效果如下：

  

![](_assets/1605558-e871a20b6f6db81a.png.webp)

image.png

###### 步骤五：配置自动建议填充

这个功能是非常实用的，可以方便我们快速的敲命令。

配置步骤，先克隆`zsh-autosuggestions`项目，到指定目录：

```
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions 
```

然后编辑`vim ~/.zshrc`文件，找到plugins配置，增加`zsh-autosuggestions`插件。  

![](_assets/1605558-9b9eda9ed3a22b4b.png.webp)

image.png

注：上面声明高亮，如果配置不生效的话，在plugins配置，再增加zsh-syntax-highlighting插件试试。

有时候因为自动填充的颜色和背景颜色很相似，以至于自动填充没有效果，我们可以手动更改下自动填充的颜色配置，我修改的颜色值为：586e75，示例：

  

![](_assets/1605558-14749644a6546407.png.webp)

image.png

效果：

  

![](_assets/1605558-c604721dcd8c5959.png.webp)

image.png

iTerm2 隐藏用户名和主机名  
有时候我们的用户名和主机名太长，比如我的xishuai@xishuaideMacBook-Pro，终端显示的时候会很不好看（上面图片中可以看到），我们可以手动去除。

编辑`vim ~/.zshrc`文件，增加`DEFAULT_USER="xishuai"`配置，示例  
我们可以通过`whoami`命令，查看当前用户，效果（另外分屏的效果）：

* * *

本文参考：[10 个 Terminal 主题，让你的 macOS 终端更好看](https://links.jianshu.com/go?to=https%3A%2F%2Fsspai.com%2Fpost%2F53008)  
配置了iterm2后的主题agnoster后，会影响terminal开头的地方出现乱码，这是因为使用了特殊主题后terminal不支持，现在讲terminal也配上这些主题

1、导入主题  
`终端`->`偏好设置`->`偏好设置`->`描述文件`->`更多`->`导入`  
选择  
`solarized`->`osx-terminal.app-colors-solarized`->`Solarized Dark ansi.terminal`文件  

![](_assets/1605558-058c84ffe04f752f.png.webp)

导入主题\[图片上传中...(截屏2022-02-11 下午4.58.29.png-8919ba-1644570516138-0)\]

2、设置为默认主题  
选中 solarized Dark，然后点击右下角的默认

3、设置字体  
选中 solarized Dark，然后点击右侧的字体->选中之前安装好的的`Meslo LG M Regular for Powerline`字体  

![](_assets/1605558-f16348f886c65162.png.webp)

修改字体设置

4、设置背景半透明特效，当然也可以直接忽略

  

![](_assets/1605558-9a6c1f8187fa15a7.png.webp)

半透明背景设置

5、设置terminal在打开新的tab和window时也是设置的这个默认漂亮主题  
`terminal`->`shell`->`新建窗口`->`Solarized Dark ansi`  

![](_assets/1605558-950741dd3ac10474.png.webp)

新窗口设置

`terminal`->`shell`->`新建标签页`->`Solarized Dark ansi`  

![](_assets/1605558-b54362d053075197.png.webp)

设置新标签页设置

`terminal`->`shell`->`将设置用作默认设置`  

![](_assets/1605558-7da898e12a71d810.png.webp)

设置默认

退出terminal，重新进入就一切都OK

* * *

#### VSCode终端乱码

参考：  
[解决Mac下VSCode打开zsh乱码](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.cn%2Fpost%2F6844903895127293959)  
[VSCode更改显示语言-如更改英语为中文或者将中文改为英语](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fsinat_34104446%2Farticle%2Fdetails%2F83033510)

安装完成后，进入vscode里的终端，还是会乱码，需要在设置下字体  
`⌘,`->`font`->`Font Family` 添加字体：`Meslo LG M for Powerline`

![](_assets/1605558-338eb8daf56e3a7b.png.webp)

中文示意图

![](_assets/1605558-de4eacdc1031cbcf.png.webp)

