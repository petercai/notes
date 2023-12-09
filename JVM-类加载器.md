# JVM-类加载器
**本次目标**

1.  理解`JVM`是什么
2.  能够描述`JVM`架构图
3.  掌握`JVM`类加载器相关概念

`JVM`是`Java`虚拟机的简称，是`Java`语言的核心组件，负责充当`Java`代码和底层操作系统之间的中间层。`JVM`的主要任务是将`Java`字节码转换为特定平台的机器码并执行。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a645e3a428f4dbbb300b0aa3fb2b7f7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1278&h=1178&s=172919&e=png&b=181817)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fffdf6af3a54cdfb04053ce871da921~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=1177&s=178343&e=jpg&b=fceee9)

`JVM`采用了一种基于栈的架构，包括类加载器、运行时数据区、执行引擎和本地方法栈等组件。这些组件协同工作，实现了`Java`程序的运行和管理。

类加载器的概念
-------

类加载器`Class Loader`是Java运行时环境中的一个重要组件，负责加载`Java`类和资源文件。它的主要功能是**根据类的全限定名查找并加载对应的字节码，将其转换为可执行的**`Java`**类**。类加载器在`Java`虚拟机`JVM`中起着至关重要的作用，确保`Java`应用程序能够正确地加载和运行类。

类加载器的分类
-------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/edbd3d2dbab14d4aab6f6b39373efba1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=867&h=466&s=173566&e=png&b=fefdfd)

1.  **启动类加载器**`Bootstrap ClassLoader`：负责加载`Java`核心库（如`java.lang`包）中的类。它主要加载`JAVA_HOME\lib\rt.jar`目录下的类库，或者通过`-Xbootclasspath`参数指定的路径中的类库。启动类加载器是用原生代码实现的，位于Java虚拟机的最顶层，没有父亲加载器。
2.  **扩展类加载器**`Extensions ClassLoader`：负责加载`Java`扩展库（如`javax`包）中的类。扩展类加载器继承自启动类加载器，因此它的搜索路径包括`Java`核心库和所有已安装的扩展库。扩展类加载器加载`JAVA_HOME\lib\ext`目录下的类库，或者通过`java.ext.dirs`系统变量指定的路径中的类库。
3.  **应用程序类加载器**`Application ClassLoader`：负责加载`用户路径classpath上的类`。应用程序类加载器根据`Java`应用程序的类路径`classpath`加载类，搜索路径包括`Java`应用程序的类路径以及`Java`核心库和扩展库。
4.  **自定义类加载器**`User ClassLoader`：负责加载应用之外的类文件。

类加载器的特点
-------

### 双亲委派机制

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42e5111d127241e891e2f06c4e2b94a0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=955&h=635&s=42870&e=png&a=1&b=98cbfe)

#### 什么是双亲委派

当一个类加载器收到类加载任务，会先交给其父亲加载器去完成，因此最终加载任务都会传递到顶层的启动类加载器，只有当父亲加载器无法完成加载时，才会尝试执行加载任务

#### 为什么需要双亲委派

1.  **避免类的重复加载**：在Java虚拟机中，每个类加载器都有自己的命名空间，负责加载并维护一组类的定义。当一个类加载器收到加载类的请求时，它会先检查自己的命名空间中是否已经加载此类，如果已经加载，则直接返回已加载的类定义，避免了重复加载。如果没有加载，则将加载请求委派给父加载器，由父加载器继续尝试加载，这样一级一级向上委派，直到找到能够加载该类的加载器或者到达顶层的启动类加载器。这种机制确保了类只会被加载一次。避免了被重复加载的问题。
2.  **保证类的安全性**：由于父加载器会在委派给子加载器加载类之前尝试加载，意味着核心`Java`类库会由顶层的启动类加载器加载，而不会被其他加载器替换。可以防止恶意代码通过替换核心类库来执行一些危险操作，确保了`Java`类库的完整性和安全性。

#### 双亲委派机制源码

