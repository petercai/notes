# 深入浅出Python——Python高级语法之面向对象_Python_何极光_InfoQ写作社区
前言：本博文主要讲解 Python 面向对象，属于 Python 高级语法。基础语法见：[https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a](https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a)，更多内容请访问博主的主页，谢谢！

一、理解面向对象
--------

面向对象是一种抽象化的编程思想，很多编程语言中都有的一种思想。

例如：洗衣服

思考：几种途径可以完成洗衣服？

答： 手洗 和 机洗。

手洗：找盆 - 放水 - 加洗衣粉 - 浸泡 - 搓洗 - 拧干水 - 倒水 - 漂洗 N 次 - 拧干 - 晾晒。

机洗：打开洗衣机 - 放衣服 - 加洗衣粉 - 按下开始按钮 - 晾晒。

思考：对比两种洗衣服途径，同学们发现了什么？

答：机洗更简单

思考：机洗，只需要找到一台洗衣机，加入简单操作就可以完成洗衣服的工作，而不需要关心洗衣机内部发生了什么事情。

> 总结：==面向对象就是将编程当成是一个事物，对外界来说，事物是直接使用的，不用去管他内部的情况。而编程就是设置事物能够做什么事。==

二、类和对象
------

思考：洗衣机洗衣服描述过程中，洗衣机其实就是一个事物，即对象，洗衣机对象哪来的呢？

答：洗衣机是由工厂工人制作出来。

思考：工厂工人怎么制作出的洗衣机？

答：工人根据设计师设计的功能图纸制作洗衣机。

总结：图纸 → 洗衣机 → 洗衣服。

在面向对象编程过程中，有两个重要组成部分：==类== 和 ==对象==。

\==类和对象的关系：用类去创建一个对象。==

### 1\. 理解类和对象

#### 1.1 类

类是对一系列具有相同==特征==和==行为==的事物的统称，是一个==抽象的概念==，不是真实存在的事物。

*   特征即是属性
    
*   行为即是方法
    

类比如是制造洗衣机时要用到的图纸，也就是说==类是用来创建对象==。

#### 1.2 对象

对象是类创建出来的真实存在的事物，例如：洗衣机。

> 注意：开发中，先有类，再有对象。

### 2\. 面向对象实现方法

#### 2.1 定义类

Python2 中类分为：经典类 和 新式类

*   语法
    

> 注意：类名要满足标识符命名规则，同时遵循==大驼峰命名习惯==。

*   体验
    

```
class Washer():
    def wash(self):
        print('我会洗衣服')
```

*   拓展：经典类
    

不由任意内置类型派生出的类，称之为经典类

#### 2.2 定义类

对象又名实例。

*   语法
    

*   体验
    

```
haier1 = Washer()


print(haier1)


haier1.wash()
```

> 注意：创建对象的过程也叫实例化对象。

#### 2.3 self

self 指的是调用该函数的对象。

```
class Washer():
    def wash(self):
        print('我会洗衣服')
        
        print(self)



haier1 = Washer()

print(haier1)

haier1.wash()


haier2 = Washer()

print(haier2)
```

> 注意：打印对象和 self 得到的结果是一致的，都是当前对象的内存中存储地址。

三、添加和获取对象属性
-----------

属性即是特征，比如：洗衣机的宽度、高度、重量...

对象属性既可以在类外面添加和获取，也能在类里面添加和获取。

### 1\. 类外面添加对象属性

*   语法
    

*   体验
    

```
haier1.width = 500
haier1.height = 800
```

### 2\. 类外面获取对象属性

*   语法
    

*   体验
    

```
print(f'haier1洗衣机的宽度是{haier1.width}')
print(f'haier1洗衣机的高度是{haier1.height}')
```

### 3\. 类里面获取对象属性

*   语法
    

*   体验
    

```
class Washer():
    def print_info(self):
        
        print(f'haier1洗衣机的宽度是{self.width}')
        print(f'haier1洗衣机的高度是{self.height}')


haier1 = Washer()


haier1.width = 500
haier1.height = 800

haier1.print_info()
```

四、魔法方法
------

在 Python 中，`__xx__()`的函数叫做魔法方法，指的是具有特殊功能的函数。

### 1\. `__init__()`  

#### 1.1 体验`__init__()`  

