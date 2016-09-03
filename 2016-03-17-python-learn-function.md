---
layout: post
title: python函数
date: 2016/03/17 21:20
categories: [python学习笔记]
tags: [函数]
---

　　python学习笔记之函数
<!--more-->

# 函数

　　个人理解，函数是为了实现特定功能而编写在一起的代码语句，这些代码组织成了代码块，并给予这个代码块一个名称，以便在其他地方调用。
那为什么要使用函数呢？使用函数是降低编程的难度，函数能够把大的问题分解成一个个小的问题，只要小的问题解决了那大的问题也迎刃而解；再者是为了能让代码重用，把在编程中经常用到的代码组织成函数，在当需要使用时直接调用函数，而不必再去编写相应的代码。

* 怎样定义一个函数

```
1. 函数以def开头，后接函数名，紧接着一个小括号，小括号内是传入函数的参数;
2. 函数定义后的第一行可以选择性的加入函数的说明性文档，说明文档若是一行用一对双引号，如果会换行，那用三对双引号或三对单引号；
3. 函数最后一般会返回一个结果，若不返回结果那会返回一个None值。
def function_name(parameters):
    """函数说明文档"""
    function_suite
    return [expression]
```


```python
def my_abs(x):
    "求一个数的绝对值"
    if not isinstance(x,(int,float)):         #做传入函数的参数做类型判断，只有整形或浮点型的数才能求绝对值
        raise TypeError('bad type')
    if x >= 0:
        return x
    else:
        return -x
print('这是{0},的说明性文档：{1}'.format(my_abs.__name__,my_abs.__doc__))    
```

    这是my_abs,的说明性文档：求一个数的绝对值


* 调用函数

　　直接键入“functionname([patameters])”即可调用函数


```python
my_abs(29)
```




    29




```python
my_abs(-23.8)
```




    23.8




```python
my_abs('ab')   #当传入的参数不能通过类型检查时触发raise异常语句
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-20-3c363f4bf733> in <module>()
    ----> 1 my_abs('ab')
    

    <ipython-input-17-90b4c1d5d37c> in my_abs(x)
          2     "求一个数的绝对值"
          3     if not isinstance(x,(int,float)):         #做传入函数的参数做类型判断，只有整形或浮点型的数才能求绝对值
    ----> 4         raise TypeError('bad type')
          5     if x >= 0:
          6         return x


    TypeError: bad type


* 函数的参数

　　在python中函数的定义非常简单，只要确定函数的名称，参数的名称，这样函数的定义就完成了，而调用者只需要知道怎样向这个函数传递参数以及此函数返回什么样的值就Ok了，而函数体调用都无需了解。
  
　　函数中的参数有多种，有必选参数、命名参数、默认参数、可变参数和关键字参数。

1. 必选参数

　　函数的必选参数要求在调用函数时一定得按照在定义函数时的参数位置传入，传入的参数数量也一定要与定义时参数的数量相同。


```python
my_abs(-39)   #上边定义的my_abs函数就定义了一个必选参数x，在调用时必须正确的传入一个参数
```




    39




```python
my_abs(23,-3)   #当传入的参数数量与定义的不同时会抛出异常
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-25-0af413d28136> in <module>()
    ----> 1 my_abs(23,-3)
    

    TypeError: my_abs() takes 1 positional argument but 2 were given



```python
def info(name,age):
    return '你的姓名是{0},你的年龄是{1}'.format(name,age)
```


```python
info('Neal',30)
```




    '你的姓名是Neal,你的年龄是30'




```python
info(30,'Neal')   #当实参传入的位置发生改变后，得到了如下结果
```




    '你的姓名是30,你的年龄是Neal'



2．命名参数

　　当函数调用时用参数的名字确定传入的参数值，参数名与参数值间用等号连接，命名参数不再像必选参数一样对实参传入的位置敏感。以上边的定义的info函数来说明。


