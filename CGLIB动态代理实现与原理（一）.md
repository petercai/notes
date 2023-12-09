# CGLIB动态代理实现与原理（一）
一、实现原理
------

*   利用ASM开源包，将真实对象类的class文件加载进来，通过修改字节码生成其子类，覆盖父类相应的方法。
    
    备注：ASM是直接操作字节码的框架。
    

二、实现方式
------

*   1）定义拦截处理器。实现MethodInterceptor接口，覆写intercept方法用于拦截处理。
*   2）生成动态代理类。修改被代理对象的class文件字节码生成子类。

```bash
    <dependency>
      <groupId>cglib</groupId>
      <artifactId>cglib-nodep</artifactId>
      <version>3.3.0</version>
      <scope>compile</scope>
    </dependency>

```

```bash
package com.java24k.example.target;

/**
 * @Description: 真实主题即被代理对象
 * @Author zhoufeng
 * @Date 2020/7/13 2:34
 * @version: V1.0
 */
public class TargetClass {

    public void targetInfo(){
        System.out.println("打印目标类信息");
     }
}


```

```bash
package com.java24k.example.interceptor;

import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * @Description: 代理拦截处理器
 * @Author zhoufeng
 * @Date 2020/7/13 2:30
 * @version: V1.0
 */
public class MyInterceptor implements MethodInterceptor {

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("------插入前置通知代码-------------");
        // 此处一定要使用proxy的invokeSuper方法来调用目标类的方法
        methodProxy.invokeSuper(o, objects);
        System.out.println("------插入后置处理代码-------------");
        return null;
    }
}

```

```bash
package com.java24k.example.test;

import com.java24k.example.interceptor.MyInterceptor;
import com.java24k.example.target.TargetClass;
import net.sf.cglib.proxy.Enhancer;

/**
 * @Description: 生成cglib代理测试类
 * @Author zhoufeng
 * @Date 2020/7/13 2:41
 * @version: V1.0
 */
public class CglibProxyTest {

    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        // 设置目标类
        enhancer.setSuperclass(TargetClass.class);
        // 设置拦截器对象
        enhancer.setCallback(new MyInterceptor());
        // 创建子类 即代理
        TargetClass targetClassProxy = (TargetClass) enhancer.create();
        targetClassProxy.targetInfo();
    }
}

```

```bash
------插入前置通知代码-------------
打印目标类信息
------插入后置处理代码-------------

Process finished with exit code 0

```

三、FastClass机制
-------------

在JDK动态代理中，调用目标对象的方法使用的是反射，而在CGLIB动态代理中使用的是FastClass机制。

*   FastClass使用：动态生成一个继承FastClass的类，并向类中写入委托对象，直接调用委托对象的方法。
*   FastClass逻辑：在继承FastClass的动态类中，根据方法签名(方法名字+方法参数)得到方法索引，根据方法索引调用目标对象方法。
*   FastClass优点：FastClass用于代替Java反射，避免了反射造成调用慢的问题。

第二节和第三节我们将剖析CGLIB生成动态代理源码和FastClass源码。