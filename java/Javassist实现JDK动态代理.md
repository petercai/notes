# Javassist实现JDK动态代理
我们就一起来看看JDK动态代理的基本原理，以及如何通过Javassist进行模拟实现。

示例
--

以下是一个基于JDK动态代理的hello world示例，在很多地方都可以看到类似的版本。

```java
public class DynamicProxyTest {

    interface IHello {
        void sayHello();
    }

    static class Hello implements IHello {
        @Override
        public void sayHello() {
            System.out.println("hello world");
        }
    }

    static class DynamicProxy implements InvocationHandler {

        Object originalObj;

        Object bind(Object originalObj) {
            this.originalObj = originalObj;
            return Proxy.newProxyInstance(originalObj.getClass().getClassLoader(), originalObj.getClass().getInterfaces(), this);
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        	System.out.println("pre method");
        	Object result = method.invoke(originalObj, args);
    		System.out.println("post method");
    		return result;
        }
    }

    public static void main(String[] args) {
    	System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
        IHello hello = (IHello) new DynamicProxy().bind(new Hello());
        hello.sayHello();
    }
}

```

生成代理类源码
-------

通过设置参数 sun.misc.ProxyGenerator.saveGeneratedFiles为true，在执行main函数之后，我们将得到一份`$Proxy0.class`文件，它就是Hello的代理类。

经过反编译，得到`$Proxy0`的源码（省略了无关内容）如下：

```java
final class $Proxy0 extends Proxy implements DynamicProxyTest.IHello {
	private static Method m1;
	private static Method m3;
	private static Method m2;
	private static Method m0;

	public $Proxy0(InvocationHandler paramInvocationHandler) {
		super(paramInvocationHandler);
	}

	public final void sayHello() {
		try {
			this.h.invoke(this, m3, null);
			return;
		} catch (Error | RuntimeException localError) {
			throw localError;
		} catch (Throwable localThrowable) {
			throw new UndeclaredThrowableException(localThrowable);
		}
	}

	static {
		try {
			m1 = Class.forName("java.lang.Object").getMethod("equals",
					new Class[] { Class.forName("java.lang.Object") });
			m3 = Class.forName("DynamicProxyTest$IHello").getMethod("sayHello", new Class[0]);
			m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
			m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
		} catch (NoSuchMethodException localNoSuchMethodException) {
			throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
		} catch (ClassNotFoundException localClassNotFoundException) {
			throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
		}
	}
}

```

**主要实现原理**

*   动态生成一个代理类，实现IHello接口；
*   代理类继承Proxy类，提供一个实例变量InvocationHandler h用于保存代理逻辑（此外，Proxy还提供了相关代理类处理逻辑）；
*   代理类声明一系列Method类变量，用于保存接口相关的反射方法；
*   代理类实现相关接口方法，核心逻辑是调用InvocationHandler的invoke方法，并传入3个参数：当前代理类对象、接口反射方法对象、实际方法参数。

前面简单分析了JDK动态代理的基本原理，其中，最核心的逻辑在于如何生成动态代理类，也就是 java.lang.reflect.Proxy.newProxyInstance(ClassLoader loader, Class<?>\[\] interfaces, InvocationHandler h)方法的实现。

接下来我们将通过Javassist一步步实现newProxyInstance方法。

1\. 定义接口
--------

接口基本与Proxy.newProxyInstance相同。为简单说明，我们这里只定义了一个接口类型参数Class<?>而不是数组。

```java
public static Object newProxyInstance(ClassLoader loader, Class<?> interfaceClass, InvocationHandler h) {
    ...
}

```

2\. 创建动态代理类
-----------

Javassist可以通过简单的Java API来操作源代码，这样就可以在不了解Java字节码相关知识的情况下，动态生成类或修改类的行为。

创建名称为NewProxyClass的代理类。

```java
ClassPool pool = ClassPool.getDefault();
CtClass proxyCc = pool.makeClass("NewProxyClass");

```

3\. 添加实例变量InvocationHandler
---------------------------

添加类型为InvocationHandler的实例变量h。

```java
CtClass handlerCc = pool.get(InvocationHandler.class.getName());

CtField handlerField = new CtField(handlerCc, "h", proxyCc);
handlerField.setModifiers(AccessFlag.PRIVATE);
proxyCc.addField(handlerField);

```

4\. 添加构造函数
----------

创建构造函数，参数类型为InvocationHandler。

```java

CtConstructor ctConstructor = new CtConstructor(new CtClass[] { handlerCc }, proxyCc);
ctConstructor.setBody("$0.h = $1;");
proxyCc.addConstructor(ctConstructor);

```

