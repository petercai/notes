# mongoDB操作手册，熟练不等于精通_mongodb_ike潮_InfoQ写作社区
标题：mongoDB 操作手册，熟练不等于精通；

引言：日常工作的各种查询语句总结，自动化安装部署，应用接入……

客户端连接 mongo 服务，身份认证之后才能操作，身份认证之前必须切换到 admin 库，身份认证函数使用`db.auth(user, password)`;

```
root@VM-218-88-centos:/
MongoDB shell version v4.0.4
connecting to: mongodb://127.0.0.1:27017
Implicit session: session { "id" : UUID("9f2bb75f-783e-4e57-a5d2-8c5301005c61") }
MongoDB server version: 4.0.4
> use admin
switched to db admin
> db.auth("user","password")
1
```

1.1、数据库
-------

1、新增（切到）目标数据库：`use database`  

2、查看当前数据库：`db`；

3、查看所有的数据库：`show databases`或者`show dbs`；

4、删除数据库，先`use database`，切换到需要删除的数据库，然后执行`db.dropDatabase()`删除当前库；

```
> use vendors
switched to db vendors
> db
vendors
> db.dropDatabase()
{ "dropped" : "vendors", "ok" : 1 }
```

> 自定义的用户可能会报没有删除权限操作，需要添加一下 root 角色权限；

1.2、集合
------

1、查看当前库中的所有集合：`show collections`或者 `show tables`；

2、创建集合（可指定配置参数）：`db.createCollection(colName, options)`，可选项参数如下；

*   capped：（可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数；**
    
*   size：（可选）为固定集合指定一个最大值，即字节数。 **如果 capped 为 true，也需要指定该字段；**
    
*   max：（可选）指定固定集合中包含文档的最大数量；
    

> 当操作的集合不存在时，直接向其插入文档，也会自动的创建集合；

3、删除集合：`db.colName.drop()`;

```
> use test
switched to db test
> db.createCollection("user")
{ "ok" : 1 }
> show tables
user
> db.user.drop()
true
> show tables
```

1.3、文档
------

### 1.3.1、增

#### 单条插入

`db.colName.insert(obj)`  

```
> db.user.insert({"name":"ikejcwang", "age":27})
WriteResult({ "nInserted" : 1 })
```

#### 多条插入

`db.colName.insertMany(arr, {writeConcern: 1, ordered: true})`  

*   arr：需要插入的条目数组，（不要求数组元素的字段一致，基于 mongo 的特性）；
    
*   options：可选项扩展；
    
*   writeConcern：写入策略，默认为 1，即要求确认写操作，0 是不要求；
    
*   ordered：指定是否按数组的顺序写入，默认 true，按顺序写入；
    

```
> db.user.insertMany([{name:"i", age:23}, {name:"k", age:24}, {name:"e", age:25}], {ordered: false})
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("633d2a67b242067655f426b6"),
                ObjectId("633d2a67b242067655f426b7"),
                ObjectId("633d2a67b242067655f426b8")
        ]
}
```

同时，mongo 支持 JS 脚本语句操作（任何场景下都行）：

```
> for(let i = 0; i <= 5; i++){
... db.user.insert({name:"ike"+i, age:30+i})
... }
WriteResult({ "nInserted" : 1 })
```

### 1.3.2、删

`db.colName.remove(<query>, {justOne: false, writeConcern: 1})`；

*   query：，删除的指定条件（全删，传空即可）；
    
*   justOne：（可选），如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配的文档；
    
*   writeConcern：（可选），抛出异常级别
    

```
> db.user.remove({name:"ike0"})
WriteResult({ "nRemoved" : 1 })
>
> db.user.remove({})
WriteResult({ "nRemoved" : 9 })
```

### 1.3.3、改

`db.colName.update(<query>, <update>, {upsert: false, multi: false, writeConcern: 1})`；

*   query：修改的查询条件（全修改传空即可）；
    
*   update：更新覆盖的内容；
    
*   upsert：（可选）如果不存在 update 的记录，是否插入 update，true 为插入，默认是 false，不插入；
    
*   multi：（可选）默认是 false，只更新找到的第一条记录，设置参数为 true，就把按条件查出来多条记录全部更新；
    
*   writeConcern：（可选），抛出异常的级别
    

