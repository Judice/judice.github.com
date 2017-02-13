---
layout:     post
title:      "Python的协程、线程、并发、并行"
subtitle:   "协程的概述"
date:       2017-02-12
author:     “Aaron”
header-img: "img/post-14.jpg"
tags:
    - 协程
    - 线程
    - 并发
    - 并行
---

### 概括

> allowing multiple entry points for suspending and resuming execution at certain locations.
  允许多个入口对程序进行挂起、继续执行等操作

协程（coroutine）也叫作轻量级线程。**通俗讲，协程是由一系列的子程序协同完成一个任务**，这些子程序可以主动挂起交出控制权，当恢复执行的时候，可以从挂起的位置继续执行，而这一切的调度由用户操作，而不是操作系统。所以有人称，协程是用户态线程。

在python中，协程是通过生成器实现的，**yeild就可以保存当前子程序上下文，并交出控制权，使用send就可以传递数据并恢复相应子程序**。这样多个生成器子程序，就可以通过yield和send相互协作完成任务。

很多人分不清楚协程和线程和进程的关系。**简单的说就是: 线程和进程的调度是由操作系统来调控， 而协程的调度由用户自己调控。** 所以协程调度器可以在协程A即将进入阻塞IO操作，比如 socket 的 read （其实已经设置为异步IO ）之前，将该协程挂起，把当前的栈信息 StackA 保存下来，然后切换到协程B，等到协程A的该 IO操作返回时，再根据 StackA 切回到之前的协程A当时的状态。

- 协程的优势

协程的特点在于是一个线程执行，最大的优势就是协程极高的执行效率。因为子程序切换不是线程切换，而是由程序自身控制，因此，没有线程切换的开销，和多线程比，线程数量越多，协程的性能优势就越明显。第二大优势就是不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。

**多进程+协程**，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。

- python的协程与生成器的关系

Python对协程的支持还非常有限，用在generator中的yield可以一定程度上实现协程，yield保存上下文，主动交出控制权的特性已经很接近协程了。
* yield从语句变为表达式, 这个是为了传值方便
* 加入send()方法用于在恢复生成器的时候，传入值
* 加入close()方法用于结束协程
* 加入throw()方法用于传入异常

```python
# coding=utf-8
import time

def consume():
    r = ''
    while True:
        n = yield r # yeild就可以保存当前子程序上下文，并交出控制权
        print 'Consume %s' % n
        r = '200ok'
        time.sleep(1)

def produce(c):
    c.next()
    n = 0
    while n < 5:
        print 'produce %s' % n
        r = c.send(n)
        print 'consume return %s' % r
        n += 1
    c.close()

if __name__ == "__main__":
    c = consume()
    produce(c)

执行结果：

produce 0
Consume 0
consume return 200ok
produce 1
Consume 1
consume return 200ok
produce 2
Consume 2
consume return 200ok
produce 3
Consume 3
consume return 200ok
produce 4
Consume 4
consume return 200ok
```

注意到consumer函数是一个generator（生成器），把一个consumer传入produce后：

* 首先调用c.next()启动生成器，切换至consume函数，通过yield保存上下文，在交出控制权，切换到produce函数；
* 一旦生产了东西，通过c.send(n)切换到consume执行，通过send传递数据，并恢复切换到相应子程序；
* consume通过yield拿到消息，处理，又通过yield把结果传回；
* produce拿到consume处理的结果，继续生产下一条消息；
* produce决定不生产了，通过c.close()关闭consume，整个过程结束。

**整个流程无锁，由一个线程执行，produce和consumer协作完成任务，所以称为“协程”，而非线程的抢占式多任务。**

- 协程的几个例子

```Python
# coding=utf-8
def jump_range(upper):
    index = 0
    while index < upper:
        jump = yield index
        if jump is None:
            jump = 1
        index += jump

jump = jump_range(5)


print jump.send(None)
# 相当于jump.next() 启动生成器 第一次 yield 0 然后交出控制权到 jump.send(3)
print jump.send(3)
print jump.send(None)

输出结果：
0
3
4
```

在python3.x ,后来又新增了 *yield from* 语法，可以将生成器串联起来：

```python
def wait_index(i):
    # processing i...
    return (yield i)

def jump_range(upper):
    index = 0
    while index < upper:
        jump = yield from wait_index(index)
        if jump is None:
            jump = 1
        index += jump

jump = jump_range(5)


print(jump.send(None))
print(jump.send(3))
print(jump.send(None))
```

