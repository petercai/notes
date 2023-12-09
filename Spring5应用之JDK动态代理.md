# Spring5应用之JDK动态代理
在我们之前的探索中，已经详细解读了AOP如何借助动态字节码技术来构建动态代理类。实际上，实现动态代理的方式不止一种。其中，`JDK动态代理`、`Cglib`、`ASM`和`Javassist`都是业界常用的技术手段。今天，我将引导大家深入 **Spring AOP的底层原理**，揭示其背后所采用的动态代理技术是如何工作的。为了更加系统地呈现这一内容，我特地选取了JDK动态代理与Cglib这两大主流方法，进行详实的解读。首先，我们将着重了解JDK动态代理的核心原理和实际应用情境。我的目标是，希望大家在了解这些深入的分析后，能够更为全面和深入地理解动态代理背后的精妙设计和实现

开发步骤
----

在之前关于AOP动态代理的探讨中，我们了解到创建AOP代理涉及三大关键要素：`原始对象`、`额外功能`，以及`一个代理对象`，**该代理对象与原始对象共同实现相同的接口**。这种结构在更为底层JDK动态代理的开发中也得到了体现。

考虑以下示例：其中，UserService代表原始对象实现的接口；UserServiceImpl是具体的原始对象，而通过InvocationHandler我们为其定义了额外功能。最终，我们利用`Proxy.newProxyInstance`方法成功地创建了代理类对象。

当我们运行这段代码，测试的输出结果与我们预期的完全一致，证明了我们成功地为原始对象UserServiceImpl注入了额外功能，而这一切都得益于JDK提供的动态代理机制 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8de54677ada7481fa55fd0f41c7be669~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1328&h=146&s=48785&e=png&b=5e72f5)

```java
public class TestJDKProxy {

    private static final Logger log = LoggerFactory.getLogger(TestJDKProxy.class);

    @Test
    public void test1() {
        
        UserService userService = new UserServiceImpl();

        
        UserService userServiceProxy = (UserService) Proxy.newProxyInstance(TestJDKProxy.class.getClassLoader(),
                userService.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        log.debug("log before");

                        
                        Object ret = method.invoke(userService, args);

                        log.debug("log after");
                        return ret;
                    }
                });

        userServiceProxy.login("admin", "123456");
    }
}

```

方法原型分析
------

从上述示例中，我们可以清晰地看到JDK动态代理对象的**创建核心**——`Proxy.newProxyInstance()`方法。这个方法接收三个关键参数：`ClassLoader`、`Class<?>[]` 和 `InvocationHandler`。它们分别代表了**类加载器**、**原始类实现的接口的Class对象**，以及用于**注入的额外功能**。

接下来我将带着深入了解这三个参数的工作原理和用途，这对于我们完整地理解JDK动态代理机制是至关重要的。希望通过这次详尽的分析，大家能够对JDK动态代理有一个更为深入和全面的认识 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b06cbd1b2f544db1a8d04e0fc9afc110~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1468&h=282&s=73058&e=png&b=ffffff)

### ClassLoader

**ClassLoader**，即类加载器，在Java中起着至关重要的作用。但要深入了解它，首先必须回顾Java程序的标准运行流程。典型情况下，**当程序启动时，类加载器首先会读取类对应的字节码文件（.class文件），将其加载到JVM中。随后，JVM会基于这些字节码数据，通过类加载器创建出对应的Class对象，并根据需要进一步实例化为具体对象**。

这个流程在遇到动态代理时遭遇了挑战。动态代理，顾名思义，**其类是在运行时动态生成的，它并没有预先准备好的.class文件**。那么，如何为这样的代理类创建一个Class对象呢？又或者说，ClassLoader在这里扮演什么角色？

实际上，当我们请求JVM创建一个动态代理时，JVM会为我们“临时”生成这个代理类的字节码。这并不是从文件系统中读取的，而是基于我们给定的接口和实现，即时生成的。 **在这里，ClassLoader的任务是加载这个“临时生成”的字节码到JVM的内存中**。这意味着，尽管代理类的字节码并没有物理存在，但ClassLoader依然可以处理它，就像处理其他常规Java类一样。

