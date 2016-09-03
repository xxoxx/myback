---
layout: post
title: python条件判断及循环语句
date: 2016/03/16 15:30
categories: [python学习笔记]
tags: [条件判断，循环]
---

　　python学习笔记之条件判断及循环语句。
<!--more-->



# 条件判断和循环

## 条件判断

* 单分支条件判断

```
语法：
if  条件表达式:
    执行命令1
    执行命令2
    ......
```


```python
age = 20
if age >= 18:
    print('你的年龄是{0}'.format(age))
    print('你是一个成年人')
```

    你的年龄是20
    你是一个成年人


* 双分支条件判断

```
语法：
if 条件表达示:
    执行命令1
    执行命令2
else:
    执行命令3
    执行命令4
```


```python
age = 16
if age >= 18:
    print('你的年龄是{0}'.format(age))
    print('你是一个成年人')
else:
    print('你的年龄是{age}'.format(age=age))
    print('你是一个未成年人。')
```

    你的年龄是16
    你是一个未成年人。


* 多分支条件判断

```
语法:
if 条件表达示1:
    执行命令1
elif 条件表达示2:
    执行命令2
elif 条件表达示3:
    执行命令3
.......
else:
    执行命令4
```



```python
age = 5
if age >= 18:
    print('你的年龄是{0}'.format(age))
    print('你是一个成年人')
elif age >= 6:
    print('你的年龄是{age}'.format(age=age))
    print('你是少年')
else:
    print('你的年龄是{0}'.format(age))
    print('你是一个小孩子')
```

    你的年龄是5
    你是一个小孩子


## 循环

### for循环

```
语法：
for i in 可迭代对象:
    执行语句1
    .......
```

* 打印出列表中的所有元素


```python
classmates = ['Neal','Tom','Cora']
for i in classmates:
    print(i)
```

    Neal
    Tom
    Cora


* 计算“1+2+...+100”的和


```python
sum = 0
for n in range(101):
    sum = sum + n
print(sum)
```

    5050


### while循环
　　给出的条件只要满足就一直循环，一旦条件不满足就退出循环。

* 计算100内所有奇数之和


```python
n = 1
sum = 0
while n <= 100:
    sum = sum + n
    n = n + 2
print(sum)
    
```

    2500


* 计算100内所有偶数之和


```python
n = 0
sum = 0
while n <= 100:
    sum = sum + n
    n = n + 2
print(sum)
```

    2550


