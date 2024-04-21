# Flowable工作流引擎概要 
一、**什么是工作流？ \- 工作流解决了什么问题**
---------------------------

工作流，是把业务之间的各个步骤以及规则进行抽象和概括性的描述。使用特定的语言为业务流程建模，让其运行在计算机上，并让计算机进行计算和推动。

**工作流是复杂版本的状态机。** 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8a4b39d6adb42468eee07cc966c070c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

上图为工作流退化为基础状态机的例子，小明的状态非常简单，站立->走路->跑步->走路->站立，无限循环，如果让我们实现小明的状态切换，那么我们只需要用一个字段来记录小明当前的状态就好了。

而对于复杂的状态或者状态维度增加且状态流转的条件极为复杂，可能单纯用字段记录状态的实现方式就会不那么理想。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b21bb2b682d44b2c88890f5673856acd~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

如上图所示，现在交给小明的选择就多了起来，当小明获得金钱的时候，他会根据金钱数额的大小来判断接下来该如何行动，如果数额小于等于3000，那么他决定买一个新手机就够了，如果数额小于等于30万，那么小明就决定去学习一下理财，好好利用这笔钱，但如果小明获得的金钱数量超过了30万，他就决定购置一套房屋，但购置房屋的流程是复杂的，小明决定同时完成交首付和贷款的手续。

其实这个流程还不算特别复杂，但到目前为止，单纯用一个字段来表明状态已经无法满足要求了。

工作流解决的痛点在于，解除业务宏观流程和微观逻辑的耦合，让熟悉宏观业务流程的人去制定整套流转逻辑，而让专业的人只需要关心他们应当关心的流程节点，就好比大家要一起修建一座超级体育场，路人甲只需要关心他身边的这一堆砖是怎么堆砌而非整座建筑。

**那么工作流有什么不能解决的问题呢？**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf85a4c4a67248e9ba2d14cc11221c1a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

工作流是一个固定好的框架，大家就按照这个框架来执行流程就行了，但某些情况他本身没有流转顺序的要求，比如：小明每天需要做作业，做运动以及玩游戏，它们之间没有关联性无法建立流程，但可以根据每一项完成的状态决定今天的任务是否完结，这种情况我们需要使用CMMN来建模，它就是专门针对这种情况而设计的，但今天我们不讲这个，而是讲讲BPMN协议。

二、**BPMN2.0协议**
---------------

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3776d47d4fad48bcb2cd2b2423c556e8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

对于业务建模，我们需要一种通用的语言来描绘，这样在沟通上和实现上会降低难度，就像中文、英文一样，BPMN2.0便是一种国际通用的建模语言，他能让自然人轻松阅读，更能被计算机所解析。

协议中元素的主要分类为，事件-任务-连线-网关。

一个流程必须包含一个事件（如：开始事件）和至少一个结束（事件）。其中网关的作用是流程流转逻辑的控制。任务则分很多类型，他们各司其职，所有节点均由连线联系起来。

下面我就以每种类型的节点简单地概括一下其作用。

**网关：** 

* * *

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0406475f0aa42368fd5dca4ba8a8333~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
 **互斥网关（Exclusive Gateway）** ，又称排他网关，他有且仅有一个有效出口，可以理解为if......else if...... else if......else，就和我们平时写代码的一样。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f22ef1584ce472085bcb1b20eeeec87~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
 **并行网关（Parallel Gateway）** ，他的所有出口都会被执行，可以理解为开多线程同时执行多个任务。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/305e0ccb4e2c4fb1aebb5138b4341746~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
 **包容性网关（Inclusive Gateway）** ，只要满足条件的出口都会执行，可以理解为 if(......) do, if (......) do, if (......) do，所有的条件判断都是同级别的。

**任务：** 

* * *

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43520efd93234632a09a7dd67a5a55b4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

BPMN2.0协议的所有任务其实是从一个抽象任务派生而来的，抽象任务会有如下行为：

1. 当流程流转到该任务时，应该做些什么？

2. 当该任务获得信号(signal)的时候，它是否可以继续向下流转，而任务获得信号的这个动作我们称为Trigger。

