# Flowable从入门到入土(3)- Activiti和Flowable的对比以及选型
> **作者：独钓寒江雪**  
> **一剑一酒一江湖, 便是此生**  
> **心中有图，何必点灯。装载请注明出处**

* * *

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/21/1719cef703943411~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)

> **前文传送门：** [**Flowable从入门到入土(1)-初始Flowable**](https://juejin.cn/post/6844904098723004423 "https://juejin.cn/post/6844904098723004423")  
> **前文传送门：** [**Flowable从入门到入土(2)-初始Flowable表关系**](https://juejin.cn/post/6844904105282895879 "https://juejin.cn/post/6844904105282895879")

* * *

Activiti和Flowable的渊源
--------------------

> `Activiti`和`Flowable`都是来自于一个叫`JBPM`的开源工作流。在早期`Jboss(现已被ReHat收购)`发行`JBPM4`的时候，因为合作伙伴关系闹的不开心。于是其中一个核心人员离职。加入了`Alfresco(Activiti所在的公司)`。并在同一年发布了`Activiti的第一个版本`即`Activiti5.0.alpha1`。这个版本号直接从`5.0`开始就很有意思了。表示其为携带了`JBPM4`的所有特性。正式叫板`JPBM4`。  
> JBPM4也是被使用的最广的一个版本之一其中正有我的`老东家(工作流只是个辅助业务的，我老东家在上面自己做了很多东西，故升级版本是不现实的)`。也在同一年，`Jboss`对JBPM4进行了重构，使用了自己公司新研发的一套`规则引擎Drools`进行重构。将`JBPM4`升级到了`JBPM5`。但是那时候国内更多软件基于`Tomcat`上进行部署，JBoss所用甚少。故JBPM后续版本在国内占据市场远远不如`Activiti`。  
> `Activiti`就一直在5.0这个版本一直迭代开发。 国外的开源软件有个好习惯就是：在当前开发的这个版本趋于稳定的时候，会开始陆陆续续构建`下一个大版本`。`Activiti`那时候也想的很美好。5.0这个版本这么稳定了，6.0应该没什么问题。但是，天要下雨，娘要嫁人。`Activiti`的创作者，也因为和合作伙伴关系闹的不开心。又一次上演了`离家出走`，先后开办了`Camunda`和`Flowable`。导致了`Activiti`最后5.0的问题修复不过来了，官方放弃了这个版本。但是Activiti5可以说的上在工作流的标杆版本之一。至今仍被N多公司进行使用。**工作流毕竟只是一个辅助业务的东西，故版本的大升级对于一个公司来说，是代价巨大的。**   
> Flowable在开办之初，比`Activiti`当初直接继承`JBPM`的版本更为直接。直接继承了他的小版本。直接就从`Flowable5.22`这个版本开始。和当时的`Activiti`的小版本一致。真是冤冤相报何时了。。。  
> 几者的关系以及热门度如下图所示。`颜色越深`表示`使用者越多`。
> 
> ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/21/1719d6748013455b~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)
> 
> 下面将着重介绍下Activiti7和Flowable6的选型。Camunda是商业收费并且不开源。故此处就不做介绍。。。  
> 为什么不介绍`Activiti5`? 因为本着`用新不用旧，更好更高吹牛逼`的原则。新研发的东西基本不会基于`Activiti`这个官方已经放弃了维护的版本进行`初次开发符合自己业务`的系统。但是老系统建议沿用，不建议进行任何所谓的升级的骚包操作。。。版本差距巨大。  
> 为什么不介绍`Activiti6`? 如上文所说，国外的开源软件有个好习惯就是：在当前开发的这个版本趋于稳定的时候，会开始陆陆续续构建`下一个大版本`。但是中途`核心人员`的离开使得`Activiti`无法继续进行。不知道是不是出于版本的衔接，还是KPI的考量?直接就发布`Activiti`并在短短时间之内，开始构建`Activiti7`。

Flowable6和Activiti7的较量
----------------------

> 由于当前`微服务和敏捷开发之风`盛行。Flwoable和Activiti为了这块的技术市场。分别推出了基于`SB(SpringBoot)`上所做的`SpringBootStarter`。来支持微服务。更推出自己的`docker`镜像并对`Jeckins`和`Kubernates`做了良好的支持。但是由于`Activiti7`正式的release版本较晚，以及刚开始发布的[`GA版`本](https://link.juejin.cn/?target=https%3A%2F%2Factiviti.gitbook.io%2Factiviti-7-developers-guide%2Freleases%2F7.0.0.ga "https://activiti.gitbook.io/activiti-7-developers-guide/releases/7.0.0.ga")居然不支持`JDK8`这个广为使用的JDK版本而是直接就到`JDK11`。这使得众多的开发者，不得不转到Flowable。`Activiti7`从[`SR1`](https://link.juejin.cn/?target=https%3A%2F%2Factiviti.gitbook.io%2Factiviti-7-developers-guide%2Freleases%2F7.0.0.sr1 "https://activiti.gitbook.io/activiti-7-developers-guide/releases/7.0.0.sr1")版本才开始支持JDK8，并在现在陆陆续续在构建`7.1`。但是大部分市场已经转型到了`Flowable`。但是也靠着忠实的`fans`拉回了不少市场。 如下图是在`Activiti7`对GA版本的截图：其更多是为了`支持微服务和镜像服务编排`以及发布了新的一款流程`Modeler(基于Camunda的bpmn.js做的开发)。Flowable的Modeler也正在往此处转型`。更多请到[`官网`](https://link.juejin.cn/?target=https%3A%2F%2Factiviti.gitbook.io%2Factiviti-7-developers-guide%2Freleases%2F7.1.0.m6 "https://activiti.gitbook.io/activiti-7-developers-guide/releases/7.1.0.m6")去看其`ReleaseNote`
> 
> ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/21/1719d37caa867df1~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)
> 
> ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/21/1719d38514bd9b03~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)

