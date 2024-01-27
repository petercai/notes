# Google Kubernetes Engine(GKE) 简单使用
**Google 的 k8s 在 2017 年已经从容器编排领域的竞争中取得主导地位，从 Docker 之前的一度排挤到最终完全拥抱 k8s，显然 k8s 已经成了目前业界的标准。** 

![](_assets/61cc6c022ab3f51d910e58bb.jpg)

但是到目前为止能提供 k8s 完全托管服务的云服务商少之又少，即便是目前在云提供商有统治力的 AWS 也没有完全提供 k8s 托管服务，仅仅提供有限的定制服务，在这一方面并不成熟。

然而 Google 的 k8s 托管服务，即 GKE，却将 k8s 托管服务做到了极致（至少目前看来），不仅提供了全套的 k8s 托管服务，更引人注目的是 Google 已然将 Autoscaler 和 k8s 集成，实现了 k8s 节点的自动伸缩机制，能根据 pod 的需求自动化添加或删除节点，当现有节点无法承载新的服务时会自动添加节点来满足需求，当现有节点足够空闲时会启用调节机制自动化收缩节点，从某种意义上来说这几乎做到了无服务器的理念。

然而这也许只是冰山一角，更多强大的功能还需要进一步探索，本文只是一个入门指南，主要指导能快速开始上手基于 Google Cloud Platform 的 GKE 服务（k8s 托管服务）。

### GKE 入门指南

接下来我们一步步指引如何使用 GKE 来部署服务，前提是对 k8s 有所了解，能简单使用 kubectl [命令](https://www.linuxcool.com/)。

#### 1\. 安装并配置 Google Cloud SDK

Google Cloud SDK 是 访问 GCP（Google Cloud Platform）平台各种资源的[命令](https://www.linuxcool.com/)行工具集，类似 aws 的 aws 命令行工具。

安装和配置就不多说了，点击下面链接选择相应操作系统版本的 tar 包下载，然后解压，在 PATH 环境变量中添加 google-cloud-sdk/bin 即可

#### 2\. 初始化 Google Cloud SDK

初始化 Google Cloud SDK 是将 gcloud 命令和 Google 账号绑定起来并设置一些其他的默认值，比如区域，代理，账号，项目（Google 账号中新建的项目）之类的。

在执行 gcloud init 初始化之前得先给 gcloud 配置 HTTP 代理（GFW 你懂得），具体配置见我之前这篇文章。然后执行 gcloud init 完成初始化，直接根据向导来即可。

#### 3\. 到 Google Cloud Platform 控制台建一个 k8s 集群，记住名称

![](_assets/61cc6c022ab3f51d910e58e7.png)

初识Google Kubernetes Engine（GKE）初识Google Kubernetes Engine（GKE）

#### 4\. 安装 gcloud kubectl 组件

```null
gcloud components install kubectl

```

#### 5\. 获取群集的身份验证凭据

创建群集后，您需要获取身份验证凭据以与群集进行交互。要为集群进行身份验证，请运行以下命令：

```null
gcloud container clusters get-credentials 

```

#### 6\. 接下来部署一个简单的 hello-server 服务到 GKE

```null
kubectl run hello-server --image gcr.io/google-samples/hello-app:1.0 --port 8080

```

### 相关链接

```null
https://cloud.google.com/kubernetes-engine/docs/quickstart
https://cloud.google.com/sdk/docs/quickstart-macos?hl=zh-cn

```

### 附录

#### gloud 常用命令

```null
gcloud auth login --no-launch-browser 
gcloud config set compute/zone [COMPUTE_ZONE] 
gcloud components list 
gcloud components install [组件名称] 
gcloud components update  
gcloud components remove [组件名称] 

```

#### 设置 gcloud http 代理

```null
gcloud config set proxy/type http
gcloud config set proxy/address 127.0.0.1
gcloud config set proxy/port 1087

```

#### 设置集群 docker 私服认证

```null
kubectl create secret docker-registry regcred --docker-server= --docker-username= --docker-password= --docker-email=

```

**注意**：设置 docker 私服后，要在 GKE 部署 k8s 服务，必须得在 k8s 资源文件（yaml 格式）中的 container 同一级指定 imagePullSecrets 键，要不然仍然无法拉取配置的私服的镜像，示例资源文件如下：

```null
apiVersion: v1
kind: Pod
metadata:
 name: private-reg
spec:
 containers:
 - name: private-reg-container
   image:
 imagePullSecrets:
 - name: regcred

```

#### 查看集群 docker 私服配置

```null
kubectl get secret regcred --output=yaml      
kubectl get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 -d 

```

以上就是[良许教程网](https://www.lxlinux.net/)为各位朋友分享的[Linu系统](https://www.lxlinux.net/zhuanti-full)相关内容。想要了解更多Linux相关知识记得关注公众号“良许Linux”，或扫描下方二维码进行关注，更多[干货](https://www.lxlinux.net/category/knowledge-collection)等着你 ！

![](_assets/61bda32b2ab3f51d910ba3f5.jpg)

本文由 [良许Linux教程网](https://www.lxlinux.net/) 发布，可自由转载、引用，但需署名作者且注明文章出处。如转载至微信公众号，请在文末添加作者公众号二维码。