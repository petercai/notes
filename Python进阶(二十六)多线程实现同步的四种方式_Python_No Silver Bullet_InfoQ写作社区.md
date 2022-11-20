# Python进阶(二十六)多线程实现同步的四种方式_Python_No Silver Bullet_InfoQ写作社区
一、前言
----

临界资源即那些一次只能被一个线程访问的资源，典型例子就是打印机，它一次只能被一个程序用来执行打印功能，因为不能多个线程同时操作，而访问这部分资源的代码通常称之为临界区。

二、锁机制
-----

`threading`的`Lock`类，用该类的`acquire`函数进行加锁，用`realease`函数进行解锁。

```
import threading
import time
 
class Num:
    def __init__(self):
        self.num = 0
        self.lock = threading.Lock()
    def add(self):
        self.lock.acquire()
        self.num += 1
        num = self.num
        self.lock.release()
        return num
 
n = Num()
class jdThread(threading.Thread):
    def __init__(self,item):
        threading.Thread.__init__(self)
        self.item = item
    def run(self):
        time.sleep(2)
        value = n.add()
        print(self.item,value)
 
for item in range(5):
    t = jdThread(item)
    t.start()
    t.join()
```

当一个线程调用锁的`acquire()`方法获得锁时，锁就进入“`locked`”状态。每次只有一个线程可以获得锁。如果此时另一个线程试图获得这个锁，该线程就会变为“`blocked`”状态，称为“**同步阻塞**”（参见多线程的基本概念）。

直到拥有锁的线程调用锁的`release()`方法释放锁之后，锁进入“`unlocked`”状态。线程调度程序从处于**同步阻塞**状态的线程中选择一个来获得锁，并使得该线程进入运行（`running`）状态。

三、信号量
-----

信号量也提供`acquire`方法和`release`方法，每当调用`acquire`方法的时候，如果内部计数器大于 0，则将其减 1，如果内部计数器等于 0，则会阻塞该线程，直到有线程调用了`release`方法将内部计数器更新到大于 1 位置。

```
import threading
import time
class Num:
    def __init__(self):
        self.num = 0
        self.sem = threading.Semaphore(value = 3)
        
 
    def add(self):
        self.sem.acquire()
        self.num += 1
        num = self.num
        self.sem.release()
        return num
     
n = Num()
class jdThread(threading.Thread):
    def __init__(self,item):
        threading.Thread.__init__(self)
        self.item = item
    def run(self):
        time.sleep(2)
        value = n.add()
        print(self.item,value)
 
for item in range(100):
    t = jdThread(item)
    t.start()
    t.join()
```

四、条件判断
------

所谓条件变量，即这种机制是在满足了特定的条件后，线程才可以访问相关的数据。

它使用`Condition`类来完成，由于它也可以像锁机制那样用，所以它也有`acquire`方法和`release`方法，而且它还有`wait，notify，notifyAll`方法。

```
"""
一个简单的生产消费者模型，通过条件变量的控制产品数量的增减，调用一次生产者产品就是+1，调用一次消费者产品就会-1.
"""
 
"""
使用 Condition 类来完成，由于它也可以像锁机制那样用，所以它也有 acquire 方法和 release 方法，而且它还有
wait， notify， notifyAll 方法。
"""
 
import threading
import queue,time,random
 
class Goods:
    def __init__(self):
        self.count = 0
    def add(self,num = 1):
        self.count += num
    def sub(self):
        if self.count>=0:
            self.count -= 1
    def empty(self):
        return self.count <= 0
 
class Producer(threading.Thread):
    def __init__(self,condition,goods,sleeptime = 1):
        threading.Thread.__init__(self)
        self.cond = condition
        self.goods = goods
        self.sleeptime = sleeptime
    def run(self):
        cond = self.cond
        goods = self.goods
        while True:
            cond.acquire()
            goods.add()
            print("产品数量:",goods.count,"生产者线程")
            cond.notifyAll()
            cond.release()
            time.sleep(self.sleeptime)
 
class Consumer(threading.Thread):
    def __init__(self,condition,goods,sleeptime = 2):
        threading.Thread.__init__(self)
        self.cond = condition
        self.goods = goods
        self.sleeptime = sleeptime
    def run(self):
        cond = self.cond
        goods = self.goods
        while True:
            time.sleep(self.sleeptime)
            cond.acquire()
            while goods.empty():
                cond.wait()
            goods.sub()
            print("产品数量:",goods.count,"消费者线程")
            cond.release()
 
g = Goods()
c = threading.Condition()
 
pro = Producer(c,g)
pro.start()
 
con = Consumer(c,g)
con.start()
```

五、同步队列
------

`put`方法和`task_done`方法，`queue`有一个未完成任务数量`num`，`put`依次`num+1`，`task`依次`num-1`。任务都完成时任务结束。

```
import threading
import queue
import time
import random
 
'''
1.创建一个 Queue.Queue() 的实例，然后使用数据对它进行填充。
2.将经过填充数据的实例传递给线程类，后者是通过继承 threading.Thread 的方式创建的。
3.每次从队列中取出一个项目，并使用该线程中的数据和 run 方法以执行相应的工作。
4.在完成这项工作之后，使用 queue.task_done() 函数向任务已经完成的队列发送一个信号。
5.对队列执行 join 操作，实际上意味着等到队列为空，再退出主程序。
'''
 
class jdThread(threading.Thread):
    def __init__(self,index,queue):
        threading.Thread.__init__(self)
        self.index = index
        self.queue = queue
 
    def run(self):
        while True:
            time.sleep(1)
            item = self.queue.get()
            if item is None:
                break
            print("序号：",self.index,"任务",item,"完成")
            self.queue.task_done()
 
q = queue.Queue(0)
'''
初始化函数接受一个数字来作为该队列的容量，如果传递的是
一个小于等于0的数，那么默认会认为该队列的容量是无限的.
'''
for i in range(2):
    jdThread(i,q).start()
 
for i in range(10):
    q.put(i)
```