更新操作必须使用修改器 $set，query 为空配合{multi: true}，表示修改全部

```
> db.user.update({}, {"$set": {name:"ikejcwang"}}, {multi: true})
WriteResult({ "nMatched" : 6, "nUpserted" : 0, "nModified" : 5 })
```

### 1.3.4、查

**查询后面继续调用**`**pretty()**`**函数，可以格式化结果集展示**

#### 标准

`db.colName.find(<query>, projection)`  

*   query：可选：查询条件（查询条件多样性），
    
*   projection：展示和屏蔽的字段。eg：{name: 1}结果集仅展示 name 字段，{name: 0}结果集屏蔽 name 字段；
    
*   > \_id 会默认展示出来，除非手动屏蔽
    

```
> db.user.find({name:"ikejcwang"}, {name:1})
{ "_id" : ObjectId("633d2e90b242067655f426c0"), "name" : "ikejcwang" }
{ "_id" : ObjectId("633d2e90b242067655f426c1"), "name" : "ikejcwang" }
{ "_id" : ObjectId("633d2e90b242067655f426c2"), "name" : "ikejcwang" }
{ "_id" : ObjectId("633d2e90b242067655f426c3"), "name" : "ikejcwang" }
{ "_id" : ObjectId("633d2e90b242067655f426c4"), "name" : "ikejcwang" }

> db.user.find({name:"ikejcwang"}, {name:0, _id:0})
{ "age" : 31 }
{ "age" : 32 }
{ "age" : 33 }
{ "age" : 34 }
{ "age" : 35 }
```

#### 统计

`db.colName.find().size()`  

或者

`db.colName.find().count()`;

#### 去重

`db.colName.distinct(field)`；

*   field：字符串，去重筛选的字段，
    

```
> db.user.distinct("name")
[ "ikejcwang" ]
```

#### 排序

`db.colName.find().sort(fieldOptions)`；

*   fieldOptions：支持多字段排序，1：生序，-1：降序。
    

```
> db.user.find().sort({name:1, age:-1})
```

#### 模糊

<query>，模糊如下：

> 正则表达式的匹配逻辑

```
db.user.find({name:/ik/})


db.user.find({name:/^ik/})


db.user.find({name:/ik$/})
```

#### 大小于

<query>，大于小于等于的条件需要用到比较器：lte（小于等于），gte（大于等于），$ne（不等于）。

```
> db.user.find({age:{$lt:33}})


> db.user.find({age:{$gt:33}})


> db.user.find({age:{$gt:33, $lt:35}})
```

#### 存在判断

<query>，需要用到操作符`$exists`，主要是操作字段内容是否为空

*   $exists：true，不为空，false，为空；
    

```
db.user.find(name:{$exists:true}).count()


select count(name) from tableName
```

#### or

<query>，需要用到操作符 $or，

```
db.tableName.find({"$or":[{"login_name":/王/},{"role_name":/王/}]},{"login_name":true,"role_name":true})


db.tableName.find({"name":"ike", "$or":[{"login_name":/王/},{"role_name":/王/}]},{"login_name":true,"role_name":true})
```

#### 分页

`limit()`和`skip()`函数

*   limit：查询条目数；
    
*   skip：查询的起始位置；
    

```
>db.tableName.find().limit(5)
 

>db.tableName.find().skip(10)
 

>db.tableName.find().skip(10).limit(5)
```

#### 包含与否

<query>，需要用到操作符`$in`，`$nin`  

*   $in：value 为数组，表示查询引用字段的值包含在数组中的条目；
    
*   $nin：value 为数组，表示查询引用字段的值不包含在数组中的条目；
    

```
db.user.find({name:{$in:["ikejcwang", "hah"]}})


db.user.find({name:{$nin:["hah"]}})
```

### 1.3.5、数组查询

构造测试的数组文档字段，如下所示：tag 字段为数组

