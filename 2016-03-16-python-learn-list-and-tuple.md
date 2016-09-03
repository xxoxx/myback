---
layout: post
title: python的列表与元组
date: 2016/03/16 20:20
categories: [python学习笔记]
tags: [列表，元组]
---

　　python学习笔记之列表和元组。
<!--more-->

# List

　　序列是Python中最基本的数据结构，列表是一种序列。列表是python的一种内置的数据类型，是一种容器类，list是一种任意对象的有序集合，通过索引访问其中的元素，可以随时添加和删除其中的元素，可以任意嵌套(元素可以是另一个列表或元组等)的一种异构类型(各个元素类型可不同)，所以是一种可变对象。
  序列都可以进行的操作包括索引，切片，加，乘，检查成员，此外，Python已经内置确定序列的长度以及确定最大和最小的元素的方法。

* 创建列表

　　创建一个列表，只要把逗号分隔的不同的各个元素用中括号括起来即可。


```python
list1 = [1,2,3,4,5]
list2 = ['Neal','Tom','Cora']
list3 = ['Neal','Hello',30]   #元素的类型可以不同
list4 = ['a',[1,2,3],'b']   #列表中可以嵌套
```

* 访问列表中的值


```python
list2
    ['Neal', 'Tom', 'Cora']
list2[0]   #以索引访问列表中的元素
    'Neal'
list2[2]
    'Cora'
list2[3]    #索引值超出范围时抛出IndexError异常


    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    <ipython-input-10-3737e39fc5b9> in <module>()
    ----> 1 list2[3]
    

    IndexError: list index out of range
```

* 修改列表中的值


```python
list3
    ['Neal', 'Hello', 30]
list3[2] = 40
list3
    ['Neal', 'Hello', 40]
```

* 删除元素


```python
list2
    ['Neal', 'Tom', 'Cora']
del list2[0]  #del操作是对列表原处修改
list2
    ['Tom', 'Cora']
list2.pop(1)  # pop方法表示弹出指定索引的元素，此方法回返回弹出的元素
    'Cora'
list2
    ['Tom']
```

* 增加元素


```python
list3
    ['Neal', 'Hello', 30]
list3.insert(1,'Tome')   # insert操作在指定索引值前插入元素
list3
    ['Neal', 'Tome', 'Hello', 30]
list3.append(100)    #append方法在列表最后增加元素
list3
    ['Neal', 'Tome', 'Hello', 30, 100]
```

* 列表切片


```python
list1
    [1, 2, 3, 4, 5]
list1[0:]
    [1, 2, 3, 4, 5]
list1[1:3] # 取得从索引1开始到3的元素，但不包括索引为3的元素，顾头不顾尾
    [2, 3]
list1[-1]
    5
list1[-4:-1]
    [2, 3, 4]
list1   #列表的切片操作不会修改列表本身
    [1, 2, 3, 4, 5]
```


* 列表操作符号

　　　列表可以进行+、*运算


```python
list1
    [1, 2, 3, 4, 5]
list2
    ['Tom']
list1 + list2
    [1, 2, 3, 4, 5, 'Tom']
list22 = list2 * 4
print(list22)
    ['Tom', 'Tom', 'Tom', 'Tom']
id(list22[0])
    140193645244688
id(list22[1])
    140193645244688
id(list22[2])   #通过*号相乘得到列表中各元素都是指向同一内存地址
    140193645244688
```


* 成员判断


```python
list1
    [1, 2, 3, 4, 5]
2 in list1
    True
10 in list1
    False
```


* 常用函数和方法

 1. 函数


```python
list1
    [1, 2, 3, 4, 5]
len(list1)   #计算列表的元素个数
    5
max(list1)  #返回列表中元素的最大值
    5
min(list1)   #返回列表中元素的最小值
    1
```


2.方法


```python
list1
    [1, 2, 3, 4, 5]
list11 = ['a','b']
list1.append(list11)  #append把list11当作一个元素附加到list1上
list1
    [1, 2, 3, 4, 5, ['a', 'b']]
list1.extend(list11)   #extend方法把list11中的元素附加到list1上
list1
    [1, 2, 3, 4, 5, ['a', 'b'], 'a', 'b']
list1.sort()   #sort方法对列表进行排序，前提是列表中的元素类型相同



    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-83-05e4223c8627> in <module>()
    ----> 1 list1.sort()
    

    TypeError: unorderable types: list() < int()




list1 = [2,6,1,6,3]
list1.sort()
list1   #sort方法是对列表进行原处修改
    [1, 2, 3, 6, 6]
list1.reverse()   #reverse方法对元素进行反转
list1
    [6, 6, 3, 2, 1]
```

# Tuple

　　元组与list类似，是任意对象的有序集合，通过索引访问其中的元素，元素支持异构，但一旦元组创建后，里面的元素不可更改，是不可变对象，即没有像list那样有append()这样类似的方法来对元组进行修改。

* 元组定义与获取元组

　　元组的定义与列表几乎一样，只是把列表的[]改为()


```python
tuple1 = (3,5,8,2)
tuple1
    (3, 5, 8, 2)
tuple2 = 6,7,8,'Tom'  #这样也可以定义一个元组
tuple2
    (6, 7, 8, 'Tom')
tuple3 = ('one',)   #定义只有一个元素的元组时在元素后要跟上一个逗号
tuple3
    ('one',)
tuple4 = ('one')   #如果在元素后边没有逗号，那定义的就不是元组，而是一个字符串
tuple4
    'one'
type(tuple4)
    str
```

* 元组间的操作符


```python
t1 = (22,4,7)
t2 = ('a','b','c')
t1 + t2   #元组间的相加与列表间的相加，得到一个拥有两个元组的所有元素的元组
    (22, 4, 7, 'a', 'b', 'c')
t1 * 3
    (22, 4, 7, 22, 4, 7, 22, 4, 7)
```

* 成员关系测试


```python
t1
    (22, 4, 7)
4 in t1
    True
9 in t1
    False
```


* 元组的可变性

　　元组本身是一种不可变对象，但它的元素可以是一个可变对象，这样就河以构建成一个看似可以被修改的元组。但元组自身的内存地址是不会发生改变的。


```python
list1 = [1,2,3]
t1 = ('a','b',list1)
t1      #元组的第三个元素是一个列表
    ('a', 'b', [1, 2, 3])
id(t1)   #获取元组的内存地址
    140193645110976
t1[2]
    [1, 2, 3]
t1[2][1]
    2
t1[2][2] = 22  #修改值
t1
    ('a', 'b', [1, 2, 22])
id(t1)  #元组中一个列表中的一个元素值被修改，但内存地址没有改变
    140193645110976
```

* 元组的方法


```python
t1
    ('a', 'b', [1, 2, 22])
t1.count('a')   #count方法统计一个元素出现在次数
    1
t1.index([1,2,22]) #index方法返回元素的索引号
    2
```

