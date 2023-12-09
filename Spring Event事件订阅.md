# Spring Event事件订阅
Spring Event事件订阅框架，被网上一些人快吹上天了，然而我们在新项目中引入后发现，这个框架缺陷很多，玩玩可以，千万不要再公司项目中使用。还不如自己手写一个监听者设计模式，那样更稳定、可靠。

之前我已经被Spring Event（事件发布订阅组件）坑过一次。那次是在服务关闭期间，有请求未处理完成，当调用Spring Event时，出现异常。

根源是：Spring关闭期间，不得调用GetBean，也就是无法使用Spring Event 。[详情点击这里查看](https://juejin.cn/post/7281159113882468371 "https://juejin.cn/post/7281159113882468371")

然而新项目大量使用了`Spring Event`，在另一个Task服务还未来得及移除`Spring Event`的情况下，出现了类似的问题。

当领导听说新引入的`Spring Event`再次出现问题时，非常愤怒，因为一个月内出现了两次故障。在复盘会议上，差点爆粗口。

在上线过程中，丢消息了？
------------

“五哥，你看一眼钉钉给你发的监控截图，线上好像有丢消息？” 旁边同事急匆匆的跟我说。

“线上有问题？强哥在上线，我让他先暂停下~”，于是我赶紧通知强哥，先暂停发布。优先排查线上问题~

怎么会有问题呢？我有点意外，正好我和强哥各有代码上线，我只改动很小一段代码。我对这次代码变更很自信，坚信不会有问题，所以我并没有慌乱和紧张。搁之前，我的小心脏早就怦怦跳了！

诡异的情况
-----

出现问题的业务逻辑是 消费A 消息，经过业务处理后，再发送B消息。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e1e17860ed84e12913977ac4b1db986~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1206&h=620&s=91040&e=png&b=fdfdfd)
 从线上监控和日志分析，Task服务收到了 A 消息，然后处理失败了。诡异之处是没有任何异常日志和异常打点，仿佛凭空消失了。

分析代码分支后，我和同事十分确信，**任何异常退出的代码分支都有打印异常日志和上报异常监控打点，出现异常不可能不留一丝痕迹。** 

正当陷入困境之时，我们发现蹊跷之处。“丢消息”的时间只有 3秒钟，之后便恢复正常。问题出在启动阶段，消息A进入Task服务，服务还未完全发布完成时，导致不可预测的情况发生。

当分析Spring 源代码以后，我们发现原因出在 Spring Event……

在详细说明问题根源前，我简单介绍一下 SpringEvent使用，熟悉它的读者，可以自行跳过。

Spring Event的简单使用
-----------------

### 声明事件

自定义事件需要继承Spring ApplicationEvent。我选择使用泛型，支持子类可以灵活关联事件的内容。

```BaseEvent定义
public class BaseEvent<T> extends ApplicationEvent {
    private final T data;
    
    public BaseEvent(T source) {
        super(source);
        this.data = source;
    }

    public T getData() {
        return data;
    }
}

```

### 发布事件

使用Spring上下文 ApplicationContext发布事件

```发布事件
applicationContext.publishEvent(new BaseEvent<>(param));

```

Idea为Spring提供了跳转工具，点击绿色按钮位置，就可以 跳转到事件的监听器列表。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1418dea000cd4bbf8ec31b24cb9295fa~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1727&h=152&s=51963&e=png&b=101020)

### 监听事件

监听器只需要 在方法上声明为 EventListener注解，Spring就会自动找到对应的监听器。Spring会根据方法入参的事件类型和 发布的事件类型 自动匹配。

```js
@EventListener
public void handleEvent(BaseEvent<PerformParam> event) {
    
}

```

服务启动阶段，`Spring Event` 注册严重滞后
----------------------------

在Kafka 消费逻辑中，通过Spring Event发布事件，业务逻辑都封装在 Event Listenr 中。经过分析和验证后，我们终于发现问题所在。

当Kafka 消费者已经开始消费消息，但Spring Event 监听者还没有注册到Spring ApplicationContext中， 所以Spring Event 事件发布后，没有Event Listener消费该事件。3秒钟以后，Event Listener被注册到Spring后，异常就消失了。

