# DBeaver 破解教程 
![](_assets/P9EIXMZ02AG2R9RC.png)

DBeaver 破解教程 亲测可用（文末附带工具下载）
---------------------------

DBeaver

> 在工作中，我们经常需要使用到数据库操作，这时有一款简洁大方的管理工具是不可缺少的，平时大家比较常用的有Navicat，MySQL Workbench，phpMyAdmin，还有Jetbrains家族下的DataGrip也不错，当然这只是其中几个，市面上还有很多其他优秀的管理工具。而小编今天要推荐给大家的是基于Java开发数据库管理工具，DBeaver，相信不少小伙伴也一定听说过，有的甚至已经在使用，他支持众多数据库，非常的强大，下面我们会简单的介绍，同时会给出相应的永久激活方法。

一、DBeaver是什么？
-------------

**DBeaver**是一个优秀的，通用的数据库管理工具和 SQL 客户端，使用java语言开发，其支持众多数据库产品，比如： MySQL, PostgreSQL, Oracle, DB2, MSSQL, Sybase, Mimer, HSQLDB, Derby, 等其他兼容 JDBC 的数据库。DBeaver 提供一个图形界面用来查看数据库结构、执行SQL查询和脚本，浏览和导出数据，处理BLOB/CLOB 数据，修改数据库结构等等。

同时，dbeaver基础版本是免费和开源（GPL）的。

