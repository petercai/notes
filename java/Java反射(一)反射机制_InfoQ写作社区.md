# Java反射(一)反射机制_InfoQ写作社区
一.反射机制
------

### 1.一个需求引出反射

*   请看下面的问题 1.根据配置文件 re.properties 指定信息，创建对象并调用方法
    
*   classfullpath=com.gbx.Cat method=hi
    
*   思考：1.使用现有技术，你能做的吗？
    

2.这样的需求在学习框架时特别多，即通过外部文件配置，在不修改源码情况下，来控制程序， 也符合设计模式的 ocp 原则（开闭原则）

3.快速入门 com.gbx.reflection.questionReflectionQuestion.java

```
* @author LeeZhi
 * @version 1.0
 * 反射问题的引入
 */
@SuppressWarnings({"all"})
public class ReflectionQuestion {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException, InvocationTargetException {

        
        

        
        
        

        

        
        Properties properties = new Properties();
        properties.load(new FileInputStream("src\\re.properties"));
        String classfullpath = properties.getProperty("classfullpath").toString();
        String method = properties.get("method").toString();
        System.out.println("classfullpath:" + classfullpath +"\n"+ "method:" + method);

        
        

        
        
        Class cls = Class.forName(classfullpath);
        
        Cat o = (Cat)cls.newInstance();
        
        
        Method method1 = cls.getMethod(method);
        
        System.out.println("===================================");
        method1.invoke(o);
    }
}
```

> 1.反射机制允许程序在执行期借助于 Reflection API 取得任何类的内部信息（比如成员变量，构造器，成员方法等等)，并能操作对象的属性及方法。反射在设计模式和框架底层都会用到
> 
> 2.加载完类之后，在堆中就产生了一个 Class 类型的对象（一个类只有一个 Classi 对象）,这个对象包含了类的完整结构信息。通过这个对象得到类的结构。这个对象就像一面镜子，透过这个镜子看到类的结构，所以，形象的称之为：反射
> 
> p 对象-->类型 Person 类对象 cls-->类型 Class 类

### 2.反射原理图

![](_assets/83fa7181988009da00d48f44ee01fc01.png)

### 3.反射相关类

*   **Java 反射机制可以完成**
    
*   在运行时判断任意一个对象所属的类
    
*   在运行时构造任意一个类的对象
    
*   在运行时得到任意一个类所具有的成员变量和方法
    
*   在运行时调用任意一个对象的成员变量和方法
    
*   生成动态代理
    
*   反射相关的主要类
    
*   java.lang.Class:代表一个类，Class 对象表示某个类加载后在堆中的对象
    
*   java.lang.reflect.Method:代表类的方法,Method 对象表示某个类的方法
    
*   java.lang.reflect.Field:代表类的成员变量,Field 对象表示某个类的成员变量
    
*   java.lang.reflect.Constructor:代表类的构造方法,Constructor 对象表示构造器
    
*   这些类在 java.lang.reflection 包内
    

```
public class Reflection01 {

    public static void main(String[] args) throws Exception {
        
        Properties properties = new Properties();
        properties.load(new FileInputStream("src\\re.properties"));
        String classfullpath = properties.getProperty("classfullpath").toString();
        String method = properties.get("method").toString();

        
        
        Class cls = Class.forName(classfullpath);
        
        Cat o = (Cat)cls.newInstance();
        
        
        Method method1 = cls.getMethod(method);
        
        System.out.println("===================================");
        method1.invoke(o);

        
        
        
        Field nameField = cls.getField("age");
        System.out.println(nameField.get(o));       

        
        Constructor constructor1 = cls.getConstructor();
        System.out.println(constructor1);
        
        Constructor constructor2 = cls.getConstructor(String.class);  
        System.out.println(constructor2);
    }
}
```

```
public int age = 2;

    public Cat(){}
    public Cat(String name){
        this.name = name;
    }
```

### 4.反射调优

#### 4.1 反射的优缺点

```
* @author LeeZhi
 * @version 1.0
 * 测试反射调用的性能,和优化方案
 */
public class Reflection02 {
    public static void main(String[] args) throws ClassNotFoundException, InvocationTargetException, InstantiationException, IllegalAccessException, NoSuchMethodException {
        m1();
        m2();
    }
    
    public static void m1(){
        Cat cat = new Cat();
        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            cat.hi();
        }
        long end = System.currentTimeMillis();
        System.out.println("m1方法耗时="+(end-start));
    }
    
    public static void m2() throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException, InvocationTargetException {
        Class cls = Class.forName("com.gbx.Cat");
        Object o = cls.newInstance();
        Method hi = cls.getMethod("hi");
        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            hi.invoke(o);
        }
        long end = System.currentTimeMillis();
        System.out.println("m2方法耗时="+(end-start));
    }
}
```

![](_assets/e2778f94946a549fa9ba905d53afceda.png)

*   反射调用优化-关闭访问检查
    
*   Method 和 Field、Constructor 对象都有 setAccessible()方法
    
*   setAccessible 作用是启动和禁用访问安全检查的开关
    
*   参数值为 tue 表示反射的对象在使用时取消访问检查，提高反射的效率。参数值为 false 则表示反射的对象执行访问检查
    
*   ![](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/00325/image-20221120200242934.png)
    
      
    

```
public static void m3() throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException, InvocationTargetException {
        Class cls = Class.forName("com.gbx.Cat");
        Object o = cls.newInstance();
        Method hi = cls.getMethod("hi");
        hi.setAccessible(true);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            hi.invoke(o);
        }
        long end = System.currentTimeMillis();
        System.out.println("m3方法耗时="+(end-start));
    }
```

![](_assets/4e6c266ec9617cd3e20e3ff06fcf1a98.png)