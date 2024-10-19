# maven插件，SpringBoot无缝集成git-commit-id-maven-plugin
`git-commit-id-maven-plugin`是什么呢？

它是一个可以帮助我们在构建过程中自动生成Git版本信息的插件。

通过它，我们可以很方便地将当前的Git提交信息`嵌入`到构建的应用中。

### 三、快速上手

#### 环境要求

在开始之前，我们需要确保以下环境：

*   SpringBoot版本：2.4.0及以上
*   Java版本：1.8及以上

#### 添加步骤

我们可以通过在`pom.xml`中添加以下依赖来集成`git-commit-id-maven-plugin`插件：

```xml
xml

<build>
    <plugins>
        <plugin>
            <groupId>pl.project13.maven</groupId>
            <artifactId>git-commit-id-plugin</artifactId>
            <version>4.0.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>revision</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>

```

### 四、插件配置详解

#### 核心配置

1.  `generateGitPropertiesFile`: 设置为`true`可以生成`git.properties`文件；
2.  `format`: 设置文件格式，可以是`properties`或`json`；
3.  `commitIdGenerationMode`: 设置提交ID的生成模式。

#### 高级配置

这里我们展示几个常见的高级配置项：

```xml
xml

<configuration>
    
    <generateGitPropertiesFile>true</generateGitPropertiesFile>
    
    <format>properties</format>
    
    <commitIdGenerationMode>full</commitIdGenerationMode>
    
    <customProperty>
        <name>git.branch</name>
        <value>${git.branch}</value>
    </customProperty>
</configuration>

```

> *   `commitIdGenerationMode`：可以设置为`full`或者`abbrev`，前者会生成完整的提交ID，后者生成简短的提交ID；
> *   `customProperty`：可以自定义属性，这里我们添加了`git.branch`属性来记录当前的分支名称。

### 五、生成git.properties

让我们来看一个实际的演练，演示如何生成`git.properties`文件。

首先，我们在`pom.xml`中添加以下配置：

```xml
xml

<build>
    <plugins>
        <plugin>
            <groupId>pl.project13.maven</groupId>
            <artifactId>git-commit-id-plugin</artifactId>
            <version>4.0.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>revision</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <generateGitPropertiesFile>true</generateGitPropertiesFile>
                <format>properties</format>
                <commitIdGenerationMode>full</commitIdGenerationMode>
            </configuration>
        </plugin>
    </plugins>
</build>

```

执行以下Maven命令：

```null
bash

mvn clean install

```

在`target/classes`目录下，你会发现一个`git.properties`文件，里面包含了当前Git提交的信息。

以下是一个示例文件的内容：

```ini
txt

git.commit.id=1a2b3c4d5e6f7g8h9i0j
git.branch=main
git.commit.time=2024-07-22T10:00:00Z

```

这些信息可以帮助我们在任何时候`追踪`当前运行的代码版本。

### 六、应用实践

我们可以在SpringBoot项目中编写一个接口，来查询这些Git提交信息。

以下是一个详细的示例代码：

```typescript
java

@RestController
public class GitInfoController {

    @Value("${git.commit.id}")
    private String commitId;

    @Value("${git.branch}")
    private String branch;

    @Value("${git.commit.time}")
    private String commitTime;

    @GetMapping("/git-info")
    public ResponseEntity<Map<String, String>> getGitInfo() {
        Map<String, String> gitInfo = new HashMap<>();
        gitInfo.put("commitId", commitId);
        gitInfo.put("branch", branch);
        gitInfo.put("commitTime", commitTime);
        return ResponseEntity.ok(gitInfo);
    }
}

```

启动SpringBoot应用，并访问`/git-info`接口，你会看到类似这样的输出：

```json
json

{
    "commitId": "1a2b3c4d5e6f7g8h9i0j",
    "branch": "main",
    "commitTime": "2024-07-22T10:00:00Z"
}

```

这里的代码包括了详细的注释，便于读者理解每一步的操作。

和SpringBoot结合在一起，这是我最喜欢的一点，程序能控制了，就相当于你开车能熟练控制方向盘一样顺手。

### 七、兼容性

这个插件对Java和Maven的版本有一定要求：

*   Java：1.8及以上
*   Maven：3.2及以上

另外，它与SpringBoot的多个版本都有良好的兼容性，确保在不同版本的SpringBoot项目中都能顺利运行。

简单总结，这就是一个 maven 插件，用来在打包的时候将 `git-commit` 信息打进jar中。

这样做的好处是可以将发布的某版本和对应的代码`关联`起来，方便查阅和线上项目的维护。

它的作用，用官方说法，这个功能对于`大型分布式项目`来说是`无价`的。

最后说一句(求关注!别白嫖！)

如果这篇文章对您有所帮助，或者有所启发的话，求一键三连：点赞、转发、在看。

关注公众号：**woniuxgg**，在公众号中回复：笔记  就可以获得蜗牛为你精心准备的java实战语雀笔记，回复面试、开发手册、有超赞的粉丝福利！