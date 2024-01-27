# 使用Graphql
> 首先哈，书接上文，我们把NodeJS的REST API 干完好像么有什么事情做啦，但是构建REST API 只是我的一个计划内容，它不是最终目标，我们的最终目标是：“通过一个Node JS 的应用”，我们能够掌握一个非常 前沿的构建NodeService 的架构，和技术栈。**前路慢慢各位同志还需加油！**, 目前我们是使用Graphql 把这个项目修改啦一下，没准下一次就是SSR 呢？没准啥啥技术WEB3 啦 ，是吧....，作为一个技术人，还是学习 舒服 🌚 🤪 勇于探索 未来的无限可能

> 其实我实在事很无语，我认为这种中文文档就很...很迷，对于阅读该文档的同学我给你们一个建议：

在下，粗浅的认为，官方的中文文档确实有点拉胯，基本上都是机器翻译的，如果你会 English 最好读原文，可能更加的准确，而且这个东西 graphql很新，它里面 有非常多的概念，机会没两句 就 是用这个概念解释 另一个概念，

```arduino
我建议的阅读顺序是 1- 去官网 点击 ：“马上开始” 找到JavaScript 支持，
找到 Apollo Server 把这个东西的代码搞下来自己跑起。 然后在官方的内容 都拿这个来做实验，2- 点
击学习，先看 入门 这一篇文章，3- 直接啦到最下面 先把 “Schema 和 类型” 下的东西 都过一遍，再
回头看 “查询和变更” 的东西，4- 看看剩下的 “验证” + “执行” + “内省” + "最佳实践"

```

**思路要搞清楚**，我建议在阅读的时候 ，抓住 这样的核心：“我如何查询？（包括各种筛选什么的，用postman这么去测试什么的）我如何修改变更？”， 好啦 话不多说 正式进入正题

必备知识：我们是基于这个服务改造的 [prisma-express-test](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FBM-laoli%2Fprisma-express-test "https://github.com/BM-laoli/prisma-express-test")，关于这个服务如何构建，如果你不清楚请你去看我前面的文章

### 理论知识

_是什么？_

首先哈这个graphql 到底是什么东西，我们需要心中有数，它实际上是一种规范，或者说是一个 “前后端交换数据的协议，你请求啥啥啥必须要在xx里这样写，我才能查出来给你”，它的演示图 如下 ![](_assets/91005ec5fd714e3aa272794eb844ae62~tplv-k3u1fbpfcp-zoom-in-crop-mark!4536!0!0!0.awebp.webp)

假设我要查询：“id=123user的name ，和他去过的所有城市的名字和所属省份 并且取前十条数据”，那么我可以这样写

```css
query {
  user (id : "233") {
    name
    citys (first: 10) {
      name
      province
    }
  }
}


```

_它都有哪些概念？_

> 前面我就说过 ，之所以这个graphql的文档 难看，是因为它总是在用 你不懂的概念去解释另一个概念，干脆我们一次性把它 一五一十 说明白！

1.  什么是 操作类型 在graphql 中，一共有三种操作类型，查询、变更、订阅

*   查询: query : 这个....非常的简单不过多介绍
*   变更mutation： 主要是对数据的 create modify 和delete 操作
*   订阅subscription： 主要是数据发生变化的时候，自动去推送

```bash

query {
  author {
    id
  }
}

```

2.  对象类型 和 标量类型 讲这两个概念之前我们先了解一下 ，一个http graphql 请求到达 之后，我们是如何解析和处理的。（这里我们举例说明query） 首先graphQL服务接受query -> 从这个root Query 开始查找 -> 找到 对象类型 （Object Type）的时候就要使用解析函数 Resolver 来获取其内容 -> 发现解析器返回又是一个ObjectType 继续获取，--> 一直找啊找，知道找到标量类型（Scalar Type）结束获取，直到获取到最后一个标量类型

用户在Schema 中定义的type 就是 对象类型，（解释一下什么是Schema，它是一种声明，有来声明graph 就能解析 你的query了 ）

```js

type User {
  name: String!
  age: Int
}

```

标量类型，就是一些graphQL 内置的类型：String Int Float Boolean ID 等...也允许用户自定义标量

3.  模式是什么Schema 上面有聊到什么是Schema 并且大概介绍了一下，我们现在来看个例子🌰 下面的就是一个生产级别的Schema 它专门为graph 服务, 顺便提一嘴哈，在query 的时候graph 是并行的，在mutation 的时候是串行的。

