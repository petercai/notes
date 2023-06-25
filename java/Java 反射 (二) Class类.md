# Java 反射 (二) Class类
二.Class 类
---------

### 1.基本介绍

*   Class 也是类，因此也继承 Object 类
    
    ![](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/00325/image-20221120205016225.png)
    
      
    
*   Class 类对象不是 new 出来的，而是系统创建的
    

```
ClassLoader类
          public Class<?>LoadClass(String name)throws ClassNotFoundException{
              return loadclass(name,false);
          }
           */
          Cat cat = new Cat();
          
          
            ClassLoader类，仍然是通过CLassLoader类加载Cat类的Class对象
            public Class<?>LoadClass(String name)throws CLassNotFoundException{
              return loadClass(name,false)
            }
           */
          Class cls1 = Class.forName("com.gbx.Cat");
```

*   对于某个类的 Class 类对象，在内存中只有一份，因为类只加载一次
    

```
Class cls2 = Class.forName("com.gbx.Cat");
          System.out.println(cls1.hashCode()==cls2.hashCode());
          Class cls3 = Class.forName("com.gbx.reflection.Dog");
          System.out.println(cls3.hashCode()==cls2.hashCode());
```

![](_assets/104f16cc56c9caf9028a78eeb9a2ebe7.png)

*   每个类的实例都会记得自己是由哪个 Class 实例所生成
    
*   通过 Class 可以完整地得到一个类的完整结构，通过一系列 API
    
*   Classi 对象是存放在堆的
    
*   类的字节码二进制数据，是放在方法区的，有的地方称为类的元数据（包括方法代码，变量名，方法名，访问权限等等)https://www.zhihu.com/question/38496907
    

### 2.常用方法

```
* @author LeeZhi
 * @version 1.0
 * 演示Class类的常用方法
 */
public class Class02 {
    public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchFieldException {

        String classAllPath = "com.gbx.Car";
        
        
        Class<?>cls = Class.forName(classAllPath);
        
        System.out.println(cls);
        System.out.println(cls.getClass());
        
        System.out.println(cls.getPackage().getName());
        
        System.out.println(cls.getName());
        
        Car car =(Car)cls.newInstance();
        System.out.println(car);
        
        Field brand = cls.getField("brand");
        System.out.println(brand.get(car));
        
        brand.set(car,"奔驰");
        System.out.println(brand.get(car));
        
        Field[] fields = cls.getFields();
        System.out.println("===============");
        for (Field f:fields){
            System.out.println(f.getName());
        }
    }
}
```

### 3.获取 Class 对象六种方式

1.前提：已知一个类的全类名，且该类在类路径下，可通过 Class 类的静态方法 forName()获取，可能抛出 ClassNotFoundException,实例：Class cls1=Class.forName("java.lang.Cat")**应用场景**：多用于配置文件，读取类全路径，加载类

2.前提：若已知具体的类，通过类的 class 获取，该方式最为安全可靠，程序性能最高实例：Class cls2=Cat.class;**应用场景**：多用于参数传递，比如通过反射得到对应构造器对象

3.前提：已知某个类的实例，调用该实例的 getClass()方法获取 Class 对象，实例：Class clazz=对象.getClass() //运行类型**应用场景**：通过创建好的对象，获取 Class 对象.

4.其他方式 ClassLoader cl =对象.getClass().getClassLoader();Class clazz4=cl.loadClass("类的全类名”);

5.基本数据(int,char,boolean,float,,double,byte,long,short)按如下方式得到 Class 类对象 Class cls=基本数据类型.class

6.基本数据类型对应的包装类，可以通过.type 得到 Class 类对象 Class cls=包装类.TYPE

```
* @author LeeZhi
 * @version 1.0
 * 演示得到Class对象的各种方式(6)
 */
public class GetClass_ {

    public static void main(String[] args) throws ClassNotFoundException {

        
        String classAllPath = "com.gbx.Car";
        Class<?>cls1 = Class.forName(classAllPath);
        System.out.println(cls1);

        
        Class<Car> cls2 = Car.class;
        System.out.println(cls2);

        
        Car car = new Car();
        Class cls3 = car.getClass();
        System.out.println(cls3);

        
        
        ClassLoader classLoader = car.getClass().getClassLoader();
        
        Class<?> cls4 = classLoader.loadClass(classAllPath);
        System.out.println(cls4);

        
        System.out.println(cls1.hashCode()==cls2.hashCode());
        System.out.println(cls2.hashCode()==cls3.hashCode());
        System.out.println(cls3.hashCode()==cls4.hashCode());

        
        Class<Integer> integerClass = int.class;
        Class<Character>characterClass = char.class;
        Class<Boolean>booleanClass = boolean.class;
        System.out.println(integerClass);

        
        Class<Integer>type1=Integer.TYPE;
        Class<Character>type2 = Character.TYPE;
        System.out.println(type1);

        System.out.println(integerClass.hashCode() == type1.hashCode());
    }
}
```

![](_assets/c20de0b593ff190653e48fdbd60288cc.png)

### 4.哪些类型有 Class 对象

*   如下类型有 Class 对象
    
*   外部类，成损内部类，静态内部类，局部内部类，匿名内部类
    
*   interface:接口
    
*   数组
    
*   enum:枚举
    
*   annotation:注解
    
*   基本数据类型
    
*   void
    

```
* @author LeeZhi
 * @version 1.0
 * 演示那些类型有Class对象
 */
public class AllTypeClass {
    public static void main(String[] args) {
        Class<String> cls1 = String.class;
        Class<Serializable> cls2 = Serializable.class;
        Class<Integer[]> cls3 = Integer[].class;
        Class<float[][]>cls4 = float[][].class;
        Class<Deprecated>cls5 = Deprecated.class;
        Class<Thread.State> cls6 = Thread.State.class;
        Class<Long>cls7 = long.class;
        Class<Void>cls8 = void.class;
        Class<Class>cls9 = Class.class;
        System.out.println(cls1);
        System.out.println(cls2);
        System.out.println(cls3);
        System.out.println(cls4);
        System.out.println(cls5);
        System.out.println(cls6);
        System.out.println(cls7);
        System.out.println(cls8);
        System.out.println(cls9);

    }
}
```