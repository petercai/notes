# stable diffusion (novelai) linux 服务器安装并本地（笔记本）使用教程 - 知乎
对于没有足够大独立显卡的人来说，想玩一玩stable diffusion二次元作画的一个选择就是在服务器上搭建stable-diffusion-webui，然后通过端口转接到笔记本（或者手机）上进行ai作画。

那就直接开始安装。

1.默认安装了conda（没有的话装一下），用conda创建一个虚拟环境

```bash
# 创建环境
conda create -n stabel python=3.10.6
# 激活环境
conda activate stabel 
```

2.新建文件夹，下载stable-diffusion-webui

```bash
mkdir /stable
cd /stable
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
```

3.安装依赖包

```text
cd /stable/stable-diffusion-webui

# 安装pytorch
# 这里pytorch的版本是1.12.1，对应的cuda版本是11.3，按理说cuda driver应该要是11.3版本，但是我的是11.2也没有问题。
# 可以使用nvidia-smi看自己的cuda driver版本，在pytorch官网下载https://pytorch.org/
pip install torch==1.12.1+cu113 torchvision==0.13.1+cu113 --extra-index-url https://download.pytorch.org/whl/cu113 --trusted-host download.pytorch.org -i https://pypi.doubanio.com/simple

# 安装一些必要的包
pip install -r requirements_versions.txt --prefer-binary -i https://pypi.douban.com/simple/

pip install git+https://gitee.com/hznn/GFPGAN.git --prefer-binary
pip install git+https://gitee.com/hznn/CLIP.git --prefer-binary
pip install git+https://gitee.com/jerrylinkun/DeepDanbooru.git --prefer-binary


# git clone https://github.com/CompVis/stable-diffusion.git ./repositories/stable-diffusion
git clone https://gitee.com/jerrylinkun/stable-diffusion.git ./repositories/stable-diffusion


git clone https://github.com/CompVis/taming-transformers.git ./repositories/taming-transformers
git clone https://github.com/crowsonkb/k-diffusion.git ./repositories/K-diffusion

# github.com.cnpmjs.org
git clone https://gitee.com/arcsion/CodeFormer.git ./repositories/CodeFormer
git clone https://github.com/salesforce/BLIP.git ./repositories/BLIP

pip install -r ./repositories/CodeFormer/requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install opencv-python-headless -i https://pypi.douban.com/simple/

pip install git+https://gitee.com/ufhy/open_clip.git --prefer-binary
```

xformers的安装（暂时没有解决办法，不安装也能运行），这个是用来加速图片生成的。xformers的安装需要特别注意，我之前尝试了很多安装都不成功其中问题包括：

[https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/2047](https://link.zhihu.com/?target=https%3A//github.com/AUTOMATIC1111/stable-diffusion-webui/issues/2047)

[https://github.com/facebookresearch/xformers/issues/390](https://link.zhihu.com/?target=https%3A//github.com/facebookresearch/xformers/issues/390)

[https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/3259](https://link.zhihu.com/?target=https%3A//github.com/AUTOMATIC1111/stable-diffusion-webui/issues/3259) 问题解决，对我没用

[https://github.com/facebookresearch/xformers](https://link.zhihu.com/?target=https%3A//github.com/facebookresearch/xformers) xformers 项目主页，推荐conda安装

[https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Xformers](https://link.zhihu.com/?target=https%3A//github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Xformers) stable-diffusion-webui给出的linux安装方法，对我没用

```text
pip install xformers==0.0.12 --prefer-binary -i https://pypi.douban.com/simple/
```

4.下载模型参数文件

下载一个.ckpt文件（模型参数文件）放到/stable/stable-diffusion-webui/models/Stable-diffusion/文件下

novelai模型参数下载链接：

```text
magnet:?xt=urn:btih:5bde442da86265b670a3e5ea3163afad2c6f8ecc 
```

只需下载stableckpt目录下的animefull-final-pruned文件夹以及animevae.pt文件（可选：/modules/modules目录下的.pt文件也可以下载，默认加载模型不需要这些，不过它们可以提供独特结果）

5.运行，这里已经可以生成图像了

```text
python launch.py --listen --port 8080
```

![](https://pic3.zhimg.com/v2-1c31e9e04228baa7016b0583f22b862e_b.jpg)

生成的图像

如果出现报错 No module 'xformers'. Proceeding without it， 可以在launch.py文件中添加一行 commandline\_args = os.environ.get('COMMANDLINE\_ARGS', "--xformers")

如果出现报错 Error No module named 'triton'，可以运行 pip install triton --prefer-binary -i [https://pypi.douban.com/simple/](https://link.zhihu.com/?target=https%3A//pypi.douban.com/simple/)

之后还可能报错：

![](https://pic4.zhimg.com/v2-247f6ad49d39496fbad2a16b7faaeb2b_b.jpg)