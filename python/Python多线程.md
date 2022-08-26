---
abbrlink: 4097981062
alias: 2017/03/23/Python多线程/index.html
categories:
- python
date: '2017-03-23T01:57:30'
description: ''
tags:
- Python
- 多线程
- threading
title: Python多线程
---








# 多线程基础概念

**并行与并发**

- 并行：同时处理多个任务，必须在多核环境下
- 一段时间内同时处理多个任务，单核也可以并发

**并发手段**

- 线程：内核空间的调度
- 进程：内核空间的调度
- 协程：用户空间的调度

线程可以允许程序在同一进程空间中并发运行多个操作。本次主要介绍Python标准库中的多线程模块threading。

<!--more-->

# threading模块

## 线程初始化

使用threading模块的Thread类初始化对象然后调用start方法启动线程。

```python
import threading
import time

def worker(num):
    time.sleep(1)
    print('worker-{}'.format(num))

# 创建线程对象 target参数是一个函数， 这个函数即线程要执行的逻辑
threads = [threading.Thread(target=worker, args=(i, ))for i in range(5)]
for t in threads:
    t.start()
    # start 方法启动一个线程， 当这个线程的逻辑执行完毕的时候，线程自动退出, Python 没有提供主动退出线程的方法

# 输出以下结果
worker-0worker-1worker-2worker-3



worker-4
```

初始化的五个线程的执行逻辑中的print方法打印字符串及换行符出现了随机分布，即出现了资源竞争。

## 给线程传递参数

```python
import threading
import time

def worker(*args, **kwargs):
    time.sleep(1)
    print(args)
    print(kwargs)

threads = threading.Thread(target=worker, args=(1, 2, 3), kwargs={'a':'b'}).start()

# 输出
(1, 2, 3)
{'a': 'b'}
```

args传递位置参数，kwargs传递关键字参数。

## Thread常用参数和方法

```
>>> help(threading.Thread)
```

可以看到Thread函数的初始化方法中的参数如下：

```python
 |  __init__(self, group=None, target=None, name=None, args=(), kwargs=None, *, daemon=None)
 |      This constructor should always be called with keyword arguments. Arguments are:
 |      
 |      *group* should be None; reserved for future extension when a ThreadGroup
 |      class is implemented.
 |      
 |      *target* is the callable object to be invoked by the run()
 |      method. Defaults to None, meaning nothing is called.
 |      
 |      *name* is the thread name. By default, a unique name is constructed of
 |      the form "Thread-N" where N is a small decimal number.
 |      
 |      *args* is the argument tuple for the target invocation. Defaults to ().
 |      
 |      *kwargs* is a dictionary of keyword arguments for the target
 |      invocation. Defaults to {}.
```

### name

表示线程名称，默认情况下，线程名称是`Thread-N`，N是一个较小的十进制数。我们可以传递name参数，控制线程名称。

以下会导入logging模块来显示线程的名称等详细信息

```python
import threading
import time
import logging

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

def worker(num):
    time.sleep(1)
    logging.info('worker-{}'.format(num))

threads = [threading.Thread(target=worker, args=(i, ), name='workerthread-{}'.format(i)) for i in range(5)]
for t in threads:
    t.start()

# 输出
2017-03-20 21:39:29,339 INFO [workerthread-0] worker-0
2017-03-20 21:39:29,340 INFO [workerthread-1] worker-1
2017-03-20 21:39:29,340 INFO [workerthread-2] worker-2
2017-03-20 21:39:29,340 INFO [workerthread-3] worker-3
2017-03-20 21:39:29,346 INFO [workerthread-4] worker-4
```

其中logging模块的basicConfig函数的format中的%(threadName)s就是用来输出当前线程的名称的。

> 线程可以重名, 线程名并不是线程的唯一标识，但是通常应该避免线程重名，通常的处理手段是加前缀

### daemon

**Daemon：守护**

和Daemon线程相对应的还有Non-Daemon线程，在此Thread初始化函数中的daemon参数即表示线程是否是Daemon线程。

- Daemon线程：会伴随主线程结束而结束（可以理解为主线程结束，守护线程结束）
- Non-Daemon线程：不会随着主线程结束而结束，主线程需要等待Non-Daemon结束

