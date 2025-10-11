# Kubernetes从入门到精通系列——服务、负载均衡和网络策略
在上一章中，我们启动了 Kubernetes 训练营，为你提供了一个快速但深入的 Kubernetes 基础知识和对象的介绍。我们首先分解了 Kubernetes 集群的主要部分，重点介绍了控制平面和工作节点。控制平面是集群的大脑，负责管理所有事务，包括任务调度、创建部署以及跟踪 Kubernetes 对象。而工作节点则用于运行应用程序，包含了如 kubelet 服务，用于保持容器健康，以及 kube-proxy，用于处理网络连接等组件。

我们还介绍了如何使用 kubectl 工具与集群进行交互，这个工具允许你直接运行命令或使用 YAML 或 JSON 清单来声明你希望 Kubernetes 执行的操作。我们还探讨了大多数 Kubernetes 资源，其中一些常见的资源包括 DaemonSets（确保在所有或特定节点上运行 Pod）、StatefulSets（用于管理具有稳定网络身份和持久存储的有状态应用程序）以及 ReplicaSets（用于保持一定数量的 Pod 副本运行）。

训练营章节应该为你提供了一个对 Kubernetes 架构、关键组件和资源以及基础资源管理的扎实理解。这些基本知识为接下来几章中的高级主题奠定了基础。

在本章中，你将学习如何管理和路由 Kubernetes 服务的网络流量。我们将首先解释负载均衡器的基础知识，以及如何设置它们以处理访问你应用程序的传入请求。你将了解使用服务对象的重要性，以确保尽管 Pod 的 IP 地址是临时的，但仍能可靠地连接到它们。

此外，我们还将介绍如何使用 Ingress 控制器将基于 Web 的服务暴露给外部流量，以及如何使用 LoadBalancer 服务来处理更复杂的、非 HTTP/S 工作负载。通过部署一个 Web 服务器，你将亲身体验这些概念的实际应用。

由于许多读者可能没有用于名称解析的 DNS 基础设施（Ingress 需要名称解析才能工作），我们将使用一个免费的互联网服务 nip.io 来管理 DNS 名称。

最后，我们将探讨如何使用网络策略来保护 Kubernetes 服务，确保内部和外部通信的安全。

本章将涵盖以下主题：

*   负载均衡器的介绍及其在流量路由中的作用
*   理解 Kubernetes 中的服务对象及其重要性
*   使用 Ingress 控制器暴露基于 Web 的服务
*   使用 LoadBalancer 服务处理复杂工作负载
*   部署 NGINX Ingress 控制器并设置 Web 服务器
*   使用 nip.io 服务管理 DNS 名称
*   通过网络策略保护服务通信

在本章结束时，你将深入了解在 Kubernetes 集群中暴露和保护工作负载的各种方法。

技术要求
----

本章的技术要求如下：

*   一台运行 Docker 的 Ubuntu 22.04+ 服务器，最低 4 GB 内存，建议使用 8 GB。
*   访问本书 GitHub 仓库中的 chapter4 文件夹脚本，链接为：[github.com/PacktPublis…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FPacktPublishing%2FKubernetes-An-Enterprise-Guide-Third-Edition "https://github.com/PacktPublishing/Kubernetes-An-Enterprise-Guide-Third-Edition")。

暴露工作负载给请求
---------

根据我们的经验，有三个 Kubernetes 概念可能会让人感到困惑：服务（Services）、Ingress 控制器和 LoadBalancer 服务。要使工作负载对外部世界可访问，了解这些对象的功能及其不同的选项至关重要。接下来，我们将深入探讨这些主题。

### 了解服务（Services）如何工作

正如我们之前提到的，当工作负载在 Pod 中运行时，它会被分配一个 IP 地址。然而，如果 Pod 重新启动，它将获取一个新的 IP 地址。因此，直接定位 Pod 的工作负载并不是一个好主意，因为其 IP 地址可能会发生变化。

Kubernetes 的一个很酷的功能是能够扩展部署（Deployments）。当你扩展一个 Deployment 时，Kubernetes 会添加更多的 Pod 来处理增加的资源需求。每个 Pod 都会获得自己独特的 IP 地址。但问题是，大多数应用程序设计只针对一个 IP 地址或名称。

想象一下，你的应用程序从只运行一个 Pod 扩展到运行 10 个 Pod。由于你只能针对一个 IP 地址，如何利用这些额外的 Pod 呢？这就是我们接下来要探讨的内容。

Kubernetes 中的服务利用标签（Labels）在服务和处理工作负载的 Pod 之间建立连接。当 Pod 启动时，它们会被分配标签，而所有具有相同标签的 Pod 会被分组在一起，如在部署中定义的那样。

让我们以 NGINX Web 服务器为例。在我们的 Deployment 中，我们将创建如下的清单：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx-frontend
  name: nginx-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx-frontend
  template:
    metadata:
      labels:
        run: nginx-frontend
    spec:
      containers:
      - image: bitnami/nginx
        name: nginx-frontend

```

这个 Deployment 将创建三个 NGINX 服务器，并且每个 Pod 都会被标记为 `run=nginx-frontend`。我们可以使用 kubectl 命令并添加 `--show-labels` 选项来验证这些 Pod 是否被正确标记：

```sql
kubectl get pods 

```

这将列出每个 Pod 及其相关的标签：

```arduino
nginx-frontend-6c4dbf86d4-72cbc           1/1     Running            0          19s    pod-template-hash=6c4dbf86d4,run=nginx-frontend
nginx-frontend-6c4dbf86d4-8zlwc           1/1     Running            0          19s    pod-template-hash=6c4dbf86d4,run=nginx-frontend
nginx-frontend-6c4dbf86d4-xfz6m           1/1     Running            0          19s    pod-template-hash=6c4dbf86d4,run=nginx-frontend

```

在此示例中，每个 Pod 都被赋予了一个名为 `run=nginx-frontend` 的标签。这个标签在配置应用程序的服务时非常重要。通过在服务配置中利用这个标签，服务将自动生成所需的端点，而无需手动干预。

#### 创建服务

在 Kubernetes 中，服务（Service）是一种使你的应用程序可以被其他程序或用户访问的方式。可以把它想象成你的应用程序的一个网关或入口点。

Kubernetes 中有四种不同类型的服务，每种服务都有特定的用途。本章将详细介绍每种服务类型，但现在我们先简单了解一下：

| **服务类型** | **描述** |
| --- | --- |
| **ClusterIP** | 创建一个只能在集群内部访问的服务。 |
| **NodePort** | 创建一个可以通过指定端口从集群内或外部访问的服务。 |
| **LoadBalancer** | 创建一个可以从集群内部或外部访问的服务。对于外部访问，还需要一个负载均衡组件来创建负载均衡对象。 |
| **ExternalName** | 创建一个不指向集群中的端点的服务，而是使用服务名称来指向外部 DNS 名称作为端点。 |

**表 4.1：Kubernetes 服务类型**

还有一种附加的服务类型称为无头服务（Headless Service）。Kubernetes 无头服务是一种允许直接与各个 Pod 通信的服务类型，而不像其他服务那样将流量分布在它们之间。与常规服务为一组 Pod 分配一个固定的 IP 地址不同，无头服务不会分配集群 IP。无头服务通过在服务定义中将 `clusterIP` 设为 `None` 来创建。

要创建一个服务，你需要创建一个包含 `kind`、`selector`、`type` 和用于连接服务的端口的服务对象。对于我们的 NGINX 部署示例，我们希望在端口 80 和 443 上暴露服务。我们在部署中将标签设置为 `run=nginx-frontend`，因此当我们创建清单时，我们将使用该名称作为选择器：

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    run: nginx-frontend
  name: nginx-frontend
spec:
  selector:
    run: nginx-frontend
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  - name: https
    port: 443
    protocol: TCP
    targetPort: 443
  type: ClusterIP

```

如果在服务清单中没有定义类型，Kubernetes 会将默认类型分配为 `ClusterIP`。

现在服务已经创建，我们可以通过几个 `kubectl` 命令来验证它是否被正确定义。首先检查服务对象是否已创建。要检查我们的服务，我们可以使用 `kubectl get services` 命令：

```scss
NAME              TYPE        CLUSTER-IP    EXTERNAL-IP     PORT(S)           AGE
nginx-frontend    ClusterIP   10.43.142.96  <none>          80/TCP,443/TCP    3m49s

```

在验证服务已创建之后，我们可以验证端点（Endpoints/EndpointSlices）是否已创建。请记住，在 Bootcamp 章节中提到过，端点是与我们在服务中使用的标签匹配的任何 Pod。使用 `kubectl`，我们可以通过执行 `kubectl get ep <服务名称>` 来验证端点：

```erlang
NAME              ENDPOINTS
nginx-frontend    10.42.129.9:80,10.42.170.91:80,10.42.183.124:80 + 3 more...

```

我们可以看到服务显示了三个端点，但在端点列表中也显示了 `+3 more`。由于输出被截断，`get` 的输出有限，无法显示所有端点。我们可以通过 `describe` 命令来获取更详细的列表。使用 `kubectl`，你可以执行 `kubectl describe ep <服务名称>` 命令：

