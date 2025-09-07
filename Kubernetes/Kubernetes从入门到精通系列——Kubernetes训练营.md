# Kubernetes从入门到精通系列——Kubernetes训练营
上一章介绍了如何使用KinD（Kubernetes in Docker）部署Kubernetes集群，这种方法通过在单个机器上使用容器而非虚拟机来创建开发集群，减少了系统资源需求并简化了整个设置过程。我们讨论了KinD的安装与配置、创建集群以及如何添加组件，例如Ingress控制器、Calico作为CNI（容器网络接口）和使用持久化存储。

我们理解，很多读者已经有Kubernetes的相关经验，无论是在生产环境中运行集群，还是通过工具如kubeadm、minikube或Docker Desktop进行实验。因此，本书的目标是深入探讨Kubernetes的高级概念，而不是重复基础知识。因此，我们将本章设计为一个训练营，专为那些对Kubernetes接触较少或者刚开始学习的人而准备。

在本章中，我们将深入探索Kubernetes集群的基本组件，包括控制平面和工作节点。我们还将详细介绍每种Kubernetes资源及其应用场景。如果你已经对Kubernetes有一定的了解，并且熟悉如何使用kubectl命令以及理解Kubernetes的资源，如DaemonSets、StatefulSets和ReplicaSets，那么本章可以作为你继续阅读第4章的复习内容。在第4章中，我们将详细讨论服务（Services）、负载均衡（Load Balancing）、ExternalDNS、全局负载均衡（Global Balancing）和K8GB。

由于这是一个训练营性质的章节，因此我们不会深入探讨每个主题。不过，通过本章的学习，你应该能够掌握Kubernetes的基础概念，这对于理解后续章节至关重要。即使你已经具备丰富的Kubernetes背景知识，本章也可以作为进阶学习前的一个复习。在本章中，你将学习以下内容：

*   Kubernetes组件概述
*   探索控制平面
*   了解工作节点组件
*   交互式使用API服务器
*   介绍Kubernetes资源

在本章结束时，你将对最常用的集群资源有一个扎实的理解。了解Kubernetes资源对于集群运维人员和集群管理员来说都是非常重要的。

技术要求
----

本章没有技术要求。

如果你希望在学习资源的过程中执行命令，可以使用上一章部署的KinD集群。

Kubernetes 组件概述
---------------

理解基础设施中系统组件的组成对于有效提供服务至关重要。在当今广泛的安装选项中，许多 Kubernetes 用户可能不需要全面了解不同 Kubernetes 组件的集成方式。

几年前，建立 Kubernetes 集群需要手动安装和配置每个组件。这一过程学习曲线陡峭，常常让人感到沮丧。因此，许多个人和组织得出结论：“Kubernetes 过于复杂。” 然而，手动安装的好处在于，它可以深入了解各组件之间的交互方式。如果安装后集群出现任何问题，你将清楚地知道需要调查哪个部分。

要理解 Kubernetes 组件如何协同工作，首先需要了解 Kubernetes 集群的不同组件。

