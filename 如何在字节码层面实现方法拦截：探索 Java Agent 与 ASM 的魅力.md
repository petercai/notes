# 如何在字节码层面实现方法拦截：探索 Java Agent 与 ASM 的魅力
**Java Agent**

Java Agent 是一种运行在 Java 虚拟机 (JVM) 上的特殊程序，可以在程序运行期间对字节码进行修改和增强，从而达到在不修改源码的情况下实现各种功能的目的。

Java Agent 的主要作用包括但不限于以下几点：

字节码增强：通过修改字节码，实现一些功能增强，比如方法拦截、性能监控等。

类加载控制：可以在类加载前对类进行修改或者替换，实现一些定制化需求。

内存分析：通过 Java Agent 可以获取到 JVM 的内存信息，对内存进行分析，帮助排查内存相关问题。

代码检查：通过 Java Agent 可以在类加载前对代码进行检查，实现一些代码质量相关的需求。

**ASM**

ASM（全称：ASMifier Class Visitor），是一个轻量级的 Java 字节码编辑和分析框架，可以直接以二进制形式读取和修改类文件。ASM 提供了许多 API 和工具，可以方便地进行字节码修改和生成。

ASM 的主要作用包括但不限于以下几点：

字节码生成：可以通过 ASM 生成 Java 类的字节码，可以用于生成代理类、动态生成类等场景。

字节码修改：可以通过 ASM 对已有的类字节码进行修改，实现一些类增强、方法拦截等功能。

字节码分析：可以通过 ASM 对已有的类字节码进行分析，实现一些类结构的分析和转换。

**实践：使用 Java Agent 和 ASM 实现方法拦截**

需求背景

在一个项目中，统一对catch异常进行处理，例如日志输出（含堆栈），由于研发人员水平不一，有很多时候打印日志格式不同意，也不利于做一些埋点工作。

**应用层代码**

```java
package com.example.demo.agent;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Test {
    public static void main(String[] args) {
        try {
            int i = 1 / 0;
        } catch (Exception e) {
            
        }
    }
}

```

**构建探针jar包**

```java
package com.example.demo.agent;

import java.lang.instrument.Instrumentation;
import java.util.Set;

public class MyAgent {
    public static void premain(String agentArgs, Instrumentation inst) {
        
        Set<String> basePackages = ConfigService.getBasePackages();

        
        MyClassTransformer transformer = new MyClassTransformer(basePackages);

        inst.addTransformer(transformer);
    }
}

```

在 Java Agent 中，premain 方法是 Java 虚拟机启动时调用的入口方法。它允许我们在应用程序启动之前对字节码进行修改或者进行一些预处理操作。

premain 方法是 Java Agent 的必要组成部分，用于指定 Java Agent 的初始化逻辑。当我们将 Java Agent JAR 文件通过 -javaagent 参数传递给 Java 虚拟机时，虚拟机会加载并初始化 Java Agent，并在应用程序启动之前调用 premain 方法。

在 premain 方法中，我们可以通过获取 Instrumentation 实例来注册自定义的转换器（Transformer），并对加载的类进行字节码转换。通过在 premain 方法中注册转换器，我们可以在类加载过程中对类的字节码进行修改，实现类似方法拦截、性能统计、日志记录等功能。

因此，实现 premain 方法是 Java Agent 的一项必要要求，它是 Java Agent 启动和初始化的入口方法，用于配置和注册自定义的转换器。

**Maven 插件配置**

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <configuration>
        <archive>
            <manifestEntries>
                <Premain-Class>com.example.demo.agent.MyAgent</Premain-Class>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>

```

这是 Maven 的插件配置，用于配置生成的 JAR 文件的元数据信息，其中 是设置 Java Agent 的入口类。

在 Java Agent 中，需要在 JAR 文件的 MANIFEST.MF 文件中指定 Java Agent 的入口类，以便 Java 虚拟机可以正确地加载和启动 Java Agent。通过 Maven 的 maven-jar-plugin 插件配置，我们可以方便地指定 Java Agent 的入口类。

在上述配置中， 元素指定了 com.example.demo.agent.MyAgent 类作为 Java Agent 的入口类。当我们使用 Maven 构建项目并生成 JAR 文件时，插件会自动生成包含这个元数据信息的 MANIFEST.MF 文件，并将其包含在生成的 JAR 文件中。

这样，当我们将生成的 JAR 文件作为 Java Agent 使用时，Java 虚拟机会读取 JAR 文件中的 MANIFEST.MF 文件，并根据其中指定的入口类启动 Java Agent。这样就能确保 Java Agent 正确加载和执行，完成相应的字节码转换或其他操作。

**ASM处理字节码**

```java
package com.example.demo.agent;

