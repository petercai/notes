# 对Spring基本概念的一些理解 
1.  Spring如何解决循环依赖。

感觉可以用 "鲁迅" 形容。 鲁就是慢的，而讯就是快的。

循环依赖

1.  如果想解决setter的循环依赖，setter表示 构造完了再注入，不是那么急切，我们就叫"鲁"，为了避免"鲁"导致的循环依赖，我们的解决方法是 "讯"。

具体实现就是三级缓存: 在对象还没完全创建好时，就**提前**将其以ObjectFactory的方式加入三级缓存。于是如果A初始化B，B可以在三级缓存中，找到A的ObjectFactory对象，进行注入。讯体现在，尽管对象处在一个未创建完毕的状态，但我们提前暴露出能访问对象的对象工厂ObjectFactory.

2.  如果想解决构造器的循环依赖，构造器肯定是急切的注入，所以我们叫"讯"，为了解决讯产生的循环依赖，解决方法是"鲁"

我们对依赖的内容进行@Lazy注解，该注解实际上会导致此对象变成一个Proxy类，尽管构造A时，需要B也构造完毕，构造B时，需要A也构造完毕，但此时A和B都是代理对象，也就是ProxyA，ProxyB，注入的也是代理的对象，并不冲突。

2.  生命周期的理解

> Spring起的名字很有特色

1.  怎么理解xxxAware

如果不加以控制，对象的生命周期将**全权**交给Spring容器进行，这个就叫IoC。一个对象到最后能作为一个 可以通过@Autowired注入的实例，经过以下步骤

1.1 对象从配置里(xml, @Bean)被解析为BeanDefinitionHolder对象。

1.2 BeanDefinition被注册到容器中

1.3 从BeanDefinition(模具), 得到Bean实例。这一步通过BeanFactory完成。

所谓生命周期，就是Spring**在容器管理的这一套流程里，加入了很多的hook点。** ，这些hook点可以由用户自行实现功能。

hook点的位置在

1.  获取到beanFactory之后

如果想定制化某个Bean的生命周期，可以实现接口InstantiationAwareBeanPostProcessor

> 这个名字是不是很形象, InstantiationAware 表示 "感知实例化"，什么叫感知? 感知什么?
> 
> 这里的Aware指的是， Bean的使用方(开发人员) 具备**感知Spring对这个Bean的实例化流程**。 原本我们是不感知的，应该是开箱即用的，但有时候我们要对开箱即用的东西进行简单处理，Spring通过这样的逻辑，可以让我们在指定的hook点处理一些逻辑
> 
> 现在我们已经介入到容器对Bean的创建流程了，但是我们得寸进尺，希望得到容器实例对象的工具。这好比老师傅来装修，你跟老师傅说，你装修的不满意，把你的刷子给我，我自己刷。
> 
> Spring会根据对象实现了 什么xxxAware，这里xxx就是 我们希望从老师傅那里接管的刷子，比如我们想获取这个对象的name，就实现BeanNameAware，想获取加载这个对象的ClassLoader，就实现ClassLoaderAware，注意，这些内容都是容器持有的，需要让容器交给我们，**容器会直接注入到Bean的实例中**，这意味着，实现了InstantiationAwareBeanPostProcessor的 bean，可以提供构造器或者setter，让容器把它的刷子注入进来。

InstantiationAwareBeanPostProcessor这个接口表示: 我允许你干预我对Bean 的实例化过程，那么提供了哪些hook点呢 ?

2.  beforeInstantiation: 在Bean的构造函数(容器通过反射获取)被调用前。
3.  afterInstantition: 在Bean的构造函数(容器通过反射获取)被调用后。
4.  postProcessProperties: 这个hook点用于让我们修改bean的属性

**当Bean被实例化后，容器才会注入相关的Aware**，然后就是BeanPostProcessor接口提供的几个hook点

