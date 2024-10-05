# Kubernetes从入门到精通系列——外部 DNS 和全局负载均衡
在本章中，我们将基于第 4 章所学的内容，讨论某些负载均衡功能的局限性以及如何配置集群以解决这些局限性。

我们知道 Kubernetes 内置了一个 DNS 服务器，可以为资源动态分配名称。这些名称用于集群内的应用程序通信，即集群内部通信。虽然这个功能对于集群内部的通信很有用，但它无法为外部工作负载提供 DNS 解析。那么，既然 Kubernetes 提供了 DNS 解析，为什么我们说它存在局限性呢？

在上一章中，我们使用了动态分配的 IP 地址来测试我们的 LoadBalancer 服务工作负载。虽然我们的例子对于学习很有帮助，但在企业中，没有人希望通过 IP 地址来访问运行在集群上的工作负载。为了解决这个限制，Kubernetes 的 SIG 开发了一个名为 ExternalDNS 的项目，它能够为 LoadBalancer 服务动态创建 DNS 条目。

此外，在企业中，你通常会在多个集群上运行服务，以为应用程序提供故障切换功能。到目前为止，我们讨论的选项尚未解决故障切换场景的问题。在本章中，我们将解释如何实施解决方案，以实现工作负载的自动故障切换，使其在多个集群中保持高可用性。

在本章中，您将学习以下内容：

*   外部 DNS 解析和全球负载均衡的介绍
*   在 Kubernetes 集群中配置和部署 ExternalDNS
*   自动化 DNS 名称注册
*   将 ExternalDNS 集成到企业 DNS 服务器中
*   使用 GSLB（全局服务器负载均衡）在多个集群中提供全球负载均衡

现在让我们进入本章内容！

**技术要求**
--------

本章有以下技术要求：

*   一台运行 Docker 的 Ubuntu 22.04+ 服务器，至少需要 4 GB 的 RAM，建议使用 8 GB。
*   一个运行 MetalLB 的 KinD 集群——如果您已经完成了第 4 章，应该已经有一个运行 MetalLB 的集群。
*   来自本书 GitHub 仓库的 chapter5 文件夹中的脚本，您可以通过以下链接访问该仓库：[github.com/PacktPublis…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FPacktPublishing%2FKubernetes-An-Enterprise-Guide-Third-Edition "https://github.com/PacktPublishing/Kubernetes-An-Enterprise-Guide-Third-Edition")

**使服务名称可供外部访问**
---------------

正如我们在介绍中提到的，您可能还记得，我们使用 IP 地址来测试创建的 LoadBalancer 服务，而在 Ingress 示例中，我们使用了域名。那么，为什么我们要为 LoadBalancer 服务使用 IP 地址，而不是主机名呢？

虽然 Kubernetes 的负载均衡器会为服务分配一个标准的 IP 地址，但它不会自动为工作负载生成 DNS 名称来访问该服务。相反，您必须依赖 IP 地址来连接集群内的应用程序，这既令人困惑又低效。此外，手动为每个由 MetalLB 分配的 IP 注册 DNS 名称也带来了维护挑战。为了提供更类似云的体验，并简化 LoadBalancer 服务的名称解析，我们需要一个可以解决这些限制的附加组件。

与维护 KinD 的团队类似，有一个 Kubernetes 特别兴趣小组（SIG）正致力于为 Kubernetes 开发此功能，称为 ExternalDNS。该项目的主要页面可以在 SIG 的 GitHub 仓库中找到：[github.com/kubernetes-…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fkubernetes-sigs%2Fexternal-dns "https://github.com/kubernetes-sigs/external-dns")。

在撰写本文时，ExternalDNS 项目支持 34 种兼容的 DNS 服务，包括以下几种：

*   AWS Cloud Map
*   Amazon 的 Route 53
*   Azure DNS
*   Cloudflare
*   CoreDNS
*   Google Cloud DNS
*   Pi-hole
*   RFC2136

根据您所运行的主 DNS 服务器，有多种方法可以扩展 CoreDNS 以解析外部名称。许多受支持的 DNS 服务器会简单地动态注册任何服务。ExternalDNS 会检测到创建的资源，并使用本地调用自动注册服务，例如 Amazon 的 Route 53。并不是所有的 DNS 服务器都默认支持这种类型的动态注册。

在这些情况下，您需要手动配置主 DNS 服务器，将所需域名请求转发到集群中运行的 CoreDNS 实例。这正是我们在本章示例中将使用的方法。

我们当前的 Kubernetes 集群使用 CoreDNS 来处理集群 DNS 名称解析。然而，可能不太为人所知的是，CoreDNS 不仅限于集群内的 DNS 名称解析，它还可以扩展其功能，以执行外部名称解析，有效地解析由 CoreDNS 部署管理的任何 DNS 区域的名称。

现在，我们来看看 ExternalDNS 的安装过程。

### **设置 ExternalDNS**

目前，我们的 CoreDNS 只解析集群内部名称，因此我们需要为新的 LoadBalancer DNS 条目设置一个区域。

在这个示例中，FooWidgets 公司希望所有 Kubernetes 服务都使用 `foowidgets.k8s` 作为域名，因此我们将使用该域名作为我们的新区域。

### **集成 ExternalDNS 和 CoreDNS**

ExternalDNS 并不是实际的 DNS 服务器；相反，它是一个控制器，会监视请求新 DNS 条目的对象。一旦控制器检测到请求，它会将信息发送给实际的 DNS 服务器（如 CoreDNS）进行注册。

以下图表展示了服务如何被注册的流程。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/bd00f34655a54478a5a307a24527cc2a~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=WgirSziWCh%2BxjZNLcqA8dyPqCsM%3D)

在我们的示例中，我们使用 CoreDNS 作为 DNS 服务器；然而，如前所述，ExternalDNS 支持 34 种不同的 DNS 服务，且支持的服务列表在不断增长。由于我们将使用 CoreDNS 作为 DNS 服务器，因此我们需要添加一个组件来存储 DNS 记录。为此，我们需要在集群中部署一个 ETCD 服务器。

对于我们的示例部署，我们将使用 ETCD Helm chart。

