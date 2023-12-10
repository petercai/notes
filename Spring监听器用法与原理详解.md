# Spring监听器用法与原理详解
我们就来深入学习一下**Spring监听器**

* * *

Spring监听器是一种==特殊的类，它们能帮助开发者监听 web 中特定的事件==，比如 ServletContext, HttpSession, ServletRequest 的创建和销毁；变量的创建、销毁等等。

**当Web容器启动后，Spring的监听器会启动监听**，监听是否创建ServletContext的对象，如果发生了创建ServletContext对象这个事件 (当web容器启动后一定会生成一个ServletContext对象，所以监听事件一定会发生)，ContextLoaderListener类会实例化并且执行初始化方法，将spring的配置文件中配置的bean注册到Spring容器中

1\. 模型介绍
--------

**观察者模式**（Observer Pattern）是一种行为设计模式，它用于在对象之间建立一对多的依赖关系。在该模式中，**当一个对象的状态发生变化时，它会自动通知其依赖对象（称为观察者），使它们能够自动更新**

观察者模式的工作原理如下：

1.  主题对象维护一个观察者列表，并**提供方法用于添加和删除观察者**。
2.  当主题的状态发生变化时，它会**遍历观察者列表，并调用每个观察者的通知方法**。
3.  观察者接收到通知后，根据通知进行相应的更新操作。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec96aa44328147faab54acbf717015d0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=665&h=420&s=56527&e=png&b=fdfdfd)

2\. 观察者模式Demo
-------------

一个观察者模式demo包括以下部分：

*   **观察者实体**
*   **主题实体**

所以我们先写个观察者接口：

```java
import java.util.ArrayList;
import java.util.List;

interface Observer {
    void update();
}

```

再构建两个观察者实现类

```java

class ConcreteObserverA implements Observer {
    @Override
    public void update() {
        System.out.println("ConcreteObserverA收到更新通知");
    }
}


class ConcreteObserverB implements Observer {
    @Override
    public void update() {
        System.out.println("ConcreteObserverB收到更新通知");
    }
}

```

然后定义主题接口

```java
interface Subject {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}

```

构建一个具体主题

```java

class ConcreteSubject implements Subject {
    private List<Observer> observers = new ArrayList<>();

    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update();
        }
    }

    public void doSomething() {
        System.out.println("主题执行某些操作...");
        notifyObservers(); 
    }
}

```

测试代码

```java
public class ObserverPatternDemo {
    public static void main(String[] args) {
        
        ConcreteSubject subject = new ConcreteSubject();
        Observer observerA = new ConcreteObserverA();
        Observer observerB = new ConcreteObserverB();

        
        subject.registerObserver(observerA);
        subject.registerObserver(observerB);

        
        subject.doSomething();
    }
}


```

最后可以看到结果

> 主题执行某些操作... ConcreteObserverA收到更新通知 ConcreteObserverB收到更新通知

* * *

1\. 新建监听器
---------

### 1.1 实现ApplicationListener接口

在详细介绍Spring监听器前，我们以一个简单的demo来说明：

```java
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.stereotype.Component;

@Component
public class MyContextRefreshedListener implements ApplicationListener<ContextRefreshedEvent> {

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println("应用程序上下文已刷新");
        
    }
}


```

1.  我们创建了一个名为**MyContextRefreshedListener**\* 的监听器，它实现了**ApplicationListener**接口，这意味着这个监听器监听的事件类型是**ContextRefreshedEvent**，并重写了 **onApplicationEvent** 方法。该方法在应用程序上下文被刷新时触发。
    
2.  使用 **@Component** 注解将该监听器声明为一个Spring管理的组件，这样Spring会自动将其纳入到应用程序上下文中，并在适当的时候触发监听
    

### 1.2 使用@EventListener注解

除了手动写个类外，我们也可以找个现成的类，**该类不需要继承或实现任何其他类**，然后在它的某个方法上加上 _@EventListener_ 注解，如下：

