# 精通WSL，零成本实操Linux：从安装到高效运维，解锁Windows下的Linux魅力
* * *

Windows 子系统 WSL 介绍：开启跨平台开发的新纪元
------------------------------

想象一下，在你的 Windows 桌面，无需重启，就能无缝切换至 Linux 环境，运行 Linux 命令行工具、开发软件甚至是服务器应用。这不是科幻电影，而是 Windows Subsystem for Linux (WSL) 带给我们的现实！

**「什么是 WSL？」** WSL，即 Windows Subsystem for Linux，是微软为 Windows 10 及之后版本引入的一项革命性功能。它让你在 Windows 内核上直接运行一个轻量级的 Linux 环境，不是虚拟机，也不是容器，却能享受到原生 Linux 体验。这意味着你可以直接在 Windows 上编译 Linux 程序、管理服务器、甚至部署 Web 应用，让开发者和 IT 运维人员的工作效率直线飙升。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c2d44e41ed34e0a9e78a276c084ef73~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1024&h=1024&s=165277&e=jpg&b=fef6d0)

Ubuntu wsl docker desktop 1panel的图标.jpg

准备工作：启用 WSL 2
-------------

首先，确保您的 Windows 系统支持并已启用 WSL 2。以下是启用步骤：

1.  **「检查 Windows 版本」**：打开“设置” > “系统” \> “关于”，确认您的 Windows 版本为 1903 及以上。
    
2.  **「启用“适用于 Linux 的 Windows 子系统”」**：打开“控制面板” > “程序” > “程序和功能” \> “启用或关闭 Windows 功能”，勾选“适用于 Linux 的 Windows 子系统”。
    
3.  **「更新到 WSL 2」**：打开 PowerShell（以管理员身份运行），输入以下命令安装 WSL 2 所需组件：
    
    ```bash
    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
    
    ```
    
    然后，下载并安装 Linux 内核更新：
    
    ```css
    wsl --update
    
    ```
    

WSL 应用：Docker Desktop 与 Ubuntu
------------------------------

WSL 该如何应用呢，我从个人的实际经验给出两个好用的方式。那就是 Docker Desktop 与 Ubuntu。下面介绍以下两种方式的安装使用方法。我是从 Docker Desktop 开始的，并且用了一年多。 多年的 VPS 使用经验让我有了更开阔的思路，最近探索出更高级的玩法，也就是 WSL 2 + Docker Desktop + Ubuntu。这也是本文核心的部分，因为在互联网上找不到教程，不敢说我是探索这种方式的第一人，估计这么玩的人并不多。

### WSL + Docker Desktop

#### 安装 Docker Desktop

1.  **「下载 Docker Desktop」**：访问 Docker 官方网站的下载页面\[1\]，点击“Get Docker”按钮下载适用于 Windows 的 Docker Desktop 安装程序。
    
2.  **「安装 Docker Desktop」**：找到下载的安装文件（通常是`Docker Desktop Installer.exe`），双击运行开始安装。按照安装向导的指示进行，一般情况下保持默认设置即可。
    
3.  **「配置 Docker 以使用 WSL 2」**：安装完成后首次启动 Docker Desktop 时，它会提示您选择使用 WSL 2 作为引擎。如果您错过了这个步骤，可以在 Docker Desktop 的设置中调整：
    
    *   打开 Docker Desktop。
    *   点击右上角的齿轮图标进入设置（Preferences）。
    *   转到“Resources”（资源）> “WSL Integration”（WSL 集成）。
    *   在列表中勾选您已经安装的 Ubuntu 或其他 WSL 2 发行版。

#### 测试 Docker 是否正常工作

1.  打开一个新的 Windows Terminal 或 PowerShell 窗口，确保它是在您刚刚配置的 WSL 2 发行版（例如 Ubuntu）环境下。
2.  输入以下命令测试 Docker 是否正确安装并运行：

```arduino
docker run hello-world

```

如果一切设置正确，您将看到 Docker 的欢迎消息，说明 Docker 已经在您的 WSL 2 环境中成功运行。

至此，已成功安装并配置了 Docker Desktop 配合 WSL 2 在 Windows 上运行，可以开始享受 Linux 环境下的 Docker 便捷开发体验了。无论是运行容器、构建镜像，还是更复杂的开发和部署任务，Docker Desktop 结合 WSL 2 都将为您的开发工作带来极大的便利，后面的使用和直接在 Linux 上使用容器类似，只是增加了直观的界面，后面就不在详细介绍 Docker Desktop 的使用，而是重点介绍 Ubuntu 上安装运维面板 1panel，并实现任意地点访问内网机器上的服务。

