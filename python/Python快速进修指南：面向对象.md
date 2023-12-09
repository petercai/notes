# Python快速进修指南：面向对象
首先，让我来介绍一下今天的主题。今天我们将讨论封装、反射以及单例模式。除此之外，我们不再深入其他内容。关于封装功能，Python与Java大致相同，但写法略有不同，因为Python没有修饰符。而对于反射来说，我认为它比Java简单得多，不需要频繁地获取方法和属性，而是有专门的方法来实现。最后，我们将简单地实现一下单例模式，这样面向对象章节就告一段落了。

封装是指将数据和方法封装在一个类中。在Python中，我们可以通过属性和方法来实现封装。属性可以通过getter和setter方法来访问和修改，而方法可以在类的内部进行访问和使用。然而，与Java不同的是，虽然方法在Python中是可以调用的，但Java不允许。另外，属性也有一些区别，如果属性以双下划线开头，并且没有声明属性，将无法直接访问。除非你动态赋值，那么将失去封装的作用。

使用双下划线开头的属性是私有属性，下面是一个简单的示例代码：

```python
class Person:
    def __init__(self, name, age):
        self.__name = name    
         
    def get_name(self):
        return self.__name

    def set_name(self, name):
        self.__name = name

person = Person("xiaoyu")
print(person.get_name())    

```

我们都是学习Java的，所以对于getter和setter方法的使用应该是基本常识了。记住在Python中，我们使用双下划线来定义私有属性，但实际上这只是一种约定，Python并没有真正的私有属性概念。我们可以通过一些特殊的方式来访问和修改私有属性，但这违背了封装的原则，不建议直接这样做。

反射是一种强大的编程技术，它使得在运行时可以动态地获取和修改对象的属性和方法。在Python中，我们可以利用内置的getattr()、setattr()和hasattr()等函数来实现反射的功能。通过反射，我们可以在运行时根据需要获取或修改对象的属性和方法，从而实现更灵活和动态的编程。不过，我还是有原则的，毕竟Java作为一种商业生态体系成熟的编程语言，在各个领域都有着强大的应用和支持，这是其他语言所无法比拟的。

下面是一个简单的示例代码：

```python
class MyClass:
    def __init__(self, name):
        self.name = name

    def hello(self):
        print("Hello, {}!".format(self.name))

    def dance(self):
        print("dance, {}!".format(self.name))

    def cmd(self):
        method_name = input("====>")
        if hasattr(obj, method_name):
            method = getattr(obj, method_name)
            method()  


obj = MyClass("xiaoyu")
obj.cmd()

```

这样就可以获取到方法然后去实现反射了，我就不演示setattr了，自行演示吧。

![](_assets/9e6f643349ee452e962cc9198c65d55a~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

单例模式是一种常用的设计模式，它可以确保一个类只有一个实例，并且提供一个全局访问点，方便其他对象对该实例进行调用。在Python中，我们可以通过使用模块级别的变量来实现单例模式，这种方式非常简洁和高效。

下面是一个简单的示例代码，展示了如何在Python中实现单例模式：

```python
class Singleton:
    _instance = None

    @classmethod
    def get_instance(cls):
        if not cls._instance:
            cls._instance = cls()
        return cls._instance
        
s1 = Singleton.get_instance()
s2 = Singleton.get_instance()

print(s1 is s2)  

```

与Java相似，Python中也可以使用classmethod装饰器来实现方法，只是在Python中我们称之为装饰器而非注解。

另外，Python中也有一种类似于Java中常用的stream流处理for循环的高级用法，只不过在Python中这种写法是倒着的。所以人们称之为字典推导或列表推导。为了方便记忆，我一直称之为推倒。

```python
student = {
    "name": "xiaoyu",
    "age": 18
}

[print(key + ": " + str(value)) for key, value in student.items() if key == "name"]


```

在今天的课上，我们深入讨论了封装、反射和单例模式这几个重要的概念。我不想过多地赘述它们的细节，但是请大家务必记住它们的基本语法规则，因为这也是面向对象章节的结束。我希望大家能够牢牢掌握这些知识点，为未来的学习打下坚实的基础。