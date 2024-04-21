# 通过 Flowable-UI 来体验一把 Flowable 流程引擎

在我们使用 Flowable 的过程中，最重要的流程绘制工具就是 Flowable-UI 了，虽然我们在上篇文章和大伙也介绍了不少流程绘制工具，但是个人感觉，还是 Flowable-UI 更好用一些。

今天我们就来聊聊 Flowable-UI 的一些玩法。

1\. Flowable-UI
---------------

Flowable-UI 说白了就是一堆 Web 应用，它主要提供了四方面的功能：

*   Flowable IDM: 身份管理应用。为所有 Flowable UI 应用提供单点登录认证功能，并且为拥有 IDM 管理员权限的用户提供了管理用户、组与权限的功能。
*   Flowable Modeler: 让具有建模权限的用户可以创建流程模型、表单、选择表与应用定义。
*   Flowable Task: 运行时任务应用，这个提供了启动流程实例、编辑任务表单、完成任务，以及查询流程实例与任务的功能。
*   Flowable Admin: 管理应用。让具有管理员权限的用户可以查询 BPMN、DMN、Form 及 Content 引擎，并提供了许多选项用于修改流程实例、任务、作业等。管理应用通过 REST API 连接至引擎，并与 Flowable Task 应用及 Flowable REST 应用一同部署。

简单来说：

*   创建用户、分配角色用 Flowable IDM。
*   画流程图用户 Flowable Modeler。
*   测试、体验流程用 Flowable Task。
*   后台管理相关的用 Flowable Admin。

2\. 安装方式
--------

在之前的版本中，前面说的这几个应用都是不同的 war 包，部署和访问都非常麻烦，不过现在，官方将之前的 5 个 war 整合成一个了，所以现在访问就非常容易了。

### 2.1 运行 war 包

由于这些应用是基于 Spring Boot2.0 开发的，因此也可以直接作为独立应用来直接运行，通过执行 `java -jar xxx.war` 的方式来启动这些应用。

这个需要大家首先去 GitHub 上下载最新的 Flowable：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dba8eee210f14bf6824a3a3a7277b937~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

下载成功之后解压，里边有一个 wars 文件夹，该文件夹中就有我们需要的 war 包，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/515f83cb967642688797cffc17fe4bfa~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

然后进入到该目录中，执行如下命令启动该 war 文件：

```shell
java -jar flowable-ui.war

```

从启动到日志中我们就可以看出来这是一个 Spring Boot：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dca2ebac78094349b774cb8742fd6698~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

既然是一个 Spring Boot，那么如果又一些参数想改，就可以直接在启动命令中修改，例如默认的端口号是 8080，现在想改为 8088，那么就在启动命令中添加参数 `--server.port=8088` 即可。

所以直接启动这些应用并不是麻烦事，反而是比较简单的。

> 需要说明的是，有的小伙伴在网上看到的教程里边可能提到了多个 war 包，这是因为 Flowable 的历史问题，早期的 Flowable 上面我们说的四个功能都对应了不同的 war 包，要单独部署，但是在新版中合成一个了，新版用起来相对更方便一些。

### 2.2 docker 安装

我看了下他这个还支持 Docker 安装，所以我还是用 Docker 吧，更省事，将来不想要了删除也方便（对 Docker 不熟悉的小伙伴可以在微信公众号后台回复 docker，有松哥写的入门教程）。

docker 安装的话，直接如下命令即可：

```shell
docker run -d --name flowableui -p 8086:8080 flowable/flowable-ui

```

没什么特别需要配置的地方，指定一下容器名字和端口映射即可。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97ea9865c6f741dfa70cb3583cde024f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 2.3 访问

无论是直接运行还是 Docker 运行，运行成功之后，浏览器输入 `http://localhost:端口/flowable-ui` 进行访问，此时会弹出来如下页面：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb9ba9de42d74e24b410e7fd0edd0ad6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

默认情况下，登录的用户名是 admin，密码是 test，注意别把密码写错了。

登录成功之后，如果看到如下页面，就表示安装成功了（一般来说应该不会有安装问题）：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67a28563409f4c55bead2deddfef0ba8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

