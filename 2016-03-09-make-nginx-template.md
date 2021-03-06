---
layout: post
title: nginx编译安装并引入优秀的第三方模块
date: 2016/03/09 18:40
categories: [应用]
tags: [nginx]
---
 
　　nginx是一个特别受欢迎的开源软件，但它有一些特性只有在商业版里才有，所以就有了一些好事者自己编写了一些和商业版相似功能的模块，这才开源的nginx功能更能接近商业版。这里介绍nginx源码编译安装并引入了一些优秀的第三方模块的编译过程，可供参考。
<!--more-->
 
# 前言
 
　　线上环境有多套nginx作为应用的负载均衡器，因历史原因，nginx存在各种遗留问题，版本不统一，调度算法单一，未实现对上游服务器的健康检测，配置缩进不规范，所有的
upstream，server都配置在nginx.conf文件中，看起配置比较冗长。为了实现环境部署的规范化，需要规范nginx的目录结构、配置文件的缩进，并制作一个nginx环境的部署模板，
在当有需要一个nginx环境需求时能通过简单的操作就能部署一个功能完整的nginx环境。
 
# 源码编译安装nginx
 
　　目前nginx的稳定版本为“nginx-1.8.1”，以此版本来编译安装。nginx有许多非常好的特性，在查看官方帮助文档时，有时看到一个功能特别强大的指令时，正当你在欢喜时，你
又会发现有如下的说明:
 
