# Flowable 开篇，流程引擎


这个专栏想和大家认真聊聊流程引擎 Flowable，今天先来和大家聊一聊我们为什么需要流程引擎，以及开发工作流时用到的一些工具。

1\. 为什么需要工作流
------------

假设我有一个请假需求，流程如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ddd37d5bdcb41edb55b11249b0de72f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

请假可以提交给我的上司，上司可以选择批准或者拒绝，无论批准还是拒绝，都会给我一个通知。

这个流程比较简单，我们很容易想到解决方案，不用工作流也能解决，有一个专门的请假表，当 A 要请假的时候，就往请假表中添加一条记录，这条记录的内容包含了请假的天数、原因、请假的审批人 B 以及一个名为 status 的字段，这个 status 字段表示这个请假申请目前的状态（待审批、已批准还是已拒绝），然后 B 登录系统之后，在请假表中查询到了 A 的请假信息，然后选择批准，此时将 status 字段的值改一下就行了。

这个流程很简单，相信小伙伴们都能想到。

然而，这是一个非常简单的流程，对于这样的流程，一般来说也确实没有必要使用工作流，但是现实中，我们涉及到的工作流往往都是非常复杂的，我举个例子，就说报销审批吧，这个可能很多小伙伴都经历过。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c769d9637614b8199442abbeac80262~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

小伙伴们看到，这个流程相对来说还是比较复杂的，此时你再用一个 status 字段去描述，就很难说的请到底是怎么回事了。每一步审批，都有可能批准也有可能拒绝，拒绝并不意味着流程结束，员工修改报销资料之后，还可以继续提交。此时如果还用 status 去描述，那么 status 将有 N 多个值去表示不同的情况，这个维护起来非常不便。

这就复杂了吗？非也非也，我们再来看一个生产笔记本电脑的例子，假设公司研发了一款新型笔记本电脑，整个研发到生产的流程可能是这样：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8841b2a579454dddbe8e1f08491c0fbe~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

相比上面两个，这个就更复杂一些了，不仅有串行任务还有并行任务，如何去设计这样一个系统？单纯的通过状态字段去描述显然已经不够用了，此时我们就得考虑一种通用的、更易维护的方案来实现这样的系统了，这种通用的、易维护的方案，也就是工作流。

2\. 三大工作流
---------

一个比较早的工作流是 jBPM，这是一个由 Java 实现的企业级流程引擎，是 JBoss 公司开发的产品之一。

jBPM 的创建者是 Tom Baeyens，这个大佬后来离开了 JBoss，并加入到 Alfresco，并推出了基于 jBPM4 的开源工作流系统 Activiti，而 jBPM 则在后续的代码中完全放弃了 jBPM4 的代码。从这个过程中也能看出来，jBPM 在发展过程中，由于意见相左，后来变成了两个 jBPM 和 Activiti。

然而戏剧的是，Activiti5 没搞多久，从 Activiti 中又分出来一个 Camunda，Activiti 继续发展，又从中分出来一个 Flowable。。。

由于开发 jBPM、Activiti、Camunda 以及 Flowable 的人多多少少有一些关联性，让人不得不猜测意见相左拉一票人出来单干是他们的企业文化。

所以现在市面上主流的流程引擎就一共有三个：

*   Activiti
*   Flowable
*   Camunda

这三个各有特点：

1.  Activiti 目前是侧重云，他目前的设计会向 Spring Cloud、Docker 这些去靠拢。
2.  Flowable 核心思想还是在做一个功能丰富的流程引擎工具，除了最最基础的工作流，他还提供了很多其他的扩展点，我们可以基于 Flowable 实现出许多我们想要的功能（当然这也是小伙伴们觉得 Flowable 使用复杂的原因之一）。
3.  Camunda 相对于前两个而言比较轻量级，Camunda 有一个比较有特色的功能就是他提供了一个小巧的编辑器，基于 bpmn.io 来实现的（松哥之前已经发文讲过了）。如果你的项目需求是做一个轻巧的、灵活的、定制性强的编辑器，工作流是嵌入式的，那么可以选择 Camunda。

如果仔细比较起这三个的差异，能列一个长长的表格，这个网上也有不少人都总结过了，松哥这里也就不啰嗦了。

3\. 流程图
-------

既然有三个不同的工作流，那么三个不同的工作流画出来的流程图是否都各不相同呢？

不是的。

