# SpringBoot整合EasyExcel超详细教学_spring-boot_陈老老老板_InfoQ写作社区
> 陈老老老板
> 
> 说明：工作了，学习一些新的技术栈和工作中遇到的问题，边学习边总结，各位一起加油。需要注意的地方都标红了，还有资源的分享. 一起加油。**本文是介绍 EasyExcel 详解与 SpringBoot 整合**
> 
> **git资源地址：https://gitee.com/lt\_xu/easy-excel-demo.git**
> 
>   

**1.EasyExcel 简介**
------------------

****简介：可以去官网看看，官网介绍文档也很全面。****[****EasyExcel官网****](https://xie.infoq.cn/link?target=https%3A%2F%2Feasyexcel.opensource.alibaba.com%2Fdocs%2Fcurrent%2F)****EasyExcel 是一个==基于 Java 的简单、省内存的读写 Excel 的开源项目==。在尽可能节约内存的情况下支持读写百 M 的 Excel。（其实总体来说就是占内存小，响应快，写法简单）****

****1.2.重写 POI 简化开发****
-----------------------

*   ****EasyExcel 重写了 POI 对 07 版 Excel 的解析，可以把内存消耗从 100M 左右降低到 10M 以内，并且再大的 Excel 不会出现内存溢出，03 版仍依赖 POI 的 SAX 模式。****
    
*   ****在上层做了模型转换的封装，让使用者更加简单方便****
    

****1.3. 特点****
---------------

1.  ****在数据模型层面进行了封装，使用简单****
    
2.  ****重写了 07 版本的 Excel 的解析代码，降低内存消耗，能有效避免 OOM****
    
3.  ****只能操作 Excel****
    
4.  ****不能读取图片****
    

****2\. Apache POI 简介****
-------------------------

******简介：******`******Apache POI******`******是 Apache 软件基金会的开源函式库，提供跨平台的******`******Java API******`******实现******`******Microsoft Office******`******格式档案读写。但是存在如下一些问题：******

### ******2.1 学习使用成本较高******

******对 POI 有过深入了解的才知道原来 POI 还有 SAX 模式（Dom 解析模式）。但 SAX 模式相对比较复杂，excel 有 03 和 07 两种版本，两个版本数据存储方式截然不同，sax 解析方式也各不一样。******

******学习这两种解析方式，在转换到自己的业务模块是需要好久的学习时间，并且由于代码过于复杂，之后维护成本非常大。******

******POI 的 SAX 模式的 API 可以一定程度的解决一些内存溢出的问题，但是 POI 还是有一些缺陷，比如 07 版 Excel 解压缩以及解压后存储都是在内存中完成的，内存消耗依然很大，一个 3M 的 Excel 用 POI 的 SAX 解析，依然需要 100M 左右内存。******

### ******2.2 POI 的内存消耗较大******

******大部分使用 POI 都是使用他的 userModel 模式。userModel 的好处是上手容易使用简单，随便拷贝个代码跑一下，剩下就是写业务转换了，虽然转换也要写上百行代码，相对比较好理解。然而 userModel 模式最大的问题是在于非常大的内存消耗，一个几兆的文件解析要用掉上百兆的内存。现在很多应用采用这种模式，之所以还正常在跑一定是并发不大，并发上来后一定会 OOM 或者频繁的 full gc。******

******总体上来说，简单写法重度依赖内存，复杂写法学习成本高。******

#### ******特点******

1.  ******功能强大******
    
2.  ******代码书写冗余繁杂******
    
3.  ******读写大文件耗费内存较大，容易 OOM******
    

![](https://img-blog.csdnimg.cn/b973356473c445e7a74da0baa5fb6a40.png)

  

******2.导入依赖坐标******
--------------------

********注：easyexcel 坐标，最新 3.1.1 版本，推荐使用，一直在更新的版本。********

```
<b><b><b><code class="language-xml">        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>easyexcel</artifactId>
            <version>3.1.1</version>
        </dependency>
</code></b></b></b>
```

********注：项目全部坐标，这里用到 lombok 插件，开发更简单。********

```
<b><b><b><code class="language-xml"><dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
        <!--easyExcel-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>easyexcel</artifactId>
            <version>3.1.1</version>
        </dependency>
</code></b></b></b>
```

******3.创建实体类******
-------------------

********注：创建 Student 实体类，用来与 excel 表头映射。 这里实体类顺序必须和 excel 中表头顺序一致，否则会报错，这里应该是最新版本的问题。也可以用注解映射在后面往 excel 写的时候会讲。********

```
<b><b><b><code class="language-java">// 基于lombok
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Student {
    /**
     * 学生姓名
     */
    private String name;
 /**
     * 学生出生日期
     */
    private Date birthday;
    /**
     * 学生性别
     */
    private String gender;
    /**
     * id
     */
    private String id;
}
</code></b></b></b>
```

******fillData 数据测试类******

```
<b><b><b><code class="language-java">@Data
public class FillData {

    private String name;
    private int age;
}
</code></b></b></b>
```

******4.创建监听器******
-------------------

********注：必须继承 AnalysisEventListener<> 实现 invoke 与 doAfterAllAnalysed 方法。********

```
<b><b><b><code class="language-java">public class StudentReadListener extends AnalysisEventListener<Student> {
    // 每读一次，会调用该invoke方法一次，就是想对获取到的数据进行什么操作
    @Override
    public void invoke(Student data, AnalysisContext context) {
        System.out.println("data = " + data);
        log.info(data + "保存成功");
    }

    // 全部读完之后，会调用该方法（这个暂时用不到）
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        // TODO......
    }
}
</code></b></b></b>
```

******5.在测试类中加入数据生成方法******
---------------------------

```
<b><b><b><code class="language-java">  //    数据生成
    private static List<Student> initData(){
        List<Student> students=new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Student data=new Student();
            data.setName("测试"+i+2);
            data.setGender("男");
            data.setBirthday(new Date());
            students.add(data);
        }
        System.out.println(students);
        return students;
    }
</code></b></b></b>
```

******1.读取******`******Excel******`******文件******
-------------------------------------------------

********注：这里 read 的第一个参数表示要读 excel 文件地址，先将文件拉取到项目中，右键点击 excel 文件，点击 Copy Path 进行复制。最后点击运行该方法就可以看到读取到 excel 中的数据。********

```
<b><b><b><code class="language-java">    @Test
    public void test01(){
     /**
         * Build excel the read  构建一个读的工作簿
         *
         * @param pathName  读文件的路径
         *            File path to read.
         * @param head  每一行数据存储的到的实体类的类型的class
         *            Annotate the class for configuration information.
         * @param readListener  监听器 没读一行内容都会调用该对象的invoke，在invoke可以操作使用读取到的数据
         *            Read listener.
         * @return Excel reader builder.
         */
//        获取工作簿对象            excel文件位置   最后一个参数是监听器类
        ExcelReaderBuilder readWorkBook = EasyExcel.read("E:\blank\EasyExcel\杭州黑马在线202003班学员信息表.xlsx", Student.class, new StudentListener());
//        获取工作表对象
        ExcelReaderSheetBuilder sheet = readWorkBook.sheet();
//        读取表中的内容
        sheet.doRead();
</code></b></b></b>
```

******2.写入 Excel******
----------------------

********注：实体类中加入注解 @ExcelProperty，注解中 value 的值代表，写入 excel 的表头。********

```
<b><b><b><code class="language-java">
@Data
@AllArgsConstructor
@NoArgsConstructor
@ColumnWidth(20)
public class Student {


    /**
     * id
     */ 
    //index 代表表头的位置，否则默认 各个参数在本类中的位置
    //@ExcelProperty(value = "编号",index = 3)
    //忽略在表中不会显示该字段
    @ExcelIgnore
    private String id;
    /**
     * 学生姓名
     */
    @ExcelProperty(value = "学生姓名")
    //表示列的宽度
    //@ColumnWidth(30)
    private String name;
    /**
     * 学生性别
     */
    @ExcelProperty(value = "学生性别")
    private String gender;

    /**
     * 学生出生日期
     */
    @ExcelProperty(value = "学生出生日期")
    //@ColumnWidth(20)
    private Date birthday;
}
</code></b></b></b>
```

********注：测试类中加入写入 excel 的方法********

```
<b><b><b><code class="language-java"> @Test
    public void test02(){
        ExcelWriterBuilder writeWorkBook = EasyExcel.write("E:\blank\EasyExcel\在线学员信息表.xlsx", Student.class);
        ExcelWriterSheetBuilder sheet = writeWorkBook.sheet();
        //doWrite(initData()之前提前加入的生成数据的方法。
        sheet.doWrite(initData());
    }
</code></b></b></b>
```

> ********总结：EasyExcel 简化开发，减少内存，必学。希望对您有帮助，感谢阅读结束语：裸体一旦成为艺术，便是最圣洁的。道德一旦沦为虚伪，便是最下流的。勇敢去做你认为正确的事，不要被世俗的流言蜚语所困扰。********