import aj.org.objectweb.asm.Opcodes;
import org.objectweb.asm.*;

import java.io.File;
import java.lang.instrument.ClassFileTransformer;

import java.security.ProtectionDomain;
import java.util.HashSet;
import java.util.Set;

import static org.objectweb.asm.Opcodes.*;

public class MyClassTransformer implements ClassFileTransformer {
    
    private final Set<String> basePackages;
    
    private static final String OWNER = MyClassTransformer.class.getCanonicalName().replace(".", File.separator);

    public MyClassTransformer(Set<String> basePackages) {
        this.basePackages = basePackages;
    }

    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
                            ProtectionDomain protectionDomain, byte[] classfileBuffer) {
        
        if (!needEnhance(className)) {
            return classfileBuffer;
        }

        System.out.println("Transforming class: " + className);

        
        try {
            
            ClassReader cr = new ClassReader(className);
            
            ClassWriter cw = new ClassWriter(ClassWriter.COMPUTE_MAXS);
            
            ClassVisitor cv = new ClassVisitor(Opcodes.ASM5, cw) {
                @Override
                public MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {
                    
                    MethodVisitor mv = super.visitMethod(access, name, descriptor, signature, exceptions);
                    
                    return new MethodVisitor(Opcodes.ASM5, mv) {
                        
                        private final Set<Label> tryCatchBlockHandlers = new HashSet<>();

                        @Override
                        public void visitTryCatchBlock(Label start, Label end, Label handler, String type) {
                            
                            tryCatchBlockHandlers.add(handler);
                            super.visitTryCatchBlock(start, end, handler, type);
                        }

                        @Override
                        public void visitLineNumber(int line, Label start) {
                            
                            if (tryCatchBlockHandlers.contains(start)) {
                                
                                mv.visitMethodInsn(INVOKESTATIC, OWNER, "logStackTrace", "(Ljava/lang/Throwable;)V", false);
                            }
                            super.visitLineNumber(line, start);
                        }
                    };
                }
            };
            
            cr.accept(cv, ClassReader.EXPAND_FRAMES);

            
            return cw.toByteArray();
        } catch (Exception e) {
            System.out.println("MyClassTransformer e=" + e);
        }
        
        return classfileBuffer;
    }

    
    public static void logStackTrace(Throwable throwable) {
        System.out.println("统一打印堆栈：");
        throwable.printStackTrace();
    }


    private boolean needEnhance(String className) {
        for (String basePackage : basePackages) {
            if (className.startsWith(basePackage)) {
                return true;
            }
        }
        return false;
    }
}

```

这段代码实现了一个 ClassFileTransformer 接口的类 MyClassTransformer，它用于对指定的类进行字节码增强。

主要做了以下事情：

1.  在构造方法中接收需要增强的类所在的包的集合 basePackages。
    
2.  实现了 transform 方法，该方法是 ClassFileTransformer 接口的核心方法，用于对类的字节码进行转换和增强。
    
3.  在 transform 方法中，首先判断当前类是否需要进行增强，如果不需要则直接返回原字节码数据。
    
4.  使用 ASM 库进行字节码的读取和修改。通过创建 ClassReader 对象读取原始字节码，创建 ClassWriter 对象进行修改，创建 ClassVisitor 对象对字节码进行访问和修改。
    
5.  在 ClassVisitor 的 visitMethod 方法中，对每个方法进行访问，并创建 MethodVisitor 对象对方法的字节码进行访问和修改。
    
6.  在 MethodVisitor 的 visitLineNumber 方法中，当该行代码处于 try-catch 块中时，在该行代码前插入方法调用指令，调用名为 logStackTrace 的静态方法，用于打印堆栈信息。
    
7.  最后，在 needEnhance 方法中判断是否需要对类进行增强，如果类的包名在 basePackages 中，则返回 true，否则返回 false。
    

总体而言，这段代码的作用是在指定的类中的每个方法中插入一段代码，在方法调用处打印堆栈信息，用于统一处理异常的情况。这样可以方便地进行日志记录或其他异常处理操作。

**指定类路径**

application.properties：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6c4949f952a4b008af30db4ee740d41~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

```java
package com.example.demo.agent;

