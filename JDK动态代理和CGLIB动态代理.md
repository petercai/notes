# JDK动态代理和CGLIB动态代理
一、什么是代理模式
---------

代理模式（Proxy Pattern）给某一个对象提供一个代理，并由代理对象控制原对象的引用。代理对象在客户端和目标对象之间起到中介作用。

代理模式是常用的结构型设计模式之一，当直接访问某些对象存在问题时可以通过一个代理对象来间接访问，为了保证客户端使用的透明性，所访问的真实对象与代理对象需要实现相同的接口。代理模式属于结构型设计模式，属于GOF23种设计模式之一。

代理模式可以分为静态代理和动态代理两种类型，而动态代理中又分为JDK动态代理和CGLIB代理两种。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6f7b217a480407a8f3b3b07543a9cb0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

**代理模式包含如下角色:**

1.  **Subject (抽象主题角色)** 抽象主题角色声明了真实主题和代理主题的共同接口,这样一来在任何使用真实主题 的地方都可以使用代理主题。客户端需要针对抽象主题角色进行编程。
2.  **Proxy (代理主题角色)** 代理主题角色内部包含对真实主题的引用，从而可以在任何时候操作真实主题对象。 在代理主题角色中提供一个与真实主题角色相同的接口，以便在任何时候都可以替代真实 主体。代理主题角色还可以控制对真实主题的使用，负责在需要的时候创建和删除真实主 题对象,并对真实主题对象的使用加以约束。代理角色通常在客户端调用所引用的真实主 题操作之前或之后还需要执行其他操作，而不仅仅是单纯的调用真实主题对象中的操作。
3.  **RealSubject (真实主题 角色)** 真实主题角色定义了代理角色所代表的真实对象，在真实主题角色中实现了真实的业 务操作,客户端可以通过代理主题角色间接调用真实主题角色中定义的方法。

### 代理模式的优点

*   代理模式能将代理对象与真实被调用的目标对象分离。
    
*   一定程度上降低了系统的耦合度，扩展性好。
    
*   可以起到保护目标对象的作用。
    
*   可以对目标对象的功能增强。
    

### 代理模式的缺点

*   代理模式会造成系统设计中类的数量增加。
    
*   在客户端和目标对象增加一个代理对象，会造成请求处理速度变慢。
    

二、JDK动态代理
---------

在java的动态代理机制中，有两个重要的类或接口，一个是 `InvocationHandler`(Interface)、另一个则是 `Proxy`(Class)，这一个类和接口是实现我们动态代理所必须用到的。

### InvocationHandler

每一个动态代理类都必须要实现InvocationHandler这个接口，并且每个代理类的实例都关联了一个handler，当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。

InvocationHandler这个接口的唯一一个方法 invoke 方法：

```java
Object invoke(Object proxy, Method method, Object[] args) throws Throwable

```

这个方法一共接受三个参数，那么这三个参数分别代表如下：

*   **proxy**:指代JDK动态生成的最终代理对象
*   **method**:指代的是我们所要调用真实对象的某个方法的Method对象
*   **args**:指代的是调用真实对象某个方法时接受的参数

### Proxy

Proxy这个类的作用就是用来动态创建一个代理对象的类，它提供了许多的方法，但是我们用的最多的就是newProxyInstance 这个方法：

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,  InvocationHandler handler)  throws IllegalArgumentException

```

这个方法的作用就是得到一个动态的代理对象，其接收三个参数，我们来看看这三个参数所代表的含义：

*   **loader**：ClassLoader对象，定义了由哪个ClassLoader来对生成的代理对象进行加载，即代理类的类加载器。
*   **interfaces**：Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口给它，那么这个代理对象就宣称实现了该接口(多态)，这样我就能调用这组接口中的方法了。
*   **Handler**：InvocationHandler对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler对象上。

所以我们所说的DynamicProxy（动态代理类）是这样一种class：它是在运行时生成的class，在生成它时你必须提供一组interface给它，然后该class就宣称它实现了这些 interface。这个DynamicProxy其实就是一个Proxy，它不会做实质性的工作，在生成它的实例时你必须提供一个handler，由它接管实际的工作。

#### JDK动态代理实例

**创建接口类**

```java
public interface HelloInterface {
    void sayHello();
}

```

**创建被代理类，实现接口**

```java

 * 被代理类
 */
public class HelloImpl implements HelloInterface{
    @Override
    public void sayHello() {
        System.out.println("hello");
    }
}