```yaml
Name:         nginx-frontend
Namespace:    default
Labels:       run=nginx-frontend
Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2020-04-06T14:26:08Z
Subsets:
  Addresses:          10.42.129.9,10.42.170.91,10.42.183.124
  NotReadyAddresses:  <none>
  Ports:
    Name        Port  Protocol
    ----        ----  --------
    http        80    TCP
    https       443   TCP
Events:  <none>

```

如果比较 `get` 和 `describe` 命令的输出，可能会发现端点数似乎不匹配。`get` 命令显示了总共六个端点：显示了三个 IP 端点，并由于被截断，还列出了 `+3`，共计六个端点。而 `describe` 命令只显示了三个 IP 地址，而不是六个。这两者看起来结果不同的原因是什么？

`get` 命令会列出每个端点和端口的地址列表。由于我们的服务定义了暴露两个端口，因此每个地址都会有两个条目，每个暴露的端口一个。地址列表始终包含服务的每个 socket，因此可能会多次列出端点地址，每个 socket 一次。

`describe` 命令处理输出的方式不同，地址列在一行，所有端口列在地址的下方。乍一看，`describe` 命令似乎少了三个地址，但由于它将输出分成多个部分，因此只列出了一次地址。所有端口都列在地址列表的下方；在我们的示例中，显示了端口 80 和 443。

这两条命令显示了相似的数据，但它们的格式不同。

现在服务已暴露给集群，你可以使用分配的服务 IP 地址来连接应用程序。虽然这种方式可以工作，但如果服务对象被删除并重新创建，地址可能会发生变化。因此，建议不要直接使用 IP 地址，而是使用在创建服务时分配的 DNS 名称。

在下一节中，我们将解释如何使用内部 DNS 名称来解析服务。

#### 使用 DNS 来解析服务

到目前为止，我们已经展示了在 Kubernetes 中创建对象时，对象会被分配一个 IP 地址。问题是，当你删除一个对象（如 Pod 或服务）并重新部署时，它很可能会获得一个不同的 IP 地址。由于 Kubernetes 中的 IP 地址是临时的，我们需要一种方法来使用非变化的地址来定位这些对象。这时，Kubernetes 集群中的内置 DNS 服务就派上了用场。

当服务被创建时，内部的 DNS 记录会自动生成，允许集群内的其他工作负载通过名称来查询它。如果所有的 Pod 都位于同一个命名空间内，我们可以使用简单的短名称（如 `mysql-web`）来方便地访问服务。然而，在多个命名空间中使用服务的情况下，如果工作负载需要与不同命名空间中的服务进行通信，则必须使用该服务的全名来进行连接。

下表展示了如何从不同命名空间访问某个服务的示例：

| **集群名称**: cluster.local | **目标服务**: mysql-web | **目标服务命名空间**: database |
| --- | --- | --- |
| **Pod 命名空间** | **有效的 MySQL 服务连接名称** |  |
| database | mysql-web |  |
| kube-system | mysql-web.database.svc mysql-web.database.svc.cluster.local |  |
| productionweb | mysql-web.database.svc mysql-web.database.svc.cluster.local |  |

**表 4.2：内部 DNS 示例**

如上表所示，你可以通过标准命名规范 `<服务名称>.<命名空间>.svc.<集群名称>` 来定位另一个命名空间中的服务。在大多数情况下，当你访问不同命名空间中的服务时，不需要添加集群名称，因为它会被自动附加。

为了进一步阐述服务的整体概念，让我们深入探讨每种服务类型的具体情况，并了解它们如何用来访问我们的工作负载。

### 理解不同类型的服务

当你创建服务时，可以指定服务类型，如果没有指定，默认会使用 ClusterIP 类型。服务类型决定了服务如何被暴露给集群本身或外部流量。

#### ClusterIP 服务

ClusterIP 是最常用但也经常被误解的服务类型。如果回顾之前的表 4.1，你会发现 ClusterIP 的描述表明它只允许集群内部的服务连接。ClusterIP 类型不允许任何外部通信访问暴露的服务。

仅向内部集群工作负载暴露服务的概念可能会让人感到困惑。接下来我们将描述一个用例，说明仅向集群本身暴露服务如何在增加安全性的同时也合理。

暂时不考虑外部流量，专注于当前的部署。我们的主要目标是理解每个组件如何协同工作以构建应用程序。以 NGINX 为例，我们将通过添加一个用于 Web 服务器的后台数据库来增强部署。

到目前为止，这是一个简单的应用程序：我们创建了部署、为 NGINX 服务器创建了一个叫做 `web frontend` 的服务，以及一个叫做 `mysql-web` 的数据库服务。为了配置 Web 服务器与数据库的连接，我们决定使用 ConfigMap 来指定目标数据库服务。

你可能会认为，因为我们只使用了一个数据库服务器，我们可以简单地使用 Pod 的 IP 地址。尽管这最初可能可行，但如果 Pod 重启，它的 IP 地址会改变，导致 Web 服务器无法连接数据库。由于 Pod 的 IP 是临时的，服务应始终被使用，即使你只是连接一个单一的 Pod。

虽然我们可能希望在某个时间点将 Web 服务器暴露给外部流量，但为什么需要暴露 `mysql-web` 数据库服务呢？由于 Web 服务器位于同一个集群，且在这个案例中与数据库在同一个命名空间，因此我们只需要使用 ClusterIP 地址类型让 Web 服务器连接数据库服务器。由于数据库无法从集群外访问，因此它更加安全，因为它不会允许任何外部流量进入集群。

通过使用服务名称而不是 Pod IP 地址，当 Pod 重启时我们不会遇到问题，因为服务是根据标签而不是 IP 地址来定位的。我们的 Web 服务器将简单地查询 Kubernetes DNS 服务器获取 `mysql-web` 服务名称，它包含匹配 `mysql-web` 标签的所有 Pod 的端点。

#### NodePort 服务

NodePort 服务提供集群内外对服务的访问。起初，它可能看起来是暴露服务的理想选择，因为它使服务对所有人可访问。然而，它通过在节点上分配一个端口来实现这一点（默认情况下，使用的端口范围是 30000-32767）。依赖 NodePort 可能会给用户带来困惑，因为他们需要记住分配给服务的特定端口。稍后你将看到如何访问 NodePort 服务，并展示为什么我们不建议在生产环境中使用它。

虽然在大多数企业环境中，你不应该将 NodePort 服务用于生产工作负载，但在某些情况下使用它是合理的，特别是在故障排除时。当应用程序出现问题并且 Kubernetes 平台或 Ingress 控制器被归咎时，我们可能会暂时将服务从 ClusterIP 改为 NodePort，以便不使用 Ingress 控制器来测试连通性。通过使用 NodePort 访问应用程序，我们绕过了 Ingress 控制器，从而将其排除为可能导致问题的组件之一。如果我们可以通过 NodePort 访问工作负载并且它正常工作，我们就知道问题不在应用程序本身，并可以将工程资源指向 Ingress 控制器或其他可能的根本原因。

要创建使用 NodePort 类型的服务，只需在清单中将类型设置为 `NodePort`。我们可以使用之前的 NGINX 部署示例清单，只需将类型更改为 `NodePort`：

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    run: nginx-frontend
  name: nginx-frontend
spec:
  selector:
    run: nginx-frontend
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  - name: https
    port: 443
    protocol: TCP
    targetPort: 443
  type: NodePort

```

可以像查看 ClusterIP 服务一样使用 `kubectl` 查看端点。运行 `kubectl get services` 将显示你新创建的服务：

```scss
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                       AGE
nginx-frontend    NodePort   10.43.164.118   <none>        80:31574/TCP,443:32432/TCP    4s

