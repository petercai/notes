# Flowable从入门到入土(2)-初始Flowable表关系
> **前文传送门：** [**Flowable从入门到入土(1)-初始Flowable**](https://juejin.cn/post/6844904098723004423 "https://juejin.cn/post/6844904098723004423")

* * *

QuickStart
----------

> `Flowable`在`6.4.0`版本上提供了一个[Flowable-tomcat](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fflowable%2Fflowable-engine%2Freleases%2Fdownload%2Fflowable-6.4.0%2Ftomcat-flowable-6.4.0.zip "https://github.com/flowable/flowable-engine/releases/download/flowable-6.4.0/tomcat-flowable-6.4.0.zip")让人能快速的启动`Flowable`进行相关`demo`。包含的模块有：`IDM`, `TASK`,`ADMIN`, `MODLER`。不默认包含`rest`模块。

依赖关系
----

> `Flowable`的表结构生成也是依赖于`Liquibase`来进行生成的。故只需要引入相应的`依赖`就会自动生成表结构。 `SpringBoot`依赖如下：当前版本基于`Flowable的6.5版本`编写。。据官网所说也是`最后一个`开源版本。建议使用一个独立的`DB Schema`。`Flowable`为辅，只提供`Engine`相关服务即。 毕竟还是业务是王道。`Flowable`本身提供的还是太过于`理想化`。为何这么说, 后面一起来看。

```xml
     
    <dependency>
        <groupId>org.flowable</groupId>
        <artifactId>flowable-spring-boot-starter-basic</artifactId>
        <version>${flowable.version}</version>
    </dependency>
    
    
    <dependency>
        <groupId>org.flowable</groupId>
        <artifactId>flowable-bpmn-converter</artifactId>
        <version>${flowable.version}</version>
    </dependency>
    
    <dependency>
        <groupId>org.flowable</groupId>
        <artifactId>flowable-json-converter</artifactId>
        <version>${flowable.version}</version>
    </dependency>
    
    <dependency>
        <groupId>org.flowable</groupId>
        <artifactId>flowable-bpmn-layout</artifactId>
        <version>${flowable.version}</version>
    </dependency>
    

```

> 启动之后就会生成基本的`46`张表。而非之前的传统的`34`张表。
> 
> ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/3/28/1711f63f0bd0ace3~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)

表结构分析
-----

> 一看`表命名`是不是有点熟悉。没错。和`Activity`简直是一模一样的风格。毕竟都是同一波核心开发人员。`有时候，英雄的离开，是以另一种方式来燃气烈火。`。不扯犊子了。

### ACT\_EVT\_*

*   ACT\_EVT\_LOG: 事件日志相关
*   ACT\_PROCDEF\_INFO: 当通过缓存保存的流程信息

### ACT\_GE\_*

*   ACT\_GE\_BYTEARRAY：保存流程的bpmn的xml以及流程的Image缩略图等信息
*   ACT\_GE\_PROPERTY：Flowable相关的基本信息。比如各个module使用的版本信息。

### ACT\_RE\_*

*   ACT\_RE\_DEPLOYMENT: 部署对象，存储流程名称 租户相关
*   ACT\_RE\_MODEL：基于流程的模型信息
*   ACT\_RE\_PROCDEF：流程定义表

### ACT\_RU\_*(Runtime相关)

*   ACT\_RU\_ACTINST：运行中实例的活动表
*   ACT\_RU\_DEADLETTER_JOB：当JOB执行很多次都无法执行，就会被记录在此表
*   ACT\_RU\_ENTITYLINK：还没使用到。后续更新此表。
*   ACT\_RU\_EVENT_SUBSCR：运行时的事件
*   ACT\_RU\_EXECUTION：运行的实例表
*   ACT\_RU\_HISTORY_JOB； 运行中的定时任务历史表
*   ACT\_RU\_IDENTITYLINK： 当前任务执行人的信息
*   ACT\_RU\_JOB：运行中的异步任务
*   ACT\_RU\_SUSPENDED_JOB：暂停的任务表。如果一个异步任务在运行中，被暂停。就会记录在词表
*   ACT\_RU\_TASK：运行中的正常节点任务
*   ACT\_RU\_TIMER_JOB：定时作业表
*   ACT\_RU\_VARIABLE：运行中的流程实例变量

### ACT\_ID\_*(IDM模块。用户相关)

> 这块其实基于我个人想法，不建议使用。但是如果是以工作流为核心，为业务的公司，比如专门做OA的公司。鄙人是制造业相关出身，故工作流这块的IDM没有使用。都是业务系统进行WorkFlowEngine的调用而已。

*   ACT\_ID\_BYTEARRAY：
*   ACT\_ID\_GROUP：用户组信息
*   ACT\_ID\_INFO：用户详情
*   ACT\_ID\_MEMBERSHIP：用户组和用户的关系
*   ACT\_ID\_PRIV：权限
*   ACT\_ID\_PRIV_MAPPING：用户组和权限之间的关系
*   ACT\_ID\_PROPERTY：用户或者用户组属性拓展表
*   ACT\_ID\_TOKEN：登录相关日志
*   ACT\_ID\_USER：用户

### ACT\_HI\_*(历史相关)

*   ACT\_HI\_ACTINST: 流程实例历史
*   ACT\_HI\_ATTACHMENT：实例的历史附件，几乎不会使用，会加大数据库很大的一个loading
*   ACT\_HI\_COMMENT：实例的历史备注
*   ACT\_HI\_DETAIL：实例流程详细信息
*   ACT\_HI\_IDENTITYLINK: 实例节点中，如果指定了`目标人`，产生的历史
*   ACT\_HI\_PROCINST：流程实例历史
*   ACT\_HI\_TASKINST：流程实例的任务历史
*   ACT\_HI\_VARINST：流程实例的变量历史

### FLW_*

*   FLW\_CHANNEL\_DEFINITION: 泳池管道定义表
*   FLW\_EVENT\_DEFINITION：事件定义
*   FLW\_EVENT\_DEPLOYMENT：事件必输
*   FLW\_EVENT\_RESOURCE：事件所需资源
*   FLW\_EV\_DATABASECHANGELOG：Liquibase执行的记录
*   FLW\_EV\_DATABASECHANGELOGLOCK：Liquibase执行锁
*   FLW\_RU\_BATCH：暂时未知
*   FLW\_RU\_BATCH_PART：暂时未知

> 具体的相应的表结构会在后面的介绍中，结合实例一一分析。此处就不多费口舌了。

* * *

> 本期内容就到此就结束了。如有大侠指定，请不吝赐教。。。下期介绍流程定义，启动，历史以及查询代办任务以及流程未来节点相关。尽情期待。。。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/3/21/170fc1fddfa3554b~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)