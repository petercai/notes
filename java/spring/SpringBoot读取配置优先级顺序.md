# SpringBoot读取配置优先级顺序
引言
--

Spring Boot作为一种轻量级的Java应用程序框架，以其开箱即用、快速搭建新项目的特性赢得了广大开发者的青睐。其核心理念之一就是简化配置过程，使开发者能够快速响应复杂多变的生产环境需求。为了实现这一点，Spring Boot支持丰富的外部化配置机制，允许应用程序根据不同的部署环境灵活加载相应的配置属性，而无需修改代码本身。

在Spring Boot生态系统中，配置属性可以从各种来源获取，比如：Java属性文件、YAML文件、环境变量、命令行参数等。这些配置属性能够在运行时动态注入到Bean中，极大地提高了系统的可扩展性和可配置性。然而，为了确保一致性和防止配置冲突，Spring Boot在加载这些外部配置时遵循一套严格的优先级顺序。掌握这套优先级规则至关重要，因为它直接影响着最终生效的配置属性值，进而决定了应用程序的行为模式。

本文将深入探讨Spring Boot加载外部配置属性的优先级规则，详尽梳理各个配置源的加载顺序，并结合实际应用场景举例说明，以便我们能够更高效地管理和迁移配置，确保在不同环境下应用程序都能稳定、准确地运行。

Spring Boot外部化配置概述
------------------

Spring Boot的核心价值之一在于其强大的外部化配置能力，这使得应用程序能够在不改变代码的情况下适应不同的运行环境。外部化配置意味着将应用程序的关键配置信息移至应用程序代码之外，便于根据不同环境（如开发、测试、生产等）进行定制化配置。Spring Boot提供了多样化的外部配置源以及便捷的属性注入方式，使得这种配置机制变得异常灵活且易于管理。

### 多样化配置源

Spring Boot支持多种类型的外部配置源，主要有如下几个方面：

1.  **Properties文件:**  通常使用`.properties`格式，采用键值对的形式存储配置信息。

```ini
server.port=8080
logging.level.root=DEBUG

```

2.  **YAML文件：**   相较于传统的properties文件，YAML提供了更直观、层次更分明的数据结构，尤其适合存储复杂配置。使用`.yml`格式。

```yaml
server:
  port: 8080
logging:
  level:
    root: DEBUG

```

3.  **环境变量**： 操作系统级别的环境变量可以被Spring Boot识别并作为配置源，这对于云环境和容器化部署尤为实用。
4.  **命令行参数**： 启动Spring Boot应用时，可以传入命令行参数（以`--`开头）直接覆盖已有配置。

### 属性注入方式

在Spring Boot中，外部配置的属性值可以通过以下几种方式方便地注入到Bean中。

*   •  **`@Value`注解**：可以直接在字段或方法参数上使用此注解，将配置属性值注入到目标对象中。
*   • **`Environment`接口**：Spring框架提供的环境抽象类，可以用来查询所有已加载的配置信息。
*   •  **`@ConfigurationProperties`注解**：用于绑定一组相关配置到一个专门的Java Bean中，提供更结构化的配置管理方式。

### 配置加载优先级

Spring Boot对来自不同配置源的同名属性可以按照一定的优先级顺序进行覆盖。其优先级从上到下变高，即后面的配置源将覆盖前面的配置源。

1.  1. 默认属性（通过`SpringApplication.setDefaultProperties`方法设置）
2.  2. `@PropertySource`注解加载的配置
3.  3. Config Data（配置数据）（本地文件系统或打包在jar中的application.properties和application-{profile}.properties）
4.  4. 特殊属性源（如随机数生成器、环境变量、系统属性、JNDI属性等）
5.  5. Servlet容器相关的初始化参数
6.  6. SPRING\_APPLICATION\_JSON格式的环境变量或系统属性
7.  7. 命令行参数
8.  8. 测试相关的属性注入方式（如`@SpringBootTest`、`@DynamicPropertySource`和`@TestPropertySource`）

