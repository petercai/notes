# Kubernetes操作概览 (principles of operations)

## 2.1 Kubernetes概览 (primer)

Kubernetes基于一系列特性实现了对容器运行时的抽象（从而可以兼容不同的底层容器运行时）。

（1）容器运行时接口（Container Runtime Interface, CRI）是Kubernetes用来与第三方容器运行时进行对接的标准化的抽象层。这样容器运行时与Kubernetes是解耦的，同时又能够以一种标准化的方式予以支持。

（2）运行时类（Runtime Class）是Kubernetes 1.12引入的新特性，并在1.14版中升级为beta。它对不同的运行时进行了归类。例如，gVisor或Kata容器运行时或许比Docker和Containerd能提供更优的隔离性。

Containerd已经赶超Docker成为Kubernetes中最普遍使用的容器运行时。它实际上是Docker的精简版本，只保留了Kubernetes需要的部分。


一个或多个主节点（Control plane node）与若干工作节点（worker node）构成一个完整的集群。

主节点负责管理整个集群，有时主节点也被称为头结点（head node）。这意味着主节点完成调度决策、监控集群、响应事件与集群变化等工作。因此通常将主节点称作控制平面（control plan）。

应用服务运行在工作节点上，有时也将工作节点称作数据平面（data plan）。每个工作节点都有属于自己的向主节点汇报关系，并监控若干任务的运行。

在Kubernetes上运行应用的过程可以简单分为如下几个部分。

●选用最喜欢的语言将应用以独立的微服务方式实现。

●将每个微服务在容器内打包。

●将每个容器集成到Pod。

●使用更高层面的控制面板将Pod在集群中部署，例如Deployment、DaemonSet、StatefulSet、CronJob等。

部署（Deployment）提供可扩展性和滚动更新，DaemonSet在集群的每个工作节点中都运行相应服务的实例，有状态应用部署于StatefulSet，同时那些需要不定期运行的临时任务则由定时任务（CronJob）来管理。

### Kubernetes is both of the following:

• A cluster
• An orchestrator

#### Kubernetes: Cluster
A Kubernetes cluster is one or more nodes providing CPU, memory, and other resources
for use by applications. Kubernetes supports two node types:
• Control plane nodes
• Worker nodes
Both types can be physical servers, virtual machines, or cloud instances, and both can
run on ARM and AMD64/x86-64. Control plane nodes must be Linux, but worker
nodes can be Linux or Windows.

**Control plane nodes** implement the Kubernetes intelligence, and every cluster needs at
least one. However, you should have three or five for high availability (HA).
Every control plane node runs every control plane service. These include the API server,
the scheduler, and the controllers that implement cloud-native features such as self-
healing, autoscaling, and rollouts.

**Worker nodes** are for running user applications.
![[Kubernetes in brief-2024825-4.png]]
It’s common to run user applications on control plan nodes in development and test environments

#### Kubernetes: Orchestrator
Orchestrator is jargon for a system that deploys and manages applications. Kubernetes is the industry-standard orchestrator and can intelligently deploy applications across nodes and failure zones for optimal performance and availability. It canalso fix them when they break, scale them when demand changes, and manage zero-downtime rolling updates.

## 2.2 主节点(Control plane node)与工作节点(Worker node)
一个Kubernetes集群由主节点（Control plane node）与工作节点（Worker node）组成。这些节点都是Linux主机，可以运行在虚拟机（VM）、数据中心的物理机，或者公有云/私有云的实例上。

2.2.1 主节点（Control plane node）
Kubernetes的主节点（Control plane node）是组成集群的控制平面的系统服务的集合。

一种最简单的方式是将所有的主节点服务（Service）运行在同一个主机上。但是这种方式只适用于实验或测试环境。在生产环境上，主节点的高可用部署是必需的。这也是为什么主流的云服务提供商都实现了自己的多主节点高可用性（Multi-master High Availability），并将其作为自身Kubernetes平台的一部分，比如Azure Kubernetes Service（AKS）、AWS Elastic Kubernetes Service（EKS）以及Google Kubernetes Engine（GKE）。

