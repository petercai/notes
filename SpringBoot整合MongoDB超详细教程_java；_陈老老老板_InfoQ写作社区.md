# SpringBoot整合MongoDB超详细教程_java；_陈老老老板_InfoQ写作社区
> 陈老老老板
> 
> 说明：工作了，学习一些新的技术栈和工作中遇到的问题，边学习边总结，各位一起加油。需要注意的地方都标红了，还有资源的分享. 一起加油。**本文是介绍 MongoDB 用法与 SpringBoot 整合**

说明： 在整合之前先简单介绍一些 MongoDB 数据库。 MongoDB 是一个由 C++语言编写的、基于分布式文件存储的、开源、高性能、无模式的文档型数据库，在高负载的情况下，添加更多的节点，可以保证服务器性能，MongoDB 旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。它是 NoSQL 数据库产品中的一种，是最像关系型数据库的非关系型数据库。

******以下是常见的使用场景：******

******1.直播数据、打赏数据、粉丝数据******

*   ******存储位置：数据库、Mongodb******
    
*   ******特征：永久性存储与临时存储相结合，修改频度极高******
    

******2.游戏装备数据、游戏道具数据******

*   ******存储位置：数据库、Mongodb******
    
*   ******特征：永久性存储与临时存储相结合、修改频度较高******
    

******3.淘宝/天猫用户数据******

*   ******存储位置：数据库******
    
*   ******特征：永久性存储，修改频度极低******
    

******4.物联网数据******

*   ******存储位置：Mongodb******
    
*   ******特征：临时存储，修改频度飞速******
    

******（1）下载安装包******
--------------------