### Github活跃度的较量

#### Activiti7

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/21/1719d459670b8905~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)

#### Flowable6

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/21/1719d4229392630e~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)

> 从如上2图可以看出，2者当前的活跃度其实不差多少的。都是很优秀的在快速迭代修正Issue。

### 对自由式流程的支持

因为基于BPMN2.0的规范来讲，对自由式流程，比如任意加减流程节点，跳回任意节点在设计规范之初就是不支持的。这也是我很好奇国外的领导人的风格？难道不为所欲为么???这种设计很明显的在国内是不太符合的。。。任意驳回跳转到任意节点是在正常不过的一种(骚包)操作了。。。2者都需要一定性的定制开发。不复杂，后续会对这块进行详细的讲解。

### 对数据库结构版本变化的追溯

> `Activiti7`和`Flowable`都基于`Liquibase`上做了数据库版本更新的统一追溯。

### 对CMMN/DMN的支持

> 这个其实在我看来，这个没啥比较的意义。无论与`CMMN`还是`DMN`都还处于太过于简单的业务上进行决策。实际使用中，对于任何一个东西的系统做自动决策，肯定都需要大量的广而复杂的规则进行验证。而非简单的类似BPMN2.0的`Gateway`的如此简单。建议此块使用`Drool`上进行规则的规划以及验证。

* * *

> 工作流引擎往往只是个辅助业务进行实现的东西。这也是Activiti7和Flowable中IDM模块被称为`鸡肋`的原因所在。框架的选择只是初步。更重要的是针对自己业务的封装。工作流上的`节点`对于用户来说，是处于`灰黑色`地带的。如果让我再次重新选择工作流，我大概率会偏向于A7。但是已经基于Flowable上已经做了业务封装的，不建议替换。。。  
> 本期内容就到此就结束了。如有大侠指定，请不吝赐教。。。下期进入正文介绍流程引擎`ProcessEnginee`和引擎的配置`ProcessEngineConfiguration`。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/21/1719d59e2be280c7~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)