```python
import logging
import time
import threading

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')


def worker():
    logging.info('starting')
    time.sleep(2)
    logging.info('stopping')


if __name__ == '__main__':
    logging.info('starting')
    t1 = threading.Thread(target=worker, name='worker1', daemon=False)
    t1.start()
    time.sleep(1)
    t2 = threading.Thread(target=worker, name='worker2', daemon=True)
    t2.start()
    logging.info('stopping')

# 输出
2017-03-20 23:28:06,404 INFO [MainThread] starting
2017-03-20 23:28:06,436 INFO [worker1] starting
2017-03-20 23:28:07,492 INFO [worker2] starting
2017-03-20 23:28:07,492 INFO [MainThread] stopping  # 主线程执行完成
2017-03-20 23:28:08,439 INFO [worker1] stopping  # 主线程执行完成之后会等Non-Daemon线程执行完成，但是并不会等Daemon线程执行完成，即Daemon线程会随着主线程执行完成而释放
```

### Thread.join()

如果想等Daemon线程执行完成之后主线程再退出，可以使用线程对象的`join()`方法

```python
import logging
import time
import threading

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')


def worker():
    logging.info('starting')
    time.sleep(2)
    logging.info('stopping')


if __name__ == '__main__':
    logging.info('starting')
    t1 = threading.Thread(target=worker, name='worker1', daemon=False)
    t1.start()
    time.sleep(1)
    t2 = threading.Thread(target=worker, name='worker2', daemon=True)
    t2.start()
    logging.info('stopping')
    t1.join()
    t2.join()

# 输出
2017-03-20 23:41:07,217 INFO [MainThread] starting
2017-03-20 23:41:07,243 INFO [worker1] starting
2017-03-20 23:41:08,245 INFO [worker2] starting
2017-03-20 23:41:08,246 INFO [MainThread] stopping
2017-03-20 23:41:09,243 INFO [worker1] stopping
2017-03-20 23:41:10,248 INFO [worker2] stopping
```

使用join函数只有主线程就需要等待Daemon线程执行完成在推出。

join函数的原型：`join(self, timeout=None)`

join方法会阻塞直到线程退出或者超时, timeout 是可选的，如果不设置timeout， 会一直等待线程退出。如果设置了timeout，会在超时之后退出或者线程执行完成退出。

因为join函数总是返回None，因此在超时时间到达之后如果要知道线程是否还是存活的，可以调用is_alive()方法判断线程是否存活。

## threading常用方法

### enumerate()

列出当前所有的**存活**的线程

```python
>>> threading.enumerate()
[<_MainThread(MainThread, started 140209670301504)>, <Thread(worker1, started 140209545410304)>, <Thread(worker2, started daemon 140209537017600)>]
```

### local()

```python
import logging
import threading

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

ctx = threading.local()
ctx.data = 5
data = 'a'

def worker():
    logging.info(data)
    logging.info(ctx.data)

worker()
threading.Thread(target=worker).start()

# 输出
2017-03-21 00:02:08,102 INFO [MainThread] a
2017-03-21 00:02:08,113 INFO [MainThread] 5
2017-03-21 00:02:08,119 INFO [Thread-34] a
Exception in thread Thread-34:
Traceback (most recent call last):
  File "/home/clg/.pyenv/versions/3.5.2/lib/python3.5/threading.py", line 914, in _bootstrap_inner
    self.run()
  File "/home/clg/.pyenv/versions/3.5.2/lib/python3.5/threading.py", line 862, in run
    self._target(*self._args, **self._kwargs)
  File "<ipython-input-28-5395bd925d87>", line 7, in worker
    logging.info(ctx.data)
AttributeError: '_thread._local' object has no attribute 'data'
```

线程共享内存、状态和资源。但是thread模块的local类的对象的属性， 只在当前线程可见。

## Thread类的派生

Python中可以通过继承 `Thread` 类并重写 `run` 方法来编写多线程的逻辑，此时逻辑函数就是run。

```python
class mythread(threading.Thread):
    def run(self):
        print('mythread run')

t = mythread()
t.run()  # 输出mythread run
t.start()  # 输出mythread run
```

通过继承方式派生而来的子类对象可以同时执行start方法和run方法，结果是一样的，都是执行子类的run方法。但是非继承的方式不能同时使用start方法和run方法，会报错。

**派生时逻辑函数的参数传递**

```python
class mythread(threading.Thread):
    def __init__(self, *args, **kwargs):
        super().__init__()  # 需要调用父类的初始化方法初始化
        self.args = args
        self.kwargs = kwargs

    def run(self):
        print('mythread run', self.args, self.kwargs)

t = mythread(1, 2, 3, a='b')
t.start()  # 输出mythread run (1, 2, 3) {'a': 'b'}
```

