---
layout: post
title: python线程中的condition特性
date: 2016-08-21 20:00
categories: [python学习笔记]
tags: [线程，condition]
---
 
python学习笔记之线程间的condition特性
<!--more-->
 
## threading的Condition方法
 
　　confition方法适合`生产-消费`的工作模型，消费者会等待生产者生产数据生消费者再对数据进行处理。以下边的例子来说明：
 
```sh
import threading
import logging
import time
 
 
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')
 
 
class Message:
    def __init__(self, message):
        self.message = message
 
 
def consumer(cond, message):
    with cond:
        cond.wait()
        logging.debug("consumer {0}".format(message.message))
 
 
def producer(cond, message):
    with cond:
        print('====')
        time.sleep(2)
        message.message = 'ha ha ha '
        logging.debug("producer {0}".format(message.message))
        cond.notify_all()
 
 
if __name__ == '__main__':
    message = Message(None)
    cond = threading.Condition()
    c1 = threading.Thread(target=consumer, args=(cond, message), name="consumer-1")
    c1.start()
    c2 = threading.Thread(target=consumer, args=(cond, message), name="consumer-2")
    c2.start()
 
    p = threading.Thread(target=producer, args=(cond, message), name="producer")
    p.start()
```
 
上边的测试代码中定义了一个`producer`方法用于生产数据，`consumer`方法用于消费数据，当运行上边代码后，先是输出了`====`，再等待2秒后输出了`2016-08-21 17:29:16,212 DEBUG [producer] producer ha ha ha `，接着跟着输出了`2016-08-21 17:29:16,212 DEBUG [consumer-1] consumer ha ha ha`和`2016-08-21 17:29:16,212 DEBUG [consumer-2] consumer ha ha ha`，如下所示：
 
```sh
====
2016-08-21 17:29:16,212 DEBUG [producer] producer ha ha ha 
2016-08-21 17:29:16,212 DEBUG [consumer-1] consumer ha ha ha 
2016-08-21 17:29:16,212 DEBUG [consumer-2] consumer ha ha ha 
```
 
可见consumer线程是在producer线程运行之后才运行，即等待生产者生产数据。
 
小结：
 
`threading.Condition()`初始化一个condition对象，当生产者准备好数据后用`notify_all()`通知监听同一个condition对象的线程消费数据，如果生产方没有数据生产出来，那消费者就被阻塞等待。
