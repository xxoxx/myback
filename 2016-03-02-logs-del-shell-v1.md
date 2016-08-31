---
layout: post
title: 日志数据删除脚本v1
date: 2016/03/02 17:00
categories: [shell script]
tags: [logs del]
---

```sh
#!/bin/bash
#Program: logs_clear.sh 
#Author: Neal
#E_mail: 419775240@qq.com
#Date: 2016-02-15
#description: 清理日志数据和备份数据
#使用：/bin/bash logs_clear.sh N，N表示根目录被占用的百分比值
#Version 1.0
#使用此脚本前提：目录结构为$CATALINA_HOME/logs/Project_Name/{info,debug,trace,warn,error}

#变量请根据实际环境进行修改
project_name=iov_mcms
log_dir=/home/tomcat/tomcat-7.0.54/logs
log_record_file=/tmp/logs_clear.log
#项目产生日志文件保留份数=num-1
num=11
#备份目录
bak_dir=/home/tomcat/bak
#版本保留份数,因升级前后会各备份一次，后边使用"tail -n +N" 的方式获取要删除的目录，所以版本保留份数=(bak_dir_num-1) * 2
bak_dir_num=7
#tomcat自身日志保留份数为(tomcat_logs_num - 1)
tomcat_logs_num=7
###########################part 1,日志清理########################################
#获取系统根目录使用率
root_rate=`df -h | egrep --color=auto '/$' | awk '{print $5}' | sed -e 's/[=/%]/ /g'`
#声明函数
log_record () {
    [ -f ${log_record_file} ] || touch ${log_record_file}
}
#在执行删除日志文件前做一些前置操作记录日志
pre_del_log () {
    #过虑掉最新的(num - 1)个日志文件后再计数
    if [ `ls -t ${log_dir}/${project_name}/$1 | tail -n +${num} | wc -l` -gt 0 ];then
        echo "删除${log_dir}/${project_name}/$1目录下的以下文件：" >> ${log_record_file}
        ls -t ${log_dir}/${project_name}/$1 | tail -n +${num} >> ${log_record_file}
    else
        echo "$1级别的日志中没有可删除的日志文件"
    fi
}
#tomcat启动时自身产生日志清理函数
catalina_clear () {
    if [ `ls -t ${log_dir}/catalina.* | tail -n +${tomcat_logs_num} | wc -l` -gt 0 ];then
        rm -f $(ls -t ${log_dir}/catalina.* | tail -n +${tomcat_logs_num})
    fi
}
host-manager_clear () {
    if [ `ls -t ${log_dir}/host-manager.* | tail -n +${tomcat_logs_num} | wc -l` -gt 0 ];then
        rm -f $(ls -t ${log_dir}/host-manager.* | tail -n +${tomcat_logs_num})
    fi
}
localhost_clear () {
    if [ `ls -t ${log_dir}/localhost.* | tail -n +${tomcat_logs_num} | wc -l` -gt 0 ];then
        rm -f $(ls -t ${log_dir}/localhost.* | tail -n +${tomcat_logs_num})
    fi
}
localhost_access_log_clear () {
    if [ `ls -t ${log_dir}/localhost_access_log.* | tail -n +${tomcat_logs_num} | wc -l` -gt 0 ];then
        rm -f $(ls -t ${log_dir}/localhost_access_log.* | tail -n +${tomcat_logs_num})
    fi
}
manager_clear () {
    if [ `ls -t ${log_dir}/manager.* | tail -n +${tomcat_logs_num} | wc -l` -gt 0 ];then
        rm -f $(ls -t ${log_dir}/manager.* | tail -n +${tomcat_logs_num})
    fi
}
#工程产生各级别日志清理函数
clear_info_log () {
    #过虑掉最新的(num - 1)个日志文件后再计数
    if [ `ls -t ${log_dir}/${project_name}/info | tail -n +${num} | wc -l` -gt 0 ];then
        cd ${log_dir}/${project_name}/info
        #以时间排序，保留最新的num-1份数据
        rm -f $(ls -t ${log_dir}/${project_name}/info | tail -n +${num})
        echo "info日志删除成功"
    fi       
}
clear_debug_log () {
    #过虑掉最新的(num - 1)个日志文件后再计数
    if [ `ls -t ${log_dir}/${project_name}/debug | tail -n +${num} | wc -l` -gt 0 ];then
        cd ${log_dir}/${project_name}/debug
        #以时间排序，保留最新的num-1份数据
        rm -f $(ls -t ${log_dir}/${project_name}/debug | tail -n +${num})
        echo "debug日志删除成功"
    fi       
}
clear_trace_log () {
    #过虑掉最新的(num - 1)个日志文件后再计数
    if [ `ls -t ${log_dir}/${project_name}/trace | tail -n +${num} | wc -l` -gt 0 ];then
        cd ${log_dir}/${project_name}/trace
        #以时间排序，保留最新的num-1份数据
        rm -f $(ls -t ${log_dir}/${project_name}/trace | tail -n +${num})
        echo "trace日志删除成功"
    fi       
}
clear_warn_log () {
    #过虑掉最新的(num - 1)个日志文件后再计数
    if [ `ls -t ${log_dir}/${project_name}/warn | tail -n +${num} | wc -l` -gt 0 ];then
        cd ${log_dir}/${project_name}/warn
        #以时间排序，保留最新的num-1份数据
        rm -f $(ls -t ${log_dir}/${project_name}/warn | tail -n +${num})
        echo "warn日志删除成功"
    fi       
}
clear_error_log () {
    #过虑掉最新的(num - 1)个日志文件后再计数
    if [ `ls -t ${log_dir}/${project_name}/error | tail -n +${num} | wc -l` -gt 0 ];then
        cd ${log_dir}/${project_name}/error
        #以时间排序，保留最新的num-1份数据
        rm -f $(ls -t ${log_dir}/${project_name}/error | tail -n +${num})
        echo "error日志删除成功"
    fi       
}
#定义脚本触发条件,ge表示大于或等于
#if [ $root_rate -ge $1 ];then
#    #tomcat环境时调用清理自身产生的日志
#    catalina_clear
#    host-manager_clear
#    localhost_clear
#    localhost_access_log_clear
#    manager_clear
#    #打开日志记录
#    log_record
#    echo "脚本执行开始时间：$(date +%F_%T)" >> ${log_record_file}
#    #调用函数并传递参数
#    pre_del_log info
#    pre_del_log debug
#    pre_del_log trace
#    pre_del_log warn
#    pre_del_log error
#    #调用删各除级别日志函数
#    clear_info_log
#    clear_debug_log
#    clear_trace_log
#    clear_warn_log
#    clear_error_log
#fi
###########################part 2,备份文件删除########################################
pre_del_bak () {
    echo "删除${bak_dir}目录下的以下文件：" >> ${log_record_file}
    ls -td ${bak_dir}/${project_name}* | tail -n +${bak_dir_num} >> ${log_record_file}   
}
#if [ `ls -td ${bak_dir}/${project_name}* | wc -l` -ge ${bak_dir_num} ];then
#    pre_del_bak
#    #保留以时间排序后的最新的dir_num - 1个备份工程目录，即保留最新的三个版本
#    rm -rf $(ls -td ${bak_dir}/${project_name}* | tail -n +${bak_dir_num})
#else
#    echo "工程的备份版本数量不足，不会删除任何数据。"
#fi
#echo -e "脚本执行完毕，脚本结束时间：$(date +%F_%T)\n" >> ${log_record_file}
```
