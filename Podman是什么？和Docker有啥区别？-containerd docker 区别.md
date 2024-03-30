# Podman是什么？和Docker有啥区别？-containerd docker 区别
容器编排工具作为当今最重要的Web开发技术之一，众多强者都在尝试争夺这一行业的主导地位。

Podman是RedHat的一款产品，旨在使用类似于Kubernetes的方法来构建、管理和运行容器，作为一款主流容器的可靠替代产品，它吸引了开发人员的关注。自RHEL 8起，Red Hat用CRI-O/Podman取代了Docker Daemon。为什么Red Hat想要摆脱Docker Daemon?这是因为使用Docker Daemon运行Docker有以下这些问题：

*   单点故障问题，Docker Daemon一旦死亡，所有容器都将死亡
*   Docker Daemon拥有运行中的容器的所有子进程
*   所有Docker操作都必须由具有跟root相同权限的用户执行
*   构建容器时可能会导致安全漏洞

Podman通过直接与Image Registry、Image和Container进行互动，而不是通过守护进程来解决以上问题。此外，Podman还允许用户在没有完全root 权限的情况下运行容器。

更棒的是，它还提供了与Docker兼容的指令，使用户可以很轻易的从原本的Docker指令切换到Podman。

[![](https://s3.51cto.com/oss/202201/17/d24e52f1b15b8730368c4e4fc97f95d9.jpg)
](https://s3.51cto.com/oss/202201/17/d24e52f1b15b8730368c4e4fc97f95d9.jpg)

Podman和Docker这两种标准的容器化工具已经有近十年的历史了，今天这篇文章我们来对比下这两个技术，它们虽有不同，但还是非常适合一起使用。

### 1\. Docker什么?

Docker是标准的容器管理技术。Docker在行业中举足轻重，以至于大多数人一想到容器，就会想到Docker。

Docker是容器编排世界的一把瑞士军刀，在其他替代方案出现之前就已经提供了诸多特性。随着容器管理复杂度的增加，它也必须成长为一个独立的、自给自足的工具，以便能提供开发人员的所有需求。

Docker也在很短的时间内，就成为All-in-one解决方案的关键工具之一。其中一款就是Docker Swarm，这是一款由Docker原生的，可以让你组建群集和调度Docker引擎，以及用来创建和管理容器群的解决方案。

Docker的诸多辅助工具处理所有与容器编排相关的任务，从负载均衡到网络，使其成为行业的首选，不光是作为行业技术参考。

尽管Docker是一个强大的系统，但这种自给自足的模式也有它的缺点。虽然可以在开发的所有阶段创建和运行容器，但其他工具在与Docker集成交互时或多或少存在些困难。

近年来，随着许多其他用于特定任务的专用工具的出现，Docker成为许多开发人员的起点，随之，他们将一些任务分配给其他更轻量级的平台和工具。

### 2 . Podman是什么?

Podman是一种开源的Linux原生工具，旨在根据开放容器倡议(Open Container Initiative，OCI)标准开发、管理和运行容器和Pod。Podman是RedHat开发的一个用户友好的容器调度器，是RedHat 8和CentOS 8中默认的容器引擎。

它是一款集合了命令集的工具，设计初衷是为了处理容器化进程的不同任务，可以作为一个模块化框架工作。它的工具集包括：

*   Podman：Pod和容器镜像管理器
*   Buildah：容器镜像生成器
*   Skopeo：容器镜像检查管理器
*   Runc：容器运行器和特性构建器，并传递给Podman和Buildah
*   Crun：可选运行时，为Rootless容器提供更大的灵活性、控制和安全性

这些工具还可以与任何OCI兼容的容器引擎(如Docker)一起工作，使其易于转换到Podman或与现有的Docker安装一起使用。Kubernetes可以使用Podman吗?答案是：是的。事实上，Kubernetes和Podman在某些方面是相似的。

Podman对于容器有着不同的方法论。正如它的名字所暗示的那样，Podman可以创建一起工作的容器“Pod”，这是一个类似Kubernetes里Pod的特性。Pod在一个共同的命名空间里，作为一个单元来管理容器。

比较主要的好处是开发人员可以共享资源，在一个Pod中为同一个应用程序使用不同的容器：一个容器用于前端，另一个容器用于后端，还有一个数据库。Pod的配置可以导到Kubernetes兼容的YAML文件，并应用到Kubernetes集群中，从而允许容器更快地进入生产。

Podman的另一个特性是它是无守护进程的。守护进程是在后台运行的程序，它处理服务、进程和请求，没有用户界面。Podman是一种独特的容器引擎，因为它实际上并不依赖于守护进程，而是作为子进程启动容器和Pod。

你可能会问：“我为什么要使用Podman?”作为一种开发和管理工具，Podman具有独特的优势，这使得它在适当的环境中成为Docker的可行和有趣的替代品。或者一个与Docker并肩工作的强大补充，因为它支持与Docker兼容的CLI接口。

### 3 . Podman vs Docker：区别

Podman和Docker有许多共同的特性，但也有一些根本的区别。技术不分好坏，只是着重于哪个更适用于某些特定的场景。

[![](https://s4.51cto.com/oss/202201/17/4aac198e4a4acf14c41c6eead98916a4.jpg)
](https://s4.51cto.com/oss/202201/17/4aac198e4a4acf14c41c6eead98916a4.jpg)

Podman与Linux内核交互，通过runC容器运行时进程而不是Daemon来管理容器。Buildah实用程序用于替代Docker build作为容器镜像构建工具，Docker push被Skopeo替代，用于在注册表和容器引擎之间移动容器镜像。

(1) 架构

Docker使用守护进程，一个正在后台运行的程序，来创建镜像和运行容器。Podman是无守护进程的架构，这意味着它可以在启动容器的用户下运行容器。Docker有一个由守护进程引导的客户端——服务器逻辑架构;但Podman不需要此类守护进程。

(2) Root特权

由于Podman没有守护进程来管理其活动，也无需为其容器分配Root特权。Docker最近在其守护进程配置中添加了Rootless模式，但Podman首先使用了这种方法，并将其作为基本特性进行了推广。原因如下。

(3) 安全

Podman比Docker安全吗?Podman允许容器使用Rootless特权。Rootless容器被认为比Root特权的容器更安全。在Docker中，守护进程拥有Root权限，这使得它们易成为攻击者的首选入侵点。

Podman中的容器默认情况下不具有Root访问权限，这在Root级别和Rootless级别之间添加了一个自然屏障，提高了安全性。不过，Podman可以同时运行Root容器和Rootless容器。

(4) Systemd

如果没有守护进程，Podman需要另一个工具来管理服务并支持后台运行的容器。Systemd为现有容器创建控制单元或用来生成新容器。Systemd还可以与Podman集成，允许它在默认情况下运行启用了Systemd的容器，从而无需进行任何修改。

通过使用Systemd，供应商可以将他们的应用程序封装为容器用来安装、运行和管理，因为现在大多数应用程序都是通过这种方式打包和交付的。

(5) 构建镜像

作为一款自给自足的工具，Docker可以自己构建容器镜像。Podman则需要另一种名为Buildah的工具的辅助，该工具充分体现了它的特殊性：它是为构建镜像而设计的，而不是为构建容器而生。

(6) Docker Swarm

Podman不支持Docker Swarm，这可能会在某些项目中被刨除在外，因为使用Docker Swarm命令会产生一个错误。然而，Podman最近增加了对Docker Compose的支持，使其与Swarm兼容，从而克服了这个限制。当然，Docker由于其原生的特性，与Swarm当然融合得很好。

(7) All in one vs 模块化

也许这就是这两种技术的关键区别：Docker是一个独立的、强大的工具，在整个循环中处理所有的容器化任务，有优点也有缺点。Podman采用模块化的方法，依靠专门的工具来完成特定的任务。

### 4 . Podman vs Docker：他们能合作吗?

作为最好的、最易应用于Docker的替代方案——用户可以将Docker别名设置为Podman(别名Docker=Podman)，且不会出现任何问题，正如本演示\[1\]中所示——Podman是一个非常强大的容器化任务工具。

Podman会是Docker的替代品吗?

如果你要从头开始一个项目，Podman可以是一个首要的容器化技术选项。如果项目正在进行，并且已经在使用Docker，这还需要具体情况具体分析，实际情况并不一定值得去改。而且作为一款Linux原生的应用，它要求相关开发人员具备Linux的相关技能。

开发人员可以在开发阶段依赖Docker，然后在运行时环境中将项目推向Podman，从而结合使用这两种工具，并受益于Podman所提供的更安全性。由于它们都是OCI兼容的，因此，兼容性不是个问题。

Docker和Podman能共存吗?是的，而且会很好。许多开发人员一直在合用Docker和Podman来创建更安全、更高效、更敏捷的框架。它们有很多共同之处，无论是从Docker到Podman的转变，亦或是二者合并使用，都可以做到无缝衔接。