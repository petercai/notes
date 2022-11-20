# 深入浅出Python——Python高级语法之函数_Python_何极光_InfoQ写作社区
前言：本博文主要讲解 Python 函数的用法，属于 Python 高级语法。基础语法见：[https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a](https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a)，更多内容请访问博主的主页，谢谢！

一. 函数的作用
--------

函数就是将一段具有独立功能的代码块 整合到一个整体并命名，在需要的位置调用这个名称即可完成对应的需求。

> 函数在开发过程中，可以更高效的实现代码重用。

二. 函数的使用步骤
----------

### 1\. 定义函数

```
def 函数名(参数):
    代码1
    代码2
    ......
```

### 2\. 调用函数

注意：

1.  不同的需求，参数可有可无。
    
2.  在 Python 中，函数必须先定义后使用。
    

\==一定是先定义函数，后调用函数。==

三. 函数的参数作用
----------

思考：完成需求如下：一个函数完成两个数 1 和 2 的加法运算，如何书写程序？

```
def add_num1():
    result = 1 + 2
    print(result)



add_num1()
```

思考：上述 add\_num1 函数只能完成数字 1 和 2 的加法运算，如果想要这个函数变得更灵活，可以计算任何用户指定的两个数字的和，如何书写程序？

分析：用户要在调用函数的时候指定具体数字，那么在定义函数的时候就需要接收用户指定的数字。函数调用时候指定的数字和定义函数时候接收的数字即是函数的参数。

```
def add_num2(a, b):
    result = a + b
    print(result)



add_num2(10, 20)
```

**函数的参数：函数调用的时候可以传入真实数据，增大函数的使用的灵活性。** 

*   形参：函数定义时书写的参数(非真实数据)
    
*   实参：函数调用时书写的参数(真实数据)
    

四. 函数的返回值作用
-----------

例如：我们去超市购物，比如买烟，给钱之后，是不是售货员会返回给我们烟这个商品，在函数中，如果需要返回结果给用户需要使用函数返回值。

```
def buy():
    return '烟'


goods = buy()
print(goods)
```

需求：制作一个计算器，计算任意两数字之和，并保存结果。

```
def sum_num(a, b):
    return a + b



result = sum_num(1, 2)
print(result)
```

**函数的返回值**

*   作用：函数调用后，返回需要的计算结果
    
*   写法
    

五. 函数的说明文档
----------

思考：定义一个函数后，程序员如何书写程序能够快速提示这个函数的作用？

答：注释

思考：如果代码多，我们是不是需要在很多代码中找到这个函数定义的位置才能看到注释？如果想更方便的查看函数的作用怎么办？

答：函数的说明文档

> 函数的说明文档也叫函数的文档说明。

### 1\. 语法

*   定义函数的说明文档
    

```
def 函数名(参数):
    """ 说明文档的位置 """
    代码
    ......
```

*   查看函数的说明文档
    

### 2\. 快速体验

```
def sum_num(a, b):
    """ 求和函数 """
    return a + b


help(sum_num)
```

