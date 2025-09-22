# 基本概念介绍

## 镜像

- Docker 镜像就是一个只读的模板。
- 镜像可以用来创建 Docker 容器。
- Docker 提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

## 容器

> 可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

- Docker 利用容器来运行应用。
- 容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

## 仓库

> 仓库是集中存放镜像文件的场所。有时候会把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。

- 仓库分为公开仓库（Public）和私有仓库（Private）两种形式
-
## 常用的docker命令

- `docker pull`

> 获取镜像

  `$ docker pull [OPTIONS] Name[:TAG] ` 
  `$ docker pull ubuntu:14.04` # 获取ubuntu 14.04版本的镜像
    

- `docker images`

> 查看镜像列表

  `$ docker images [OPTIONS] [REPOSITORY]`
    

- `docker rmi`

> 移除镜像（使用中的镜像不能被移除）

  `$ docker rmi [OPTIONS] IMAGE [IMAGE...]  
  `$ docker rmi -f ubuntu:14.04` # 强制移除ubuntu:14.04镜像
    

- `docker run`

> 创建并运行一个新的容器

`$ docker run [OPTIONS] IMAGE [COMMAND] [ARG...] 
  创建一个基于ubuntu:14.04的容器 
$ docker run -it --name hello ubuntu:14.04 /bin/bash 
  `-t 表示返回一个 tty 终端，`
  `-i 表示打开容器的标准输入，使用这个命令可以得到一个容器的 shell 终端` 
  `--name 表示容器的名称`
    

- `docker ps`

> 查看容器列表（默认状态为运行中的容器）

      `# Usage $ docker ps [OPTIONS] # 查看所有容器 $ docker ps -a`
    

- `docker exec`

> 进入容器（运行中）

  `$ docker exec [OPTIONS] CONTAINER COMMAND [ARG...] 
  `$ docker exec -it hello /bin/bash`
    

- `docker rm`

> 移除一个或多个容器（不能移除运行中的容器）

  `$ docker rm [OPTIONS] CONTAINER [CONTAINER...] 
  强制移除容器 
  `$ docker rm -f hello`
    

## 配置nginx服务器

1.获取`nginx`镜像

  `docker pull nginx`
    

2.在`$HOME/www`目录下创建一个`index.html`文件

  `$ mkdir $HOME/www && cd $HOME/www $ echo '欢迎使用docker' > index.html`
    

3.使用`nginx`镜像创建一个`web`容器

`$ sudo docker run --name web -d -v $(pwd):/usr/share/nginx/html -p 80:80 nginx 
   `-d表示让容器在后台运行;-v表示指定当前目录为数据卷，提供nginx文件目录;`
   `-p表示映射主机80端口到容器80端口`

# 构建镜像

> 构建Golang编译环境

## 使用容器创建镜像

- 创建并运行一个容器（基于ubuntu:14.04镜像）

	`$ sudo docker run --name golang -it ubuntu:14.04 /bin/bash`

- 安装Golang编译器

  `$ apt-get update $ apt-get install -y golang 
  清除apt缓存 
  `$ rm -rf /var/lib/apt/lists/*`
    

- 配置Golang编译环境

  `$ mkdir /gopath && cd /gopath && mkdir bin src 
  `$ export GOPATH=/gopath 
  `$ export PATH=$PATH:$GOPATH/bin`
    