利用如上的抽象行为，我们来解释一些比较常见且具有代表性的任务类型。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5bec0ac20774a0ca9da48263df585d6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
 **人工任务（User Task）** ，它是使用得做多的一种任务类型，他自带有一些人工任务的变量，例如签收人（Assignee），签收人就代表该任务交由谁处理，我们也可以通过某个特定或一系列特定的签收人来查找待办任务。利用上面的行为解释便是，当到达User Task节点的时候，节点设置Assignee变量或等待设置Assignee变量，当任务被完成的时候，我们使用Trigger来要求流程引擎退出该任务，继续流转。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/157cd77f0cfb496cb6b01b86b1085952~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
 **服务任务（Service Task），** 该任务会在到达的时候执行一段自动的逻辑并自动流转。从“到达自动执行一段逻辑”这里我们就可以发现，服务任务的想象空间就可以非常大，我们可以执行一段计算，执行发送邮件，执行RPC调用，而使用最广泛的则为HTTP调用，因为HTTP是使用最广泛的协议之一，它可以解决大部分第三方调用问题，在我们的使用中，HTTP服务任务也被我们单独剥离出来作为一个特殊任务节点。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ace095e7c80f4f799ed2dc571c8c9d52~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
 **接受任务（Receive Task）** ，该任务的名字让人费解，但它又是最简单的一种任务，当该任务到达的时候，它不做任何逻辑，而是被动地等待Trigger，它的适用场景往往是一些不明确的阻塞，比如：一个复杂的计算需要等待很多条件，这些条件是需要人为来判断是否可以执行，而不是直接执行，这个时候，工作人员如果判断可以继续了，那么就Trigger一下使其流转。

  ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d8ae64776194f6780b09fab8c3e9329~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
 **调用活动（Call Activity）** ，调用活动可以理解为函数调用，它会引用另外一个流程使之作为子流程运行，调用活动跟函数调用的功能一样，使流程模块化，增加复用的可能性。

上面大概介绍了一下常用的节点，下面的图就展示了一个以BPMN2.0为基础的流程模型，尽量覆盖到所介绍的所有节点。

  ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/acf1d1cf7f6c4b8c9ec9c0f006a0eef2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
  

这里是一个生产汽车的流程，从“汽车设计”节点到“批准生产”节点是一个串行的任务，而审批的结果会遇到一个互斥网关，上面讲过，互斥网关只需要满足其中一个条件就会流转，而这里表达的意义就是审批是否通过。“载入图纸”是一个服务任务，它是自动执行的，之后会卡在“等待原材料”这个节点，因为这个节点是需要人为去判断（比如原材料涨价，原材料不足等因素），所以需要在一种自定义的条件下Trigger，而该图的条件应该为“原材料足够”，原材料足够之后，我们会开始并行生产汽车零件。

需要注意的是，并行网关在图中是成对出现的，他的作用是开始一系列并行任务和等待并行任务一起完成，拿一个Java中的东西举例子，就是**CountDownLatch，** fork-join模型也可以类比。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29c36b0bf61b4087bf8c80c6237c1b0a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

说到这里，网关的底层逻辑我作为拓展提一句，没听懂也无伤大雅。**网关的本质其实是计数器和出口逻辑的混合**，它跟其他节点没什么区别，只是他的推动逻辑需要使他的计数器为0，而计数器的总数为网关入口线段的数量，比如“组装”节点前面的并行网关，他的计数器就为4，而前面4个节点，每完成一个就会触发该网关计数器-1。

**当计数器为0的时候，网关会触发选择后续流转的逻辑。** 

**互斥网关的逻辑为：遍历所有出口连线，满足条件则流出并打断（也就是break掉）。** 

**并行网关的逻辑为：遍历所有出口连线，无条件为所有连线流出。** 

**包容性网关的逻辑为：遍历所有出口连线，满足条件的就流出。** 

三、**Flowable以及与其他工作流引擎的对比**
---------------------------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dffcd4e3aeb8483299ca325f2879cfd2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

Flowable是BPMN2.0协议的一种Java版本的实现。

Flowable项目提供了一组核心的开源业务流程引擎，这些引擎紧凑且高效。它们为开发人员、系统管理员和业务用户提供了一个工作流和业务流程管理（BPM）平台。它的核心是一个非常快速且经过测试的动态BPMN流程引擎。它基于Apache2.0开源协议，有稳定且经过认证的社区。

