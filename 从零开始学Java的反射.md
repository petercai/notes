# 从零开始学Java的反射
1\. 概念
------

反射(Reflection)是一种在Java程序运行时动态获取类信息，以及动态操作类的属性、方法和构造方法、注解等元素的技术。

通过这种技术，**我们可以更加深入地控制程序的运行过程，在程序运行时动态地获取类的成员变量、方法、构造方法、注解等所有信息，即使这些信息是私有的**。拿到这些信息之后，就可以帮助我们更好地了解类的结构和特性，更加灵活地应对不同的业务需求，进而实现一些高级的功能。总的来说，Java的反射机制主要具有以下功能：

> *   在运行时判断任意一个对象所属的类；
> *   在运行时构造任意一个类的对象；
> *   在运行时判断任意一个类所具有的成员变量和方法；
> *   在运行时调用任意一个对象的方法；
> *   生成动态代理。

看到这里，有些小伙伴会很好奇，难道被private修饰的元素也可以被操作吗？正常情况下，private修饰的元素，不是只能被该类自己进行内部访问吗？在正常情况下，确实是这样的！但反射却是一种“不正常”的技术，它能让我们直接操作任何类的私有属性，甚至可以在运行时构造出任意一个类的对象。你可以这么理解，有了反射，任何Java类都不再有隐私可言，你想怎么“蹂躏”这个类都可以。当然，前提条件是我们得先得到这个类的Class字节码才行！

2\. 编译期与运行时
-----------

另外看到这里，可能有不少小伙伴还是搞不清，运行时是啥意思。在这里，**壹哥**再跟大家强调一下运行时与编译期这两个概念。

> **编译期**就是编写好源代码，然后把源码交给编译器编译成计算机可执行文件的过程，也就是把Java代码编成class文件的过程。在Java中，编译期只是做了一些翻译功能，把代码当成文本进行操作，比如检查错误，并没有把代码放在内存中真正地运行起来。  
> **运行期**则是把编译后的class字节码文件交给计算机运行，直到程序运行结束。我们把在磁盘中的代码放到内存中执行起来就是运行期。

Java的反射机制就是在运行期工作的，我们可以在运行时获取到类的自身信息。正常情况下，Java源码已经被编译成了字节码，并被加载到了JVM虚拟机中，所以如果没有反射，我们是无法获取到一个正在运行的类信息的。

3\. 使用场景
--------

为了让大家更好地理解反射的作用，**壹哥**给大家举个例子，比如在框架开发中，许多框架都用到了反射来实现一些高级功能。

> *   **Spring的IoC容器**：通过反射，Spring能在运行时动态地创建和管理Java对象，实现松耦合和可配置化的特性。
> *   **Hibernate的ORM映射**：Mybatis使用反射来映射Java对象和数据库表，自动完成对象关系的持久化和查询。
> *   **MyBatis的Mapper接口**：MyBatis使用反射来动态生成Mapper接口的实现类，实现SQL语句的自动映射和执行。

除了框架开发，反射技术还被广泛地应用在其他方面，这些应用可以大大提高我们的开发效率和代码灵活性，尤其是在以下场景中：

> *   **框架开发**：Java中的许多框架都使用了反射来实现一些高级功能，例如Spring框架的IoC容器、Hibernate框架的ORM映射等。
> *   **动态代理**：通过反射技术，我们可以在运行时动态地创建一个实现某个接口的代理类，用于处理一些通用的逻辑。
> *   **注解处理器**：注解处理器通常是在编译期间扫描类的注解信息并生成相应的代码，而通过反射技术，我们就可以在运行时动态地获取类的注解信息，进而实现注解处理器的功能。
> *   **插件化开发**：插件化开发可以在运行时动态地加载外部类，并动态地创建对象和调用方法等，通过反射我们就可以实现插件化开发。

4\. 注意事项
--------

**壹哥**一直跟自己的学生强调，这个世界上没有十全十美的技术，反射虽然给我们提供了一种强大的动态编程方式，但也有一些需要我们注意的地方：

> *   **反射的性能较低**，因此在实际使用中应避免过度使用反射技术；
> *   **反射操作可能会破坏原有的封装性**，因此在使用时应格外谨慎，避免对程序的可维护性造成影响；
> *   **反射操作可能会导致代码更加复杂**，因此在使用反射时应尽量保持代码的简洁性，避免使用过于复杂的反射操作；
> *   **反射操作可能会影响程序的安全性**，因此在使用反射时应进行必要的安全检查，避免恶意代码通过反射入侵程序。