工作流程图这块其实有一个统一的标准，那就是 BPMN。BPMN 全称是 Business Process Model and Notation，中文译作业务流程模型和标记法，这个中文太绕口了，还是简称 BPMN 吧。

这是一套图形化表示法，用图形来表示业务流程模型。BPMN 最初由业务流程管理倡议组织（BPMI, Business Process Management Initiative）开发，BPMI 于 2005 年与对象管理组织（OMG, Object Management Group）合并，并于 2011 年 1 月 OMG 发布 2.0 版本，同时改为现在的名称。

一句话，就是流程图这块有一个特别古老的规范，那就是 BPMN，而我们前面所说的无论是 Activiti、Flowable 还是 Camunda，都是支持这个规范的，所以呢，无论你使用哪一个流程引擎，都可以使用同一套流程图。

那么这个规范究竟都说了些什么事情呢？

我们以上面生产笔记本的流程图为例，来和小伙伴们做一个简单介绍：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/585d7396bb754cba9e308b5c501a2fa8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

从上图中可以看到，一个流程图中主要包含四方面的内容：

*   事件
*   连线
*   任务
*   网关

我们一个一个来说。

**事件**

首先在一个流程图中应该有开始事件和结束事件，也就是上图大家看到的两个圆圈。另外还有一些中间事件、边界事件等。举个中间定时事件的例子，比如用户下单之后，可以有一个中间定时事件，延迟 5 分钟发货。

**连线**

连线就是将事件、任务、网关等连在一起的线条，一般情况下就是普通连线，有的时候连线会有一些条件，例如松哥之前文章和大家分享的请假，如果经理同意请假申请，就走哪一个线条，如果经理不同意请假申请，就走哪一个线条。对应上图的笔记本生产，如果经理审批通过，就载入图纸准备生产，如果经理审批不通过，就重新设计。

**任务**

任务这块其实有很多分类。

如果细分大致上可以分为如下几种：

*   接收任务

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/103216801d594dd1917d9c03e4662d30~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

在上面的流程图中，等待准备工作完成这一项就是一个接收任务。这个任务里并不需要额外做什么事情，流程到这一步就自动停下来了，需要人工去点一下，推动流程继续向下执行。

*   发送任务

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b080d215f0594cb98f80b1841e3d2c44~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

这个一般用来把消息发送给外部参与者。

*   服务任务

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10f0d4b5069448849fbaf079ad6349f6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

这个一般由系统自动完成，其实说白了就是我们的一个自定义类，可以在一个自定义类里边完成想要做的事情。

*   脚本任务

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a974218dc7442c69b7d2d56ffc22b39~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

一个自动化活动。当流程执行到脚本任务时，自动执行相应的脚本。

*   业务规则任务

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58afed386a2947c1bf8c87c7a56b2d0a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

BPMN2.0 新引入用来对接业务规则引擎，业务规则任务用于同步执行一个或多个规则。

*   用户任务

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7729d78abd824128b2b0ca6b3dc33725~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

用于为那些需要由人工参与者完成的工作建模。

虽然细分类别很多，但是仔细看，其实这几种又可以归为两大类：

1.  用户任务：表示人工要介入做的事情。比如同意与否，或者输入一些参数，要让人工完成任务，就需要一个表单系统，让人工输入数据，或者显示数据给人看，这也是为什么用户任务和表单系统结合在一起的原因，用户任务需要用户向引擎提交一个完成任务的动作，否则流程会暂停在这里等待。
2.  服务任务：表示机器自动做的事情。调用服务的任务，这个服务可以是一个 Spring JavaBean，也可以是一个远程 REST 服务，流程会自动执行服务任务。

**活动**

活动可以算是一种特殊的任务。活动可以调用另外一个流程使之作为当前流程的子流程去运行。活动也可以分为用户活动、脚本活动等等。从显示上来说，活动比任务边框深一些。仅此而已。

**网关**

网关要是细分起来，也有很多不同类型的网关。

*   互斥网关

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8619ebc83464c0db87fe279c6c10c34~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

这种网关也叫排他性网关，我们之前请假流程中的那个网关，就是互斥网关。这种网关有且仅有一个有效出口。

*   相容网关

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eab3343e89f44f2fa20c131d50585d89~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

这种网关会有多个出口，只要条件满足，都会执行。

*   事件网关

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0d61894874f4d17bd8547b73ab47c66~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

事件网关是通过中间事件驱动，它在等待的事件发生后才会触发决策。基于事件的网关允许基于事件作出决策。

