# java核心技术-多线程基础
**进程、线程**

​ 进程(Process) 是程序的运行实例。例如，一个运行的 Eclipse 就是一个进程。进程是程序向操作系统申请资源(如内存空间和文件句柄)的基本单位。线程(Thread)是进程中可独立执行的最小单位。一个进程可以包含多个线程。进程和线程的关系，好比一个营业中的饭店与其正在工作的员工之间的关系。

**1.1 线程的创建、启动与运行**

在 Java 中实现多线程主要用两种手段，一种是继承 Thread 类，另一种就是实现 Runnable 接口。**(当然还有 Callable 和线程池)**。下面我们就分别来介绍这两种方式的使用,其他请关注此博客下文。

**(1).继承 Thread 的类**

```
public class PrimeThread extends Thread{
 
 @Override
 public void run(){
 
 for(int i = 0; i < 100; i++){
 if(i % 2 == 0){
 System.out.println(Thread.currentThread().getName() + "=" + i);
 }
 }
 
 }
 
}
public class TestThread {
 public static void main(String[] args) {
 
 PrimeThread p1 = new PrimeThread();
 
 p1.start();
 
 PrimeThread p2 = new PrimeThread();
 p2.start();
 
 
 for(int i = 0; i < 100; i++ ){
 if(i % 2 == 0){
 System.out.println(Thread.currentThread().getName() + "=" + i);
 }
 }
 
 }
}
```

**(2).实现 Runnable 接口**

```
public class Ticket implements Runnable{
 
 private int ticket = 100;
 @Override
 public void run() {
 while(ticket > 0){
 System.out.println(Thread.currentThread().getName() + "=" + --ticket);
 }
 
 }
}
public class TestThread2 {
 public static void main(String[] args) {
 
 Ticket ticket = new Ticket();
 
 
 Thread t1 = new Thread(ticket,"售票窗口一");
 t1.start();
 
 Thread t2 = new Thread(ticket,"售票窗口二");
 t2.start();
 
 Thread t3 = new Thread(ticket,"售票窗口三");
 t3.start();
 }
}
```

**两种实现方式的对比：** 

1.从面向对象编程角度看：第一种创建方式(继承 Thread 类) 是一种基础继承的技术,第二种创建方式(以 Runnable 接口实例为构造器参数直接通过 new 创建 Thread 实例)是一种基础组合的技术。方式二不仅会避免单继承的尴尬，也会降低类与类之间的耦合性。

2.从对象共享角度看：第二种创建方式意味着多个线程实例可以共享同一个 Runnable 实例。而第一种方式则需要依赖 static 关键字来完成操作。

**1.2 线程的控制**

> Java 的调度方法
> 
> 同优先级线程组成先进先出队列（先到先服务），使用时间片策略
> 
> 对高优先级，使用优先调度的抢占式策略

**Thread 类的相关方法：** 

*   sleep(long millis) : 是 Thread 类中的静态方法，使当前线程进入睡眠状态
    
*   join() / join(long millis) : 是一个实例方法，使当前线程进入阻塞状态
    
*   interrupt() : 中断阻塞状态的线程
    
*   isAlive() : 判断当前线程是否处于存活状态
    
*   yield() : 线程让步
    

```
public class TestThread3 {
 public static void main(String[] args) throws Exception {
 
 Thread t1 = new Thread(new Runnable() {
 @Override
 public void run() {
 for(int i = 1; i < 100;i++){
 if(i % 2 == 0){
 System.out.println(Thread.currentThread().getName() + "=" + i);
 }
 }
 
 }
 },"线程1");
 t1.start();
 
 t1.sleep(10000);
 
 t1.join();
 Thread t2 = new Thread(new Runnable() {
 @Override
 public void run() {
 for(int i = 1; i < 100;i++){
 if(i % 2 != 0){
 System.out.println(Thread.currentThread().getName() + "=" + i);
 }
 }
 }
 },"线程2");
 t2.start();
 
 }
}
```

![](_assets/0205b2fd5ddcae5b8041b649c8dd92c2.png)

**1.3 线程的同步**

线程同步：模拟售票程序，实现三个窗口同时售票 100 张 （1.1 案例）

问题：当三个窗口同时访问共享数据时，产生了无序、重复、超额售票等多线程安全问题

解决：将需要访问的共享数据“包起来”，视为一个整体，确保一次只能有一个线程执行流访问该“共享数据”

Java 给上述问题提供了几种相应的解决方法

**(1).同步代码块**

> synchronized(同步监视器){
> 
> //需要访问的共享数据
> 
> }
> 
> 同步监视器 ： 俗称“锁”，可以使用任意对象的引用充当，注意确保多个线程持有同一把锁(同一个对象)

**(2).同步方法**

> 同步方法 : 在方法声明处加 synchronized. 注意：非静态同步方法隐式的锁 ---- this
> 
> 例如：
> 
> public synchronized void show(){}

**(3).同步锁**

> 同步锁 ： Lock 接口

同步代码块

```
public class SafeTicket implements Runnable{
 
 private int ticket = 100;
 
 @Override
 public void run() {
 while(true){
 
 synchronized (this) {
 if(ticket > 0){
 System.out.println(Thread.currentThread().getName() + " 完成售票,余票:" + --ticket);
 }
 }
 
 
 }
 }
}
```

