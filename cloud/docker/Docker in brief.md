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
# 常用的docker命令

## docker info
## docker search
## docker pull

> 获取镜像

  `$ docker pull [OPTIONS] Name[:TAG] ` 
  `$ docker pull ubuntu:14.04` # 获取ubuntu 14.04版本的镜像
    

## docker images

> 查看镜像列表

  `$ docker images [OPTIONS] [REPOSITORY]`
    

## docker rmi

> 移除镜像（使用中的镜像不能被移除）

  `$ docker rmi [OPTIONS] IMAGE [IMAGE...]  
  `$ docker rmi -f ubuntu:14.04` # 强制移除ubuntu:14.04镜像
    

## docker run

> 创建并运行一个新的容器, 用docker run命令从镜像启动一个容器时，如果该镜像不在本地，Docker会先从Docker Hub下载该镜像。如果没有指定具体的镜像标签，那么Docker会自动下载latest标签的镜像。

`$ docker run [OPTIONS] IMAGE [COMMAND] [ARG...] 

  创建一个基于ubuntu:14.04的容器, 打开终端并运行bash
$ docker run -it --name hello ubuntu:14.04 /bin/bash 
  `-t 表示返回一个 tty 终端，`
  `-i 表示打开容器的标准输入，使用这个命令可以得到一个容器的 shell 终端` 
  `--name 表示容器的名称`
  -d 选项，告诉Docker以分离（detached）的方式在后台运行
  -p [host].[port]:container_port
	我们也可以将端口绑定限制在特定的网络接口（即IP地址）上:
	```
	docker run -d -p 127.0.0.1:80:80 --name static_web jamtur01/static_web nginx -g "daemon off;"
	```
	这里，我们将容器内的80端口绑定到了本地宿主机的127.0.0.1这个IP的80端口上。我们也可以使用类似的方式将容器内的80端口绑定到	一个宿主机的随机端口上
	```
	docker run -d -p 127.0.0.1::80 --name static_web jamtur01/static_web nginx -g "daemon off;"
	```
	我们可以使用docker inspect或者docker port命令来查看容器内的80端口具体被绑定到了宿主机的哪个端口上
	Docker还提供了一个更简单的方式，即-P参数，该参数可以用来对外公开在Dockerfile中通过EXPOSE指令公开的所有端口:
	```
	docker run -d -P --name static_web jamtur01/static_web nginx -g "daemon off;"
	```
	我们也可以组合使用ENTRYPOINT和CMD指令来完成一些巧妙的工作。
	```
	ENTRYPOINT ["/usr/sbin/nginx"]
	CMD ["-h"]
	```
	此时当我们启动一个容器时，任何在命令行中指定的参数都会被传递给Nginx守护进程。比如，我们可以指定-g "daemon off";参数让Nginx守护进程以前台方式运行。如果在启动容器时不指定任何参数，则在CMD指令中指定的-h参数会被传递给Nginx守护进程，即Nginx服务器会以/usr/sbin/nginx -h的方式启动，该命令用来显示Nginx的帮助信息。

## docker start
## docker kill/stop

## docker ps

> 查看容器列表（默认状态为运行中的容器）

      `# Usage $ docker ps [OPTIONS] # 查看所有容器 $ docker ps -a`
    

## docker exec

> 进入容器（运行中）

  `$ docker exec [OPTIONS] CONTAINER COMMAND [ARG...] 
  `$ docker exec -it hello /bin/bash`
    

## docker rm

> 移除一个或多个容器（不能移除运行中的容器）

  `$ docker rm [OPTIONS] CONTAINER [CONTAINER...] 
  强制移除容器 
  `$ docker rm -f hello`
    
## docker inspect 
	查看新创建的镜像的详细信息


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
  $ sudo docker build --tag golang:1.2 .`

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

## **FROM:** `镜像`
	命令格式 ：
	`FROM < image＞或 FROM <image>: <tag＞，
	`指定使用的基础镜像，如 FROM java : 8 代表基于 JDK8 开始构建自己的镜像 。`
	
