# JDK动态代理之实现与原理(二)【JDK篇】
一、JDK动态代理实现原理
-------------

*   动态代理类的生成是通过Proxy.newProxyInstance方法，如下面来自第一节的例子：

```bash
    // 创建jdk动态代理
    UserService jdkProxy = (UserService) Proxy.newProxyInstance(UserService.class.getClassLoader(),
            new Class[]{UserService.class}, handler);

```

*   进入Proxy.newProxyInstance源码

```bash
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
    throws IllegalArgumentException
    // 删除了影响阅读的异常捕获部分，并加上了响应注释。
    // 拦截处理器不能为空 为空抛出异常
    Objects.requireNonNull(h);
    
    // 防御性拷贝 避免修改 interfaces数组对象的属性
    final Class<?>[] intfs = interfaces.clone();
    
    // 获取java安全管理器
    final SecurityManager sm = System.getSecurityManager();
    // 安全管理器不为空
    if (sm != null) {
        // 校验代理参数访问权限
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }

    // 1. 寻找或生成代理类信息 即 Class对象
    Class<?> cl = getProxyClass0(loader, intfs);

    try {
        if (sm != null) {
            checkNewProxyPermission(Reflection.getCallerClass(), cl);
        }
        // 2. 根据类信息对象c1生成构造器
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        final InvocationHandler ih = h;
        // 设置构造起访问权限
        if (!Modifier.isPublic(cl.getModifiers())) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    cons.setAccessible(true);
                    return null;
                }
            });
        }
        // 3. 用构造器生成代理对象 将h拦截处理器设置到代理对象中
        return cons.newInstance(new Object[]{h});
    }

```

我们看到关键代码注释 1、2、3 。得知生成代理类逻辑： 生成代理类的class对象c1--> 根据class对象c1生成类构造器对象cons--> 根据构造器cons生成代理。

*   进入注释1的源码查看如何生成代理类的class信息

```bash
   private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
        // 接口数组超过65535抛出异常
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }
        // 如果通过给定类加载器定义并实现指定接口的代理类的class对象存在，则直接返回。
        // 否则通过ProxyClassFactory创建代理类的class对象
        return proxyClassCache.get(loader, interfaces);
    }

```

我们看到 proxyClassCache.get 方法是生成代理类class对象的关键。

*   进入proxyClassCache.get方法查看如何生成代理类的class信息

```bash
public V get(K key, P parameter) {
    // 忽略其他代码我们只看 创建代理Class的代码
    // 创建subkey
    Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
    // 根据subkey 找回可能的被存储的supplier
    Supplier<V> supplier = valuesMap.get(subKey);
    V value = supplier.get();
    // 返回代理类Class对象
    return value;
}

```

这里用到了缓存机制获取代理类Class对象，我们在第三节进行详细讲述。

二、为什么JDK动态代理必须实现接口
------------------

*   因为JDK动态代理类继承了Proxy类，Java又是单继承的，所以被代理对象只能通过实现接口，让代理对象生成相应接口的实现。  
    那么，问题来了。为什么JDK动态代理要继承Proxy类呢？留给大家思考！

三、JDK生成的动态代理类实例
---------------

第一节生成的动态代理类

```bash
public final class $Proxy0 extends Proxy implements UserService {
    private static Method m1;
    private static Method m2;
    private static Method m0;
    private static Method m3;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
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
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void addUser() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
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
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
            m3 = Class.forName("com.java24k.example.service.UserService").getMethod("addUser");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}


```

*   调用动态代理类的任何方法，都是调用它的父类的拦截处理器的相应方法。
*   拦截处理器持有真实对象，在拦截处理器的invoke方法中回调用真实对象的相应方法。