# Mongodb支持事务吗？
1.1、MongoDB事务简介
---------------

MongoDB 是一个非关系型数据库管理系统，最初并不支持事务。然而，随着时间的推移，MongoDB 在其4.0版本中引入了多文档事务支持，使得在单个集合中执行多个操作成为可能。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d86421773cdb4cf2af474cbd16af9ec5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1776&h=319&s=92980&e=png&b=fefdfd)

In MongoDB, an operation on a single document is atomic. Because you can use embedded documents and arrays to capture relationships between data in a single document structure instead of normalizing across multiple documents and collections, this single-document atomicity obviates the need for distributed transactions for many practical use cases.

For situations that require atomicity of reads and writes to multiple documents (in a single or multiple collections), MongoDB supports distributed transactions. With distributed transactions, transactions can be used across multiple operations, collections, databases, documents, and shards.

译文：

在MongoDB中，对单个文档的操作是原子的。因为您可以使用嵌入式文档和数组来捕获单个文档结构中的数据之间的关系，而不是在多个文档和集合之间进行规范化，所以这种单文档原子性消除了许多实际用例中对多文档事务的需求。

对于需要原子性地读写多个文档（在单个或多个集合中）的情况，MongoDB支持多文档事务。使用分布式事务，可以跨多个操作，集合，数据库，文档和分片使用事务。

1.2、事务和原子性
----------

在MongoDB中，对单个文档的操作是原子的。从MongoDB 4.2开始，分布式事务和多文档事务是同义词。 分布式事务是指分片群集和副本集上的多文档事务。 从MongoDB 4.2开始，多文档事务（无论是在分片群集或副本集上）也称为分布式事务。

2.1、配置事务
--------

在测试mongodb之前要先配置一下事务，MongoDB 的事务只能在开启副本集的时候才能使用，一般安装后默认是单副本，我们配置将其配置成多副本后再运行事务。不然的话会报以下错误。↓↓↓

_**MongoServerError: Transaction numbers are only allowed on a replica set member or mongos**_

step1:

vim /usr/local/mongodb/mongodb.conf

在最后面增加

replication:

replSetName: rs0

step2:

保存后重启mongodb，然后进入执行

/usr/local/mongodb/bin/mongo

step3:

rs.initiate()

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b90c75ba793e4c2590f9b422a6afff08~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=862&h=128&s=6823&e=png&b=283033)

到这里就成功后，后面就可以去测试事务了。

2.2、事务示例
--------

### 1）多文档事务支持（Multi-document Transactions）

多文档事务允许在一个事务中对多个文档执行读写操作，跨越多个集合或单个集合的多个文档。这确保了这些操作要么全部成功提交，要么全部失败回滚，从而保持数据的一致性。

示例命令行事务示例（使用 MongoDB Shell）:

```ini
// 开启一个会话
session = db.getMongo().startSession()
//创建两个集合
coll1 = session.getDatabase("mydb1").collection1
coll2 = session.getDatabase("mydb1").collection2
// 在会话中启动事务
session.startTransaction()

try {
    coll1.insertOne({ id: 1 ,name:"001"})
    coll2.insertOne({ id: 2, name: "002" })
    
    // 如果一切顺利，提交事务
    session.commitTransaction()
} catch (error) {
    // 发生错误时，回滚事务
    session.abortTransaction()
}

```

在多文档事务中支持以下读/写操作： |

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/571fa3e8d3c64d618415e9a5d0f11ee2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=870&h=1468&s=120502&e=png&b=ffffff)

### 2）回滚与提交（Rollback and Commit）

如果事务中的任何操作失败，整个事务将被回滚，之前所做的修改都不会被应用到数据库中。只有当所有操作都成功完成时，事务才会被提交并应用到数据库中。

示例：

```php

session = db.getMongo().startSession();

coll1 = session.getDatabase("mydb1").collection1;
coll2 = session.getDatabase("mydb1").collection2;

session.startTransaction();

try {
    coll1.insertOne({ id: 1 ,name:"001"});
    coll2.insertOne({ id: 2, name: "002" });
  	
    throw "Some error occurred";
    
    
    session.commitTransaction();
} catch (error) {
    
    session.abortTransaction();
}

```