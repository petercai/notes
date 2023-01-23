# Python 操作mongodb库_mongodb_度假的鱼🐟_InfoQ写作社区
MongoDB 是一个基于分布式文件存储的数据库。由 C++语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决

MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富,最像关系数据库的。它支持的数据结构非常松散，是类似 json 的 bson 格式，因此可以存储比较复杂的数据类型。Mongo 最大的特点是它支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

1.1 mongodb 基础概念
----------------

在 mongodb 中是通过数据库、集合、文档的方式来管理数据,下边是 mongodb 与关系数据库的一些概念对比:

![](https://static001.geekbang.org/infoq/d6/d682392cf78873bf1d30cc07538b9851.png)

1、一个 mongodb 实例可以创建多个数据库 2、一个数据库可以创建多个集合 3、一个集合可以包括多个文档。

![](https://static001.geekbang.org/infoq/c1/c17cc63834964729e8253d92d71ba80c.png)

1.2 mongodb 优缺点
---------------

```
优点：
1.读写效率高:由于文档模型把相关数据集中在-块,在普通机械盘上读数据的时候不用花太
多时间去定位磁头,因此在IO性能上有先天独厚的优势
2.可扩展能力强:关系型数据库很难做分布式的原因就是多节点海量数据关联有巨大的性能问
题。如果不考虑关联，数据分区分库,水平扩展就比较简单;
3.动态模式:文档模型支持可变的数据模式,不要求每个文档都具有完全相同的结构,例如在
同一个文档中支持同一一个字段拥有不同的数据类型,对很多异构数据场景支持非常好。
4.模型自然:文档模型最接近于我们熟悉的对象模型,支持数组和嵌套对象作为值。从内存到
存储,无需经过ORM的双向转换,性能上和理解上都很自然易懂。
5.强大的查询语言:丰富而富有表现力的查询语言,使您可以按任意字段进行过滤和排序,无
论它在文档中有多嵌套。支持聚合和其他现代用例,例如基于地理的搜索,图形搜索和文本搜
索。查询本身就是JSON ,因此很容易组合。不再需要串联字符串来动态生成SQL查询。
```

```
缺点：
与关系型数据库,比如mysq|截然不同的查询语法，需 要重新学习mongodb的操作语法。 哈哈
```

可以看 无羡 博主的内容 MongoDB 数据库入门到精通 [https://blog.csdn.net/yuan2019035055/article/details/123031732](https://xie.infoq.cn/link?target=https%3A%2F%2Fblog.csdn.net%2Fyuan2019035055%2Farticle%2Fdetails%2F123031732)  

随着不断的学习内容 python 也安装了很多库，使用之前如何查找是否已经安装了该库可以将将已安装的库列表保存到文本文件夹中然后去文件里搜索是否已经安装了

```
pip freeze >C:\Users\Administrator\Desktop\install.txt
```

**安装文本文件中所有库**

```
pip install -r  C:\Users\Administrator\Desktop\install.txt
```

3.1 python 安装 pymongo 库
-----------------------

**普通安装** :

![](https://static001.geekbang.org/infoq/c0/c026cc167a0dbceeb280e3d6b56d2d31.png)

**卸载已安装的库**:

![](https://static001.geekbang.org/infoq/72/72c9b58979bd69ecf2266a4afc7df3fc.png)

3.2 Python 对 mongodb 增删改查
-------------------------

```
1. 导入pymongodb 模块：import pymongo

待补充
```

代码

```
import pymongo


client = pymongo.MongoClient()


db = client['py_mongo']









collection = db['mongo_col']
```

**insert() 、 remove() 、 update() 、 find()**