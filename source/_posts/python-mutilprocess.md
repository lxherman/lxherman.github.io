---
title: 多线程&多进程(上)
date: 2019-12-30 21:04:51
tags:
- python
- 多线程
- 多进程
categories: 
- python
---
&#160; &#160; &#160; &#160;大学的时候面试，时常被问到线程和进程的区别。时至今日，碰到爬虫中正好也要运用，再拿出来梳理一波。

首先明确一些概念：
>- 计算机的核心是CPU，承担了所有的计算任务。一个CPU，在一个时间切片里只能运行一个程序。
>- 一个cpu一次只能执行一个进程，其它进程处于非运行状态。  
<!--more-->
>- **进程：表示程序的一次执行**(打开、执行、保存...),一个进程可以包含多个线程。
>- **线程：进程执行程序时候的最小调度单位**，执行a，执行b...)。

>- 每个进程都有自己独立的内存空间，**不同进程之间的内存空间不共享**。
>- **单个进程的内存空间是共享的**，每个进程里的线程共享进程的内存空间，*通讯效率高，切换开销小*。

>- 一条线程指的是进程中一个单一顺序的控制流。
>- 一个线程在使用这个共享空间的时候，其它的线程必须等待（阻塞状态）。

>- **协程**：又称微线程，在单线程上执行多个任务，用函数切换，开销极小。不通过操作系统调度，没有进程、线程的切换开销。

>- **python的多进程适用于大量的密集并行计算**。
>    - cpu密集型：cpu使用率较高（一些复杂计算或者逻辑处理过程）。
>    - **多进程缺陷：多个进程之间通信成本高，切换开销大**。

>- **python的多线程适用于大量密集的I/O处理**(网络 - I/O，磁盘I/O，数据库I/O--读写文件、在网络间通信、以及与显示器等设备相交互等)
>    - 程序中会存在大量I/O操作占据时间，导致线程空余时间出来。
>    - **多线程缺陷**：同一个时间切片只能运行一个线程，不能做到高并行，但是可以做到高并发。共享内存意味着竞争，导致数据不安全，为了保护内存空间的数据安全，引入"*互斥锁* "。

>- **并行**：多个CPU核心，不同的程序就分配给不同的CPU来运行。可以让多个程序同时执行。
>- **并发**：单个CPU核心，在一个时间切片里一次只能运行一个程序，如果需要运行多个程序，则交替执行。

>- **GIL**(Global Interpreter Lock**全局解释器锁**)
>    - python为什么不能发挥多核cpu的优势，主要就是因为GIL。GIL就像是一个通行证，拿到通行证的线程就可以进入CPU执行任务。没有GIL的线程就不能执行任务。宏观看来，**其也是一把互斥锁，控制了一个进程中同一时刻只能有一个线程在执行**。因为在python的字节码解释器执行程序的时候，必须是协调一致的，不能允许其他线程打断，否则会破坏解释器的状态。

>- **互斥锁**
>    - **python解释器会给每个线程都分配大致相等的处理器时间**，由于**GIL并不会保护开发者自己写的代码**，所以当代码中需要某些**原子操作**的时候，就要自行添加互斥锁，保证一个线程在操作数据结构时，不被其他线程干扰，而破坏了数据结构的一致性。
>    - Python 在 **threading** 模块中提供了最简单、最有用的工具：Lock 类，该类相当于互斥锁。在开发中我们可以使用互斥锁来保护某个对象，使得在多线程同时访问某个对象的时候，不会将该对象破坏。
___
&#160; &#160; &#160; &#160;下面来看看python中如何实现多线程和多进程。
## 多线程
&#160; &#160; &#160; &#160;**threading**模块介绍：它是python中专门提供用来做多线程编程的模块。
  
