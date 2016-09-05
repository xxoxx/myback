---
layout: post
title: 线程的daemon与join特性
date: 2016-08-14 22:00
categories: [python学习笔记]
tags: [多线程]
---

python学习笔记之多线程中的daemon与join特性
<!--more-->
  
## none daemon与daemon　　
 　　在python 3中，threading属于标准库中的模块，在version 3.3中增加了daemon参数，值是一个bool值，默认为`False`。
  
 　　如果一个线程的`daemon=False`，即我们说这个线程是一个`none daemon`的线程，那主线程`永远`会等待子线程全部退出后自己才退出。
  
 看下边的例子，下边的例子全部都在pytharm的IDE中进行：
  
 ```py
import threading
import logging
import time
  
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')
  
  
def worker(message):
    time.sleep(5)
    logging.debug("worker is started, {0}".format(message))
  
if __name__ == '__main__':
    t = threading.Thread(target=worker, name='worker', kwargs={'message': 'ha,ha'})
    t.daemon = False
    t.start()
    logging.debug("main thread exiting")
  
```
  
当运行这段代码时，先打印出了`主线程`的日志信息，如下：
  
```py
2016-08-14 14:06:24,358 DEBUG [MainThread] main thread exiting
  
```
主线程被阻塞，然后等待5秒后再打印出了`子线程`的日志信息，如下：
  
```py
2016-08-14 14:06:24,358 DEBUG [MainThread] main thread exiting   # 被阻塞
2016-08-14 14:06:29,359 DEBUG [worker] worker is started, ha,ha  # 5秒后才打印出来
```
  
所以当线程是一个`none daemon`线程时，主线程会被阻塞，直到全部子线程退出。
  
当线程是一个daemon线程时呢？把上边的`t.daemon = False`修改成`t.daemon = True`后再次运行代码，结果如下：
  
```py
2016-08-14 14:12:39,644 DEBUG [MainThread] main thread exiting
  
Process finished with exit code 0
```
  
从上边输出的结果可知，主线程没有等待子线程运行结束，而主线程自己就退出了。而当主线程退出后子线程也会跟着退出。
  
小结：
  
　　在`none daemon`的线程中主线程永远会等待子线程退出后才退出，在`daemon`的线程中主线程不会等待子线程退出后自己才退出，在`none daemon`线程中子线程因程序bug或其他原因而不能正常的退出时，那主线程就一直被阻塞，一直等待着子线程退出，这样线程所占用的资源就一直不能被释放。join函数能解决这样的问题。
  
## join( )
  
　　 线程调用join函数后，主线程就会等待子线程退出后才退出，而子线程执行的代码如果不能正常退出时那主线程也一直会等待，如果在join时加上一个超时时间，那主线程只等待设置的超时时间后主线程就退出。
  
- none daemon中的join()
  
看下边的例子：
  
```py
import threading
import logging
import time
  
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')
  
  
def worker(message):
    print('exec worker')
    time.sleep(5)
    logging.debug("worker is started, {0}".format(message))
  
if __name__ == '__main__':
    t = threading.Thread(target=worker, name='worker', kwargs={'message': 'ha,ha'})
    t.daemon = False
    t.start()
    t.join()
    logging.debug("main thread exiting")
```
  
当运行上边代码时，先输出了：
  
```py
exec worker
```
  
再等待5秒后输出了下边的日志信息：
  
```py
2016-08-14 15:41:16,925 DEBUG [worker] worker is started, ha,ha
2016-08-14 15:41:16,925 DEBUG [MainThread] main thread exiting
```
  
这说明当一个线程是`none daemon`时，并调用了`join()`函数，那主线程会等待子线程执行完成。这和不调用`join()`函数的`none daemon`线程是有区别的，调用`join()`函数时是主线程等待子线程完成，不调用`join()`函数时是主线程被阻塞后，一直等待子线程执行完成，从打印的日志可看出两者的区别。
  
join()函数可以给定一个超时时间，如下：
  
```py
import threading
import logging
import time
  
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')
  
  
def worker(message):
    print('exec worker')
    time.sleep(5)
    logging.debug("worker is started, {0}".format(message))
  
if __name__ == '__main__':
    t = threading.Thread(target=worker, name='worker', kwargs={'message': 'ha,ha'})
    t.daemon = False
    t.start()
    t.join(timeout=2)
    logging.debug("main thread exiting")
```
  
运行上边的代码后，首先输出`exec worker`，等待2秒，再输出`2016-08-14 16:10:59,695 DEBUG [MainThread] main thread exiting`,再等待3秒，最后输出`2016-08-14 16:11:02,692 DEBUG [worker] worker is started, ha,ha`，如下：
  
```py
exec worker  # 2秒后输出下一行
2016-08-14 16:10:59,695 DEBUG [MainThread] main thread exiting # 3秒后输出下一行
2016-08-14 16:11:02,692 DEBUG [worker] worker is started, ha,ha
```
  
在`none daemon`的线程中，如果给`join()`函数一个超时时间，当超过这个时间后，即使这个线程没有执行完成，程序就直接输出了`2016-08-14 16:10:59,695 DEBUG [MainThread] main thread exiting`，并且被阻塞在这里等待子线程的完成，因为线程是`none daemon`的。这种场景在现实的编程中应该不多。
  
- daemon中的join()
  
　　在实际的编程中，一般会把线程设置成daemon，并启用`join()`函数，并适当给一个超时时间，这样主线程即能等待子线程，又能兼顾子线程因一些原因被卡住后无法退出时导致主线程也无法退出的情况。如下：
  
```py
import threading
import logging
import time
  
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')
  
  
def worker(message):
    print('exec worker')
    time.sleep(5)
    logging.debug("worker is started, {0}".format(message))
  
if __name__ == '__main__':
    t = threading.Thread(target=worker, name='worker', kwargs={'message': 'ha,ha'})
    t.daemon = True
    t.start()
    t.join(timeout=6)
    logging.debug("main thread exiting")
```
  
运行上边的代码，先是输出`exec worker`，再等待5秒，依次输出`2016-08-14 16:31:59,287 DEBUG [worker] worker is started, ha,ha`和`2016-08-14 16:31:59,287 DEBUG [MainThread] main thread exiting`。这样，采用join的方式让主线程等待子线程正常退出，如果在调用worker函数时有bug，执行时一直退不出来，那到join的超时时间后，主线程同样能退出，把上边的`t.join(timeout=6)`修改成`t.join(timeout=3)`来模拟这样一个场景，如下：
  
```py
import threading
import logging
import time
  
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')
  
  
def worker(message):
    print('exec worker')
    time.sleep(5)
    logging.debug("worker is started, {0}".format(message))
  
if __name__ == '__main__':
    t = threading.Thread(target=worker, name='worker', kwargs={'message': 'ha,ha'})
    t.daemon = True
    t.start()
    t.join(timeout=3)
    logging.debug("main thread exiting")
```
  
执行上边的代码后，先是输出`exec worker`,再等待3秒后直接输出`2016-08-14 16:36:23,608 DEBUG [MainThread] main thread exiting`,worker函数里的内容没有被执行，如下：
  
```py
exec worker
2016-08-14 16:36:23,608 DEBUG [MainThread] main thread exiting
  
Process finished with exit code 0
```
