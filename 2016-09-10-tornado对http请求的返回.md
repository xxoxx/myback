---
layout: post
title: tornado怎样返回信息
date: 2016-09-10 10:50
categories: [web框架]
tags: [tornado,返回请求]
---
 
　　本节记录tornado返回给浏览器信息的一些方法，在上一节中反复使用了`tornado.web.RequestHandler`类的`write`方法，此方法直接把数据返回给浏览器。在官方文档中tornado把这部分的内容放在了`RequestHandler`类的`Output`这一节，官方文档请点击[这里](http://www.tornadoweb.org/en/stable/web.html#output)。
<!--more-->
 
## Output
 
### set_status
 
　　此方法设置返回状态码，接收`status_code`和`reason`两个参数，使用如下：
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class ForbiddenHandler(RequestHandler):
    def get(self, *args, **kwargs):
        self.set_status(403)
        self.finish()
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/403', ForbiddenHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
尝试访问`/403`这个路由结果如下：
 
![2016-09-12-forbidden.png](/images/2016-09-12-forbidden.png)
 
 
要注意的是要显示的调用`self.finish()`方法来结束， `self.finish()`代表回应生成的终结，否则客户端还会保持连接。
 
### set_header
 
　　此方法设置响应报文的头部信息，事例如下：
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class HeaderHandler(RequestHandler):
    def get(self, *args, **kwargs):
        self.set_header('X-Header', 'XXXXX')
        self.write('set header')
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/head', HeaderHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
程序运行后访问结果如下：
 
![2016-09-12-set_header-1.png](/images/2016-09-12-set_header-1.png)
 
![2016-09-12-set_header-2.png](/images/2016-09-12-set_header-2.png)
 
 
`set_header`方法如果多次对同一个header进行赋值，那结果是最后那一个，如下测试：
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class HeaderHandler(RequestHandler):
    def get(self, *args, **kwargs):
        self.set_header('X-Header', 'XXXXX')
        self.set_header('X-Header', 'YYYYY')
        self.write('set header')
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/head', HeaderHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
程序启动后测试结果如下：
 
![2016-09-12-set_header-3.png](/images/2016-09-12-set_header-3.png)
 
 
### add_header
 
　　此方法增加给定响应头的值，测试代码如下：
 
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class AddHeadlerHandler(RequestHandler):
    def get(self, *args, **kwargs):
        self.add_header('X-Header', 'add-XXX')
        self.finish()
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/addhead', AddHeadlerHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
程序运行访问结果如下：
 
![2016-09-12-add_header-1.png](/images/2016-09-12-add_header-1.png)
 
如果多次调用`add_header`方法，那最后的响应头的值会被叠加，代码如下：
 
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class AddHeadlerHandler(RequestHandler):
    def get(self, *args, **kwargs):
        self.add_header('X-Header', 'add-XXX')
        self.add_header('X-Header', 'add-YYY')
        self.finish()
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/addhead', AddHeadlerHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
程序运行后访问结果如下：
 
![2016-09-12-add_header-2.png](/images/2016-09-12-add_header-2.png)
 
 
### write
 
　　此方法上边都多次用到，表示把给定的数据块写入输出缓冲区，即当用浏览器访问时直接输出到浏览器上。
 
### flush
 
　　此方法刷新当前的输出缓冲区到网络，看下边的例子：
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
import time
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class FlushHandler(RequestHandler):
    def get(self, *args, **kwargs):
        self.write('start\n')
        self.flush()
        for x in range(5):
            self.write('{0}\n'.format(x))
            self.flush()
            time.sleep(0.1)
        self.finish('stop\n')
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/flush', FlushHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
flush这个特性在`postman`工具中不能演示出效果来，可以使用`curl`来测试，如下：
 
```sh
neal@neal-System-Product-Name:~$ curl http://172.20.12.174:8000/flush
start
0
1
2
3
4
stop
neal@neal-System-Product-Name:~$ 
```
 
上边的输出是有停顿的，停顿的时间就是代码中的`time.sleep(0.1)`。
 
 
### finish
 
　　此方法表示完成这个响应，HTTP的请求结束。前边的事例代码也有说明。
 
 
　　更多信息请参考[官方文档](http://www.tornadoweb.org/en/stable/web.html#output)
 
 
 