```

输出显示类型是 `NodePort`，并且我们暴露了服务的 IP 地址和端口。如果你查看端口，会注意到，与 ClusterIP 服务不同，NodePort 服务显示两个端口而不是一个。第一个端口是内部集群服务可以连接的暴露端口，第二个端口号是可以从集群外部访问的随机生成的端口。

由于我们为服务暴露了端口 80 和 443，两个 NodePorts 都被分配了。如果有人需要从集群外部连接到服务，他们可以通过提供的端口访问任何工作节点。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/27925aa89cfb4c6da8a5cca4fe83f8a6~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728727859&x-signature=AQ4iWo2QO6lvAW6iGJ2DSWHYl1g%3D)

每个节点都会维护一个 NodePort 列表及其对应的服务。由于该列表在所有节点之间共享，你可以通过使用指定的端口访问任何正常运行的节点，Kubernetes 会将请求路由到一个正在运行的 Pod。

为了便于理解流量流向，我们创建了一个图示，展示了到 NGINX Pod 的网络请求流程：

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/10018aee1ebb4ed59e306558045678ba~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728727859&x-signature=clz8jiklj8KNSgmPUw81kMGt%2FuA%3D)

使用 NodePort 服务暴露服务时有几个需要注意的问题：

1.  如果删除并重新创建服务，分配的 NodePort 端口将发生变化。
2.  如果你访问的节点离线或出现问题，请求将会失败。
3.  使用太多的 NodePort 服务可能会让人混淆。你需要记住每个服务的端口，并且这些服务没有关联的外部名称。这可能会让用户在访问集群内的服务时感到困惑。

由于这些限制，建议在生产环境中尽量减少使用 NodePort 服务。

#### LoadBalancer 服务

许多人在初学 Kubernetes 时，会发现 LoadBalancer 类型的服务可以为服务分配一个外部 IP 地址。由于外部 IP 地址可以被网络上的任何机器直接访问，这对于服务来说是一个有吸引力的选项，许多人因此优先尝试使用它。不过，很多用户在使用本地 Kubernetes 集群时，尝试创建 LoadBalancer 服务时会失败。

LoadBalancer 服务依赖于与 Kubernetes 集成的外部组件来创建分配给服务的 IP 地址。大多数本地 Kubernetes 部署不包含这种服务类型。如果缺少这些外部组件，当你尝试使用 LoadBalancer 服务时，服务的 EXTERNAL-IP 状态栏会显示 `<pending>`，表示正在等待分配外部 IP。

在本章的后面，我们将详细解释 LoadBalancer 服务的工作原理以及如何实现它。

#### ExternalName 服务

ExternalName 服务是一种独特的服务类型，适用于特定的用例。当你查询使用 ExternalName 类型的服务时，最终的目标并不是集群中运行的 Pod，而是一个外部的 DNS 名称。

用 Kubernetes 外的常见场景举例来说，这类似于使用 CNAME 别名记录。当你查询 CNAME 记录时，它解析为主机记录，而不是 IP 地址。

在使用这种服务类型之前，你需要了解它可能对应用程序带来的潜在问题。如果目标端点使用了 SSL 证书，而你查询的主机名与目标服务器证书上的名称不同，你的连接可能会因为名称不匹配而失败。如果你遇到这种情况，你可以使用包含替代名称（SAN）的证书。将多个名称关联到一个证书上，能帮助解决这种问题。

为了解释为何使用 ExternalName 服务是一个不错的选择，我们通过以下例子说明：

**FooWidgets 应用需求**

*   FooWidgets 在其 Kubernetes 集群上运行一个应用程序，该程序需要连接到一个运行在 Windows 2019 服务器上的数据库，名为 `sqlserver1.foowidgets.com`（192.168.10.200）。
*   当前应用程序部署在命名空间 `finance` 中。
*   SQL 服务器将在下季度迁移到一个容器中。

**需求：** 

1.  配置应用程序仅使用集群的 DNS 服务器连接外部数据库服务器。
2.  在 SQL 服务器迁移后，FooWidgets 不得对应用程序做任何配置更改。

基于这些需求，使用 ExternalName 服务是一个完美的解决方案。实现方法如下：

首先，创建一个 ExternalName 服务的 manifest 文件来为数据库服务器创建 ExternalName 服务：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sql-db
  namespace: finance
spec:
  type: ExternalName
  externalName: sqlserver1.foowidgets.com

```

然后，将应用程序配置为使用新服务的名称 `sql-db`。因为服务和应用程序在同一命名空间中，应用程序可以直接查询 `sql-db` 名称。

当应用程序查询 `sql-db` 时，它会解析为 `sqlserver1.foowidgets.com`，并将 DNS 请求转发到外部 DNS 服务器，最终解析为 IP 地址 `192.168.10.200`。

这样就完成了第一项需求：使用 Kubernetes DNS 服务器连接到外部数据库服务器。

你可能会疑惑，为什么不直接在应用程序中配置数据库服务器的名称。关键在于第二项需求：在 SQL 服务器迁移到集群后，不能重新配置应用程序。

因为我们不能在 SQL 服务器迁移后更改应用程序的设置，如果最初配置应用程序使用 `sqlserver1.foowidgets.com`，那么在迁移后应用程序将无法工作。通过使用 ExternalName 服务，我们可以在删除 ExternalName 服务后，创建一个普通的 Kubernetes 服务，并指向集群中的 SQL 服务器。

实现第二项目标的步骤如下：

1.  删除 ExternalName 服务，因为它已不再需要。
2.  创建一个新的服务，仍然使用名称 `sql-db`，选择器为 `app=sql-app`。manifest 文件如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: sql-db
  name: sql-db
  namespace: finance
spec:
  ports:
  - port: 1433
    protocol: TCP
    targetPort: 1433
  name: sql
  selector:
    app: sql-app
  type: ClusterIP

```

由于我们为新服务使用了相同的服务名称，应用程序无需进行任何更改。应用程序仍然会查询 `sql-db`，而这个名称现在会指向集群中的 SQL 服务器。

现在你了解了服务的概念，接下来我们将讨论负载均衡器，它能让你使用标准的 URL 和端口从外部访问服务。

负载均衡器简介
-------

在本节中，我们将讨论使用第七层（Layer 7）和第四层（Layer 4）负载均衡器的基础知识。要理解不同类型负载均衡器之间的差异，首先需要了解开放系统互连（OSI）模型。理解OSI模型的不同层次将有助于你理解不同的解决方案如何处理传入的请求。

### 了解OSI模型

在Kubernetes中暴露应用程序有多种方法，你经常会遇到第七层或第四层负载均衡的提法。这些术语指的是它们在OSI模型中的位置，每一层都提供不同的功能。位于第七层的组件提供的功能与第四层的组件不同。

首先，我们来简要回顾一下OSI模型的七层及其描述。在本章中，我们主要关注两个部分：第四层和第七层：

| OSI 层 | 名称 | 描述 |
| --- | --- | --- |
| 7 | 应用层 | 提供应用流量，包括HTTP和HTTPS |
| 6 | 表示层 | 数据打包和加密 |
| 5 | 会话层 | 控制流量传输 |
| 4 | 传输层 | 设备间通信流量，包括TCP和UDP |
| 3 | 网络层 | 设备间的路由，包括IP |
| 2 | 数据链路层 | 物理连接的错误检查（MAC地址） |
| 1 | 物理层 | 设备的物理连接 |

你不需要成为OSI模型的专家，但你应该理解第四层和第七层负载均衡器的作用，以及如何在集群中使用它们。

**深入了解第四层和第七层：** 

*   **第四层**：如表格所述，第四层负责设备之间的流量通信。运行在第四层的设备能够访问TCP/UDP信息。基于第四层的负载均衡器为你的应用程序提供服务，处理任何TCP/UDP端口的传入请求。
*   **第七层**：第七层负责为应用程序提供网络服务。当我们提到应用程序流量时，并不是指像Excel或Word这样的应用程序，而是指支持这些应用的协议，如HTTP和HTTPS。

对于一些人来说，这些概念可能很新，要完全理解每一层可能需要多个章节的内容——这超出了本书的范围。我们希望你从这个介绍中了解的主要观点是，像数据库这样的应用程序不能通过第七层负载均衡器暴露给外部。要暴露不使用HTTP/S流量的应用程序，需要使用第四层负载均衡器。

在下一节中，我们将解释每种负载均衡器类型以及如何在Kubernetes集群中使用它们来暴露服务。

第七层负载均衡器
--------

Kubernetes 提供了**Ingress 控制器**作为第七层负载均衡器，用于访问应用程序。启用 Ingress 的方式有多种选择，包括以下几种：

*   **NGINX**
*   **Envoy**
*   **Traefik**
*   **HAProxy**

你可以将第七层负载均衡器想象成网络流量的指挥官，其作用是将传入的请求分配到多个托管网站或应用程序的服务器。

当你访问网站或使用应用程序时，你的设备会向服务器发送请求，要求提供特定的网页或数据。使用第七层负载均衡器时，请求并不会直接到达单个服务器，而是通过负载均衡器发送流量。第七层负载均衡器会检查请求的内容，理解所请求的网页或数据，并根据后端服务器的健康状况、当前工作负载，甚至是用户的位置等因素，智能地选择最合适的服务器来处理请求。

第七层负载均衡器确保所有服务器都得到高效利用，并为用户提供流畅的响应体验。可以将其类比为商店中的多个结账柜台，商店经理会引导顾客前往最不忙碌的柜台，从而减少等待时间，确保每个人都能及时得到服务。

简而言之，第七层负载均衡器优化了整体系统性能和可靠性。

### 名称解析和第七层负载均衡器

为了在 Kubernetes 集群中处理第七层流量，你需要部署一个 **Ingress 控制器**。Ingress 控制器依赖传入的名称来将流量路由到正确的服务。这比传统服务器部署模型中的流程要简单得多，后者需要创建 DNS 记录并将其映射到 IP 地址，用户才能通过名称从外部访问应用程序。

在 Kubernetes 集群中部署的应用程序也不例外——用户将通过分配的 DNS 名称访问应用程序。最常见的实现方式是创建一个新的通配符域名，该域名将通过外部负载均衡器（如 F5、HAProxy 或 Seesaw）指向 Ingress 控制器。通配符域名会将所有给定域的流量指向同一个目的地。例如，如果你的通配符域名是 `foowidgets.com`，那么域名中的主要入口是 `*.foowidgets.com`。任何使用通配符域名分配的 Ingress URL 名称都会将流量导向外部负载均衡器，再由负载均衡器根据 Ingress 规则 URL 将流量导向定义的服务。

以 `foowidgets.com` 域为例，我们有三个 Kubernetes 集群，由外部负载均衡器和多个 Ingress 控制器端点前置。我们的 DNS 服务器将为每个集群创建记录，使用一个指向负载均衡器虚拟 IP 地址的通配符域名：

| 域名 | IP 地址 | Kubernetes 集群 |
| --- | --- | --- |
| \*.cluster1.foowidgets.com | 192.168.200.100 | Production001 |
| \*.cluster2.foowidgets.com | 192.168.200.101 | Production002 |
| \*.cluster3.foowidgets.com | 192.168.200.102 | Development001 |

**表 4.4：Ingress 的通配符域名示例**

以下图示展示了整个请求的流程：

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/8cc7a37efc8b4d4680c4035729b9942f~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728727859&x-signature=SkwPTkiREU35UZz0oGcB5gISWpk%3D)

图 4.3 中的每个步骤详细说明如下：

1.  用户通过浏览器请求此 URL: `https://timesheets.cluster1.foowidgets.com`。
2.  DNS 查询发送到 DNS 服务器。DNS 服务器查找 `cluster1.foowidgets.com` 的区域详细信息。在 DNS 区域中有一个条目，解析为在负载均衡器上为该域分配的虚拟 IP（VIP）地址。
3.  负载均衡器的 VIP 地址（`cluster1.foowidgets.com`）有三个后端服务器分配，指向我们部署了 Ingress 控制器的三个工作节点。
4.  使用其中一个端点，请求被发送到 Ingress 控制器。
5.  Ingress 控制器会将请求的 URL 与 Ingress 规则列表进行比较。当找到匹配的请求时，Ingress 控制器会将请求转发给分配给 Ingress 规则的服务。