```scss
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            
            
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                    
                long t0 = System.nanoTime();
                try {
                        
                        
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    
                    
                    
                }

                if (c == null) {
                    
                    
                    
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

```

#### 为什么需要破坏双亲委派

在实际应用中，双亲委派机制解决了`Java`基础类统一加载的问题，但是存在缺陷。`JDK`基础类作为典型的`API`被用户调用，但是也存在**基础类调用用户代码**的情况：

比如在使用`JDBC`时， 利用`DriverManager.getConnection`获取连接时，`DriverManager`是由根类加载器`Bootstrap`加载的，在加载`DriverManager`时，会执行其静态方法，加载初始驱动程序，也就是`Driver`接口的实现类；但是这些实现类基本都是第三方厂商提供的，根据双亲委派原则，第三方的类不可能被根类加载器加载。

#### 如何破坏双亲委派

1.  继承`ClassLoader`抽象类，重写`loadClass`方法，在这个方法可以自定义要加载的类使用的类加载器
2.  使用线程上下文加载器，可以通过`java.lang.Thread`类的`setContextClassLoader()`方法来设置当前类使用的类加载器类型

##### JDBC打破双亲委派源码

1.  `DriverManager`类由引导类加载器加载：当调用`DriverManager.class.getClassLoader()`时，如果输出为`null`，表示`DriverManager`类是由引导类加载器加载的。这意味着`DriverManager`类的加载过程将按照引导类加载器的加载规则进行。
2.  加载和注册驱动程序：在调用`DriverManager.getConnection()`方法时，`DriverManager`会尝试加载所有可用的驱动程序并注册它们。这是通过服务提供者机制完成的，即通过查找和加载驱动程序实现类来实现的。
3.  加载驱动程序实现类：在加载驱动程序实现类时，因为引导类加载器主要负责加载`Java`运行时环境的核心类库，而非第三方提供的类。`DriverManager`使用线程上下文类加载器来加载驱动程序实现类。它会根据类路径下的配置文件（通常是`META-INF/services/java.sql.Driver`）中指定的驱动程序实现类来加载。这种机制允许第三方驱动程序提供者将自己的实现类配置在类路径中，并由`DriverManager`加载和注册。

```xml
 <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.27</version>
        </dependency>
    </dependencies>

```

```java
package com.sn.knit.classloader;

import java.sql.Connection;
import java.sql.Driver;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Enumeration;


 * @author trow
 * @desc 探索JDBC是如何打破双亲委派机制
 * @date 2023/12/04
 */
public class JdbcDemo {

    
     * 1. DriverManager类由引导类加载器加载：当调用DriverManager.class.getClassLoader()时，如果输出为null，表示DriverManager类是由引导类加载器加载的。
     * 这意味着DriverManager类的加载过程将按照引导类加载器的加载规则进行。
     * 2. 加载和注册驱动程序：在调用DriverManager.getConnection()方法时，DriverManager会尝试加载所有可用的驱动程序并注册它们。
     * 这是通过服务提供者机制完成的，即通过查找和加载驱动程序实现类来实现的。
     * 3. 加载驱动程序实现类：在加载驱动程序实现类时，因为引导类加载器主要负责加载Java运行时环境的核心类库，而非第三方提供的类。
     * DriverManager使用线程上下文类加载器来加载驱动程序实现类。它会根据类路径下的配置文件（通常是META-INF/services/java.sql.Driver）中指定的驱动程序实现类来加载。
     * 这种机制允许第三方驱动程序提供者将自己的实现类配置在类路径中，并由DriverManager加载和注册。
     */
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:mysql://localhost:3306/test?useSSL=false";

        Connection connection = DriverManager.getConnection(url, "root", "123456");
        
        System.out.println(DriverManager.class.getClassLoader());
        Enumeration<Driver> drivers = DriverManager.getDrivers();
        while (drivers.hasMoreElements()) {
            Driver driver = drivers.nextElement();
            System.out.println(driver.toString());
            System.out.println(driver.getClass().getClassLoader());
        }
        System.out.println(connection);
    }
}

```