```java
@Component
public class MyListener {

	@EventListener(ContextRefreshedEvent.class)
	public void methodA(ContextRefreshedEvent event) {
 		System.out.println("应用程序上下文已刷新");
        
	}
}


```

1.  在一个现有的类的某个方法上，加上@EventListener(ContextRefreshedEvent.class)，**Spring会在加载这个类时，为其创建一个监听器**，这个监听器监听的事件类型是**ContextRefreshedEvent**，**当此事件发生时，将触发执行该方法methodA**。
    
2.  使用 **@Component** 注解将该类声明为一个Spring管理的组件，这样Spring会自动将其纳入到应用程序上下文中，并在适当的时候触发监听
    
3.  **我们可以在这个类中写上多个方法，每个方法通过注解监听着不同的事件类型，这样我们就仅需使用一个类，却构建了多个监听器**
    

**上述两种方法的效果是一样的。那么最后，我们就完成了Spring中一个内置监听器的简单示例**：当启动一个基于Spring的应用程序时，当应用程序上下文被刷新时，ContextRefreshedEvent事件将被触发，然后MyContextRefreshedListener监听器的onApplicationEvent方法将被调用。**

2\. 内置的事件类型
-----------

我们在demo中使用了一个 **ContextRefreshedEvent** 的事件，这个事件是Spring内置的事件，除了该事件，Spring还内置了一些其他的事件类型，分别在以下情况下触发：

1.  **ContextRefreshedEvent**： 当应用程序==上下文被刷新==时触发。这个事件在ApplicationContext初始化或刷新时被发布，适用于执行初始化操作和启动后的后续处理。例如，初始化缓存、预加载数据等。
    
2.  **ContextStartedEvent**： 当应用程序==上下文启动==时触发。这个事件在调用ApplicationContext的start()方法时被发布，适用于在应用程序启动时执行特定的操作。例如，启动定时任务、启动异步消息处理等。
    
3.  **ContextStoppedEvent**： 当应用程序==上下文停止==时触发。这个事件在调用ApplicationContext的stop()方法时被发布，适用于在应用程序停止时执行清理操作。例如，停止定时任务、关闭数据库连接等。
    
4.  **ContextClosedEvent**： 当应用程序==上下文关闭==时触发。这个事件在调用ApplicationContext的close()方法时被发布，适用于在应用程序关闭前执行最后的清理工作。例如，释放资源、保存日志等。
    
5.  **RequestHandledEvent**： 在Web应用程序中，当一个==HTTP请求处理完成==后触发。这个事件在Spring的DispatcherServlet处理完请求后被发布，适用于记录请求日志、处理统计数据等。
    
6.  **ApplicationEvent**： **这是一个抽象的基类，可以用于定义自定义的应用程序事件。你可以创建自定义事件类，继承自ApplicationEvent，并定义适合你的应用场景的事件类型。** 
    

3\. 自定义事件与监听器Demo
-----------------

在学习完上面的内容后，我们现在可以手动写个Spring的事件，以及对应的监听器的demo了

### 3.1 构建两个自定义事件

建立继承自ApplicationEvent的自定义事件类

```java

public class CustomEventA extends ApplicationEvent {
    private String message;

    public CustomEventA(Object source, String message) {
        super(source);
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

public class CustomEventB extends ApplicationEvent {
    private String message;

    public CustomEventB(Object source, String message) {
        super(source);
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

```

### 3.2 构建监听

我们选用@EventListener注解来实现监听

```java
@Component
public class MyListener {

	@EventListener(CustomEventA.class)
	public void methodA(CustomEventA event) {
 		System.out.println("========我监听到事件A了:" + event.getMessage());
        
	}

	@EventListener(CustomEventB.class)
	public void methodB(CustomEventB event) {
 		System.out.println("========我监听到事件B了:" + event.getMessage());
        
	}
}

```

### 3.3 发布事件