Flowable可以嵌入Java应用程序中运行，也可以作为服务器、集群运行，更可以提供云服务。

与大多数故事一样，Flowable也是因为其与Activiti对未来规划的路线不认同而开辟了一条自己的道路。目前主流的工作流开源框架就是Activiti/Camunda/Flowable，它们都有一个共同的祖先jbpm。先是有了jbpm4，随后出来了一个Activiti5，Activiti5经过一段时间的发展，核心人员出现分歧，又分出来了一个Camunda。activiti5发展了4年左右，紧接着就出现了Flowable。

相比于Activiti，Flowable的核心思想更像是在做一个多彩的工具，它在工作流的基础功能上，提供了很多其他的扩展，使用者可以随心所欲地把Flowable打造成自己想要的样子。例如：Camel节点，Mule节点。他不仅有bpmn引擎，还有cmmn（案例管理模型），dmn（决策模型），content（内容管理），form（表单管理），idm（用户鉴权）等等，但这也间接导致了Flowable的包结构非常繁多，上手非常困难。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9cbe77e5b82047719d701a1503a91cf8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03ffc5626ffd49169f4717f5a6ccb1ab~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

而Activiti则只着重于处理bpmn，它的方向在于云，他的设计会尽量像例如Spring Cloud、Docker、K8S靠拢。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e87870b795b641cfb950b6e0a92f1157~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

如果你单纯地想快速上手bpmn引擎，建议使用Activiti，如果你想做出花样繁多的工作流引擎，建议使用Flowable。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cf8d21c5758406eb38a61e6cfc0e2d7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

而Camunda（卡蒙达）则更加的轻巧灵活，他的初衷就是为开发人员设计的“小工具”，但我个人的感觉而言，Camunda从代码上看并没有Activiti和Flowable好，而且他的社区是最不活跃的一个（至少从国内的角度来看），所以不太建议使用（当然这带了很多个人主观感受，如有不同意见，欢迎讨论）。

四、**Flowable的核心架构和一些改进**
------------------------

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e169835c2ea4e7bb2d94fcf94f58250~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

上图是我个人所理解的架构，从架构图中可以看出，Flowable对于数据的处理是冷热分离的，热数据存在ACT\_RU\_系列表中，历史冷数据存在ACT\_HI\_系列表中，定义相关的存在ACT\_RE\_系列表中，而对于在途和定义相关的数据，有一层缓存，他缓存的具体实现比较复杂，这里不多赘述。

对于协议到运行态的转化，有专门一层Converter来实现，也就是说，如果你想自己定义一些协议外的东西，就需要关心这个部分。

最下面是节点行为层，这个很好理解，对于每种节点，都会有不同的实现，你也可以自定义实现。

Flowable在最新的版本中，对于历史归档和异步任务做了新的优化，具体的看下面。

### 1. **Flowable开发了新的异步执行器（ASYNC EXECUTOR）**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/136058e2e9a8406187af7523101f939a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

在工作流中，异步执行大概分为两类，timer和message，类似于定时事件就是timer，而异步的服务任务则为message，如上图所示，“Task A”附着的边界定时事件，在时间触发之后，会执行“Escalate”任务，而“Async Service Task”在“Task A”流转之后，会启用一个异步任务去调用其服务。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75f491b6bdba48b1a694deba14c723c3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

对于一种全是异步服务任务的极端情况，如上图所示，他常常出现于服务编排之类的场景之中，我们经常性的需要同时处理一系列的任务，这时候异步调度的作用就非常重要

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f71ec721ab434e3fac85b35fdb86cfc9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

为了提高性能，Flowable也采用了冷热数据分离的思想，他把数据分为了4类，异步Job，定时器Timer，挂起任务，死信队列。通过测试发现，数据库是异步任务性能低下的主要瓶颈，特别是多实例竞争Job会存在潜在的问题。在分表的时候同时加上了一个全局锁，保证了同一个实例只能由一个实例去获取并锁定job（job表中有字段会被update，内容为抢占到的实例代号），这样反而能提升不少性能。为了保证各个实例不被饿死，还调整了一系列参数。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14850a30c30f488eb64e9dae1a8bea65~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

