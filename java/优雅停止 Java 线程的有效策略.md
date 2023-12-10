# 优雅停止 Java 线程的有效策略
这篇文章主要是如何安全有效停止 Java 线程的，确保多线程应用程序平稳运行并实现最佳资源管理

在Java中停止线程意味着在完成其任务之前停止正在进行的操作，本质上是放弃当前操作。

虽然可以使用 Thread.stop() 方法停止线程，但强烈建议不要这样做。虽然它确实终止了正在运行的线程，但此方法被认为是不安全的并且已被弃用。

在Java中，有3种方法可以终止正在运行的线程：

1.  使用标志：您可以创建一个boolean类型的标志，线程定期检查该标志。当该标志设置为某个值时，线程可以优雅地退出其执行。
2.  使用interrupt()方法：可以使用interrupt()方法向线程发送中断信号。
3.  使用Thread.stop()方法（不推荐）：可以使用Thread.stop()方法强行停止正在运行的线程。然而，这种方法不被鼓励并且被认为是不安全的，因为它可能导致应用程序中出现不可预测为。它已被弃用，应该避免。

使用interrupt()方法不会像带有break语句的for循环一样立即停止循环。它会在当前线程内设置一个停止标志，但它不会立即停止线程。

```arduino
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            System.out.println("i="+(i+1));
        }
    }
}

public class Run {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        try {
            Thread.sleep(2000);
            thread.interrupt();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

结果：
...
i=499994
i=499995
i=499996
i=499997
i=499998
i=499999
i=500000

```

Java中的提供了两种方法：

1.  this.interrupted()：该方法检测当前线程（**调用该方法的线程**）是否已被中断。它还具有清除当前线程的中断状态的作用。
2.  this.isInterrupted()：此方法测试调用它的线程（**不一定是当前线程**）是否已被中断。它不会清除线程的中断状态。

**这两种方法有什么区别？**

我们先看一下this.interrupted()方法：

```arduino
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            i++;
            
        }
    }
}

public class Run {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        try {
            Thread.sleep(2000);
            thread.interrupt();

            System.out.println("stop 1??" + thread.interrupted());
            System.out.println("stop 2??" + thread.interrupted());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
---------------------------
结果：
stop 1??false
stop 2??false

```

可以看到 Run 类中调用Thread对象的 Interrupt() 方法来检查线程对象的线程是否已停止，控制台输出的内容表明线程尚未停止。

这也证实了interrupted()方法，它用于检测当前线程（在此上下文中为主线程）是否已被中断。由于主线程从未被中断过，所以打印的结果是两个false。

```arduino
public class Run2 {
    public static void main(String args[]){
        Thread.currentThread().interrupt();
        System.out.println("stop 1??" + Thread.interrupted());
        System.out.println("stop 2??" + Thread.interrupted());

        System.out.println("End");
    }
}
------------------
结果：
stop 1??true
stop 2??false
End

```

第二个值是 false，因为根据 Interrupted() 方法的官方文档，它会检测当前线程（在这里就是主线程）的中断状态，它会在调用时清除线程的中断状态。

换句话说，如果连续调用它，第二次调用返回 false，因为它已经清除了第一次调用设置的中断状态。

因此，如果连续调用interrupted()两次，第二次调用将返回false，因为第一次调用清除了线程的中断状态。

现在让我们看一下 isInterrupted() 方法。

```arduino
public class Run3 {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        thread.interrupt();
        System.out.println("stop 1??" + thread.isInterrupted());
        System.out.println("stop 2??" + thread.isInterrupted());
    }
}
---------------
结果：
stop 1??true
stop 2??true

```

isInterrupted() 不会清除中断状态，这就是为什么在输出中看到两个true。

有了上面获得的知识，就可以在线程中使用for循环来检查线程是否处于停止状态。 如果处于停止状态，后续的代码将不再运行。

```csharp
public class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 500000; i++) {
            if (Thread.interrupted()) {
                System.out.println("Thread is interrupted. Exiting...");
                return; 
            }
            System.out.println("i="+(i+1));
        }
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();
        try {
            Thread.sleep(2000);
            thread.interrupt(); 
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
-------------
结果：
...
i=202053
i=202054
i=202055
i=202056
Thread is interrupted. Exiting...

```