```
> for(let i = 0; i <= 10; i++){
... let tags = [];
... for(let t = 0; t <= i; t++){
... tags.push("tag"+t);
... }
... db.user.insert({name:"ik"+i, age:"30"+i, tag:tags})
... }
WriteResult({ "nInserted" : 1 })
> 
> db.user.find()
{ "_id" : ObjectId("633d4434b242067655f426c5"), "name" : "ik0", "age" : "300", "tag" : [ "tag0" ] }
{ "_id" : ObjectId("633d4434b242067655f426c6"), "name" : "ik1", "age" : "301", "tag" : [ "tag0", "tag1" ] }
{ "_id" : ObjectId("633d4434b242067655f426c7"), "name" : "ik2", "age" : "302", "tag" : [ "tag0", "tag1", "tag2" ] }
{ "_id" : ObjectId("633d4434b242067655f426c8"), "name" : "ik3", "age" : "303", "tag" : [ "tag0", "tag1", "tag2", "tag3" ] }
{ "_id" : ObjectId("633d4434b242067655f426c9"), "name" : "ik4", "age" : "304", "tag" : [ "tag0", "tag1", "tag2", "tag3", "tag4" ] }
{ "_id" : ObjectId("633d4434b242067655f426ca"), "name" : "ik5", "age" : "305", "tag" : [ "tag0", "tag1", "tag2", "tag3", "tag4", "tag5" ] }
{ "_id" : ObjectId("633d4434b242067655f426cb"), "name" : "ik6", "age" : "306", "tag" : [ "tag0", "tag1", "tag2", "tag3", "tag4", "tag5", "tag6" ] }
{ "_id" : ObjectId("633d4434b242067655f426cc"), "name" : "ik7", "age" : "307", "tag" : [ "tag0", "tag1", "tag2", "tag3", "tag4", "tag5", "tag6", "tag7" ] }
{ "_id" : ObjectId("633d4434b242067655f426cd"), "name" : "ik8", "age" : "308", "tag" : [ "tag0", "tag1", "tag2", "tag3", "tag4", "tag5", "tag6", "tag7", "tag8" ] }
{ "_id" : ObjectId("633d4434b242067655f426ce"), "name" : "ik9", "age" : "309", "tag" : [ "tag0", "tag1", "tag2", "tag3", "tag4", "tag5", "tag6", "tag7", "tag8", "tag9" ] }
{ "_id" : ObjectId("633d4434b242067655f426cf"), "name" : "ik10", "age" : "3010", "tag" : [ "tag0", "tag1", "tag2", "tag3", "tag4", "tag5", "tag6", "tag7", "tag8", "tag9", "tag10" ] }
```

#### 匹配

要对数组字段进行相等条件的查询，条件肯定也必须为数组，且顺序也必须一致；

<field>: <value>

```
> db.user.find({tag:["tag0", "tag1"]})
{ "_id" : ObjectId("633d4434b242067655f426c6"), "name" : "ik1", "age" : "301", "tag" : [ "tag0", "tag1" ] }
```

#### 包含

如果只是要找到一个包含指定元素的数组，但不考虑目标字段数组中的顺序或其他元素，需要使用`$all`操作符

```
> db.user.find({tag:{$all:["tag0", "tag1"]}}).count()
10
```

#### 元素查询

查询数组字段是否包含指定值的元素，使用普通的过滤器<query>；

```
> db.user.find({tag:"tag9"}).count()
2
```

> 同理，它也支持其他的条件过滤：比如大小于：
> 
> `db.user.find({tag:{$gt: "10"}}).count()`；`db.user.find({tag:{$gt: 10}}).count()`  

#### 多条件元素查询

1、复合条件单个满足即可：

```
db.user.find( { tagNum: { $gt: 15, $lt: 20 } } )
```

2、复合条件多个满足：

```
db.user.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } )
```

3、按目标字段数据元素的索引位置匹配查询

<field>.<index>的形式

```
> db.user.find({"tag.9": "tag9"}).count()
2
```

4、按目标字段数组的长度查询

```
> db.user.find({tag:{$size:5}})
{ "_id" : ObjectId("633d4434b242067655f426c9"), "name" : "ik4", "age" : "304", "tag" : [ "tag0", "tag1", "tag2", "tag3", "tag4" ] }
```

### 1.3.6、聚合查询

`aggregate`：聚合运算函数

*   $match：查询过滤条件；
    
*   $lookup：连表查询；
    
*   符号来定义别名；
    
*   …………
    

#### 字段别名

`find`只能展示和屏蔽字段，但是不能给字段定义别名，这里只能依靠聚合操作；