思考：洗衣机的宽度高度是与生俱来的属性，可不可以在生产过程中就赋予这些属性呢？

答：理应如此。

\==`__init__()`方法的作用：初始化对象。==

```
class Washer():
    
    
    def __init__(self):
        
        self.width = 500
        self.height = 800


    def print_info(self):
        
        print(f'洗衣机的宽度是{self.width}, 高度是{self.height}')


haier1 = Washer()
haier1.print_info()
```

> 注意：
> 
> *   `__init__()`方法，在创建一个对象时默认被调用，不需要手动调用
>     
> *   `__init__(self)`中的 self 参数，不需要开发者传递，python 解释器会自动把当前的对象引用传递过去。
>     

#### 1.2 带参数的`__init__()`  

思考：一个类可以创建多个对象，如何对不同的对象设置不同的初始化属性呢？

答：传参数。

```
class Washer():
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def print_info(self):
        print(f'洗衣机的宽度是{self.width}')
        print(f'洗衣机的高度是{self.height}')


haier1 = Washer(10, 20)
haier1.print_info()


haier2 = Washer(30, 40)
haier2.print_info()
```

### 2\. `__str__()`  

当使用 print 输出对象的时候，默认打印对象的内存地址。如果类定义了`__str__`方法，那么就会打印从在这个方法中 return 的数据。

```
class Washer():
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def __str__(self):
        return '这是海尔洗衣机的说明书'


haier1 = Washer(10, 20)

print(haier1)
```

### 3\. `__del__()`  

当删除对象时，python 解释器也会默认调用`__del__()`方法。

```
class Washer():
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def __del__(self):
        print(f'{self}对象已经被删除')


haier1 = Washer(10, 20)


del haier1
```

五、综合应用
------

### 1\. 烤地瓜

#### 1.1 需求

需求主线：

1.  被烤的时间和对应的地瓜状态：
    
2.  0-3 分钟：生的
    
3.  3-5 分钟：半生不熟
    
4.  5-8 分钟：熟的
    
5.  超过 8 分钟：烤糊了
    
6.  添加的调料：
    
7.  用户可以按自己的意愿添加调料
    

#### 1.2 步骤分析

需求涉及一个事物： 地瓜，故案例涉及一个类：地瓜类。

##### 1.2.1 定义类

*   地瓜的属性
    
*   被烤的时间
    
*   地瓜的状态
    
*   添加的调料
    
*   地瓜的方法
    
*   被烤
    
*   用户根据意愿设定每次烤地瓜的时间
    
*   判断地瓜被烤的总时间是在哪个区间，修改地瓜状态
    
*   添加调料
    
*   用户根据意愿设定添加的调料
    
*   将用户添加的调料存储
    
*   显示对象信息
    

##### 1.2.2 创建对象，调用相关实例方法

#### 1.3 代码实现

##### 1.3.1 定义类

*   地瓜属性
    
*   定义地瓜初始化属性，后期根据程序推进更新实例属性
    

```
class SweetPotato():
    def __init__(self):
        
        self.cook_time = 0
        
        self.cook_static = '生的'
        
        self.condiments = []
```

##### 1.3.2 定义烤地瓜方法

```
class SweetPotato():
    ......
    
    def cook(self, time):
        """烤地瓜的方法"""
        self.cook_time += time
        if 0 <= self.cook_time < 3:
            self.cook_static = '生的'
        elif 3 <= self.cook_time < 5:
            self.cook_static = '半生不熟'
        elif 5 <= self.cook_time < 8:
            self.cook_static = '熟了'
        elif self.cook_time >= 8:
            self.cook_static = '烤糊了'
```

##### 1.3.3 书写 str 魔法方法，用于输出对象状态

```
class SweetPotato():
        ......

    def __str__(self):
        return f'这个地瓜烤了{self.cook_time}分钟, 状态是{self.cook_static}'
```

##### 1.3.4 创建对象，测试实例属性和实例方法

```
digua1 = SweetPotato()
print(digua1)
digua1.cook(2)
print(digua1)
```

##### 1.3.5 定义添加调料方法，并调用该实例方法

