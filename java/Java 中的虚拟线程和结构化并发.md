# Java 中的虚拟线程和结构化并发
#### 那么什么是并发呢？这是为什么呢？

并发是编程中一个复杂但重要的主题，随着行业和需求的发展，我们的工具也在不断进步。那么，您可能想知道并发究竟是什么，它与异步有什么区别？答案可以归结为一个简单的区别：并发性是指同时发生多件事情的过程，而异步性则是指在做其他事情的同时，要求发生某件事情，并在完成后得到通知。在并发性中，多个进程至少看起来是同时发生的，而在异步性中，一个进程是在 "等待 "某件事情的发生。

在 Java 中，并发涉及使用多个系统线程同时处理不同的进程，以充分利用计算机的能力。虽然线程从 Java 1.5 开始就成为 Java 的一个元素，但开发人员的需求和不断发展的技术环境要求它随着时间的推移而发展。创建线程很容易，但是管理线程呢？这完全是另一回事。为此，Java 5 引入了执行器服务（Executor Service），而 Java 8 中新增的 ForkJoinPool 和 CompletableFutures 则进一步简化了这一过程。这些功能在当时非常有用，现在也依然强大，但随着需求的增加，这种模式已经开始落后。Java 并发的问题可以用两个词来概括：成本和无知。

#### 为什么说线程的代价是高昂的?

那么，我说线程代价高昂是什么意思呢？你看，启动线程需要耗费大量系统资源。它的启动时间约为 1 毫秒，占用约 2 兆内存（在堆栈上），你还需要切换上下文，这需要约 100 纳秒（取决于你的操作系统）。现在只有一个线程？看起来已经够小了，但当你讨论的是同时占用数千个线程的进程时，你就会开始注意到明显的速度减慢，这当然有点适得其反。这也意味着不可能有百万线程的规模，因为这需要大约 15 分钟的时间和 2 TB 的内存。这看起来似乎很多，但考虑到如今电脑使用的密集程度，我们就会开始明白，也许我们希望能在同一时间做更多的事情。

此外，如果你拥有如此昂贵的资源，你会希望尽可能多地利用它，但要让系统保持忙碌并不容易。空闲的线程仍然会占用资源，但如果不小心，线程可能在其整个生命周期中都处于空闲状态。例如，当你使用单个线程处理需要等待的事情时，它只需等待，所有线程都以相同的速度等待。为了解决这个问题，我们引入了 Executor Service、ForkJoinPool 和 CompletableFuture，这些工具可以为你管理线程池，为异步代码提供便利。让我们写一些代码作为参考。

让我们举一个通常需要异步性的常见例子：API 调用！为了便于说明和相互确保毁灭，我将使用 DadJokes API。我们的调用函数看起来是这样的

```java
private DadJoke callApi() {  
ResponseEntity<DadJoke> response =    
        template.getForEntity("https://icanhazdadjoke.com/", DadJoke.class);

    if (response.getStatusCode() != HttpStatus.OK){  
        throw new ConnectionException(response.getStatusCode());  
    }  
    return response.getBody();  
}

```

假设，我想返回一个包含三个DadJoke数组的实体。如果我以一种同步的、有点天真的和低效的方式来写这个，我可以这样做：

```java
public DadJoke[] getDadJokes(){  
    DadJoke[] jokes = new DadJoke[3];  

    jokes[0] = callApi();  
    jokes[1] = callApi();  
    jokes[2] = callApi();  

    return jokes;  
}

```

我调用 API 3 次，收到三个随机的DadJoke，我将它们放入一个数组中，然后将该数组发回。比较简单。现在，对 API 的每次调用都会被阻塞。下一个函数等待前一个函数完成。这是一个问题，因为它会减慢速度。当然，对于单个调用来说这并不是什么大问题，但是如果我们有更多调用呢？如果我们有数百个怎么办？好吧，让我们使用可完成的 future 来使这三个调用同时发生：