```scss
        synchronized(DriverManager.class) {
            
            if (callerCL == null) {
                callerCL = Thread.currentThread().getContextClassLoader();
            }
        }

```

```sql
null
com.mysql.cj.jdbc.Driver@27bc2616
sun.misc.Launcher$AppClassLoader@18b4aac2
com.mysql.cj.jdbc.ConnectionImpl@3dd3bcd

Process finished with exit code 0

```

##### 自定义类加载器打破双亲委派源码

```java
package com.sn.knit.classloader;

import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;


 * 自定义类加载器打破双亲委派：重写loadClass方法
 *
 * @author trow
 * @date 2023/12/05
 */
public class BreakCustomClassLoader extends ClassLoader {

    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        
        ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
        
        System.out.println("Class loader: " + contextClassLoader.getClass());
        try {
            return contextClassLoader.loadClass(name);

        } catch (ClassNotFoundException e) {
            
            return findClass(name);
        }
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        
        byte[] classBytes = loadClassBytes(name);
        
        return defineClass(name, classBytes, 0, classBytes.length);
    }

    
    private byte[] loadClassBytes(String className) {
        
        

        
        String filePath = "E:\small\small-java\" + className.replace('.', '/') + ".class";

        try (InputStream inputStream = new FileInputStream(filePath)) {
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];
            int bytesRead;
            while ((bytesRead = inputStream.read(buffer)) != -1) {
                outputStream.write(buffer, 0, bytesRead);
            }
            return outputStream.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }

    public static void main(String[] args) {
        // 创建自定义类加载器实例
        BreakCustomClassLoader classLoader = new BreakCustomClassLoader();
        try {
            Class<?> loadedClass = classLoader.loadClass("com.sn.knit.classloader.User");
            System.out.println("Loaded class: " + loadedClass.getName());

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}


```

### 缓存机制

JVM中的类加载器通常会使用缓存来存储已加载的类信息，如果某个类已经被其父亲加载器加载过，那么子类加载器就不会再次加载该类，直接使用缓存中的类。

类加载器的工作原理
---------

### 类加载的时机

1.  **主动引用时加载**：当程序使用到某个类时，如果该类还未被加载、链接和初始化，则会触发类的加载过程。主动引用包括创建类的实例、访问类的静态变量、调用类的静态方法等操作。
2.  **被动引用时加载**：当访问某个类的静态常量时，只有声明该常量的类会被加载，而不会触发该类的初始化。被动引用的几种情况包括访问类的静态常量、通过数组定义类的引用类型、引用类的子类等。
3.  **反射调用时加载**：使用`Java`反射机制来动态创建类的实例、访问类的成员等操作时，会触发类的加载。

### 类加载的生命周期

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d359d9acd4e44e529ca3689835e29777~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=830&h=343&s=28474&e=jpg&b=f0e5db)

1.  **加载阶段**：加载阶段是类加载过程的第一步，它负责查找类的字节码，并将其加载到内存中。在加载阶段，会执行以下操作：
2.  1.  通过类的全限定名获取类的字节码文件。
    2.  将字节码文件加载到内存中，并在方法区创建一个代表该类的`Class`对象。
    3.  解析类的字节码，生成虚拟机可以使用的数据结构。
3.  **链接阶段**：链接阶段是类加载过程的第二步，它负责将已经加载的类与其他类和符号引用进行关联。链接阶段又可以细分为以下几个步骤：
4.  1.  验证：验证阶段确保加载的字节码符合`Java`虚拟机规范，包括检查字节码的结构、语义和安全性等。
    2.  准备：准备阶段为类的静态变量分配内存，并设置默认初始值。
    3.  解析：解析阶段将符号引用转换为直接引用，即将类、方法、字段等的符号引用解析为内存中的直接指针。
5.  **初始化阶段**：初始化阶段是类加载过程的最后一步，它负责执行类的初始化代码，包括静态变量的赋值和静态代码块的执行。初始化阶段会按照程序的执行顺序依次执行，且只会执行一次。

