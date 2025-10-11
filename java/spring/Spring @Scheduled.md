# Spring @Scheduled


 `@Scheduler` 这个注解

如果你的项目中用到了这个注解，我真的建议你花几分钟看一下这个问题



废话不多说，我们一起来看看这个注解有啥隐藏的问题

朋友线上的架构如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa85884229e049de85f374a143b484b7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1939&h=876&s=94622&e=jpg&b=fceee4)

简单介绍下业务逻辑：

*   调度引擎将一些`SQL`封装成`SQL`任务提交至大数据平台
*   大数据平台执行任务返回结果于调度引擎执行后续逻辑
*   监控系统启动定时任务定时扫描`SQL Task`，若超过`90`分钟还未返回结果，重新提交该任务

排查之后发现：线上`SQL Task`全部超时导致离线表未产出

奇怪，按照上述的描述，我们的监控系统应该能识别到超时的任务并将其重新启动

当然，这种危机关头也不是查询原因的时候，赶紧先重新将这些任务启动再说

后面排查之后发现：**监控系统的超时扫描定时任务未启动，从而导致线上超时离线任务未重新提交**

这里先埋一个伏笔：**监控系统不止一个定时任务**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d73c136dc574e7da3423f86b4297b6a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=908&h=230&s=21032&e=jpg&b=dbe0fc)

OK~终于到了解密了时刻了

### 1、业务逻辑

我们先看下业务代码如何写的

```java
@SpringBootApplication
@EnableScheduling
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}

```

定时任务代码：

```java
@Component
public class SchedulerTest {

    
    @Scheduled(initialDelay = 5000, fixedRate = 5000)
    public void scheduled1() {
        System.out.println("I am scheduled1, thread = " + Thread.currentThread());
    }

    @Scheduled(initialDelay = 5000, fixedRate = 5000)
    public void scheduled2() {
        System.out.println("I am scheduled2, thread = " + Thread.currentThread());
    }

    @Scheduled(initialDelay = 5000, fixedRate = 5000)
    public void scheduled3() {
        System.out.println("I am scheduled3, thread = " + Thread.currentThread());
    }
}

```

启动程序输出：

```java
I am scheduled1, thread = Thread[scheduling-1,5,main]
I am scheduled2, thread = Thread[scheduling-1,5,main]
I am scheduled3, thread = Thread[scheduling-1,5,main]

```

通过输出你能看出来什么原因嘛？

### 2、报错逻辑

我们将上述的 `SchedulerTest`稍微修改一下：**将第一个定时任务改为死循环**

```java
@Component
public class SchedulerTest {

    
    @Scheduled(initialDelay = 5000, fixedRate = 5000)
    public void scheduled1() throws InterruptedException {
        while (true) {
            System.out.println("I am scheduled1, thread = " + Thread.currentThread());
            Thread.sleep(5000);
        }
    }

    @Scheduled(initialDelay = 5000, fixedRate = 5000)
    public void scheduled2() throws InterruptedException {
        while (true) {
            System.out.println("I am scheduled2, thread = " + Thread.currentThread());
            Thread.sleep(5000);
        }
    }

    @Scheduled(initialDelay = 5000, fixedRate = 5000)
    public void scheduled3() throws InterruptedException {
        while (true) {
            System.out.println("I am scheduled3, thread = " + Thread.currentThread());
            Thread.sleep(5000);
        }
        
    }
}

```

输出数据：

```java
I am scheduled1, thread = Thread[scheduling-1,5,main]
I am scheduled1, thread = Thread[scheduling-1,5,main]
I am scheduled1, thread = Thread[scheduling-1,5,main]
I am scheduled1, thread = Thread[scheduling-1,5,main]
I am scheduled1, thread = Thread[scheduling-1,5,main]
I am scheduled1, thread = Thread[scheduling-1,5,main]
I am scheduled1, thread = Thread[scheduling-1,5,main]
I am scheduled1, thread = Thread[scheduling-1,5,main]
I am scheduled1, thread = Thread[scheduling-1,5,main]

```

有没有发现什么问题？

### 3、原理揭露

通过上面两个案例，我们很明显的可以看出来：**我们 **`@Scheduled`**是单线程的！！！**

所以，当我们存在三个定时任务时，其中一个定时任务卡死，就会导致其余定时任务无法启动 从而造成线上故障

那问题来了，整个 `Spring`是如何对 `@Scheduled`解析和运行的呢，底层又是如何结合实现的

黄哥主打的就是源码，接下来开始源码揭露

### 4、源码揭露

我们可以直接看其源码：

#### 4.1 扫描注解

我们先从 `ScheduledAnnotationBeanPostProcessor`这个类看起

相信看过之前`Spring`源码解析系列的人应该认识这个这个类吧

对的，经典的后置处理器

主要的作用：**在`Bean`初始化过程中扫描目标`Bean`中的`@Scheduled`注解，为其创建相应的调度任务**

**浅挖下源码：** 