5.  beforeInitialization: 在Bean的初始化函数(@Init, 或者init-method属性指定)调用前调用。
6.  afterInitialization: 在Bean的初始化函数(@Init, 或者init-method属性指定)调用后调用。

注意，这里一个是实例化(Instantiation), 表示Bean被反射创建的过程，一个是初始化(Initialization), 表示bean的初始化方法调用的过程。

3.  @Autowired啥时候被处理。

只要是Bean，无论是@Bean注入的，xml配置的，@Component被容器反射扫出来的，都是Bean，只要是Bean，其模具就是BeanDefinition

换句话说，Bean的生命周期就是照着BeanDefinition来装修，如果觉得容器修的不满意，容器说: 我把刷子给你，你来刷。

但是@Autowired 肯定是容器来处理的，处理注解我们应该没啥需要介入的。

首先理解一点: Spring提供的大量注解，充其量就是一个**标记(tag)**，并不具备什么实际意义。单纯就是让Spring知道，这个方法/字段/类，被这个注解修饰到了。

Spring是咋知道的呢? Spring对组件扫描的时候，是通过反射扫的，既然是反射，就全方位细粒度的把整个组件类一览无余的扫一遍，包括它的类上有哪些注解，它的每个方法都有啥注解，每个方法的参数上都有啥注解，这些都是反射的。反射后存哪里了呢? 别忘了BeanDefinition。

Spring设计者牛逼的一点在于， 原生的Bean实例化流程，是不考虑注解这些的。与其让解析注解作为一个原生的功能，不如让注解通过 干涉生命周期的方式，在hook点完成功能。好处在于，如果又有一个新注解和Bean的实例化/初始化有关，那么可以方便的再注册一个PostProcessor。

那么，用于解析 @Autowired注解的类，就是

```kotlin
public class AutowiredAnnotationBeanPostProcessor implements SmartInstantiationAwareBeanPostProcessor,
      MergedBeanDefinitionPostProcessor, PriorityOrdered, BeanFactoryAware {

```

这里SmartInstantiationAwareBeanPostProcessor 我们理解下

> InstantiationAwareBeanPostProcessor 表示 这个处理器是对Bean实例化hook进行回调的。那这个Smart呢? 要理解@Autowired也有循环依赖的问题，所以必要的时候需要从三级缓存里拿数据，这个Smart简单理解支持用户在实例化对象期间访问三级缓存。

> MergedBeanDefinitionPostProcessor 个人理解是，因为@Autowired相当于引入了其他依赖，在xml里没有的，所以在解析@Autowired之前的BeanDefinition需要被修改，这个就是做这件事的

> PriorityOrdered 应该是说定义BeanFactory的先后顺序

> BeanFactoryAware: 表示 我想了解BeanFactory这把刷子， 麻烦你(容器)在必要的时候注入下。

> 看下时序图就行

这里，这个postBean好像也能同时处理被@Lookup修饰的方法，以及被@Value修饰的字段，简单理解这个方法可以是抽象的，其实例会有cglib动态生成，每次获取的对象都是不同的。..

[\[spring\] 注解@Autowired是如何实现的_普通字符串的构造方法如何autowired-CSDN博客](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Ftopdeveloperr%2Farticle%2Fdetails%2F87971446 "https://blog.csdn.net/topdeveloperr/article/details/87971446")

截取下 这个处理器如何判断一个Field是不是被@Autowired修饰的

所以，@Autowired的作用，就是在容器启动时，让这些postProcessor看到对应的字段上也没有被标记，有的话就在这里完成自动注入。

```java

@Nullable
private MergedAnnotation<?> findAutowiredAnnotation(AccessibleObject ao) {
   MergedAnnotations annotations = MergedAnnotations.from(ao);
   for (Class<? extends Annotation> type : this.autowiredAnnotationTypes) {
      MergedAnnotation<?> annotation = annotations.get(type);
      if (annotation.isPresent()) {
         return annotation;
      }
   }
   return null;
}

```