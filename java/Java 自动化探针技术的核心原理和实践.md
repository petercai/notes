# Java 自动化探针技术的核心原理和实践
JVMTI 技术
--------

JVM 在设计之初，就考虑到了虚拟机状态的监控、程序 Debug、线程和内存分析等功能。

在 JDK1.5 之前，JVM 规范就定义了 JVMPI（Java Virtual Machine Profiler Interface）也就是 JVM 分析接口以及 JVMDI(Java Virtual Machine Debug Interface)也就是 JVM 调试接口，JDK1.5 以及以后的版本，这两套接口合并成了一套，也就是 Java Virtual Machine Tool Interface，就是 JVMTI 。通过 JVMTI 可以探查 JVM 内部的一些运行状态，甚至控制 JVM 应用程序的执行。

JVMTI 大体声明支持以下功能：

1\. Java Heap 与 GC

获取所有类的信息，对象信息，对象引用关系，Full GC 开始/结束，对象回收事件等。

2\. 线程与堆栈

获取所有线程的信息，线程组信息，控制线程（start,suspend,resume,interrupt…）,

Thread Monitor(Lock)，得到线程堆栈，控制出栈，方法强制返回，方法栈本地变量等。

3\. Class & Object & Method & Field 元信息

Class 信息，符号表，方法表，fields 信息，Method 信息等，Object 信息。

redefine class（hotswap）, retransform class 这里看到 JVMTI 设计强大地方，他可以重新定义类对象！后面会聊到。

4\. 工具类

线程 CPU 消耗，ClassLoader 路径修改，系统属性获取等。

这里需要注意的是：

*   JVMTI 是一套 JVM 的接口规范，不同的 JVM 实现方式可以不同，有的 JVM 提供了拓展性的功能，比如 openJ9，当然也可能存在 JVM 不提供这个接口的实现。
    
*   JVMTI 提供的是 Native 方式调用的 API，也就是常说的 JNI 方式。JVMTI 接口用 C/C++的语言提供，最终以动态链接库的形式由 JVM 加载并运行
    

使用 JNI 方式调用 JVMTI 接口访问目标虚拟机的大体流程图如下：

![](_assets/35961ac0e257ecd62409a5390b01639d.png)

其中，jvmti.h 头文件中定义了 JVMTI 接口提供的方法，我们简单看看 JDK 7 里面源代码一些重要方法：