### WSL + Ubuntu on WSL

WSL 安装 Ubuntu 有两种方式，一种是通过应用商店，一种是命令行。

#### 应用商店安装 Ubuntu

安装 Ubuntu 是最受欢迎的 WSL 发行版之一的选择。打开 Microsoft Store，搜索“Ubuntu”，选择最新版本点击安装即可。首次启动时，系统会提示你创建一个新的 Linux 用户账户和密码。随后，一个全新的 Ubuntu 终端就展现在眼前，你可以开始执行熟悉的`apt-get`命令来安装软件包了。Ubuntu on WSL 为开发者提供了完整的 Linux Shell 环境，无论是 Git、Python 还是 Node.js，一切手到擒来。

#### 命令行安装 Ubuntu

在 cmd 终端运行如下命令：

```css
wsl --install -d Ubuntu

```

注意：**「我在安装的时候用的是命令行的方式，不过开始连接的时候有问题，所以先开了全局代理才成功安装。」** **「另外在安装过程中会提示设置用户名和密码，这个要记好，以后执行命令可能需要输入这次设置的密码。」**

在 Ubuntu WSL 中安装 1Panel 运维面板：一键管理，轻松运维
--------------------------------------

### **「什么是 1Panel？」**

1Panel 是一个轻量级的运维面板，旨在简化服务器管理，特别是对于中小规模的网站和应用部署。它提供了一站式的解决方案，包括环境部署、应用管理、数据库管理、SSL 证书申请等，让运维工作变得直观且高效。

### **「安装 1Panel：」**

首先运行

```null
wsl -d ubuntu

```

启动 Ubuntu。

接着执行下面命令安装 1panel。这一步骤需要用户密码，输入上面安装 Ubuntu 时设置的密码即可。

```bash
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sudo bash quick_start.sh

```

接着就开始自动下载

```yaml
开始下载 1Panel v1.10.9-lts 版本在线安装包
安装包下载地址： https://resource.fit2cloud.com/1panel/package/stable/v1.10.9-lts/release/1panel-v1.10.9-lts-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 49.6M  100 49.6M    0     0  6329k      0  0:00:08  0:00:08 --:--:-- 7578k
Warning: Got more output options than URLs
1panel-v1.10.9-lts-linux-amd64/1panel.service
1panel-v1.10.9-lts-linux-amd64/1pctl
1panel-v1.10.9-lts-linux-amd64/LICENSE
1panel-v1.10.9-lts-linux-amd64/README.md
1panel-v1.10.9-lts-linux-amd64/install.sh
1panel-v1.10.9-lts-linux-amd64/1panel

 ██╗    ██████╗  █████╗ ███╗   ██╗███████╗██╗
███║    ██╔══██╗██╔══██╗████╗  ██║██╔════╝██║
╚██║    ██████╔╝███████║██╔██╗ ██║█████╗  ██║
 ██║    ██╔═══╝ ██╔══██║██║╚██╗██║██╔══╝  ██║
 ██║    ██║     ██║  ██║██║ ╚████║███████╗███████╗
 ╚═╝    ╚═╝     ╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚══════╝
[1Panel Log]: ======================= 开始安装 =======================
设置 1Panel 安装目录（默认为/opt）：
[1Panel Log]: 您选择的安装路径为 /opt
[1Panel Log]: ... 在线安装 docker
选择延迟最低的源 https://mirrors.cernet.edu.cn/docker-ce，延迟为 0.148943 秒


WSL DETECTED: We recommend using Docker Desktop for Windows.
Please get Docker Desktop from https://www.docker.com/products/docker-desktop/


You may press Ctrl+C now to abort this script.
+ sleep 20
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
+ sh -c install -m 0755 -d /etc/apt/keyrings
+ sh -c curl -fsSL "https://mirrors.cernet.edu.cn/docker-ce/linux/ubuntu/gpg" -o /etc/apt/keyrings/docker.asc
+ sh -c chmod a+r /etc/apt/keyrings/docker.asc
+ sh -c echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.cernet.edu.cn/docker-ce/linux/ubuntu jammy stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-ce-rootless-extras docker-buildx-plugin >/dev/null
+ sh -c docker version
Client: Docker Engine - Community
 Version:           26.1.3
 API version:       1.45
 Go version:        go1.21.10
 Git commit:        b72abbb
 Built:             Thu May 16 08:33:29 2024
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          26.1.3
  API version:      1.45 (minimum version 1.24)
  Go version:       go1.21.10
  Git commit:       8e96db1
  Built:            Thu May 16 08:33:29 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.31
  GitCommit:        e377cd56a71523140ca6ae87e30244719194a521
 runc:
  Version:          1.1.12
  GitCommit:        v1.1.12-0-g51d5e94
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

```