*   并行网关

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3097f46d9c4f457dbea0b3d96daf6db9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

并行网关一般是成对出现的，上面生产笔记本的那个流程中，生产屏幕、键盘等并行操作，就是通过并行网关来实现的。

4\. 流程绘制工具
----------

流程绘制有一个经典工具：Flowable Eclipse Designer，但是，这是一个 Eclipse 插件。

那么我们现在大部分小伙伴都是用 IDEA，IDEA 中该如何处理流程绘制呢？

1.  IDEA 默认就有一个流程图绘制工具，当在 IDEA 中打开一个流程图的 XML 文件的时候，可以选择 Designer，就可以通过可视化的方式去查看这个流程图，默认的不推荐。
2.  IDEA 中也有一个流程绘制插件 flowable-bpmn-visualizer，但是这个插件也不好用，不过勉强能用。

其他的绘制工具：

1.  flowable-ui 这是官方提供的一个 flowable 的工具，里边有很多功能，包括画流程图。
2.  bpmn.js 这个工具是 Camunda 提供的，可以嵌入到我们当前的项目中，利用这个 bpmn.js 可以开发一个流程绘制工具。原生的 bpmn.js 画出来的流程图只能在 Camunda 中使用，但是经过改造之后，就可以在 flowable 中使用了。

接下来松哥和大家介绍 IDEA 中的流程绘制插件和 flowable 以及 bpmn.js 这三个工具。我们分别来看。

### 4.1 flowable-bpmn-visualizer

插件安装：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/856b30408b8e46e28fc57c26b3418efc~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

装好之后重启 IDEA 即可。

在 IDEA 中，当我们安装了这个插件之后，新建文件文件的时候，就有相应的选项：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7b145468bdc44dc96a5c72deb62bb85~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

选择这个就可以新建一个流程图了，假设我们现在想绘制下面这样一个流程图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6c972aa3588436eb8cecfe4ded2d4d5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

绘制关键节点：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e292168baa68487183054e92d867a4ea~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

注意，这里如果是传递变量需要用 ${} 表达式，如果是字符串，直接写即可，例如，这个节点由一个名为 javaboy 的用户来处理，那么写法如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c2b9fe87d4c4cffbd844ef731cda9fe~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

注意，从排他性网关出来的线条中，有一个 Condition expression，这个表示这个线条执行的条件。以下图为例，具体来说，就是当用户在审批的时候，本质上其实就是传递一个变量，变量值为 true 或者 false。下图中的 `${approve}`表示这个变量的名字为 approve。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b362bc1cadda4a8ba338ce3836071c67~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

然后再来看发邮件的服务：

齿轮表示这是一个服务任务，也就是系统自动完成的，系统自动完成的方式有很多种，其中一种是提前将自己的业务逻辑在 Java 类中写好，然后这里配置一下类的完整路径即可。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a4b5e73a3624cd8b9d189c2b04f19f2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

下图表示的是从网关出来之后，approve 变量如果为 false，那么就进入到请求被拒绝的服务中。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21f8bf7244404d8d8e73d37d34e6e132~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 4.2 flowable-ui

这个是 Flowable 官方推荐的一个流程引擎辅助工具。

#### 4.2.1. 安装

flowable-ui 有两种安装方式：

官方提供的是一个 war 包，这个虽然是一个 war 包，但是除了将之扔到 Tomcat 中去运行之外，也可以直接执行 java -jar xxx.war 这个命令去启动这个 war 包。

war 下载地址：[github.com/flowable/fl…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fflowable%2Fflowable-engine%2Freleases%2Fdownload%2Fflowable-6.7.2%2Fflowable-6.7.2.zip "https://github.com/flowable/flowable-engine/releases/download/flowable-6.7.2/flowable-6.7.2.zip")，这个 zip 包下载之后，里边有一个 wars 文件夹，里边包含了 flowable-ui 的 war 包。然后，就像启动 Spring Boot 一样，直接启动这个 war 包即可：

文件位置：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f6834c5bf82430983b4eebd09e7ee19~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

启动命令：

```null
java -jar flowable-ui.war

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f51c9bdf574403dac8618b8db7cf007~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

启动之后，默认的端口号是 8080。

启动之后，浏览器输入 [http://localhost:8080/flowable-ui/idm/#/login](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%3A8080%2Fflowable-ui%2Fidm%2F%23%2Flogin "http://localhost:8080/flowable-ui/idm/#/login")，如果看到如下页面，表示启动成功：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04c3bee454004d23a4af314794d3fbba~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

另外，我们也可以使用 docker 来安装，命令如下：

```arduino
docker run -p 8086:8080 -d  flowable/flowable-ui

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3e435ffd29f42afa7eec12953dda6cf~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 4.2.2. 登录