yield from/send 似乎已经满足了协程所定义的需求，最初也确实是用 。@types.coroutine 修饰器将生成器转换成协程来使用，在 Python 3.5 之后则以专用的 async/await 取代了 @types.coroutine/yield from：

```Python
class Wait(object):
    """
    由于 Coroutine 协议规定 await 后只能跟 awaitable 对象，
    而 awaitable 对象必须是实现了 __await__ 方法且返回迭代器
    或者也是一个协程对象，
    因此这里临时实现一个 awaitable 对象。
    """
    def __init__(self, index):
        self.index = index

    def __await__(self):
        return (yield self.index)

async def jump_range(upper):
    index = 0
    while index < upper:
        jump = await Wait(index)
        if jump is None:
            jump = 1
        index += jump

jump = jump_range(5)


print(jump.send(None))
print(jump.send(3))
print(jump.send(None))
```

与线程比协程执行结果如下：

```Python
import asyncio
import time
import types

@types.coroutine
def _sum(x, y):
    print("Compute {} + {}...".format(x, y))
    yield time.sleep(1) # 线程阻塞1秒后,切换控制权到 yield from,再次取得控制权执行return
    return x+y

@types.coroutine
def compute_sum(x, y):
    result = yield from _sum(x, y) # 切换控制权到_sum
    print("{} + {} = {}".format(x, y, result))

loop = asyncio.get_event_loop()
loop.run_until_complete(compute_sum(1,2))

输出结果是：

Compute 1 + 2...
1 + 2 = 3
```
![py-01](/img/in-post/post-py-version/py-01.png)

这张图（来自: PyDocs: 18.5.3. Tasks and coroutines）清楚地描绘了由事件循环调度的协程的执行过程，上面的例子中事件循环的队列里只有一个协程，如果要与上一部分中线程实现的并发的例子相比较，只要向事件循环的任务队列中添加协程即可：

```python
import asyncio  
import time

# 上面的例子为了从生成器过度，下面全部改用 async/await 语法
async def _sum(x, y):  
    print("Compute {} + {}...".format(x, y))
    await asyncio.sleep(2.0)
    return x+y

async def compute_sum(x, y):  
    result = await _sum(x, y)
    print("{} + {} = {}".format(x, y, result))

start = time.time()  
loop = asyncio.get_event_loop()

tasks = [  
    asyncio.ensure_future(compute_sum(0, 0)),
    asyncio.ensure_future(compute_sum(1, 1)),
    asyncio.ensure_future(compute_sum(2, 2)),
]

loop.run_until_complete(asyncio.wait(tasks))  
loop.close()  
print("Total elapsed time {}".format(time.time() - start))

输出结果：
Compute 0 + 0...
Compute 1 + 1...
Compute 2 + 2...
0 + 0 = 0
1 + 1 = 2
2 + 2 = 4
Total elapsed time 2.0042951107025146
```

### 线程

由于线程是操作系统直接支持的执行单元，因此，高级语言通常都内置多线程的支持，Python也不例外。
启动一个线程就是把一个函数传入并创建Thread实例，然后调用start()开始执行：

```Python
from threading import Thread
import time

def _sum(x, y):
    print 'compute {} + {}'.format(x, y)
    time.sleep(2)
    return x+y

def compute_sum(x,y):
    result = _sum(x,y)
    print 'compute {} + {} = {}'.format(x, y, result)

start = time.time()
threads = [Thread(target=compute_sum, args=(0,0)),
           Thread(target=compute_sum, args=(1,1)),
           Thread(target=compute_sum, args=(2,2))]
for t in threads:
    t.start()
for t in threads:
    t.join()
print 'total time:{}'.format(time.time()-start)

# do not use thread
start = time.time()
compute_sum(0,0)
compute_sum(1,1)
compute_sum(2,2)
print 'total time:{}'.format(time.time()-start)
```

除了通过将函数传递给 Thread 创建线程实例之外，还可以直接继承 Thread 类：

```python
# coding=utf-8
import time
from threading import Thread

class Computer_sum(Thread):
    def __init__(self,x,y):
        super(Computer_sum, self).__init__()
        self.x = x
        self.y = y

    def run(self):
        res = self._sum(self.x,self.y)
        print 'computer {} + {} = {}'.format(self.x, self.y, res)

    def _sum(self,x,y):
        print 'computer {} + {}'.format(x,y)
        time.sleep(2) # 此处会发生阻塞,故打印结果随机排列
        return x+y

threading = [Computer_sum(0,0), Computer_sum(1,1), Computer_sum(2,2)]

start = time.time()
for t in threading:
    t.start()
for t in threading:
    t.join()
print 'total time:{}'.format(time.time()-start)
```

