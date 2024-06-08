# Flowable从入门到入土(1)-初始Flowable

简介
--

> 当前周围好多人都对flowable有所了解乃至投入到`生产`使用，此文章作为`抛砖引玉`之用。引出各位大侠进行指正。  
> 如果你对工作流引擎有所了解，那么一定知道Java领域当前主流的工作流引擎无非就是`Jboss旗下的JBPM`和`Alfresco旗下的Activity`。 Flowable是Activiti原班主创人员从Activiti分离出来的一套工作流引擎，是一个业务流程管理(BPM)和工作流系统，适用于开发人员和系统管理员。其核心是`超快速`、`稳定`的BPMN2流程引擎，易于与 Spring集成使用。 Flowable的诞生简直和Acitiviti的诞生如出一辙！当年JBMP的主创Tom已经离开Alfresco多年，后辈们也开始步前人后尘。`Tijs Rademakers`、`Joram Barrez`等Activiti的原班核心人马，由于与Alfresco公司在项目的未来发展方向上出现分歧，于是选择集体出走，创建了Flowable，并且将第一个版本定义为5.22，而且在两周前发布了6.0版本！要知道，Activiti当前版本依然还是5.22，6.0处于Beta阶段。 Flowable团队对用户说道，`“如果你还在犹豫是否加入我们，请看看Activiti源码里的作者们，再看看Flowable项目的成员们。我们是最懂Activiti、在过去几年里推动了整个社区、为社区做出贡献和改革的那帮人。”` 选择Flowable的要点如下：

*   社区强大并开源。截止文章写时，`flowable`已经在计划后续版本不开源使用。但是有老版本开源在手，天下我有。毕竟`工作流`只是业务上的`辅助平台`。
*   更新力度强大 源源不断的更新。
*   主力开发团队是原activity的主力团队。
*   相比`camuda`这个工作流引擎。可定制化程度高很多。
*   在代码风格上,Flowable是比较尊重并且推崇DDD的开发准则的。这读起来还是让人比较舒服。

> 任何的`工作流引擎`主要工作都体现为`节点的流转以及表单的填写`。其还是以`业务为主，工作流为辅`，切不可工作流为主，业务为辅进行实际生产使用。

源码下载
----

> Flowable的[**GitHub官网地址**](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fflowable%2Fflowable-engine.git "https://github.com/flowable/flowable-engine.git") 源码目录说明

*   modules:该文件夹下存储了Activity项目所有模块的Java源文件。因为flowable和Activity是同一波开发团队，故此处会看到和Activity非常相似的模块。
*   qa: 一些通用的流程配置文件样例。
*   scripts: Linux平台下的一些启动脚本文件。
*   doc: 用户操作手册，包括了基础的`UserGuide和PublicApi`
*   docker和k8s: 容器镜像和管理
*   pom.xml: 所有Maven工程的parent。Flowable各个子模块项目中依赖的第三方包均定义在该文件中

服务介绍
----

> `Flowable`整体是通过`ProcessEngine`来操作的。即不管什么框架操作流程，都需要通过`ProcessEngine`这个类来处理。`ProcessEngine`是`Flowable`对外公开的`门面`。UML类图如下：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/3/21/170fc025f13b64df~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

> 关系图如下：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/3/21/170fc02e3be42183~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

> 介绍下如下这些`实际经常使用`的到的`类`以及`关键方法`。

### RepositoryService

> 流程建立相关。比如流程定义，流程的部署，流程文件的查询，验证`BpmnModel`等，流程转换成`bpmnModel`，`BpmnModel转化成流程文件(.bpm)`等。关键方法如下：

```java


 List<ValidationError> validationErrors = repositoryService.validateProcess(bpmnModel);


repositoryService.createProcessDefinitionQuery().processDefinitionName(route.getId().toString()).singleResult();


Deployment deployment = repositoryService.createDeployment().addBpmnModel(bpmnModel.getProcesses().get(0).getName()+".bpmn", bpmnModel).deploy();


repositoryService.getBpmnModel(processDefinition.getId());


InputStream inputStream = repositoryService.getResourceAsStream(deploymentId, resourceName);


repositoryService.deleteDeployment(processDefinition.getDeploymentId(), cascade);

```

### RumtimeService

> 流程实例管理。管理流程实例的启动，推进，删除，获取流程实例当前状态等。

```java

runtimeService.startProcessInstanceById(processDefinition.getId(), businessId);


ProcessInstance processInstance = runtimeService.createProcessInstanceQuery().processInstanceBusinessKey(businessId).singleResult();

Execution execution = runtimeService.createExecutionQuery().processInstanceId(processInstance.getId()).activityId("S1").singleResult();

runtimeService.trigger(execution.getId());

```

### HistoryService

> 流程历史查询，所有有关历史的查询都从这个Service中来。

```java

List<HistoricProcessInstance> list = historyService.createHistoricProcessInstanceQuery().processInstanceBusinessKey(lotId).list();
String instanceId = "";
for (HistoricProcessInstance hpi : list) {
    instanceId = hpi.getId();
    System.out.println("流程定义文件->" + hpi.getProcessDefinitionId());
    System.out.println("ProcessInstance->" + hpi.getId());
    System.out.println("StartActivityId->" + hpi.getStartActivityId());
    System.out.println("EndActivityId->" + hpi.getEndActivityId());
    System.out.println("startTime ->" + hpi.getStartTime() + "EndTime->" + hpi.getEndTime() + "duration" + hpi.getDurationInMillis());
}


List<HistoricActivityInstance> list1 = historyService.createHistoricActivityInstanceQuery().processInstanceId(instanceId).list().stream().sorted(Comparator.comparing(HistoricActivityInstance :: getStartTime)).collect(Collectors.toList());
for (HistoricActivityInstance historicActivityInstance : list1) {
    System.out.println("------");
    System.out.println(historicActivityInstance.getActivityId());
    System.out.println(historicActivityInstance.getActivityName());
    System.out.println(historicActivityInstance.getActivityType());
    System.out.println("startTime ->" +historicActivityInstance.getStartTime() + "EndTime->" + historicActivityInstance.getEndTime() + "duration" + historicActivityInstance.getDurationInMillis());
}

```