```js
# src/schema.graphql

# Query 入口
type Query {
    hello: String
    users: [User]!
    user(id: String): [User]!
}

# Mutation 入口
type Mutation {
    createUser(id: ID!, name: String!, email: String!, age: Int,gender: Gender): User!
    updateUser(id: ID!, name: String, email: String, age: Int, gender: Gender): User!
    deleteUser(id: ID!): User
}

# Subscription 入口
type Subscription {
    subsUser(id: ID!): User
}

type User implements UserInterface {
    id: ID!
    name: String!
    age: Int
    gender: Gender
    email: String!
}

# 枚举类型
enum Gender {
    MAN
    WOMAN
}

# 接口类型
interface UserInterface {
    id: ID!
    name: String!
    age: Int
    gender: Gender
}

```

4.  解析函数 Resolver 实际上它就是一个函数，提供数据用的，比如现在我有这样的query 和这样的resolver 他们就是这样组合在来一起

```js

query {
  author
}


Query: { 
  author ( parent, args, context, info ) {
    return .....
  }
}



1. 参数1 ：当前上一个解析函数的返回值

2. 参数2：查询中传入的参数

3. 参数3：提供给所有解析器的上下文信息

4. 参数4：一个保存与当前查询相关的字段特定信息以及 schema 详细信息的值
*/

```

5.  请求的格式 前面都是在说如何做如何做，偏向理论化了，现在我们来说说，客户端如何发一条http 请求到graphql service 呢？

_如果你是GET 你可以这样_

```ini
http://myapi/graphql?query={me{name}}

```

_如果你是POST 你可以这样_

```json
{
  "query": "...",
  "operationName": "...",
  "variables": { "myVariable": "someValue", ... }
}

```

### 项目实践指南

> 好啦，上面讲啦很多废话，讲啦很多理论的东西，现在我们先看看如何实际运用哈。首先这里说明一下，我们是基于已经构建好的REST API 进行的修改，如果你不晓得我这个API 是如何构建的，请移步看另一片文章，那里有详细的说明，它大概长这样

_项目结构_ ![](_assets/be1161b5c8814964931114d8150499b4~tplv-k3u1fbpfcp-zoom-in-crop-mark!4536!0!0!0.awebp.webp)

_POST MAN_ ![](_assets/a92e5f7c0fc84a0c83a943feb2cdb77e~tplv-k3u1fbpfcp-zoom-in-crop-mark!4536!0!0!0.awebp.webp)

1.  首先我们开始我们项目第一步，工欲善必先利其器也， 首先我们准备 graphql、express-graphql、graphql-tools，第一个是核心必须要的，第二个 是和express 配套的，第三个是一个tools 工具可以方便整理和管理你的 schema等内容
    
2.  构建项目结构，我们新增一个文件用来存放 我们这个项目的shcema ,并且写入下面的 schema （query）
    

```js


const typeDefs =  `
  type Query {
    author(id: String): Author
    authors: [Author]!
  }

  type Author {
    id: ID!
    first_name: String
    family_name: String
    date_of_birth: String
    date_of_death: String
    age: Int
  }
`;

module.exports = {
  typeDefs: typeDefs,
};


const { queryAuthor } = require('../service/authorService');
const resolvers = {
  Query: {
    authors(parent, args, ctx, info) {
      return queryAuthor({});
    },
    author(parent, args, ctx, info) {
      const { id } = args;
      return queryAuthor({ id: id });
    },
  },
};

module.exports = {
  resolvers: resolvers,
};


const { resolvers } = require('./resolvers');
const { typeDefs } = require('./schema');
const { makeExecutableSchema } = require('@graphql-tools/schema');


const schema = makeExecutableSchema({
  typeDefs,
  resolvers,
});

module.exports = {
  schema: schema,
};


++++
const { graphqlHTTP } = require('express-graphql');
const { schema } = require('./graphql');
++++

app.use(
  '/graphql',
  graphqlHTTP({
    schema,
  }),
);
++++


```

下面的c-url