为了帮助加深对 Ingress 的理解，您可以在集群上创建 Ingress 规则并观察它们的运行效果。此时，关键要点是 Ingress 使用请求的 URL 来将流量引导到正确的 Kubernetes 服务。

### 使用 nip.io 进行名称解析

许多个人开发集群（例如我们的 KinD 安装）可能没有访问 DNS 基础设施的权限，或者缺乏添加记录的必要权限。为了测试 Ingress 规则，我们需要使用 Ingress 控制器将唯一的主机名映射到 Kubernetes 服务。如果没有 DNS 服务器，您需要创建一个 localhost 文件，里面包含多个指向 Ingress 控制器 IP 地址的名称。

例如，如果您部署了四个 Web 服务器，您需要将所有四个名称添加到本地主机文件中。如下所示：

```lua
192.168.100.100 webserver1.test.local
192.168.100.100 webserver2.test.local
192.168.100.100 webserver3.test.local
192.168.100.100 webserver4.test.local

```

也可以将它们表示为单行，而不是多行：

```lua
192.168.100.100 webserver1.test.local webserver2.test.local webserver3.test.local webserver4.test.local

```

如果您使用多台机器来测试部署，您需要在每台用于测试的机器上编辑主机文件。管理多台机器上的多个文件是一场管理噩梦，这将导致测试中的各种问题。

幸运的是，有一些免费 DNS 服务可以使用，而无需为 KinD 集群配置复杂的 DNS 基础设施。

我们将使用 nip.io 服务来满足 KinD 集群的名称解析需求。按照之前的 Web 服务器示例，我们不需要创建任何 DNS 记录。我们仍然需要将不同服务器的流量发送到运行在 `192.168.100.100` 上的 NGINX 服务器，以便 Ingress 可以将流量路由到适当的服务。nip.io 使用包含 IP 地址的主机名格式来解析名称到 IP。例如，假设我们有四个 Web 服务器，分别命名为 `webserver1`、`webserver2`、`webserver3` 和 `webserver4`，并且在 `192.168.100.100` 上的 Ingress 控制器上有相应的 Ingress 规则。

如前所述，我们不需要创建任何记录来实现这一点。相反，我们可以使用命名约定，让 nip.io 为我们解析名称。每个 Web 服务器将使用以下命名标准：

```xml
<desired name>.<INGRESS IP>.nip.io

```

下表列出了四个 Web 服务器的名称：

| Web 服务器名称 | Nip.io DNS 名称 |
| --- | --- |
| webserver1 | webserver1.192.168.100.100.nip.io |
| webserver2 | webserver2.192.168.100.100.nip.io |
| webserver3 | webserver3.192.168.100.100.nip.io |
| webserver4 | webserver4.192.168.100.100.nip.io |

**表 4.5：nip.io 示例域名**

当您使用上述任意一个名称时，nip.io 会将它们解析为 `192.168.100.100`。以下截图展示了每个名称的示例 ping 结果：

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/e5eb6db78f2941cabc560e39999da7b0~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728727859&x-signature=dOZbIZrTsUYv4erw0sfwFx3da%2Fg%3D)

请记住，Ingress 规则需要唯一的名称才能正确地将流量路由到相应的服务。虽然在某些情况下可能不需要知道服务器的 IP 地址，但对于 Ingress 规则来说，这是必不可少的。每个名称都应是唯一的，通常使用完整名称的第一部分。在我们的示例中，唯一的名称是 `webserver1`、`webserver2`、`webserver3` 和 `webserver4`。

通过提供此服务，nip.io 允许您为 Ingress 规则使用任何名称，而无需在开发集群中拥有 DNS 服务器。

现在您已经了解如何使用 nip.io 为集群解析名称，接下来我们将解释如何在 Ingress 规则中使用 nip.io 名称。

### 创建 Ingress 规则

请记住，Ingress 规则使用名称将传入的请求路由到正确的服务。

以下是传入请求的图形表示，展示了 Ingress 如何路由流量：

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/98b69250eacc480aada321455a3716ae~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728727859&x-signature=zK%2FPs5GqBHgwWEpptRU95EtkKq0%3D)

图 4.5 显示了 Kubernetes 处理传入 Ingress 请求的高级概览。为了更详细地解释每个步骤，让我们深入探讨以下五个步骤。根据图 4.5 中提供的图示，我们将详细说明每个编号的步骤，以展示 Ingress 如何处理请求：

1.  用户在浏览器中请求名为 `http://webserver1.192.168.200.20.nip.io` 的 URL。一个 DNS 请求被发送到本地 DNS 服务器，最终转发至 nip.io DNS 服务器。
2.  nip.io 服务器将域名解析为 IP 地址 `192.168.200.20`，并将其返回给客户端。
3.  客户端将请求发送给运行在 `192.168.200.20` 上的 Ingress 控制器。请求包含完整的 URL 名称 `webserver1.192.168.200.20.nip.io`。
4.  Ingress 控制器在配置的规则中查找请求的 URL 名称，并将该 URL 名称与服务匹配。
5.  服务的端点（Endpoints）将用于将流量路由到分配的 Pods。
6.  请求最终被路由到运行 Web 服务器的某个 Pod 端点。

通过上述示例的流量流程，让我们创建一个 NGINX Pod、服务和 Ingress 规则来实际展示这一过程。在 `chapter4/ingress` 目录中，我们提供了一个名为 `nginx-ingress.sh` 的脚本，该脚本将部署 Web 服务器并使用名为 `webserver.w.x.y.nip.io` 的 Ingress 规则来暴露它。执行脚本时，它将输出一个完整的 URL，您可以使用该 URL 来测试 Ingress 规则。

该脚本将执行以下步骤来创建新的 NGINX 部署并使用 Ingress 规则来暴露它：

1.  部署一个名为 `nginx-web` 的 NGINX 部署，Web 服务器使用端口 8080。
2.  我们创建一个名为 `nginx-web` 的服务，使用 ClusterIP 服务（默认设置），端口为 8080。
3.  脚本会自动检测主机的 IP 地址，并使用该 IP 地址创建新的 Ingress 规则，该规则将使用主机名 `webserver.w.x.y.z.nip.io`，其中 `w.x.y.z` 会被主机的 IP 地址替换。
4.  部署完成后，您可以从本地网络中的任何机器上使用脚本提供的 URL 测试 Web 服务器。比如，在我们的示例中，主机的 IP 地址为 `192.168.200.20`，因此我们的 URL 为 `webserver.192.168.200.20.nip.io`。

通过这些步骤，您可以轻松地测试 Kubernetes 中的 Ingress 配置。

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/7439b1762b804ee4b32692c4b1ee04e2~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728727859&x-signature=dp2kt2ZIgafwyhHBBA5W6h9cQh8%3D)

通过本节提供的详细信息，可以为多个容器生成使用唯一主机名的 Ingress 规则。需要注意的是，您并不局限于使用像 nip.io 这样的服务来进行名称解析；您可以采用环境中任何可用的名称解析方法。在生产集群中，通常会有企业级 DNS 基础设施。然而，在像我们 KinD 集群这样的实验环境中，nip.io 是一个用于测试场景的优秀工具，尤其是在需要准确命名约定时。

由于我们会在整本书中使用 nip.io 的命名标准，因此在继续学习下一章之前，理解这种命名约定非常重要。

### 在 Ingress 控制器中解析名称

