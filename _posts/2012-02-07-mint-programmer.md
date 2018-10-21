---
layout: post
title: 开发运维相关
category: [Linux-Mint,CentOs6,SpringBoot,Redis,证书,Jenkins,自动打包部署]
comments: false
---

* content
{:toc}

### 安装Redis
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

### 安装Let's Encrypt证书
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

### SpringBoot服务自启动

#### CentOS6使用init.d方式
```
> ln -s /data/deploy/xxx-mytest-0.01.jar /etc/init.d/mytest

> vim /data/deploy/xxx-mytest-0.01.conf:

LOG_FOLDER=/data/logs
JAVA_HOME="/data/java/jdk1.8.0_181"
JAVA_OPTS=-Xmx300M
RUN_ARGS="--spring.profiles.active=prod"

> chkconfig --add mytest
> chkconfig --list mytest

service mytest start/stop/status

tail -f /data/logs/mytest.log
```

#### CentOS7/其它高版本系统使用systemd方式

```
> vim /etc/systemd/system/mytest.service

[Unit]
Description=service-mytest
After=syslog.target

[Service]
ExecStart=/home/jdk1.8.0_181/bin/java -Dspring.profiles.active=prod -Dserver.port=8080 -jar /home/mytest/target/mytest-0.01.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target

> systemctl daemon-reload

启动或停止服务
> systemctl start/stop mytest.service

```

### Jenkins自动打包部署启动脚本

```

> vim deploy.sh

#!/bin/sh

mvn_bin=/data/maven/apache-maven-3.5.4/bin/mvn
project_dir=/data/deploy/mytest
cd $project_dir

git remote prune origin

if [ $# -eq 0 ];then
echo "分支名不能为空！"
git branch -la
exit 0
fi

service mytest stop

branch_name=$1
git pull
git reset --hard origin/$branch_name
git checkout $branch_name
git pull

rm -rf target/*

$mvn_bin clean package -Dmaven.test.skip=true -e

error_code=$?
if [ $error_code -eq 0 ];then
    error_msg="部署成功！"
else
    error_msg="部署失败！" 
fi

service mytest start 
service mytest status

tail -1f /data/logs/mytest.log | sed '/Started MyTestSpringBootApplication in/Q'

echo $error_msg

curl 'https://oapi.dingtalk.com/robot/send?access_token=xxxxxx' \
   -H 'Content-Type: application/json' \
   -d '
  {"msgtype": "text", 
    "text": {
        "content":"'$error_msg'"
     }
  }'

exit 0


Jenkins - 自由风格 - 构建 - Send files or execute commands over SSH
Exec command：sh /data/mytest/deploy.sh master  
```





