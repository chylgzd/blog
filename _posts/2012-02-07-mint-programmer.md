---
layout: post
title: 开发运维相关
category: [Linux-Mint,CentOs6,SpringBoot,Redis,证书,Jenkins,自动打包部署,Nginx]
comments: false
---

* content
{:toc}

### Redis相关

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

#### Redis命令相关

```

> ./src/redis-client

127.0.0.1:6379> SELECT 5

127.0.0.1:6379[5]> get My_CACHE_USER:N1:M1

> ./src/redis-client -n 5 --scan --pattern My_CACHE_USER* | xargs redis-cli -n 5 del

```

### nginx相关
```
# 查看是否支持多个https到一台机器
nginx -V

TLS SNI support enabled


# 强制跳转HTTPS的
server{
    listen 80;
    ...
    location / {
        rewrite ^/(.*)$ https://mytest.com/$1 permanent;
    }
    ...
}
server{
    listen 443;
    ...
}

# 同时支持HTTP和HTTPS的
server{
    listen 80;
    listen 443;
    ...
    location / {
        rewrite ^/(.*)$ https://mytest.com/$1 permanent;
    }
    ...
}

#允许请求head参数里使用下划线
> vim /etc/nginx/nginx.conf
http {
    ...
    underscores_in_headers on;
    ...
}

#禁用ip访问
> vim /etc/nginx/conf.d/default.conf
server{
    listen 80 default;
    server_name _;
    return 403;
}

> vim /etc/nginx/conf.d/mytest.com.conf：

server{
    listen       80;
    server_name test.com www.test.com;
    error_page 502 =502 @502;
    error_page 503 =503 @503;
    error_page 504 =504 @504;
    location @502 {
         default_type application/json;
         return 502 '{"code":"502","msg":"Bad Gateway","success":false,"result":null}';
    }
    location @504 {
         default_type application/json;
         return 504 '{"code":"504","msg":"Gateway Time-out","success":false,"result":null}';
    }
    location @503 {
         default_type application/json;
         return 503 '{"code":"503","msg":"Service Temporarily Unavailable","success":false,"result":null}';
    }
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    #location ~ \.(htm|html|gif|jpg|jpeg|png|ico|rar|css|js|zip|txt|flv|swf|doc|ppt|xls|pdf)$ {
    location / {
        root /data/www/myhtml/dist;
        index index.html index.htm;
        access_log off;
        expires 1d;
    }
    location /test/m-demo {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
    }
}

```

### 安装Let's Encrypt证书

#### 自动方式(推荐)
```
#选择操作系统与http服务，比如ubuntu16-nginx
https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx

$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx 

# centos6-nginx
https://certbot.eff.org/lets-encrypt/centos6-nginx

# 多个域名共用生成到/data/webserver/letsencrypt/live/my-test目录下,公用名是第一个-d的域名
# 生成证书-ubuntu
> certbot --cert-name my-test -d mytest.com -d b.com -d c.com --config-dir /data/webserver/letsencrypt --register-unsafely-without-email --nginx certonly
# 生成证书-centos6，
> ./certbot-auto --cert-name my-test -d mytest.com -d b.com -d c.com --config-dir /data/webserver/letsencrypt --register-unsafely-without-email --nginx certonly

a，多个域名逗号分隔1,2,4

#出现下面提示成功，并且证书地址复制到nginx:
Congratulations! Your certificate and chain have been saved at:

> vim /etc/nginx/conf.d/mytest.com.conf 
server{
    listen 80;
    listen 443 ssl;
    server_name mytest.com www.mytest.com;
    ssl_certificate /etc/letsencrypt/live/mytest.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/mytest.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/mytest.com/chain.pem;
    ssl_session_tickets on;
    if ($scheme = http) {
        rewrite ^/(.*)$ https://mytest.com/$1 permanent;
    }
    location / {
        root /data/www/mytest.com/dist;
        index index.html index.htm;
        access_log off;
        expires 1d;
    }
    location /backend {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    ....
}

> nginx -s reload

#重新生成之前的所有证书，执行dry-run模拟测试是否成功：
> certbot renew --config-dir /data/webserver/letsencrypt --dry-run
# 成功后加入真正的定时任务执行
> certbot renew --config-dir /data/webserver/letsencrypt 
> vim /data/webserver/letsencrypt/flush_certbot.sh
#!/bin/bash
/data/webserver/letsencrypt/certbot-auto renew --config-dir /data/webserver/letsencrypt 
nginx -s reload

> crontab -e
0 0 1 * * /data/webserver/letsencrypt/flush_certbot.sh >/dev/null 2>&1
```

