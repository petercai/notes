# 一文带你入门Go云原生微服务
> Go 云原生开发有着天然的优势，云原生系统需要可扩展、耦合、弹性可管理。Go 微服务框架很多，包括：go-micro、go-zero、go-kit、go-kratos、tars-go、dubbo-go、jupiter 等等...

Go 的设计就是为云原生时代构建的语言：简单高效、快速编译、支持现代网络和多核计算、支持高并发、内存安全,帮助用户专注于解决问题而不是受限于语言的复杂性。

要想一文入门 Go 微服务还是需要有些前提条件的，打好基础：

1.  已经掌握了 Golang 的基础知识，如果基础薄弱，欢迎学习的我的[Go语言学习专栏](https://xie.infoq.cn/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fmp%2Fappmsgalbum%3F__biz%3DMzIyNjM0MzQyNg%3D%3D%26action%3Dgetalbum%26album_id%3D2560252212141162497%26scene%3D173%26from_msgid%3D2247484678%26from_itemidx%3D1%26count%3D3%26nolastread%3D1%23wechat_redirect)，查漏补缺。
    
2.  了解 Protobuf，知晓相关知识点。补课可以阅读这篇文章：[\# 一文带你玩转ProtoBuf](https://xie.infoq.cn/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIyNjM0MzQyNg%3D%3D%26mid%3D2247484451%26idx%3D1%26sn%3D92b25ba066c202ba6af3d8bad047a082%26chksm%3De870ab4edf072258bd96a15a854ed8912710527ce6e3d0660b4b26830965b290cf07781d7c62%26token%3D2066671154%26lang%3Dzh_CN%23rd)  
    
3.  了解 RPC 的概念，知晓 gRPC 如何使用。可以阅读这 2 篇文章快速入门：[\# Go RPC入门指南：RPC的使用边界在哪里？如何实现跨语言调用？](https://xie.infoq.cn/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIyNjM0MzQyNg%3D%3D%26mid%3D2247484350%26idx%3D1%26sn%3D7988754c24e93a5a6d503acfd59ea021%26chksm%3De870acd3df0725c584cc88c645938c5d462486fa7ea90eeada39204e009d5132ed8042d8e3c1%26token%3D2066671154%26lang%3Dzh_CN%23rd)和 [\# 开发gRPC总共分三步](https://xie.infoq.cn/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIyNjM0MzQyNg%3D%3D%26mid%3D2247484496%26idx%3D1%26sn%3D85fb5996f3f4fc8b5dda5f1c7ddd9bb9%26chksm%3De870ab3ddf07222b12b4b7e1b043c5ae852a1412adaa1ed5ac111524eb2067fdbaf13b5af3b8%26token%3D2066671154%26lang%3Dzh_CN%23rd)。
    

我简单整理了一下，让大家先有个整体的认识：

简单横评各个框架
--------

我计划系统的介绍 2 个微服务框架的_入门和实战教程_：**“go-micro”和“go-zero”**。

之所以选择这两个框架是因为：

1.  我司使用的 go-micro，发现这个框架开发微服务够经典，够纯粹，非常适合入门学习微服务。
    
2.  通过和有经验的小伙伴交流，go-zero 也是一个非常优秀的框架，在国内社区生态建设和维护上，完美适配国内开源的现状，社区非常活跃，使用率会越来越高。
    

go-zero 的入门指南，在后面会为大家做分享，欢迎关注我。

概念
--

> 微服务是一种软件架构模式，用于将大型整体应用程序分解为更小的可管理独立服务，这些独立服务通过跨语言的协议进行通信，每个服务都专注于做好一件事情。

软件架构演进史
-------

如果你对上面这段定义表示晦涩难懂，可以阅读这篇文章：[\# 给想转Go或者Go进阶同学的一些建议](https://xie.infoq.cn/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FaNmutZr8-dKCgBD5Od7SaQ)：

我结合自己的经历聊了聊软件架构演进史：从单机架构到集中式架构，再到当前主流的分布式微服务架构，介绍了微服务和领域驱动设计 DDD 的知识点。

简单理解
----

用我的话解释微服务就是：

**把复杂的项目拆成微(小的)服务，明确服务与服务之间的边界，服务内部高内聚，服务之间松耦合。** 

微服务的好处
------

1.  **降低开发难度**：微服务架构使得团队各个成员只需要关注自己的业务，管理好自己的服务；
    
2.  **独立部署**：各个微服务之间可以分开部署，不影响其他服务的使用，上线部署不需要其他团队的配合支持。
    
3.  **提高容错和容灾能力**：微服务之间独立，最大程度降低一个服务对另外一个服务的影响；
    
4.  **提高团队执行效率**：各个团队独立开发、部署、维护项目，不需要等待其他团队的支持。
    
5.  **提高项目复用性**：微服务的架构设计，能将高复用利用到极致，不同的服务不会做同一件事情，避免资源浪费。
    
6.  **降低企业成本**：微服务支持跨语言、跨团队开发，不需要大规模的替换单体架构或者集中式架构中的工程师。
    

我们在了解微服务的好处之后，继续探究一下 go-micro 的特点，了解一下 go-micro 是如何实现微服务开发的：

> go-micro 是一个简化分布式开发的微服务生态系统。它为开发分布式应用程序提供了基本的构建模块。

go-miro 的设计哲学是：通过提供组件工具，明确微服务开发的边界，让我们专注于开发业务本身。

下面就重点介绍一下构成组件：

构成组件
----

### Go Micro

*   用于在 Go 中编写微服务的**插件式 RPC 框架**。它提供了**用于服务发现、客户端负载平衡、编码、同步和异步通信库**。
    
*   这是我们学习 go-micro 最重要的库，我们开发微服务的重点也是编写 RPC 服务。
    

### API

*   提供并将 HTTP 请求**路由到对应微服务的 API 网关**。它充当单个入口点，可以用作反向代理或将 HTTP 请求转换为 RPC。
    
*   我们开发微服务项目，说到底是编写 RPC 服务，如何让外部请求可以访问到 RPC 服务呢？就是通过 **API**
    

### Sidecar

*   一种对语言透明的 RPC 代理（也就是和语言无关），具有 go-micro 作为 HTTP 端点的所有功能。虽然 Go 是构建微服务的优选语言，但商业项目中也经常使用其他语言一起开发微服务。因此 Sidecar 提供了一种将其他语言应用程序集成到 Micro 世界的方法。
    
*   _Sidecar 又被称为服务网格，其作用是：第一，微服务治理与业务逻辑的解耦。第二，异构系统的统一治理。_
    

### Web

*   用于 Micro Web 应用程序的仪表板和反向代理。官方认为基于微服务建立 web 应用是通用场景，因此 Web 被视为微服务领域的一等公民。它的行为非常像 API 反向代理，另外也包括对 web sockets 的支持。
    
*   _我们可以通过仪表盘直观的观察到微服务项目的运行情况_，后面会重点介绍。
    

### CLI

*   提供命令行界面让我们和微服务进行交互。CLI 还可以使我们利用 Sidecar 作为代理。
    
*   **CLI** 是 microservices toolkit micro 的命令行界面，包括获取服务、查看服务、查询服务健康状态等功能；我们还可以通过 Sidecar 代理 CLI。
    

### Bot

*   Hubot 风格的 bot。在我们的微服务平台中，可以通过 Slack，HipChat，XMPP 等进行交互。它通过消息传递提供 CLI 的功能。可以添加其他命令来自动执行常见的操作任务。
    
*   微型机器人 Bot 是一个位于微服务环境中的机器人，机器人是如何工作的呢？
    
*   _Bot 使用它的命名空间监视服务注册中心的服务。默认名称空间是_`_go.micro.bot_`_。该名称空间内的任何服务都将自动添加到可用命令列表中。执行命令时，机器人将使用_`_Command.Exec_`_方法调用该服务。_
    

总结
--

相比于 GoFrame、Gin 这类 Web 框架，我们发现微服务框架的组件构成更为复杂。

**Go Micro** 是我们用于编写微服务的 RPC 框架，**入门阶段重点理解 Go Micro 组件即可**，其他的组件会在后续文章中详细介绍。

下面我们重点看一下 Go Micro 组件的架构：

> Go Micro 为微服务提供了基本的构建模块，其目标是简化分布式系统开发。

因为微服务是一种架构模式，Micro 的架构思路是通过工具组件进行拆分，简化我们开发微服务项目的难度；Micro 的设计哲学是可插拔的插件化架构。

**Go-micro 是微服务的独立 RPC 框架，也是我们学习 go-micro 的核心。** 

我们看下面的架构图：

![](https://static001.geekbang.org/infoq/0f/0fbceb7337690b303a353b80e85c3625.png)

*   最顶层是 service，代表一个微服务
    
*   服务下面是两个端：客户端和服务端。（_注意区分 Service 和 Server，我在刚接触的时候混淆了这两个概念，service 指的是服务；server 服务端包含在 service 中，和 client 客户端一起作为 service 的底层支撑_）
    
*   **服务端 Server**：用于构建微服务的接口，提供用于 RPC 请求的方法。
    
*   **客户端 Client**：提供 RPC 查询方法，它结合了注册表，选择器，代理和传输。它还提供重试机制，超时机制，使用上下文等，是我们入门阶段的重点。
    

架构图中最底层的组件对于初学微服务的同学肯定不熟悉，下面来重点介绍一下：

### Registry 注册中心

**注册中心提供可插入的服务发现库，来查找正在运行的服务。**  默认的实现方式是 consul。

我们也可以很方便的修改为 etcd，kubernetes 等。毕竟可插拔是 go-micro 重要特性。

### Selector 负载均衡

**Selector 选择器实现 go-micro 的负载均衡机制。** 

原理是这样的：**当客户端向服务器发出请求时，首先查询服务的注册中心，注册中心会返回一个正在运行服务的节点列表，选择器将选择这些节点中的其中一个用于查询请求。** 

多次调用选择器将使用平衡算法。目前的方法是循环法、随机哈希、黑名单。go-micro 就是通过这种机制实现负载均衡的。

### Broker 事件驱动：发布订阅

**Broker 是发布和订阅的可插入接口。** 

微服务是一个事件驱动的架构，发布和订阅事件应该是一流的公民。目前的实现包括 nats，rabbitmq 和 http。

### Transport 消息传输

**传输是通过点对点传输消息的可插拔接口。** 

目前的实现是 http，rabbitmq 和 nats。通过提供这种抽象，运输可以无缝地换出。

总结
--

以上这些就是 go-micro RPC 框架的底层支持组件。

我们了解微服务和 go-micro 的知识点后可能还是有些懵，这很正常，毕竟知识点过于密集。

下面和我一起动手实践吧：

注意：go-micro 版本不兼容的问题最被大家吐槽，**下面我演示的依赖安装和示例代码，均以我司使用的 go-micro v2 版本。** 

我决定使用[\# 和大象装冰箱一样：开发gRPC总共分三步](https://xie.infoq.cn/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIyNjM0MzQyNg%3D%3D%26mid%3D2247484496%26idx%3D1%26sn%3D85fb5996f3f4fc8b5dda5f1c7ddd9bb9%26chksm%3De870ab3ddf07222b12b4b7e1b043c5ae852a1412adaa1ed5ac111524eb2067fdbaf13b5af3b8%26token%3D2066671154%26lang%3Dzh_CN%23rd)的示例，编写 go-micro 的入门示例：_通过原生开发 gRPC 和使用 go-micro 开发做个对比，你会发现使用 go-micro 开发微服务多么的简单。_

我们参考 go-micro 的官方示例，实现一个问候服务，实现经典的 Hello World：

准备工作
----

### 1\. 安装 micro v2

```
#安装go-micro
go get github.com/micro/go-micro/v2
#安装工具集
go get github.com/micro/micro/v2
```

### 2\. 安装 protobuf 插件

```
#安装protobuf
go get github.com/golang/protobuf/{proto,protoc-gen-go}
#指定版本安装生成micro代码的工具集
go get github.com/micro/micro/v2/cmd/protoc-gen-micro
```

我们在编写 proto 文件之后，使用 _protoc-gen-go_ 自动生成代码，所以需要提前安装好依赖。

### 3\. 创建项目结构

我们在合适的目录创建项目目录，比如我选择在我的 Go 安装目录`/Users/wangzhongyang/go/src/`新建 go-micro 目录，用于统一管理 go-micro 相关的项目。

```
cd /Users/wangzhongyang/go/src/
mkdir go-micro
```

在 go-micro 目录下创建 helloworld 目录，用于编写本次的演示项目。

```
cd go-micro
mkdir helloworld
cd helloworld
```

在 helloworld 目录下新建 proto 目录用于编写 proto 文件，另外创建 main.go 文件作为服务的入口文件：

准备工作做好之后，下面开发微服务的步骤和“**开发 gRPC 总共分三步**”是一样的：

1.  写 proto 文件定义服务和消息
    
2.  使用 protoc 工具生成代码
    
3.  编写业务逻辑代码提供服务
    

1\. 编写 proto 文件
---------------

我们在 _helloword/proto_ 目录下 新建 _greeter.proto_ 文件

编写 proto 文件和用什么微服务框架没有关系，我们都需要定义 syntax、service 和 message

```
syntax = "proto3";


option go_package ="./;pb";

service Greeter {
  rpc Hello(Request) returns (Response) {}
}

message Request {
  string name = 1;
}

message Response {
  string greeting = 1;
}
```

2\. 使用 protoc 工具生成代码
--------------------

我们打开控制台，切换到 proto 目录下，比如我的目录是：

```
cd  /Users/wangzhongyang/go/src/go-micro/helloworld/proto
```

执行自动生成代码命令：

**注意：我们必须使用带有 micro plugin 的 protoc 编译它。** 

```
protoc --proto_path=. --micro_out=. --go_out=. greeter.proto
```

我们发现执行上述命令后，生成了 pb.go 文件和 pb.micro.go 文件。

但是有报错，不用担心，接着往下看：

![](https://static001.geekbang.org/infoq/12/1224be5e022aaa7624dfebb42a6a8fa4.png)

### 2.1 解决报错

报错原因是因为没有导入依赖，我们在项目根目录下执行：

同步依赖后，发现报错消失了：

![](https://static001.geekbang.org/infoq/ba/ba4dea734b5e4b5166936acc7aa8623d.png)

3\. 编写业务逻辑代码提供服务
----------------

### 3.1 编写服务端

1.  我们直接在 main.go 中编写服务
    
2.  import 中的 proto 对应的目录改成自己的。你自己的 module 名称可以在 go.mod 中查看：
    

![](https://static001.geekbang.org/infoq/01/019fb21c3c7d9cc217bba9ac7541a94c.png)

3.  关键代码已加注释，逻辑和[\# 开发gRPC总共分三步](https://xie.infoq.cn/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIyNjM0MzQyNg%3D%3D%26mid%3D2247484496%26idx%3D1%26sn%3D85fb5996f3f4fc8b5dda5f1c7ddd9bb9%26chksm%3De870ab3ddf07222b12b4b7e1b043c5ae852a1412adaa1ed5ac111524eb2067fdbaf13b5af3b8%26token%3D2066671154%26lang%3Dzh_CN%23rd) 的入门实践部分非常像。
    

```
package main

import (
   "context"
   "fmt"
   micro "github.com/micro/go-micro/v2"
   proto "go-micro/helloworld/proto" 
)


type Greeter struct{}


func (g *Greeter) Hello(ctx context.Context, req *proto.Request, rsp *proto.Response) error {
   rsp.Greeting = "Hello " + req.Name
   return nil
}

func main() {
   
   service := micro.NewService(
      micro.Name("greeter"),
   )
   
   
   service.Init()

   
   err := proto.RegisterGreeterHandler(service.Server(), new(Greeter))
   if err != nil {
      return
   }

   
   if err := service.Run(); err != nil {
      fmt.Println(err)
   }
}
```

服务端编写好之后我们再编写客户端：

### 3.2 编写客户端

在项目根目录下，新建 client 目录，新建 client.go 文件，用于编写客户端代码

创建目录和文件：

```
mkdir client
cd client
touch client.go
```

编写代码：

1.  关键逻辑已经添加注释
    
2.  NewGreeterService()和 Hello()都是 proto 文件自动生成的的，定义在.pb.micro 文件中
    

```
package main

import (
   "context"
   "fmt"
   micro "github.com/micro/go-micro/v2"
   proto "go-micro/helloworld/proto"
)

func main() {
   
   service := micro.NewService(micro.Name("greeter.client"))
   
   service.Init()

   
   greeter := proto.NewGreeterService("greeter", service.Client())

   
   rsp, err := greeter.Hello(context.TODO(), &proto.Request{Name: "World"})
   if err != nil {
      fmt.Println(err)
   }

   
   fmt.Println(rsp.Greeting)
}
```

我们来看一下 .pb.micro.go 文件的源码，重点看一下 Hello 方法：

![](https://static001.geekbang.org/infoq/a9/a9aa38bb5dd5a4c15c71193b365d1ffc.png)

到这里我们就编码完毕，看下执行效果：

### 3.3 启动服务，进行调用

#### 1.先启动服务端

我们打开控制台，切换到项目根目录下，执行命令：

执行效果如下，服务端已经启动：

![](https://static001.geekbang.org/infoq/63/63f1def188d3a60fe4082a9829b164ea.png)

#### 2.再启动客户端

我们另外打开一个控制台，启动客户端服务：

```
cd go-micro/helloworld/client/

go run client.go
```

执行效果如下，和我们预期的效果一样，成功的打印出了 Hello World：

![](https://static001.geekbang.org/infoq/d2/d2c8245a8e2255cd5a41dc0f8025dbf5.png)

分布式微服务架构已成趋势，越来越多的公司在从单体应用或集中式应用向分布式应用转型。开篇类比了主流 Go 微服务框架的特点，Go 的微服务生态可以说是百家争鸣。

本文也介绍了微服务的特点和优势，go-micro 的架构和构成组件；类比原生开发 rpc 项目，用 go-micro 实现了经典的 Hello World 问候服务，带大家入门了微服务开发。

近期会更新一系列 Go 实战进阶的文章，欢迎大家关注我的：[\# Go语言进阶实战专栏](https://xie.infoq.cn/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fmp%2Fappmsgalbum%3F__biz%3DMzIyNjM0MzQyNg%3D%3D%26action%3Dgetalbum%26album_id%3D2560252212141162497%26scene%3D173%26from_msgid%3D2247484678%26from_itemidx%3D1%26count%3D3%26nolastread%3D1%23wechat_redirect)。

![](https://static001.geekbang.org/infoq/cf/cfb1b99578b0bfcdb129154bd4d04dcc.png)

