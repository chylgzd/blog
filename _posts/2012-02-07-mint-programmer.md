---
layout: post
title: 开发运维相关
category: [Linux-Mint,CentOs6,SpringBoot,Redis]
comments: false
---

* content
{:toc}

### Linux系统通用相关

#### 安装Redis
```
> tar -zxf redis-4.0.11.tar.gz -C /data/redis/
> cd  /data/redis/redis-4.0.11/
> make
> vim redis.conf :

bind 0.0.0.0（远程机器访问需要）
port 6377
protected-mode no（是否允许远程机器访问）
daemonize yes（是否后台进程启动）
requirepass pwd123456（设置redis访问密码）

> ./src/redis-server ./redis.conf
> ./src/redis-client -h 127.0.0.1 -p 6377
>>KEYS *
(error) NOAUTH Authentication required.
>>auth pwd123456
ok

```

#### 安装Let's Encrypt证书
```
参考：
https://imququ.com/post/letsencrypt-certificate.html
https://imququ.com/post/my-nginx-conf.html
https://github.com/diafygi/acme-tiny

> mkdir -p /data/dev/letsencrypt/mytest.com/ssl
> cd /data/dev/letsencrypt/mytest.com/ssl
> openssl genrsa 4096 > account.key
> openssl ecparam -genkey -name secp384r1 | openssl ec -out domain.key
> openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:mytest.com,DNS:www.mytest.com")) > domain.csr
(
如果出现cat: /etc/ssl/openssl.cnf: 没有那个文件或目录,
则先find / -name openssl.cnf，再替换命令里的openssl.cnf文件目录（centos:/etc/pki/tls/openssl.cnf）
)
> wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py
> mkdir -p /data/dev/letsencrypt/mytest.com/www/challenges
> vim /etc/nginx/conf.d/mytest.com.conf 
server {
    listen       80;
    server_name mytest.com www.mytest.com;
    location ^~ /.well-known/acme-challenge/ {
        alias /data/dev/letsencrypt/mytest.com/www/challenges/;
        try_files $uri =404;
    }
    location / {
        rewrite ^/(.*)$ https://mytest.com/$1 permanent;
    }
}
server {
    listen 443 ssl;
    server_name mytest.com www.mytest.com;
    #ssl_certificate /data/dev/letsencrypt/mytest.com/ssl/chained.pem;
    #ssl_certificate_key /data/dev/letsencrypt/mytest.com/ssl/domain.key;
    
    #location ~ \.(htm|html|gif|jpg|jpeg|png|ico|rar|css|js|zip|txt|flv|swf|doc|ppt|xls|pdf)$ {
    location / {
        root /data/project-frontend/dist;
        index index.html index.htm
        access_log off;
        expires 1d;
    }
    location /myproject/m-test {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
}
> nginx -s reload 
> python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /data/dev/letsencrypt/mytest.com/www/challenges/ > ./signed.crt
(
如果出现ImportError: No module named argparse,
则需要安装python-argparse模块
yum install python-argparse 或者 apt-get install python-argparse
)
> wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
> cat signed.crt intermediate.pem > chained.pem
> wget -O - https://letsencrypt.org/certs/isrgrootx1.pem > root.pem
> vim /etc/nginx/conf.d/mytest.com.conf
(
去掉证书目录#注释
ssl_certificate
ssl_certificate_key
)
> nginx -s reload
> cat intermediate.pem root.pem > full_chained.pem

由于证书签发只有90天，到期后需更新证书脚本
> vim /data/dev/letsencrypt/mytest.com/flush_cert.sh：

#!/bin/bash

domain=mytest.com

cd /data/dev/letsencrypt/$domain/ssl

python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /data/dev/letsencrypt/$domain/www/challenges/ > ./signed.crt  || exit

wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem

cat signed.crt intermediate.pem > chained.pem

nginx -s reload

> chmod 700 /data/dev/letsencrypt/mytest.com/flush_cert.sh

使用crontab定时任务每月自动更新证书：
> crontab -e
0 0 1 * * /data/dev/letsencrypt/mytest.com/flush_cert.sh >/dev/null 2>&1
```

### CentOs6相关

#### SpringBoot服务自启动
```
> ln -s /data/deploy/xxx-mytest-0.01.jar /etc/init.d/myservice

> vim /data/deploy/xxx-mytest-0.01.conf:

LOG_FOLDER=/data/logs
JAVA_HOME="/data/java/jdk1.8.0_181"
JAVA_OPTS=-Xmx300M
RUN_ARGS="--spring.profiles.active=prod"

> chkconfig --add myservice
> chkconfig --list myservice

service myservice start/stop/status

tail -f /data/logs/myservice.log
```






