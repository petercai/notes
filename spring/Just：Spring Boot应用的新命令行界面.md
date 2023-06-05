# Just：Spring Boot应用的新命令行界面
命令行工具[Just](https://just.maciejwalkowiak.com/)，无需任何配置即可自动加载有变动的源码、构建文件或 Docker 编排文件，提升 Spring Boot 应用构建时的 Java 开发体验，同时该项目也支持生成（原生）应用及（原生）Docker 镜像。

自由职业建筑师兼开发者[Maciej Walkowiak](https://www.linkedin.com/in/maciejwalkowiak/)，在代码首次提交恰好一个月后，正式[发布](https://www.linkedin.com/posts/maciejwalkowiak_java-development-springboot-activity-7008736461634084864-KiUs)了 Just。这款被编译为原生二进制文件的 Spring Boot 应用借助[picocli](https://picocli.info/)编写出功能丰富的命令行应用、[Testcontainers](https://www.testcontainers.org/)运行容器的 JUnit 测试、[Sentry](https://sentry.io/welcome/?utm_source=google&utm_medium=cpc&utm_campaign=9575834316&utm_content=g&utm_term=sentry&device=c&gclid=CjwKCAiAnZCdBhBmEiwA8nDQxZFxbL9mxyZDFYYNTXDhGLT2ryvDzmhpacSGQIRT1zDXtQtCpeOTnBoCl-sQAvD_BwE&gclid=CjwKCAiAnZCdBhBmEiwA8nDQxZFxbL9mxyZDFYYNTXDhGLT2ryvDzmhpacSGQIRT1zDXtQtCpeOTnBoCl-sQAvD_BwE)监测问题错误，以及[JReleaser](https://jreleaser.org/)发布项目。

Just 可以自动检测源码变动，并在自动重构后使用[Spring Boot开发工具](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools)重新加载应用。此外，修改`pom.xml`或`build.gradle`文件的构建都会导致应用暂停，并在构建文件刷新后重新启动。与 Spring Boot 开发工具不同，执行`run`子命令时 Just 会启动数据库、通过[Docker编排](https://docs.docker.com/compose/)定义的服务等基础设施服务，执行`just`命令可以自动触发应用构建配置检测。Just 支持 Maven 和 Gradle 对应的封装器，也支持 Maven Daemon。仅需执行一次`run`子命令，Just 就能够处理好应用中的变更。

与`run`子命令相比，`build`子命令执行时会根据构建目标正确地转换成对应的 Maven 或 Gradle 命令：

使用其中的`quick`选项会跳过测试、文档生成、格式检测以及静态分析。Just 提供`jar`、`native`、`image`，以及原生`native-image`几种不同`buildTarget`选项以创建（原生）应用或（原生）Docker 镜像。另外，`format`子命令会根据项目配置中默认设置、[Spring Java格式](https://github.com/spring-io/spring-javaformat)、[Spotless](https://github.com/diffplug/spotless)配置规则格式化代码库。运行中进程可通过`kill`子命令终止，默认设置下端口 8080 上运行的进程会被终止，但端口号也可以通过`-p` 参数指定，`-9`参数则会强行执行`kill`子命令。

Just 可通过命令行执行，在 IntelliJ IDEA 则需要先通过`init idea`子命令新增运行配置，手动新增配置则可以在“运行”菜单栏的下拉选项中选择“修改配置”，新增“Shell 脚本”并重命名，“执行”选项选择“Script Text”，输入框“Script Text”中输入`just run`。取消勾选“命令行执行”后应用配置，“运行”菜单中就会显示行 shell 脚本的名称，我们也可以点击启动 Just 了。

在 MacOS 上安装 Just 可通过[Homebrew](https://brew.sh/)执行：

```
brew install maciejwalkowiak/brew/just
```

在 Windows 上则通过[Scoop](https://scoop.sh/)：

```
scoop bucket add maciejwalkowiak https:
scoop install just
```

此外，也可以手动安装应用至 maxOS、Windows 或 Linux，以 Linux 命令为例：

其中的`help`子命令可用于验证安装结果。

Just 并非开源项目，其在 GitHub[仓库](https://github.com/maciejwalkowiak/just)中仅包含二进制、发布说明以及问题追踪，并没有发布源码。目前项目仍处于 Alpha 测试阶段且可免费使用，所有的发布版本中都含有内置过期时间，过期后可能需要购买应用或安装最新版本。

关于 Just 更多信息可查看“[开始使用](https://just.maciejwalkowiak.com/docs/getting-started/)”文档。

**原文链接：** 

[Just, a New CLI for Spring Boot Applications](https://www.infoq.com/news/2023/01/just-spring-boot-cli/)  

**相关阅读：** 

[Spring Boot 3 和 Spring Framework 6 使用 Java 17 和 Jakarta EE 9，并支持基于 GraalVM 的原生 Java](https://www.infoq.cn/article/iCQ44j3XyAEl2FgHSPQy "xxx")  

[Spring Boot Migrator 简介](https://www.infoq.cn/article/M8Tcely7QZhZYx4od2t1)  

[Dubbo 正式支持 Spring 6&Spring Boot 3](https://www.infoq.cn/article/LAvbFBiTzeXeqQ2CzAsi)