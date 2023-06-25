# 实战开发arthas-spring-boot-starter监控你的微服务是否健康_Java_
前提介绍
----

相信如果经历了我的上一篇 Arthas 的文章\[【JVM 实战系列】「监控调优体系」针对于 Alibaba-Arthas 的安装入门及基础使用开发实战指南\]之后，相信你对 Arthas 的功能和使用应该有了一定的理解了。那么我们就要进行下一步的探索功能。

Arthas 对于 SpringBoot2 的支持和监控体系
------------------------------

在 SpringBoot2 应用中加入 arthas-spring-boot-starter 后，Spring 会启动 arthas 服务，并且进行 attach 自身进程，并配合 tunnel server 实现远程管理。这样的方案非常适合在微服务容器环境中进行远程诊断，在容器网络环境中仅需要对外暴露 tunnel server 的端口。

### Arthas 的监控体系所需要的组件支持

*   Arthas Tunnel Server/Client（Java agent 探针的管理和监控，方便我们管理服务和探针）
    
*   Web Console
    

### 什么是 Arthas Tunnel

在容器化部署的环境内部，Java 进程可以是在不同的机器启动的，想要使用 Arthas 去诊断会比较麻烦，因为用户通常没有机器的权限，即使登陆机器也分不清是哪个 Java 进程。在这种情况下，可以使用 Arthas Tunnel Server/Client。

#### Arthas Tunnel 的作用和目的

整个 Arthas 的功能体系中，可以通过 Arthas Tunnel Server/Client 来远程管理/连接多个 Agent（也就代表着可以监控多个 JVM 进程）。主要目的用于监控和获取目标的 JVM 的进程数据信息。

#### 下载部署 Arthas tunnel server

##### Github 源码仓库下载

