# springboot监听器模式源码精讲
很多时候我们看源码的时候看不下去，其中一个原因是系统往往使用了许多设计模式，如果你不清楚这些设计模式，这无疑增加了你阅读源码的难度。

`springboot`中就大量使用了设计模式，本文主要介绍其中的一种**监听器模式**，这是观察者模式中的一种。

这篇文章主要分为3个部分:

第一个部分主要讲下监听模式的几个步骤以及一个简单的例子，这样我们在阅读源码的时候就知道是怎么回事了。

第二个部分是这篇文章的核心，也就是`springboot`中是如何使用监听模式的。

第三个部分主要是在`spingboot`中自定义实现监听功能

实现监听模式的四个步骤：

*   创建事件
*   创建事件的监听器
*   创建广播器
*   发布事件

我们就按照上面的步骤实现一个监听功能

[](https://link.juejin.cn/?target=)2.1新建工程
------------------------------------------

创建一个`springboot`工程，主要的依赖是这个

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

```

我们的`springboot`版本是2.7.17

[](https://link.juejin.cn/?target=)2.2创建事件
------------------------------------------

先创建一个基础的模板事件

```csharp
package com.example.springbootcode.order.event;

public abstract class OrderEvent {
    abstract void desc();
}

```

这是一个抽象类，里面定义了抽象方法，接下里就创建具体的事件

**订单开始创建事件**

```scala
package com.example.springbootcode.order.event;

public class OrderCreateStartingEvent extends OrderEvent{
    @Override
    public void desc() {
        System.out.print("第一步：开始创建订单，……");
    }
}

```

**订单创建完成事件**

```scala
package com.example.springbootcode.order.event;

public class OrderCreateFinishEvent extends OrderEvent{
    @Override
    public void desc() {
        System.out.print("最后一步：订单创建完成,……");
    }
}

```

这里定义了两个事件，都继承了`OrderEvent`，实现了里面的`desc`方法

[](https://link.juejin.cn/?target=)2.3创建监听器
-------------------------------------------

创建一个监听器接口

```csharp
package com.example.springbootcode.order.event;

public interface OrderListener {
    void onOrderEvent(OrderEvent event);
}

```

定义了一个监听事件的方法，其参数就是事件，接下来我们实现具体的监听器

**订单创建过程监听器**

```csharp
public class OrderLogListener implements OrderListener{
    @Override
    public void onOrderEvent(OrderEvent event) {
        if (event instanceof OrderCreateStartingEvent) {
            event.desc();
        } else if (event instanceof OrderCreateFinishEvent){
            event.desc();
        }
    }
}

```

监听事件，并调用事件里面的`desc()`方法

[](https://link.juejin.cn/?target=)2.4创建广播器
-------------------------------------------

定义事件广播器接口

```csharp
package com.example.springbootcode.order.event;

public interface EventMulticaster {
    void multicasterEvent(OrderEvent event);

    void addListener(OrderListener listener);

    void removeListener(OrderListener listener);
}

```

广播器我们可以这样理解：**主要复杂监听器的管理，并将事件广播给所有监听器，如果该监听器对广播的事件感兴趣，那么就会处理事件**。

接下来实现一下具体的广播器

```java
package com.example.springbootcode.order.event;


import java.util.ArrayList;
import java.util.List;

public class OrderEventMulticaster implements EventMulticaster{
    List<OrderListener> listenerList = new ArrayList<>();

    @Override
    public void multicasterEvent(OrderEvent event) {
        listenerList.forEach(listener -> listener.onOrderEvent(event));
    }

    @Override
    public void addListener(OrderListener listener) {
        listenerList.add(listener);
    }

    @Override
    public void removeListener(OrderListener listener) {
        listenerList.remove(listener);
    }
}

```

这里主要实现了3个方法，事件广播、新增监听器、删除监听器

[](https://link.juejin.cn/?target=)2.5发布事件
------------------------------------------

发布事件，其实说白了就是在项目中调用`OrderEventMulticaster`

```typescript
@SpringBootApplication
public class SpringbootCodeApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootCodeApplication.class, args);

        OrderEventMulticaster orderEventMulticaster = new OrderEventMulticaster();
        orderEventMulticaster.addListener(new OrderLogListener());
        orderEventMulticaster.multicasterEvent(new OrderCreateStartingEvent());
    }
}

