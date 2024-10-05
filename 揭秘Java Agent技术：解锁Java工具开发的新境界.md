# 揭秘Java Agent技术：解锁Java工具开发的新境界
作为JDK提供的关键机制，Java Agent技术不仅为Java工具的开发者提供了一个强大的框架，还为性能监控、故障诊断和动态代码修改等领域带来了革命性的变革。本文旨在全面解析Java Agent技术的应用场景以及实现方式，特别是静态加载模式和动态加载模式这两种关键模式。

* * *

Java Agent技术是JDK提供的关键机制，专门用于编写高级Java工具。通过这项技术，开发者能够创建特殊的JAR包，这些JAR包能够在Java程序运行时执行其中的代码。Java Agent技术赋予了Java程序执行独立Java Agent程序中代码的能力，这种执行方式分为静态加载模式和动态加载模式两种。

1.静态加载模式
--------

静态加载模式允许在Java程序启动的初始阶段就执行指定的代码，因此特别适用于APM（应用性能管理）等需要从一开始就监控程序执行性能的场景。为实现静态加载，开发者需要在Java Agent项目中编写一个premain方法，并将其打包成JAR包。

```typescript
public static void premain(string agentArgs, Instrumentation inst){

}

```

之后，通过以下命令启动Java程序，Java虚拟机将自动加载并执行agent中的代码：

```bash
java -jar -javaagent:./agent.jar test.jar

```

| \-javaagent: | 告诉JVM在启动应用程序之前加载并应用这个Agent |
| ./agent.jar | Java Agent JAR文件的路径 |
| test.jar | 要运行的Java应用程序的JAR文件 |

premain方法将在主线程中执行。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11594d77dff949abb243fe92c305703a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1342&h=132&s=20178&e=png&b=9fbb52)
​

2.动态加载模式
--------

与静态加载模式不同，动态加载模式允许在任意时刻执行Java Agent代码，因此特别适用于Arthas等诊断系统。实现动态加载，开发者需要编写一个agentmain方法，并将其打包成JAR包。

```typescript
public static void agentmain(String agentArgs, Instrumentation inst){

}

```

要执行Java Agent代码，开发者可以使用以下代码片段动态连接到指定进程ID的Java程序：

```ini
// 动态连接到指定进程ID的java程序
VirtualMachine vm= VirtualMachine.attach("进程ID")
// 加载iava agent
vm.loadAgent("./agent.jar")

```

与premain方法不同，agentmain方法将在独立线程中执行。这种灵活性使得动态加载模式在需要动态修改或监控运行中的Java应用程序时特别有用。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a4ef8dac2a047b39bdead2e29cfb92a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1330&h=232&s=29406&e=png&b=9fbb52)
​

> 1.搭建Java Agent静态加载模式的案例
> -----------------------
> 
> **步骤一：Maven项目构建与插件集成**
> 
> 首先，创建一个新的Maven项目，在项目的pom.xml文件中，添加maven-assembly-plugin插件的配置。这个插件的主要作用是将项目依赖和编译后的类文件打包成一个单独的JAR文件，该文件将作为Java Agent的部署包。
> 
> ```xml
>             <plugin>
>                 <groupId>org.apache.maven.plugins</groupId>
>                 <artifactId>maven-assembly-plugin</artifactId>
>                 <configuration>
>                    
>                     <descriptorRefs>
>                         <descriptorRef>jar-with-dependencies</descriptorRef>
>                     </descriptorRefs>
>                     
>                     <archive>
>                         <manifestFile>src/main/resources/MANIFEST.MF</manifestFile>
>                     </archive>
>                 </configuration>
>             </plugin>
> 
> ```
> 
> **步骤二：编写Java Agent核心代码**
> 
> 在项目的源代码目录下，编写一个包含premain方法的类。premain方法是Java Agent的生命周期入口，它会在Java应用启动之前执行。在这个方法中，可以编写自定义的逻辑，例如打印一行信息。
> 
> ```typescript
> public class AgentMain {
>     
>     public static void premain(String agentArgs, Instrumentation inst){
>         System.out.println("premain方法");
>     }
> }
> 
> ```
> 
> **步骤三：定义Java Agent的清单文件**
> 
> 创建一个名为MANIFEST.MF的清单文件，并放置在项目的src/main/resources/META-INF目录下。这个文件描述了Java Agent的一些配置属性，比如使用哪个类的premain方法。
> 
> ```yaml
> Manifest-Version: 1.0
> Premain-Class: com.rye.javaagent.AgentMain
> Can-Redefine-Classes: true
> Can-Retransform-Classes: true
> Can-Set-Native-Method-Prefix: true
> 
> ```
> 
> | **Manifest-Version: 1.0** | 指定了清单文件的版本，通常为`1.0，`它用于标识清单文件遵循的规范版本。 |
> | **Premain-Class: com.rye.javaagent.AgentMain** | Premain-Class属性指定了Java Agent的主要入口点（包含premain方法的类）。当Java应用程序启动并加载该Agent时，premain方法会被调用。 |
> | **Can-Redefine-Classes: true** | 这个属性允许Java Agent在运行时重新定义已加载的类。如果设置为`true`，Agent可以在不卸载和重新加载整个类的情况下修改类的字节码。这对于实现某些高级功能（如热修复、动态修改代码）非常有用。 |
> | **Can-Retransform-Classes: true** | 这个属性允许Java Agent在运行时重新转换已加载的类。与Can-Redefine-Classes不同，Retransform要求类的原始字节码仍然可用，并且重新转换的字节码必须与原始字节码在结构上是兼容的（即具有相同的方法签名和字段）。这允许Agent在保持类结构一致性的情况下修改类的行为。 |
> | **Can-Set-Native-Method-Prefix: true** | 这个属性允许Java Agent设置原生方法（native methods）的前缀。如果设置为true，Agent可以为加载的类中的原生方法名添加指定的前缀。这可以用于拦截或修改对原生方法的调用。然而，请注意，这个特性在某些Java版本和平台上可能不可用或受到限制。 |
> 
> **步骤四：使用Maven Assembly插件进行打包**
> 
> 在命令行或IDE中使用maven-assembly-plugin插件进行打包。插件会将项目的所有依赖和编译后的类文件打包成一个JAR文件，该文件将作为Java Agent的部署包。
> 
> ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09033186ab034da99818f873216571c4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=545&h=251&s=36885&e=png&b=3c3f41)
> ​
> 
> **步骤五：集成Java Agent到Spring Boot应用中**
> 
> 创建一个新的Spring Boot项目或使用现有的Spring Boot应用。在Spring Boot应用的启动脚本或配置文件中，配置Java Agent的静态加载。这需要在JVM启动参数中添加-javaagent选项，并指定Java Agent的JAR文件路径。
> 
> ```bash
> java -jar -javaagent:./agent.jar test.jar
> 
> ```
> 
> | \-javaagent: | 告诉JVM在启动应用程序之前加载并应用这个Agent |
> | ./agent.jar | Java Agent JAR文件的路径 |
> | test.jar | 要运行的Java应用程序的JAR文件 |