```
> db.user.aggregate([{$match:{name:/ik/}}, {$project:{"userName":"$name", "_id":0}}])
{ "userName" : "ikejcwang" }
{ "userName" : "ikejcwang" }
{ "userName" : "ikejcwang" }
{ "userName" : "ikejcwang" }
{ "userName" : "ikejcwang" }
{ "userName" : "ikejcwang" }
{ "userName" : "ik0" }
{ "userName" : "ik1" }
{ "userName" : "ik2" }
{ "userName" : "ik3" }
{ "userName" : "ik4" }
{ "userName" : "ik5" }
{ "userName" : "ik6" }
{ "userName" : "ik7" }
{ "userName" : "ik8" }
{ "userName" : "ik9" }
{ "userName" : "ik10" }
```

#### 联合查询

使用`$lookup`操作符，相当于连表 join，$lookup 参数介绍：

*   from：需要连接的集合名称；
    
*   as：对 from 的集合取别名；
    
*   localField：当前集合中对 from 的关联字段；
    
*   foreignField：from 集合中关联当前集合的字段；
    

```
> db.acs_policy.aggregate([
    {"$match":{"subjects":{"$in":["d466d7a9-5152-47e7-ba5c-1c9bb8d0ad6c", "62a0ca2d-413a-4484-ac08-b911595b70d9"]}}},
    {"$lookup":{"from":"acs_resource_member", "as":"resourceMember", "localField":"resource_member_ids", "foreignField":"id"}},
    {"$project": {"resource_member_id_list": "$resourceMember.id", "role_ids": "$subjects", "_id":0}},
]).pretty()
{
        "resource_member_id_list" : [
                "146804b5-95bb-444a-8bd8-811d0364866c",
                "1c0c338f-b63f-48b5-9134-f48460d4ab10",
                "1f69910a-71d0-4d02-995c-215816758b6c",
                "c14b7e97-60cb-4a14-aa31-91ee542089b0",
                "c774b0c4-2774-4fcf-8760-5b49951c12e8",
                "dbc8d823-5a08-406f-8670-28ccb74ec959",
                "f6e03c36-0f64-4b77-a6dc-f6ae4af76abc"
        ],
        "role_ids" : [
                "62a0ca2d-413a-4484-ac08-b911595b70d9"
        ]
}
{
        "resource_member_id_list" : [
                "10d514b6-38a5-4f1a-8f61-a5874c1788ef",
                "3022c03e-bcac-4d98-8952-d118b581d803",
                "3d2e3770-0d1c-402b-a7f7-f7f5a665f69d",
                "96945cde-a5bb-4ba3-ae28-04f163bb296c",
                "9b0b1146-ae05-46da-9a42-07d13e1169c7",
                "ceb77748-39af-47e8-817d-282754058eed",
                "f8d9a9e5-2ede-4445-93d7-aba6698cd9cd"
        ],
        "role_ids" : [
                "d466d7a9-5152-47e7-ba5c-1c9bb8d0ad6c",
                "eeeee"
        ]
}
```

#### 分组求和

对 service 集合，以"app.id"，"sys.id"分组，求和，统计每个"app.id"，"sys.id"下所有的数目，然后排序生序，并且最终对结果集进行格式化取别名处理，

> $group 操作符的\_id 为固定格式，下面包裹的是需要分组的字段；
> 
> count 是自定义的统计展示字段，下面包含统计方式

```
> db.service.aggregate([
...     {
...         "$group":{
...           "_id":{
...             "appName":"$app._id",
...             "sysName":"$sys._id"
...           },
...           "count":{
...             "$sum":1
...           }
...         }
...     },
...     {
...         "$sort":{"count":-1}
...     },
...     {"$project":{"userCount":"$count", "sysName":"$_id.sysName", "appName":"$_id.appName", "_id":0}}
... ])
{ "userCount" : 137, "sysName" : "defaultsystem", "appName" : "amp" }
{ "userCount" : 60, "sysName" : "defaultsystem", "appName" : "acc" }
{ "userCount" : 53, "sysName" : "defaultsystem", "appName" : "rbac" }
{ "userCount" : 46, "sysName" : "defaultsystem", "appName" : "op" }
{ "userCount" : 25, "sysName" : "defaultsystem", "appName" : "vmp" }
{ "userCount" : 20, "sysName" : "defaultsystem", "appName" : "sc" }
{ "userCount" : 20, "sysName" : "defaultsystem", "appName" : "omp" }
{ "userCount" : 18, "sysName" : "defaultsystem", "appName" : "upf" }
{ "userCount" : 13, "sysName" : "defaultsystem", "appName" : "atm" }
{ "userCount" : 13, "sysName" : "defaultsystem", "appName" : "asm" }
{ "userCount" : 10, "sysName" : "defaultsystem", "appName" : "dt" }
{ "userCount" : 5, "sysName" : "defaultsystem", "appName" : "aud" }
{ "userCount" : 4, "sysName" : "defaultsystem", "appName" : "dbs" }
{ "userCount" : 3, "sysName" : "defaultsystem", "appName" : "ikejcwang" }
{ "userCount" : 2, "sysName" : "defaultsystem", "appName" : "network" }
{ "userCount" : 2, "sysName" : "defaultsystem", "appName" : "js_server_demo" }
{ "userCount" : 1, "sysName" : "defaultsystem", "appName" : "java_server_demo" }
```

