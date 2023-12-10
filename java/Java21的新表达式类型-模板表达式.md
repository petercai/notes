# Java21的新表达式类型-模板表达式


在Java编程语言里，模板表达式这个新玩意儿给大家带来了全新的表达式类型。这篇文章会给大家详细讲解模板表达式是什么，如何用它，以及通过具体的代码示例来展示它的优点。

模板表达式是Java21里新出的一个表达式类型。它可以帮助我们用编程的方式连接字符串，这样开发人员就能够安全有效地拼接字符串了。而且，不只是拼接字符串，模板表达式还能将结构化的文本转化成任何东西。

模板表达式看起来类似于一个用前缀封闭的字符串。以下代码示例演示了Java中的模板表达式：

```ini
String name = "superqi"
String info = STR."My name is {name}"
System.out.println(info)

//result ：My name is superqi

```

在这个例子里面，我们用一个带有前缀“STR.”的模板表达式。这个模板表达式就像这样组成的：

1.  **模板处理器，比如“STR”，**
2.  **点号（就是那个U+002E），跟其他表达式一样，**
3.  **还有一段带有点号的文本：“My name is {name}”。** 

等到了运行的时候，这个模板表达式就会把里面的那些值拼起来，生成一个结果。

所以呢，模板处理器的结果和模板表达式的评估结果通常都是一个字符串，但也不一定就是这样。

STR模板处理器是Java里一个用来演示如何使用模板表达式的例子。当然，除了STR之外，还有Freemarker、Thymeleaf等其他可用的模板处理器。

这些模板处理器就像魔法一样，可以让开发人员根据一些特定的规则和逻辑动态地生成文本或其他类型的输出。

比如，STR这个模板处理器，它能在Java平台上工作。它的主要任务是执行字符串插值，意思就是用表达式的值去替换文本中的那些嵌入式表达式。

当我们用STR处理器评估那个模板表达式的时候，最后会得到一个字符串，就像“My name is superqi”这样的结果。

以下是使用 STR 模板处理器的更多示例：

> 示例 1：嵌入式表达式可以是字符串

```ini
String firstName = "superqi"
String lastName = "laoqi"
String fullName = STR.format("{0}, {1}", lastName, firstName)
// fullName: "superqi laoqi"
String sortName = STR.format("{0}, {1}", firstName, lastName)
// sortName: "laoqi, superqi"

```

> 示例 2：嵌入式表达式可以执行算术运算

```ini
int x = 10, y = 20
String s = STR.format("{0} + {1} = {2}", x, y, x + y)
// s: "10 + 20 = 30"


```

> 示例 3：嵌入式表达式可以调用方法和访问字段

```ini
String s = STR.format("You have a {0} waiting for you!", getOfferType())
// s: "You have a gift waiting for you!"
String t = STR.format("Access at {0} {1} from {2}", req.date, req.time, req.ipAddress)
// t: "Access at 2023-10-28 15:34 from 127.0.0.1"

```

用双引号在模板表达式里写，就可以不用点（.）也能转义嵌套表达式。这样一来，嵌套表达式在模板中就像在外部一样呈现，使得字符串连接后转为模板表达式更加简单。

例如：

```ini
String filePath = "tmp.dat"
File file = new File(filePath)
String old = "The file " + filePath + " " + (file.exists() ? "does" : "does not") + " exist"
String msg = STR."The file {filePath} {file.exists() ? "does" : "does not"} exist"

```

允许嵌入式表达式在源文件中跨多行意味着嵌入式表达式的值将与字符串中的结果进行连接，而不添加换行符。

```less
String time = STR."The time is {
    
    DateTimeFormatter
      .ofPattern("HH:mm:ss")
      .format(LocalTime.now())
} right now";


```

在字符串模板表达式中使用多个嵌入表达式没有限制。与方法调用表达式一样，嵌入式表达式从左到右依次求值。

```ini
int index = 0
String data = STR."{index++}, {index++}, {index++}, {index++}"
// data: "0, 1, 2, 3"

```

任何 Java 表达式都可以嵌入。不过，在模板表达式中保持类型安全的同时，不建议使用需要简单的复杂表达式。

Java 的基本模板处理器是 STR 和 FMT。STR 执行字符串插值，而 FMT 则将其扩展到字符串插值和自定义格式选项。

```ini
String formatted = FMT."The price is: {price, number, currency}"
// formatted: "The price is: $50.00"

```

FMT模板处理器可让你拥有更多、更灵活的格式化选择，比如货币、百分比、小数点、指数和布尔值等。这能让你轻松地把数据转化成字符串形式。

Java中的模板表达式优先考虑安全性。模板表达式是调用模板处理器的process方法的简写。这种设计可以防止不正确的字符串插值通过程序传播，并确保准确和安全的插值。

**模板处理器API**

开发人员可以创建自己的模板处理器。自定义模板处理器可以返回任何类型的对象，甚至抛出检查异常。实现StringTemplate。处理器接口允许开发人员处理自定义用例。

模板表达式简化了数据库查询的安全创建和执行。QueryBuilder示例演示了如何使用模板处理程序来防止SQL注入漏洞。

通过将模板映射到资源包，模板处理器可以更容易地处理本地化任务。这种方法使开发人员能够创建模板处理器，自动从资源包中获取本地化字符串，从而使本地化工作更容易、更易于管理。

总之，Java中的模板表达式为安全高效地处理字符串插值和文本生成过程提供了强大的工具。像STR和FMT这样的模板处理器使开发人员能够轻松地创建动态的、本地化的和安全的字符串。创建自定义模板处理器的能力为该特性增加了灵活性和可扩展性，使其成为Java工具箱中用于处理文本和数据的重要补充。

* * *

如果各位觉得老七的文章还不错的话，麻烦大家动动小手，

**点赞、关注、转发**走一波！！

有任何问题可以评论区留言或者私信我，我必将**知无不言言无不尽**!