综上所述，类加载的过程可以归纳为加载、链接和初始化三个阶段。其中，**加载阶段将类的字节码加载到内存中，链接阶段将类与其他类和符号引用关联，初始化阶段执行类的初始化代码**。这三个阶段的顺序是依次进行的，加载阶段首先发生，然后是链接阶段，最后是初始化阶段。

自定义类加载器
-------

目标：自定义类加载器，加载指定路径在`D`盘下的`test`文件夹下的类

步骤：

1.  新建一个类`User.java`，编译User类成User.class并放在`classPath`以外的目录下,目录必须于User类的全限定类名的包保持一致;如果放在`classPath`目录下，则打印出`AppClassLoader`

```csharp
package com.sn.knit.jvm;

public class User {

    public void say() {
        System.out.println("Hello World");
    }
}

```

1.  创建了一个名为 `CustomClassLoader` 的自定义类加载器，并重写了其 `findClass` 方法来加载类文件。你的类加载器会在指定的 `classPath` 目录下查找类文件，并将其读取为字节数组，然后使用 `defineClass` 方法定义并返回对应的类。

```java
package com.sn.knit.classloader;

import java.io.*;
import java.lang.reflect.Method;

public class CustomClassLoader extends ClassLoader {
    private String classPath;

    public CustomClassLoader(String classPath) {
        this.classPath = classPath;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            byte[] data = loadClassData(name);
            return defineClass(name, data, 0, data.length);
        } catch (IOException e) {
            throw new ClassNotFoundException("Failed to load class " + name, e);
        }
    }

    private byte[] loadClassData(String name) throws IOException {
        String className = name.replace('.', File.separatorChar) + ".class";
        try (InputStream inputStream = new FileInputStream(classPath + className);
             ByteArrayOutputStream outputStream = new ByteArrayOutputStream()) {
            int bytesRead;
            byte[] buffer = new byte[4096];
            while ((bytesRead = inputStream.read(buffer)) != -1) {
                outputStream.write(buffer, 0, bytesRead);
            }
            return outputStream.toByteArray();
        }
    }

```

2.  在 `main` 方法中，使用了 `CustomClassLoader` 来加载名为 `com.sn.knit.jvm.User` 的类。根据代码中的包路径和类名，我们可以推断出该类应该位于 `com.sn.knit.jvm` 包下，输出`User`类的`sayHello`方法和`User`类的类加载器名称

```python
    public static void main(String[] args) throws Exception {
        String classPath = "D:\test\";
        CustomClassLoader classLoader = new CustomClassLoader(classPath);
        Class<?> clazz = classLoader.loadClass("com.sn.knit.jvm.User");
        Object obj = clazz.newInstance();
        Method method = clazz.getDeclaredMethod("sayHello", null);
        method.invoke(obj, null);
        System.out.println(clazz.getClassLoader().getClass().getName());
    }
}


```

1.  测试结果

```vbnet
Hello
com.sn.knit.classloader.CustomClassLoader

Process finished with exit code 0

```

类加载器在实际开发中的应用
-------------

1.  动态模块加载：许多应用需要动态地加载类或模块，通过自定义类加载器可以实现在运行时动态加载和卸载模块，实现插件式的架构。
2.  热部署：在某些场景下，需要在应用程序运行过程中更新程序代码而不需要停止整个应用程序。自定义类加载器可以用于实现热部署，动态加载新的类文件并且卸载旧的类文件，以实现更新代码而不中断服务的功能。
3.  Class文件加密保护：某些安全要求较高的应用场景需要保护Class文件的安全性，通过自定义类加载器可以对加载的Class文件进行解密和校验，以确保Class文件的安全加载。
4.  特定类的加载扩展：有些应用需要在启动时提前加载某些类，以提高启动速度，自定义类加载器可以在启动时提前加载需要的类，以加快应用程序的启动速度。
5.  扩展类加载机制：在一些特殊的应用场景下，可能需要定制化的类加载机制，通过自定义类加载器可以扩展类加载机制，实现更加灵活的加载策略。