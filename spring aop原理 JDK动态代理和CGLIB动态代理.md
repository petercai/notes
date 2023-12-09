# spring aop原理 JDK动态代理和CGLIB动态代理
> Spring的两大特性是IOC和AOP IOC负责将对象动态的注入到容器，从而达到一种需要谁就注入谁，什么时候需要就什么时候注入的效果。理解spring的ioc也很重要。 但是今天主要来和大家讲讲aop。 AOP 广泛应用于处理一些具有横切性质的系统级服务，**AOP 的出现是对 OOP 的良好补充**，用于处理系统中分布于各个模块的横切关注点，比如事务管理、日志、缓存等等。

AOP实现的关键在于AOP框架自动创建的AOP代理。
--------------------------

AOP代理主要分为静态代理和动态代理，

*   静态代理的代表为AspectJ；
*   动态代理则以Spring AOP为代表

#### 1，AspectJ

AspectJ是静态代理的增强，采用编译时生成 AOP 代理类，因此也称为编译时增强，具有更好的性能。 缺点：但需要使用特定的编译器进行处理

### 2，Spring AOP

Spring AOP使用的动态代理，运行时生成 AOP 代理类，所谓的动态代理就是说AOP框架不会去修改字节码，而是在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。 缺点：由于 Spring AOP 需要在每次运行时生成 AOP 代理，因此性能略差一些。

由于aspectj的使用还需要使用特定的编译器进行处理，处理起来有点麻烦。今天重要来讲解Spring AOP

Spring AOP动态代理主要有两种方式，JDK动态代理和CGLIB动态代理。

*   JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。JDK动态代理的核心是InvocationHandler接口和Proxy类。
*   如果目标类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类（通过修改字节码来实现代理）。 注意，CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。 jdk和cglib动态代理来共同实现我们的aop面向切面的功能。

下面就来用简单的代码来演示下jdk和cglib动态代理的实现原理。

一，jdk动态代理实现AOP拦截
----------------

*   1，为target目标类定义一个接口JdkInterface，这是jdk动态代理实现的前提

```bash
/**
 * Created by qcl on 2018/11/29
 * desc: jdk动态aop代理需要实现的接口
 */
public interface JdkInterface {
    public void add();
}

```

*   2,用我们要代理的目标类JdkClass实现上面我们定义的接口，我们的实验目标就是在不改变JdkClass目标类的前提下，在目标类的add方法的前后实现拦截，加入自定义切面逻辑。这就是aop的魅力所在：代码与代码之间没有耦合。

```bash
/**
 * Created by qcl on 2018/11/29
 * desc: 被代理的类，即目标类target
 */
public class JdkClass implements JdkInterface {
    @Override
    public void add() {
        System.out.println("目标类的add方法");
    }
}

```

-3 ，到了关键的一步，用我们的MyInvocationHandler，实现InvocationHandler接口，并且实现接口中的invoke方法。仔细看invoke方法，就是在该方法中加入切面逻辑的。目标类方法的执行是由mehod.invoke(target,args)这条语句完成。

```bash
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * Created by qcl on 2018/11/29
 * desc:这里加入切面逻辑
 */
public class MyInvocationHandler implements InvocationHandler {
    private Object target;

    public MyInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before-------切面加入逻辑");
        Object invoke = method.invoke(target, args);//通过反射执行，目标类的方法
        System.out.println("after-------切面加入逻辑");
        return invoke;
    }
}

```

*   4，测试结果

```bash
/**
 * Created by qcl on 2018/11/29
 * desc:测试
 */
public class JdkTest {
    public static void main(String[] args) {
        JdkClass jdkClass = new JdkClass();
        MyInvocationHandler handler = new MyInvocationHandler(jdkClass);
        // Proxy为InvocationHandler实现类动态创建一个符合某一接口的代理实例
        //这里的proxyInstance就是我们目标类的增强代理类
        JdkInterface proxyInstance = (JdkInterface) Proxy.newProxyInstance(jdkClass.getClass().getClassLoader(),
                jdkClass.getClass()
                        .getInterfaces(), handler);
        proxyInstance.add();
        //打印增强过的类类型
        System.out.println("=============" + proxyInstance.getClass());

    }
}

```

