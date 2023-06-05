# 【Java基础】Java对象创建的几种方式
先上关键内容，所用到的代码请参考文末示例代码。

这是一种最常用的创建对象的方式。

```
Student student1 = new Student();
```

需要有一个无参构造方法，这个 newInstance()方法调用无参的构造函数创建对象。

> 类名.calss.newInstance( )

```
Student student2 = Student.class.newInstance();
```

该方法就是反射机制，事实上 Class 的 newInstance()方法内部就是调用 Constructor 的 newInstance()方法。

Class 类的 newInstance 只能触发无参构造方法创建对象，而构造器类的 newInstance 能触发有参数或者任意参数的构造方法来创建对象。

java.lang.reflect.Constructor 类里也有一个 newInstance()方法可以创建对象。我们可以通过这个 newInstance()方法调用有参数的和私有的构造函数。

```
Constructor student3 = Constructor.class.newInstance();
```

Tips：要使用 clone()方法，我们需要先实现 Cloneable 接口并实现其定义的 clone()方法。

无论何时我们调用一个对象的 clone()方法，jvm 就会创建一个新的对象，将前面对象的内容全部拷贝进去。

用 clone()方法创建对象并不会调用任何构造函数。

```
Student student4 = new Student().clone();
```

Java 序列化是指把 Java 对象转换为字节序列的过程，而 Java 反序列化是指把字节序列恢复为 Java 对象的过程。

使用反序列化：当我们序列化和反序列化一个对象，jvm 会给我们创建一个单独的对象。

在反序列化时，jvm 创建对象并不会调用任何构造函数。

> 为了反序列化一个对象，我们需要让我们的类实现 Serializable 接口。

```
ObjectInputStream ois = new ObjectInputStream(new FileInputStream(FILE_NAME));



Object student5 = ois.readObject();
```

六、创建对象的 5 种方式调用构造器总结

![](https://static001.geekbang.org/infoq/3a/3a469e52c20f50fdf330aad96e00d84a.png)

Java 创建实例对象是不是必须要通过构造函数？这其实是衍生出来的一个面试题。上面问题的答案很明显了：Java 创建实例对象，并不一定必须要调用构造器的。

以下是本文所用到的所有示例代码。

7.1 编写 Student 学生类
------------------

```
package com.effective.chapter2.other;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Student implements Cloneable, Serializable {
    private String name;
    private Integer age;

    @Override
    public Student clone() {
        try {
            Student clone = (Student) super.clone();
            
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

7.2 编写测试类
---------

```
package com.effective.chapter2.other;

import java.io.*;
import java.lang.reflect.Constructor;

public class CreateObjectTest {

    private static final String FILE_NAME = "student.obj";

    public static void main(String[] args) throws InstantiationException, IllegalAccessException, IOException {

        
        Student student1 = new Student();
        System.out.println("使用new关键字创建对象：" + student1);

        
        Student student2 = Student.class.newInstance();
        System.out.println("使用Class类的newInstance()方法创建对象：" + student2);

        
        Constructor student3 = Constructor.class.newInstance();
        System.out.println("使用Constructor类的newInstance()方法创建对象：" + student3);

        
        Student student4 = new Student().clone();
        System.out.println("使用clone()方法创建对象：" + student4);

        try {
            
            
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(FILE_NAME));
            oos.writeObject(student1);

            ObjectInputStream ois = new ObjectInputStream(new FileInputStream(FILE_NAME));
            
            Object student5 = ois.readObject();
            System.out.println("使用反序列化创建对象：" + student5);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
}
```

