# Spring Cloud Feign工作原理
**一、Feign 的工作原理**

Spring Cloud Feign 是一个声明式的 Web 服务客户端，它使得编写 Web 服务客户端变得非常容易。它是基于 Netflix Feign 开发的，是一个轻量级的 RESTful HTTP 客户端。

让我们来看一下 Feign 的工作原理：

*   声明式 REST 客户端: Feign 提供了一种更简单的方法来定义和创建 REST 客户端。通过创建接口并用注解来配置请求，开发者可以非常容易地定义服务端点。
    
*   集成 Ribbon 和 Hystrix: Feign 自然集成了 Ribbon（负载均衡器）和 Hystrix（断路器），这意味着在使用 Feign 时，您可以非常容易地实现服务的负载均衡和容错。
    
*   注解支持: Feign 使用注解如 @FeignClient 来声明一个接口是一个 Feign 客户端，这个注解包含了服务的名称和配置信息。通过注解 @RequestMapping 或 @GetMapping 等，您可以定义请求的 URL、方法类型等。
    
*   动态代理: 当您定义一个 Feign 客户端接口时，Feign 使用动态代理来生成实现。这个实现内部处理了 HTTP 请求的发送和结果的映射。
    
*   请求和响应的处理: Feign 内部使用了 HttpClient 来发送 HTTP 请求，并且会自动处理请求和响应。您可以使用不同的编码器和解码器来处理请求和响应的数据格式。
    
*   易于扩展: Feign 提供了多种插件，例如编码器、解码器、拦截器等，这些都可以用来自定义 Feign 的行为。
    

**二、@FeignClient注解工作原理**

@FeignClient 注解的工作原理是在运行时动态创建带注解接口的代理。该接口中的每个方法对应于对注释中指定的服务的 HTTP 请求。当调用该接口的方法时，Spring会拦截该调用并将其转换为HTTP请求，包括URL映射、请求和响应正文转换以及标头设置。然后，它将请求发送到目标服务，处理响应，并将其作为方法的返回值返回。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc2eec03082f41bfbf1d208b29717c7a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=756&h=777&s=125235&e=png&b=ffffff)

**三、@FeignClient源码解析**

1\. 获取客户端名称

源码中可以看到类FeignClientsRegistrar的方法registerFeignClients注册客户端

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dfe425c5df34bf791f26a3ab40164ff~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1424&h=504&s=95469&e=jpg&b=2c2c2b)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0d148fb57334170822d9d822be15ed4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1554&h=672&s=135867&e=jpg&b=2c2c2b)

2\. 获取configuration

接着就是将configuration中的配置类加载到BeanDefinitionRegistry中了

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d730d8f08af4845b1dd97cb55d0257c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1464&h=600&s=106941&e=jpg&b=2d2d2d)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e0daa7a1ebf44998af53dbb909dcbe1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1556&h=520&s=145740&e=jpg&b=2d2c2c)

3\. 注册客户端

接着就是registerFeignClient 注册客户端了，首先会将注解属性添加到definition

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d58b01cc538f4da7b3ee6f0b5da18def~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1464&h=480&s=81358&e=jpg&b=2d2d2c)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e72da88abb154118b4d3ca363a134eed~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1698&h=852&s=248309&e=jpg&b=2c2c2b)

最后Feign接口注册到了注册器中，名称为包名+接口名，Feign接口实际生成的Bean对象为FeignClientFactoryBean

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18ab3d01d40741e1bac19e2926f6034d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1766&h=726&s=174764&e=jpg&b=2d2d2d)