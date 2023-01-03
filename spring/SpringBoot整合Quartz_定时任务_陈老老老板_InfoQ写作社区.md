# SpringBoot整合Quartz_定时任务_陈老老老板_InfoQ写作社区
![](https://static001.geekbang.org/infoq/a3/a3f0d7b148bcd51f919156fcf339e28c.gif)

> 陈老老老板
> 
> 说明：工作了，学习一些新的技术栈和工作中遇到的问题，边学习边总结，各位一起加油。需要注意的地方都标红了，还有资源的分享. 一起加油。**本文是介绍 Quartz 与 SpringBoot 整合**  

**1.简介**
--------

**可以先看一下 Quartz 官网：** [**Quartz官网**](https://xie.infoq.cn/link?target=http%3A%2F%2Fwww.quartz-scheduler.org%2F)****说明：在整合之前先学习一下什么是 QUartz：Quartz 是一个功能丰富的开源作业调度库，可以集成到几乎任何 Java 应用程序中，Quartz 是 OpenSymphony 开源组织在 Job Scheduling 领域又一个开源项目，是完全由 Java 开发的一个开源任务日程管理系统，“任务进度管理器”就是一个在预先确定（被纳入日程）的时间到达时，负责执行（或者通知）其他软件组件的系统。 **Quartz 是一个开源的作业调度框架，它完全由 Java 写成，并设计用于 J2SE 和 J2EE 应用中，它提供了巨大的灵活性而不牺牲简单性。（也就是定时任务，确定好时间，就按指定时间执行）******

******2.核心概念：******
-------------------

*   ******工作（Job）：用于定义具体执行的工作，是一个==接口==，只定义一个方法==execute()== 方法，在实现接口的 execute() 方法中编写所需要定时执行的 Job。******
    
*   ******工作明细（JobDetail）：用于描述定时工作相关的信息，包括了任务的==唯一标识和具体要执行的任务==，可以通过 JobDataMap 往任务中传递数据******
    
*   ******触发器（Trigger）：描述了工作明细与调度器的对应关系，是一个==类==，描述触发 Job (工作)执行的==时间触发规则==。说明：主要有 SimpleTrigger 和 CronTrigger 这两个子类。SimpleTrigger 当且仅当需调度一次或者以固定时间间隔周期执行调度，是最适合的选择；CronTrigger 则可以通过 Cron 表达式定义出各种复杂时间规则的调度方案：（如工作日周一到周五的 15：00 ~ 16：00 执行调度等）******
    
*   ******调度器（Scheduler）：用于描述触发工作的==执行规则==，通常使用==cron 表达式==定义规则。通过 Trigger 和 JobDetail 可以用来调度、暂停和删除任务。调度器就相当于一个容器，装载着任务和触发器，该类是一个==接口==，代表一个 Quartz 的独立运行容器，Trigger 和 JobDetail 可以注册到 Scheduler 中，两者在 Scheduler 中拥有各自的组及名称，组及名称是 Scheduler 查找定位容器中某一对象的依据，Trigger 的组及名称必须唯一，JobDetail 的组和名称也必须唯一（但可以和 Trigger 的组和名称相同，因为它们是不同类型的）******
    

********注：简单说就是你定时干什么事情，这就是工作，工作不可能就是一个简单的方法，还要设置一些明细信息。********

******3.作业存储类型******
--------------------

*   ********RAMJobStore：==RAM 也就是内存==，默认情况下 Quartz 会将任务调度存储在内存中，这种方式性能是最好的，因为内存的速度是最快的。不好的地方就是数据缺乏持久性，但程序崩溃或者重新发布的时候，所有运行信息都会丢失********
    
*   ********JDBC 作业存储：存到数据库之后，可以做单点也可以做集群，==当任务多了之后，可以统一进行管理，随时停止、暂停、修改任务==。关闭或者重启服务器，运行的信息都不会丢失。缺点就是运行速度快慢取决于连接数据库的快慢********
    

******这里演示使用的是第一种内存存储作业信息，较为简单，只需在 properties 中设置即可。******

******1、创建项目******
------------------

![](https://img-blog.csdnimg.cn/e2b34af38e364e6f9ed9f81f68f97384.png)

  

******2.添加依赖******
------------------

```
<font color="red"><b><b><b><code class="language-xml"><dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
</code></b></b></b></font>
```

******整体依赖******

```
<font color="red"><b><b><b><code class="language-xml"> <dependencies>
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-quartz</artifactId>
        </dependency>
    </dependencies>
</code></b></b></b></font>
```

******2.配置文件******
------------------

### ******（1）创建 quartz.properties******

********注：默认情况下，Quartz 会加载 classpath 下的 quartz.properties 作为配置文件。如果找不到，则会使用 quartz 框架自己 jar 包下 org/quartz 底下的 quartz.properties 文件********

```
<font color="red"><b><b><b><code class="language-yaml">#主要分为scheduler、threadPool、jobStore、dataSource等部分


org.quartz.scheduler.instanceId=AUTO
org.quartz.scheduler.instanceName=DefaultQuartzScheduler
#如果您希望Quartz Scheduler通过RMI作为服务器导出本身，则将“rmi.export”标志设置为true
#在同一个配置文件中为'org.quartz.scheduler.rmi.export'和'org.quartz.scheduler.rmi.proxy'指定一个'true'值是没有意义的,如果你这样做'export'选项将被忽略
org.quartz.scheduler.rmi.export=false
#如果要连接（使用）远程服务的调度程序，则将“org.quartz.scheduler.rmi.proxy”标志设置为true。您还必须指定RMI注册表进程的主机和端口 - 通常是“localhost”端口1099
org.quartz.scheduler.rmi.proxy=false
org.quartz.scheduler.wrapJobExecutionInUserTransaction=false


#实例化ThreadPool时，使用的线程类为SimpleThreadPool
org.quartz.threadPool.class=org.quartz.simpl.SimpleThreadPool
#threadCount和threadPriority将以setter的形式注入ThreadPool实例
#并发个数  如果你只有几个工作每天触发几次 那么1个线程就可以,如果你有成千上万的工作，每分钟都有很多工作 那么久需要50-100之间.
#只有1到100之间的数字是非常实用的
org.quartz.threadPool.threadCount=5
#优先级 默认值为5
org.quartz.threadPool.threadPriority=5
#可以是“true”或“false”，默认为false
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread=true


#在被认为“misfired”(失火)之前，调度程序将“tolerate(容忍)”一个Triggers(触发器)将其下一个启动时间通过的毫秒数。默认值（如果您在配置中未输入此属性）为60000（60秒）
org.quartz.jobStore.misfireThreshold=5000
# 默认存储在内存中,RAMJobStore快速轻便，但是当进程终止时，所有调度信息都会丢失
#org.quartz.jobStore.class=org.quartz.simpl.RAMJobStore

#持久化方式，默认存储在内存中，此处使用数据库方式
org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
#您需要为JobStore选择一个DriverDelegate才能使用。DriverDelegate负责执行特定数据库可能需要的任何JDBC工作
# StdJDBCDelegate是一个使用“vanilla”JDBC代码（和SQL语句）来执行其工作的委托,用于完全符合JDBC的驱动程序
org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
#可以将“org.quartz.jobStore.useProperties”配置参数设置为“true”（默认为false），以指示JDBCJobStore将JobDataMaps中的所有值都作为字符串，
#因此可以作为名称 - 值对存储而不是在BLOB列中以其序列化形式存储更多复杂的对象。从长远来看，这是更安全的，因为您避免了将非String类序列化为BLOB的类版本问题
org.quartz.jobStore.useProperties=true
#表前缀
org.quartz.jobStore.tablePrefix=QRTZ_
#数据源别名，自定义
org.quartz.jobStore.dataSource=qzDS
</code></b></b></b></font>
```

[******关于quartz全部配置详情可以点击查看******](https://xie.infoq.cn/link?target=https%3A%2F%2Fblog.csdn.net%2Fzixiao217%2Farticle%2Fdetails%2F53091812)********注：其实不需要这么多想简单入门可以按如下的配置 **quartz.properties**********

```
<font color="red"><b><b><b><code class="language-yaml">org.quartz.scheduler.instanceName = MyScheduler
org.quartz.threadPool.threadCount = 3
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
</code></b></b></b></font>
```

*   ******org.quartz.scheduler.instanceName - 设置调度程序 scheduler 的名称为 MyScheduler******
    
*   ******org.quartz.threadPool.threadCount - 线程里设置了 3 个线程，意味着最多同时运行 3 个 job******
    
*   ******org.quartz.jobStore.class - 指定为 RAMJobStore，表示所有 Quartz 的数据，包括 jobDetail、trigger 等均运行在内存中（而不是数据库中）。 如果你想 Quartz 使用你的数据库，还是建议你在使用数据库配置之前使用 RAMJobStore 进行工作。通过使用一个数据库，你可以打开一个全新的维度，但在这之前，建议你使用 RAMJobStore。******
    

******3、定义任务 Bean，按照 Quartz 的开发规范制作，继承 QuartzJobBean******
----------------------------------------------------------

```
<font color="red"><b><b><b><code class="language-java">public class MyQuartz extends QuartzJobBean {
    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
    //这里是你想执行的任务信息
    //可以使用context参数获取其他信息，联系上下文
        System.out.println("quartz task run...");
    }
}
</code></b></b></b></font>
```

******4、创建 Quartz 配置类，定义工作明细（JobDetail）与触发器的（Trigger）bean******
---------------------------------------------------------------

```
<font color="red"><b><b><b><code class="language-java">@Configuration
public class QuartzConfig {
    @Bean
    public JobDetail printJobDetail(){
        //绑定具体的工作（newJob中的参数，就是要执行的Job类）
        //storeDurably()用来做持久化
        return JobBuilder.newJob(MyQuartz.class).storeDurably().build();
    }
    @Bean
    public Trigger printJobTrigger(){
    //  创建cron表达式表示什么时间执行，一般日期和星期不一起出现                   每两秒执行一次
        ScheduleBuilder schedBuilder = CronScheduleBuilder.cronSchedule("0/2 * * * * ?");
        //绑定对应的工作明细   forJob中写上面JobDetail（任务明细）的方法名
        return  TriggerBuilder.newTrigger().forJob(printJobDetail()).withSchedule(schedBuilder).build();
    }
}
</code></b></b></b></font>
```

******CronScheduleBuilder.cronSchedule("0/5 \* \* \* \* ?");这里使用的是 cron 表达式，下一篇博客会有详解。******

******工作明细中要设置对应的具体工作，使用 newJob()操作传入对应的工作任务类型即可。******

******5.测试******
----------------

******点击启动类，启动项目，就会发现每两秒执行一次。******

![](https://img-blog.csdnimg.cn/32d062eb43dc49f591ad6cba010e79dc.png)

![](https://img-blog.csdnimg.cn/39b057c56b7b47a0947ec65cec2e769e.png)

  

********注：触发器需要绑定任务，使用 forJob()操作传入绑定的工作明细对象。此处可以为工作明细设置名称然后使用名称绑定，也可以直接调用对应方法绑定。触发器中最核心的规则是执行时间，此处使用调度器定义执行时间，执行时间描述方式使用的是 cron 表达式。有关 cron 表达式的规则，各位小伙伴可以去参看相关课程学习，略微复杂，而且格式不能乱设置，不是写个格式就能用的，写不好就会出现冲突问题。********

********说明：spring 根据定时任务的特征，将定时任务的开发简化到了极致。要做定时任务总要告诉容器，然后定时执行什么任务直接告诉对应的 bean 什么时间执行就行了。********

******1、开启定时任务功能，在引导类上开启定时任务功能的开关，使用注解 @EnableScheduling******
--------------------------------------------------------------

```
<font color="red"><b><b><b><code class="language-java">@SpringBootApplication
//开启定时任务功能
@EnableScheduling
public class Springboot22TaskApplication {
    public static void main(String[] args) {
        SpringApplication.run(Springboot22TaskApplication.class, args);
    }
}
</code></b></b></b></font>
```

******2、定义 Bean，在对应要定时执行的操作上方，使用注解 @Scheduled 定义执行的时间，执行时间的描述方式还是 cron 表达式******
--------------------------------------------------------------------------------

```
<font color="red"><b><b><b><code class="language-java">@Component
public class MyBean {
    @Scheduled(cron = "0/1 * * * * ?")
    public void print(){
        System.out.println(Thread.currentThread().getName()+" :spring task run...");
    }
}
</code></b></b></b></font>
```

```
<font color="red"><b><b><b><code>    如何想对定时任务进行相关配置，可以通过配置文件进行
</code></b></b></b></font>
```

```
<font color="red"><b><b><b><code class="language-yaml">spring:
  task:
       scheduling:
      pool:
           size: 1              # 任务调度线程池大小 默认 1
      thread-name-prefix: ssm_        # 调度线程名称前缀 默认 scheduling-      
        shutdown:
          await-termination: false    # 线程池关闭时等待所有任务完成
          await-termination-period: 10s  # 调度线程关闭前最大等待时间，确保最后一定关闭
</code></b></b></b></font>
```

> ********总结：定时任务是工作中必须掌握的内容。希望对您有帮助，感谢阅读结束语：裸体一旦成为艺术，便是最圣洁的。道德一旦沦为虚伪，便是最下流的。勇敢去做你认为正确的事，不要被世俗的流言蜚语所困扰。********