其中，`$0`代表this, `$1`代表构造函数的第1个参数。

5\. 实现IHello接口声明
----------------

```java

CtClass interfaceCc = pool.get(interfaceClass.getName());
proxyCc.addInterface(interfaceCc);

```

6\. 实现IHello相关接口方法
------------------

### 6.1 遍历接口方法

```bash
CtMethod[] ctMethods = interfaceCc.getDeclaredMethods();
for (int i = 0; i < ctMethods.length; i++) {
	// 核心逻辑在下方
}

```

### 6.2 代理方法实现

由于代理类调用invoke方法需要传入接口的反射方法对象（Method），因此，我们需要为每个方法添加一个可复用的Method类变量。

#### 6.2.1 反射方法对象声明及初始化

```java

String classParamsStr = "new Class[0]";
if (ctMethods[i].getParameterTypes().length > 0) {
	for (CtClass clazz : ctMethods[i].getParameterTypes()) {
		classParamsStr = ((classParamsStr == "new Class[0]") ? clazz.getName() : classParamsStr + "," + clazz.getName()) + ".class";
	}
	classParamsStr = "new Class[] {" + classParamsStr + "}";
}

String methodFieldTpl = "private static java.lang.reflect.Method %s=Class.forName(\"%s\").getDeclaredMethod(\"%s\", %s);";

String methodFieldBody = String.format(methodFieldTpl, "m" + i, interfaceClass.getName(), ctMethods[i].getName(), classParamsStr);

CtField methodField = CtField.make(methodFieldBody, proxyCc);
proxyCc.addField(methodField);

```

通过以上逻辑，将生成类似代码如下：

```java
private static Method m0 = Class.forName("chapter9.javassistproxy3.IHello").getDeclaredMethod("sayHello", new Class[0]);
private static Method m1 = Class.forName("chapter9.javassistproxy3.IHello").getDeclaredMethod("sayHello2", new Class[] { Integer.TYPE });
private static Method m2 = Class.forName("chapter9.javassistproxy3.IHello").getDeclaredMethod("sayHello3", new Class[] { String.class, Boolean.TYPE, Object.class });

```

#### 6.2.2 接口方法体实现

```java

String methodBody = "$0.h.invoke($0, " + methodFieldName + ", $args)";


if (CtPrimitiveType.voidType != ctMethods[i].getReturnType()) {
	
	
	if (ctMethods[i].getReturnType() instanceof CtPrimitiveType) {
		CtPrimitiveType ctPrimitiveType = (CtPrimitiveType) ctMethods[i].getReturnType();
		methodBody = "return ((" + ctPrimitiveType.getWrapperName() + ") " + methodBody + ")." + ctPrimitiveType.getGetMethodName() + "()";
	}
	
	else {
		methodBody = "return (" + ctMethods[i].getReturnType().getName() + ") " + methodBody;
	}
}
methodBody += ";";


CtMethod newMethod = new CtMethod(ctMethods[i].getReturnType(), ctMethods[i].getName(),
		ctMethods[i].getParameterTypes(), proxyCc);
newMethod.setBody(methodBody);
proxyCc.addMethod(newMethod);

```

通过以上逻辑，将生成类似代码如下：

```java
public void sayHello() {
    this.h.invoke(this, m0, new Object[0]);
}

public String sayHello2(int paramInt) {
    return (String)this.h.invoke(this, m1, new Object[] { new Integer(paramInt) });
}

public int sayHello3(String paramString, boolean paramBoolean, Object paramObject) {
    return ((Integer)this.h.invoke(this, m2, new Object[] { paramString, new Boolean(paramBoolean), paramObject })).intValue();
}

```

7\. 生成代理类字节码
------------

以下语句，将生成代理类字节码：D:/tmp/NewProxyClass.class

```java
proxyCc.writeFile("D:/tmp"); 

```

8\. 生成代理对象
----------

最后，通过调用第3步创建的构造函数，传入InvocationHandler对象，生成并返回代理类。

```java
Object proxy = proxyCc.toClass().getConstructor(InvocationHandler.class).newInstance(h);
return proxy;

```