### TaskService

> 用于指定节点任务的主要Service.比如ReceiveTask, UserTask, MailTask,HttpTask等等。不同的Task具备了不同的处理方式。

```java
Execution execution = runtimeService.createExecutionQuery().processInstanceId(processInstance.getId()).activityId("S1").singleResult();
runtimeService.trigger(execution.getId());

execution = runtimeService.createExecutionQuery().processInstanceId(processInstance.getId()).activityId("S2").singleResult();
runtimeService.trigger(execution.getId());

execution = runtimeService.createExecutionQuery().processInstanceId(processInstance.getId()).activityId("S3").singleResult();
runtimeService.trigger(execution.getId());

```

> 其他`Service`会在`后续的文章`中在逐一讲解。

构建流程
----

### 通过流程的bpmn文件进行部署流程

> 可以通过flowable提供的`eclipse插件`,`Idea插件`或者`webui(需引入FlowableModuler。不推荐)`上进行创建流程并保存为bpmn文件。并放入项目的`classpath`下进行引入部署。 **注意：deployment的时候ResourceName要是bpmn结尾。不然不会被解析。相关源码参考`ResourcesUtils`**

```java
public static final String[] BPMN_RESOURCE_SUFFIXES = new String[] { "bpmn20.xml", "bpmn" };
public static final String[] DIAGRAM_SUFFIXES = new String[] { "png", "jpg", "gif", "svg" };

```

### 使用bpmn模型构建并部署流程

*   可以通过[**bpmn.js**](https://link.juejin.cn/?target=https%3A%2F%2Fbpmn.io%2Ftoolkit%2Fbpmn-js%2F "https://bpmn.io/toolkit/bpmn-js/")。其为`Camuda`开源的一个BPM的前端JS库。其可以直接使用并定义相应的流程。建议用此种。比较方便且快捷。bpmn.js使用后续会有相应的文章来吹牛*。敬请期待。
*   可以通过自己公司的前端开发进行构建传递数据。

```java
BpmnModel bpmnModel = new BpmnModel(); 

StartEvent start = new StartEvent();
start.setId("start1");
start.setName("开始节点");

SequenceFlow flow1 = new SequenceFlow(); 
3eflow1.setId("flow1"); 
flow1.setName("开始节点-->任务节点1"); 
flow1.setSourceRef("start1"); 
flow1.setTargetRef("receiveTask1");

ReceiveTask receiveTask1 = vbnew ReceiveTask();
receiveTask1.setId("receiveTask1");
receiveTask1.setName("节点1");

SequenceFlow flow2 = new SequenceFlow(); 
flow2.setId("flow2"); 
flow2.setName("节点1-->结束节点"); 
flow2.setSourceRef("receiveTask1"); 
flow2ow1.setTargetRef("endEvent");

EndEvent end = new EndEvent();
endEvent.setId("endEvent"); 
endEvent.setName("结束节点"); 

Process process=new Process(); 
process.setId("process1"); 
process.setName("process1"); 

process.addFlowElement(start); 
process.addFlowElement(flow1); 
process.addFlowElement(receiveTask1); 
process.addFlowElement(flow2); 
process.addFlowElement(end); 

bpmnModel.addProcess(process);

repositoryService.createDeployment().addBpmnModel(bpmnModel.getProcesses().get(0).getName()+".bpmn", bpmnModel).deploy();

```

踩坑总结
----

### 数据库脚本创建策略

> 因为flowable在数据库版本管控上使用的是`Liquibase`进行管控。所以在自动创建完flowable的数据库之后。一定要将`ProcessEngineConfiguration`的`database-schema-update`改成false。不然当不同的人更改了flowable的版本，会自动更新表结构。会导致问题。

### 所有节点的ID不能是数字

> Flowable在定义的节点中不能是数字作为节点的Id。数字作为节点的ID会出现异常。异常部分堆栈如下

```java
org.flowable.bpmn.exceptions.XMLException: javax.xml.stream.XMLStreamException: org.xml.sax.SAXParseException; lineNumber: 4; columnNumber: 6; cvc-datatype-valid.1.2.1: '1' 不是 'NCName' 的有效值。
    at org.flowable.bpmn.converter.BpmnXMLConverter.convertToBpmnModel(BpmnXMLConverter.java:273)
    at org.flowable.engine.impl.bpmn.parser.BpmnParse.execute(BpmnParse.java:148)
    at org.flowable.engine.impl.bpmn.deployer.ParsedDeploymentBuilder.createBpmnParseFromResource(ParsedDeploymentBuilder.java:97)
    at org.flowable.engine.impl.bpmn.deployer.ParsedDeploymentBuilder.build(ParsedDeploymentBuilder.java:55)
    at org.flowable.engine.impl.bpmn.deployer.BpmnDeployer.deploy(BpmnDeployer.java:76)
    at org.flowable.engine.impl.persistence.deploy.DeploymentManager.deploy(DeploymentManager.java:62)

```
