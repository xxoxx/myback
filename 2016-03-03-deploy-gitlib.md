---
layout: post
title: shell脚本部署gitlab环境
date: 2016/03/03 16:00
categories: [版本管理工具]
tags: [gitlab部署]
---

　　部署一套gitlab环境比较复杂，这里把部署的过程写成了shell脚本，这大大简化的gitlab部署的成本。
<!--more-->

# 前言
　　在51cto的博客上已介绍过[企业级GitLab仓库环境构建](http://zhaochj.blog.51cto.com/368705/1737738)，这里利用两个shell脚本来完成gitlib的部署。

# shell scripts
　　以下脚本已在debian 8 x64系统下测试通过。


```sh
#!/bin/bash
#Program: config_gitlab.sh
#Author: Neal
#E_mail: 419775240@qq.com
#Date: 2015-12-04
#Version 1.0
#安装，包下载地址 http://mirror.tuna.tsinghua.edu.cn/gitlab-ce/
sudo dpkg -i gitlab-ce_8.2.1-ce.0_amd64.deb
#
# 修改gitlab数据存放目录
sudo vim /var/opt/gitlab/gitlab-shell/config.yml    # repos_path: "/data/git-data/repositories"
sudo vim /var/opt/gitlab/gitlab-rails/etc/gitlab.yml
#satellites:
#     path: /data/git-data/gitlab-satellites
#     ...
#gitlab_shell:
#     path: /opt/gitlab/embedded/service/gitlab-shell/
#     repos_path: /data/git-data/repositories
#
# 创建数据存放目录并修改权限
sudo mkdir  -pv /data/git-data/gitlab-satellites
sudo mkdir -pv /data/git-data/repositories
sudo chown -R git.git /data/git-data/
sudo chmod 2770 /data/git-data/repositories
#
# restart gitlab service
sudo gitlab-ctl restart
```

- open gitlab https access

```sh
#!/bin/bash
#Program: open gitbal https access
#Author: Neal
#E_mail: 419775240@qq.com
#Date: 2015-12-04
#此脚本适用于Debian 8 amd64，其他平台未做测试
#Version 1.0
#
# SUBJECT为CA服务的机构信息
SUBJECT="/C=CN/ST=ChongQing/L=YuBei/O=SJKJ/OU=CA/CN=$DOMAIN"
# SUBJECT_GITLAB为gitlab主机的机构信息
SUBJECT_GITLAB="/C=CN/ST=ChongQing/L=YuBei/O=SJKJ/OU=OP/CN=$DOMAIN_GITLAB"
#
#处理依赖
#apt-get -y install openssl
#
## 自建CA 
read -p "Enter your CA domain [www.example.com]: " DOMAIN
sudo bash -c 'mkdir -pv /etc/ssl/demoCA/{private,newcerts} > /dev/null'
cd /etc/ssl
# 生成密钥对
sudo bash -c '(umask 077;openssl genrsa -out ./demoCA/private/cakey.pem 2048)'
#sudo bash -c 'ln -s /etc/ssl/demoCA/private/cakey.pem /etc/ssl/demoCA/cakey.pem'
# 生成自签证书
#sudo bash -c 'openssl req -new -x509 -key ./demoCA/private/cakey.pem -out ./demoCA/cacert.pem -days 3650'
sudo bash -c "openssl req -new -subj $SUBJECT -x509 -key ./demoCA/private/cakey.pem -out ./demoCA/cacert.pem -days 3650"
sudo touch ./demoCA/index.txt
sudo bash -c "echo 01 > ./demoCA/serial"
echo -e "\033[33mCertificate services is created...\033[0m"
##自建CA完成
#
#
# 为gitlab主机创建存放私钥和证书的目录，/etc/gitlab/ssl是一个固定目录，不能更改，请参考：
# https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md#enable-https
read -p "Enter your gitlab domain [www.example.com]: " DOMAIN_GITLAB
sudo bash -c 'mkdir -p /etc/gitlab/ssl'
cd /etc/gitlab/ssl
# 生成私钥
sudo bash -c "(umask 077;openssl genrsa -out ${DOMAIN_GITLAB}.key 1024)"
# 生成证书签署请求
sudo bash -c "openssl req -new -subj $SUBJECT_GITLAB -key ${DOMAIN_GITLAB}.key -out ${DOMAIN_GITLAB}.csr"
# CA签署证书
cd /etc/ssl
sudo bash -c "openssl ca -in /etc/gitlab/ssl/${DOMAIN_GITLAB}.csr -out /etc/gitlab/ssl/${DOMAIN_GITLAB}.crt -days 3650"
# 更改目录权限
sudo chmod 700 /etc/gitlab/ssl
#
## 开启gitlab的https支持
sudo vim /etc/gitlab/gitlab.rb #external_url 'https://${DOMAIN_GITLAB}:2443'
sudo bash -c "cat <<- EOF >> /etc/gitlab/gitlab.rb
	##### open htts #####################
	nginx['redirect_http_to_https'] = true
	nginx['ssl_certificate'] = \"/etc/gitlab/ssl/${DOMAIN_GITLAB}.crt\"
	nginx['ssl_certificate_key'] = \"/etc/gitlab/ssl/${DOMAIN_GITLAB}.key\"
EOF"
#
sudo gitlab-ctl reconfigure # 使配置生效
sudo gitlab-ctl restart
```

# 参考资料
* https://gist.github.com/jhjguxin/c1583284bb7a0bb2f117
* https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/backups.md
* https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md
* https://github.com/xxrenzhe/YYShare_Magedu/tree/master/Gitlab%E6%9E%84%E5%BB%BA%E4%BC%81%E4%B8%9A%E7%BA%A7%E4%BB%A3%E7%A0%81%E4%BB%93%E5%BA%93_20151017/scripts