默认的登录用户名是 admin，默认的登录密码是 test。

看到如下页面，表示登录成功。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc94377caccb426a9c14538827b0eb34~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 4.2.3. 功能模块

flowable-ui 是完整的 flowable 体验 DEMO，而不仅仅只是一个流程图的绘制工具。所以它里边不仅可以画流程图，还可以运行流程图，既然能够运行流程图，那么就需要身份管理。

1.  任务应用程序：我们绘制好的流程图，可以直接将之发布到一个应用中，然后在这里进行部署，这个模块其实就是这些部署的应用程序。
2.  建模器应用程序：这个专门用来画流程图的。
3.  管理员应用程序：这个主要用来管理应用，一些具有管理员权限的用户，可以通过这个功能模块去查询 BPMN、DMN、FORM 等等信息。
4.  身份管理应用程序：这个功能模块，为所有的 flowable-ui 应用程序提供一个单点登录功能，并且还可以为这些用户设置用户组、用户权限等。

#### 4.2.4. 身份管理应用程序

创建用户流程：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/149bf8d9a2fc40aabe9e338206c81f34~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

填入用户的基本信息，点击保存按钮，就可以完成用户的创建了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c46c71b3ca86496e84ba99a5557c9416~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

新建的用户不属于任何用户组，所以这个新建的用户是没有权限的，我们现在就可以使用这个新建的用户登录，但是登录成功后，看不到任何功能模块。

用户创建成功之后，可以点击上面的用户组功能，创建用户组：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11f2d30ac3ef4ea19c8102acd54bda23~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

将来我们在画流程图的时候，可以设置某一个 UserTask 由某一个用户组来处理，这个用户组中的所有用户，将来都可以处理这个 UserTask。

组创建成功之后，可以为这个组添加用户：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30afa8a92f2546668dae2ab3904b1f86~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

最后，我们可以在权限控制中，为用户或者用户组添加相应的权限。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00f272fe163b40ad8b047e17a8755424~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

*   访问 idm 应用：访问功能4.
*   访问 admin 应用：访问功能3。
*   访问 modeler 应用：访问功能2.
*   访问 workflow 应用：访问功能1.
*   访问 REST API：访问 REST API 接口的。

#### 4.2.5. 管理员应用程序

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f32510d68cd4a48acd88ee66c1c5f64~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 4.2.6. 建模器应用程序

核心功能，主要就是画流程图。

绘制一个报销流程图，大致流程：

1.  启动一个流程。
2.  执行一个用户任务，这个用户任务交给流程的启动人去执行。这个用户任务中，填入报销材料，例如用户名、金额、用途。
3.  系统自动判断一下/或者人工判断报销金额是否大于 1000。
4.  如果报销金额小于等于 1000，那么这个报销任务交给 组长审批：

```markdown
1.  组长审批通过，则流程结束。
1.  组长审批不通过，则流程回到第 2 步，用户重新去填写报销资料。

```

5.  如果报销金额大于 1000，那么这个报销任务先交给经理审批：

```markdown
1. 经理审批通过，则交给 CEO 审批：

```

```null
 CEO 审批通过，流程结束。
 CEO 审批不通过，流程回到步骤 2 中。

```

```markdown
2.  经理审批不通过，则流程回到步骤 2 中。

```

接下来就开始绘制：

##### 绘制流程

首先创建一个流程：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13b997c329924a47b3714dd4dd94a6f4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**注意，模型的 key 在当前应用中必须是唯一的，将来我们通过 Java 代码去操作这个模型的时候，就是通过模型 key 去识别这个模型。** 

绘制出来的流程图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da54d215080f43c5942fa3f4491aba55~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**注意，在一个流程图中，开始节点必须有且只有一个，结束节点可以有多个。** 

##### 表单问题

在流程中，传递流程参数有两种方式：

*   流程变量。
*   表单。

这两种方式都可以传递参数，区别在于，流程变量是零散的，而表单是整的。

