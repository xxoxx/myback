---
layout: post
title: 日志数据删除脚本v1
date: 2016/03/02 18:40
categories: [shell script]
tags: [logs del]
---

　　在第一版中，设计的逻辑过于复杂，只要不是有特别的需求，生产环境下不必如此，只保留指定天数的数据即可，而日志和备份数据又在每天同步到备份服务器上，数据不会导致丢失。
<!--more-->

```sh
#!/bin/bash
#Program: logs_clear.sh
#Author: Neal
#E_mail: 419775240@qq.com
#Date: 2016-02-15
#description: 清理日志数据和备份数据
#使用：/bin/bash logs_clear.sh $1 $2 $3，$1表示根目录被占用的百分比值,$2表示日志保留的天数，$3表示备份目录中备份文件保留天数
#Version 1.0
#变量请根据实际环境进行修改
log_dir=/home/tomcat/tomcat-7.0.54/logs
bak_dir=/home/tomcat/bak
log_record_file=/tmp/logs_clear.log
export PATH=/usr/local/bin:/usr/bin:/bin
#获取系统根目录使用率
root_rate=`df -h | egrep --color=auto '/$' | awk '{print $5}' | sed -e 's/[=/%]/ /g'`
[ -f ${log_record_file} ] || touch ${log_record_file}
#if [ ${root_rate} -ge $1 ];then
#    [ -f /tmp/tmp.tmp ] && echo "" > /tmp/tmp.tmp || touch /tmp/tmp.tmp
#    echo "数据开始清理时间：$(date +%F_%T)" >> ${log_record_file}   
#
#    #清理日志文件
#    find ${log_dir} -type f -mtime +$2 | tee /tmp/tmp.tmp >> ${log_record_file}
#    if [ ! -s /tmp/tmp.tmp ];then
#        echo "日志目录没有可删除的文件。"
#    else
#        for i in `cat /tmp/tmp.tmp`;do
#            rm -f $i
#        done
#    fi
#
#    #清理备份文件
#    find ${bak_dir} -maxdepth 1 -type d -mtime +$3 | grep -v "/`basename ${bak_dir}`$" | tee /tmp/tmp.tmp >> ${log_record_file}
#    if [ ! -s /tmp/tmp.tmp ];then
#        echo "备份目录没有可删除的文件。"
#    else
#        for i in `cat /tmp/tmp.tmp`;do
#            rm -rf $i
#        done
#    fi
#
#    #清理空目录
#    find ${log_dir} -type d -empty | egrep -v "(debug$|trace$|info$|error$|warn$)" | tee /tmp/tmp.tmp >> ${log_record_file}
#    if [ ! -s /tmp/tmp.tmp ];then
#        echo "日志目录中没有空目录可清理。"
#    else
#        for i in `cat /tmp/tmp.tmp`;do
#            rmdir $i
#        done
#    fi
#    echo -e "数据清理完成时间：$(date +%F_%T)\n" >> ${log_record_file}
#fi
```