```

加了两个监听器，然后我们发布了一个**订单开始创建事件**，控制台会打印"第一步：开始创建订单，……"

到这里为止，一个简单的监听模式就实现了，接下来我们根据这个例子去阅读一下`springboot`源码中关于监听器的部分。

由于`springboot`有太多地方使用了监听模式，我们不可能去阅读所有代码，这也不是我写这篇文章的目的。我们只需了解其中的一个事件、一个监听器就知道`springboot`大概是怎么实现事件的监听和发布。

下面我们以应用程序启动为例，看看其是怎么实现事件的监听和发布。

[](https://link.juejin.cn/?target=)3.1启动事件
------------------------------------------

在讲启动事件之前，我们先来看看`springboot`跟生命周期相关的事件脉络  
![](_assets/f0bfa89e0f184b80bafa28ba16379df6~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

由图可以发现，启动程序开始到完成会发布`ApplicationStartingEvent`、`ApplicationPreparedEvent`、`ApplicationReadyEvent`、`ApplicationStartedEvent`……这些事件。

其实由名字大概就知道它们的意思，这里我们以 `ApplicationStartingEvent` 这个开始启动事件为例，去看了解其实现过程。

**EventObject**

是所有事件状态对象的根类， 这是 `java.util` 包中的 `EventObject` 类的源代码，里面有个方法 `getSource() `用于返回事件最初发生的对象。

**ApplicationEvent**

提供了一个基本的事件模型，为应用程序中的事件提供了一种通用的表示方式 。

**SpringApplicationEvent**

提供了一个 与 `Spring Boot` 应用程序的启动过程相关的事件通用的基类，这里就是类似于我们前面例子中的`OrderEvent`事件

```scala
public abstract class SpringApplicationEvent extends ApplicationEvent {

	private final String[] args;

	public SpringApplicationEvent(SpringApplication application, String[] args) {
		super(application);
		this.args = args;
	}

	public SpringApplication getSpringApplication() {
		return (SpringApplication) getSource();
	}

	public final String[] getArgs() {
		return this.args;
	}
}

```

总得来说就是获取事件的源和事件相关的参数

**ApplicationStartingEvent**

这个就是启动事件了，类似于我们例子中的`OrderCreateStartingEvent`，我们看看这个启动事件的代码

```scala
public class ApplicationStartingEvent extends SpringApplicationEvent {

	private final ConfigurableBootstrapContext bootstrapContext;

	public ApplicationStartingEvent(ConfigurableBootstrapContext bootstrapContext, SpringApplication application,
			String[] args) {
		super(application, args);   
		this.bootstrapContext = bootstrapContext; 
	}

	public ConfigurableBootstrapContext getBootstrapContext() {
		return this.bootstrapContext;
	}

}

```

它干了下面这些事：

**Ⅰ：**   调用父类的构造函数，将 `application` 设置为事件的源，将参数数组 `args` 设置为事件的相关参数

**Ⅱ：**初始化一个上下文接口，并通过`getBootstrapContext()`来获取该对象

总得来说这个事件是比较简单的。

[](https://link.juejin.cn/?target=)3.2监听器
-----------------------------------------

还是一样，我们先上图，先了解一下关于启动程序事件监听器的实现脉络

![](_assets/58fa975244ef453a89eecc156ad80309~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

**EventListener**

这是一个标记接口 ，在 Java 编程中常用于为一组类提供一个共同的标记，而不包含任何方法。标记接口通常用于指示类具有某种特定的性质、行为或能力。

**ApplicationListener**

这是 Spring Framework 中的 `ApplicationListener` 接口，用于定义应用程序事件监听器，这里就类似于上面例子中的`OrderListener`

```csharp
@FunctionalInterface 
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
	
	void onApplicationEvent(E event); 

	static <T> ApplicationListener<PayloadApplicationEvent<T>> forPayload(Consumer<T> consumer) {
		return event -> consumer.accept(event.getPayload());
	}

}

```

**Ⅰ：**   `@FunctionalInterface` 是 Java 注解，用于标记一个接口，指示它是一个函数式接口。函数式接口是只包含一个抽象方法的接口，通常用于支持 Lambda 表达式和函数引用。

**Ⅱ：**  `onApplicationEvent(E event)`处理事件，其参数是一个泛型，但必须是继承`ApplicationEvent`

**SmartApplicationListener**和**GenericApplicationListener**主要是为了扩展**ApplicationListener**，这里就不深入讲解了。`springboot`这样设计的目的其实就是后续的扩展，而不是把所有东西都写在**ApplicationListener**，基本上所有的系统都是这么个原则。

**LoggingApplicationListener**

最后我们的主角登场了，从名字大概可以猜到这是一个跟启动日志记录监听器，它会在启动过程的某个阶段记录日志，下面是核心代码

```scss
public class LoggingApplicationListener implements GenericApplicationListener {
    
    
	@Override
    public void onApplicationEvent(ApplicationEvent event) {
        if (event instanceof ApplicationStartingEvent) {
            onApplicationStartingEvent((ApplicationStartingEvent) event);
        }
        else if (event instanceof ApplicationEnvironmentPreparedEvent) {
            onApplicationEnvironmentPreparedEvent((ApplicationEnvironmentPreparedEvent) event);
        }
        else if (event instanceof ApplicationPreparedEvent) {
            onApplicationPreparedEvent((ApplicationPreparedEvent) event);
        }
        else if (event instanceof ContextClosedEvent) {
            onContextClosedEvent((ContextClosedEvent) event);
        }
        else if (event instanceof ApplicationFailedEvent) {
            onApplicationFailedEvent();
        }
    }
    
}