执行上面测试类可以得到如下结果

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/29/1675f89c2e1d19d5~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

可以看到，目标类的add方法前后已经加入了自定义的切面逻辑，AOP拦截机制生效了。再看class com.sun.proxy.$Proxy0。这里进一步证明\_\_JDK动态代理的核心是InvocationHandler接口和Proxy类\_\_

二，cglib动态代理实现AOP拦截
------------------

*   1，定义一个要被代理的Base目标类（cglib不需要定义接口）

```bash
/**
 * Created by qcl on 2018/11/29
 * desc:要被代理的类
 */
public class Base {
    public void add(){
        System.out.println("目标类的add方法");
    }
}

```

*   2，定义CglibProxy类，实现MethodInterceptor接口，实现intercept方法。该代理的目的也是在add方法前后加入了自定义的切面逻辑，目标类add方法执行语句为proxy.invokeSuper(object, args)

```bash
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

/**
 * Created by qcl on 2018/11/29
 * desc:这里加入切面逻辑
 */
public class CglibProxy implements MethodInterceptor {
    @Override
    public Object intercept(Object object, Method method, Object[] args, MethodProxy methodProxy)
            throws Throwable {
        System.out.println("before-------切面加入逻辑");
        methodProxy.invokeSuper(object, args);
        System.out.println("after-------切面加入逻辑");
        return null;
    }
}

```

*   3，测试类

```bash
/**
 * Created by qcl on 2018/11/29
 * desc:测试类
 */
public class CglibTest {
    public static void main(String[] args) {
        CglibProxy proxy = new CglibProxy();
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Base.class);
        //回调方法的参数为代理类对象CglibProxy,最后增强目标类调用的是代理类对象CglibProxy中的intercept方法
        enhancer.setCallback(proxy);
        //此刻，base不是单车的目标类，而是增强过的目标类
        Base base = (Base) enhancer.create();
        base.add();

        Class<? extends Base> baseClass = base.getClass();
        //查看增强过的类的父类是不是未增强的Base类
        System.out.println("增强过的类的父类："+baseClass.getSuperclass().getName());
        System.out.println("============打印增强过的类的所有方法==============");
        FanSheUtils.printMethods(baseClass);


        //没有被增强过的base类
        Base base2 = new Base();
        System.out.println("未增强过的类的父类："+base2.getClass().getSuperclass().getName());
        System.out.println("=============打印增未强过的目标类的方法===============");
        FanSheUtils.printMethods(base2.getClass());//打印没有增强过的类的所有方法

    }
}

```

下面是打印结果

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/29/1675f89c2e0318be~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

通过打印结果可以看到

*   cglib动态的拦截切入成功了
*   cglib动态代理的方式是在运行时动态的生成目标类（Base）的子类,并且在目标类现有方法的基础上添加了很多cglib特有的方法。 下面贴出用反射打印类所有方法的工具类

```bash
public class FanSheUtils {

    //打印该类的所有方法
    public static void printMethods(Class cl) {
        System.out.println();
        //获得包含该类所有其他方法的数组
        Method[] methods = cl.getDeclaredMethods();
        //遍历数组
        for (Method method : methods) {
            System.out.print("  ");
            //获得该方法的修饰符并打印
            String modifiers = Modifier.toString(method.getModifiers());
            if (modifiers.length() > 0) {
                System.out.print(modifiers + " ");
            }
            //打印方法名
            System.out.print(method.getName() + "(");

            //获得该方法包含所有参数类型的Class对象的数组
            Class[] paramTypes = method.getParameterTypes();
            //遍历数组
            for (int i = 0; i < paramTypes.length; i++) {
                if (i > 0) {
                    System.out.print(",");
                }
                System.out.print(paramTypes[i].getName());
            }
            System.out.println(");");
        }
    }
}

```

注意：上面用到了cglib.jar和asm.jar。在我们的maven的pom.xml里引入下面类库即可

```bash
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

```