## Timer类

Timer类：Thread类的派生类，也在threading模块中。意为定时器，用作线程的延迟执行。

```python
>>> help(threading.Timer)
```

Timer类的初始化方法：`__init__(self, interval, function, args=None, kwargs=None)`

- interval：时间间隔，即几秒之后开始执行function
- function：线程执行的逻辑函数
- args：位置参数
- kwargs：关键字参数

**代码**

```python
import threading
import time
import logging

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

def worker():
    logging.info('worker running')

t1 = threading.Timer(interval=3, function=worker)
t2 = threading.Timer(interval=3, function=worker)
t1.setName('t1')
t2.setName('t2')
logging.info('start')
t1.start()
t2.start()
time.sleep(2)
logging.info('canceling {}'.format(t1.name))
t1.cancel()  # 2s之后仍然可以取消t1
logging.info('end')

# 输出
2017-03-21 19:28:52,801 INFO [MainThread] start
2017-03-21 19:28:54,811 INFO [MainThread] canceling t1
2017-03-21 19:28:54,819 INFO [MainThread] end
2017-03-21 19:28:55,808 INFO [t2] worker running
```

**Timer.cancel()**：取消仍然存活的定时器，如果定时器已经开始执行function，则无法取消。

**Timer.setDaemon(True)**：设置定时器为守护线程

# 线程同步

当使用多个线程来访问同一个数据时，会经常出现资源争用等线程安全问题(比如多个线程都在操作同一数据导致数据不一致)，这时候我们就可以使用一些同步技术来解决这类问题。比如Event，Lock，Condition，Barrier，Semaphore等等。

## Event

```python
>>> help(threading.Event)
```

Event对象内置一个标志，这个标志可以由**set()**方法和**clear()**方法设定。线程可以使用**wait()**方法进行阻塞等待，知道Event对象内置标志被set。

1. **clear(self)**：设置内置标志为False
2. **set(self)**：设置内置标志为True
3. **wait(self, timeout=None)**：开始阻塞，直到内置标志被设置为True（即wait会阻塞线程直到set方法被调用或者超时）
4. **is_set(self)**：当且仅当内置标志为True的时候返回True

代码

以下代码实现的逻辑是：一个boss和五个睡觉工人，只要有一个工人完成了睡觉任务，那么就唤醒boss和其他工人。

```python
import datetime
import threading
import logging
import random

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

def worker(event: threading.Event):
    s = random.randint(1, 5)
    event.wait(s)  # wait方法而不使用sleep方法，可以让其他工人收到通知后不再等待
    logging.info('sleep {}'.format(s))
    event.set()


def boss(event:threading.Event):
    start = datetime.datetime.now()
    event.wait()
    end = datetime.datetime.now()
    logging.info('that boss exit takes {}s'.format(end - start))

def start():
    event = threading.Event()
    b = threading.Thread(target=boss, args=(event, ), name='boss')
    b.start()
    for i in range(5):
        t = threading.Thread(target=worker, args=(event, ), name='worker-{}'.format(i))
        t.start()
```

执行start()方法，测试结果

```python
>>> start()

2017-03-21 21:20:17,195 INFO [worker-2] sleep 1
2017-03-21 21:20:17,198 INFO [boss] that boss exit takes 0:00:01.004954s
2017-03-21 21:20:17,199 INFO [worker-0] sleep 2
2017-03-21 21:20:17,199 INFO [worker-1] sleep 3
2017-03-21 21:20:17,199 INFO [worker-3] sleep 2
2017-03-21 21:20:17,198 INFO [worker-4] sleep 1
```

可以看到：worker-2退出之后，boss和另外四个worker也瞬间就退出了。所以event对象的内置状态被set之后，相关线程就不再wait了。

- event：在线程之间发送信号，通常用于某个线程需要等待其他线程处理完成某些动作之后才能启动

**wait()方法的timeout参数**

```python
def worker(event: threading.Event):
    while not event.wait(3):
        logging.info('run run run')

event = threading.Event()
threading.Thread(target=worker, args=(event, )).start()

# 输出
2017-03-21 21:32:47,275 INFO [Thread-8] run run run
2017-03-21 21:32:50,277 INFO [Thread-8] run run run
2017-03-21 21:32:53,281 INFO [Thread-8] run run run
2017-03-21 21:32:56,284 INFO [Thread-8] run run run
...
```

