# Python快速进修指南：面向对象进阶
在上一期中，我们对Python中的对象声明进行了初步介绍。这一期，我们将深入探讨对象继承、组合以及多态这三个核心概念。不过，这里不打算赘述太多理论，因为我们都知道，Python与Java在这些方面的主要区别主要体现在语法上。例如，Python支持多重继承，这意味着一个类可以同时继承多个父类的属性和方法，而Java则只支持单一继承。另一个值得注意的差异是对super关键字的使用。在Java中，我们经常需要显式地使用super来调用父类的构造器，而在Python中，这一步骤是可选的。如果没有指定，Python会自动调用父类的构造器，这让代码看起来更加简洁。

当然，我们这里不会举出太多复杂的例子来让大家头疼。毕竟我们的目标是理解和应用这些概念，而不是准备考试。我们将通过一些简单直观的例子，帮助大家清晰地把握Python在继承、组合和多态方面的特点和优势。

Python中的继承是一种用于创建新类的机制，新类可以继承一个或多个父类的特性。在面向对象编程中，和Java一样继承提供了代码复用的强大工具。

```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        pass

class Dog(Animal):
    
        

    def speak(self):
        return "Woof!"


dog = Dog("Buddy")
print(dog.speak())  
print(dog.name)  

```

不声明\_\_init\_\_默认使用super调用父类的构造器，如果你一旦声明了\_\_init\_\_那么记得显式的调用super，否则将无效

在Python中，多继承是一个强大的特性，允许一个类同时继承多个父类。下面是一个多继承的例子，来帮助你更好地理解这一概念：

```python
class Father:
    def __init__(self):
        self.father_name = "Father's name"

    def gardening(self):
        return "I enjoy gardening"

class Mother:
    def __init__(self):
        self.mother_name = "Mother's name"

    def cooking(self):
        return "I love cooking"

class Child(Father, Mother):
    def __init__(self):
        Father.__init__(self)
        Mother.__init__(self)
        self.child_name = "Child's name"

    def activities(self):
        return f"{self.father_name} likes {self.gardening()} and {self.mother_name} likes {self.cooking()}."


child = Child()
child.father_name = "father"
child.mother_name = "mom"
print(child.activities())

```

在Python中，我们可以在程序运行过程中根据需要向对象动态地添加新的行为或数据，这种方式为处理各种复杂和不可预见的编程情况提供了极大的便利。相比之下，Java由于其静态类型的特性，对于运行时的动态变化就显得有些束手束脚。在Java中，所有的属性和方法都必须在编译时明确定义。

```python
class Printer:
    def print_document(self, content):
        return f"Printing: {content}"

class Scanner:
    def scan_document(self):
        return "Scanning a document"

class MultifunctionMachine:
    pass


mfm = MultifunctionMachine()
mfm.printer = Printer()
mfm.scanner = Scanner()


print(mfm.printer.print_document("Hello World"))  
print(mfm.scanner.scan_document())                

```

pass 这个看似简单却不可忽视的关键字，我不太确定之前是否有提及。在Python的世界里，pass 是一种特别有趣的存在。想象一下，在编写代码时，我们常常会遇到一种场景：逻辑上需要一个语句，但此刻我们可能还没有准备好具体的实现，或者我们故意想留下一个空的代码块，以备将来填充。这时，pass 就像是一个灵巧的小助手，默默地站在那里，告诉Python解释器：“嘿，我在这里，但你可以暂时忽略我。”

为了让大家更直观地理解多态，我准备了一个例子，相信通过这个实例，多态的概念会变得生动而明晰。在Python中，多态表现得非常直观，而且它与Java在这方面的处理几乎如出一辙。

```python
class Animal:
    def speak(self):
        raise NotImplementedError("Subclass must implement abstract method")

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

def animal_speak(animal):
    print(animal.speak())


my_dog = Dog()
my_cat = Cat()

animal_speak(my_dog)  
animal_speak(my_cat)  

```

在本期文章中，我们深入探讨了Python的对象继承、组合以及多态这三个核心概念。从继承的灵活性，如Python的多重继承和super关键字的使用，到组合中的动态属性添加，我们逐一解析了Python与Java在这些方面的相似之处和差异。通过具体的例子，我们展示了Python中多态的直观表现，强调了它与Java的相似性。这些讨论不仅帮助理解Python的对象模型，而且对比了Java和Python在面向对象编程方面的不同处理方式