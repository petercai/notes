# 详解Java之Future和Callable 

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03da3283dd4b444a81d4af576c6683fa~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=981&h=394&s=39436&e=png&b=ffffff)
](https://link.juejin.cn/?target=https%3A%2F%2Fimgse.com%2Fi%2FpisOocQ "https://imgse.com/i/pisOocQ")

### 引言

大家好，我是小黑！今天咱们来聊聊Java中的两个重要概念：`Future`和`Callable`。在Java的世界里，多线程和并发编程是个老大难问题，但也是提升性能的利器。`Future`和`Callable`就是这个领域的两个超级英雄。它们让处理复杂的异步任务变得简单，让代码既高效又易于管理。咱们会先理解它们各自的作用，然后看看如何巧妙地把它们组合起来，解决实际问题。

### 基本概念

首先，让我们先来了解一下 `Callable`。在Java的多线程世界里，大家可能更熟悉`Runnable`接口，它是用来创建可以在线程中运行的任务。但`Runnable`有个局限性——它不能直接返回执行结果。这时候，`Callable`就派上用场了。`Callable`类似于`Runnable`，但它可以返回一个结果，而且还能抛出异常。这就为处理复杂的业务逻辑提供了更大的灵活性。

接下来是 `Future`。当咱们提交一个`Callable`任务给线程池执行时，线程池会返回一个 `Future` 对象。这个 `Future` 对象就代表了异步计算的结果。它提供了方法来检查计算是否完成，等待其完成，以及检索计算结果。简而言之，`Future` 是对未来某个时刻计算结果的一个占位符。

### 深入Callable

好，现在让我们更深入地了解一下 `Callable`。首先看一下它的基本用法：

```java
import java.util.concurrent.Callable;

public class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        
        return "任务完成!";
    }
}

```

这个例子中，小黑创建了一个 `MyCallable` 类，它实现了 `Callable` 接口，并且指定了返回类型为 `String`。在 `call` 方法里，咱们可以执行任何复杂的逻辑，然后返回一个字符串结果。

接下来，咱们如何执行这个 `Callable` 呢？这就需要线程池的帮助了。看下面这段代码：

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        MyCallable task = new MyCallable();
        Future<String> future = executor.submit(task);

        

        try {
            String result = future.get(); 
            System.out.println("结果: " + result);
        } catch (Exception e) {
            e.printStackTrace();
        }

        executor.shutdown();
    }
}

```

在这个例子中，小黑使用了一个线程池 `ExecutorService` 来提交 `MyCallable` 任务。`submit` 方法会返回一个 `Future` 对象，咱们可以用它来获取任务的结果。注意，`future.get()` 会阻塞，直到任务执行完成，所以如果任务很耗时，咱们可能不想在主线程中调用它。

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be33fe72a3f149dd8dc2bbfb7512749d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=975&h=703&s=26949&e=png&b=ffffff)
](https://link.juejin.cn/?target=https%3A%2F%2Fimgse.com%2Fi%2FpisOI1g "https://imgse.com/i/pisOI1g")

### 探索Future

咱们来聊聊 `Future` 吧。想象一下，小黑在一个超大的购物中心里，让一个店员去后库房找件衣服。这时候，店员给了小黑一个小票，上面写着“等会儿来拿”。这小票就像是Java里的 `Future`。它不是衣服本身，而是一个承诺，告诉小黑“你的请求正在处理，稍后你就可以得到你想要的东西”。

#### 如何使用Future

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<String> futureTask = executor.submit(new Callable<String>() {
    @Override
    public String call() throws Exception {
        
        Thread.sleep(2000);
        return "结果";
    }
});

```

在这段代码里，`submit()` 方法提交了一个 `Callable` 任务到线程池，然后立刻返回了一个 `Future` 对象。这就像是小黑给了店员一个要求，然后得到了那个小票。

#### 从Future获取结果

```java
try {
    String result = futureTask.get(); 
    System.out.println("结果是: " + result);
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}

executor.shutdown();

```