了解了以上这些理论内容之后，你可能已经迫不及待地要动手实操一把了，接下来我们就来看看反射该怎么使用吧。

1\. 常用API
---------

要想实现反射操作，我们需要用到以下几个常用的反射API：

| 类名 | 描述 |
| --- | --- |
| Class | 表示一个类的运行时信息，包括类的成员变量、方法、构造函数、注解等信息。 |
| Constructor | 表示一个类的构造函数信息。 |
| Field | 表示一个类的成员变量信息。 |
| Method | 表示一个类的方法信息。 |
| Modifier | 表示一个类的修饰符信息，包括public、private、protected、static等修饰符。 |
| Annotation | 表示注解类型的信息。 |

这些API给我们提供了丰富的反射功能，让我们编写出更加灵活和高效的代码，可以在运行时动态地获取到类的信息、调用类的方法、访问类的成员变量等。接下来**壹哥**就具体给大家讲一下这些API的具体用法。

2\. 获取Class字节码
--------------

Java反射类位于java.lang.reflect包中，其中的核心类是java.lang.Class，它表示一个类或接口的运行时信息。**要想进行反射，首先第一步就是要得到Class字节码对象，只有得到了Class对象，我们才可以进一步获取到类的成员变量、方法、构造函数、注解等信息，** 所以Class类是实现反射的关键。Class没有公开的构造方法，Class实例会由JVM在类加载时自动创建。

在Java中，目前主要有以下三种获取字节码的方式：

> *   **类名.class：**  该方式最为简单，直接在类名后面加上.class即可，该方式可以直接获取已知的类。
> *   **对象.getClass()：**  该方式比较灵活，可以在运行时获取已知对象的实际类型。
> *   **Class.forName("类的全路径")：**  该方式最为灵活，可以根据类名的字符串动态获取类，适用于需要根据外部条件动态加载类的场景。

### 2.1 **类名.class**

这是最简单的一种方式，我们只需要在类名的后面加上`.class`即可获取到Class对象，例如：

```java
Class<String> clazz = String.class;

```

### 2.2 对象.getClass()

如果我们已经创建出一个对象，那就可以通过该对象的getClass()方法获取到对应的Class对象，例如：

```java
String s = "hello";
Class<? extends String> clazz = s.getClass();

```

### 2.3 Class.forName()

最后一种方式是利用Class.forName()方法，通过类名来获取Class对象，例如：

```java
Class<?> clazz = Class.forName("java.lang.String");

```

### 2.4 Class相关方法

Class也是一个类，在该类中具有以下这些常用的方法，可以供我们得到其他的类型。

| 类型 | 访问方法 | 返回值类型 | 说明 |
| --- | --- | --- | --- |
| 包路径 | getPackage() | Package对象 | 获取该类的存放路径 |
| 类名称 | getName() | String对象 | 获取该类的名称 |
| 继承类 | getSuperclass() | Class对象 | 获取该类继承的父类 |
| 实现接口 | getlnterfaces() | Class型数组 | 获取该类实现的所有接口 |
| 构造方法 | getConstructors() | Constructor型数组 | 获取所有public的构造方法 |
| getDeclaredContruectors() | Constructor对象 | 获取当前对象的所有构造方法 |  |
| 方法 | getMethods() | Methods型数组 | 获取所有public的方法 |
| getDeclaredMethods() | Methods对象 | 获取当前对象的所有方法 |  |
| 成员变量 | getFields() | Field型数组 | 获取所有public的成员变量 |
| getDeclareFileds() | Field对象 | 获取当前对象的所有成员变量 |  |
| 内部类 | getClasses() | Class型数组 | 获取所有public的内部类 |
| getDeclaredClasses() | Class型数组 | 获取所有内部类 |  |
| 内部类的声明类 | getDeclaringClass() | Class对象 | 如果该类为内部类，则返回它的成员类，否则返回null |

### 2.5 小结

以上三种获取字节码的方式都有各自的适用场景，并有其优缺点，**壹哥**给大家总结如下：

| **方式** | **优点** | **缺点** |
| --- | --- | --- |
| **类名.class** | 简单易用，性能高 | 只能获取已知的类 |
| **对象.getClass()** | 可以获取到已知对象的类 | 必须先有对象才能获取 |
| **Class.forName()** | 可以根据类名动态获取到类对象 | 可能会抛出ClassNotFoundException异常 |