同步方法：

```
public class SafeTicket implements Runnable{
 
 private int ticket = 100;
 
 @Override
 public void run() {
 while(true){
 
 sale();
 
 }
 }
 
 public synchronized void sale(){
 if(ticket > 0){
 System.out.println(Thread.currentThread().getName() + " 完成售票,余票:" + 
--ticket);
 }
 }
 
}
```

同步锁

```
public class SafeTicket implements Runnable{
 
 private int ticket = 100;
 
 private Lock l = new ReentrantLock();
 
 @Override
 public void run() {
 while(true){
 l.lock();
 try {
 if(ticket > 0){
 System.out.println(Thread.currentThread().getName() + " 完成售票,余票:" + 
--ticket);
 }
 } finally {
 l.unlock();
 }
 
 }
 }
 
}
```

**死锁**

_死锁_ 是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。此时称系统处于 _死锁_ 状态或系统产生了 _死锁_ ，这些永远在互相等待的进程称为 _死锁_ 进程

```
public class TestDeadLock {
 public static void main(String[] args) {
 final StringBuffer s1 = new StringBuffer();
 final StringBuffer s2 = new StringBuffer();
 
 new Thread() {
 public void run() {
 synchronized (s1) {
 s2.append("A");
 
 try {
 Thread.sleep(10);
 } catch (InterruptedException e) {
 e.printStackTrace();
 }
 
 
 synchronized (s2) {
 s2.append("B");
 System.out.print(s1);
 System.out.print(s2);
 }
 }
 }
 }.start();
 
 new Thread() {
 public void run() {
 synchronized (s2) {
 s2.append("C");
 synchronized (s1) {
 s1.append("D");
 System.out.print(s2);
 System.out.print(s1);
 }
 }
 }
 }.start();
 }
}
```

**1.4 线程的通信**

> 在 java.lang.Object 类中：
> 
> wait() : 使当前“同步监视器”上的线程进入等待状态。同时释放锁
> 
> notify() / notifyAll() : 唤醒当前“同步监视器”上的（一个/所有）等待状态的线程
> 
> 注意：上述方法必须使用在同步中

**场景 1：使用两个线程打印 1-100 线程 1 和线程 2 交替打印**

```
public class MyThread implements Runnable{
 
 int i = 0;
 
 @Override
 public void run() {
 
 while(true){
 synchronized (this) {
 this.notify();
 
 if(i <= 100){
 System.out.println(Thread.currentThread().getName() + "=" + i++);
 }
 
 try {
 this.wait();
 } catch (InterruptedException e) {
 }
 }
 
 }
 
 }
 
}
public class TestThread4 {
 public static void main(String[] args) {
 MyThread myThread = new MyThread();
 
 Thread t1 = new Thread(myThread,"线程1");
 Thread t2 = new Thread(myThread,"线程2");
 
 t1.start();
 t2.start();
 }
}
```

**经典例题:生产者/消费者问题**

*   生产者(Productor)将产品交给店员(Clerk)，而消费者(Customer)从店员处取走产品，
    
*   店员一次只能持有固定数量的产品(比如:20），如果生产者试图生产更多的产品，店员会叫生产者停一下，
    
*   如果店中有空位放产品了再通知生产者继续生产；
    
*   如果店中没有产品了，店员会告诉消费者等一下，如果店中有产品了再通知消费者来取走产品。
    

```
public class TestProduct {
 
 public static void main(String[] args) {
 Clerk clerk = new Clerk();
 
 Productor pro = new Productor(clerk);
 Customer cus = new Customer(clerk);
 
 new Thread(pro).start();
 new Thread(cus).start();
 }
}

class Clerk {
 private int product;
 
 public synchronized void getProduct() {
 if (product >= 20) {
 System.out.println("产品已满！");
 
 try {
 wait();
 } catch (InterruptedException e) {
 }
 
 } else {
 System.out.println("生产者生产了第" + ++product + " 个产品");
 
 notifyAll();
 }
 }
 
 public synchronized void saleProduct() {
 if (product <= 0) {
 System.out.println("缺货！");
 
 try {
 wait();
 } catch (InterruptedException e) {
 }
 
 } else {
 System.out.println("消费者消费了第" + --product + " 个产品");
 
 notifyAll();
 }
 }
}

class Productor implements Runnable {
 private Clerk clerk;
 public Productor() {
 }
 public Productor(Clerk clerk) {
 this.clerk = clerk;
 }
 public Clerk getClerk() {
 return clerk;
 }
 public void setClerk(Clerk clerk) {
 this.clerk = clerk;
 }
 @Override
 public void run() {
 while (true) {
 clerk.getProduct();
 }
 }
}

class Customer implements Runnable {
 private Clerk clerk;
 public Customer() {
 }
 public Customer(Clerk clerk) {
 this.clerk = clerk;
 }
 public Clerk getClerk() {
 return clerk;
 }
 public void setClerk(Clerk clerk) {
 this.clerk = clerk;
 }
 @Override
 public String toString() {
 return "Customer [clerk=" + clerk + "]";
 }
 @Override
 public void run() {
 while(true){
 clerk.saleProduct();
 }
 }
}
```