程序每隔3s就会输出一次结果，直到执行set()方法才会停止。因此我们可以写一个定时器（类似于Thread类的派生类Timer）。

**代码**

```python
class Timer:
    def __init__(self, interval, function, *args, **kwargs):
        self.interval = interval
        self.function = function
        self.args = args
        self.kwargs = kwargs
        self.event = threading.Event()
        self.thread = threading.Thread(target=self.__target(), args=args, kwargs=kwargs)

    def __target(self):
        if not self.event.wait(self.interval):
            return self.function

    def start(self):
        self.thread.start()

    def cancel(self):
        self.event.set()

def worker(act):
    logging.info('run-{}'.format(act))

t = Timer(5, worker, 'hahaha')
t.start()  # 输出2017-03-21 22:14:59,645 INFO [Thread-20] run-hahaha
```

延迟5s之后执行了逻辑函数，也可以使用cancel函数取消。（**要注意参数的传递，此处Timer初始化不能使用关键字参数**）

## Lock

event是用来同步线程之间的操作的，但是如果要控制共享资源的访问那就需要用到锁机制了，在Python标准库中的实现就是内置的lock类。

```python
>>> help(threading.Lock)
```

**threading.Lock()**函数会创建一个lock类的对象。

```python
>>> help(threading.Lock())
```

锁对象是一个同步原语（synchronization primitive），lock对象主要有以下三个方法：

1. acquire()： acquire(blocking=True, timeout=-1) -> bool 获得锁（即锁定锁）。成功获得锁返回True，没有获得锁则返回False。
2. release()： release() 释放锁
3. locked()：  locked() -> bool 检查锁是否被锁住

**代码**

以下代码实现了在多个进程同时对资源进行访问时，进行加锁和解锁的操作，保证加减操作和赋值操作组合之后的原子性。

```python
class Counter:  # 计时器有加减方法，都会修改value值，因此都需要加锁处理
    def __init__(self, start=0):
        self.value = start
        self.lock = threading.Lock()

    def inc(self):
        self.lock.acquire()
        try:
            self.value += 1
        finally:
            self.lock.release()  # 需要用finally语句保证锁一定会被释放，否则资源永远不可访问

    def dec(self):
        self.lock.acquire()
        try:
            self.value -= 1
        finally:
            self.lock.release()

def inc_worker(c: Counter):
    pause = random.random()
    logging.info('sleeping-{}'.format(pause))
    time.sleep(pause)
    c.inc()
    logging.info('cur_value:{}'.format(c.value))

def dec_worker(c: Counter):
    pause = random.random()
    logging.info('sleeping-{}'.format(pause))
    time.sleep(pause)
    c.dec()
    logging.info('cur_value:{}'.format(c.value))

c = Counter()
for i in range(2):
    threading.Thread(target=inc_worker, args=(c, ), name='inc_worker-{}'.format(i)).start()

for i in range(3):
    threading.Thread(target=dec_worker, args=(c, ), name='dec_worker-{}'.format(i)).start()
```

测试输出

```
2017-03-21 23:17:44,761 INFO [inc_worker-0] sleeping-0.6542416949220327
2017-03-21 23:17:44,766 INFO [inc_worker-1] sleeping-0.48615543229897873
2017-03-21 23:17:44,771 INFO [dec_worker-0] sleeping-0.12355589507242459
2017-03-21 23:17:44,776 INFO [dec_worker-1] sleeping-0.5276710391905681
2017-03-21 23:17:44,784 INFO [dec_worker-2] sleeping-0.5546251407611247
2017-03-21 23:17:44,900 INFO [dec_worker-0] cur_value:-1
2017-03-21 23:17:45,258 INFO [inc_worker-1] cur_value:0
2017-03-21 23:17:45,312 INFO [dec_worker-1] cur_value:-1
2017-03-21 23:17:45,351 INFO [dec_worker-2] cur_value:-2
2017-03-21 23:17:45,421 INFO [inc_worker-0] cur_value:-1
```

可见，各项操作之间保持相互原子性，没有出现干扰。

因为lock类实现了`__enter__`和`__exit__`两个魔术方法，因此支持上下文管理器，可以修改以上Counter类的实现方法如下：

```python
class Counter:
    def __init__(self, start=0):
        self.value = start
        self.lock = threading.Lock()

    def inc(self):
        self.lock.acquire()
        with self.lock:
            self.value += 1

    def dec(self):
        self.lock.acquire()
        with self.lock:
            self.value -= 1
```

