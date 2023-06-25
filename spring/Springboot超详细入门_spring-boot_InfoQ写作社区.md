# Springboot超详细入门_spring-boot_InfoQ写作社区
> 陈老老老板
> 
> 说明：在整体的复习一遍知识，边复习边总结，基础真的重要，需要注意的地方都标红了，还有资源的分享. 一起加油。  

JC-1.快速上手 SpringBoot
--------------------

```
学SpringBoot技术由Pivotal团队研发制作，功能的话简单概括就是加速Spring程序的开发，这个加速要从如下两个方面来说
```

*   Spring 程序初始搭建过程
    
*   Spring 程序的开发过程
    
*   通过上面两个方面的定位，我们可以产生两个模糊的概念：
    

1.  SpringBoot 开发团队认为原始的 Spring 程序初始搭建的时候可能有些繁琐，这个过程是可以简化的，那原始的 Spring 程序初始搭建过程都包含哪些东西了呢？为什么觉得繁琐呢？最基本的 Spring 程序至少有一个配置文件或配置类，用来描述 Spring 的配置信息，莫非这个文件都可以不写？此外现在企业级开发使用 Spring 大部分情况下是做 web 开发，如果做 web 开发的话，还要在加载 web 环境时加载时加载指定的 spring 配置，这都是最基本的需求了，不写的话怎么知道加载哪个配置文件/配置类呢？那换了 SpringBoot 技术以后呢，这些还要写吗？谜底稍后揭晓，先卖个关子
    
2.  SpringBoot 开发团队认为原始的 Spring 程序开发的过程也有些繁琐，这个过程仍然可以简化。开发过程无外乎使用什么技术，导入对应的 jar 包（或坐标）然后将这个技术的核心对象交给 Spring 容器管理，也就是配置成 Spring 容器管控的 bean 就可以了。这都是基本操作啊，难道这些东西 SpringBoot 也能帮我们简化？
    
3.  带着上面这些疑问我们就着手第一个 SpringBoot 程序的开发了，看看到底使用 SpringBoot 技术能简化开发到什么程度。
    
4.  **温馨提示**
    

### JC-1-1.SpringBoot 入门程序制作（一）

```
下面让我们开始做第一个SpringBoot程序吧，本课程基于Idea2020.3版本制作，使用的Maven版本为3.6.1，JDK版本为1.8。如果你的环境和上述环境不同，可能在操作界面和操作过程中略有不同，只要软件匹配兼容即可（说到这个Idea和Maven，它们两个还真不是什么版本都能搭到一起的，说多了都是泪啊）。

下面使用SpringBoot技术快速构建一个SpringMVC的程序，通过这个过程体会<font color="#ff0000"><b>简化</b></font>二字的含义
```

**步骤①**：创建新模块，选择 Spring Initializr，并配置模块相关基础信息

![](_assets/32485b5a94f1bd490dffc5d476c06131.png)

```
<font color="#ff0000"><b>特别关注</b></font>：第3步点击Next时，Idea需要联网状态才可以进入到后面那一页，如果不能正常联网，就无法正确到达右面那个设置页了，会一直<font color="#ff0000"><b>联网</b></font>转转转

<font color="#ff0000"><b>特别关注</b></font>：第5步选择java版本和你计算机上安装的JDK版本匹配即可，但是最低要求为JDK8或以上版本，推荐使用8或11
```

**步骤②**：选择当前模块需要使用的技术集

![](_assets/c3229756985bdbc8c38be840d893f3a7.png)

```
按照要求，左侧选择web，然后在中间选择Spring Web即可，选完右侧就出现了新的内容项，这就表示勾选成功了

<font color="#ff0000"><b>关注</b></font>：此处选择的SpringBoot的版本使用默认的就可以了，需要说一点，SpringBoot的版本升级速度很快，
```

**步骤③**：开发控制器类

```
@RestController
@RequestMapping("/books")
public class BookController {
    @GetMapping
    public String getById(){
        System.out.println("springboot is running...");
        return "springboot is running...";
    }
}
```

```
<font color="#ff0000"><b>关注</b></font>: 入门案例制作的SpringMVC的控制器基于Rest风格开发，当然此处使用原始格式制作SpringMVC的程序也是没有问题的，上例中的@RestController与@GetMapping注解是基于Restful开发的典型注解

<font color="#ff0000"><b>关注</b></font>：做到这里SpringBoot程序的最基础的开发已经做完了，现在就可以正常的运行Spring程序了。Tomcat服务器没有配置，Spring也没有配置，这就是SpringBoot技术的强大之处。关于内部工作流程后面再说.
```

**步骤④**：运行自动生成的 Application 类

```
使用带main方法的java程序的运行形式来运行程序，运行完毕后，控制台输出上述信息。
```

![](_assets/537c2580de3b06e58403bf70028e25d7.png)

测试功能是否工作正常.

*   pom.xml
    
*   配置中有两个信息需要关注，一个是 parent，也就是当前工程继承了另外一个工程，干什么用的后面再说，还有依赖坐标，干什么用的后面再说
    
*   Application 类
    
*   通过上面的制作，我们不难发现，SpringBoot 程序简直太好写了，几乎什么都没写，功能就有了，这也是 SpringBoot 技术为什么现在这么火的原因，和 Spirng 程序相比，SpringBoot 程序在开发的过程中各个层面均具有优势
    

```
一句话总结一下就是<font color="#ff0000"><b>能少写就少写</b></font>，<font color="#ff0000"><b>能不写就不写</b></font>，这就是SpringBoot技术给我们带来的好处。
```

**总结**

1.  开发 SpringBoot 程序可以根据向导进行联网快速制作
    
2.  SpringBoot 程序需要基于 JDK8 以上版本进行制作
    
3.  SpringBoot 程序中需要使用何种功能通过勾选选择技术，也可以手工添加对应的要使用的技术（后期讲解）
    
4.  运行 SpringBoot 程序通过运行 Application 程序入口进行
    

### JC-1-2.SpringBoot 入门程序制作（二）

IDEA 不能联网创建 SpringBoot 程序。

其实在 SpringBoot 的官网里面就可以直接创建 SpringBoot 程序

```
SpringBoot官网和Spring的官网是在一起的，都是  spring.io  。你可以通过项目一级一级的找到SpringBoot技术的介绍页，然后在页面中间部位找到如下内容
```

![](_assets/674e9061487a23ac02f9eff4cabbed16.png)

![](_assets/d1bcce1cb03929666b626f4671e1d2a4.png)

**步骤①**：点击 Spring Initializr 后进入到创建 SpringBoot 程序的界面上，下面是输入信息的过程，和前面的一样，只是界面变了而已，根据自己的要求，在左侧选择对应信息和输入对应的信息即可

![](_assets/7fd5763db1f324e235fccaf2ba08151a.png)

**步骤②**：右侧的 ADD DEPENDENCIES 用于选择使用何种技术，和之前勾选的 Spring WEB 是在做同一件事，仅仅是界面不同而已，点击后打开网页版的技术选择界面

![](_assets/ddd2cf47414042142aadcd9c6be939ca.png)

**步骤③**：所有信息设置完毕后，点击下面左侧按钮，生成一个文件包

![](_assets/b45beb1c86f723ff80367d9b372bddbe.png)

**步骤④**：保存后得到一个压缩文件，这个文件打开后就是创建的 SpringBoot 工程文件夹了