如前所述，Ingress 控制器主要是第 7 层负载均衡器，主要处理 HTTP/S 流量。那么，Ingress 控制器如何获取主机名呢？你可能会认为它包含在网络请求中，但实际上不是。客户端使用的是 DNS 名称，但在网络层中没有名称，只有 IP 地址。

那么，Ingress 控制器如何知道您要连接的主机是什么呢？这取决于您使用的是 HTTP 还是 HTTPS。如果使用 HTTP，您的 Ingress 控制器将从 HTTP 请求头中的 `Host` 字段中获取主机名。例如，下面是一个 HTTP 客户端向集群发出的简单请求：

```makefile
GET / HTTP/1.1
Host: k8sou.apps.192-168-2-14.nip.io
User-Agent: curl/7.88.1
Accept: */*

```

第二行告诉 Ingress 控制器请求应该去哪个主机和哪个服务。使用 HTTPS 时情况则更复杂，因为连接是加密的，解密需要在读取 `Host` 头之前完成。

当使用 HTTPS 时，Ingress 控制器会根据您想连接的服务和主机名提供不同的证书。为了在没有读取 `Host` 头的情况下进行路由，Ingress 控制器会使用一种叫做 SNI（服务器名称指示）的协议，该协议在 TLS 密钥交换过程中包含请求的主机名。通过 SNI，Ingress 控制器可以在请求解密之前确定哪个 Ingress 配置对象适用于该请求。

### 在非 HTTP 流量中使用 Ingress 控制器

使用 SNI 产生了一个有趣的副作用，即当使用 TLS 时，Ingress 控制器可以模拟第 4 层负载均衡器。大多数 Ingress 控制器提供一个名为 TLS 直通（TLS passthrough）的功能，即 Ingress 控制器不解密流量，而是根据请求的 SNI 将其路由到某个服务。以我们之前的 Web 服务器后端数据库为例，如果您在 Ingress 对象中配置了 TLS passthrough 注释（每个控制器的注释方式不同），您就可以通过 Ingress 曝露数据库。

您可能会认为创建 Ingress 对象非常容易，这可能带来安全隐患。这就是为什么本书中有很大篇幅讨论了安全问题的原因之一。配置错误的环境是很容易的！

除了潜在的安全问题之外，使用 TLS passthrough 的主要缺点是您失去了许多 Ingress 控制器原生的路由和控制功能。例如，如果您部署了一个维护会话状态的 Web 应用程序，通常会将 Ingress 对象配置为使用粘性会话（sticky sessions），以确保每个用户的请求都返回到同一个容器。这是通过在 HTTP 响应中嵌入 Cookie 来实现的，但如果控制器只是传递流量，它就无法做到这一点。

第 7 层负载均衡器（如 NGINX Ingress）通常用于各种工作负载，包括 Web 服务器。然而，某些部署可能需要在 OSI 模型中更低层次运行的更复杂的负载均衡器。随着我们逐层深入模型，某些工作负载可能需要访问额外的低层功能。

在深入探讨第 4 层负载均衡器之前，如果您已经在集群上部署了 NGINX 示例，请在继续之前删除所有对象。您可以执行 `nginx-ingress-remove.sh` 脚本（位于 `chapter4/ingress` 目录中）来轻松删除部署、服务和 Ingress 规则。

第4层负载均衡器
--------

与第7层负载均衡器类似，第4层负载均衡器也是网络的流量控制器，但与第7层负载均衡器相比有一些区别。

第7层负载均衡器能够理解传入请求的内容，并基于具体信息（如网页或数据请求）做出决策。而第4层负载均衡器则工作在更低的层次，它仅查看传入网络流量中的基础信息，如IP地址和端口，而不会检查实际数据的内容。

当您访问网站或使用应用时，设备会向服务器发送包含唯一IP地址和特定端口号（即所谓的套接字）的请求。第4层负载均衡器会观察这些地址和端口，将流量有效分配给多个服务器。可以将其想象为一个交通警察，高效地将驶入的汽车引导到高速公路上的不同车道。负载均衡器并不知道每辆车的确切目的地或用途，它只会查看车牌号码并将它们引导到合适的车道，以确保交通顺畅。

通过这种方式，第4层负载均衡器确保服务器能够公平接收传入的请求，并且网络能够高效运行。这是确保网站和应用程序在用户数量较多时不至于过载，保持网络稳定性和可靠性的关键工具。

在这个过程中涉及一些更低级别的网络操作，这超出了本书的范围。HAProxy 在其官网提供了关于术语和配置示例的总结，可以访问 [www.haproxy.com/fr/blog/loa…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.haproxy.com%2Ffr%2Fblog%2Floadbalancing-faq%2F "https://www.haproxy.com/fr/blog/loadbalancing-faq/") 查看详情。

总结来说，第4层负载均衡器是一种基于IP地址和端口号分配流量的网络工具，帮助网站和应用程序高效运行，提供无缝的用户体验。

### 第4层负载均衡器的选项

如果您想为Kubernetes集群配置第4层负载均衡器，有多种可选方案，其中包括：

*   HAProxy
*   NGINX Pro
*   Seesaw
*   F5 Networks
*   MetalLB

这些选项都提供第4层负载均衡功能，但本书将重点介绍MetalLB，它已成为为Kubernetes集群提供第4层负载均衡器的热门选择。

#### 使用 MetalLB 作为第4层负载均衡器

回想一下，在第2章《使用KinD部署Kubernetes》中，我们展示了一张图，描述了工作站与KinD节点之间的流量流向。由于KinD运行在嵌套的Docker容器中，第4层负载均衡器在网络连接方面会有一定的限制。如果不对Docker主机进行额外的网络配置，您将无法在Docker主机之外访问使用LoadBalancer类型的服务。然而，如果将MetalLB部署在运行于主机上的标准Kubernetes集群中，您将不受限于只能在主机内部访问服务。

MetalLB 是一个免费、易于配置的第4层负载均衡器。它包括强大的配置选项，能够在开发实验室或企业集群中运行。由于其高度灵活性，MetalLB已经成为需要第4层负载均衡的集群中的热门选择。

