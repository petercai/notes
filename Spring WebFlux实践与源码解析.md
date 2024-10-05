# Spring WebFlux实践与源码解析
Spring WebFlux和Spring MVC区别
---------------------------

如下来自官网Spring MVC和Spring WebFlux技术栈的区别。 可以看到Spring MVC主要是基于Servlet API使用同步阻塞IO架构，每来一个请求都需要启动一个线程进行处理。这种架构对于大量I/O密集型的请求，需要同时启动大量的线程来处理请求，消耗更多的资源。而Spring WebFlux使用的是全流程的异步非阻塞的架构，处理I/O任务不需要阻塞等待，只需少量的线程，就能处理大量的请求，能够极大提升系统吞吐量。

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/21fa171bd8604363a648393e75048f90~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5ZCN5a2X55qE6Zeu6aKY:q75.awebp?rk3s=f64ab15b&x-expires=1728480458&x-signature=63YXr6sfalFPY%2FGSYCRfnU8hWl4%3D)

网上关于 Spring MVC和Spring WebFlux对比：

| 特性 | **Spring MVC** (Servlet 3.1) | **Spring WebFlux** |
| --- | --- | --- |
| **底层架构** | 基于 Servlet API，虽然支持异步和非阻塞，但主要是阻塞架构 | 完全非阻塞、异步架构，基于反应式流 |
| **并发性** | 提供了异步处理和非阻塞 I/O，但本质上仍是线程池模型 | 高并发、低线程占用，处理更多 I/O 密集型任务 |
| **I/O 模型** | 基于 Servlet 3.1 的非阻塞 I/O，依赖于 Servlet 容器 | 支持多种 I/O 模型（Netty、Undertow、Servlet 容器等） |
| **编程模型** | 传统的 MVC 模式，使用 `Callable` 和 `DeferredResult` 进行异步 | 反应式编程，使用 `Mono` 和 `Flux` 处理异步和流式数据 |
| **使用场景** | 适合传统 Web 应用程序，部分异步请求处理，轻量异步场景 | 高并发场景、流式数据处理、WebSocket、长连接等 |
| **生态系统** | 依赖于 Servlet 生态和传统的 Java EE/Web 容器 | 反应式生态，支持 Netty 和其他现代非阻塞网络技术。 |

响应式编程
-----

### Reactive Streams 规范

Reactive Streams 是一项旨在为具有非阻塞背压的异步流处理提供规范。定义了Publisher,Subscriber,Subscription,Processor 4个API组件。

1.  Publisher 发布者是潜在无限数量序列元素的提供者，它收到订阅者订阅需求发布这些元素

```java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}

```

Publisher接口定义了subscribe 方法，该方法接受一个 Subscriber 对象作为参数。

Publisher向onNext发出信号的 的总数必须始终小于或等于Subscriber 的 Subscription 请求的元素总数。

Publisher可以通过调用onComplete或onError终止订阅。

2.  Subscriber 定义了订阅者（消费者）的接口，负责订阅并处理数据流

```java
public interface Subscriber<T> {
    
    public void onSubscribe(Subscription s); 
    
    public void onNext(T t);
    
    public void onError(Throwable t);
    
    public void onComplete();
}

```

订阅者通过 Subscription.request(long n) 方法来请求数据，并接收 onNext 信号。如果订阅者不再需要订阅需要调用Subscription.cancel()方法取消订阅。

3.  Subscription 定义了订阅者和发布者之间的订阅关系，用于控制数据流的速度

```java
public interface Subscription {
    public void request(long n);
    public void cancel();
}

```

Subscription 的 request 和 cancel 方法必须只在 Subscriber 的上下文中被调用

4.  Processor 是一个特殊类型的发布者，它也是订阅者，可以用来处理数据流。Processor可以接收来自上游发布者的数据，进行处理（如转换、过滤等），然后将处理后的数据发送给下游的订阅者

```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}

```

已实现 Reactive Streams 规范的库：

*   RxJava
*   Project Reactor
*   JDK9

### Project Reactor

Project Rector是一个完全非阻塞的基础框架，支持背压（Back-pressure）。是Spring生态系统响应式技术栈的核心，广泛应用于Spring WebFlux、Spring Data 和 Spring Cloud Gateway。

Reactor 提供了Mono和Flux两种API类型,用于表示和处理异步数据流，Mono和Flux提供了大量的操作符，用于对数据流进行各种转换、过滤、组合、错误处理等操作。