即使用上下文管理器来代替`try...finally...`语句，测试输出应该以以上结果一致。

**acquire方法的blocking参数**

当blocking=True时，A线程中执行了lock.acquire()方法之后并且没有执行到lock.release()方法，如果在B线程中再次执行lock.acquire()方法，则B线程阻塞。

- 正如以上代码实现，当有n个线程需要修改一个共享资源的时候，其他线程在获取锁之前都处于阻塞状态。（python的阻塞都会让出cpu的时间片，因此不是忙等待）

当blocking=Fasle时，A线程中执行了lock.acquire()方法之后并且没有执行到lock.release()方法，如果在B线程中再次执行lock.acquire()方法，则B线程不会阻塞，并且acquire函数返回False。

**acquire方法的timeout参数**

当blocking=True并且timeout>0时，acquire会一直阻塞到超时或者锁被释放。

**acquire(0)的参数传递**

模拟acquire方法的默认参数，编写一下函数进行模拟参数传递的过程：

```python
def print1(blocking=True, timeout=-1):
    print(blocking, timeout)

print1(0)  # 输出0 -1
print1(10)  # 输出10 -1
```

可见第一个位置参数，替代了blocking。也就是说lock.acquire(0)等效于lock.acquire(blocking=False)

## RLock

正常的lock对象是不能多次调用`acquire`的，但是可重用锁`RLock`可以多次调用 `acquire` 而不阻塞，而且 `release` 时也要执行和 `acquire` 一样的次数。

## Condition

除了Event对象之外，线程同步还可以使用条件同步机制Condition。一类线程等待特定条件，而另一类线程发出特定条件满足的信号。

```
>>> help(threading.Condition)
```

在Condition的帮助中有以下几个方法：

- 初始化方法：**init(self, lock=None)**。如果给定了lock参数，那么必须是Lock或者Rlock对象，并且被当做底层锁来使用。如果没有指定，那么会创建一个RLock对象的锁，也被当做底层锁来使用。
- 实现了`__enter__`和`__exit__`方法，支持上下文管理器。
- notify(self, n=1)：唤醒一个或多个在当前Condition上等待的其他线程，如果此方法的调用线程没有获得锁，那么在调用的时候就会报错RuntimeError
- notify_all(self)：唤醒所有线程
- wait(self, timeout=None)：一直等待着知道被notifyed或者发生超时

**实例代码**

以下代码实现的是：有一个生产者线程，会生产若干次，每次生产结束后需要通知所有的消费者线程来消费，因此下面代码使用的是notify_all方法。

```python
import threading
import time
import logging
import random
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

class Producer_Consumer_Model:
    def __init__(self):
        self.data = None
        self.event = threading.Event()  # 用来控制消费者退出
        self.condition = threading.Condition()

    def Consumer(self):
        while not self.event.is_set():
            with self.condition:
                self.condition.wait()  # 一直等待直到收到生产者通知notify_all
                logging.info(self.data)  # 收到通知之后，开始执行消费者的业务逻辑部分

    def Producer(self):
        for _ in range(4):  # 每个生产者生产4次
            data = random.randint(0, 100)
            logging.info(data)
            with self.condition:
                self.data = data  # 写入成功就表示生产成功，因此需要在此加锁并且能够通知消费者线程去消费，因此选择使用condition来处理
                self.condition.notify_all()  # 生产成功之后通知所有的消费者去消费
            self.event.wait(1)  # 没生产一次等待1s
        self.event.set()  # 所有的生产完成之后通知消费者退出

m = Producer_Consumer_Model()
for i in range(3):
    threading.Thread(target=m.Consumer, name='Consumer-{}'.format(i)).start()

p = threading.Thread(target=m.Producer, name='Producer')
p.start()
```

测试结果（一个生产者，三个消费者）