但这里有一个细节值得注意：这个用于加载动态代理的ClassLoader并不是新创建的，而是`借用了现有的一个类加载器`。这点尤为重要，因为在Java项目中，每个类都有它自己对应的类加载器，确保了类的隔离和安全性。在动态代理的场景中，我们实际上是复用了某个现有类的加载器来加载代理类，确保代理类能够顺利地与原始 类在同一个上下文中工作

### Class<?>\[\]

在`Proxy.newProxyInstance()`方法中，第二个参数Class<?>\[\]起着至关重要的作用。它是一个Class对象的数组，代表了一组接口。当我们希望创建一个动态代理对象时，**这些接口定义了创建的代理对象将额外功能加在哪些原始类方法上**。

为什么是接口而不是具体的类呢？这是因为`JDK的动态代理机制建立在接口的基础之上`。具体来说，动态代理生成的代理类会实现指定的一组接口，而不是继承某个类。这使得动态代理具有很大的灵活性，因为**一个Java类可以实现多个接口，但只能继承一个父类**。

通过传递一个Class对象的数组作为参数，我们**告诉JVM我们希望代理类实现哪些接口，将额外功能加在哪些原始类方法上**。然后，动态生成的代理类将会实现这些接口，并在每个接口方法的实现中，根据我们的需求，调用InvocationHandler来处理方法调用。

简而言之，`Class<?>[]`参数为`Proxy.newProxyInstance()`方法提供了一个蓝图，说明代理类应如何构建，并且定义了其行为特征

### InvocationHandler

`InvocationHandler`是JDK动态代理机制中的一个关键接口，其定义了如何在代理对象上处理方法调用。该接口中，仅包含一个名为`invoke`的方法。此方法在设计上，旨在**调用原始对象的方法，同时为其注入额外的功能**。

**当代理对象上的一个方法被调用时，invoke方法就会被触发**。它提供了我们一个场所，允许我们在原始方法执行前后添加自定义的行为或功能，从而扩展或改变原始方法的行为。 关于invoke方法的三个参数，它们分别为：

1.  **Proxy**：这是正在调用的方法所属的代理实例。大多数情况下，我们并不直接使用它，因为在InvocationHandler实现内部调用该代理对象会导致无限递归。
2.  **Method**：这代表了**被代理对象的某个具体方法的反射对象**它为我们提供了调用原始对象方法的能力，这可以通过`method.invoke(targetObject, args)`来完成，其中**targetObject是原始对象的实例**。
3.  **Object\[\]**：这是被代理方法的参数数组，表示在代理对象上调用方法时传递的参数。

通过组合上述三个参数，我们可以在invoke方法中灵活地调用原始方法，同时根据需要为其添加额外的逻辑或功能，从而实现对原始方法行为的定制。

在我们深入了解JDK的InvocationHandler接口后，不禁让人回想起Spring AOP中的一个相似结构——`MethodInterceptor接口`。Spring AOP在动态代理实现中提供了这个接口，它与JDK的动态代理机制的核心思想相似，但是Spring对其进行了封装。

`MethodInterceptor接口`的设计是简洁而聚焦的。它的中心是一个invoke方法，这个方法的目的与`InvocationHandler`中的`invoke`相似： **拦截并增强方法调用**。但不同的是，Spring选择了一个集成的方法来传递信息。而不是分开的多个参数，MethodInterceptor的invoke方法接受一个封装了方法调用详情的`MethodInvocation对象`。这个对象包含了调用的方法、目标对象、参数等所有必要的信息，而且还提供了一个proceed方法，用于执行原始的方法调用。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53727b0aa04b40b6a2cee42327238e36~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1344&h=590&s=137595&e=png&b=fcfcfc)

这种设计方式优雅地集合了所有的方法调用信息，使得代理的实现既简洁又集中。开发者不再需要从多个参数中提取信息，而是可以直接与MethodInvocation对象交互，从而更直观、高效地实现代理逻辑。

总结来说，**Spring的MethodInterceptor接口提供了一种高效、简化的方式来实现动态代理**，使得开发者可以更聚焦于业务逻辑的增强，而不是方法调用的底层细节

```java
package java.lang.reflect;
public interface InvocationHandler {
    
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}

```

```java
package org.aopalliance.intercept;

@FunctionalInterface
public interface MethodInterceptor extends Interceptor {
    Object invoke(MethodInvocation var1) throws Throwable;
}

```