总的来说，以上这三种方式各有优缺点，至于选择哪种方式，要取决于具体的应用场景和需求。当我们获取到了类的字节码之后，就可以进行下一步开发了。

3\. 创建Person类
-------------

为了方便后面的测试，我们先创建一个Person类，代码如下：

```java

 * @author 一一哥Sun
 * @company 千锋教育
 */
public class Person {
    public String name;
    private int age;
    protected String address;

    public Person() {}

    public Person(String name, int age, String address) {
        this.name=name;
        this.age=age;
        this.address=address;
    }

    private Person(int age) {
        this.age=age;
    }

    protected Person(String name) {
        this.name=name;
    }

    public String getName() {
        return name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "Person [name=" + name + ", age=" + age + ", address=" + address + "]";
    }
}

```

4\. 获取成员变量
----------

### 4.1 Field相关方法

我们可以通过下列任意方法得到Field类型的对象或数组：

> *   **getFields()**
> *   **getField(String name)**
> *   **getDeclaredFields()**
> *   **getDeclaredField(String name)**

得到了Field类，我们再来看看Field类的一些常用方法，这些方法主要用于获取和操作类的成员变量。各方法的主要功能含义如下表：

| 方法 | 描述 |
| --- | --- |
| public Class<?> getType() | 返回该成员变量的类型 |
| public String getName() | 返回该成员变量的名称 |
| public int getModifiers() | 返回该成员变量的修饰符，以整数方式存储 |
| public void setAccessible(boolean flag) | 设置该成员变量的可访问标志 |
| public Object get(Object obj) | 返回指定对象上该成员变量的值 |
| public void set(Object obj, Object value) | 将指定对象上该成员变量设置为指定的新值 |
| public boolean isAccessible() | 判断该成员变量是否可访问 |
| public boolean isEnumConstant() | 判断该成员变量是否是枚举常量 |
| public boolean isSynthetic() | 判断该成员变量是否是合成成员变量 |

了解了这些方法的基本含义之后，我们再来看一个具体的实现案例。

### 4.2 代码实现

在获取到Class对象之后，我们可以通过Class对象的getField()方法或getDeclaredField()方法来获取类的成员变量。其中，**getField()方法只能获取道公开的成员变量，而getDeclaredField()方法可以获取到所有的成员变量，包括公有、私有或受保护的成员变量**。

```java
import java.lang.reflect.Field;


 * @author 一一哥Sun
 * @company 千锋教育
 */
public class Demo01 {
    public static void main(String[] args) {
        try {
            
            Person person=new Person();
            person.setName("一一哥");
            person.setAge(18);
            person.setAddress("千锋上海");
			
            
            Class<Person> clazz = Person.class;
            
            Field nameField = clazz.getField("name");
            
            Object nameValue = nameField.get(person);
            System.out.println("name = "+nameValue);
			
            
            Field ageField = clazz.getDeclaredField("age"); 
            
            
            ageField.setAccessible(true);
            Object ageValue = ageField.get(person);
            System.out.println("age = "+ageValue);
			
            
            Field addressField = clazz.getDeclaredField("address"); 
            Object addressValue = addressField.get(person);
            System.out.println("address = "+addressValue);
			
            
            Field[] fields = clazz.getFields();
            for(Field field: fields) {
                Object value = field.get(person);
                System.out.println("public value---"+value);
            }
			
            
            Field[] declaredFields = clazz.getDeclaredFields();
            for(Field field: declaredFields) {
                
                field.setAccessible(true);
                Object value = field.get(person);
                System.out.println("value---"+value);
            }
        } catch (NoSuchFieldException | SecurityException | IllegalArgumentException | IllegalAccessException e) {
            e.printStackTrace();
        } 
    }
}

```

在上面的代码中，我们用到了一个**setAccessible()方法，该方法是反射类java.lang.reflect.AccessibleObject中的一个方法，用于设置成员变量、方法、构造函数等的可访问性**。默认情况下，Java会限制对非公有的成员变量、方法、构造函数的访问，如果我们直接调用这些非公有的成员，会抛出IllegalAccessException异常。**我们可以通过调用setAccessible(true)方法来打破这种限制，使得非公有的成员也可以被访问**。所以在上面的代码中，壹哥在获取私有的age属性值之前，先setAccessible(true)，这样才能得到age的值，否则就会报错。

