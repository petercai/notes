# 如何理解微服务体系结构中的 CQRS
> 本文翻译自 [How To Understand CQRS In Microservices Architecture](https://link.juejin.cn/?target=https%3A%2F%2Fdatamify.com%2Farchitecture%2Fhow-to-understand-cqrs-in-microservices-architecture%2F "https://datamify.com/architecture/how-to-understand-cqrs-in-microservices-architecture/")，原作者 OLEKSII。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93dd15a6b9ee44fa8151f74c492b5c20~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1280&h=723&s=70493&e=jpg&b=ffffff)

问题描述
----

在典型的软件应用程序中，有一个负责写入和读取操作的数据存储。通常，应用程序实现一些 CRUD 操作，并且非常简单。你存储了一些东西并读取了相同的结果。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee1bcab83f8a48f4954fedb8906fade5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=201&h=443&s=15474&e=jpg&b=fffcfc)

然而，在复杂的应用程序中，情况并不是那么简单。每个`读`和`写`操作都需要很多操作和约束。

例如：

*   `写`：数据库是以规范化的方式构造的，以消除数据重复并强制约束。`读`：复杂的读取查询需要大量对性能产生负面影响的联接（joins）/联合(unions)/聚合（aggregations）。
*   `读`：优化查询需要索引。`写`：每次`写`操作都会更新索引。
*   读/写表示可能不同，这可能是附加逻辑的原因。
*   在对同一数据进行并行操作期间，可能会发生数据争用。
*   由于数据库的负载，性能可能很差。

* * *

CQRS
----

上述问题的解决方案是 `CQRS`（Command Query Responsibility Segregation）。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/832cb2ab1ffa41c089b0a72c54faf679~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=500&h=393&s=31871&e=jpg&b=fefcfb)

让我们回顾一下上面的图片。它包含两个服务：_旅馆管理服务_和_旅馆预订服务_（实际上，如果需要的话，可以有多个不同的_预定服务_及其各自的数据库）。这些服务之间的数据应该同步。为此，通常使用一些消息代理（例如 Kafka。这些服务的数据格式不同。

_旅馆管理服务_负责`创建`/`更新`/`删除`操作，每个操作都被描述为一个命令：

*   生成/批准/拒绝一个酒店创建请求。
*   添加/更新酒店房间到系统。
*   更改酒店房间的价格。
*   预订特定时间段的酒店房间。
*   请求其他服务。
*   生成请求留下/批准/拒绝评论。
*   取消预订。

_旅馆预订服务_仅负责向用户显示酒店/房间列表：

*   列出一个城市中所有/可用的酒店房间。
*   在地图上显示所有酒店（按地理位置过滤）。
*   向用户显示可用的过滤选项。
*   通过指定的过滤器过滤酒店房间（例如，显示两个酒店房间，而不是一个多人房间）。

正如你所见，服务提供的责任是不同的。此外，指定服务的性能约束也不同。用户在预订之前通常会经过许多预订选项。因此，在这个系统中，`读`操作的数量远远高于`写`操作的数量。因此，这些服务的缩放将有所不同。

因此，一些 RDBMS 可以用作_旅馆管理服务_的数据存储，例如 MySQL或 PosgreSQL。RDBMS 将为保持数据安全提供它的所有好处。

_旅馆预订服务_可以基于一些 NoSQL 解决方案，如 MongoDB或 Elasticsearch。这背后的原因是执行高效搜索查询的能力：反规范化、地理位置搜索支持、索引、水平缩放等。

### CQRS 优势

*   关注点分离。
*   单个读写服务缩放。
*   适合读写操作的数据存储及其模式（schema）。
*   简单高效的查询。

### CQRS 劣势

*   复杂性。
*   最终一致性。

* * *

在本文中，我们回顾了 `CQRS` 模式。值得一提的是，这种模式的必要性可能发生在复杂的系统中。因此，应谨慎选择，以免给系统带来不必要的复杂性。

* * *