- 线程锁

多线程和多进程最大的不同在于，多进程中，同一个变量，各自有一份拷贝存在于每个进程中，互不影响，而多线程中，所有变量都由所有线程共享，所以，任何一个变量都可以被任何一个线程修改，因此，线程之间共享数据最大的危险在于多个线程同时改一个变量，把内容给改乱了。

如果我们要确保balance计算正确，就要给change_it()上一把锁，当某个线程开始执行change_it()时，我们说，该线程因为获得了锁，因此其他线程不能同时执行change_it()，只能等待，直到锁被释放后，获得该锁以后才能改。由于锁只有一个，无论多少线程，同一时刻最多只有一个线程持有该锁，所以，不会造成修改的冲突

```python
# coding=utf-8
import threading
balance = 0
lock = threading.Lock()

def change_it(n):
    global balance
    balance = balance + n
    balance = balance - n

def run_threading(n):
    lock.acquire()   # 可以使用with
    try:
       for i in range(100000):
           change_it(n)
    finally:
        lock.release()



t1 = threading.Thread(target=run_threading, args=(5,))
t2 = threading.Thread(target=run_threading, args=(8,))
t1.start()
t2.start()
t1.join()
t2.join()
print balance
```

当多个线程同时执行lock.acquire()时，只有一个线程能成功地获取锁，然后继续执行代码，其他线程就继续等待直到获得锁为止。

获得锁的线程用完后一定要释放锁，否则那些苦苦等待锁的线程将永远等待下去，成为死线程。所以我们用try...finally来确保锁一定会被释放,**也可以用 with 上下文管理器来管理锁的获取和释放** 例如：

```python
import threading
balance = 0
lock = threading.Lock()

def change_it(n):
    global balance
    balance = balance + n
    balance = balance - n

def run_threading(n):
    with lock：
         for i in range(100000):
            change_it(n)
```

锁的好处就是确保了某段关键代码只能由一个线程从头到尾完整地执行，坏处当然也很多，首先是阻止了多线程并发执行，包含锁的某段代码实际上只能以单线程模式执行，效率就大大地下降了。其次，由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁，导致多个线程全部挂起，既不能执行，也无法结束，只能靠操作系统强制终止。

- 死锁

线程的一大问题就是通过加锁来”抢夺“共享资源的时候有可能造成死锁，例如下面的程序：

```python
import threading
from threading import Lock
_base_lock = Lock()
_pos_lock  = Lock()


def _sum(x, y):
    # Time 1
    with _base_lock:
        # Time 3
        with _pos_lock:
            result = x + y
    print result

def _minus(x, y):
    # Time 0
    with _pos_lock:
        # Time 2
        with _base_lock:
            result = x - y
    print result
```

由于线程的调度执行顺序是不确定的，在执行上面两个线程 *_sum* *_minus* 的时候就有可能出现注释中所标注的时间顺序，即 # Time 0 的时候运行到 *with _pos_lock* 获取了 *_pos_lock* 锁，而接下来由于阻塞马上切换到了 *_sum* 中的 # Time 1 ，并获取了 *_base_lock*,接下来由于两个线程互相锁定了彼此需要的下一个锁，将会导致死锁，即程序无法继续运行。而与线程相比，协程（尤其是结合事件循环）无论在编程模型还是语法上，看起来都是非常友好的单线程同步过程。

## 并发与并行

关于并发，最好的例子应该是gevent吧，基本的原理是，将函数变为协程，每触发I/O阻塞就yield交出控制权，并将事件注册到epoll，当I/O就绪就是用send方法，传入I/O数据，并恢复逻辑。这么描述其实和tornado、nodejs的网络模型很像，但是协程对于程序员更加友好。tornado和nodejs默认还是通过回调函数完成这个事件循环的，这样代码并不直观，但使用协程可以用同步的方式完成回调函数的工作。

**并发：**

并发当有多个线程在操作时,如果系统只有一个CPU,则它根本不可能真正同时进行一个以上的线程，它只能把CPU运行时间划分成若干个时间段,再将时间 段分配给各个线程执行，在一个时间段的线程代码运行时，其它线程处于挂起状。.这种方式我们称之为并发(Concurrent)。

 **并行：**

 当系统有一个以上CPU时,则线程的操作有可能非并发。当一个CPU执行一个线程时，另一个CPU可以执行另一个线程，两个线程互不抢占CPU资源，可以同时进行，这种方式我们称之为并行(Parallel)。