装好之后，接下来我们就来逐步体验这里的功能，因为之前和小伙伴们讲过这里的大部分功能了，所以今天我主要和大家讲讲这里的身份管理和流程体验功能（因为流程体验中要用到身份管理）。

3\. 身份管理（IDM）
-------------

身份管理就是用户、用户组的管理，我们点进到身份管理页面之后，可以看到如下内容：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a365afcd31224abd992b5add285b697c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

可以看到，默认只有一个 admin 用户，也就是我们刚刚登录时候的用户。

### 3.1 用户管理

接下来点击左边的创建用户按钮，我们可以创建新的用户出来：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a98c97629c6d4113b1baa3541f97b13e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

填入用户的基本信息和密码即可。

我一共创建了四个用户，最终结果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79c259ed3d444ebe816ab97c3baf21a8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 3.2 组管理

接下来点击上面的**组**，我们可以创建用户组，这个用户组相当于我们在 vhr 中所说的角色，给用户分组，相当于给用户分配一个角色。

默认情况下，没有任何组，组是空的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03ae322ee4f3491b9d50c61d16f41744~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们点击创建组按钮，先来创建一个经理组：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46896fff4abb482bb788c77496f24e3a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

组添加成功之后，点击添加用户按钮，为用户组中添加用户：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90d496cbe0814529bca0618fc35cefa1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

假设 zhangsan 是经理，最终添加结果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ccc86381a0841cea589eb7b87e4d500~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

利用相同的方式，我再创建一个主管的组，并为之添加两个用户 lisi 和 wangwu。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00de6b1ce01c47e885e63ddf8924fd80~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 3.3 权限控制

我们前面创建的用户现在是没有任何权限的，例如现在如果使用 zhangsan/123 进行登录，登录成功后页面是空的，没有任何东西：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/862573fb8dca471f9b8b49a8f68df25b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

所以我们要为用户添加相应的权限。点击顶部的权限控制一栏，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e094c5f459546809bc471929ba59c3d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们可以为这五种访问分别设置对应的用户/用户组：

*   访问 idm 应用：这个就是访问身份管理应用，如果用户没有访问这个的权限，那么用户在登录成功的后的首页上就看不到身份管理应用程序这个菜单项。
*   访问 admin 应用：这个是访问管理员应用程式，如果没有没有这个的访问权限，那么用户在登录成功之后的首页上就看不到管理员应用程式这个菜单项。
*   访问 modeler 应用：这个是访问建模器应用程序，如果没有没有这个的访问权限，那么用户在登录成功之后的首页上就看不到建模器应用程序这个菜单项。
*   访问 workflow 应用：这个是访问任务应用程序，如果没有没有这个的访问权限，那么用户在登录成功之后的首页上就看不到任务应用程序这个菜单项。
*   访问 REST API：这个是指用户通过 REST API 访问工作流的权限。

以访问 idm 应用为例，在设置的时候，我们可以直接设置用户，也可以设置用户组，设置用户组的话，则这个组中的所有用户都能访问这个菜单项。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d9991931e314897971649ef84e40164~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我这里设置的是经理和 javaboy 可以访问所有应用，而主管只可以访问 workflow 应用。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b4e7d9df9c04473b7328c702f352264~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

好啦，准备工作完成后，接下来我们就来绘制一个报销的流程图，这个流程图稍微复杂一些，并且带有表单，这是松哥之前从未写过的内容。

4\. 流程图绘制
---------

我先大致上用文字描述下我们的报销流程：

1.  启动一个流程。
2.  员工填写报销单，然后进行提交。
3.  如果报销金额小于等于 1000 元，则主管审批，主管审批通过后，流程结束；主管审批拒绝，则回到步骤 2 中员工重新填写报销单。
4.  如果报销金额大于 1000 元，则先由经理进行审批，经理审批通过后，再由 CEO 审批，两者都审批通过，则流程结束；经理和 CEO 任意一个审批不通过，则流程重新回到步骤 2 中。

好啦，这就是我们一个大致的流程描述，接下来我们就在 Flowable-UI 中来绘制这样一个流程图我们来看下。

