# 为什么你的下一个API应该是GraphQL而不是REST
> REST 在几年前曾经风靡一时，但现在 GraphQL 拥有更好的工具和开发者体验。

几年前，我所有的 API 都是[REST](https://www.infoq.cn/article/jfkZ7LHF1HbONN2sPOyw "xxx") API。我知道 GraphQL 不可小觑，但也并没有花太多时间学习它。早期的探索表明，[GraphQL](https://www.infoq.cn/video/YIShfqdvkUw0M8PC9DYu "xxx")并不是一颗神奇的 API 银弹。你仍然需要编写所有的逻辑，仍然需要使用奇怪的模式文件。既然如此，我们为什么要选择为额外的复杂性而烦恼呢？REST API 很简单，不需要额外的模式。每个资源都有简单的端点，大多数时候，与 API 端点相关的代码都是简单的 CRUD。

在过去的一年里，我们从 RPC API 切换到更好的 GraphQL。这对我们的应用程序来说是件好事，因为 RPC 接口只是 GraphQL 的一个糟糕实现，而且应用程序中有复杂的关系，因此使用 GraphQL 更为合适。

虽然我对此持怀疑态度，但也很想看看事情将会如何发展。事实证明，现在的工具非常棒，而且这样做的好处是巨大的。

一年后，我迷上了它。我的朋友开始了一个业余项目，当我想到需要再次与一堆 REST API 打交道时，我的心一沉。GraphQL 提供了更好的开发者体验。那么，为什么 GraphQL 如此神奇？

在编写处理请求的代码时，处理逻辑的主要代码几乎是相同的。

例如，这是一些获取给定 ID 资源的伪代码。

```
func resolver(parent, args, context): Resource {
  const id = args.id
  const resource = db.Resource.FindOne({id: id})
  return resource
}
```

通过 ID 获取资源（GraphQL）

这是 GraphQL 还是 REST 的端点？

代码几乎一模一样。主要的区别在于函数签名。获取、更新和删除资源集合也是如此。

二者都是将参数传递给函数并处理请求。有一些小的语法变化，但大部分逻辑保持不变。

不过 GraphQL 有一个值得注意的好处，就是可以获取资源间的关系。

假设你有一个包含两个资源的模式：一个 Author（作者），它有许多 Post（帖子）。你只能根据作者获得帖子，所以你的 REST API 看起来像这样。

```
/authors/
/authors/:author_id/
/authors/:author_id/posts
/authors/:author_id/posts/:post_id
```

你实现了四个不同的 API 来获取每个层级的数据。

但你可以使用 GraphQL 更简单地达到同样的目的。

```
type Query {
  authors(id: Int): [Author!]!
}

type Author {
  posts(id: Int): [Post!]!
}
```

还有其他方法也可以实现这一点，但这种模式可以让你：

1.  不指定 ID 获取所有作者（这是可选的）；
    
2.  通过特定的 ID 获取作者；
    
3.  获取所有作者的所有文章；
    
4.  获取单个作者的所有文章；
    
5.  只获取给定作者的给定文章。
    

就我个人而言，我的根查询经常将 author 和 authors 分开，避免为单个资源返回一个数组。将 Post 放在 Author 下，然后在 Author 的上下文中编写 Post 解析器。这个简单的嵌套让用户可以一起查询作者和他们的帖子。

GraphQL 是一种强类型模式，可以为客户端和服务器库生成代码。

因此，客户端可以获取具有完美类型信息的数据。与手动编写的 REST API 相比，GraphQL 的类型安全是一个巨大的优势。有些工具可以为 REST API 生成类型，例如 Swagger/OpenAPI，但这些工具并没有内置到规范中，所以你不会自动获得这些功能。

GraphQL 在模式中内置了注释，还提供了一个自检 API，可以实现自文档化。有了编写良好的注释，就不需要单独维护文档。

类似的，因为模式是强类型的，所以实际上你都不需要构建自定义客户端库。服务通常会提供客户端库，这些库提供了易于使用的类型和方法。GraphQL 内置了这些，你只需要用它编写一个客户端。

你还可以获得为服务器解析器构建的类型。有些语言，比如 Go，会生成整个解析器函数。你需要做的是填充内容。其他的，比如 TypeScript，会生成所有的类型，你可以在解析器中使用它们来保证类型安全。

你在 GraphQL 模式中定义数据的类型，这为你提供了一种便利的方式来剔除敏感信息。例如，假设你有一个 User 类型，它映射到数据库中的一条用户记录，你可能在对象种保存了个人信息，如电子邮件地址或散列过的密码。

如果没有合适的工具，你可能会这么操作：

```
func handler(request, response) {
  const user = db.User.findCurrent()
  
  return user
}
```

它会返回用户的所有字段。你需要把敏感信息剔除掉。

```
func handler(request, response) {
  const user = db.User.findCurrent()
  delete user.email
  delete user.password
  return user
}
```

GraphQL 通过特定的类型（在 Go 或 TypeScript 中）可以自动完成这个操作，只公开模式中定义的字段。你可以将对象返回给 GraphQL 解析器，并只公开模式中定义的字段。

测试 GraphQL 也变得更容易了，因为一些工具内置了强大的支持。我目前最喜欢的是 Insomnia，用于在应用程序之外测试 GraphQL 查询。Insomnia 会获取模式，并提供自动完成查询的功能，支持变量输入。此外，你还可以导出项目并将其包含在源代码中，方便人们进行快速的探索和使用它们。

还有其他一些很好的工具，比如 Apollo（[https://www.apollographql.com/](https://www.apollographql.com/)）。

随着时间的推移，我注意到 REST 有一个缺点——并不是每个操作都能很好地映射成 CRUD。有一类操作可以被映射成这种格式，但可能没有意义。

*   一次创建多条记录；
    
*   一次更新多条记录；
    
*   启动长时间运行的作业；
    
*   取消作业。虽然我相信你可以写出有意义的 REST API（使用 POST /job 启动作业，它将返回 HTTP 202 而不是 200！），但也存在争议，例如，它究竟是取消作业还是删除作业，还是修改作业？
    

将批量更新作为一个资源，还是操作多个资源？

REST 没有针对这些操作提供有意义的语义定义，而 GraphQL 的 mutation 可以被任意命名，这样你就可以：

```
mutation cancelJob(id: Int!): Job
```

不管是 PUT 还是 DELETE 操作，都是取消作业——这种灵活性带来了更有表现力的 API。

在为页面请求数据时，REST API 只返回它们的资源。通常情况下，如果你想获取相关的资源，需要先获取 X，然后是 X 的 Y。

一些 REST API 允许你获取相关的资源，这很好。但你不能获取不相关的资源，如 X 和 Z，但 GraphQL 可以，你可以用多个根查询来获取它们。

在使用 GraphQL 时，你只需要发出一个 HTTP 请求就可以获取所有数据。

现在，如果你获取的数据太多，仍然可能发生灾难性的错误。但在大多数情况下，服务器可以有效地缓存数据并在单个请求中返回大量信息。

GraphQL 是一种强大的查询语言，它在过去几年里不断发展。它提供了令人难以置信的工具，让你可以专注于业务逻辑。此外，你在定义 API 时具有很大的灵活性，让你拥有了更多的控制权。

REST 比之前的 API 要好很多，它提供了一条重新思考数据和以一种朴素的方式创建 API 的途径。

但 GraphQL 更强大，更容易使用，并且提供了更好的开发者体验。你的下一个 API 应该是 GraphQL，而不是 REST。

原文链接：

[https://ethanmick.com/why-graphql-is-better-than-rest/](https://ethanmick.com/why-graphql-is-better-than-rest/)