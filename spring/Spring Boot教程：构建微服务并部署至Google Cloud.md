# Spring Boot教程：构建微服务并部署至Google Cloud
**本文要点**

*   组合使用 Google Kubernetes Engine（GKE）和 Spring Boot，能够让我们更加轻松快速地搭建微服务。
    
*   Jib 是容器化 Java 应用的好办法。借助 Maven 或 Gradle，它能够让我们在不使用 Docker 的情况下，就能创建优化的镜像。
    
*   Google 的 Spring Cloud GCP 实现允许开发人员通过很少的配置和某些 Spring 模式使用 Google Cloud Platform（GCP）提供的服务。
    
*   使用 Cloud Code 搭建 Skaffold 能够让开发人员有一个很棒的开发周期。对于初始化一个新的服务原型时，这特别有用。
    

引言
--

随着微服务在我们行业中的日益普及，在如何构建应用方面，相关的技术和平台得到了繁荣发展。有时候，开始该选用哪些技术是很难抉择的。在本文中，我将会向你展示如何创建一个基于 Spring Boot 的应用，该应用会用到 Google Cloud 所提供的一些服务。这些方法我们在 Google 已经用了很长时间了。我希望它对你有用。

基础
--

我们首先来定义一下要做些什么。我们会从一个非常基础的基于[Spring Boot](https://spring.io/projects/spring-boot)的 Java 应用开始。Spring 是一个成熟的框架，借助它，我们能够快速创建强大且特性丰富的应用。

随后，我们会对其进行一些修改，使用[Jib](https://github.com/GoogleContainerTools/jib)（能够在不使用 Docker 的情况下，为 Java 应用构建优化的 Docker 和 OCI 镜像）和 distroless 版本的 Java 11 容器化该应用。Jib 能够与 Maven 和 Gradle 协作，在样例中我们会使用 Maven。

接下来，我们会创建一个 Google Cloud Platform（GCP）项目，并借助[Spring Cloud GCP](https://spring.io/projects/spring-cloud-gcp)来使用[Cloud Firestore](https://cloud.google.com/firestore/)的功能。Spring Cloud GCP 允许基于 Spring 的应用便利地消费 Google 服务，如数据库（Cloud Firestore、Cloud Spanner 甚至 Cloud SQL）、Google Cloud Pub/Sub 以及用于日志和跟踪的 Stackdriver 等。

在此之后，我们将会对应用做一些修改，将其部署到[Google Kubernetes Engine（GKE）](https://cloud.google.com/kubernetes-engine/)。GKE 是一个托管的、生产就绪的环境，用于部署容器化的、基于 Kubernetes 的应用。

最后，我们将会使用[Skaffold](https://skaffold.dev/)和[Cloud Code](https://cloud.google.com/code/)让部署过程更简便。Skaffold 会处理构建、推送和部署应用的工作流。Cloud Code 是一个适用于 VS Code 和 IntelliJ 的插件，它可以与 Skaffold 和 IDE 协作，这样的话，你只需要点击一个按钮就可以部署至 GKE。在本文中，我们将会组合使用 IntelliJ 和 Cloud Code。

搭建工具
----

在编写代码之前，我们首先确保有一个 Google Cloud 项目并且所有的工具都已经安装就绪。

### 创建 Google Cloud 项目

搭建 GCP 实例非常容易。你可以按照[这些指南](https://console.cloud.google.com/freetrial)完成该过程。这个新的项目允许我们将应用部署到 GKE 和访问数据库（Cloud Firestore），随后当我们容器化应用的时候，它还提供了一个让我们推送镜像的地方。

### 安装 Cloud Code

接下来，我们要安装 Cloud Code。你可以按照[这些指南](https://cloud.google.com/code/docs/intellij/install)学习如何将 Cloud Code 安装到 IntelliJ 中。Cloud Code 会管理 Skaffold 的 Google SDK 安装，在本文后续的内容中，我们将会用到它们。Cloud Code 还允许我们探查 GKE deployment 和 service。最重要的是，它还有一个很聪明的 GKE 开发模式，它会持续监听代码的变化，当它探测到变化时，就会构建应用、构建镜像、推送镜像到仓库、部署应用到 GKE 集群、启动流式日志并打开 localhost 通道，这样的话就可以在本地测试应用了。这个过程就像魔法一样！

为了使用 Cloud Code 继续处理我们的应用，首先确保你已经使用 Cloud Code 插件进行了登录，这可以通过点击 IntelliJ 窗口右上角的图标来实现：

![](_assets/7bba6e442bc7884c4ec320c6364660d0.jpg)

除此之外，我们还会运行一些命令确保应用可以在你的机器上运行并且能够与 Google Cloud 上项目所需的服务进行通信。我们要确保指向了正确的项目并进行认证：

```
gcloud config set project <YOUR PROJECT ID>
gcloud auth login
```

接下来，需要确保你的机器具有应用凭证（application credential），以便于在本地运行应用：

```
gcloud auth application-default login
```

### 启用 API

现在，所有的准备工作都已经搭建完成，我们接下来需要启用应用中所使用的 API：

*   Google Container Registry API：该 API 允许我们有一个自己的 registry，可以将私有的镜像推送到这里。
    
*   Datastore 模式中的 Cloud Firestore：它会允许我们将实体存储到 NoSQL 数据库中。请务必选择 Datastore 模式，这样我们才能使用 Spring Cloud GCP 为其提供的支持功能。
    

通过访问项目的[API Dashboard](https://console.cloud.google.com/apis/library)，你可以管理项目中已经启用的 API。

创建我们的 Dog 服务
------------

要事优先！我们需要从一个可以本地运行的简单应用开始。我们将会创建一个 Dog 微服务。因为我使用的是 IntelliJ Ultimate，所以可以依次访问“File -> New -> Project…”并选择“Spring Initializr”。在这里，我会选择 Maven、Jar 和 Java 11，并且会将其命名为`dog`，如下所示：

![](_assets/3ff0eb2edf522a618d5e079fab9726c9.jpg)

点击 next 并添加：Lombok、Spring Web 和 GCP Support，如下所示：

![](_assets/25df32ed43c71f6a98348428fa0cb4b5.jpg)

如果一切正常的话，我们应该会得到一个可以运行的应用。如果你不想使用 IntelliJ 来完成该步骤的话，那么可以使用你的 IDE 的对等功能或者直接使用[Spring的Initilizr](https://start.spring.io/)。

接下来，我们要为 Dog 服务添加一个 POJO 以及几个 REST 端点，以便于测试我们的应用。我们的 Dog 对象会有 name 和 age 属性，在这里，我们会使用 Lombok 的 @**Data** 注解来标注它们，这样就不用为它们编写 setter 和 getter 方法了。另外，我们还会使用 @**AllArgsConstructor** 注解，这样 Lombok 会为我们创建一个构造器。在稍后创建 Dog 的时候，我们将会使用该构造器。

```
@Data
@AllArgsConstructor
public class Dog {
 private String name;
 private int age;
}
```

接下来，我们要为 Dog 创建控制器类和 REST 端点：

```
@RestController
@Slf4j
public class DogController {

  @GetMapping("/api/v1/dogs")
  public List<Dog> getAllDogs() {
    log.debug("->getAllDogs");
    return ImmutableList.of(new Dog("Fluffy", 5),
        new Dog("Bob", 6),
        new Dog("Cupcake", 11));
  }

  @PostMapping("/api/v1/dogs")
  public Dog saveDog(@RequestBody Dog dog) {
    log.debug("->saveDog {}", dog);
    return dog;
  }
}
```

端点会返回预先定义好的 dog 列表，**saveDog** 其实没有实现什么功能，但是作为起步来讲，这已经足够了。

使用 Cloud Firestore
------------------

现在，我们已经有了一个应用的骨架，接下来，我们会尝试使用 GCP 的一些服务。Spring Cloud GCP 为 Datastore 模式的 Google Cloud Firestore 添加了 Spring Data 的支持。我们会使用该特性来存储 Dog，而不是使用简单的列表。这样，用户也可以真正将 Dog 保存到数据库中。

首先，我们需要将 Spring Cloud GCP Data Datastore 依赖添加到 POM 文件中：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-data-datastore</artifactId>
</dependency>
```

现在，我们需要修改 Dog，使其能够进行存储。我们将会添加一个 @**Entity** 注解，并为 **Long** 值类型添加 @**Id** 注解，使其作为实体的标识符：

```
@Entity
@Data
@AllArgsConstructor
public class Dog {
  @Id private Long id;
  private String name;
  private int age;
}
```

随后，我们可以创建一个常规的 Spring Repository 类，如下所示：

```
@Repository
public interface DogRepository extends DatastoreRepository<Dog, Long> {}
```

和一般的 Spring Repository 类似，这里没有必要编写接口的实现，因为我们只会使用非常基础的方法。

现在，可以修改控制器类了，我们会将 **DogRepository** 注入到 **DogController** 中，然后修改这个类并按照如下的方式使用 repository：

```
@RestController
@Slf4j
@RequiredArgsConstructor
public class DogController {

  private final DogRepository dogRepository;

  @GetMapping("/api/v1/dogs")
  public Iterable<Dog> getAllDogs() {
    log.debug("->getAllDogs");
    return dogRepository.findAll();
  }

  @PostMapping("/api/v1/dogs")
  public Dog saveDog(@RequestBody Dog dog) {
    log.debug("->saveDog {}", dog);
    return dogRepository.save(dog);
  }
}
```

注意，我们使用了 Lombok 的 @**RequiredArgsConstructor** 注解来创建构造器，以便于注入 **DogRepository**。在运行应用的时候，端点会调用 Dog 服务，进而会尝试使用 Cloud Firestore 来检索和存储 Dog。

**小提示**：要快速对其进行测试的话，我们可以在 IntelliJ 中使用如下的内容创建 HTTP 请求：

```
POST http://localhost:8080/api/v1/dogs
Content-Type: application/json

{
  "name": "bob",
  "age": 5
}
```

只需简单的几步，我们就将应用运行了起来，并且能够消费来自 GCP 的服务。太棒了！现在，我们将其变成一个容器并部署它。

容器化 Dog 服务
----------

现在，我们可以编写 Dockerfile 文件来容器化上述的应用。但是，我们采用一个不同的方案，也就是 Jib。Jib 有一点我非常喜欢，那就是它会把应用程序分割为多个层，将依赖从类中分离出来。这样做的好处在于，我们会有更快的构建速度，这样不必等待 Docker 重新构建整个 Java 应用，而是只部署那些发生了变化的层。除此之外，Jib 还有一个[Maven插件](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin)，我们只需修改应用的 POM 文件就可以轻松将其搭建起来。

要开始使用该插件，我们需要修改 POM 文件并添加如下的内容：

```
<plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>1.8.0</version>
        <configuration>
          <from>
            <image>gcr.io/distroless/java:11</image>
          </from>
          <to>
            <image>gcr.io/<YOUR_GCP_REGISTRY>/${project.artifactId}</image>
          </to>
        </configuration>
</plugin>
```

注意，我们使用了 Google 的 Java 11 distroless 镜像。“Distroless”镜像只包含了你的应用及其运行时依赖。它们不包含包管理器、shell 或其他在标准 Linux 发行版中会存在的其他程序。

我们限制运行时容器中所包含的内容恰好是应用所需，这是 Google 和其他在生产环境使用容器多年的其他技术巨头的最佳实践。它会改善扫描器（如 CVE）的信噪比并降低确定问题出处的负担。

记住，需要将上述代码替换为你的 GCP registry 并匹配项目的名称。

完成之后，就可以尝试通过如下的命令构建并推送应用的镜像了：

```
$ ./mvnw install jib:build
```

运行该命令会构建并测试应用，创建镜像并最终将新创建的镜像推送至你的 registry。

**注意**：通常来讲，使用 distro 和一个特定的摘要，而不是使用“latest”是一个好的实践。我将决定权留给读者，让你们根据所使用的 Java 版本自行确定该使用哪个基础镜像和摘要。

部署 Dog 服务
---------

到这里，我们的应用几乎就可以部署了。为了实现这一点，我们先创建一个 GKE 集群，应用将会部署到这里面。

### 创建 GKE 集群

关于如何创建 GKE 集群，可以参考[这些指南](https://cloud.google.com/kubernetes-engine/docs/quickstart)。基本上来讲，你需要访问 GKE 页面，等待 API 启用，然后点击按钮来创建集群。你可以使用默认的配置，但是要确保点击“More options”按钮，并启用对 Cloud API 的完全访问：

![](_assets/7f32f731701768c86eda5640f2f387a2.jpg)

这将允许你的 GKE 节点具有访问所有其他 Google Cloud 服务的权限。稍等片刻之后，集群就能创建出来了：

![](_assets/cf01800ce50db9e94aa228ae76495eea.jpg)

### 在 Kubernetes 中，探测应用的存活状态

Kubernetes 会希望监控你的应用，从而确保应用处于正常运行的状态。在出现故障的时候，Kubernetes 能够知道应用停机了，因此它会启动一个新的实例。为了实现这一点，我们需要确保当 Kubernetes 问询应用程序的时候，应用能够给出响应。我们需要添加一个 actuator 和 Spring Cloud Kubernetes。

添加如下的依赖到 POM 文件中：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

如果你的应用在 **src/main/resources** 目录下存在 **application.properties** 文件的话，请删除掉它，并创建一个包含如下内容的 **application.yaml** 文件：

```
spring:
  application:
    name: dog-service

management:
  endpoint:
    health:
      enabled: true
```

这样会给我们的应用一个名称，并且会暴露前文所述的健康端点。为了校验它是否正常运行，你可以访问应用的 **localhost:8080/actuator/health** 地址。你将会看到如下的响应：

### 配置应用在 Kubernetes 中运行

为了将应用部署到新 GKE 集群中，我们编写一些额外的 YAML，以便于创建 deployment 和 service。我们可以使用如下的 deployment，但是需要记住要将 GCR 改成你自己项目的名称：

```
deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dog-service
spec:
  selector:
    matchLabels:
      app: dog-service
  replicas: 1
  template:
    metadata:
      labels:
        app: dog-service
    spec:
      containers:
        - name: dog-service
          image: gcr.io/<YOUR GCR REGISTRY NAME>/dog
          ports:
            - containerPort: 8080
          livenessProbe:
            initialDelaySeconds: 20
            httpGet:
              port: 8080
              path: /actuator/health
          readinessProbe:
            initialDelaySeconds: 30
            httpGet:
              port: 8080
              path: /actuator/health
```

添加一个具有如下内容的 **service.yaml** 文件：

```
apiVersion: v1
kind: Service
metadata:
  name: dog-service
spec:
  type: NodePort
  selector:
    app: dog-service
  ports:
    - port: 8080
      targetPort: 8080
```

deployment 包含了几个针对就绪状态和存活状态探针的修改。Kubernetes 会使用这些端点来询问应用是否处于活跃状态。service 暴露了 deployment，这样其他的服务就可以消费它了。

完成之后，我们现在就可以使用在本文开始时安装的 Cloud Code 插件进行启动了。通过 Tools 菜单，选择 Cloud Code -> Kubernetes -> Add Kubernetes Support。这将会自动向你的应用中添加 Skaffold YAML 并且会搭建一些内容，这样的话，我们只需点击一个按钮就可以将其部署到集群中了。要确认所有的这些都能正常运行，我们可以通过 IntelliJ 的 Run/Debug Configurations 探查配置。如果点击 Develop on Kubernetes 话，它会自动获取你的 GKE 和 Kubernetes 配置文件，如下所示：

![](_assets/50bf84ed7f5d79d8238ee398fded1fa7.jpg)

点击 Ok 按钮，然后点击右上角的绿色"Play"按钮：

![](_assets/add3f3bb3961ebd6eb6868fc116ccb96.jpg)

随后，Cloud Code 会自动构建应用、创建镜像、部署应用到 GKE 集群并将 Stackdriver 的日志转发到本地机器上。它还会打开一个通道，这样的话，我们就可以通过 **localhost:8080** 来消费该服务。你还可以在 Google Cloud Console 的工作负载页面查看：

![](_assets/04af2966775367d580c01a34d954685d.jpg)

结论
--

祝贺你完成到了这一步！我们在本文中构建的应用展现了大多数基于微服务的应用都会用到的一些核心技术：快速的、完全托管的、serverless 的、云原生 NoSQL 文档数据库（Cloud Firestore），以及 GKE，这是一个用于部署基于 Kubernetes 的容器化应用的托管的、生产就绪的环境，最后我们使用 Spring Boot 构建了一个云原生微服务。在这个过程中，我们还学习了如何通过常见的 Java 模式，使用 Cloud Code 等工具来简化开发工作流，并使用 Jib 来构建容器化应用。

希望这篇文章对你有用，并且能尝试一下这些技术。如果你觉得这很有意思的话，可以看一下 Google 提供的[codelabs](https://codelabs.developers.google.com/spring/)，通过它们你可以学习 Spring 和 Google Cloud 产品。

**作者简介：** 

Sergio Feilx 是 Google Cloud 的一名软件工程师，他就职于 Cloud Engineering Productivity 团队，这是 Google Cloud 中的一个组织，致力于让开发更加流畅并提升产品和工程质量。

**原文链接：** 

[Spring Boot Tutorial: Building Microservices Deployed to Google Cloud](https://www.infoq.com/articles/spring-boot-tutorial/)