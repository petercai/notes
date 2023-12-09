# Jdk动态代理原理

阅读本文，你将会收获：
-----------

*   理解为什么`Jdk`动态代理，代理类必须实现接口
    
*   理解`Jdk`动态代理全过程
    
*   看到`Jdk`的动态代理类的类的内容 ~如果只想看这个可以直接翻到最后/(ㄒoㄒ)/~
    
*   如何自己得到一个代理类的内容
    
*   如何看源码
    
*   变强
    

老规矩，上代码观现象
----------

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/8/169f8abeab86647f~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

依赖的类，明白上面的过程可以跳过不看

```java
public interface HelloService {
    void sayHello();
}

public class HelloServiceImpl implements HelloService {
  @Override
  public void sayHello() {
    System.out.println("给掘金大佬们低头 (:");
  }
}

public class HelloInvocationHandle implements InvocationHandler {

    private Object target;

    public HelloInvocationHandle(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("proxy before=======");
        Object result = method.invoke(target, args);
        System.out.println("proxy end=======");
        return result;
    }
}

```

上面的代码，相信大家在学动态代理的时候，都有写过，我就不赘述了，大家有想过他是怎么实现的吗？为什么调代理类的sayHello()就可以直接使用`InvocationHandler`里的`invoke()`逻辑呢？为什么代理类必须要实现一个接口这么麻烦呢？我们继续往下面看。

源码分析
----

我们看上面的`Main`方法，可以看出来`Proxy.newProxyInstance()`这个方法承载了代理类的所有的逻辑，所有的魔法都在这里面

```bash
//生成一个代理对象
HelloService proxy = (HelloService) Proxy.newProxyInstance(
    Thread.currentThread().getContextClassLoader(),
    hello.getClass().getInterfaces(),handle);


```

我们点进去，看看里面长啥样

```java
    private static final Class<?>[] constructorParams =
        { InvocationHandler.class };

    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h) {
        ...
        
        
        final Class<?>[] intfs = interfaces.clone();
        ...
        
        Class<?> cl = getProxyClass0(loader, intfs);
        ...
        
        
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        
        return cons.newInstance(new Object[]{h});

```

简单的分析一下上面的代码，流程也很简单

*   传入`HelloService.class`类进行代理类的生成，这样代理类就有了原始类的方法信息，例如`sayHello()`
*   通过一个`InvocationHandler`这个构造对象拿到该代理类的构造方法
*   传入之前我们定义的`HelloInvocationHandle`，构造器实例化了活生生的代理对象 从这里我们大致就能明白了`Jdk`动态代理的原理：**克隆一个接口，生成一个新类作为代理类，这个类里面有着我们定义的HelloInvocationHandle进行代理逻辑的处理**

> 这里就回答了我们上面提到的一个问题： Jdk动态代理，代理类必须实现接口，是因为跟他的实现有关系，他规定了必须要传一个接口去生成代理类

所以所有的谜团就在生成代理类的`getProxyClass0(loader, intfs)`这个方法里

```java
private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
       ...
        
        
        
        
        return proxyClassCache.get(loader, interfaces);
    }


private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
     proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory()); 

```

看上面的注释我们可以看出到`ProxyClassFactory`负责生成代理类

我这里看的是`jdk1.8`的源码，用`lambda`重构过，旧版本看到的可以不一样，主逻辑应该没什么变化

```bash
// prefix for all proxy class names
private static final String proxyClassNamePrefix = "$Proxy";

public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
    ...
    //这里定义了代理类的名字，所以你每次看到的代理类都是$Proxy开头的
    String proxyName = proxyPkg + proxyClassNamePrefix + num;
    //关键点在这里，这里生成一个代理类
    byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
    proxyName, interfaces, accessFlags);
}

```

`ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags)`这个方法如果感兴趣的大佬点进去可以看到，里面是用`StringBuilder`拼出了这个代理类的字节码（：，然后转成了`Byte`数组。

Bingo
-----

**关键点来了！** 这个代理类长啥样怎么看？`Debug`没有暴露出来啊亲，看不了，小之看到上面的`byte[] proxyClassFile`这个变量，露出了一丝坏笑，前面说过，这个变量里存的是代理类字节码内容啊！没错！**输出流伺候！** 小之一顿操作，QWER！飞起

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/8/169f8e20cdb6bb79~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

### 哦呵

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/8/169f8e27881d47a5~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

### 反编译，哦呵

```bash
public final class $Proxy0 extends Proxy implements HelloService {
    private static Method m1; //equals()方法
    private static Method m3; //Bingo 我们的sayHello方法()
    private static Method m2; //toString()方法
    private static Method m0; //hashCode()方法
    
    //这里就是我们之前提交的InvocationHandler这个构造方法！！
    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return ((Boolean)super.h.invoke(this, m1, new Object[]{var1})).booleanValue();
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    //勇士来吧！看一看我们的代理类的sayHello()方法长什么样子呀！！
    public final void sayHello() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return ((Integer)super.h.invoke(this, m0, (Object[])null)).intValue();
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
            m3 = Class.forName("proxy.service.HelloService").getMethod("sayHello", new Class[0]);
            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}

```

筒子们，请聚焦上面的代理类`sayHello()`方法。真相大白了吧？代理类里实现了接口里的方法，全部调用了你的`HelloInvocation`的`invoke()`方法啦！

这里说一下上面的`super.h.invoke(this, m3, (Object[])null);`中`super.h`是什么东西，明眼的大佬都应该看出来了吧？继承了Proxy类，是`Proxy`里定义的`InvocationHandler`变量，也就是你的`HelloInvocation`

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/8/169f8e88d8e4d8c2~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

