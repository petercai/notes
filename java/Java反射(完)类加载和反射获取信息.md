# Java反射(完)类加载和反射获取信息
三.类加载
-----

### 1.动态加载和静态加载

*   基本说明
    
*   反射机制是 java 实现动态语言的关键，也就是通过反射实现类动态加载 1.静态加载：编译时加载相关的类，如果没有则报错，依赖性太强 2.动态加载：运行时加载需要的类，如果运行时不用该类,即使不存在该类，则不报错，降低了依赖性 3.举例说明
    
*   类加载时机
    
*   1.当创建对象时(new)）2.当子类被加载时 3.调用类中的静态成员时 4.通过反射 Class.forName("com.test.Cat");
    

### 2.类加载流程图

![](_assets/06f8e4579003b020818eb95d7a4ff0b1.png)

![](_assets/b5255c35ce723543dd72570e12a77cd3.png)

*   类加载各阶段完成任务
    
*   ![](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/00325/image-20221121170256381.png)
    

### 3.类加载的五个阶段

#### 3.1 加载阶段

*   JVM 在该阶段的主要目的是将字节码从不同的数据源（可能是 class 文件、也可能是 jar 包，甚至网络)转化为**二进制字节流加载到内存中**，并生成一个代表该类的 java.lang.Class 对象
    

#### 3.2 连接阶段

##### 3.2.1 验证

1.目的是为了确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。2.包括：文件格式验证（是否以魔数 oxcafebabe 开头）、元数据验证、字节码验证和符号引用验证 3.可以考虑使用-Xverify:none 参数来关闭大部分的类验证措施，缩短虚拟机类加载的时间。

##### 3.2.2 准备

JVM 会在该阶段对静态变量，分配内存并初始化（对应数据类型的默认初始值如 0、0L、null、false 等)。这些变量所使用的内存都将在方法区中进行分配

```
* @author LeeZhi
 * @version 1.0
 * 我们说明一个类加载的连接阶段-准备
 */
public class ClassLoad02 {
    public static void main(String[] args) {

    }
}
class A{
    
    
    
    
    
    public int n1 =10;
    public static int n2 = 20;
    public static final int n3 = 30;
}
```

##### 3.2.3 解析

虚拟机将常量池内的符号引用替换为直接引用的过程。

#### 3.3 初始化

*   Initialization(初始化)
    
*   1.到初始化阶段，才真正开始执行类中定义的 Java 程序代码，此阶段是执行<clinit>()方法的过程。2.<clinit>()方法是由编译器按语句在源文件中出现的顺序，依次自动收集类中的所有**静态变量**的赋值动作和静态代码块中的语句，并进行合并。3.虚拟机会保证一个类的<clinit>()方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的<clinit>()方法，其他线程都需要阻塞等待，直到活动饯程执行<clinit>()方法完毕\[**debug 源码**\]
    

```
* @author LeeZhi
   * @version 1.0
   * 演示类加载-初始化阶段
   */
  public class ClassLoad03 {
      public static void main(String[] args) {
          
          
          
          
          
              clinit(){
                  System.out.println("B静态代码块被执行")；
                  //num=300:
                  num=100;
              }
              合并:num = 100
           */
          
          
          System.out.println(B.num);
      }
  }
  class B{
      static {
          System.out.println("B 静态代码块被执行");
          num=300;
      }
      static int num = 100;
      public B(){
          System.out.println("B () 构造器被执行");
      }
  }
```

### 四.反射获取类的结构信息

#### 1.第一组：java.lang.Class 类

> 1.getName:获取全类名 2.getSimpleName:获取简单类名 3.getFields:获取所有 publicf 修饰的属性，包含本类以及父类的 4.getDeclaredFields:获取本类中所有属性 5.getMethods:获取所有 public 修饰的方法，包含本类以及父类的 6.getDeclaredMethods:获取本类中所有方法 7.getConstructors:获取所有 public 修饰的构造器，包含本类 8.getDeclaredConstructors:获取本类中所有构造器 9.getPackage:以 Package\]形式返回包信息 10.getSuperClass:以 Class 形式返回父类信息 11.getInterfaces:以 Class\[\]形式返回接口信息 12.getAnnotations:以 Annotation\[\]形式返回注解信息