## **MAINTAINER:** `镜像创建者`
## **RUN:** `执行命令`
	`命令格式 ：
		RUN <command ＞或 RUN［” executable”，”param1”， ’param2＂），在当前镜像基础上执行指定命令，并提交为新的镜像。`

## **ENV:** `设置环境变量`
	命令格式 ：ENV <key> <value＞，该命令用于指定一个环境变量，会被前面的 RUN 命令所使用
## **USER:** `使用哪个用户跑container`
## **EXPOSE:** `container内部服务开启的端口`
	命令格式 ：
		EXPOSE <port> [<port> ... ］，该命令告诉 Docker 服务端容器暴露的端口号供
		系统交互时使用，如 EXPOSE 8080 。
		
##  **COPY:** 
	将文件<src>拷贝到container的文件系统对应的路径<dest>
	COPY指令非常类似于ADD，它们根本的不同是COPY只关心在构建上下文中复制本地文件，而不会去做文件提取（extraction）和解压（decompression）的工作。
	文件源路径必须是一个与当前构建环境相对的文件或者目录，本地文件都放到和Dockerfile同一个目录下。不能复制该目录之外的任何文件，因为构建环境将会上传到Docker守护进程，而复制是在Docker守护进程中进行的。任何位于构建环境之外的东西都是不可用的。COPY指令的目的位置则必须是容器内部的一个绝对路径。任何由该指令创建的文件或者目录的UID和GID都会设置为0。
	如果目的位置不存在，Docker将会自动创建所有需要的目录结构，就像mkdir -p命令那样
## **VOLUME:** 
	可以将本地文件夹或者其他container的文件夹挂载到container中
	
	VOLUME指令用来向基于镜像创建的容器添加卷。一个卷是可以存在于一个或者多个容器内的特定的目录，这个目录可以绕过联合文件系统，并提供如下共享数据或者对数据进行持久化的功能。卷可以在容器间共享和重用。一个容器可以不是必须和其他容器共享卷。对卷的修改是立时生效的。对卷的修改不会对更新镜像产生影响。卷会一直存在直到没有任何容器再使用它。
	卷功能让我们可以将数据（如源代码）、数据库或者其他内容添加到镜像中而不是将这些内容提交到镜像中，并且允许我们在多个容器间共享这些内容。我们可以利用此功能来测试容器和内部的应用程序代
码，管理日志，或者处理容器内部的数据库。
可以通过指定数组的方式指定多个卷:
```
VOLUME ["/opt/project", "/data" ]
```
## **WORKDIR:** `切换目录，同cd`
## **ONBUILD:** 
ONBUILD指令能为镜像添加触发器（trigger）。当一个镜像被用做其他镜像的基础镜像时（比如用户的镜像需要从某未准备好的位置添加源代码，或者用户需要执行特定于构建镜像的环境的构建脚本），该镜像中的触发器将会被执行。触发器会在构建过程中插入新指令，我们可以认为这些指令是紧跟在FROM之后指定的。触发器可以是任何构建指令
```
ONBUILD ADD . /app/src
ONBUILD RUN cd /app/src && make
```
## **CMD**
1. container启动时执行的命令，但是一个Dockerfile中只能有一条CMD命令，多条则只执行最后一条CMD.
2. CMD主要用于container时启动指定的服务，当docker run command的命令匹配到CMD command时，会替换CMD执行的命令

	CMD指令用于指定一个容器启动时要运行的命令。这有点儿类似于RUN指令，只是RUN指令是指定镜像被构建时要运行的命令，而CMD是指定容器被启动时要运行的命令。这和使用docker run命令启动容器时指定要运行的命令非常类似:
	```
	docker run -i -t jamtur01/static_web /bin/true
	vs.
	CMD ["/bin/true"]
	```
3. 使用docker run命令可以覆盖CMD指令。如果我们在Dockerfile里指定了CMD指令，而同时在docker run命令行中也指定了要运行的命令，命令行中指定的命令会覆盖Dockerfile 中的CMD指令。

## **ENTRYPOINT**
	`命令格式 ：
		`ENTRYPOINT ["exect」 tabl e ", "paraml ”，＂ param2＂），代表配置容器启动后执行的命令`. e.g. ENTRYPOINT [”java”, ”- jar”, ”eureka . jar” ]
	1. container启动时执行的命令，但是一个Dockerfile中只能有一条ENTRYPOINT命令，如果多条，则只执行最后一条
	2. ENTRYPOINT指令与CMD指令非常类似。这两个指令的什么区别：我们可以在docker run命令行中覆盖CMD指令。有时候，我们希望容器会按照我们想象的那样去工作，这时候CMD就不太合适了。而ENTRYPOINT指令提供的命令则不容易在启动容器时被覆盖。实际上，docker run命令行中指定的任何参数都会被当做参数再次传递给ENTRYPOINT指令中指定的命令。

## **ADD**
	命令格式 ：ADD <src> <dest＞，代表复制指定的＜src＞到容器中的＜dest＞
	1. 将文件拷贝到container的文件系统对应的路径
	2. 所有拷贝到container中的文件和文件夹权限为0755,uid和gid为0
	3. 如果文件是可识别的压缩格式，则docker会帮忙解压缩
	4. 只有在build镜像的时候运行一次，后面运行container的时候不会再重新加载了
	5. 可拷贝url路径的文件
	
	在ADD文件时，Docker通过目的地址参数末尾的字符来判断文件源是目录还是文件。如果目标地址以/结尾，那么Docker就认为源位置指向的是一个目录。如果目的地址以/结尾，那么Docker就认为源位置指向的是目录。如果目的地址不是以/结尾，那么Docker就认为源位置	指向的是文件。

## Label
LABEL指令用于为Docker镜像添加元数据.
```
LABEL version="1.0"
LABEL location="New York" type="Data Center" role="Web Server"
```
LABEL指令以label="value"的形式出现。可以在每一条指令中指定一个元数据，或者指定多个元数据，不同的元数据之间用空格分隔。推荐将所有的元数据都放到一条LABEL指令中，以防止不同的元数据指令创建过多镜像层。可以通过docker inspect命令来查看Docker镜像中的标签信息.

## STOPSIGNAL
STOPSIGNAL指令用来设置停止容器时发送什么系统调用信号给容器。这个信号必须是内核系统调用表中合法的数，如9，或者注意警告SIGNAME格式中的信号名称，如SIGKILL。

## ARG
ARG指令用来定义可以在docker build命令运行时传递给构建运行时的变量，我们只需要在构建时使用--build-arg标志即可。用户只能在构建时指定在Dockerfile文件中定义过的参数。
```
ARG build
ARG webapp_user=user
```
如果ARG指令设置了一个默认值，如果构建时没有为该参数指定值，就会使用这个默认值。
```
docker build --build-arg build=1234 -t jamtur01/webapp .
```


# 运行自己的Docker Registry

## 从容器运行Registry

```
docker run -p 5000:5000 registry:2
```
该命令将会启动一个运行Registry应用2.0版本的容器，并将5000端口绑定到本地宿主机。

## 测试新Registry
```
docker images jamtur01/static_web
docker tag 22d47c8cb6e5
docker push localhost:5000/jamtur01/static_web
```

# configure `pom.xml` to build a Docker image

you can use the **Jib** plugin or the **Spotify Docker Maven plugin**. Here, I'll demonstrate both methods:

### 1. **Using Jib Plugin**
The Jib plugin by Google is a popular option for building Docker images directly from a Maven project without needing a Dockerfile.

#### **Step 1: Add Jib Plugin to `pom.xml`**

```xml
<project>
  <!-- other configurations -->
  <build>
    <plugins>
      <plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>3.3.2</version>
        <configuration>
          <from>
            <image>openjdk:17-jdk-slim</image>
          </from>
          <to>
            <image>myapp:latest</image>
          </to>
          <container>
            <mainClass>com.example.Main</mainClass> <!-- Replace with your main class -->
            <ports>
              <port>8080</port>
            </ports>
          </container>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

#### **Step 2: Build the Docker Image**

Run the following Maven command to build the Docker image:

```bash
mvn compile jib:dockerBuild
```

This will build your application and create a Docker image named `myapp:latest`.

### 2. **Using Spotify Docker Maven Plugin**
The Spotify Docker Maven plugin is another method, although it's less actively maintained compared to Jib.

#### **Step 1: Add Spotify Docker Plugin to `pom.xml`**

```xml
<project>
  <!-- other configurations -->
  <build>
    <plugins>
      <plugin>
        <groupId>com.spotify</groupId>
        <artifactId>dockerfile-maven-plugin</artifactId>
        <version>1.4.13</version>
        <executions>
          <execution>
            <id>default</id>
            <goals>
              <goal>build</goal>
            </goals>
            <phase>package</phase>
          </execution>
        </executions>
        <configuration>
          <repository>myapp</repository>
          <tag>latest</tag>
          <dockerfile>Dockerfile</dockerfile>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

#### **Step 2: Create a `Dockerfile`**
You will need a `Dockerfile` as described in the previous response.

#### **Step 3: Build the Docker Image**

Run the following Maven command:

```bash
mvn package
```

This command will build your Maven project and the Docker image as part of the package phase.

### Summary
- **Jib Plugin**: No need for a Dockerfile, integrates easily with Maven.
- **Spotify Docker Maven Plugin**: Requires a Dockerfile but offers more control over the Docker build process.


# 使用 Docker Compose 编排服务
在 Docker 中运行微服务的方式主要有两种 ， 一种是前面介绍的使用 Dockerfile 构建镜像
并启动容器。Dockerfile 方式面向的是单个微服务和单个容器 ， 但微服务架构中往往包含多个微服务。服务数量的增加也就意味着容器数量的增多，逐渐增多的容器数量为容器部署、运行及管理带来了挑战。接下来我们介绍的基于 Docker Compose 编排微服务的方式就是针对这种场景。通过 Docker Compose 可以解决本章开篇提到的另一个核心问题 ， 即“如何组装不同的服务容器构成一个服务体系” 。
Docker Compose 面向多个微服务和多个容器，专门用于解决多个容器的部署问题并提高
部署方案的可移植性。Docker Compose 是用来定义和运行复杂应用的Docker 工具 ， 使用
Docker Compose ，就可以允许用户在一个模板（ YAM L 格式）中定义多容器应用，然后通过
一条命令来启动整个服务体系 。

## Docker Compose 命令
Docker Compose 的常见命令罗列如下，通过英文命名基本都可以明白具体的含义和执行
效果。
- up  创建并启动容器。
- kill 杀掉容器。
- ps 显示容器。
- rm 删除停止的容器。
- scale 设置服务的容器数量。
- pull 拉取服务镜像。
- restart 重启容器。
- build 构建或重建容器。
- start 启动容器。
- stop 停止容器。
- down 停止服务并移除容器。
- logs 查看服务的日志事件


## docker-compose.yml 命令
想要通过 Docker Compose 编排服务需要提供 docker-compose.yml配置文件 ， 该配置文件中同样包含了一系列命令用于完成服务的编排 ， 常见的命令如下所示 。
- image 指定镜像名称或镜像id 。 如果镜像在本地不存在,  Docker Compose 将会尝试拉取这个镜像。
- build 指定 Dockerfile 所在文件夹的路径 。Docker Compose 将会利用它自动构建并使用这个镜像。
- command 覆盖容器启动后默认执行的命令。
- links 链接到其他服务中的容器。
- ports 暴露端口信息。
	使用的是“主机端口 : 容器端口 ”的格式，也可以只指定容器端口: ": 容器端口"（主机端口会随机分自己）。 默认情况下，Docker Compose 会为应用创建一个私有网络，服务的每个容器都会加入到该网络中，每个容器也就可以被该网络中的其他容器所访问 。 同时，每个容器还能够以服务名称作为主机名被其他容器访问 .
- environment 设置环境变量 。
- depends_on 用于指定服务依赖，一旦指定了服务依赖，将会优先于自身服务创建并启动依赖。
```
version :’ 2 ’
services :
	eureka :
		build : ./eureka
		ports :
			- "8761 : 8761"
	config:
		build : ./config
		depends on :
			- eureka
		ports :
			- "8888 : 8888"
``` 
 
在上面的例子中， config服务依赖 eureka 服务，通过 depends_on 命令确保在服务启动顺序上会先启动 eureka 服务，然后再启动 config 服务.


# Docker vs. Docker Compose 
Both tools related to containerization, but they serve different purposes and are used in different contexts. Here's a breakdown of their differences:

### 1. **Purpose and Use Case**
   - **Docker**: 
     - **Purpose**: Docker is primarily used to create, deploy, and manage individual containers. A container is a lightweight, standalone, and executable package that includes everything needed to run an application, including the code, runtime, libraries, and system tools.
     - **Use Case**: Docker is ideal for running a single service or application in isolation. It’s commonly used for development, testing, and deployment of microservices or small applications.

   - **Docker Compose**: 
     - **Purpose**: Docker Compose is a tool for defining and running multi-container Docker applications. It allows you to define multiple services in a single YAML file (`docker-compose.yml`) and manage their lifecycle together.
     - **Use Case**: Docker Compose is best used when you have a complex application that consists of multiple services, such as a web server, database, cache, and message queue, all of which need to be orchestrated together.

### 2. **Configuration and Execution**
   - **Docker**:
     - **Configuration**: Docker typically uses individual `Dockerfile` configurations for building images. You run containers using the `docker run` command, where you can pass various options such as port mapping, environment variables, volumes, and more.
     - **Execution**: You interact with Docker primarily via the command line (CLI) to build, run, stop, and manage containers individually.

   - **Docker Compose**:
     - **Configuration**: Docker Compose uses a `docker-compose.yml` file to define and configure multiple services, networks, and volumes. This file specifies all the services, their images, volumes, networks, environment variables, and other dependencies.
     - **Execution**: You use the `docker-compose` command to manage the entire application stack. For example, you can start all services with `docker-compose up`, and stop them with `docker-compose down`.

### 3. **Complexity and Scalability**
   - **Docker**:
     - **Complexity**: Docker is simpler when dealing with individual containers or small applications.
     - **Scalability**: Managing complex multi-container applications can become cumbersome with Docker alone, as you need to manually start, stop, and configure each container.

   - **Docker Compose**:
     - **Complexity**: Docker Compose simplifies the management of multi-container applications by handling inter-service dependencies, networks, and volumes in a single configuration file.
     - **Scalability**: Docker Compose is well-suited for development and small-scale deployments, but for more complex, large-scale, production-grade deployments, you might consider container orchestration tools like Kubernetes.

### 4. **Networking**
   - **Docker**: Containers by default run on a single network, and networking between multiple containers needs to be configured manually.
   - **Docker Compose**: Automatically creates a default network for all services defined in the `docker-compose.yml` file, allowing easy communication between services using their service names.

### 5. **Multi-Container Management**
   - **Docker**: Manages individual containers; you would need to run multiple commands to manage multiple containers.
   - **Docker Compose**: Manages multiple containers together, handling dependencies, start order, and shared resources.

### **Summary**
- **Docker** is great for managing individual containers and is widely used for building, shipping, and running applications.
- **Docker Compose** simplifies the management of applications that consist of multiple interconnected containers, making it easier to define, run, and manage multi-container Docker applications.