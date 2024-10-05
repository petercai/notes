# 深入Spring ApplicationArguments 接口及其默认实现
[《Spring Boot 源码学习系列》](https://juejin.cn/column/7323793129709387816 "https://juejin.cn/column/7323793129709387816")

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e9fe32052694b9c89bf6a1254be250f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1018&h=534&s=464071&e=png&b=cad5ca)

一、引言
----

在 [《SpringApplication 的 run 方法核心流程介绍》](https://juejin.cn/post/7406253583317434420 "https://juejin.cn/post/7406253583317434420") 博文中，我们知道了 `ApplicationArguments` 是 **Spring Boot** 中用于获取 **应用程序启动参数** 的接口，其默认实现是 `DefaultApplicationArguments`。

不过有关内容尚未详细介绍，本篇就带大家深入分析下 `ApplicationArguments` 接口及其默认实现。

二、主要内容
------

> 注意： 以下涉及 **Spring Boot** 源码 均来自版本 **2.7.9**，其他版本有所出入，可自行查看源码。

### 2.1 ApplicationArguments

首先来看 **应用程序启动参数接口类** `ApplicationArguments` 的源码：

```java
public interface ApplicationArguments {
  String[] getSourceArgs();
  Set<String> getOptionNames();
  boolean containsOption(String name);
  List<String> getOptionValues(String name);
  List<String> getNonOptionArgs();
}

```

`ApplicationArguments` 接口共包含 `5` 个方法，均用于运行 `SpringApplication` 的参数的访问：

*   `getSourceArgs`：该方法返回传递给应用程序的原始未处理参数。
*   `getOptionNames`：该方法返回所有选项参数的名称，如果没有则返回一个空集。例如，如果参数是 `"--foo=bar --debug"`，则返回值应为 `["foo", "debug"]`。
*   `containsOption`：该方法返回从参数中解析出的选项参数集合中是否包含给定名称的选项。如果参数中包含给定名称的选项，则返回 `true`。
*   `getOptionValues`：该方法返回与给定名称的参数选项关联的值集合。
    *   如果选项存在且没有参数（例如：`"--foo"`），则返回空集合（`[]`）。
    *   如果选项存在且有一个值（例如：`"--foo=bar"`），则返回包含一个元素的集合（`["bar"]`）。
    *   如果选项存在且有多个值（例如：`"--foo=bar --foo=baz"`），则返回一个包含每个值的元素的集合（`["bar", "baz"]`）。
    *   如果选项不存在，则返回 `null`。
*   `getNonOptionArgs`：该方法返回解析出的非选项参数的集合，如果没有则返回一个空列表。

### 2.2 DefaultApplicationArguments

在 `SpringApplication` 的 `run` 方法中，我们可以看到如下标红的内容：

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/f2903f1da7f14e058c61805553813549~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=%2FUU2iNCl8vO9mPKFpsNy%2Fc3wufk%3D)

`DefaultApplicationArguments` 就是 `ApplicationArguments` 接口的一个默认实现，还是来看看相关源码： ![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/2d48f78caeac4663a6863051becf0f3c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=a7BUWXhfWiwVvyV8ojW5NddwIkw%3D)

#### 2.2.1 成员变量

`DefaultApplicationArguments` 的成员变量有两个：

*   `Source source` ：私有的静态内部类，继承自 `org.springframework.core.env.SimpleCommandLinePropertySource`【**spring-core** 包中的一个类，旨在提供解析命令行参数的最简单方法】
*   `String[] args`：原始的命令行参数数组

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/f595a4815d9a4069a935cd342aecff9a~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=8KZ8df0GTIoCVEIYnkfts57Mhk8%3D)

这个 `Source` 类是对 `SimpleCommandLinePropertySource` 的一个简单封装，可以看到它这里只是简单地调用了父类的实现，没有添加任何新的功能或逻辑。当然就目前而言，这个类似乎是多余的，**Huazie** 猜测这也许是为了将来的扩展吧。

#### 2.2.2 构造方法

```java
public DefaultApplicationArguments(String... args) {
  Assert.notNull(args, "Args must not be null");
  this.source = new Source(args);
  this.args = args;
}

```

构造方法主要用来初始化上述两个成员变量，它接受一个可变长度的字符串数组作为参数，该参数就是运行 `SpringApplication` 的命令行参数。

#### 2.2.3 成员方法

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/237534ce24884e9b895478c943b083bd~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=5C7EI6dHH8z6vp11p0ghNnQssg4%3D)

阅读上述源码，可以看到实现的 **5** 个方法中都跟成员变量 `source` 有关系。

在 `getOptionNames` 方法中，可以看到返回的 `Set` 集合是通过 `Collections.unmodifiableSet` 包装过的 `Set` 集合的不可修改视图。`Collections.unmodifiableSet` 允许模块向用户提供对内部集合的“只读”访问权限。对返回集合的查询操作将“穿透”到指定的集合，而尝试修改返回的集合（无论是直接修改还是通过其迭代器）都将导致`UnsupportedOperationException` 异常。如果指定的集合是可序列化的，那么返回的集合也将是可序列化的。

