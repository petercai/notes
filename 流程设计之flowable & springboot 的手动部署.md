# 流程设计之flowable & springboot 的手动部署 
> 什么是flowable？

#### Flowable是一个使用Java编写的轻量级业务流程引擎。Flowable流程引擎可用于部署BPMN 2.0流程定义（用于定义流程的行业XML标准）， 创建这些流程定义的流程实例，进行查询，访问运行中或历史的流程实例与相关数据Flowable可以十分灵活地加入你的应用/服务/构架。可以将JAR形式发布的Flowable库加入应用或服务，来_嵌入_引擎。 以JAR形式发布使Flowable可以轻易加入任何Java环境：Java SE；Tomcat、Jetty或Spring之类的servlet容器；JBoss或WebSphere之类的Java EE服务器，等。

> 工作流有哪些好处

#### 工作流是一个固定好的框架，大家就按照这个框架来执行流程就行了，但某些情况他本身没有流转顺序的要求，比如：员工A需要请假需要通过经理和组长审批。这种情况我们需要使用CMMN来建模，它就是专门针对这种情况而设计的。  
工作流解决的痛点在于，解除业务宏观流程和微观逻辑的耦合，让熟悉宏观业务流程的人去制定整套流转逻辑，而让专业的人只需要关心他们应当关心的流程节点，总而言之，言而总之就是每个人只需要负责自己的部分。不用关心其他的步骤。

> 这里我们先说一下 Flowable UI 应用- Flowable IDM: 身份管理应用。为所有Flowable UI应用提供单点登录认证功能，并且为拥有IDM管理员权限的用户提供了管理用户、组与权限的功能。

1.  Flowable Modeler: 让具有建模权限的用户可以创建流程模型、表单、选择表与应用定义。
2.  Flowable Task: 运行时任务应用。提供了启动流程实例、编辑任务表单、完成任务，以及查询流程实例与任务的功能。
3.  Flowable Admin: 管理应用。让具有管理员权限的用户可以查询BPMN、DMN、Form及Content引擎，并提供了许多选项用于修改流程实例、任务、作业等。管理应用通过REST API连接至引擎，并与Flowable Task应用及Flowable REST应用一同部署。

> BPMN2.0 是一个广泛接受与支持的，展现流程的注记方法。BPMN 2.0标准对流程的所有的参与者都很有用。最终用户不会因为依赖专有解决方案，而被供应商“绑架”。Flowable之类的开源框架，也可以提供与大型供应商的解决方案相同（经常是更好;-）的实现。有了BPMN 2.0标准，从大型供应商解决方案向Flowable的迁移，可以十分简单平滑。

缺点则是标准通常是不同公司（不同观点）大量讨论与妥协的结果。作为阅读BPMN 2.0 XML流程定义的开发者，有时会觉得某些结构或方法十分笨重。Flowable将开发者的感受放在最高优先，因此引入了一些**_Flowable BPMN扩展（extensions）_ **。这些“扩展”并不在BPMN 2.0规格中，有些是新结构，有些是对特定结构的简化。

尽管BPMN 2.0规格明确指出可以支持自定义扩展，我们仍做了如下保证：

#### 需要放在Tomcat 容器中部署 Flowable UI

