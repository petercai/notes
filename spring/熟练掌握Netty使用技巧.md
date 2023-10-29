# 熟练掌握Netty使用技巧
👈👈👈 **欢迎点赞收藏关注哟**

> **首先分享之前的所有文章 >>>>** 😜😜😜  
> **文章合集 :** 🎁 [juejin.cn/post/694164…](https://juejin.cn/post/6941642435189538824 "https://juejin.cn/post/6941642435189538824")  
> **Github :** 👉 [github.com/black-ant](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fblack-ant "https://github.com/black-ant")  
> **CASE 备份 :** 👉 [gitee.com/antblack/ca…](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fantblack%2Fcase "https://gitee.com/antblack/case")

前言
--

**Netty 祭天，法力无边。** 

哈哈哈 ， Netty 这组件，哪怕你从来没主动用过，但是它无时无刻都在你身边。可以说，阿里系或者一些高级开源工具在实现 Client 和 Server 的时候都选择使用了 Netty。

第一步 ： 理解场景
----------

**首先我们要理解 Netty 到底能做什么？**

Netty 是一个基于 JAVA NIO 类库的`异步通信框架`，用于创建异步非阻塞、基于事件驱动、高性能、高可靠性和高可定制性的`网络客户端和服务器端`.

那么很明显了，Netty 是为了通信的 ，其优势就是`性能好，不会阻塞`。这两点就足够大多数框架用它来做通信。

**那么问题来了，系统间的通信用来做什么？**

我们其实时时刻刻都在用 Netty ，比较常见的就是开源框架，包括 ：`Nacos` ，`Seata` ,`Sentinel`

相对而言这些访问量都不会太高，更高并发的使用就包括 : `Dubbo` ，`gRPC` , `RokcetMQ`

而从业务角度讲主要2个方向 ：**通信** （`IM 即时通讯`） 和 **数据传递** （`消息发送工具`）

第二步 ： 了解组件
----------

### 2.1 组件的层级展示

*   **单个组件：** 
*   **Channel** : 一个**网络连接**的抽象表示 ， 可以用于读取或者写入数据
*   **ChannelPipeline** ：一个**链路**，channel 接收的数据会通过 ChannelPipeline 进行传递
*   **ChannelHandler** ：一个**处理器**，在一个通道中有多个 Handler 用于处理事件和数据，执行各种特定的任务和逻辑
*   **EventLoop** ： 一个**线程**，用于处理事件和执行任务，可以处理 Channel 上面所有的物理操作
*   **容器类型：** 
*   **Bootstrap** & **ServerBootstrap** ： 用于创建和配置服务器，从而启动服务器入口
*   **ChannelHandlerContext** ：用于绑定 `ChannelHandler` 和 `ChannelPipeline`

> 问题一 ： Channel 和 EventLoop 的关系？

*   一个EventLoopGroup 包含`一个或者多个`EventLoop；
*   一个EventLoop 在它的生命周期内只和`一个`Thread 绑定；
*   所有由EventLoop 处理的I/O 事件都将在它专有的Thread 上被处理；
*   一个Channel 在它的生命周期内只注册于一个EventLoop，**所有的事件处理和任务都会在这个 EventLoop 上处理**
*   一个EventLoop 可能会被分配给`一个或多个`Channel。
*   多个 EventLoop 可以存在一个 EventLoopGroup 里面

> 问题二 ： EventLoopGroup 有什么作用？

*   `EventLoopGroup` 用于管理 EventLoop ，包括将 EentLoop 分配给 Channel ，包括 EventLoop 的协同

### 2.2 关系图

> Channel 和 EventLoop 的关系

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/645e34f5790a4d8fb15fd64ed96b0738~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=958&h=567&s=47347&e=png&b=fafafa)

> Channel 和 Handler 的关系

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fac638f885574269963593f58ec263c1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

第三步 ： 熟悉用法
----------

### 3.1 一个最简单的 Netty 案例

首先需要我们理清楚思路，从无到有搭建 Server 和 Client 需要哪些步骤 ：

*   首先要准备好 Server 端和 Client 端 ，他们要`监听同一个端口`
*   准备好 `Handler` 类用于对数据进行处理
*   双端进行通信，`然后互相发送消息`
*   完成后记得 Close ，和 IO 一样，都`需要关闭流`

### 3.2 搭建好 Server 端服务器

*   1.  创建好 `ServerBootstrap` 对象用于开始搭建整个服务器
*   2.  为 `ServerBootstrap` 配置好 `eventLoopGroup` ， 这里的 Group 有2个 ：
    
    *   **BossGroup** : 用于处理连接请求 ，本身包含多个 EventLoop ，用于创建 Channel 转发给 WorkerGroup
    *   **workerGroup** : 处理已建立的连接 ，同样包含自己的 EventLoop