```
class SweetPotato():
        ......

    def add_condiments(self, condiment):
        """添加调料"""
        self.condiments.append(condiment)
    def __str__(self):
        return f'这个地瓜烤了{self.cook_time}分钟, 状态是{self.cook_static}, 添加的调料有{self.condiments}'
      

digua1 = SweetPotato()
print(digua1)

digua1.cook(2)
digua1.add_condiments('酱油')
print(digua1)

digua1.cook(2)
digua1.add_condiments('辣椒面')
print(digua1)

digua1.cook(2)
print(digua1)

digua1.cook(2)
print(digua1)
```

#### 1.4 代码总览

```
class SweetPotato():
    def __init__(self):
        
        self.cook_time = 0
        
        self.cook_static = '生的'
        
        self.condiments = []

    def cook(self, time):
        """烤地瓜的方法"""
        self.cook_time += time
        if 0 <= self.cook_time < 3:
            self.cook_static = '生的'
        elif 3 <= self.cook_time < 5:
            self.cook_static = '半生不熟'
        elif 5 <= self.cook_time < 8:
            self.cook_static = '熟了'
        elif self.cook_time >= 8:
            self.cook_static = '烤糊了'

    def add_condiments(self, condiment):
        """添加调料"""
        self.condiments.append(condiment)

    def __str__(self):
        return f'这个地瓜烤了{self.cook_time}分钟, 状态是{self.cook_static}, 添加的调料有{self.condiments}'


digua1 = SweetPotato()
print(digua1)

digua1.cook(2)
digua1.add_condiments('酱油')
print(digua1)

digua1.cook(2)
digua1.add_condiments('辣椒面')
print(digua1)

digua1.cook(2)
print(digua1)

digua1.cook(2)
print(digua1)
```

### 2\. 搬家具

#### 2.1 需求

将小于房子剩余面积的家具摆放到房子中。

#### 2.2 步骤分析

需求涉及两个事物：房子 和 家具，故被案例涉及两个类：房子类 和 家具类。

##### 2.2.1 定义类

*   房子类
    
*   实例属性
    
*   房子地理位置
    
*   房子占地面积
    
*   房子剩余面积
    
*   房子内家具列表
    
*   实例方法
    
*   容纳家具
    
*   显示房屋信息
    
*   家具类
    
*   家具名称
    
*   家具占地面积
    

##### 2.2.2 创建对象并调用相关方法

#### 2.3 代码实现

##### 2.3.1 定义类

*   家具类
    

```
class Furniture():
    def __init__(self, name, area):
        
        self.name = name
        
        self.area = area
```

*   房子类
    

```
class Home():
    def __init__(self, address, area):
        
        self.address = address
        
        self.area = area
        
        self.free_area = area
        
        self.furniture = []

    def __str__(self):
        return f'房子坐落于{self.address}, 占地面积{self.area}, 剩余面积{self.free_area}, 家具有{self.furniture}'

    def add_furniture(self, item):
        """容纳家具"""
        if self.free_area >= item.area:
            self.furniture.append(item.name)
            
            self.free_area -= item.area
        else:
            print('家具太大，剩余面积不足，无法容纳')
```

##### 2.3.2 创建对象并调用实例属性和方法

```
bed = Furniture('双人床', 6)
jia1 = Home('北京', 1200)
print(jia1)

jia1.add_furniture(bed)
print(jia1)

sofa = Furniture('沙发', 10)
jia1.add_furniture(sofa)
print(jia1)

ball = Furniture('篮球场', 1500)
jia1.add_furniture(ball)
print(jia1)
```

六、三大特征
------

*   封装
    
*   将属性和方法书写到类的里面的操作即为封装
    
*   封装可以为属性和方法添加私有权限
    
*   继承
    
*   子类默认继承父类的所有属性和方法
    
*   子类可以重写父类属性和方法
    
*   多态
    
*   传入不同的对象，产生不同的结果
    

七、封装
----

**什么是封装：** 

1.  封装是面向对象编程的一大特点
    
2.  面向对象编程的第一步，将属性和方法封装到一个抽象的类中(为什么说是抽象的，因为类不能直接使用)
    
3.  外界使用类创建对象，然后让对象调用方法
    
4.  对象方法的细节都被封装在类的内部
    