*   Mono
    
    *   Mono.just(T data)：创建一个包含指定数据的 Mono 实例
    *   Mono.empty()：创建一个不包含数据的 Mono 实例
    *   Mono.error(Throwable error)：创建一个以错误结束的 Mono 实例。
*   Flux
    
    *   Flux.just(T... data)：创建一个包含多个指定数据的 Flux 实例。
    *   Flux.fromIterable(Iterable iterable)：根据一个可迭代对象创建一个 Flux
    *   Flux.range(int start, int count)：生成从指定起始值开始的 count 个整数值

Spring WebFlux
--------------

### 编程模型

Spring WebFlux 支持两种编程模型，

*   基于注解的编程模型（Annotated Controllers）
    
    类似于传统的 Spring MVC，使用注解来定义控制器、路由和处理请求的方法。开发者可以复用熟悉的 Spring MVC 注解，如 @Controller、@RequestMapping、@GetMapping、@PostMapping 等，同时可以利用 WebFlux 提供的非阻塞、异步特性。
    

```java
@RestController
public class GreetingController {

    @GetMapping("/hello")
    Mono<String> greeting() {
        return Mono.just("hello WebFlux");
    }
}


```

*   基于Lambda 的轻量级函数式编程模型
    
    这种模型是 Spring WebFlux 新引入的一种编程方式，基于 Java 8 的 lambda 表达式，提供了一种更为灵活和声明式的方式来定义路由和处理逻辑。这种模式没有使用注解，而是直接通过函数式 API 来定义路由和处理逻辑。
    

```java
@Configuration(proxyBeanMethods = false)
public class GreetingRouter {
    @Bean
    public RouterFunction<ServerResponse> route(GreetingHandler greetingHandler) {

        return RouterFunctions
                .route(GET("/hello").and(accept(MediaType.APPLICATION_JSON)), greetingHandler::hello);
    }
}

```

### WebFluxConfigurer

spring WebFluxConfigurer 是 Spring WebFlux 提供的一种接口，允许开发人员通过实现该接口来自定义 WebFlux 应用的配置。它类似于 Spring MVC 中的 WebMvcConfigurer，但用于反应式的 WebFlux 应用程序。通过实现 WebFluxConfigurer 接口，可以覆盖一些默认的配置，定制你的 WebFlux 应用行为，比如路径匹配、消息转换器、CORS 配置等。

*   configureContentTypeResolver 用于配置请求的 Content-Type 解析器，决定如何根据请求中的 Content-Type 头来选择合适的处理器。
*   addCorsMappings 配置跨域资源共享 (CORS) 策略，控制哪些跨域请求被允许。
*   configurePathMatching 配置路径匹配策略，包括是否对路径进行尾部斜杠敏感匹配、路径变量的编码规则等
*   addResourceHandlers 配置静态资源的处理器，比如配置静态文件的存储路径和缓存控制等。
*   configureArgumentResolvers 配置自定义的控制器方法参数解析器。通过这种方式，你可以添加额外的参数解析逻辑，处理不属于默认参数类型的控制器方法参数。
*   configureHttpMessageCodecs 配置 HTTP 消息编解码器（message codecs），控制 WebFlux 如何序列化和反序列化请求和响应。常见的用例是自定义 JSON 序列化或添加额外的消息格式支持
*   addFormatters 用于注册格式化程序和转换器，格式化数据类型或对象，比如将字符串解析为日期格式。
*   configureViewResolvers 用于自定义 ViewResolver，它决定了如何将控制器的返回值解析为视图对象。
*   getWebSocketService

如下所示，为所有RestController注解方法路径添加前缀/api

```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

	@Override
	public void configurePathMatching(PathMatchConfigurer configurer) {
		configurer.addPathPrefix(
				"/api", HandlerTypePredicate.forAnnotation(RestController.class));
	}
}

```

### HttpHandler

在Spring WebFlux中，HttpHandler是一个核心接口，用于处理HTTP请求并生成响应,用于适配Reactor Netty、Undertow、Tomcat、Jetty及任何Servlet容器。Spring WebFlux提供了多个实现类来支持不同的功能和场景。

*   HttpWebHandlerAdapter 通常作为WebFlux应用程序中请求处理的桥梁。
*   FilteringWebHandler 装饰器模式实现，用于在WebHandler处理请求之前和之后添加过滤器逻辑。
*   ExceptionHandlingWebHandler 用于处理WebHandler处理过程中抛出的异常

### Webhandler