*   3.  配置服务器处理的 `Channel` 连接的类型 ， 包括 **NIO** 类型 ， **EPOLL** 类型等
*   4.  配置整个处理流程的 `ChannelHandler` ，用于特定的处理
*   5.  为这个 Server 端 `bind()` 一个端口 , 同时阻塞等待绑定完成

**基于这个流程，整个 Server 端的创建可以分为2个步骤 ：** 

```java

public class NettyServerHandler extends ChannelInboundHandlerAdapter {
    
     * 客户端连接会触发
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("S2 : 服务端建立连接，触发 Active .....");
    }

    
     * 客户端发消息会触发
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("服务器收到消息: " + msg.toString());
        
        ctx.write(msg + "World!");
        ctx.flush();
    }

    
     * 发生异常触发
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}



public class ServerChannelInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {
        
        socketChannel.pipeline().addLast("decoder", new StringDecoder(CharsetUtil.UTF_8));
        socketChannel.pipeline().addLast("encoder", new StringEncoder(CharsetUtil.UTF_8));
        
        socketChannel.pipeline().addLast(new NettyServerHandler());
    }
}


public class NettyServer extends Thread {

    public void run() {

        
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workGroup = new NioEventLoopGroup(200);

        
        ServerBootstrap bootstrap = new ServerBootstrap()
                .group(bossGroup, workGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ServerChannelInitializer());

        try {
            InetSocketAddress socketAddress = new InetSocketAddress("127.0.0.1", 8090);
            
            
            ChannelFuture future = bootstrap.bind(socketAddress).sync();
            System.out.println("S1 : 服务端构建完成 .....");
            future.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            
            bossGroup.shutdownGracefully();
            
            workGroup.shutdownGracefully();
        }
    }
}

```

### 3.3 搭建好客户端

*   1.  不同于 Server ，Client 端只需要一个 `EventLoopGroup`
*   2.  同样的，创建 Bootstrap ，绑定 `Group / channel / handler`
*   3.  通过 `writeAndFlush` 发送消息

```java

public class NettyClientHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("S2 : 客户端建立连接，触发 Active .....");
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("客户端收到消息: " + msg.toString());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}



public class NettyClientInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {
        socketChannel.pipeline().addLast("decoder", new StringDecoder());
        socketChannel.pipeline().addLast("encoder", new StringEncoder());
         
        socketChannel.pipeline().addLast(new NettyClientHandler());
    }
}



public class NettyClient extends Thread {

    public void run() {
        EventLoopGroup group = new NioEventLoopGroup();
        Bootstrap bootstrap = new Bootstrap()
                .group(group)
                .channel(NioSocketChannel.class)
                .handler(new NettyClientInitializer());
        try {

            ChannelFuture future = bootstrap.connect("127.0.0.1", 8090).sync();
            System.out.println("S2 : 客户端构建完成 .....");

            for (int i = 0; i < 10; i++) {
                String message = "第" + i + "条消息 ， " + "Hello ";
                future.channel().writeAndFlush(message);
                Thread.sleep(1000); 
            }

            future.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }
}


```

### 3.4 发起请求

在这个环节里面通过2个线程模拟消息的发送和接收 ：

```java
public class DemoService {

    public static void main(String[] args) throws InterruptedException {
        System.out.println("Start ：开始整个 Netty 搭建流程");

        
        NettyServer server = new NettyServer();
        server.start();

        NettyClient client = new NettyClient();
        client.start();
    }

}

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/457d0d6c4eff452bb5c0362bf3b50eb0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=907&h=701&s=74342&e=png&b=2b2b2b)

可以看到流程很清晰了，先创建双端，然后双端建立连接，随后发生交互

四. API 高级用法
-----------

### 4.1 Bootstrap 包含哪些方法 @ ChatGPT

*   **group** ：设置用于处理事件循环（EventLoop）的 EventLoopGroup
*   **channel** ：指定 Channel 的类型 (NioSocketChannel / NioServerSocketChannel)
*   **handler** : 在初始化过程中添加一个 ChannelHandler
*   **childHandler** : 用于 ServerBootstrap，在连接被接受后，为新注册的 Channel 设置 ChannelHandler
*   **localAddress** : 指定本地端口或地址，用于连接或绑定
*   **remoteAddress** : 指定远程服务器的地址
*   **connect** : 用于客户端连接指定远程地址
*   **bind** : 用于服务器绑定到指定的本地地址

总结
--

东西很简单，因为我在开发过程中对 Netty 的底层代码也只是轻度使用，大部分都是基于外层封装的组件在使用。

这一篇主要是为了学习，所以会写不深。后续会陆陆续续从高级用法 高并发用法 源码 等多个角度深入，欢迎关注。

相关文档 ：
------

[\# Netty : 大纲及笔记](https://juejin.cn/post/6991432041917071367 "https://juejin.cn/post/6991432041917071367")