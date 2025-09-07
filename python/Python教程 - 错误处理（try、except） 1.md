# Python教程 - 错误处理（try、except）
在执行Python程序时，我们可能会遇到“非预期错误”的情况。如果没有妥善处理这些错误，可能会导致整个程序崩溃并停止运行。因此，通过“异常处理”机制，我们可以在发生错误时采取相应的行动。这不仅能够保护整个程序的流程，还能够定位问题出现的位置，从而快速进行修正。

使用`try`和`except`
----------------

下面的例子在执行后，会发生`TypeError`错误（因为输入的是文字，文字无法与数字相加），由于错误发生，导致程序停止，后续程序无法正常执行。

```python
a = input('输入数字：')
print(a + 1)  
print('hello')  


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/630cc24a94124773b573767bb989d5a7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=389&s=89013&e=png&b=fcfcfc)

为了避免程序因错误而停止，我们可以使用`try`和`except`进行保护（或测试）。当`try`区域内的程序发生错误时，就会执行`except`里的内容。如果`try`的程序没有错误，就不会执行`except`的内容。当程序修改成下面的样子，就会顺利打印出后面的hello。

```python
try:  
    a = input('输入数字：')
    print(a + 1)
except:  
    print('发生错误')
print('hello')


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/601e966bb33241caac2d075d349b4da8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=674&h=366&s=40648&e=png&b=f9f9f9)

加入`pass`略过
----------

在编写`try...except`时，有时会遇到“不想做任何动作”的情况（连`print`都不想使用）。这时可以使用`pass`语句来略过（什么也不做）。以下述程序为例，当发生错误时，进入`except`后就会自动忽略并跳过。

```python
try:  
    a = input('输入数字：')
    print(a + 1)
except:  
    pass  
print('hello')


```

`except`的错误信息
-------------

只要程序发生错误，控制台中都会出现对应的错误信息。下面列出常见的几种错误信息：

*   `NameError`：使用未定义的对象
    
*   `IndexError`：索引值超出了序列的大小
    
*   `TypeError`：数据类型错误
    
*   `SyntaxError`：Python语法规则错误
    
*   `ValueError`：传入值错误
    
*   `KeyboardInterrupt`：程序被手动强制终止
    
*   `AssertionError`：程序断言后面的条件不成立
    
*   `KeyError`：键发生错误
    
*   `ZeroDivisionError`：除以0
    
*   `AttributeError`：使用不存在的属性
    
*   `IndentationError`：Python语法错误（没有对齐）
    
*   `IOError`：输入/输出异常
    
*   `UnboundLocalError`：局部变量和全局变量发生重复或错误
    

下面的程序执行时，因为变量`a`还未被定义，所以会进入`except NameError`的区域，打印出“使用未定义的对象”。

```python
try:
    print(a)
except TypeError:
    print('类型发生错误')
except NameError:
    print('使用未定义的对象')
print('hello')




```

如果不知道错误的类型，只想打印出错误信息，除了单纯用`except`，也可以使用`except Exception`，将所有的异常信息都包含在内。

```python
try:
    print(1/0)
except TypeError:
    print('类型发生错误')
except NameError:
    print('使用未定义的对象')
except Exception:
    print('不知道怎么回事，反正发生错误了')
print('hello')




```

如果不知道错误的类型，只想打印出错误信息，除了单纯用`except`，也可以使用`except Exception`，将所有的异常信息都包含在内。

```php
try:
    a = 1
    b = '1'
    print(a+b)
except Exception as e:
    print(e)
    



```

raise 和 assert
--------------

在执行`try`的过程中，如果遇到需要“强制中断”的情况，可以使用`raise`强制中断。

```python
try:
    a = int(input('输入 0～9：'))
    if a>9:       
        raise       
    print(a)
except :
    print('有错误喔～')   


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66cb4aac0a1d4c6dbdc463bad56d66d7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=800&h=376&s=40866&e=png&b=f9f9f9)

`raise`后面可以加上错误信息，错误信息可以包含要显示的消息。例如，下面的示例强制停止时报告`ValueError`信息，接着使用`except`隔离错误信息，就能展示真实的错误状况。

```python
try:
    a = int(input('输入 0～9：'))
    if a > 10:
        raise ValueError('数字不在范围内')
    print(a)
except ValueError as msg:  
    print(msg)
except:  
    print('有错误哦～')
print('继续执行')


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a456d647699f4c55bfbf9747d4f8ef7c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=838&h=514&s=75051&e=png&b=f9f9f9)

使用`assert`中断的方法为`assert False, '错误信息'`，用法和`raise`类似，执行后就会中断程序，并将错误信息提供给`except`显示。下面的程序如果输入123，会执行`AssertionError`里的程序，如果输入abc则会执行`except`里的程序。

```python
try:
    a = int(input('输入 0～9：'))
    if a > 10:
        assert False, '数字不在范围内'
    print(a)
except AssertionError as msg:
    print(msg)
except:
    print('有错误哦～')
print('继续执行')


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64c4ad89cc62401f980875714ec7f328~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=426&s=159332&e=png&b=f5f5f5)

加入 else 和 finally
-----------------

在`except`结束后，可以加入`else`或`finally`两个额外的判断，`else`表示完全没有错误，就会执行该区域的程序，`finally`则不论程序对错，都会执行该区域的程序。

```python
try:
    a = int(input('输入 0～9：'))
    if a > 10:
        raise
    print(a)
except:
    print('有错误哦～')
else:  
    print('没有错！继续执行！')
finally:  
    print('管他有没有错，继续执行！')


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6767bb0b3e224958b6f045ded89cd949~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=419&s=139665&e=png&b=f4f3f3)