> 以上优先级顺序来源于官网：Spring Boot Reference Documentation

Spring Boot配置加载顺序详解
-------------------

### 默认属性

默认属性是指Spring Boot框架内置的一些默认配置值。可以在创建`SpringApplication`实例时，通过调用`setDefaultProperties(Map<String, Object> defaultProperties)`方法来提供一组默认属性，这些属性将被优先加载，但是也会被其他配置覆盖。

```typescript
@SpringBootApplication
public class SpringBootBaseApplication {

    public static void main(String[] args) {
        Map<String, Object> defaultProperties = new HashMap<>();
        defaultProperties.put("server.port", "9000"); 
        SpringApplication app = new SpringApplication(SpringBootBaseApplication.class);
        app.setDefaultProperties(defaultProperties);
        app.run(args);
    }
}

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b7e5daad60d43f0b113585fcfeaa162~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2716&h=172&s=98474&e=png&b=272727)

image.png

### @PropertySource注解

`@PropertySource`注解用于在Spring Boot的`@Configuration`类上加载外部属性文件。当我们在配置类上使用`@PropertySource`时，需要注意的是，这些属性源并不会立即被添加到Spring的`Environment`中。它们是在Spring应用上下文刷新（refresh）阶段才会被真正加载并合并到环境变量中。

> 有兴趣的可以跟一下源码，`org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors`中执行的。

Spring Boot的主引导配置，如服务器端口（server.port）、日志框架的初始化（例如日志级别设置）等，也是在应用上下文刷新之前就被读取并应用的。因此，对于这类早期就需要读取的配置，应该直接在`application.properties`或者环境变量等更早被加载的配置源中进行设置。

我们创建一个`propertysource.properties`文件：

```ini
server.port = 9001
coderacademy.name = CoderAcademy

```

然后我们在`@Configuration`配置上使用`@PropertySource`导入`propertysource.properties`文件。

```less
@PropertySource(value = "classpath:propertysource.properties")
@Configuration
public class MyConfig {

}

```

我们在应用启动后看一下上述配置：

```typescript
@SpringBootApplication
public class SpringBootBaseApplication {

    public static void main(String[] args) {
        Map<String, Object> defaultProperties = new HashMap<>();
        defaultProperties.put("server.port", "9000"); 
        SpringApplication app = new SpringApplication(SpringBootBaseApplication.class);
        app.setDefaultProperties(defaultProperties);
        ConfigurableApplicationContext context = app.run(args);
        Environment environment = context.getEnvironment();
        System.out.println("coderacademy.name: " + environment.getProperty("coderacademy.name"));
    }
}

```

打印结果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8dbf15b696ce4921ab763d8b9572e873~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=3068&h=250&s=138203&e=png&b=262626)

image.png

可以看出`server.port`变成了9001，即`@PropertySource`加载的配置覆盖了`SpringBoot`默认的属性值。

### Config Data（配置数据）

Config Data（配置数据）是Spring Boot中用于外部化应用配置的核心部分。主要由内部配置文件以及外部配置文件。

#### 内部配置文件

内部配置文件最基础的应用配置文件，位于项目构建后的jar包内部。位于`src/main/resource`目录下的文件。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81436ef3805e4b6589bdea87757da24a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=820&h=968&s=110345&e=png&b=45423a)

image.png

#### 外部配置文件

可以将配置文件放在jar包外面的某个路径下。这种方式有助于在不修改jar包的情况下变更配置。比如我们使用的配置中心(`nacos`，`apollo`等)，也可以通过`spring.config.location`或者`spring.config.additional-location`指定的文件等。

SpringBoot在启动时会默认从特定的目录中加载这些配置文件。我们可以从`ConfigDataEnvironment`中找到这些目录：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94cf390cda7e4fd080121f8e95898fcf~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2028&h=300&s=99997&e=png&b=262626)

image.png

其目录的加载顺序由低到高为：

```makefile
file:./
file:./config/
file:./config/*/
classpath:/
classpath:/config/

```

其中`file`代表应用根目录下的文件，而`classpath`为`resources`下的文件。

这些配置文件的配置优先级顺序由低到高为：

```makefile
classpath:/
classpath:/config/
file:./
file:./config/
file:./config/*/