一般来说，建议使用3或5个副本来完成一个主节点高可用性部署方案。

主节点的各个服务间有什么不同:

1.API Server
API Server（API服务）是Kubernetes的中央车站。所有组件之间的通信，都需要通过API Server来完成。全部组件的通信都是通过API Server来完成，这包括了系统内置组件以及外部用户组件。

API Server对外通过HTTPS的方式提供了RESTful风格的API接口，读者上传YAML配置文件也是通过这种接口实现的。这些YAML文件有时也被称作manifest文件，它们描述了用户希望应用在运行时达到的期望状态（desired state）。期望状态中包含但不限于如下内容：需要使用的容器镜像、希望对外提供的端口号，以及希望运行的Pod副本数量。

访问API Server的全部请求都必须经过授权与认证，一旦通过之后，YAML文件配置就会被认为是有效的，并被持久化到集群的存储中，最后部署到整个集群。

2.集群存储 (cluster store)
在整个控制层中，只有集群存储是有状态（stateful）的部分，它持久化地存储了整个集群的配置与状态。因此，这也是集群中的重要组件之一——没有集群存储，就没有集群。

通常集群存储底层会选用一种常见的<span style="background:#fff88f">分布式数据库etcd</span>。因为这是整个集群的唯一存储源，用户需要运行3～5个etcd副本来保证存储的高可用性，并且需要有充分的手段来应对可能出现的异常情况。

在关于集群的可用性（availability）这一点上，etcd认为一致性比可用性更加重要。这意味着etcd在出现脑裂的情况时(etcd prefers an odd number of replicas to help avoid split brain
conditions)，会停止为集群提供更新能力，来保证存储数据的一致性。但是，就算etcd不可用，应用仍然可以在集群性持续运行，只不过无法更新任何内容而已。

对于所有分布式数据库来说，写操作的一致性都至关重要。例如，分布式数据库需要能够处理并发写操作来尝试通过不同的工作节点对相同的数据进行更新。etcd使用业界流行的RAFT一致性算法来解决这个问题。

3.controller管理器 (controllers and controller manager)
Kubernetes uses controllers to implement a lot of the cluster intelligence. They all run on
the control plane, and some of the more common ones include:
• The Deployment controller
• The StatefulSet controller
• The ReplicaSet controller
They all run as background watch loops, reconciling observed state with desired state.

controller管理器实现了全部的后台控制循环，完成对集群的监控并对事件作出响应。

controller管理器是controller的管理者（controller of controller），负责创建controller，并监控它们的执行。Kubernetes also runs a controller manager that is responsible for spawning and managing the individual controllers.
![[Kubernetes in brief-2024825-5.png]]
一些控制循环包括：工作节点controller、终端controller，以及副本controller。对应的每个controller都在后台启动了独立的循环监听功能，负责监控API Server的变更。这样做的目的是保证集群的当前状态（current state）可以与期望状态（desired state）相匹配。

每个控制循环实现上述功能的基础逻辑大致如下。
（1）获取期望状态。
（2）观察当前状态。
（3）判断两者间的差异。
（4）变更当前状态来消除差异点。

上面的逻辑是Kubernetes与声明式设计模式的关键所在。

每个控制循环都极其定制化，并且仅关心Kubernetes集群中与其相关的部分。感知系统的其他部分并调用这种复杂的事情我们是绝对不会尝试的，每个控制循环都只关心与自己相关的逻辑，剩下的部分会交给其他控制循环来处理。这就是如Kubernetes这样的分布式系统设计的关键点所在，也与UNIX设计哲学不谋而合：每个组件都专注做好一件事，复杂系统是通过多个专一职责的组件组合而构成的。