在停止线程的同时，将继续执行 for 循环之后的任何代码。

改下代码，让我们看一下下面的例子：

```scala
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            if(this.interrupted()) {
                System.out.println("Thread is interrupted. Exiting...");
                break;
            }
            System.out.println("i="+(i+1));
        }

        System.out.println("Out of for");
    }
}
结果：
...
i=180136
i=180137
i=180138
i=180139
Thread is interrupted. Exiting...
Out of for

```

```scala
public class MyThread extends Thread {
    public void run(){
        super.run();
        try {
            for(int i=0; i< 500000; i++){
                if(this.interrupted()) {
                    System.out.println("Thread is interrupted. Exiting...");
                    throw new InterruptedException();
                }
                System.out.println("i="+(i+1));
            }

            System.out.println("Out of for");
        } catch (InterruptedException e) {
            System.out.println("In catch...");
            e.printStackTrace();
        }
    }
}
--------------------------------------------------------------------------
结果：
...
i=203798
i=203799
i=203800
Thread is interrupted. Exiting...
In catch...
java.lang.InterruptedException
 at thread.MyThread.run(MyThread.java:13)

```

如果线程在 sleep() 状态下停止会有什么影响？

```scala
public class MyThread extends Thread {
    public void run(){
        super.run();

        try {
            System.out.println("Thread begin...");
            Thread.sleep(200000);
            System.out.println("Thread end...");
        } catch (InterruptedException e) {
            System.out.println("Stop while sleeping" + this.isInterrupted());
            e.printStackTrace();
        }
    }
}
-----------------------------------------------------------------------
 结果：
Thread begin...
Stop while sleeping，the result of isInterrupted() is：：false
java.lang.InterruptedException: sleep interrupted
 at java.lang.Thread.sleep(Native Method)
 at thread.MyThread.run(MyThread.java:12)

```

从打印结果来看，如果线程在睡眠状态下停止，它将抛出InterruptedException异常进入catch块并清除停止状态值，将其设置为false。

使用 stop() 方法终止线程是一种非常激进的方法。

```arduino
public class MyThread extends Thread {
    private int i = 0;
    public void run(){
        super.run();
        try {
            while (true){
                System.out.println("i=" + i);
                i++;
                Thread.sleep(200);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public class Run {
    public static void main(String args[]) throws InterruptedException {
        Thread thread = new MyThread();
        thread.start();
        Thread.sleep(2000);
        thread.stop();
    }
}
-----------------------------------------------
  结果：
i=0
i=1
i=2
i=3
i=4
i=5
i=6
i=7
i=8
i=9

Process finished with exit code 0

```

当调用 stop() 方法时，它会抛出 java.lang.ThreadDeath 异常，但大多数情况下，不需要显式捕获该异常。

```java
public class MyThread extends Thread {
    private int i = 0;
    public void run(){
        super.run();
        try {
            this.stop();
        } catch (ThreadDeath e) {
            System.out.println("In catch");
            e.printStackTrace();
        }
    }
}

public class Run {
    public static void main(String args[]) throws InterruptedException {
        Thread thread = new MyThread();
        thread.start();
    }
}

```

stop() 方法已被弃用，因为强制停止线程可能会阻止某些必要的清理工作完成。

此外，它还可能导致锁定对象解锁，从而导致数据同步问题和数据不一致问题。 由

于 stop() 方法在 JDK 中已被标记为已弃用/过时，因此很明显它存在功能缺陷。 因此，不建议在程序中使用 stop() 方法。

将interrupt()方法与return结合起来也可以达到停止线程的效果。

```arduino
public class MyThread extends Thread {
    public void run(){
        while (true){
            if(this.isInterrupted()){
                System.out.println("The thread has been stopped.");
                return;
            }
            System.out.println("Time: " + System.currentTimeMillis());
        }
    }
}

public class Run {
    public static void main(String args[]) throws InterruptedException {
        Thread thread = new MyThread();
        thread.start();
        Thread.sleep(2000);
        thread.interrupt();
    }
}

------------------------------------
结果：...
Time: 1696990194000
Time: 1696990194000
Time: 1696990194000
The thread has been stopped.

```

但是，仍然建议使用“抛出异常”方法来停止线程，因为它允许您通过在 catch 块中重新抛出异常来传播停止事件。

