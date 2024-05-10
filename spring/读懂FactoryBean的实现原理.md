# 读懂FactoryBean的实现原理

       不同于普通Java Bean，FactoryBean是一种特殊的Bean类型，它的存在并非为了提供业务逻辑，而在于它具备动态创建和返回其他Bean实例的能力。

       这一特性使Spring容器在管理Bean时拥有了更高的灵活性和智能化水平。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c7832b8d043437dbb438d7629e8a296~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=500&h=262&s=50776&e=png&b=eeeeee)

       FactoryBean接口的核心方法主要包括以下几个：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7664dfa29194f559264a776cb7bb709~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=550&h=204&s=46453&e=png&b=f9f3ef)

*   `T getObject() throws Exception:` 这个方法是FactoryBean的核心实现，负责创建并返回所需的对象实例。具体实现时，开发者可以根据业务需求编写逻辑来构造Bean对象。例如，如果需要创建一个复杂对象，或者需要基于某些条件动态地生成不同的Bean，那么在这个方法中就可以实现这些复杂的创建逻辑。
    
*   `Class<T> getObjectType():` 返回由FactoryBean创建的Bean的类型。Spring容器利用此方法来确定Bean的类型，以便正确处理依赖关系和类型转换。
    
*   `boolean isSingleton():` 指定由FactoryBean创建的Bean是否为单例。若返回true，则Spring容器会在整个应用程序上下文中仅创建一次Bean实例；若返回false，则每次请求都会创建一个新的Bean实例。
    

       当Spring容器解析配置并遇到FactoryBean定义时，它并不直接实例化FactoryBean作为最终的Bean去使用。取而代之的是，容器会调用FactoryBean的getObject()方法，以此来获取真正需要的Bean实例。

       以下是FactoryBean发挥作用的具体步骤：

1.  `初始化FactoryBean实例：`Spring容器负责创建并初始化FactoryBean对象。
    
2.  `执行Bean创建逻辑：`容器随后调用该FactoryBean的getObject()方法，按照预定义的规则或用户自定义逻辑生产目标Bean。
    
3.  `遵守Bean生命周期：`由FactoryBean生成的Bean同样遵循Spring Bean的标准生命周期，包括但不限于依赖注入、初始化回调等过程。
    
4.  `交付目标Bean：`最终，客户端从Spring容器获取的实际上是FactoryBean通过getObject()方法生产的Bean，而非FactoryBean自身。
    

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b87d50de381d40d7b9c066f77f865726~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=809&h=383&s=47897&e=png&b=ffffff)

       下面是一个简单的Java代码示例，展示了如何实现一个自定义的FactoryBean来创建一个`Book`对象：

```Java
    import org.springframework.beans.factory.FactoryBean;
    import org.springframework.stereotype.Component;

    
    public class Book {
        private String name;
        private double price;

        

        
        @Override
        public String toString() {
            return "Book{" +
                    "name='" + name + '\'' +
                    ", price=" + price +
                    '}';
        }
    }

    
    @Component
    public class BookFactoryBean implements FactoryBean<Book> {

        @Override
        public Book getObject() throws Exception {
            
            Book book = new Book();
            book.setName("Spring in Action");
            book.setPrice(49.99);
            return book;
        }

        @Override
        public Class<?> getObjectType() {
            return Book.class;
        }

        @Override
        public boolean isSingleton() {
            
            return true;
        }
    }

    
    
    
    @Autowired
    private Book bookFromFactoryBean;

```

       在上述例子中，`BookFactoryBean`实现了`FactoryBean<Book>`接口，并在`getObject()`方法中创建并返回了一个`Book`实例。当我们在Spring容器中通过Bean名称获取`Book`时，实际上得到的是由`BookFactoryBean`创建的`Book`对象。如果需要获取`BookFactoryBean`自身，需要在Bean名称前加上"&"符号。

       面试中被问到关于FactoryBean的问题，实质上面试官是在考察候选人对Spring框架核心原理的理解深度。深入学习和理解FactoryBean有助于我们在日常开发中更好地运用Spring框架，也有利于我们进一步领会控制反转（IoC）和依赖注入（DI）的设计理念。

\*\*MORE | 更多精彩文章\*\*

*   [面试官：说说单点登录都是怎么实现的？](https://juejin.cn/post/7358732381490298919 "https://juejin.cn/post/7358732381490298919")
*   [JWT重放漏洞如何攻防？你的系统安全吗？](https://juejin.cn/post/7356485240804245558 "https://juejin.cn/post/7356485240804245558")
*   [JWT vs Session：到底哪个才是你的菜？](https://juejin.cn/post/7355433339547238419 "https://juejin.cn/post/7355433339547238419")
*   [JWT：你真的了解它吗？](https://juejin.cn/post/7354308608044072996 "https://juejin.cn/post/7354308608044072996")