对于通过表单传递的参数，我们也可以按照流程变量的方式去访问单个的表单参数，例如在上面的流程图中，我们有 `${money <= 1000}`，这里的 money 实际上是表单中的参数，但是我们可以直接通过 表达式去访问。还有如‘ 表达式去访问。还有如 `{managers\_approve\_or\_reject\_radio_button=="拒绝"}`，也是直接访问表单中的变量。

##### 任务处理人

对于一个 UserTask 而言，任务处理人有四种：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/680f3a0cb87040e58e0aefe84e92cf75~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

*   流程发起人，由流程的启动人/发起人来处理这个流程。
*   单个用户，直接指定某一个具体的用户来处理这个流程，注意这里只能指定一个用户，并且这个用户将来在处理任务的时候，不需要认领，直接就可以处理。
*   候选用户：可以同时指定多个用户来处理这个 UserTask，将来用户在处理的时候，需要先认领（Claim）任务，然后才能处理。
*   候选组：可以同时指定多个用户组来处理这个 UserTask，这个处理的时候，也需要先认领，再处理。

##### 基本概念

*   流程定义（ProcessDefinition）：我们绘制的流程图、流程的 XML 文件，就是我们的流程定义。
*   流程（ProcessInstance）：一个启动了流程实例就是一个流程，流程可以是已经执行完毕的，也可以是正在执行中的。流程的定义相当于是一个类，而流程则相当于是一个对象。
*   任务（Task）：一个 ProcessInstance 中，需要具体处理的节点就是一个任务。

#### 4.2.7. 任务应用程序

在 flowable-ui 中，绘制好的流程图，可以直接部署称为一个 App。

### 4.3 bpmn.js

bpmn.js 是一个工具包，利用这个工具包，我们可以非常方便的在浏览器中创建、嵌入或者扩展一个 BPMN 流程图，重要的是，这个过程非常 Easy，我们只需要少量代码即可实现这一目标。

不知道看文章的小伙伴们日常工作中接触流程图多不多，如果经常接触的话，我估计有不少小伙伴可能都见过基于 bpmn.js 构建出来的流程图绘制工具。

因为 flowable-ui 这种太重量级了，如果我们单纯的只是想在自己的项目中嵌入一个流程绘制工具，那么显然 bpmn.js 是最佳选择。

bpmn.js 自己从头构建比较麻烦，给大家推荐两个常用的库。

#### 4.3.1 workflow-bpmn-modeler

workflow-bpmn-modeler 基于 Vue 和 bpmn.io@7.0，实现了 flowable 的工作流设计器。使用这个流程绘制工具，建议采用 flowable6.4.1 版本，flowable6.4.2 版本开始进行商业化重构，为了方便刨码学习，推荐使用 flowable6.4.1 版本。

这个用法其实很简单，首先我们创建一个 Vue2 的项目，注意 Vue 的版本不要选错了，项目创建好之后，添加 workflow-bpmn-modeler 依赖，执行如下任意命令添加：

```css
npm i workflow-bpmn-modeler

```

或者：

```csharp
yarn add workflow-bpmn-modeler

```

添加完成后，package.json 内容如下：

```json
{
  "name": "bpmn_demo02",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build"
  },
  "dependencies": {
    "core-js": "^3.8.3",
    "element-ui": "^2.15.9",
    "vue": "^2.6.14",
    "vue-router": "^3.5.1",
    "workflow-bpmn-modeler": "^0.2.8"
  },
  "devDependencies": {
    "@vue/cli-plugin-babel": "~5.0.0",
    "@vue/cli-plugin-router": "~5.0.0",
    "@vue/cli-service": "~5.0.0",
    "vue-template-compiler": "^2.6.14"
  }
}

```

**注意看版本号。** 

接下来我们就可以在一个 .vue 文件中使用这个组件了，代码如下：

```vue
<template>
  <div>
    <bpmn-modeler
            ref="refNode"
            :xml="xml"
            :users="users"
            :groups="groups"
            :categorys="categorys"
            :is-view="false"
            @save="save"
    />
  </div>
</template>

<script>
  import bpmnModeler from "workflow-bpmn-modeler";

  export default {
    components: {
      bpmnModeler,
    },
    data() {
      return {
        xml: "", // 后端查询到的xml
        users: [
          { name: "javaboy", id: 1 },
          { name: "itboyhub", id: 2 },
          { name: "江南一点雨", id: 3 },
        ],
        groups: [
          { name: "经理", id: 4 },
          { name: "组长", id: 5 },
          { name: "员工", id: 6 },
        ],
        categorys: [
          { name: "OA", id: "oa" },
          { name: "财务", id: "finance" },
        ],
      };
    },
    methods: {
      getModelDetail() {
        // 发送请求，获取xml
        // this.xml = response.xml
      },
      save(data) {
        console.log(data);  // { process: {...}, xml: '...', svg: '...' }
      },
    },
  };
</script>

```