```

> 本例基于SpringBoot2.7版本。 关于SpringBoot加载内部配置文件的执行流程以及原理，请参考： [华为二面：SpringBoot读取_配置文件_的原理是什么？加载顺序是什么？](https://juejin.cn/post/7340927432319123482 "https://juejin.cn/post/7340927432319123482")

我们分别在这些目录下创建配置文件`application.properties`：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75f8edc89c1648f6867610d796577f0a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=844&h=916&s=99633&e=png&b=353739)

我们在对应文件中写入他们的目录路径：

```lua
1: config.data.path = classpath:./
2: config.data.path = classpath:./config/
3: config.data.path = file:./
4: config.data.path = file:./config/
5: config.data.path = file:./config/dev

```

我们在SpringBoot启动时打印`config.data.path`的值：

```typescript
@SpringBootApplication
public class SpringBootConfigApplication {

    public static void main(String[] args) {
        Map<String, Object> defaultProperties = new HashMap<>();
        defaultProperties.put("server.port", "9000"); 
        SpringApplication app = new SpringApplication(SpringBootConfigApplication.class);
        app.setDefaultProperties(defaultProperties);
        ConfigurableApplicationContext context = app.run(args);
        Environment environment = context.getEnvironment();
        System.out.println("config.data.path: " + environment.getProperty("config.data.path"));
    }
}

```

我们分步进行验证，先验证1,2，打印结果:

```lua
config.data.path: classpath:./config/

```

继续验证1,2,3，打印结果：

```lua
config.data.path: file:./

```

验证1,2,3,4,打印结果：

```lua
config.data.path: file:./config/

```

验证1,2,3,4,5,打印结果：

```lua
config.data.path: file:./config/dev

```

### 随机值属性源

**RandomValuePropertySource** 在Spring Boot中，`RandomValuePropertySource`是一个特殊属性源，它并不来源于固定的配置文件或环境变量，而是由Spring Boot框架在启动时自动添加。这个属性源提供的属性名以`random.*`开头，可以用于生成随机值。例如，你可以在配置文件中引用`random.int`或`random.long`等属性，Spring Boot在启动时会为这些属性生成随机整数值。这对于需要在运行时生成一些临时或随机值的场景非常有用，如临时密码、缓存密钥等。

比如我们在`application.properties`中设置`random.int=100`

```arduino
random.int=100

```

我们在SpringBoot启动时获取\`\`random.int\`的值：

```java
@SpringBootApplication
public class ConfigApplication
{
    public static void main( String[] args )
    {
        SpringApplication app = new SpringApplication(ConfigApplication.class);
        ConfigurableApplicationContext context = app.run(args);
        Environment environment = context.getEnvironment();
        System.out.println("random.int: " + environment.getProperty("random.int"));
    }
}

```

打印结果为：

```arduino
random.int: -510589238

```

并且每次重新启动应用，打印的结果都不一样。

### 操作系统环境变量

在Spring Boot中，环境变量可以用作配置源，Spring Boot会自动检测并加载这些环境变量作为应用的配置属性。例如，如果在操作系统中设置了环境变量`MY_APP_PORT=8080`，那么在Spring Boot应用中可以通过`${MY_APP_PORT}`来引用这个值。

我们设置环境变量为`config.data.path=环境变量`:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73d35f9abc134286a8dc14b861068fe4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1356&h=536&s=82834&e=png&b=37393b)

image.png

我们启动引用，依然打印`config.data.path`的结果为：

```lua
config.data.path: 环境变量

```

### Java系统属性

Java系统属性是通过`System.setProperty()`方法设置一系列键值对。

```java
@SpringBootApplication
public class ConfigApplication
{
    static {
        System.setProperty("config.data.path", "SystemProperty"); 
    }