```java
@Component
public class CustomEventPublisher implements ApplicationContextAware, ApplicationListener<ContextRefreshedEvent> {
    
    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
    
    
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        CustomEventA eventA = new CustomEventA(applicationContext , "我是AAAA");
        CustomEventB eventB = new CustomEventB(applicationContext , "我是BBBB");
        applicationContext.publishEvent(eventA);
        applicationContext.publishEvent(eventB);
    }
}


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67da831d3a354e529f53d673de629bac~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=315&h=97&s=19433&e=png&b=2b2b2b)

1\. Spring监听器模型
---------------

前面我们讲了观察者模式的模型，它的模型主要是由 **观察者实体** 和 **主题实体** 构成，而Spring的监听器模式则结合了Spring本身的特征，也就是容器化。**在Spring中，监听器实体全部放在ApplicationContext中，事件也是通过ApplicationContext来进行发布**，具体模型如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0925051f1734903918bdfdeb1e75c18~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=894&h=645&s=93815&e=png&b=fdfdfd)
 我们不难看到，虽说是通过**ApplicationContext**发布的事件，但其并不是自己进行事件的发布，而是引入了一个处理器—— ==**EventMulticaster**==，直译就是事件多播器，它负责在大量的监听器中，针对每一个要广播的事件，找到事件对应的监听器，然后调用该监听器的响应方法，图中就是调用了监听器1、3、6。

**PS: 只有在某类事件第一次广播时，EventMulticaster才会去做遍历所有监听器的事，当它针对该类事件广播过一次后，就会把对应监听器保存起来了，最后会形成一个缓存Map，下一次就能直接找到这些监听器**

```java
final Map<ListenerCacheKey, ListenerRetriever> retrieverCache = new ConcurrentHashMap<>(64);

```

2\. @EventListener原理
--------------------

直接实现监听器接口，然后注册成Bean，这种方式比较好理解，因为我们自己写的实现类就是监听器。但是使用 **@EventListener** 时，监听器又是怎么产生呢？我们以上面的【**自定义事件与监听器Demo**】为例，来看一下关键代码。

我们知道，在生成流程中，会对每个Bean都使用PostProcessor来进行加工，而其中就有这么一个类_**EventListenerMethodProcessor**_，**这个类会在Bean实例化后进行一系列操作** （PS: 首先，不了解Bean生成过程的同学，可以先去看看另一篇文章：[SpringBean生成流程详解](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fu011709538%2Farticle%2Fdetails%2F129303025 "https://blog.csdn.net/u011709538/article/details/129303025") ）

```java
private void processBean(final String beanName, final Class<?> targetType) {
          ......省略前面代码
	
	ConfigurableApplicationContext context = this.applicationContext;
	Assert.state(context != null, "No ApplicationContext set");
	List<EventListenerFactory> factories = this.eventListenerFactories;
	Assert.state(factories != null, "EventListenerFactory List not initialized");
	
	for (Method method : annotatedMethods.keySet()) {
	    
		for (EventListenerFactory factory : factories) {
		    
			if (factory.supportsMethod(method)) {
				Method methodToUse = AopUtils.selectInvocableMethod(method, context.getType(beanName));
				
				ApplicationListener<?> applicationListener =
						factory.createApplicationListener(beanName, targetType, methodToUse);
				if (applicationListener instanceof ApplicationListenerMethodAdapter) {
					((ApplicationListenerMethodAdapter) applicationListener).init(context, this.evaluator);
				}
				
				context.addApplicationListener(applicationListener);
				break;
			}
		}
	}
	......省略后面代码
}

```

如上，遍历Bean每个带@EventListener注解的方法，然后利用_**DefaultEventListenerFactory**_开始创建监听器，实际上这些监听器类型都是一个适配器类——_**ApplicationListenerMethodAdapter**_，只是因为这些监听器具体的参数不一样，所以可以监听不同的事件，做不同的响应

```java
public class DefaultEventListenerFactory implements EventListenerFactory, Ordered {
	
	@Override
	public ApplicationListener<?> createApplicationListener(String beanName, Class<?> type, Method method) {
		
		return new ApplicationListenerMethodAdapter(beanName, type, method);
	}

}

