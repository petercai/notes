# shell 语法介绍
变量定义
----

首先看下shell中对变量的定义，其中分为环境变量和局部变量。

环境变量在子shell进程中是可见的，可以通过export 关键字进行定义，如下所示，

```shell
[root@localhost ~]# export VAR=value

```

局部变量是指在某个shell中生效的变量，这个变量在其他shell中是无效。

### 变量的定义

变量的定义可以通过如下方式进行定义，

变量名=变量值,

```shell
[root@localhost ~]# name=john 

```

> 注意点一：变量名和变量值之间用等号紧紧相连，之间没有任何空格 注意点二：当变量中有空格时必须用引号(单引号，双引号都可以)括起，否则会出现错误

### 变量的引用

定义了变量，那么如何对其进行引用呢？可以通过如下方式对变量进行引用，

在变量前面加上$ 符号即可。

```shell
(base) ➜  ~ name=lanpangzi
(base) ➜  ~ echo $name
lanpangzi

```

更标准点的写法是用${}将变量名括起来。

```shell
(base) ➜  ~ echo ${name}
lanpangzi

```

#### 位置参数

除了通过${变量名} 方式引用变量，还可以通过 $数字 方式获取shell脚本的参数，$0 代表第一个参数，$1 代表第二个参数，依次类推。 另外 $# 代表参数的个数， $* 或者 $@代表所有参数，例如我写一个脚本 输出这些变量。

脚本如下，

```shell
!/bin/sh
echo "第一个参数: $0"
echo "第二个参数: $1"
echo "所有参数: $@"
echo "参数个数: $#"

```

运行这个脚本

```shell
(base) ➜  ~ sh print.sh wudi lanoangzi
print.sh: line 1: !/bin/sh: No such file or directory
第一个参数: print.sh
第二个参数: wudi
所有参数: wudi lanoangzi
参数个数: 2

```

> 注意下shell脚本中单引号和双引号的区别，如果要让输出的语句中引用变量，那么要用双引号。

### 数组的语法

另外，在shell脚本中还有个经常用到的类型，数组，与其他语言不同的是，shell脚本中的数据只支持一维数组。

数组的定义方式如下，

declare 关键字定义数组，其中元素用()括起来，并且元素之间用空格隔开。

```shell
declare arr1=(元素1 元素2)

```

数组中的元素引用方式如下,

```shell
echo ${数组名[索引号]}

```

比较特殊的是可以通过 ${数组名\[@\]} 或者 ${数组名\[*\]} 获取数组中的 元素, 可以通过 ${#数组名\[@\]} 或者 ${#数组名\[*\]} 获取数组的长度。

对数组中元素替换和新增数组元素可以按如下操作,

对指定位置的变量进行替换

```shell
数组名[索引值]=30

```

假设数组名是arr，对数组末尾进行元素添加，

```shell
arr[${

```

循环以及判断语句
--------

看了变量的定义，我们再来看看shell脚本中的跳转指令，在学跳转指令前还需要对shell中如何对表达式获取表达式的结果有所了解，因为一般跳转指令都是条件跳转，像if else之类的语句，总有个判断条件。

### 表达式结果

在shell中可以通过$? 获取上一个shell语句的执行结果，shell命令中规定0才是命令正确执行后的返回结果，其余结果都是不正确的。

让表达式执行比较逻辑的方式有两种，

1, 第一种是通过test expression 的方式，test后面跟表达式，如下所示,比较了两个数字是否相等，返回1说明相等。

```shell
(base) ➜  ~ test 1 -eq 2
(base) ➜  ~ echo $?
1

```

2, 第二种方式是使用\[\] 把表达式括起来，这种方式注意\[\] 内变量需要与括号相隔一个空格才行。

```shell
(base) ➜  ~ [ 1 -eq 2 ]
(base) ➜  ~ echo $?
1

```

### 字符串比较

接着来看下shell中如何对字符串进行比较，

![](_assets/238dd3f923b349a2aaf15e2c80531ba3~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

字符串的比较是可以用\> < 这种符号的，数字则不同。

### 数字比较

![](_assets/70572d8d40f740a8852f9888dde4c4f9~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### 文件相关的判断

除了数字和字符串的比较，我们平时还经常会用到对文件的判断，比如判断文件是否存在等，如下是对文件相关操作的判断。

![](_assets/31ba95a2d3df4985b1d0e6b07e9872c6~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### if 语句

了解了判断语句就可以看看跳转指令的语法，首先我们看下if语句在shell中是如何写的。

```shell
if expression; then 
		command 
fi

```

通过if then fi三个关键字组成了最简单的if语句，其中expression则是前面讲到的判断语句的表达式，如下，执行这个脚本将会输出，123 和456两行数据。

```shell
!/bin/sh
num=1
if test $num -eq 1 ;then
        echo 123
fi
if [ $num -eq 1 ] ;then
        echo 456
fi

```

if else 语句也类似，它的语法结构如下，

```shell
if expression; then 
	command 
else 
	command 
fi

```

### for while循环语句

#### for 语法结构

```shell
for VARIABLE in (list) 
do 
	command 
done

```

for语句可以遍历一个列表然后对其中每一个元素进行遍历。上述语法中，list既可以是变量也可以是固定数组表达式，也可以命令输出。

案例1，数组变量 循环

```shell
!/bin/sh
arr="1 2 3 4"
for num in ${arr}
do
   echo $num
done

```

案例2，固定数组表达式循环

```shell
!/bin/sh
for num in 1 2 3 4
do
   echo $num
done

```

案例3，命令输出结果 循环

```shell
!/bin/sh
for num in $(ls)
do
   echo $num
done

```

#### while 语法结构

```shell
while expression 
do 
	command 
done

```

表达式的语法也和之前if 语句那里讲的语法结构类似，这里就不再展开了。