![](_assets/42dcf2e9283a2c1cdd0950349dc0dc44.png)

**步骤⑤**：解压缩此文件后，得到工程目录，在 Idea 中导入即可使用，和之前创建的东西完全一样。下面就可以自己创建一个 Controller 测试一下是否能用了。

**温馨提示**

```
Idea工具中创建SpringBoot工程其实连接的就是SpringBoot的官网，走的就是这个过程，只不过Idea把界面给整合了一下，读取到了Spring官网给的信息，然后展示到了Idea的界面中而已，不信你可以看看下面这个步骤
```

### JC-1-3.SpringBoot 入门程序制作（三）

解决无法访问 spring.io 官网，如何创建 springboot 项目

```
创建工程时，切换选择starter服务路径，然后手工收入阿里云提供给我们的使用地址即可。
地址：http://start.aliyun.com或https://start.aliyun.com
```

![](_assets/56275821bd347dd4d964436d887be207.png)

```
阿里为了便于自己开发使用，因此在依赖坐标中添加了一些阿里相关的技术，也是为了推广自己的技术吧，所以在依赖选择列表中，你有了更多的选择。
```

不过有一点需要说清楚，阿里云地址默认创建的 SpringBoot 工程版本是 **2.4.1**，所以如果你想更换其他的版本，创建项目后手工修改即可，别忘了刷新一下，加载新版本信息。

![](_assets/4e6ed965b9fc18a520003dd4d284db67.png)

```
<font color="#ff0000"><b>注意</b></font>：阿里云提供的工程创建地址初始化完毕后和实用SpringBoot官网创建出来的工程略有区别。主要是在配置文件的形式上有区别。这个信息在后面讲解Boot程序的执行流程时给大家揭晓
```

**总结**

1.  选择 start 来源为自定义 URL
    
2.  输入阿里云 start 地址
    
3.  创建项目
    

### JC-1-4.SpringBoot 入门程序制作（四）

```
不能上网，还想创建SpringBoot工程，能不能做呢？能做，但是你要先问问自己联网和不联网到底差别是什么？这个信息找到以后，你就发现，你把联网要干的事情都提前准备好，就无需联网了。

联网做什么呢？首先SpringBoot工程也是基于Maven构建的，而Maven工程当使用了一些自己需要使用又不存在的东西时，就要去下载。其实SpringBoot工程创建的时候就是去下载一些必要的组件的。你把这些东西给提前准备好就可以了吗？是的，就是这样。

下面咱们就一起手工创建一个SpringBoot工程
```

**步骤①**：创建工程时，选择手工创建 Maven 工程

![](_assets/c247dfa166456dbed2a6a3a119dd131c.png)

**步骤②**：参照标准 SpringBoot 工程的 pom 文件，书写自己的 pom 文件即可

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.4</version>
    </parent>

    <groupId>com.itheima</groupId>
    <artifactId>springboot_01_04_quickstart</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```

**步骤③**：之前运行 SpringBoot 工程需要一个类，这个缺不了，自己手写一个就行了，建议按照之前的目录结构来创建，先别玩花样，先学走后学跑。类名可以自定义，关联的名称一切修改即可

```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(<Application.class);
    }
}
```

```
<font color="#ff0000"><b>关注</b></font>：类上面的注解@SpringBootApplication千万别丢了，这个是核心，后面再介绍

<font color="#ff0000"><b>关注</b></font>：类名可以自定义，只要保障下面代码中使用的类名和你自己定义的名称一样即可，也就是run方法中的那个class对应的名称
```

**步骤④**：下面就可以自己创建一个 Controller 测试一下是否能用了，和之前没有差别了

```
看到这里其实应该能够想明白了，通过向导或者网站创建的SpringBoot工程其实就是帮你写了一些代码，而现在是自己手写，写的内容都一样，仅此而已。
```

**温馨提示**

```
如果你的计算机上从来没有创建成功过SpringBoot工程，自然也就没有下载过SpringBoot对应的坐标，那用手写创建的方式在不联网的情况下肯定该是不能用的。所谓手写，其实就是自己写别人帮你生成的东西，但是引用的坐标对应的资源必须保障maven仓库里面有才行，如果没有，还是要去下载的
```

**总结**

1.  创建普通 Maven 工程
    
2.  继承 spring-boot-starter-parent
    
3.  添加依赖 spring-boot-starter-web
    
4.  制作引导类 Application
    

到这里其实学习了 4 种创建 SpringBoot 工程的方式，其实本质是一样的，就是根据 SpringBoot 工程的文件格式要求，通过不同时方式生成或者手写得到对应的文件，效果完全一样。

### JC-1-5.SpringBoot 简介

```
SpringBoot是由Pivotal团队提供的全新框架，其设计目的是用来<font color="#ff0000"><b>简化Spring应用的初始搭建以及开发过程</b></font>。

都简化了了哪些东西呢？其实就是针对原始的Spring程序制作的两个方面进行了简化：
```

*   Spring 程序缺点
    
*   依赖设置繁琐
    
*   以前写 Spring 程序，使用的技术都要自己一个一个的写，现在不需要了，如果做过原始 SpringMVC 程序的小伙伴应该知道，写 SpringMVC 程序，最基础的 spring-web 和 spring-webmvc 这两个坐标时必须的，就这还不包含你用 json 啊等等这些坐标，现在呢？一个坐标搞定面
    
*   配置繁琐
    
*   以前写配置类或者配置文件，然后用什么东西就要自己写加载 bean 这些东西，现在呢？什么都没写，照样能用
    

SpringBoot 程序的核心功能及优点：

*   起步依赖（简化依赖配置）
    
*   依赖配置的书写简化就是靠这个起步依赖达成的
    
*   自动配置（简化常用工程相关配置）
    
*   配置过于繁琐，使用自动配置就可以做响应的简化，但是内部还是很复杂的，后面具体展开说
    
*   辅助功能（内置服务器，……）
    
*   除了上面的功能，其实 SpringBoot 程序还有其他的一些优势，比如我们没有配置 Tomcat 服务器，但是能正常运行，这是 SpringBoot 程序的一个可以感知到的功能，也是 SpringBoot 的辅助功能之一。
    
*   下面结合入门程序来说说这些简化操作都在哪些方面进行体现的，一共分为 4 个方面
    
*   parent
    
*   starter
    
*   引导类
    
*   内嵌 tomcat
    

#### parent

```
SpringBoot关注到开发者在进行开发时，往往对依赖版本的选择具有固定的搭配格式，并且这些依赖版本的选择还不能乱搭配。比如A技术的2.0版与B技术的3.5版可以合作在一起，但是和B技术的3.7版合并使用时就有冲突。其实很多开发者都一直想做一件事情，就是将各种各样的技术配合使用的常见依赖版本进行收集整理，制作出了最合理的依赖版本配置方案，这样使用起来就方便多了。

SpringBoot一看这种情况so easy啊，于是将所有的技术版本的常见使用方案都给开发者整理了出来，以后开发者使用时直接用它提供的版本方案，就不用担心冲突问题了，相当于SpringBoot做了无数个技术版本搭配的列表，这个技术搭配列表的名字叫做<font color="#ff0000"><b>parent</b></font>。

