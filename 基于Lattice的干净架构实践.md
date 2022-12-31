# 基于Lattice的干净架构实践
干净的架构（The Clean Architecture），这是著名软件大师 Bob 大叔提出的一种架构，也是当前各种语言开发架构。干净架构提出了一种单向依赖关系，从而从逻辑上形成一种向上的抽象系统。

> 同⼼圆分别代表了软件系统中的不同层次，通常越靠近中⼼，其所在的软件层次就越⾼。基本上，外层圆代表的是机制，内层圆代表的是策略。 当然这其中有⼀条贯穿整个架构设计的规则，即它的依赖关系规则：源码中的依赖关系必须只指向同⼼圆的内层，即由低层机制指向⾼层策略。

*   Entities（业务实体）： 通过领域分析与建模，我们可以将领域实体以及针对这些领域实体的生命周期管理的相关业务逻辑，封装在这一层。关于领域实体对象的分析与识别过程，可参考 我另一篇文章[《设计实战 – 场景分析 & 领域划分》](https://xie.infoq.cn/link?target=https%3A%2F%2Fwww.ryu.xin%2F2018%2F04%2F08%2Fdomain%2F)  
    
*   Use Cases (用例)： 用例层里一般放置和“特定场景”相关的逻辑，这些逻辑会对实体层定义的实体进行信息聚合。比如，我们以电商系统 “买家下单”这个场景为例，这个场景中，下单时会涉及到的实体有 商品对象、包裹、优惠券、买家信息、卖家信息、服务信息（比如配送服务）等。下单处理逻辑，需要对这些信息进行加工后，聚合成订单信息让用户确认或者创建。对这些实体根据特定场景进行编排的逻辑，它不属于任何一个实体所在的域。
    
*   接口适配器：将用例层的信息以某种形式对外暴露。比如，买家下单场景，我们可以以 Restful 的形式暴露接口，用于和 Web 前端集成等。
    

分层与逻辑构建划分的建议
------------

在构建一个复杂的业务系统中，我们需要考虑对实体访问的 API、SPI 设计，也要考虑在场景中去沉淀可复用的业务资产。可复用的业务资产要能做到跨项目、跨客户复用，就必然需要预留可扩展点 SPI。关于场景级开放，可参考 [面向场景级的业务资产沉淀和开放](https://xie.infoq.cn/article/02eb9bd35e90ca35c26055559)。 在经过笔者电信领域、电商领域、智慧城市领域多个大型项目的实践，同时参考了干净架构的一些建议，我总结了一套分层合理、非常适用于团队协作的逻辑工程划分方法。大体的工程结构和逻辑模块划分如下：

![](https://static001.geekbang.org/infoq/03/03180c57555b1ea6832d0e47be2a3253.png)

*   **业务实体层：** 通过 domain-sdk 对外暴露实体服务的接口定义，以及在对实体处理过程中，会有一些特殊的定制要求，也以 SPI 方式暴露出去。SPI 的表现形式，就是以扩展点的方式暴露出去，在运行期通过动态加载，获取具体的实现，从而实现对业务实体处理逻辑的扩展。
    
*   **用例层：** 在用例层中，主要是由各类场景的定义以及可复用的业务资产组成。场景在执行的过程中，可以安装这些业务资产来满足业务的需要。同时，这些业务资产在跨项目、跨行业的复用过程中，也可以预留一些可扩展点，以 SPI 形式暴露出去。从而做到，项目级定制和平台逻辑的分离。
    
*   **接口适配器（Endpoints）：** 主要由各类 web、interface 等各类对外接口形式存在，用于适配更外层相关设备/系统的接入需要。
    
*   **业务插件（Business Plugins)：** 业务插件主要是实现业务资产的 SPI，实现不同项目、不同行业中个性化业务逻辑的定制与增强。
    

以电商交易系统为例的工程划分样例
----------------

> 工程样例可以通过访问 [https://github.com/hiforce/lattice-clean-arch-practice-sample](https://xie.infoq.cn/link?target=https%3A%2F%2Fgithub.com%2Fhiforce%2Flattice-clean-arch-practice-sample) 获取

![](https://static001.geekbang.org/infoq/73/73e1e0b43435bd2d8239731d2bc58182.png)

在这个工程中，我们运行 org.hiforce.sample.buynow.web.starter.BuyNowWebStarter，然后打开浏览器输入 http://localhost:8080/buy/1 ，可以看到下面输出：

```
{
  "success": true,
  "orders": [
    {
      "orderId": 1,
      "buyerId": "rocky",
      "orderLines": [
        {
          "orderLineId": 1,
          "itemId": "2919311334001001",
          "buyQuantity": 1,
          "unitPrice": 4000
        }
      ]
    }
  ]
}
```

继续输入 http://localhost:8080/buy/2 ， 可进一步观察输出的变化，如下：

```
{
  "success": true,
  "orders": [
    {
      "orderId": 2,
      "buyerId": "rocky",
      "orderLines": [
        {
          "orderLineId": 2,
          "itemId": "2919311334001001",
          "buyQuantity": 1,
          "unitPrice": 10000
        }
      ]
    }
  ]
}
```

两个结果主要在商品单价上不一样，原因是业务定制包实现了预售的付款比例扩展点。 在不同场景下，预售资产会生效，从而导致单价会有不同。有兴趣的同学，可以自行 debug 做进一步观察。

扩展点在工程结构上的组织建议
--------------

### 业务实体层扩展点的定义建议

在实体层，一般我们会做一个领域扩展点门面，在这个扩展点门面下，会按照实体来进行分层定义。比如，在交易域，有 Order、OrderLine 等实体，那么交易域对外可扩展点门面，我们如下定义：

```
public interface TradeEntityExt extends IBusinessExt {

    TradeOrderExt getTradeOrderExt();

    TradeOrderLineExt getTradeOrderLineExt();

    TradePageExt getTradePageExt();
}
```

这样定义的好处是，开发人员、第三方 ISV 通过浏览该领域的可扩展门面，就能很容易按照实体来找到其对应的扩展点。我们继续以 OrderLine 这个实体为例，这个实体对外提供的扩展点，我们再进一步按照实体的“行为”进行分类，如下：

```
public interface TradeOrderLineExt extends IBusinessExt {

    
     * @return The extension of Order Line entity in init behavior.
     */
    TradeOrderLineInitExt getTradeOrderLineInitExt();

    
     * @return The extension of Order Line entity in check behavior.
     */
    TradeOrderLineCheckExt getTradeOrderLineCheckExt();

    
     * @return The extension of Order Line entity in saving behavior.
     */
    TradeOrderLineSaveExt getTradeOrderLineSaveExt();
}
```

当我们要想要对某个实体进行扩展时，很自然的一个想法就是要考虑是在对该实体做何种操作时需要扩展。比如，是实体在保存前要做个自定义加工处理，还是保存后要做个额外信息的补充？等等。在具体的实体行为扩展点门面下，才是具体的扩展点定义，比如上面 OrderLine 实体，我们在其“保存”行为上，定义如下扩展点：

```
public interface TradeOrderLineSaveExt extends IBusinessExt {

    @Extension(reduceType = ReduceType.NONE)
    Map<String, String> getCustomOrderLineAttributes(SaveOrderLineExtInput input);

    @Extension(reduceType = ReduceType.NONE)
    Void processOrderLineBeforeSaving(SaveOrderLineExtInput input);

    @Extension(reduceType = ReduceType.NONE)
    Void finalProcessOrderLineAfterSaving(SaveOrderLineExtInput input);
}
```

### 业务用例层扩展点的定义建议

和业务实体层不同，业务用例层里定义的场景，往往是跨业务活动的，是以业务流程形式进行组织与管理的。所以，在业务用例层，我建议扩展点门面是按照场景中所定义的业务活动来进行聚合。 我们在讨论需求时，往往会按照业务活动来讨论该如何实现需求。 比如，我们需要在 “买家下单”时，能够自定义某个下单提示组件；我们需要在 “卖家发货”时，能够定义最晚发货时长，等等。所以，按照“业务活动”来组织用例层的扩展点，会更加利于后期维护以及知识传播。

我们以预售交易业务资产为例，它对外提供的扩展点门面是 PreSaleTradeExt，定义如下：

```
public interface PreSaleTradeExt extends IBusinessExt {

    
     * @return The extensions of PreSale trade in buyer place order.
     */
    PreSalePlaceOrderExt getPreSalePlaceOrderExt();

    
     * @return The extensions of PreSale trade in fulfillment.
     */
    PreSaleFulfillmentExt getPreSaleFulfillmentExt();

    
     * @return The extensions of PreSale trade in refund.
     */
    PreSaleRefundExt getPreSaleRefundExt();
}
```

我分别按照卖家下单、卖家履约、买家退款等几个业务活动进行了聚合。 我们进一步以买家下单为例展开，可以看到预售交易业务资产在这个业务活动上定义了以下扩展点：

```
public interface PreSalePlaceOrderExt extends IBusinessExt {

    @Extension(name = "Custom PreSale Down Payment Ratio", reduceType = FIRST)
    Double getCustomDownPaymentRatio(PreSaleOrderLine orderLine);
}
```

在预售这个场景，业务开发人员以及第三方 ISV，可以很容易从预售交易 SDK 门面，按照业务活动快速找到这个扩展点，并实现业务定制。我以某业务 A 为例，他的定制实现如下：

```
@Realization(codes = SampleBusinessA.CODE)
public class BusinessAPreSaleExt extends BlankPreSaleTradeExt {
    @Override
    public BlankPreSalePlaceOrderExt getPreSalePlaceOrderExt() {
        return new BlankPreSalePlaceOrderExt() {
            @Override
            public Double getCustomDownPaymentRatio(PreSaleOrderLine orderLine) {
                return 0.4;
            }
        };
    }
}
```

总结
--

干净的架构是笔者经历过各种大型工程项目，在实战中的确颇有感悟和心得的一种架构。基于 Lattice 框架，可以很好的进行业务扩展点的管理，以及业务资产沉淀。希望本文，能对各位程序员、架构师朋友们有所帮助。

Lattice 是一个强大的、轻量级的，面向业务定制高可扩展的业务管理框架。通过使用 Lattice 框架，可以对复杂的业务定制进行高效的组织与管理。关于 Lattice 框架简介，可参考：[https://xie.infoq.cn/article/85fe94515a1ce0a8315550d62](https://xie.infoq.cn/article/85fe94515a1ce0a8315550d62)  

* * *

![](https://static001.geekbang.org/infoq/67/678f99ef3aa4707dede7a671a2b73586.jpeg?x-oss-process=image%2Fresize%2Cp_80%2Fauto-orient%2C1)