```shell
# 你如何请求呢？ 我们以postman 为例子 c-url 如下
curl --location -g --request GET 'http://localhost:3000/graphql?query={authors {first_name  id  family_name 
age }} ' \
--header 'Content-Type: application/json' \
--data-raw '{
    "first_name":"Ace",
    "family_name":"Y",
    "date_of_birth":"2022-05-21T15:40:50.926Z",
    "date_of_death":"2022-05-21T15:40:50.926Z"
}'

# 带条件的查询
curl --location -g --request GET 'http://localhost:3000/graphql?query={author(id: "62890ef9ecfd69398ee75752") { id }} ' \
--header 'Content-Type: application/json' \
--data-raw '{
    "first_name":"Ace",
    "family_name":"Y",
    "date_of_birth":"2022-05-21T15:40:50.926Z",
    "date_of_death":"2022-05-21T15:40:50.926Z"
}'

```

**关联查询如何构建？**

> 我们还是以上面的get来说，这一次我们要获取 book 我们需要查询它的author ，这如何做呢？

```js

const typeDefs =  `
  type Query {
    author(id: String): Author
    authors: [Author]!
    books: [Book]
  }

  type Author {
    id: ID!
    first_name: String
    family_name: String
    date_of_birth: String
    date_of_death: String
    age: Int
  }

  type Book {
    id: ID!
    title: String
    author: Author
    summary: String
    isbn: String
  }
`;


const { queryAuthor } = require('../service/authorService');
const { queryBook } = require('../service/bookService');
const resolvers = {
  Query: {
    authors(parent, args, ctx, info) {
      return queryAuthor({});
    },
    books() {
      queryBook({}).then((res) => {
        console.log('--->', res);
      });

      return queryBook({}); 
    },
  },
};

module.exports = {
  resolvers: resolvers,
};


curl --location -g --request GET 'http://localhost:3000/graphql?query={books { id summary author {  id first_name  } }} ' \
--header 'Content-Type: application/json' \
--data-raw '{
    "first_name":"Ace",
    "family_name":"Y",
    "date_of_birth":"2022-05-21T15:40:50.926Z",
    "date_of_death":"2022-05-21T15:40:50.926Z"
}'

```

3.  mutation 变更数据 上面我们主要是把graphQL 的query 都讲了一遍，接下来我们来说说 它的mutation, 比如我现在需要 创建名为xx 的author、 修改id = xxxx 的author的名称、以及删除 id 为xxx 的author

```js

+++
`
  type Mutation {
    createAuthor(first_name: String, family_name: String, age: Int): Author!
    updateAuthor(
      id: ID!
      first_name: String
      family_name: String
      age: Int
    ): Author!
    deleteAuthorByID(id: ID!): Author!
  }
  `
  +++


+++
  Mutation: {
    createAuthor: async (parent, args) => {
      const { id, first_name, family_name, age } = args;
      return await createAuthor({ id, first_name, family_name, age });
    },
    deleteAuthorByID: async (parent, args) => {
      return await findAndDelete(args.id);
    },
    updateAuthor: async (parent, args) => {
      const { id, first_name, family_name, age } = args;
      const author = await queryAuthor({ id: id });
      
      
      if (!author) {
        throw new Error('查无此人');
      }
      await update(id, {
        first_name,
        family_name,
        age,
      });
      return await queryAuthor({ id: id });
    },
  },
+++

```

在Postman 上，实际上在body 参数上是有快捷的 GraphQL 操作的，特别骚气的是 它可以 自动获取你的所有 schema 并且如果在你写的时候有自动提示，如果你写错啦，它还会自动报错，啊，这个功能还是香的啊，

![](_assets/8fd6843f2f304786860616a253e64505~tplv-k3u1fbpfcp-zoom-in-crop-mark!4536!0!0!0.awebp.webp)

下面我给了一个update的时候的 c-url供你体验

```shell
curl --location --request POST 'http://localhost:3000/graphql' \
--header 'Content-Type: application/json' \
--data-raw '{"query":"mutation {\n    updateAuthor(id:\"62890ef9ecfd69398ee75752\", first_name:\"Joney\",\n        family_name:\"joney\", age:23){\n        id\n        family_name\n    }\n}","variables":{}}'

```

4.  订阅数据变化

> 这里主要是做了这样的一个操作：“我们实现了类似Job 的功能，发现 某个id 的author 中的age 改变后，打印一个log” （TODO 这是一个复杂的部分，因为如果是单机器性能有限，如果是多节点比如丢k8s上还会存在别的问题，因此如何做 才能满足，这是一个大话题，这里暂时按下不表）
