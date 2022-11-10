# Java中的内部类与匿名内部类详解_Java_共饮一杯无_InfoQ写作社区
🦘内部类
-----

### 🐓什么是内部类

将一个类 A 定义在另一个类 B 里面，里面的那个类 A 就称为内部类，B 则称为外部类。

### 🐣成员内部类

成员内部类 ：**定义在类中方法外的类。** 定义格式：

> class 外部类 {class 内部类{}}

在描述事物时，若一个事物内部还包含其他事物，就可以使用内部类这种结构。比如，电脑类 Computer 中包含中央处理器类 Cpu ，这时， Cpu 就可以使用内部类来描述，定义在成员位置。代码举例：

```
class Computer { 
    class Cpu { 
    }
}
```

### 🐤访问特点

内部类可以直接访问外部类的成员，包括私有成员。外部类要访问内部类的成员，必须要建立内部类的对象。创建内部类对象格式：

> 外部类名.内部类名 对象名 = new 外部类().new 内部类()；

访问演示，代码如下：定义类：

```
public class Person {
    private boolean live = true;
    class Heart {
        public void jump() {
            
            if (live) {
                System.out.println("心脏在跳动💓");
            } else {
                System.out.println("心脏不跳了💔");
            }
        }
    }
    public boolean isLive() {
        return live;
    }
    public void setLive(boolean live) {
        this.live = live;
    }
}
```

定义测试类：

```
public class InnerDemo {
    public static void main(String[] args) {
        
        Person p = new Person();
        
        Heart heart = p.new Heart();
        
        heart.jump();
        
        p.setLive(false);
        
        heart.jump();
    }
}
```

输出结果：

> 心脏在跳动💓心脏不跳了💔

💡内部类仍然是一个独立的类，在编译之后会内部类会被编译成独立的.class 文件，但是前面冠以外部类的类名和Heart.class

🐦匿名内部类
-------

匿名内部类 ：是内部类的简化写法。它的本质是一个带具体实现的父类或者父接口的 匿名的 子类对象。开发中，最常用到的内部类就是匿名内部类了。以接口举例，当你使用一个接口时，似乎得做如下几步操作，

1.  定义子类
    
2.  重写接口中的方法
    
3.  创建子类对象
    
4.  调用重写后的方法
    

我们的目的，最终只是为了调用方法，那么能不能简化一下，把以上四步合成一步呢？匿名内部类就是做这样的快捷方式。

### 🦅前提

匿名内部类必须继承一个父类或者实现一个父接口。

### 🦆格式

> new 父类名或者接口名(){// 方法重写 @Overridepublic void method() {// 执行语句}};

### 🦢使用方式

以接口为例，匿名内部类的使用，代码如下：定义接口：

```
public abstract class FlyAble{
    public abstract void fly();
}
```

创建匿名内部类，并调用：

```
public class InnerDemo {
    public static void main(String[] args) {
        
        1.等号右边:是匿名内部类，定义并创建该接口的子类对象
        2.等号左边:是多态赋值,接口类型引用指向子类对象
        */
        FlyAble f = new FlyAble(){
            public void fly() {
                System.out.println("芜湖，起飞！！🕊");
            }
        };
        
        f.fly();
    }
}
```

通常在方法的形式参数是接口或者抽象类时，也可以将匿名内部类作为参数传递。代码如下：

```
public class InnerDemo2 {
    public static void main(String[] args) {
        
        1.等号右边:是匿名内部类，定义并创建该接口的子类对象
        2.等号左边:是多态赋值,接口类型引用指向子类对象
        */
        FlyAble f = new FlyAble(){
            public void fly() {
                System.out.println("芜湖，起飞！！✈️✈️✈️");
            }
        };
        
        showFly(f);
    }
    public static void showFly(FlyAble f) {
        f.fly();
    }
}
```

以上两步，也可以简化为一步，代码如下：

```
public class InnerDemo3 {
    public static void main(String[] args) {
        
        创建匿名内部类,直接传递给showFly(FlyAble f)
        */
        showFly(new FlyAble(){
            public void fly() {
                System.out.println("芜湖，起飞！！✈️✈️✈️");
            }
        });
    }
    public static void showFly(FlyAble f) {
        f.fly();
    }
}
```

> 本文内容到此结束了，
> 
> 如有收获欢迎点赞👍收藏💖关注✔️，您的鼓励是我最大的动力。
> 
> 如有错误❌疑问💬欢迎各位大佬指出。
> 
> **主页**：[共饮一杯无的博客汇总👨‍💻](https://www.infoq.cn/u/zhanjq/publish)  
> 
> **保持热爱，奔赴下一场山海**。🏃🏃🏃