4.调度器  (scheduler)
从整体上来看，调度器的职责就是通过监听API Server来启动新的工作任务，并将其分配到适合的且处于正常运行状态的节点中。为了完成这样的工作，调度器实现了复杂的逻辑，过滤掉不能运行指定任务的工作节点，并对过滤后的节点进行排序。排序系统非常复杂，在排序之后会选择分数最高的节点来运行指定的任务。

当确定了可以执行任务的具体节点之后，调度器会进行多种前置校验。这些前置校验包括：节点是否仍然存在、是否有affinity或者anti-affinity规则、任务所依赖的端口在当前工作节点是否可以访问、工作节点是否有足够的资源等。不满足任务执行条件的工作节点会被直接忽略，剩下的工作节点会依据下面的判定规则计算得分并排序，具体包括：工作节点上是否已经包含任务所需的镜像、剩余资源是否满足任务执行条件，以及正在执行多少任务。每条判定规则都有对应的得分，得分最高的工作节点会被选中，并执行相应任务。

如果调度器无法找到一个合适的工作节点，那么当前任务就无法被调度，并且会被标记为暂停状态。

调度器并不负责执行任务，只是为当前任务选择一个合适的节点运行。

5.云controller管理器(cloud controller manager)
如果用户的集群运行在诸如AWS、Azure、GCP、DO和IBM等类似的公有云平台上，则控制平面会启动一个云controller管理器（cloud controller manager）。云controller管理器负责集成底层的公有云服务，例如实例、负载均衡以及存储等。举一个例子，如果用户的应用需要一个面向互联网流量的负载均衡器，则云controller管理器负责提前在对应的公有云平台上创建好相应的负载均衡器。
![[Kubernetes in brief-2024825-7.png]]


2.2.2 工作节点 (worker node)
工作节点是Kubernetes集群中的工作者。从整体上看，工作节点主要负责如下3件事情。
1.监听API Server分派的新任务。
2.执行新分派的任务。
3.向控制平面回复任务执行的结果（通过API Server）。
![[Kubernetes in brief-2024825-8.png]]

工作节点的3个主要功能。

#### 1.Kubelet

Kubelet是每个工作节点上的核心部分，是Kubernetes中重要的代理端，并且在集群中的每个工作节点上都有部署。实际上，<u>通常工作节点与Kubelet这两个术语基本上是等价的</u>。

在一个新的工作节点加入集群之后，Kubelet就会被部署到新节点上。然后Kubelet负责将当前工作节点注册到集群当中，集群的资源池就会获取到当前工作节点的CPU、内存以及存储信息，并将工作节点加入当前资源池。

Kubelet的一个重要职责就是负责监听API Server新分配的任务。每当其监听到一个任务时，Kubelet就会负责执行该任务，并维护与控制平面之间的一个通信频道，准备将执行结果反馈回去。

如果Kuberlet无法运行指定任务，就会将这个信息反馈给控制平面，并由控制平面决定接下来要采取什么措施。例如，如果Kubelet无法执行一个任务，则其并不会负责寻找另外一个可执行任务的工作节点。Kubelet仅仅是将这个消息上报给控制平面，由控制平面决定接下来该如何做。

#### 2.容器运行时 (runtime)
Kubelet需要一个容器运行时（container runtime）来执行依赖容器才能执行的任务，例如拉取镜像并启动或停止容器。

在早期的版本中，Kubernetes原生支持了少量容器运行时，例如Docker。而在最近的版本中，Kubernetes将其迁移到了一个叫作容器运行时接口（CRI）的模块当中。从整体上来看，CRI屏蔽了Kubernetes内部运行机制，并向第三方容器运行时提供了文档化接口来接入。

Kubernetes目前支持丰富的容器运行时。一个非常著名的例子就是cri-containerd。这是一个开源的、社区维护的项目，将CNCF运行时通过CRI接口接入Kubernetes。该项目得到了广泛的支持，在很多Kubernetes场景中已经替代Docker成为当前最流行的容器运行时。