接下来自动安装，安装过程需要设置端口，登录路径，用户名和密码等，都可以保持默认，暗转完成会显示出这些信息，保存下来即可。 内容如下（已经把敏感信息用 xxxxxxxxx 代替）：

```less
================================================================================

[1Panel Log]: ... 启动 docker
Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable docker
[1Panel Log]: docker 安装成功
[1Panel Log]: ... 在线安装 docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 59.8M  100 59.8M    0     0  12.8M      0  0:00:04  0:00:04 --:--:-- 13.6M
[1Panel Log]: docker-compose 安装成功
设置 1Panel 端口（默认为xxxxx）：
[1Panel Log]: 您设置的端口为：xxxxx
[1Panel Log]: 防火墙开放 xxxxx 端口
Rules updated
Rules updated (v6)
Firewall not enabled (skipping reload)
设置 1Panel 安全入口（默认为xxxxxxxxx）：
[1Panel Log]: 您设置的面板安全入口为：xxxxxxxxx
设置 1Panel 面板用户（默认为dc00cedbb4）：xxxxxxxxx
[1Panel Log]: 您设置的面板用户为：xxxxxxxxx
设置 1Panel 面板密码（默认为xxxxxxxxx）：
[1Panel Log]: 配置 1Panel Service
Created symlink /etc/systemd/system/multi-user.target.wants/1panel.service → /etc/systemd/system/1panel.service.
[1Panel Log]: 启动 1Panel 服务
[1Panel Log]: 1Panel 服务启动成功!
[1Panel Log]:
[1Panel Log]: =================感谢您的耐心等待，安装已经完成==================
[1Panel Log]:
[1Panel Log]: 请用浏览器访问面板:
[1Panel Log]: 外网地址: http:
[1Panel Log]: 内网地址: http:
[1Panel Log]: 面板用户: xxxxxxxxx
[1Panel Log]: 面板密码: xxxxxxxxx
[1Panel Log]:
[1Panel Log]: 项目官网: https:
[1Panel Log]: 项目文档: https:
[1Panel Log]: 代码仓库: https:
[1Panel Log]:
[1Panel Log]: 如果使用的是云服务器，请至安全组开放 xxxxx 端口
[1Panel Log]:
[1Panel Log]: 为了您的服务器安全，在您离开此界面后您将无法再看到您的密码，请务必牢记您的密码。
[1Panel Log]:
[1Panel Log]: ================================================================

```

### 访问 1panel