```less
1.下载最新稳定版本的[Flowable 6](http:
2.将Flowable发行包中，*wars*文件夹下的flowable-admin.war、flowable-idm.war、flowable-modeler.war与flowable-task.war文件，复制到Tomcat的webapps文件夹下。(注意：新版可能只有两个)

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b912a48d22f467eb0c9bc50ea75488c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 将这两个war 放在 tomcat 的webapp目录下启动tomcat就可以了（第一次启动会很慢）启动成功后之际访问[http://192.168.1.14:8080/flowable-ui/](https://link.juejin.cn/?target=http%3A%2F%2F192.168.1.14%3A8080%2Fflowable-ui%2F "http://192.168.1.14:8080/flowable-ui/") 当然这里的IP是自己的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44f05c5fb34b4952a27449853765d56a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 默认情况下，Flowable IDM应用将创建一个具有访问所有Flowable UI应用所需的权限的admin用户。使用admin/test登录，浏览器会跳转至Flowable Modeler应用：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10dcf780a01a497b8c971f85ffffc5d9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

> 当然 作为可执行的Spring Boot应用，可以直接以独立应用模式运行UI，而不需要应用服务器。

`java -jar flowable-idm.war 直接运行也是没有问题的`

> Flowable IDM应用

#### Flowable IDM应用，用于其他三个Flowable web应用的认证与授权。因此如果你想要运行Modeler，Task或者Admin应用，就需要运行IDM应用。Flowable IDM应用是一个简单的身份管理应用，目标是为Flowable web应用提供单点登录能力，并提供定义用户、组与权限的能力。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccf9c7cc98844912867b6ac84211281d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 可以创建用户

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/028337e12466445bbbfadae6231ac9f5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

> 用户创建完成之后 我们就可以开始设计自己的流程了

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80fca924a22e425e9ae0f86b03f75aa7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/041a437d8baa44a087259b3dbf2291b7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6a71ff1fc434b8ba2bdbf5c2574cae9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/443a45ce1f954f5f84201fc10a6c1456~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 可以先设计一个简单的 请假流程

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2be56283cc774b2999fc6fd0432e01ef~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 设计完成之后可以将这个流程下载下来

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54bdf43fea394ed993c35058fb056ff1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 这个文件就是我们下载下来的BNMN2 文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:flowable="http://flowable.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.flowable.org/processdef">
  <process id="MyHolidayUI" name="MyHolidayUI" isExecutable="true">
    <documentation>北京龙猫集团员工请假流程</documentation>
    <startEvent id="startEvent1" flowable:formFieldValidation="true"></startEvent>
    <userTask id="sid-445DE801-76DB-43AE-8E5F-C8867F89066C" name="请假流程
" flowable:assignee="${assignee0}" flowable:formFieldValidation="true">
      <extensionElements>
        <modeler:initiator-can-complete xmlns:modeler="http://flowable.org/modeler"><![CDATA[false]]></modeler:initiator-can-complete>
      </extensionElements>
    </userTask>
    <userTask id="sid-B3C13F8A-C1A5-405B-8FA9-D0FB48C8D247" name="总经理审批流程
" flowable:assignee="${assignee1}" flowable:formFieldValidation="true">
      <extensionElements>
        <modeler:initiator-can-complete xmlns:modeler="http://flowable.org/modeler"><![CDATA[false]]></modeler:initiator-can-complete>
      </extensionElements>
    </userTask>
    <endEvent id="sid-7CD37596-456E-4A11-AFAF-0076715DB14C"></endEvent>
    <sequenceFlow id="sid-DF1E41BD-0096-4D7E-8E28-B390473F394F" sourceRef="sid-B3C13F8A-C1A5-405B-8FA9-D0FB48C8D247" targetRef="sid-7CD37596-456E-4A11-AFAF-0076715DB14C"></sequenceFlow>
    <sequenceFlow id="sid-34B46151-045D-4854-92A4-C3FB57F96AF3" sourceRef="startEvent1" targetRef="sid-445DE801-76DB-43AE-8E5F-C8867F89066C"></sequenceFlow>
    <sequenceFlow id="sid-5D19CE99-7ED2-4A0F-9761-F9733A165EE5" sourceRef="sid-445DE801-76DB-43AE-8E5F-C8867F89066C" targetRef="sid-B3C13F8A-C1A5-405B-8FA9-D0FB48C8D247"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_MyHolidayUI">
    <bpmndi:BPMNPlane bpmnElement="MyHolidayUI" id="BPMNPlane_MyHolidayUI">
      <bpmndi:BPMNShape bpmnElement="startEvent1" id="BPMNShape_startEvent1">
        <omgdc:Bounds height="30.0" width="30.0" x="100.0" y="163.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-445DE801-76DB-43AE-8E5F-C8867F89066C" id="BPMNShape_sid-445DE801-76DB-43AE-8E5F-C8867F89066C">
        <omgdc:Bounds height="80.0" width="100.0" x="185.5" y="132.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-B3C13F8A-C1A5-405B-8FA9-D0FB48C8D247" id="BPMNShape_sid-B3C13F8A-C1A5-405B-8FA9-D0FB48C8D247">
        <omgdc:Bounds height="80.0" width="100.0" x="615.0" y="120.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-7CD37596-456E-4A11-AFAF-0076715DB14C" id="BPMNShape_sid-7CD37596-456E-4A11-AFAF-0076715DB14C">
        <omgdc:Bounds height="28.0" width="28.0" x="802.5" y="146.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-DF1E41BD-0096-4D7E-8E28-B390473F394F" id="BPMNEdge_sid-DF1E41BD-0096-4D7E-8E28-B390473F394F">
        <omgdi:waypoint x="714.9499999999999" y="160.0"></omgdi:waypoint>
        <omgdi:waypoint x="802.5" y="160.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-34B46151-045D-4854-92A4-C3FB57F96AF3" id="BPMNEdge_sid-34B46151-045D-4854-92A4-C3FB57F96AF3">
        <omgdi:waypoint x="129.93183438270103" y="177.25401957038176"></omgdi:waypoint>
        <omgdi:waypoint x="185.4999999999996" y="174.48713692946058"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-5D19CE99-7ED2-4A0F-9761-F9733A165EE5" id="BPMNEdge_sid-5D19CE99-7ED2-4A0F-9761-F9733A165EE5">
        <omgdi:waypoint x="285.45000000000005" y="170.60302677532013"></omgdi:waypoint>
        <omgdi:waypoint x="614.9999999999997" y="161.39557625145517"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>

```