注意：containerd（发音如“container-dee”）是基于Docker Engine剥离出来的容器管理与运行逻辑。该项目由Docker公司捐献给CNCF，并获得了大量的社区支持。同期也存在其他的符合CRI标准的容器运行时。

#### 3.kube-proxy

关于工作节点的最后一个知识点就是kube-proxy。kube-proxy运行在集群中的每个工作节点，负责本地集群网络。例如，kube-proxy保证了每个工作节点都可以获取到唯一的IP地址，并且实现了本地IPTABLE以及IPVS来保障Pod间的网络路由与负载均衡。


## [2.3 Kubernetes DNS]

除种类丰富的控制平面与工作节点组件外，每个Kubernetes集群都有自己内部的DNS服务，这对于集群操作也是非常重要的。

集群DNS服务有一个静态IP地址，并且这个IP地址集群总每个Pod上都是硬编码的，这意味着每个容器以及Pod都能找到DNS服务。每个新服务都会自动注册到集群DNS服务上，这样所有集群中的组件都能根据名称找到相应的服务。一些其他的组件也会注册到集群DNS服务，例如Statefulset以及由Statefulset管理的独立Pod。

集群DNS是基于CoreDNS来实现的。



## 2.4 Kubernetes的应用打包
一个应用想要在Kubernetes上运行，需要完成如下几步。
1.将应用按容器方式进行打包。
2.将应用封装到Pod中。
3.通过声明式manifest文件部署。

打包的详细过程如下：用户需要选择某种语言来编写一个应用程序，完成之后需要将应用构建到容器镜像当中并存放于某个私有仓库（Registry）。<u>此时，已经完成了应用服务的容器化（containerized）。
</u>
接下来，用户需要定义一个可以运行容器化应用的Pod。抽象来说，Pod就是一层简单的封装，允许容器在Kubernetes集群上运行。在用户定义好Pod之后，就可以随时将其部署到集群上运行了。

在Kubernetes集群中可以运行一个Pod单例，但是通常情况下不建议这么做。推荐的方式是通过更上层的controller来完成。最常见的controller就是Deployment，它提供了可扩容、自愈与滚动升级等特性。读者只需要通过一个YAML文件就可以指定Deployment的配置信息，包括需要使用哪个镜像以及需要部署多少副本。

下图展示了应用如何被容器化（container），并以Pod方式运行，通过部署（Deployment） controller来进行管理。


![[Kubernetes in brief-2024817-3.png]]

在一个Deployment文件定义好之后，读者可以通过API Server来指定应用的期望状态（desired state）并在Kubernetes上运行。

## 2.5 声明式模型与期望状态 (declarative model and desired state)

声明式模型（declarative modle）以及期望状态（desired state）是Kubernetes中非常核心的概念。

在Kubernetes中，声明式模型的工作方式如下所述。
1.在manifest文件中声明一个应用（微服务）期望达到的状态。
2.将manifest文件发送到API Server。
3.Kubernetes将manifest存储到集群存储，并作为应用的期望状态。
4.Kubernetes在集群中实现上述期望状态。
5.Kubernetes启动监控循环，保证应用的当前状态（current state）不会与期望状态出现差异。

深入了解上述步骤：

manifest文件是按照简版YAML格式进行编写的，用户通过manifest文件来告知Kubernetes集群自己希望应用运行的样子。这就是所谓的期望状态。其中包括：需要使用哪个镜像、有多少副本需要运行、哪个网络端口需要监听，以及如何执行更新。

在用户创建了manifest文件之后，需要将其发送到API Server。最常见的方式是通过kubectl命令行工具来完成这个操作。kubectl会将manifest文件通过HTTP POST请求发送到控制平面，通常HTTP服务使用的端口是443。

