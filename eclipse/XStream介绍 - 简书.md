# XStream介绍 - 简书
[![](_assets/7bb6532d8f48.jpeg.webp)
](https://www.jianshu.com/u/c349fdb89f87)

0.1782020.07.04 05:58:12字数 800阅读 2,893

XStream是一个实现javaBean与XML互相转换的工具，极大地简化了开发人员的对XML的处理

### 1、添加依赖

```
<dependency>
    <groupId>com.thoughtworks.xstream</groupId>
    <artifactId>xstream</artifactId>
    <version>1.4.11.1</version>
</dependency> 
```

### 2、UML类图

```
<Document TaskGuid="560eec8a-be14-4398-8c9a-73b68651a129" DataGuid="" DataType=" QueryJYYKTByHourList" >
    <Data>
        <KBH Type="TEXT">卡编号</KBH>
        <DEALRQ Type="TEXT">刷卡日期</DEALRQ>
        <DEALSJ Type="TEXT">刷卡时间</DEALSJ>
        <XLBM Type="TEXT">线路编码</XLBM>
    </Data>
    <Data>
        <KBH Type="TEXT">卡编号</KBH>
        <DEALRQ Type="TEXT">刷卡日期</DEALRQ>
        <DEALSJ Type="TEXT">刷卡时间</DEALSJ>
        <XLBM Type="TEXT">线路编码</XLBM>
    </Data>
…数据重复出现
</Document> 
```

假设现在需要将上面的xml转换为一个JavaBean，那么首先需要分析这个XML的结构，根节点是Document，有三个属性TaskGuid、DataGuid、DataType，有许多个子节点Data，每个Data节点都有四个子节点：KBH、DEALRQ、DEALSJ、XLBM。可以采用这种思路：用一个Document类：有四个字段，TaskGuid、DataGuid、DataType与一个Data的集合。每一个Data都有四个字段：KBH、DEALRQ、DEALSJ、XLBM，每个字段都有一个Type和一个值。

UML类图如下：

![](_assets/1287379-ad2a0dfda31c4fed.png.webp)

类的关系.png

Document和Data是聚合关系，Data脱离了Document可以独立存在，Data和Kbh、Dealrq、Dealsj与Xlbm是组合关系，这四个脱离了Data就不能独立存在。

### 3、XStream注解

**主要使用**  
**`@XStreamAlias(“alis”)`java对象在xml中以标签的形式显示时，如果名字与类名或者属性名不一致，可以使用该标签并在括号内注明别名。**   
`@XStreamOmitField`在输出XML的时候忽略该属性  
**`@XStreamImplicit`如果该属性是一个列表或者数组，在XML中不显示list或者Array字样**  
**`@XStreamAsAttribute`该属性不单独显示成XML节点，而是作为属性显示出来**  
`@XStreamConverter`设置转换器  
`@XStreamConverters` converter主要用于将某些字段进行复杂的转换，转换过程写在一个类中。  
然后将其注册到XStream。

工具类：

```
public class XsteamUtil {
    public static Object toBean(Class<?> clazz, String xml) {
        Object xmlObject = null;
            XStream xstream = new XStream();
            xstream.processAnnotations(clazz);
            xstream.autodetectAnnotations(true);
            xmlObject= xstream.fromXML(xml);
            return xmlObject;
    }
} 
```

在SpringBoot中可以使用@Bean来进行IOC，设置xstream.autodetectAnnotations(true);来处理注解

如果要进行xml转JavaBean的操作，需要设施xstream.alias()

### 4、具体使用

ReturnResult.java

```
import com.thoughtworks.xstream.annotations.XStreamAlias;
import com.thoughtworks.xstream.annotations.XStreamAsAttribute;
import com.thoughtworks.xstream.annotations.XStreamImplicit;
import lombok.Data;
import lombok.ToString;
import lombok.experimental.Accessors;

import java.util.List;

@Data
@ToString
@Accessors(chain = true)
@XStreamAlias("Document")
public class ReturnResult {
    @XStreamAlias("TaskGuid")
    @XStreamAsAttribute
    private String taskGuid;
    @XStreamAlias("DataGuid")
    @XStreamAsAttribute
    private String dataGuid;
    @XStreamAlias("DataType")
    @XStreamAsAttribute
    private String dataType;
    @XStreamImplicit
    private List<Record> records;
} 
```

使用了`@XStreamAlias`注解给类与字段起别名，`ReturnResult`就对应根节点`Document`，Java代码中变量名提倡使用小写驼峰命名，所以我对字段也使用的别名。`@XStreamAsAttribute`不将字段作为节点来对待而是作为属性。 `@XStreamImplicit`不显示最外层的集合的根节点。

Record.java

```
import com.thoughtworks.xstream.annotations.XStreamAlias;
import lombok.Data;
import lombok.experimental.Accessors;

@Data
@Accessors(chain = true)
@XStreamAlias("Data")
public class Record {
    @XStreamAlias("KBH")
    private CardNo cardNo;
    @XStreamAlias("DEALRQ")
    private DealDate dealDate;
    @XStreamAlias("DEALSJ")
    private DealTime dealTime;
    @XStreamAlias("XLBM")
    private RouteCode routeCode;
} 
```

由于XML文件的节点名不够见名知意，所以我使用`@XStreamAlias`的方式来解决。

CardNo.java

```
import com.thoughtworks.xstream.annotations.XStreamAlias;
import com.thoughtworks.xstream.annotations.XStreamConverter;
import com.thoughtworks.xstream.converters.extended.ToAttributedValueConverter;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.Accessors;

@Data
@NoArgsConstructor
@Accessors(chain = true)
@XStreamAlias("KBH")
@XStreamConverter(value= ToAttributedValueConverter.class, strings={"value"})
public class CardNo {
    @XStreamAlias("Type")
    private String type;
    private String value;
} 
```

一个`<KBH Type="TEXT">卡编号</KBH>`节点既有属性又有值，所以使用了`@XStreamConverter`，用xstream自带的转换器进行转换。

```
@Test
public void testReturnResult2(){
    XStream xStream = new XStream();
    
    xStream.autodetectAnnotations(true);
    List<Record> records = new ArrayList<>();
    
    
    CardNo k1 = new CardNo().setType("TEXT").setValue("36283723");
    CardNo k2 = new CardNo().setType("TEXT").setValue("121211");
    DealTime dealTime1 = new DealTime().setType("TEXT").setValue("12");
    DealTime dealTime2 = new DealTime().setType("TEXT").setValue("1");
    DealDate dealDate1 = new DealDate().setType("TEXT").setValue("20200617");
    DealDate dealDate2 = new DealDate().setType("TEXT").setValue("20200520");
    RouteCode routeCode1 = new RouteCode().setType("TEXT").setValue("212");
    RouteCode routeCode2 = new RouteCode().setType("TEXT").setValue("112");
    records.add(new Record().setCardNo(k1).setDealDate(dealDate1).setDealTime(dealTime1).setRouteCode(routeCode1));
    records.add(new Record().setCardNo(k2).setDealDate(dealDate2).setDealTime(dealTime2).setRouteCode(routeCode2));
    ReturnResult result = new ReturnResult().setDataType("QueryJYYKTByHourList").setTaskGuid("560eec8a-be14-4398-8c9a-73b68651a129")
            .setDataGuid("d78whe28eh").setRecords(records);
    
    
    String s = xStream.toXML(result);
    String s1 =
            "<Document TaskGuid=\"560eec8a-be14-4398-8c9a-73b68651a129\" DataGuid=\"d78whe28eh\" DataType=\"QueryJYYKTByHourList\">\n" +
                    "  <Data>\n" +
                    "    <KBH Type=\"TEXT\">36283723</KBH>\n" +
                    "    <DEALRQ Type=\"TEXT\">20200617</DEALRQ>\n" +
                    "    <DEALSJ Type=\"TEXT\">12</DEALSJ>\n" +
                    "    <XLBM Type=\"TEXT\">212</XLBM>\n" +
                    "  </Data>\n" +
                    "  <Data>\n" +
                    "    <KBH Type=\"TEXT\">121211</KBH>\n" +
                    "    <DEALRQ Type=\"TEXT\">20200520</DEALRQ>\n" +
                    "    <DEALSJ Type=\"TEXT\">1</DEALSJ>\n" +
                    "    <XLBM Type=\"TEXT\">112</XLBM>\n" +
                    "  </Data>\n" +
                    "</Document>";
    System.out.println(s);
    System.out.println(xStream.fromXML(s1));
} 
```

上面的例子将JavaBean转换为xml。

```
@Test
    public void testReturnData(){
        XStream xStream = new XStream();
        xStream.autodetectAnnotations(true);
        
        xStream.alias("Document",ReturnResult.class);
        String s = "<Document TaskGuid=\"560eec8a-be14-4398-8c9a-73b68651a129\" DataGuid=\"dhshdisds89e\" DataType=\"QueryJYYKTByHourList\">\n" +
                "  <Data>\n" +
                "    <KBH Type=\"TEXT\">12121</KBH>\n" +
                "    <DEALRQ Type=\"TEXT\">2020212</DEALRQ>\n" +
                "    <DEALSJ Type=\"TEXT\">12</DEALSJ>\n" +
                "    <XLBM Type=\"TEXT\">12</XLBM>\n" +
                "  </Data>\n" +
                "  <Data>\n" +
                "    <KBH Type=\"TEXT\">37612</KBH>\n" +
                "    <DEALRQ Type=\"TEXT\">2020312</DEALRQ>\n" +
                "    <DEALSJ Type=\"TEXT\">15</DEALSJ>\n" +
                "    <XLBM Type=\"TEXT\">23</XLBM>\n" +
                "  </Data>\n" +
                "</Document>";
        System.out.println(xStream.fromXML(s));
    } 
```

上面的例子将xml形式的字符串转换为JavaBean，需要注意的是必须要加**xStream.alias("Document",ReturnResult.class);**这行代码，否则会报**com.thoughtworks.xstream.mapper.CannotResolveClassException: Document**这个异常。

可以看出使用XStream的一般步骤：

1.  创建XStream对象:XStream xStream = new XStream();
2.  开始自动检测注解:xStream.autodetectAnnotations(true);
3.  进行转换：fromXML或者toXML

既让这个XStrem对象经常使用，那么在Spring项目中可以使用Spring IOC将它交给Spring容器进行管理，在需要的时候进行注入即可。

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![](_assets/7bb6532d8f48.jpeg.webp)
](https://www.jianshu.com/u/c349fdb89f87)

总资产5共写了17.3W字获得90个赞共47个粉丝

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

*   最近项目有一个跟三方交互的接口，三方用的xml数据，简单记录下玩xml的历程。 首先，对于正常的xml解析，推荐J...
    
    [![](_assets/4992676-d18326b0fb51b7b5.png.webp)
    ](https://www.jianshu.com/p/634bf6b5de12)
*   1\. Java基础部分 基础部分的顺序：基本语法，类相关的语法，内部类的语法，继承相关的语法，异常的语法，线程的语...
    
    [![](_assets/c8f05531-a36a-4357-9b04-4091e73ebf19.jpg.webp)
    子非鱼\_t\_](https://www.jianshu.com/u/0ac615e458d5)阅读 30,489评论 18赞 399
    
*   XStream的介绍 使用问题及解决方法 XStream的基本使用 pom依赖 Xstream序列化XML 程序运...
    
*   在现今的项目开发中，虽然数据的传输大部分都是用json格式来进行传输，但是xml毕竟也会有一些老的项目在进行使用，...
    
    [![](_assets/2582c104-c45e-4b14-8791-27f69f278a01.jpg.webp)
    意识流丶](https://www.jianshu.com/u/f192766abeab)阅读 39,982评论 3赞 25
    
*   5月以来，哪怕对市场风向再不敏感的人，也感觉到阵阵凉意。二级市场连续下挫，一级市场融资环境恶化，不论企业融资数量还...
    
    [![](_assets/12667735-98f5a5fbd6f10663.webp)
    ](https://www.jianshu.com/p/7021ed6a64e4)
*   推荐指数： 6.0 书籍主旨关键词：特权、焦点、注意力、语言联想、情景联想 观点： 1.统计学现在叫数据分析，社会...
    
*   昨天，在回家的路上，坐在车里悠哉悠哉地看着三毛的《撒哈拉沙漠的故事》，我被里面的内容深深吸引住了，尽管上学时...
    
    [![](_assets/e6d62766-fb51-42d3-9cc7-c7e370d2b89b.jpg.webp)
    夜阑晓语](https://www.jianshu.com/u/80cbc3a42257)阅读 3,344评论 2赞 8
    
*   [![](_assets/12994822-34dc7ca7bbbb7c10.jpg.webp)
    ](https://www.jianshu.com/p/70d19adb54c2)
*   一月四号的大沙有个想法。从昨晚到现在就一直围绕在脑子里。或许深受那些小说的影响，或许真的就是我自己脑子或者精神么有...