<font color="#ff0000"><b>parent</b></font>自身具有很多个版本，每个<font color="#ff0000"><b>parent</b></font>版本中包含有几百个其他技术的版本号，不同的parent间使用的各种技术的版本号有可能会发生变化。当开发者使用某些技术时，直接使用SpringBoot提供的<font color="#ff0000"><b>parent</b></font>就行了，由<font color="#ff0000"><b>parent</b></font>帮助开发者统一的进行各种技术的版本管理

比如你现在要使用Spring配合MyBatis开发，没有parent之前怎么做呢？选个Spring的版本，再选个MyBatis的版本，再把这些技术使用时关联的其他技术的版本逐一确定下来。当你Spring的版本发生变化需要切换时，你的MyBatis版本有可能也要跟着切换，关联技术呢？可能都要切换，而且切换后还可能出现问题。现在这一切工作都可以交给parent来做了。你无需关注这些技术间的版本冲突问题，你只需要关注你用什么技术就行了，冲突问题由<font color="#ff0000"><b>parent</b></font>负责处理。

有人可能会提出来，万一<font color="#ff0000"><b>parent</b></font>给我导入了一些我不想使用的依赖怎么办？记清楚，这一点很关键，<font color="#ff0000"><b>parent</b></font>仅仅帮我们进行版本管理，它不负责帮你导入坐标，说白了用什么还是你自己定，只不过版本不需要你管理了。整体上来说，<font color="#ff0000"><b>使用parent可以帮助开发者进行版本的统一管理</b></font>



那SpringBoot又是如何做到这一点的呢？可以查阅SpringBoot的配置源码，看到这些定义
```

*   项目中的 pom.xml 中继承了一个坐标
    

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.4</version>
</parent>
```

*   打开后可以查阅到其中又继承了一个坐标
    

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.5.4</version>
</parent>
```

*   这个坐标中定义了两组信息，第一组是各式各样的依赖版本号属性，下面列出依赖版本属性的局部，可以看的出来，定义了若干个技术的依赖版本号
    

```
<properties>
    <activemq.version>5.16.3</activemq.version>
    <aspectj.version>1.9.7</aspectj.version>
    <assertj.version>3.19.0</assertj.version>
    <commons-codec.version>1.15</commons-codec.version>
    <commons-dbcp2.version>2.8.0</commons-dbcp2.version>
    <commons-lang3.version>3.12.0</commons-lang3.version>
    <commons-pool.version>1.6</commons-pool.version>
    <commons-pool2.version>2.9.0</commons-pool2.version>
    <h2.version>1.4.200</h2.version>
    <hibernate.version>5.4.32.Final</hibernate.version>
    <hibernate-validator.version>6.2.0.Final</hibernate-validator.version>
    <httpclient.version>4.5.13</httpclient.version>
    <jackson-bom.version>2.12.4</jackson-bom.version>
    <javax-jms.version>2.0.1</javax-jms.version>
    <javax-json.version>1.1.4</javax-json.version>
    <javax-websocket.version>1.1</javax-websocket.version>
    <jetty-el.version>9.0.48</jetty-el.version>
    <junit.version>4.13.2</junit.version>
</properties>
```

```
第二组是各式各样的的依赖坐标信息，可以看出依赖坐标定义中没有具体的依赖版本号，而是引用了第一组信息中定义的依赖版本属性值
```

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**关注**：上面的依赖坐标定义是出现在标签中的，其实是对引用坐标的依赖管理，并不是实际使用的坐标。因此当你的项目中继承了这组 parent 信息后，在不使用对应坐标的情况下，前面的这组定义是不会具体导入某个依赖的

**关注**：因为在 maven 中继承机会只有一次，上述继承的格式还可以切换成导入的形式进行，并且在阿里云的 starter 创建工程时就使用了此种形式

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**总结**

1.  开发 SpringBoot 程序要继承 spring-boot-starter-parent
    
2.  spring-boot-starter-parent 中定义了若干个依赖管理
    
3.  继承 parent 模块可以避免多个依赖使用相同技术时出现依赖版本冲突
    
4.  继承 parent 的形式也可以采用引入依赖的形式实现效果
    

#### starter

```
SpringBoot关注到开发者在实际开发时，对于依赖坐标的使用往往都有一些固定的组合方式，比如使用spring-webmvc就一定要使用spring-web。每次都要固定搭配着写，非常繁琐，而且格式固定，没有任何技术含量。

SpringBoot一看这种情况，看来需要给开发者带来一些帮助了。安排，把所有的技术使用的固定搭配格式都给开发出来，以后你用某个技术，就不用一次写一堆依赖了，还容易写错，我给你做一个东西，代表一堆东西，开发者使用的时候，直接用我做好的这个东西就好了，对于这样的固定技术搭配，SpringBoot给它起了个名字叫做<font color="#ff0000"><b>starter</b></font>。

starter定义了使用某种技术时对于依赖的固定搭配格式，也是一种最佳解决方案，<font color="#ff0000"><b>使用starter可以帮助开发者减少依赖配置</b></font>

这个东西其实在入门案例里面已经使用过了，入门案例中的web功能就是使用这种方式添加依赖的。可以查阅SpringBoot的配置源码，看到这些定义
```

*   项目中的 pom.xml 定义了使用 SpringMVC 技术，但是并没有写 SpringMVC 的坐标，而是添加了一个名字中包含 starter 的依赖
    

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

*   在 spring-boot-starter-web 中又定义了若干个具体依赖的坐标
    

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
        <version>2.5.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-json</artifactId>
        <version>2.5.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <version>2.5.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.9</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.9</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

```
之前提到过开发SpringMVC程序需要导入spring-webmvc的坐标和spring整合web开发的坐标，就是上面这组坐标中的最后两个了。

但是我们发现除了这两个还有其他的，比如第二个，叫做spring-boot-starter-json。看名称就知道，这个是与json有关的坐标了，但是看名字发现和最后两个又不太一样，它的名字中也有starter，打开看看里面有什么？
```

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
        <version>2.5.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.9</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.12.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.datatype</groupId>
        <artifactId>jackson-datatype-jdk8</artifactId>
        <version>2.12.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.datatype</groupId>
        <artifactId>jackson-datatype-jsr310</artifactId>
        <version>2.12.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.module</groupId>
        <artifactId>jackson-module-parameter-names</artifactId>
        <version>2.12.4</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

```
我们可以发现，这个starter中又包含了若干个坐标，其实就是使用SpringMVC开发通常都会使用到Json，使用json又离不开这里面定义的这些坐标，看来还真是方便，SpringBoot把我们开发中使用的东西能用到的都给提前做好了。你仔细看完会发现，里面有一些你没用过的。的确会出现这种过量导入的可能性，没关系，可以通过maven中的排除依赖剔除掉一部分。不过你不管它也没事，大不了就是过量导入呗。

到这里基本上得到了一个信息，使用starter可以帮开发者快速配置依赖关系。以前写依赖3个坐标的，现在写导入一个就搞定了，就是加速依赖配置的。
```

**starter 与 parent 的区别**

```
<font color="#ff0000"><b>starter</b></font>是一个坐标中定了若干个坐标，以前写多个的，现在写一个，<font color="#ff0000"><b>是用来减少依赖配置的书写量的</b></font>

