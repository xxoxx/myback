---
layout: post
title: 批量替换文件中指定字符串
date: 2016/03/18 21:30
categories: [文本处理]
tags: [shell，python]
---

# 业务需求
　　现有两个文件，文件1里有两列，第一列为字符串A，第二列为字符串B；文件2中有大量的sql语句，单个文件
上百兆，现需要把文件1中的第一列的每行的字符串A拿到文件2中去查找，如果能查找到则替换为这一行中的字符
串B。

　　两个文件内的内容大致如下：
　　
![](/images/2016-03-18-text-process-01.jpg)

# shell实现

```sh
#!/bin/bash
#Program: process.sh
#Author: Neal
#E_mail: 419775240@qq.com
#Date: 2016-3.16
#Version 1.0
	
source_file=source.txt.bak
chang_file=chang_file.txt.bak
	
while read line;do
    raw=`echo $line | awk '{print $1}'`
    now=`echo $line | awk '{print $2}'`
    process_now=`echo ${now%^M}`   #处理now变量最后有^M符号,此符号是“Ctrl+v + Ctrl+m”组合键的结果
    grep "$raw" ${source_file} > /dev/null 2>&1
        if [ $? == 0 ];then
            echo "已匹配到$raw，会更换为$process_now"
            sed -i "s/${raw}/${process_now}/g" ${source_file} > /dev/null 2>&1
        fi
done < ${chang_file}
```

# python实现

```python
#!/home/ansible/.pyenv/versions/python_3.5.1/bin/python
# -*- coding: utf-8 -*-
		
import re
	
	
def read_source_file():
    """
    read the source file.
    """
    with open('source.txt.bak',mode='r') as f:
        source_file = f.read()
    return source_file
	
	
def replace(x,y):
    return re.sub(x[0],x[1],y)
	

def rw(n):
    """
    write str to new file
    """
    with open('result.txt',mode='w') as chg:
        chg.write(n)
	

if __name__ == '__main__':
    str_source_file = read_source_file()
    with open('chang_file.txt.bak',mode='r') as f:
        for line in f:
            line_list = line.split()
            str_source_file = replace(line_list,str_source_file)
    rw(str_source_file)
```