```

**创建InvocationHandler实现类**

```java

 * 每次生成动态代理类对象时都需要指定一个实现了InvocationHandler接口的调用处理器对象
 */
public class ProxyHandler implements InvocationHandler{
    private Object subject; 
    public ProxyHandler(Object subject) {
        this.subject = subject;
    }
    
     *当代理对象调用真实对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用
     */
    @Override
    public Object invoke(Object obj, Method method, Object[] objs)
            throws Throwable {
        Object result = null;
        System.out.println("可以在调用实际方法前做一些事情");
        System.out.println("当前调用的方法是" + method.getName());
        result = method.invoke(subject, objs);
        System.out.println(method.getName() + "方法的返回值是" + result);
        System.out.println("可以在调用实际方法后做一些事情");
        System.out.println("------------------------");
        return result;
    }
}

```

**测试**

```java
public class Mytest {
    public static void main(String[] args) {
        
        HelloImpl hello = new HelloImpl();
        
        ProxyHandler handler = new ProxyHandler(hello);
        
        HelloInterface helloProxy = (HelloInterface) Proxy.newProxyInstance(
                HelloInterface.class.getClassLoader(), 
                new Class[]{HelloInterface.class}, handler);
        
        helloProxy.sayHello();
    }
}

```

**结果**

```markdown
可以在调用实际方法前做一些事情
当前调用的方法是sayHello
hello
sayHello方法的返回值是null
可以在调用实际方法后做一些事情
------------------------

```

### JDK动态代理步骤

JDK动态代理分为以下几步：

1.  拿到被代理对象的引用，并且通过反射获取到它的所有的接口。
2.  通过JDK Proxy类重新生成一个新的类，同时新的类要实现被代理类所实现的所有的接口。
3.  动态生成 Java 代码，把新加的业务逻辑方法由一定的逻辑代码去调用。
4.  编译新生成的 Java 代码.class。
5.  将新生成的Class文件重新加载到 JVM 中运行。

所以说JDK动态代理的核心是通过重写被代理对象所实现的接口中的方法来重新生成代理类来实现的，那么假如被代理对象没有实现接口呢？那么这时候就需要CGLIB动态代理了。

三、CGLIB动态代理
-----------

JDK动态代理是通过重写被代理对象实现的接口中的方法来实现，而CGLIB是通过继承被代理对象来实现，和JDK动态代理需要实现指定接口一样，CGLIB也要求代理对象必须要实现`MethodInterceptor`接口，并重写其唯一的方法`intercept`。

CGLib采用了非常底层的字节码技术，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。（利用ASM开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理）

**注意**：因为CGLIB是通过继承目标类来重写其方法来实现的，故而如果是final和private方法则无法被重写，也就是无法被代理。

```xml
<dependency>
    <groupId>cglib</groupId>
	<artifactId>cglib-nodep</artifactId>
	<version>2.2</version>
</dependency>

```

### CGLib核心类

**1、 net.sf.cglib.proxy.Enhancer**：主要增强类，通过字节码技术动态创建委托类的子类实例；

`Enhancer`可能是CGLIB中最常用的一个类，和Java1.3动态代理中引入的Proxy类差不多。和Proxy不同的是，Enhancer既能够代理普通的class，也能够代理接口。Enhancer创建一个被代理对象的子类并且拦截所有的方法调用（包括从Object中继承的toString和hashCode方法）。Enhancer不能够拦截final方法，例如Object.getClass()方法，这是由于Java final方法语义决定的。基于同样的道理，Enhancer也不能对fianl类进行代理操作。这也是Hibernate为什么不能持久化final class的原因。

**2、net.sf.cglib.proxy.MethodInterceptor**：常用的方法拦截器接口，需要实现intercept方法，实现具体拦截处理；

```java
 public java.lang.Object intercept(java.lang.Object obj,
                                   java.lang.reflect.Method method,
                                   java.lang.Object[] args,
                                   MethodProxy proxy) throws java.lang.Throwable{}

```

*   **obj**：动态生成的代理对象
*   **method**:实际调用的方法
*   **args**：调用方法入参
*   **net.sf.cglib.proxy.MethodProxy**：java Method类的代理类，可以实现委托类对象的方法的调用；常用方法：methodProxy.invokeSuper(proxy, args)；在拦截方法内可以调用多次。

#### CGLib代理实例

**创建被代理类**

```java
public class SayHello {
    public void say(){
        System.out.println("hello");
    }
}

