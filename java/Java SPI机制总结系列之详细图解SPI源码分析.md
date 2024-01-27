# Java SPI机制总结系列之详细图解SPI源码分析

重温一下SPI机制的概念：SPI，是Service Provider Interface的缩写，即服务提供者接口，单从字面上看，可以这样理解，该机制提供了一种可根据接口类型去动态加载出接口实现类对象的功能。打一个比喻，该机制就类似Spring容器，通过IOC将对象的创建交给Spring容器处理，若需要获取某个类的对象，就从Spring容器里取出使用即可。同理，在SPI机制当中，提供了一个类似Spring容器的角色，叫**【服务提供者】**，在代码运行过程中，若要使用到实现了某个接口的服务实现类对象，只需要将对应的接口类型交给服务提供者，服务提供者将会动态加载出所有实现了该接口的服务实现类对象，最后给到服务使用者使用。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eaaf786bad064e3b85252350f8fbd8be~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=940&h=780&s=104645&e=png&b=fcfcfc)

接着前文的分享，可从以下三个步骤目录去深入分析Java SPI机制源码实现——

1.  **创建服务提供者ServiceLoader对象，其内部生成一个可延迟加载接口对应实现类对象的迭代器LazyIterator，主要作用是读取并解析META-INF/services/目录下的配置文件中service类名字，进而通过反射加载生成service类对象。** 
2.  **调用serviceLoader.iterator()返回一个内部实际是调用LazyIterator迭代器的匿名迭代器对象。** 
3.  **遍历迭代器，逐行解析接口全类名所对应配置文件中的service实现类的名字，通过反射生成对象缓存到链表，最后返回。** 

```java

ServiceLoader<UserService> serviceLoader = ServiceLoader.load(UserService.class);

Iterator<UserService> serviceIterator = serviceLoader.iterator();

    UserService service = serviceIterator.next();
    service.getName();
    }
}

```

整个过程这里先做一个全面概括——ServiceLoader类会延迟加载UserService接口全名对应的META-INF/services/目录下的配置文件com.zhu.service.UserService。当找到对应接口全名文件后，会逐行读取文件里Class类名的字符串，假如存储的是“com.zhu.service.impl.AUserServiceImpl”和“com.zhu.service.impl.BUserServiceImpl”这两个类名，那么就会逐行取出，再通过反射【“Class类名”.newInstance()】，就可以创建出UserService接口对应的服务提供者对象。这些对象会以结构为<**实现类名, 实现类对象**>的Map形式，存储到LinkedHashMap链表里。该链表将由迭代器循环遍历，取出每一个实现类对象。

画一个流程图说明，大概如下—— ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/286860a6560a44a88fa8d00e4be932bd~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1250&h=837&s=164171&e=png&b=fcfcfc)

接下来，基于该全貌流程图，分别对源码作分析。

**一、创建服务提供者ServiceLoader对象，其内部生成一个可延迟加载接口对应实现类对象的迭代器LazyIterator，主要作用是读取并解析META-INF/services/目录下的配置文件中service类名字，进而通过反射加载生成service类对象。** 
-----------------------------------------------------------------------------------------------------------------------------------------

先看第一部分代码——

```java
ServiceLoader<UserService> serviceLoader = ServiceLoader.load(UserService.class);

```

进入到ServiceLoader.load(UserService.class)方法里，里面基于当前线程通Thread.currentThread().getContextClassLoader()创建一个当前上下文的类加载器ClassLoader，该加载器在这里主要是用来加载META-INF.services目录下的文件。

在load方法里，将UserService.class和类加载器ClassLoader当作参数，交给ServiceLoader中的另一个重载方法ServiceLoader.load(service, cl)去做进一步具体实现。

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}

```

进入到ServiceLoader.load(service, cl)，该方法里创建了一个ServiceLoader对象，该对象默认执行了参数值分别为UserService.class和ClassLoader的带参构造方法。

```java
public static <S> ServiceLoader<S> load(Class<S> service,
                                        ClassLoader loader)
{
    return new ServiceLoader<>(service, loader);
}

```

**根据字面意义，可以看出，ServiceLoader是一个专门负责加载服务的对象，在SPI机制里，它充当专门提供接口实现服务对象的角色。** 

**这里就有两个问题，它怎么提供服务对象，它提供的是哪个接口的服务？**

**针对这两个问题，基于传进来的参数值UserService.class和类加载器ClassLoader，就已经能猜出答案里，它将通过类加载器ClassLoader去加载实现UserService接口的具体服务类对象。** 

进入到ServiceLoader的带参构造函数——

```java
private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    reload();
}