比HttpHandler更高层次的通用Web API，处理带有@Controller注解或functional endpoints的请求。

*   DispatcherHandler WebFlux处理请求的核心组件
    
*   FilteringWebhandler 过滤器
    
*   WebHandlerDecorator WebHandler装饰类
    

### WebFilter

过滤器，可以拦截和处理传入的 HTTP 请求和传出的 HTTP 响应，通过实现WebFilter接口实现自定义的过滤器

```java
@Component
public class CustomFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        
        System.out.println("Request received: " + exchange.getRequest().getPath());
        return chain.filter(exchange); 
    }
}

```

在执行DispatcherHandler#handle方法之前，先执行会FilteringWebhandler，FilteringWebhandler控制先执行过滤器，再执行handler

org.springframework.web.server.handler.FilteringWebHandler#

```java
	@Override
	public Mono<Void> handle(ServerWebExchange exchange) {
		return this.chain.filter(exchange);
	}

```

执行默认WebFilterChain org.springframework.web.server.handler.DefaultWebFilterChain#filter

```java
	public Mono<Void> filter(ServerWebExchange exchange) {
		return Mono.defer(() ->
				this.currentFilter != null && this.chain != null ?
						invokeFilter(this.currentFilter, this.chain, exchange) :
						this.handler.handle(exchange));
	}

```

### DispatcherHandler

DispatcherHandler类似于 Spring MVC 中的 DispatcherServlet，但DispatcherHandler是非阻塞的，实现了WebHandler接口，充当前端控制器的角色，它是请求处理的中心节点，所有请求都先经过DispathcerHandler再通过配置委托给其他组件执行。 DispatcherHandler包含3个List bean类型

```java
	@Nullable
	private List<HandlerMapping> handlerMappings;

	@Nullable
	private List<HandlerAdapter> handlerAdapters;

	@Nullable
	private List<HandlerResultHandler> resultHandlers;

```

*   HandlerMapping 基于一些映射的规则，如：带注解的控制器，简单URL模式映射等，将请求映射到处理程序（handler）。主要的实现包括：
    *   RequestMappingHandlerMapping 负责基于注解（如 @RequestMapping、@GetMapping、@PostMapping 等）将请求映射到控制器方法上
    *   RouterFunctionMapping 基于函数式编程模型（RouterFunction 和 HandlerFunction）来处理请求。
    *   SimpleUrlHandlerMapping 用于将静态 URL 或者固定路径直接映射到处理器（Handler）上
*   HandlerAdapter 帮助DispatcherHandler调用映射到请求的处理程序，而不管该处理程序的实际调用方式如何。
*   HandlerResultHandler 处理handler执行结果并完成响应

**DispatcherHandler处理流程**

1.  查找处理器，通过HandlerMapping找到第一个匹配的处理程序（handler）
2.  适配和执行处理器，通过HandlerAdpater执行handler,并将执行的返回值显示为HandlerResult
3.  最后，HandlerResult 会被传递给一个合适的 HandlerResultHandler，该组件负责将处理结果转换为最终的 HTTP 响应，可能是直接输出数据，也可能是通过视图进行渲染。
    *   ResponseEntityResultHandler 通常来自@Controller实例
    *   ServerResponseResultHandler 通常来自 functional endpoints
    *   ResponseBodyResultHandler 处理 @ResponseBody 方法或 @RestController 类中的返回值
    *   ViewResolutionResultHandler 处理返回类型为 View 或其他可以被视图解析器处理的响应结果

org.springframework.web.reactive.DispatcherHandler#handle

```java
	@Override
	public Mono<Void> handle(ServerWebExchange exchange) {
		if (this.handlerMappings == null) {
			return createNotFoundError();
		}
		if (CorsUtils.isPreFlightRequest(exchange.getRequest())) {
			return handlePreFlight(exchange);
		}
		return Flux.fromIterable(this.handlerMappings) 
				.concatMap(mapping -> mapping.getHandler(exchange))
				.next() 
				.switchIfEmpty(createNotFoundError())
				.flatMap(handler -> invokeHandler(exchange, handler)) 
				.flatMap(result -> handleResult(exchange, result)); 
	}

```

### 调用流程

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/cd7df48236ea48f2908f99a173c2b117~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5ZCN5a2X55qE6Zeu6aKY:q75.awebp?rk3s=f64ab15b&x-expires=1728480458&x-signature=YmmKMrpxuxVppnQm%2FaEEsCvvWew%3D)