同理，在 `getOptionValues` 方法中的 `Collections.unmodifiableList` 方法返回的是 `List` 集合的不可修改视图。

### 2.3 SimpleCommandLinePropertySource

上述 2.2.3 中，我们可以看到最终成员方法的处理都是来自 `SimpleCommandLinePropertySource` 类中的实现方法。

我们来看看相关源码：

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/88d883f30e7849979b8764feb16d8461~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=inNKvXQksFCfnZweEA2rYGdqZDU%3D)

`SimpleCommandLinePropertySource` 继承自 `CommandLinePropertySource<CommandLineArgs>`；

`CommandLinePropertySource<T>` 是一个抽象基类，用于实现由命令行参数支持的`PropertySource`。泛型类型 `T` 代表命令行选项的底层来源。

在 `SimpleCommandLinePropertySource` 中，`T` 是 `CommandLineArgs`，它是命令行参数的简单表示，分为 **"选项参数"** 和 **"非选项参数"**。

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/c9335b7e55224c8a875fcb03fb2c6180~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=9x2srHtbzadNHnbwoKS%2FcqJnQjk%3D)

继续看 `SimpleCommandLinePropertySource` 的构造方法，可以看到 命令行参数 `CommandLineArgs` 是 通过 `SimpleCommandLineArgsParser` 的 `parse(String... args)` 来解析的。

```java
new SimpleCommandLineArgsParser().parse(args);

```

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/704af5a4e895478c88c1ef16eb075f71~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=JjdB13DuXvov5LY8eQ8btS3g2v8%3D)

### 2.4 应用场景

有关 `ApplicationArguments` 的应用场景，我们一步步跟着源码来看：

#### 2.4.1 准备和配置应用环境

首先是 `run` 方法中，先创建一个 `DefaultApplicationArguments` 对象，并赋值给 `applicationArguments` 变量；

接着调用 `prepareEnvironment` 方法来准备和配置应用环境，并传入 `applicationArguments` 参数；

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/3f28dd27f62c413cac6016eebd79eacf~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=NZZCF0ooC3oKw5FyN%2BpOXIrvgvU%3D)

进入 `prepareEnvironment` 方法，可以看到如下：

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/d39a71a0b30d4c518561893ff6f29378~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=EEMD0SjDVgnsvNKIC%2BRRaT7llBg%3D)

这里是获取了原始的参数数组，并作为入参，传入 `configureEnvironment` 方法中；

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/6affde08b74b443092d8ddd65c63b9bf~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=4GDNXiisbJ1%2Fd8opt7hQpVlqCwI%3D)

进入 `configurePropertySources` 方法，它是用来配置应用程序的环境属性源，可以看到最终 args 参数都会被构建成 `SimpleCommandLinePropertySource` 的属性源。

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/03ef12bb0e8a40e0a0e24d24070a6f8e~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=gLpR91c%2BmyaKHKFVVtgVe5CjHTU%3D)

至于 `configureProfiles` 方法，目前是空实现，留着未来扩展。

#### 2.4.2 准备和配置应用上下文

调用 `prepareContext` 方法准备和配置应用上下文，并传入 `applicationArguments` 参数；

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/3ed7bb38a7b840b7b95325b710af7b46~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=0GemZ1VxwxTNnT9BpV4oRKMdjCk%3D)

进入 `prepareContext` 方法，可以看到：

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/68d08cee33f34751a3a650b05439ee82~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=1Q%2FkvzoCOdl6u1X1ckLExXfmXto%3D)

如上标红处，将 `applicationArguments` 注册为单例的 **Bean** 实例，其名称为 `springApplicationArguments`，后续其他地方需要时，就可以通过该名称从 **Spring** 容器中获取 `applicationArguments`。

#### 2.4.3 刷新应用上下文之后【afterRefresh方法】

`afterRefresh` 方法的实现默认为空，可由开发人员自行扩展。

#### 2.4.4 调用运行器【callRunners方法】

`callRunners` 方法里，主要是调用 `ApplicationRunner` 和 `CommandLineRunner` 的运行方法。

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/2d98239f9eac4ded8334c5d30c45a84b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=XYgqMWR%2Fa8GWGbeNKx%2BViU6hSpo%3D)
 ![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/78f7615a79a84587b00da17739e88e54~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=9EB4uSf1Z%2BBdTq7%2FctaFV%2B13TLs%3D)
 ![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/a1dc72dba0ae42cf80facfc339287d3f~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSHVhemll:q75.awebp?rk3s=f64ab15b&x-expires=1728541599&x-signature=Ew7XtS5O%2F06lT8tAaywVznR%2F9ZA%3D)

三、总结
----

本篇博文 **Huazie** 同大家一起深入分析了 `ApplicationArguments` 接口及其默认实现，相信这些可以进一步加深大家对于 **Spring Boot** 启动运行阶段中命令参数获取和使用的理解。接下来的博文将会继续聚焦 **Spring Boot** 启动运行阶段，敬请期待！！！