# 性能篇：LinkedList循环为什么使用Iterator而不是for？
今天我们来聊一个平时开发中可能会遇到的问题——在使用LinkedList时，为什么要避免使用for循环来读取元素，以及如何优化性能。

首先，让我们简单回顾一下LinkedList的基本概念。LinkedList是一种链表数据结构，由节点组成，每个节点包含数据元素和指向下一个节点的引用。相比于ArrayList，LinkedList具有动态大小、插入和删除更高效的特点。

在使用LinkedList时，我们通常会使用迭代器（Iterator）来遍历元素，而不是采用传统的for循环。为什么呢？原因在于LinkedList的数据存储方式。在ArrayList中，我们可以通过索引直接访问元素，但是在LinkedList中，要通过遍历找到目标元素，这就导致了使用for循环的性能问题。

让我们看一个简单的例子：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfac41c75a974818893d19fb26d5a232~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1070&h=386&s=41553&e=jpg&b=fcf8f8)
[](https://link.juejin.cn/?target=)

这样的代码在LinkedList上运行会非常慢，因为每次调用**get(i)** 都需要从头开始遍历链表，时间复杂度是O(n)，总体性能会大大降低。

为了避免这种性能问题，我们应该使用迭代器来遍历LinkedList。迭代器内部维护了一个指针，可以直接访问下一个元素，而无需每次都从头开始查找。修改上述代码：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7085740363bd4ac3827908d7e1239cf1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1072&h=418&s=46655&e=jpg&b=fcfafa)
[](https://link.juejin.cn/?target=)

通过使用迭代器，我们避免了每次都从头开始遍历链表的性能问题，使得代码更加高效。

为了更直观地感受性能的差异，我们可以通过简单的测试来对比使用for循环和迭代器的性能。这里我用JMH（Java Microbenchmarking Harness）来进行简单的性能测试：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed1799d66573405485ed7378e1b141c5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=1235&s=126431&e=jpg&b=fcfbfb)
[](https://link.juejin.cn/?target=)

测试结果很明显，使用迭代器的性能要远远优于使用for循环，特别是在数据量较大的情况下。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31754a99b4ab4f1bb4ed0c8cfe711a07~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=760&h=480&s=23468&e=jpg&b=333333)
[](https://link.juejin.cn/?target=)

我们经常使用的 LinkedList 集合，如果使用 for 循环遍历该容器，将大大降低读的效率，但这种效率的降低很难导致系统性能参数异常。

这时有经验的同学，就会改用 Iterator （迭代器）迭代循环该集合，这是**因为 LinkedList 是链表实现的，如果使用 for 循环获取元素，在每次循环获取元素时，都会去遍历一次List，这样会降低读的效率。** 

通过这篇文章，我们了解了在使用LinkedList时为什么要避免使用for循环，以及如何通过使用迭代器来优化性能。在实际开发中，优化性能是我们应该时刻关注的问题之一，合理选择数据结构和遍历方式是其中的一个重要方面。