但需要注意的是，使用setAccessible(true)方法会打破Java的访问控制机制，可能会破坏代码的封装性，导致代码变得更加脆弱和不易维护。因此，我们要谨慎使用setAccessible()方法，尽量避免影响程序的可维护性。

5\. 属性的取值与赋值
------------

### 5.1 取值

如果我们想要获取Java对象的属性值，可以使用反射类java.lang.reflect.Field的get()方法。**使用get()方法需要两个参数：要获取属性值的对象和属性对象**。例如，假设我们有一个名为person的对象，以及一个名为age的int类型属性，我们可以使用如下代码来获取person对象的age属性值：

```java

 * @author 一一哥Sun
 * @company 千锋教育
 */
public class Person {
    private int age;
    
}

Person person = new Person();
person.age = 20;

Class<Person> personClass = Person.class;
Field ageField = personClass.getDeclaredField("age");

ageField.setAccessible(true); 

int age = (int) ageField.get(person); 

```

需要注意的是，我们在使用get()方法获取私有的属性值时，必须首先使用setAccessible(true)方法打破访问限制，否则会抛出IllegalAccessException异常。此外，使用get()方法获取属性值时，需要指定属性的类型，否则可能会发生类型转换错误。

### 5.2 赋值

而如果我们想要给Java对象的属性赋值，则可以使用反射类java.lang.reflect.Field的set()方法。**使用set()方法需要三个参数：要设置属性值的对象、属性对象和属性值**。例如，假设我们有一个名为person的对象，以及一个名为age的int类型属性，我们可以使用如下代码将person对象的age属性值设置为20：

```java

 * @author 一一哥Sun
 * @company 千锋教育
 */
public class Person {
    private int age;
    
}

Person person = new Person();

Class<Person> personClass = Person.class;
Field ageField = personClass.getDeclaredField("age");

ageField.setAccessible(true); 

ageField.set(person, 20); 

```

需要注意的是，我们在使用set()方法给私有属性设置属性值时，也必须首先使用setAccessible(true)方法来打破访问限制，否则也会抛出IllegalAccessException异常。此外，使用set()方法设置属性值时，还需要指定正确的属性类型，否则可能会发生类型转换错误。

6\. 获取方法
--------

### 6.1 Method相关方法

我们可以通过以下方法来得到一个Method类或Method数组：

> *   **getMethods()**
> *   **getMethods(String name,Class<?> …parameterTypes)**
> *   **getDeclaredMethods()**
> *   **getDeclaredMethods(String name,Class<?>...parameterTypes)**

得到Method类之后，我们再来看看它的常用方法有哪些。

| 方法 | 描述 |
| --- | --- |
| public Class<?> getReturnType() | 返回该方法的返回值类型 |
| public Class<?>\[\] getParameterTypes() | 返回该方法的参数类型 |
| public int getModifiers() | 返回该方法的修饰符，以整数方式存储 |
| public void setAccessible(boolean flag) | 设置该方法的可访问标志 |
| getExceptionTypes() | 以Class数组的形式，获得该方法可能抛出的异常类型 |
| isVarArgs() | 查看该方法是否允许带有可变数量的参数，如果允许则返回 true，否则返回false |
| public Object invoke(Object obj, Object... args) | 调用该方法 |
| public boolean isAccessible() | 判断该方法是否可访问 |
| public boolean isBridge() | 判断该方法是否是桥方法 |
| public boolean isDefault() | 判断该方法是否是默认方法 |
| public boolean isSynthetic() | 判断该方法是否是合成方法 |

### 6.2 代码实现

其实获取类的方法与获取成员变量类似，同样有getMethod()方法和getDeclaredMethod()方法两种方式。其中，getMethod()方法只能获取公有的方法，而getDeclaredMethod()方法则可以获取所有的方法，包括公开、私有和受保护的方法。

```java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;


 * @author 一一哥Sun
 * @company 千锋教育
 */
public class Demo02 {
    public static void main(String[] args) {
        try {
            
            Person person = new Person();
            person.setAge(18);
            person.setAddress("千锋上海");

            
            Class<Person> clazz = Person.class;
            
            
            
            Method getNameMethod = clazz.getMethod("setName", String.class);
            
            
            
            
            Object result = getNameMethod.invoke(person, "一一哥");
            System.out.println("name= " + person.getName());

            
            Method getAgeMethod = clazz.getDeclaredMethod("getAge");
            
            Object ageValue = getAgeMethod.invoke(person, null);
            System.out.println("age= " + ageValue);

            
            Method getAddressMethod = clazz.getDeclaredMethod("getAddress");
            Object addressValue = getAddressMethod.invoke(person, null);
            System.out.println("address= " + addressValue);
        } catch (SecurityException | IllegalArgumentException 
        | NoSuchMethodException | IllegalAccessException| InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}

```