首先点击建模器应用程序，我们开始绘制：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c7c271614664ee9b731e03b26654fdd~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

点击右上角创建流程，创建一个流程：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e18af0e1141433da562a5f2040e38c2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

然后就开始绘制。

首先第一步是用户提交报销材料，报销材料需要填写一个表单，所以我们在下面的属性中，找到表单引用，为这个用户任务设置一个外部表单：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af45e67cf5cd4643b9d0f63cfba7d0f7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

如果有提前绘制好的表单，这里就会显示出来，那么直接引用即可，如果没有提前绘制好的表单，那么大家看到的就如同上图那样。

现在我们点击新表单，创建一个新表单：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f28fa53b35934effa577cfb9e9b6e306~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

为新表单设置名称、key 等内容：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2004603c6e34ff788299cda0a9e3855~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

创建成功之后，我们就可以看到表单设计页面了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf5390f5e20e444f8eec03fcfb92909a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

左边是表单组件区域，右边是表单绘制区域。思考用户需要提交哪些信息来报销，直接将相应的表单拖过来即可。

> 其实大家看最上面一栏的顶部菜单，也自动切换到表单菜单了，这也就意味着，当我们想要创建一个表单的时候，也可以不用从流程绘制那个入口进来，可以直接提前绘制好表单，然后在画流程的时候直接引用即可。

例如我首先拖一个文本框过来，作为用户名，然后点击右边的编辑按钮进行编辑，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/882ca59ea5194d319a901877cb87b714~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

有如下属性：

*   标签：这个文本框将来展示的信息。
*   覆盖 id：勾上这个，就可以自定义 id 了，否则 id 和标签是一样的。
*   id：这个是这个组件的唯一名称，将来在代码中，如果我们想要获取这个表单的值，就需要通过这个 id 去访问。
*   设置表单是否只读或者必填。
*   默认值：这个相当于是这个表单的 placeholder。

好了，理解了这个，我们再来随便加两个组件，按照相同的思路进行配置：

报销金额，这是一个小数组件：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a01dc6c9f33249e68f05012b59c9b704~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

报销用途是一个多行文本组件：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/415dce65424643939be06e982595e7c3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

最终设计结果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d72c724977b44a695060c257623d2cc~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

> 标签后面有一个 `*` 表示这是一个必填项。

绘制完成后，点击左上角的保存按钮，保存成功后，会自动回到流程绘制页面。

接下来，我们还需要设置这个用户任务由谁来处理，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dc16f78ed3545ba8a5ba47dfe66a125~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

很明显，这个流程是谁发起的，谁就来填写这个表单，所以，配置如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50f2e9a25b2047bfa538fb4c981d1b15~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

好了，这个用户任务就配置完成了，接下来要根据报销金额进行划分了，我一口气画完吧，再来和大家逐步分析：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/339378bf1ef543ea809c2e6ce4e7ff75~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

先来看报销金额小于等于 1000 的那条线。

首先，我们要为这条线设置条件，也就是流程从互斥网关中出来之后，什么情况下会进入到主管审批这个节点中，选中这条出线，配置流条件，配置如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b49a2438f06c478aa5e6f5bd6b68449b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13d21634a3e04ae082cdb58ec820b78f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

这里的 money 就是我们刚刚在表单中填写的 money，表单中各个字段的值，都会被映射成为一个流程变量，我们可以直接访问。

接下来配置主管审批，首先我们设置分配用户，即由谁来执行这个用户任务：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a561b8645ce4527b315116cfb19860f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们设置候选组为主管，也就是所有的主管都可以审批这个节点：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfd72693ab11403b98a9c55f2adea2bf~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

主管审批的时候，无非就是同意或者拒绝，通过表单我们可以定义出同意或者拒绝这两个按钮。配置方式如下，首先为主管审批设置表单引用：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb8c439d2ad14741a79b4a94294c2b30~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

给这个新建的表单取一个名字和 id，这个 id 大家要记牢了，将来我们会用到：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b31466972d654c47b410edca098c0019~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