**Helm** 是一个用于 Kubernetes 的工具，使部署和管理应用程序变得更简单。它使用 Helm charts，这些 charts 是包含应用程序所需配置和资源值的模板。Helm 自动化了复杂应用程序的设置，确保其配置的一致性和可靠性。它是一个强大的工具，许多项目和供应商都默认使用 Helm charts 来发布他们的应用程序。你可以在 [Helm 官方主页](https://link.juejin.cn/?target=https%3A%2F%2Fv3.helm.sh%2F "https://v3.helm.sh/") 上了解更多关于 Helm 的信息。

Helm 之所以如此强大，是因为它允许在运行 `helm install` 命令时使用自定义选项。这些选项还可以在文件中声明，并通过 `-f` 选项传递给安装命令。这些选项使得部署复杂系统变得更容易且可复现，因为相同的 values 文件可以在任何部署中重复使用。

对于我们的部署示例，我们在 `chapter5/etcd` 目录下包含了一个 `values.yaml` 文件，我们将用它来配置 ETCD 的部署。

现在，终于可以部署 ETCD 了！我们在 `chapter5/etcd` 目录下包含了一个名为 `deploy-etcd.sh` 的脚本，该脚本将在一个名为 `etcd-dns` 的新命名空间中部署一个单副本的 ETCD。进入 `chapter5/etcd` 目录后，执行该脚本。

得益于 Helm，该脚本只包含两个命令——它会创建一个新的命名空间，然后执行 Helm install 命令来部署我们的 ETCD 实例。在实际环境中，为了实现高可用性，通常需要将副本数设置为至少 3 个，但为了限制 KinD 服务器的资源需求，我们在此只部署一个副本。

现在我们有了用于 DNS 的 ETCD，可以继续将 CoreDNS 服务与新的 ETCD 部署进行集成。

### **向 CoreDNS 添加 ETCD 区域**

正如我们在上一节的图表中所展示的，CoreDNS 将在 ETCD 实例中存储 DNS 记录。为此，我们需要配置一个 CoreDNS 服务器，指定我们想要注册名称的 DNS 区域以及将记录存储到的 ETCD 服务器。

为了降低资源需求，我们将使用 Kubernetes 大多数安装中随集群创建提供的默认 CoreDNS 服务器来为新域名服务。在真实场景中，建议部署一个专门处理 ExternalDNS 注册的独立 CoreDNS 服务器。

在本节的最后，你将执行一个脚本，该脚本会部署一个完全配置好的 ExternalDNS 服务，其中包含本节讨论的所有选项和配置。本节中使用的命令仅供参考，你无需在集群上执行这些命令，因为脚本会为你处理。

在我们集成 CoreDNS 之前，需要先知道新 ETCD 服务的 IP 地址。你可以通过列出 `etcd-dns` 命名空间中的服务来获取该地址，使用以下命令：

```arduino
kubectl get svc etcd-dns -n etcd-dns

```

这将显示我们的 ETCD 服务以及服务的 IP 地址：

```scss
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
etcd-dns   ClusterIP   10.96.149.223   <none>        2379/TCP,2380/TCP   4m

```

服务列表中的 `Cluster-IP` 将用于配置新 DNS 区域，作为存储 DNS 记录的位置。

部署 ExternalDNS 时，可以通过以下两种方式配置 CoreDNS：

1.  向 Kubernetes 集成的 CoreDNS 服务添加一个区域
2.  部署一个新的 CoreDNS 服务，用于 ExternalDNS 注册

为了方便测试，我们将向 Kubernetes 的 CoreDNS 服务添加一个区域。为此，我们需要编辑位于 `kube-system` 命名空间中的 CoreDNS `ConfigMap`。当你执行本节末尾的脚本时，脚本会为你完成这个修改，它会将以下加粗的部分添加到 `ConfigMap` 中：

```yaml
apiVersion: v1
data:   
  Corefile: |
    .:53 {
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
            ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf
        etcd foowidgets.k8s {
            stubzones
            path /skydns
            endpoint http://10.96.149.223:2379
        }
        cache 30
        loop
        reload
        loadbalance
    }


```

添加的行配置了一个名为 `foowidgets.k8s` 的 ETCD 集成区域。我们添加的第一行告诉 CoreDNS，`foowidgets.k8s` 区域集成了一个 ETCD 服务。

第二行 `stubzones` 允许 CoreDNS 将 DNS 服务器设置为某个特定区域的 "stub 解析器"。作为一个 stub 解析器，此 DNS 服务器可以直接查询该区域的特定名称服务器，而无需递归解析整个 DNS 层次结构。

第三个添加的选项 `path /skydns` 可能看起来有些困惑，因为它没有提到 CoreDNS。尽管值为 `skydns`，但这也是 CoreDNS 集成的默认路径。

最后一行告诉 CoreDNS 将记录存储在哪里。在我们的示例中，我们有一个运行的 ETCD 服务，使用 IP 地址 `10.96.149.223` 和默认的 ETCD 端口 `2379`。

你也可以在这里使用服务的主机名而不是 IP 地址。我们使用 IP 是为了展示 Pod 和服务之间的关系，但主机名 `etcd-dns.etcd-dns.svc` 也能正常工作。选择哪种方法取决于你的情况。在我们的 KinD 集群中，我们不必担心 IP 地址的变更，因为该集群是临时的。而在真实环境中，你可能需要使用主机名来避免 IP 地址更改带来的问题。

现在你已经了解了如何在 CoreDNS 中添加一个 ETCD 集成区域，接下来我们将更新 ExternalDNS 所需的部署选项，以与 CoreDNS 集成。

### **ExternalDNS 配置选项**

ExternalDNS 可以配置为注册 `Ingress` 或 `Service` 对象。这通过 ExternalDNS 的部署文件中的 `source` 字段来配置。以下示例展示了我们将在本章使用的部署选项部分。

我们还需要配置 ExternalDNS 使用的提供者，因为我们使用的是 CoreDNS，所以我们将提供者设置为 `coredns`。

最后一个选项是我们想要设置的日志级别，我们将其设置为 `info`，这样日志文件更小且易于阅读。我们将在部署文件中使用的参数如下所示：

```sql
    spec:
      serviceAccountName: external-dns
      containers:
      - name: external-dns
        image: registry.k8s.io/external-dns/external-dns:v0.13.5
        args:
        - 
        - 
        - 

```

现在我们已经讨论了 ETCD 选项和部署、如何配置一个新区域以使用 ETCD，以及如何配置 ExternalDNS 使用 CoreDNS 作为提供者，接下来我们可以在集群中部署 ExternalDNS。

我们在 `chapter5/externaldns` 目录下提供了一个名为 `deploy-externaldns.sh` 的脚本。在该目录中执行该脚本，可以将 ExternalDNS 部署到 KinD 集群中。执行脚本时，它将完整配置并部署一个与 ETCD 集成的 ExternalDNS。

**注意：**  如果在脚本更新 `ConfigMap` 时看到警告，可以安全地忽略它。因为我们的 `kubectl` 命令使用 `apply` 来更新对象，Kubernetes 会查找 `last-applied-configuration` 注解，如果存在该注解则会使用它。由于现有对象中可能没有此注解，你会看到警告提示它缺失。这只是一个警告，不会阻止 `ConfigMap` 的更新，你可以通过查看 `kubectl update` 命令的最后一行来确认，那里显示 `ConfigMap` 已成功更新：`configmap/coredns configured`。

现在，我们已经为开发人员添加了动态注册服务 DNS 名称的功能。接下来，让我们通过创建一个新的服务，在 CoreDNS 服务器中自动注册它，来看看这个功能的实际效果。

### **创建带有 ExternalDNS 集成的 LoadBalancer 服务**

我们的 ExternalDNS 将监视所有带有包含所需 DNS 名称注释的服务。这是一个简单的注释，使用格式为 `external-dns.alpha.kubernetes.io/hostname`，其值为你想要注册的 DNS 名称。在本例中，我们希望注册 `nginx.foowidgets.k8s` 这个名称，因此我们将在我们的 NGINX 服务中添加以下注释：`external-dns.alpha.kubernetes.io/hostname: nginx.foowidgets.k8s`。

在 `chapter5/externaldns` 目录中，我们包含了一个清单文件，该文件将使用带有注释的 LoadBalancer 服务部署 NGINX Web 服务器，以注册 DNS 名称。

使用 `kubectl create -f nginx-lb.yaml` 部署清单，该操作会将资源部署到默认命名空间。该部署是一个标准的 NGINX 部署，但该服务包含必要的注释，以告诉 ExternalDNS 服务你想注册一个新的 DNS 名称。服务的清单如下，注释部分用粗体显示：

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    external-dns.alpha.kubernetes.io/hostname: nginx.foowidgets.k8s
  name: nginx-ext-dns
  namespace: default
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer

```

当 ExternalDNS 看到注释后，它将注册请求的名称到区域中。注释中的主机名会在 ExternalDNS pod 中记录一个条目——如下所示，我们的新条目 `nginx.foowidgets.k8s` 的注册日志如下：

```sql
time="2023-08-04T19:16:00Z" level=debug msg="Getting service (&{ 0 10 0 "heritage=external-dns,external-dns/owner=default,external-dns/resource=service/default/nginx-ext-dns" false 0 1  /skydns/k8s/foowidgets/a-nginx/39ca730e}) with service host ()"
time="2023-08-04T19:16:00Z" level=debug msg="Getting service (&{172.18.200.101 0 10 0 "heritage=external-dns,external-dns/owner=default,external-dns/resource=service/default/nginx-ext-dns" false 0 1  /skydns/k8s/foowidgets/nginx/02b5d1d5}) with service host (172.18.200.101)"
time="2023-08-04T19:16:00Z" level=debug msg="Creating new ep (nginx.foowidgets.k8s 0 IN A  172.18.200.101 []) with new service host (172.18.200.101)"

```

如日志的最后一行所示，记录作为 A 记录添加到 DNS 服务器中，指向 IP 地址 `172.18.200.101`。

最后一步是确认 ExternalDNS 是否正常工作，通过测试连接到应用程序。由于我们使用的是 KinD 集群，我们需要从集群中的 pod 测试此功能。要向外部资源提供新的名称，我们需要将主 DNS 服务器配置为将 `foowidgets.k8s` 域的请求转发到我们的 CoreDNS 服务器。此部分结束时，我们将展示如何将 Windows DNS 服务器（或网络中的任何主 DNS 服务器）与我们的 Kubernetes CoreDNS 服务器集成。

现在我们可以使用注释中的 DNS 名称来测试 NGINX 部署。由于你可能未使用 CoreDNS 服务器作为主要 DNS 提供者，因此我们需要在集群中使用一个容器来测试名称解析。`Netshoot` 是一个包含多个有用的故障排除工具的实用程序，它是测试和排查集群和 pod 的一个强大工具。

要运行 Netshoot 容器，我们可以使用 `kubectl run` 命令。我们只需要 pod 在运行测试时运行即可，因此我们将告诉 `kubectl run` 命令运行一个交互式 shell，并在退出后删除 pod。执行以下命令以运行 Netshoot：

```bash
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash

```

pod 可能需要一分钟左右才能启动，但一旦启动，你将看到一个 `tmp-shell` 提示符。在该提示符下，我们可以使用 `nslookup` 来验证 DNS 条目是否已成功添加。如果你尝试查找 `nginx.foowidgets.k8s`，应该会收到带有服务 IP 地址的回复。

```null
nslookup nginx.foowidgets.k8s

```

你的回复应类似于以下示例：

```yaml
Server:		10.96.0.10
Address:	10.96.0.10
Name:   nginx.foowidgets.k8s
Address: 172.18.200.101

```

这确认了我们的注释成功了，并且 ExternalDNS 已在 CoreDNS 服务器中注册了主机名。

`nslookup` 只能证明 `nginx.foowidgets.k8s` 有一个条目，但它并不能测试应用程序。为了证明我们有一个成功的部署，并且当有人在浏览器中输入名称时可以工作，我们可以使用 Netshoot 包含的 `curl` 实用程序。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/457d4ad096d84da4a8e6d8172919630e~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=nG2Ik0lJiLWQOXnWp8SRPVu8Hmc%3D)

`curl` 输出确认我们可以使用动态创建的服务名称访问 NGINX Web 服务器。

我们意识到，其中一些测试可能不是很令人兴奋，因为你不能通过标准浏览器进行测试。为了让 CoreDNS 能够在集群外部使用，我们需要将 CoreDNS 与主 DNS 服务器集成，并且需要为 CoreDNS 中的区域委派所有权。当你委派一个区域时，任何发往主 DNS 服务器且属于该委派区域的请求都会被转发到包含该区域的 DNS 服务器。

在下一节中，我们将把运行在集群中的 CoreDNS 与 Windows DNS 服务器集成。虽然我们使用的是 Windows 作为 DNS 服务器，但不同操作系统和 DNS 服务器之间的区域委派概念是相似的。

#### 将 CoreDNS 与企业 DNS 服务器集成

本节将展示如何使用主 DNS 服务器将 `foowidgets.k8s` 区域的名称解析请求转发到 Kubernetes 集群中运行的 CoreDNS 服务器。

这里提供的步骤是将企业 DNS 服务器与 Kubernetes DNS 服务集成的一个示例。由于涉及外部 DNS 需求和额外设置，这些步骤仅供参考，建议不要在你的 KinD 集群上执行。

要在委派的区域中查找记录，主 DNS 服务器使用一种称为递归查询的过程。递归查询是指由 DNS 解析器代表用户发起的 DNS 查询。在递归查询过程中，DNS 解析器负责联系多个 DNS 服务器，按照分层结构查找。其目标是找到请求域的权威 DNS 服务器并检索所请求的 DNS 记录。

下图展示了通过将区域委派给企业环境中的 CoreDNS 服务器，如何提供 DNS 解析的流程。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/62845c0b3ee34e46a9b1193ba968c3e5~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=2hAetsCywoBP%2Bjf0ikNb1h1xLMg%3D)

1.  本地客户端会首先查看其 DNS 缓存中是否存在所请求的名称。
    
2.  如果名称不在本地缓存中，则会向主 DNS 服务器发送关于 `nginx.foowidgets.k8s` 的请求。
    
3.  DNS 服务器接收到查询后，会查看其已知的区域，并找到 `foowidgets.k8s` 区域，发现该区域已经被委派给运行在 `192.168.1.200` 的 CoreDNS。
    
4.  主 DNS 服务器将查询发送给被委派的 CoreDNS 服务器。
    
5.  CoreDNS 在 `foowidgets.k8s` 区域中查找名称 `nginx`。
    
6.  CoreDNS 将 `foowidgets.k8s` 的 IP 地址发送回主 DNS 服务器。
    
7.  主 DNS 服务器将包含 `nginx.foowidgets.k8s` 地址的回复发送给客户端。
    
8.  客户端使用 CoreDNS 返回的 IP 地址连接到 NGINX 服务器。
    

接下来，我们来看一个实际应用的例子。在这个场景中，主 DNS 服务器运行在 Windows 2019 服务器上，我们将一个区域委派给 CoreDNS 服务器。

部署的组件如下：

*   网络子网是 `10.2.1.0/24`
*   运行 DNS 的 Windows 2019 或更高版本的服务器
*   一个 Kubernetes 集群
*   一个地址池为 `10.2.1.70-10.2.1.75` 的 MetalLB 地址池
*   集群中部署的 CoreDNS 实例，使用 LoadBalancer 服务，IP 地址为 `10.2.1.74`，来自我们的 IP 池
*   本章配置的附加组件，包括 ExternalDNS、CoreDNS 的 ETCD 部署，以及新的 CoreDNS ETCD 集成区域
*   用于测试委派的 Bitnami NGINX 部署

现在，让我们逐步进行 DNS 服务器集成的配置步骤。

将 CoreDNS 公开给外部请求
-----------------

我们已经介绍了如何部署集成所需的大部分资源——ETCD、ExternalDNS，以及如何配置带有 ETCD 集成的新区域的 CoreDNS。为了提供对 CoreDNS 的外部访问，我们需要创建一个新的服务，将 CoreDNS 暴露在 TCP 和 UDP 端口 53 上。完整的服务清单如下所示。

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: CoreDNS
  name: kube-dns-ext
  namespace: kube-system   
spec:
  ports:
  - name: dns
    port: 53
    protocol: UDP
    targetPort: 53
  selector:
    k8s-app: kube-dns
  type: LoadBalancer
  loadBalancerIP: 10.2.1.74

```

在服务中有一个我们还未讨论的新选项——我们在部署中添加了 `spec.loadBalancerIP`。此选项允许您为服务分配一个 IP 地址，使其拥有一个稳定的 IP，即使服务被重新创建也不会改变。我们需要一个静态 IP，因为我们需要启用从主 DNS 服务器到 Kubernetes 集群中 CoreDNS 服务器的转发。

一旦 CoreDNS 使用 LoadBalancer 暴露在端口 53 上，我们就可以配置主 DNS 服务器，将 `foowidgets.k8s` 域中主机的请求转发到我们的 CoreDNS 服务器。

### 配置主 DNS 服务器

我们在主 DNS 服务器上要做的第一件事是创建一个指向运行 CoreDNS pod 的节点的条件转发器。

在 Windows DNS 主机上，我们需要为 `foowidgets.k8s` 创建一个新的条件转发器，指向我们为新的 CoreDNS 服务分配的 IP 地址。在我们的示例中，CoreDNS 服务已分配给主机 `10.2.1.74`。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/f6b767acc7ed4c96b3c3915f25be5119~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=PpYxW4z8eoe9soZ9r7pC0YyS6n8%3D)