下图来自 [Kubernetes.io](https://link.juejin.cn/?target=http%3A%2F%2FKubernetes.io "http://Kubernetes.io") 网站，展示了 Kubernetes 集群组件的高级概述：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/052358e2e2894b19a42e9e2c005f5b99~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728558197&x-signature=OSrfNIi6HUEbzHNzXjYtLl0Keo8%3D)

正如你所见，Kubernetes 集群由多个组件组成。随着本章的进展，我们将讨论这些组件及其在 Kubernetes 集群中的作用。

探索控制平面
------

顾名思义，控制平面对集群的每个方面都具有控制权。没有正常运行的控制平面，集群将失去调度工作负载、创建新部署以及管理 Kubernetes 对象的能力。鉴于控制平面的关键性，强烈建议部署具备高可用性 (HA) 支持的集群，至少部署三个控制平面节点。许多生产环境甚至使用了超过三个控制平面节点，但关键原则是使用奇数节点，以便在丢失一个 etcd 节点时仍然能够保持高可用的控制平面。

现在，让我们深入探讨控制平面的重要性及其组件，全面理解它们在集群中发挥的关键作用。

### Kubernetes API 服务器

集群中的第一个重要组件是 `kube-apiserver`。由于 Kubernetes 是 API 驱动的，因此集群中的每个请求都必须经过 API 服务器。让我们通过使用 API 端点的一个简单例子来了解请求，例如使用控制平面的 IP 地址。在企业环境中，这通常由负载均衡器处理。在我们的例子中，负载均衡器为三台控制平面节点提供了 10.240.100.100 的入口：

```bash
https://10.240.100.100:6443/api/v1/nodes?limit=500

```

如果没有任何凭证尝试发出 API 调用，你将收到权限拒绝的请求。直接使用纯 API 请求在应用部署或 Kubernetes 附加组件的管道中很常见。然而，用户与 Kubernetes 交互最常用的方法是通过 `kubectl` 工具。

每个通过 `kubectl` 发出的命令都在幕后调用了 API 端点。在上面的例子中，执行 `kubectl get nodes` 命令时，会向 `kube-apiserver` 发送一个 API 请求，使用地址 `10.240.100.100` 和端口 `6443`。

该 API 请求调用了 `/api/v1/nodes` 端点，并返回了集群中的节点列表：

```arduino
NAME                     STATUS    ROLES                  AGE     VERSION
home-k8s-control-plane   Ready     control-plane,master   45d     v1.27.3
home-k8s-control-plane2  Ready     control-plane,master   45d     v1.27.3
home-k8s-control-plane3  Ready     control-plane,master   45d     v1.27.3
home-k8s-worker          Ready     worker                 45d     v1.27.3
home-k8s-worker2         Ready     worker                 45d     v1.27.3
home-k8s-worker3         Ready     worker                 45d     v1.27.3

```

如果 API 服务器无法正常工作，所有发送到集群的请求都会失败。因此，确保 `kube-apiserver` 的持续运行和健康状况至关重要。

通过运行三个或更多的控制平面节点，可以最大限度地减少因失去一个控制平面节点而造成的潜在影响。记住，在运行多个控制平面节点时，需要在集群 API 服务器前配置一个负载均衡器。Kubernetes API 服务器可以使用大多数标准解决方案进行负载均衡，包括 F5、HAProxy 和 Seesaw。

### etcd 数据库

将 `etcd` 描述为 Kubernetes 集群的基础毫不为过。`etcd` 是一个强大且高效的分布式键值数据库，Kubernetes 依赖它来存储所有集群数据。集群中的每个资源都与 `etcd` 数据库中的某个特定键相关联。如果你可以访问托管 `etcd` 的节点或 Pod，就可以使用 `etcdctl` 可执行文件来浏览数据库中存储的所有键。下面的代码片段展示了从基于 KinD 的集群中提取的一个示例：

```ruby
etcdctl-3.5.12 --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --key=/etc/kubernetes/pki/etcd/server.key --cert=/etc/kubernetes/pki/etcd/server.crt get / --prefix --keys-only

```

上述命令的输出数据量很大，无法在本章列出所有条目。一个基本的 KinD 集群将返回大约 314 个条目。

所有键都以 `/registry/<resource>` 开头。例如，返回的键之一是集群管理员 `ClusterRole` 的键，如下所示：`/registry/clusterrolebindings/cluster-admin`。

我们可以使用 `etcdctl` 实用程序稍微修改前面的命令，通过键名来检索值，如下所示：

```ruby
etcdctl-3.5.12 --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --key=/etc/kubernetes/pki/etcd/server.key --cert=/etc/kubernetes/pki/etcd/server.crt get /registry/clusterrolebindings/cluster-admin

```

输出将包含无法被你的 shell 解释的字符，但你会了解 `etcd` 中存储的数据。对于 `cluster-admin` 键，输出如下：

```bash
/registry/clusterrolebindings/cluster-admin
k8s
2
rbac.authorization.k8s.io/v1ClusterRoleBinding▒
▒
cluster-admin"*$7b235e81-52de-4001-b354-994dad0279ee2▒▒▒▒Z,
rbac-defaultsb3otstrapping
+rbac.authorization.kubernetes.io/autoupdatetrue▒▒
kube-apiserverUpdaterbac.authorization.k8s.io/v▒▒▒▒FieldsV1:▒
▒{"f:metadata":{"f:annotations":{".":{},"f:rbac.authorization.kubernetes.io/autoupdate":{}},"f:labels":{".":{},"f:kubernetes.io/bootstrapping":{}}},"f:roleRef":{},"f:subjects":{}}B4
Grouprbac.authorization.k8s.iosystem:masters"7
rbac.authorization.k8s.io
cluster-admin"           ClusterRole


```

我们描述了 `etcd` 中的条目，目的是帮助你理解 Kubernetes 如何存储和利用数据来管理集群对象。虽然你已经看到了 `cluster-admin` 键的直接数据库输出，但在典型场景下，你会使用命令 `kubectl get clusterrolebinding cluster-admin -o yaml` 来通过 API 服务器查询相同的数据。使用 `kubectl`，该命令将返回如下信息：

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/9459fc594de14c05995791c45acca42e~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728558197&x-signature=LG5mXzAgelq%2BOQ7LN0exWXk0jf8%3D)

如果你查看 `kubectl` 命令的输出并将其与 `etcdctl` 查询的输出进行比较，你会发现它们的信息是一致的。你很少需要直接与 `etcd` 交互；相反，你通常会执行 `kubectl` 命令，请求会发送到 API 服务器，然后 API 服务器查询 `etcd` 数据库以获取资源信息。

值得注意的是，虽然 `etcd` 是 Kubernetes 最常用的后端数据库，但它并不是唯一的选择。例如，最初为边缘场景开发的轻量级 Kubernetes 版本 k3s 使用了关系型数据库替代 `etcd`。当我们深入探讨 vclusters（虚拟集群）时，你会发现它使用 SQLite 代替 `etcd`。

### kube-scheduler（调度器）

顾名思义，`kube-scheduler` 负责将 Pod 分配给节点。它的主要任务是持续监控那些尚未分配到任何特定节点的 Pod。调度器通过评估每个 Pod 的资源需求来确定最合适的节点分配位置。这一评估考虑了多个因素，包括节点资源的可用性、约束条件、选择器以及亲和性/反亲和性规则。满足这些条件的节点会被视为可用节点。最终，调度器会从兼容节点列表中选择最合适的节点来调度 Pod。

### kube-controller-manager（控制器管理器）

Kubernetes 控制器管理器是 Kubernetes 控制平面的核心控制系统，负责管理和协调多个控制器，这些控制器处理集群中的特定任务。

控制器管理器包含多个控制器，每个控制器负责集群中的特定功能。这些控制器持续监控集群的当前状态，并动态适应以保持期望的配置。

所有控制器都包含在一个可执行文件中，从而简化了复杂性和管理。表 3.1 列出了一些控制器。

