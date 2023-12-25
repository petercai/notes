# Java SPI机制总结系列之开发入门实例

Java SPI简单的介绍。

SPI，是Service Provider Interface的缩写，即服务提供者接口，单从字面上看比较抽象，你可以理解成，该机制就像Spring容器一样，通过IOC将对象的创建交给了Spring容器处理，若需要获取某个类的对象，就从Spring容器里取出使用即可。同理，在SPI机制当中，提供了一个类似Spring容器的角色，叫【服务提供者】，在代码运行过程中，若要使用到实现了某个接口的服务实现类对象，只需要将对应的接口类型交给服务提供者。服务提供者将会动态加载实现了该接口的所有服务实现类对象。  
服务提供者的角色用下图来表示。  
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c8256f711554f78a2c7e64589d8cbf2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=940&h=780&s=116284&e=png&b=fcfcfc)

举一个例子来说明。

假如，假如Maven项目里有这样一个interface接口，接口全名“com.zhu.service.UserService”——

```csharp
package com.zhu.service;

public interface UserService {
    void getName();
}

```

创建一个“com.zhu.service.impl.AUserServiceImpl”实现类——

```typescript
public class AUserServiceImpl implements UserService {
    @Override
    public void getName() {
        System.out.println("这是A用户姓名");
    }
}

```

接着在resource资源里，创建一个META-INF.services目录，在该目录里，创建一个文件名与接口com.zhu.service.UserService一致的文件——  
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34cd15393206483ca903543fb5730464~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=465&h=62&s=9894&e=png&b=46494b)

该com.zhu.service.UserService文件里写下com.zhu.service.impl.UserServiceImpl类名字——  
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6504e46b2ebe461db6794cc74fad3067~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=385&h=104&s=9873&e=png&b=353535)

这时候，就可以基于Java SPI动态加载到接口的实现类并执行了，我们写一个简单的测试类做验证——

```ini
public class Test {
    public static void main(String[] args) {
    ServiceLoader<UserService> serviceLoader = ServiceLoader.load(UserService.class)
    Iterator<UserService> serviceIterator = serviceLoader.iterator()
    while (serviceIterator.hasNext()) {
         UserService service = serviceIterator.next()
         service.getName()
    }
}
    }
}

```

执行该代码，ServiceLoader会加载到META-INF.services目录下的配置文件，找到对应接口全名文件，读取文件里的类名，再通过反射就可以进行实现类的实例化。既然能找到实现类的对象，那么不就可以基于父类引用指向子类对象，进而调用到实现类的getName()方法。该方法里执行打印语句 System.out.println("打印用户姓名")，打印结果如下，说明基于接口UserService，在程序动态加载并执行UserService接口实现。  
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c6f52f3d43e4c5ea3f5b065c10b061c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=671&h=125&s=14979&e=png&b=353535)

Java SPI的机制玩法，就如上文这一整个过程的实现。

该机制存在一个缺陷，假如该接口对应的文件存在多份实现类，那么，它都会一起执行了。

我们增加多一个实现类BUserServiceImpl——

```typescript
public class BUserServiceImpl implements UserService {
    @Override
    public void getName() {
        System.out.println("这是B用户姓名");
    }
}

```

然后，在resource资源里的META-INF.services目录接口对应com.zhu.service.UserService文件里，将BUserServiceImpl实现类的全名增加到文件里——  
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2d3246eb19d4ebaa3bf1b847fd17b40~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=431&h=116&s=13029&e=png&b=333333)

其他原有的代码无需改动，直接执行Test的main方法，打印结果如下，可以看到，新增的BUserServiceImpl实现类的getName()也被运行了。  
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb7af20be87741e39554cbc252675f01~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=668&h=166&s=17603&e=png&b=343434)

这就说明，Java SPI机制会将文件里配置的所有实现类都动态加载运行，稍微思考了一下，不难发现，若当中某个实现类的getName()出现异常，那么后面还没有执行到的其他实现类就会终止了。

SPI机制的优点很明显，当我们需要基于已有接口新增一个实现类功能时，只需要新增一个实现类代码，无需在原有代码逻辑上做改动，就可以实现新增类的功能逻辑了。

