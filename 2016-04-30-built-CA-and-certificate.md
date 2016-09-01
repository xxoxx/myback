---
layout: post
title: 自建CA及证颁发证书shell脚本
date: 2016/04/30 16:40
categories: [shell]
tags: [自建CA]
---

   自建CA及为证书颁发是一个复杂有繁琐的事情，所以这里写一个shell脚本来简化相应的工作。
<!--more-->

　　以下shell脚本能简化手动搭建CA及证书颁发的操作，已在 `debian 8 x64`平台验证通过。


<pre><code>  
#!/bin/bash
#Program: ca.sh
#Author: Neal
#E_mail: 419775240@qq.com
#Date: 2016-4-30
#platform: Debian 8 x86_64
#Version 1.0

# SUBJECT为CA服务的机构信息
SUBJECT="/C=CN/ST=ChongQing/L=YuBei/O=SJKJ/OU=CA"
# SUBJECT_REQUEST为需要申请证书的机构信息
SUBJECT_REQUEST="/C=CN/ST=ChongQing/L=YuBei/O=SJKJ/OU=OP"
#证书存放目录
SSL_DIR="/usr/local/nginx18/conf/ssl"

#apt-get -y install openssl  依赖包
## 自建CA 
read -p "Enter your CA domain [www.example.com]: " DOMAIN
mkdir -pv /etc/ssl/demoCA/{private,newcerts} > /dev/null
cd /etc/ssl
# 生成密钥对
(umask 077;openssl genrsa -out ./demoCA/private/cakey.pem 2048)
# 生成自签证书
#手动输入ca机构信息时输入命令：openssl req -new -x509 -key ./demoCA/private/cakey.pem -out ./demoCA/cacert.pem -days 3650
openssl req -new -subj "${SUBJECT}/CN=$DOMAIN" -x509 -key ./demoCA/private/cakey.pem -out ./demoCA/cacert.pem -days 3650
result=$?
touch ./demoCA/index.txt
echo 01 > ./demoCA/serial
[ $result == 0 ] && echo -e "\033[33mCertificate services is created...\033[0m" || echo -e '\033[33mCertificate services is NOT created...\033[0m'
#自建CA完成


# 开始为你的域名申请证书
read -p "Enter your domain [www.example.com]: " DOMAIN_GITLAB
[ ! -d ${SSL_DIR} ] && mkdir -p ${SSL_DIR}
cd ${SSL_DIR}
# 生成私钥
(umask 077;openssl genrsa -out ${DOMAIN_GITLAB}.key 1024)
# 生成证书签署请求
openssl req -new -subj "$SUBJECT_REQUEST/CN=$DOMAIN_GITLAB" -key ${DOMAIN_GITLAB}.key -out ${DOMAIN_GITLAB}.csr
# CA签署证书
cd /etc/ssl
openssl ca -in ${SSL_DIR}/${DOMAIN_GITLAB}.csr -out ${SSL_DIR}/${DOMAIN_GITLAB}.crt -days 3650
# 更改目录权限
chmod 700 ${SSL_DIR}
</code></pre>
