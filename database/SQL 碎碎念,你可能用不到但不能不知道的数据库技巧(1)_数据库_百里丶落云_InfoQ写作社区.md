# SQL 碎碎念,你可能用不到但不能不知道的数据库技巧(1)_数据库_百里丶落云_InfoQ写作社区
作为一个程序员,时时刻刻都要忙碌于将实际业务转化为代码的繁忙工作中.你可能从事前端工作,展示漂亮的界面,你是后端,可能按照业务逻辑疯狂封装着接口. 最终种种数据落地都要涉及到数据,

将 CURD 到数据库中. 现在很多框架集成了多种 SQL 功能,但是逻辑复杂的业务仍然要使用原生 SQL.

结合多年经验,百里这里列举出你可能未发现的 SQL 一些'用法'!!

大家好,这里是百里,SQL 碎碎念, 希望大家喜欢.

工作中会遇到非常多的多种业务相互嵌套,然后按照条件在修改某个表的数据,然而传统的书,视频学习资料中,告诉的你就是

`UPDATA 字段1 = 内容1 , 字段2 = 内容2 FROM 表 where 条件1 条件2`  

这样的内容.对于新手来说肯定看到的一个头两个大 .

解决办法
----

创建临时表，将复杂的数据逻辑写入到临时表中，并将需要改的关联字段去重，设定好条件，

使用`UPDATA 被修改表字段1 = 临时表内容1 , 被修改表2 = 临时表内容2 from 表1 inner join 临时表 on 关系1 = 关系2 where 条件1 条件2`  

例子：
---

假定我们有两张表，一张表为 Product 表存放产品信息，其中有产品价格列 Price；另外一张表是 ProductPrice 表，我们要将 ProductPrice 表中的价格字段 Price 更新为 Price 表中价格字段的 80%。 

在 Mysql 中我们有几种手段可以做到这一点，一种是 update table1 t1, table2 ts ...的方式：···

```
UPDATE product p 
INNER JOIN productPrice pp 
ON p.productId = pp.productId 
SET pp.price = pp.price * 0.8 
WHERE p.dateCreated < '2022-11-01'
```

我们知道数据库中是有 WHILE ,和游标的, 结合新增数据我们也是按照传中的`insert (字段1,字段2..) into values(值1,值2...)` ,循环逐条新增数据反倒麻烦, 可能对于很多高手来说这条可能觉得小题大做!但是百里当入手时,领导让我批量增加数据,我这还 where 加条件判断呢....ε=(´ο｀\*)))唉究极黑历史 .

解决办法
----

在 SQL 中 可以使用 ,`INSERT INTO 表 select 值 ... from (表) where 条件1` 的形式批量增加数据,前提要保证字段位置对的上.

例子
--

假定我们有 3 张表,

表 A : 工号,姓名,性别,年龄.表 B : 工号,地址,电话,这个时候有个接口需要自建立表 C, 取工号,姓名,地址电话 其他字段. 这个时候就使用批量增加的方式, 减少代码量

```
insert into 表C from
select a.姓名 , b.地址,b.电话 from  表A a inner join  表b on a.工号 = b.工号 where  条件...
```

工作中肯定会遇到各种奇葩数据,然后又需要你整理成领导想要的数据,其中最常规也是最头疼的就是,行列转换了.比如这种

![](_assets/89157f3ed64c166f66bd54ee5429687e.png)

让你统计出 user ,no, type 对应的数量, 这时就很麻烦.

解决办法
----

```
#定义数据
declare @t table(usser int ,no int ,a int,b int, c int)  
insert into @t select 1,1,21,34,24  
union all select 1,2,42,25,16
#转换
SELECT usser,no,Type=attribute, Num=value  
FROM @t  
  UNPIVOT  
  (  
    value FOR attribute IN([a], [b], [c])  
  ) AS UPV
```

![](_assets/53e9a7f6219f96a55d021e3c7fe9fb4e.png)

今天的不开心就到此为止吧, 明天依旧光芒万丈~ 这里是百里,如果我帮到您,谢谢点个赞~.