******去官网地址，windows 版安装包下载地址：******[******https://www.mongodb.com/try/download******](https://xie.infoq.cn/link?target=https%3A%2F%2Fwww.mongodb.com%2Ftry%2Fdownload) ******在下面可以选择安装包。下载的安装包也有两种形式，一种是一键安装的 msi 文件，还有一种是解压缩就能使用的 zip 文件，我们采用解压缩 zip 文件进行安装。******

![](https://img-blog.csdnimg.cn/a7ed178bfcb945f4b4a3c068d0d0cb0d.png)

  

******（2）解压并创建 data 文件夹******
-----------------------------

******解压缩，其中 bin 目录包含了所有 mongodb 的可执行命令。**注：mongodb 在运行时需要指定一个数据存储的目录，所以创建一个数据存储目录，通常放置在安装目录中，此处创建 data 的目录用来存储数据，具体如下********

******（3）启动 MongoDB******
-------------------------

![](https://img-blog.csdnimg.cn/d07c3ec41af14de39f28bf8d95cf908b.png)

  

### ******a.启动服务端******

******在路径进入 cmd 输入一下命令：******

```
<b><b><b><code class="language-CMD">mongod --dbpath=..\data\db
</code></b></b></b>
```

********注：启动服务器时需要指定数据存储位置，通过参数--dbpath 进行设置，可以根据需要自行设置数据存储路径。默认服务端口 27017。********

![](https://img-blog.csdnimg.cn/6e57acf9e59a4e3e9ecead8100de5574.png)

![](https://img-blog.csdnimg.cn/228a8a3f1a26423482135675c2770870.png)

  

### ******b.启动客户端******

******进入 cmd，输入以下命令：******

```
<b><b><b><code class="language-CMD">mongo --host=127.0.0.1 --port=27017
</code></b></b></b>
```

******可以使用数据库客户端，这里使用的是 Navicat，左下角测试，连接成功。******

![](https://img-blog.csdnimg.cn/06f220ffea444207a1d983a4126eaf08.png)

  

********注：连接成功后在命令输入区域输入命令即可操作 MongoDB。这里只说一些简单操作，之后会有专门的 MongoDB 的博客。********

#### ******a.创建数据库：******

******在左侧菜单中使用右键创建，输入数据库名称即可******

#### ******b.创建集合：******

******在 Collections 上使用右键创建，输入集合名称即可，集合等同于数据库中的表的作用******

#### ******c.新增文档：******

******（文档是一种类似 json 格式的数据，初学者可以先把数据理解为就是 json 数据）******

```
<b><b><b><code class="language-CMD">db.集合名称.insert/save/insertOne(文档)
</code></b></b></b>
```

#### ******d.删除文档：******

```
<b><b><b><code class="language-CMD">db.集合名称.remove(条件)
</code></b></b></b>
```

#### ******e.修改文档：******

```
<b><b><b><code class="language-cmd">db.集合名称.update(条件，{操作种类:{文档}})
</code></b></b></b>
```

#### ******f.查询文档：******

```
<b><b><b><code class="language-CMD">基础查询
查询全部：       db.集合.find();
查第一条：       db.集合.findOne()
查询指定数量文档：  db.集合.find().limit(10)          //查10条文档
跳过指定数量文档：  db.集合.find().skip(20)          //跳过20条文档
统计：          db.集合.count()
排序：        db.集合.sort({age:1})            //按age升序排序
投影：        db.集合名称.find(条件,{name:1,age:1})     //仅保留name与age域

条件查询
基本格式：      db.集合.find({条件})
模糊查询：      db.集合.find({域名:/正则表达式/})      //等同SQL中的like，比like强大，可以执行正则所有规则
条件比较运算：       db.集合.find({域名:{$gt:值}})        //等同SQL中的数值比较操作，例如：name>18
包含查询：      db.集合.find({域名:{$in:[值1，值2]}})    //等同于SQL中的in
条件连接查询：       db.集合.find({$and:[{条件1},{条件2}]})     //等同于SQL中的and、or
</code></b></b></b>
```

******（1）创建项目******
-------------------

![](https://img-blog.csdnimg.cn/60ce94ff4a3947a38db9d23d4f4e3627.png)

******这里用的阿里创建的项目******

![](https://img-blog.csdnimg.cn/bde54f58bb1d488cbc47c2de73934145.png)

![](https://img-blog.csdnimg.cn/8de21778e499407ab75823fbc010c1e2.png)

  

******（2）导入 springboot 整合 MongoDB 的 starter 坐标******
----------------------------------------------------

******当创建项目时候就已经有这个坐标了。******

```
<b><b><b><code class="language-xml">  <!-- 引入mongodb-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
      
</code></b></b></b>
```

******（3）基础配置******
-------------------

******这里只进行简单的配置无密码：******

```
<b><b><b><code class="language-yaml">spring:
  data:
    mongodb:
      uri: mongodb://服务器IP:端口/数据库名
</code></b></b></b>
```

******有密码：******

```
<b><b><b><code class="language-yaml">spring:
  data:
    mongodb:
      uri: mongodb://用户名:密码@服务器IP:端口/数据库名
      # 上方为明确指定某个数据的用户进行连接
      # 也可以使用admin 数据库中的用户进行连接  统一到admin 数据库进行认证
      # admin 用户认证 url 写法： mongodb://账户:密码%40@ip:端口/数据库名?authSource=admin&authMechanism=SCRAM-SHA-1
</code></b></b></b>
```

******例：******

```
<b><b><b><code class="language-yaml">spring:
  data:
    mongodb:
      uri: mongodb://localhost/mongotest
</code></b></b></b>
```

******​******

******（4）使用 MongoTemplate******
-------------------------------

********注：使用 springboot 整合 MongoDB 的专用客户端接口 MongoTemplate 来进行操作实体类：********

```
<b><b><b><code class="language-java">package com.test;


public class Book {
    private Integer id;
    
    private String name;
    
    private String type;
    
    private String description;

    public Book(Integer id, String name, String type, String description) {
        this.id = id;
        this.name = name;
        this.type = type;
        this.description = description;
    }

    public Book() {
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    @Override
    public String toString() {
        return "Book{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", type='" + type + '\'' +
                ", description='" + description + '\'' +
                '}';
    }
}
</code></b></b></b>
```

******测试类：******

```
<b><b><b><code class="language-java">@SpringBootTest
class Springboot17MongodbApplicationTests {
    @Autowired
    private MongoTemplate mongoTemplate;
    @Test
    void contextLoads() {
        Book book = new Book();
        book.setId(10);
        book.setName("testMongoDB");
        book.setType("testMongoDB");
        book.setDescription("testMongoDB");
        mongoTemplate.save(book);
    }
    @Test
    void find(){
        List<Book> all = mongoTemplate.findAll(Book.class);
        System.out.println(all);
    }
}
</code></b></b></b>
```

******先支持插入，在执行查询，可以查询到数据（这里有我之前测试的数据），如下：******

> ********总结：MongoDB 是工作中很常见的数据库，MongoTemplate 中有非常多的方法，之后会有更细致的有关 mongoDB 的博客。希望对您有帮助，感谢阅读结束语：裸体一旦成为艺术，便是最圣洁的。道德一旦沦为虚伪，便是最下流的。勇敢去做你认为正确的事，不要被世俗的流言蜚语所困扰。********