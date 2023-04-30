# GKE教程：Google Kubernetes Engine入门
[Kubernetes](https://kubernetes.io/)将拯救我们所有人。 如果只有我们可以弄清楚如何安装和维护它。 对于刚起步的人来说，[Kubernetes](https://so.csdn.net/so/search?q=Kubernetes&spm=1001.2101.3001.7020)（在您当地的社区时髦开发人员中也称为K8）是一个开源平台，用于运行和编排基于容器的应用程序和服务。 这些通常部署在Docker容器中，但支持其他容器运行时（例如[Containerd](https://kubernetes.io/blog/2018/05/24/kubernetes-containerd-integration-goes-ga/)和[Rkt](https://github.com/rkt/rkt/blob/v1.29.0/Documentation/using-rkt-with-kubernetes.md) ）。

在过去的十五年中，Google积累了有关在其操作中运行容器的大量知识。 Kubernetes代表了Google的第三代容器管理系统，仅次于[Borg](https://ai.google/research/pubs/pub43438)和[Omega](https://ai.google/research/pubs/pub41684) ，并且在过去几年中已成为主要的容器平台，超越了其他产品，如[Mesos](http://mesos.apache.org/)和[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)的[Swarm](https://docs.docker.com/engine/swarm/) 。 对于企业而言，Kubernetes提供了一个接近圣杯的东西：“如果使用[OpenStack](https://www.openstack.org/) ，但它确实有效呢？”

但是，许多踏足Kubernetes的人发现自己有点不知所措。 这是一个由许多活动部件组成的系统，安装，维护和操作起来可能有些困难。 但是，如果您无论如何都在云上运行，为什么不让您的云提供商为您运行Kubernetes，这样您就可以继续运行应用程序和工作负载这一更重要的业务了？

自2015年以来一直使用Google Kubernetes Engine（GKE）服务，您的Kubernetes主服务器由Google网站可靠性工程师管理和维护。 您无需安装，升级甚至无需付费！ 此外，您的Kubernetes集群可以分布在一个区域内的各个区域中，以实现高可用性，当然，它们还与所需的企业功能集成在一起，例如出于安全目的的Cloud IAM（身份访问管理）和VPC（虚拟私有云）。

创建一个Kubernetes集群
----------------

我们来看一个例子，对吧？ 我们将使用GKE建立一个新集群并以Google的hello-app Docker容器为例。 首先，您需要一个Google Cloud Platform帐户以及[gcloud](https://cloud.google.com/sdk/gcloud/)命令实用程序。 您还需要在系统上复制[kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)才能与我们正在创建的集群进行交互。 尽管此小教程会产生费用，但如果您刚刚创建了一个新的Google Cloud帐户，则我们的活动很乐意满足您的初始免费信用额度，前提是您随后删除集群。

我们将授权gcloud使用我们的帐户（它将打开一个网页以获取凭据）并在us-central1-a中创建一个新项目（根据距离您最近的[区域进行](https://cloud.google.com/compute/docs/regions-zones/)调整）。

```null
 auth logingcloud projects create kubernetes-testgcloud config set compute/zone us-central1-a
```

完成之后，让我们疯狂地创建一个Kubernetes集群：

```null
gcloud containers clusters create myfirstcluster
```

就是这样。 晚安各位！

好吧，显然您想要的不止于此。 但是，如果您曾经参与过从头开始建立Kubernetes集群的过程，那么您会知道在那条线的幕后发生了很多事情，为您节省了数小时甚至数天的时间。 集群很快就准备就绪。

默认情况下，gcloud创建一个三节点集群，每个节点都是一个n1-standard-1实例（您可以使用`--machine-type parameter`更改实例选择）。 也许不是大多数生产工作负载所需要的，但是对于少量测试集群而言却是完美的选择。

注册凭据以使用kubectl
--------------

接下来，我们将使gcloud注册集群的凭据以使用kubectl，然后获取有关集群的一些信息：

```null
gcloud container clusters get-credentials myfirstclusterkubectl cluster-info
```

这应该给你这样的东西：

```null
etes master is running at https://35.192.176.225GLBCDefaultBackend is running at https://35.192.176.225/api/v1/namespaces/kube-system/services/default-http-backend:http/proxyHeapster is running at https://35.192.176.225/api/v1/namespaces/kube-system/services/heapster/proxyKubeDNS is running at https://35.192.176.225/api/v1/namespaces/kube-system/services/kube-dns:dns/proxykubernetes-dashboard is running at https://35.192.176.225/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
```

向我们的Kubernetes集群添加服务
--------------------

我们有一个Kubernetes集群正在运行，但是目前并没有做很多事情。 我们可以通过向集群添加新服务来更改它。 我们将以Google的hello-app容器为例。 使用以下命令，我们在端口8080上将服务hello-world与1.0版本的hello-app添加在一起：

```null
l run hello-world --image gcr.io/google-samples/hello-app:1.0 --port 8080
```

然后， `kubectl get deployments`应该向您显示hello-world正在运行。 但是我们想谈谈它，因此让我们使用负载平衡器将其暴露给外界：

```null
l expose deployment hello-world --type "LoadBalancer"
```

我们可以使用以下方法获取负载均衡器IP：

```null
kubectl get service hello-server
```

您可能需要等待一分钟左右，负载均衡器才能启动。 请注意，此负载平衡器是标准的Google负载平衡器，会在您的Google帐户上产生费用，不过，在此简短示例中，这些费用最多为几美分。 可见外部IP后，您可以转到浏览器中的地址或使用curl：

```null
5.184.69.149:8080Hello, world!Version: 1.0.0
```

为我们的Kubernetes服务创建自动缩放器
-----------------------

然后，我们可以设置一些扩展策略。 如何在集群中运行10到20个副本，将CPU利用率提高50％？ 简单！

```null
l autoscale deployment hello-world --max 20 --min 10 --cpu-percent 50
```

您可以运行`kubectl get pod`来查看GKE为满足新的自动缩放策略而旋转的所有各种`kubectl get pod` 。 （在Kubernetes _分离舱_是一组一个或多个集装箱的，它是计算，该系统涉及。的最小单位）在几秒钟内，你会看到所有的10个荚高兴地运行。

最后，让我们做一些非常酷的事情（至少如果您花了一些时间在运营渠道中）。 GKE提供了对节点自动修复的支持，允许Kubernetes响应集群中节点的问题。 让我们为测试集群启用此功能：

```null
gcloud container node-pools update default-pool 
```

接下来， `run kubectl get nodes` 。 一切看起来都应该很好，而且开心，到处都有“就绪”状态。 让我们改变它。 进入Google Cloud Dashboard，然后执行大多数操作员最讨厌的事情：从集群中随机删除VM。 还是要小心风！ 删除两个，甚至全部三个！

删除尽可能多的VM之后，请继续运行`kubectl get nodes`和`kubectl get pods` 。 您应该看到删除的所有节点都标记为“未就绪”，并且某些Pod可能已脱机（因为它们正在该节点上运行）。 但是，如果您继续观察，将会看到群集反弹。 首先，节点将返回（您可以在仪表板中进行验证），然后容器也将返回。

GKE的特点和优势
---------

除了节点自动修复之外，在GKE中运行Kubernetes集群还有其他好处。 节点自动升级可让您的节点自动升级，因此您知道自己始终在运行GKE团队推荐的最安全，最稳定的Kubernetes版本，而无需自己做。

默认情况下，群集中的每个节点都运行[Container-Optimized OS](https://cloud.google.com/container-optimized-os/) ，这是Google自己定制，精简和强化的Linux版本。 虽然您可以使用Ubuntu，但是运行[Container](https://so.csdn.net/so/search?q=Container&spm=1001.2101.3001.7020)-Optimized OS可以使您处于由Google自己维护的安全基础上。

另一个有用的功能是能够利用Google广泛的网络功能。 您只需花费很少的精力，即可在全球范围内设置Kubernetes集群并配置负载平衡，以便当用户访问您的一项服务时，它们将被定向到地理位置最接近它们的集群。

GKE还与Google的Stackdriver产品集成，可轻松处理集群的日志记录和监视。 Stackdriver的一大特色是它与平台无关，因此，如果您在本地或AWS上拥有Kubernetes集群，那么您也可以将其与Stackdriver集成。 使用Stackdriver，您可以深入研究服务，工作负载，Pod，甚至深入到容器级别，以洞悉诸如延迟，故障率以及其他对企业状况重要的指标。

最后，如您所料，GKE与其他Google Cloud Platform服务无缝集成，因此，如果您要创建在可抢占式VM上运行并可以访问高端GPU的节点池，那就去吧！

托管Kubernetes的领跑者
----------------

鉴于Kubernetes'谷歌血统，并在Azure上Kubernetes服务或亚马逊EKS GKE的三年先声夺人，它不是完全不足为奇了GKE是领先于其他主要管理Kubernetes产品**I** N为Kubernetes支持方面的特点。 （什么_是_也许令人惊讶的是Azure的Kubernetes支持可以说是超前亚马逊的，但那是另一次的故事。）

Kubernetes的巨大希望之一就是，它使我们更进一步地迈向了一个世界，在这个世界上，您可以以最小的努力将工作负载从一个云提供商转移到另一个云提供商。 不过，就目前而言，在GKE上运行是搭乘Kubernetes火车最简单的方法之一。

From: [https://www.infoworld.com/article/3292637/gke-tutorial-get-started-with-google-kubernetes-engine.html](https://www.infoworld.com/article/3292637/gke-tutorial-get-started-with-google-kubernetes-engine.html)