| 控制器 | 职责 |
| --- | --- |
| Endpoints | 监控新服务并为具有匹配标签的 Pod 创建端点 |
| Namespace | 监控命名空间的操作 |
| Node | 监控集群中节点的状态，检测节点故障或新增，并采取适当行动以维持期望的节点数量 |
| Replication | 监控 Pod 的副本数量，移除或添加 Pod 以达到期望的状态 |
| Service Accounts | 监控服务账户 |

表 3.1：控制器及其功能

每个控制器运行一个不终止的（永不结束的）控制循环。这些控制循环会监控每个资源的状态，并根据需要进行更改以使资源状态恢复正常。例如，如果你需要将一个部署从一个 Pod 扩展到三个 Pod，复制控制器会发现当前状态只有一个 Pod 正在运行，而期望状态是三个 Pod。因此，复制控制器会请求启动另外两个 Pod 以达到期望状态。

### cloud-controller-manager（云控制器管理器）

这是一个你可能没有遇到过的组件，这取决于集群的配置方式。类似于 `kube-controller-manager`，这个控制器包含四个控制器在一个二进制文件中。

云控制器为特定云提供商的 Kubernetes 服务提供集成，允许利用云特有的功能，如负载均衡器、持久存储、自动缩放组等功能。

理解工作节点组件
--------

工作节点，顾名思义，在 Kubernetes 集群中负责执行任务。在前面讨论控制平面的 `kube-scheduler` 元素时，我们提到，当有新 pod 需要调度时，`kube-scheduler` 会选择合适的节点执行该 pod。`kube-scheduler` 依赖于工作节点提供的数据来做出决策。这些数据会定期更新，以确保 pod 在集群中的分布能够最大化利用集群资源。

每个工作节点都有两个主要组件：`kubelet` 和 `kube-proxy`。

### kubelet

你可能听说过工作节点有时被称为 `kubelet` 节点。`kubelet` 是运行在所有工作节点上的代理，负责确保节点上的容器处于运行状态并保持健康。

### kube-proxy

与名字相反，`kube-proxy` 并不是一个代理服务器（尽管它在 Kubernetes 的早期版本中是这样的）。

根据你集群中部署的 CNI（容器网络接口），节点上可能会有也可能不会有 `kube-proxy` 组件。像 Cilium 这样的 CNI 可以与 `kube-proxy` 一起运行，也可以以无 `kube-proxy` 的模式运行。在我们的 KinD 集群中，我们部署了 Calico，它依赖于 `kube-proxy` 的存在。

当 `kube-proxy` 被部署时，它的主要作用是管理集群中 pod 和服务的网络连接，负责将网络流量路由到目标 pod。

### 容器运行时

每个节点还需要一个容器运行时。容器运行时负责运行容器。你可能首先想到的是 Docker，虽然 Docker 是一种容器运行时，但它并不是唯一的选择。在过去几年里，其他选项逐渐取代了 Docker，成为集群中更受欢迎的容器运行时。

目前，取代 Docker 的两大容器运行时是 CRI-O 和 containerd。在编写本章时，KinD 仅官方支持 Docker 和 Red Hat 的 Podman。

与 API 服务器交互
-----------

如前所述，您可以通过直接的 API 请求或 `kubectl` 工具与 API 服务器进行交互。在本书的大部分操作中，我们将重点介绍如何使用 `kubectl`，但在适用的情况下，我们也会指出如何使用直接 API 调用。

### 使用 Kubernetes 的 kubectl 工具

`kubectl` 是一个允许您通过命令行接口 (CLI) 与 Kubernetes API 进行交互的单一可执行文件。它适用于大多数主流操作系统和架构，包括 Linux、Windows 和 macOS。