import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Arrays;
import java.util.HashSet;
import java.util.Properties;
import java.util.Set;

public class ConfigService {
    private static final String CONFIG_FILE_PATH = "demo/src/main/resources/application.properties";
    private static final String BASE_PACKAGES_PROPERTY = "basePackages";
    private static Set<String> basePackages;

    static {
        Properties props = new Properties();
        try (InputStream is = new FileInputStream(CONFIG_FILE_PATH)) {
            props.load(is);
            String basePackagesStr = props.getProperty(BASE_PACKAGES_PROPERTY);
            basePackages = new HashSet<>(Arrays.asList(basePackagesStr.split(",")));
            System.out.println("basePackages=" + basePackages);
        } catch (IOException e) {
            
        }
    }

    public static Set<String> getBasePackages() {
        return basePackages;
    }
}

```

这段代码是一个配置服务类 ConfigService，主要用于读取配置文件并提供基础包名的集合。

具体功能如下：

1.  定义了配置文件的路径 CONFIG\_FILE\_PATH，这里假设配置文件为 application.properties，位于 demo/src/main/resources/ 目录下。
    
2.  定义了配置文件中基础包名的属性名称 BASE\_PACKAGES\_PROPERTY，用于读取配置文件中的基础包名。
    
3.  声明了一个静态的 Set 类型的变量 basePackages，用于存储从配置文件中读取到的基础包名集合。
    
4.  在静态代码块中，通过 Properties 对象读取配置文件，并将配置文件中的基础包名字符串拆分为数组，然后转换为集合存储在 basePackages 变量中。
    
5.  最后，提供了一个静态方法 getBasePackages()，用于获取读取到的基础包名集合。
    

总体而言，这段代码的作用是从配置文件中读取基础包名集合，并提供访问该集合的方法。这样可以将需要进行方法拦截的类所在的包名配置到配置文件中，以便在 MyClassTransformer 类中使用。

**Idea执行**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1991c3a5d0664e27963591bd9b2d50f5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

Run/Debug Configurations：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01b6ad94f97041ad8a5d91f47491c8b5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

\-noverify是Java虚拟机的一个启动选项，用于禁用类验证器（Class Verifier）。类验证器是Java虚拟机的一部分，负责验证字节码的结构和语义是否符合Java语言规范。它检查类文件中的字节码指令，确保它们不会违反虚拟机的安全性和完整性。

**运行效果**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebfd370797204c08bdd65cf825afb331~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9b6567677c64662892caaed7e03397c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

在本文中，我们深入探索了如何在字节码层面实现方法拦截，并发现了 Java Agent 和 ASM 的魅力。Java Agent 是一种强大的工具，允许我们在应用程序启动时通过字节码转换来修改类的行为。而 ASM 是一个强大而灵活的字节码操作库，提供了丰富的API来读取、修改和生成字节码。

通过结合 Java Agent 和 ASM，我们可以实现方法拦截的功能。我们首先编写了一个 Java Agent，并使用 Premain-Class 来指定其入口点。在 Java Agent 中，我们使用 Instrumentation API 注册了一个 ClassFileTransformer，该转换器负责对加载的类进行转换。然后，我们定义了一个实现 ClassFileTransformer 接口的类，使用 ASM 对字节码进行操作。

具体来说，我们使用 ASM 创建了一个 ClassVisitor，用于访问和修改类的字节码。在 ClassVisitor 中，我们重写了 visitMethod 方法，用于访问和修改类中的方法字节码。我们利用 MethodVisitor 对方法字节码进行访问和修改，实现了方法拦截的功能。在示例中，我们演示了如何在方法的异常处理器（try-catch 块）中插入代码，以实现异常抛出时的统一堆栈打印。

通过本文的探索和实践，我们深刻体会到了 Java Agent 和 ASM 的魅力，它们为我们提供了无限的可能性，让我们能够更加灵活和精确地控制和改变程序的行为。无论是在调试和分析应用程序，还是在实现特定的需求和功能方面，掌握字节码级别的方法拦截技术都是非常有价值的。希望本文能为读者提供有关 Java Agent 和 ASM 的深入理解，并启发读者在实际项目中尝试和应用这些强大的技术。

> 欢迎关注公众号：程序员的思考与落地

> 公众号提供大量实践案例，Java入门者不容错过哦，可以交流！！