> This directive is available as part of our [commercial subscription](http://nginx.com/products/?_ga=1.198063855.1530886050.1455804017).
 
不知道此你有何感想。好在开源的力量是不可估量的，nginx许多商业化版本中才支持的功能在github上都能找到类似的开源项目。这次制作nginx为模板时也加进了几个第三方模块，
使nginx的功能更完善。
 
我的编译平台如下：
 
```sh
root@nginx-02:~/tools# uname -r
3.16.0-4-amd64
root@nginx-02:~/tools# uname -v
#1 SMP Debian 3.16.7-ckt20-1+deb8u1 (2015-12-14)
```
 
引入的第三方模块列表如下：
 
- [nginx_upstream_check_module](https://github.com/yaoweibin/nginx_upstream_check_module)　
 
　　此模块能实现对上游服务器的基于tcp、http、ssl、hello、mysql、ajp、fastcgi的健康检测，并能把被标记为down的主机踢出负载调度。
 
- [nginx_tcp_proxy_module](https://github.com/yaoweibin/nginx_tcp_proxy_module)　
 
　　此模块实现nginx的四层调度，nginx-1.9已原生支持四层调度了，但目前不是稳定版本。此模块还提供了一个对四层调试的状态监控页面。
 
- [nginx-upstream-fair](https://github.com/gnosek/nginx-upstream-fair)　
 
　　此模块增强了round-robin负载均衡算法，可以跟踪后端服务器的负载来分发请求，觉得有点类似least_conn算法。
 
- [nginx-sticky-module](https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng/get/master.tar.gz)　
 
　　此模块实现了session保持，比ip_hash的粒度更小，它是基于用户浏览器cookie来保持会话。
 
- [ngx_log_if](https://github.com/cfsego/ngx_log_if/)
 
　　此模块实现日志过虑功能，此模块是做好模板后才有这样的功能需求，所以下边的演示没有加入此模块。
 
## 第三方模块安装
 
```sh
root@nginx-02:~/tools# pwd
/root/tools
root@nginx-02:~/tools# ls
nginx-1.8.1  nginx-1.8.1.tar.gz  part3
root@nginx-02:~/tools/part3# pwd
/root/tools/part3
root@nginx-02:~/tools/part3# ls -l
total 524
drwxrwxr-x 4 root root   4096 Aug  6  2015 nginx-goodies-nginx-sticky-module-ng-c78b7dd79d0d
-rw-r--r-- 1 root root 120553 Mar  8 09:53 nginx-goodies-nginx-sticky-module-ng-c78b7dd79d0d.tar.gz
drwxr-xr-x 7 root root   4096 Aug 18  2015 nginx_tcp_proxy_module-master
-rw-r--r-- 1 root root 213069 Mar  8 10:28 nginx_tcp_proxy_module-master.zip
drwxr-xr-x 6 root root   4096 Jul  1  2015 nginx_upstream_check_module-master
-rw-r--r-- 1 root root 167495 Mar  8 16:50 nginx_upstream_check_module-master.zip
drwxr-xr-x 2 root root   4096 Apr  8  2012 nginx-upstream-fair-master
-rw-r--r-- 1 root root  10845 Mar  8 09:28 nginx-upstream-fair-master.zip
```
 
**特别注意**
 
　　各个模块的安装方法请参数源码目录下的帮助文档，经过反复验证，在安装以上第三方模块时，有些坑是值得注意的。nginx_upstream_check_module与nginx_tcp_proxy_module
不能同时导入到nginx源码，不然在make时会报错，所以请严格按照以下顺序来安装。
 
- 第一步：先解决编译安装nginx的依赖并创建运行nginx work进程的用户
 
```sh
root@nginx-02:~/tools/nginx-1.8.1# apt-get -y install openssl libssl-dev libpcre3 libpcre3-dev
```
 
> libpcre3 libpcre3-dev    HTTP rewrite module requires the PCRE library
 
```sh
root@nginx-02:~/tools/nginx-1.8.1# useradd -m -d /home/nginx -s /bin/bash nginx   #可不配置密码
```
 
- 第二步：导入第三方nginx_upstream_check_module模块
 
```sh
root@nginx-02:~/tools/nginx-1.8.1# patch -p1 < ../part3/nginx_upstream_check_module-master/check_1.7.5+.patch 
patching file src/http/modules/ngx_http_upstream_hash_module.c
Hunk #3 succeeded at 528 (offset 10 lines).
patching file src/http/modules/ngx_http_upstream_ip_hash_module.c
patching file src/http/modules/ngx_http_upstream_least_conn_module.c
patching file src/http/ngx_http_upstream_round_robin.c
patching file src/http/ngx_http_upstream_round_robin.h
```
 
- 第三步：编译安装nginx
 
nginx-upstream-fair-master和nginx-goodies-nginx-sticky-module-ng-c78b7dd79d0d模块不需要做额外处理，只需要在编译时用--add-module指令加入即可。
 
```sh
root@nginx-02:~/tools/nginx-1.8.1# ./configure --prefix=/usr/local/nginx18 \
--user=nginx \
--group=nginx \
--with-file-aio \
--pid-path=/var/run/nginx18.pid \
--lock-path=/var/lock/subsys/nginx18 \
--with-http_ssl_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-http_auth_request_module \
--with-http_realip_module \
--with-http_secure_link_module \
--http-client-body-temp-path=/var/tmp/nginx18/client \
--http-proxy-temp-path=/var/tmp/nginx18/proxy \
--http-fastcgi-temp-path=/var/tmp/nginx18/fastcgi \
--http-uwsgi-temp-path=/var/tmp/nginx18/uwsgi \
--http-scgi-temp-path=/var/tmp/nginx18/scgi \
--add-module=../part3/nginx-upstream-fair-master \
--add-module=../part3/nginx-goodies-nginx-sticky-module-ng-c78b7dd79d0d \
--add-module=../part3/nginx_upstream_check_module-master
 
root@nginx-02:~/tools/nginx-1.8.1# make
root@nginx-02:~/tools/nginx-1.8.1# make install
root@nginx-02:~/tools/nginx-1.8.1# mkdir -pv /var/tmp/nginx18/{proxy,fastcgi,uwsgi,scgi}
root@nginx-02:~/tools/nginx-1.8.1# tree /usr/local/nginx18/
/usr/local/nginx18/
├── conf
│   ├── fastcgi.conf
│   ├── fastcgi.conf.default
│   ├── fastcgi_params
│   ├── fastcgi_params.default
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types
│   ├── mime.types.default
│   ├── nginx.conf
│   ├── nginx.conf.default
│   ├── scgi_params
│   ├── scgi_params.default
│   ├── uwsgi_params
│   ├── uwsgi_params.default
│   └── win-utf
├── html
│   ├── 50x.html
│   └── index.html
├── logs
└── sbin
    └── nginx
```
 
查看nginx编辑进的模块：
 
![](/images/2016-03-09-nginx-module-1.jpg)
 
- 第四步：导入第三方nginx_tcp_proxy_module模块
 
```sh
root@nginx-02:~/tools/nginx-1.8.1# patch -p1 < ../part3/nginx_tcp_proxy_module-master/tcp.patch 
patching file src/core/ngx_log.c
Hunk #1 succeeded at 86 (offset 20 lines).
patching file src/core/ngx_log.h
Hunk #1 succeeded at 30 (offset 1 line).
Hunk #2 succeeded at 38 (offset 1 line).
patching file src/event/ngx_event_connect.h
Hunk #1 succeeded at 33 (offset 1 line).
Hunk #2 succeeded at 45 with fuzz 2 (offset 2 lines).
```
 
- 第五步：重新编译nginx
 
```sh
root@nginx-02:~/tools/nginx-1.8.1# ./configure --prefix=/usr/local/nginx18 \
--user=nginx \
--group=nginx \
--with-file-aio \
--pid-path=/var/run/nginx18.pid \
--lock-path=/var/lock/subsys/nginx18 \
--with-http_ssl_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-http_auth_request_module \
--with-http_realip_module \
--with-http_secure_link_module \
--http-client-body-temp-path=/var/tmp/nginx18/client \
--http-proxy-temp-path=/var/tmp/nginx18/proxy \
--http-fastcgi-temp-path=/var/tmp/nginx18/fastcgi \
--http-uwsgi-temp-path=/var/tmp/nginx18/uwsgi \
--http-scgi-temp-path=/var/tmp/nginx18/scgi \
--add-module=../part3/nginx-upstream-fair-master \
--add-module=../part3/nginx-goodies-nginx-sticky-module-ng-c78b7dd79d0d \
--add-module=../part3/nginx_upstream_check_module-master \
--add-module=../part3/nginx_tcp_proxy_module-master             #增加了nginx_tcp_proxy_module-master模块
root@nginx-02:~/tools/nginx-1.8.1# make   #make后一定不要make install操作
```
 
拷贝重新编译好的nginx二进制文件到nginx的安装目录，如下：
 
```sh
root@nginx-02:~/tools/nginx-1.8.1# ls
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  Makefile  man  objs  README  src
root@nginx-02:~/tools/nginx-1.8.1# ls objs/
addon  autoconf.err  Makefile  nginx  nginx.8  ngx_auto_config.h  ngx_auto_headers.h  ngx_modules.c  ngx_modules.o  src
root@nginx-02:~/tools/nginx-1.8.1# cp objs/nginx /usr/local/nginx18/sbin/
```
 
再次查看编译进nginx的模块：
 
![](/images/2016-03-09-nginx-module-2.jpg)
 
很明显多了nginx_tcp_proxy_module这个模块。
 
- 第六步：提供systemctl控制的启动脚本及收尾工作
 
```sh
root@nginx-02:~/tools/nginx-1.8.1# vim /lib/systemd/system/nginx.service 
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network.target
 
[Service]
Type=forking
PIDFile=/var/run/nginx18.pid
ExecStartPre=/usr/local/nginx18/sbin/nginx -t -c /usr/local/nginx18/conf/nginx.conf
ExecStart=/usr/local/nginx18/sbin/nginx -c /usr/local/nginx18/conf/nginx.conf
ExecReload=/usr/local/nginx18/sbin/nginx -s reload
ExecStop=/usr/local/nginx18/sbin/nginx -s stop
TimeoutStopSec=5
KillMode=mixed
 
[Install]
WantedBy=multi-user.target
 
root@nginx-02:~/tools/nginx-1.8.1# systemctl enable nginx.service 
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /lib/systemd/system/nginx.service.
root@nginx-02:~/tools/nginx-1.8.1# systemctl start nginx.service  #启动
root@nginx-02:~/tools/nginx-1.8.1# ss -tnlp | grep nginx
LISTEN     0      128                       *:80                       *:*      users:(("nginx",pid=10935,fd=6),("nginx",pid=10934,fd=6))
 
 
root@nginx-02:~/tools/nginx-1.8.1# mkdir -pv /var/tmp/nginx18/{proxy,client,fastcgi,uwsgi,scgi}   #创建临时目录
root@nginx-02:~/tools/nginx-1.8.1# vim /etc/profile.d/nginx18.sh  #导出二进制文件
export PATH=/usr/local/nginx18/sbin:$PATH
root@nginx-02:~/tools/nginx-1.8.1# source /etc/profile.d/nginx18.sh
```
 
## 关于线程池
 
　　线程池在nginx-1.8是支持的，但在当我启用了“--with-threads”功能，并加增加“--add-module=../part3/nginx_tcp_proxy_module-master”模块后，make时会报如下错：
 
```sh
../part3/nginx_tcp_proxy_module-master/ngx_tcp_upstream_round_robin.c: In function ‘ngx_tcp_upstream_get_round_robin_peer’:
../part3/nginx_tcp_proxy_module-master/ngx_tcp_upstream_round_robin.c:459:16: error: ‘ngx_event_t’ has no member named ‘lock’
         c->read->lock = c->read->own_lock;
                ^
../part3/nginx_tcp_proxy_module-master/ngx_tcp_upstream_round_robin.c:459:32: error: ‘ngx_event_t’ has no member named ‘own_lock’
         c->read->lock = c->read->own_lock;
                                ^
../part3/nginx_tcp_proxy_module-master/ngx_tcp_upstream_round_robin.c:460:17: error: ‘ngx_event_t’ has no member named ‘lock’
         c->write->lock = c->write->own_lock;
                 ^
../part3/nginx_tcp_proxy_module-master/ngx_tcp_upstream_round_robin.c:460:34: error: ‘ngx_event_t’ has no member named ‘own_lock’
         c->write->lock = c->write->own_lock;
                                  ^
objs/Makefile:1298: recipe for target 'objs/addon/nginx_tcp_proxy_module-master/ngx_tcp_upstream_round_robin.o' failed
make[1]: *** [objs/addon/nginx_tcp_proxy_module-master/ngx_tcp_upstream_round_robin.o] Error 1
make[1]: Leaving directory '/root/tools/nginx-1.8.1'
Makefile:8: recipe for target 'build' failed
make: *** [build] Error 2
```
只好作罢，如果真的业务需要启用threads，再去掉“nginx_tcp_proxy_module”模块后重新编译。
 
# 规范
 
　　至此，我们编译安装的nginx已准备好。在生产环境中，gninx的配置文件改动得比较频繁，如果没有规划好配置文件该怎么修改？各个目录该存放什么文件？缩进是否要严格要求？
随时间的推移，nginx的配置文件将变得不易维护，所以约定如下：
 
- nginx.conf配置文件只启用nginx健康检测的页面的虚拟主机，并使用include语句把“/usr/local/nginx18/conf”目录下的"layer4"和“layer7”目录下的任何以“.conf”结尾的文件
包含进来，目录结构如下：
 
```sh
root@nginx-02:/usr/local/nginx18/conf# pwd
/usr/local/nginx18/conf
root@nginx-02:/usr/local/nginx18/conf# ll
total 68
-rw-r--r-- 1 root staff 1034 Mar  9 22:03 fastcgi.conf
-rw-r--r-- 1 root staff 1034 Mar  9 22:03 fastcgi.conf.default
-rw-r--r-- 1 root staff  964 Mar  9 22:03 fastcgi_params
-rw-r--r-- 1 root staff  964 Mar  9 22:03 fastcgi_params.default
-rw-r--r-- 1 root staff 2837 Mar  9 22:03 koi-utf
-rw-r--r-- 1 root staff 2223 Mar  9 22:03 koi-win
drwxr-sr-x 2 root staff 4096 Mar  9 22:03 layer4
drwxr-sr-x 2 root staff 4096 Mar  9 22:03 layer7
-rw-r--r-- 1 root staff 3957 Mar  9 22:03 mime.types
-rw-r--r-- 1 root staff 3957 Mar  9 22:03 mime.types.default
-rw-r--r-- 1 root staff 3480 Mar  9 22:03 nginx.conf
-rw-r--r-- 1 root staff 2656 Mar  9 22:03 nginx.conf.default
-rw-r--r-- 1 root staff  596 Mar  9 22:03 scgi_params
-rw-r--r-- 1 root staff  596 Mar  9 22:03 scgi_params.default
-rw-r--r-- 1 root staff  623 Mar  9 22:03 uwsgi_params
-rw-r--r-- 1 root staff  623 Mar  9 22:03 uwsgi_params.default
-rw-r--r-- 1 root staff 3610 Mar  9 22:03 win-utf
```
- 上边规划的layer4和layer7两个目录是分别放置4层调度和7层调度时各配置文件，文件以“.conf”结尾；
- 各虚拟主机的配置文件名称以“server_name”中的值来命名，点号转换成“_”，如“server_name www.test.cn”，那相应的配置文件为“www_test_cn.conf”，在layer4目录下的文件也
类似，只是它没有“server_name”字段，以upstream字段的值来命名配置文件，如“upstream cmdserver”，那配置文件为“cmdserver.conf”;
- layer7目录下各虚拟主机产生的访问日志和错误日志名称单独规划，参照配置文件的名称，日志名称如“www_test_cn.access.log”和"www_test_cn.error.log"；
- 配置文件中的代码缩进严格按照各代码块的层级关系缩进。
 
# 配置示例
 
- nginx.conf配置示例
 
```sh
root@nginx-02:/usr/local/nginx18/conf# pwd
/usr/local/nginx18/conf
root@nginx-02:/usr/local/nginx18/conf# cat nginx.conf
 
#user  nobody;
user  nginx;
worker_processes  1;
 
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
 
#pid        logs/nginx.pid;
pid        /var/run/nginx18.pid;
 
 
events {
    #worker_connections  1024;
    worker_connections  65535;
}
 
 
http {
    include       mime.types;
    default_type  application/octet-stream;
 
    # $upstream_cache_status 记录缓存命中率
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"'
                      '"$upstream_cache_status"';
 
    access_log  logs/access.log  main;
 
    proxy_temp_path /usr/local/nginx18/proxy_temp;
    proxy_cache_path /usr/local/nginx18/proxy_cache levels=1:2 keys_zone=cache_one:100m inactive=2d max_size=2g;
 
    sendfile        on;
    #tcp_nopush     on;
 
    #keepalive_timeout  0;
    keepalive_timeout  65;
 
    client_header_buffer_size 4k;
 
    #gzip  on;
 
    # http://nginx.org/en/docs/http/server_names.html
    proxy_headers_hash_max_size 512;
    proxy_headers_hash_bucket_size 128;
 
    server {
        listen       80;
        server_name  localhost;
 
        #charset koi8-r;
 
        #access_log  logs/host.access.log  main;
 
        location / {
            root   html;
            index  index.html index.htm;
        }
        location /layer4-status {
            tcp_check_status;
            access_log off;
        }
        location /layer7-status {
            check_status;
            access_log off;
        #    allow 172.16.0.0/24;
        #    deny all;
        }
 
        #error_page  404              /404.html;
 
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
 
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}
 
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
 
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
 
 
    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
 
 
    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;
 
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;
 
    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;
 
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    include       layer7/*.conf;
}
include       layer4/*.conf;
root@nginx-02:/usr/local/nginx18/conf# 
```
 
- 四层调度配置示例
 
```sh
root@nginx-02:/usr/local/nginx18/conf/layer4# pwd
/usr/local/nginx18/conf/layer4
root@nginx-02:/usr/local/nginx18/conf/layer4# ls
cmdserver.conf
root@nginx-02:/usr/local/nginx18/conf/layer4# cat cmdserver.conf 
tcp {
        upstream cmdserver {
            ip_hash;
            # simple round-robin
            server 172.31.11.70:9100;
            server 172.31.11.71:9100;
            check interval=3000 rise=2 fall=5 timeout=1000 type=tcp;
        }
 
        server {
            listen 8500;
            proxy_pass cmdserver;
        }
    }
root@nginx-02:/usr/local/nginx18/conf/layer4#
```
 
- 七层调度配置示例
 
```sh
root@nginx-02:/usr/local/nginx18/conf/layer7# pwd
/usr/local/nginx18/conf/layer7
root@nginx-02:/usr/local/nginx18/conf/layer7# ls
www_test_cn.conf
root@nginx-02:/usr/local/nginx18/conf/layer7# cat www_test_cn.conf 
upstream test_server {
    sticky;  # or round-robin or fair
    server 172.31.0.10:80 weight=1;
    server 172.31.11.96:80 weight=1 down;
    # interval表示健康检查发送的间隔时间，单位为ms，rise表示把服务器标记为up状态前需要检查成功的次数，
    # fall表示把服务器标记为down状态前需要检查失败的次数，timeout表示健康检查时的超时时间，超过此时间就标记为一次检查失败.
    check interval=5000 rise=2 fall=5 timeout=1000 type=http port=80;
    check_http_send "HEAD / HTTP/1.0\r\n\r\n";
    check_http_expect_alive http_2xx http_3xx;
}
 
server {
    listen 8080;
    client_max_body_size 50M;
    server_name www.test.cn;
    access_log  logs/www_test_cn.access.log  main;
    error_log logs/www_test_cn.error.log debug;
 
    location ~ .*\.(gif|jpg|html|css|js|ico|swf|pdf)(.*) {
        proxy_pass http://test_server;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
 
        proxy_cache cache_one;
        add_header Nginx-Cache $upstream_cache_status;
        proxy_cache_valid 200 304 301 302 8h;
        proxy_cache_valid 404 500 1m;
        proxy_cache_valid any 2d;
        proxy_cache_key $host$uri$is_args$args;
        expires 30d;
 
    }
    
    location / {
        proxy_pass http://test_server;
 
        # 配置nginx转发客户端IP
        proxy_set_header X-Real-IP $remote_addr;
        # needed for HTTPS
        # proxy_set_header X_FORWARDED_PROTO https;
        
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
 
        # NginxHttpRealIpModule，指向上一级反向代理服务器(HAproxy or nginx or other)
        # 把从172.31.0.0/16网段来的请求全部使用X-Forwarded-For里的头信息作为remote_addr,
        set_real_ip_from 172.31.0.0/16;
        set_real_ip_from 172.31.0.100;
        real_ip_header X-Real-IP;
    }
}
root@nginx-02:/usr/local/nginx18/conf/layer7#
```
 
# 总结
 
　　以后若有nginx环境的需要，可以直接把“/usr/local/nginx18”目录拷贝到目标主机，再把“/lib/systemd/system/nginx.service”文件拷贝到目标主机，一个nginx环境就可以运行了。