现在访问显示的内网地址[http://172.29.206.19:xxxxx/xxxxxxxxx就可以看到登录界面了，然后用显示的账户登录即可。](https://link.juejin.cn/?target=http%3A%2F%2F172.29.206.19%3Axxxxx%2Fxxxxxxxxx%25E5%25B0%25B1%25E5%258F%25AF%25E4%25BB%25A5%25E7%259C%258B%25E5%2588%25B0%25E7%2599%25BB%25E5%25BD%2595%25E7%2595%258C%25E9%259D%25A2%25E4%25BA%2586%25EF%25BC%258C%25E7%2584%25B6%25E5%2590%258E%25E7%2594%25A8%25E6%2598%25BE%25E7%25A4%25BA%25E7%259A%2584%25E8%25B4%25A6%25E6%2588%25B7%25E7%2599%25BB%25E5%25BD%2595%25E5%258D%25B3%25E5%258F%25AF%25E3%2580%2582 "http://172.29.206.19:xxxxx/xxxxxxxxx%E5%B0%B1%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B0%E7%99%BB%E5%BD%95%E7%95%8C%E9%9D%A2%E4%BA%86%EF%BC%8C%E7%84%B6%E5%90%8E%E7%94%A8%E6%98%BE%E7%A4%BA%E7%9A%84%E8%B4%A6%E6%88%B7%E7%99%BB%E5%BD%95%E5%8D%B3%E5%8F%AF%E3%80%82")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eda414bddd6e4dc48a4a3730de9c4b94~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=936&h=608&s=335840&e=png&b=d5e2f8)

image.png

到现在就可以愉快的使用 1panel 管理自己的中多应用了，由于本地电脑配置高，延迟低，体验比 VPS 还好。唯一的缺点是不能公网访问。安装结束后显示的公网访问地址正常是无法实现公网访问的，因为我们处于 net 之中，没有独立公网 ip。

那么该怎么做才能实现公网访问呢？由于需要有公网 ip 的 VPS 才能实现，大多数人不具备这个条件，我只是简单给出我的实现方法和配置，不再具体展开。

### 远程访问本地 1panel（可选）

#### 在本地安装 frpc（内网穿透工具客户端）

在刚才登录的 1panel 应用商店中搜索 frp 然后安装客户端。 修改"\\wsl.localhost\\Ubuntu\\opt\\1panel\\apps\\frpc\\frpc\\data\\frpc.toml"配置文件内容如下：

```ini
serverAddr = "VPS的ip地址"
serverPort = 7000

auth.method = "token"
auth.token = "随便填一串数字但要和服务端保持一致"

[[proxies]]
name = "wsl 1panel"
type = "http"
localPort = 这里填写本地登录ipanel的端口
customDomains = ["这里写你想用于远程访问的域名，例如a.b.com"]

```

然后重启客户端服务。

#### 在本地安装 frps（内网穿透工具服务端）

在具有公网 IP 的 VPS 的 1panel 应用商店中搜索 frp 然后安装服务端。

```ini
bindAddr = "0.0.0.0"
bindPort = 7000

vhostHTTPPort = 自己设置一个端口，记得防火墙放行

auth.method = "token"
auth.token = "随便填一串数字但要和客户端保持一致"

```

然后重启服务端服务。

然后新建一个反向代理网站，方向代理地址填 127.0.0.1:端口（这个端口是 vhostHTTPPort 处填写的端口）可以根据你的实际情况决定是否开启 https 访问。 如果不设置方向代理网站也可以，但在访问的时候需要在域名后面增加 vhostHTTPPort 处填写的端口才能正常访问。

#### 域名解析

将客户端设置的域名解析到服务器 IP。

如果上面三个步骤都没问题的话，现在访问设置的域名+安全入口就能打开最开始访问的 1panel 登录界面了。只不过现在的域名访问可以随处访问，而刚才智能在本地访问。 下面是我在手机上通过公网的访问效果。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/052e4b833dbc4dc5875ed8360fcd2912~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=1545&s=379688&e=jpg&b=d4e3f9)

探索出这种玩法很激动，然而，当我关闭了 cmd 窗口后发现就不能访问了，应该是 docker 进程随着终端的关闭也停止了。还好我找到了解决办法。

WSL Ubuntu 可以保证在后台运行
--------------------

我搜索到的方法是下面这种

### ~命令窗口运行(win+r)~

```r
powershell.exe -WindowStyle Hidden -c ubuntu

```

我实测上面的方式不行，会跳出 powershell 窗口。

接着想到我以前写过隐藏窗口的 bat 启动脚本，修改一下就可以了。

### 把下面内容保存 bat 执行

```less
@echo off
if "%1"=="h" goto begin
start mshta vbscript:createobject("wscript.shell").run("""%~nx0"" h",0)(window.close)&&exit
:begin

wsl -d ubuntu

```

隐藏启动后想停止该怎么办呢？方法如下。

WSL 怎么停止 ubuntu
---------------

### 使用命令行关闭

使用下面命令关闭 Ubuntu

```c
wsl --terminate Ubuntu

```

使用下面命令关闭 wsl 所有。

```arduino
wsl --shutdown

```

### 使用任务管理器关闭进程

1.  打开任务管理器（可通过右击任务栏，选择“任务管理器”，或快捷键 Ctrl+Shift+Esc）。
2.  切换到“详细信息”标签。
3.  找到与 WSL 相关的进程，通常名为 wsl.exe 或带有 Linux 发行版名称的进程。
4.  右击选择“结束任务”来终止进程。  
    终止止 WSL 的发行版不会影响其持久化数据，下次启动时仍能恢复到上次的状态。

到这里，我的全部探索过程就写完了，由于时间和篇幅的关系，有很多地方还是不够详细，如果你对此感兴趣或者有什么问题，可以找我交流，你遇到的问题我很有可能遇到过。

后续可能会根据需要再完善文章内容，可以收藏然后点击阅读原文查看最新修改的文章。

### 参考资料

\[1\]

下载页面: _[www.docker.com/products/do…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.docker.com%2Fproducts%2Fdocker-desktop "https://www.docker.com/products/docker-desktop")_