在表单设计的页面，有一个结果选项卡，这个表示表单的输出内容，这个结果选项卡决定了这个表单上的最终按钮，默认情况下，只有一个完成按钮，我们可以自定义配置：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569b615e68924101afbc304340f80fb2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们为这个表单设置同意和拒绝两个按钮，方式如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efab50fe9e7349e188b568879189d97b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

> 这块也有其他设置方式，我就先以这种方式来和大家演示，将来在视频中再来和大家聊一聊其他方式。

表单配置完成后，保存即可，保存之后，就会回到流程绘制页面。

接下来为同意这条出线设置条件：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ea44ea8b83646e5b21fba93fd825d05~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

大家注意这个表单的命名规则，是 `form_表单名称_outcome` 这个就表示表单的输出结果，也就是我们刚刚在表单中配置的结果选项卡中的内容：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d3e9a1697af42df9095ed20d7b4c7ca~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

配置完成后，相同的方式，将同意改为拒绝，再来配置一下拒绝那条线。

好啦，上面这条线配置好之后，接下来相同的方式配置下面大于 1000 的情况，其中经理这个用户任务就由经理这个组来处理，CEO 审批这个用户任务就由指定用户 javaboy 来审批即可，具体细节我就不多说了，都跟上面一样。

绘制完成后，记得点一下左上角的勾，看下流程有没有漏洞，如下图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcd79216f2094f2094d9eda8ec52bb81~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

至此，我们的流程图就画好了。

> 一个流程图只能有一个开始，但是可以有多个结束。

5\. 创建应用
--------

流程图画好之后，接下来我们可以下载这个流程图对应的 XML 文件，然后去开发自己的 Java 代码。但是，这不是我们本文的工作，本文的工作是直接在 Flowable-UI 这个工具中，创建一个应用，然后发布这个流程。

点击上面的应用程序菜单，然后点击右上角的创建应用程序按钮，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b60588f87aae433a8cb5668e0329ee34~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbff9f7cac074d6b997766dbab44ad04~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

接下来可以为你的应用设置图标、主题啥的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a3d4a0e6c444a579113aeb122bd2430~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

然后点击**编辑包含的模型**按钮，为这个应用选择一个流程：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f431900d98f04a5ba7ed056b3493ff75~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

然后点击左上角的保存按钮，保存并发布这个应用，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5311dbf5cb964e758105ab89732b85fb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

发布成功之后，回到首页，就可以看到这个应用了，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39a49f9de5f548959480c2cfc31cb156~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

6\. 体验报账
--------

接下来我们就来体验一把这个报账流程，我们目前的身份是 admin，也就是说 admin 这个用户现在要报账了。

首先点击应用图标，进入到应用中，任务是空的，也就是目前没有 admin 需要审批的任务：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff28d0b81a7542b0b6f90246838d6759~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

然后我们点击上方的流程菜单，如下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70a03575966840b58b640217b3efcccc~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

首先点击左边的启动流程，然后点击右边的启动流程，流程启动之后，我们可以点击右上角的显示图按钮，查看流程目前走到哪一步了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/824e87a4eed54a8497e5fd527895b2fc~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

可以看到，流程目前走到用户提交报销材料这一步了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b355aa131d2a4696b968f072e2a48207~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

用户提交报销材料这一步是由流程的发起人完成的，也就是 admin 自己完成，此时我们回到任务菜单，就可以看到 admin 有需要完成的任务了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2924df9f45c34a11b4b9f7f4677f3d0f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

填入报销资料，然后点击完成按钮。

接下来再点击流程菜单，查看流程图，可以看到，此时进入到主管审批这一步了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08883dd484db414a9dfd8a2033a62de1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

按照我们第 3 小节的配置，lisi 和 wangwu 是主管，所以，我们先注销登录，然后重新以 lisi 或者 wangwu 的身份登录，假设我这里以 lisi 的身份登录，登录成功之后，进入到这个应用中，进来之后，首先将筛选规则改为我是其中一个候选人的任务：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c8a367478064d58b7616d2c8ce5397b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

然后在任务中就可以看到自己需要处理的任务了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/778a1e3f55084bca98ea012fb07905b2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