我们将专注于在第2层模式下安装MetalLB。这是一个简单的安装过程，适用于开发环境或小型Kubernetes集群。MetalLB还提供BGP模式的部署选项，它允许您建立对等伙伴关系以交换网络路由。如果您想了解MetalLB的BGP模式，可以访问MetalLB的网站：[metallb.universe.tf/concepts/bg…](https://link.juejin.cn/?target=)

### 安装 MetalLB

在我们部署MetalLB之前，建议从新建集群开始。虽然这不是必需的，但它可以避免之前测试的资源引发的问题。要删除并重新部署一个新的集群，请按照以下步骤操作：

1.  使用 `kind delete` 命令删除集群：
    
    ```sql
    kind delete cluster 
    
    ```
    
2.  切换到chapter2目录，使用脚本 `create-cluster.sh` 重新创建一个新的集群。
    
3.  部署完成后，切换到chapter4/metallb目录。
    

我们在chapter4/metallb目录中提供了一个脚本 `install-metallb.sh`。该脚本将使用预先配置的文件 `metallb-config.yaml` 部署MetalLB v0.13.10。部署完成后，集群中将部署MetalLB组件，包括控制器和扬声器。

该脚本的注释详细解释了每个步骤的操作，它执行以下步骤将MetalLB部署到集群中：

1.  MetalLB 被部署到集群中，脚本会等待MetalLB控制器完全部署完成。
2.  脚本会找到Docker网络上使用的IP范围，并为LoadBalancer服务创建两个不同的地址池。
3.  使用找到的地址池，脚本会将IP范围注入到 `metallb-pool.yaml` 和 `metallb-pool-2.yaml` 中。
4.  第一个地址池通过 `kubectl apply` 命令进行部署，并同时部署 `l2advertisement` 资源。
5.  脚本会显示MetalLB命名空间中的pod，确认它们已成功部署。
6.  最后，脚本会部署一个名为 `nginx-lb` 的NGINX web服务器pod，并通过MetalLB IP地址提供一个LoadBalancer服务来访问该部署。

接下来，我们将解释MetalLB的配置文件，该文件配置了MetalLB如何处理请求。

#### 理解 MetalLB 的自定义资源

MetalLB 通过两个自定义资源进行配置，这些资源包含了MetalLB的配置。我们将使用MetalLB的第2层模式，并创建两个自定义资源：第一个用于IP地址范围，称为 `IPAddressPool`；第二个用于配置广告的池，称为 `L2Advertisement` 资源。

OSI模型的第2层可能对很多读者来说是新的概念——第2层指的是OSI模型中的一层，它在本地网络内的通信中起着关键作用。这一层确定了如何使用网络基础设施（例如以太网电缆）以及如何识别其他设备。第2层仅处理本地网络段，它不负责在不同网络之间进行流量路由，这是OSI模型第3层（网络层）的职责。

简单来说，第2层为同一网络中的设备提供了通信的便利。它通过分配MAC地址（唯一地址）给设备，并提供一种发送和接收数据的方法，这些数据被组织成网络数据包。我们在chapter4/metallb目录中提供了预先配置的资源文件 `metallb-pool.yaml` 和 `l2advertisement.yaml`，这些文件将在第2层模式下配置MetalLB，并使用Docker网络的一部分IP地址范围通过 `L2Advertisement` 资源进行广告宣传。

#### MetalLB 组件

我们的部署使用了MetalLB项目提供的标准清单文件，该文件将创建一个部署来安装MetalLB控制器，并使用DaemonSet将第二个组件部署到所有节点，即所谓的“扬声器”（speaker）。

##### 控制器

控制器将接收来自每个工作节点上speaker的公告。这些公告显示了每个请求LoadBalancer服务的服务，以及控制器为该服务分配的IP地址。例如：

```json
{"caller":"main.go:49","event":"startUpdate","msg":"start of service update","service":"my-grafana-operator/grafana-operator-metrics","ts":"2020-04-21T21:10:07.437701161Z"}
{"caller":"service.go:98","event":"ipAllocated","ip":"10.2.1.72","msg":"IP address assigned by controller","service":"my-grafana-operator/grafana-operator-metrics","ts":"2020-04-21T21:10:07.438079774Z"}
{"caller":"main.go:96","event":"serviceUpdated","msg":"updated service object","service":"my-grafana-operator/grafana-operator-metrics","ts":"2020-04-21T21:10:07.467998702Z"}

```

在上述输出中，一个名为 `my-grafana-operator/grafana-operator-metrics` 的服务已被部署，并且MetalLB分配了IP地址 `10.2.1.72`。

##### Speaker

Speaker组件是MetalLB用于向本地网络公告LoadBalancer服务的IP地址的部分。该组件运行在每个节点上，确保网络配置和路由器了解LoadBalancer服务分配的IP地址，从而使服务无需在每个节点上进行额外的网络接口配置即可接收流量。

Speaker组件的主要任务是向本地网络传达如何访问Kubernetes集群中的服务。可以将其视为向其他网络设备发送数据路由指示的信使。

它主要负责四项任务：

1.  **服务检测**：当Kubernetes中创建服务时，Speaker组件时刻监控LoadBalancer服务的创建。
2.  **IP地址管理**：Speaker负责管理IP地址，决定为使服务能够进行外部通信而应分配哪些IP地址。
3.  **路由公告**：当MetalLB的Speaker识别到需要外部访问的服务并分配了IP地址后，它会将这些路由信息传播到整个本地网络，告知网络如何通过指定的IP地址连接到服务。
4.  **负载均衡**：MetalLB执行网络负载均衡。如果有多个Pod（通常应用程序应该有多个），Speaker会在这些Pod之间分配传入的网络流量，确保负载均衡，以优化性能和可靠性。

默认情况下，MetalLB通过DaemonSet部署，以确保冗余性——无论部署了多少个Speaker，只会有一个处于活跃状态。如果该Speaker Pod出现故障，另一个Speaker实例将接管公告。

以下是从一个节点的Speaker日志中看到的公告示例：

```json
{"caller":"main.go:176","event":"startUpdate","msg":"start of service update","service":"my-grafana-operator/grafana-operator-metrics","ts":"2020-04-21T21:10:07.437231123Z"}
{"caller":"main.go:189","event":"endUpdate","msg":"end of service update","service":"my-grafana-operator/grafana-operator-metrics","ts":"2020-04-21T21:10:07.437516541Z"}
{"caller":"main.go:176","event":"startUpdate","msg":"start of service update","service":"my-grafana-operator/grafana-operator-metrics","ts":"2020-04-21T21:10:07.464140524Z"}
{"caller":"main.go:246","event":"serviceAnnounced","ip":"10.2.1.72","msg":"service has IP, announcing","pool":"default","protocol":"layer2","service":"my-grafana-operator/grafana-operator-metrics","ts":"2020-04-21T21:10:07.464311087Z"}
{"caller":"main.go:249","event":"endUpdate","msg":"end of service update","service":"my-grafana-operator/grafana-operator-metrics","ts":"2020-04-21T21:10:07.464470317Z"}

```

上述公告针对的是Grafana组件。可以看到，服务被分配了IP地址 `10.2.1.72`，这个公告也会发送给MetalLB控制器。

现在你已经安装了MetalLB并了解了这些组件如何创建服务，接下来我们将在KinD集群中创建我们的第一个LoadBalancer服务。

### 创建一个LoadBalancer服务

在第7层负载均衡器部分中，我们创建了一个运行NGINX的部署，并通过创建服务和Ingress规则将其暴露。在该部分结束时，我们删除了所有资源，为此次测试做准备。如果你按照Ingress部分的步骤操作，但尚未删除服务和Ingress规则，请在创建LoadBalancer服务之前进行删除。

MetalLB部署脚本已经包含了一个带有LoadBalancer服务的NGINX服务器。该脚本将在80端口上创建一个带有LoadBalancer服务的NGINX部署。LoadBalancer服务将从我们定义的地址池中分配一个IP地址，由于这是第一个使用地址池的服务，它可能会被分配到`172.18.200.100`这个IP地址。

你可以通过在Docker主机上使用`curl`测试这个服务。使用分配给服务的IP地址，输入以下命令：

```null
curl 172.18.200.100

```

你将会收到以下输出：

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/973e2877b6554c318a2777364c04a64c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728727859&x-signature=IzEluWjPg0VjIN3QyDRsa797Zh0%3D)

### 高级地址池配置

MetalLB 的 `IPAddressPool` 资源提供了一些高级选项，可用于不同场景，包括禁用自动分配地址、使用静态IP地址和多个地址池、将池范围限定在特定命名空间或服务中，以及处理不稳定的网络问题。

#### 禁用自动地址分配

默认情况下，当一个地址池被创建后，它将自动开始为任何请求 `LoadBalancer` 类型的服务分配地址。虽然这是一种常见的实现方式，但有时你可能会遇到特殊的用例，例如只有在明确请求时，才允许地址池分配地址。

#### 为服务分配静态IP地址

当服务从地址池中分配到一个IP地址后，该IP地址将保留，直到该服务被删除并重新创建。根据创建的 `LoadBalancer` 服务数量，有可能会在重新创建时分配到相同的IP地址，但不能保证每次都会分配相同的地址，因此应假定IP可能会更改。

如果你使用像 `external-dns` 这样的插件（将在下一章中讨论），你可能不关心服务的IP地址变化，因为你可以使用与分配IP地址相关联的域名。不过，在某些场景下，如果服务的IP地址在重新部署时更改，可能会导致问题。

截至本书撰写时，Kubernetes 已经提供了通过在服务资源中添加 `spec.loadBalancerIP` 字段来为服务分配特定IP地址的功能。使用此选项，你可以“静态”地为服务分配IP地址，并且即使服务被删除并重新部署，IP地址也会保持不变。这在多种场景中非常有用，例如可以将已知的IP地址添加到Web应用防火墙（WAF）和防火墙规则中。

从 Kubernetes 1.24 开始，`loadBalancerIP` 字段已被弃用，尽管它仍可在 Kubernetes 1.27 中使用，但该字段可能会在未来的版本中被移除。鉴于此选项将被移除，建议使用所部署的第4层负载均衡器中包含的解决方案。对于 MetalLB，他们添加了一个名为 `metallb.universe.tf/loadBalancerIPs` 的注释来分配IP地址。设置此字段为所需的IP地址将实现与已弃用的 `spec.loadBalancerIP` 相同的功能。

你可能会想到，分配静态IP可能会带来一些潜在风险，比如IP地址冲突，导致连接问题。幸运的是，MetalLB 提供了一些功能来缓解这些潜在风险。如果 MetalLB 不是请求地址的所有者，或者该地址已被其他服务使用，则IP分配将失败。如果发生此故障，MetalLB 将生成一个警告事件，可以通过运行 `kubectl describe service <service name>` 命令查看。

以下清单展示了如何使用Kubernetes原生的 `loadBalancerIP` 和 MetalLB 的注释来为服务分配静态IP地址。第一个示例展示了弃用的 `spec.loadBalancerIP`，为服务分配了 `172.18.200.210` 的IP地址：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-web
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: nginx-web
  type: LoadBalancer
  loadBalancerIP: 172.18.200.210

```

以下示例展示了如何设置 MetalLB 的注释来分配相同的IP地址：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-web
  annotations:
    metallb.universe.tf/loadBalancerIPs: 172.18.200.210
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: nginx-web
  type: LoadBalancer

```

#### 使用多个地址池

在我们的原始示例中，我们为集群创建了一个地址池。在更复杂的环境中，可能需要添加额外的地址池来将流量定向到某个网络，或者因为原始池中的地址已用完而需要添加额外的池。

你可以在集群中创建任意数量的地址池。我们在第一个池中分配了一些地址，现在我们需要添加一个额外的池来处理集群中的工作负载。创建新池只需部署一个新的 `IPAddressPool`，如下所示：

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: pool-02
  namespace: metallb-system
spec:
  addresses:
  - 172.18.201.200-172.18.201.225