> 总结：类里面不光有属性还有方法。这种将属性通过方法直接在类内部操作的形式叫做封装。这里的封装是把属性封装在类内部。这样对类形成了一种“黑盒”状态，我们不需要知道类内部是什么样的，只要对对象进行操作就可以。在封装时，我们也可以根据自身的需求，设置一定的访问/设置条件（权限），对数据和代码起到一定的保护作用。

八、继承
----

### 1\. 继承的概念

生活中的继承，一般指的是子女继承父辈的财产。

*   拓展 1：经典类或旧式类
    

不由任意内置类型派生出的类，称之为经典类。

*   拓展 2：新式类
    

Python 面向对象的继承指的是多个类之间的所属关系，即子类默认继承父类的所有属性和方法，具体如下：

```
class A(object):
    def __init__(self):
        self.num = 1

    def info_print(self):
        print(self.num)


class B(A):
    pass


result = B()
result.info_print()
```

> 在 Python 中，所有类默认继承 object 类，object 类是顶级类或基类；其他子类叫做派生类。

### 2\. 单继承

所谓单继承意思就是一个子类只能继承一个父类。

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')

        

class Prentice(Master):
    pass



daqiu = Prentice()

print(daqiu.kongfu)

daqiu.make_cake()
```

### 3\. 多继承

所谓多继承意思就是一个类同时继承了多个父类。

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')



class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    pass


daqiu = Prentice()
print(daqiu.kongfu)
daqiu.make_cake()
```

> 注意：当一个类有多个父类的时候，默认使用第一个父类的同名属性和方法。

### 4\. 子类重写父类同名方法和属性

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')



class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


daqiu = Prentice()
print(daqiu.kongfu)
daqiu.make_cake()


print(Prentice.__mro__)
```

> 子类和父类具有同名属性和方法，默认使用子类的同名属性和方法。

### 5\. 子类调用父类的同名方法和属性

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'

    def make_cake(self):
        
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    
    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)


daqiu = Prentice()

daqiu.make_cake()

daqiu.make_master_cake()

daqiu.make_school_cake()

daqiu.make_cake()
```

### 6\. 多层继承

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)



class Tusun(Prentice):
    pass


xiaoqiu = Tusun()

xiaoqiu.make_cake()

xiaoqiu.make_school_cake()

xiaoqiu.make_master_cake()
```

### 7\. super()调用父类方法

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(Master):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')

        
        
        

        
        super().__init__()
        super().make_cake()


class Prentice(School):
    def __init__(self):
        self.kongfu = '[独创煎饼果子技术]'

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    
    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)

    
    def make_old_cake(self):
        
        
        
        
        

        
        
        
        

        
        super().__init__()
        super().make_cake()


daqiu = Prentice()

daqiu.make_old_cake()
```

> 注意：使用 super() 可以自动查找父类。调用顺序遵循 `__mro__` 类属性的顺序。比较适合单继承使用。

### 8\. 私有权限

#### 8.1 定义私有属性和方法

在 Python 中，可以为实例属性和方法设置私有权限，即设置某个实例属性或实例方法不继承给子类。

设置私有权限的方法：在属性名和方法名 前面 加上两个下划线 `__`。

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'
        
        self.__money = 2000000

    
    def __info_print(self):
        print(self.kongfu)
        print(self.__money)

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)



class Tusun(Prentice):
    pass


daqiu = Prentice()




xiaoqiu = Tusun()
```

> 注意：私有属性和私有方法只能在类里面访问和修改。

#### 8.2 获取和修改私有属性值

在 Python 中，一般定义函数名`get_xx`用来获取私有属性，定义`set_xx`用来修改私有属性值。

```
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[现代煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'
        self.__money = 2000000

    
    def get_money(self):
        return self.__money

    
    def set_money(self):
        self.__money = 500

    def __info_print(self):
        print(self.kongfu)
        print(self.__money)

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)



class Tusun(Prentice):
    pass


daqiu = Prentice()

xiaoqiu = Tusun()

print(xiaoqiu.get_money())

xiaoqiu.set_money()
print(xiaoqiu.get_money())
```

### 9\. 总结

*   继承的特点
    
*   子类默认拥有父类的所有属性和方法
    
*   子类重写父类同名方法和属性
    
*   子类调用父类同名方法和属性
    
*   super()方法快速调用父类方法
    
*   私有权限
    
*   不能继承给子类的属性和方法需要添加私有权限
    
*   语法
    

```
class 类名():
  
  __属性名 = 值

  
  def __函数名(self):
    代码
