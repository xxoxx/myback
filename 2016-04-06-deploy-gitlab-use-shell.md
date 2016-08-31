---
layout: post
title: 使用shell部署gitlab
date: 2016/04/06 13:40
categories: [gitlab]
tags: [gitlab,git]
---


　　在51cto的博文[《企业级GitLab仓库环境构建》](http://zhaochj.blog.51cto.com/368705/1737738)中已全面的介绍过怎样搭建一个gitlab，这里把整个环境的搭建过程整理后用shell脚本的方式组织起来。

　　gitlab软件包安装和基础配置：

<pre><code> 
gitlab@gitlab-01:~/tools$ cat install_config_gitlab.sh 
#!/bin/bash
#Program: configure gitlab
#Author: Neal
#E_mail: 419775240@qq.com
#Date: 2015-12-04
#platform: Debian 8 x86_64
#Version 1.0

# 通用安装包到http://mirror.tuna.tsinghua.edu.cn/gitlab-ce/进行下载
sudo dpkg -i gitlab-ce_8.2.1-ce.0_amd64.deb

# 修改gitlab数据存放目录
sudo vim /var/opt/gitlab/gitlab-shell/config.yml    # repos_path: "/data/git-data/repositories"
sudo vim /var/opt/gitlab/gitlab-rails/etc/gitlab.yml
#satellites:
#     path: /data/git-data/gitlab-satellites
#     ...
#gitlab_shell:
#     path: /opt/gitlab/embedded/service/gitlab-shell/
#     repos_path: /data/git-data/repositories

# 创建数据存放目录并修改权限
sudo mkdir  -pv /data/git-data/gitlab-satellites
sudo mkdir -pv /data/git-data/repositories
sudo chown -R git.git /data/git-data/
sudo chmod 2770 /data/git-data/repositories

# restart gitlab service
sudo gitlab-ctl restart
</code></pre>

启用https的安全访问：

```sh  
#!/bin/bash
#Program: open gitbal https access
#Author: Neal
#E_mail: 419775240@qq.com
#Date: 2015-12-04
#platform: Debian 8 x86_64
#Version 1.0

# SUBJECT为CA服务的机构信息
SUBJECT="/C=CN/ST=ChongQing/L=YuBei/O=SJKJ/OU=CA"
# SUBJECT_GITLAB为gitlab主机的机构信息
SUBJECT_GITLAB="/C=CN/ST=ChongQing/L=YuBei/O=SJKJ/OU=OP"

#apt-get -y install openssl
## 自建CA 
read -p "Enter your CA domain [www.example.com]: " DOMAIN
sudo bash -c 'mkdir -pv /etc/ssl/demoCA/{private,newcerts} > /dev/null'
cd /etc/ssl
# 生成密钥对
sudo bash -c '(umask 077;openssl genrsa -out ./demoCA/private/cakey.pem 2048)'
#sudo bash -c 'ln -s /etc/ssl/demoCA/private/cakey.pem /etc/ssl/demoCA/cakey.pem'
# 生成自签证书
#sudo bash -c 'openssl req -new -x509 -key ./demoCA/private/cakey.pem -out ./demoCA/cacert.pem -days 3650'
sudo bash -c "openssl req -new -subj "${SUBJECT}/CN=$DOMAIN" -x509 -key ./demoCA/private/cakey.pem -out ./demoCA/cacert.pem -days 3650"
result=$?
sudo touch ./demoCA/index.txt
sudo bash -c "echo 01 > ./demoCA/serial"
[ $result == 0 ] && echo -e "\033[33mCertificate services is created...\033[0m" || echo -e '\033[33mCertificate services is NOT created...\033[0m'
##自建CA完成


# 为gitlab主机创建存放私钥和证书的目录，这是一个固定目录，不能更改，请参考：
# https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md#enable-https
read -p "Enter your gitlab domain [www.example.com]: " DOMAIN_GITLAB
sudo bash -c 'mkdir -p /etc/gitlab/ssl'
cd /etc/gitlab/ssl
# 生成私钥
sudo bash -c "(umask 077;openssl genrsa -out ${DOMAIN_GITLAB}.key 1024)"
# 生成证书签署请求
sudo bash -c "openssl req -new -subj ""$SUBJECT_GITLAB/CN=$DOMAIN_GITLAB"" -key ${DOMAIN_GITLAB}.key -out ${DOMAIN_GITLAB}.csr"
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

sudo gitlab-ctl reconfigure # 使配置生效
sudo gitlab-ctl restart
```

启用smtp邮箱功能：

```sh  
gitlab@gitlab-01:~/tools$ cat open_gitlab_smtp.sh 
#!/bin/bash
#Program: open gitlab smtp
#Author: Neal
#E_mail: 419775240@qq.com
#Date: 2015-12-04
#platform: Debian 8 x86_64
#Version 1.0

#以163邮箱为例
sudo bash -c "cat <<- EOF >> /etc/gitlab/gitlab.rb

	##### open smtp #####################
	gitlab_rails['smtp_enable'] = true
	gitlab_rails['smtp_address'] = \"smtp.163.com\"
	gitlab_rails['smtp_port'] = 465
	gitlab_rails['smtp_user_name'] = \"XXXXX@163.com\"
	gitlab_rails['smtp_password'] = \"***********\"
	gitlab_rails['smtp_domain'] = \"163.com\"
	gitlab_rails['smtp_authentication'] = \"login\"
	gitlab_rails['smtp_enable_starttls_auto'] = true
	gitlab_rails['smtp_tls'] = true
	gitlab_rails['gitlab_email_from'] = \"XXXX@163.com\"
EOF"

sudo gitlab-ctl reconfigure
``` 



end...