上图是在100万异步任务（100万流程同时推动）同时并发的情况下的测试数据，很明显，添加更多实例可以提高吞吐量和可伸缩性，这正是引入一个全局锁所带来的效果。同样有趣的是，更改每次Job的获取量也可以显著提高吞吐量。在以前的老版本或Activiti中来说，提高实例反而会降低性能。

PS：对于这部分的优化，我翻译了官方的四篇文章，课后我会发出来，有兴趣的同学可以去看一下。

### 2. **流程归档异步化**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e0b1bddc780f432d94834790a7b279f2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

Flowable提供了一个更加优化的冷热数据分离方案，在数据敏感性比较高的领域，我们一般会把引擎的历史记录级别调到最高（包括流程流转历史、变量变动历史、签收人变动历史等等），这些历史记录在以前是在同一个上下文中执行的，虽然在最开始设计的时候，在途数据和历史数据就冷热分离了，但从权衡在途和历史的重要性的角度来讲，历史其实不是最重要的，所以Flowable提供一系列的方法使历史记录这个行为异步化，与之对应的方法可以是jms，rabbitMQ或Spring Boot listener application。

这个改动可以提升在途流程效率20%-96%。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/684c1dd27a2d4f07b599803845170836~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

上图是一个简单的流程图，从start节点直接到end节点的性能提升。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b42a266324e84ca69df1109433cf8648~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

上图是全顺序执行的服务任务流程性能测试结果。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68a03c10ee134bcd942fb0c7eebdf427~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

上图是多变量的流程图性能测试结果。

PS：对于这部分的优化，其实不参考官方实现，自己也可以做，只要处理好历史查询的延迟容忍度和历史查询ACT\_HI\_PROCINST流程实例表分裂的问题就可以，因为该表是在途和历史数据共存的，比较特殊。

五、**云上的工作流应该关心什么**
------------------

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2326db1a737d4f7980690f6c48eb9f09~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

首先，从用户的角度来看，使用者其实只需要关心三件事 **。** 

* * *

l **我如何把我的业务逻辑转化为流程图-即容易理解的绘图工具。** 

l **我如何使流程流转-即开箱即用的API。** \*\*\*\*

l **我需要引擎告诉我，我现在该处理什么节点-即丰富且鲜明的事件机制。** 

**图中是流程图的整个生命周期，从画图到部署，然后启动流程，流程经过人工或自动的方式流转，最后结束。** 

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65aa98ab1d8b4bac9b65f20653c9cd2b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

**对于生命周期中画图这一块，我们必须要剔除一些不常用且费解的节点类型，比如：异常边界事件、信号或消息事件、泳道。还有一些费解的属性，比如，排他、同步异步、相对的，我们只保留一些常见的节点类型，就如前面介绍的：用户任务、服务任务、互斥网关、并行网关等。** 

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3099756845dd42e8855a2b51d2d381dd~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

**而开箱即用的API就需要我们尽量减少API的复杂度和个数，把API分类为以下三类比较合适。** \*\*\*\*

1. **流程定义类-负责流程定义的查看**

2. **流程实例类-负责流程实例的查看与操作**

3. **任务实例类-负责任务实例的查看与操作**

* * *

**而对于“我现在该处理什么节点”的问题，我们提供了一种解决方案。** 

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ccb8550c1b3415494e29ab47a1eaa55~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

**用户的角色可以分为三类逻辑，第一、和工作流沟通的逻辑，它负责启动流程和通知流程的流转，第二类为服务提供者，即工作流不能提供的服务，需要第三方或业务方自己计算结果，比如：支付接口。第三类为消息处理逻辑，这里的消息大概为任务的创建，任务的签收，任务的完成，流程的创建，流程的结束等等，处理消息的角色可以根据自己的职责处理不同的任务类型或流程类型。** 

**这样区分的好处在于，如果有一个流程图，他的流程涉及到不同系统甚至是不同部门之间的合作，我们不可能让所有部门都去关心整个流程，甚至有些部门根本不知道工作流的存在，他们所关心的是“我需要做什么，我给出什么结果就OK了”。**