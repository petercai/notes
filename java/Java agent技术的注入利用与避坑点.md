# Java agent技术的注入利用与避坑点
什么是Java agent技术？
----------------

Java代理（Java agent）是一种Java技术，它允许开发人员在运行时以某种方式修改或增强Java应用程序的行为。Java代理通过在Java虚拟机（JVM）启动时以"代理"（agent）的形式加载到JVM中，以监视、修改或甚至完全改变目标应用程序的行为。

Java agent 可以做什么？
-----------------

1.  **安全监控和审计：** 
    
    > 通过Java代理，可以在应用程序中注入代码以监视其行为并记录关键事件。这可以用于安全审计目的，以确保应用程序不受到恶意行为或违规操作的影响。
    
2.  **安全验证和授权：** 
    
    > Java代理可以拦截对受保护资源的访问，并执行安全验证和授权操作。通过代理，可以实现访问控制策略，确保只有经过授权的用户或系统可以访问特定资源。
    
3.  **安全加固：** 
    
    > 通过Java代理，可以对应用程序进行安全加固，例如实时检测和防御攻击，包括代码注入、SQL注入、跨站点脚本攻击等。代理可以拦截请求，并根据安全策略进行处理，从而提高应用程序的安全性。
    
4.  **加密和解密：** 
    
    > Java代理可以用于实现端到端的数据加密和解密，保护敏感数据在传输过程中的安全性。代理可以拦截数据流，对数据进行加密或解密操作，以确保数据在传输过程中不会被窃取或篡改。
    
5.  **安全日志记录：** 
    
    > Java代理可以用于记录应用程序的安全日志，包括用户操作、异常事件、安全警报等。通过代理，可以将安全日志发送到中央日志服务器进行集中管理和分析，以便及时发现和应对安全威胁。
    

静态Agent使用
---------

创建Maven项目，写一个类PreMainTraceAgent，使用Maven编译并打成jar包。

```java
package com.example;​import java.lang.instrument.ClassFileTransformer;import java.lang.instrument.IllegalClassFormatException;import java.lang.instrument.Instrumentation;import java.security.ProtectionDomain;​public class PreMainTraceAgent {​public static void premain(String agentArgs, Instrumentation inst) {System.out.println("agentArgs : " + agentArgs);inst.addTransformer(new DefineTransformer(), true);}​static class DefineTransformer implements ClassFileTransformer {static int counts=0;@Overridepublic byte[] transform(ClassLoader loader,String className,Class<?> classBeingRedefined,ProtectionDomain protectionDomain,byte[] classfileBuffer) throws IllegalClassFormatException {System.out.println("premain load Class:" + className);System.out.println("filter "+(counts++)+" class");return classfileBuffer;}}}

```

打成jar包之后我们要注意META-INF目录下的MSNIFEST.MF文件，**MANIFEST.MF**文件是 Java 归档文件（如 JAR文件）的一部分，用于描述归档文件的元数据信息和配置。它通常位于归档文件的根目录下。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d060c0a21d04ba2b010967041f4fab2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=454&h=110&s=4484&e=png&b=ffffff)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bcfa832919a49d6a07f8f4ac9b4c1f6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=367&h=99&s=5529&e=png&b=ffffff)

一些常见的属性我们需要了解

1.  Manifest-Version: 描述了 MANIFEST.MF 文件的版本。
    
2.  Created-By: 描述了创建该归档文件的工具名称和版本。
    
3.  Main-Class: 描述了可执行 JAR 文件的入口类（Main类），当您执行 JAR
    
    > 文件时，Java虚拟机会自动寻找并执行该类中的main方法。
    
4.  Class-Path: 描述了归档文件中包含的依赖项 JAR 文件的路径，以便 Java
    
    > 虚拟机在运行时能够找到并加载这些依赖项。
    

在构建和部署 Java 应用程序时，MANIFEST.MF文件可以帮助指定各种元数据信息，使得应用程序可以更好地被管理和执行。例如，当您创建一个可执行的JAR 文件时，通过指定 **Main-Class** 属性，可以告诉 Java 虚拟机该 JAR文件的入口点是哪个类。

另外创建一个项目，写一个主函数，内容随意，配置**虚拟机选项**。这里-javaagent:后面跟上上面项目jar包的**绝对路径**。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38b58fe10a144c8bad4f7d75f89418c9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=854&s=149593&e=png&b=252a2f)