<font color="#ff0000"><b>parent</b></font>是定义了几百个依赖版本号，以前写依赖需要自己手工控制版本，现在由SpringBoot统一管理，这样就不存在版本冲突了，<font color="#ff0000"><b>是用来减少依赖冲突的</b></font>
```

**实际开发应用方式**

*   实际开发中如果需要用什么技术，先去找有没有这个技术对应的 starter
    
*   如果有对应的 starter，直接写 starter，而且无需指定版本，版本由 parent 提供
    
*   如果没有对应的 starter，手写坐标即可
    
*   实际开发中如果发现坐标出现了冲突现象，确认你要使用的可行的版本号，使用手工书写的方式添加对应依赖，覆盖 SpringBoot 提供给我们的配置管理
    
*   方式一：直接写坐标
    
*   方式二：覆盖中定义的版本号，就是下面这堆东西了，哪个冲突了覆盖哪个就 OK 了
    

**温馨提示**

```
SpringBoot官方给出了好多个starter的定义，方便我们使用，而且名称都是如下格式
```

```
命名规则：spring-boot-starter-技术名称
```

```
所以以后见了spring-boot-starter-aaa这样的名字，这就是SpringBoot官方给出的starter定义。那非官方定义的也有吗？有的，具体命名方式到整合章节再说
```

**总结**

1.  开发 SpringBoot 程序需要导入坐标时通常导入对应的 starter
    
2.  每个不同的 starter 根据功能不同，通常包含多个依赖坐标
    
3.  使用 starter 可以实现快速配置的效果，达到简化配置的目的
    

#### 引导类

```
配置说完了，我们发现SpringBoot确实帮助我们减少了很多配置工作，下面说一下程序是如何运行的。目前程序运行的入口就是SpringBoot工程创建时自带的那个类了，带有main方法的那个类，运行这个类就可以启动SpringBoot工程的运行
```

```
@SpringBootApplication
public class Springboot0101QuickstartApplication {
    public static void main(String[] args) {
        SpringApplication.run(Springboot0101QuickstartApplication.class, args);
    }
}
```

```
SpringBoot本身是为了加速Spring程序的开发的，而Spring程序运行的基础是需要创建自己的Spring容器对象（IoC容器）并将所有的对象交给Spring的容器管理，也就是一个一个的Bean。那还了SpringBoot加速开发Spring程序，这个容器还在吗？这个疑问不用说，一定在。<font color='red'>当前这个类运行后就会产生一个Spring容器对象，并且可以将这个对象保存起来，通过容器对象直接操作Bean。
```

```
@SpringBootApplication
public class Springboot0101QuickstartApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext ctx = SpringApplication.run(Springboot0101QuickstartApplication.class, args);
        BookController bean = ctx.getBean(BookController.class);
        System.out.println("bean======>" + bean);
    }
}
```

```
通过上述操作不难看出，其实SpringBoot程序启动还是创建了一个Spring容器对象。这个类在SpringBoot程序中是所有功能的入口，称这个类为<font color="#ff0000"><b>引导类</b></font>。

作为一个引导类最典型的特征就是当前类上方声明了一个注解<font color="#ff0000"><b>@SpringBootApplication</b></font>
```

**总结**

1.  SpringBoot 工程提供引导类用来启动程序
    
2.  SpringBoot 工程启动后创建并初始化 Spring 容器
    

#### 内嵌 tomcat

```
当前我们做的SpringBoot入门案例勾选了Spirng-web的功能，并且导入了对应的starter。
```

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```
由于这个功能不属于程序的主体功能，可用可不用，于是乎SpringBoot将其定位成辅助功能，别小看这么一个辅助功能，它可是帮我们开发者又减少了好多的设置性工作。

下面就围绕着这个内置的web服务器，也可以说是内置的tomcat服务器来研究几个问题
```

1.  这个服务器在什么位置定义的
    
2.  这个服务器是怎么运行的
    
3.  这个服务器如果想换怎么换？虽然这个需求很垃圾，搞得开发者会好多 web 服务器一样，用别人提供好的不香么？非要自己折腾
    

**内嵌 Tomcat 定义位置**

```
说到定义的位置，我们就想，如果我们不开发web程序，用的着web服务器吗？肯定用不着啊。那如果这个东西被加入到你的程序中，伴随着什么技术进来的呢？肯定是web相关的功能啊，没错，就是前面导入的web相关的starter做的这件事。
```

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
        <version>2.5.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-json</artifactId>
        <version>2.5.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <version>2.5.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.9</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.9</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

```
第三个依赖就是这个tomcat对应的东西了，居然也是一个starter，再打开看看
```

```
<dependencies>
    <dependency>
        <groupId>jakarta.annotation</groupId>
        <artifactId>jakarta.annotation-api</artifactId>
        <version>1.3.5</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-core</artifactId>
        <version>9.0.52</version>
        <scope>compile</scope>
        <exclusions>
            <exclusion>
                <artifactId>tomcat-annotations-api</artifactId>
                <groupId>org.apache.tomcat</groupId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-el</artifactId>
        <version>9.0.52</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-websocket</artifactId>
        <version>9.0.52</version>
        <scope>compile</scope>
        <exclusions>
            <exclusion>
                <artifactId>tomcat-annotations-api</artifactId>
                <groupId>org.apache.tomcat</groupId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

```
这里面有一个核心的坐标，<font color='red'>tomcat-embed-core</font>，叫做tomcat内嵌核心。就是这个东西把tomcat功能引入到了我们的程序中。目前解决了第一个问题，找到根儿了，谁把tomcat引入到程序中的？spring-boot-starter-web中的spring-boot-starter-tomcat做的。之所以你感觉很奇妙的原因就是，这个东西是默认加入到程序中了，所以感觉很神奇，居然什么都不做，就有了web服务器对应的功能，再来说第二个问题，这个服务器是怎么运行的
```

**内嵌 Tomcat 运行原理**

```
Tomcat服务器是一款软件，而且是一款使用java语言开发的软件，熟悉的小伙伴可能有印象，tomcat安装目录中保存有jar，好多个jar。

tomcat服务器运行其实是以对象的形式在Spring容器中运行的，怪不得我们没有安装这个tomcat，而且还能用。是以一个对象的形式存在，保存在Spring容器中悄悄运行的。具体运行的是什么呢？其实就是上前面提到的那个tomcat内嵌核心
```

```
<dependencies>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-core</artifactId>
        <version>9.0.52</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

```
那既然是个对象，如果把这个对象从Spring容器中去掉是不是就没有web服务器的功能呢？是这样的，通过依赖排除可以去掉这个web服务器功能
```

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

```
上面对web-starter做了一个操作，使用maven的排除依赖去掉了使用tomcat的starter。这下好了，容器中肯定没有这个对象了，重新启动程序可以观察到程序运行了，但是并没有像之前那样运行后会等着用户发请求，而是直接停掉了，就是这个原因了。
```

**更换内嵌 Tomcat**

```
那根据上面的操作我们思考是否可以换个服务器呢？根据SpringBoot的工作机制，用什么技术，加入什么依赖就行了。SpringBoot提供了3款内置的服务器
```

*   tomcat(默认)：apache 出品，粉丝多，应用面广，负载了若干较重的组件
    
*   jetty：更轻量级，负载性能远不及 tomcat
    
*   undertow：负载性能勉强跑赢 tomcat
    
*   想用哪个，加个坐标就 OK。前提是把 tomcat 排除掉，因为 tomcat 是默认加载的。
    

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
    </dependency>
</dependencies>
```

```
现在就已经成功替换了web服务器，核心思想就是用什么加入对应坐标就可以了。如果有starter，优先使用starter。
```