#### 新建一个springboot 项目 下面是依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.flowable</groupId>
        <artifactId>flowable-spring-boot-starter</artifactId>
        <version>6.6.0</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.14</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>

```

#### yml 文件

```yml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://192.168.1.14:3306/flowable1?autoReconnect=true&useUnicode=true&characterEncoding=utf-8&nullCatalogMeansCurrent=true&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    initialSize: 5
    minIdle: 5
    maxActive: 20
    
    maxWait: 60000
flowable:
  async-executor-activate: false
  database-schema-update: true
server:
  port: 8002

```

#### 创建一个空的数据库启动一下项目

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d08c6cc7db349c48b6ff9bad8f675d2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8edc4f02496a413ca08590a245a2d1e4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c76a6f62c44492183799db892262525~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

### 数据库表名说明

Flowable的所有数据库表都以**ACT_** 开头。第二部分是说明表用途的两字符标示符。服务API的命名也大略符合这个规则。

1.  **ACT\_RE\_** *: 'RE’代表`repository`。带有这个前缀的表包含“静态”信息，例如流程定义与流程资源（图片、规则等）。
2.  **ACT\_RU\_** *: 'RU’代表`runtime`。这些表存储运行时信息，例如流程实例（process instance）、用户任务（user task）、变量（variable）、作业（job）等。Flowable只在流程实例运行中保存运行时数据，并在流程实例结束时删除记录。这样保证运行时表小和快。
3.  **ACT\_HI\_** *: 'HI’代表`history`。这些表存储历史数据，例如已完成的流程实例、变量、任务等。
4.  **ACT\_GE\_** *: 通用数据。在多处使用。

#### 历史表 `HistoryService`

1.  **act\_hi\_actinst**：历史节点表，存放流程实例运转的各个节点信息（包含开始、结束等非任务节点）；
2.  **act\_hi\_attachment**：历史附件表，存放历史节点上传的附件信息；
3.  **act\_hi\_comment**：历史意见表；
4.  **act\_hi\_detail**：历史详情表，存储节点运转的一些信息；
5.  **act\_hi\_identitylink**：历史流程人员表，存储流程各节点候选、办理人员信息，常用于查询某人或部门的已办任务；
6.  **act\_hi\_procinst**：历史流程实例表，存储流程实例历史数据（包含正在运行的流程实例）；
7.  **act\_hi\_taskinst**：历史流程任务表，存储历史任务节点；
8.  **act\_hi\_varinst**：流程历史变量表，存储流程历史节点的变量信息；流程定义表：
9.  **act\_re\_deployment**：部属信息表，存储流程定义、模板部署信息；
10.  **act\_re\_procdef**：流程定义信息表，存储流程定义相关描述信息，但其真正内容存储在act\_ge\_bytearray表中，以字节形式存储；
11.  **act\_re\_model**：流程模板信息表，存储流程模板相关描述信息，但其真正内容存储在act\_ge\_bytearray表中，以字节形式存储；

#### 流程运行时表 `RuntimeService`

1.  **act\_ru\_task**：运行时流程任务节点表，存储运行中流程的任务节点信息，重要，常用于查询人员或部门的待办任务时使用；
2.  **act\_ru\_event_subscr**：监听信息表;
3.  **act\_ru\_execution**：运行时流程执行实例表，记录运行中流程运行的各个分支信息（当没有子流程时，其数据与act\_ru\_task表数据是一一对应的）；
4.  **act\_ru\_identitylink**：运行时流程人员表，重要，常用于查询人员或部门的待办任务时使用；
5.  **act\_ru\_job**：运行时定时任务数据表，存储流程的定时任务信息；
6.  **act\_ru\_variable**：运行时流程变量数据表，存储运行中的流程各节点的变量信息；

#### 用户相表`IdentityService`

1.  **act\_id\_group**：用户组信息表，对应节点选定候选组信息；
2.  **act\_id\_info**：用户扩展信息表，存储用户扩展信息；
3.  **act\_id\_membership**：用户与用户组关系表；
4.  **act\_id\_user**：用户信息表，对应节点选定办理人或候选人信息；

#### 流程定义、流程模板相关表 `RepositoryService`

1.  **act\_re\_deployment**：部属信息表，存储流程定义、模板部署信息；
2.  **act\_re\_procdef**：流程定义信息表，存储流程定义相关描述信息，但其真正内容存储在act\_ge\_bytearray表中，以字节形式存储；
3.  **act\_re\_model**：流程模板信息表，存储流程模板相关描述信息，但其真正内容存储在act\_ge\_bytearray表中，以字节形式存储；

#### 下面让我们把我们刚才画的 流程图 手动部署一下 将下载下来的 bpmn2文件复制到springboot项目中resources目录下

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e263f9e3b11442ebbd1b58d5b07972f6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 使用RepositoryService 手动部署流程

```java
@Autowired
private RepositoryService repositoryService;
public  void testDeploy() {
        Deployment holiday = repositoryService.createDeployment()
                .addClasspathResource("MyHolidayUI.bpmn20.xml")
                .name("holiday01")
                .deploy();
        System.out.println("getId:"+holiday.getId());
        System.out.println("getName:"+holiday.getName());
    }

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04e48dbcdedf4c6aa7462317954896de~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 在DB ACT\_RE\_PROCDEF表中会添加一条记录

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4517a6d68a8c4c0c8c3cdd83087c4b50~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 下面我们启动 一下流程

