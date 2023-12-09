# SpringEvent
使用Spring Event 优雅的实现观察者模式。这种方式能实现业务逻辑上的解耦，但 Spring Event远远没有那么简单。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52d02d5055a4402887a9f34cd56ecf38~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=3052&h=531&s=461329&e=png&b=f6f6f6)

前几天，线上系统出现两条异常日志Get Bean时找不到对应的bean，调用堆栈让我非常迷惑，为什么Get Bean找不到对应的Bean呢? 堆栈中的信息 解释了原因。

```css
Do not request a bean from a BeanFactory in a destroy method implementation

```

**在应用上下文关闭时，不得从上下文中Get Bean。**  恰好，问题出现在服务关闭期间..... 由于系统流量较高，日订单千万级，即便在低峰期单机的并发度也是比较高的，所以服务在关闭期间有少量流量进来或未处理完。

要明白问题原因和解决办法，需要先简单了解 Spring Event如何使用，总共分为三步。

1.  声明事件
2.  发布事件
3.  监听事件

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

我遇到的线上问题恰恰出现在发布事件后，Spring匹配Listener时，Spring需要从上下文中Get Bean，每发布一次事件需要Get Bean一次。正常情况下没事，在服务关闭期间，如果恰好发布了一个事件，就凉了。

在调研Spring Evnet期间，我重点考虑了使用方式，使用的优缺点，但确实没有想到Spring会有这个缺陷。如何避免这个问题呢？

1.  **弃用SpringEvent**。
2.  允许Spring Event 出现异常，但上层调用捕获异常，上报MQ重试。
3.  彻底服务优雅关闭。Rpc、Http、MQ入口在进程关闭前，先禁用流量。

结合三个方案的改动成本，我选择忍痛弃用Spring Event。

这件事让我明白了，全面调研真的很困难，某些框架确实很好用，但没准在哪个点上就有坑。有时候保守不是一件坏事，至少能保证线上环境能稳定运行啊。