**总结**

1.  内嵌 Tomcat 服务器是 SpringBoot 辅助功能之一
    
2.  内嵌 Tomcat 工作原理是将 Tomcat 服务器作为对象运行，并将该对象交给 Spring 容器管理
    
3.  变更内嵌服务器思想是去除现有服务器，添加全新的服务器
    
4.  到这里第一章快速上手 SpringBoot 就结束了，这一章我们学习了两大块知识
    
5.  使用了 4 种方式制作了 SpringBoot 的入门程序，不管是哪一种，其实内部都是一模一样的
    
6.  学习了入门程序的工作流程，知道什么是 parent，什么是 starter，这两个东西是怎么配合工作的，以及我们的程序为什么启动起来是一个 tomcat 服务器等等
    

JC-2.SpringBoot 基础配置
--------------------

```
入门案例做完了，下面就要研究SpringBoot的用法了。通过入门案例，各位小伙伴能够感知到一个信息，SpringBoot没有具体的功能，它在辅助加快Spring程序的开发效率。我们发现现在几乎不用做任何的配置，功能就有了，确实很好用。但是仔细想想，没有做配置意味着什么？意味着配置已经做好了，不用你自己写了。但是新的问题又来了，如果不想用已经写好的默认配置，该如何干预呢？这就是这一章咱们要研究的问题。

如果我们想修改默认的配置i，这个信息应该写在什么位置呢？目前我们接触的入门案例中一共有3个文件，第一是pom.xml文件，设置项目的依赖的。第二是引导类，这个是执行SpringBoot程序的入口，也不像是做配置的地方，其实还有一个信息，就是在resources目录下面有一个空白的文件，叫做application.properties。一看就是个配置文件，咱们这一章就来说说配置文件怎么写，能写什么，怎么干预SpringBoot的默认配置，修改成自己的配置。
```

​

### JC-2-1.属性配置

```
SpringBoot通过配置文件application.properties就可以修改默认的配置，那咱们就先找个简单的配置下手，当前访问tomcat的默认端口是8080，好熟悉的味道，但是不便于书写，我们先改成80，通过这个操作来熟悉一下SpringBoot的配置格式是什么样的
```

![](_assets/8bda005f55a80778545c76b94496d3d2.png)

```
那该如何写呢？properties格式的文件书写规范是key=value
```

```
这个格式肯定是不能颠覆的，那就尝试性的写就行了，改端口，写port。当你输入port后，神奇的事情就发生了，这玩意儿带提示，太好了
```

![](_assets/19cad217ca2820cb3aa6828143d5145a.png)

```
下面就可以直接运行程序，测试效果了。



其实到这里我们应该得到如下三个信息
```

1.  SpringBoot 程序可以在 application.properties 文件中进行属性配置
    
2.  application.properties 文件中只要输入要配置的属性关键字就可以根据提示进行设置
    
3.  SpringBoot 将配置信息集中在一个文件中写，不管你是服务器的配置，还是数据库的配置，总之都写在一起，逃离一个项目十几种配置文件格式的尴尬局面
    

**总结**

1.  SpringBoot 默认配置文件是 application.properties
    

**关闭运行日志图表（banner)**

```
spring.main.banner-mode=off
```

**设置运行日志的显示级别**

```
你会发现，现在这么搞配置太爽了，以前你做配置怎么做？不同的技术有自己专用的配置文件，文件不同格式也不统一，现在呢？不用东奔西走的找配置文件写配置了，统一格式了。

打开SpringBoot的官网，找到SpringBoot官方文档，打开查看附录中的Application Properties就可以获取到对应的配置项了，网址奉上：https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties

能写什么的问题解决了，再来说第二个问题，这个配置项和什么有关。在pom中注释掉导入的spring-boot-starter-web，然后刷新工程，你会发现配置的提示消失了。闹了半天是设定使用了什么技术才能做什么配置。也合理，不然配置的东西都没有使用对应技术，配了也是白配。
```

**温馨提示**

```
所有的starter中都会依赖下面这个starter，叫做spring-boot-starter。这个starter是所有的SpringBoot的starter的基础依赖，里面定义了SpringBoot相关的基础配置，关于这个starter我们到开发应用篇和原理篇中再深入讲解。
```

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.5.4</version>
    <scope>compile</scope>
</dependency>
```

**总结**

1.  SpringBoot 中导入对应 starter 后，提供对应配置属性
    
2.  书写 SpringBoot 配置采用关键字+提示形式书写
    

### JC-2-2.配置文件分类

```
现在已经能够进行SpringBoot相关的配置了，但是properties格式的配置写起来总是觉得看着不舒服，所以就期望存在一种书写起来更简便的配置格式提供给开发者使用。SpringBoot除了支持properties格式的配置文件，还支持另外两种格式的配置文件。分别如下:
```

*   properties 格式
    
*   yml 格式
    
*   yaml 格式
    
*   一看到全新的文件格式，各位小伙伴肯定想，这下又要学习新的语法格式了。SpringBoot 的配置在 Idea 工具下有提示啊，跟着提示走就行了。下面列举三种不同文件格式配置相同的属性范例，先了解一下
    
*   application.properties（properties 格式）
    

*   application.yml（yml 格式）
    

*   application.yaml（yaml 格式）
    

```
仔细看会发现yml格式和yaml格式除了文件名后缀不一样，格式完全一样，是这样的，yml和yaml文件格式就是一模一样的，只是文件后缀不同，所以可以合并成一种格式来看。那对于这三种格式来说，以后用哪一种比较多呢？记清楚，以后基本上都是用yml格式的，本课程后面的所有知识都是基于yml格式来制作的，以后在企业开发过程中用这个格式的机会也最多，一定要重点掌握。
```

**总结**

1.  SpringBoot 提供了 3 种配置文件的格式
    
2.  properties（传统格式/默认格式）
    
3.  **yml**（主流格式）
    
4.  yaml
    

#### 配置文件优先级

```
其实三个文件如果共存的话，谁生效说的就是配置文件加载的优先级别。先说一点，虽然以后这种情况很少出现，但是这个知识还是可以学习一下的。我们就让三个配置文件书写同样的信息，比如都配置端口，然后我们让每个文件配置的端口号都不一样，最后启动程序后看启动端口是多少就知道谁的加载优先级比较高了。
```

*   application.properties（properties 格式）
    

*   application.yml（yml 格式）
    

*   application.yaml（yaml 格式）
    

```
启动后发现目前的启动端口为80，把80对应的文件删除掉，然后再启动，现在端口又改成了81。现在我们就已经知道了3个文件的加载优先顺序是什么
```

```
application.properties  >  application.yml  >  application.yaml
```

```
虽然得到了一个知识结论，但是我们实际开发的时候还是要看最终的效果为准。也就是你要的最终效果是什么自己是明确的，上述结论只能帮助你分析结论产生的原因。这个知识了解一下就行了，因为以后同时写多种配置文件格式的情况实在是较少。

最后我们把配置文件内容给修改一下
```

*   application.properties（properties 格式）
    

```
server.port=80
spring.main.banner-mode=off
```

*   application.yml（yml 格式）
    

```
server:
  port: 81
logging: 
  level: 
    root: debug