```java
public Object postProcessAfterInitialization(Object bean, String beanName) {
    
    
    Class<?> targetClass = AopProxyUtils.ultimateTargetClass(bean);

    
    
    if (!this.nonAnnotatedClasses.contains(targetClass) && AnnotationUtils.isCandidateClass(targetClass, Arrays.asList(Scheduled.class, Schedules.class))) {

        
        Map<Method, Set<Scheduled>> annotatedMethods = MethodIntrospector.selectMethods(targetClass,
                                                                                        (MethodIntrospector.MetadataLookup<Set<Scheduled>>) method -> {
                                                                                            Set<Scheduled> scheduledAnnotations = AnnotatedElementUtils.getMergedRepeatableAnnotations(
                                                                                                method, Scheduled.class, Schedules.class);
                                                                                            return (!scheduledAnnotations.isEmpty() ? scheduledAnnotations : null);
                                                                                        });

        
        if (annotatedMethods.isEmpty()) {
            this.nonAnnotatedClasses.add(targetClass);
        }else {
            
            annotatedMethods.forEach((method, scheduledAnnotations) ->
                                     scheduledAnnotations.forEach(scheduled -> processScheduled(scheduled, method, bean)));
            if (logger.isTraceEnabled()) {
                logger.trace(annotatedMethods.size() + " @Scheduled methods processed on bean '" + beanName +
                             "': " + annotatedMethods);
            }
        }
    }
}

```

比如上述例子： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff94e9bbc3454d4faabcb7f66e6ee715~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2051&h=703&s=184781&e=png&b=232427)

#### 4.2 封装任务

在上面的 `processScheduled(scheduled, method, bean)`方法里面，封装了我们的任务

```java
protected void processScheduled(Scheduled scheduled, Method method, Object bean) {
    
    Runnable runnable = createRunnable(bean, method);

    
    long initialDelay = convertToMillis(scheduled.initialDelay(), scheduled.timeUnit());
    
    String cron = scheduled.cron();

    
    
    long fixedDelay = convertToMillis(scheduled.fixedDelay(), scheduled.timeUnit());
    if (fixedDelay >= 0) {
        tasks.add(this.registrar.scheduleFixedDelayTask(new FixedDelayTask(runnable, fixedDelay, initialDelay)));
    }

    long fixedRate = convertToMillis(scheduled.fixedRate(), scheduled.timeUnit());
    if (fixedRate >= 0) {
        tasks.add(this.registrar.scheduleFixedRateTask(new FixedRateTask(runnable, fixedRate, initialDelay)));
    }
}

```

所以，我们的重点就在这一句了：`tasks.add(this.registrar.scheduleFixedRateTask(new FixedRateTask(runnable, fixedRate, initialDelay)))`

```go
public ScheduledTask scheduleFixedRateTask(FixedRateTask task) {
    ScheduledTask scheduledTask = this.unresolvedTasks.remove(task);
    boolean newTask = false;
    if (scheduledTask == null) {
        scheduledTask = new ScheduledTask(task);
        newTask = true;
    }
    
    if (this.taskScheduler != null) {
        if (task.getInitialDelay() > 0) {
            Date startTime = new Date(this.taskScheduler.getClock().millis() + task.getInitialDelay());
            scheduledTask.future =
            this.taskScheduler.scheduleAtFixedRate(task.getRunnable(), startTime, task.getInterval());
        }
        else {
            scheduledTask.future =
            this.taskScheduler.scheduleAtFixedRate(task.getRunnable(), task.getInterval());
        }
    }
    else {
        
        addFixedRateTask(task);
        this.unresolvedTasks.put(task, scheduledTask);
    }
    return (newTask ? scheduledTask : null);
}

```

我们在第一次运行时，该函数只会向`fixedRateTasks`中添加对应的`task`

#### 4.3 任务运行

我们继续回到 `ScheduledAnnotationBeanPostProcessor`中，找到 `onApplicationEvent`这个方法

> onApplicationEvent是Spring框架中的一个方法，用于处理应用程序事件。它是ApplicationListener接口的方法之一，用于响应特定类型的应用程序事件。

```go
public void onApplicationEvent(ContextRefreshedEvent event) {
		if (event.getApplicationContext() == this.applicationContext) {
			
			
			
			finishRegistration();
		}
	}
private void finishRegistration() {
    if (this.registrar.hasTasks() && this.registrar.getScheduler() == null) {
        this.registrar.setTaskScheduler(resolveSchedulerBean(this.beanFactory, TaskScheduler.class, false));
    }

    
    this.registrar.afterPropertiesSet();
}

```

OK，这一句也就是我们的重点：`this.registrar.setTaskScheduler(resolveSchedulerBean(this.beanFactory, TaskScheduler.class, false));`

```go

public void setTaskScheduler(TaskScheduler taskScheduler) {
    this.taskScheduler = taskScheduler;
}

```

最后执行 `afterPropertiesSet`方法