```

这里暂时只需要关注loader和 reload()，而acc是专门用在服务实现类的安全权限访问方面的，本文暂未涉及到acc，后续会考虑专门写一篇文分享下SPI下，如何实现服务实现类的安全权限访问。

传进来的loader如果为空，那么就使用ClassLoader.getSystemClassLoader()，即系统类加载器，可以简单理解，无论如何，都会得到一个非空的类加载器。

接着进入到reload()方法里——

```java

 * Clear this loader's provider cache so that all providers will be reloaded.
 * 清除此加载器的提供程序缓存，以便重新加载所有提供程序。
 * <p> After invoking this method, subsequent invocations of the {@link
 * #iterator() iterator} method will lazily look up and instantiate providers from scratch, 
   just as is done by a newly-created loader.
   调用此方法后，后续调用{@link #iterator() iterator}方法将从零开始惰性查找并实例化提供商，
   就像新创建的加载器一样。
 *
 * <p> This method is intended for use in situations in which new providers
 * can be installed into a running Java virtual machine.
   此方法旨在用于新提供者可以安装到正在运行的Java虚拟机中。
 */
public void reload() {
    providers.clear();
    lookupIterator = new LazyIterator(service, loader);
}

```

根据reload() 方法的注释说明，可以看到，该方法做了两件事：

1.  providers是一个Map结构的链表LinkedHashMap，专门存储服务实例（在这里是存储**UserService接口实现类对象**）的集合，通过clear()方法做了清除，即清空了里面的所有记录。
2.  LazyIterator实现了Iterator迭代器接口，根据类名可以看出，这是一个Lazy懒加载形式的迭代器。

需要额外解释一下延迟加载是什么意思。延迟加载，说明项目启动时不会立马加载，而是需要被用到的时候，才会动态去加载。实现了Iterator迭代器接口的LazyIterator对象，就具备延迟加载的功能。

简单看一下，该LazyIterator的结构——

```java
private class LazyIterator implements Iterator<S>
{
    
    Class<S> service;
    
    ClassLoader loader;
    
    Enumeration<URL> configs = null;
    
    Iterator<String> pending = null;
    
    String nextName = null;

    private LazyIterator(Class<S> service, ClassLoader loader) {
        this.service = service;
        this.loader = loader;
    }
    ......
 }

```

总结这部分源码，主要是创建一个可加载接口服务提供者实例的ServiceLoader类对象，其内部创建一个具有延迟加载功能的迭代器LazyIterator。该LazyIterator迭代器能够延迟去逐行遍历解析出接口全类名所对应配置文件中的Class类名字符串，再将Class类名字符串通过反射生成服务提供者对象，存储到链表，用于外部迭代遍历。

接下来，会基于该延迟加载LazyIterator迭代器，做进一步处理。

到目前为止，只是在ServiceLoader类对象的内部，创建了一个存储接口UserService.class，类加载器loader的LazyIterator迭代器，暂时还没涉及到如何获取接口对应的服务提供者。

简单理解成，菜刀和锅都准备好了，就等切菜和煮菜了。

**二、调用serviceLoader.iterator()返回一个内部实际是调用LazyIterator迭代器的匿名迭代器对象**
------------------------------------------------------------------

这里通过serviceLoader.iterator()得到了一个类型为UserService的迭代器。

```java
Iterator<UserService> serviceIterator = serviceLoader.iterator();

```

先进入到serviceLoader.iterator()内部——

```java
public Iterator<S> iterator() {
    return new Iterator<S>() {

        Iterator<Map.Entry<String,S>> knownProviders
            = providers.entrySet().iterator();

        public boolean hasNext() {
            if (knownProviders.hasNext())
                return true;
            return lookupIterator.hasNext();
        }

        public S next() {
            if (knownProviders.hasNext())
                return knownProviders.next().getValue();
            return lookupIterator.next();
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }

    };
}

```

该方法里，\*\*return new Iterator() { ... }\*\*表示创建一个实现了Iterator接口的匿名内部类实例对象，并返回该实例对象作为一个迭代器。

至于这个匿名对象是叫张三还是李四，都不重要。重要的是，其内部具有能被外部正常调用的hasNext()和next()就可以了。

我画了一幅简单的漫画，举例说明一下，这里为何可以直接返回一个实现Iterator接口的匿名内部类实例对象。

故事是这样的，有一个老板，想要招一个工具人，哦，不对，是打工人（反正都一样......）—— ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fef7125e2a9b4ed1b1e8dfa3f61fc313~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=850&h=421&s=45489&e=png&b=fdfdfd)
 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adee8891ac4347fd85bab6279cce2e20~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1064&h=440&s=63562&e=png&b=fdfdfd)
 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c20096b2c2f14ec89178d52818c1db6e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=912&h=414&s=46837&e=png&b=fdfdfd)

**故事到这里就结束了，这个return new Iterator() { ... }返回的匿名内部类，就像无数籍籍无名的底层打工人一样，或许自始自终都无人知道他们的名字，但他们用自己辛勤的手（hasNext()方法）脚（next()方法），在平凡的岗位上，默默做着不平凡的工作，提供着可以帮助其他人（服务使用者）的服务。** 

接下来，让我们看看这些打工人那布满皱纹的手和脚——

```java
Iterator<Map.Entry<String,S>> knownProviders
    = providers.entrySet().iterator();

public boolean hasNext() {
    if (knownProviders.hasNext())
        return true;
    return lookupIterator.hasNext();
}

public S next() {
    if (knownProviders.hasNext())
        return knownProviders.next().getValue();
    return lookupIterator.next();
}

```

knownProviders是一个包装了LinkedHashMap providers = new LinkedHashMap<>()链表的迭代器。

当调用hasNext()或者next()时，都会判断providers里是否还有可以遍历获取的值，如果空了，就会调用lookupIterator.hasNext()或者lookupIterator.next()。

**这个lookupIterator，正是前文创建的LazyIterator迭代器对象的引用。** 

匿名迭代器对象中的这两个方法，分别是以下两种功能：

*   hasNext()判断迭代器是否存在下一个元素。
*   next()获取迭代器中的下一个元素。

可见，这部分源码调用serviceLoader.iterator()返回一个提供hasNext()和next()方法的匿名迭代器对象，实际上，hasNext()和next()方法内真实调用的是迭代器LazyIterator的hasNext()和next()方法。

**三、遍历迭代器，逐行解析接口全类名所对应配置文件中的service实现类的名字，通过反射生成对象缓存到链表，最后返回。** 
----------------------------------------------------------------

该分析最后的代码了，这里已经到遍历循环迭代器，通过serviceIterator.next()取出存储接口服务提供者对象——

```java
while (serviceIterator.hasNext()) {
    UserService service = serviceIterator.next();
    service.getName();
    }
}

```

这里的hasNext()和next()，正是前文\*\*return new Iterator() { ... }\*\*匿名对象里的hasNext()和next()方法。故而在执行serviceIterator.hasNext()或者serviceIterator.next()，将跳转到#ServiceLoader类#iterator() 中，执行该匿名内部类的hasNext()和next()方法。

先来看hasNext()方法——

```java
public boolean hasNext() {
    if (knownProviders.hasNext())
        return true;
    return lookupIterator.hasNext();
}

```

若是第一次执行时，knownProviders迭代器里的LinkedHashMap链表必定是空的，这时候，就会执行lookupIterator.hasNext()——

```java
public boolean hasNext() {
    if (acc == null) {
    
        return hasNextService();
    } else {
        PrivilegedAction<Boolean> action = new PrivilegedAction<Boolean>() {
            public Boolean run() { return hasNextService(); }
        };
        return AccessController.doPrivileged(action, acc);
    }
}

```

这里acc为空，故而执行的是return hasNextService()语句——

```java
private boolean hasNextService() {
    if (nextName != null) {
        return true;
    }
    if (configs == null) {
        try {
            
            String fullName = PREFIX + service.getName();
            if (loader == null)
                configs = ClassLoader.getSystemResources(fullName);
            else
            
                configs = loader.getResources(fullName);
        } catch (IOException x) {
            fail(service, "Error locating configuration files", x);
        }
    }
    while ((pending == null) || !pending.hasNext()) {
        if (!configs.hasMoreElements()) {
            return false;
        }
        pending = parse(service, configs.nextElement());
    }
    nextName = pending.next();
    return true;
}

```

初次调用，configs是null，而类加载器loader非空，故而会执行configs = loader.getResources(fullName)这行代码。

基于该执行步骤，分析一下这里的configs作用是什么，先看以下两个逻辑——

1.  **PREFIX的值为private static final String PREFIX = "META-INF/services/"，表示正是目录META-INF/services/路径。** 
2.  **service.getName()是获取Class的name值，我们传进来的是UserService.class，故而这里service.getName()获取到的，便是接口全名com.zhu.service.UserService。** 

两者结合，即代码String fullName = PREFIX + service.getName()得到的，便是“METAINF/services/com.zhu.service.UserService”字符串，表示文件路径名。

这时候，我们的类加载器就开始派上用场了——

```javascript
configs = loader.getResources(fullName);

```

没错，到这里已经拿到UserService接口全类名对应的文件路径，就可以通过类加载器读取到该文件资源了。

读取到该文件之后，之后就可以解析存放在文件里的接口的服务实现类信息了，故而具体实现在pending =parse(service, configs.nextElement())这行代码里——

```javascript
while ((pending == null) || !pending.hasNext()) {
    if (!configs.hasMoreElements()) {
        return false;
    }
    
    pending = parse(service, configs.nextElement());
}

```

进入到parse方法里，可以看到，这里开始通过while((lc =parseLine(service, u, r, lc, names))>=0)对文件内容逐行读取，同时创建一个ArrayList names，用来缓存读取出来的类名，具体实现就在parseLine(service, u, r, lc, names))方法里——

```javascript
private Iterator<String> parse(Class<?> service, URL u)
    throws ServiceConfigurationError
{
    InputStream in = null;
    BufferedReader r = null;
    
    ArrayList<String> names = new ArrayList<>();
    try {
        in = u.openStream();
        r = new BufferedReader(new InputStreamReader(in, "utf-8"));
        int lc = 1;
        
        while ((lc = parseLine(service, u, r, lc, names)) >= 0);
    } catch (IOException x) {
        fail(service, "Error reading configuration file", x);
    } finally {
        try {
            if (r != null) r.close();
            if (in != null) in.close();
        } catch (IOException y) {
            fail(service, "Error closing configuration file", y);
        }
    }
    
    return names.iterator();
}

```

进入到parseLine(service, u, r, lc, names))方法，代码String ln = r.readLine()表示读取出文件每一行的字符串赋值给ln。

若遇到有#注释符号的就跳过，只读取非#号注释的类名字符串，以names.add(ln)保存到一个ArrayList里。

```javascript
private int parseLine(Class<?> service, URL u, BufferedReader r, int lc,
                      List<String> names)
    throws IOException, ServiceConfigurationError
{
    String ln = r.readLine();
    if (ln == null) {
        return -1;
    }

    int ci = ln.indexOf('#');
    if (ci >= 0) ln = ln.substring(0, ci);
    ln = ln.trim();
    int n = ln.length();
    
    if (n != 0) {
        if ((ln.indexOf(' ') >= 0) || (ln.indexOf('\t') >= 0))
            fail(service, u, lc, "Illegal configuration-file syntax");
        int cp = ln.codePointAt(0);
        if (!Character.isJavaIdentifierStart(cp))
            fail(service, u, lc, "Illegal provider-class name: " + ln);
        for (int i = Character.charCount(cp); i < n; i += Character.charCount(cp)) {
            cp = ln.codePointAt(i);
            if (!Character.isJavaIdentifierPart(cp) && (cp != '.'))
                fail(service, u, lc, "Illegal provider-class name: " + ln);
        }
        
        if (!providers.containsKey(ln) && !names.contains(ln))
            names.add(ln);
    }
    return lc + 1;
}

```

将读取文件里的类名存到ArrayList后，最后return names.iterator()返回一个iterator迭代器，可debug打印看一下，可以看到该ArrayList缓存了从文件里读取出来的类名—— ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93531072997a4dc2b8d3f880eb45076e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=539&h=156&s=32398&e=png&b=2c2c2c)

该迭代器在解析完成后，会执行一次nextName = pending.next()，表示通过迭代器方式取出ArrayList中的第一个字符串，即“com.zhu.service.impl.AUserServiceImpl”，同时return true。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/153741fb6b9047ac947af2748e75dde9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=964&h=216&s=37650&e=png&b=2b2b2b)

这里nextName = pending.next()和return true就呼应了外部服务使用者的调用，可见serviceIterator.hasNext()内部，若迭代器下一个元素不为空，那么就将下一个元素通过取出，赋值给nextName，同时返回true，让while循环正常遍历下去—— ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87f556416d96472182ebd757f93ea75e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=747&h=196&s=24764&e=png&b=2c2c2c)

前面的nextName = pending.next()将会在serviceIterator.next()里有所体现。

接下来，在next()中，第一次调用，也是lookupIterator.next()方法——

```js
public S next() {
    if (knownProviders.hasNext())
        return knownProviders.next().getValue();
    return lookupIterator.next();
}

```

进入到lookupIterator.next()方法——

```javascript
public S next() {
    if (acc == null) {
        
        return nextService();
    } else {
        PrivilegedAction<S> action = new PrivilegedAction<S>() {
            public S run() { return nextService(); }
        };
        return AccessController.doPrivileged(action, acc);
    }
}

```

同样，实现的是nextService()——

```javascript
private S nextService() {
    if (!hasNextService())
        throw new NoSuchElementException();
    String cn = nextName;
    nextName = null;
    Class<?> c = null;
    try {
        
        *nextName即将前文的com.zhu.service.impl.AUserServiceImpl
        *String cn = nextName
        *通过Class.forName(cn, false, loader)，即可生成AUserServiceImpl的Class类对象
        */
        c = Class.forName(cn, false, loader);
    } catch (ClassNotFoundException x) {
        fail(service,
             "Provider " + cn + " not found");
    }
    if (!service.isAssignableFrom(c)) {
        fail(service,
             "Provider " + cn  + " not a subtype");
    }
    try {
        
        S p = service.cast(c.newInstance());
        
        providers.put(cn, p);
        return p;
    } catch (Throwable x) {
        fail(service,
             "Provider " + cn + " could not be instantiated",
             x);
    }
    throw new Error();          
}

```

在这里面，主要做了这样几件事：

1.  **将nextName字符串赋值给cn，首次调用时，这里的nextName值为“com.zhu.service.impl.AUserServiceImpl”；**
2.  **通过 c = Class.forName(cn,false, loader)生成AUserServiceImpl类的Class对象；**
3.  **通过反射通过c.newInstance()生成AUserServiceImpl类实例对象；**
4.  **生成的对象会以结构为<实现类名, 实现类对象>的Map形式，存储到LinkedHashMap链表里；**
5.  **将生成的对象返回；**

因此，在第一次调用完UserService service = serviceIterator.next()后，就能拿到了接口UserService的第一个实现类对象com.zhu.service.impl.AUserServiceImpl，进而就可以执行相应的重写方法service.getName()。

到while的第二次遍历时，执行serviceIterator.hasNext()后，会取出ArrayList中的第二个缓存类名“com.zhu.service.impl.BUserServiceImpl”赋值给nextName，这样在执行UserService service = serviceIterator.next()时，就会重复执行nextService()里的逻辑。一直迭代遍历，直到将配置里的类名都遍历完，serviceIterator才最终结束该UserService接口的服务提供功能。

首次调用就是以上流程，值得提的一个地方是，在反射创建完成的对象后，将以结构为<实现类名, 实现类对象>的Map形式。存储到LinkedHashMap链表里。

这个LinkedHashMap链表缓存的作用是什么呢？

这时回头去看下这行代码，还记得它里面创建了一个匿名内部类吗—— ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f95c9527838c4a6eb64f1f1f8f4be87d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=727&h=171&s=23881&e=png&b=2c2c2c)

这个匿名内部类里，其hasNext()和next()方法，会判断knownProviders是否为空，不为空才去调用knownProviders里的方法。

这里的knownProviders正是使用到了LinkedHashMap链表缓存里的对象。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d803b580ff754a87b3dc215bddd28a47~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=684&h=562&s=63943&e=png&b=2b2b2b)

这个链表的作用，就是方便出现重复创建一个匿名迭代器去后去获取接口的服务对象时，直接从LinkedHashMap链表缓存里读取即可，无需再次去解析接口对应的配置文件，起到了查询优化的作用。

类似这样的场景，第二次生成一个迭代器去提供接口的服务功能时，就直接从从LinkedHashMap链表缓存里读取了。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45b7a220efa04bd8b31c3f8115c48ace~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=743&h=359&s=40663&e=png&b=2c2c2c)