```

**创建代理类**

```java

 *代理类 
 */
public class ProxyCglib implements MethodInterceptor{
     private Enhancer enhancer = new Enhancer();  
     public Object getProxy(Class clazz){  
          
          enhancer.setSuperclass(clazz);  
          enhancer.setCallback(this);  
          
          return enhancer.create();  
     }  

     
     public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {  
          System.out.println("可以在调用实际方法前做一些事情");  
          
          Object result = proxy.invokeSuper(obj, args);  
          System.out.println("可以在调用实际方法后做一些事情");  
          return result;  
     } 
}

```

**测试**

```java
public class Mytest {

    public static void main(String[] args) {
          ProxyCglib proxy = new ProxyCglib();  
          
          SayHello proxyImp = (SayHello)proxy.getProxy(SayHello.class);  
          proxyImp.say();  
    }
}

```

**结果**

```null
可以在调用实际方法前做一些事情
hello
可以在调用实际方法后做一些事情

```

### CGLIB动态代理实现分析

CGLib动态代理采用了FastClass机制，其分别为代理类和被代理类各生成一个FastClass，这个FastClass类会为代理类或被代理类的方法分配一个 index(int类型)。这个index当做一个入参，FastClass 就可以直接定位要调用的方法直接进行调用，这样省去了反射调用，所以调用效率比 JDK 动态代理通过反射调用更高。

但是我们看上面的源码也可以明显看到，JDK动态代理只生成一个文件，而CGLIB生成了三个文件，所以生成代理对象的过程会更复杂。

四、JDK和CGLib动态代理对比
-----------------

JDK 动态代理是实现了被代理对象所实现的接口，CGLib是继承了被代理对象。 JDK和CGLib 都是在运行期生成字节码,JDK是直接写Class字节码，CGLib 使用 ASM 框架写Class字节码，Cglib代理实现更复杂，生成代理类的效率比JDK代理低。

JDK 调用代理方法，是通过反射机制调用,CGLib 是通过FastClass机制直接调用方法,CGLib 执行效率更高。

### 原理区别：

java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。核心是实现InvocationHandler接口，使用invoke()方法进行面向切面的处理，调用相应的通知。

而cglib动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。核心是实现MethodInterceptor接口，使用intercept()方法进行面向切面的处理，调用相应的通知。

1、如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP

2、如果目标对象实现了接口，可以强制使用CGLIB实现AOP

3、如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

### 性能区别：

1、CGLib底层采用ASM字节码生成框架，使用字节码技术生成代理类，在jdk6之前比使用Java反射效率要高。唯一需要注意的是，CGLib不能对声明为final的方法进行代理，因为CGLib原理是动态生成被代理类的子类。

2、在jdk6、jdk7、jdk8逐步对JDK动态代理优化之后，在调用次数较少的情况下，JDK代理效率高于CGLIB代理效率，只有当进行大量调用的时候，jdk6和jdk7比CGLIB代理效率低一点，但是到jdk8的时候，jdk代理效率高于CGLIB代理。

### 各自局限：

1、JDK的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现JDK的动态代理。

2、cglib是针对类来实现代理的，他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对final修饰的类进行代理。

| 类型 | 机制 | 回调方式 | 适用场景 | 效率 |
| --- | --- | --- | --- | --- |
| JDK动态代理 | 委托机制，代理类和目标类都实现了同样的接口，InvocationHandler持有目标类，代理类委托InvocationHandler去调用目标类的原始方法 | 反射 | 目标类是接口类 | 效率瓶颈在反射调用稍慢 |
| CGLIB动态代理 | 继承机制，代理类继承了目标类并重写了目标方法，通过回调函数MethodInterceptor调用父类方法执行原始逻辑 | 通过FastClass方法索引调用 | 非接口类、非final类，非final方法 | 第一次调用因为要生成多个Class对象，比JDK方式慢。多次调用因为有方法索引比反射快，如果方法过多，switch case过多其效率还需测试 |

五、静态代理和动态的本质区别
--------------

*   静态代理只能通过手动完成代理操作，如果被代理类增加新的方法，代理类需要同步新增，违背开闭原则。
*   动态代理采用在运行时动态生成代码的方式，取消了对被代理类的扩展限制，遵循开闭原则。
*   若动态代理要对目标类的增强逻辑扩展，结合策略模式，只需要新增策略类便可完成，无需修改代理类的代码。