    public static void main( String[] args )
    {
        SpringApplication app = new SpringApplication(ConfigApplication.class);
        ConfigurableApplicationContext context = app.run(args);
        Environment environment = context.getEnvironment();
        System.out.println("config.data.path: " + environment.getProperty("config.data.path"));
    }
}

```

打印结果为：

```lua
config.data.path: SystemProperty

```

### SPRING\_APPLICATION\_JSON环境变量中的内嵌JSON属性

`SPRING_APPLICATION_JSON` 是 Spring Boot 提供的一种机制，允许通过环境变量传递 JSON 格式的配置给应用程序。这个环境变量的内容会被解析成一个 JSON 对象，并合并到Spring的`Environment`中，就像其他属性源一样。

```java
@SpringBootApplication
public class ConfigApplication
{
    static {
        System.setProperty("config.data.path", "SystemProperty"); 
        System.setProperty("SPRING_APPLICATION_JSON", "{"config.data.path":"SPRING_APPLICATION_JSON环境变量中的内嵌JSON属性"}");
    }

    public static void main( String[] args )
    {
        SpringApplication app = new SpringApplication(ConfigApplication.class);
        ConfigurableApplicationContext context = app.run(args);
        Environment environment = context.getEnvironment();
        System.out.println("config.data.path: " + environment.getProperty("config.data.path"));
    }
}

```

打印结果：

```lua
config.data.path: SPRING_APPLICATION_JSON环境变量中的内嵌JSON属性

```

### **命令行参数**

启动Spring Boot应用时，可以直接通过命令行参数来覆盖或设置配置属性。命令行参数通常以`--`开头，后面紧跟属性名和值，如`--server.port=8080`。这种方式可以在不修改配置文件的前提下临时调整应用配置。命令行参数具有较高的优先级，可以覆盖其它配置源中的属性值。

我们使用`java -jar`启动SpringBoot：

```arduino
java -jar ./springboot-config-1.0-SNAPSHOT.jar --config.data.path=命令行参数

```

打印结果为：

```lua
config.data.path: 命令行参数

```

> 关于SpringBoot的jar包，可以通过java -jar命令直接执行的原因请参考：[字节二面：为什么SpringBoot的 jar 可以直接运行？我说内嵌了Tomcat容器，他让我出门左转](https://juejin.cn/post/7353582927680208933 "https://juejin.cn/post/7353582927680208933")

至于其他的跟单测有关的配置，我们就不一一细说了

总结
--

Spring Boot配置加载优先级的设计具有深远的实际意义和重要性。这一机制确保了应用在不同环境和部署场景下的高度灵活性和可移植性，同时也极大提升了开发和运维团队的生产力和协同效率。

优先级顺序的严谨性使得开发者能够精细地控制配置的覆盖层级，从而使同一份代码可以根据不同环境的需求加载不同的配置属性。例如，在开发、测试和生产环境中分别启用不同的数据库连接、日志级别或API密钥等敏感信息，而无需在代码中硬编码。

Spring Boot的配置加载流程首先考虑了默认配置，然后逐步加载用户通过`@PropertySource`注解引入的属性源、打包在jar包内外的各种application.properties和application-{profile}.properties或YAML文件、环境变量、系统属性，直至命令行参数等。这种分层加载策略确保了越靠后的配置源拥有更高的优先级，从而可以覆盖之前的配置，这也体现了配置的灵活性和即时性。

理解并合理运用Spring Boot配置加载的优先级，对于保障应用的安全性、可维护性以及降低部署复杂度至关重要。特别是在大规模微服务架构中，合理的配置管理和迁移对于整体系统的稳定性有着不可忽视的作用。通过对配置优先级的深入掌控，开发者和运维人员能够轻松应对复杂环境下的配置管理挑战，使得Spring Boot应用具备良好的扩展性和适应性。