对于这种候选人或者候选组的任务，需要先点击右上角的认领，然后再处理（如果是直接分配给某一个用户的，就不需要认领了，可以直接处理了），认领之后，就可以选择同意或者拒绝了，如下图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0defa1e31a6c411fbdd0641c96a6fb09~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

假设我们点击拒绝按钮，拒绝之后，我们点击流程菜单，查看流程图，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ebb47e8c4684900b3ed56c1a5c4f1ca~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

可以看到，流程在进入到主管审批这个节点之后，被拒绝了，然后回到了用户提交报销材料这个节点上，现在 admin 要重新登录，登录之后，在自己的任务中又可以看到提交报销材料了，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb247f72c1d64ab09679ecbaf59e154a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

随便改一下，然后继续提交。

切换到 wangwu 登录，同意报销，流程结束。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/772483beb71c42ffa30e1771d907ca39~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

好啦，下面那条超过 1000 块钱的线，小伙伴们可以自行测试，我就不演示啦～

7\. 数据表分析
---------

对于前面的案例，数据都存入到哪里去了呢？我们也来分析下。

如果是 war 包直接启动的话，它默认使用的是 H2 数据库，数据存在了系统当前用户目录下的 h2 文件夹里，我们通过 h2 数据库的连接工具即可操作这个数据库。IDEA 上自带的数据库连接工具就可以连接 H2 数据库，这里我就不演示具体连接方式了，我们来看下表的查询：

