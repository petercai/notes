# 4 种方法，帮你快速新建 Java 项目
如何快速初始化 Java 项目？
----------------

### 1、使用开发工具

Java 开发者最常用的开发工具当属 JetBrains IDEA 了！

IDEA 不仅功能完善、插件丰富，而且其实对新手比较友好。

比如在 IDEA 中，你可以快速安装需要的指定版本的 JDK，不用自己到官网下载：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f66bbf627234855b68208b3f7858132~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2344&h=962&s=357048&e=png&b=3c4043)

使用 IDEA 来创建初始化项目也是最常用的方法了，点击左上角的 File => New => Project：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76a38e32dc7448e8bd6241b59353d1a0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1272&h=696&s=369741&e=png&b=313437)

然后进入项目创建界面，左侧选择需要的模板，右侧填写项目信息，即可完成创建：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f4e0f31de64466d81b99734eeb72e6f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1926&h=994&s=198437&e=png&b=3d4042)

最常用的模板当属 `Spring Initializr` 了，可以快速初始化 Spring Boot 项目：

> 注意选择 Java 的版本号

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12a7273153114d2f9f1c07f07a2fcf83~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1916&h=1318&s=254866&e=png&b=3d4042)

支持可视化地选择项目的依赖，一般不用自己去写依赖配置或者粘贴了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74596559cf464e129ba99745b09abbc3~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1932&h=1242&s=201013&e=png&b=3c3f41)

如果要引入更多 Java 的包，可以到 Maven 中心仓库寻找：[mvnrepository.com/](https://link.juejin.cn/?target=http%3A%2F%2Fmvnrepository.com%2F "http://mvnrepository.com/") 。

### 2、项目管理工具

对于 Java 开发者，最常用的项目管理工具是 Maven 和 Gradle。它们不仅可以管理项目依赖、打包构建项目，也可以快速创建新项目。

不过对于不熟悉这些工具的同学来说，不推荐使用这种方式创建项目，仅做了解即可。

下面分别演示 2 种工具创建新项目的方法。

#### 使用 Maven 创建项目

安装 Maven 后，使用以下命令创建 Spring Boot 项目（仅供参考）：

```ini
mvn archetype:generate \
    -DgroupId=com.example \
    -DartifactId=my-spring-boot-app \
    -DarchetypeArtifactId=maven-archetype-quickstart \
    -DinteractiveMode=false

```

解释一下上面命令中的参数：

*   `-DgroupId`: 你的项目的组 ID
*   `-DartifactId`: 你的项目的 Artifact ID
*   `-DarchetypeArtifactId`: Maven 快速启动项目的模板
*   `-DinteractiveMode=false`: 禁用交互模式，使其自动创建项目

#### 使用 Gradle 创建项目

Gradle 的项目模板相比 Maven 来说少了一些。安装 Gradle 后，使用以下命令创建项目：

```csharp
gradle init

```

然后跟着操作提示输入选项，即可创建出不同的项目：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e503202c63e425094d416612a1f0643~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1604&h=1066&s=138341&e=png&b=2b2b2b)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08c2b4fe6a1c47a7a2ea463b20d818ef~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2618&h=920&s=180140&e=png&b=2b2b2b)

### 3、项目模板生成器

有很多专门用来创建初始化项目模板的工具和网站，这里分享其中 4 种：

#### Spring Initializr

Spring 官方的项目模板生成器，可以使用可视化界面来选择项目配置，并快速生成 Spring Boot 项目的初始代码。

> 指路：[start.spring.io/](https://link.juejin.cn/?target=https%3A%2F%2Fstart.spring.io%2F "https://start.spring.io/")

界面如下，还可以分享自己的配置给别人：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c7b3a4297684c60a67b6c1bfb255d14~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2564&h=1484&s=239051&e=png&b=ffffff)

不过 IDEA 开发工具内已经集成了 `Spring Initializr`，一般没必要专门在网站中使用。

#### 微服务模板生成器

阿里提供了一款云原生应用脚手架，如果你的项目需要用到 Spring Cloud Alibaba 组件，那么强烈建议使用该脚手架来创建项目，可以保证各组件依赖版本号的一致性。

> 指路：[start.aliyun.com/](https://link.juejin.cn/?target=https%3A%2F%2Fstart.aliyun.com%2F "https://start.aliyun.com/")

用法和 Spring Initializr 几乎完全一致，可以自己选择依赖：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c3442b292334cd88ae70f3d68d94641~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1860&h=1440&s=159147&e=png&b=fcfcfc)

#### JHipster

专门用于生成 Java 项目的工具，模板和选项非常丰富。

> 指路：[www.jhipster.tech/cn/](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jhipster.tech%2Fcn%2F "https://www.jhipster.tech/cn/")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82f9c2f5637842b89dde0f385edc31e2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1688&h=492&s=99411&e=png&b=5289c7)

JHipster 的功能还是很强大的，但只是创建初始化项目的话，用法非常简单，只需要输入 `jhipster` 命令：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0a8defe41274d8f9e830b83fb49f57c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1772&h=716&s=84564&e=png&b=2b2b2b)

然后跟着命令行的提示输入选项即可：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b7899c0d0414c2588ed3985253b7b93~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1956&h=610&s=137259&e=png&b=2b2b2b)

#### Yeoman

Yeoman 是一个生成项目模板的工具，通常用于前端项目的初始化。

虽然 Yeoman 主要用于前端开发，但也有一些 Java 项目的初始化模板。而且你可以编写自己的 Yeoman 生成器来生成 Java 代码或者任何其他类型的代码。

> 指路：[yeoman.io/generators/](https://link.juejin.cn/?target=https%3A%2F%2Fyeoman.io%2Fgenerators%2F "https://yeoman.io/generators/")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1a8c6281142415da70cc245920e0c19~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1988&h=1374&s=276409&e=png&b=fefefe)

### 4、开源项目

除了生成项目外，我们也可以直接下载并使用 GitHub 上的开源项目代码，也就是直接用别人创建好的项目。

比较有名的有 Jeecg Boot：

> 指路：[github.com/jeecgboot/j…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fjeecgboot%2Fjeecg-boot "https://github.com/jeecgboot/jeecg-boot")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/219cb2a55fe148399a6faa1d8ff47c79~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2094&h=894&s=349054&e=png&b=ffffff)

项目效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d5448ea495f4fe9989ba18c8c423235~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1600&h=757&s=371417&e=png&b=fefefe)

还有若依：

> 指路：[github.com/yangzongzhu…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fyangzongzhuan%2FRuoYi "https://github.com/yangzongzhuan/RuoYi")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63704a00935145ca8fd3d6c2cdf65b0f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1996&h=578&s=182474&e=png&b=ffffff)

项目效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54477a41e26e4466a2a84ef865e032de~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1366&h=768&s=275264&e=png&b=b3b0aa)

这些项目一般都是大而全的、功能十分丰富的管理系统，对于企业来说会比较实用，但是对于编程学习者来说，不是很推荐，想要自定义开发一些额外的功能会比较麻烦。

* * *

除了以上方法外，最推荐的方法还是在学习和开发过程中，持续整理和沉淀一套属于自己的万用项目模板，企业中也通常都会有适应业务的基础建设代码。这样一来，绝大多数功能都不用重复写第 2 遍，以后开发新项目会越来越快。

