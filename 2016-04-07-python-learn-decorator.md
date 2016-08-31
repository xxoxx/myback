---
layout: post
title: python装饰器
date: 2016/04/07 15:40
categories: [python]
tags: [装饰器，decorator]
---


# 高阶函数

　　一个函数能接受另一个函数作为参数传入，这样的一个函数就是高阶函数。

　　在python中一切皆对象，函数也不例外，函数是可以赋予给一个变量的，看下边代码：


```python
# 求绝对值函数
def fn(x):
    val = x if x >= 0 else -x
    return val
```


```python
fn(-34)
```




    34




```python
fn(20)
```




    20



在上边的代码中定义了一个函数 `fn()`，这个函数接收一个参数，函数名是`fn`，`fn(-34)`和`fn(20)`是函数调用。函数名`fn`是什么呢？其实它就是指向能计算绝对值函数的一个变量。


```python
f1 = fn
```


```python
type(f1)
```




    function




```python
f1(-12)
```




    12



如上，变量`f1`已指向了`fn`函数，调用`fn()`与调用`f1()`是一样的效果。得到的结论是变量是可以指向一个函数。

那函数的参数能否接收一个变量呢？以代码来验证：


```python
a = -23
```


```python
fn(a)
```




    23



上边把变量`a`指向了`-23`，再调用`fn()`函数时把变量`a`传入，得到了正确的结果，所以函数的参数是能接收一个变量。

**总结：在python中，变量可以指向一个函数，函数的参数可以接收一个变量，那么函数的参数也就可以是一个函数。下边以实际代码来验证**


```python
def num(x,y):
    return y(x)
```


```python
def f(a):
    return a ** 2
```


```python
num(4,f)
```




    16



上边直接调用`num()`函数，把指向函数`f(a)`的函数名`f`，即`f`实质也是一个变量，作为一个变量传递给了`num(x,y)`函数中的参数`y`，这样也得到了正确的答案。所以函数中的参数可以接收一个函数。

总结：高阶函数的实质就是函数的参数能够接收另一个函数。

# 装饰器


```python
def deco(fn):
    print('ha ha ha')
    return fn
```


```python
def myfun():
    print('call myfun()')
```


```python
f4 = deco(myfun) #调用deco(fn)函数，把myfun函数作为参数传入，deco(fn)函数会先执行print语句，再返回fn函数，即这里的myfun函数。再让变量f4指向myfun()这个函数对象
```

    ha ha ha



```python
f4()   #f4是指向myfun()函数对象的一个变量，因为变量可以指向一个函数，所以执行f4()就可以调用myfun()这个函数
```

    call myfun


采用python的魔法也可以实现上边的相同效果，如下：


```python
@deco
def myfun():
    print('call myfun')
```

    ha ha ha


`@deco`中的`@`是语法糖，表示下边定义的函数将会被`@`后的那个函数所修饰，即`myfun()`函数会被`deco(fn)`这个函数所修饰，实质就是执行了`myfun = deco(myfun)`，所以打印出了`ha ha ha`，并把原来的`myfun()`函数返回回来，下边执行`myfun()`时就调用了原来的`myfun()`函数，输出了`call myfun`


```python
myfun()
```

    call myfun


再对上边的deco函数进行修改：


```python
def deco(fn):
    def wrap():
        print('ha ha ha')
        print('call {0} funtion'.format(fn.__name__))
        fn()
    return wrap
```


```python
@deco
def myfun():
    print('call myfun')
```


```python
myfun()
```

    ha ha ha
    call myfun funtion
    call myfun


`myfun()`函数被`deco(fn)`函数进行修饰后，调用`myfun()`时就像被施加了魔法一样在执行此函数前附加的执行了一些操作，当然也可以在执行函数后附加一些操作。

上边被装饰的`myfun()`函数是一个无参的函数，如果被装饰的函数需要接收参数呢？这时装饰器应该这样的来定义：


```python
def deco(fn):
    def wrap(*args,**kwargs):
        print('ha ha ha')
        print('call {0} funtion'.format(fn.__name__))
        return fn(*args,**kwargs)
    return wrap
```

考虑到被装饰的函数可以接收的参数类型的不确定性，可以用可变位置参数和可变关键字参数来捕捉，即`(*args,**kwargs)`


```python
@deco
def myfun(x):
    return x ** 2
```


```python
myfun(5)
```

    ha ha ha
    call myfun funtion





    25




```python
myfun.__name__
```




    'wrap'



`myfun(x)`函数被`deco(fn)`函数装饰后，函数对象的`__name__`属性会发生改变，发上输出。因为`@deco`就相当于执行了`myfun = deco(myfun)`，即变量`myfun`已经指向了`wrap(*args,**kwargs)`函数，这时`myfun`变量指向函数的`__name__`属性就是`wrap`，不再是原来的`myfun()`函数的`__name__`属性。如果要修正这个问题，可以直接引用python内置的`functools.wraps`方法，如下：


```python
import functools
def deco(fn):
    @functools.wraps(fn)
    def wrap(*args,**kwargs):
        print('ha ha ha')
        print('call {0} funtion'.format(fn.__name__))
        return fn(*args,**kwargs)
    return wrap
```


```python
@deco
def myfun(x):
    return x ** 2
```


```python
myfun(25)
```

    ha ha ha
    call myfun funtion





    625




```python
myfun.__name__
```




    'myfun'



总结：装饰器其实也是一个函数，此函数可以接收一个函数作为参数，并返回一个函数，即也是一个高阶函数。装饰器这个函数能让一个函数在调用时的前或后额外的执行一些操作来修改原调用的函数。