```

从代码中可以看到它是监听启动过程的所有事件，其中第一个就是我们上面讲到的`ApplicationStartingEvent`开始启动事件，这里类似于我们上面例子中的`OrderLogListener`

[](https://link.juejin.cn/?target=)3.3广播器
-----------------------------------------

跟上面一样，我们还是先上图

![](_assets/10b454ff2546442aa52ff36e6225806e~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

**ApplicationEventMulticaster**

```java
package org.springframework.context.event;

import java.util.function.Predicate;

import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.core.ResolvableType;
import org.springframework.lang.Nullable;

public interface ApplicationEventMulticaster {

	void addApplicationListener(ApplicationListener<?> listener);

	void addApplicationListenerBean(String listenerBeanName);

	void removeApplicationListener(ApplicationListener<?> listener);

	void removeApplicationListenerBean(String listenerBeanName);

	void removeApplicationListeners(Predicate<ApplicationListener<?>> predicate);

	void removeApplicationListenerBeans(Predicate<String> predicate);

	void removeAllListeners();

	void multicastEvent(ApplicationEvent event);

	void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType);

}

```

定义事件广播器接口，类似于前面例子中的`EventMulticaster`

**AbstractApplicationEventMulticaster**

```java
public abstract class AbstractApplicationEventMulticaster
		implements ApplicationEventMulticaster, BeanClassLoaderAware, BeanFactoryAware {
    
    
    
    @Override
	public void addApplicationListener(ApplicationListener<?> listener) {
		synchronized (this.defaultRetriever) {
			
			
			Object singletonTarget = AopProxyUtils.getSingletonTarget(listener);
			if (singletonTarget instanceof ApplicationListener) {
				this.defaultRetriever.applicationListeners.remove(singletonTarget);
			}
			this.defaultRetriever.applicationListeners.add(listener);
			this.retrieverCache.clear();
		}
	}
    
    
    
    @Override
	public void removeApplicationListener(ApplicationListener<?> listener) {
		synchronized (this.defaultRetriever) {
			this.defaultRetriever.applicationListeners.remove(listener);
			this.retrieverCache.clear();
		}
	}
    
}

```

这是一个抽象类，实现了监听器的新增和删除的具体逻辑，这样做的目的就是封装一些通用的功能，至于说要不要加一个这样的抽象类完全就是根据你的实际业务情况而定。

**SimpleApplicationEventMulticaster**

```java
public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster {
    
    
    
    @Override
	public void multicastEvent(ApplicationEvent event) {
		multicastEvent(event, resolveDefaultEventType(event));
	}

	@Override
	public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
		Executor executor = getTaskExecutor();
		for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			if (executor != null) {
				executor.execute(() -> invokeListener(listener, event));
			}
			else {
				invokeListener(listener, event);
			}
		}
	}
    
    
    
    private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
		try {
			listener.onApplicationEvent(event); 
		}
		catch (ClassCastException ex) {
			String msg = ex.getMessage();
			if (msg == null || matchesClassCastMessage(msg, event.getClass()) ||
					(event instanceof PayloadApplicationEvent &&
							matchesClassCastMessage(msg, ((PayloadApplicationEvent) event).getPayload().getClass()))) {
				
				
				Log loggerToUse = this.lazyLogger;
				if (loggerToUse == null) {
					loggerToUse = LogFactory.getLog(getClass());
					this.lazyLogger = loggerToUse;
				}
				if (loggerToUse.isTraceEnabled()) {
					loggerToUse.trace("Non-matching event type for listener: " + listener, ex);
				}
			}
			else {
				throw ex;
			}
		}
	}
}

```

这个类继承`AbstractApplicationEventMulticaster`，并且实现了`multicastEvent()`广播事件，这类似于例子中的`OrderEventMulticaster`

[](https://link.juejin.cn/?target=)3.4发布事件
------------------------------------------

前面所有东西都准备好了，看看程序启动时是如何监听事件的。我们从启动类开始跟跟踪

```typescript
@SpringBootApplication
public class SpringbootCodeApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootCodeApplication.class, args);
    }
}

```

进入run方法

```typescript
public class SpringApplication {
    
    
    