使用 `futureTask.get()` 可以获取任务的结果。但这个方法会阻塞，意味着它会在那儿等着，直到衣服从后库房送来。如果小黑不想等，他可以去逛逛其他店，也就是做一些其他的事情，然后回来拿结果。

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed878c3e1e0b4b25aa08acf191a2dca4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=928&h=885&s=30901&e=png&b=ffffff)
](https://link.juejin.cn/?target=https%3A%2F%2Fimgse.com%2Fi%2FpisO59S "https://imgse.com/i/pisO59S")

### Callable与Runnable的比较

咱们再来看看 `Callable` 和 `Runnable` 的区别。这就像是小黑在餐厅点餐。如果小黑点的是快餐（`Runnable`），他就得到了食物，但不知道食物里有什么（没有返回值）。如果他点的是定制菜单（`Callable`），那他不仅得到食物，还能知道里面具体有什么材料（有返回值）。

#### 为什么要用Callable

```java

new Thread(new Runnable() {
    @Override
    public void run() {
        
    }
}).start();


Future<String> future = Executors.newSingleThreadExecutor().submit(new Callable<String>() {
    @Override
    public String call() throws Exception {
        return "带返回值的任务结果";
    }
});

```

在 `Runnable` 的世界里，咱们只能让线程去执行任务，但不知道任务执行的结果。但用 `Callable` 就不一样了，它不仅可以执行任务，还能告诉咱们任务执行的结果是什么。

总的来说，如果咱们只是想让线程去做某件事，不关心结果，那用 `Runnable` 就够了。但如果咱们想知道线程做的这件事最后怎么样了，那 `Callable` 就是更好的选择。这就像是小黑在买东西，有时候只是想快点买完走人，有时候则想知道更多详情。

### 实际应用场景

来看看，`Callable` 和 `Future` 在咱们的日常开发中怎么用。首先，咱们来聊聊网络请求。想象一下，小黑要从多个数据源并行获取数据，每个数据源的响应时间都不一样。这时候，如果小黑用 `Callable`，就可以异步地发起请求，而通过 `Future` 去跟踪每个请求的状态，一旦数据返回，就可以处理它。这样做的好处是显而易见的，提高了程序的响应速度和效率。

再说一个例子，假设小黑正在开发一个股票分析应用，需要从不同的API获取实时股价，然后进行汇总分析。使用 `Callable` 和 `Future`，小黑可以同时发起多个API调用，然后等所有调用完成后，再统一处理结果。这样一来，整个过程变得更加高效，用户也不需要等待太久。

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
Future<Double> futurePrice = executor.submit(new Callable<Double>() {
    @Override
    public Double call() throws Exception {
        
        return fetchStockPrice("AAPL");
    }
});



try {
    Double price = futurePrice.get(); 
    System.out.println("股票价格: " + price);
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}

executor.shutdown();

```

### 性能和最佳实践

使用 `Callable` 和 `Future` 时，性能是个大话题。咱们得知道，每次创建 `Future` 都会有一些开销。所以，如果任务非常小，可能直接执行会更快。还有，得注意线程池的管理，不要创建太多线程，否则会消耗大量资源，还可能导致系统崩溃。

最佳实践方面，最好使用现有的线程池框架，比如 `ExecutorService`，这样可以有效管理线程生命周期，还能重用线程。另外，处理 `Future` 的返回值时要小心，如果直接调用 `get()`，可能会阻塞主线程，特别是在处理大量任务时。所以，考虑使用 `isDone()` 方法来检查任务是否完成，这样可以避免阻塞。

### 总结

咱们今天聊了不少关于 `Callable` 和 `Future` 的东西。记住，这两个工具在处理异步任务时特别有用，能帮助咱们写出更高效、更可靠的代码。但同时，也要注意它们的使用场景和潜在的性能影响。希望咱们都能在未来的项目中灵活运用这些知识，写出更加出色的Java程序！

以上就是关于 `Future` 和 `Callable` 的深入解析，结合实际案例和代码示例，希望能帮助咱们更好地理解和应用这两个重要的Java并发工具。

* * *

面对寒冬，我们更需团结！小黑收集整理了一份超级强大的复习面试资料包，也强烈建议你加入我们的Java后端报团取暖微信群，一起复习，共享各种学习资源，互助成长。无论是新手还是老手，这里都有你的位置。在这里，我们共同应对职场挑战，分享经验，提升技能，闲聊副业，共同抵御不确定性，携手走向更稳定的职业未来。让我们在Java的路上，不再孤单！进群方式以及资料，点击如下链接即可获取！