```
2017-03-22 22:07:42,875 INFO [Producer] 16
2017-03-22 22:07:42,883 INFO [Consumer-0] 16
2017-03-22 22:07:42,890 INFO [Consumer-2] 16
2017-03-22 22:07:42,894 INFO [Consumer-1] 16
2017-03-22 22:07:43,884 INFO [Producer] 76
2017-03-22 22:07:43,888 INFO [Consumer-0] 76
2017-03-22 22:07:43,895 INFO [Consumer-2] 76
2017-03-22 22:07:43,898 INFO [Consumer-1] 76
2017-03-22 22:07:44,889 INFO [Producer] 31
2017-03-22 22:07:44,891 INFO [Consumer-0] 31
2017-03-22 22:07:44,911 INFO [Consumer-2] 31
2017-03-22 22:07:44,913 INFO [Consumer-1] 31
2017-03-22 22:07:45,892 INFO [Producer] 17
2017-03-22 22:07:45,894 INFO [Consumer-0] 17
2017-03-22 22:07:45,907 INFO [Consumer-2] 17
2017-03-22 22:07:45,910 INFO [Consumer-1] 17
```

可见，生产者每生产一次，所有的消费者就会去消费。如果想控制每次生产之后通知几个消费者来消费，那么就可以使用notify方法，指定消费者线程个数。

代码如下

```python
import threading
import time
import logging
import random
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

class Producer_Consumer_Model:
    def __init__(self):
        self.data = None
        self.event = threading.Event()  # 用来控制消费者退出
        self.condition = threading.Condition()

    def Consumer(self):
        while not self.event.is_set():
            with self.condition:
                self.condition.wait()  # 一直等待直到收到生产者通知notify_all
                logging.info(self.data)  # 收到通知之后，开始执行消费者的业务逻辑部分

    def Producer(self):
        for _ in range(4):  # 每个生产者生产4次
            data = random.randint(0, 100)
            logging.info(data)
            with self.condition:
                self.data = data  # 写入成功就表示生产成功，因此需要在此加锁并且能够通知消费者线程去消费，因此选择使用condition来处理
                self.condition.notify(1)  # 生产成功之后通知所有的消费者去消费
            self.event.wait(1)  # 没生产一次等待1s
        self.event.set()  # 所有的生产完成之后通知消费者退出

m = Producer_Consumer_Model()
for i in range(3):
    threading.Thread(target=m.Consumer, name='Consumer-{}'.format(i)).start()

p = threading.Thread(target=m.Producer, name='Producer')
p.start()
```

测试结果（一个生产者，三个消费者，每次生产之后只通知一个消费者去消费）

```
2017-03-22 22:24:52,933 INFO [Producer] 11
2017-03-22 22:24:52,948 INFO [Consumer-0] 11
2017-03-22 22:24:53,949 INFO [Producer] 47
2017-03-22 22:24:53,967 INFO [Consumer-1] 47
2017-03-22 22:24:54,967 INFO [Producer] 14
2017-03-22 22:24:54,983 INFO [Consumer-2] 14
2017-03-22 22:24:55,986 INFO [Producer] 54
2017-03-22 22:24:55,993 INFO [Consumer-0] 54
```

> 1. Condition 通常用于生产者消费者模式， 生产者生产消息之后， 使用notify 或者 notify_all 通知消费者消费。
> 2. 消费者使用wait方法阻塞等待生产者通知
> 3. notify通知指定个wait的线程， notify_all通知所有的wait线程
> 4. 无论notify/notify_all还是wait 都必须先acqurie， 完成后必须确保release， 通常使用with语法

## Barrier

Barrier类存在于threading模块中，中文可以翻译成栅栏

```python
>>> help(threading.Barrier)
```

可以看到Barrier的主要方法和属性：

- `__init__(self, parties, action=None, timeout=None)`：初始化方法，创建一个Barrier
  - `parties`：所有参与的线程的数量
  - `action`：所有的线程都wait之后并且在线程释放之前就会执行这个action函数，相当于集结之后要做的事情。
  - `timeout`：相当于给需要等待的每个线程的wait方法加上timeout参数，超时则barrier不再生效
- `abort(self)`：将Barrier设置成broken状态
- `reset(self)`：将Barrier重置为最初状态
- `wait(self, timeout=None)`：在Barrier前等待，返回在Barrier前等待的下标，从0到parties-1
- `broken`：如果Barrier处于broken状态则返回True
- `n_waiting`：当前已经在Barrier处等待的线程的数量
- `parties`：需要在Barrier处等待的线程的数量

**示例代码**

```python
import threading
import logging

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

barrier = threading.Barrier(parties=3)

def worker(barrier: threading.Barrier):
    logging.info('waiting for barrier with {} others'.format(barrier.n_waiting))
    worker_id = barrier.wait()
    logging.info('after barrier {}'.format(worker_id))

for i in range(3):
    threading.Thread(target=worker, args=(barrier, ), name='worker-{}'.format(i)).start()
```

测试结果