```python
info(age=30,name='Neal')
```




    '你的姓名是Neal,你的年龄是30'



3.　默认参数

　　所谓默认参数，就是在调用函数时参数的值没有显示的传入时，那参数就被赋予一个默认的值，如果参数被赋予了一个值时，那就用传入的值赋予此参数。默认参数在定义函数时以“参数名 = 参数值”的方式定义。


```python
def infomations(name,age,add = 'ChongQing'):
    return '你的姓名是{0},你好年龄是{1},你的家庭住址是：{2}'.format(name,age,add)
```


```python
infomations('Neal',30)
```




    '你的姓名是Neal,你好年龄是30,你的家庭住址是：ChongQing'




```python
infomations('Neal',30,'BeiJin')
```




    '你的姓名是Neal,你好年龄是30,你的家庭住址是：BeiJin'



3．可变参数

　　可变参数，从字面上理解就是传递给函数的参数的个数是变化的，可以是1个、2个或多个。在不使用可变参数来定义时要想实现类似的功能就需要向参数传递时传递的主是一个list或tuple。
  
　　定义一个可变参数只需要在必选参数前加上*号即可。


```python
def var(n):
    for i in n:
        print(i)
```


```python
var([1,2,3,4])      #当上边定义的var函数是需要传入一个必选参数时，以列表的形式传入参数
```

    1
    2
    3
    4



```python
def vars(*n):
    print(type(n))
    print(n)
    for i in n:
        print(i)
```


```python
vars(1,2,3,4)     #函数中的可变参数接收到传递的实参时，都会把实参组装成一个元组
```

    <class 'tuple'>
    (1, 2, 3, 4)
    1
    2
    3
    4



```python
vars(22,'a','b')   #当n是一个可变参数时，调用函数传入的参数个数是可变的
```

    <class 'tuple'>
    (22, 'a', 'b')
    22
    a
    b


4.　关键字参数

　　可变参数是在必须参数前加了一个星号，关键字参数则是加两个星号，关键参数在函数内部被组装成一个字典。而调用方式与“命名的参数”相同。


```python
def key_var(**keyword):
    print(keyword)
```


```python
key_var(name='Neal',age=30)
```

    {'age': 30, 'name': 'Neal'}


5．参数的混合使用

　　在实际使用中函数的各种参数可能会混合使用，当混合使用时，在定义函数的参数时是有顺序的，从左到右依次是：必选参数--默认参数--可选参数--关键字参数。


```python
def blend(name,gender='F',*args,**kwargs):
    print('你的姓名是{n},你的姓别是{g}，你的年龄是{e}，你的地址是{a}'.format(n=name,g=gender,e=args,a=kwargs))
```


```python
blend('Neal','M',30,add='ChonQing')
```

    你的姓名是Neal,你的姓别是M，你的年龄是(30,)，你的地址是{'add': 'ChonQing'}



```python
blend('Neal')
```

    你的姓名是Neal,你的姓别是F，你的年龄是()，你的地址是{}


* 匿名函数

　　匿名函数即没有函数名称的函数，用lambda来表示，lambda函数是一个表达式。
  
  语法如下：
  
  lambda [arg1 [,arg2,.....argn]]:expression


```python
def sum(x,y):
    if isinstance(x,(int,float)) and isinstance(y,(int,float)):
        return x + y
    else:
        raise TypeError('bad type')
```


```python
sum(5,9)
```




    14




```python
sum = lambda x,y: x + y
```


```python
sum(3,8)
```




    11




```python
sum(2,'r')   #lambda函数在python中使用有局限性，这里怎么才能检查传入参数是整形或是浮点型呢？
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-78-0d64ee959b5f> in <module>()
    ----> 1 sum(2,'r')
    

    <ipython-input-75-8a59f5f93a03> in <lambda>(x, y)
    ----> 1 sum = lambda x,y: x + y
    

    TypeError: unsupported operand type(s) for +: 'int' and 'str'