```go
public void afterPropertiesSet() {
    scheduleTasks();
}

protected void scheduleTasks() {
    if (this.fixedRateTasks != null) {
        for (IntervalTask task : this.fixedRateTasks) {
            addScheduledTask(scheduleFixedRateTask(task));
        }
    }
}

public ScheduledTask scheduleFixedRateTask(FixedRateTask task) {
    
    if (this.taskScheduler != null) {
        if (task.getInitialDelay() > 0) {
            Date startTime = new Date(this.taskScheduler.getClock().millis() + task.getInitialDelay());
            
            scheduledTask.future =
            this.taskScheduler.scheduleAtFixedRate(task.getRunnable(), startTime, task.getInterval());
        }
        else {
            scheduledTask.future =
            this.taskScheduler.scheduleAtFixedRate(task.getRunnable(), task.getInterval());
        }
    }
}

```

那这里是如何提交任务运行的呢

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1f6827aaefa49a99004e9014a9e460e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1354&h=380&s=72877&e=png&b=1e1f22)

继续往下追代码：

```go
ScheduledFuture<?> scheduleAtFixedRate(Runnable task, Date startTime, long period);

public ScheduledFuture<?> scheduleAtFixedRate(Runnable task, Date startTime, long period) {
	
    ScheduledExecutorService executor = getScheduledExecutor();
		long initialDelay = startTime.getTime() - this.clock.millis();
		try {
            
			return executor.scheduleAtFixedRate(errorHandlingTask(task, true), initialDelay, period, TimeUnit.MILLISECONDS);
		}
		catch (RejectedExecutionException ex) {
			throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, ex);
		}
	}

```

OK，到这里我们基本梳理清楚了

但还有最后一个问题，那就是 `ThreadPoolTaskScheduler`是如何来的？

回到我们一开始的`this.registrar.setTaskScheduler(resolveSchedulerBean(this.beanFactory, TaskScheduler.class, false));`

我们可以看到 `resolveSchedulerBean(this.beanFactory, TaskScheduler.class, false)`中使用 `TaskScheduler.class`

继续往下看源码

```java
private <T> T resolveSchedulerBean(BeanFactory beanFactory, Class<T> schedulerType, boolean byName) {
    if (beanFactory instanceof AutowireCapableBeanFactory) {
        
        NamedBeanHolder<T> holder = ((AutowireCapableBeanFactory) beanFactory).resolveNamedBean(schedulerType);
        if (this.beanName != null && beanFactory instanceof ConfigurableBeanFactory) {
            ((ConfigurableBeanFactory) beanFactory).registerDependentBean(holder.getBeanName(), this.beanName);
        }
        return holder.getBeanInstance();
    }
    else {
        return beanFactory.getBean(schedulerType);
    }
}

```

在往下追的话：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e60f5a4b802045f89b8472f2060557c0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1997&h=225&s=79947&e=png&b=25262a)

这个类应该不用介绍了吧，讲了几百遍了

那为什么我们通过 `TaskScheduler.class`可以得到 `ThreadPoolTaskScheduler`呢？

很简单，因为这哥们实现了`TaskScheduler`接口

```java
public class ThreadPoolTaskScheduler extends ExecutorConfigurationSupport
		implements AsyncListenableTaskExecutor, SchedulingTaskExecutor, TaskScheduler {

```

但这哥们有个问题：

```java
public class ThreadPoolTaskScheduler extends ExecutorConfigurationSupport
implements AsyncListenableTaskExecutor, SchedulingTaskExecutor, TaskScheduler {
    private volatile int poolSize = 1;
    
    
	 * Set the ScheduledExecutorService's pool size.
	 * Default is 1.
	 * <p><b>This setting can be modified at runtime, for example through JMX.</b>
	 */
    public void setPoolSize(int poolSize) {
        Assert.isTrue(poolSize > 0, "'poolSize' must be 1 or higher");
        if (this.scheduledExecutor instanceof ScheduledThreadPoolExecutor) {
            ((ScheduledThreadPoolExecutor) this.scheduledExecutor).setCorePoolSize(poolSize);
        }
        this.poolSize = poolSize;
    }
}

```

是的，这坑爹的`ThreadPoolTaskScheduler`线程池默认为`1`

然后所有使用 **`@Scheduled`的定时任务共有这一个注解**

所以，我们任务如果一个运行不完，其余任务都在阻塞着

这就是问题的源码根因

### 4、解决方式

有两种解决方式：

*   增加配置类
*   配置文件新增配置

#### 4.1 增加配置类

```java
@Configuration
public class ScheduleConfig {
    
     * 修复同一时间无法执行多个定时任务问题。
     * @Scheduled默认是单线程的
     */
    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        
        taskScheduler.setPoolSize(Runtime.getRuntime().availableProcessors() * 2);
        return taskScheduler;
    }
}

```

#### 4.2 配置文件

```properties
spring.task.scheduling.pool.size=10

```

