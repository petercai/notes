# GraphQL实战经验
> 本文作者分享了在生产环境中使用 GraphQL 的一些经验和解决方法，并给出了一些构建实用 GraphQL 查询和变更（Mutation）的建议。

_本文最初发布于_[Medium](https://medium.com/better-programming/2-years-of-graphql-in-production-a1a6dbaddbda)_，经原作者 Stein Janssen 授权由 InfoQ 中文站翻译并分享。_

[GraphQL](https://graphql.org/)已得到广泛认可并日益流行。我们在使用中遇到了一些非常有挑战性的问题，值得撰文分享。本文将使用一个示例配置来阐释问题，并给出相应的解决方法。

![](https://static001.infoq.cn/resource/image/57/65/579da5c1ec3c8dca78faa361d1714d65.png)

本文作者使用 GraphQL Voyager 生成的关系概览图

首先谈谈我们为什么会选择 GraphQL?

*   无需操心如何更新文档，所有的查询（Query）和变更会自动形成文档。
    
*   无需获取整个数据集，我们可以编写仅仅返回所请求数据的查询。
    
*   对前端提供统一的访问点。从数十个不同 API 中获取数据并非易事。GraphQL 支持开发人员将所有 API 进行拼接（Stitching）。
    

拼接（Stitching）
-------------

拼接（Stitching）让我们可以从同一端点获取所有数据。这听上去不错，但它也会导致一些非常棘手的问题。举例说明：

![](https://static001.infoq.cn/resource/image/66/1b/661ceb0389f98accafe94e7715f3a21b.png)

如上图所示，一个前端与 Public API 通信。Public API 拼接了 Order API，后者又拼接了 Product API。前端唯一能访问的是 Public API。看上去这种链式拼接方式并没有太大的问题，但是在面对数十层级的 API 拼接时，应用发布将成为一场灾难。

问题出在哪里？一旦 Product API 的 Schema 发生更改，那么需要依次重新启动 Order 和 Public API 才能重新加载 Schema。GraphQL Schema 每次更新时，都必须重新启动多个 API。这非常繁琐。

另一个可能出现的问题是，如果应用需要逆链反向查询，而非顺链而下查询，这时拼接无法工作。例如从 Product 访问 Order，由于 Order API 需要加载 Product 的 Schema，因此只有在 Product API 运行时才能启动 Order API。

这意味着，虽然可以获取属于给定 Order 的 Product：

```
order {
    identifier
    products {
        identifier
    }
}
```

但无法获取给定 Product 所在的 Order：

```
products {
    identifier
    order {
        identifier
    }
}
```

为解决这个问题，我们需要重新拼接 API。我们可以让 Public 负责加载所有的 Schema。这样只需重启该 API，即可加载所有 Schema。

![](https://static001.infoq.cn/resource/image/0d/9e/0d1fa894eyyd66019a40f1f6b7a1e39e.png)

鉴于现在 Public API 获取所有的 Schema，我们可以添加处理 order 属性的代码，扩展 Product 的 Schema。这样，我们就可以通过查询 Product 获取 Order 信息。扩展 Schema 的详细做法，参见[Apollo文档](https://www.apollographql.com/docs/apollo-server/federation/entities/#extending)。

现在，我们并不需要通过 Public API 获得所有查询和变更的访问权。例如，我们并不想让客户能够通过触发变更去更改支付的状态。对此，一种解决方法是过滤掉特定查询和变更。具体而言，应用遍历 Schema 中所有的查询和变更，并与给定的列表做对比。如果查询存在于列表中，则设为可见。如果不在列表中，就从 Schema 中移除。另一个解决方法是添加中间件，由中间件检查当前用户是否有权限触发特定的查询和变更。

实践中，我们组合使用了上面两种方法。但现在我们面对一个新的问题。当前并非所有的变更都可通过 Public API 访问，因此在更新支付的状态时需要直接调用 Payment API。这对于变更不存在问题，但并不适用于所有的查询，因为父对象和子对象只是在 Public API 做拼接。为解决这个问题，我们需要再次重新编排配置，如下图所示：

![](https://static001.infoq.cn/resource/image/65/8d/6531cce20d44fc15b4da8a6628fe1e8d.png)

这里，我们新建了一个 Gateway API，负责拼接所有 Schema。而 Public API 只拼接 Gateway API，并移除所有前端无需访问的查询和变更。这样，Gateway 可与后端服务部署在同一网络，后端在进行查询和变更时可直接使用 Gateway API。

查询分页（Paginated）
---------------

一些情况下，实现[查询分页](https://graphql.org/learn/pagination/#pagination-and-edges)很有必要。我们采用了基于游标的方法，实践中很好用。需要获取 Product 时，可使用如下查询：

```
products(first: 5, after: "cursor") {
    edges {
        node {
            identifier
        }
    }
}
```

但是，由于我们更改了 Schema，在获取 Product 对应的 Order 时会生成分页结构：

```
order {
    products(first: 5, after: "cursor") {
        edges {
            node {
                identifier
            }
        }
    }
}
```

但是这里我们并不需要分页结构，因为给定 Order 的 Product 数量并不多。针对该问题，我们考虑分别编写两个查询，一个实现了分页，另一个则不考虑分页。另一个做法是针对拼接 Product 到 Order 的情况，使用 Schema 包装（Schema Wrapping）移除分页。Schema 包装是一个非常强大的方法，尤其是针对同一 API 种拼接了所有远程 Schema 的情况。详细信息，参见[Schema包](https://www.graphql-tools.com/docs/schema-wrapping/)[装的官方文档](https://www.graphql-tools.com/docs/schema-wrapping/)。

一些建议
----

上面我们列举了部分主要问题。下面给出一些有助于构建可维护 GraphQL API 的小建议：

*   类型和枚举的命名必须唯一。例如，如果需要对 Product 添加一个状态，建议命名为 ProductStatus，而不要直接使用 Status，以免出现类型冲突问题。
    
*   建议在查询中添加过滤，以免额外单独编写查询。推荐一个[很好的查询实现例子](https://swapi.graph.cool/)，访问页面右侧的“doc”选项卡,并搜索 assetFilter。
    
*   对查询和变更定义自己的命名规则，以简化对查询和变更的查找。
    
*   在使用查询分页时，设置默认值和最大上限。以免他人运行 API 时导致崩溃。
    
*   推荐使用[GraphQL Voyager](https://github.com/APIs-guru/graphql-voyager)，可生成对 Schema 所有查询、变更、关系的概览图。
    

**原文链接：**  [2 Years of GraphQL in Production](https://medium.com/better-programming/2-years-of-graphql-in-production-a1a6dbaddbda)