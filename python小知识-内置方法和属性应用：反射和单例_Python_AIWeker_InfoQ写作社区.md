# python小知识-内置方法和属性应用：反射和单例_Python_AIWeker_InfoQ写作社区
1\. 前言
------

python 除了丰富的第三方库外，本身也提供了一些内在的方法和底层的一些属性，大家比较常用的如 dict、list、set、min、max、range、sorted 等。笔者最近在做项目框架时涉及到一些不是很常用的方法和属性，在本文中和大家做下分享。

2\. 内置方法和函数介绍
-------------

*   enumerate
    
*   如果你需要遍历可迭代的对象，有需要获取它的序号，可以用`enumerate`, 每一个 next 返回的是一个 tuple
    

```
list1 = [1, 2, 3, 4]
    list2 = [4, 3, 2, 1]
    for idx, value in enumerate(list1):
        print(idx, value, list2[idx])
```

*   zipzip 从参数中的多个迭代器取元素组合成一个新的迭代器;
    

```
b = [4, 3, 2, 1]
    for i in zip(range(len(b)), b):
        print(i)
```

*   globals()： 一个描述当前执行过程中全局符号表的字典，可以看出你执行的所有过程
    
*   id(object)： python 对象的唯一标识
    
*   staticmethod 类静态函数注解
    

```
@staticmethod  
    def test(): 
        print('this is static method')

    Foo.test = test
    Foo.test()
```

*   类的属性我们来看下一个类的申明，如下：
    

```
class Foo():
        """this is test class"""
        def __init__(self, name):
            self.name = name
        
        def run(self):
            print('running')
```

```
dir(Foo)
    ['__class__',
    '__delattr__',
    '__dict__',
    '__dir__',
    '__doc__',
    '__eq__',
    '__format__',
    '__ge__',
    '__getattribute__',
    '__gt__',
    '__hash__',
    '__init__',
    '__init_subclass__',
    '__le__',
    '__lt__',
    '__module__',
    '__ne__',
    '__new__',
    '__reduce__',
    '__reduce_ex__',
    '__repr__',
    '__setattr__',
    '__sizeof__',
    '__str__',
    '__subclasshook__',
    '__weakref__',
    'run']


    
    Foo.__doc__
    

    
    Foo.__dict__
    mappingproxy({'__module__': '__main__',
              '__doc__': 'this is test class',
              '__init__': <function __main__.Foo.__init__(self, name)>,
              'run': <function __main__.Foo.run(self)>,
              '__dict__': <attribute '__dict__' of 'Foo' objects>,
              '__weakref__': <attribute '__weakref__' of 'Foo' objects>})

    
    Foo.__base__

    
    Foo.__name__
```

```
foo.name
    foo.run()

    
    def method():
        print("cow")
    
    setattr(foo, "type", "cow")
    setattr(foo, "getcow", method)
    
    foo.type
    foo.getcow()

    
    delattr(foo, "type")

    
    if hasattr(foo, "run"): 
        func = getattr(foo, "run")
        func()
```

3\. 单例模式应用
----------

单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一。单例模式要求在类的使用过程中只实例化一次，所有对象都共享一个实例。创建的方法是在实例的时候判断下是否已经实例过了，有则返回实例化过的全局实例。python 是如何实现的呢？关键是找到实例化的地方，对就是前面说的`__new__`  

```
class Singleton(object):
    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            cls._instance = object.__new__(cls)
        return cls._instance
    
    def __init__(self, name):
        self.name = name


a = Singleton('name1')
b = Singleton('name2')
print(id(a), id(b))
print(a.name, b.name)
```

4\. 反射应用
--------

反射在许多框架中都有使用到，简单就是通过类的名称（字符串）来实例化类。一个典型的场景就是通过配置的方式来动态控制类的执行，比如定时任务的执行，通过维护每个定时任务类的执行时间，在执行时间到的时候，通过反射方式实例化类，执行任务，在 java 中也非常的常见。

python 的实现可以通过上面说的`getattr`获取模块中的类， 通过 methodcaller 来调用方法。我们来看一个简单的例子

```
import importlib
from operator import methodcaller

class Foo():
    """this is test class"""
    def __init__(self, name):
        self.name = name
    
    def run(self, info):
        print('running %s' % info)


api_module = importlib.import_module('__main__') 

clazz = getattr(api_module, 'Foo')


params = ["milk"]
instance = clazz(*params)


task_result = methodcaller("run", "reflection")(instance)
```

5\. 总结
------

本文通过分享了 python 内置方法和属性， 并在单例模式和反射中进行应用。希望对你有帮助，欢迎交流 @mintel 要点总结如下：

*   dir 下类
    
*   查看类自定义属性\_\_dict\_\_
    
*   \_\_new\_\_实例化类，\_\_init\_\_初始化类
    
*   getattr 获取属性
    
*   setattr 设置属性
    
*   记住 importlib 和 methodcaller