```
* @author LeeZhi
 * @version 1.0
 * 演示如何通过反射获取类的结构信息
 */
public class ReflectionUtils {
    public static void main(String[] args) {

    }
    
    @Test
    public void api_01() throws ClassNotFoundException {
        
         * 1.getName:获取全类名
         * 2.getSimpleName:获取简单类名
         * 3.getFields:获取所有publicf修饰的属性，包含本类以及父类的
         * 4.getDeclaredFields:获取本类中所有属性
         * 5.getMethods:获取所有public修饰的方法，包含本类以及父类的
         * 6.getDeclaredMethods:获取本类中所有方法
         * 7.getConstructors:获取本类所有public修饰的构造器
         * 8.getDeclaredConstructors:获取本类中所有构造器
         * 9.getPackage:以Package]形式返回包信息
         * 10.getSuperClass:以Class形式返回父类信息
         * 11.getInterfaces:以Class[]形式返回接口信息
         * 12.getAnnotations:以Annotation[]形式返回注解信息
         */
        
        Class<?> personCls = Class.forName("com.gbx.reflection.Person");
        System.out.println(personCls.getName());
        System.out.println(personCls.getSimpleName());
        Field[] fields = personCls.getFields();
        for (Field field : fields) {
            System.out.println("本类以及父类的属性:" + field.getName());
        }
        Field[] declaredFields = personCls.getDeclaredFields();
        for (Field declaredField : declaredFields) {
            System.out.println("本类所有属性:"+declaredField.getName());
        }
        Method[] methods = personCls.getMethods();
        for (Method method : methods) {
            System.out.println("本类及父类的方法:" + method.getName());
        }
        Method[] declaredMethods = personCls.getDeclaredMethods();
        for (Method declaredMethod : declaredMethods) {
            System.out.println("本类所有方法:" + declaredMethod.getName());
        }
        Constructor<?>[] constructors = personCls.getConstructors();
        for (Constructor<?> constructor : constructors) {
            System.out.println("本类的构造器:" + constructor.getName());
        }
        Constructor<?>[] declaredConstructors = personCls.getDeclaredConstructors();
        for (Constructor<?> declaredConstructor : declaredConstructors) {
            System.out.println("本类中所有构造器:" + declaredConstructor.getName());
        }
        System.out.println(personCls.getPackage());
        Class<?> superclass = personCls.getSuperclass();
        System.out.println("父类的class对象"+superclass);
        Class<?>[] interfaces = personCls.getInterfaces();
        for (Class<?> anInterface : interfaces) {
            System.out.println("接口信息:" + anInterface);
        }
        Annotation[] annotations = personCls.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println("注解信息:" + annotation);
        }
    }
}
class A{
    public String hobby;
    public void hi(){
    }

    public A() {
    }
}
interface IA{

}
interface IB{

}
class Person extends A implements IA,IB{
    
    public String name;
    protected int age;
    String job;
    private double sal;

    public Person() {
    }

    public Person(String name) {
        this.name = name;
    }

    private Person(String name,int age){

    }
    
    public void m1(){

    }
    protected void m2(){

    }
    void m3(){

    }
    private void m4(){

    }
}
```

#### 2.第二组：java.lang.reflect.Field 类

> 1.getModifiers:以 int 形式返回修饰符\[说明：默认修饰符是 0，public 是 1，private 是 2，protected 是 4，static 是 8，final 是 16\] public(1)+static (8)=92.getType:以 Class 形式返回类型 3.getName:返回属性名

```
@Test
    public void api_02() throws ClassNotFoundException {
        
        Class<?> personCls = Class.forName("com.gbx.reflection.Person");
        Field[] declaredFields = personCls.getDeclaredFields();
        for (Field declaredField : declaredFields) {
            System.out.println("本类所有属性:"+declaredField.getName()
                    +"该属性的修饰值:"+declaredField.getModifiers()
                    +"该属性的类型"+declaredField.getType());
        }
    }
```

#### 3.第三组：java.lang.reflect.Method 类

> 1.getModifiers:以 int 形式返回修饰符\[说明：默认修饰符是 0，public 是 1，private 是 2，protected 是 4，static 是 8，final 是 16\]2.getReturnType:以 Classj 形式获取返回类型 3.getName:返回方法名 4.getParameterTypes:以 Class\[\]返回参数类型数组

