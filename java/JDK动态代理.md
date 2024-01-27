# JDK动态代理

1\. 什么是 JDK 动态代理?
-----------------

JDK 动态代理是 Java 中一种实现代理模式的机制。它允许在运行时创建代理类和对象，用于替代原始对象进行方法调用，并可以在方法调用前后添加额外的逻辑。

2\. 为什么需要 JDK 动态代理?
-------------------

使用动态代理可以实现横切关注点（cross-cutting concerns）的分离，例如日志记录、性能监控、事务管理等。通过将这些通用的功能从业务逻辑中抽离出来，可以提高代码的可维护性和复用性。

另外，动态代理还可以用于解耦合，当我们需要对某个接口的多个实现类进行统一处理时，可以使用动态代理来实现，而不需要修改原有的代码。

3\. JDK 动态代理的实现原理?
------------------

JDK 动态代理基于 Java 的反射机制实现。当我们使用 JDK 动态代理时，首先需要定义一个接口，然后创建一个实现了 InvocationHandler 接口的代理类。在代理类中，我们可以指定要代理的目标对象以及在方法调用前后执行的逻辑。当客户端调用代理对象的方法时，代理对象会将方法调用转发给 InvocationHandler 的 invoke 方法，在该方法中可以根据需要执行相应的逻辑。

具体的实现步骤如下：

1.  定义一个接口，该接口是被代理对象和代理对象共同实现的。
2.  创建一个实现 InvocationHandler 接口的代理类，并在其中实现 invoke 方法。在 invoke 方法中，可以根据需要执行额外的逻辑，然后调用目标对象的方法。
3.  使用 Proxy 类的静态方法 newProxyInstance 创建代理对象。该方法接收三个参数：ClassLoader、要实现的接口数组以及 InvocationHandler 对象。
4.  通过代理对象调用方法。

4\. JDK 动态代理的使用示例
-----------------

```java

public interface UserService {
    void addUser(String username);
}


public class UserServiceImpl implements UserService {
    @Override
    public void addUser(String username) {
        System.out.println("添加用户：" + username);
    }
}


public class LogProxy implements InvocationHandler {
    private Object target;

    public LogProxy(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("开始记录日志");
        Object result = method.invoke(target, args);
        System.out.println("结束记录日志");
        return result;
    }
}


public class Main {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        LogProxy logProxy = new LogProxy(userService);
        UserService proxy = (UserService) Proxy.newProxyInstance(
                userService.getClass().getClassLoader(),
                userService.getClass().getInterfaces(),
                logProxy);

        proxy.addUser("Alice");
    }
}

```

输出结果：

```null
开始记录日志
添加用户：Alice
结束记录日志

```

5\. JDK 动态代理的优点
---------------

*   简化了代码结构，将通用逻辑与业务逻辑分离。
*   可以在运行时动态地创建代理对象，无需提前编写代理类。
*   支持对接口的代理，可以实现多个接口的代理。

6\. JDK 动态代理的缺点
---------------

*   只能代理接口，无法代理具体类。
*   被代理的目标对象必须实现至少一个接口。
*   动态代理的性能相对较低，因为涉及到反射操作。

7\. JDK 动态代理的使用注意事项
-------------------

*   接口方法中不能使用 final 修饰符，否则会抛出异常。
*   在 invoke 方法中需要判断是否调用了 Object 类的方法，避免死循环。
*   如果目标对象是通过 Spring 容器管理的，建议使用 Spring AOP 来实现代理，而不是手动使用 JDK 动态代理。

8\. 总结
------

JDK 动态代理是 Java 中一种实现代理模式的机制，它基于 Java 的反射机制实现。使用 JDK 动态代理可以实现横切关注点的分离和解耦合，提高代码的可维护性和复用性。然而，JDK 动态代理只能代理接口，被代理的目标对象必须实现至少一个接口，并且性能相对较低。在使用时需要注意一些细节，如不能使用 final 修饰符、避免死循环等。
