# Java设计模式：（1）设计模式七大设计原则-里氏替换原则 - 腾讯云开发者社区-腾讯云
Save From : [Java设计模式：（1）设计模式七大设计原则-里氏替换原则 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1599626?from=article.detail.1599628) 

## Content
Java设计模式：（1）设计模式七大设计原则-里氏替换原则
=============================

发布于2020-03-17 03:12:49阅读 2620

### 面向对象中关于继承的思考

1）继承包含这样一层含义：父类中凡是已经实现好的方法，实际上是在设定规范和契约，虽然它并不强制要求所有的子类必须遵循这些契约，但是如果子类对这些已经实现的方法任意修改，就会对整个继承体系造成破坏

2）继承在给程序实际带来便利的同时，也带来了弊端。比如使用继承会给程序带来侵入型，程序的可移植性降低，增加对象间的耦合性，如果一个类被其他的类所继承，则当这个类需要修改时，必须考虑 到所有的子类，并且父类修改后，所有涉及到子类的功能都有可能产生故障

### 里氏替换原则

1）在使用继承时，遵循里氏替换规则，在子类中尽量不要重写父类的方法

2）里氏替换原则表示：继承实际上让两个类耦合性增强了，在适当的情况下，可以通过聚合，组合，依赖来解决问题

**举个例子1**

```
public class Liskov01 {
    public static void main(String[] args) {
        A a = new A();
        System.out.println("11-3=" + a.func1(11, 3));
        System.out.println("1-8=" + a.func1(1, 8));

        B b = new B();
        System.out.println("11+3=" + b.func1(11, 3));
        System.out.println("1-8=" + b.func1(1, 8));
        System.out.println("11+3+9=" + b.func2(11, 3));
    }
}

class A {
    public int func1(int num1, int num2) {
        return num1 - num2;
    }
}

class B extends A {
    public int func1(int a, int b) {
        return a + b;
    }

    public int func2(int a, int b) {
        return func1(a, b) + 9;
    }
}
```

复制

分析：在例子1中，运行程序后会发现原来运行正常相减功能发生了错误。原因是因为B继承了父类的方法并重写了此方法。

**举个例子2**

```
public class Liskov {
    public static void main(String[] args) {
        A a = new A();
        System.out.println("11-3=" + a.func1(11, 3));
        System.out.println("1-8=" + a.func1(1, 8));

        B b = new B();
        System.out.println("11+3=" + b.func1(11, 3));
        System.out.println("1+8=" + b.func1(1, 8));
        System.out.println("11+3+9=" + b.func2(11, 3));
        System.out.println("11-3=" + b.func3(11, 3));
    }
}

class Base{
}

class A extends Base{
    public int func1(int num1,int num2){
        return num1 - num2;
    }
}

class B extends Base{
    private A a = new A();

    public int func1(int a,int b){
        return a + b;
    }

    public int func2(int a,int b){
        return func1(a,b) + 9;
    }

    public int func3(int a,int b){
        return this.a.func1(a,b);
    }
}
```

复制

分析：在例子2中，原来的父类和子类都继承一个更通俗的基类，原有的继承关系去掉，才用依赖，聚合，组合等关系代替
## Note