在请求经过认证以及授权后，Kubernetes会检查manifest文件，确定需要将该文件发送到哪个控制器（例如Deployment controller），并将其保存到集群存储当中，作为整个集群的期望状态（desired state）中的一部分。在上述工作都完成之后，就要在集群中执行相应的调度工作了。调度工作包括拉取镜像、启动容器、构建网络以及启动应用进程。

最终，Kubernetes通过后台的reconciliation（调谐）循环来持续监控集群状态。如果集群当前状态（current state）与期望状态（desired state）存在差异，则Kubernetes会执行必要的任务来解决对应的差异点。

理解<u>声明式模式</u>与<u>传统的命令式模式</u>（imperative model）之间的区别是一件非常重要的事情。命令式模式是指用户需要指定一大串针对特定平台的命令来解决相应的问题。

但是关于声明式模式的介绍到此还未结束：当前系统可能出现变化或者出现某些问题。这意味着集群的当前状态（current state）与期望状态（desired state）不再一致。当这种情况发生时，Kubernetes会尝试采取相应措施来消除这种不一致的情况。

接下来通过一个例子来看一下具体情况。

声明式示例

假设读者有一个前端Web应用Pod，期望状态中需要运行10个副本。在其中某个工作节点运行的两个副本失败之后，当前状态（current state）就会减少到8个副本，但是期望状态（desired state）仍然是10个副本。这种情况会被调谐循环检测到，Kubernetes也会重新调度两个新的副本来保证当前状态副本数量重新恢复到10。

当读者尝试主动地调高或者调低副本数量的时候，上述流程也会被触发执行。读者甚至可以改变应用运行所需的镜像。比如，当前应用使用了v2.00版本的镜像，而读者希望将期望状态中的镜像版本调整到v2.01，Kubernetes会检测到这两者之间的差异，并会通过更新全部副本的镜像版本到期望状态（desired state）中的版本这种方式来实现该功能。

需要说明的一点是，与编写一长串命令来将每个副本更新到最新版本这种方式相比，读者现在只需要简单地告诉Kubernetes自己期望升级的新版本，剩下的大量工作由Kubernetes替读者来完成。

尽管一切看起来都非常简单，但这个功能确实很强大，并且是Kubernetes工作原理中很重要的一部分。读者在Kubernetes中提交了一个声明式的manifest文件，其中描述了读者希望应用所达到的运行状态。这个文件构成了应用期望状态的基础。Kubernetes控制平面记录下这个配置文件并会实现文件描述中的内容，同时还会在后台运行一个调谐循环来持续检查当前运行状态是否满足读者诉求。如果当前状态与期望状态相符，则皆大欢喜；但是如果存在差异，Kubernetes也会马上修复这些问题。

## [2.6 Pod]

在VMware的世界中，调度的原子单位是虚拟机（VM）；在Docker的世界中，调度的原子单位是容器；而在Kubernetes的世界中，调度的原子单位是Pod。

Kubernetes的确支持运行容器化应用。但是，用户无法直接在Kubernetes集群中运行一个容器，容器必须并且总是需要在Pod中才能运行。

### [2.6.1 Pod与容器]

首先需要介绍一下Pod这个术语的起源：在英语中，会将a group of whales（一群鲸鱼）称作a Pod of whales，Pod就是来源于此。因为Docker的Logo是鲸鱼，所以在Kubernetes中会将包含了一组容器的事物称作Pod。

一种最简单的方式就是，在每个Pod中只运行一个容器。当然，还有一种更高级的用法，在一个Pod中会运行一组容器。像这种多容器Pod（multi-container Pod）不在当前的讨论范围之内，不过可以先列举几种常见场景。
●服务网格（Service Mesh）。
●需要依赖helper容器来获取最新内容的Web容器。
●那些强依赖log清理工具的容器。
这其中最关键的地方即Pod是一种包含了一个或多个容器的结构。

### [2.6.2 Pod深度剖析]