```java
private CompletableFuture<DadJoke> getDadJokeFuture(){  
    return CompletableFuture  
            .supplyAsync(this::callApi)  
            .exceptionally(ex -> new DadJoke("error", ex.getMessage(),
                         HttpStatus.BAD_REQUEST.value()));  
}

public DadJoke[] getDadJokes() {   
    CompletableFuture<DadJoke> future1 = getDadJokeFuture();  
    CompletableFuture<DadJoke> future2 = getDadJokeFuture();  
    CompletableFuture<DadJoke> future3 = getDadJokeFuture();  

    return Stream.of(future1, future2, future3)  
            .map(CompletableFuture::join)  
            .toArray(DadJoke[]::new);  

}

```

我创建了一个小辅助函数，它只是为我提供了一个调用 api 的 CompletableFuture。然后，我并行运行可完成的 future，并将结果连接在一起，形成数组。Completable future 使用系统线程，默认情况下，线程数量与系统的核心数相同。这有效！但这很费劲......而且看起来有点混乱。此外，只有在任务完成后才能处理异常。线程彼此不知道，如果一个任务失败，其他任务将不必要地继续他们的工作，即使我们的代码无论如何都会失败。此外，如果线程以某种方式被中断或抛出异常，则两者都不会传播到子任务。当然，所有这一切都是对非常宝贵资源的浪费！这就像在希尔顿租了一个房间五天，但只在那里住了一天！

#### 虚拟线程

技术在不断进步，我们对计算机的使用也在飞速发展。上述并发模型可以解决近十年前出现的问题，但现在问题越来越多。我们该如何解决这个问题呢？进入虚拟线程和结构化并发。要解释虚拟线程和结构化并发如何解决成本和无知的问题，首先让我解释一下什么是虚拟线程。

让我给你一小段代码，告诉你如何实例化虚拟线程。虚拟线程有各种工厂方法，可以用来实例化虚拟线程。让我们比较一下虚拟线程和系统线程的输出。

```java
public static void main(String[] args) throws InterruptedException {  
    Thread platformThread = Thread.ofPlatform()  
            .name("system-", 0)  
            .start(() -> {  
                System.out.println(String.format("system %s", Thread.currentThread()));  
            });  

    platformThread.join();  

    Thread virtualThread = Thread.ofVirtual()  
            .name("virtual-", 0)  
            .start(() -> {  
                System.out.println(String.format("virtual %s", Thread.currentThread()));  
            });  
    virtualThread.join();  
}

```

在这里您可以看到如何实例化两种不同类型的线程。他们输出以下内容：

```csharp
system Thread[#21,system-0,5,main]
virtual VirtualThread[#22,virtual-0]/runnable@ForkJoinPool-1-worker-1

```

有趣的是，虚拟线程的内容似乎更多一些。斜线后面的文字是什么意思？虚拟线程是运行在系统线程之上的特殊轻量级线程。它们由 JVM 管理，是可分离的。这意味着创建虚拟线程不需要进行特殊的系统调用，JVM 知道它们的运行情况，它们可以按需分离和重新连接，这意味着当虚拟线程闲置时，系统线程可以自由地处理其他事情。虚拟线程的成本很低，这意味着你可以创建数百万个虚拟线程。请注意

java

```scss
public static void main(String[] args) throws InterruptedException {  
    var threads =  
            IntStream.range(0, 1_000_000)  
                    .mapToObj(index ->  
                            Thread.ofVirtual()  
                                    .name("virtual-", index)  
                                    .unstarted(() -> {  
                                        try {  
                                            System.out.printf("virtual %s%n", Thread.currentThread());  
                                            Thread.sleep(2_000);  
                                        } catch (InterruptedException e) {  
                                            throw new RuntimeException(e);  
                                        }  
                                    }))  
                    .toList();  

    Instant begin = Instant.now();  
    threads.forEach(Thread::start);  
    for (Thread thread : threads) {  
        thread.join();  
    }  
    Instant end = Instant.now();  
    System.out.println("Duration = " + Duration.between(begin, end).toSeconds());  
}

```

所以…。那么它是如何运作的呢？有多快？让我们看一下输出的一部分。

