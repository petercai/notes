# 业务设计——责任链验证推翻 if-else 炼狱
1\. 什么是责任链模式
------------

        在责任链模式中，多个处理器依次处理同一个请求。一个请求先经过 A 处理器处理，然后再把请求传递给 B 处理器，B 处理器处理完后再传递给 C 处理器，以此类推，形成一个链条，链条上的每个处理器各自承担各自的处理职责。

![](_assets/0ef5bd999fe749739a9bde36a5d62cae~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

2\. 责任链模式优点
-----------

        责任链模式的优点在于，它可以动态地添加、删除和调整处理者对象，从而灵活地构建处理链。同时，它也避免了请求发送者和接收者之间的紧耦合，增强了系统的灵活性和可扩展性。

* * *

购票请求验证
------

        在实际购票业务场景中，用户发起一次购票请求后，购票接口在真正完成创建订单和扣减余票行为前，需要验证当前请求中的参数是否正常请求，或者说是否满足购票情况。

1.  购票请求用户传递的参数是否为空，比如：车次 ID、乘车人、出发站点、到达站点等。
2.  购票请求用户传递的参数是否正确，比如：车次 ID 是否存在、出发和到达站点是否存在等。
3.  需要购票的车次是否满足乘车人的数量，也就是列车对应座位的余量是否充足。
4.  乘客是否已购买当前车次，或者乘客是否已购买当天时间冲突的车次。
5.  可能实际场景中需要验证的还有很多，就不逐一举例了。

对于完成这些前置校验逻辑，示例代码如下：

```typescript
public TicketPurchaseRespDTO purchaseTickets(PurchaseTicketReqDTO requestParam) {
	
	
	
	
	
}

```

        解决前置校验需求需要实现一堆逻辑【**if-else炼狱**】，常常需要写上几百上千行代码。并且，上面的代码不具备开闭原则，以及代码扩展性，整体来说复杂且臃肿。

        为了避免这种坏代码味道【**shi山**】，我们可以运用责任链设计模式，对购票验证逻辑进行抽象。把每一个if的判断抽象为一个handler过滤器，多个if就化解成了一个过滤器链（职责链），这样子就可以避免在service业务层写太多校验逻辑，让代码更加清爽！并且如果后续还需要新增说明校验的判断，直接新增一个处理器对象就行，无需改动源代码，符合开闭原则

* * *

下面我们以一个购买商品的业务来搭建责任链架构

1.定义所有过滤链的抽象接口
--------------

        为了方便对责任链流程中的任务进行顺序处理，我们需要继承 Spring 框架中的排序接口 Ordered。这将有助于保证责任链中的处理器的顺序执行

```csharp
public interface AbstractChainHandler <T> extends Ordered { 
    
     * 在这个方法里面定义过滤器要干的事儿，每个过滤器链实例通过重写该方法来实现响应的处理逻辑
     * @param requestParam 请求的参数（过滤链要加工校验的对象实际上就是源自于前端传过来的参数）
     */
    void handler(T requestParam);

    
     * 一个项目可以有多条过滤链，得通过mark来锁定指定的那条链，也就是说mark就是众多处理器的划分参考
     * @return 一条过滤链组件标识
     */
    String mark();
}

```

2.定义职责链初始化器和启用函数
----------------

这个类中只要做两件事：

*   项目一启动就把多个过滤器链初始化加载好
*   准备好调用过滤器链路的总开关函数，我传入一条过滤链的标识mark和请求参数requestParam，你就得给我把这个链路跑起来依次调用我的处理器去校验

```java
public final class AbstractChainContext<T> implements CommandLineRunner { 
                            

    
     * 初始化好的一条条过滤器链都放到集合容器中存好 Key：过滤器链的标识mark   Value：过滤器链中的处理器集合
     */
    private final Map<String, List<AbstractChainHandler>> abstractChainHandlerContainer = new HashMap<>();

    
     * 把这个mask对应的链路跑起来依次调用我的处理器去校验
     * @param mark         过滤链组件标识
     * @param requestParam 请求参数
     */
    public void handler(String mark, T requestParam) {
        
        List<AbstractChainHandler> abstractChainHandlers = abstractChainHandlerContainer.get(mark);
        if (CollectionUtils.isEmpty(abstractChainHandlers)) {
            throw new RuntimeException(String.format("[%s] Chain of Responsibility ID is undefined.", mark));
        }
        
        abstractChainHandlers.forEach(each -> each.handler(requestParam));
    }

    
     * 初始化项目中的所有过滤链
     */
    @Override
    public void run(String... args) throws Exception {
        
        Map<String, AbstractChainHandler> chainFilterMap = ApplicationContextHolder
                .getBeansOfType(AbstractChainHandler.class);
        
        chainFilterMap.forEach((beanName, bean) -> {
            List<AbstractChainHandler> abstractChainHandlers = abstractChainHandlerContainer.get(bean.mark());
            if (CollectionUtils.isEmpty(abstractChainHandlers)) {
                abstractChainHandlers = new ArrayList();
            }
            abstractChainHandlers.add(bean);
            
            List<AbstractChainHandler> actualAbstractChainHandlers = abstractChainHandlers.stream()
                    .sorted(Comparator.comparing(Ordered::getOrder))
                    .collect(Collectors.toList());
            abstractChainHandlerContainer.put(bean.mark(), actualAbstractChainHandlers);
        });
    }

```

3.定义一条责任链的抽象
------------

在这里，我们定义一个针对购买操作的责任链的抽象，这样子就可以将处理器实例通过接口进行一个划分

```java
public interface PurchaseFilter <T extends 请求参数的封装类> extends AbstractChainHandler<请求参数的封装类> {

    @Override
    default String mark() {
        
        return FilterChainMarkEnum.PURCHASE_FILTER.name();
    }
}

```

4.定义责任链中的处理器实例
--------------

定义查询该商品库存是否满足购买数量的处理器

```java
@Component
public class QueryHandler implements PurchaseFilter {

    @Override
    public void handler(PurchaseTicketReqDTO requestParam) {
        
    }

    @Override
    public int getOrder() {
        return 处理器在链路中的order值;
    }
}

```

定义查询用户余额是否充足的处理器

```java
@Component
public class HavaMoneyHandler implements PurchaseFilter {

    @Override
    public void handler(PurchaseTicketReqDTO requestParam) {
        
    }

    @Override
    public int getOrder() {
        return 处理器在链路中的order值;
    }
}

```

**如果还有什么校验逻辑就直接加处理器就行，不需要动原代码，满足开闭原则**

业务层代码实现
-------

```java
private AbstractChainContext abstractChainContext = new AbstractChainContext();

public void purchaseService(T requestParam){
    try {
        
        abstractChainContext.handler(FilterChainMarkEnum.PURCHASE_FILTER.name(), requestParam);
    } catch (Exception e) {
        
        log.error({"职责链中报错：{}"}, e.getMessage());
    }
    
    xxxxxxxxx
}

```

        经过责任链模式的重构，你是否发现业务逻辑变得更加清晰易懂了？采用这种设计模式后，增加或删除相关的业务逻辑变得非常方便，不再需要担心更改上千行代码的几行代码，导致整个业务逻辑受到影响的情况。

本文详细介绍了责任链模式的概念，并通过商品下单场景模拟了真实使用场景。

        为了复用责任链接口定义和上下文，我们通过抽象的方式将责任链门面接口加入到基础组件库中，实现快速接入责任链的目的。

        虽然在本文中，我们没有使用 boolean 类型的返回值，而是通过异常来终止流程，但在后续的增强中，我们可以考虑添加布尔类型的返回值。

        此外，我们还可以在 `AbstractChainHandler` 中增加是否异步执行的方法，以提高方法执行性能和减少接口响应时间。

        架构设计总是在不断演进，本文的设计也有优化和进步的空间，让我们继续探索责任链模式的更多可能性。