- 构建Golang镜像

  `docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]] 
  `$ docker commit -m 'Golang compiler.' golang golang:test`
    

## 使用Dockerfile构建镜像

> 以上述操作为例，介绍一些常用指令，具体了解可参考附录A

- Dockerfile指令

  `# 基础镜像 # Usage: FROM image:tag 
  `FROM ubuntu:14.04  
  `# 指定维护者信息 
  `# Usage: 
  `MAINTAINER name 
  `MAINTAINER tiannianshou@gmail.com  
  `# 在shell中运行命令 
  `# Usage: RUN command 
  `RUN apt-get update && \     
		`apt-get install -y golang && \     
		`rm -rf /var/lib/apt/lists/* && \     
		`mkdir -p /gopath  
	`# 指定环境变量 
	`# Usage: ENV key value 
	`ENV GOPATH /gopath 
	`ENV PATH $GOPATH/bin:$PATH  
	`# 工作目录 # Usage: WORKDIR /path 
	`WORKDIR /gopath  
	`# 创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等 `# `Usage: VOLUMES  
	`VOLUMES ["/gopath"]  
	`# 容器暴漏给外侧的端口号 
	`# Usage: EXPOSE port [port...] 
	`EXPOSE 80  
	`# 指定启动容器时执行的命令 
	`# Usage: CMD ["executable","param1","param2"] 
	`CMD ["/bin/bash"]`
    

- 构建镜像

  `# 使用指定的Dockerfile创建镜像 # Usage: docker build [OPTIONS] PATH | URL | - `# .表示当前目录 
  $ sudo docker build -t golang:1.2 .`

# 数据卷的使用
数据卷是一个位于容器中的特殊目录, 其目的是绕过UFS文件系统提供持久化和数据共享(UFS的详细介绍请参考附录B).

- 数据卷可以在容器之间共享和重用数据
- 对数据卷的修改是直接的
- 更新镜像不会包含对数据卷的数据
- 数据卷一直存在, 直到不再有容器使用

## 添加数据卷
在创建容器时通过指定-v参数创建数据卷，可使用-v多次挂载多个数据卷

`# Usage
`# 其中HOST_PATH表示宿主系统的文件系统路径,CONTAINER_PATH表示容器的文件系统路径
`# 文件系统路径都为绝对路径
`$ docker run -v [HOST_PATH]:[CONTAINER_PATH] IMAGE [COMMAND] [ARG...]

`# 创建一个基于ubuntu:14.04的数据卷容器
$ sudo docker run --name mydata -d -v /data ubuntu:14.04 /bin/bash
    
## 挂载宿主系统目录作为数据卷

`# 创建一个基于golang:1.2的容器，并指定宿主系统/gopath路径为GOPATH路径
`# 宿主系统中如果不存在该目录,Docker会自动创建
`$ sudo docker run --name mygolang -d -v /gopath:/gopath golang:1.2 /bin/bash
  
挂载数据卷容器
挂载数据卷容器主要用于容器间数据共享

`# 在创建容器时通过指定--volumes-from参数挂载数据卷容器
`# 将上面创建的mydata容器中/data数据卷，挂载到另外一个容器中
`$ sudo docker --name db1 -d --volumes-from mydata ubuntu:14.04 /bin/bash
  
## 数据卷的备份与恢复
### 备份

  `# 备份mydata容器中的数据卷/data,到当前目录下的backup.tar文件中
`$ sudo docker run --rm --volumes-from mydata -v $(pwd):/backup ubuntu:14.04 tar cvf /backup/backup.tar /data
    
### 恢复

`# 恢复本地目录下的backup.tar文件到mydata容器下的/data数据卷
`$ sudo docker run --rm --volumes-from mydata -v $(pwd):/backup ubuntu:14.04 tar xvf /backup/backup.tar

# 容器的连接

> 提供容器之间的安全交互，在创建容器时通过指定--link参数即可实现容器之间的交互

- 创建基于`MongoDB`的容器(不指定映射端口)

      `$ sudo docker run --name db2 -d mongo:3.1`
    

- 查看`db2`容器内部的ip地址

      `$ sudo docker exec -i db2 cat /etc/hosts`
    

- 更改`web.tar.gz`中的配置文件，并创建`web:2.0`镜像

      `$ sudo docker build -t web:2.0 .`
    

- 创建基于`web:2.0`的`web2`容器，并连接到`db2`容器(`db2`容器必须处于运行状态)

      `# --like name:alias $ sudo docker run --name web2 --link db2:db2 -p 8080:5800 -d web:2.0`

# 镜像的备份与恢复

- `docker save`

> 导出镜像到本地文件

  `# Usage 
  `$ docker save [OPTIONS] IMAGE [IMAGE...] 
  `# 导出golang镜像 
  `$ sudo docker save --output golang.tar golang:1.2`


- `docker load`

> 从本地文件导入文件到镜像库

`# Usage 
`$ docker load [OPTIONS] 
`# 导入golang镜像 
`$ sudo docker load --input golang.tar`  

- `docker export`

> 导出容器快照到本地文件

  `# Usage 
  `$ docker export [OPTIONS] CONTAINER 
  `# 导出hello容器快照 
  `$ sudo docker export --output hello.tar`
    

- `docker import`

> 从容器快照文件中再导入为镜像

 `# Usage 
 `$ docker import [OPTIONS] URL|- [REPOSITORY[:TAG]] 
 `# 导入hello快照，并制定镜像标签为hello:1.0 
 `$ cat hello.tar | sudo docker import - hello:1.0`


# Dockerfile

- **FROM:** `镜像`
- **MAINTAINER:** `镜像创建者`
- **RUN:** `执行命令`
- **ENV:** `设置环境变量`
- **USER:** `使用哪个用户跑container`
- **EXPOSE:** `container内部服务开启的端口`
- **COPY:** `将文件<src>拷贝到container的文件系统对应的路径<dest>`
- **VOLUME:** `可以将本地文件夹或者其他container的文件夹挂载到container中`
- **WORKDIR:** `切换目录，同cd`
- **ONBUILD:** `指定的命令在构建镜像时并不执行，而是在它的子镜像中执行`
    
- **CMD**
	1. container启动时执行的命令，但是一个Dockerfile中只能有一条CMD命令，多条则只执行最后一条CMD.
	2. CMD主要用于container时启动指定的服务，当docker run command的命令匹配到CMD command时，会替换CMD执行的命令

- **ENTRYPOINT**
	1. container启动时执行的命令，但是一个Dockerfile中只能有一条ENTRYPOINT命令，如果多条，则只执行最后一条
	2. ENTRYPOINT没有CMD的可替换特性

- **ADD**
	1. 将文件拷贝到container的文件系统对应的路径
	2. 所有拷贝到container中的文件和文件夹权限为0755,uid和gid为0
	3. 如果文件是可识别的压缩格式，则docker会帮忙解压缩
	4. 只有在build镜像的时候运行一次，后面运行container的时候不会再重新加载了
	5. 可拷贝url路径的文件