[threading模块常用的类(脑图)](http://naotu.baidu.com/file/8d8bb7f6c8e9bf9d831d12ff7f8f83a8?token=05e872c76bd75221)
  
&#160; &#160; &#160; &#160;threading模块中最常用的类是Thread，包含以下方法：

|方法|说明|
|:--:|:--|
|start()|线程准备就绪，等待CPU调度|
|join()|逐个执行每个线程，执行完毕后继续往下执行|
|run()|线程被调度后会执行该方法，如果想自定义线程类，需要重写run()方法|
|is_alive()|返回线程是否在运行|
|setName()|为线程设置名称|
|getName()|获取线程名称|
|setDaemon()|设置为守护线程|
### 1. 线程的普通创建方式
&#160; &#160; &#160; &#160;主线程不会等待子线程执行完再结束。(计算花费时间时，主线程已经结束)
```
import threading
import time
 
def coding():
    for x in range(3):
        print('%s正在写代码' % x)
        time.sleep(1)
 
def drawing():
    for x in range(3):
        print('%s正在画图' % x)
        time.sleep(1)
 
def single_thread():
    coding()
    drawing()
 
def multi_thread():
    t1 = threading.Thread(target=coding)
    t2 = threading.Thread(target=drawing)
 
    t1.start()
    t2.start()
 
if __name__ == '__main__':
    start_time = time.time()
    single_thread()
    print('花费的时间为:',time.time() - start_time)
    print('--'*10)
    start_time = time.time()
    multi_thread()
    print('花费的时间为:',time.time() - start_time)

```
输出为：
`0正在写代码`
`1正在写代码`
`2正在写代码`
`0正在画图`
`1正在画图`
`2正在画图`
`花费的时间为: 6.004063367843628`
`--------------------`
`0正在写代码`
`0正在画图`
`花费的时间为: 0.011080503463745117`
`1正在写代码`
`1正在画图`
`2正在写代码`
`2正在画图`
  
### 2. 自定义线程类
&#160; &#160; &#160; &#160;为了让线程代码更好的封装。可以使用threading模块下的Thread类，继承自threading.Thread这个类，然后实现run方法，线程就会自动运行run方法中的代码。
```
import threading
import time
 
class CodingThread(threading.Thread):
    def run(self):
        for x in range(3):
            print('%s正在写代码' % threading.current_thread())
            time.sleep(1)
 
class DrawingThread(threading.Thread):
    def run(self):
        for x in range(3):
            print('%s正在画图' % threading.current_thread())
            time.sleep(1)
 
def multi_thread():
    t1 = CodingThread()
    t2 = DrawingThread()
 
    t1.start()
    t2.start()
 
if __name__ == '__main__':
    multi_thread()

```
  
### 3. 计算子线程执行的时间
&#160; &#160; &#160; &#160;主线程不会等待子线程执行完毕再结束自身。可以使用Thread类的**join()**方法来使得子线程执行完毕以后，主线程再关闭。
```
import threading
import time


def coding():
    for x in range(3):
        print('%s正在写代码' % x)
        time.sleep(1)
 
def drawing():
    for x in range(3):
        print('%s正在画图' % x)
        time.sleep(1)
 
 
def single_thread():
    coding()
    drawing()
 
def multi_thread():
    t1 = threading.Thread(target=coding)
    t2 = threading.Thread(target=drawing)
 
    t1.start()
    t1.join()
    t2.start()
    t2.join()

if __name__ == "__main__":
    start_time = time.time()
    multi_thread()
    print('花费的时间为:',time.time() - start_time)

```
输出：
`0正在写代码`
`1正在写代码`
`2正在写代码`
`0正在画图`
`1正在画图`
`2正在画图`
`花费的时间为: 6.0084145069122314`
  

### 4. 守护线程 
&#160; &#160; &#160; &#160;线程的**setDaemon(True)**将线程变成主线程的守护线程，意思是当主进程结束后，子线程也会随之退出。意味着当主线程结束后，程序就结束了。
```
import threading
import time

def coding():
    for x in range(3):
        print('%s正在写代码' % x)
        time.sleep(1)
 
def drawing():
    for x in range(3):
        print('%s正在画图' % x)
        time.sleep(1)
 
def single_thread():
    coding()
    drawing()
 
def multi_thread():
    t1 = threading.Thread(target=coding)
    t2 = threading.Thread(target=drawing)
    t1.setDaemon(True)
    t1.start()
    t2.setDaemon(True)
    t2.start()

if __name__ == "__main__":
    start_time = time.time()
    multi_thread()
    print('花费的时间为:',time.time() - start_time)

```
输出：
`0正在写代码`
`0正在画图`
`花费的时间为: 0.0030105113983154297`
### 5. GIL(全局解释器锁)
&#160; &#160; &#160; &#160;在Python的运行环境中，无论电脑是单核还是双核，操作系统同时只会执行一个线程。究其原因，是因为GIL（全局解释器锁）。

&#160; &#160; &#160; &#160;在Python中，一个线程要想要执行，必须要先拿到GIL。可以吧GIL想象成一个“通行证”，并且在一个进程中，GIL只有一个。没有通行证的线程就不会被执行。
  
**Python多线程的工作过程：**
- 拿到公共数据
- 申请GIL
- Python解释器调用os的原生线程
- os操作CPU执行运算
- 当该线程的执行时间到了之后，无论是否执行完，GIL被释放
- 其他线程重复上面的操作
- 其他进程执行完成后，切换到原来的线程（从记录的上下文继续执行）

### 6. 线程锁（Lock,RLock）
&#160; &#160; &#160; &#160;多线程都是在同一个进程中运行的。因此在进程中的全局变量所有线程都是可共享的。这就造成了一个问题，因为线程执行的顺序是无序的。当多个线程同时修改同一条数据时可能会出现脏数据。例如：
```
import threading
 
tickets = 0
 
def get_ticket():
    global tickets
    for x in range(1000000):
        tickets += 1
    print('tickets:%d'%tickets)
 
def main():
    for x in range(2):
        t = threading.Thread(target=get_ticket)
        t.start()
 
if __name__ == '__main__':
    main()
```
输出：
`tickets:1150827`
`tickets:1227635`
- #### 互斥锁（Lock） 
&#160; &#160; &#160; &#160;为了解决以上使用共享全局变量的问题。threading提供了一个Lock类(互斥锁)，这个类可以在某个线程访问某个变量的时候加锁，其他线程此时就不能进来，直到当前线程处理完后，把锁释放了，其他线程才能进来处理。示例代码如下：
```
import threading
 
tickets = 0
gLock = threading.Lock()
#使用acquire/release
def get_ticket():
    global tickets
    gLock.acquire()
    for x in range(1000000):
        tickets += 1
    gLock.release()
    print('tickets:%d'%tickets)
 
#使用with lock（效果与上相同）
'''def get_ticket():
    global tickets
    with gLock:
        for x in range(1000000):
            tickets += 1
    print('tickets:%d'%tickets)'''

def main():
    for x in range(2):
        t = threading.Thread(target=get_ticket)
        t.start()
 
if __name__ == '__main__':
    main()
```
输出：
`tickets：1000000`
`tickets：2000000`
- #### 递归锁（RLock） 
&#160; &#160; &#160; &#160;Lock与RLock用法大部分是相同的，很多情况下可以通用，但有细微的区别：
在同一线程内，对Lock进行多次acquire()操作，程序会阻塞，而Rlock不会。所以在多个锁没有释放的时候一般会使用Rlock类。
  
### 7. 信号量（Semaphore） 
&#160; &#160; &#160; &#160;互斥锁同时只允许一个线程更改数据，而Semaphore是同时允许一定数量的线程更改数据 ，比如银行有3个窗口，那最多只允许3个人办理业务，后面的人只能等着，有人办理完了才能过去办理。示例代码如下:
```
import threading
 
tickets = 0
money = 0
gLock1 = threading.BoundedSemaphore(1)
gLock2 = threading.BoundedSemaphore(3)
#使用acquire/release
def get_ticket():
    global tickets
    gLock1.acquire()
    for x in range(1000000):
        tickets += 1
    gLock1.release()
    print('tickets:%d'%tickets)

def get_money():
    global money
    with gLock2:
        for x in range(1000000):
            money += 1
    print('money:%d'%money)


def main():
    for x in range(3):
        t1 = threading.Thread(target=get_ticket)
        t2 = threading.Thread(target=get_money)

        t1.start()
        t2.start()


 
if __name__ == '__main__':
    main()

```
输出：
`tickets:1000000`
`money:1466215`
`money:1865015`
`tickets:2000000`
`money:1524079`
`tickets:3000000`
&#160; &#160; &#160; &#160;可以看出，使用了gLock1(信号量为1)的get_ticket输出数据正常，而get_money使用gLock2信号量为3，允许3个线程同时访问共享资源，导致出现脏数据。
  
### 8. 事件（Event）
&#160; &#160; &#160; &#160;python线程的事件用于主线程控制其他线程的执行，事件是一个简单的线程同步对象，主要提供了以下几种方法:
|方法|说明|
|:--:|--|
|clear()|将flag设置为“false”|
|set()|将flag设置为“true”|
|is_set()|判断是否设置了flag|
|wait()|一直监听flag，没有检测到会一直处于阻塞状态|
&#160; &#160; &#160; &#160;事件处理的机制：全局定义了一个“Flag”，如果“Flag”值为 False，那么当程序执行 event.wait 方法时就会阻塞，如果“Flag”值为True，那么event.wait 方法时便不再阻塞。示例代码：
```
import threading,time

event = threading.Event()  # 创建事件对象

def lighter():
    count = 0
    event.set()   #初始值为绿灯
    while 1:
        if 5 < count <= 10:
            event.clear() #红灯，清楚标志位
            print('\33[41;1mred light is on...\033[0m')
        elif count > 10:
            event.set()   # 绿灯，设置标志位
            count = 0
        else:
            print('\33[41;1mred light is on...\033[0m')
        time.sleep(1)
        count += 1

def car(name):
    while True:
        if event.is_set():  # 判断是否设置了标志位
            print("[%s] 绿灯亮，请行驶..." % name)
            time.sleep(1)
        else:
            print("[%s] 红灯亮,请等待..." % name)
            event.wait()
            print("[%s] 绿灯亮,开始行驶..." % name)

light = threading.Thread(target=lighter,)

car = threading.Thread(target=car, args=('test',))
light.start()  
car.start()

```
输出：
![](/python-mutilprocess/1.png)

### 9. 条件（Condition） 
&#160; &#160; &#160; &#160;使得线程等待，只有满足某条件时，才释放n个线程。
&#160; &#160; &#160; &#160;python提供的Condition对象提供了对复杂线程同步问题的支持。Condition被称为条件变量，除了提供与Lock类似的acquire和release方法外，还提供了wait、notify和notify_all方法。
&#160; &#160; &#160; &#160;线程首先acquire一个条件变量，然后判断一些条件。如果条件不满足则wait；如果条件满足，进行一些处理改变条件后，通过notify方法通知其他线程，其他处于wait状态的线程接到通知后会重新判断条件。不断的重复这一过程，从而解决复杂的同步问题。
&#160; &#160; &#160; &#160;可以认为Condition对象维护了一个锁（Lock/RLock)和一个waiting池。线程通过acquire获得Condition对象，当调用wait方法时，线程会释放Condition内部的锁并进入blocked状态，同时在waiting池中记录这个线程。当调用notify方法时，Condition对象会从waiting池中挑选一个线程，通知其调用acquire方法尝试取到锁。
&#160; &#160; &#160; &#160;Condition对象的构造函数可以接受一个Lock/RLock对象作为参数，如果没有指定，则Condition对象会在内部自行创建一个RLock。
&#160; &#160; &#160; &#160;除了notify方法外，Condition对象还提供了notifyAll方法，可以通知waiting池中的所有线程尝试acquire内部锁。由于上述机制，处于waiting状态的线程只能通过notify方法唤醒，所以notifyAll的作用在于防止有线程永远处于沉默状态。
  
&#160; &#160; &#160; &#160;**后面将对比Lock/RLock和condition版本的生产者与消费者的问题。**

### 10. 生产者和消费者模式
&#160; &#160; &#160; &#160;生产者和消费者模式是多线程开发中经常见到的一种模式。生产者的线程专门用来生产一些数据，然后存放到一个中间的变量中。消费者再从这个中间的变量中取出数据进行消费。但是因为要使用中间变量，中间变量经常是一些全局变量，因此需要使用锁来保证数据完整性。
- 以下是一个使用Lock锁的实现：
```
#lock版本
import threading
import random
import time

wallet = 1000
lock = threading.Lock()

class Producer(threading.Thread):
    def run(self):
        global wallet
        while True:
            money = random.randint(500,1000)
            lock.acquire()
            #钱包里大于两千元，就停止存入
            if wallet > 2000:
                lock.release()
                break
            else:
                wallet += money
                print("%s存入%s元钱，还剩%s元钱" % (threading.current_thread(), money, wallet))
                time.sleep(1)
            lock.release()
            time.sleep(2)

class Consumer(threading.Thread):
    def run(self):
        global wallet
        while True:
            money = random.randint(100,500)
            lock.acquire()
            #钱包里钱不够取了，就停止取钱
            if wallet < money:
                print("%s取出%s元钱，还剩%s元钱，余额不足" % (threading.current_thread(), money, wallet))
                lock.release()
                break
            else:               
                wallet -= money
                print("%s取出%s元钱，还剩%s元钱" % (threading.current_thread(), money, wallet))
                time.sleep(1)
            lock.release()            

def main():
    for x in range(3):
        Consumer(name='消费者线程%d'%x).start()
    for x in range(5):
        Producer(name='生产者线程%d'%x).start()            

if __name__ == "__main__":
    main()

```

&#160; &#160; &#160; &#160;虽然lock版本的生产者消费者模式能正常运行，但是这种方式存在一种弊端，一直使用while True死循环上锁判断钱都不够的方法是很耗费cpu资源的一种行为，还有一种更好的方式便是使用threading.Condition来实现。
&#160; &#160; &#160; &#160;threading.Condition可以在没有数据的时候处于阻塞等待状态。一旦有合适的数据了，还可以使用notify相关的函数来通知其他处于等待状态的线程。这样就可以不用做一些无用的上锁和解锁的操作。可以提高程序的性能。首先对threading.Condition相关的函数做个介绍，threading.Condition类似threading.Lock，可以在修改全局数据的时候进行上锁，也可以在修改完毕后进行解锁。
- condition的方法介绍：
   - acquire：上锁。
   - release：解锁。
   - wait：将当前线程处于等待状态，并且会释放锁。可以被其他线程使用notify和notify_all函数唤醒。被唤醒后会继续等待上锁，上锁后继续执行下面的代码。
   - notify：通知某个正在等待的线程，默认是第1个等待的线程。
   - notify_all：通知所有正在等待的线程。notify和notify_all不会释放锁。并且需要在release之前调用。

- 以下是使用condition的实现：
```
#condition版本
import threading
import random
import time

wallet = 1000
con = threading.Condition()

class Producer(threading.Thread):
    def run(self):
        global wallet
        while True:
            money = random.randint(500,1000)
            con.acquire()
            #钱包里大于两千元，就暂停存入
            if wallet > 2000:
                con.wait()
            else:
                wallet += money
                print("%s存入%s元钱，还剩%s元钱" % (threading.current_thread(), money, wallet))
                time.sleep(1)
                con.notify()
            con.release()
            time.sleep(2)
    
class Consumer(threading.Thread):
    def run(self):
        global wallet
        while True:
            money = random.randint(100,500)
            con.acquire()
            #钱包里钱不够取了，就暂停取钱
            if wallet < money:
                print("%s取出%s元钱，还剩%s元钱，余额不足" % (threading.current_thread(), money, wallet))
                con.wait()
            else:               
                wallet -= money
                print("%s取出%s元钱，还剩%s元钱" % (threading.current_thread(), money, wallet))
                time.sleep(1)
                con.notify()
            con.release()

def main():
    for x in range(2):
        Consumer(name='消费者线程%d'%x).start()
    for x in range(1):
        Producer(name='生产者线程%d'%x).start()

if __name__ == "__main__":
    main()

```

>还需要时间沉淀以下（关于多线程应用于生产者消费者模型的问题）。
### 11. queue线程安全队列
&#160; &#160; &#160; &#160;在线程中，访问一些全局变量，加锁是一个经常的过程。如果你是想把一些数据存储到某个队列中，那么Python内置了一个线程安全的模块叫做queue模块。Python中的queue模块中提供了同步的、线程安全的队列类，包括FIFO（先进先出）队列Queue，LIFO（后入先出）队列LifoQueue。这些队列都实现了锁原语（可以理解为原子操作，即要么不做，要么都做完），能够在多线程中直接使用。可以使用队列来实现线程间的同步。相关的函数如下：
queue模块有三种队列及构造函数

- Python queue模块的FIFO队列先进先出。 queue.Queue(maxsize)

- LIFO类似于堆，即先进后出。 queue.LifoQueue(maxsize)
- 优先级队列级别越低越先出来。 queue.PriorityQueue(maxsize)
queue模块中的常用方法:

|方法|说明|
|--|--|
|queue.qsize()|返回队列的大小|
|queue.empty()|如果队列为空，返回True，反之False|
|queue.full()|如果队列满了，返回True，反之False (queue.full 与 maxsize 大小对应)|
|queue.get([block[, timeout]])|获取队列，立即取出**一个元素**， timeout-超时时间|
|queue.put(item[, timeout]])|写入队列，立即放入**一个元素**， timeout-超时时间|
|queue.join()|阻塞调用线程，直到队列中的所有任务被处理掉, 实际上意味着等到队列为空，再执行别的操作|
|queue.task_done()|在完成一项工作之后，queue.task_done()函数向任务已经完成的队列发送一个信号|

详见：https://www.cnblogs.com/wl443587/p/9911721.html

____
#####一个多线程的面试题：
&#160; &#160; &#160; &#160;创建两个线程，其中一个输出1-52，另外一个输出A-Z。输出格式要求：12A 34B 56C 78D。
```
import threading
import time


# 大致思路
# 获取对方的锁，运行一次后，释放自己的锁
def show1():
    for i in range(1, 52, 2):
        lock_show2.acquire()
        print(i, end='')
        print(i+1, end='')
        time.sleep(0.2)
        lock_show1.release()


def show2():
    for i in range(26):
        lock_show1.acquire()
        print(chr(i + ord('A')))
        time.sleep(0.2)
        lock_show2.release()


lock_show1 = threading.Lock()
lock_show2 = threading.Lock()

show1_thread = threading.Thread(target=show1)
show2_thread = threading.Thread(target=show2)

lock_show1.acquire()  # 因为线程执行顺序是无序的，保证show1()先执行

show1_thread.start()
show2_thread.start()

```
·