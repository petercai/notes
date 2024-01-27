# 什么是云原生？ - Thoughtworks洞见
自从谷歌于2015年基于自己的Kubernetes容器编排开源项目，发起成立“云原生计算基金会”（Cloud Native Computing Foundation，CNCF）之后，“云原生”这一概念开始逐渐火热起来。

究竟什么是“云原生”？是谁最先提出来的？它和微服务、容器化、云计算、DevOps等等相关概念是什么关系？

在可以查阅到的文献中，“云原生”这一概念最早出现于Pivotal公司的Matt Stine在2015年所撰写的[“迁移到云原生应用架构”](https://learning.oreilly.com/library/view/migrating-to-cloud-native/9781492047605/)报告中。

这份报告总结了云原生应用程序架构的5条关键特征：

> 1.  [十二要素应用程序](https://12factor.net/)
> 2.  微服务
> 3.  敏捷的自助式基础设施
> 4.  基于API进行服务间协作
> 5.  反脆弱

时间一晃就过去了5年。这期间，Pivotal于2019年12月被VMware公司收购，而CNCF基金会基于Kubernetes所孵化的云原生开源项目，用[ThoughtWorks公司2020年5月发表的技术雷达第22期](https://assets.thoughtworks.com/assets/technology-radar-vol-22-cn.pdf)的话说，就是出现了寒武纪大爆发，出现了大量受到开发人员热捧的明星开源项目，Kubernetes生态系统欣欣向荣。

在2020年接近尾声的时候，让我们看看这两大意见领袖，对云原生的最新定义。

[VMware官网将云原生定义](https://tanzu.vmware.com/cloud-native) 为：

> 云原生是一种利用云计算软件交付模型的优势，来构建和运行应用程序的方法。当公司使用云原生架构构建和运行应用程序时，他们能更快地将新想法推向市场，并更快地响应客户需求。
> 
> 尽管每个行业对基础设施进行投资时都会考虑公有云，但基于云的软件交付并不仅限于公有云。云原生开发既适用于公有云，也适用于私有云。相比是在公有还是私有云上构建应用，云原生应用更关注如何创建和部署应用程序。
> 
> 云原生更重要的方面，是能够为开发人员提供能按需访问的计算能力，以及现代化的数据和应用程序服务的能力。云原生开发要与DevOps、持续交付、微服务和容器的概念相结合。

[由谷歌公司所主导的CNCF基金会在官网上展示了2018年6月11日所发布的“云原生”的定义](https://github.com/cncf/toc/blob/master/DEFINITION.md)为：

> 云原生技术使组织能够在现代化和动态的环境（例如公共云、私有云和混合云）中，构建和运行可进行容量伸缩的应用程序。其代表技术包括容器、服务网格、微服务、不可变基础设施和声明性API。
> 
> 这些技术能够构建富有韧性、便于管理和易于观测的松耦合系统。结合强大的自动化功能，这些技术能使工程师们可以轻松、频繁且可预测地对系统做出重大变更。

对比上面从2015年到2020年有关“云原生”的描述和定义，可以看出，“云原生”的定义，也是随着时代和技术的发展，不断演化。

对于国内进行云原生应用开发的企业，该如何把握不断演化的“云原生”的定义呢？

可以从3个方面来把握：

1.  关注意见领袖的最新定义。可以先从目前的“云原生”意见领袖CNCF基金会和VMware公司关注起
    
2.  将“云原生”的定义分为“价值”（why）、“目标”（what）和“方法”（how）这3个层次。我们不能为了“云原生”而“云原生”，而必须能让其满足企业自身的价值和目标
    
3.  对于“云原生”，找准企业自身的“价值”和“目标”，跟上“方法”的演化节奏。一旦找准了企业自身的“价值”和“目标”，那么这两者演化的节奏会较慢，但云原生实现“方法”的演化节奏会较快，需要设法跟上。
    

在参考了上面的定义和本文结尾所列6本书中的描述，现在以一个通常的云原生互联网应用为例，从“价值”、“目标”和“方法”这3个层次，尝试定义一下什么是2020年的云原生。

_价值（Why）_

1.  系统能长久稳定正常运行
2.  新业务能频繁发布
3.  能在任何时间和任何地点支持任何设备
4.  能支持物联网

_目标（What）_

1.  系统具有韧性，涉及冗余性和适应性
2.  系统具有敏捷性，涉及微服务化
3.  系统能处理容量大且波动大的请求和数据，涉及动态容量伸缩性(Scalability)和动态资源伸缩性(Elasticity)
4.  系统具有可观测性

_方法（How）_

1.  云计算
2.  敏捷、精益、DevOps、持续交付
3.  微服务
4.  容器集群编排
5.  服务网格：能让开发人员在编写应用程序时，不必实现可观测性、可靠性和安全性等非功能性需求，而将其转交给服务网格来实现
6.  不可变基础设施：用替换而不是修改的方式，来管理基础设施，以提高基础设施的一致性和可靠性
7.  声明式API：能避免开发人员重新发明轮子
8.  分布式系统数据一致性的CAP定理：当分布式系统出现网络分区故障时，如何在一致性和可用性上做出权衡
9.  [十二要素应用程序](https://12factor.net/)：构建现代化的、可容量伸缩及可维护的SaaS云应用的方法论
10.  混沌工程：在分布式系统上进行实验，以建立对该系统能够承受生产环境的动荡条件的信心

_可以参考近两年出版的有关云原生的6本书籍：_

1.  [Cloud Native: Using Containers, Functions, and Data to Build Next-Generation Applications 1st Edition, 2019.08](https://learning.oreilly.com/library/view/cloud-native/9781492053811/)
2.  [Cloud Native DevOps with Kubernetes: Building, Deploying, and Scaling Modern Applications in the Cloud 1st Edition, 2019.03](https://learning.oreilly.com/library/view/cloud-native-devops/9781492040750/)
3.  [Cloud Native Transformation: Practical Patterns for Innovation 1st Edition, 2019.12](https://learning.oreilly.com/library/view/cloud-native-transformation/9781492048893/)
4.  [Cloud Native Development Patterns and Best Practices: Practical architectural patterns for building modern, distributed cloud-native systems, 2018.02.09](https://learning.oreilly.com/library/view/cloud-native-development/9781788473927/)
5.  [Cloud Native Patterns: Designing change-tolerant software 1st Edition, 2019.05](https://learning.oreilly.com/library/view/cloud-native-patterns/9781617294297/)
6.  [What Is Cloud Native? 2019.04](https://learning.oreilly.com/library/view/what-is-cloud/9781492055136/)