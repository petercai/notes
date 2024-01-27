# 容器的那些事儿 - Thoughtworks洞见
近两年Docker可谓充满了争议，例如去年底K8s宣布不打算支持Docker，消息一出，大家争相讨论Docker的可替代方案，Colima作为Docker Desktop的热门开放替代方案，Podman作为Docker的替代方案，收到许多开发者和企业的关注，分别收录在Thoughtworks的[最新一期技术雷达](https://insights.thoughtworks.cn/technology-radar-26/)中。在今年Docker公司又宣布了Docker Desktop准备向中大型企业用户收费，“再见Docker”的声音再度高涨。从各方面来看，无论从技术的领导力，还是盈利能力，Docker公司都已经开始掉队，不禁要问，曾经风头无二的容器化大佬究竟怎么了？

### 容器兴起

把时间往回推几年，大家提起Docker，都会把它与容器技术画上等号，Docker引领容器技术变革的光荣岁月，依然让人满怀激动。在2013年，Cloud Foundry是云服务领域的绝对焦点，以Cloud Foundry为核心的PaaS平台服务能力变革席卷了整个云服务领域，同时也涌现了一批优秀的PaaS平台，包括Salesforce Heroku、IBM Red Hat等。其中一家叫dotCloud的公司，也是这股 PaaS 热潮中的一份子。但在当时的Cloud Foundry项目中，容器是最底层、最没人关注的那一部分，因此dotCloud的产品一直无人问津。为改变这种被动的现状，dotCloud公司决定开源自家的容器项目Docker。就在短短的几个月的时间里，Docker项目迅速崛起，很快就颠覆了Cloud Foundry定义的PaaS平台的玩法。那相比于Cloud Foundry，Docker到底存在什么样的决定性优势呢？

这要从Cloud Foundry的应用托管说起，Cloud Foundry最核心的组件就是一套应用的打包和分发机制。用户只需要执行一个命令，就能一键将应用发布到云端。为支持这样的特性，Cloud Foundry实际上会为不同语言或框架发布不同的打包器（BuildPack），然后把本地的可执行应用打包，并上传到Cloud Foundry，Cloud Foundry会调度合适的agent来下载应用，并完成部署。为保障agent上的服务相互隔离，Cloud Foundry会通过Cgroup和Namespace为应用提供隔离的“沙箱”运行环境。

Cloud Foundry 的首席产品经理在对比Cloud Foundry和Docker之后，也说过Docker并没有什么黑科技，和Cloud Foundry一样，也是依赖Cgroup和Namespace创建了“沙箱”运行环境而已。

那究竟Docker比Cloud Foundry强在哪呢？答案就在镜像这种特别的应用打包方式。

[![](https://insights.thoughtworks.cn/wp-content/uploads/2022/05/docker-colima-podman-container-1.png)
](https://insights.thoughtworks.cn/wp-content/uploads/2022/05/docker-colima-podman-container-1.png)

典型的应用运行环境包括代码、依赖和操作系统，Cloud Foundry可以保证代码和依赖一致，但无法保障系统环境，所以有时本地运行正常，在云端却不行，出了问题，也很难定位到系统环境的差异性。但Docker正好解决了这个痛点，通过Image构建出Rootfs，完美保障本地和云端运行环境的一致性。在此基础上，Docker还提供了友好的Docker CLI来创建Image，同时提供了Docker Hub来快速分发Image和同性交友。在Docker迅速取得成功之后，dotCloud公司也顺势改名为Docker，Docker公司俨然已经成为容器技术的代言人。

### 容器云之争

单纯解决应用打包并没有价值，企业真正需要解决的是应用部署问题。Docker公司也意识到容器平台化能力才是致胜关键。在2014年Docker公司很快发布了Swarm项目，依然保持着Docker的友好命令风格，几个命令就可以完成多机集群部署。在此基础上，不断招兵买马，例如收购了Fig项目(后改名为Compose)，以及专门负责容器网格的SocketPlane等，充实着自己的平台化能力。此时Docker公司掌握着容器的绝对话语权，而Docker公司的平台化战略，又切实挑战到了其它云服务平台的切身利益，其中就包括Google和RedHat等公司。为了改变Docker一家独大的局面，由Docker、Google、RedHat等公司共同成立了OCI（Open Container Initiative），旨在定义出镜像和容器运行时标准，将标准从Docker项目实现中抽离出来。显然，Docker公司不会花精力在这种削弱自己影响力的项目上，所以OCI规范一直进度缓慢。

[![](https://insights.thoughtworks.cn/wp-content/uploads/2022/05/docker-colima-podman-container-2.png)
](https://insights.thoughtworks.cn/wp-content/uploads/2022/05/docker-colima-podman-container-2.png)

但Google和RedHat没有坐以待毙，随后祭出了大杀器 --- Kubernetes，大家拉帮结伙，共同成立了CNCF基金会，以 Kubernetes 项目为基础，建立一个由开源基础设施领域厂商主导的、按照独立基金会方式运营的平台级社区，来对抗以 Docker 公司为核心的容器商业生态。Kubernetes凭借强大的设计和RedHat的优秀社区运营，很快催生了一个与众不同的容器编排与管理的生态，催生了Prometheus、Istio等一批优秀的开源产品。为了应对Kubernetes带来的挑战，Docker公司主打的则是Docker Native的战略，放弃现有的Swarm 项目，将容器编排和集群管理功能全部内置到 Docker 项目当中，这也为将来埋下了隐患 --- 尾大难掉。即使这样，Docker也无法与日益壮大的Kubernates社区相抗衡，最终以失败收场，Docker公司将容器运行时runc项目捐赠给 CNCF 社区，将 Docker 项目改名为 Moby，交给社区自行维护，而 Docker 公司将占有 Docker 这个注册商标，将Docker的客户转移到自己的手上，彻底地开始了自己的商业化运营。

### Docker的困局

容器云之争，最终以Docker公司的失败收场，痛失云平台高地之后，留给Docker公司的盈利点也不算太多：

*   软件付费
*   私有Hub
*   容器云平台
*   专业技术支持和培训

容器云平台的低占有率限制了收益，Docker公司为了保障盈利，宣布了Docker Hub、Docker Desktop收费的商业决策，颇有饮鸩止渴的感觉。从技术层面，Kubernates已经成为了容器编排的事实标准，而Container在整个版图中，也只属于最底层的部分。OCI的规范已初见雏形，同类容器运行时containerd已经取代Docker，成为大多数托管 Kubernetes服务采用的容器运行时。再谈Docker Hub，基本上成熟的云平台都有发布自己的Container Registry，例如Amazon Elastic Container Registry (ECR)等，私有Hub的收费方式还很脆弱。

由于设计理念的不同，Docker添加了许多组件来保障友好的用户体验，这是Kubernates不需要的，此外它不支持Kubernates的CRI规范，从下图可以看到，社区需要维护一个Docker Shim项目来桥接Kubernates Node和Docker。一开始Kubernetes为了接入Docker生态，自然愿意维护这一套项目，但随着OCI、CRI规范和产品的成熟，社区便失去继续维护该项目的动力。

[![](https://insights.thoughtworks.cn/wp-content/uploads/2022/05/docker-colima-podman-container-3.png)
](https://insights.thoughtworks.cn/wp-content/uploads/2022/05/docker-colima-podman-container-3.png)  
[![](https://insights.thoughtworks.cn/wp-content/uploads/2022/05/docker-colima-podman-container-4.png)
](https://insights.thoughtworks.cn/wp-content/uploads/2022/05/docker-colima-podman-container-4.png)

(图片来源：[Kubernetes is Removing Docker Support, Kubernetes is Not Removing Docker Support](https://cloud.redhat.com/blog/kubernetes-is-removing-docker-support-kubernetes-is-not-removing-docker-support))

痛失容器编排阵地，容器标准的话语权被逐渐削弱，再加上饮鸩止渴般的收费商业模式，仅凭借优秀的用户体验，Docker真的能留住大家吗？不禁让人想问，廉颇老矣，尚能饭否？

