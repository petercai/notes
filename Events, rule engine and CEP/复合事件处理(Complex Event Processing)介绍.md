# 复合事件处理(Complex Event Processing)介绍
近年来，面向服务架构 SOA一直是热门的议题。面向服务架构SOA 使用了比组件、程序(procedure)层次更高的服务做为处理单元，通过开放格式交换标准例如XML、Web Service 来交换数据，避免不同平台间的差异带来的不便，达到在异构IT 环境中有效且弹性的组合企业逻辑，并且更快速的产生响应，期望达到所谓实时化的企业。

事件驱动架构(Event-Driven Architecture, EDA)以面向服务架构为基础，将面向服务中的服务进一步转化成以事件作为单位来处理，当某一个事件产生即触发下一个事件。事件驱动架构不仅可以依讯息发送端决定目的，更可以动态依据讯息内容决定后续流程。更能灵活符合日益复杂的商业逻辑架构。

一个事件可以看作是在一个系统中可观察到的状态改变。例如下一笔订单、RFID 传感器回报的信息。在事件驱动架构中包含了两个部份，事件产生者、事件消费者。事件产生者发布信息给管理者，而事件消费者则向管理者订阅信息，事件则触发了下一个事件或是服务(services)，当某个事件发生时，系统及做出相对应的动作。

[![](https://images.cnblogs.com/cnblogs_com/shanyou/WindowsLiveWriter/ComplexEventProcessing_143A0/image_thumb.png)
](http://images.cnblogs.com/cnblogs_com/shanyou/WindowsLiveWriter/ComplexEventProcessing_143A0/image_2.png)

事件驱动架构中也包含了SOA 的概念，称为SOA2.0(reference)，同时也是异步的软件架构。

1.  简单事件处理**(Simple event** **processing) ：** 简单事件处理可看作是消息导向处理的架构，主要处理单一事件，其中事件则定义为可直接观察到的改变。在这个架构中，简单事件处理只做两件事情，过滤(filtering)和路由(routing) 。过滤功能决定事件是否应该被传送出去，路由则决定事件应该传给谁。例如，温度传感器感测到了某个时间变化，就把事件发生直接透过事件处理引擎传给订阅者，一切的工作流程都是实时的。如此一来，使用者将大大的减少了时间跟成本。
2.  复合事件处理**(Complex event** **processing)：** 复合事件是由史丹佛大学的David Luckham 与Brian Fraseca 所提出，David Luckham 与Brian Fraseca 于1990年提出复合事件架构，使用模式比对、事件的相互关系、事件间的聚合关系，目的从事件云(event cloud)中找出有意义的事件，使得IT 架构可以更能弹性使用事件驱动架构，并且能使企业更能快速的开发出更复杂的逻辑架构。在事件驱动架构下，结合简单事件、事件串流处理(Event streaming processing)以及复合事件(Complex event)。相较于简单事件，复杂事件处理不仅处理单一的事件，也处理由多个事件所组成的复合事件。复杂事件处理监测分析事件流(Event streaming)，当特定事件发生时去触发某些动作。
    
    [![](https://images.cnblogs.com/cnblogs_com/shanyou/WindowsLiveWriter/ComplexEventProcessing_143A0/image_thumb_1.png)
    ](http://images.cnblogs.com/cnblogs_com/shanyou/WindowsLiveWriter/ComplexEventProcessing_143A0/image_4.png)
    
    复杂事件处理可看作一种处理串流(Streaming)的数据库处理。在关系数据库中所处理的资料是有许多行(Row)的数据表(table)，复杂事件处理将事件串流当作是数据表来处理，事件类型里的属性相当于数据表的字段。以往使用关联式数据库的时候是将数据先存入关系型数据库后，再用SQL 语法将数据库里的数据表做处理。使用复杂事件处理则把处理数据的过程往前，不用通过保存的动作就在串流中将事件做处理。因此在处理事件的方式上采用SQL-Like 的语言。复杂事件处理中除了过滤和路由之外，还有模式比对的能力。找出事件集合的各种活动，事件聚合，过去历史中的各种因果关系，逻辑以及运算等等，触发新的事件反应。在复杂事件处理中，为了要达到高吞吐量(throughput)、高度利用性(availability)、以及低度延迟(latency)，让企业能够达到实时决策，因此使用事件串流处理(event streamprocessing)。使用EPL(Event Processing Language)为SQL-LIKE 的语言，可以方便的对事件串流提供复杂的逻辑处理，使事件串流在内存中做模式比对处理，及查询的动作。这些过程中，都在内存内进行，不须经由储存装置。
    
    [![](https://images.cnblogs.com/cnblogs_com/shanyou/WindowsLiveWriter/ComplexEventProcessing_143A0/image_thumb_2.png)
    ](http://images.cnblogs.com/cnblogs_com/shanyou/WindowsLiveWriter/ComplexEventProcessing_143A0/image_6.png)
    
3.  StreamInsight 是 SQL Server 2008 R2 中的新模块，它提供了**复杂事件处理（CEP, Complex Event Processing）**的功能。
    
4.  [![](https://images.cnblogs.com/cnblogs_com/shanyou/WindowsLiveWriter/ComplexEventProcessing_143A0/image_thumb_3.png)
    ](http://images.cnblogs.com/cnblogs_com/shanyou/WindowsLiveWriter/ComplexEventProcessing_143A0/image_8.png) 
    
    园子里的相关文章：[【原】StreamInsight 浅入浅出（一）](http://www.cnblogs.com/smjack/archive/2010/08/23/1806745.html)、[【原】StreamInsight 浅入浅出（二）—— 流与事件](http://www.cnblogs.com/smjack/archive/2010/09/06/1819255.html)、 [【原】StreamInsight 浅入浅出（三）—— 适配器](http://www.cnblogs.com/smjack/archive/2010/09/08/1821377.html)
    
5.  **相关开源项目**
    
    Esper – Complex Event Processing
    
    [http://esper.codehaus.org/](http://esper.codehaus.org/)
    
    JBoss – Drools Fusion
    
    [http://www.jboss.org/drools/drools-fusion.html](http://www.jboss.org/drools/drools-fusion.html)
    
    Open ESB IEP SE
    
    [http://wiki.open-esb.java.net/Wiki.jsp?page=IEPSE](http://wiki.open-esb.java.net/Wiki.jsp?page=IEPSE)
    
    ActiveInsight
    
    [http://www.activeinsight.net/](http://www.activeinsight.net/)
    
    其他产品或开源项目，可以参考：[Complex Event Processing Vendors](http://rulecore.com/CEPblog/?page_id=47) 
    
6.  ##### **参考资料**
    
    [深入浅出复合事件处理（CEP）](http://www.slideshare.net/Fenng/cep-3915346)
    
    [轻松理解复合事件处理](http://www.programmer.com.cn/3276/)
    
    [Esper：CEP Engine](http://www.slideshare.net/aparnachaudhary/esper-cep-engine)
    
    [Complex Event Processing：An attempt at clarity on an often confusing topic](http://knol.google.com/k/complex-event-processing)
    
    [Sybase CEP：新颖的数据流分析平台](http://www.sybase.com.mx/files/Legal_Docs/Sybase_CEP.pdf)
    
