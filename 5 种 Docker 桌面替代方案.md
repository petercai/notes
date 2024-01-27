# 5 种 Docker 桌面替代方案
对于 Windows 和 macOS 用户，Docker Desktop 多年来一直是使用 Docker 容器的主要方式。虽然对于业余爱好者和小型开发团队来说，它仍然是一个可行且可用的选择，但最近针对更大用户群的价格变化促使人们寻找替代品。我并不想自己取代 Docker Desktop，但我有兴趣尝试替代方案并看看它们之间的比较。

容器本身并不是一个新的技术概念，但早在 2000 年代中期，Docker 就普及了这个概念，并在正确的时间以正确的方式进行营销，使这个概念成为主流。

将 Docker 公司与 Docker 项目分开是值得的，因为它们很容易混淆，而且它们是不同的实体。由于这种混乱，Docker 公司重新命名并开源了许多与容器相关的技术，为现在所谓的“[开放容器倡议](https://opencontainers.org/?ref=hackernoon.com)”（OCI）做出了贡献。

我在这里进行了很多抽象和总结，但是当本文的其余部分提到“OCI 兼容”容器和类似术语时，请将其视为类似于您可能认为的“Docker 容器”。所有这些事件和变化实际上都发生在技术时代的某个时间之前，但它仍然是一个持续的混乱源。 Tldr……这篇文章中的所有选项都可以运行相同的容器定义，这包括来自 Docker Desktop 或使用_Dockerfile_创建的预先存在的容器。另外请注意，项目通常将使用 Docker 运行的容器互换称为“dockerd”和“Moby”。

1.豆豆人
-----

[Podman](https://podman.io/?ref=hackernoon.com)可能是最受欢迎的替代方案，它有许多来自 Red Hat 的贡献者，而且 Red Hat 似乎正在计划 Podman 的商业版本，因此可以安全地说这是一个“Red Hat 项目”。

它适用于 Windows、macOS 和 Linux，并且与此处介绍的许多其他工具一样，遵循与 Docker 类似的语法，但有两个注意事项：

1.  默认情况下，您使用`podman`而不是 Docker，但您可以创建一个别名而忘记该命令更改。
2.  默认情况下，Docker Desktop 假设您想要使用来自 Docker hub 的容器镜像，而所有其他替代方案都可以理解地不做这样的假设。这意味着您需要指定您可能经常使用的许多图像的完整路径，例如“ [docker.io/library/busybox](http://docker.io/library/busybox) ”。

Podman 与此处介绍的其他替代方案（包括 Docker Desktop）之间的主要区别之一是它是无守护进程的。这意味着每个正在运行的容器都作为自己的运行时进程运行，而不是通过一个守护进程。如果 Docker 守护进程失败，所有正在运行的容器都会失败，而使用 Podman，只有单个容器会失败。也就是说，我个人从未让 Docker 守护程序失败，但我没有运行生产工作负载。

与这里的所有其他工具一样，在首次运行时，Podman 需要在 macOS 和 Windows 上创建一个虚拟机来托管容器。在 macOS 和 Windows 上，这并不总是必要的，但为了最大的跨平台（和体系结构）兼容性，这是有意义的。 Podman 使用 Fedora CoreOS（又是那个 red Hat 连接）和 QEMU 来运行 VM。

### Podman 桌面

Podman 的图形伴侣是[Podman Desktop](https://podman-desktop.io/?ref=hackernoon.com) ，但理论上它应该列出由其他运行时公司创建的图像和容器。与许多其他图形工具一样，它还添加了与 Kubernetes 交互的功能，但我将在以后的文章中介绍这些功能。它提供了可能与 Docker Desktop 相同的功能，包括我没有意识到遵循任何标准的功能，例如扩展，但不够完善，并且缺少 Docker Desktop 提供的一些特定于操作系统的功能。

2.科利马州
------

[Colima](https://github.com/abiosoft/colima?ref=hackernoon.com)仅适用于 Linux 和 macOS，它使用[Lima](https://github.com/lima-vm/lima?ref=hackernoon.com)在 macOS 上启用 Linux VM。它支持 Docker、Containerd 和 Kubernetes 运行时，在所有情况下，您都需要与 Colima 一起安装该运行时。对于 macOS 上的 Docker，这与 Docker Desktop 并不完全相同。

尽管 Colima 使用起来很简单，但您仍然需要安装运行时这一事实让我想知道“Colima 是什么？”，老实说，最少的文档并没有使它变得更清楚。标语是“具有最少设置的 macOS（和 Linux）上的容器运行时”，但这仍然没有让我明白为什么我需要它。据我所知，主要原因是使用 containerd 或作为 Kubernetes 后端（而不是 Docker Dkestop、minikube 等），也许支持 Docker 的主要原因是向后兼容性。

3.Rancher桌面
-----------

虽然它主要将自己标榜为 Kubernetes 管理工具，但[Rancher Desktop](https://rancherdesktop.io/?ref=hackernoon.com)还在 Kubernetes 之外提供了一些容器管理功能。它支持使用 containerd 或 Docker 运行的容器，并提供与此列表中其他图形工具大部分相同的功能。同样，QEMU 提供了一切都在其上运行的 VM，没有选项可以更改它。这是一个非常不错的工具，也是更成熟的替代品之一，但我发现它需要在您进行更改时多次重启和重置 VM，这有点乏味。

4\. VMWare 融合
-------------

如果您已经使用 VMWare Fusion 来运行 Windows 和 Linux VM，[它也支持容器](https://blogs.vmware.com/teamfusion/2020/05/fusion-11-5-now-supports-containers.html?ref=hackernoon.com)。然而，该功能目前仅适用于基于 Intel 的 Mac，尽管安装程序仍在安装 CLI 工具并给你错误的希望。

5.平行线
-----

同样，如果您已经拥有并使用适用于 Linux 和 Windows 虚拟机的[Parallels Desktop](https://www.parallels.com/au/?ref=hackernoon.com) ，那么您可以将它用作 minikube 的后端，它主要针对 Kubernetes 使用，但它与容器相邻，所以我将它作为一种选项包含在内.