这将配置 Windows DNS 服务器，将 `foowidgets.k8s` 域中任何主机的请求转发到运行在 IP 地址 `10.2.1.74` 上的 CoreDNS 服务。

### 测试 DNS 转发到 CoreDNS

为了测试配置，我们将使用配置为使用 Windows DNS 服务器的主网络上的一台工作站。

首先，我们将在命令提示符下运行 `nslookup nginx.foowidgets.k8s` 命令，以查找通过 MetalLB 注解创建的 NGINX 记录：

```makefile
Server:  AD2.hyper-vplanet.com
Address:  10.2.1.14
Name:    nginx.foowidgets.k8s
Address:  10.2.1.75

```

由于查询返回了我们期望的记录 IP 地址，因此我们可以确认 Windows DNS 服务器已正确将请求转发到 CoreDNS。

我们还可以通过笔记本电脑的浏览器进一步测试 NGINX。在 Chrome 中，我们可以使用在 CoreDNS 中注册的 URL `nginx.foowidgets.k8s` 进行访问。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/31990cd26afe4ad7909cea870b8fd4c0~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=9g025YITcSXLevery7%2FQjm9c9zU%3D)

一次测试可以确认转发功能正常，但我们希望创建一个额外的部署以验证系统的完整性。

为了测试一个新的服务，我们部署了另一个名为 **microbot** 的 NGINX 服务器，并为其服务添加了一个注解，将其名称分配为 `microbot.foowidgets.k8s`。MetalLB 为该服务分配了 IP 地址 `10.2.1.65`。