```

最后效果如图，成功的创建了两个监听器 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9cbfd001efd7468e8ce879d54664e26a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=945&h=199&s=59817&e=png&b=3c3f41)

3\. @EventListener错误尝试
----------------------

知道了@EventListener的原理，我们其实可以做一些猜测，如下: **methodA**是正常的用法; **methodB**方法的修饰符是private; **methodC**则是监听的ContextRefreshedEvent，但下面方法的入参却是ContextClosedEvent; ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d5ba66f9a6d49a2affcfda4346cf57e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=651&h=349&s=67886&e=png&b=2c2c2c)
 后两者都有问题： 可以看到，编译器直接黄底提示了methodB的@EventListener注解，其实从前面我们已经猜到，因为最后我们的调用是由监听器ApplicationListenerMethodAdapter对象直接调用的方法ABC，所以方法必须可被其他对象调用，即**public** ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5fa25fca8764934800859b52c53453e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=540&h=133&s=12497&e=png&b=2e2e2e)
 而后者会在执行广播响应事件时报参数非法异常也是意料之中。

通过模型，我们不难看出，事件的发布其实由业务线程来发起，那么哪些监听器的触发呢，==是仍由业务线程一个个同步地去通知监听器，还是有专门的线程接手，收到事件后，再转手通知监听器们？==

1\. 默认同步通知
----------

其实，==因为spring默认的多播器没有设置执行器，所以默认采用的是第一种情况，即哪个线程发起的事件，则由哪个线程去通知监听器们==，关键代码如下所示

```java
	
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
	
	@Nullable
	protected Executor getTaskExecutor() {
		return this.taskExecutor;
	}

```

我们可以看到，对于每个监听器的调用是同步还是异步，取决于多播器内部是否含有一个执行器，如果有则交给执行器去执行，如果没有，只能让来源线程一一去通知了。

2\. 异步通知设置
----------

两种方式，一种是在多播器创建时内置一个线程池，使多播器能够调用自身的线程池去执行事件传播。另一种是不再多播器上做文章，而是在每个监视器的响应方法上标注异步@Async，毫无疑问，第一种才是正道，我们来看看如何做到。其实有多种方法，我们这里说两种。

第一种，直接自定义一个多播器，然后顶替掉Spring自动创建的多播器

```java
@Configuration
public class EventConfig {
    @Bean("taskExecutor")
    public Executor getExecutor() {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(10,
                15,
                60,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue(2000));
        return executor;
    }
    
    
    
    @Bean(AbstractApplicationContext.APPLICATION_EVENT_MULTICASTER_BEAN_NAME)
    public ApplicationEventMulticaster initEventMulticaster(@Qualifier("taskExecutor") Executor taskExecutor) {
        SimpleApplicationEventMulticaster simpleApplicationEventMulticaster = new SimpleApplicationEventMulticaster();
        simpleApplicationEventMulticaster.setTaskExecutor(taskExecutor);
        return simpleApplicationEventMulticaster;
    }
}

```

第二种，为现成的多播器设置设置一个线程池

```java
@Component
public class WindowsCheck implements ApplicationContextAware {
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SimpleApplicationEventMulticaster caster = (SimpleApplicationEventMulticaster)applicationContext
                .getBean(AbstractApplicationContext.APPLICATION_EVENT_MULTICASTER_BEAN_NAME);
        ThreadPoolExecutor executor = new ThreadPoolExecutor(10,
                15,
                60,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue(2000));
        caster.setTaskExecutor(executor);
    }
}

```

当然，这里推荐第一种，第二种方法会在spring启动初期的一些事件上，仍采用同步的方式。直至被注入一个线程池后，其才能使用线程池来响应事件。而第一种方法则是官方暴露的位置，让我们去构建自己的多播器。

我们可以看到，一个Spring监听器内容其实并不少，而且用到了**观察者模式**，**工厂模式(EventListenerFactory)**，**适配器模式(ApplicationListenerMethodAdapter)**。除了这些设计模式，还需要对Spring的基础有些了解，比如**Bean生成过程(PostProcessor)**，不过，相信你看完了本篇，已经对Spring监听器用法及原理已经有了相当的理解了，只需要在后续开发实践中，注意相互印证即可。