```java
public class ProxyFactory {
	
	public static Object newProxyInstance(ClassLoader loader, Class<?> interfaceClass, InvocationHandler h) throws Throwable {
		ClassPool pool = ClassPool.getDefault();
		
		
		CtClass proxyCc = pool.makeClass("NewProxyClass");

		
		CtClass handlerCc = pool.get(InvocationHandler.class.getName());
		CtField handlerField = new CtField(handlerCc, "h", proxyCc); 
		handlerField.setModifiers(AccessFlag.PRIVATE);
		proxyCc.addField(handlerField);

		
		CtConstructor ctConstructor = new CtConstructor(new CtClass[] { handlerCc }, proxyCc);
		ctConstructor.setBody("$0.h = $1;"); 
		proxyCc.addConstructor(ctConstructor);
		
		
		CtClass interfaceCc = pool.get(interfaceClass.getName());
		
		
		proxyCc.addInterface(interfaceCc);
		
		
		CtMethod[] ctMethods = interfaceCc.getDeclaredMethods();
		for (int i = 0; i < ctMethods.length; i++) {
			String methodFieldName = "m" + i; 

			
			
			
			String classParamsStr = "new Class[0]"; 
			if (ctMethods[i].getParameterTypes().length > 0) { 
				for (CtClass clazz : ctMethods[i].getParameterTypes()) {
					classParamsStr = (("new Class[0]".equals(classParamsStr)) ? clazz.getName() : classParamsStr + "," + clazz.getName()) + ".class";
				}
				classParamsStr = "new Class[] {" + classParamsStr + "}";
			}
			String methodFieldTpl = "private static java.lang.reflect.Method %s=Class.forName(\"%s\").getDeclaredMethod(\"%s\", %s);";
			String methodFieldBody = String.format(methodFieldTpl, "m" + i, interfaceClass.getName(), ctMethods[i].getName(), classParamsStr);
			
			CtField methodField = CtField.make(methodFieldBody, proxyCc);
			proxyCc.addField(methodField);
			
			System.out.println("methodFieldBody: " + methodFieldBody);
			
			
			
			String methodBody = "$0.h.invoke($0, " + methodFieldName + ", $args)";
			
			if (CtPrimitiveType.voidType != ctMethods[i].getReturnType()) {
				
				
				if (ctMethods[i].getReturnType() instanceof CtPrimitiveType) {
					CtPrimitiveType ctPrimitiveType = (CtPrimitiveType) ctMethods[i].getReturnType();
					methodBody = "return ((" + ctPrimitiveType.getWrapperName() + ") " + methodBody + ")." + ctPrimitiveType.getGetMethodName() + "()";
				} else { 
					methodBody = "return (" + ctMethods[i].getReturnType().getName() + ") " + methodBody;
				}
			}
			methodBody += ";";
			
			CtMethod newMethod = new CtMethod(ctMethods[i].getReturnType(), ctMethods[i].getName(),
					ctMethods[i].getParameterTypes(), proxyCc);
			newMethod.setBody(methodBody);
			proxyCc.addMethod(newMethod);
			
			System.out.println("Invoke method: " + methodBody);
		}
		
		proxyCc.writeFile("D:/tmp");
		
		
		@SuppressWarnings("unchecked")
		Object proxy = proxyCc.toClass().getConstructor(InvocationHandler.class).newInstance(h);

		return proxy;
	}

}

public interface IHello {
	int sayHello3(String a, boolean b, Object c);
}

public class Hello implements IHello {
	@Override
	public int sayHello3(String a, boolean b, Object c) {
		String abc = a + b + c;
		System.out.println("a + b + c=" + abc);
		return abc.hashCode();
	}
}

public class CustomHandler implements InvocationHandler {

	private Object obj;

	public CustomHandler(Object obj) {
		this.obj = obj;
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("pre method");
		Object result = method.invoke(obj, args);
		System.out.println("post method");
		return result;
	}

}

public class ProxyTest {

	public static void main(String[] args) throws Throwable {
		IHello hello = new Hello();
		CustomHandler customHandler = new CustomHandler(hello);
		IHello helloProxy = (IHello) ProxyFactory.newProxyInstance(hello.getClass().getClassLoader(), IHello.class, customHandler);
		System.out.println();
		System.out.println("a+false+Object=" + helloProxy.sayHello3("a", false, new Object()));
	}
	
}

```

**执行结果**：

> methodFieldBody: private static java.lang.reflect.Method m0=Class.forName("chapter9.javassistproxy3.IHello").getDeclaredMethod("sayHello3", new Class\[\] {java.lang.String.class,boolean.class,java.lang.Object.class});  
> Invoke method: return ((java.lang.Integer) $0.h.invoke(![](https://juejin.cn/equation?tex=0%2C%20m0%2C)
> args)).intValue();
> 
> pre method  
> a + b + c=afalsejava.lang.Object@504bae78  
> post method  
> a+false+Object=-903110407

* * *

**参考**

Javassist官网：[www.javassist.org/](https://link.juejin.cn/?target=http%3A%2F%2Fwww.javassist.org%2F "http://www.javassist.org/")