> [https://github.com/openjdk-mirror/jdk7u-jdk/blob/master/src/share/javavm/export/jvmti.h](https://github.com/openjdk-mirror/jdk7u-jdk/blob/master/src/share/javavm/export/jvmti.h)  

```
jvmtiError (JNICALL *GetMethodName) (jvmtiEnv* env,
    jmethodID method,
    char** name_ptr,
    char** signature_ptr,
    char** generic_ptr);


  
  jvmtiError (JNICALL *GetMethodDeclaringClass) (jvmtiEnv* env,
    jmethodID method,
    jclass* declaring_class_ptr);


  
  jvmtiError (JNICALL *GetMethodModifiers) (jvmtiEnv* env,
    jmethodID method,
    jint* modifiers_ptr);
```

JVMTI API 里序号 64、65 方法是从 JVM 中获取程序定义所有方法和相关类。66 比较有意思了，它获取修改后的方法，这说明借助 JVMTI 可以动态修改 Java 程序的方法。

```
jvmtiError (JNICALL *RetransformClasses) (jvmtiEnv* env,
    jint class_count,
    const jclass* classes);
```

序号 152 是一个非常重要的方法，JVMTI 的 `RetransformClasses`函数来完成类的重定义过程，这也说明 JVMTI 可以修改 Java 程序整个类。

```
jvmtiError JvmtiEnv::RedefineClasses(jint class_count, const jvmtiClassDefinition* class_definitions) {

  VM_RedefineClasses op(class_count, class_definitions, jvmti_class_load_kind_redefine);
  VMThread::execute(&op);
  return (op.check_error());
}
```

JVMTI 服务有两种打开方式：

*   在 Java 进程启动的时候通过 `-agentpath:<path-to-agent>=<options>`方式启动，`path-to-agent` 对应的 JVMTI 接口实现的动态库文件的绝对路径，后面可以追加 JVMTI 程序需要的参数。Linux 动态库文件的后缀为 `.so`；
    
*   运行时挂载 Attach ，然后加载 JVMTI 接口实现的动态库文件。
    

JVMTI 的 Agent、Attach 用 C、C++编写，具体感兴趣想实践的朋友可以看看下面的例子：[https://github.com/liuzhengyang/jvmti_examples](https://github.com/liuzhengyang/jvmti_examples "xxx")  

用 C、C++ 实现 JVMTI 功能对大部分 Java 工程师的确强人所难。于是，Sun 公司出了 [Java Agent](https://xie.infoq.cn/article/a3814f9326781409a05edc23d "xxx")，一个用 Java 实现 JVMTI 的方案，方案相当优雅和容易上手。

Java Agent 技术由来
---------------

Java Agent 直译为 Java 代理，中文圈也流行另外一个称呼 Java 探针 `Probe` 技术。

它在 JDK1.5 引入，是一种可以动态修改 Java 字节码的技术。Java 类编译后形成字节码被 JVM 执行，在 JVM 在执行这些字节码之前获取这些字节码的信息，并且通过字节码转换器

`ClassFileTransformer` 对这些字节码进行修改，以此来完成一些额外的功能。

Java Agent 是一个不能独立运行 jar 包，它通过依附于目标程序的 JVM 进程，进行工作。

```
java -javaagent:myagent.jar=mode=test Test
```

Agent 启动拦截提供以下两种方式。

一种是程序运行前：在`Main`方法执行之前，通过一个叫 `premain`方法来执行

启动时需要在目标程序的启动参数中添加 `-javaagent`参数，Java Agent 内部通过注册 ClassFileTransformer ，这个转化器在 Java 程序 `Main`方法前加了一层拦截器。在类加载之前，完成对字节码修改。

![](_assets/6cc4e0b481ec7a2c0527111f6526a778.png)

Premain 完整工作流程图

另一种是程序运行中修改，需通过 JVM 中 Attach 技术实现，Attach 的实现也是基于 JVMTI

总结下，Java Agent 具备以下的能力：

*   Java Agent 能够在加载 Java 字节码之前拦截并对字节码进行修改;
    
*   Java Agent 能够在 Jvm 运行期间修改已经加载的字节码。
    

### Java Agent 的价值

Java Agent 有着成熟的技术架构和对字节码通用的重写能力。它应用场景非常广泛：

*   IDE 的调试功能，例如 Eclipse、IntelliJ IDEA；
    
*   热部署功能，例如 JRebel、XRebel、spring-loaded；
    
*   各种线上诊断工具，例如 Btrace、Greys，国内阿里的 Arthas；
    
*   各种性能分析工具，例如 Visual VM、JConsole 等；
    
*   全链路性能检测工具，例如 OpenTelemetry、Skywalking、Pinpoint 等。
    

接下来，我们用案例实现性能检测工具的 Java Agent 探针。

### Java Agent 和 JVMTI 关系

我们大致了解下 Java Agent 底层源代码实现过程：

![](_assets/3165f240fe60107db60db50b7ae4d2a6.png)

首先弄清几个概念。

#### JVMTIAgent

JVMTIAgent 是一个动态库，它可以利用 JVMTI 暴露出的一些接口来实现一些特殊的功能。我们常用 Eclipse、Idea 等 IDE 工具代码调试就是利用它。Java Agent 也是利用了其中一个 JVMTIAgent 来实现的。在 Linux 里面，这个库叫做`libinstrument.so`，在 BSD 系统中叫做`libinstrument.dylib`，该动态链接库在`{JAVA_HOME}/jre/lib/`目录下。因为源代码里面`add_init_agent`函数里面传递进去的是一个叫做 `instrument`的字符串，所以也称它为 instrument 动态库，对应启动 Agent 称为 Instrument Agent。

#### Instrument 动态链接库

Instrument 支持使用 Java Instrumentation API 来编写 Java Agent。

Java Instrumentation : 在 Jdk1.5 后，Java 语言中提供的调用动态库的 Java API 接口 。

在 Instrument 中有一个非常重要的类称为：`JPLISAgent`（Java Programming Language Instrumentation Services Agent），它的作用是初始化所有通过 Java Instrumentation API 编写的 Agent。很容易猜到，Java Instrumentation API 其实就是底层调用 JVMTI。

JVMTIAgent 包含这个几个基本函数：

```
JNIEXPORT jint JNICALL
Agent_OnLoad(JavaVM *vm, char *options, void *reserved);
 
JNIEXPORT jint JNICALL
Agent_OnAttach(JavaVM* vm, char* options, void* reserved);


JNIEXPORT void JNICALL
Agent_OnUnload(JavaVM *vm);
```

*   `Agent_OnLoad`如果 Agent 是在目标 JVM 启动时加载(通过 VM 参数 `-agentpath:<path-to-agent>=<options>`方式)，在启动过程中会去执行 Agent 里`Agent_OnLoad`函数；
    
*   `Agent_OnAttach`如果 Agent 通过 Attach 方法启动，执行 Attach 的 JVM 会给目标 JVM 进程发送 Load 命令来加载 Agent，在加载过程中调用就是`Agent_OnAttach`函数；
    
*   `Agent_OnUnload`在 Agent 做卸载的时候调用。
    

Instrument 实现了`Agent_OnLoad`和`Agent_OnAttach` 两方法，所以 Java Agent 既可以在 JVM 启动时，也就是加载 Java 字节码之前启动，也可以在 JVM 运行时启动，这很有价值。

![](_assets/c4d707f9c2da6a4bd7bb7e786f902edc.png)

大致画了一下 Java Agent 和 JVMTI 的关系

### Java Instrument Package

上面提到实现 Agent 需要 Instrument 动态链接库支持。Java 语言中也提供了调用动态库的 Java API 接口 Instrumentation。有了 Instrumentation，开发者可以轻松使用 Java 语言操作字节码，来实现 Java Agent 相关功能，Instrument Package 大致结构：

```
java.lang.instrument.*
java.lang.instrument.Instrumentation;
public interface Instrumentation {}
```

#### Instrumentation 工作原理

SUN 工具包(sun.instrument.InstrumentationImpl)编写了一些 Native 方法，JDK 里提供了这些 Native 方法的实现类`(jdk\src\share\instrument\JPLISAgent.c)`，通过 JNI 方式访问 JVMTI 提供的方法，这些方法就是定义在`jvmti.h`头文件中。

#### Instrumentation 核心功能

Instrumentation 接口有一个最重要方法 `addTransformer`，它用于添加多个`ClassFileTransformer`。类似下面 Java Agent 实现的例子：

![](_assets/25a7b706bc822f82c8bcf1b25e6e6924.png)

`ClassFileTransformer` 中文类转换器，`ClassFileTransformer`提供了`tranform()`方法，用于对加载的类进行增强重定义，返回新的类字节码流。

Instrumentation 有一个 TransformerInfo 数组保存`ClassFileTransformer`，像拦截器链表一样，顺序的进行字节码的重定义。

```
void Instrumentation.addTransformer(ClassFileTransformer transformer, boolean canRetransform)









byte[] ClassFileTransformer.transform(ClassLoader loader, String className, Class classBeingRedefined, ProtectionDomain protectionDomain, byte classfileBuffer)
```

下面我们写 Java Agent 时候，就会实现这两个重要的方法。

通过 Instrument API 方式使用到了 JVMTI 提供部分功能，对开发者来说，主要提供的是对 JVM 加载的类字节码进行增强操作，等价于有了全局、动态修改 Java 程序代码的能力。

### 总结

到这里，是不是想到：我们是否可以通过 Agent 在所有类方法中插入额外的字节码，这些字节码功能就是获取程序内部数据，然后上报给某个地方。是的，我们很多通用的 Java 监控、Debug、日志记录工具就是基于它来实现的。而且，插入的字节码是附加的，这些更变不会修改原来程序正常逻辑和状态，只是会有一些性能损耗，对应用程序本身基本是安全可靠的。

#### JVMTI 方式 和 Java Instrument 对比

|  | JVMTI方式 | Java Instrument API |
| --- | --- | --- |
| 性能 | 独立进程，不受目标JVM影响 | 在目标JVM内，GC时会受到影响 |
| 功能性 | 方法众多，功能非常全面 | 字节码的操作 |
| 易用性 | 需要掌握C/C++，以及JNI开发相关知识 | Java代码开发，上手快<br> |

Agent 启动方式
----------

### 程序运行前加载

目标 JVM 启动时指定`-javaagent:xxx.jar`参数来启动 Java Agent, 这里 xxx.jar 是探针的 JAR 包. 比如 OpenTelemetry 运行 Java 探针的指令：

```
java -javaagent:path/to/opentelemetry-javaagent.jar \
     -jar myapp.jar
```

程序启动时，优先加载 Java Agent，执行里面的 `premain`方法。这个时候，其实大部分的类没有被加载。

#### Jar 打包规则

探针的 JAR 包需要做以下配置。

JAR 包里`MANIFEST.MF`文件添加一个`Premain-Class`属性，指定一个实现了`premain`方法的类，加入 Can-Redefine-Classes 和 Can-Retransform-Classes 选项

`MANIFEST.MF` 大致如下配置：

```
PreMain-Class: AgentMain
Can-Redefine-Classes: true
Can-Retransform-Classes: true
```

`premain` 方法声明：

```
public static void premain(String agentArgs, Instrumentation inst);
public static void premain(String agentArgs);
```

下面是我们实现`premain` 方法的一个测试类：

![](_assets/72894bb92001a165bacd66ca72c5df21.png)

### Premain 方法工作原理

目标 JVM 启动时，运行 JNI 的 Agent_OnLoad 函数，执行如下步骤：

*   创建 InstrumentationImpl 对象；
    
*   监听 ClassFileLoadHook 事件；
    
*   调用 InstrumentationImpl 的 loadClassAndCallPremain 方法，此方法里去 Agent Jar 包中找到`MANIFEST.MF`声明的 Premain-Class 类，执行类里面的 `premain`方法；
    
*   `premain`方法里面，我们可以调用 Java Instrumentation API 完成字节码增强功能。
    

![](_assets/072ee6032330d6cf9bb2086dbda6b37b.png)

### 复习 Java Byte-code 字节码概念

维基百科字节码中文解释：

> 字节码（英语：Bytecode）通常指的是已经经过编译，但与特定机器代码无关，需要解释器转译后才能成为机器代码的中间代码。字节码通常不像源码一样可以让人阅读，而是编码后的数值常量、引用、指令等构成的序列。
> 
> 字节码主要为了实现特定软件运行和软件环境、与硬件环境无关。字节码的实现方式是通过编译器和虚拟机。编译器将源码编译成字节码，特定平台上的虚拟机将字节码转译为可以直接执行的指令。字节码的典型应用为[Java bytecode](https://zh.wikipedia.org/zh-cn/Java_bytecode)。

![](_assets/bad56c59c8bd3d856b4bc9b40a3d6140.png)

Java 程序运行原理

Java Byte-code: Java 语言写出的源代码首先需要编译成 class 文件，即字节码文件，然后被 JVM 加载并运行，每个 class 文件具有如下固定的数据格式：

```
ClassFile {
    u4             magic;           
    u2             minor_version;   
    u2             major_version;   
    u2             constant_pool_count;                     
    cp_info        constant_pool[constant_pool_count-1];    
    u2             access_flags;    
    u2             this_class;      
    u2             super_class;     
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

class 文件总是一个魔数开头，后面跟着版本号，然后就是常量定义、访问标志、类索引、父类索引、接口个数和索引表、字段个数和索引表、方法个数和索引表、属性个数和索引表。

### 字节码增强技术

Agent 本质是通过操作字节码，动态修改运行时 Java 对象。

我们把一类对现有字节码进行修改或者动态生成全新字节码文件的技术叫做字节码增强技术。

字节码增强技术的实现有很多方式，简单整理下目前比较成熟的一些操作字节码的框架。

![](_assets/792d4f67257afe73ebc78044e9cd2213.png)

*   `JDK动态代理`运行期动态的创建代理类，只支持接口；
    
*   `ASM`一个 Java 字节码操控框架。它能够以二进制形式修改已有类或者动态生成类。不过 ASM 在创建 class 字节码的过程中，操纵的级别是底层 JVM 的汇编指令级别，这要求 ASM 使用者要对 class 组织结构和 JVM 汇编指令有一定的了解；
    
*   `Javassist`一个开源的分析、编辑和创建 Java 字节码的类库（源码级别的类库）。Javassist 是 Jboss 的一个子项目，其主要的优点，在于简单，而且快速。直接使用 Java 编码的形式，而不需要了解虚拟机指令，就能动态改变类的结构，或者动态生成类；
    
*   `Byte Buddy`是一个较高层级的抽象的字节码操作工具，相较于 ASM 而言。Byte Buddy 本身也是基于 ASM API 实现的。Byte Buddy 以出色的性能，被著名的框架和工具（例如 Mockito，Hibernate，Jackson，Google 的 Bazel 构建系统等）使用。
    

#### ASM

[ASM](https://xie.infoq.cn/article/e799c87255186c4c5626749c3 "xxx") 可以直接产生二进制`.class`文件，也可以在类被加载入 Java 虚拟机之前动态改变类行为（也就是生成的代码可以覆盖原来的类也可以是原始类的子类）。ASM 从类文件中读入信息后，能够改变类行为，分析类信息，甚至能够根据用户要求生成新类。

不过 ASM 在创建 class 字节码的过程中，操纵的级别是底层 JVM 的汇编指令级别，这要求 ASM 使用者要对 class 组织结构和 JVM 汇编指令有一定的了解。ASM 提供了两组 API: Core API 和 Tree API，Core API 是基于访问者模式来操作类的，而 Tree 是基于树节点来操作类的。

简单写一个 ASM 运行例子：

```
public class ASMDemo extends ClassLoader{
    public static <T> T getProxy(Class clazz) throws Exception {

        ClassReader classReader = new ClassReader(clazz.getName());
        ClassWriter classWriter = new ClassWriter(classReader, ClassWriter.COMPUTE_MAXS);
        classReader.accept(new ClassVisitor(ASM5, classWriter) {
            @Override
            public MethodVisitor visitMethod(int access, final String name, String descriptor, String signature, String[] exceptions) {
                
                if (!"hi".equals(name))
                    return super.visitMethod(access, name, descriptor, signature, exceptions);
                final MethodVisitor methodVisitor = super.visitMethod(access, name, descriptor, signature, exceptions);
                return new AdviceAdapter(ASM5, methodVisitor, access, name, descriptor) {
                    @Override
                    protected void onMethodEnter() {
                        
                        methodVisitor.visitFieldInsn(Opcodes.GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
                        
                        methodVisitor.visitLdcInsn("方法名: "+name + "  你被代理了，By ASM！");
                        
                        methodVisitor.visitMethodInsn(Opcodes.INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
                        super.onMethodEnter();
                    }
                };
            }
        }, ClassReader.EXPAND_FRAMES);
        byte[] bytes = classWriter.toByteArray();
        return (T) new ASMDemo().defineClass(clazz.getName(), bytes, 0, bytes.length).newInstance();
    }
}
```

我们通过 ASM 动态代理一个简单的测试接口和实现类：

```
public class HelloImpl implements Hello{
    @Override
    public String hi(String msg) {
        return ("hello " + msg);
    }}
public interface Hello {
    public String hi(String msg);}
```

写一个测试程序，通过 ASM 代理模式，增强字节码后调用方法的效果：

![](_assets/64e251f0c7193e21d6ccbf621391a9ec.png)

当然，基于 ASM 开发门槛比较高一些，你必须了解一定汇编原理和指令。

#### Javassist

[Javassist](https://xie.infoq.cn/article/ed80658e424c76c68ef3adf0e "xxx")是一个开源的分析、编辑和创建 Java 字节码的类库。是由东京工业大学的数学和计算机科学系的 Shigeru Chiba （千叶滋）所创建的。它已加入了开放源代码 JBoss 应用服务器项目，通过使用 Javassist 对字节码操作为 JBoss 实现动态"AOP"框架。

Javassist 其主要的优点，在于简单，而且快速。直接使用 Java 编码的形式，而不需要了解虚拟机指令，就能动态改变类的结构，或者动态生成类。

参考文档：

*   [http://www.javassist.org/](http://www.javassist.org/ "xxx")  
    
*   [https://github.com/jboss-javassist/javassist](https://github.com/jboss-javassist/javassist "xxx")  
    

下面我们有一个完整 Java 探针实例用 Javassist 来实现。

#### Byte Buddy

[Byte Buddy](https://www.infoq.cn/article/2018/08/byte-buddy-java11 "xxx") 是致力于解决字节码操作和 Instrumentation API 的复杂性的开源框架。Byte Buddy 所声称的目标是将显式的字节码操作隐藏在一个类型安全的领域特定语言背后。通过使用 Byte Buddy，任何熟悉 Java 编程语言的人都容易地进行字节码操作。

官网的示例展现了如何生成一个简单的类，这个类是 Object 的子类，并且重写了 toString 方法，用来返回`“Hello World!”`与原始的 ASM 类似，intercept 会告诉 Byte Buddy 为拦截到的指令提供方法实现。

```
Class<?> dynamicType = new ByteBuddy()
  .subclass(Object.class)
  .method(ElementMatchers.named("toString"))
  .intercept(FixedValue.value("Hello World!"))
  .make()
  .load(getClass().getClassLoader())
  .getLoaded();
System.out.println(dynamicType.getSimpleName());
```

Demo 来源 bytebuddy.net

#### 字节码增强工具对比

| 框架 | ASM | Javassist | JDK Proxy | Cglib | ByteBuddy |
| --- | --- | --- | --- | --- | --- |
| 起源时间 | 2002 | 1999 | 2000 | 2011 | 2014 |
| 增强方式 | 字节码指令 | 字节码指令和源码（注：源码文本） | 源码 | 源码 | 源码 |
| 源码编译 | NA | 不支持 | 支持 | 支持 | 支持 |
| Agent支持 | 支持 | 支持 | 不支持，依赖框架 | 不支持，依赖框架 | 支持 |
| 性能 | 高 | 中 | 低 | 中 | 中 |
| 维护状态 | 是 | 是 | 停止升级 | 停止维护 | 活跃 |
| 优点 | 超高性能，应用场景广泛 | 同时支持字节码指令和源码两种增强方式 | JDK原生类库支持 |  | 零侵入，提供良好的API扩展编程 |
| 缺点 | 字节码指令对应用开发者不友好 |  | 场景非常局限，只适用于Java接口 | 已经不再维护，对于新版JDK17+支持不好，官网建议切换到ByteBuddy |  |
| 应用场景 | 小，高性能，广泛用于语言级别 |  |  | 广泛用于框架场景 | 广泛用于Trace场景 |

### 一个完整的 Java Agent 探针实现过程

#### 目标 

> 实现一个简单性能工具，通过探针统计 Java 程序所有方法的执行时间。

1\. 构建 Maven 项目工程，添加 MANIFEST.MF , 目录大致如下。

![](_assets/001e6aef7748821312744723afa07800.png)

在 MANIFEST.MF 文件中定义 Premain-Class 属性，指定一个实现类。类中实现了 Premain 方法，这就是 Java Agent 在类加载启动入口。

```
Manifest-Version: 1.0
Premain-Class: com.laziobird.MyAgentDemo
Agent-Class: com.laziobird.MyAgentDemo
Can-Redefine-Classes: true
Can-Retransform-Classes: true
```

*   `Premain-Class`包含`Premain`方法的类；
    
*   `Can-Redefine-Classes`为`true`时表示能够重新定义`Class`；
    
*   `Can-Retransform-Classes`为`true`时表示能够重新转换`Class`，实现字节码替换。
    

2\. 构建 Premain 方法。

```
public class MyAgentDemo {
    
    public static void premain(String args, Instrumentation inst) {
        System.out.println(" premain agent loaded !");
        inst.addTransformer(new PreMainTransformerDemo());
        System.out.println(" agent addTransformer start !");
    }
 ....   
}
```

我们实现 Premain 方法类叫 `MyAgentDemo`，里面添加一个类转化器 `PreMainTransformerDemo`，这个转化器具体来实现统计方法调用时间。

3\. 编写类转换器。

在编写类转化器时，我们通过 Javassist 来具体操作字节码，首先`pom.xml` 里面添加依赖：

```
<dependency>
   <groupId>org.javassist</groupId>
   <artifactId>javassist</artifactId>
   <version>3.25.0-GA</version>
</dependency>
```

接下来具体实现：

```
public class PreMainTransformerDemo implements ClassFileTransformer{
   final static String prefix = "\nlong startTime = System.currentTimeMillis();\n";
   final static String postfix = "\nlong endTime = System.currentTimeMillis();\n";
   @Override
   public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
                           ProtectionDomain protectionDomain, byte[] classfileBuffer){
       
       className = className.replace("/", ".");
       
       if(className.startsWith("java") || className.startsWith("sun")|| !className.contains("com.laziobird")){
           return null;
       }
       CtClass ctclass = null;
       try {
           
           ctclass = ClassPool.getDefault().get(className);
           for(CtMethod ctMethod : ctclass.getDeclaredMethods()){
               String methodName = ctMethod.getName();
               
               String newMethodName = methodName + "$old";
               
               ctMethod.setName(newMethodName);
               
               CtMethod newMethod = CtNewMethod.copy(ctMethod, methodName, ctclass, null);
               
               StringBuilder bodyStr = new StringBuilder();
               bodyStr.append("{");
               bodyStr.append("System.out.println(\"==============Enter Method: " + className + "." + methodName + " ==============\");");
               
               bodyStr.append(prefix);
               bodyStr.append(newMethodName + "($);\n");
               
               bodyStr.append(postfix);
               
               bodyStr.append("System.out.println(\"==============Exit Method: " + className + "." + methodName + " Cost:\" +(endTime - startTime) +\"ms " + "===\");");
               bodyStr.append("}");
               
               newMethod.setBody(bodyStr.toString());
               ctclass.addMethod(newMethod);
           }
           
           return ctclass.toBytecode();
       } catch (Exception e) {
           e.printStackTrace();
       }
       return null;
   }
```

这段程序等价于：把指定 Java 类下所有方法进行了如下转换，重新生成字节码加载执行。

![](_assets/8a90dbd42ece50a407fc530f3608c78a.png)

4\. 打包生成 Java Agent 的 Jar 包。

在`pom.xml`配置好`maven assembly`，进行编译打包。

![](_assets/05932c9f193e1b18849e69ebd45c09d3.png)

5\. 写一个 Java 测试程序，验证探针是否生效。

类`AgentTest` 有两个简单方法`test`、`testB`。

为了演示，其中 testB 调用了另外一个类`ClassC` 的 `methodD`方法。

可以看到，类包名是 `com.laziobird`，刚才的 Agent 只会对`com.laziobird` 的类起作用。

```
package com.laziobird;
public class AgentTest {
    public void test() {
        System.out.println("hello the method: agentTest.test ");
    }
    public void testB() {
        ClassC c = new ClassC();
        c.methodD();
        System.out.println("hello the method: agentTest.testB ");
    }
    public static void main(String[] args) {
        AgentTest agentTest = new AgentTest();
        agentTest.test();
        agentTest.testB();
    }
}
package com.laziobird;
public class ClassC {
    public void methodD(){
        try {
            System.out.println(" methodD start!");
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

我们给测试程序打成可执行的 Jar 包，Jar 指定默认运行的类是 `AgentTest` 。

![](_assets/b6e9c79d96981a8ba03cd2da415218a1.png)

运行测试程序，通过`-javaagent`启动我们写的 Java Agent 探针。

```
java -javaagent:/path/agentdemo/target/javaagent-demo-0.0.1-SNAPSHOT-jar-with-dependencies.jar  
-jar  /path/gitproject/TestAgentDemo/out/artifacts/TestAgentDemo_jar/TestAgentDemo.jar
```

#### 运行效果

![](_assets/21f0e57daf9b6ae6a58ede6bf04f8f5c.png)

程序运行时加载
-------

在 JDK1.6 版本中，Java Agent 支持了可以在 JVM 运行时动态修改 Java 字节码的能力。这种能力需要 JVM Attach 来实现。

JVM Attach：简单来说就是 JVM 提供一种 JVM 进程间通信的机制。它能让一个进程传命令给另外一个进程，并让它执行内部的一些操作。

常见场景，比如做故障定位时，有可能我们觉得某些 Java 的线程程序卡住了。于是想把一个 JVM 进程的线程 Dump 出来。那么我们会跑一个 JStack 的进程，然后传进程 Id 参数，告诉 Jstack 指定哪个进程进行线程 Dump。Attach 机制完成两个进程间如何通信和传输协议的定义。

### JVM Attach 实现原理

存在一个 Attach Listener 线程，监听其他 JVM 的[Attach](https://mp.weixin.qq.com/s?__biz=MzIzNjI1ODc2OA==&mid=2650886799&idx=1&sn=108c5fdfcd2695594d4f80ff02fc9a70&scene=27#wechat_redirect "xxx") 请求，其通信方式基于 socket，JVM Attach 机制底层从 Kernel 到 Application 层完整流程图。

![](_assets/442a193579a7cf08d2d011dcbb003d2d.png)

具体 C 语言源代码实现，可以参考李嘉鹏这篇深入分享：

[http://lovestblog.cn/blog/2014/06/18/jvm-attach/?spm=ata.13261165.0.0.26d52428n8NoAy](http://lovestblog.cn/blog/2014/06/18/jvm-attach/?spm=ata.13261165.0.0.26d52428n8NoAy)  

### Agentmain 工作原理

Java Agent 在运行时和启动时加载机制其实很像，主要区别在 Agent 进行字节码增强前，对于拦截入口不同而已。一个叫`Premain`，一个叫`Agentmain` 。 这一点很好理解：启动时，Agent 直接通过启动参数`-javaagent`吸附于当前 JVM 进程。运行时加载，其实当前 JVM 进程已经启动了。这时借助另一个 JVM 进程通信，调用 Attach API 再把 Agent 启动起来。后面的字节码修改和重加载的过程那就是一样的。

![](_assets/dd3f9c8351c1dfc6e3fa3a86dd8928f0.png)

#### 运行时 Java Agent 配置

和启动时代理类似：

*   JAR 包的`MANIFEST.MF`清单文件中定义`Agent-Class`属性，指定一个实现类。加入`Can-Redefine-Classes`和 `Can-Retransform-Classes` 选项；
    
*   JAR 包中包含清单文件中定义的这个类，类中包含`agentmain`方法，方法逻辑自己实现 JAR 包内对应的`MANIFEST.MF`有如下配置：
    

```
Agent-Class: com.laziobird.MyAgentDemo
Can-Redefine-Classes: true
Can-Retransform-Classes: true
```

需要注意：`agentmain`方式由于是采用 Attach 机制，被代理的目标程序 VM 已经先于 Agent 启动，其所有类已经被加载完成。这个时候需要执行 Instrumentation 的

`retransformClasses`方法让类进入重新转换，重定义的过程：它会激活类执行`ClassFileTransformer`列表中的回调，完成字节码操作，最后让类加载器重新加载。

### Attach Agentmain 和 PreMain 对比

1\. 字节码增强的限制。

虽然运行时 Agent 可以在 JVM 运行时动态的修改某个类的字节码，但是为了保证 JVM 的正常运行，新定义的类相较于原来的类需要满足：

*   父类是同一个
    
*   实现的接口数也要相同，并且是相同的接口
    
*   类访问符必须一致
    
*   字段数和字段名要一致
    
*   新增或删除的方法必须是 private static/final 的
    
*   可以修改方法内部代码
    

2\. 显式调用重定义方法。

因为 JVM 启动时，字节码 Class 文件已经提前生成好再进行 Class Load 过程。但是 JVM 运行后，把已经加载后的类动态修改 `redefine an already loaded class`，我们修改完类定义后，显式在内存中进行重加载 `reload Class`。

Java Agent 显式给我们提供了`retransformClasses`方法，下面我摘取它的详细说明。

```
void retransformClasses(Class<?>... classes) {}

Returns whether or not the current JVM configuration supports redefinition of classes. The ability to redefine an already loaded class is an optional capability of a JVM. Redefinition will only be supported if the Can-Redefine-Classes manifest attribute is set to true in the agent JAR file (as described in the package specification) and the JVM supports this capability. During a single instantiation of a single JVM, multiple calls to this method will always return the same answer.**/
```

### 一个基于 Attach 的 Java Agent 探针实现过程

### 目标

> 实现一个简单性能工具，通过 Java Agent 探针统计 Java 应用程序下所有方法的执行时间。

1\. 还是之前 Maven 项目工程，在 `MANIFEST.MF` 文件中定义`Agentmain-Class`属性，指定一个实现类。类中实现了`Agentmain`方法，这就是 Java Agent 在 JVM 运行时加载的启动入口：

```
Agent-Class: com.laziobird.MyAgentDemo
```

2\. 构建 Agentmain 方法。

```
public class MyAgentDemo {
  
  public static void agentmain(String args, Instrumentation inst) {
      System.out.println(" agentmain agent loaded !");
      Class[] allClass = inst.getAllLoadedClasses();
      for (Class c : allClass) {
          inst.addTransformer(new AgentMainTransformerDemo(), true);
          try {
          
          inst.retransformClasses(c);
               } catch (UnmodifiableClassException e) {
                 throw new RuntimeException(e); }
          }    }
 ....   
}
```

我们在类 `MyAgentDemo`实现`agentmain`方法，里面添加一个类转化器`AgentMainTransformerDemo`，这个转化器插入实现统计方法调用时间的字节码片段。

```
public class AgentMainTransformerDemo implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
                            ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        className = className.replace("/", ".");
        
        if (className.contains("com.laziobird")) {
            try {
                
                CtClass ctclass = ClassPool.getDefault().get(className);
                for (CtMethod ctMethod : ctclass.getDeclaredMethods()) {
                    
                    ctMethod.addLocalVariable("start", CtClass.longType);
                    
                    ctMethod.insertBefore("start = System.currentTimeMillis();");
                    String methodName = ctMethod.getLongName();
                    ctMethod.insertAfter("System.out.println(\"" + methodName + " cost: \" + (System" +
                            ".currentTimeMillis() - start));");
                    
                    return ctclass.toBytecode();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}
```

3\. 重新打包生成新的 Jar 包。

运行 `maven assembly`，进行编译打包。

4\. 写测试的 Java 程序。

类 AgentAttachTest 定义一个方法，为了方便查看 Attach 效果，我们让 JVM 主进程一直循环执行这个方法。同时为了区分，通过随机数改变方法的运行时间。这样看到探针每次统计结果也不同。类的包名是`com.laziobird`，Agent 只会对`com.laziobird` 的类起作用。

```
public void test(int x) {
    try {
        long sleepTime = x*1000;
        Thread.sleep(sleepTime);
        System.out.println("the method: AgentAttachTest.test | sleep time = " + sleepTime+ "ms");
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
}
public static void main(String[] args) {
    AgentAttachTest agentTest = new AgentAttachTest();
    while (1==1){
        int x = new Random().nextInt(10);
        agentTest.test(x);
    }
}
```

5\. 编写一个演示 Attach 通信的 JVM 程序，用于启动 Agent。

```
public class AttachJVM {
    public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {
        
        List<VirtualMachineDescriptor> vmList = VirtualMachine.list();
        
        String agentJar = "/Users/jiangzhiwei/eclipse-workspace/agentdemo/target/javaagent-demo-0.0.1-SNAPSHOT-jar-with-dependencies.jar";
        for (VirtualMachineDescriptor vmd : vmList) {
            
            System.out.println("vmd name: "+vmd.displayName());
            if (vmd.displayName().endsWith("AgentAttachTest")) {
                
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());
                
                virtualMachine.loadAgent(agentJar);
                virtualMachine.detach();
}}}}
```

### 运行效果

1. 运行测试的 Java 程序，为了方便，也可以不用打成 Jar 运行。

![](_assets/34ee2f4a11939dea653bfbe05e2f9cfa.png)

2. 我们启动 Attach 的 JVM 程序。它主要动作：

*   通过 Attach API，找到要监听的 JVM 进程，我们称为 VirtualMachine；
    
*   VirtualMachine 借助 Attach API 的`LoadAgent`方法将 Agent 加载进来。
    

![](_assets/6198a1fcd1e09b01b51889c98047e8e6.png)

3. Agent 开始工作！我们回过头来看看探针在测试程序的运行效果。

![](_assets/2ec027879f7f708a522095b28e3cb053.png)

我们手写 Java 探针在 JVM 运行时也能动态改变字节码。

Github 案例地址
-----------

为了方便大家上手实践，我贡献案例到 Github，其实基于 Java Agent 性能诊断工具、链路分析的 Java 探针基本都是类似实现，大部分区别在于字节码增强实现的差异。

当然，要求更高的性能和底层功能，可以直接编写 C、C++的 JVMT 动态链接库。

> 探针实现
> 
> [https://github.com/laziobird/java-agent-demo](https://github.com/laziobird/java-agent-demo)  
> 
> 测试程序
> 
> [https://github.com/laziobird/java-agent-demo/tree/main/Agentdemo](https://github.com/laziobird/java-agent-demo/tree/main/Agentdemo)  
> 
> Attach 用例
> 
> [https://github.com/laziobird/java-agent-demo/tree/main/TestAgentDemo](https://github.com/laziobird/java-agent-demo/tree/main/TestAgentDemo)  
> 
> 本文的教程
> 
> [https://github.com/laziobird/java-agent-demo/tree/main/JVMAttach](https://github.com/laziobird/java-agent-demo/tree/main/JVMAttach/)  