```ruby
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
virtual VirtualThread[
Duration = 19

```

这组代码表明，实际上有......很多线程。不仅如此，这些线程只使用了有限的系统线程！我可以告诉你，使用的最大系统线程实际上是 8 个。最后的持续时间显示，从开始到结束共用了 19 秒，但这总比不可能好得多，不是吗？更何况，它还不会像小糖果一样吃内存。它还能在单个系统线程上运行多个线程。这才叫多线程。

从本质上讲，启动和虚拟线程的成本低得像土一样，这意味着启动和管理虚拟线程不再是个大问题。尽管如此，管理大量线程可能仍然是个令人头疼的问题。不过，你仍然可以像以前一样利用 Executor 服务来构建虚拟线程池，或者使用结构化并发。这相当于 Kotlin 的 coroutines 或 Go 的 goroutines。

请注意，截至 Java 21，结构化并发仍是一项预览功能，因此情况一定会发生变化，功能也仍然有限。话不多说，让我们使用结构化并发重写 api 调用。

```csharp
public DadJoke[] getDadJokes(){  

    try (StructuredTaskScope<DadJoke> scope = new StructuredTaskScope<>()) {  

        Subtask<DadJoke> fork1 = scope.fork(this::callApi);  
        Subtask<DadJoke> fork2 = scope.fork(this::callApi);  
        Subtask<DadJoke> fork3 = scope.fork(this::callApi);  

        scope.join();  

        if (forkIsUnsuccessful(fork1, fork2, fork3)){  

            throw new StructuredConcurrencyException("A thread failed");  
        }  

        return new DadJoke[]{fork1.get(), fork2.get(), fork3.get()};  

    } catch (InterruptedException e) {  
        e.printStackTrace();   
    }  
}

@SafeVarargs  
private <T> boolean forkIsUnsuccessful(Subtask<T>... tasks){  

    return Arrays  
            .stream(tasks)  
            .anyMatch(task -> task.state() == State.FAILED);  
};

```

首先，我们在一个 try-with-resources 语句中实例化我们的作用域。这是什么，作用域？它是一个允许我们执行结构化并发的对象。它支持将一个任务分割成多个子任务，这些子任务都是......并发执行的。并发操作的生命周期仅限于其语法块，就像顺序操作一样。并发操作的工作方式与 Executor Service 很相似，都是由用户向其提供任务。fork 方法定义了要执行的子任务。子任务包含一个值和一个状态。该状态可用于检查任务是否成功。join 方法是阻塞的，它会等待所有子任务都完成，然后提取子任务的值和状态。每个子任务都在单独的虚拟线程上运行，阻塞这些线程也没有问题，因为它们比系统线程便宜得多。重要的是，要实现这一点，范围内的值是不可变的，但可以在线程之间共享。

在上述代码中，所有的检查等工作都是手动完成的，但 StructuredStaskScope 有一些特殊的工厂方法，允许它在成功和失败时关闭。ShutdownOnSuccess 作用域只是在第一个子任务成功后关闭作用域。当然，ShutdownOnFailure 作用域的作用与此相反，您可以通过抛出异常来跟进。

像这样的东西

```csharp

public DadJoke[] getDadJokes(){  

    try (StructuredTaskScope.ShutdownOnFailure scope = new
                    StructuredTaskScope.ShutdownOnFailure()) {  

        Subtask<DadJoke> fork1 = scope.fork(this::callApi);  
        Subtask<DadJoke> fork2 = scope.fork(this::callApi);  
        Subtask<DadJoke> fork3 = scope.fork(this::callApi);  

        scope.join();  
        scope.throwIfFailed(ex -> new
                StructuredConcurrencyException(ex.getMessage()));  
                
                

        return new DadJoke[]{fork1.get(), fork2.get(), fork3.get()};  

    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
}

```

这就是 Java 中的虚拟线程和结构化并发！希望它能让你对如何使用有所了解。虚拟线程的作用是让你在特定范围内提高某些任务的性能。这样，当所有线程在退出前都完成了自己的任务时，你就可以在多线程进程中应用显而易见的控制流。