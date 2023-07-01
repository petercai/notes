# GraphQL初探 - 简书

0.3122018.06.14 06:23:24字数 4,655阅读 1,287

1.什么是 GraphQL ？
---------------

### A query language for your API

> GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. GraphQL provides a complete and understandable description of the data in your API, gives clients the power to ask for exactly what they need and nothing more, makes it easier to evolve APIs over time, and enables powerful developer tools.

GraphQL 是 Facebook 在2012年创建、2015年形成规范的一种应用层查询语言

很多优点：

*   客户端可以自定义查询语句，通过自定义不仅提高了灵活性，而且服务端只返回客户端所需要的数据，减少网络的开销，提高了性能
*   服务端收到客户端的请求，首先做类型检查，及时反馈，而且更加安全；这个对于node.js 就效果很明显了。
*   自动生成文档，降低维护成本；
*   服务端可以通过新增字段，弃用字段，避免版本的繁杂和紊乱

上面所述也都是Rest API弊端，GraphQL却能很好的解决他们。

[中文网站GraphQL](http://graphql.cn/)

\[图片上传失败...(image-e3e659-1529652402420)\]

大家都应该知道REST API需要为每个请求匹配相应的数据需求。而GraphQL的优点在与客户端数据要求发生变化时, 不需要修改后端. 因此, 你不必因为客户端数据需求的变更而改变你的后端。这解决了管理REST API中的最大的问题。

### GraphQL的特征：

*   可描述性的：使用GraphQL，你获取的都是你想要的数据，不多也不会少；
*   分级性的：GraphQL天然遵循对象间的关系，通过一个简单的请求，我们可以获取到一个对象及其相关的对象，比如说，通过一个简单的请求，我们可以获取一个作者和他创建的所有文章，然后可以获取文章的所有评论；
*   强类型的：使用GraphQL的类型系统，我们可以描述能够被服务器查询的可能的数据，然后确保从服务器获取到的数据和我们查询的一致；
*   不做语言限制：并不绑定于某一特定的语言，实际上现在已经有一些不同的语言有了实践；
*   兼容于任何后台：GraphQL不限于某一特定数据库，可以使用已经存在的数据，代码，甚至可以连接第三方的APIs.
*   好反省的：GraphQL服务器能够查询架构的细节。

### 总结：

它的官方宣传语是“为你的 API 量身定制的查询语言”，用传统的方式来解释就是：

相当于将你所有后端 API 组成的集合看成一个数据库，用户终端发送一个查询语句，你的 GraphQL 服务解析这条语句并通过一系列规则从你的“ API 数据库”里面将查询的数据结果返回给终端，而 GraphQL 就相当于这个系统的一个查询语言，像 SQL 之于 MySQL 一样。

### 来看看 GraphQL 查询demo

Graph的查询语句非常类似JSON，如客户端的查询语句

查询用户信息的name和sex字段数据，服务端返回结果如

```
{
  "data": {
    "user": {
      "name": "zhaiqianfeng",
      "sex": "男"
    }
  }
} 
```

使用如下查询语句时

```
{
  "data": {
    "user": {
      "name": "zhaiqianfeng",
      "intro": "博主，专注于Linux,Java,nodeJs,Web前端:Html5,JavaScript,CSS3"
    }
  }
} 
```

* * *

2\. GraphQL 组成
--------------

> GraphQL的核心包括Query,Mutation,Schemas等等，每个概念下又有一些子概念,下面分别做简单的介绍：

### Query

Queries用做读操作，也就是从服务器获取数据。

Queries定义了我们对模式执行的行为。下面是一个简单的查询及相应的结果：

客服端：

```
 {
      author {
        name
        posts {
          title
        }
      }
    } 
```

服务端返回：

```
 {
      "data": {
        "author": {
          "name": "Chimezie Enyinnaya",
          "posts": [
            {
              "title": "How to build a collaborative note app using Laravel"
            },
            {
              "title": "Event-Driven Laravel Applications"
            }
          ]
        }
      }
    } 
```

===============================================================================================》》》

### Mutation

> 传统的API使用场景中，我们会有需要修改服务器上数据的场景，mutations就是应这种场景而生。mutations被用以执行写操作，通过mutations我们会给服务器发送请求来修改和更新数据，并且会接收到包含更新数据的反馈。mutations和queries具有类似的语法，仅有些许的差别。

```
mutation UpdateAuthorDetails($authorID: Int!, $twitterHandle: String!) {
      updateAuthor(id: $authorID, twitterHandle: $twitterHandle) {
        twitterHandle
      }
    } 
```

在mutation中以数据为载物发送，比如上面的例子中，我们发送了下面的数据：

```
# Update data

    {
     "authorID": 5,
     "twitterHandle": "ammezie"
    } 
```

更新完成后，我们将从服务器获取以下内容作为反馈。反馈数据中包含我们更新的数据

```
# Response after update

    {
      "data": {
        "id": 5,
        "twitterHandle": "ammezie"
      }
    } 
```

和queries类似，mutations也能够接受，多重fields,不过queries和mutations的一个重大不同之处在于，为了保证数据的完整性mutations是串形执行，而queries可以并行执行。

===============================================================================================》》》

### Schema

> GraphQL中非常重要的一个概念是Schema, Schema由服务端来定义，用于定义API接口，并依靠Schema来生成文档以及对客户端请求进行校验。Schema只是一个概念，它是由各种数据类型及其字段组成，而每个类型的每个字段都有相应的函数来返回数据，这是其灵活的关键所在，如Schema

#### Schemas 描述了 数据的组织形态 以及服务器上的那些数据能够被查询，Schemas提供了你数据中可用的数据的对象类型，GraphQL中的对象是强类型的，因此schema中定义的所有的对象必须具备类型。类型允许GraphQL服务器确定查询是否有效或者是否在运行时。Schemas可用是两种类型Query和Mutation。

#### 示例一：

```
type User{
    name: String
    sex: String
    intro: String
}
type Query {
    user:User
} 
```

上面定义了两个类型，User和Query：User有三个字段分别是name、sex和intro；Query是个特殊的类型，用于查询数据，内含user字段；

如果客户端输入的如下查询语句：

#### 示例二：

又或者是下面是一个示例：

```
type Author {
      name: String! 
      posts: [Post]  
    } 
```

schemas定义了一个Author对象，它包含两个fields(name和posts),这意味着当我们操作（读取）Author时，我们只能使用name和fields,每个field都可以是必须的或者可选的,比如上面的namefield是必须的，因为其后有符号!,而posts是可选的。

* * *

* * *

3\. GraphQL vs. REST
--------------------

> GraphQL目前被认为是革命性的API工具，因为它可以让客户端在请求中指定希望得到的数据，而不像传统的REST那样只能呆板地在服务端进行预定义。这样它就让前、后端团队的协作变得比以往更加的通畅，从而能够让组织更好地运作。而实际上，GraphQL与REST都是基于HTTP进行数据的请求与接收，而且GraphQL也内置了很多REST模型的元素在里面。

从API的各个组件分别来讨论GraphQL和REST都分别是如何处理的。

### 资源（Resources）

> REST的核心思想就是资源，每个资源都能用一个URL来表示，你能通过一个GET请求访问该URL从而获取该资源。根据当今大多数API的定义，你很有可能会得到一份JSON格式的数据响应，整个过程大概是这样：

```
GET /books/1
{
  "name": "Black Hole Blues",
  "author": { 
    "firstName": "zhang",
    "lastName": "liang"
  }
  
} 
```

以上形式它的路径与所请求到的数据是高度耦合的，基本无法变动。

在这一点上GraphQL就大为不同，因为在GraphQL里这两个概念是完全分离的。比如说在你的schema定义中，你可能会有Book和Author两个类型（type）：

```
type Book {
  id: ID
  title: String
  published: Date
  price: String
  author: Author
}
type Author {
  id: ID
  firstName: String
  lastName: String
  books: [Book]
} 
```

#### 注意这里我们虽然定义了数据类型，但却不知道该如何获取这些数据。这是REST与GraphQL的一个核心差异：资源的描述信息与其获取方式相分离。

如果要去访问某个特定的book或者author资源，我们需要在schema中创建一个Query类型：

```
type Query {
  book(id: ID!): Book
  author(id: ID!): Author
} 
```

前端可以这样请求我们的数据

```
GET /graphql?query={ book(id: "1") { title, author { firstName } } }
{
  "title": "Black Hole Blues",
  "author": {
    "firstName": "Janna",
  }
} 
```

### 小结：

异同点有：

相同：

*   都有资源这个概念，而且都能通过ID去获取资源。
*   都可以通过HTTP GET方式来获取资源
*   都可以使用JSON作为响应格式

不同：

*   在REST中，你所访问的路径就是该资源的唯一标识（ID）；在GraphQL中，该标识与访问方式并不相关
*   在REST中，资源的返回结构与返回数量是由服务端决定；在GraphQL，服务端只负责定义哪些资源是可用的，由客户端自己决定需要得到什么资源。简单来说就是前者中 是后端强行给你一些数据要通过HTTP，而后者是将一些数据定义在了服务端，让前端自助从这些数据中来选择性的通过HTTP到前端。

===============================================================================================》》》

### 路由（URL Route） vs. GraphQL Schema

如今的REST API通常会由一系列的URL端点组成：

```
GET /books/:id
GET /authors/:id
GET /books/:id/comments
POST /books/:id/comments 
```

你可以把这种API的形态称之为线性结构——因为这就是一个列表嘛。当你要获取数据时，第一个事情就是搞清楚你要访问的是哪个端点。

在GraphQL中——可以通过查看GraphQL schema获得相关信息：

```
type Query {
  book(id: ID!): Book
  author(id: ID!): Author
}
type Mutation {
  addComment(input: AddCommentInput): Comment
}
type Book { ... }
type Author { ... }
type Comment { ... }
input AddCommentInput { ... } 
```

REST会使用类似GET、POST这样的动词去请求相同的URL来表示这到底是一个读操作还是写操作，而GraphQL会使用不同的预定义类型：Mutation和Query。在GraphQL请求中，你可以通过不同的关键字进行不同的操作：

```
query { ... }
mutation { ... } 
```

这里的Query类型定义与上面的REST路由是完全契合的，同样表示了数据的访问入口，因此这是GraphQL中最能与REST的URL端点所对应的概念。

如果是对资源的简单查询，GraphQL与REST是类似的，都是通过指定资源的名称以及相关参数来取得，但不同的是，在GraphQL中，你可以根据资源之间的关联关系来发起一个复杂请求，而在REST中你只能定义一些特殊的URL参数来获取到特殊的响应，或者是通过发起多个请求、再自行把响应得到的数据进行组装才行。

===============================================================================================》》》

### 路由处理器（Route Handlers）vs. 解析器（Resolvers）

首先使用Express实现一个hello world：

```
app.get('/hello', function (req, res) {
  res.send('Hello World!')
}) 
```

golang gin :

```
router := gin.Default()
router.GET("/hello", func(c *gin.Context) {
    c.String(http.StatusOK, "Hello World")
}) 
```

#### 从上面例子我们可以看到一个REST API请求的的生命周期：

*   服务器收到请求并提取出HTTP方法名（比如这里就是GET方法）与URL路径
*   API框架找到提前注册好的、请求路径与请求方法都匹配的代码
*   该段代码被执行，并得到相应结果
*   API框架对结果进行序列化，添加上适当的状态码与响应头后，返回给客户端

#### GraphQL差不多也是这样工作的，我们来看下这个对应的

```
const resolvers = {
  Query: {
    hello: () => {
      return 'Hello world!';
    },
  },
}; 
```

发起一个查询：

返回如下：

```
{ "hello": "Hello, world!" } 
```

服务器对一个GraphQL请求的执行过程：

*   服务器收到HTTP请求，取出其中的GraphQL查询
*   遍历查询语句，调用里面每个字段所对应的Resolver。在这个例子里，只有Query这个类型中的一个字段hello
*   Resolver函数被执行并返回相应结果
*   GraphQL框架把结果根据查询语句的要求进行组装

![](https://upload-images.jianshu.io/upload_images/6135025-0701ae97f92567db.png)

image

上图形象地说明了使用REST和GraphQL进行多种资源获取的方式的差异

这里放上一个现在体验 [在线体验GraphQL](https://launchpad.graphql.com/new)

* * *

首先说一下单一的入口这一点。传统的 RESTful API 里，不管前端还是后端都要对 API 做管理，一是版本管理，二是路径管理，非常麻烦，增加了工程管理的复杂度。但是如果使用 GraphQL，只需要一个入口就可以了。刚刚也说到 GraphQL 相当于一个数据库，它的入口只有一个，我们只需要访问这个入口，将我们要查询的语句发送给这个入口，就可以拿到相应的数据，所以说它是一个单端点+多样化查询方式的这么一个结构。

第二点是文档，这里文档虽然不能完全替代传统的文档，但是它能在一定程度上方便我们。传统的 RESTful API 文档管理，市面上有很多工具，像 Swagger、阿里开源的 RAP 以及 showdoc 等。但使用这些 API 文档管理工具的时候其实是有一定的学习成本的。像 Swagger， 可能对于老手来说使用起来不是很复杂，但是对于刚上手的开发者来说上手还是需要一点时间的。然后还有在使用这些平台的时候都会遇到让人头痛的“ API 和文档同步”的问题，很多时候需要自己去做 API 和文档同步的插件来解决。

> 如果使用 GraphQL 就可以在一定程度上解决 API 文档的一些问题：在做 GraphQL 类型定义的时候我们可以对类型以及类型的属性增加描述 (description) , 这相当于是对类型做注释，当类型被编译以后就可以在相应的工具上面看到我们编辑的类型详情了

除了GraphiQl 一款浏览器在线调试和文档查看工具外（代码中给予演示！）

GraphQL 还有一个比较棒的功能，就是每一个 GraphQL 类型 其实相当于 mongo 里面的一个 collection，或者 mongoose 里面的 Model, 而每一个类型之间关系也可以用工具很形象的表现出来。像系统上用到这么一个模型，它对应到哪些和它有关系的模型都高亮出来了。

[在线演示](https://apis.guru/graphql-voyager/)

使用 GraphQL 的第三点好处就是可以避免数据冗余。我们在传统的 RESTful 处理冗余的数据字段大约有这么三种处理方式：

*   一是前端选择要不要展示这些字段；
*   二是要么做一个中间层（BFF）去筛选这些字段，然后再返回终端来展示出来；
*   三则比较传统也比较麻烦，还不一定能生效，就是前端和后端去做约定，如果说这一个接口这一个字段已经不要，可以和后端商量一下把这个删掉，但是有一种情况可能造成冗余字段删不掉的，那就是后端的同学做这个接口可能是“万能接口”，也就是说这个接口在这个页面会用，在另外一个页面也能用，在这个应用会用，在另外一个应用也可能会用，多端之间存在部分数据共享，后端同学为了方便可能会写这么一个“万能”的接口来应付这种情况，久而久之，发现字段冗余到很多了，但是随便删除又可能会影响到很多地方，导致这个接口大而不能动，所以前后端都不得不忍受它。

但如果使用 GraphQL，就可以避免接口字段冗余这个问题，使用 GraphQL 的话，前端可以自己决定自己想要的返回的数据结构。

GraphQL 实际上是一种查询语言，我们在使用时就像是在数据库里面查询数据一样，查询的某一个数据要哪些字段可以在查询语句里写好，要哪些字段就返回给我们哪些字段。

最重要一点当然是数据聚合，数据聚合在使用传统的 RESTful 的方式时有多种解决方案：

*   一种前端发针对这个页面上的多数据源单独发起数据请求，然后一一展示出来，这样可能会出现页面数据加载不同步的情况。
    
*   第二种就是开发做数据拼装的中间层（BFF），用于拼装后端提供的数据，然后返回给前端。
    
*   还有一种 那就是后端同学编写针对页面的 API，即所谓胶水代码，来拼接各个服务的数据，返回给前端。
    