#### 手动方式
```
参考：
https://imququ.com/post/letsencrypt-certificate.html
https://imququ.com/post/my-nginx-conf.html
https://github.com/diafygi/acme-tiny
https://www.liaosam.com/use-cron-service-and-certbot-for-renewal-of-letsencrypt-ssl-certificates.html
https://www.vpser.net/build/letsencrypt-certbot.html

> mkdir -p /data/dev/letsencrypt/mytest.com/ssl
> cd /data/dev/letsencrypt/mytest.com/ssl
#创建私钥
> openssl genrsa 4096 > account.key
#创建 CSR 文件
> openssl ecparam -genkey -name secp384r1 | openssl ec -out domain.key
> openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:mytest.com,DNS:www.mytest.com")) > domain.csr
(
如果出现cat: /etc/ssl/openssl.cnf: 没有那个文件或目录,
则先find / -name openssl.cnf，再替换命令里的openssl.cnf文件目录（centos:/etc/pki/tls/openssl.cnf）
)
# acme-tiny
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
        #rewrite ^(.*)$  https://$host$1 permanent;(当server_name为多个域名的时候)
    }
}
server {
    #listen 443 default ssl;
    listen 443 ssl;
    server_name mytest.com www.mytest.com;
    ssl on;
    #ssl_certificate /data/dev/letsencrypt/mytest.com/ssl/chained.pem;
    #ssl_certificate_key /data/dev/letsencrypt/mytest.com/ssl/domain.key;
    ssl_session_tickets on;
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
# 获取签名证书
> python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /data/dev/letsencrypt/mytest.com/www/challenges/ > ./signed.crt
(
如果出现ImportError: No module named argparse,
则需要安装python-argparse模块
yum install python-argparse 或者 apt-get install python-argparse
)
# 安装证书
> wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
> cat signed.crt intermediate.pem > chained.pem
# 为了后续启用OCSP Stapling把根证书和中间证书合在一起
> wget -O - https://letsencrypt.org/certs/isrgrootx1.pem > root.pem
> cat intermediate.pem root.pem > full_chained.pem

> vim /etc/nginx/conf.d/mytest.com.conf
(
去掉证书目录#注释
ssl_certificate
ssl_certificate_key
)
> nginx -s reload

由于证书签发只有90天，到期后需更新证书脚本(注意：每个域名每个证书每周只能签发限制5次，不要多次刷新证书,否则会出现ERR_SSL_PROTOCOL_ERROR)
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

#### log4j2_prod.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
	<properties>
			<property name="project-name">mytest-prod</property>
			<property name="logfile-dir">/data/logs/mytest-prod/</property>
			<property name="console-pattern">%d{yyyyMMdd HH:mm:ss.SSS} [%level:%thread] %logger{36} - %msg%n</property>
			<property name="logfile-pattern">%d{yyyyMMdd HH:mm:ss.SSS} [%level:%thread] %logger - %msg%n</property>
	</properties>
	<Appenders>
		<Console name="stdout" target="SYSTEM_OUT" follow="true">
			<PatternLayout pattern="${console-pattern}" />
		</Console>
		<RollingFile name="${project-name}"  fileName="${logfile-dir}${project-name}.log" filePattern="${logfile-dir}${project-name}.%d{yyyyMMdd}-%i.log.gz" bufferedIO="true" immediateFlush="true">
			<PatternLayout pattern="${logfile-pattern}" />
			 <Policies>
                    <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                    <SizeBasedTriggeringPolicy size="1 GB" />
            </Policies>
		</RollingFile>
		<RollingFile name="${project-name}-info" fileName="${logfile-dir}${project-name}-info.log" filePattern="${logfile-dir}${project-name}-info.%d{yyyyMMdd}-%i.log.gz" bufferedIO="true" immediateFlush="false">
			<PatternLayout pattern="${logfile-pattern}" />
			<filters>
					<ThresholdFilter level="FATAL" onMatch="DENY" onMismatch="NEUTRAL"/>
					<ThresholdFilter level="ERROR" onMatch="DENY" onMismatch="NEUTRAL"/>
					<ThresholdFilter level="WARN" onMatch="DENY" onMismatch="NEUTRAL"/>
					<ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY"/>
	        </filters>
			 <Policies>
                    <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                    <SizeBasedTriggeringPolicy size="1 GB" />
            </Policies>
		</RollingFile>
		<RollingFile name="${project-name}-error" fileName="${logfile-dir}${project-name}-error.log" filePattern="${logfile-dir}${project-name}-error.%d{yyyyMMdd}-%i.log.gz" bufferedIO="true" immediateFlush="false">
			<PatternLayout pattern="${logfile-pattern}" />
			<filters>
			     <ThresholdFilter level="ERROR" onMatch="ACCEPT" onMismatch="DENY"/>
	        </filters>
			<Policies>
                    <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                    <SizeBasedTriggeringPolicy size="1 GB" />
            </Policies>
		</RollingFile>
	</Appenders>
	<Loggers>
		<Root level="info">
			<AppenderRef ref="stdout" />
			<AppenderRef ref="${project-name}" />
			<AppenderRef ref="${project-name}-info" />
			<AppenderRef ref="${project-name}-error" />
		</Root>
	</Loggers>
</Configuration>
```

#### CentOS6使用init.d方式
```
> vim   /data/deploy/mytest/target/config/application-prod.properties

server.port=8080
spring.devtools.restart.enabled=false
logging.config=log4j2_prod.xml

> vim   /data/deploy/mytest/target/log4j2_prod.xml

> ln -s /data/deploy/mytest/target/xxx-mytest-0.01.jar /etc/init.d/mytest

> vim   /data/deploy/mytest/target/xxx-mytest-0.01.conf:

LOG_FOLDER=/data/logs
JAVA_HOME="/data/java/jdk1.8.0_181"
JAVA_OPTS="-Xmx500M -Dfile.encoding=UTF8 -Dsun.jnu.encoding=UTF8"
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
project_config_dir=/data/deploy/config
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

cp $project_config_dir/log4j2_prod.xml $project_dir/target/
cp $project_config_dir/mytest-0.01.conf $project_dir/target/
mkdir -p $project_dir/target/config
cp $project_config_dir/application-prod.properties $project_dir/target/config/

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