**注意**: 我们在第 2 章创建集群时已经使用 KinD 脚本安装了 `kubectl`。大多数操作系统的安装说明可在 Kubernetes 网站上找到：[kubernetes.io/docs/tasks/…](https://link.juejin.cn/?target=https%3A%2F%2Fkubernetes.io%2Fdocs%2Ftasks%2Ftools%2Finstall-kubectl%2F "https://kubernetes.io/docs/tasks/tools/install-kubectl/")。由于我们在本书的练习中使用Linux 作为操作系统，因此我们将介绍如何在 Linux 机器上安装 `kubectl`。按照以下步骤操作：

要下载 `kubectl` 的最新版本，您可以运行以下 curl 命令：

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

```

下载后，您需要运行以下命令使文件可执行：

```bash
chmod +x ./kubectl

```

最后，我们将可执行文件移动到系统路径中：

```bash
sudo mv ./kubectl /usr/local/bin/kubectl

```

现在，您的系统上已经安装了最新的 `kubectl` 工具，并可以从任何工作目录执行 `kubectl` 命令。

Kubernetes 每 4 个月左右更新一次。这包括 Kubernetes 集群基本组件和 `kubectl` 工具的升级。您可能会遇到集群和 `kubectl` 命令版本不匹配的情况，要求您升级或下载新的 `kubectl` 可执行文件。您可以随时通过运行 `kubectl version` 命令检查 API 服务器和 `kubectl` 客户端的版本。以下代码片段展示了版本检查的输出 —— 请注意，您的输出可能与示例不同：

```yaml
Client Version: v1.30.0-6+43a0480e94cee1
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.30.0-6+43a0480e94cee1

```

如上所示，`kubectl` 客户端运行的是 1.30.0 版本，而集群也运行的是 1.30.0 版本。两者之间的一个小版本差异不会造成任何问题。实际上，官方支持的版本差异是在一个主要版本内。因此，如果您的客户端运行的是 1.29 版本，而集群运行的是 1.30.0 版本，您依然在支持的版本范围内。虽然这是支持的，但如果您尝试使用更高版本中包含的新命令或资源，可能会遇到问题。一般来说，您应尽量保持集群和客户端版本同步，以避免任何问题。

在本章的余下部分，我们将讨论 Kubernetes 资源以及如何与 API 服务器交互来管理每个资源。但在深入讨论不同的资源之前，我们想提到一个常被忽视的 `kubectl` 选项：**详细模式（verbose option）** 。

### 了解详细模式

当您执行 `kubectl` 命令时，默认情况下您只会看到与命令直接相关的输出。如果您查看 `kube-system` 命名空间中的所有 pod，您将收到所有 pod 的列表。在大多数情况下，这是您想要的输出。但是，如果您发出了 `get Pods` 请求并从 API 服务器收到错误，您如何获取更多关于可能导致错误的信息呢？

通过在 `kubectl` 命令中添加详细模式，您可以获得有关 API 调用本身及 API 服务器回复的更多详细信息。API 服务器的回复通常包含有助于找到问题根本原因的额外信息。

详细模式有多个级别，范围从 0 到 9，数字越大，您将收到的输出信息越多。

以下截图取自 Kubernetes 网站，详细说明了每个级别及其包含的输出信息:

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/777d2ee80f0f47c08acc74309caf8374~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728558197&x-signature=iYWRRd96%2BNJI5McFOOHe3kZNGTU%3D)

您可以通过在任何 `kubectl` 命令中添加 `-v` 或 `--v` 选项来尝试不同的详细级别。

### 常用的 `kubectl` 命令

命令行接口（CLI）允许您以命令式（imperative）和声明式（declarative）的方式与 Kubernetes 进行交互。使用命令式命令意味着您直接告诉 Kubernetes 应该做什么，例如：`kubectl run nginx --image nginx`。这条命令告诉 API 服务器创建一个名为 `nginx` 的新 pod，并运行名为 `nginx` 的镜像。虽然命令式命令在开发、快速修复或测试时很有用，但在生产环境中，您更常使用声明式命令。

在声明式命令中，您告诉 Kubernetes 您想要的结果。使用声明式命令时，您需要向 API 服务器发送一个 JSON 或 YAML 格式的清单（manifest），声明您希望 Kubernetes 创建的内容。

`kubectl` 包含可以提供集群总体信息或资源信息的命令和选项。下表是常用命令的速查表，列出了每个命令的用途。我们将在后续章节中使用许多这些命令，因此在本书中您将看到它们的实际应用：

| **集群命令** | **功能** |
| --- | --- |
| `api-resources` | 列出支持的 API 资源 |
| `api-versions` | 列出支持的 API 版本 |
| `cluster-info` | 列出集群信息，包括 API 服务器和其他集群端点 |
| **对象命令** | **功能** |
| --- | --- |
| `get <object>` | 获取所有对象的列表（例如 pods、ingress 等） |
| `describe <object>` | 提供对象的详细信息 |
| `logs <pod name>` | 获取某个 pod 的日志 |
| `edit <object>` | 交互式编辑对象 |
| `delete <object>` | 删除对象 |
| `label <object>` | 给对象打标签 |
| `annotate <object>` | 给对象添加注解 |
| `run` | 创建一个 pod |

**表 3.2：集群和对象命令**

通过理解每个 Kubernetes 组件以及如何使用命令式命令与 API 服务器交互，现在我们可以继续了解 Kubernetes 资源及如何使用 `kubectl` 来管理它们。

介绍 Kubernetes 资源
----------------

在本节中，我们将提供大量关于 Kubernetes 资源的信息。然而，由于这是一个速成课程，我们不会深入探讨每个资源的细节。需要注意的是，每个资源都可以单独构成一个专门的章节，甚至在书中展开多个章节。因为许多关于 Kubernetes 基础的书籍已经详尽地覆盖了这些资源，我们将专注于基本理解所必需的要点。随着后续章节的推进，我们将在实际练习中补充关于这些资源的更多细节。

在深入理解 Kubernetes 资源之前，首先介绍 Kubernetes manifests 的概念。

### Kubernetes Manifests

我们用于创建 Kubernetes 资源的文件称为 manifests（清单）。清单可以使用 YAML 或 JSON 格式创建——大多数清单使用 YAML 格式，本文全书将使用 YAML 格式。

需要注意的是，尽管我们使用的是 YAML 文件，`kubectl` 会在与 API 服务器交互时将所有 YAML 转换为 JSON。所有 API 调用都会以 JSON 形式进行，即使清单是以 YAML 编写的。

清单的内容会根据要创建的资源而变化。所有清单至少需要包含基本配置，包括 `apiVersion`（API 版本）、`kind`（资源类型）和 `metadata`（元数据）字段，如下所示：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana
  name: grafana
  namespace: monitoring

```

上面的清单并不完整，它只是一个完整 Deployment 清单的开始部分。如您所见，文件中有三项必需字段：`apiVersion`、`kind` 和 `metadata`。

您还会注意到文件中的字段格式。YAML 对格式要求非常严格，如果某一行的格式甚至出现一个空格的错误，您在尝试部署清单时会收到错误。这需要时间来适应，即使在编写清单很长一段时间后，格式问题仍可能偶尔出现。

### 什么是 Kubernetes 资源？

当你想在集群中添加或删除某些内容时，实际上是在与 Kubernetes 资源进行交互。通过这种交互，你声明了资源的期望状态，比如创建、删除或扩展资源。API 服务器会根据声明的期望状态，确保当前状态与期望状态一致。例如，如果你有一个启动时只有一个副本的部署，可以将部署资源从 1 个副本修改为 3 个副本。当 API 服务器发现当前状态为 1 时，它会创建另外两个 pod，将部署扩展到 3 个副本。

要获取集群支持的资源列表，可以使用 `kubectl api-resources` 命令。API 服务器会返回所有资源的列表，包括有效的简称、命名空间支持和支持的 API 组。

Kubernetes 集群中大约包含 58 个基本资源，但在生产集群中，资源数量通常会多于 58 个。许多附加组件（例如 Calico）会扩展 Kubernetes API 并引入新的对象。当集群中部署了不同的附加组件时，不要惊讶于资源列表中会出现 100 多个资源。

下表简要列出了 Kubernetes 集群中最常见的一些资源：

| **名称** | **简称** | **API 版本** | **是否命名空间化** |
| --- | --- | --- | --- |
| apiservices |  | apiregistration.k8s.io/v1 | FALSE |
| certificatesigningrequests | csr | certificates.k8s.io/v1 | FALSE |
| clusterrolebindings |  | rbac.authorization.k8s.io/v1 | FALSE |
| clusterroles |  | rbac.authorization.k8s.io/v1 | FALSE |
| configmaps | cm | v1 | TRUE |
| daemonsets | ds | apps/v1 | TRUE |
| deployments | deploy | apps/v1 | TRUE |
| persistentvolumeclaims | pvc | v1 | TRUE |
| services | svc | v1 | TRUE |
| pods | po | v1 | TRUE |
| replicasets | rs | apps/v1 | TRUE |
| rolebindings |  | rbac.authorization.k8s.io/v1 | TRUE |
| statefulsets | sts | apps/v1 | TRUE |
| secrets |  | v1 | TRUE |
| storageclasses | sc | storage.k8s.io/v1 | FALSE |

**表 3.3: Kubernetes API 资源**

由于本章是速成课程，我们将简要介绍表 3.3 中的资源。为了有效理解后续章节，掌握这些对象及其功能是至关重要的。

某些资源也将在后续章节中进行更详细的解释，例如 Ingress、RoleBindings、ClusterRoles、StorageClasses 等。

### 审查 Kubernetes 资源

在 Kubernetes 集群中，大多数资源都运行在命名空间内。如果你想创建、编辑或读取它们，应在 `kubectl` 命令中使用 `-n <namespace>` 选项。要查找支持命名空间选项的资源列表，你可以参考表 3.3 的输出。如果某个资源可以通过命名空间引用，则 NAMESPACED 列会显示 TRUE；如果资源只能在集群级别引用，则 NAMESPACED 列会显示 FALSE。

#### ApiServices

ApiServices 是 Kubernetes 组件与所有外部资源（如用户、应用程序和其他服务）之间通信和交互的主要入口点。它们提供了一组端点，允许用户和应用程序执行各种操作，例如创建、更新和删除 Kubernetes 资源（如 pods、deployments、services 和 namespaces）。

ApiServices 处理认证、授权和请求验证，确保只有授权的用户和应用程序才能访问或修改资源。它们还处理资源版本控制和集群中的其他关键方面。

ApiServices 还可以通过开发自定义控制器、操作器或与 API Services 交互的其他组件来扩展 Kubernetes 的功能，以管理和自动化集群行为的各个方面。比如，我们的 CNI Calico 就向集群中添加了 31 个额外的 API 资源。

#### CertificateSigningRequests

CertificateSigningRequest (CSR) 允许你从证书颁发机构请求证书。它们通常用于获取可信任的证书，以确保 Kubernetes 集群内的通信安全。

#### ClusterRoles

ClusterRole 是一组权限，用于与集群的 API 交互。它将操作（动词）与 API 组配对，以定义特定的权限。例如，如果你想限制 CI/CD 流水线仅能够为更新镜像标签而修改 Deployments，可以使用如下的 ClusterRole：

```makefile
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: patch-deployment
  rules:
  - apiGroups: ["apps/v1"]
    resources: ["deployments"]
    verbs: ["get", "list", "patch"]

```

ClusterRole 可以应用于集群级别和命名空间级别的 API。

#### ClusterRoleBindings

一旦定义了 ClusterRole，接下来的步骤是通过 ClusterRoleBinding 将 ClusterRole 与某个主体（如用户、组或服务账户）关联，从而授予其 ClusterRole 中定义的权限。

我们将在第 7 章《RBAC 策略和审计》中详细探讨 ClusterRoleBinding。

#### ComponentStatus

Kubernetes 控制平面是集群的重要组成部分，它对集群的正常运行至关重要。ComponentStatus 是一个对象，用于显示不同 Kubernetes 控制平面组件的健康状态。它提供组件的总体健康状况指示，显示其是否正常运行或出现错误。

#### ConfigMaps

ConfigMap 是一种资源，用于以键值对的形式存储数据，使配置与应用程序分离。ConfigMap 可以包含各种类型的数据，包括文字值、文件或目录，从而为管理应用程序配置提供了灵活性。

以下是一个命令式示例：

```xml
kubectl create configmap <name> <data>

```

其中 `<data>` 选项会根据 ConfigMap 的来源而变化。如果使用文件或目录，可以使用 `--from-file` 选项，并提供文件或整个目录的路径，如下所示：

```javascript
kubectl create configmap config-test --from-file=/apps/nginx-config/nginx.conf

```

这将创建一个名为 `config-test` 的 ConfigMap，nginx.conf 文件的内容将作为键的值存储。

如果你需要在单个 ConfigMap 中添加多个键，可以将每个文件放入一个目录中，并使用目录中的所有文件创建 ConfigMap。例如，假设你在 `~/config/myapp` 目录中有三个文件，分别是 config1、config2 和 config3。要创建一个包含这些文件的 ConfigMap，可以使用以下命令：

```javascript
kubectl create configmap config-test --from-file=/apps~/config/myapp

```

这样会创建一个包含三个键的 ConfigMap，分别是 config1、config2 和 config3。每个键的值对应于该目录中相应文件的内容。

要快速查看 ConfigMap，可以使用 `kubectl get configmaps config-test` 命令，结果如下：

```arduino
NAME             DATA    AGE
config-test      3       7s

```

ConfigMap 由三个键组成，DATA 列中的数字 3 表示键的数量。要详细查看 ConfigMap，可以使用 `-o yaml` 选项：

```arduino
kubectl get configmaps config-test -o yaml

```

结果如下：

```yaml
apiVersion: v1
data:
  config1: |
    First Configmap Value
  config2: |
    Yet Another Value from File2
  config3: |
    The last file - Config 3
kind: ConfigMap
metadata:
  creationTimestamp: "2023-06-10T01:38:51Z"
  name: config-test
  namespace: default
  resourceVersion: "6712"
  uid: a744d772-3845-4631-930c-e5661d476717

```

正如你所见，ConfigMap 中的每个键对应于目录中的文件名，键的值来自相应文件中的数据。

需要注意的是，ConfigMap 中的数据对任何有权限访问资源的人都是可见的。因此，不要在 ConfigMap 中存储敏感信息（如密码）。后面我们将介绍用于存储敏感数据的 Secret 资源。

#### ControllerRevisions

ControllerRevision 类似于控制器设置的某个版本或更新的快照，主要用于特定控制器（如 StatefulSet 控制器）来跟踪和管理配置随时间的更改。每当由控制器管理的资源配置发生修改或更新时，就会创建一个新的 ControllerRevision。每个修订版本都包含控制器的所需配置和一个修订号。这些修订存储在 Kubernetes API 服务器中，允许你随时引用或恢复到以前的版本。

#### CronJobs

如果你曾使用过 Linux 的 cronjobs，那么你已经知道 Kubernetes 的 CronJob 资源是什么。如果没有 Linux 背景，可以将 CronJob 看作是用于创建计划任务的工具，类似于 Windows 中的计划任务。下面的代码片段展示了一个创建 CronJob 的示例：

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-world
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello-world
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello World!
  restartPolicy: OnFailure

```

调度格式遵循标准的 cron 格式。从左到右，分别表示：

*   分钟（0–59）
*   小时（0–23）
*   日（1–31）
*   月（1–12）
*   星期几（0–6，星期日为 0，星期六为 6）

CronJob 支持步长值，可以用来创建每分钟、每2分钟或每小时执行的计划任务。上面的示例每分钟运行一次镜像 `hello-world`，并在 Pod 日志中输出 “Hello World!”。

#### CSI 驱动程序

Kubernetes 使用 CsiDriver 资源将节点连接到存储系统。你可以通过执行 `kubectl get csidriver` 列出集群中可用的所有 CSI 驱动程序。

#### CSI 节点

CSINode 资源用于存储 CSI 驱动程序生成的信息，包括 Kubernetes 节点名称到 CSI 节点名称的映射、CSI 驱动程序的可用性及卷拓扑等信息。

#### CSI 存储容量

CSIStorageCapacity 组件存储给定 CSI 的存储容量信息，代表给定 StorageClass 的可用存储容量。此信息在 Kubernetes 决定在哪里创建新的 PersistentVolumes 时使用。

#### 自定义资源定义 (CRD)

CustomResourceDefinition (CRD) 允许用户在 Kubernetes 集群中创建自定义资源。CRD 描述了自定义资源的结构、格式和行为，包括它的 API 端点和支持的操作。创建并添加到集群后，它就成为一个内置的资源类型，能够使用常规的 Kubernetes 工具和 API 进行管理。

#### DaemonSets

DaemonSet 用于在集群中的每个节点或特定节点集上部署 Pod，通常用于在集群的每个节点上部署日志记录等必需组件。DaemonSet 一旦设置，就会自动在每个现有节点上创建一个 Pod，并在新节点加入集群时，确保 Pod 也会部署在这些新节点上。

#### 部署 (Deployments)

我们之前提到过，不应该直接部署 Pod。一个重要原因是直接创建的 Pod 无法扩展或滚动升级。部署 (Deployment) 提供了许多优势，包括以声明式方式管理升级，并能够回滚到以前的版本。创建 Deployment 实际上是由 API 服务器执行的三步过程：创建 Deployment，Deployment 创建 ReplicaSet，ReplicaSet 然后创建应用程序的 Pod。

即使你不打算扩展或进行滚动升级，也应该默认使用 Deployment 以便将来可以利用这些功能。

#### 端点 (Endpoints)

Endpoints 将服务映射到一个或多个 Pod。稍后我们在解释服务资源时，这会更加清晰。你可以使用 CLI 通过 `kubectl get endpoints` 检索端点。 在一个新的 KinD 集群中，你会看到一个 Kubernetes API 服务器在默认命名空间中的值：

```vbnet
NAMESPACE   NAME         ENDPOINTS         AGE
default     Kubernetes   172.17.0.2:6443   22h

```

输出显示集群中有一个名为 `kubernetes` 的服务，其端点位于 IP 地址 `172.17.0.2` 的 `6443` 端口。

#### EndPointSlices

EndPoints 不擅长扩展——它们将所有端点存储在单个资源中。对于较小的部署，这不是问题，但随着集群和应用程序的扩展，端点大小也会增加，从而影响控制平面的性能，并在端点更改时产生额外的网络流量。

EndPointSlices 为需要可扩展性和精确控制网络端点的大型场景而设计。默认情况下，每个 EndPointSlice 可以容纳多达 100 个端点， 你可以通过 `--max-endpoints-per-slice` 选项增加这一数量。

#### 事件 (Events)

Events 资源将显示某个命名空间的所有事件。你可以使用 `kubectl get events -n kube-system` 查看 `kube-system` 命名空间的事件列表。

#### HorizontalPodAutoscalers

Horizontal Pod Autoscalers (HPAs) 使应用程序能够基于设定的指标自动扩展。

#### MutatingWebhookConfiguration

**MutatingWebhookConfiguration** 用于创建能够拦截并修改发送到 API 服务器请求的 webhook。这为自动修改请求的资源提供了一种方式。  
MutatingWebhookConfiguration 包含一组规则，确定哪些请求应该被 webhook 拦截和处理。当请求匹配定义的规则时，MutatingWebhookConfiguration 触发相应的 webhook，该 webhook 可以修改请求的负载，修改内容可以包括添加、删除或修改资源中的字段和注解。

#### Namespaces

**Namespace** 是一种将集群划分为逻辑单元的资源。每个 Namespace 都可以对资源进行细粒度管理，包括权限、配额和报告。  
Namespace 资源用于执行集群级操作，比如创建、删除、编辑和获取命名空间信息。你可以使用 `kubectl <verb> ns <namespace name>` 执行命令。  
例如，使用 `kubectl describe namespaces kube-system` 可以查看 `kube-system` 命名空间的信息，包括标签、注解和分配的配额。

#### NetworkPolicies

**NetworkPolicy** 资源让你定义集群中网络流量（包括入站和出站）如何流动。通过它可以定义哪些 Pods 可以与其他 Pods 通信。  
例如，以下策略允许 `myns` 命名空间中所有 Pods 从带有 `app.kubernetes.io/name: ingress-nginx` 标签的任何命名空间接收端口 443 上的流量：

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-ingress
  namespace: myns
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
    ports:
    - protocol: TCP
      port: 443

```

#### Nodes

**Nodes** 资源用于与集群的节点进行交互，可执行 `get`、`describe`、`label` 和 `annotate` 等操作。  
例如，使用 `kubectl get nodes` 可以获取集群中所有节点的列表。

#### PersistentVolumeClaims (PVCs)

**PVC** 是一个命名空间级别的资源，Pods 使用它来消费持久存储。PVC 使用 **PersistentVolume (PV)** 来映射实际的存储资源，存储可以位于 NFS、iSCSI 等支持的存储系统上。

#### PersistentVolumes (PVs)

**PV** 是供 PVC 使用的资源，用来将 PVC 与底层存储系统连接。手动管理 PV 非常麻烦，因此应尽量避免，而应该通过 Kubernetes 提供的 **容器存储接口 (CSI)** 来自动管理 PV 和 PVC。

#### PodDisruptionBudgets (PDB)

**PDB** 是一种限制在任何时候最大不可用 Pod 数量的资源，防止多 Pod 同时终止导致服务中断。通过定义 `minAvailable` 参数，PDB 可以确保在维护或其他中断情况下，集群内有足够数量的 Pod 可用。

#### Pods

**Pod** 资源用于与运行你的容器的 Pods 进行交互。你可以使用 `kubectl get pods -n <namespace>` 获取指定命名空间中所有 Pods 的列表。

#### PodTemplates

**PodTemplates** 提供了一种创建 Pod 模板或蓝图的方式，包含 Pod 的元数据和规范，如名称、标签、容器和卷等。通常用于生成多个具有相同配置的 Pods。

#### PriorityClasses

**PriorityClasses** 允许根据重要性为 Pods 分配优先级，调度器会根据优先级决定资源分配。你可以创建一个 **PriorityClass** 资源，并为其分配数值，数值越大，优先级越高。

#### ReplicaSets

**ReplicaSets** 用于创建和维护指定数量的 Pods。如果 Pods 数量不足，Kubernetes 会创建缺失的 Pods；如果 Pods 数量过多，则会删除多余的 Pods。通常推荐使用 **Deployment** 来创建和管理 ReplicaSets。

#### ResourceQuotas

**ResourceQuotas** 用于在多租户集群中限制某个命名空间使用的资源总量，防止单个租户耗尽所有资源。可以为 CPU、内存、PVCs 等资源设定限额。

#### RoleBindings

**RoleBinding** 资源用于将一个 Role 或 ClusterRole 关联到一个特定的主体和命名空间。例如，以下 RoleBinding 将允许 `aws-codebuild` 用户在 `openunison` 命名空间内应用 `patch-openunison` ClusterRole：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: patch-openunison
  namespace: openunison
subjects:
- kind: User
  name: aws-codebuild
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: patch-deployment
  apiGroup: rbac.authorization.k8s.io

```

即使引用的是 ClusterRole，但该权限仅适用于 `openunison` 命名空间。如果 `aws-codebuild` 用户试图在其他命名空间中修改 Deployment，API 服务器将阻止该操作。

#### Roles

与 ClusterRole 类似，**Role** 将 API 组和操作结合起来定义一组权限，但 **Role** 只能在命名空间级别上定义资源，且这些权限仅在特定的命名空间中生效。

#### RuntimeClasses

**RuntimeClasses** 用于配置不同的容器运行时环境，使你能够根据工作负载需求调整容器运行时的行为。每个 RuntimeClass 都与一个特定的容器运行时（如 Docker 或 Containerd）相关联，并包括资源限制、安全配置和环境变量等可配置参数。

#### Secrets

**Secrets** 用于安全存储敏感信息，如密码、API 密钥等。Secrets 以 Base64 编码形式存储，虽然这种编码不是加密，但提供了独立的访问控制和使用外部密钥管理系统注入敏感信息的方式。

你可以通过文件、目录或字面字符串来创建 Secrets。例如，可以使用以下命令将名为 `dbpwd` 的文件内容作为 Secret：

```ini
kubectl create secret generic mysql-admin --from-file=./dbpwd

```

#### SelfSubjectAccessReviews

**SelfSubjectAccessReviews** 允许用户检查自己是否有权限在命名空间内对资源执行特定操作。提供用户名、操作和资源后，API 服务器会验证权限并返回允许或拒绝的结果。

#### SelfSubjectRulesReviews

**SelfSubjectRulesReviews** 用于确定用户或实体在命名空间内的权限规则。相比 SelfSubjectAccessReviews，SelfSubjectRulesReviews 提供了更全面的视图，帮助用户了解其权限规则。

#### Service accounts

**ServiceAccounts** 用于管理工作负载的访问权限。当你创建一个 Deployment 时，可能需要访问其他服务或 Kubernetes 资源。通过将服务账户与 RoleBindings 和 Roles 结合，可以为应用程序授予所需的访问权限。

#### Services

**Service** 资源为应用程序提供网络暴露。由于 Pods 是临时的，IP 地址在生命周期中可能会更改，而服务的 IP 地址是持久的。服务会根据标签选择器动态维护 Pods 列表，并根据这些 Pods 创建服务的 Endpoints。

服务有几种类型：

*   **ClusterIP**：仅在集群内部可访问。
*   **NodePort**：在所有节点上暴露服务，使用一个随机分配的端口。
*   **LoadBalancer**：创建一个外部负载均衡器（云提供商支持）。
*   **ExternalName**：将 Kubernetes 内部 DNS 名称映射到外部服务。

#### StatefulSets

**StatefulSets** 提供有序部署、已知的 Pod 名称和持久化存储等特性。它们适用于需要这些功能的应用程序，如数据库。在 StatefulSet 中，Pods 会按顺序创建，并附带持久化卷。

#### StorageClasses

**StorageClasses** 定义存储端点，允许用户为持久数据选择最合适的存储解决方案。可以为不同性能的存储创建不同的存储类，并通过 PVC 使用。

#### SubjectAccessReviews

**SubjectAccessReviews** 用于检查某个实体是否有权限在资源上执行特定操作，有助于验证权限并排查权限不足导致的部署失败问题。

#### **TokenReviews**

**TokenReviews** 是用于验证与集群中的用户或实体关联的身份验证令牌的 API 对象。如果令牌有效，API 服务器会检索与该令牌相关的用户或实体的详细信息。

当用户向 Kubernetes API 服务器提交身份验证令牌时，API 服务器会根据内部身份验证系统验证令牌的合法性，并确定与该令牌相关的用户或实体。API 服务器会提供有关该令牌有效性的信息以及相关的用户或实体信息，包括用户名、用户标识符 (UID) 和组成员身份。

#### **ValidatingWebhookConfigurations**

**ValidatingWebhookConfiguration** 是一组规则，确定哪些入站请求会被拦截并由 webhook 处理。每条规则都包含 webhook 应处理的特定资源和操作。

它提供了一种通过将验证逻辑应用于入站请求来强制执行特定策略或规则的方式。许多 Kubernetes 插件都会提供 **ValidatingWebhookConfiguration**，最常见的之一是 NGINX Ingress 控制器。

你可以通过执行以下命令查看集群中的所有 **ValidatingWebhookConfiguration**：

```arduino
kubectl get validatingwebhookconfigurations

```

在我们部署的 KinD 集群中，会有一个 NGINX Ingress Admission 的条目：

```null
NAME                      		WEBHOOKS   	AGE
ingress-nginx-admission   		1       	3d10h

```

#### **VolumeAttachments**

**VolumeAttachments** 负责在集群中的外部存储卷和节点之间创建连接。它们控制持久卷与特定节点的关联，使节点能够访问和使用存储资源。

**总结**
------

在本章中，你快速学习了 Kubernetes 的基础知识，并接触了大量的技术信息。随着你深入 Kubernetes 的世界，相关概念会变得更加易于理解。需要注意的是，本章讨论的许多资源将在后续章节中进一步深入探讨和解释，以帮助你获得更深的理解。

你了解了各个 Kubernetes 组件及其相互依赖关系，这些组件共同构成了一个完整的集群。有了这些知识，你现在具备了调查和识别集群中错误或问题根本原因的必要技能。我们探讨了控制平面，其中包括 API 服务器、kube-scheduler、etcd 以及控制器管理器。此外，你还熟悉了 Kubernetes 节点上的 kubelet 和 kube-proxy 组件，以及容器运行时。

我们还深入探讨了 kubectl 工具的实际使用，它将是你与集群交互的主要工具。你学习了一些基本命令，例如访问日志和查看描述信息的命令，这些都是日常操作中常用的工具。

在下一章中，我们将创建一个开发用的 Kubernetes 集群，它将作为接下来章节中的基础集群。在接下来的内容中，我们将引用本章中介绍的许多资源，并通过实际例子进行详细解释。