# 10 大 Helm 教程，助您开启 Kubernetes 之旅 
![](https://www.jfrogchina.com/wp-content/uploads/2020/08/HelmChartChampions-03.png)

Kubernetes 发展惊人，K8s 应用程序的重要性日益增加，其复杂性更高。 如今，即使配置单个应用程序，也可能需要创建较多相互依赖的 K8s 源，每个源都依托于编写详细的 YAML 清单文件。 考虑到这一点，可主要发挥 Helm 作为 Kubernetes 的包管理器的作用，用户可对其 K8s 配置进行重用。

Helm 初学
-------

Helm 是 Kubernetes 的首选应用程序包管理器，您可使用 [Helm Charts](https://www.jfrog.com/confluence/display/JFROG/Kubernetes+Helm+Chart+Repositories) 描述应用程序的结构。 使用 Helm 命令行界面，您可以回滚部署、监控应用程序的状态并跟踪每项部署的历史记录。 在定义、存储和管理服务器端应用程序的方式方面，Helm 带来了巨大变化。 2019 年 4 月，CNCF [宣布 Helm 的孵化期结束，成为一个完整的项目](https://www.cncf.io/announcement/2020/04/30/cloud-native-computing-foundation-announces-helm-graduation/)，这意味着 Helm 将获得比过去更多的资源。

Helm 的主要功能包括：

*   查找和使用打包成 Helm Charts 的流行 K8s 软件
*   将 K8s 应用程序作为 Helm Charts 进行分享
*   为 Kubernetes 应用程序创建可复制的构建过程
*   管理 Kubernetes 清单文件
*   对 Helm 包发布进行管理

### 为何选择 Helm Charts？

Helm 配置文件被称为 Chart ，由一些 YAML 文件组成，元数据和模板呈现在 Kubernetes 清单文件中。 Chart 的基本目录结构包括：

package-name/
  charts/
  templates/
  Chart.yaml
  LICENSE
  README.md
  requirements.yaml
  values.yaml

使用 Helm 命令，您可从本地目录或上述目录结构的 .tar.gz 打包版本中安装 Chart 。 此类打包的 Chart 也支持从 Chart 制品库中自动下载和安装。

轻松驾驭 Helm
---------

有大量资源，可帮助您了解如何成功使用 Helm 部署 Kubernetes 应用程序。 此类资源中有较多为教程，旨在帮助初学者了解 Helm 及其工作原理。

以下是我最喜欢的一些视频教程，可帮助探究从基本到高级的 Helm 概念和实践。

### 1\. 什么是 Helm？

这段[关于 Helm 的介绍性视频教程](https://u2c.oss-cn-beijing.aliyuncs.com/jfrog_video/blog-10-helm-tutorials-to-start-your-kubernetes-journey/jfrog_video/What%20is%20Helm_.mp4)是由 IBM Cloud 的 David Okun 制作的。 该快速教程介绍了使用 Helm 在 Kubernetes 中快速定义、管理和轻松部署应用程序和服务的典型场景。

### 2\. Helm 简介

该视频由云原生计算基金会 (CNCF) 主操，[涵盖了 Helm 的基础知识和 Chart 的构成](https://u2c.oss-cn-beijing.aliyuncs.com/jfrog_video/blog-10-helm-tutorials-to-start-your-kubernetes-journey/jfrog_video/An%20Introduction%20to%20Helm%20-%20Matt%20Farina%2C%20Samsung%20SDS%20%26%20Josh%20Dolitsky%2C%20Blood%20Orange.mp4)。 还有对共享和使用 Helm Charts 的方法的阐述。

### 3\. [什么是 Kubernetes 中的 Helm？](https://www.jfrogchina.com/knowledge-base/what-is-helm-in-kubernetes/)

这段[来自 Techworld 的视频](https://u2c.oss-cn-beijing.aliyuncs.com/jfrog_video/blog-10-helm-tutorials-to-start-your-kubernetes-journey/jfrog_video/What%20is%20Helm%20in%20Kubernetes_%20Helm%20and%20Helm%20Charts%20explained%20%20_%20Kubernetes%20Tutorial%2023.mp4)涵盖了 Helm 的基础知识、模板引擎，甚至还指出了 Helm 的弊端。 视频描述中包含时间戳，可让您轻松找到您需要的教程部分！

###  4\. Helm 和 Kubernetes 简介

Matthew Palmer [为针对 Kubernetes 的 Helm 引入了 Node.js、Ruby 和 PHP 开发人员](https://u2c.oss-cn-beijing.aliyuncs.com/jfrog_video/blog-10-helm-tutorials-to-start-your-kubernetes-journey/jfrog_video/Helm%20and%20Kubernetes%20Tutorial%20-%20Introduction.mp4)。 该视频概述了 Helm 的 Chart 和发布，并深入研究 Helm 架构。 还提供示例代码，可用于将常规 Node.js 和 MongoDB Web 应用程序转换为 Helm Chart。

### 5\. 创建 Helm Charts

Bitnami 在 YouTube 上有[完整的 Helm Chart 教程](https://u2c.oss-cn-beijing.aliyuncs.com/jfrog_video/blog-10-helm-tutorials-to-start-your-kubernetes-journey/jfrog_video/Create%20a%20Helm%20chart.mp4)。 本教程面向 Helm 初学者，指导初学者如何创建 Helm Charts、部署示例应用程序、添加依赖项、对其进行打包和共享。

### 6\. Helm Charts 模式

该[视频教程来自 CNCF](https://u2c.oss-cn-beijing.aliyuncs.com/jfrog_video/blog-10-helm-tutorials-to-start-your-kubernetes-journey/jfrog_video/Helm%20Chart%20Patterns%20%5BI%5D%20-%20Vic%20Iglesias%2C%20Google.mp4)，深入讲解了 [Helm Charts 模式及最佳实践](https://www.jfrogchina.com/blog/helm-charts-best-practices/)，以在公共 Helm Charts 制品中查看和维护图表。

### 7\. 从零基础开始构建 Helm Charts

该[视频教程来自 CNCF](https://u2c.oss-cn-beijing.aliyuncs.com/jfrog_video/blog-10-helm-tutorials-to-start-your-kubernetes-journey/jfrog_video/Building%20Helm%20Charts%20From%20the%20Ground%20Up_%20An%20Introduction%20to%20Kubernetes%20%5BI%5D%20-%20Amy%20Chen%2C%20Heptio.mp4)，对构建 Helm Charts 时的关键 Kubernetes 概念进行了更详细的阐述。 这是一份关于构建 Helm Charts 的综合指南。

### 8\. 深入了解 Helm 安全

Matt Farina [阐释了 Helm 安全的一些基础知识](https://u2c.oss-cn-beijing.aliyuncs.com/jfrog_video/blog-10-helm-tutorials-to-start-your-kubernetes-journey/jfrog_video/Webinar_%20Helm%20Security%20%E2%80%93%20A%20Look%20Below%20Deck.mp4)，并极好地概述了社区如何共同构建和改进众多流程，以确保 Kubernetes [应用程序安全](https://www.jfrogchina.com/xray/features/)。

### 9\. 深入研究 Helm v3

该[视频](https://u2c.oss-cn-beijing.aliyuncs.com/jfrog_video/blog-10-helm-tutorials-to-start-your-kubernetes-journey/jfrog_video/Helm%203%20Deep%20Dive%20-%20Taylor%20Thomas%2C%20Microsoft%20Azure%20%26%20Martin%20Hickey%2C%20IBM.mp4)由 CNCF 主操。 Microsoft Azure 的 Taylor Thomas 和 IBM 的 Martin Hickey 就 Helm v3 中发生的变化进行讨论。 他们谈到了新的功能和支持此类功能的架构。 涵盖的话题多样，包括从 CLI 库的更改到 Chart 添加和新的客户端安全模型。

### 10\. 深入研究 Helm： [DevOps](https://www.jfrogchina.com/) 走在前沿

该[高级 Helm 教程](https://u2c.oss-cn-beijing.aliyuncs.com/jfrog_video/blog-10-helm-tutorials-to-start-your-kubernetes-journey/jfrog_video/Delve%20into%20Helm_%20Advanced%20DevOps%20%5BI%5D%20-%20Lachlan%20Evenson%20%26%20Adam%20Reese%2C%20Deis.mp4)深入探讨了 Helm，重点介绍了 Kubernetes 原生应用程序在不同环境中的生命周期管理和持续交付情况。 其展示了如何使用插件和附加服务扩展 Helm 的功能。

Helm 尽在掌握！

标签：[容器镜像仓库](https://www.jfrogchina.com/container-registry/)  [开源组件漏洞](https://www.jfrogchina.com/xray/features/)  [制品库](https://www.jfrogchina.com/integration/maven-repository/)  [ARTIFACTORY](https://www.jfrogchina.com/artifactory/)  [xray](https://www.jfrogchina.com/xray/)