像之前的测试一样，我们使用 `nslookup` 测试名称解析：

```makefile
Name:    AD2.hyper-vplanet.com
Address:  10.2.1.65

```

为了确认 Web 服务器正常运行，我们可以从工作站访问该 URL 进行验证。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/0f761bc002ac45658636739fbf291218~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=DEOqXq%2BBLXRHZaamu%2FLLRsymqmo%3D)

成功了！我们现在已将企业 DNS 服务器与 Kubernetes 集群上运行的 CoreDNS 服务器集成在一起。通过这种集成，用户只需在服务上添加一个注解，就可以动态地注册服务名称。

在多集群之间进行负载均衡配置
--------------

在多个集群之间进行负载均衡有多种配置方法，通常涉及到复杂且昂贵的附加组件，如全球负载均衡器。全球负载均衡器可以被视为交通指挥官，它能够知道如何在多个端点之间引导传入的流量。从宏观上讲，你可以创建一个由全球负载均衡器控制的新 DNS 记录。这个新记录会将后端系统添加到端点列表中，并根据健康状况、连接数或带宽等因素，将流量引导至合适的端点节点。如果某个端点因任何原因不可用，负载均衡器会将其从端点列表中移除。通过将其移除，流量只会被发送到健康的节点，确保用户获得顺畅的体验。对客户来说，最糟糕的情况莫过于尝试访问网站时，遇到“网站找不到”的错误。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/3e0980be40634c569db4981e86e28bca~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=Hl6c3KOE39t70TJ6IYCNLinVL%2F4%3D)

上图显示了一个健康的工作流，其中两个集群都运行了我们在集群之间进行负载均衡的应用程序。当请求到达负载均衡器时，它将以轮询的方式在两个集群之间分配流量。对于 `nginx.foowidgets.k8s` 的请求，流量最终会被发送到 `nginx.clustera.k8s` 或 `nginx.clusterb.k8s`。

在图 5.8 中，集群 B 的 NGINX 工作负载出现故障。由于全球负载均衡器对正在运行的工作负载进行了健康检查，它会将集群 B 中的端点从 `nginx.foowidgets.k8s` 条目中移除。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/8eb0b1b0c736490ab40f6b4eaa25f65f~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=Icifb6cn1VAsJl6W3RMfjs3nPC0%3D)

现在，对于任何进入负载均衡器并请求 `nginx.foowidgets.k8s` 的流量，唯一可用于处理流量的端点将是运行在集群 A 上的端点。一旦集群 B 的问题得到解决，负载均衡器将自动将集群 B 的端点重新添加到 `nginx.foowidgets.k8s` 记录中。