> GitHub地址：[https://github.com/dbeaver/dbeaver](https://github.com/dbeaver/dbeaver)
> 
> 官网地址：[https://dbeaver.com/](https://dbeaver.com/)

二、为什么推荐DBeaver
--------------

**优点：** Dbeaver支持众多数据库，数据仓库，不仅可以MySQL，Oracle，PostgreSQL，SQL Server，DB2，MariaDB等常用关系数据库连接操作，以及Redis,ClickHouse等 NoSQL，还支持Apache Hive、Spark Hive、Apache Drill、Apache Phoenix、Apache Impala和Apache Cassandra,PrestoSQL等的操作，可以说是真正地一统江湖。既可以满足前端和后台开发者的需求，也可以满足大数据开发者的日常操作，大大提供工作效率，非常适合当下的技术发展，是开发者的不二之选。

**缺点：** 当然，缺点也是有的，因为DBeaver是基于java开发，需要运行在java环境之上，并且需要占用较大的内存资源。但是瑕不掩瑜，个人还是非常推荐。

三、下载安装
------

*   **版本介绍**

DBeaver有简洁版，企业版，旗舰版，社区版（免费版）。除了社区版，其他几个版本都是需要付费的，当然相对来说，功能也要更完善些，其中，旗舰版功能是最完善，今天我们以最新旗舰版作为演示安装和激活操作。

[![](_assets/QQ%E6%88%AA%E5%9B%BE20220309162324-818x1024.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/QQ%E6%88%AA%E5%9B%BE20220309162324-818x1024.jpg)

*   **下载安装包**

下载地址：https://dbeaver.com/download/

[![](_assets/E4R4SYC4XHSLYIAM.png)
](https://www.eyabc.cn/wp-content/uploads/2022/09/E4R4SYC4XHSLYIAM.png)

菜单栏进入下载页面

[![](_assets/002-1024x397.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/002-1024x397.jpg)

选择对应操作系统的DBeaver版本

*   **开始安装**

1\. 双击安装程序，开始向导安装，选择语言

[![](_assets/003.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/003.jpg)

2.选择“下一步”，继续

[![](_assets/004.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/004.jpg)

3.我这里选择仅对当前用户

[![](_assets/0050.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/0050.jpg)

4.选择需要安装的组件，这里“默认”。直接“下一步”

[![](_assets/006.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/006.jpg)

5.选择程序安装目录，我这里更换到F盘下，自定义的目录，下一步

[![](_assets/007.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/007.jpg)

6.开始安装

[![](_assets/008.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/008.jpg)

7.安装完成，添加桌面快捷图标

[![](_assets/009.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/009.jpg)

8.双击桌面图标，进入到DBeaver应用界面

[![](_assets/010.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/010.jpg)

到这里，DBeaver安装已经完成。因为安装的旗舰版，第一次可以免费试用（有期限），也可以通过购买的官方正版激活码进行导入登录，如下图：

[![](_assets/011-1024x575.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/011-1024x575.jpg)

四、开始激活教程
--------

*   **下载激活工具**

激活工具小编已经放到了云盘，在文章末尾的附件中可以直接下载，存放到合适的位置，然后解压后，如下所示，里面包含一个dbeaver-agent.jar的文件。

[![](_assets/018.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/018.jpg)

*   **修改并添加配置**

这里需要手动修改两个地方：

**①、指定自己本地JDK**

因为DBeaver基于java开发，应用内部使用自己的JDK，但是属于被阉割版，所以我们首先需要修改其默认JDK，这里找到自己DBeaver安装目录中dbeaver.ini文件，以文本形式打开，并编辑修改初始化配置文件参数信息。

[![](_assets/013.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/013.jpg)

在首行，添加如下信息：

> -vm  
> C:\\Program Files\\Java\\jdk-11.0.9\\bin

即指定自己本地安装的JDK。如下图：

[![](_assets/017.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/017.jpg)

**②、添加工具路径到DBeaver中加载**

将上面下载的激活工具，放到合适的位置（存放路径中不要有汉字和空格），然后在打开的dbeaver.ini文件末尾行，添加激活工具的路径信息，如上图所示。添加如下信息，下面是我存放激活工具的路径，换成你自己的路径。

> -javaagent:D:\\idejihuo\\dbeaver\\dbeaver-agent\\dbeaver-agent.jar

添加完上面的两处之后，重新启动DBeaver应用，启动正常则进入到应用界面，然后点击“Import License”添加激活码，如下：

[![](_assets/011-1-1024x575.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/011-1-1024x575.jpg)

然后添加激活码，点击“Import”导入。

[![](_assets/012-1.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/012-1.jpg)

激活码如下：

```
aYhAFjjtp3uQZmeLzF3S4H6eTbOgmru0jxYErPCvgmkhkn0D8N2yY6ULK8oT3fnpoEu7GPny7csN
sXL1g+D+8xR++/L8ePsVLUj4du5AMZORr2xGaGKG2rXa3NEoIiEAHSp4a6cQgMMbIspeOy7dYWX6
99Fhtpnu1YBoTmoJPaHBuwHDiOQQk5nXCPflrhA7lldA8TZ3dSUsj4Sr8CqBQeS+2E32xwSniymK
7fKcVX75qnuxhn7vUY7YL2UY7EKeN/AZ+1NIB6umKUODyOAFIc8q6zZT8b9aXqXVzwLJZxHbEgcO
8lsQfyvqUgqD6clzvFry9+JwuQsXN0wW26KDQA==
```

ok，成功激活！！

[![](_assets/0160-1024x577.jpg)
](https://www.eyabc.cn/wp-content/uploads/2022/09/0160-1024x577.jpg)

*   **测试是否激活成功**

点击界面菜单栏Help -> DBeaver License Info，查看激活信息，如下：

[![](_assets/OX6YGMD0A97HEH4QHL.png)
](https://www.eyabc.cn/wp-content/uploads/2022/09/OX6YGMD0A97HEH4QHL.png)

到这里，大功告成，大家可以看到DBeaver的结束日期为空，已经永久激活，可以放心使用了。

常见问题
----

问1：添加完工具，DBeaver无法启动？

答：请确认添加的agent路径是否正确，路径一定要正确，且不可以添加完之后，随便移动。

问2：点击应用启动，提示：”Error opening zip file or JAR manifest missing : dbeaver-agent.jar”

答：需要解压下载的工具包，如果没有解压工具，请先下载解压工具。

问3：明明工具添加正确，一直无法激活DBeaver？

答：请确认工具路径，一定不能含有汉字和空格等特殊字符。

五、最后
----

本教程于Windows系统环境下，基于JDK11开发环境，以官网最新版本DBeaver UE 21.3作为测试演示，以上教程仅供学习参考，请勿用于一切商业用途。

百度网盘: [https://pan.baidu.com/s/1UVK5QrgjyfsM1Y3e57f2WA?pwd=uaxg](https://pan.baidu.com/s/1UVK5QrgjyfsM1Y3e57f2WA?pwd=uaxg)

提取码: uaxg

迅雷网盘：[https://pan.xunlei.com/s/VNAw4pVLierWsh3pPUM5FmcvA1?pwd=cjz6#](https://pan.xunlei.com/s/VNAw4pVLierWsh3pPUM5FmcvA1?pwd=cjz6#)

提取码：cjz6

来源：[http://blog.idejihuo.com/](http://blog.idejihuo.com/db-tools/dbeaver-ultimate-21-3-flagship-free-activation-method.html)