```

*   application.yaml（yaml 格式）
    

```
我们发现不仅端口生效了，最终显示80，同时其他两条配置也生效了，看来每个配置文件中的项都会生效，只不过如果多个配置文件中有相同类型的配置会优先级高的文件覆盖优先级的文件中的配置。如果配置项不同的话，那所有的配置项都会生效。
```

**总结**

1.  配置文件间的加载优先级 properties（最高）> yml > yaml（最低）
    
2.  不同配置文件中相同配置按照加载优先级相互覆盖，不同配置文件中不同配置全部保留
    

### JC-2-3.yaml 文件

```
SpringBoot的配置以后主要使用yml结尾的这种文件格式，并且在书写时可以通过提示的形式加载正确的格式。但是这种文件还是有严格的书写格式要求的。下面就来说一下具体的语法格式。

YAML（YAML Ain't Markup Language），一种数据序列化格式。具有容易阅读、容易与脚本语言交互、以数据为核心，重数据轻格式的特点。常见的文件扩展名有两种：
```

*   .yml 格式（主流）
    
*   .yaml 格式
    
*   对于文件自身在书写时，具有严格的语法格式要求，具体如下：
    

1.  大小写敏感
    
2.  属性层级关系使用多行描述，**每行结尾使用冒号结束**
    
3.  使用缩进表示层级关系，同层级左侧对齐，只允许使用空格（不允许使用 Tab 键）
    
4.  属性值前面添加空格（属性名与属性值之间使用冒号+空格作为分隔）
    
5.  #号 表示注释
    

上述规则不要死记硬背，按照书写习惯慢慢适应，并且在 Idea 下由于具有提示功能，慢慢适应着写格式就行了。核心的一条规则要记住，**数据前面要加空格与冒号隔开**

```
boolean: TRUE              
float: 3.14                
int: 123                   
null: ~                    
string: HelloWorld            
string2: "Hello World"        
date: 2018-02-17              
datetime: 2018-02-17T15:02:31+08:00
```

```
此外，yaml格式中也可以表示数组，在属性名书写位置的下方使用减号作为数据开始符号，每行书写一个数据，减号与数据间空格分隔
```

```
subject:
    - Java
    - 前端
    - 大数据
enterprise:
    name: itcast
    age: 16
    subject:
        - Java
        - 前端
        - 大数据
likes: [王者荣耀,刺激战场]      
users:               
  - name: Tom
       age: 4
  - name: Jerry
    age: 5
users:               
  -  
    name: Tom
    age: 4
  -   
    name: Jerry
    age: 5          
users2: [ { name:Tom , age:4 } , { name:Jerry , age:5 } ]
```

**总结**

1.  yaml 语法规则
    
2.  大小写敏感
    
3.  属性层级关系使用多行描述，每行结尾使用冒号结束
    
4.  使用缩进表示层级关系，同层级左侧对齐，只允许使用空格（不允许使用 Tab 键）
    
5.  属性值前面添加空格（属性名与属性值之间使用冒号+空格作为分隔）
    
6.  #号 表示注释
    
7.  注意属性名冒号后面与数据之间有一个**空格**
    
8.  字面值、对象数据格式、数组数据格式
    

### JC-2-4.yaml 数据读取

```
对于yaml文件中的数据，其实你就可以想象成这就是一个小型的数据库，里面保存有若干数据，每个数据都有一个独立的名字，如果你想读取里面的数据，肯定是支持的，下面就介绍3种读取数据的方式
```

#### 读取单一数据

```
yaml中保存的单个数据，可以使用Spring中的注解直接读取，使用@Value可以读取单个数据，属性名引用方式：<font color="#ff0000"><b>${一级属性名.二级属性名……}</b></font>
```

![](_assets/989f94114e64381edb6ec59ab53f5338.png)

```
记得使用@Value注解时，要将该注入写在某一个指定的Spring管控的bean的属性名上方。现在就可以读取到对应的单一数据行了
```

**总结**

1.  使用 @Value 配合 SpEL 读取单个数据
    
2.  如果数据存在多层级，依次书写层级名称即可
    

#### 读取全部数据

```
读取单一数据可以解决读取数据的问题，但是如果定义的数据量过大，这么一个一个书写肯定会累死人的，SpringBoot提供了一个对象，能够把所有的数据都封装到这一个对象中，这个对象叫做Environment，使用自动装配注解可以将所有的yaml数据封装到这个对象中
```

![](_assets/aaa363ec9f4381a472b6e0419cbba173.png)

```
数据封装到了Environment对象中，获取属性时，通过Environment的接口操作进行，具体方法时getProperties（String），参数填写属性名即可
```

**总结**

1.  使用 Environment 对象封装全部配置信息
    
2.  使用 @Autowired 自动装配数据到 Environment 对象中
    

#### 读取对象数据

```
单一数据读取书写比较繁琐，全数据封装又封装的太厉害了，每次拿数据还要一个一个的getProperties（）,总之用起来都不是很舒服。由于Java是一个面向对象的语言，很多情况下，我们会将一组数据封装成一个对象。SpringBoot也提供了可以将一组yaml对象数据封装一个Java对象的操作

首先定义一个对象，并将该对象纳入Spring管控的范围，也就是定义成一个bean，然后使用注解@ConfigurationProperties指定该对象加载哪一组yaml中配置的信息。
```

![](_assets/0316bfd35f5650259844561e9450cd52.png)

```
这个@ConfigurationProperties必须告诉他加载的数据前缀是什么，这样当前前缀下的所有属性就封装到这个对象中。记得数据属性名要与对象的变量名一一对应啊，不然没法封装。其实以后如果你要定义一数据自己使用，就可以先写一个对象，然后定义好属性，下面到配置中根据这个格式书写即可。
```

![](_assets/48ce4453e4aec2f2261f824ee83ff07d.png)

**总结**

1.  使用 @ConfigurationProperties 注解绑定配置信息到封装类中
    
2.  封装类需要定义为 Spring 管理的 bean，否则无法进行属性注入
    

#### yaml 文件中的数据引用

```
如果你在书写yaml数据时，经常出现如下现象，比如很多个文件都具有相同的目录前缀
```

```
center:
    dataDir: /usr/local/fire/data
    tmpDir: /usr/local/fire/tmp
    logDir: /usr/local/fire/log
    msgDir: /usr/local/fire/msgDir
```

或者

```
center:
    dataDir: D:/usr/local/fire/data
    tmpDir: D:/usr/local/fire/tmp
    logDir: D:/usr/local/fire/log
    msgDir: D:/usr/local/fire/msgDir
```

```
这个时候你可以使用引用格式来定义数据，其实就是搞了个变量名，然后引用变量了，格式如下：
```

```
baseDir: /usr/local/fire
    center:
    dataDir: ${baseDir}/data
    tmpDir: ${baseDir}/tmp
    logDir: ${baseDir}/log
    msgDir: ${baseDir}/msgDir
