# Java8 Stream 的源码
小伙伴们好呀，我是 4ye，今天来分享下 Java8 Stream 的源码

![](_assets/4768bf7d847ea53177707fa2167aa3c0.png)

### 核心回顾

> stream 是一次性的，不是数据结构，不存储数据，不改变源数据.。API 分为终端和中间操作，中间操作是惰性的，碰到终端才去执行。中间操作有无状态和有状态之分，有状态需要更改上一步操作获得的所有元素，才可以进行下一步操作，比如 排序 sorted，去重 distinct，跳过 skip，限制 limit 这四个，需要多迭代一次。终端操作有短路与否之分，短路操作有 anyMatch, allMatch, noneMatch, findFirst, findAny

### 整体概览

这里列出一些重要的类，是看源码过程中必须了解的。

![](_assets/69033f50d0d01535eb3756684b992174.png)

比如 ：

Sink 接口是数据实际操作的地方，核心方法： begin，accept，endAbstractPipeline 抽象类是核心的数据结构，双链表

### demo代码

这里沿用上文的例子

`Student aStud = new Student(1, "a"); Student bStud = new Student(2, "b"); Student cStud = new Student(3, null); // 集合的创建 一 List`

### 步骤解析

都在这里了 👇

![](_assets/0bb27f5f4fde04a72c04bb1637ce169a.png)

这里步骤太多了，就不一一放出来了 ，列下核心

wrapSink, 创建 Sink 链，将管道的 Sink 操作连接在一起copyInto , 处理数据

### wrapSink()

开始套娃，从 ReducingSink 往前套

![](_assets/a8fd68bfc5cf046d3a169a2d4290df36.png)

opWarpSink 方法调用的是每一步 中间操作 中的方法

![](_assets/2a320ec39d25447d43365bdf36e0f84b.png)

通过 单链表 的形式将他们联系在一起

![](_assets/78fb29af024eeae788a96df4ffeab515.png)

Sink 链创建结果

![](_assets/cae90ee18b6b334226f89ce61d7cbb77.png)

### copyInto()

这里判断是不是 短路操作 ，然后就去执行 Sink 的 begin，accept，end 方法。

通过 forEachRemaining 进行内部迭代，这个是 Spliterator 的方法。

![](_assets/d988be5447c107ed83cfc26537486356.png)

map 链节点，直接调用传进来的方法，

![](_assets/d6fbf04011dc4222e9a44f9fea5553f1.png)

filter 链节点，多一步判断

![](_assets/7160082a2d3bcefcb6295784e1935cd1.png)

sorted 节点，添加到 list 中。

> 注意，此时没有继续调用 downstream.accept 方法！意味着，我们代码中的 5 个中间步骤只执行了前 3 个。

![](_assets/2ea663434c9a51ce9ec6bee04e75f68a.png)

不过别担心， sorted 链节点中它重写了这个 end，并开启对新数据的新一轮遍历！

> 这就是我们提到的，有状态中间操作多一次迭代的原因

![](_assets/1bc839a88e40aa7d13b877739f74efdd.png)

最后呢，是来到终止操作 TerminalOp 中的 accept，这里执行的是 list 的 add 方法（我们调用 Collectors.toList() 中构建的），至此，数据添加到 state 中

![](_assets/24e0625eec2a020e0ae1c8ae6eea3ea6.png)

![](_assets/b1207a06c6ad9bfe1b2bb7003f657e33.png)

获取数据，ReducingSink 继承了 Box 这个抽象类，最后 get 方法得到结果。

![](_assets/17da16098d953d96e8c786527e4eedcb.png)

### 总结

代码对应的执行流程👇

先创建流，出现了 Head 节点创建中间管道 Pipeline调用终端操作后有三步 👇（一）将中间管道的 Sink 操作连接在一起 （wrapSink）（二）处理数据 （copyInto），主要调用 Sink 中的 begin，accept（核心），end 操作（三）返回结果，ReducingSink 中的 get 方法

![](_assets/778abe7161ded6ba585e90d303349213.png)

> 主要记住这个 wrapSink 方法 和 copyInto 方法。一个套娃，一个调用 begin，accept，end 等方法。

那么，这个 stream 的原理机制就出来了：

> 利用 wrapSink 方法将各个中间操作中的 Sink 嵌套在一起，然后来到 copyInto 方法，调用 begin 通知各个 Sink 做好准备，接着进行内部迭代调用 accept 方法，再调用 end 方法完成数据的操作，最后通过 get 方法，获取新容器中的数据，便是结果了。

此外，源码的 链式调用API 写法 ，**设计模式 **的使用以及 泛型 ，四大函数式接口 组合构建的高度抽象，封装写法，对我们的编码能力，源码阅读能力也有很大的帮助！

比如 这个 Consumer+Function 接口的组合，配合泛型上下限的使用

![](_assets/d6fbf04011dc4222e9a44f9fea5553f1.png)

源码中 访问者模式，工厂模式 等设计模式的影子

> 访问者模式： 将数据结构与数据操作分离。对应源码：数据结构是 Pipeline ，操作是 Sink

![](_assets/a57baab22bc1c60fb90d9c9c523475f5.png)

对 stream 的特点更加熟悉

比如：

stream 是一次性的，不是数据结构，不存储数据，不改变源数据.。中间操作是惰性的，遇到 终端操作才真正执行有状态的中间操作的特殊之处在于多迭代一次内部迭代终端操作主要做了两件事，串连中间操作，调用 accept 方法处理数据

当然这里我也只测试了 非短路模式，短路模式又是怎样设计的呢，小伙伴们可以自己上手 debug 下，加深印象。

### 最后

> 有所收获的话，别忘了 三连(点赞，在看，转发） 鼓励下 4ye 呀，谢谢&下期见！😝

![](_assets/14ab06e18531a5935fe2379657a1f929.png)