我们来分析一下这段代码：

1.  首先从 workflow-bpmn-modeler 中导入 bpmnModeler。
2.  注册 bpmnModeler 组件。
3.  在页面中直接使用 bpmnModeler 组件，使用该组件时候，有五个属性一个方法，我们挨个来说：
    1.  xml：这个属性是 bpmnModeler 要展示的流程图的 XML 字符串，我们可以提前给一个流程图的 XML 字符串，这样 bpmnModeler 组件就会将这个 XML 字符串所对应的流程图给画出来，如果我们只是单纯的想自己绘制流程图，那么这个可以不用管，给一个空字符串就行了。
    2.  users：这是一个数组，当我们配置 UserTask 的时候，可以设置这个 UserTask 由谁来处理，users 配置的就是这里用到的用户。
    3.  groups：这个类似于 users，也是在 UserTask 中，如果我们想要配置一个 UserTask 的候选组的话，那么就会用到 groups 中的内容。
    4.  categorys：这个属性亲测没有任何功能，源代码我也看了，源代码中也没有用这个属性，这完全就是一个没有用的属性，可忽略之。
    5.  is-view：这个表示当前 bpmnModeler 是要画流程图还是单纯的只是将流程图展示出来，false 表示我们是用 bpmnModeler 画流程图的，如果设置为 true，就表示 bpmnModeler 只是用来展示流程图（提前准备好流程图的 XML 文件，可以用 bpmnModeler 将之展示出来）。
    6.  @save：这个是点击网页上的保存模型按钮的时候，会触发的一个回调函数。

好啦，这就可以了。

接下来，我们启动 Vue 项目，就可以看到这个流程图绘制页面了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89c46d5cf388493a8be091707c67b619~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

现在就可以愉快的画流程图了～

接下来，松哥就用这个，手把手教大家画一下请假流程图，流程图是这样的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f008493a166644ed9c709f3e835c3846~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们来看下如何绘制：

1.  首先我们先来定义一下流程的基本信息：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/960ac1e7a6384f2fb9e488688df9c718~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

2.  接下来绘制经理批准还是拒绝流程：

点击这个扳手按钮可以设置任务类型：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78c895cf13af49299eef24f0d05beb55~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

为这个任务设置一个监听器：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/451a4306972848ba9b5f6e8bbf6333fa~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

> 设置监听器的原因是因为前端用户在提交请假申请的时候，选择审批人可以直接选择审批人，也可以选择审批人的身份（例如经理），这两种选择都是被允许的。所以我们就添加一个任务监听器，当流程执行到这个 Task 的时候，我们就在任务监听器中，根据前端传来的参数去设置这个任务是由候选人处理还是候选用户组处理。

3.  添加互斥网关：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e21b8cb7a3845daa62685d4aaa36966~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

4.  审批通过线

接下来，先是审批通过，审批通过的条件是 approved 字段为 true 就表示审批通过：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d61be7dd629f4d6d906320f9fc4b795b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

5.  审批通过发送通知

审批通过后，给用户发送一个通知，这是一个服务任务，发送通知的类是我们自己写的，所以也需要配置一下自定义类的位置：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/271144a76c504908967fe4e7dcb5247c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

6.  结束

最后进入到审批通过 UserTask 并且结束：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc9c68c4a2844c79979ce951d30e23ba~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43dab03b47dd49728d2ca93a4e76611d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

7.  绘制拒绝线

按照如上流程，继续绘制请假被拒绝的流程：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c6eda8b37984ff5864baf0f6e1e1246~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 4.3.2 muheflow-bpmn-modeler

松哥要和大家介绍的第二个工具就是 muheflow-bpmn-modeler，这个基于 Vue 和 bpmn.io@8.0，实现了 flowable 的工作流设计器。使用这个流程绘制工具，建议采用 flowable6.4.1 版本，flowable6.4.2 版本开始进行商业化重构，为了方便刨码学习，推荐使用 flowable6.4.1 版本。

没找到这个的源代码，但是我发现这个的用法和 workflow-bpmn-modeler 的用法毫无差别～所以我就不废话了，照着上面的用这个就行了。