```

```
还有一个注意事项，在书写字符串时，如果需要使用转义字符，需要将数据字符串使用双引号包裹起来
```

```
lesson: "Spring\tboot\nlesson"
```

**总结**

1.  在配置文件中可以使用 ${属性名}方式引用属性值
    
2.  如果属性中出现特殊字符，可以使用双引号包裹起来作为字符解析
    
3.  到这里有关 yaml 文件的基础使用就先告一段落，在实用篇中再继续研究更深入的内容。
    

JC-3.基于 SpringBoot 实现 SSMP 整合
-----------------------------

```
重头戏来了，SpringBoot之所以好用，就是它能方便快捷的整合其他技术，这一部分咱们就来聊聊一些技术的整合方式，通过这一章的学习，大家能够感受到SpringBoot到底有多酷炫。这一章咱们学习如下技术的整合方式
```

*   整合 JUnit
    
*   整合 MyBatis
    
*   整合 MyBatis-Plus
    
*   整合 Druid
    
*   上面这些技术都整合完毕后，我们做一个小案例，也算是学有所用吧。涉及的技术比较多，综合运用一下。
    

### JC-3-1.整合 JUnit

```
SpringBoot技术的定位用于简化开发，再具体点是简化Spring程序的开发。所以在整合任意技术的时候，如果你想直观感触到简化的效果，你必须先知道使用非SpringBoot技术时对应的整合是如何做的，然后再看基于SpringBoot的整合是如何做的，才能比对出来简化在了哪里。

我们先来看一下不使用SpringBoot技术时，Spring整合JUnit的制作方式
```

```
@RunWith(SpringJUnit4ClassRunner.class)

@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTestCase {
    
    @Autowired
    private AccountService accountService;
    @Test
    public void testGetById(){
        
        System.out.println(accountService.findById(2));
    }
}
```

```
其中核心代码是前两个注解，第一个注解@RunWith是设置Spring专用于测试的类运行器，简单说就是Spring程序执行程序有自己的一套独立的运行程序的方式，不能使用JUnit提供的类运行方式了，必须指定一下，但是格式是固定的，琢磨一下，<font color="#ff0000"><b>每次都指定一样的东西，这个东西写起来没有技术含量啊</b></font>，第二个注解@ContextConfiguration是用来设置Spring核心配置文件或配置类的，简单说就是加载Spring的环境你要告诉Spring具体的环境配置是在哪里写的，虽然每次加载的文件都有可能不同，但是仔细想想，如果文件名是固定的，这个貌似也是一个固定格式。似然<font color="#ff0000"><b>有可能是固定格式，那就有可能每次都写一样的东西，也是一个没有技术含量的内容书写</b></font>

SpringBoot就抓住上述两条没有技术含量的内容书写进行开发简化，能走默认值的走默认值，能不写的就不写，具体格式如下
```

```
@SpringBootTest
class Springboot04JunitApplicationTests {
    
    @Autowired
    private BookDao bookDao;
    @Test
    void contextLoads() {
        
        bookDao.save();
        System.out.println("two...");
    }
}
```

```
看看这次简化成什么样了，一个注解就搞定了，而且还没有参数，再体会SpringBoot整合其他技术的优势在哪里，就两个字——<font color="#ff0000"><b>简化</b></font>。使用一个注解@SpringBootTest替换了前面两个注解。至于内部是怎么回事？和之前一样，只不过都走默认值。

这个时候有人就问了，你加载的配置类或者配置文件是哪一个？就是我们前面启动程序使用的引导类。如果想手工指定引导类有两种方式，第一种方式使用属性的形式进行，在注解@SpringBootTest中添加classes属性指定配置类
```

```
@SpringBootTest(classes = Springboot04JunitApplication.class)
class Springboot04JunitApplicationTests {
    
    @Autowired
    private BookDao bookDao;
    @Test
    void contextLoads() {
        
        bookDao.save();
        System.out.println("two...");
    }
}
```

```
第二种方式回归原始配置方式，仍然使用@ContextConfiguration注解进行，效果是一样的
```

```
@SpringBootTest
@ContextConfiguration(classes = Springboot04JunitApplication.class)
class Springboot04JunitApplicationTests {
    
    @Autowired
    private BookDao bookDao;
    @Test
    void contextLoads() {
        
        bookDao.save();
        System.out.println("two...");
    }
}
```

```
<font color="#f0f"><b>温馨提示</b></font>

    使用SpringBoot整合JUnit需要保障导入test对应的starter，由于初始化项目时此项是默认导入的，所以此处没有提及，其实和之前学习的内容一样，用什么技术导入对应的starter即可。
```

**总结**

1.  导入测试对应的 starter
    
2.  测试类使用 @SpringBootTest 修饰
    
3.  使用自动装配的形式添加要测试的对象
    
4.  测试类如果存在于引导类所在包或子包中无需指定引导类
    
5.  测试类如果不存在于引导类所在的包或子包中需要通过 classes 属性指定引导类
    

### JC-3-2.整合 MyBatis

```
整合完JUnit下面再来说一下整合MyBatis，这个技术是大部分公司都要使用的技术，务必掌握。如果对Spring整合MyBatis不熟悉的小伙伴好好复习一下，下面列举出原始整合的全部内容，以配置类的形式为例进行
```

*   导入坐标，MyBatis 坐标不能少，Spring 整合 MyBatis 还有自己专用的坐标，此外 Spring 进行数据库操作的 jdbc 坐标是必须的，剩下还有 mysql 驱动坐标，本例中使用了 Druid 数据源，这个倒是可以不要
    
*   Spring 核心配置
    
*   MyBatis 要交给 Spring 接管的 bean
    
*   数据源对应的 bean，此处使用 Druid 数据源
    
*   数据库连接信息（properties 格式）
    
*   上述格式基本上是简格式了，要写的东西还真不少。
    
*   下面看看 SpringBoot 整合 MyBaits 格式
    

**步骤①**：创建模块时勾选要使用的技术，MyBatis，由于要操作数据库，还要勾选对应数据库

![](_assets/be8c3ac470969db02378e07643dbf522.png)

![](_assets/dc6cdd59c5f631e6c5e986067ff36a5a.png)

```
或者手工导入对应技术的starter，和对应数据库的坐标
```

```
<dependencies>
    
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.2.0</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

**步骤②**：配置数据源相关信息，没有这个信息你连接哪个数据库都不知道

```
#2.配置相关信息
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql:
    username: root
    password: root
```

```
完了，就这么多，没了。有人就很纳闷，这就结束了？对，这就结束了，SpringBoot把配置中所有可能出现的通用配置都简化了。下面就可以写一下MyBatis程序运行需要的Dao（或者Mapper）就可以运行了
```

**实体类**

```
public class Book {
    private Integer id;
    private String type;
    private String name;
    private String description;
}
```

**映射接口（Dao）**

```
@Mapper
public interface BookDao {
    @Select("select * from tbl_book where id = #{id}")
    public Book getById(Integer id);
}
```

**测试类**

```
@SpringBootTest
class Springboot05MybatisApplicationTests {
    @Autowired
    private BookDao bookDao;
    @Test
    void contextLoads() {
        System.out.println(bookDao.getById(1));
    }
}
```

```
完美，开发从此变的就这么简单。再体会一下SpringBoot如何进行第三方技术整合的，是不是很优秀？具体内部的原理到原理篇再展开讲解

<font color="#ff0000"><b>注意</b></font>：当前使用的SpringBoot版本是2.5.4，对应的坐标设置中Mysql驱动使用的是8x版本。当SpringBoot2.4.3（不含）版本之前会出现一个小BUG，就是MySQL驱动升级到8以后要求强制配置时区，如果不设置会出问题。解决方案很简单，驱动url上面添加上对应设置就行了
```

```
#2.配置相关信息
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql:
    username: root
    password: root
```

```
这里设置的UTC是全球标准时间，你也可以理解为是英国时间，中国处在东八区，需要在这个基础上加上8小时，这样才能和中国地区的时间对应的，也可以修改配置不写UTC，写Asia/Shanghai也可以解决这个问题。
```

