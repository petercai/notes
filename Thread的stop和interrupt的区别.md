# Thread的stop和interrupt的区别
Thread.stop
-----------

`Thread.stop()`方法已被废弃。

因为本质上它是不安全的，使用该方法可能会导致数据、资源不一致的问题，

```java
public class ThreadDemo {
    static class MyThread extends Thread {
        @Override
        public void run() {
            while (true) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    
                }
                
            }
        }
    }
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
        
        try {
            Thread.sleep(2000);
        } catch (Exception e) {
            e.printStackTrace();
        }
        
        myThread.stop();
        System.out.println("Thread has been stopped (using deprecated stop method).");
    }
}

```

### Thread.stop测试结果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/228cf087bcc94f61a00328d57a94270f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=4096&h=2090&s=524116&e=png&b=2c2c2c)

在控制台可以看到输出：Thread has been stopped (using deprecated stop method).可以明确看到：stop方法已被废弃。

Thread interrupt
----------------

使用`stop`方法会导致线程突然终止，可能导致如：线程持有的资源没有被正确释放，使得程序状态不一致问题。因此建议使用更安全的方式来停止线程，比如使用`interrupt`发出终端请求来实现停止一个正在运行的线程。

```java

 * @author: liu_pc
 * Date:        2024/5/6
 * Description: 安全的停止一个线程
 * Version:     1.0
 */
@Slf4j
public class InterruptThreadExample {
    static class MyThread extends Thread {
        public void run() {
            while (!Thread.currentThread().isInterrupted()) {
                
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    
                    
                    Thread.currentThread().interrupt(); 
                    break; 
                }
                
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();
        myThread.start();

        
        Thread.sleep(2000);

        
        myThread.interrupt();

        
        myThread.join();

        System.out.println("Thread has been stopped gracefully using interruption.");
    }
}

```

### Thread interrupt测试结果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3f8ee0f44674f9fa005559fe4729d73~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2006&h=546&s=68144&e=png&b=2c2c2c)