```
2017-03-22 23:25:03,992 INFO [worker-0] waiting for barrier with 0 others
2017-03-22 23:25:03,995 INFO [worker-1] waiting for barrier with 1 others
2017-03-22 23:25:03,998 INFO [worker-2] waiting for barrier with 2 others
2017-03-22 23:25:04,001 INFO [worker-2] after barrier 2
2017-03-22 23:25:04,001 INFO [worker-0] after barrier 0
2017-03-22 23:25:04,001 INFO [worker-1] after barrier 1
```

可见，所有的线程都会一直等待，知道所有的线程都到期了，然后就通过barrier，继续执行后续操作。

> Barrier会建立一个控制点，所有参与的线程都会阻塞，直到所有参与的“各方”达到这一点。 它让线程分开启动，然后暂停，直到它们都准备好再继续。因此，这一点可以理解为各个线程的一个集结点。

**abort函数的使用**

将Barrier设置成broken状态。所有线程在参与集结过程中，只要执行了barrier.abort方法，那么正在等待的线程都会抛出threading.BrokenBarrierError异常。可以理解为，只要有一个线程确定已经到不了Barrier并且通知了Barrier，那么Barrier就会执行abort方法，通知所有正在wait的线程放弃集结。

**实例代码**

```python
import threading
import logging

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

def worker(barrier: threading.Barrier):
    logging.info('waiting for barrier with {} others'.format(barrier.n_waiting))
    try:
        worker_id = barrier.wait()
    except threading.BrokenBarrierError:
        logging.info('aborting')
    else:
        logging.info('after barrier {}'.format(worker_id))

barrier = threading.Barrier(4)  # 需要等待4个线程
for i in range(3):
    threading.Thread(target=worker, args=(barrier, ), name='worker-{}'.format(i)).start()  # 3个线程都开始wait
barrier.abort()  # 还有一个线程没有到wait，此时执行abort方法，则所有正在wait的线程都抛出异常
```

测试结果

```
2017-03-22 23:47:43,184 INFO [worker-0] waiting for barrier with 0 others
2017-03-22 23:47:43,192 INFO [worker-1] waiting for barrier with 1 others
2017-03-22 23:47:43,201 INFO [worker-2] waiting for barrier with 2 others
2017-03-22 23:47:43,211 INFO [worker-2] aborting
2017-03-22 23:47:43,207 INFO [worker-1] aborting
2017-03-22 23:47:43,207 INFO [worker-0] aborting
```

## Semaphore

Semaphore类存在于threading模块中

```python
help(threading.Semaphore)
```

信号量内部管理者一个计数器，这个计数器的值等于release()方法调用的次数减去acquire()方法调用的次数然后再加上初始值value，value默认为1。

可以看到Semaphore的主要方法：

- `__init__(self, value=1)`：初始化一个信号量，value为内部计数器赋初值，默认为1
- `acquire(self, blocking=True, timeout=None)`：获取信号量，内部计数器减一
- `release(self)`：释放信号量，内部计数器加一

**示例代码**

```python
import threading
import time
import logging
import random
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

class Pool:
    def __init__(self, num):
        self.num = num
        self.conns = [self._make_connect(i) for i in range(num)]  # 存放连接
        self.sem = threading.Semaphore(num)  # 信号量内部计数器初始为连接数

    def _make_connect(self, name):  # 根据连接名称创建连接
        conn = 'connect-{}'.format(name)
        return conn

    def get_connect(self):  # 从连接池获取连接
        self.sem.acquire()
        return self.conns.pop()

    def return_connect(self, conn):  # 将连接conn返还到连接池中
        self.conns.insert(0, conn)
        self.sem.release()

def worker(pool: Pool):
    logging.info('starting')
    conn = pool.get_connect()  # 如果获取不到则会阻塞在acquire处
    logging.info('get connect {}'.format(conn))
    t = random.randint(1, 3)
    time.sleep(t)
    logging.info('takes {}s'.format(t))
    pool.return_connect(conn)
    logging.info('return connect {}'.format(conn))

pool = Pool(2)  # 连接池中有两个连接可以使用
for i in range(3):  # 三个线程使用两个连接开始任务
    threading.Thread(target=worker, args=(pool, ), name='worker-{}'.format(i)).start()
```

测试结果