[下载地址Arthas tunnel server](https://xie.infoq.cn/link?target=https%3A%2F%2Fgithub.com%2Falibaba%2Farthas%2Freleases)，目前最新版本为 arthas-all-3.6.7 版本，如下图所示。

![](_assets/60d029e7e1b0e69eda2db0012fd32b3f.png)

针对于 Arthas 的安装包进行下载资料进行介绍：

*   arthas-3.6.7.deb：主要用于 debian 操作系统去运行的安装包
    
*   arthas-bin.zip：二进制可运行执行包
    
*   arthas-doc.zip：针对于 arthas 的文档
    
*   **arthas-tunnel-server-3.6.7-fatjar.jar**：Arthas tunnel server 服务的 Jar 可以执行包
    
*   Source code(zip)：源码 zip 压缩包
    
*   Source code(tar.gz)：源码 tar 包
    

##### Maven 仓库下载

阿里云的下载地址：[https://arthas.aliyun.com/download/arthas-tunnel-server/latest_version?mirror=aliyun](https://xie.infoq.cn/link?target=https%3A%2F%2Farthas.aliyun.com%2Fdownload%2Farthas-tunnel-server%2Flatest_version%3Fmirror%3Daliyun)  

#### 直接运行对应的 Arthas tunnel server

Arthas tunnel server 是一个 Spring boot fat jar 应用，直接 java -jar 启动：

```
java -jar  arthas-tunnel-server.jar
```

默认情况下，arthas tunnel server 的 web 端口是 8080，Arthas agent 连接的端口是 7777

![](_assets/34b4fd9a1444a345c3d9c1e6f4b77bd8.png)

打开 WebConsole，分别输入 Arthas agent 的 ip(127.0.0.1)和 port(7777)，和 SpringBoot 应用里配置的 agent-id(URJZ5L48RPBR2ALI5K4V)，点 Connect 即可。

##### Web Console

如果希望可以通过浏览器连接 Arthas 服务，此时这里的 Arthas 服务指的不是 Arthas tunnel server，Arthas 是总体的服务控制端，发送指令的部分，而 Arthas tunnel server 属于对接和管理 agent 的专门服务(依赖于 Arthas Spring Boot Starter 的服务)。

出了 CLI 模式之外，Arthas 目前支持 Web Console，用户在 attach 成功之后，可以直接访问：[http://127.0.0.1:8563/](https://xie.infoq.cn/link?target=http%3A%2F%2F127.0.0.1%3A8563%2F)。可以填入 IP，远程连接其它机器上的 arthas。启动之后，可以访问 [http://127.0.0.1:8080/](https://xie.infoq.cn/link?target=http%3A%2F%2F127.0.0.1%3A8080%2F) ，再通过 agentId 连接到已注册的 arthas agent 上，如下图所示。

![](_assets/f0cfbeab6eedfe0996c38bc0a8c1e300.png)

通过 Spring Boot 的 Endpoint，可以查看到具体的连接信息： [http://127.0.0.1:8080/actuator/arthas](https://xie.infoq.cn/link?target=http%3A%2F%2F127.0.0.1%3A8080%2Factuator%2Farthas) ，

![](_assets/42d45a04622c83255e1f675996d8f753.png)

登陆用户名是 arthas，密码在 arthas tunnel server 的日志里可以找到，比如：

![](_assets/ae7663c430e76c3ac14c319570501dc0.png)

> **注意：默认情况下，arthas 只 listen 127.0.0.1，所以如果想从远程连接，则可以使用 --target-ip 参数指定 listen 的 IP，更多参考-h 的帮助说明。 注意会有安全风险，考虑下面的 tunnel server 的方案。** 

### 如何将服务连接 Arthas tunnel server

主要有两种模式连接 Arthas tunnel server：

1.  远程运行的 Arthas server 连接 Arthas tunnel server
    
2.  远程运行的 Arthas Spring Boot Starter 的 agent 探针服务连接 Arthas tunnel server
    

![](_assets/46be2bd96dffb255d3a2dbb49479fa36.png)

#### 启动 arthas 时连接到 tunnel server

在启动 arthas，可以传递--tunnel-server 参数，比如：

```
as.sh --tunnel-server 'ws://127.0.0.1:7777/ws'
```

如果有特殊需求，可以通过--agent-id 参数里指定 agentId。默认情况下，会生成随机 ID。attach 成功之后，会打印出 agentId。

```
,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.
 /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'

wiki      https://arthas.aliyun.com/doc
tutorials https://arthas.aliyun.com/doc/arthas-tutorials.html
version   3.1.2
pid       86183
time      2022-11-30 15:40:53
id        URJZ5L48RPBR2ALI5K4V
```

即使是启动时没有连接到 tunnel server，也可以在后续自动重连成功之后，通过 session 命令来获取 agentId：

```
[arthas@86183]$ session
 Name           Value
-----------------------------------------------------
 JAVA_PID       86183
 SESSION_ID     f7273eb5-e7b0-4a00-bc5b-3fe55d741882
 AGENT_ID       URJZ5L48RPBR2ALI5K4V
 TUNNEL_SERVER  ws://127.0.0.1:7777/ws
```

在浏览器里访问 http://localhost:8080/arthas，输入 agentId，就可以连接到本机/其他机器上上的 arthas 了。

##### tunnel server 的注意要点

*   agentId 要保持唯一，否则会在 tunnel server 上冲突，不能正常工作。
    
*   如果 arthas agent 配置了 appName，则生成的 agentId 会带上 appName 的前缀。
    

##### 添加对应的 app-name 参数

启动参数

```
as.sh --tunnel-server 'ws://127.0.0.1:7777/ws' --app-name demoapp ，则生成的 agentId 可能是demoapp_URJZ5L48RPBR2ALI5K4V。
```

> **Tunnel server 会以_做分隔符，提取出 appName，方便按应用进行管理**。

配置参数

解压的 arthas 目录下的 arthas.properties，或者在 spring boot 应用的 application.properties 里配置 appName。

#### Arthas Spring Boot Starter 的 agent 服务连接 Jar

> **只支持 springboot2**

*   maven 的仓库地址：https://search.maven.org/search?q=a:arthas-spring-boot-starter
    

![](_assets/4d80950fbb3415a64055d74f5edef4eb.png)

*   配置 maven 依赖：
    

arthas.version:3.6.7

```
<dependency>
     <groupId>com.taobao.arthas</groupId>
    <artifactId>arthas-spring-boot-starter</artifactId>
    <version>${arthas.version}</version>
</dependency>
```

应用启动后，spring 会启动 arthas，并且 attach 自身进程。如果你不知道如何创建或者引入哪些依赖，可以采用一键创建包含 Arthas Spring Boot Starter 的工程：[点击跳转到云原生脚手架](https://xie.infoq.cn/link?target=https%3A%2F%2Fstart.aliyun.com%2Fbootstrap.html%2F%23!dependencies%3Darthas)  

![](_assets/4cfa81c5f79e40204f15fe21e1087d4a.png)

可以看到最下面已经自动勾选了 arthas 的监控机制体系。

![](_assets/97bce0bc9256e25cd6ee37dcbe4a0525.png)

##### application.yml 配置

```
arthas:
  agent-name: nihaotest
  agent-id: URJZ5L48RPBR2ALI5K4V  #需手工指定agent-id
  tunnel-server: ws:
```

#### 查看 Endpoint 信息

需要配置 spring boot 暴露 endpoint：假定 endpoint 端口是 8080，则通过下面 url 可以查看：

> http://localhost:8080/actuator/arthas

```
{
    "arthasConfigMap": {
        "agent-id": "hsehdfsfghhwertyfad",
        "tunnel-server": "ws://47.75.156.201:7777/ws",
    }
}
```

> **最后，启动 SpringBoot 服务即可**。

#### 非 spring boot 应用使用方式

非 Spring Boot 应用，可以通过下面的方式来使用：

```
<dependency>
     <groupId>com.taobao.arthas</groupId>
     <artifactId>arthas-agent-attach</artifactId>
     <version>${arthas.version}</version>
 </dependency>
 <dependency>
      <groupId>com.taobao.arthas</groupId>
     <artifactId>arthas-packaging</artifactId>
     <version>${arthas.version}</version>
</dependency>
```

##### attach 本身的服务进行探针探测。

```
import com.taobao.arthas.agent.attach.ArthasAgent;
public class ArthasAttachExample {
  public static void main(String[] args) {
    ArthasAgent.attach();
  }
}
```

也可以配置属性：

```
HashMap<String, String> configMap = new HashMap<String, String>();
configMap.put("arthas.appName", "demo");
configMap.put("arthas.tunnelServer", "ws://127.0.0.1:7777/ws");
ArthasAgent.attach(configMap);
```

### Tunnel Server 的管理页面

需要在 spring boot 的 application.properties 里配置 arthas.enable-detail-pages=true

> 注意，开放管理页面有风险！管理页面没有安全拦截功能，务必自行增加安全措施。

在本地启动 tunnel-server，然后使用 as.sh attach，并且指定应用名--app-name test：

```
$ as.sh --tunnel-server 'ws://127.0.0.1:7777/ws' --app-name test
telnet connecting to arthas server... current timestamp is 1627539688
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
  ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.
 /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'

wiki       https://arthas.aliyun.com/doc
tutorials  https://arthas.aliyun.com/doc/arthas-tutorials.html
version    3.5.3
main_class demo.MathGame
pid        65825
time       2022-07-29 14:21:29
id         test_PE3LZO9NA9ENJYTPGL9L
```

然后访问 tunnel-server，可以看到所有连接的应用列表：

```
http://localhost:8080/apps.html
```

![](_assets/17ae0a9fbb7260497fac9acb64f61d1c.png)

再打开详情，则可以看到连接的所有 agent 列表：

```
http://localhost:8080/agents.html?app=test
```

![](_assets/bf79910867a12d7bebf6c07e4162b990.png)