在上面的案例中，我们是通过以下语句来获取类中的方法：

```java
Method method = clazz.getMethod("methodName", parameterTypes);

```

其中，methodName是方法名，parameterTypes是方法参数的类型。如果该方法没有参数，则可以省略。找到Method之后，我们可以用以下语句进行方法的调用：

```java
Object result = method.invoke(object, args);

```

其中，object是方法所属的对象，如果该方法是静态的，则可以传入null。args是方法的参数列表，如果该方法没有参数，则传入null。result是该方法的返回值，如果没有返回值，则result会为null。

7\. 获取构造方法
----------

### 7.1 Constructor常用方法

下表中是Constructor类的常用方法，主要用于获取和操作类的构造方法。

| 方法 | 描述 |
| --- | --- |
| public Class<?>\[\] getParameterTypes() | 返回构造方法的参数类型 |
| isVarArgs() | 查看该构造方法是否允许带可变数量的参数，如果允许，返回 true，否则返回false |
| getExceptionTypes() | 以 Class 数组的形式获取该构造方法可能抛出的异常类型 |
| public int getModifiers() | 返回构造方法的修饰符，以整数方式存储 |
| public void setAccessible(boolean flag) | 设置该构造方法的可访问标志 |
| public Object newInstance(Object... initargs) | 使用指定的初始化参数创建类的新实例 |
| public boolean isAccessible() | 判断该构造方法是否可访问 |
| public boolean isSynthetic() | 判断该构造方法是否是合成构造方法 |

### 7.2 代码实现

我们可以通过以下方法来得到一个Constructor对象或数组：

> *   **getConstructors()**
> *   **getConstructor(Class<?>…parameterTypes)**
> *   **getDeclaredConstructors()**
> *   **getDeclaredConstructor(Class<?>...parameterTypes)**

其中，getConstructor()方法只能获取公开的构造方法，而getDeclaredConstructor()方法则可以获取所有的构造方法，包括公开、私有和受保护的构造方法。

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;


 * @author 一一哥Sun
 * @company 千锋教育
 */
public class Demo03 {
    public static void main(String[] args) {
        try {
            
            Class<Person> clazz = Person.class;
            
            Constructor<Person> publicConstructor = clazz.getConstructor(String.class, int.class, String.class);
            
            Person newInstance = publicConstructor.newInstance("一一哥",18,"千锋上海");
            System.out.println("person="+newInstance.toString());
			
            
            Constructor<Person> privateConstructor = clazz.getDeclaredConstructor(int.class);
            
            privateConstructor.setAccessible(true);
            
            Person newInstance2 = privateConstructor.newInstance(18);
            System.out.println("person2="+newInstance2.toString());
			
            
            Constructor<Person> protectedConstructor = clazz.getDeclaredConstructor(String.class);
            Person newInstance3 = protectedConstructor.newInstance("壹哥");
            System.out.println("person3="+newInstance3.toString());
        } catch (SecurityException | IllegalArgumentException | NoSuchMethodException | InstantiationException | IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}

```

我们拿到了类的Class对象和构造方法之后，就可以在运行时动态地创建对象了，主要是使用Constructor对象的newInstance()方法来创建对象。对于私有的构造方法，同样需要设置setAccessible(true)。

8\. getMocMers()方法
------------------

**壹哥**在上面给大家介绍各种API时，多次提到过一个getMocMers()方法。通过该方法，我们可以得到一个 java.lang.reflect.Modifier类，进而可以可以解析出修饰符的信息。这些修饰符信息如下表所示：

| 静态方法名称 | 说明 |
| --- | --- |
| isStatic(int mod) | 如果使用static修饰符修饰，则返回 true，否则返回 false |
| isPublic(int mod) | 如果使用public修饰符修饰，则返回 true，否则返回 false |
| isProtected(int mod) | 如果使用protected修饰符修饰，则返回 true，否则返回 false |
| isPrivate(int mod) | 如果使用private修饰符修饰，则返回 true，否则返回 false |
| isFinal(int mod) | 如果使用final修饰符修饰，则返回 true，否则返回 false |
| toString(int mod) | 以字符串形式返回所有的修饰符 |