运行结果如图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dbf9b78621e4a669a638fa2b62a4a0f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=482&s=179281&e=png&b=252a2f)

可以看到premain方法中的代码成功的执行在了Main函数之前。这种使用premain方法在Main函数前执行的也被成为**静态agent**

帮助网安学习，全套资料S信领取：

① 网安学习成长路径思维导图  
② 60+网安经典常用工具包  
③ 100+SRC漏洞分析报告  
④ 150+网安攻防实战技术电子书  
⑤ 最权威CISSP 认证考试指南+题库  
⑥ 超1800页CTF实战技巧手册  
⑦ 最新网安大厂面试题合集（含答案）  
⑧ APP客户端安全检测指南（安卓+IOS）

动态Agent使用
---------

首先是被代理部分（单独的项目）

```java
package com.example;​import java.lang.instrument.ClassFileTransformer;import java.lang.instrument.IllegalClassFormatException;import java.lang.instrument.Instrumentation;import java.security.ProtectionDomain;​public class AgentMain {public static void agentmain(String agentArgs, Instrumentationinstrumentation) {instrumentation.addTransformer(new MyTransformer(),true);​}public static class MyTransformer implements ClassFileTransformer {static int count = 0;​@Overridepublic byte[] transform(ClassLoader loader,String className,Class<?> classBeingRedefined,ProtectionDomain protectionDomain,byte[] classfileBuffer) throws IllegalClassFormatException {System.out.println("hello world");

```

接下来就是使用Maven打成jar包

默认情况下META-INFMANIFEST.MF文件中有这些内容

Manifest-Version: 1.0

Created-By: Maven JAR Plugin 3.3.0

Build-Jdk-Spec: 11

但是这些是不够的，我们需要指出被代理的类。

Manifest-Version: 1.0

Agent-Class: com.example.AgentMain

Can-Redefine-Classes: true

Can-Retransform-Classes: true

1.  **Agent-Class**：指定了代理的入口类。这个属性告诉 Java
    
    > 虚拟机代理应该从哪个类的 **premain** 或 **agentmain**方法开始执行。**premain** 方法用于静态代理（在 JVM启动时加载），而 **agentmain** 方法用于动态代理（在 JVM运行时加载）。代理的入口类必须包含其中一个方法。
    
2.  **Can-Redefine-Classes**：指定了代理是否可以重新定义类。如果设置为
    
    > **true**，代理将允许重新定义已经加载的类，这意味着你可以修改已经加载的类的字节码。这对于某些代理操作，如热代码替换，非常有用。
    
3.  **Can-Retransform-Classes**：指定了代理是否可以重新转换类。如果设置为
    
    > **true**，代理将允许重新转换已经加载的类，这意味着你可以多次修改已经加载的类的字节码。这对于一些特定的代理操作也是非常有用的，如AOP（面向切面编程）。
    

因为是动态加载所以我们不需要在虚拟机启动选项中指定jar包的路径。

接下来写主程序的测试类

```java
package org.example;​import com.sun.tools.attach.VirtualMachine;​import java.io.File;import java.lang.management.ManagementFactory;​public class TestMain {public static void main(String[] args) {String agentJarPath ="C:Users86186DesktopstudyJavauntitledtargetuntitled-1.0-SNAPSHOT.jar";File agentJarFile = new File(agentJarPath);if (!agentJarFile.exists()) {System.err.println("Agent JAR file not found.");return;}String name = ManagementFactory.getRuntimeMXBean().getName();String pid = name.split("@")[0];​if (pid == null) {System.err.println("Unable to find process ID.");return;}String targetClassName = "AgentMain";try {VirtualMachine vm = VirtualMachine.attach(pid);vm.loadAgent(agentJarPath,targetClassName);vm.detach();} catch (Exception e) {e.printStackTrace();}}​}

```

这里在获取进程号的时候会因为版本的不同而出现错误，java9以下默认是正常的，java9以上会出现报错，我们需要在虚拟机启动参数中加上-Djdk.attach.allowAttachSelf=true。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b64ed78408844bb97ca62ddeb2185cc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=992&s=172071&e=png&b=303540)

运行结果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de8ec63bd6904deea44080631a29d72e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=421&s=67015&e=png&b=272d35)

为什么结果中有多个helloworld
-------------------

这里有讲一下为什么我们在代码中之用了一次sout，但是在结果中却出现了多个helloworld。