```

当前版本的 MetalLB 需要重新启动 MetalLB 控制器，新的地址池才可用。

#### 为特定命名空间配置地址池范围

你可能需要为特定命名空间中的服务分配IP地址。在这种情况下，可以通过将地址池配置为特定命名空间来限制哪些服务可以从池中请求IP地址。这被称为地址池范围配置。通过配置不同优先级的池，可以有效地管理多个池和服务请求。

MetalLB 的高级配置选项为网络管理和IP分配提供了高度的灵活性和定制能力。

#### IP地址池范围配置

在企业环境中，多租户集群十分常见。默认情况下，MetalLB 的地址池对所有部署的 `LoadBalancer` 服务都可用。虽然这对许多组织来说可能不是问题，但有时你可能需要将某些地址池限制为仅限特定命名空间使用，以限制哪些工作负载可以使用特定的地址池。

要对地址池进行范围配置，我们需要在 `IPAddressPool` 资源中添加一些字段。以下是一个示例，我们希望部署一个地址池，该池的整个 Class C 地址范围仅对 `web` 和 `sales` 这两个命名空间可用：

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ns-scoped-pool
  namespace: metallb-system
spec:
  addresses:
    - 172.168.205.0/24
   serviceAllocation:
    priority: 50
    namespaces:
      - web
      - sales

```

部署此资源后，只有在 `web` 或 `sales` 命名空间中的服务才能从该地址池请求IP地址。如果来自其他命名空间的请求试图使用 `ns-scoped-pool`，该请求将被拒绝，且不会为服务分配 172.168.205.0 范围内的IP地址。

#### 处理网络中的问题

MetalLB 提供了一个字段，可以帮助处理某些网络设备无法正确处理 .0 或 .255 结尾的IP地址的问题。较旧的网络设备可能会将这些流量标记为潜在的 "Smurf 攻击" 并阻止流量。如果你遇到这种情况，可以在 `IPAddressPool` 资源中将 `AvoidBuggyIPs` 字段设置为 `true`。

高层次来看，"Smurf 攻击" 是通过发送大量网络消息到特殊地址，使这些消息广播到网络上的所有计算机。所有计算机会认为这些流量来自特定的地址，从而向该地址返回响应，导致大量流量涌向一个机器，形成拒绝服务攻击 (DoS)，使目标机器无法工作，干扰其上的服务。

为了避免这个问题，设置 `AvoidBuggyIPs` 字段可以防止使用 .0 和 .255 结尾的IP地址。以下是一个示例清单：

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: buggy-example-pool
  namespace: metallb-system
spec:
  addresses:
    - 172.168.205.0/24
    avoidBuggyIPs: true

```

通过将 MetalLB 作为第4层负载均衡器添加到集群中，可以迁移一些可能无法处理简单第7层流量的应用程序。

随着越来越多的应用程序被迁移到容器中，或者为容器进行了重构，你可能会遇到许多需要为单个服务支持多个协议的应用程序。在接下来的部分中，我们将解释一些需要为单个服务支持多个协议的场景。

### 使用多协议

早期版本的 Kubernetes 不允许在 `LoadBalancer` 服务中分配多个协议。如果你尝试为单个服务同时分配 TCP 和 UDP，你会收到一个错误，提示不支持多协议，资源部署将失败。

虽然 MetalLB 仍然支持这种操作，但由于 Kubernetes 在 1.20 版本中引入了名为 `MixedProtocolLBService` 的 alpha 功能，现在已经没有必要继续使用 MetalLB 的这种注释了。这个功能自 Kubernetes 1.26 版本起正式进入通用可用性（GA），成为基础功能，允许在定义多个端口时为 `LoadBalancer` 类型的服务使用不同的协议。

以 `CoreDNS` 为例，我们需要将 `CoreDNS` 暴露到外部网络中。在下一章中，我们将讨论一个用例，涉及使用 TCP 和 UDP 将 `CoreDNS` 实例暴露给外部世界。

由于 DNS 服务器在某些操作中同时使用 TCP 和 UDP 端口 53，因此我们需要创建一个 `LoadBalancer` 类型的服务，同时监听 TCP 和 UDP 端口 53。以下是一个示例，我们创建了一个新服务，同时定义了 TCP 和 UDP：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: coredns-ext
  namespace: kube-system
spec:
  selector:
    k8s-app: kube-dns
  ports:
  - name: dns-tcp
    port: 53
    protocol: TCP
    targetPort: 53
  - name: dns-udp
    port: 53
    protocol: UDP
    targetPort: 53
  type: LoadBalancer

```

如果我们部署这个清单并查看 `kube-system` 命名空间中的服务，会看到该服务成功创建，并且同时暴露了 TCP 和 UDP 的 53 端口：

```scss
NAME          TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)                     AGE
coredns-ext   LoadBalancer   10.96.192.4   172.18.200.101   53:31889/TCP,53:31889/UDP   6s

```

你会看到新的服务 `coredns-ext` 被创建，分配了 IP 地址 172.18.200.101，并且同时在 TCP 和 UDP 的 53 端口上进行了暴露。这使得该服务现在可以通过 53 端口接受这两种协议的连接。

许多负载均衡器存在的一个问题是，它们不提供服务 IP 的名称解析。用户通常希望使用易记的名称来访问服务，而不是随机的 IP 地址。Kubernetes 并不提供为服务创建外部可访问名称的功能，但有一个孵化项目正在实现这一特性。在第 5 章中，我们将解释如何为 Kubernetes 服务提供外部名称解析。

在本章的最后一部分，我们将讨论如何通过网络策略来保护工作负载的安全。

介绍网络策略
------

安全性是所有 Kubernetes 用户从第一天就应该考虑的事情。默认情况下，集群中的每个 Pod 都可以与集群中的其他 Pod 进行通信，甚至可以与您可能不拥有的其他命名空间中的 Pod 通信。虽然这是 Kubernetes 的一个基本概念，但对于大多数企业来说并不理想，尤其是在使用多租户集群时，这成为一个很大的安全问题。我们需要增强工作负载的安全性和隔离性，这可能是一个非常复杂的任务，而网络策略正是为了解决这个问题而引入的。

`NetworkPolicies` 允许用户通过定义一组规则，在 Pod、命名空间和外部端点之间控制出站和入站的网络流量。可以将网络策略看作是集群的防火墙，它提供基于各种参数的细粒度访问控制。通过网络策略，您可以控制哪些 Pod 被允许与其他 Pod 通信，限制流量到特定的协议或端口，并强制执行加密和身份验证要求。

就像我们讨论过的大多数 Kubernetes 对象一样，网络策略允许基于标签和选择器进行控制。通过匹配网络策略中指定的标签，Kubernetes 可以决定哪些 Pod 和命名空间应该被允许或拒绝网络访问。

网络策略是 Kubernetes 中的可选功能，并且集群中使用的 CNI（容器网络接口）必须支持它们。在我们创建的 KinD 集群中，部署了 Calico，它支持网络策略。然而，并不是所有的网络插件默认支持网络策略，因此在部署集群之前规划需求非常重要。

在本节中，我们将解释网络策略提供的选项，以增强您的应用程序和集群的整体安全性。

### 网络策略对象概述

网络策略提供了许多选项来控制入站和出站流量。它们可以非常细致地允许仅特定的 Pod、命名空间，甚至 IP 地址控制网络流量。

网络策略由四个部分组成，每个部分的描述如下表所示：

| 部分 | 描述 |
| --- | --- |
| `podSelector` | 使用标签选择器限制策略应用的工作负载范围。如果未提供选择器，则策略将影响命名空间中的所有 Pod。 |
| `policyTypes` | 定义策略规则。有效类型包括 `ingress`（入站）和 `egress`（出站）。 |
| `ingress` | （可选）定义入站流量的规则。如果没有定义规则，将匹配所有传入流量。 |
| `egress` | （可选）定义出站流量的规则。如果没有定义规则，将匹配所有传出流量。 |

**表4.6：网络策略的组成部分**

`ingress` 和 `egress` 部分是可选的。如果您不想阻止任何出站流量，只需省略 `egress` 规范。如果未定义规范，所有流量都将被允许。

#### podSelector

`podSelector` 字段用于指示网络策略将影响哪些工作负载。如果您希望策略仅影响某个特定的部署，您需要定义一个标签，该标签与该部署中的标签相匹配。标签选择器不限于单个条目；您可以在网络策略中添加多个标签选择器，但所有选择器都必须匹配，策略才能应用于 Pod。如果您希望策略应用于所有 Pod，可以将 `podSelector` 留空，这样策略就会应用于命名空间中的每个 Pod。

在以下示例中，我们定义了该策略仅适用于匹配标签 `app=frontend` 的 Pod：

```yaml
spec:
  podSelector:
    matchLabels:
      app: frontend

```

接下来的字段是策略的类型，您将在这里为 `ingress`（入站）和 `egress`（出站）定义策略。

#### policyTypes

`policyType` 字段用于指定正在定义的策略类型，决定 `NetworkPolicy` 的范围和行为。`policyType` 有两个可用选项：

| policyType | 描述 |
| --- | --- |
| ingress | `ingress` 控制流入 Pod 的网络流量。它定义了控制源的规则，允许从指定的 IP CIDR 范围、命名空间或集群中的其他 Pod 访问匹配 `podSelector` 的 Pod。 |
| egress | `egress` 控制从 Pod 发出的网络流量。它定义了控制目标的规则，限制 Pod 允许与之通信的目标。出站流量可以通过特定的 IP CIDR 范围、命名空间或集群中的其他 Pod 进行限制。 |