![](https://static001.geekbang.org/infoq/2d/2d50279af7ea79c34ca880be1aaa0d8b.png)

**函数的说明文档**

*   作用：保存函数解释说明的信息
    
*   写法
    

```
def 函数名():
    """ 函数说明文档 """
```

六. 函数嵌套调用
---------

所谓函数嵌套调用指的是==一个函数里面又调用了另外一个函数==。

*   示例
    

```
def testB():
    print('---- testB start----')
    print('这里是testB函数执行的代码...(省略)...')
    print('---- testB end----')

def testA():
    print('---- testA start----')
    testB()
    print('---- testA end----')

testA()
```

*   效果
    

![](https://static001.geekbang.org/infoq/fc/fca0adbc5a658de9d0af17023bb96a7e.png)

*   执行流程
    
*   ![](https://img-blog.csdnimg.cn/20200727075957983.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MDM0Mzg0,size_16,color_FFFFFF,t_70)
    

> 如果函数 A 中，调用了另外一个函数 B，那么先把函数 B 中的任务都执行完毕之后才会回到上次 函数 A 执行的位置。

七. 函数应用
-------

### 1\. 打印图形

1.  打印一条横线
    

```
def print_line():
    print('-' * 20)


print_line()
```

![](https://static001.geekbang.org/infoq/ac/ac49995f5e410ff4776c0a5f556b00ee.png)

2.  打印多条横线
    

```
def print_line():
    print('-' * 20)


def print_lines(num):
    i = 0
    while i < num:
        print_line()
        i += 1


print_lines(5)
```

![](https://img-blog.csdnimg.cn/20200727080308284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MDM0Mzg0,size_16,color_FFFFFF,t_70)

### 2\. 函数计算

1.  求三个数之和
    

```
def sum_num(a, b, c):
    return a + b + c


result = sum_num(1, 2, 3)
print(result)
```

2.  求三个数平均值
    

```
def average_num(a, b, c):
    sumResult = sum_num(a, b, c)
    return sumResult / 3

result = average_num(1, 2, 3)
print(result)
```

八. 变量作用域
--------

变量作用域指的是变量生效的范围，主要分为两类：==局部变量==和==全局变量==。

*   局部变量
    

所谓局部变量是定义在函数体内部的变量，即只在函数体内部生效。

```
def testA():
    a = 100

    print(a)


testA()  
print(a)
```

> 变量 a 是定义在`testA`函数内部的变量，在函数外部访问则立即报错。

局部变量的作用：在函数体内部，临时保存数据，即当函数调用完成后，则销毁局部变量。

全局变量：指的是在函数体内、外都能生效的变量。

> 思考：如果有一个数据，在函数 A 和函数 B 中都要使用，该怎么办？
> 
> 答：将这个数据存储在一个全局变量里面。

```
a = 100


def testA():
    print(a)  


def testB():
    print(a)  


testA()  
testB()
```

思考：`testB`函数需求修改变量 a 的值为 200，如何修改程序？

```
a = 100


def testA():
    print(a)


def testB():
    a = 200
    print(a)


testA()  
testB()  
print(f'全局变量a = {a}')
```

思考：在`testB`函数内部的`a = 200`中的变量 a 是在修改全局变量`a`吗？

答：不是。观察上述代码发现，15 行得到 a 的数据是 100，仍然是定义全局变量 a 时候的值，而没有返回

`testB`函数内部的 200。综上：`testB`函数内部的`a = 200`是定义了一个局部变量。

思考：如何在函数体内部修改全局变量？

```
a = 100


def testA():
    print(a)


def testB():
    
    global a
    a = 200
    print(a)


testA()  
testB()  
print(f'全局变量a = {a}')
```

九. 多函数程序执行流程
------------

一般在实际开发过程中，一个程序往往由多个函数（后面知识中会讲解类）组成，并且多个函数共享某些数据，如下所示：

*   共用全局变量
    

```
glo_num = 0


def test1():
    global glo_num
    
    glo_num = 100


def test2():
    
    print(glo_num)
    


test1()

test2()
```

*   返回值作为参数传递
    

```
def test1():
    return 50


def test2(num):
    print(num)



result = test1()



test2(result)
```

十. 函数的返回值
---------

思考：如果一个函数如些两个 return (如下所示)，程序如何执行？

```
def return_num():
    return 1
    return 2


result = return_num()
print(result)
```

答：只执行了第一个 return，原因是因为 return 可以退出当前函数，导致 return 下方的代码不执行。

思考：如果一个函数要有多个返回值，该如何书写代码？

```
def return_num():
    return 1, 2


result = return_num()
print(result)
```

> 注意：
> 
> 1.  `return a, b`写法，返回多个数据的时候，默认是元组类型。
>     
> 2.  return 后面可以连接列表、元组或字典，以返回多个值。
>     

十一. 函数的参数
---------

### 1\. 位置参数

位置参数：调用函数时根据函数定义的参数位置来传递参数。

```
def user_info(name, age, gender):
    print(f'您的名字是{name}, 年龄是{age}, 性别是{gender}')


user_info('TOM', 20, '男')
```

> 注意：传递和定义参数的顺序及个数必须一致。

### 2\. 关键字参数

函数调用，通过“键=值”形式加以指定。可以让函数更加清晰、容易使用，同时也清除了参数的顺序需求。

```
def user_info(name, age, gender):
    print(f'您的名字是{name}, 年龄是{age}, 性别是{gender}')


user_info('Rose', age=20, gender='女')
user_info('小明', gender='男', age=16)
```

注意：**函数调用时，如果有位置参数时，位置参数必须在关键字参数的前面，但关键字参数之间不存在先后顺序。** 

### 3\. 缺省参数

缺省参数也叫默认参数，用于定义函数，为参数提供默认值，调用函数时可不传该默认参数的值（注意：所有位置参数必须出现在默认参数前，包括函数定义和调用）。

```
def user_info(name, age, gender='男'):
    print(f'您的名字是{name}, 年龄是{age}, 性别是{gender}')


user_info('TOM', 20)
user_info('Rose', 18, '女')
```

> 注意：函数调用时，如果为缺省参数传值则修改默认参数值；否则使用这个默认值。

### 4\. 不定长参数

不定长参数也叫可变参数。用于不确定调用的时候会传递多少个参数(不传参也可以)的场景。此时，可用包裹(packing)位置参数，或者包裹关键字参数，来进行参数传递，会显得非常方便。

*   包裹位置传递
    

```
def user_info(*args):
    print(args)



user_info('TOM')

user_info('TOM', 18)
```

> 注意：传进的所有参数都会被 args 变量收集，它会根据传进参数的位置合并为一个元组(tuple)，args 是元组类型，这就是包裹位置传递。

*   包裹关键字传递
    

```
def user_info(**kwargs):
    print(kwargs)



user_info(name='TOM', age=18, id=110)
```

> 综上：无论是包裹位置传递还是包裹关键字传递，都是一个组包的过程。

十二. 拆包和交换变量值
------------

### 1\. 拆包

*   拆包：元组
    

```
def return_num():
    return 100, 200


num1, num2 = return_num()
print(num1)  
print(num2)
```

*   拆包：字典
    

```
dict1 = {'name': 'TOM', 'age': 18}
a, b = dict1


print(a)  
print(b)  

print(dict1[a])  
print(dict1[b])
```

### 2\. 交换变量值

需求：有变量`a = 10`和`b = 20`，交换两个变量的值。

*   方法一
    

借助第三变量存储数据。

```
c = 0


c = a


a = b


b = c

print(a)  
print(b)
```

*   方法二
    

```
a, b = 1, 2
a, b = b, a
print(a)  
print(b)
```

十三. 引用
------

### 1\. 了解引用

在 python 中，值是靠引用来传递来的。

**我们可以用**`**id()**`**来判断两个变量是否为同一个值的引用。**  我们可以将 id 值理解为那块内存的地址标识。

```
a = 1
b = a

print(b)  

print(id(a))  
print(id(b))  

a = 2
print(b)  

print(id(a))  
print(id(b))  



aa = [10, 20]
bb = aa

print(id(aa))  
print(id(bb))  


aa.append(30)
print(bb)  

print(id(aa))  
print(id(bb))
```

### 2\. 引用当做实参

代码如下：

```
def test1(a):
    print(a)
    print(id(a))

    a += a

    print(a)
    print(id(a))



b = 100
test1(b)


c = [11, 22]
test1(c)
```

![](https://img-blog.csdnimg.cn/20200727170928179.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MDM0Mzg0,size_16,color_FFFFFF,t_70)

十四. 可变和不可变类型
------------

所谓可变类型与不可变类型是指：数据能够直接进行修改，如果能直接修改那么就是可变，否则是不可变。

*   可变类型
    
*   列表
    
*   字典
    
*   集合
    
*   不可变类型
    
*   整型
    
*   浮点型
    
*   布尔
    
*   字符串
    
*   元组
    

十五. 总结
------

*   变量作用域
    
*   全局：函数体内外都能生效
    
*   局部：当前函数体内部生效
    
*   函数多返回值写法
    

*   函数的参数
    
*   位置参数
    
*   形参和实参的个数和书写顺序必须一致
    
*   关键字参数
    
*   写法： `key=value`  
    
*   特点：形参和实参的书写顺序可以不一致；关键字参数必须书写在位置参数的后面
    
*   缺省参数
    
*   缺省参数就是默认参数
    
*   写法：`key=vlaue`  
    
*   不定长位置参数
    
*   收集所有位置参数，返回一个元组
    
*   不定长关键字参数
    
*   收集所有关键字参数，返回一个字典
    

> 引用：Python 中，数据的传递都是通过引用。

\==友情链接：==

1.  [深究Python中的递归【建议收藏】](https://xie.infoq.cn/link?target=https%3A%2F%2Fluckylifes.blog.csdn.net%2Farticle%2Fdetails%2F107682376)  
    
2.  [浅谈Python匿名函数（lambda表达式）](https://xie.infoq.cn/link?target=https%3A%2F%2Fluckylifes.blog.csdn.net%2Farticle%2Fdetails%2F107697395)  
    
3.  [浅谈Python高阶函数](https://xie.infoq.cn/link?target=https%3A%2F%2Fluckylifes.blog.csdn.net%2Farticle%2Fdetails%2F107698729)  
    
4.  [Python学员管理系统【函数实现】](https://xie.infoq.cn/link?target=https%3A%2F%2Fluckylifes.blog.csdn.net%2Farticle%2Fdetails%2F107681516)  
    
5.  [深入浅出Python——Python高级语法之文件操作](https://xie.infoq.cn/link?target=https%3A%2F%2Fluckylifes.blog.csdn.net%2Farticle%2Fdetails%2F107699888)