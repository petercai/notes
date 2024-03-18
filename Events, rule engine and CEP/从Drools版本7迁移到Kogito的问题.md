# 从Drools版本7迁移到Kogito的问题
Kogito是一种用于构建可扩展和可移植业务[应用](http://www.volcengine.com/product/cp)程序的框架。它是基于Drools的，但是有一些不同之处。以下是从Drools v7迁移到Kogito的步骤：

1.更新pom.xml文件以使用最新[版](http://www.volcengine.com/product/flink)本的Kogito[引擎](http://www.volcengine.com/product/mse)：

```
<dependency>
    <groupId>org.kie.kogito</groupId>
    <artifactId>kogito-api</artifactId>
    <version>${kogito.version}</version>
</dependency>

<dependency>
    <groupId>org.kie.kogito</groupId>
    <artifactId>kogito-quarkus-rules</artifactId>
    <version>${kogito.version}</version>
</dependency>

```

2.更改DRL文件的语法。Kogito使用基于Qu[ark](https://www.volcengine.com/product/ark)us的Kogito编译器，因此语法略有不同。以下是一个示例DRL文件：

Drools v7语法：

```
rule "rule1"
    when
        $order : Order()
    then
        $order.setDiscount(10);
        modify($order);
end

```

Kogito语法：

```
rule rule1
    dialect "mvel"
when
    $order : Order()
then
    $order.setDiscount(10);
    update($order);
end

```

3.修改Java代码以使用Kogito [API](http://www.volcengine.com/product/apig)代替Drools [API](http://www.volcengine.com/product/apig)。以下是一个示例：

Drools v7代码：

```
KieSession ksession = kContainer.newKieSession();
ksession.insert(order);
ksession.fireAllRules();

```

Kogito代码：

```
RuleUnit<OrderDetails> ruleUnit = RuleUnitFactory.create(unit.(OrderDetails.class));
ruleUnit.bind(orderDetails);
ruleUnit.evaluate();

```

这些是从Drools v7迁移到Kogito的基本步骤，但是具体的迁移可能会因[应用](http://www.volcengine.com/product/cp)程序而异。