**表4.7：策略类型**

策略可以包含 `ingress`、`egress` 或两者。如果没有包含某个策略类型，则它不会影响该流量类型。例如，如果只包含 `ingress` 类型策略，则所有出站流量都将被允许到网络上的任何位置。

如前所述，当您为 `ingress` 或 `egress` 流量创建规则时，您可以不提供标签、提供单个标签或多个标签，所有标签必须匹配，策略才能生效。以下示例显示了一个 `ingress` 块，包含三个标签；要使策略影响工作负载，必须匹配所有声明的字段：

```yaml
ingress:
  - from:
      - ipBlock:
          cidr: 192.168.0.0/16
      - namespaceSelector:
          matchLabels:
            app: backend
      - podSelector:
          matchLabels:
            app: database

```

在上述示例中，任何传入的流量都必须来自以下条件的工作负载：IP 地址在 `192.168.0.0` 子网内、命名空间标签为 `app=backend`，且请求的 Pod 必须有 `app=database` 标签。

虽然示例显示的是 `ingress` 流量的选项，`egress` 流量也可以使用相同的选项。

现在我们已经介绍了网络策略中可用的选项，接下来我们将通过一个实际例子来创建完整的策略。

### 创建网络策略

在本书的 GitHub 仓库中的 `chapter4/netpol` 目录下包含了一个网络策略脚本，名为 `netpol.sh`。当您执行该脚本时，它将创建一个名为 `sales` 的命名空间，其中包含带有标签的几个 Pod 以及一个网络策略。这个创建的策略将作为我们在本节讨论的基础。

创建网络策略时，您需要对预期的网络限制有明确的理解。最了解应用程序流量的人最适合帮助制定可行的策略。您需要考虑哪些 Pod 或命名空间应能够进行通信、允许哪些协议和端口，以及是否需要其他安全措施，例如加密或认证。

与其他 Kubernetes 对象一样，您需要创建一个 `NetworkPolicy` 对象的清单，并提供元数据，如策略名称和部署的命名空间。

让我们使用一个示例，假设您有一个运行 PostgreSQL 的后端 Pod，与同一命名空间中的 Web 服务器共同工作。我们知道，唯一需要与数据库服务器通信的 Pod 是 Web 服务器本身，不应允许其他通信访问数据库。

首先，我们需要创建清单文件，首先声明 API、类型、策略名称和命名空间：

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-netpol
  namespace: sales

```

接下来是添加 `podSelector`，用于指定该策略将影响的 Pod。通过创建 `podSelector` 部分，您可以基于匹配标签的 Pod 来定义选择器。对于我们的示例，我们希望策略适用于属于后端数据库应用程序的 Pod。应用程序的 Pod 已全部标记为 `app=backend-db`：

```yaml
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: postgresql

```

现在我们已经声明了要匹配的 Pod，我们需要定义 `ingress`（入站）和 `egress`（出站）规则，它们在策略中通过 `spec.ingress` 或 `spec.egress` 部分定义。对于每种规则类型，您可以为应用程序设置允许的协议和端口，并控制从哪里允许外部请求访问该端口。基于我们的示例，我们将添加一个 `ingress` 规则，允许标签为 `app=frontend` 的 Pod 访问 `5432` 端口：

```yaml
spec:
  podSelector:
    matchLabels:
      app: frontend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 5432

```

最后一步，我们将定义策略类型。由于我们只关心 PostgreSQL Pod 的传入流量，因此我们只需声明一种类型 `ingress`：

```markdown
  policyTypes:
    - Ingress

```

一旦部署了该策略，标签为 `app=backend-db` 的 Pod 将只接受来自标签为 `app=frontend` 的 Pod 在 TCP 端口 `5432` 上的流量。来自 `frontend` Pod 的任何其他端口请求将被拒绝。此策略使 PostgreSQL 部署非常安全，因为所有传入流量都严格限制为特定工作负载和 TCP 端口。

当我们从 GitHub 仓库中执行脚本时，它将 PostgreSQL 部署到集群并为该部署添加标签。我们将使用该标签将网络策略绑定到 PostgreSQL Pod。为了测试连接性和网络策略，我们将运行一个 `netshoot` Pod，并使用 `telnet` 来测试连接到 `5432` 端口上的 Pod。

我们需要知道数据库服务器的 IP 地址以测试网络连接。只需使用 `-o wide` 选项列出命名空间中的 Pod，即可列出 Pod 的 IP 地址。现在 PostgreSQL 已经在运行，我们将模拟连接，运行一个标签不匹配 `app=frontend` 的 `netshoot` Pod，这将导致连接失败，以下是示例：

```css
kubectl get pods -n sales -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP
db-postgresql-0   0/1     Running   0          45s   10.240.189.141
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot --labels app=wronglabel -n sales
tmp-shell ~ telnet 10.240.189.141:5432

```

由于传入请求的 Pod 标签为 `app=wronglabel`，连接最终将超时。策略会查看传入请求的标签，如果它们不匹配，将拒绝连接。

最后，让我们检查策略是否创建正确。我们再次运行 `netshoot`，但这次使用正确的标签，可以看到连接成功：

```css
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot --labels app=frontend -n sales
tmp-shell ~ telnet 10.240.189.141:5432
Connected to 10.240.189.141:5432

```

注意这行 "Connected to 10.240.189.141:5432"。这表明 PostgreSQL Pod 接受了来自 `netshoot` Pod 的请求，因为 Pod 的标签与网络策略中所要求的 `app=frontend` 标签匹配。

那么，为什么网络策略只允许 `5432` 端口？我们没有设置任何拒绝流量的选项；我们只定义了允许的流量。网络策略默认拒绝所有未定义的策略通信。在我们的示例中，我们只定义了 `5432` 端口，因此任何不是来自 `5432` 端口的请求都会被拒绝。对于未定义的通信设置默认拒绝可以有效地保护您的工作负载，防止任何意外的访问。

如果您想创建一个拒绝所有流量的网络策略，只需创建一个包含 `ingress` 和 `egress` 的新策略，并且没有其他值。以下是一个示例：

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: sales
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

```

在这个示例中，我们将 `podSelector` 设置为空 `{}`，这意味着该策略将应用于命名空间中的所有 Pod。在 `spec.ingress` 和 `spec.egress` 选项中，我们没有设置任何值，并且由于默认行为是拒绝所有没有规则的通信，因此该规则将拒绝所有入站和出站流量。

### 创建网络策略的工具

手动创建网络策略可能会很困难。特别是当您不是应用程序的所有者时，确定需要打开哪些端口可能具有挑战性。

在第13章《KubeArmor 保护您的运行时》中，我们将讨论一个名为 **KubeArmor** 的工具，这是由 AccuKnox 公司捐赠的 CNCF 项目。KubeArmor 最初是一个用于保护容器运行时的工具，但最近他们增加了监控 Pod 之间网络流量的功能。通过监控流量，KubeArmor 能够了解 Pod 网络连接的“正常行为”，并为命名空间中观察到的每个网络策略创建一个 ConfigMap。

我们将在第13章中详细介绍该工具；目前我们只想让您知道有工具可以帮助您创建网络策略。

在下一章中，我们将学习如何使用 **CoreDNS** 通过一个名为 **external-dns** 的孵化项目来创建服务名称的 DNS 条目。我们还将介绍一个名为 **K8GB** 的令人兴奋的 CNCF 沙箱项目，该项目为集群提供了 Kubernetes 原生的全球负载均衡功能。

总结
--

在本章中，您学习了如何将 Kubernetes 中的工作负载暴露给其他集群资源和外部流量。

本章的第一部分介绍了服务及其多种类型，主要的三种服务类型是 **ClusterIP**、**NodePort** 和 **LoadBalancer**。请记住，服务类型的选择将决定您的应用程序如何被暴露。

在第二部分中，我们介绍了两种负载均衡器类型，分别是 **第4层** 和 **第7层**，每种类型都具备独特的功能，用于暴露工作负载。通常，您会使用 **ClusterIP** 服务配合 **Ingress 控制器** 来提供对使用第7层的服务的访问。有些应用程序可能需要第7层负载均衡器无法提供的额外通信，这时就需要使用第4层负载均衡器来外部暴露服务。在负载均衡部分，我们演示了安装和使用 **MetalLB**，这是一款流行的开源第4层负载均衡器。

最后，我们讨论了如何保护入站和出站的网络流量。由于 Kubernetes 默认允许集群中所有 Pod 之间进行通信，大多数企业环境需要一种方法来确保工作负载之间的通信仅限于应用程序所需的流量。**网络策略** 是一种强大的工具，可以保护集群并限制进出流量的流向。

您可能还有一些关于暴露工作负载的问题，例如：如何处理使用 **LoadBalancer** 类型服务的 DNS 条目？或者，如何使部署在两个集群之间实现高可用性？

在下一章中，我们将进一步讨论如何使用一些工具来暴露您的工作负载，比如名称解析和全球负载均衡。