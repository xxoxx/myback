---
layout: post
title: 日志数据删除脚本v3
date: 2016/03/03 18:40
categories: [shell script]
tags: [logs del]
---


　　在第二版中，备份目录是以保留天数的方式进行删除，当一个项目版本迭代的不是那么频繁，有可能会把备份目录下的所有备份删除，
而在备份服务器上删除备份数据也是按只保留多少天的数据，所以备份服务器上此项目的备份数据也有可能会被删除，所以在第三版中把备
份目录中备份项目的删除方式进行了修改，可以以传入参数来指定保留多少个备份，这样更为安全。
<!--more-->

```sh
#!/bin/bash
#Program: logs_clear.sh
#Author: Neal
#E_mail: 419775240@qq.com
#Date: 2016-02-15
#description: 清理日志数据和备份数据
#使用：/bin/bash logs_clear.sh $1 $2 $3，$1表示根目录被占用的百分比值,$2表示日志保留的天数，$3表示备份目录中备份文件保留份数
#Version 3.0
#变量请根据实际环境进行修改
log_dir=/home/tomcat/tomcat-7.0.54/logs.bak
bak_dir=/home/tomcat/bak
project_name=iov_mcms
log_record_file=/tmp/logs_clear.log
export PATH=/usr/local/bin:/usr/bin:/bin
#获取系统根目录使用率
root_rate=`df -h | egrep --color=auto '/$' | sed -e 's/[=/%]/ /g' | awk '{print $6}'`
[ -f ${log_record_file} ] || touch ${log_record_file}
if [ ${root_rate} -ge $1 ];then
    [ -f /tmp/tmp.tmp ] && echo "" > /tmp/tmp.tmp || touch /tmp/tmp.tmp
    echo "数据开始清理时间：$(date +%F_%T)" >> ${log_record_file}
    #清理日志文件
    find ${log_dir} -type f -mtime +$2 | tee /tmp/tmp.tmp >> ${log_record_file}  #tee是重定向操作，不是追加操作
    if [ ! -s /tmp/tmp.tmp ];then
        echo "日志目录没有可删除的文件。"
    else
        for i in `cat /tmp/tmp.tmp`;do
            rm -f $i
        done
    fi
    #清理备份文件
    let "bak_keep_num= $3 + 1"  #备份保留份数加1才能进行tail过虑
    if [ `ls -td ${bak_dir}/${project_name}* | wc -l` -ge ${bak_keep_num} ];then
        ls -td ${bak_dir}/${project_name}* | tail -n +${bak_keep_num} | tee /tmp/tmp.tmp >> ${log_record_file}
        if [ ! -s /tmp/tmp.tmp ];then
            echo "备份目录没有可删除的文件。"
        else
            for i in `cat /tmp/tmp.tmp`;do
                rm -rf $i
            done
        fi
    else
        echo "备份目录没有可删除的文件。"
    fi
    #清理空目录
    find ${log_dir} -type d -empty | egrep -v "(debug$|trace$|info$|error$|warn$)" | tee /tmp/tmp.tmp >> ${log_record_file}
    if [ ! -s /tmp/tmp.tmp ];then
        echo "日志目录中没有空目录可清理。"
    else
        for i in `cat /tmp/tmp.tmp`;do
            rmdir $i
        done
    fi
    echo -e "数据清理完成时间：$(date +%F_%T)\n" >> ${log_record_file}
fi
```
