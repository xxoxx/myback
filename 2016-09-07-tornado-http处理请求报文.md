---
layout: post
title: tornado处理请求报文
date: 2016-09-08 09:50
categories: [web框架]
tags: [tornado,处理请求]
---
 
　　tornado是一个轻量、异步的web微框架，本文记录tornado怎样处理一个http请求报文。
<!--more-->
 
## Hello World
 
　　tornado使用`RequestHandler`类来处理http请求，所以在使用tornado来处理http请求时，自定义的类需要继承`RequestHandler`类。HTTP的方法，如：GET、PUT、DELETE、POST、HEAD等，都一一对应`RequestHandler`类中的方法。在使用tornado的web框架特性时一般是从`tornado`这个包中引入`web`模块里的相应类，`web`这个模块的源码请点击[这里](https://github.com/tornadoweb/tornado/blob/master/tornado/web.py)。
　　接下来看一下tornado是怎样来处理http的请求的。首先来写一个经典的`Hello World`程序，代码如下：
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class MainHandler(RequestHandler):
    def get(self):
        self.write('Hello World\n')
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/', MainHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
把程序启动后会监听在8000端口，使用curl来访问得到如下：
 
```sh
neal@neal-System-Product-Name:~$ curl http://172.20.12.174:8000
Hello World
neal@neal-System-Product-Name:~$
```
 
这个`Hello World`事例中首先从`tornado.web`这个模块中引入了`RequestHandler`和`Application`两个类，`RequestHandler`类用来处理HTTP的请求，`Application`用来处理路由，这个类实例化后返回一个可调用的`tornado.web.Application`类，然后调用此类的`listen`方法使用程序监听在本地任意地址的`8000`端口，最后使用`IOLoop.current().start()`来把程序运行在起来循环处理用户请求。Tornado为了实现高并发和高性能，使用了这个IOLoop来处理socket的读写事件，IOLoop基于epoll，可以高效的响应网络事件。这是Tornado高效的保证。
 
 
在tornado的官方文档中把`Request handlers`分成了`Entry points`、`Input`和`Output`等多个章节，这里只对`Input`和`Output`的某些方法进行讲解，更详细资料请看[官方文档](http://www.tornadoweb.org/en/stable/web.html)。
 
## Input
 
### get_argument方法
 
此方法返回用户传递参数的内容。
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class ArgumentHandler(RequestHandler):
    def get(self):
        self.write(self.get_argument('name'))
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/arg', ArgumentHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
`ArgumentHandler`类是用户自定义类，此类继承`RequestHandler`,接着对`get`方法进行重写，这就是我们在使用`tornado`进行web开发时经常需要做的工作。HTTP协议定义的`get`、`post`、`delete`等方法在`tornado`中对应的同名方法是直接返回`HTTPError(405)`，即是没有实现的方法，这些方法就需要程序员自己根据业务逻辑来实现。源码请[参考](http://www.tornadoweb.org/en/stable/_modules/tornado/web.html#RequestHandler.get)。运行此代码再访问返回如下：
 
![2016-09-08-get_argument](/images/2016-09-08-get_argument.png)
 
结果是直接把传递的参数`name`的值`zhaochj`直接返回。如果给`name`这个变量传递多个值呢？如下：
 
![2016-09-08-get_argument-1.png](/images/2016-09-08-get_argument-1.png)
 
结果是得到了最后一个值。
 
 
### get_arguments
 
此方法同样返回用户传递参数的内容，但与`get_argument`不同的是它能接收同名的的多个参数，并且返回一个`list`，测试代码如下：
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class ArgumentHandler(RequestHandler):
    def get(self):
        self.write(self.get_argument('name'))
 
 
class ArgumentsHandler(RequestHandler):
    def get(self):
        self.write('{0}\n'.format(str(type(self.get_arguments('name')))))
        self.write('{0}'.format(', '.join(self.get_arguments('name'))))
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/arg', ArgumentHandler),
            (r'/args', ArgumentsHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
运行上边代码后访问得到如下结果：
 
![2016-09-08-get_arguments.png](/images/2016-09-08-get_arguments.png)
 
 
### args
 
在实际编程中可能会需要捕获一个路径下的值，使用`args`就可以把捕获到一个路径下的所有内容，并会把这个值存放在一个位置参数中，通过访问`args[0]`便可访问，代码如下：
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class PathArgsHandler(RequestHandler):
    def get(self, *args, **kwargs):
        self.write('{0}'.format(args[0]))
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/path/args/(.*)', PathArgsHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
在正规中`(.*)`表示匹配任意内容，所以在URL中`/path/args/`后的所有内容都会被捕获，访问后结果如下：
 
![2016-09-08 17-25-41-args.png](/images/2016-09-08 17-25-41-args.png)
 
### kwargs
 
同`args`一样，`kwargs`可以捕获一个路径下的`K/V`键值数据，并存放在字典中，代码如下：
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class PathKwArgsHandler(RequestHandler):
    def get(self, *args, **kwargs):
        self.write('{0}'.format(kwargs['name']))
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/path/kwargs/(?P<name>.*)', PathKwArgsHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
`（?P<name>.*)`表示命名捕获，这是`re`的语法，请参照[这里](https://docs.python.org/3/library/re.html)，程序运行后访问结果如下：
 
![2016-09-08-kwargs.png](/images/2016-09-08-kwargs.png)
 
### 获取请求报文的其他内容
 
　　一个请求报文中包含了非常多的内容，如：method，uri，path，query，version，body等，详细请参考[这里](http://www.tornadoweb.org/en/stable/httputil.html#tornado.httputil.HTTPServerRequest)，以下事例说明怎样获取请求端的IP地址有body内容，代码如下：
 
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
import json
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
 
 
class RemoteIpHander(RequestHandler):
    def get(self):
        self.write('{0}'.format(self.request.remote_ip))
 
 
class BodyHandler(RemoteIpHander):
    def post(self):
        body = json.loads(self.request.body.decode())
        self.write('{0}'.format(body['name']))
 
 
if __name__ == '__main__':
    app = Application(
        [
            (r'/ip', RemoteIpHander),
            (r'/body', BodyHandler)
        ]
    )
    app.listen(port=8000, address='0.0.0.0')
    IOLoop.current().start()
```
 
`RemoteIpHander`类中直接调用`self.request.remote_ip`即可打印出客户端的IP地址，而`BodyHandler`类定义`post`方法，采用`json.lodads`来解析上传的`json`数据，注意最后的`decode`方法，因为通过客户端发送过来的数据是`bytes`，而`json`对象只能是`str`，所以需要`decode`来解码。程序运行后测试结果如下：
 
![2016-09-08-remoteip.png](/images/2016-09-08-remoteip.png)
 
![2016-09-08-postbody.png](/images/2016-09-08-postbody.png)
 
 
 
　　`RequestHandler`这个类还有许多方法，详细信息参考[官方文档](http://www.tornadoweb.org/en/stable/web.html)。