    public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
		return run(new Class<?>[] { primarySource }, args);
	}

	public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
		
        return new SpringApplication(primarySources).run(args); 
	}
    
    
    public SpringApplication(Class<?>... primarySources) {
		this(null, primarySources);
	}
	
    
	@SuppressWarnings({ "unchecked", "rawtypes" })
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
		this.bootstrapRegistryInitializers = new ArrayList<>(
				getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
        
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
	}
    
    
    public void setListeners(Collection<? extends ApplicationListener<?>> listeners) {
		this.listeners = new ArrayList<>(listeners);
	}
    
}

```

**Ⅰ**：我们先进入`SpringApplication`构造函数，最后我们发现会来到**Ⅰ->Ⅲ->Ⅳ->Ⅴ**，`this.listeners`存储了启动时需要加载的监听器。

我们看看`this.listeners`存储了哪些监听器

![](_assets/d9f315425ca44a5db866a30767b4998d~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

我们可以发现**LoggingApplicationListener**监听器也保存在里面，接下来就是怎么把这些监听器加载到广播器中

> 构造函数实际上时初始化一些变量

还记得上面**Ⅱ：**run这个注释吗，构造函数执行完后，开始执行run方法，我们进入run方法

```ini
public ConfigurableApplicationContext run(String... args) {
    long startTime = System.nanoTime()
    DefaultBootstrapContext bootstrapContext = createBootstrapContext()
    ConfigurableApplicationContext context = null
    configureHeadlessProperty()
    SpringApplicationRunListeners listeners = getRunListeners(args)
    // Ⅵ
    listeners.starting(bootstrapContext, this.mainApplicationClass)
 	// 省略代码
    
    return context
}

```

**Ⅵ：**这里就是发布一个启动开始事件了，进入`starting`方法

```scss
package org.springframework.boot;
class SpringApplicationRunListeners {

	
	void starting(ConfigurableBootstrapContext bootstrapContext, Class<?> mainApplicationClass) {
		doWithListeners("spring.boot.application.starting", (listener) -> listener.starting(bootstrapContext),
				(step) -> {
					if (mainApplicationClass != null) {
						step.tag("mainApplicationClass", mainApplicationClass.getName());
					}
				});
	}
	

}

```

这里都是发布启动相关的事件，`starting`里面调用了`doWithListeners`方法，其中最主要的是第二个参数，是一个表达式，我们进去看看，最后我们来到这里

```kotlin
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {
	
    
    
	public EventPublishingRunListener(SpringApplication application, String[] args) {
		this.application = application;
		this.args = args;
        
		this.initialMulticaster = new SimpleApplicationEventMulticaster();
		for (ApplicationListener<?> listener : application.getListeners()) {
            
			this.initialMulticaster.addApplicationListener(listener);
		}
	}
	
    
	@Override
	public void starting(ConfigurableBootstrapContext bootstrapContext) {

		this.initialMulticaster
            
			.multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args));
	}

    
}

```

这里的步骤就像我们上面例子中的一样

```typescript
@SpringBootApplication
public class SpringbootCodeApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootCodeApplication.class, args);

        OrderEventMulticaster orderEventMulticaster = new OrderEventMulticaster();
        orderEventMulticaster.addListener(new OrderLogListener());
        orderEventMulticaster.multicasterEvent(new OrderCreateStartingEvent());
    }
}

```

new一个广播器，然后添加监听器，最后是广播事件。

了解了`spingboot`的监听模式后，接下来我们看看如何在`springboot`项目中使用它。

[](https://link.juejin.cn/?target=)4.1创建事件
------------------------------------------

```scala
package com.example.springbootcode.my;

import org.springframework.context.ApplicationEvent;

public class UserMsgEvent extends ApplicationEvent {

    public UserMsgEvent(Object source) {
        super(source);
    }
	
    
    public void sendMsg(){
        System.out.print("向用户发送消息");
    }
}

```

这里需要注意的是我们继承的是`ApplicationEvent`，不继承**SpringApplicationEvent**主要是因此其跟启动相关的，我们定义的事件明显跟应用启动没啥关系。

[](https://link.juejin.cn/?target=)4.2创建监听器
-------------------------------------------

```java
package com.example.springbootcode.my;

import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

@Component
public class UserListener implements ApplicationListener<UserMsgEvent> {

    @Override
    public void onApplicationEvent(UserMsgEvent event) {
        event.sendMsg();
    }
}

```

这里继承`ApplicationListener`

[](https://link.juejin.cn/?target=)4.3自定义触发事件
---------------------------------------------

```java
package com.example.springbootcode.my;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    private final ApplicationEventPublisher eventPublisher;

    public UserService(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    public void send(){
        eventPublisher.publishEvent(new UserMsgEvent(this));
    }
}

```

通过构造函数注入 `ApplicationEventPublisher `， 这是一个Spring框架提供的接口，用于发布事件

最后在需要调用的地方调用`UserService`里面的`send`方法即可