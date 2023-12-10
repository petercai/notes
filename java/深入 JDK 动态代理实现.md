# 深入 JDK 动态代理实现
基本使用
----

没有花里胡哨的，就下面的代码：

```java
UserService userService = new UserServiceImpl();
UserService o = (UserService) Proxy.newProxyInstance(UserService.class.getClassLoader(), new Class[]{UserService.class}, new InvocationHandler() {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("--before--");
        Object invoke = method.invoke(userService, args);
        System.out.println("--after--");
        return invoke;
    }
});

```

可以看出，关键在于代理的创建：`Proxy.newProxyInstance`。后面我们也以这里为入口深入。

关键步骤解析
------

### 生成代理类

生成代理类是动态代理中的最复杂的步骤，而真正生成代理类字节码的方法位于：`java.lang.reflect.Proxy.ProxyClassFactory#apply`。

精简一下方法，主要有以下几个步骤：

```java
    
    interfaceClass = Class.forName(intf.getName(), false, loader);
    if(xxx){
        throw new IllegalArgumentException(xxx);
    }
    ...
    
    int accessFlags = Modifier.PUBLIC | Modifier.FINAL;
    ...
    String proxyName = proxyPkg + proxyClassNamePrefix + num;
    ...
    
    byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
        proxyName, interfaces, accessFlags);
    ...
    
    return defineClass0(loader, proxyName,
        proxyClassFile, 0, proxyClassFile.length);


```

生成代理类与加载是动态代理的核心。

#### 组装 ProxyMethod

> Step 1: Assemble ProxyMethod objects for all methods to generate proxy dispatching code for.

这里只是将收集到的方法元信息封装为`sun.misc.ProxyGenerator.ProxyMethod` 类存储，并没有真正的生成 code。

这里还特地也保存了了`hashCode`,`equals`,`toString`方法。

```java
addProxyMethod(hashCodeMethod, Object.class);
addProxyMethod(equalsMethod, Object.class);
addProxyMethod(toStringMethod, Object.class);
        
...

sigmethods.add(new ProxyMethod(name, parameterTypes, returnType,
    exceptionTypes, fromClass));

```

#### 组装 FieldInfo / MethodInfo

> Step 2: Assemble FieldInfo and MethodInfo structs for all of fields and methods in the class we are generating.

添加构造方法，并根据上一步组装好的`ProxyMethod`添加成员属性。

在这一步中，会调用`sun.misc.ProxyGenerator.ProxyMethod#generateMethod`方法，为其生成code。

```java

this.methods.add(this.generateConstructor());
...
    
    fields.add(new FieldInfo(pm.methodFieldName,
        "Ljava/lang/reflect/Method;",
         ACC_PRIVATE | ACC_STATIC))；
    
    methods.add(pm.generateMethod());
...

methods.add(generateStaticInitializer());
     

```

#### 构造最终类

> Step 3: Write the final class file.

根据上面几步收集到的信息，生成最终 byte 数组。

```java
                
dout.writeInt(0xCAFEBABE);
                            
dout.writeShort(CLASSFILE_MINOR_VERSION);
                            
dout.writeShort(CLASSFILE_MAJOR_VERSION);

cp.write(dout); 

                            
dout.writeShort(accessFlags);
                            
dout.writeShort(cp.getClass(dotToSlash(className)));
                            
dout.writeShort(cp.getClass(superclassName));
...

```

#### 最终结果

通过配置 `sun.misc.ProxyGenerator.saveGeneratedFiles=true` 可以将生成的代理类字节码保存下来。

删除了我们无需关心 `hashCode`,`toString`等方法后，反编译结果如下：

```java
package com.sun.proxy;

import git.frank.proxy.UserService;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy implements UserService {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final String getUserInfo() throws  {
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
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("git.frank.proxy.UserService").getMethod("getUserInfo");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}


```

### 加载代理类

我们以 openJdk 中的代码为参考，这里实际调用的方法是：`JavaLangAccess#defineClass`。

> Defines a class with the given name to a class loader.

他真正的代码位于 `java.lang.System` 中，以匿名内部类的形式。

```java
    
    private static void setJavaLangAccess() {
     SharedSecrets.setJavaLangAccess(new JavaLangAccess() {
            ...
            public Class<?> defineClass(ClassLoader loader, String name, byte[] b, ProtectionDomain pd, String source) {
                return ClassLoader.defineClass1(loader, name, b, 0, b.length, pd, source);
            }
            ...
    }

```

所以，这里就是使用给定的类加载器去加载刚才生成的class。当被代理方法`getUserInfo`被调用时，便会被代理类转至传入的 `InvocationHandler#invoke` 方法中。

**那么，为什么在代码中没有直接调用 `ClassLoader.defineClass1`，而是这样绕了一圈呢？**

我们再仔细看看，因为 `ClassLoader` 中的 `defineClass` 相关代码都是 protected 的呀~，根本没办法直接调用。

所以 openJdk 才创造了一个 `JavaLangAccess` 类，专门用于让外界调用 `java.lang` 包内的内容。

### 构造对象

构造对象就简单很多，单纯的使用反射调用构造方法去创建刚才生成的代理类实例。

使用入参为 `InvocationHandler` 的构造方法，并传入用户编写的 InvocationHandler 对象。

```java
final Constructor<?> cons = cl.getConstructor(constructorParams);
...
cons.newInstance(new Object[]{h})

```

这里构造的实际上就是上面生成的 `com.sun.proxy.$Proxy0` 类实例。

动态代理中的缓存机制
----------

使用动态代理时并不是每次都要经过上面那一大串代码的，只要同 classLoader + interfaces 相同，都可以使用同一个 proxy 类。

所以 jdk 为动态代理生成的 class 也提供了缓存机制，下次使用的时候，直接通过反射创建 proxy 实例就可以，无需再重新生成 clazz 。

这里的缓存主要通过 `java.lang.reflect.WeakCache` 实现。

使用 key + sub-key 唯一确定一个 value。

其中，key 为传入的 classLoader，sub-key 为通过 key + 传入的接口 class 计算得出。

缓存构成如下图：

![](_assets/725e6f25eb83480b8572952d6d2dd44e~tplv-k3u1fbpfcp-zoom-in-crop-mark!1512!0!0!0.awebp.webp)

### 懒加载

起初 `valuesMap` 中缓存的并不是最中的代理类，而是一个 `ProxyClassFactory`。

当调用 `valueFactory.apply(key, parameter)` 生成真正的代理类后，会将缓存由 factory 对象更换为真正的代理。

```java
if (!valuesMap.replace(subKey, this, cacheValue)) {
    throw new AssertionError("Should not reach here");
}

```

完整的获取缓存流程如下：

![](_assets/b22e31c19f3a4e258a358193af822618~tplv-k3u1fbpfcp-zoom-in-crop-mark!1512!0!0!0.awebp.webp)