**MyTransformer**类中的**transform**方法中的输出语句只会在类被加载时执行一次，但是它会对每个类文件调用一次。由于一个类可能会由多个ClassLoader加载，或者同一个ClassLoader可能会加载多次，因此会导致多次输出。

这种情况通常在Java应用程序中使用了多个ClassLoader时发生，例如Web应用程序中的热部署或者OSGi环境中。每次类被加载，**transform**方法都会被调用一次，因此会看到多次输出。

我们可以修改一下代码做测试，这里我在每个helloworld后添加了被加载类的名字

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a133f38f91840d68e57c7cb49f25eac~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=230&s=63251&e=png&b=262b33)

修改后的输出结果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70423ea035104db1a35164cd91b60b90~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=405&s=130814&e=png&b=282d36)

实战示例：修改目标虚拟机中执行的程序
------------------

### 第一步

首先我们写出我们正在执行的程序：循环打印helloworld。

```java
package org.example;​import static java.lang.Thread.sleep;​public class Main {public static void main(String[] args) throws InterruptedException {while(true) {hello();sleep(1500);}}public static void hello(){System.out.println("Hello World!");}}

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c05dcb65ea4489c8fffa5040bf31301~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=745&s=97329&e=png&b=272d35)

### 第二步

准备我们的agentmain和ClassFileTransformer实现类。

```vbnet
package com.example;​import java.lang.instrument.ClassFileTransformer;import java.lang.instrument.IllegalClassFormatException;import java.lang.instrument.Instrumentation;import java.lang.instrument.UnmodifiableClassException;import java.security.ProtectionDomain;​public class AgentMain {public static void agentmain(String agentArgs, Instrumentationinstrumentation) throws UnmodifiableClassException {Class [] classes = instrumentation.getAllLoadedClasses();​//获取目标JVM加载的全部类for(Class cls : classes){​if (cls.getName().equals("org.example.Main")){​instrumentation.addTransformer(new HackTransform(),true);instrumentation.retransformClasses(cls);}// System.out.println(cls.getName());}​}​}​package com.example;​import javassist.ClassClassPath;import javassist.ClassPool;import javassist.CtClass;import javassist.CtMethod;​import java.io.IOException;import java.lang.instrument.ClassFileTransformer;import java.lang.instrument.IllegalClassFormatException;import java.security.ProtectionDomain;​public class HackTransform implements ClassFileTransformer {​@Overridepublic byte[] transform(ClassLoader loader, String className,Class<?> classBeingRedefined, ProtectionDomain protectionDomain,byte[] classfileBuffer) throws IllegalClassFormatException {if (className.equals("org/example/Main")) {try {System.out.println(className);​ClassPool classPool = ClassPool.getDefault();​​if (classBeingRedefined != null) {ClassClassPath ccp = new ClassClassPath(classBeingRedefined);classPool.insertClassPath(ccp);}​CtClass ctClass = classPool.get("org.example.Main");System.out.println(ctClass);​​CtMethod ctMethod = ctClass.getDeclaredMethod("hello");​//设置方法体String body = "{System.out.println("[+]Hacker!！");}";ctMethod.setBody(body);ctClass.defrost();​return ctClass.toBytecode();​} catch (Exception e) {e.printStackTrace();}}​return null;}}

```

### 第三步

把第二步中的两个类打成jar包。并修改其中MANIFEST.MF中的内容。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e5391095377444c99dccfc1a024f6d6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=595&s=182194&e=png&b=2b3039)

MANIFEST.MF中的内容

**Manifest-Version: 1.0Agent-Class:com.example.AgentMainCan-Redefine-Classes: trueCan-Retransform-Classes:true**

### 第四步

写我们的注入代码

```java
package org.example;​import com.sun.tools.attach.*;​import java.io.IOException;import java.util.List;​public class inject {​public static void main(String[] args) throws IOException,AttachNotSupportedException, AgentLoadException,AgentInitializationException {

```

### 第五步

执行即可（先运行主java程序，后运行注入程序）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1121c83fdedb47cab3bc93c762c18222~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=322&s=42943&e=png&b=292e37)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4334efa5e32648f4921eea3bea48d1cf~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=365&s=112431&e=png&b=292e37)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37bedbee00464d78811ddf5672f0fc35~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=350&s=71320&e=png&b=292e37)