```
Method[] declaredMethods = personCls.getDeclaredMethods();
        for (Method declaredMethod : declaredMethods) {
            System.out.println("本类所有方法:" + declaredMethod.getName()
            +"该方法的访问修饰符"+declaredMethod.getModifiers()
            +"该方法返回类型"+declaredMethod.getReturnType());
            
            
            Class<?>[] parameterTypes = declaredMethod.getParameterTypes();
            for (Class<?> parameterType : parameterTypes) {
                System.out.println("该方法形参类型:" + parameterType);
            }
        }
```

#### 4.第四组：java.lang.reflect.Constructor 类

> 1.getModifiers:以 int 形式返回修饰符 2.getName:返回构造器名（全类名）3.getParameterTypes:以 Class\[\]返回参数类型数组

```
Constructor<?>[] declaredConstructors = personCls.getDeclaredConstructors();
        for (Constructor<?> declaredConstructor : declaredConstructors) {
            System.out.println("本类中所有构造器:" + declaredConstructor.getName());
            Class<?>[] parameterTypes = declaredConstructor.getParameterTypes();
            for (Class<?> parameterType : parameterTypes) {
                System.out.println("该构造器的形参类型:" + parameterType);
            }
        }
```

### 五.通过反射创建对象

1.方式一：调用类中的 oublic 修饰的无参构造器 2.方式二：调用类中的指定构造器

3.Class 类相关方法

*   newInstance：调用类中的无参构造器，获取对应类的对象
    
*   getConstructor(Class...clazz):根据参数列表，获取对应的 public 构造器对象
    
*   getDecalaredConstructor((Class...clazz):根据参数列表，获取对应的所有构造器对象
    

4.Constructor 类相关方法

*   setAccessible:暴破
    
*   newlnstance(Object...obj):调用构造器
    

#### 5.1 通过反射访问类中的成员

*   访问属性
    

1.根据属性名获取 Field 对象

Field f=clazz 对象.getDeclaredField(属性名)：

2.暴破：f.setAccessible(true);//f 是 Field

3.访问

f.set(o,值);syso(f.get(o));

4.如果是静态属性，则 set 和 get 中的参数 o,可以写成 null

```
* @author LeeZhi
 * @version 1.0
 */
public class ReflectAccessProperty {

    public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchFieldException {
        
        Class<?> stuClass = Class.forName("com.gbx.reflection.Student");
        
        Object o = stuClass.newInstance();
        
        Field age = stuClass.getField("age");
        age.set(o,18);
        System.out.println(o);

        
        Field name = stuClass.getDeclaredField("name");
        
        name.setAccessible(true);
        name.set(o,"小黄");
        name.set(null,"子");
        System.out.println(o);
    }
}
class Student{
    public int age;
    private static String name;

    public Student() {
    }

    @Override
    public String toString() {
        return "Student{" +
                "age=" + age +
                '}';
    }
```

*   访问方法
    

1.根据方法名和参数列表获取 Method 方法对象：Method m=clazz.getMethod(方法名，XX.class);

2.获取对象：Object o=clazz.newInstance();

3.暴破：m.setAccessible(true):

4.访问：Object returnValue=m.invoke(o,实参列表)：//o 就是对象

5.注意：如果是静态方法，则 invoke 的参数 o,可以写成 null!

```
public class ReflectAccessMethod {
    public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException, InvocationTargetException {
        
        Class<?> bossCls = Class.forName("com.gbx.reflection.Boss");
        
        Object o = bossCls.newInstance();
        
        
        
        Method hi1 = bossCls.getDeclaredMethod("hi",String.class);
        
        hi1.invoke(o,"Lee");

        
        
        Method say = bossCls.getDeclaredMethod("say", int.class, String.class, char.class);
        say.setAccessible(true);
        System.out.println(say.invoke(100, "小黄", "女"));
        
        System.out.println(say.invoke(null,200,"李四",'女'));
        
        Object reVal=say.invoke(null,300,"王五",'男');
        System.out.println("reVal的运行类型="+reVal.getClass());
    }

}
class Boss{
    public int age;
    private static String name;

    private static String say(int n,String s,char c){
        return n +" "+s +" "+c;
    }
    public void hi(String s){
        System.out.println("hi" + s);
    }
}
```