#### 分组求最值，平均值

以 app.id 分组，统计每个 app.\_id 下最大的 status，并且排序降序，并且最终对结果集进行格式化取别名处理，

> 同理：avg（平均值），

```
> db.service.aggregate([
...     {
...         "$group":{
...           "_id":{
...             "appName":"$app._id"
...           },
...           "count":{
...             "$max":"$status"
...           }
...         }
...     },
...     {
...         "$sort":{"count":-1}
...     },
...     {"$project":{"_id":0, "appName":"$_id.appName", "maxStatus":"$count"}}
... ])
{ "appName" : "dt", "maxStatus" : 2 }
{ "appName" : "java_server_demo", "maxStatus" : 2 }
{ "appName" : "dbs", "maxStatus" : 2 }
{ "appName" : "vmp", "maxStatus" : 2 }
{ "appName" : "aud", "maxStatus" : 2 }
{ "appName" : "asm", "maxStatus" : 2 }
{ "appName" : "op", "maxStatus" : 2 }
{ "appName" : "omp", "maxStatus" : 2 }
{ "appName" : "network", "maxStatus" : 2 }
{ "appName" : "amp", "maxStatus" : 2 }
{ "appName" : "acc", "maxStatus" : 2 }
{ "appName" : "js_server_demo", "maxStatus" : 2 }
{ "appName" : "rbac", "maxStatus" : 2 }
{ "appName" : "sc", "maxStatus" : 2 }
{ "appName" : "upf", "maxStatus" : 2 }
{ "appName" : "ikejcwang", "maxStatus" : 2 }
{ "appName" : "atm", "maxStatus" : 2 }
```

### 1.3.7、其他

1、ObjectId 查询

```
db.service.find({"svcObjectID":ObjectId("62c3e52e79a2a529fb51b80d")})

db.service.find({"svcObjectID":{"$in":[ObjectId("62c3e52e79a2a529fb51b80d"), ObjectId("62c3e52e79a2a52hehehehehehehe"),ObjectId("62c3e52e79a2a529hahahahaha")]})
```

2、字段内容为空查询（不是字段不存在），除了 $exists，支持直接传 null

```
db.service.find({"svcObjectID":null})
```

1.4、索引
------

#### 查看索引

1、查看当前索引：`db.colName.getIndexes()`；

2、查询当前索引的大小：`db.colName.totalIndexSize()`；

```
> db.acs_action.getIndexes()
[
        {
                "v" : 2,
                "unique" : true,
                "key" : {
                        "appid" : 1,
                        "code" : 1
                },
                "name" : "appid_1_code_1"
        }
]
> db.acs_action.totalIndexSize()
110592
```

#### 删除索引

1、删除所有索引：`db.colName.dropIndexes()`；

2、删除指定的索引，指定 name：`db.colName.dropIndex("indexName")`  

#### 创建索引

`db.colName.createIndex(keys, options)`；

*   keys：需要创建的索引字段，1 升序创建，-1 降序创建，包含组合索引；
    
*   > 单一索引：
    > 
    > `db.colName.createIndex({"id": 1})`  
    > 
    > 复合索引：
    > 
    > `db.colName.createIndex({"id": 1, "createTime": -1})`  
    
*   options：可选参数配置，非必填，可选参数列表如下：
    

```
> db.user.createIndex({"name":1, "age": -1}, {unique: true})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
```