> 2.搭建Java Agent动态加载模式的案例
> -----------------------
> 
> **步骤一：创建Maven项目并配置打包插件**
> 
> 首先，创建一个新的Maven项目，在项目的pom.xml文件中，添加maven-assembly-plugin插件的配置。这个插件的主要作用是将项目依赖和编译后的类文件打包成一个单独的JAR文件，该文件将作为Java Agent的部署包。
> 
> ```xml
>             <plugin>
>                 <groupId>org.apache.maven.plugins</groupId>
>                 <artifactId>maven-assembly-plugin</artifactId>
>                 <configuration>
>                    
>                     <descriptorRefs>
>                         <descriptorRef>jar-with-dependencies</descriptorRef>
>                     </descriptorRefs>
>                     
>                     <archive>
>                         <manifestFile>src/main/resources/MANIFEST.MF</manifestFile>
>                     </archive>
>                 </configuration>
>             </plugin>
> 
> ```
> 
> **步骤二：编写Java Agent的核心代码**
> 
> 在项目中创建一个新的Java类，用于实现Java Agent的逻辑。在该类中编写agentmain方法。与premain方法不同，agentmain方法支持在Java应用程序运行时动态加载Agent。在这个方法中，可以编写自定义的逻辑，例如打印一行日志信息，以验证Agent已成功加载。
> 
> ```typescript
> public class AgentMain {
>     
>     public static void agentmain(String agentArgs, Instrumentation inst){
>         System.out.println("agentmain方法");
>     }
> }
> 
> ```
> 
> **步骤三：配置Java Agent的清单文件**
> 
> 在MANIFEST.MF文件中，指定使用哪个类的agentmain方法作为Java Agent的入口点。这告诉JVM在动态加载Agent时应该调用哪个方法。
> 
> ```yaml
> Manifest-Version: 1.0
> Agent-Class: com.rye.javaagent.AgentMain
> Can-Redefine-Classes: true
> Can-Retransform-Classes: true
> Can-Set-Native-Method-Prefix: true
> 
> ```
> 
> **步骤四：构建和打包Java Agent**
> 
> 在命令行或IDE中使用maven-assembly-plugin插件进行打包。插件会将项目的所有依赖和编译后的类文件打包成一个JAR文件，该文件将作为Java Agent的部署包。
> 
> ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87bb7c1a90fd461abbd285fc5ef36810~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=545&h=251&s=36885&e=png&b=3c3f41)
> ​
> 
> **步骤五：编写Java应用程序以动态加载Java Agent**
> 
> 创建一个新的Java类或在现有的Java项目中添加一个类，用于编写动态加载Java Agent的逻辑。在该类中编写main方法，并使用Java的VirtualMachine类来连接到正在运行的Java应用程序。通过调用VirtualMachine.loadAgent方法，可以动态地将打包好的Java Agent JAR文件加载到目标应用程序中。在加载Agent之前，确保目标Java应用程序已经启动并且处于可连接状态。这涉及到设置适当的JVM参数以启用远程调试或JMX（Java Management Extensions）连接。一旦Agent成功加载，它将开始执行在agentmain方法中定义的逻辑，从而可以在不重启目标应用程序的情况下修改或监视其行为。
> 
> ```arduino
> public class AttachMain {
>     public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {
>         
>         VirtualMachine vm=VirtualMachine.attach("进程ID");
>         
>         vm. loadAgent("./agent.jar");
>     }
> }
> 
> ```

* * *

本文探讨了Java Agent技术的核心原理和应用价值，展示了它在Java工具开发中的关键作用。通过静态加载和动态加载两种模式，Java Agent能够灵活地在Java程序的不同阶段执行自定义代码，为性能监控、故障诊断和动态代码修改提供了有力支持。希望对大家有所帮助。