# Java | JDK 动态代理的原理其实很简单
*   代理模式（Proxy Pattern）也称委托模式（Delegate Pattern），是一种结构型设计模式，也是一项基础设计技巧；
*   其中，动态代理有很多有意思的应用场景，比如 AOP、日志框架、全局性异常处理、事务处理等。这篇文章，我们主要讨论最基本的 JDK 动态代理。

* * *

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09d36aec78614337b4e66d7ac4daa1b1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

这篇文章的内容会涉及以下前置 / 相关知识，贴心的我都帮你准备好了，请享用~

*   [Java 反射机制（含 Kotlin）](https://juejin.cn/post/6889833658669072397 "https://juejin.cn/post/6889833658669072397")
*   [关于 Java 泛型能问的都在这里了（含Kotlin）](https://juejin.cn/post/6888345234653052941 "https://juejin.cn/post/6888345234653052941")

* * *

*   **什么是代理 (模式)？** 代理模式 (Proxy Pattern) 也称委托模式 (Deletage Pattern)，属于结构型设计模式，也是一项基本的设计技巧。通常，代理模式用于处理两种问题：
    *   **1、控制对基础对象的访问**
    *   **2、在访问基础对象时增加额外功能**

这是两种非常朴素的场景，正因如此，我们常常会觉得其它设计模式中存在代理模式的影子。UML 类图和时序图如下：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9773f45228f441bab350e1311a32120~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/539e027eb76d4d95a0beff2631497ea9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

*   **代理的基本分类：**  静态代理 \+ 动态代理，分类的标准是 **“代理关系是否在编译期确定；**
    
*   **动态代理的实现方式：**  JDK、CGLIB、Javassist、ASM
    

* * *

#### 2.1 静态代理的定义

静态代理是指代理关系在编译期确定的代理模式。使用静态代理时，通常的做法是为每个业务类抽象一个接口，对应地创建一个代理类。举个例子，需要给网络请求增加日志打印：

```typescript
1、定义基础接口
public interface HttpApi {
    String get(String url);
}

2、网络请求的真正实现
public class RealModule implements HttpApi {
     @Override
     public String get(String url) {
         return "result";
     }
}

3、代理类
public class Proxy implements HttpApi {
    private HttpApi target;

    Proxy(HttpApi target) {
        this.target = target;
    }

    @Override
    public String get(String url) {
        
        Log.i("http-statistic", url);
        
        return target.get(url);
    }
}

```

#### 2.2 静态代理的缺点

*   **1、重复性：**  需要代理的业务或方法越多，重复的模板代码越多；
*   **2、脆弱性：**  一旦改动基础接口，代理类也需要同步修改（因为代理类也实现了基础接口）。

* * *

#### 3.1 动态代理的定义

动态代理是指代理关系在运行时确定的代理模式。需要注意，JDK 动态代理并不等价于动态代理，前者只是动态代理的实现之一，其它实现方案还有：CGLIB 动态代理、Javassist 动态代理和 ASM 动态代理等。因为代理类在编译前不存在，代理关系到运行时才能确定，因此称为动态代理。

#### 3.2 JDK 动态代理示例

我们今天主要讨论JDK 动态代理（Dymanic Proxy API），它是 JDK1.3 中引入的特性，核心 API 是 Proxy 类和 InvocationHandler 接口。它的原理是利用反射机制在运行时生成代理类的字节码。

我们继续用打印日志的例子，使用动态代理时：

```typescript
public class ProxyFactory {
    public static HttpApi getProxy(HttpApi target) {
        return (HttpApi) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new LogHandler(target));
    }

    private static class LogHandler implements InvocationHandler {
        private HttpApi target;

        LogHandler(HttpApi target) {
            this.target = target;
        }
        
        @Override
        public Object invoke(Object proxy, Method method, @Nullable Object[] args)       
               throws Throwable {
            
            Log.i("http-statistic", (String) args[0]);
            
            return method.invoke(target, args);
        }
    }
}

```

如果需要兼容多个业务接口，可以使用泛型：

```typescript
public class ProxyFactory {
    @SuppressWarnings("unchecked")
    public static <T> T getProxy(T target) {
        return (T) Proxy.newProxyInstance(
        target.getClass().getClassLoader(),
        target.getClass().getInterfaces(),
        new LogHandler(target));
    }

    private static class LogHandler<T> implements InvocationHandler {
        
    }
}

```

客户端调用：

```ini
HttpAPi proxy = ProxyFactory.getProxy<HttpApi>(target)
OtherHttpApi proxy = ProxyFactory.getProxy<OtherHttpApi>(otherTarget)

```

通过泛型参数传递不同的类型，客户端可以按需实例化不同类型的代理对象。基础接口的所有方法都统一到 InvocationHandler#invoke() 处理。静态代理的两个缺点都得到解决：

*   1、重复性：即使有多个基础业务需要代理，也不需要编写过多重复的模板代码；
*   2、脆弱性：当基础接口变更时，同步改动代理并不是必须的。

#### 3.3 静态代理 & 动态代理对比

*   共同点：两种代理模式实现都在不改动基础对象的前提下，对基础对象进行访问控制和扩展，符合开闭原则。
*   不同点：静态代理存在重复性和脆弱性的缺点；而动态代理（搭配泛型参数）可以实现了一个代理同时处理 N 种基础接口，一定程度上规避了静态代理的缺点。从原理上讲，静态代理的代理类 Class 文件在编译期生成，而动态代理的代理类 Class 文件在运行时生成，代理类在 coding 阶段并不存在，代理关系直到运行时才确定。

* * *

这一节，我们来分析 JDK 动态代理的源码，核心类是 Proxy，主要分析 Proxy 如何生成代理类，以及如何将方法调用统一分发到 InvocationHandler 接口。

#### 4.1 API 概述

Proxy 类主要包括以下 API：

| Proxy | 描述 |
| --- | --- |
| getProxyClass(ClassLoader, Class...) : Class | 获取实现目标接口的代理类 Class 对象 |
| newProxyInstance(ClassLoader,Class<?>\[\],InvocationHandler) : Object | 获取实现目标接口的代理对象 |
| isProxyClass(Class<?>) : boolean | 判断一个 Class 对象是否属于代理类 |
| getInvocationHandler(Object) : InvocationHandler | 获取代理对象内部的 InvocationHandler |

#### 4.2 核心源码

`Proxy.java`

```ini
1、获取代理类 Class 对象
public static Class<?> getProxyClass(ClassLoader loader,Class<?>... interfaces){
    final Class<?>[] intfs = interfaces.clone()
    ...
    1.1 获得代理类 Class 对象
    return getProxyClass0(loader, intfs)
}

2、实例化代理类对象
public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h){
    ...
    final Class<?>[] intfs = interfaces.clone()
    2.1 获得代理类 Class对象
    Class<?> cl = getProxyClass0(loader, intfs)
    ...
    2.2 获得代理类构造器 (接收一个 InvocationHandler 参数)
    // private static final Class<?>[] constructorParams = { InvocationHandler.class }
    final Constructor<?> cons = cl.getConstructor(constructorParams)
    final InvocationHandler ih = h
    ...
    2.3 反射创建实例
    return newInstance(cons, ih)
}

```

可以看到，实例化代理对象也需要先通过 getProxyClass0(...) 获取代理类 Class 对象，而 newProxyInstance(...) 随后会获取参数为 InvocationHandler 的构造函数实例化一个代理类对象。

我们先看下代理类 Class 对象是如何获取的：

`Proxy.java`

```java
-> 1.1、2.1 获得代理类 Class对象
private static Class<?> getProxyClass0(ClassLoader loader,Class<?>... interfaces) {
    ...
    从缓存中获取代理类，如果缓存未命中，则通过ProxyClassFactory生成代理类
    return proxyClassCache.get(loader, interfaces);
}

private static final class ProxyClassFactory implements BiFunction<ClassLoader, Class<?>[], Class<?>>{

    3.1 代理类命名前缀
    private static final String proxyClassNamePrefix = "$Proxy";

    3.2 代理类命名后缀，从 0 递增（原子 Long）
    private static final AtomicLong nextUniqueNumber = new AtomicLong();

    @Override
    public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
        Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
        3.3 参数校验
        for (Class<?> intf : interfaces) {
            
            
            
            
        }
        

        3.4（一般地）代理类包名
        
        String proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";

        3.5 代理类的全限定名
        long num = nextUniqueNumber.getAndIncrement();
        String proxyName = proxyPkg + proxyClassNamePrefix + num;

        3.6 生成字节码数据
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces);

        3.7 从字节码生成 Class 对象
        return defineClass0(loader, proxyName,proxyClassFile, 0, proxyClassFile.length); 
    }
}

-> 3.6 生成字节码数据
public static byte[] generateProxyClass(final String var0, Class[] var1) {
    ProxyGenerator var2 = new ProxyGenerator(var0, var1);
    ...
    final byte[] var3 = var2.generateClassFile();
    return var3;
}

```

`ProxyGenerator.java`

```csharp
private byte[] generateClassFile() {
    3.6.1 只代理Object的hashCode、equals和toString
    this.addProxyMethod(hashCodeMethod, Object.class);
    this.addProxyMethod(equalsMethod, Object.class);
    this.addProxyMethod(toStringMethod, Object.class);

    3.6.2 代理接口的每个方法
    ...
    for(var1 = 0; var1 < this.interfaces.length; ++var1) {
        ...
    }
    
    3.6.3 添加带有 InvocationHandler 参数的构造器
    this.methods.add(this.generateConstructor());
    var7 = this.proxyMethods.values().iterator();
    while(var7.hasNext()) {
        ...
        3.6.4 在每个代理的方法中调用InvocationHandler#invoke()
    }

    3.6.5 输出字节流
    ByteArrayOutputStream var9 = new ByteArrayOutputStream();
    DataOutputStream var10 = new DataOutputStream(var9);
    ...
    return var9.toByteArray();
}

```

以上代码已经非常简化了，主要关注核心流程：JDK 动态代理生成的代理类命名为 com.sun.proxy$Proxy\[从0开始的数字\]（例如：com.sun.proxy$Proxy0），这个类继承自 java.lang.reflect.Proxy。其内部还有一个参数为 InvocationHandler 的构造器，对于代理接口的方法调用都会分发到 InvocationHandler#invoke()。

UML 类图如下，需要注意图中红色箭头，表示代理类和 HttpApi 接口的代理关系在运行时才确定：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4efed8e68ae4598b058f3cc733ba7e5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

> **提示：**  Android 系统中生成字节码和从字节码生成 Class 对象的步骤都是 native 方法：
> 
> *   private static native Class<?> generateProxy(…)
> *   对应的native方法：dalvik/vm/native/java\_lang\_reflect_Proxy.cpp

#### 4.3 查看代理类源码

可以看到，ProxyGenerator#generateProxyClass() 其实是一个静态 public 方法，所以我们直接调用，并将代理类 Class 的字节流写入磁盘文件，使用 IntelliJ IDEA 的反编译功能查看源代码。

输出字节码：

```ini
byte[] classFile = ProxyGenerator.generateProxyClass("$proxy0",new Class[]{HttpApi.class})
// 直接写入项目路径下，方便使用IntelliJ IDEA的反编译功能
String path = "/Users/pengxurui/IdeaProjects/untitled/src/proxy/HttpApi.class"
try(FileOutputStream fos = new FileOutputStream(path)){
    fos.write(classFile)
    fos.flush()
    System.out.println("success")
} catch (Exception e){
    e.printStackTrace()
    System.out.println("fail")
}

```

反编译结果：

```scala
public final class $proxy0 extends Proxy implements HttpApi {
    
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m0;

    public $proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    
     * Object#hashCode()
     * Object#equals(Object)
     * Object#toString()
     */

    
    public final String get() throws  {
        try {
            
            return (String)super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            
            
            
            m3 = Class.forName("HttpApi").getMethod("get");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}

```

#### 4.4 常见误区

*   **~基础对象必须实现基础接口，否则不能使用动态代理~**

这个想法可能来自于一些没有实现任何接口的类，因此就没有办法得到接口的Class对象作为Proxy#newProxyInstance() 的参数，这确实会带来一些麻烦，举个例子：

```kotlin
package com.domain;
public interface HttpApi {
    String get();
}


package com.domain.inner;
interface OtherHttpApi{
    String get();
}

package com.domain.inner;

public class OtherHttpApiImpl  /**extends OtherHttpApi**/{
    public String get() {
        return "result";
    }
}


 HttpApi api = (HttpApi) Proxy.newProxyInstance(...}, new InvocationHandler() {
    OtherHttpApiImpl impl = new OtherHttpApiImpl();
  
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        
        
        return method.invoke(impl,args);
    }
});
api.get();

```

在这个例子里，OtherHttpApiImpl 类因为历史原因没有实现 HttpApi 接口，虽然方法签名与 HttpApi 接口的方法签名完全相同，但是遗憾，无法完成代理。也有补救的办法，找到 HttpApi 接口中签名相同的 Method，使用这个 Method 来转发调用。例如：

```scss
HttpApi api = (HttpApi) Proxy.newProxyInstance(...}, new InvocationHandler() {
    OtherHttpApiImpl impl = new OtherHttpApiImpl();

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        
        if (method.getDeclaringClass() != impl.getClass()) {
            
            Method realMethod = impl.getClass().getDeclaredMethod(method.getName(), method.getParameterTypes());
            return realMethod.invoke(impl, args);
        }else{
            return method.invoke(impl,args);
        }
    }
});

```

* * *

静态代理在设计模式中随处可见，但存在重复性和脆弱性的缺点，动态代理的代理关系在运行时确定，可以实现一个代理处理 N 种基础接口，一定程度上规避了静态代理的缺点。在我们熟悉的一个网络请求框架中，就充分利用了动态代理的特性，你知道是在说哪个框架吗？

* * *

#### 参考资料

*   Android源码设计模式解析与实战 —— 何红辉,关爱民 著
*   修炼Java开发技术:在架构中体验设计模式和算法之美 —— 于广 著
*   深入理解Android内核设计思想 —— 林学森 著
*   [Wikipedia:Aspect-oriented programming](https://link.juejin.cn/?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FAspect-oriented_programming "https://en.wikipedia.org/wiki/Aspect-oriented_programming")
*   [Explore the Dynamic Proxy API](https://link.juejin.cn/?target=https%3A%2F%2Fwww.javaworld.com%2Farticle%2F2076233%2Fexplore-the-dynamic-proxy-api.html%3Fnsdr%3Dtrue "https://www.javaworld.com/article/2076233/explore-the-dynamic-proxy-api.html?nsdr=true") —— Jeremy Blosser 著
*   [Generically chain dynamic proxies](https://link.juejin.cn/?target=https%3A%2F%2Fwww.javaworld.com%2Farticle%2F2071723%2Fgenerically-chain-dynamic-proxies.html%3Fnsdr%3Dtrue "https://www.javaworld.com/article/2071723/generically-chain-dynamic-proxies.html?nsdr=true") —— Srijeeb Roy 著
*   [动态代理竟然如此简单！](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FO1zXLyMQx_4CsA0Ksikqdw "https://mp.weixin.qq.com/s/O1zXLyMQx_4CsA0Ksikqdw") —— cxuan 著

* * *