这种解决方案在企业中非常普遍，许多组织使用 F5、Citrix、Kemp 和 A10 等公司的产品，以及云服务提供商（CSP）原生的解决方案，如 Route 53 和 Traffic Director，来管理跨多个集群的工作负载。然而，也有与 Kubernetes 集成的类似功能的项目，并且它们几乎不需要任何成本。虽然这些项目可能不提供某些供应商解决方案的全部功能，但它们通常能够满足大多数用例的需求，而无需昂贵的全面功能。

其中一个这样的项目是 K8GB，这是一个创新的开源项目，旨在为 Kubernetes 引入全球服务器负载均衡（GSLB）。通过 K8GB，组织可以轻松地将传入的网络流量分布到位于不同地理位置的多个 Kubernetes 集群上。通过智能路由请求，K8GB 能够保证低延迟、最佳响应时间和冗余性，为企业提供了一个卓越的解决方案。

本节将向您介绍 K8GB 项目，但如果您想深入了解该项目，可以访问其主页：[www.k8gb.io](https://link.juejin.cn/?target=https%3A%2F%2Fwww.k8gb.io "https://www.k8gb.io")。由于我们使用的是KinD 和单一主机集群，本书的这一部分旨在介绍该项目及其带来的优势。本节仅供参考，因为这是一个复杂的主题，涉及多个组件，其中一些在 Kubernetes 之外。如果您决定自行实施该解决方案，我们已在本书的仓库中的 `chapter5/k8gs-example` 目录下包含了示例文档和脚本。

K8GB 是 CNCF 沙箱项目，这意味着它处于早期阶段，在撰写本章之后的任何新版本可能会对对象和配置进行更改。

引入 Kubernetes 全球负载均衡器 (K8GB)

### 为什么你应该关心像 K8GB 这样的项目？

让我们以一个内部企业云为例，该企业在生产站点运营 Kubernetes 集群，同时在灾难恢复站点拥有另一个集群。为了确保顺畅的用户体验，必须在这些数据中心之间无缝切换应用程序，并在灾难恢复事件中不需要手动干预。挑战在于如何满足企业对微服务高可用性的需求，尤其是在多个集群同时提供这些应用程序的情况下。我们需要有效解决在地理上分散的 Kubernetes 集群之间提供连续、不中断服务的需求。

这就是 K8GB 的用武之地。

那么，是什么让 K8GB 成为满足我们高可用性需求的理想解决方案呢？根据他们网站上的说明，主要功能包括以下几点：

*   负载均衡通过经过时间验证的 DNS 协议提供，该协议非常可靠，适用于全球部署。
*   无需管理集群。
*   无单点故障。
*   使用 Kubernetes 原生的健康检查进行负载均衡决策。
*   配置仅需一个 Kubernetes 自定义资源定义 (CRD)。
*   适用于任何 Kubernetes 集群——无论是本地还是云端。
*   免费使用！

在本节中，你会看到，K8GB 提供了简便直观的配置，使组织轻松实现全球负载均衡。虽然 K8GB 可能看起来功能简单，但实际上它提供了许多高级特性，包括：

*   **全球负载均衡**：促进在不同地理区域的多个 Kubernetes 集群之间分配传入网络流量，从而优化应用程序交付，确保延迟减少并改善用户体验。
*   **智能流量路由**：利用复杂的路由算法，智能地将客户端请求引导至最接近或最合适的 Kubernetes 集群，考虑到地理位置、服务器健康状态和应用程序特定规则等因素。这种方式确保了高效、响应迅速的流量管理，优化了应用程序性能。
*   **高可用性和冗余**：通过在集群、应用程序或数据中心发生故障时自动重定向流量，确保应用程序的高可用性和容错性。此故障切换机制可在灾难恢复场景中最大限度地减少停机时间，确保用户的连续服务交付。
*   **自动故障切换**：通过启用数据中心之间的自动故障切换简化操作，无需手动干预。这消除了人工触发灾难恢复 (DR) 事件或任务的需求，确保快速、不间断的服务交付，并简化操作流程。
*   **与 Kubernetes 集成**：提供与 Kubernetes 的无缝集成，简化部署在集群上的应用程序的全球负载均衡 (GSLB) 的设置和配置。通过利用 Kubernetes 的原生功能，K8GB 提供了一个可扩展的解决方案，增强了全球负载均衡操作的整体管理和效率。
*   **本地和云提供商支持**：为企业提供高效管理多个 Kubernetes 集群 GSLB 的方式，实现复杂的多区域部署和混合云场景的无缝处理。这确保了在不同环境中的应用程序交付优化，增强了基础设施的整体性能和弹性。
*   **自定义和灵活性**：为用户提供定义个性化规则和流量路由策略的自由，使企业能够根据其特定需求精确地自定义 GSLB 配置。这赋予企业根据其需求优化流量管理的能力，并确保能够适应不断变化的应用程序需求。
*   **监控、指标和跟踪**：包括监控、指标和跟踪功能，管理员可以访问跨多个集群的流量模式、健康状况和性能指标的见解。这提供了增强的可见性，使管理员能够做出明智的决策，优化 GSLB 设置的整体性能和可靠性。

现在我们已经讨论了 K8GB 的主要功能，接下来让我们深入了解细节。

### K8GB 的要求

对于像全球负载均衡这样复杂的功能，K8GB 并不需要大量的基础设施或资源来为集群提供负载均衡。最新版本在本章撰写时是 0.12.2，只有一些基本的要求：

*   在主 DNS 区域内为 CoreDNS 服务器配置负载均衡 IP 地址，使用命名标准 `gslb-ns-<k8gb-name>-gb.foowidgets.k8s`，例如 `gslb-ns-us-nyc-gb.foowidgets.k8s` 和 `gslb-ns-us-buf-gb.foowidgets.k8s`。
    
*   如果你使用的是 Route 53、Infoblox 或 NS1 等服务，CoreDNS 服务器会自动添加到域中。由于我们的示例使用的是 Windows 2019 服务器上运行的本地 DNS 服务器，因此需要手动创建这些记录。
    
*   一个 Ingress 控制器。
    
*   在集群中部署一个 K8GB 控制器，该控制器将部署：
    
    *   K8GB 控制器
    *   一个配置了 CoreDNS CRD 插件的 CoreDNS 服务器（这包括在 K8GB 的部署中）

由于我们在之前的章节中已经探索了 NGINX Ingress 控制器，现在我们将重点放在额外的要求上：在集群中部署和配置 K8GB 控制器。

在下一节中，我们将讨论实现 K8GB 的步骤。

### 向集群部署 K8GB

我们在 GitHub 仓库的 `chapter5/k8gb-example` 目录中包含了示例文件。脚本基于我们将在本章剩余部分使用的示例。如果你决定在开发集群中使用这些文件，则需要满足以下要求：

*   两个 Kubernetes 集群（每个集群可以是一个单节点的 kubeadm 集群）
*   在每个集群中部署 CoreDNS
*   在每个集群中部署 K8GB
*   一个可用于为 K8GB 委托域的边缘 DNS 服务器

安装 K8GB 变得非常简单——你只需要使用一个 `values.yaml` 文件来为你的基础设施配置 Helm 图表，然后部署该 Helm 图表即可。

要安装 K8GB，你需要将 K8GB 仓库添加到 Helm 仓库列表中，然后更新图表：

```csharp
helm repo add k8gb https:
helm repo update

```

在执行 `helm install` 命令之前，我们需要为每个集群部署自定义 Helm `values.yaml` 文件。我们在 `chapter5/k8gb-example` 目录中为两个集群提供了示例文件 `k8gb-buff-values.yaml` 和 `k8gb-nyc-values.yaml`。在下一节中，我们将讨论如何自定义这些 `values` 文件中的选项。

#### 理解 K8GB 负载均衡选项

在我们的示例中，我们将 K8GB 配置为两个本地集群之间的故障转移负载均衡器；然而，K8GB 不仅仅局限于故障转移。像大多数负载均衡器一样，K8GB 提供多种解决方案，可以根据每个负载均衡 URL 的不同需求进行配置。它提供了最常见的策略，包括轮询（Round Robin）、加权轮询（Weighted Round Robin）、故障转移（Failover）和基于地理位置的负载均衡（GeoIP）。

以下是每种策略的说明：

*   **轮询 (Round Robin)** : 如果未指定策略，默认将使用简单的轮询负载均衡配置。使用轮询意味着请求将在配置的集群之间分配：请求 1 会发送到集群 1，请求 2 会发送到集群 2，依此类推。
*   **加权轮询 (Weighted Round Robin)** : 类似于轮询，此策略允许指定发送到某个集群的流量百分比，例如 75% 的流量发送到集群 1，15% 的流量发送到集群 2。
*   **故障转移 (Failover)** : 所有流量将发送到主集群，除非部署中的所有 Pod 都不可用。如果集群 1 中的所有 Pod 都关闭，集群 2 将接管工作负载，直到集群 1 中的 Pod 再次可用，此时它将重新成为主集群。
*   **基于地理位置的负载均衡 (GeoIP)** : 将请求发送到离客户端连接最近的集群。如果最近的集群出现故障，它将使用其他集群，类似于故障转移策略的工作方式。要使用此策略，您需要创建一个 GeoIP 数据库（可以在此处找到示例：[github.com/k8gb-io/cor…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fk8gb-io%2Fcoredns-crd-plugin%2Ftree%2Fmain%2Fterratest%2Fgeogen "https://github.com/k8gb-io/coredns-crd-plugin/tree/main/terratest/geogen")），并且您的DNS 服务器需要支持 EDNS0 扩展。

EDNS0 基于 RFC 2671，它概述了 EDNS0 的工作原理及其各种组件，包括支持 EDNS0 的 DNS 消息的格式、EDNS0 选项的结构以及实现它的指南。RFC 2671 的目标是为扩展 DNS 协议的功能提供标准化方法，超越其最初的局限性，允许新功能、选项和增强功能的加入。

现在您已经了解了可用的策略，接下来我们将介绍集群示例基础设施：

| 集群/服务器详细信息 | 详细信息 |
| --- | --- |
| 企业 DNS 服务器 – 纽约市 | IP: 10.2.1.14 |
| 主企业域 | foowidgets.k8s |
| CoreDNS 服务器的主机记录 | gslb-ns-us-nyc-gb.foowidgets.k8s |
|  | gslb-ns-us-buf-gb.foowidgets.k8s |
| 已配置的全球域，委派给集群中的 CoreDNS 服务器 | gb.foowidgets.k8s |
| 纽约市，纽约 – 集群 1 | 主站点 |
| CoreDNS LB IP | 10.2.1.221 |
| Ingress IP | 10.2.1.98 |
| 通过 HostPort 暴露的 NGINX Ingress 控制器 | MetalLB 暴露的 CoreDNS 部署 |
| 布法罗，纽约 – 集群 2 | 次要站点 |
| CoreDNS LB IP | 10.2.1.224 |
| Ingress IP | 10.2.1.167 |
| 通过 HostPort 暴露的 NGINX Ingress 控制器 | MetalLB 暴露的 CoreDNS 部署 |

_表 5.1：集群详细信息_

我们将使用上表中的详细信息来解释如何在示例基础设施中部署 K8GB。

接下来，我们将基于这些基础设施详细信息创建每个部署的 Helm `values.yaml` 文件。在下一节中，我们将展示如何使用示例基础设施配置每个值，并解释每个值的含义。

#### 自定义 Helm chart 的 values 文件

每个集群都会有类似的 values 文件，主要的区别在于我们使用的标记（tag）值。以下是纽约市集群的 values 文件：

```yaml
k8gb:
  dnsZone: "gb.foowidgets.k8s" 
  edgeDNSZone: "foowidgets.k8s" 
  edgeDNSServers:
    - 10.2.1.14     
  clusterGeoTag: "us-buf" 
  extGslbClustersGeoTags: "us-nyc"
coredns:
  isClusterService: false
  deployment:
    skipConfig: true
  image:
    repository: absaoss/k8s_crd
    tag: v0.0.11
  serviceAccount:
    create: true
    name: coredns
  serviceType: LoadBalancer

```

对于纽约市集群使用相同的文件内容，除了 `clusterGeoTag` 和 `extGslbClustersGeoTags` 值外，它们需要设置为：

```vbnet
clusterGeoTag: "us-nyc"
extGslbClustersGeoTags: "us-buf"

```

如您所见，这个配置文件并不复杂，只需要少量的选项即可配置通常非常复杂的全球负载均衡配置。

现在，让我们详细解释 values 文件中的主要内容。

**主要配置说明：** 

以下是我们正在使用的 K8GB 部分中的配置项，它们用于配置 K8GB 的负载均衡功能。

| 配置项 | 描述 |
| --- | --- |
| **dnsZone** | 这是为 K8GB 使用的 DNS 区域，基本上是用于存储全球负载均衡 DNS 记录的 DNS 区域。 |
| **edgeDNSZone** | 包含用于上一选项（dnsZone）的 CoreDNS 服务器 DNS 记录的主 DNS 区域。 |
| **edgeDNSServers** | 边缘 DNS 服务器，通常是用于名称解析的主 DNS 服务器。 |
| **clusterGeoTag** | 如果您有多个 K8GB 控制器，则此标记用于指定各实例之间的差异。在我们的示例中，集群设置为 us-buf 和 us-nyc。 |
| **extGslbClusterGeoTags** | 指定要配对的其他 K8GB 控制器。在我们的示例中，每个集群都会添加其他集群的 `clusterGeoTag`，如布法罗集群添加 us-nyc，纽约市集群添加 us-buf。 |
| **isClusterService** | 设置为 true 或 false。用于服务升级，您可以在 [K8GB 文档](https://link.juejin.cn/?target=https%3A%2F%2Fwww.k8gb.io%2Fdocs%2Fservice_upgrade.xhtml "https://www.k8gb.io/docs/service_upgrade.xhtml") 中了解更多。 |
| **exposeCoreDNS** | 如果设置为 true，将创建一个 LoadBalancer 服务，公开部署在 k8gb 命名空间中的 CoreDNS，以便外部访问（使用 53/UDP 端口）。 |
| **deployment.skipConfig** | 设置为 true 或 false。将其设置为 false 会告诉部署使用 K8GB 附带的 CoreDNS。 |
| **image.repository** | 配置要使用的 CoreDNS 镜像的存储库。 |
| **image.tag** | 配置拉取镜像时使用的标签。 |
| **serviceAccount.create** | 设置为 true 或 false。设置为 true 时，将创建一个服务账户。 |
| **serviceAccount.name** | 设置上一个选项创建的服务账户名称。 |
| **serviceType** | 配置 CoreDNS 的服务类型。 |

_表 5.2：K8GB 选项_

#### 使用 Helm 安装 K8GB

在完成了对 K8GB 的概述和 Helm values 文件的解释后，我们现在可以继续在集群中安装 K8GB。我们提供了脚本来将 K8GB 部署到布法罗（Buffalo）和纽约市（NYC）集群中。在 `chapter5/k8gb-example/k8gb` 目录下，您会看到两个脚本 `deploy-k8gb-buf.sh` 和 `deploy-k8gb-nyc.sh`，它们应该在对应的集群中运行。

脚本将执行以下步骤：

1.  将 K8GB Helm 仓库添加到服务器的仓库列表中
2.  更新仓库
3.  使用相应的 Helm values 文件部署 K8GB
4.  创建 Gslb 记录（将在下一节中介绍）
5.  在名为 `demo` 的命名空间中创建一个用于测试的部署

一旦部署完成，您将在 `k8gb` 命名空间中看到两个正在运行的 Pod，一个用于 K8GB 控制器，另一个用于解析负载均衡名称的 CoreDNS 服务器：

```sql
NAME                           READY   STATUS    RESTARTS    AGE
k8gb-8d8b4cb7c-mhglb           1/1     Running   0           7h58m
k8gb-coredns-7995d54db5-ngdb2  1/1     Running   0           7h37m

```

我们还可以验证已创建的服务是否用于处理传入的 DNS 请求。由于我们使用 LoadBalancer 类型暴露服务，因此我们将在使用 UDP 协议的 53 端口上看到 LoadBalancer 服务：

```scss
NAME		TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)		AGE   
k8gb-coredns	LoadBalancer   10.96.185.56   10.2.1.224    53:32696/UDP	2d18h

```

在两个集群中完成并验证 K8GB 的部署后，让我们进入下一节，解释如何为负载均衡区域创建边缘委派。

#### **委派负载均衡区域**

在我们的示例中，我们使用 Windows 服务器作为边缘 DNS 服务器，这是 Kubernetes DNS 名称将被注册的地方。在 DNS 端，我们需要为 CoreDNS 服务器添加两个 DNS 记录，然后需要将负载均衡区域委派给这些服务器。

这两个 A 记录需要存在于主边缘 DNS 区域。在我们的示例中，该区域是 `foowidgets.k8s`。在此区域中，我们需要为通过 LoadBalancer 服务暴露的 CoreDNS 服务器添加两个条目：

1.  **A 记录 1**：指向布法罗集群的 CoreDNS 服务器
2.  **A 记录 2**：指向纽约市集群的 CoreDNS 服务器

通过这些步骤，您可以完成负载均衡 DNS 区域的委派，使得 K8GB 可以在两个集群之间进行全球负载均衡。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/4c628bdb1ba845ebae4574227ca43021~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=p38rgWgoG5BmEGjpbpgBEALB%2BZw%3D)

接下来，我们需要创建一个新的委派区域，该区域将用于我们的负载均衡服务名称。在 Windows 系统中，这可以通过右键点击区域并选择“新建委派”（New Delegation）来完成。在委派向导中，系统会要求输入委派的域名。在我们的示例中，我们将 `gb` 作为我们的委派域名。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/0d8fac83713d48e5b2a800813e8cb3ba~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=cRpbOoPX6l346iznTnS90FnVnxc%3D)