整体来看，Pod是一个用于运行容器的有限制的环境。Pod本身并不会运行任何东西，只是作为一个承载容器的沙箱而存在。换一种说法，<u>Pod就是为用户在宿主机操作系统中划出有限的一部分特定区域，构建一个网络栈，创建一组内核命名空间，并且在其中运行一个或者多个容器，这就是Pod</u>。

如果在Pod中运行多个容器，那么多个容器间是共享相同的Pod环境的。共享环境中包括了IPC命名空间，共享的内存，共享的磁盘、网络以及其他资源等。举一个具体的例子，运行在相同Pod中的所有容器都有相同的IP地址（Pod的IP地址）。
![[Kubernetes in brief-2024817-4.png]]

如果在同一个Pod中运行的两个容器之间需要通信（在Pod内部的容器间），那么就可以使用Pod提供的localhost接口来完成。
对于存在强绑定关系的多个容器，比如需要共享内存与存储，多容器Pod就是一个非常完美的选择。但是，如果容器间并不存在如此紧密的关系，则更好的方式是将容器封装到不同的Pod，通过网络以松耦合的方式来运行。这样可以在任务级别实现容器间的隔离，降低相互之间的影响。当然这样也会<u>导致大量的未加密的网络流量产生</u>。读者在使用时，最好考虑通过服务网格来完成Pod与应用服务间的加密。

### [2.6.3 调度单元]

Kubernetes中最小的调度单元也是Pod。如果读者需要扩容或缩容自己的应用，可以通过添加或删除Pod来实现。

## 2.6.4 原子操作单位
Pod的部署是一个原子操作。这意味着，<u>只有当Pod中的所有容器都启动成功且处于运行状态时，Pod提供的服务才会被认为是可用的</u>。对于部分启动的Pod，绝对不会响应服务请求。整个Pod要么全部启动成功，并加入服务；要么被认为启动失败。

一个Pod只会被唯一的工作节点调度。这一点对于多容器Pod来说也是一样的，一个多容器Pod中的全部容器都会运行在相同的工作节点上。

## 2.6.5 Pod的生命周期
Pod的生命周期是有限的。Pod会被创建并运行，并且最终被销毁。如果Pod出现预期外的销毁，用户无须将其重新启动。因为Kubernetes会启动一个新的Pod来取代有问题的Pod。尽管新启动的Pod看起来跟原来的Pod完全一样，本质上却并不是同一个。这是一个有全新的ID与IP地址的Pod。

这种方式可能会对应用程序的设计思路产生影响。不要在设计程序的时候使其依赖特定的Pod，相反，需要使程序满足Pod特性，使其可以在某个Pod出现问题后，启动一个全新的Pod（有新的ID与IP地址）并无缝地取代原来Pod的位置

# 2.7 Deployment
大多数时间，用户通过更上层的控制器来完成Pod部署。上层的控制器包括Deployment、DaemonSet以及StatefulSet:
- Deployment是对Pod的更高一层的封装，除Pod之外，还提供了如扩缩容管理、不停机更新以及版本控制等其他特性。
- Deployment、DaemonSet以及StatefulSet还实现了自己的controller与监控循环，可以持续监控集群状态，并确保当前状态与期望状态相符。


# 2.8 服务与稳定的网络
Pod是非常重要的，并且可能会出现故障并销毁。如果通过Deployment或者DaemonSet对Pod进行管理，出现故障的Pod会自动被替换。但是替换后的新Pod会拥有完全不同的IP地址。这种情况也会在水平扩/缩容时发生，扩容时会创建拥有新IP地址的Pod，缩容时会移除掉现有的Pod。这些都会引起IP地址变化（IP churn）。

这其中的关键点是Pod是不可靠的，这时就出现一个新的挑战：假设用户需要通过一个由多个Pod构成的微服务来完成视频渲染工作。如果这个时候应用的其他部分需要依赖渲染服务，但是由于Pod不可靠，又不能直接依赖Pod服务，这时候该怎么办呢？