```

九、多态
----

### 1\. 了解多态

多态指的是一类事物有多种形态，（一个抽象类有多个子类，因而多态的概念依赖于继承）。

*   定义：多态是一种使用对象的方式，子类重写父类方法，调用不同子类对象的相同父类方法，可以产生不同的执行结果
    
*   好处：调用灵活，有了多态，更容易编写出通用的代码，做出通用的编程，以适应需求的不断变化！
    
*   实现步骤：
    
*   定义父类，并提供公共方法
    
*   定义子类，并重写父类方法
    
*   传递子类对象给调用者，可以看到不同子类执行效果不同
    

### 2\. 体验多态

```
class Dog(object):
    def work(self):  
        print('指哪打哪...')


class ArmyDog(Dog):  
    def work(self):  
        print('追击敌人...')


class DrugDog(Dog):
    def work(self):
        print('追查毒品...')


class Person(object):
    def work_with_dog(self, dog):  
        dog.work()


ad = ArmyDog()
dd = DrugDog()

daqiu = Person()
daqiu.work_with_dog(ad)
daqiu.work_with_dog(dd)
```

十、类属性和实例属性
----------

### 1\. 类属性

#### 1.1 设置和访问类属性

*   类属性就是 **类对象** 所拥有的属性，它被 **该类的所有实例对象所共有**。
    
*   类属性可以使用 **类对象** 或 **实例对象** 访问。
    

```
class Dog(object):
    tooth = 10


wangcai = Dog()
xiaohei = Dog()

print(Dog.tooth)  
print(wangcai.tooth)  
print(xiaohei.tooth)
```

> 类属性的优点
> 
> *   **记录的某项数据 始终保持一致时**，则定义类属性。
>     
> *   **实例属性** 要求 **每个对象** 为其 **单独开辟一份内存空间** 来记录数据，而 **类属性** 为全类所共有 ，**仅占用一份内存**，**更加节省内存空间**。
>     

#### 1.2 修改类属性

类属性只能通过类对象修改，不能通过实例对象修改，如果通过实例对象修改类属性，表示的是创建了一个实例属性。

```
class Dog(object):
    tooth = 10


wangcai = Dog()
xiaohei = Dog()


Dog.tooth = 12
print(Dog.tooth)  
print(wangcai.tooth)  
print(xiaohei.tooth)  


wangcai.tooth = 20
print(Dog.tooth)  
print(wangcai.tooth)  
print(xiaohei.tooth)
```

### 2\. 实例属性

```
class Dog(object):
    def __init__(self):
        self.age = 5

    def info_print(self):
        print(self.age)


wangcai = Dog()
print(wangcai.age)  

wangcai.info_print()
```

十一、类方法和静态方法
-----------

### 1\. 类方法

#### 1.1 类方法特点

需要用装饰器`@classmethod`来标识其为类方法，对于类方法，**第一个参数必须是类对象**，一般以`cls`作为第一个参数。

#### 1.2 类方法特点

*   当方法中 **需要使用类对象** (如访问私有类属性等)时，定义类方法
    
*   类方法一般和类属性配合使用
    

```
class Dog(object):
    __tooth = 10

    @classmethod
    def get_tooth(cls):
        return cls.__tooth


wangcai = Dog()
result = wangcai.get_tooth()
print(result)
```

### 2\. 静态方法

#### 2.1 静态方法特点

*   需要通过装饰器`@staticmethod`来进行修饰，**静态方法既不需要传递类对象也不需要传递实例对象（形参没有 self/cls）**。
    
*   静态方法 也能够通过 **实例对象** 和 **类对象** 去访问。
    

#### 2.2 静态方法使用场景

*   当方法中 **既不需要使用实例对象**(如实例对象，实例属性)，**也不需要使用类对象** (如类属性、类方法、创建实例等)时，定义静态方法
    
*   **取消不需要的参数传递**，有利于 **减少不必要的内存占用和性能消耗**
    

```
class Dog(object):
    @staticmethod
    def info_print():
        print('这是一个狗类，用于创建狗实例....')


wangcai = Dog()

wangcai.info_print()
Dog.info_print()
```