在输入区域名称并点击“下一步”后，您将看到一个新的屏幕，用于为委派的域添加 DNS 服务器。当您点击“添加”时，您需要输入 CoreDNS 服务器的 DNS 名称。记住，我们之前在主域 `foowidgets.com` 中创建了两个 A 记录。当您添加这些条目时，Windows 会验证输入的名称是否正确解析，并检查 DNS 查询是否正常工作。一旦您添加了两个 CoreDNS 服务器，摘要屏幕将显示它们及其对应的 IP 地址。

![](https://p3-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/c5f9e113e0eb4f3b828729a9e433d276~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5pWw5o2u5pm66IO96ICB5Y-45py6:q75.awebp?rk3s=f64ab15b&x-expires=1728553738&x-signature=ZqdrX6vcoJPGpahC9qUALgRx4oU%3D)

这就完成了边缘服务器的配置。对于某些边缘服务器，K8GB 会自动创建委派记录，但只有少数服务器兼容该功能。对于那些无法自动创建委派服务器的边缘服务器，您需要像我们在本节中所做的那样手动创建它们。

现在我们已经在集群中配置了 CoreDNS，并且已经委派了负载均衡域，接下来我们将部署一个具备全局负载均衡的应用程序来测试我们的配置。

### **使用 K8GB 部署高可用应用程序**

有两种方法可以为应用程序启用全局负载均衡。你可以使用 K8GB 提供的自定义资源创建新记录，或者可以为 Ingress 规则添加注解。为了演示 K8GB 的功能，我们将部署一个简单的 NGINX Web 服务器，并使用 K8GB 提供的自定义资源将其添加到 K8GB。

#### 使用自定义资源向 K8GB 添加应用程序

当我们部署 K8GB 时，一个名为 `Gslb` 的新自定义资源定义 (CRD) 被添加到了集群中。这个 CRD 负责管理标记为全局负载均衡的应用程序。在 `Gslb` 对象中，我们定义了与 Ingress 名称相匹配的规范，格式与普通 Ingress 对象类似。唯一的区别在于清单的最后部分，即策略部分。

策略定义了我们希望使用的负载均衡类型。在我们的例子中，策略为 **failover**（故障转移），并指定了对象的主 GeoTag。在我们的示例中，NYC 集群是主集群，因此我们的 `Gslb` 对象将设置为 `us-buf`。

要部署利用负载均衡的应用程序，我们需要在两个集群中创建以下内容：

*   一个标准的应用程序部署和服务。我们将其命名为 `nginx`，使用标准的 NGINX 镜像。
*   每个集群中的一个 `Gslb` 对象。对于我们的示例，我们将使用下面的清单，它声明了 Ingress 规则并将策略设置为故障转移，使用 `us-buf` 作为主 K8GB。

由于 `Gslb` 对象已经包含了 Ingress 规则的信息，因此你不需要创建 Ingress 规则，`Gslb` 会为我们创建 Ingress 对象。以下是一个示例：

```yaml
apiVersion: k8gb.absa.oss/v1beta1
kind: Gslb
metadata:
  name: gslb-failover-buf
  namespace: demo
spec:
  ingress:
    ingressClassName: nginx
    rules:
    - host: fe.gb.foowidgets.k8s      
      http:
        paths:
        - backend:
            service:
              name: nginx             
              port:
                number: 80
          path: /
          pathType: Prefix
  strategy:
    type: failover                    
    primaryGeoTag: us-buf              

```

当你为 `Gslb` 对象部署此清单时，它会创建两个 Kubernetes 对象：`Gslb` 对象和 Ingress 对象。

如果我们查看 Buffalo 集群中的 `demo` 命名空间下的 `Gslb` 对象，会看到如下内容：

```vbnet
NAMESPACE   NAME                STRATEGY   GEOTAG
demo        gslb-failover-buf   failover   us-buf

```

而如果我们查看 NYC 集群中的 Ingress 对象，会看到：

```css
NAME                CLASS   HOSTS                  ADDRESS      PORTS   AGE
gslb-failover-buf   nginx   fe.gb.foowidgets.k8s   10.2.1.167   80      15h

```

NYC 集群中也会有类似的对象，这些对象将在“**理解 K8GB 如何提供全局负载均衡**”部分中解释。

#### 使用 Ingress 注解向 K8GB 添加应用程序

向 K8GB 添加应用程序的第二种方法是为标准的 Ingress 规则添加两个注解。这主要是为了允许开发人员将现有的 Ingress 规则添加到 K8GB。

要将 Ingress 对象添加到全局负载均衡列表中，你只需要在 Ingress 对象中添加两个注解：`strategy` 和 `primary-geotag`。以下是注解示例：

```bash
k8gb.io/strategy: "failover"
k8gb.io/primary-geotag: "us-buf"

```

这将使用故障转移策略和 `us-buf` GeoTag 作为主标签，将 Ingress 添加到 K8GB。

现在，我们已经部署了所有必需的基础设施组件和对象，以便为应用程序启用全局负载均衡，让我们来看它的实际效果。

### **理解 K8GB 如何提供全局负载均衡**

K8GB 的设计非常复杂，但一旦你部署了应用程序并了解了 K8GB 如何维护区域文件，这个过程将变得更为简单。本节涉及 DNS 的一些基础知识，但我们将逐步解释如何使 K8GB 工作。

#### 保持 K8GB CoreDNS 服务器同步

首先要讨论的是 K8GB 如何通过同步多个区域文件来为我们的部署提供无缝故障转移。无缝故障转移是一种确保应用程序即使在系统出现问题或故障时仍能继续运行的过程，它通过自动切换到备用系统或资源，确保用户体验不中断。

正如我们之前提到的，每个集群中的 K8GB CoreDNS 服务器都必须在主 DNS 服务器中有一个入口。这是我们在 `values.yaml` 文件中为边缘 DNS 设置的配置：

```vbnet
edgeDNSZone: "foowidgets.k8s"
edgeDNSServer: "10.2.1.14"

```

因此，在边缘 DNS 服务器（`10.2.1.14`）中，我们有一个使用 K8GB 命名规范的每个 CoreDNS 服务器的主机记录：

```scss
gslb-ns-us-nyc-gb.gb.foowidgets.k8s    10.2.1.221  (纽约 CoreDNS 负载均衡器 IP)
gslb-ns-us-buf-gb.gb.foowidgets.k8s    10.2.1.224  (布法罗 CoreDNS 负载均衡器 IP)

```

K8GB 会在所有 CoreDNS 服务器之间进行通信，并在记录被添加、删除或更新时进行同步。

**示例说明**

假设我们在集群中部署了一个 NGINX Web 服务器，并在两个集群中创建了所有必需的对象。在部署之后，我们将在每个集群中拥有一个 `Gslb` 和一个 `Ingress` 对象，如下所示：

| 集群 | 部署 | Gslb | Ingress | NGINX Ingress IP |
| --- | --- | --- | --- | --- |
| 纽约 | nginx | gslb-failover-nyc | fe.gb.foowidgets.k8s | 10.2.1.98 |
| 布法罗（主集群） | nginx | gslb-failover-buf | fe.gb.foowidgets.k8s | 10.2.1.167 |

由于两个集群中的部署均正常运行，CoreDNS 服务器会将 `fe.gb.foowidgets.k8s` 记录解析为布法罗集群的 IP 地址 `10.2.1.167`。我们可以通过运行 `dig` 命令来验证此记录：

```null
dig fe.gb.foowidgets.k8s

```

输出如下：

```css
fe.gb.foowidgets.k8s.   30      IN      A       10.2.1.167

```

如果我们通过 `curl` 访问 DNS 名称，将会收到来自布法罗集群的 NGINX 服务器响应：

```css
curl fe.gb.foowidgets.k8s
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! This is a webserver in Buffalo for our K8GB example... </h1>

```

#### 模拟故障转移

我们将通过将布法罗集群中的部署副本缩放到 0 来模拟故障。当 K8GB 控制器发现没有健康的端点时，它会更新 CoreDNS 记录，并将服务故障转移到次要集群（纽约集群）。

缩放后，我们再次使用 `dig` 命令查看更新的主机：

```css
fe.gb.foowidgets.k8s.   27      IN      A       10.2.1.98

```

使用 `curl` 命令验证工作负载是否已经转移到纽约集群：

```css
curl fe.gb.foowidgets.k8s
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! This is a webserver in NYC for our K8GB example... </h1>
</html>

```

IP 地址现在返回的是 `10.2.1.98`，表明工作负载已经切换到纽约集群。

#### 恢复主集群

一旦主集群中的应用程序恢复健康，K8GB 将更新 CoreDNS 并将请求重新解析到主集群。我们通过将布法罗集群的部署缩放回 1 并再次运行 `dig` 命令来验证这一点：

```css
fe.gb.foowidgets.k8s.   30      IN      A       10.2.1.167

```

此时，`curl` 输出也显示工作负载重新回到了布法罗集群：

```css
curl fe.gb.foowidgets.k8s
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! This is a webserver in Buffalo for our K8GB example... </h1>
</html>

```

K8GB 是一个来自 CNCF 的开源项目，为 Kubernetes 提供类似于昂贵产品的全局负载均衡功能。随着项目的逐渐成熟，K8GB 有望成为多个集群应用的全局负载均衡解决方案。

**总结**
------

在本章中，你学习了如何为使用 LoadBalancer 服务的任何服务提供自动 DNS 注册。你还学习了如何使用 CNCF 项目 K8GB 部署高可用服务，K8GB 为 Kubernetes 集群提供全局负载均衡。

这些项目已经成为众多企业的核心部分，为用户提供了以前需要多个团队努力，并且通常需要大量文书工作的能力，以便向客户交付应用程序。现在，你的团队可以使用标准的敏捷开发实践快速部署和更新应用程序，从而为组织提供竞争优势。

在下一章《将身份验证集成到集群中》中，我们将探讨在 Kubernetes 中实施安全身份验证的最佳方法和实践。你将学习如何使用 OpenID Connect 协议集成企业身份验证，以及如何使用 Kubernetes 的身份冒充功能。此外，我们还将讨论在集群中管理凭据的挑战，并提供实际解决方案，以便对用户和管道进行身份验证。