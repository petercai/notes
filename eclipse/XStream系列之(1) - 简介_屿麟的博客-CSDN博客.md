# XStream系列之(1) - 简介_屿麟的博客-CSDN博客
[XStream](https://so.csdn.net/so/search?q=XStream&spm=1001.2101.3001.7020)基础
----------------------------------------------------------------------------

> XStream是一个简单的基于Java库，Java对象[序列化](https://so.csdn.net/so/search?q=%E5%BA%8F%E5%88%97%E5%8C%96&spm=1001.2101.3001.7020)到XML，反之亦然(即：可以轻易的将Java对象和xml文档相互转换)。

**特点**

*   使用方便 - XStream的API提供了一个高层次外观，以简化常用的用例。
*   无需创建映射 - XStream的API提供了默认的映射大部分对象序列化。
*   性能 - XStream快速和低内存占用，适合于大对象图或系统。
*   干净的XML - XStream创建一个干净和紧凑XML结果，这很容易阅读。
*   不需要修改对象 - XStream可序列化的内部字段，如私有和最终字段，支持非公有制和内部类。默认构造函数不是强制性的要求。
*   完整对象图支持 - XStream允许保持在对象模型中遇到的重复引用，并支持循环引用。
*   可自定义的转换策略 - 定制策略可以允许特定类型的定制被表示为XML的注册。
*   安全框架 - XStream提供了一个公平控制有关解组的类型，以防止操纵输入安全问题。
*   错误消息 - 出现异常是由于格式不正确的XML时，XStream抛出一个统一的例外，提供了详细的诊断，以解决这个问题。
*   另一种输出格式 - XStream支持其它的输出格式，如JSON。

**常见用途**  
传输, 持久化, 配置, 单元测试。

**下载**  
官网下载：http://x-stream.github.io/download.html  
或者 maven 引入：

```java
<!-- https://mvnrepository.com/artifact/com.thoughtworks.xstream/xstream -->
<dependency>
    <groupId>com.thoughtworks.xstream</groupId>
    <artifactId>xstream</artifactId>
    <version>1.4.11.1</version>
</dependency>

```

**记住以下步骤**

1.  创建XStream 对象

```java
XStream xstream = new XStream(new StaxDriver());

```

2.  序列化对象到XML

```java

String xml = xstream.toXML(student);

```

3.  反序列化XML获得对象

```java

Student student1 = (Student) xstream.fromXML(xml);

```

**类混叠**  
用来创建一个类的[XML](https://so.csdn.net/so/search?q=XML&spm=1001.2101.3001.7020)完全限定名称的别名

```java
XStream.alias(String name, Class type)

```

例如

```java
xstream.alias("student", Student.class);

```

![](https://img-blog.csdnimg.cn/2020071013465872.png)
  
**字段混叠**  
字段混叠用于创建以XML字段的别名。

```java
XStream.aliasField(String alias, Class definedIn, String fieldName)

```

例如

```java
xstream.aliasField("name", Student.class, "studentName");

```

![](https://img-blog.csdnimg.cn/20200710134716391.png)
  
**隐式集合混叠**  
使用的集合是表示在XML无需显示根。例如，在我们的例子中，我们需要一个接一个，但不是在根节点来显示每一个节点。让我们再次修改例子，下面的代码添加到它。

```java
XStream.addImplicitCollection(Class ownerType, String fieldName)

```

例如

```java
xstream.addImplicitCollection(Student.class, "notes");

```

![](https://img-blog.csdnimg.cn/20200710134848944.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAzNDkyNzI=,size_16,color_FFFFFF,t_70)
  
**属性混叠**  
属性混叠用于创建一个成员变量作为XML属性序列化。

```java
xstream.useAttributeFor(Student.class, "studentName");
xstream.aliasField("name", Student.class, "studentName");

```

![](https://img-blog.csdnimg.cn/20200710134933474.png)
  
**包混叠**  
包装混叠用于创建一个类XML的完全限定名称的别名到一个新的限定名称。  
将

```java
xstream.aliasPackage("my.company.xstream", "com.yiibai.xstream");

```

![](https://img-blog.csdnimg.cn/20200710135055646.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAzNDkyNzI=,size_16,color_FFFFFF,t_70)
  
**XStream注解**  
XStream支持注释做同样的任务。在前面的章节中，我们已经看到了下面的代码配置。

```
 `xstream.alias("student", Student.class);
xstream.alias("note", Note.class);

xstream.addImplicitCollection(Student.class, "notes");

xstream.useAttributeFor(Student.class, "studentName");

xstream.aliasField("name", Student.class, "studentName");

@XStreamAlias("student")    
class Student {    
    @XStreamAlias("name")   
    @XStreamAsAttribute     
    private String studentName;
    
    @XStreamImplicit        
    private List<Note> notes = new ArrayList<Note>();
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21


```

为了告诉XStream框架来处理注释，需要XML序列化之前添加下面的命令。

```java
xstream.processAnnotations(Student.class);

```

或者

```java
xstream.autodetectAnnotations(true);

```

XStream高级
---------

**XStream对象流**  
XStream提供java.io.ObjectInputStream和java.io.ObjectOutputStream替代实现，使对象流可以被序列化或XML序列化。当大对象集要被处理，保持在存储器中的一个对象，这是特别有用的。

语法 : createObjectOutputStream()

```java
ObjectOutputStream objectOutputStream = xstream.createObjectOutputStream(new FileOutputStream("test.txt"));

```

语法 :createObjectInputStream()

```java
ObjectInputStream objectInputStream = xstream.createObjectInputStream(new FileInputStream("test.txt"));

```

**XStream自定义转换器**  
XStream允许从无到有写入转换器，这样开发人员可以编写一个完全新的实现，如何对象序列化到XML，反之亦然。 转换器接口提供了三种方法。

*   canConvert - 检查支持的对象类型的序列化。
*   marshal - 序列化对象到XML。
*   unmarshal - 从XML对象反序列化。

```
`package xx.yy;

import com.thoughtworks.xstream.XStream;
import com.thoughtworks.xstream.annotations.XStreamAlias;
import com.thoughtworks.xstream.converters.Converter;
import com.thoughtworks.xstream.converters.MarshallingContext;
import com.thoughtworks.xstream.converters.UnmarshallingContext;
import com.thoughtworks.xstream.io.HierarchicalStreamReader;
import com.thoughtworks.xstream.io.HierarchicalStreamWriter;
import com.thoughtworks.xstream.io.xml.StaxDriver;

public class XStreamTester {
    public static void main(String args[]) {
        XStreamTester tester = new XStreamTester();
        XStream xstream = new XStream(new StaxDriver());
        Student student = tester.getStudentDetails();
        xstream.autodetectAnnotations(true);
        xstream.registerConverter(new StudentConverter()); 
        
        String xml = xstream.toXML(student);
        System.out.println(xml);
    }

    private Student getStudentDetails() {
        Student student = new Student("Mahesh", "Parashar");
        return student;
    }

}

@XStreamAlias("student")
class Student {

    @XStreamAlias("name")
    private Name studentName;

    public Student(String firstName, String lastName) {
        this.studentName = new Name(firstName, lastName);
    }

    public Name getName() {
        return studentName;
    }
}

class Name {
    private String firstName;
    private String lastName;

    public Name(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }
}

class StudentConverter implements Converter {

    public void marshal(Object value, HierarchicalStreamWriter writer,
            MarshallingContext context) {
        Student student = (Student) value;
        writer.startNode("name");
        writer.setValue(student.getName().getFirstName() + ","
                + student.getName().getLastName());
        writer.endNode();
    }

    public Object unmarshal(HierarchicalStreamReader reader,
            UnmarshallingContext context) {
        reader.moveDown();
        String[] nameparts = reader.getValue().split(",");
        Student student = new Student(nameparts[0], nameparts[1]);
        reader.moveUp();
        return student;
    }

    public boolean canConvert(Class object) {
        return object.equals(Student.class);
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48
*   49
*   50
*   51
*   52
*   53
*   54
*   55
*   56
*   57
*   58
*   59
*   60
*   61
*   62
*   63
*   64
*   65
*   66
*   67
*   68
*   69
*   70
*   71
*   72
*   73
*   74
*   75
*   76
*   77
*   78
*   79
*   80
*   81
*   82
*   83
*   84
*   85
*   86
*   87


```

**XStream编写JSON**  
XStream支持JSON通过初始化XStream对象适当的驱动程序。 XStream目前支持JettisonMappedXmlDriver和JsonHierarchicalStreamDriver。 现在，让我们使用XStream处理JSON的代码测试。

```
`package xxx.yyy;

import java.io.Writer;

import com.thoughtworks.xstream.XStream;
import com.thoughtworks.xstream.annotations.XStreamAlias;
import com.thoughtworks.xstream.io.HierarchicalStreamWriter;
import com.thoughtworks.xstream.io.json.JsonHierarchicalStreamDriver;
import com.thoughtworks.xstream.io.json.JsonWriter;

public class XStreamTester {
    public static void main(String args[]) {
        XStream xstream = new XStream(new JsonHierarchicalStreamDriver() {
            public HierarchicalStreamWriter createWriter(Writer writer) {
                return new JsonWriter(writer, JsonWriter.DROP_ROOT_MODE);
            }
        });

        Student student = new Student("Mahesh", "Parashar");
        xstream.setMode(XStream.NO_REFERENCES);
        xstream.alias("student", Student.class);
        System.out.println(xstream.toXML(student));
    }
}

@XStreamAlias("student")
class Student {

    public String firstName;
    public String lastName;

    public Student(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String toString() {
        return "Student [ firstName: " + firstName + ", lastName: " + lastName
                + " ]";
    }
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41


```

**参考**  
XStream教程™  
https://www.yiibai.com/xstream