```sql
SELECT COUNT(*) TABLES, table_schema FROM information_schema.TABLES  WHERE table_schema = 'flowable02' GROUP BY table_schema;

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ab31d006c8e4c34aad7419abcd8ab0c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

> 这里显示了 82 张表，是因为我自己手动创建了三个跟用户相关的表，其他 79 张表都是 Flowable 自动创建的。

好了，接下来我们就对这 79 张表进行一个简单的分类整理。

1.  `ACT_APP_*`（5）
2.  `ACT_CMMN_*`（12）
3.  `ACT_CO_*`（3）
4.  `ACT_DMN_*`（6）
5.  `ACT_EVT_*`（1）
6.  `ACT_FO_*`（6）
7.  `ACT_GE_*`（2）
8.  `ACT_HI_*`（10）
9.  `ACT_ID_*`（9）
10.  `ACT_PROCDEF_*`（1）
11.  `ACT_RE_*`（3）
12.  `ACT_RU_*`（13）
13.  `FLW_CHANNEL_*`（1）
14.  `FLW_EV_*`（2）
15.  `FLW_EVENT_*`（3）
16.  `FLW_RU_*`（2）

`*` 是通配符，后面的数字表示这种类型的表有多少个。

大致看一下，其实还是有很多规律的。

最明显的规律就表名通过两个 `_` 隔开成了三部分，接下来我们就以此为线索，一部分一部分来讲。

### 7.1 表名前缀

首先搭建看这个表的前缀，分了两种：

*   ACT_
*   FLW_

松哥在之前的文章中已经和大家介绍过了，Flowable 是基于 Activiti 开发出来的流程引擎，所以我们在很多地方都还能看到 Activiti 的影子，这个表名中 `ACT_` 就是 Activiti。

具体来说，与 Flowable 开源代码库相关的数据库表名以 `ACT_` 开头。特定于 Flowable Work 或 Engage 的数据库表以 `FLW_` 前缀开头。

### 7.2 表名中间部分

紧跟着 ACT_ 或者 FLW_ 后面的内容，基本上也都是简称，例如：

*   APP 表示这都是跟应用程序相关的表。
*   CMMN 表示这都是跟 CMMN 协议相关的表。
*   CO（CONTENT）表示这都是跟内容引擎相关的表。
*   DMN 表示这都是跟 DMN 协议相关的表。
*   EVT（EVENT）表示这都是跟事件相关的表。
*   FO（FORM）表示这都是跟表单相关的表。
*   GE（GENERAL）表示这都是通用表，适用于各种用例的。
*   HI（HISTORY）这些是包含历史数据的表。当从运行时表中删除数据时，历史表仍然包含这些已完成实例的所有信息。
*   ID（IDENTITY）表示这都是跟用户身份认证相关的表。
*   PROCDEF（PROCESSDEFINE） 表示这都是跟记录流程定义相关的表。
*   RE（REPOSITORY）表示这都是跟流程的定义、流程的资源等等包含了静态信息相关的表。
*   RU（RUNTIME）代表运行时，这些是包含尚未完成的流程、案例等的运行时数据的运行时表。Flowable 仅在执行期间存储运行时数据，并在实例结束后删除记录，这使运行时表保持小而快。
*   CHANNEL 表示这都是跟泳道相关的表。
*   EV 表示这个是跟 FLW_ 搭配的，在这里似乎并没有一个明确的含义，相关的表也都是跟 Liquibase 相关的表。
*   EVENT 表示这都是跟事件相关的表。

把这些简称的单词搞明白了，接下来的就容易看懂每一个表的含义了。

表名是由三部分构成的，我们现在已经分析了前两部分了，接下来我们来看第三部分。

### 7.3 表名后缀

表名的后缀，有一些是通用的后缀名词，我们先来看下。

*   DATABASECHANGELOG：表名中包含这个单词的，表示这个表是 Liquibase 执行的记录，Liquibase 是一个数据库脚本管理的工具，有点像 flyway，松哥之前写过 flyway 的文章，我这里就不啰嗦了。包含 DATABASECHANGELOG 后缀的表一共是 6 张。
*   DATABASECHANGELOGLOCK：表名中包含这个单词的，表示这个表记录 Liquibase 执行锁的，用以确保一次只运行一个 Liquibase 实例，包含 DATABASECHANGELOGLOCK 后缀的表也是 6 张。

这两个加起来就 12 张表了，一共 79 张，现在还剩 67 张，我们来详细看下。

> 在后面的介绍中，凡是涉及到 DATABASECHANGELOG 和 DATABASECHANGELOGLOCK 的表，我就直接省略了。

#### 7.3.1 ACT\_APP\_*

以 `ACT_APP_*` 开头的表负责应用引擎存储和应用部署定义。相关的表一共是五张，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f57d963df90458c93228d7f428e404e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**ACT\_APP\_APPDEF**

应用程序模型产生应用程序定义。此定义，如流程/案例/等。是成功部署到应用引擎的应用模型的表示。

**ACT\_APP\_DEPLOYMENT**

当通过应用引擎部署应用模型时，会存储一条记录以保存此部署。部署的实际内容被存储在 ACT\_APP\_DEPLOYMENT_RESOURCE 表中，并从该表中引用。

**ACT\_APP\_DEPLOYMENT_RESOURCE**

此表包含构成应用程序部署的实际资源（存储为字节）。当引擎需要实际模型时，将从该表中获取资源。

#### 7.3.2 ACT\_CMMN\_*

Flowable CMMN Engine 的数据库名称都以 `ACT_CMMN_` 开头。这里涉及到一个东西就是 CMMN，CMMN 与 BPMN 协议一致，也是一种流程内容的规范，CMMN 这类表一般用于存储处理 BPMN 所不能适用的业务场景数据，CMMN 通常与 BPMN 搭配使用，不过只有符合 CMMN 规范的模型数据才会使用这类表。

这里涉及到的相关表一共是 12 张，如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e92998321794270a72f763a17fbc6cb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

*   **ACT\_CMMN\_CASEDEF**
*   **ACT\_CMMN\_DEPLOYMENT**
*   **ACT\_CMMN\_DEPLOYMENT_RESOURCE**

这三个都是没有附加前缀的表，主要定义了静态信息，例如 case 的定义和部署和以及相关的资源等。

接下来这些以 `ACT_CMMN_HI_` 开头的表代表历史数据，例如过去的案例实例、计划项目等。

**ACT\_CMMN\_HI\_CASE\_INST**

此表记录由 CMMN 引擎启动的每个案例实例的数据。

**ACT\_CMMN\_HI\_MIL\_INST**

此表记录了在案例实例中达到的每个里程碑的数据。

**ACT\_CMMN\_HI\_PLAN\_ITEM_INST**

此表记录了作为案例实例执行的一部分创建的每个计划项实例的数据。

接下来以 `ACT_CMMN_RU_` 开始的表代表运行时的数据，这些数据包含案例实例、计划项等的运行时数据。Flowable 仅在案例实例执行期间存储运行时数据，并在案例实例结束时删除记录，这使运行时表保持小且查询速度快。

**ACT\_CMMN\_RU\_CASE\_INST**

此表包含每个已启动但尚未完成的案例实例的条目。

**ACT\_CMMN\_RU\_MIL\_INST**

此表包含作为运行案例实例的一部分达到的每个里程碑的条目。

**ACT\_CMMN\_RU\_PLAN\_ITEM_INST**

案例实例执行由案例定义中定义的计划项的多个实例组成，此表包含在案例实例执行期间创建的每个实例的条目。

**ACT\_CMMN\_RU\_SENTRY\_PART_INST**

计划项目实例可以有守卫状态转换的哨兵，这样的哨兵在状态改变之前可以包含多个部分，这个表就是专门用来存储这种哨兵。

#### 7.3.3 ACT\_DMN\_*

Flowable DMN 的数据库名称都以 `ACT_DMN_` 开头，这里涉及到的表一共是 6 张：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f369336a99e47989b0bdf8d0cfc9f7e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

*   **ACT\_DMN\_DEPLOYMENT**
*   **ACT\_DMN\_DEPLOYMENT_RESOURCE**

这两个是就不需要我多说了吧，跟前面的都一样，只不过这里部署的是 DMN。

**ACT\_DMN\_DECISION**

此表包含已部署决策表的元数据，并与来自其他引擎的定义相对应。

**ACT\_DMN\_HI\_DECISION\_EXECUTION**

此表包含有关 DMN 决策表执行的审计信息。

#### 7.3.4 ACT\_RU\_*

以 `ACT_RU_` 开头的表都是和流程引擎运行时信息相关的一些表。涉及到的表一共有 13 张：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e354204c1dc4e1993e52bec731194f9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b453fff45f144c19b459a7396ed59714~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**ACT\_RU\_ACTINST**

流程实例中的每个活动在此表中都有一行来指示活动的当前状态。

*   **ACT\_RU\_JOB**
*   **ACT\_RU\_TIMER_JOB**
*   **ACT\_RU\_SUSPENDED_JOB**
*   **ACT\_RU\_EXTERNAL_JOB**
*   **ACT\_RU\_HISTORY_JOB**
*   **ACT\_RU\_DEADLETTER_JOB**

Flowable 引擎使用作业表来实现异步逻辑、计时器或历史处理。这些表存储每个作业所需的数据。

**ACT\_RU\_ENTITYLINK**

此表存储有关实例的父子关系的信息。例如，如果流程实例启动子案例实例，则此关系存储在此表中。这样可以轻松查询关系。

**ACT\_RU\_EVENT_SUBSCR**

当流程定义使用事件（信号/消息/等或启动/中间/边界）时，引擎将对该表的引用存储在该表中。这简化了查询哪些实例正在等待某种类型的事件。

**ACT\_RU\_EXECUTION**

存储流程实例和指向流程实例当前状态的指针（称为执行）。

**ACT\_RU\_IDENTITYLINK**

此表存储有关用户或组的数据及其与（流程/案例/等）实例相关的角色。该表也被其他需要身份链接的引擎使用。

**ACT\_RU\_TASK**

此表包含正在运行的实例的每个未完成用户任务的条目。然后在查询用户的任务列表时使用此表。CMMN 引擎也使用此表。

**ACT\_RU\_VARIABLE**

此表存储与实例相关的变量。 CMMN 引擎也使用此表。

#### 7.3.5 ACT\_HI\_*

以 `ACT_HI_*` 开头的表包含正在运行和已完成的实例的历史数据，这些表的名称遵循其运行时对应的名称，这里一共涉及到 10 张表：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70989105c6704022844f364bc574d9a1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**ACT\_HI\_ACTINST**

历史活动信息。这里记录流程流转过的所有节点，与 ACT\_HI\_TASKINST 不同的是， ACT\_HI\_TASKINST 只记录 Task 内容。

**ACT\_HI\_ATTACHMENT**

历史附件表。

**ACT\_HI\_COMMENT**

流程的历史评论表。

**ACT\_HI\_DETAIL**

历史详情表：流程中产生的变量详细，包括控制流程流转的变量，业务表单中填写的流程需要用到的变量等。

**ACT\_HI\_ENTITYLINK**

历史参与的人员表。

**ACT\_HI\_IDENTITYLINK**

任务参与者数据表，主要存储历史节点参与者的信息，可能是 Group 也可能是 User。

**ACT\_HI\_PROCINST**

保存每一个历史流程，创建时就生成，一条流程实例对应一个记录。

**ACT\_HI\_TASKINST**

记录每一个历史节点，一个 Task 对应一个记录。

**ACT\_HI\_TSK_LOG**

每一次执行可能会带上数据，存在这里。

**ACT\_HI\_VARINST**

流程历史变量表。

#### 7.3.6 ACT\_ID\_*

以 `ACT_ID_*` 开头的都是和用户身份相关的表，一共是有 9 张表：

**ACT\_ID\_BYTEARRAY**

这是用户组的部署内容。

**ACT\_ID\_GROUP**

这是用户组的表。

**ACT\_ID\_INFO**

这是所有的用户的信息，账号密码。

**ACT\_ID\_MEMBERSHIP**

用户和用户组的关联表。

**ACT\_ID\_PRIV**

权限表。

**ACT\_ID\_PRIV_MAPPING**

用户、用户组以及权限之间的关联表。

**ACT\_ID\_PROPERTY**

用户的变量表。

**ACT\_ID\_TOKEN**

用户访问记录表。

**ACT\_ID\_USER**

用户表。

#### 7.3.7 ACT\_FO\_FORM_*

以 `ACT_FO_FORM_` 开头的表存储表单引擎和围绕表单模型和这些表单的实例数据。

**ACT\_FO\_FORM_DEFINITION**

表单定义表。

**ACT\_FO\_FORM_DEPLOYMENT**

表单部署表。

**ACT\_FO\_FORM_INSTANCE**

表单实例表。

**ACT\_FO\_FORM_RESOURCE**

表单源数据表。

#### 7.3.8 ACT\_GE\_*

以 `ACT_GE_` 开头的表表示一些通用信息表，涉及到的表一共是两个。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59863f1ea9e64150b36ed1bb472b2301~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**ACT\_GE\_BYTEARRAY**

存储每个流程的部署记录，bytes_ 字段中保存流程的具体内容。

**ACT\_GE\_PROPERTY**

存储 Flowable 自身的一些变量，主要是版本号。

#### 7.3.9 ACT\_RE\_*

以 `ACT_RE_` 开头的表表示这些表都是跟流程的定义、流程的资源等等包含了静态信息相关的表。

**ACT\_RE\_DEPLOYMENT**

流程部署记录，每次服务重启会部署一次，这里会新增一条记录。

**ACT\_RE\_MODEL**

创建模型时，额外定义的一些模型相关信息，存在这张表，默认不保存。

**ACT\_RE\_PROCDEF**

记录流程的变更，流程每变更一次存一条记录，version_ 字段加 1。

#### 7.3.10 FLW\_EVENT\_*

**FLW\_EVENT\_DEFINITION**

已部署事件定义的元数据。

**FLW\_EVENT\_DEPLOYMENT**

已部署事件部署元数据。

**FLW\_EVENT\_RESOURCE**

事件所需资源。

#### 7.3.11 其他表

还剩一些规则不太明显的表，如下：

**FLW\_RU\_BATCH** **FLW\_RU\_BATCH_PART**

这两个是批量迁移流程时使用。

**ACT\_PROCDEF\_INFO**

流程定义信息，对流程的说明。

**ACT\_CO\_CONTENT_ITEM**

每项内容在此表中都有个条目。

**ACT\_EVT\_LOG**

Flowable 引入了事件日志机制，默认会在数据库中创建 `ACT_EVT_LOG` 表保存事件日志，如果不使用事件日志，则可以删除这个表。

**FLW\_CHANNEL\_DEFINITION**

泳池管道定义表。