```
2017-03-23 00:54:36,056 INFO [worker-0] starting
2017-03-23 00:54:36,062 INFO [worker-0] get connect connect-1
2017-03-23 00:54:36,074 INFO [worker-1] starting
2017-03-23 00:54:36,079 INFO [worker-1] get connect connect-0
2017-03-23 00:54:36,089 INFO [worker-2] starting
2017-03-23 00:54:39,074 INFO [worker-0] takes 3s
2017-03-23 00:54:39,076 INFO [worker-0] return connect connect-1
2017-03-23 00:54:39,076 INFO [worker-2] get connect connect-1
2017-03-23 00:54:39,093 INFO [worker-1] takes 3s
2017-03-23 00:54:39,097 INFO [worker-1] return connect connect-0
2017-03-23 00:54:40,093 INFO [worker-2] takes 1s
2017-03-23 00:54:40,107 INFO [worker-2] return connect connect-1
```

这个测试结果显示：三个线程获取连接池中的两个连接，结果出现了一个线程等待其他线程执行完成之后再获取连接的过程。

## Queue

Condition线程同步部分用来传递数据的是一个封装在生产者消费者模型中的元素data（正常使用情况下一般封装的都是一个列表，类似与Barrier部分的连接池中的conns列表）。

Python的queue模块中提供了同步的、线程安全的队列类，包括三种队列：

- FIFO（先入先出)队列Queue
- LIFO（后入先出）队列LifoQueue
- 优先级队列PriorityQueue

这些队列都实现了锁原语，能够在多线程中直接使用。可以使用队列来实现线程间的同步。因此我们可以使用queue模块来替换掉生产者消费者中的全局元素，代码如下：

```python
import random
import queue
import threading

class Producer_Consumer_Model:
    def __init__(self):
        self.q = queue.Queue()
        self.event = threading.Event()

    def Consumer(self):
        while not self.event.is_set():
            logging.info(self.q.get())

    def Producer(self):
        while not self.event.wait(3):
            data = random.randint(1, 100)
            logging.info(data)
            self.q.put(data)

m = Producer_Consumer_Model()
threading.Thread(target=m.Consumer, name='Consumer').start()
threading.Thread(target=m.Producer, name='Producer').start()
```

测试结果

```
2017-03-23 10:11:22,990 INFO [Producer] 26
2017-03-23 10:11:22,993 INFO [Consumer] 26
2017-03-23 10:11:25,993 INFO [Producer] 89
2017-03-23 10:11:26,003 INFO [Consumer] 89
2017-03-23 10:11:29,004 INFO [Producer] 14
2017-03-23 10:11:29,006 INFO [Consumer] 14
2017-03-23 10:11:32,007 INFO [Producer] 17
2017-03-23 10:11:32,009 INFO [Consumer] 17
```

每生产一次，消费者就会消费一次。当消费者线程，读取Queue则调用Queue.get()方法，若Queue为空时消费者线程获取不到内容，就会阻塞在这里，直到成功获取内容。

##线程同步总结

- Event：主要用于线程之间的事件通知
- Lock,Rlock：主要用于保护共享资源
- Condition：主要用于生产者消费者模型，可以理解为Event和Lock的结合体
- Barrier：同步指定个等待的线程
- Semaphore：主要用于保护资源，和Lock的区别在于可以多个线程访问共享资源，而锁一次只能一个线程访问到共享资源，即锁是value=1的信号量
- Queue：使用FIFO队列进行同步，适用于生产者消费者模型

# GIL

GIL（Global Interpreter Lock）：全局解释器锁

Python代码的执行由Python 主循环来控制，Python 在设计之初就考虑到要在解释器的主循环中，同时只有一个线程在执行，即在任意时刻，只有一个线程在解释器中运行。对Python 主循环的访问由全局解释器锁（GIL）来控制，正是这个锁能保证同一时刻只有一个线程在运行。

因此Python多线程程序的执行顺序如下：

1. 设置GIL
2. 切换到一个线程去运行
3. 运行
4. 结束线程
5. 解锁GIL
6. 重复以上步骤

因此，Python的多线程并没有实现并行，只是实现了并发而已。如果要实现真正的并行，那就需要使用Python的多进程模块multiprocessing（multiprocessing模块的宗旨是像管理线程一样来管理进程）。

---

参考资料

1. [threading — Manage Concurrent Operations Within a Process](https://pymotw.com/3/threading/index.html)
2. [Python线程同步机制: Locks, RLocks, Semaphores, Conditions, Events和Queues](http://yoyzhou.github.io/blog/2013/02/28/python-threads-synchronization-locks/)