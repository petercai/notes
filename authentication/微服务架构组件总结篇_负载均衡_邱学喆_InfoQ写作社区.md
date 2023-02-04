# 微服务架构组件总结篇_负载均衡_邱学喆_InfoQ写作社区
在早期介绍了有关 spring cloud 中的[注册中心](https://xie.infoq.cn/article/7271dcdd5c4651e3406373fde)，[配置中心](https://xie.infoq.cn/article/d96f6e5f0ebf993a374ba6020)，[负载均衡](https://xie.infoq.cn/article/2f71dcf2e202bf324f19ee195)，[熔断器](https://xie.infoq.cn/article/45b2c194f2286dc302b9c3383)等有关内容。借此机会，尝试从设计者的角度来总结这四个组件。

![](assets/d44b931c778359310c38eb43c75ab9f1.png)

2.1 功能
------

### 2.1.1 配置管理

既然是配置中心，首当其冲就是对配置内容进行管理。配置管理首要考虑的是数据模型以及存储方式；

*   数据模型——采用的是 Key-Value 的文本形式。
    
*   配置分类——对 Key-Value 在进行分类管理，至于 Value 是什么类型，取决于对配置内容的解析。
    

在业务系统中的常规配置分类如下；

![](assets/62c55b63b4fbda169fb920f5e00ef428.png)

在配置中心模型如下：

![](assets/35fc711233a2875df980694045a3c306.png)

*   作用域——在分类管理下，出现重复的 Key-Value 场景时，就需要确定其优先级——>全局，局部。
    
*   存储方式——有数据库，文件，redis 等形式多样的存储系统来存放配置内容。在 spring-cloud-config 中默认采用的是 git 来管理。
    

#### 2.1.1.1 多环境管理

由于不同的环境有不同的配置，那么在数据模型中支持多环境处理。如下图：

![](assets/6dc219d096183ed942c540dd56f3d7ea.png)

#### 2.1.1.2 版本管理 &权限管理

*   版本管理——可以看到配置的历史修改记录，可以在排查问题时用来追溯。
    
*   权限管理——可以避免被恶意的修改。
    

由于 spring-cloud-config-server 并没有提供把配置的维护交给 git 来操作了；所以，git 中已经有版本管理和权限管理等特性，所以 config-server 不需要实现该特性。

#### 2.1.1.3 协议

业务系统与配置中心的交互基于 TCP 交互，在 spring-cloud-config 是采用 http 协议进行交互；

其具体的请求是 GET 方式，格式如下：

```
{config-server-url}/{name}/{profile}/{label}
```

响应报文结构如下：

```
{
	"name": ${应用名称},
	"profiles": [
		${环境key}
	],
	"label": ${分支},
	"version": ${版本号},
	"state": ${状态值},
	"propertySources": [
		##配置内容
	]
}
```

#### 2.1.1.4 小结

在 spring-cloud-config 中，只提供了配置的拉取服务，配置管理默认是交给 git 来维护。当 config-server 会向 git 拉取对应的配置文件放到本地文件系统；

在 config-server 中，有关 git 的管理。config-server 支持多仓库、动态仓库等特性。

*   多仓库——通过模式匹配到一个动态仓库。
    
*   动态仓库——通过占位符的形式替换目标仓库
    

![](assets/d2f4560790a826a0140af515139a8c0f.png)

举个例子来说一下：

![](assets/4acb81d21f8a615bfe3cffcb564b5658.png)

当客户端系统的名称是 payment-_reconciliation，向服务端请求对应的配置时。服务端匹配到_第二个动态仓库，接着对 url 进行替换后，就可以得到 git 真实仓库地址：https://\*\*payment/\*\*/payment-reconciliation/{profiles}/{label}.git。至于其中 profiles 以及 label 也是同样得操作替换。如果没有这些占位符，则不用替换。

当拿到 git 真实仓库地址后，config-server 就会向 git 拉取工程文件到本地文件系统中。工程得层级目录五花八门，那么 config-server 是如何匹配对应得目录以及对应的文件的呢？

这里有这么一个属性 searchPath，通过该属性来匹配到对应的目录，也可以理解为该变量就是表明工程的层级目录结构。其逻辑较为简单，如下：

```
将searchPath中带有{application}、{profiles}、{label}字符串替换成对应的真实值
```

拿到目录后，就是访问该目录对应的文件，那么对应的文件规则如下：

```
{application}-{profiles}.(properties|yml)
```

经过上面的介绍，由此可以得知，config-server 对配置结构的管理，可谓是非常灵活。下面提供一下常规的结构，如下图：

![](assets/979f67b291b92af9936e3b6541cbad2f.png)

对应的配置，如下：

```
##服务端的配置如下
spring:
  application:
    name: config-server #设置名称
  cloud:
    config:
      server:
        git:
          repos:
            domain-A:
              name: domain-A
              pattern:
                - /domain-A*
              uri: https://gitee.com/******{application}.git
              force-pull: true #设置强行pull拉取
              searchPaths:
                - / ##可以不用配置，这个是默认
              username: ***
              password: ****
              basedir: ****
            domain-B:
              name: domain-B
              pattern:
                - /domain-B*
              uri: https://gitee.com/******{application}.git
              force-pull: true #设置强行pull拉取
              searchPaths:
                - /{application}/{profile}/
              username: ***
              password: ****
              basedir: ****
          uri: https://gitee.com/Michael_Chan/config-demo-application.git #设置git仓库地址
          force-pull: true #设置强行pull拉取
          searchPaths:
            - /{application}/{profile}/
          username: chendejin2008@163.com
          password: Gitee@Michael@410523
          basedir: D:\\config-server
##客户端的配置如下；
spring:
  profiles:
    active: sit
  application:
    name: *****
  cloud:
    config:
      uri:
       - config-server-url
```

思考问题：当数据模型如下图时，config-server 该如何配置，客户端才能红色圈出来的配置文件呢？

![](assets/c780496aae8a80e96a0fbf8681da7108.png)

### 2.1.2 动态更新

*   配置内容变更
    

当配置中心的配置内容发生了变更，对应的业务系统需要有所感知，从而刷新业务系统的本地配置。

在 spring-cloud-config 中，由于配置变更是在 git 上操作的，config-server 是无法感知的。所幸 git 有提供一个特性 webhook，当配置变更时可以回调指定的接口，从而告知 config-server 配置已经发生了变更；

问题由来了，config-client(也就是业务系统）是如何感知的呢？有三种方案，如下：

方案一：业务系统定时去拉取，当发现 config.client.state 值变了，意味着配置已经发生变更。这一块需要定制开发。主要是拉取 git 配置的时候 stage 始终为 null。

方案二：服务端发送请求来告知，意味着服务端需要存放所有业务系统的相关信息。这个方案对于 config-server 架构有点重，而且支持的 MQ 只有 kafka 以及 rabbitmq。不再其考虑范畴内；

方案三：通过 MQ 来监听。如下图。

![](assets/472a1ebf6821ee1c00637726238eca45.png)

个人比较倾向方案一。

基于方案一，梳理的交互如下：

![](assets/225ddc3e838bf434ea0234e9b3a200f0.png)

*   刷新配置——借助于 @RefreshScope 注解来实现，其实现原理是基于 spring-context 中的 scope 特性来实现。
    

### 2.1.3 加解密

当有一些敏感配置信息，例如密码等，如果直接在 git 上维护明文，容易被泄露。因此，spring-cloud-config 服务端提供了解密环节，也就是我们直接维护加密后的密文到 git 对应的工程，然后服务端进行解密。虽然用处不大，但有一定的防止敏感信息被泄露。解密的操作在 EnvironmentEncryptor 类，然后委托给 CipherEnvironmentEncryptor 进行解密；

2.2 客户端
-------

客户端的相对于较为简单；具体交互图如下：

![](assets/c4fcd64e678c2f1a140f884eb2709a31.png)

具体的配置如下：具体可以查看 ConfigClientProperties 类

```
spring:
  profiles:
    active: ${profiles}
  application:
    name: ${application-name}
  cloud:
    config:
      uri:
       - ${config-server-url}
```

2.3 服务端
-------

在功能小节说，对服务端介绍了差不多，这里只罗列时序图即可。如下；

![](assets/fd6cbfb57acd082aa46732c8d83f8217.png)

2.4 部署架构
--------

在早期文章也介绍了，个人倾向直接的单机部署，适当的定制开发，增加缓存机制。再加上系统运行监控即可。

![](assets/64d0cdae13b037be1329d7b0d99bf2f3.png)

3.1 集成方式
--------

当下游系统或者中间件有多台服务器时，意味着就需要一个模块，如何选择出合适的目标服务器去处理请求。那么这个模块具体落在哪一测，具体得根据

*   客户端路由
    
*   服务端路由
    
*   代理端路由，也可以叫做网关，也可以叫做企业总线
    

留下一个思考问题：在公司层面做架构规划时，该如何抉择呢？

3.2 负载算法分类
----------

*   OS 负载
    
*   TCP 负载
    
*   请求报文负载
    
*   IP 负载
    

3.3 代码
------

有关很多的细节，在早期的文章已经有所介绍，这里只罗列关键的接口之间的交互，如图所示：

![](assets/fc0150958d1e7141dc9a607b424dd0ea.png)

4.1 基础功能
--------

*   **服务注册**
    

当业务系统启动时，立即注册或者延迟注册当前实例信息到注册中心，注册中心接收到该请求，会将其保存起来。

*   **心跳检测**
    

当业务系统在运行期间，会不断的告知注册中心，业务系统正常运行着。避免注册中心认为该业务系统宕机，将其踢下线。这里面有两种实现方式，一种是客户端定时发送心跳信息，另外一种是服务端（注册中心）发送心跳请求给到客户端。由于后者方案要实现就必须依赖客户端提供服务接口，这样子对于客户端的侵入性太强，另外服务端的职责边界就有所大，同时也会增加服务端的压力，以及复杂度。所以，个人认为采用客户端主动发起心跳的方案较为友好。

*   **服务发现**
    

通俗的说，也叫拉取服务列表。既然注册中心记录着业务系统的相关服务信息，那么就会对应有拉取服务的接口，提供给客户端（业务系统）拉取服务。拉取服务列表，不会每次拉取全量的信息，这样子会导致网络拥堵。所以注册中心提供了两种方式，一种是全量拉取，另外一种是增量拉取。

4.2 特性
------

*   自我保护机制
    

由于网络的不稳定，大量的业务系统发送心跳的请求会出现短暂的超时，从而导致服务端认为其已经宕机，会将其踢下线。因此服务端为了这种情况，做了自我保护。逻辑如下：

![](assets/103ffce9d1c013e020ef190794bff466.png)

具体有关的阈值计算逻辑如下：

```
判断【最近一分钟接收到的心跳数】是否大于【自我保护阈值】，如果大于，则认为不是网络问题。
否则认为是网络的问题导致的。
1. 【自我保护阈值】-numberOfRenewsPerMinThreshold
自我保护阈值=【期望客户端实例数量(expectedNumberOfClientsSendingRenews)】
						*【每分钟心跳数量（可配置）】
            *【阈值（可配置）-renewalPercentThreshold】
其中客户端实例数量是动态变化的，例如新注册的实例，以及主动下线的实例。
一旦实例数量变更，就会重新计算【自我保护阈值】
当然还有服务端定时剔除过期的实例，也会导致实例数量。但其需要判断是否满足指定的条件，如果满足才能会去触发计算。
指定条件=【当前实例数量】 > 【期望客户端数量】*【阈值】

2. 【每分钟接收到的心跳数】
每次收到心跳请求，都会进行累加。
同时也会有定时任务，每隔一分钟就会将当前的心跳数保存起来后清零。
```

*   分类管理
    

在 eureka-server 中提出两个概念，一个 region，另外一个是 zone；region 所指定的范围大于 zone 的。可以通俗的理解为 region 是国家，zone 是省份；在计算领取，可以理解为 region 是地区，例如华东地区，华北地区等；而 zone 是华北地区的北京机房等；

然而服务实例并没有强制与 region，zone 绑定关系，需要额外配置当前实例属于哪个 region，zone 才行。这样子便于在负载时可以优先使用，例如：

```
eureka: 
	instance: 
  	metadataMap: 
    	region: North
      zone: bejing
```

4.3 业务系统
--------

### 4.3.1 服务消费者

消费者在启动时，可以采取立即拉取依赖的服务列表或者延迟拉取。这个是拉取动作是全量拉取。然而在 eureka 框架中，并不是只拉取当前业务系统自己依赖的服务列表，而是拉取注册中心的所有服务实例列表。

启动后，依赖的服务实例状态有可能会变更或者被踢下线，那么就需要后台任务来定期拉取。如果每次拉取全量的列表，无疑会导致网络拥堵，因此提供出增量拉取。

![](assets/67f984cdb0a66d56b95da5ec67cc3481.png)

有关拉取远程服务的逻辑，大体如下：

![](assets/1ab2c9d4ec36f582d85777bc11bffb4e.png)

所涉及的配置信息如下：

```
eureka: 	
	client: 
  	fetchRegistry: true //是否需要拉取注册中心的服务列表
  	registryFetchIntervalSeconds: 30 //拉取服务的频率-定时任务
    cacheRefreshExecutorThreadPoolSize: 2 //拉取服务的线程池-最大线程数
    cacheRefreshExecutorExponentialBackOffBound: 10 //规避倍数
    disableDelta： false //是否允许增量拉取服务列表
```

### 4.3.2 服务提供者

当作为服务的提供方，那么在服务启动时就需要向注册中心注册当前服务实例，从而被其他上游服务所感知到。

当运行时，需要不停向注册中心，当前服务实例还存活者，可以被调用。具体的流程图：

![](assets/996983649a2fe84947225b5d795ea873.png)

所涉及的配置信息如下：

```
eureka: 	
	instance: 
  	appname: //服务名称
  	appGroupName: //该服务实例所属组，可以在负载层面使用
    instanceEnabledOnit: true/false //指示是否注册后立即获取流量，也就是对应的状态为STARTING还是UP状态
    leaseRenewalIntervalInSeconds: 30 //发送心跳的频率，30秒一次
    leaseExpirationDurationInSeconds: 90 //
    metadataMap: //服务实例的额外的属性信息，可以用来给负载使用
    	key1: value1
      key2: value2
	client:   	
  	registerWithEureka: true //是否需要向注册中心注册当前服务实例
    heartbeatExecutorThreadPoolSize: 2 //心跳定时任务线程池-最大线程数
    //规避倍数，当发送心跳一直超时，就需要进行规避
    //发送心跳频率为30秒，一旦超时，就会延迟发送的周期；
    //呈2的倍数。具体代码逻辑在TimedSupervisorTask
    heartbeatExecutorExponentialBackOffBound: 10
```

### 4.3.3 网络传输

服务消费者以及服务提供者都需要与注册中心交互，首先就需要配置注册中心的服务地址信息，以及网络交互等其他细节控制，具体配置参数如下：

```
eureka: 
	client: 
  	region: // 所属地区
    availabilityZones: //有效的zone列表
   		region-1: zone-1, zone-2,....
      region-2: zone-11, zone-22,....
    serviceUrl： //zone对应的eureka地址信息
    	zone-1: url-1, url-2, ....
      zone-2: url-11, url-22, ...
  	//下面的是有关网络连接的配置信息
    eurekaServerReadTimeoutSeconds: 8 //读超时8秒
    eurekaServerConnectTimeoutSeconds: 5 //连接5秒
```

4.4 注册中心
--------

由于 eureka 采用的去中心化，也就是注册中心即是客户端也是服务端。有关客户端的可以查看上一节的内容；

### 4.4.1 自我保护

在特性中介绍了其逻辑，有关的配置事项如下：

```
eureka: 
	server: 
		enableSelfPreservation: true //自我保护机制是否开启
		renewalPercentThreshold: 0.85 //续约百分比阈值
    renewalThresholdUpdateIntervalMs: 15*60*1000 //每15分钟去检测是否服务提供者数量是否发生变化
    expectedClientRenewalIntervalSeconds: 30 //期望客户端续约（心跳）的频率
```

### 4.4.2 服务实例剔除

注册中心会定期将过期的服务提供者给剔除出去，逻辑在自我保护中有粗略介绍。这里罗列相关的可配置的参数：

```
eureka: 
	server: 
    evictionIntervalTimerInMs: 60*1000 //每1分钟触发服务剔除任务
```

### 4.4.3 服务列表拉取

提供给服务消费者拉取服务实例列表，这里有全量以及增量。

*   全量——拉取的逻辑如下图：
    

![](assets/4a2631d600fd74db1da592bb9f098406.png)

*   增量——大体逻辑跟全量所列的逻辑图差不多。稍微特殊，其数据来源是来自 recentlyChangedQueue。
    

虽然服务实例信息也是采用内存存放，但获取服务列表时还要经过一层数据转换，所以增加缓存。

如果开启了一级缓存，也就是 readOnlyCacheMap 属性。这一块会起一个定时任务，定期会将二级缓存（readWriteCacheMap 属性）的内容同步到一级缓存中去。

增量的数据来源是 recentlyChangedQueue 集合，这个数据是一直在追加。容易导致信息堆积，这个时候就需要定时去清理这个数据；清理的也较为简单，只要找过指定的时间就将其丢弃调；

这里面所涉及到的可配置的信息如下：

```
eureka: 
	server: 
  	disableDelta: false //是否禁止增量拉取
  	useReadOnlyResponseCache: true //是否开启一级缓存
    disableTransparentFallbackToOtherRegion: false //是否允许从其他region区拉取服务列表
    responseCacheUpdateIntervalMs: 30 * 1000 //缓存更新频率
    initialCapacityOfResponseCache: 1000 //缓存初始化容量
    responseCacheAutoExpirationInSeconds: 180 //缓存过指定时间后自动过期
    deltaRetentionTimerIntervalInMs: 30 * 1000 // 增量数据清理任务执行频率
    retentionTimeInMSInDeltaQueue: 3*60*1000 //超过这个时间的数据进行清理
```

### 4.4.4 数据同步

为了高可用，需要多台注册中心来配合。这个时候就需要数据同步的操作；

同步到集群中的其他目标节点，是复用业务系统配置的配置信息，如下：

```
eureka: 
	client: 
  	region: // 所属地区
    availabilityZones: //有效的zone列表
   		region-1: zone-1, zone-2,....
      region-2: zone-11, zone-22,....
    serviceUrl： //zone对应的eureka地址信息
    	zone-1: url-1, url-2, ....
      zone-2: url-11, url-22, ...
```

数据同步动作是采用异步的，但其实现方式有两种，如下：

*   批处理器——每个目标节点默认有 1 个 Acceptor 线程+20 个 Process 线程。具体的逻辑如下：
    

![](assets/2930a1c5d8ae8daa413fb5866a6c2b9d.png)

*   单处理器——每个目标节点默认有 1 个 Acceptor 线程+1 个 Process 线程。逻辑与批处理有所类似。
    

之所以需要同时出现这两种，是为了区别对待，也可以理解优先级的问题，优先级高的交给单处理器，优先级低的交给批处理器。也就是人工手动修改服务实例状态（statusUpdate）的，则优先处理，那么其就会交给单处理器来处理。

所涉及的可配置项如下：

```
eureka: 
	server: 
  	//可等待同步的时间，一旦超时这个时间，就会将其丢掉，不再同步给其他节点
  	maxTimeForReplication: 30000 
    maxElementsInPeerReplicationPool: 10000 //批处理器的acceptor的最大数量
    maxThreadsForPeerReplication: 20 //批处理器的process线程数量
    maxElementsInStatusReplicationPool: 10000 //单处理器的最大数量
    maxThreadsForStatusReplication: 1 //单处理器的process线程数量
    //连接的相关配置
    peerNodeConnectTimeoutMs: 200
    peerNodeReadTimeoutMs: 200
    peerNodeTotalConnections: 1000
    peerNodeTotalConnectionsPerHost: 500
    peerNodeConnectionIdleTimeoutSeconds: 30
```

*   目标节点动态更新
    

如果目标节点列表发生了变更，eureka 支持动态更新。那么就需要一个定时任务来检测并替换操作；具体的代码在 PeerEurekaNodes 类，所涉及可配置项如下：

```
eureka: 
	server: 
  	peerEurekaNodesUpdateIntervalMs: 10*60*1000 //10分钟检测目标节点列表是否发生变更
```

### 4.4.5 限流

为了避免注册中心的压力过大，提供出了限流处理。其采用的令牌的实现方案，具体逻辑可以查看 RateLimiter 类

```
eureka: 
	server: 
  	rateLimiterThrottleStandardClients: false
    rateLimiterPrivilegedClients: 
    	- client_name1
      - client_name2
    rateLimiterBurstSize: 10
    rateLimiterRegistryFetchAverageRate: 500 
    rateLimiterFullFetchAverageRate: 100
```

### 4.4.6 部署架构

在 CAP 模式中，之所以采用 AP 模式，个人感觉是出现短暂的数据不一致性对负载策略的使用并没有造成很大影响，而可用性以及分区容错性影响较低。而且当网络恢复后，数据会最终一致。

当下游系统出现超时或者不可用时，避免出现服务雪崩的问题。很多细节已经在《[Spring cloud 之 CircuitBreaker 篇](https://xie.infoq.cn/article/45b2c194f2286dc302b9c3383)》有所阐述。稍微补充一下有关配置的介绍。

```
# 配置可以去查看CircuitBreakerProperties类
resilience4j: 
	circuitbreaker: 
  	instances: 
    	instanceName: 
      	#可以查看早期的文章介绍
		instanceName2: 
    	.....
	configs: 
  	config-name: 
    	......
```

通过再次的总结输出，也纠正了早期的对其的认知，对其四大组件的认知也更加深入。上面介绍的主要是关键特性，并没有对其额外的特性进行介绍，例如注册中心的 aws，DNS 等的使用。随着对代码的深入了解，发现其很多写法不是那么好友，不像 spring framework 那么纯粹，但其思想是具有参考价值的，可以通过对其了解，重复造更加轻量，更加灵活的轮子。一般来讲，中小型企业，直接使用就好，毕竟没有那么多的人力去重复造轮子，而针对大型企业来说，建议是基于 spring framework 框架，参考 spring boot, spring cloud 的思想，创建自己公司层面的技术框架。

![](assets/bfd95f7148d54c25bc0173e57b4e0efc.png)