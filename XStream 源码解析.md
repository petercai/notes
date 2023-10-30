# XStream 源码解析

项目地址：[XStream](https://github.com/x-stream/xstream)， 分析的版本: [v1.4.11](https://github.com/x-stream/xstream/tree/XSTREAM_1_4_11_1)，Demo地址：[TestStream](https://github.com/x-stream/xstream/tree/XSTREAM_1_4_11_1/xstream/src/test/com/thoughtworks/xstream)， 相关jar[下载地址](https://github.com/jakingting/Xtream-jar)

本文结构

1、功能介绍  
2、总体设计  
3、详细设计  
4、如何提升XStream 解析速度  
5、自定义Mapper和自定义Converter  
6、总结

1、功能介绍
------

### 1.1简介

提供给Java使用的XML序列化工具

### 1.2 如何使用

```
public void testPopulationOfAnObjectGraphStartingWithALiveRootObject() throws Exception {
        final XStream xstream = new XStream();
        
        xstream.allowTypesByWildcard(this.getClass().getName() + "$*");
        
        xstream.autodetectAnnotations(true);
        final String xml = ""
                + "<component>"
                + "  <Host>host</Host>"
                + "  <port>8000</port>"
                + "</component>";
        
        xstream.alias("component", Component.class);
        
        final Component component = xstream.fromXML(xml);
    }
    
    public static class Component {
                
                @XStreamAlias("Host")
        String host;
        int port;
    } 
```

2、总体设计
------

### 2.1 XStream 总体由五部分组成

*   **XStream** 作为客户端对外提供XML解析与转换的相关方法。
    
*   **AbstractDriver** 为XStream提供流解析器和编写器的创建。目前支持XML（DOM，PULL）、JSON解析器。解析器**HierarchicalStreamReader**，编写器**HierarchicalStreamWriter**（PS：**XStream**默认使用了**XppDriver**）。
    
*   **MarshallingStrategy** 编组和解组策略的核心接口，两个方法：  
    marshal：编组对象图  
    unmarshal:解组对象图  
    **TreeUnmarshaller** 树解组程序，调用mapper和Converter把XML转化成java对象，里面的start方法开始解组，convertAnother方法把class转化成java对象。  
    **TreeMarshaller** 树编组程序，调用mapper和Converter把java对象转化成XML，里面的start方法开始编组，convertAnother方法把java对象转化成XML。  
    它的抽象子类**AbstractTreeMarshallingStrategy**有抽象两个方法  
    createUnmarshallingContext  
    createMarshallingContext  
    用来根据不同的场景创建不同的**TreeUnmarshaller**子类和**TreeMarshaller**子类，使用了**策略模式**，如ReferenceByXPathMarshallingStrategy创建ReferenceByXPathUnmarshaller，ReferenceByIdMarshallingStrategy创建ReferenceByIdUnmarshaller（PS：**XStream**默认使用**ReferenceByXPathMarshallingStrategy**）。
    
*   **Mapper** 映射器，XML的elementName通过mapper获取对应类、成员、属性的class对象。支持解组和编组，所以方法是成对存在real 和serialized，他的子类**MapperWrapper**作为装饰者，包装了不同类型映射的映射器，如AnnotationMapper，ImplicitCollectionMapper，ClassAliasingMapper。
    
*   **ConverterLookup** 通过Mapper获取的Class对象后，接着调用lookupConverterForType获取对应Class的转换器，将其转化成对应实例对象。**DefaultConverterLookup**是该接口的实现类，同时实现了**ConverterRegistry**的接口，所有**DefaultConverterLookup**具备查找converter功能和注册converter功能。所有注册的转换器按一定优先级组成由**TreeSet**保存的有序集合(PS:**XStream** 默认使用了**DefaultConverterLookup**)。
    

3、详细设计
------

![](https://upload-images.jianshu.io/upload_images/89809-1a18dc65647577ef.png)

image.png

  
[StartUML写的XStreamUML原文件和图片下载地址](https://github.com/jakingting/Xtream-jar/tree/master/XStreamUML)

4、核心功能介绍
--------

### 4.1 Xstream中Mapper的原理

#### 1\. Mapper的构建过程

```
 private Mapper buildMapper() {
        Mapper mapper = new DefaultMapper(classLoaderReference);
        if (useXStream11XmlFriendlyMapper()) {
            mapper = new XStream11XmlFriendlyMapper(mapper);
        }
        mapper = new DynamicProxyMapper(mapper);
        mapper = new PackageAliasingMapper(mapper);
        mapper = new ClassAliasingMapper(mapper);
        mapper = new ElementIgnoringMapper(mapper);
        mapper = new FieldAliasingMapper(mapper);
        mapper = new AttributeAliasingMapper(mapper);
        mapper = new SystemAttributeAliasingMapper(mapper);
        mapper = new ImplicitCollectionMapper(mapper, reflectionProvider);
        mapper = new OuterClassMapper(mapper);
        mapper = new ArrayMapper(mapper);
        mapper = new DefaultImplementationsMapper(mapper);
        mapper = new AttributeMapper(mapper, converterLookup, reflectionProvider);
        mapper = new EnumMapper(mapper);
        mapper = new LocalConversionMapper(mapper);
        mapper = new ImmutableTypesMapper(mapper);
        if (JVM.isVersion(8)) {
            mapper = buildMapperDynamically("com.thoughtworks.xstream.mapper.LambdaMapper", new Class[]{Mapper.class},
                new Object[]{mapper});
        }
        mapper = new SecurityMapper(mapper);
        mapper = new AnnotationMapper(mapper, converterRegistry, converterLookup, classLoaderReference,
            reflectionProvider);
        mapper = wrapMapper((MapperWrapper)mapper);
        mapper = new CachingMapper(mapper);
        return mapper;
    } 
```

```
 public PackageAliasingMapper(final Mapper wrapped) {
        
        super(wrapped);
    } 
```

```
 public MapperWrapper(final Mapper wrapped) {
        this.wrapped = wrapped;
        
        if (wrapped instanceof MapperWrapper) {
            
            final MapperWrapper wrapper = (MapperWrapper)wrapped;
            final Map<String, Mapper> wrapperMap = new HashMap<>();
            wrapperMap.put("aliasForAttribute", wrapper.aliasForAttributeMapper);
            wrapperMap.put("aliasForSystemAttribute", wrapper.aliasForSystemAttributeMapper);
            wrapperMap.put("attributeForAlias", wrapper.attributeForAliasMapper);
            wrapperMap.put("defaultImplementationOf", wrapper.defaultImplementationOfMapper);
            wrapperMap.put("getConverterFromAttribute", wrapper.getConverterFromAttributeMapper);
            wrapperMap.put("getConverterFromItemType", wrapper.getConverterFromItemTypeMapper);
            wrapperMap.put("getFieldNameForItemTypeAndName", wrapper.getFieldNameForItemTypeAndNameMapper);
            wrapperMap.put("getImplicitCollectionDefForFieldName", wrapper.getImplicitCollectionDefForFieldNameMapper);
            wrapperMap.put("getItemTypeForItemFieldName", wrapper.getItemTypeForItemFieldNameMapper);
            wrapperMap.put("getLocalConverter", wrapper.getLocalConverterMapper);
            wrapperMap.put("isIgnoredElement", wrapper.isIgnoredElementMapper);
            wrapperMap.put("isImmutableValueType", wrapper.isImmutableValueTypeMapper);
            wrapperMap.put("isReferenceable", wrapper.isReferenceableMapper);
            wrapperMap.put("realClass", wrapper.realClassMapper);
            wrapperMap.put("realMember", wrapper.realMemberMapper);
            wrapperMap.put("serializedClass", wrapper.serializedClassMapper);
            wrapperMap.put("serializedMember", wrapper.serializedMemberMapper);
            wrapperMap.put("shouldSerializeMember", wrapper.shouldSerializeMemberMapper);

            final Method[] methods = wrapped.getClass().getMethods();
            for (final Method method : methods) {
                
                if (method.getDeclaringClass() != MapperWrapper.class) {
                    final String name = method.getName();
                    if (wrapperMap.containsKey(name)) {
                        
                        wrapperMap.put(name, wrapped);
                    }
                }
            }
            
            aliasForAttributeMapper = wrapperMap.get("aliasForAttribute");
            aliasForSystemAttributeMapper = wrapperMap.get("aliasForSystemAttribute");
            attributeForAliasMapper = wrapperMap.get("attributeForAlias");
            defaultImplementationOfMapper = wrapperMap.get("defaultImplementationOf");
            getConverterFromAttributeMapper = wrapperMap.get("getConverterFromAttribute");
            getConverterFromItemTypeMapper = wrapperMap.get("getConverterFromItemType");
            getFieldNameForItemTypeAndNameMapper = wrapperMap.get("getFieldNameForItemTypeAndName");
            getImplicitCollectionDefForFieldNameMapper = wrapperMap.get("getImplicitCollectionDefForFieldName");
            getItemTypeForItemFieldNameMapper = wrapperMap.get("getItemTypeForItemFieldName");
            getLocalConverterMapper = wrapperMap.get("getLocalConverter");
            isIgnoredElementMapper = wrapperMap.get("isIgnoredElement");
            isImmutableValueTypeMapper = wrapperMap.get("isImmutableValueType");
            isReferenceableMapper = wrapperMap.get("isReferenceable");
            realClassMapper = wrapperMap.get("realClass");
            realMemberMapper = wrapperMap.get("realMember");
            serializedClassMapper = wrapperMap.get("serializedClass");
            serializedMemberMapper = wrapperMap.get("serializedMember");
            shouldSerializeMemberMapper = wrapperMap.get("shouldSerializeMember");
        } else {
            
            aliasForAttributeMapper = wrapped;
            aliasForSystemAttributeMapper = wrapped;
            attributeForAliasMapper = wrapped;
            defaultImplementationOfMapper = wrapped;
            getConverterFromAttributeMapper = wrapped;
            getConverterFromItemTypeMapper = wrapped;
            getFieldNameForItemTypeAndNameMapper = wrapped;
            getImplicitCollectionDefForFieldNameMapper = wrapped;
            getItemTypeForItemFieldNameMapper = wrapped;
            getLocalConverterMapper = wrapped;
            isIgnoredElementMapper = wrapped;
            isImmutableValueTypeMapper = wrapped;
            isReferenceableMapper = wrapped;
            realClassMapper = wrapped;
            realMemberMapper = wrapped;
            serializedClassMapper = wrapped;
            serializedMemberMapper = wrapped;
            shouldSerializeMemberMapper = wrapped;
        }

    } 
```

#### 2\. Mapper的映射的查找过程

比如，根据elementName查找对应的Class，首先调用realClass方法，然后realClass方法会在所有包装层中一层层往下找，并还原elementName的信息，比如在ClassAliasingMapper根据component别名得出Component类，最后在DefaultMapper中调用realClass创建出Class。  
CachingMapper——>SecurityMapper——>ArrayMapper———>ClassAliasingMapper——>PackageAliasingMapper———>DynamicProxyMapper———>DefaultMapper  
DefaultMapper的realClass方法

```
@Override
    public Class<?> realClass(final String elementName) {
            return Class.forName(elementName, initialize, classLoader);
    } 
```

比如对于一个elementNode为component，下图是调用的方法栈。

  

![](https://upload-images.jianshu.io/upload_images/89809-918ada361885ee8c.png)

image.png

### 4.2、XStream中Converter的原理

#### 1.Converter的注册

注册转化器并设置优先级，Null最高，基本类型第二高，常用bean普通优先级，反射低最低优先级，序列化对象和Externalizable低优先级，。

```
 registerConverter(new ReflectionConverter(mapper, reflectionProvider), PRIORITY_VERY_LOW);

   registerConverter(new SerializableConverter(mapper, reflectionProvider, classLoaderReference), PRIORITY_LOW);
   registerConverter(new ExternalizableConverter(mapper, classLoaderReference), PRIORITY_LOW);

   registerConverter(new NullConverter(), PRIORITY_VERY_HIGH);
   registerConverter(new IntConverter(), PRIORITY_NORMAL);
   registerConverter(new FloatConverter(), PRIORITY_NORMAL); 
```

使用二叉树TreeSet来保存Converter并根据优先级排序所有转换器，排好序的集合  
方便后面对converter的查找。

```
private final PrioritizedList<Converter> converters = new PrioritizedList<>();
@Override
    public void registerConverter(final Converter converter, final int priority) {
        typeToConverterMap.clear();
        converters.add(converter, priority);
    }

public class PrioritizedList<E> implements Iterable<E> {
    
    private final Set<PrioritizedItem<E>> set = new TreeSet<>();
} 
```

#### 2\. 查找Converter

```
public Converter lookupConverterForType(final Class<?> type) {
        
        final Converter cachedConverter = type != null ? typeToConverterMap.get(type.getName()) : null;
        if (cachedConverter != null) {
            
            return cachedConverter;
        }
        
        final Map<String, String> errors = new LinkedHashMap<>();
        
        for (final Converter converter : converters) {
            try {
                
                if (converter.canConvert(type)) {
                    if (type != null) {
                        
                        typeToConverterMap.put(type.getName(), converter);
                    }
                    
                    return converter;
                }
            } catch (final RuntimeException | LinkageError e) {
                errors.put(converter.getClass().getName(), e.getMessage());
            }
        }
} 
```

4.3、XML中HierarchicalStreamDriver的原理
-----------------------------------

HierarchicalStreamDriver通过下面方法创建Writer和Reader

```
HierarchicalStreamWriter createWriter(Writer out);
HierarchicalStreamReader createReader(Reader in); 
```

HierarchicalStreamDriver的实现类为AbstractDriver，使用XML DOM解析，把整个XML加载进内存，缺点是占内存。

```
private HierarchicalStreamReader createReader(final InputSource source) {
            documentBuilderFactory = createDocumentBuilderFactory();
            final DocumentBuilder documentBuilder = documentBuilderFactory.newDocumentBuilder();
            if (encoding != null) {
                source.setEncoding(encoding);
            }
} 
```

AbstractDriver实现类XppDriver，创建的是XmlPullParser 解析器XML的PULL解析，他是Xpp版本1（XStream默认使用该解析器）

```
public static synchronized XmlPullParser createDefaultParser() throws XmlPullParserException {
        if (factory == null) {
            factory = XmlPullParserFactory.newInstance();
        }
        return factory.newPullParser();
    } 
```

AbstractDriver实现类Xpp3Driver，创建的是Xpp3的XML解析器MXParser，他是Xpp3 解析器速度要快于Xpp，也是PULL解析XML。

```
@Override
    protected XmlPullParser createParser() {
        return new MXParser();
    } 
```

AbstractDriver实现类JsonHierarchicalStreamDriver创建的是Json解析器，当目前只支持把java对象序列化成JSON，不支持把JSON反序列化成java对象。

4.4、XML是如何转化成Java对象的
--------------------

### Xstream 调用fromXML

1.  把String转化成StringReader，HierarchicalStreamDriver通过StringReader创建HierarchicalStreamReader，最后调用MarshallingStrategy的unmarshal方法开始解组

```
fromXML(final String xml)
fromXML(new StringReader(xml)); 
unmarshal(hierarchicalStreamDriver.createReader(xml), root);
final T t = (T)marshallingStrategy.unmarshal(root, reader, dataHolder, converterLookup, mapper); 
```

2.  marshallingStrategy创建出TreeUnmarshaller来并启动解析

```
final TreeUnmarshaller context = createUnmarshallingContext(root, reader, converterLookup, mapper);

context.start(dataHolder); 
```

3.  开始组码—————>TreeUnmarshaller的start方法

```
public Object start(final DataHolder dataHolder) {
        this.dataHolder = dataHolder;
        
        final Class<?> type = HierarchicalStreams.readClassType(reader, mapper);
        
        final Object result = convertAnother(null, type);
        for (final Runnable runnable : validationList) {
            runnable.run();
        }
        return result;
    } 
```

4.  通过节点名获取Mapper中对应的Class

```
public static Class<?> readClassType(final HierarchicalStreamReader reader, final Mapper mapper) {
        if (classAttribute == null) {
        
        Class<?> type = mapper.realClass(reader.getNodeName());
        return type;
    } 
```

5.  根据Class把它转化成对应的java对象—————>TreeUnmarshaller的convertAnother方法

```
 public Object convertAnother(final Object parent, Class<?> type, Converter converter) {
        
        type = mapper.defaultImplementationOf(type);
        if (converter == null) {
            
            converter = converterLookup.lookupConverterForType(type);
        } else {
            if (!converter.canConvert(type)) {
                final ConversionException e = new ConversionException("Explicitly selected converter cannot handle type");
                e.add("item-type", type.getName());
                e.add("converter-type", converter.getClass().getName());
                throw e;
            }
        }
         
        return convert(parent, type, converter);
    } 
```

6.  如何查找对应的Converter———>ConverterLookup中的lookupConverterForType方法

```
public Converter lookupConverterForType(final Class<?> type) {
        
        final Converter cachedConverter = type != null ? typeToConverterMap.get(type.getName()) : null;
        if (cachedConverter != null) {
            return cachedConverter;
        }
        
        for (final Converter converter : converters) {
                if (converter.canConvert(type)) {
                    if (type != null) {
                        把找到的放入缓存集合中
                        typeToConverterMap.put(type.getName(), converter);
                    }
                    return converter;
                }
        }
    } 
```

7.  根据找到的Converter把Type转化成java对象————>TreeUnmarshaller的convert()

```
protected Object convert(final Object parent, final Class<?> type, final Converter converter) {
            
            return converter.unmarshal(reader, this);
} 
```

8.  组码的过程，如当Class对应的Converter为AbstractReflectionConverter时，  
    根据获取的对象，继续读取子节点，并转化成对象对应的变量。

```
public Object unmarshal(final HierarchicalStreamReader reader, final UnmarshallingContext context) {
        
        Object result = instantiateNewInstance(reader, context);
        
        result = doUnmarshal(result, reader, context);
        return serializationMembers.callReadResolve(result);
    }

protected Object instantiateNewInstance(final HierarchicalStreamReader reader, final UnmarshallingContext context) {
            
            return reflectionProvider.newInstance(context.getRequiredType());
    }

public Object doUnmarshal(final Object result, final HierarchicalStreamReader reader,
            final UnmarshallingContext context) {
    
     while (reader.hasMoreChildren()) {
            reader.moveDown();
            
            Object field = reflectionProvider.getFieldOrNull(fieldDeclaringClass, fieldName);
            
            final String classAttribute = HierarchicalStreams.readClassAttribute(reader, mapper);
            if (classAttribute != null) {
                 type = mapper.realClass(classAttribute);
            } else {
                 type = mapper.defaultImplementationOf(field.getType());
                      }
            
            value = unmarshallField(context, result, type, field);
            
            reflectionProvider.writeField(result, fieldName, value, field.getDeclaringClass());
  }
} 
```

9.  获取Class变量的值过程同5到8，是一个循环过程，直到读取到最后一个节点退出循环。最终获取到java对象中的变量值也都设置，整个XML解析过程就结束了。

```
protected Object unmarshallField(final UnmarshallingContext context, final Object result, final Class<?> type,
            final Field field) {
        
        return context.convertAnother(result, type, mapper.getLocalConverter(field.getDeclaringClass(), field
            .getName()));
    } 
```

4、如何提升XStream 解析速度
------------------

1.  减少Converter的查找时间  
    我们XML都是JavaBean对象的话，那我们registerConverter时设置为高优先级，这样在查找对应的转换器时就减少了查找的时间。
2.  减少Mapper和Converter的查找时间  
    持久化Mapper查找缓存集合和Converter的查找缓存集合，在初始化时加载这两个集合后再开始解析，。
3.  使用Xpp3Driver的解析器  
    XStream 默认使用的是XppDriver，解析速度要慢于第三版本的Xpp3Driver。原因：第三版本的Xpp3Driver使用了Xpp3 parser，具体原理本文不做分析。

5、自定义Mapper和自定义Converter
------------------------

[demo连接](https://github.com/jakingting/Xtream-jar/tree/master/demo)

6、总结
----

全文通过举XStream的使用例子作为文章开头部分，接着分析了XStream的总体设计，并把XStream分为五部分一一做了介绍。接着详细地介绍了Mapper的原理、Converter的原理、HierarchicalStreamDriver的原理，XML是如何转化成Java对象的，最后根据原理讲了如何提高XStream 解析XML速度。