此时需要正式介绍Service（服务）机制。Service为一组Pod提供了可靠且稳定的网络。
![[Kubernetes in brief-2024825-1.png]]
上图展示了上传这个微服务并通过Kubernetes中的Service机制来与渲染微服务进行交互的过程。Kubernetes Service提供了一个稳定的服务名称与IP地址，并且对于其下的两个Pod提供了请求级别的负载均衡机制。

展开介绍一下其中的细节。<span style="background:#fff88f">Service在Kubernetes API中是一个成熟且稳定的组件，就像Pod与Deployment一样。</span>DNS名称、IP地址与端口共同组成了其前端。在其后端实现了对一组Pod间的动态负载均衡。同时，跟Pod一样，Service也实现了自我监控机制，可以自动更新，并持续提供稳定的网络终端。

如果对于Pod进行数量上的增加，则在Service中同样会生效。新的Pod会被无缝添加到Service并承担请求流量。已经终止的Pod会被平滑地从当前Service中移除，并不再处理请求流量。

这就是<span style="background:#fff88f">Service的职责：一个稳定的网络终端，提供了基组动态Pod集合的TCP以及UDP负载均衡能力。</span>

因为Service作用于TCP以及UDP层，所以Service层并不具备应用层的智能，即无法提供应用层的主机与路径路由能力。因此，用户需要一个入口来解析HTTP请求并提供基于主机与路径的路由能力。

## 将Pod连接到Service

Service使用标签（label）与一个标签选择器（label selector）来决定应当将流量负载均衡到哪一个Pod集合。Service中维护了一个标签选择器（label selector），其中包含了一个Pod所需要处理的全部请求对应的标签。基于此，Service层才能将流量路由到对应的Pod。

下图展示了一个特定的服务配置：需要将流量路由到集群中的所有标记了如下标签的Pod。
●zone=prod
●Env=be
●ver=1.3
![[Kubernetes in brief-2024825-2.png]]
当前图中所展示的Pod都具有这3种标签，Service会把流量均匀地分配到这些Pod之中。

下图展示了一个类似的场景。但是在图中右侧的增加了一个额外的Pod，这个Pod上面的标签与Service中标签选择器中的配置并不相符。这意味着服务不会将请求路由给这个新增的Pod。
![[Kubernetes in brief-2024825-3.png]]
Service只会将流量路由到健康的Pod，这意味着如果Pod的健康检查失败，那么Pod就不会接收到任何流量。
Service为不稳定的Pod集合提供了稳定的IP地址以及DNS名称。

## [2.9 总结](javascript:void(0))



- **控制平面(control plane)** 运行在主节点上。在其背后，还有几个系统服务，包括对集群提供了RESTful接口的API Server。
- **主节点(master)** 负责所有的部署与调度决策，并且对于生产环境来说，基于多主节点的HA方案是非常重要的。
- **工作节点(node)** 上运行了用户的应用程序。每个工作节点上都运行了Kubelet这个服务，负责将当前工作节点注册到集群，并且通过API Server与集群进行交互。Kubelet通过监听API的方式来获取新任务，并维护相应的上报通道。工作节点上还有一个容器运行时，以及kube-proxy服务。容器运行时，比如Docker或者containerd，负责响应全部与容器相关的操作。kube-proxy负责节点上的网络功能。
- **Pod**是执行构建的最小单元。
- **Deployment**在Pod基础上增加了自愈、水平扩缩容与更新能力。
- **Service**提供了稳定的网络与负载均能能力。






# Setting up a Kubernetes cluster

## INSTALLING MINIKUBE

```
brew install minikube
brew install kubectrl

minikube start

```

To verify your cluster is working, you can use the **kubectl cluster-info** command
shown in the following listing.
```
$ kubectl cluster-info
```