```java

 * 启动流程
 */
@Test
void testStart() {
    Map<String,Object> map=new HashMap<>();
    map.put("assignee0","张三");
    map.put("assignee1","李四");
    
    runtimeService.startProcessInstanceById("MyHolidayUI:1:9824b6ed-cb5e-11ec-8000-666ee0fc370d",map);
}

```

#### 就可以在 看到 ACT\_RU\_TASK中多一条记录

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f9e2c2673bb4dcfba41a798f79424ef~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 下面我们模拟完成一下 流程

```java

 *完成流程
 */
@Test
void testcomlete() {
    Task task = taskService.createTaskQuery()
            
            .processInstanceId("32551110-cb5f-11ec-8778-666ee0fc370d")
            .taskAssignee("张三")
            .singleResult();
    if (task!=null){
        taskService.complete(task.getId());
        System.out.println("完成");
        
    }
}

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a82526ccd824845b162577e22cd92cb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 完成之后我们去 历史记录中去获取

```java
@Test
void testHistory(){
    List<HistoricActivityInstance> list =
            historyService.createHistoricActivityInstanceQuery()
            .finished()
            
             .processDefinitionId("MyHolidayUI:1:9824b6ed-cb5e-11ec-8000-666ee0fc370d")
            .orderByHistoricActivityInstanceEndTime().asc()
            .list();
    for (HistoricActivityInstance historicActivityInstance : list) {
        System.out.println("ActivityName:"+historicActivityInstance.
                getActivityName()+"ActivityId:"+historicActivityInstance.getActivityId()+
                "耗时："+historicActivityInstance.getDurationInMillis()+"执行人："+historicActivityInstance.getAssignee()
                );

    }
}

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fb3e963051940f895ac8a63530ea070~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

### 整个流程完成 关于其他的API的使用 大家可以 参考  
[Flowable BPMN 用户手册](https://link.juejin.cn/?target=https%3A%2F%2Ftkjohn.github.io%2Fflowable-userguide%2F%23_getting_started "https://tkjohn.github.io/flowable-userguide/#_getting_started")

