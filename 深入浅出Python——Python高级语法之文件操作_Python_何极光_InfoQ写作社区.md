# 深入浅出Python——Python高级语法之文件操作_Python_何极光_InfoQ写作社区
前言：本博文主要讲解 Python 文件操作的写法，属于 Python 高级语法。基础语法见：[https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a](https://xie.infoq.cn/article/a1b348be0074e38f1b307f50a)，更多内容请访问博主的主页，谢谢！

一、文件操作的作用
---------

思考：文件操作包含什么？

答：打开、关闭、读、写、复制....

思考：文件操作的的作用是什么？

答：读取内容、写入内容、备份内容......

> 总结：文件操作的作用就是==把一些内容(数据)存储存放起来，可以让程序下一次执行的时候直接使用，而不必重新制作一份，省时省力==。

二、文件的基本操作
---------

### 1\. 文件操作步骤

1.  打开文件
    
2.  读写等操作
    
3.  关闭文件
    

> 注意：可以只打开和关闭文件，不进行任何读写操作。

#### 1.1 打开

在 python，使用 open 函数，可以打开一个已经存在的文件，或者创建一个新文件，语法如下：

name：是要打开的目标文件名的字符串(可以包含文件所在的具体路径)。

mode：设置打开文件的模式(访问模式)：只读、写入、追加等。

##### 1.1.1 打开文件模式

##### 1.1.2 快速体验

```
f = open('test.txt', 'w')
```

> 注意：此时的`f`是`open`函数的文件对象。

#### 2.1 文件对象方法

##### 1.2.1 写

*   语法
    

*   体验
    

```
f = open('test.txt', 'w')


f.write('hello world')


f.close()
```

> 注意：
> 
> 1.  `w` 和`a`模式：如果文件不存在则创建该文件；如果文件存在，`w`模式先清空再写入，`a`模式直接末尾追加。
>     
> 2.  `r`模式：如果文件不存在则报错。
>     

##### 1.2.2 读

*   read()
    

> num 表示要从文件中读取的数据的长度（单位是字节），如果没有传入 num，那么就表示读取文件中所有的数据。

*   readlines()
    

readlines 可以按照行的方式把整个文件中的内容进行一次性读取，并且返回的是一个列表，其中每一行的数据为一个元素。

```
f = open('test.txt')
content = f.readlines()


print(content)


f.close()
```

*   readline()
    

readline()一次读取一行内容。

```
f = open('test.txt')

content = f.readline()
print(f'第一行：{content}')

content = f.readline()
print(f'第二行：{content}')


f.close()
```

##### 1.2.3 seek()

作用：用来移动文件指针。

语法如下：

> 起始位置：
> 
> *   0：文件开头
>     
> *   1：当前位置
>     
> *   2：文件结尾
>     

#### 3.1 关闭

三、文件备份
------

需求：用户输入当前目录下任意文件名，程序完成对该文件的备份功能(备份文件名为 xx\[备份\]后缀，例如：test\[备份\].txt)。

### 1\. 步骤

1.  接收用户输入的文件名
    
2.  规划备份文件名
    
3.  备份文件写入数据
    

### 2\. 代码实现

1.  接收用户输入目标文件名
    

```
old_name = input('请输入您要备份的文件名：')
```

2.  规划备份文件名
    
3.  2.1 提取目标文件后缀
    
4.  2.2 组织备份的文件名，xx\[备份\]后缀
    

```
index = old_name.rfind('.')






new_name = old_name[:index] + '[备份]' + old_name[index:]
```

3.  备份文件写入数据
    
4.  3.1 打开源文件 和 备份文件
    
5.  3.2 将源文件数据写入备份文件
    
6.  3.3 关闭文件
    

```
old_f = open(old_name, 'rb')
new_f = open(new_name, 'wb')


while True:
    con = old_f.read(1024)
    if len(con) == 0:
        break
    new_f.write(con)


old_f.close()
new_f.close()
```

### 3\. 思考

如果用户输入`.txt`，这是一个无效文件，程序如何更改才能限制只有有效的文件名才能备份？

答：添加条件判断即可。

```
old_name = input('请输入您要备份的文件名：')

index = old_name.rfind('.')


if index > 0:
    postfix = old_name[index:]

new_name = old_name[:index] + '[备份]' + postfix

old_f = open(old_name, 'rb')
new_f = open(new_name, 'wb')

while True:
    con = old_f.read(1024)
    if len(con) == 0:
        break
    new_f.write(con)

old_f.close()
new_f.close()
```

四、文件和文件夹的操作
-----------

在 Python 中文件和文件夹的操作要借助 os 模块里面的相关功能，具体步骤如下：

1.  导入 os 模块
    

2.  使用`os`模块相关功能
    

### 1\. 文件重命名

### 2\. 删除文件

### 3\. 创建文件夹

### 4\. 删除文件夹

### 5\. 获取当前目录

### 6\. 改变默认目录

### 7\. 获取目录列表

五、应用案例
------

需求：批量修改文件名，既可添加指定字符串，又能删除指定字符串。

*   步骤
    

1.  设置添加删除字符串的的标识
    
2.  获取指定目录的所有文件
    
3.  将原有文件名添加/删除指定字符串，构造新名字
    
4.  os.rename()重命名
    

*   代码
    

```
import os


flag = 1


dir_name = './'


file_list = os.listdir(dir_name)




for name in file_list:

    
    if flag == 1:
        new_name = 'Python-' + name
    
    elif flag == 2:
        num = len('Python-')
        new_name = name[num:]

    
    print(new_name)
    
    
    os.rename(dir_name+name, dir_name+new_name)
```

六、总结
----

*   文件操作步骤
    
*   打开
    
*   操作
    
*   读
    
*   写
    
*   seek()
    
*   关闭
    
*   主访问模式
    
*   w：写，文件不存在则新建该文件
    
*   r：读，文件不存在则报错
    
*   a：追加
    
*   文件和文件夹操作
    
*   重命名：`os.rename()`  
    
*   获取当前目录：`os.getcwd()`  
    
*   获取目录列表：`os.listdir()`