问题根源在：`Event Listener` 注册的时间点滞后于 `init-method` 的时间点！

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb178aa77c084ca09e3bc39c717cf23e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=516&h=676&s=54432&e=png&b=fdfdfd)

### init-method ——— Kafka 开始监听的时间点

Kafka 消费者的启动点 在 Spring `init-method`中，例如下面的 XML中，init-method 声明 HelloConsumer 的初始化方法为 init方法。在该方法中注册到Kafka中，抢占分片，开始消费消息。

`<bean id="kafkaConsumer" class="com.helloworld.KafkaConsumer" init-method="init" destroy-method="destroy">`

如果在init-method 方法中，成功注册到Kafka，抢占到分片，然而 Spring Event Listener还未注册到Spring ，就会 “**Spring事件丢失**” 的现象。

### EventListener注册到Spring 的时间点

在Spring的启动过程中，`EventListener` 的启动点滞后于 `init-method` 。如下图Spring的启动顺序所示。

其中`init-method`在`InitializingBean`中被触发，而 `EventListener` 在 `SmartInitializingSingleton` 中初始化。由于启动顺序的先后关系，当init-method的执行时间较长时（例如连接超时），就会出现Kafka已开始消费，但`EventListener`还未注册的问题。

**Spring 启动顺序** ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/986480364c894f97a0df6eb296eef1a9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1684&h=1744&s=1091278&e=png&b=fefefe)

#### InitializingBean 的初始化代码

通过分析 Spring源代码。InitializingBean 阶段， invokeInitMethod 会执行init-method方法，Kafka消费者就是在init-method 执行完成后开始消费kafka消息。 ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39512e6ba4c6449aa18728c04d8db09e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2142&h=1108&s=276353&e=png&b=101020)

#### SmartInitializingSingleton

继续分析Spring源代码。 `EventListenerMethodProcessor` 是 `SmartInitializingSingleton` 子类，该类负责解析Spring 中所有的Bean，如果有方法添加`EventListener`注解，则将 `EventListener`方法 注册到 Spring 中

以下是代码截图 ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6c57be2d21a48c4a7bbc98ca4377000~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2436&h=1536&s=511208&e=png&b=101020)

Spring Event很好，我劝你别用
--------------------

通过代码分析可以发现，在Spring中，init-method方法会先执行，然后才会解析和注册Event Listener。因此，在消费Kafka和注册EventListener之间存在一个时间间隔，如果在这期间发布了Spring Event，该事件将无法被消费。

通常情况下，这个时间间隔非常短暂，但是当init-method执行较慢时，比如Kafka消费者 A 初始化很快，但是Kafka消费者 B 建立连接超时导致init-method执行时间较长，就会出现问题。在这段时间内，Kafka消费者 A 发布的Spring事件无法被消费。

尽管这不是一个稳定必现的问题，但是当线上流量较大时，它发生的概率会增加，后果也会更严重。我们在上线3个月后，线上环境才首次遇到这个问题。

在[《服务关闭期，Spring Event消费失败》](https://juejin.cn/post/7281159113882468371 "https://juejin.cn/post/7281159113882468371")这篇文章中，有读者评论提到了这个问题。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e99df7b3b81d426eb50284420eb57add~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1584&h=300&s=81462&e=png&b=ffffff)

> 有朋友说： 这和spring event有什么关系，自己实现一套，不也有同样的问题吗？关键是得优雅停机啊!

他所说的是正确的，如果服务能够完美地进行优雅发布，即使是在大流量场景下，Spring Event也不会出现问题。

然而，要确保服务的优雅发布是非常困难的，尤其是公司的项目，很多框架和组件都是基础架构组提供的，与其费力去改造公司的项目，不如不使用Spring Event。

一般情况下，公司的项目通常会在 init-method 方法中，统一初始化消息队列 MQ 消费者。如果想要安全地使用Spring Event，必须等到Spring完全发布完成之后才能初始化 Kafka 消费者。与其按照Spring Event的要求改造公司的项目，还不如不使用Spring Event。

对于公司的项目来说，稳定性是最重要的。

与其冒着巨大的风险使用Spring Event，不如自己手写事件监听模式，这样反而简易、安全、可靠。