```
#2.配置相关信息
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql:
    username: root
    password: root
```

```
如果不想每次都设置这个东西，也可以去修改mysql中的配置文件mysql.ini，在mysqld项中添加default-time-zone=+8:00也可以解决这个问题。其实方式方法很多，这里就说这么多吧。

此外在运行程序时还会给出一个提示，说数据库驱动过时的警告，根据提示修改配置即可，弃用**com.mysql.jdbc.Driver**，换用<font color="#ff0000"><b>com.mysql.cj.jdbc.Driver</b></font>。前面的例子中已经更换了驱动了，在此说明一下。
```

```
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
```

**总结**

1.  整合操作需要勾选 MyBatis 技术，也就是导入 MyBatis 对应的 starter
    
2.  数据库连接相关信息转换成配置
    
3.  数据库 SQL 映射需要添加 @Mapper 被容器识别到
    
4.  MySQL 8.X 驱动强制要求设置时区
    
5.  修改 url，添加 serverTimezone 设定
    
6.  修改 MySQL 数据库配置
    
7.  驱动类过时，提醒更换为 com.mysql.cj.jdbc.Driver
    

### JC-3-3.整合 MyBatis-Plus

```
做完了两种技术的整合了，各位小伙伴要学会总结，我们做这个整合究竟哪些是核心？总结下来就两句话
```

*   导入对应技术的 starter 坐标
    
*   根据对应技术的要求做配置
    
*   虽然看起来有点虚，但是确实是这个理儿，下面趁热打铁，再换一个技术，看看是不是上面这两步。
    
*   接下来在 MyBatis 的基础上再升级一下，整合 MyBaitsPlus（简称 MP），国人开发的技术，符合中国人开发习惯，谁用谁知道。来吧，一起做整合
    

**步骤①**：导入对应的 starter

```
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3</version>
</dependency>
```

```
关于这个坐标，此处要说明一点，之前我们看的starter都是spring-boot-starter-？？？，也就是说都是下面的格式
```

```
而这个坐标的名字书写比较特殊，是第三方技术名称在前，boot和starter在后。此处简单提一下命名规范，后期原理篇会再详细讲解
```

**温馨提示**

```
有些小伙伴在创建项目时想通过勾选的形式找到这个名字，别翻了，没有。截止目前，SpringBoot官网还未收录此坐标，而我们Idea创建模块时读取的是SpringBoot官网的Spring Initializr，所以也没有。如果换用阿里云的url创建项目可以找到对应的坐标
```

**步骤②**：配置数据源相关信息

```
#2.配置相关信息
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql:
    username: root
    password: root
```

```
没了，就这么多，剩下的就是写MyBaitsPlus的程序了
```

**映射接口（Dao）**

```
@Mapper
public interface BookDao extends BaseMapper<Book> {
}
```

```
核心在于Dao接口继承了一个BaseMapper的接口，这个接口中帮助开发者预定了若干个常用的API接口，简化了通用API接口的开发工作。
```

![](_assets/ead70224bf26cc2491397c08004e026f.png)

**温馨提示**

```
目前数据库的表名定义规则是tbl_模块名称，为了能和实体类相对应，需要做一个配置，相关知识各位小伙伴可以到MyBatisPlus课程中去学习，此处仅给出解决方案。配置application.yml文件，添加如下配置即可，设置所有表名的通用前缀名
```

```
mybatis-plus:
  global-config:
    db-config:
      table-prefix: tbl_    #设置所有表的通用前缀名称为tbl_
```

**总结**

1.  手工添加 MyBatis-Plus 对应的 starter
    
2.  数据层接口使用 BaseMapper 简化开发
    
3.  需要使用的第三方技术无法通过勾选确定时，需要手工添加坐标
    

### JC-3-4.整合 Druid

```
使用SpringBoot整合了3个技术了，发现套路基本相同，导入对应的starter，然后做配置，各位小伙伴需要一直强化这套思想。下面再整合一个技术，继续深入强化此思想。

前面整合MyBatis和MP的时候，使用的数据源对象都是SpringBoot默认的数据源对象，下面我们手工控制一下，自己指定了一个数据源对象，Druid。

在没有指定数据源时，我们的配置如下：
```

```
#2.配置相关信息
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql:
    username: root
    password: root
```

```
此时虽然没有指定数据源，但是根据SpringBoot的德行，肯定帮我们选了一个它认为最好的数据源对象，这就是HiKari。通过启动日志可以查看到对应的身影。
```

```
2021-11-29 09:39:15.202  INFO 12260 
2021-11-29 09:39:15.208  WARN 12260 
2021-11-29 09:39:15.551  INFO 12260
```

```
上述信息中每一行都有HiKari的身影，如果需要更换数据源，其实只需要两步即可。
```

1.  导入对应的技术坐标
    
2.  配置使用指定的数据源类型
    
3.  下面就切换一下数据源对象
    

**步骤①**：导入对应的坐标（注意，是坐标，此处不是 starter）

```
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.16</version>
    </dependency>
</dependencies>
```

**步骤②**：修改配置，在数据源配置中有一个 type 属性，专用于指定数据源类型

```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql:
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource
```

```
这里其实要提出一个问题的，目前的数据源配置格式是一个通用格式，不管你换什么数据源都可以用这种形式进行配置。但是新的问题又来了，如果对数据源进行个性化的配置，例如配置数据源对应的连接数量，这个时候就有新的问题了。每个数据源技术对应的配置名称都一样吗？肯定不是啊，各个厂商不可能提前商量好都写一样的名字啊，怎么办？就要使用专用的配置格式了。这个时候上面这种通用格式就不能使用了，怎么办？还能怎么办？按照SpringBoot整合其他技术的通用规则来套啊，导入对应的starter，进行相应的配置即可。
```

**步骤①**：导入对应的 starter

```
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.2.6</version>
    </dependency>
</dependencies>
```

**步骤②**：修改配置

```
spring:
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql:
      username: root
      password: root
```

```
注意观察，配置项中，在datasource下面并不是直接配置url这些属性的，而是先配置了一个druid节点，然后再配置的url这些东西。言外之意，url这个属性时druid下面的属性，那你能想到吗？除了这4个常规配置外，还有druid专用的其他配置。通过提示功能可以打开druid相关的配置查阅
```

![](_assets/b85d78b56d5f90a1d83d942a920afbad.png)

```
与druid相关的配置超过200条以上，这就告诉你，如果想做druid相关的配置，使用这种格式就可以了，这里就不展开描述了，太多了。

这是我们做的第4个技术的整合方案，还是那两句话：<font color="#ff0000"><b>导入对应starter，使用对应配置</b></font>。没了，SpringBoot整合其他技术就这么简单粗暴。
```

**总结**

1.  整合 Druid 需要导入 Druid 对应的 starter
    
2.  根据 Druid 提供的配置方式进行配置
    
3.  整合第三方技术通用方式
    
4.  导入对应的 starter
    
5.  根据提供的配置格式，配置非默认值对应的配置项
    

> **总结：springboot 简化开发，配置更简单，希望对您有帮助，感谢阅读**
> 
> **结束语：裸体一旦成为艺术，便是最圣洁的。道德一旦沦为虚伪，便是最下流的。勇敢去做你认为正确的事，不要被世俗的流言蜚语所困扰。**