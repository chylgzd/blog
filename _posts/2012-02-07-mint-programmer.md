---
layout: post
title: 开发运维相关
category: [Linux-Mint,CentOs6,SpringBoot,Redis,证书,Jenkins,自动打包部署,Nginx,Jetty,kubectl]
comments: false
---

* content
{:toc}

### Redis相关

#### 安装Redis
```
> tar -zxf redis-5.0.5.tar.gz -C /data/redis/
> cd  /data/redis/redis-5.0.5/
> make
> vim redis.conf :

bind 0.0.0.0（远程机器访问需要）
port 6377
protected-mode no（是否保护模式,如果需要远程机器访问则no）
daemonize yes（是否后台进程启动）
requirepass pwd123456（设置redis访问密码）

> ./src/redis-server ./redis.conf
> ./src/redis-client -h 127.0.0.1 -p 6377
127.0.0.1:6377> KEYS *
(error) NOAUTH Authentication required.
127.0.0.1:6377> AUTH pwd123456
ok

```

#### Redis自启动服务

```
centos下：

> cp /data/redis/redis-5.0.5/utils/redis_init_script /etc/init.d/redis
> cd /etc/init.d
> vim redis (只改动几个属性配置,其它不变，以后redis升级只需要修改REDIS_DIR即可)
...
REDIS_DIR=/data/redis/redis-5.0.5
REDISPORT=6379
EXEC=$REDIS_DIR/src/redis-server
CLIEXEC=$REDIS_DIR/src/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="$REDIS_DIR/redis.conf"
...

> chmod +x redis (默认文件复制过去有可执行权限可以不用执行)
> chkconfig redis on
> service redis start
> service redis stop
> chkconfig --list | grep redis 查看redis服务是否有自启动开关

```

#### Redis命令相关

```

> ./src/redis-client

# 查询server信息
127.0.0.1:6379> INFO

# 查询client连接列表
127.0.0.1:6379> CLIENT LIST

# 选择db
127.0.0.1:6379> SELECT 5

# 查看db5下所有键
127.0.0.1:6379[5]> KEYS

# 获取KEY值
127.0.0.1:6379[5]> GET "My_CACHE_USER:N1:M1"

# 查看失效时间（秒）
127.0.0.1:6379[5]> TTL "My_CACHE_USER:N1:M1"

# 删除key
127.0.0.1:6379[5]> DEL "My_CACHE_USER:N1:M1"

> ./src/redis-client -n 5 --scan --pattern My_CACHE_USER* | xargs redis-cli -n 5 del

如果有密码:
127.0.0.1:6379> SELECT 5
127.0.0.1:6379> AUTH yourpasswd

清除当前select的DB 5 下的所有数据：
127.0.0.1:6379> SELECT 5
127.0.0.1:6379> FLUSHDB

清除redis所有DB的数据：
127.0.0.1:6379> FLUSHALL

关闭所有客户端停止服务端
127.0.0.1:6379> SHUTDOWN

```

### nginx相关
```
自动安装(CentOS推荐)：
http://nginx.org/en/linux_packages.html#RHEL-CentOS

开机自启动（centos7）
> systemctl enable nginx.service
> systemctl status nginx

# 查看是否支持多个https到一台机器
nginx -V

TLS SNI support enabled

# www域名跳转到非www域名
server {
    listen 80;
    server_name www.test.com;
    return 301 https://test.com$request_uri;
}
server {
    listen 80;
    listen 443 ssl;
    server_name test.com;
    ssl_certificate /data/webserver/letsencrypt/certbotcfg/live/test.com/fullchain.pem;
    ssl_certificate_key /data/webserver/letsencrypt/certbotcfg/live/test.com/privkey.pem;
    ssl_trusted_certificate /data/webserver/letsencrypt/certbotcfg/live/test.com/chain.pem;
    ssl_session_tickets on;
    location ^~ /.well-known/acme-challenge/ {
        alias /data/webserver/letsencrypt/test.com/www/challenges/;
        try_files $uri =404;
    }
    if ($scheme = http) {
        rewrite ^/(.*)$ https://test.com/$1 permanent;
    }
    location / {
        root /data/nginx/html;
        index index.html index.htm;
        if_modified_since before;
        etag off;
        access_log off;
        expires 1d;
    }
}

# 强制跳转HTTPS的（301永久：permanent,302临时：redirect）
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

# 允许最大文件上传1M
> vim /etc/nginx/nginx.conf
http {
    ...
    client_max_body_size 1m;
    ...
}
> vim /etc/nginx/conf.d/www.mytest.com.conf
server {
    listen 80;
    listen 443 ssl;
    server_name www.mytest.com;
    error_page 413 =200 @413;
    location @413 {
         default_type application/json;
         return 413 '{"code":"EF0001","msg":"文件超过1M"}';
    }
    location / {
        root /data/www/dist;
        index index.html index.htm;
        if_modified_since before;
        access_log off;
        etag off;
        expires 1h;
    }
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
    #listen       443 ssl;
    server_name test.com www.test.com;
    error_page 502 =502 @502;
    error_page 503 =503 @503;
    error_page 504 =504 @504;
    #if ($scheme = http) {
    #    rewrite ^/(.*)$ https://www.test.com/$1 permanent;
    #}
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
    #location ~ \.(htm|html|gif|jpg|jpeg|png|ico|rar|css|js|zip|txt|flv|swf|doc|ppt|xls|pdf)$ {}
    location / {
        root /data/www/myhtml/dist;
        index index.html index.htm;
        if_modified_since before;
        access_log off;
        etag off;
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

#### 自动方式(推荐,最好先手动配置完后再自动)
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

# 多个域名共用生成到/data/webserver/letsencrypt/live/my-test目录下,公用名是第一个-d的域名(新加域名从这里开始即可)
# 生成证书-ubuntu 
> certbot --cert-name my-test -d mytest.com -d b.com -d c.com --config-dir /data/webserver/letsencrypt --register-unsafely-without-email --nginx certonly
# 生成证书-centos6，
> ./certbot-auto --cert-name my-test -d mytest.com -d b.com -d c.com --config-dir /data/webserver/letsencrypt --register-unsafely-without-email --nginx certonly

# 生成证书-centos7
> certbot --cert-name my-test -d mytest.com -d b.com -d c.com --config-dir /data/webserver/letsencrypt/certbotcfg --register-unsafely-without-email --nginx certonly
(
    如果出现ImportError: No module named 'requests.packages.urllib3'
    尝试运行> pip install --upgrade --force-reinstall 'requests==2.6.0' urllib3
)

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
        if_modified_since before;
        etag off;
        access_log off;
        expires 1d;
    }
    location /backend {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
        access_log off;
    }
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    ....
}

> nginx -s reload

#重新生成之前的所有证书，执行dry-run模拟测试是否成功：
> (ubuntu) certbot renew --config-dir /data/webserver/letsencrypt --dry-run
> (centos6) ./certbot-auto renew --config-dir /data/webserver/letsencrypt --dry-run
> (centos7) certbot renew --config-dir /data/webserver/letsencrypt/certbotcfg --dry-run
(
    如果出现'ascii' codec can't decode byte 0xe6 in position...
    则先执行检查目录下是否有非ASCII字符,去掉或修改(中文转ASCII:https://tool.oschina.net/encode?type=3):
    > grep -r -P '[^\x00-\x7f]' /data/webserver/letsencrypt /etc/nginx
)
# 成功后加入真正的定时任务执行
> certbot renew --config-dir /data/webserver/letsencrypt 
> vim /data/webserver/letsencrypt/flush_certbot.sh
#!/bin/bash
(ubuntu：)
certbot renew --config-dir /data/webserver/letsencrypt
(centos6：)
/data/webserver/letsencrypt/certbot-auto renew --config-dir /data/webserver/letsencrypt 
(centos7：)
certbot renew --config-dir /data/webserver/letsencrypt/certbotcfg
nginx -s reload

> crontab -e (无效则使用vim /etc/crontab)
0 0 1 * * root /data/webserver/letsencrypt/flush_certbot.sh >/dev/null 2>&1
0 23 15 * *(每月15号晚上23点0分执行)
```

#### 手动方式1
```
> vim /etc/nginx/conf.d/my.test.com.conf

server {
    listen 80;
    #listen 443 ssl;
    server_name my.test.com;
    #ssl_certificate /data/webtest/letsencrypt/my.test.com/ssl/chained.pem;
    #ssl_certificate_key /data/webtest/letsencrypt/my.test.com/ssl/domain.key;
    ssl_session_tickets on;
    location ^~ /.well-known/acme-challenge/ {
        alias /data/webtest/letsencrypt/my.test.com/www/challenges/;
        try_files $uri =404;
    }
    #if ($scheme = http) {
    #   rewrite ^/(.*)$ https://my.test.com/$1 permanent;
    #}
    location / {
        root /data/www/static;
	    index index.html index.htm
        if_modified_since before;
        etag off;
        access_log off;
        expires 1d;
    }
    location /backend/m-test {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
        access_log off;
    }
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
}

> nginx -s reload

> vim /usr/local/aegis/globalcfg/letsencrypt.sh (使用 ./letsencrypt.sh my.test.com)

#!/bin/bash
# /etc/ssl/openssl.cnf(ubuntu) or /etc/pki/tls/openssl.cnf (centos)

if [ $# -eq 0 ];then
echo "domain is null!!"
exit 0
fi

domain=$1
letsencrypt_dir=/data/webtest/letsencrypt
openssl_cnf=/etc/pki/tls/openssl.cnf

ssl_dir=$letsencrypt_dir/$domain/ssl
challenges_dir=$letsencrypt_dir/$domain/www/challenges/

mkdir -p $ssl_dir
mkdir -p $challenges_dir

cd $ssl_dir

openssl genrsa 4096 > account.key
openssl ecparam -genkey -name secp384r1 | openssl ec -out domain.key
openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config <(cat $openssl_cnf <(printf "[SAN]\nsubjectAltName=DNS:$domain")) > domain.csr

wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py

echo $$challenges_dir
# vim nginx config
# location ^~ /.well-known/acme-challenge/ {
#    alias $challenges_dir;
#    try_files $uri =404;
# }

python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir $challenges_dir > ./signed.crt

wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
cat signed.crt intermediate.pem > chained.pem
wget -O - https://letsencrypt.org/certs/isrgrootx1.pem > root.pem
cat intermediate.pem root.pem > full_chained.pem

echo ssl_certificate $ssl_dir/chained.pem;
echo ssl_certificate_key $ssl_dir/domain.key;
echo ssl_session_tickets on;

# vim nginx config
#ssl_certificate $ssl_dir/chained.pem;
#ssl_certificate_key $ssl_dir/domain.key;
#ssl_session_tickets on;

> vim /etc/nginx/conf.d/my.test.com.conf 去掉ssl注释#

> nginx -s reload

```

#### 手动方式2
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
    #if ($scheme = http) {
    #    rewrite ^/(.*)$ https://pay.dmpzg.com/$1 permanent;
    #}
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
> crontab -e (无效则使用vim /etc/crontab)
0 0 1 * * root /data/dev/letsencrypt/mytest.com/flush_cert.sh >/dev/null 2>&1
```

### SpringBoot 相关问题
```
使用undertow时文件上传必须设置如下值，否则parse multipart error
spring.servlet.multipart.location=/data/tmp
```

### SpringBoot 单元测试相关

#### 模式1-完全不依赖src/main,独立依赖test目录
```
test目录的包路径需要完全脱离src/main/java:
如 src/main/java下包为com.github.mydemo
则 src/test/java下包为test.github.mydemo或任意与主包不同的包路径即可
这种模式下是完全脱离src/main里的自动注解,要么需要独立写组件，
要么使用@Import加载main里组件且不能过度复杂依赖,否则需要@Import很多依赖类,
好处在于是独立的环境加载启动测试快,如果模块依赖比较复杂建议使用第二个模式

TestSpringbootMain.java:

package test.github.mydemo;
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringbootTestApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class TestSpringbootMain{
}

SpringbootTestApplication.java:

package test.github.mydemo;
@SpringBootApplication
public class SpringbootTestApplication {
}


```

#### 模式2-依赖src/main,但修改部分代码用于替换测试

```
test目录的包路径需要保持与src/main/java一致:
如 src/main/java下包为com.github.mydemo
则 src/test/java下包也必须以com.github.mydemo开始
这种模式下是依赖src/main/java里组件的，适合复杂依赖场景的单元测试,避免模式1下重复写太多依赖

TestSpringbootMain.java:

package com.github.mydemo;
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringbootTestApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class TestSpringbootMain{
}

由于在一个包路径下会启动主@SpringBootApplication文件,此时需要排除,避免双重配置
SpringbootTestApplication.java:

package com.github.mydemo;
@SpringBootApplication
@ComponentScan(excludeFilters = {
		#这里可以排除不需要进入单元测试的组件,比如定时任务或其它不需要测试的组件
		@Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {MainApplication.class,XxxComponent.class}),
})
#其它这里可以和MainApplication里保持一致,比如启用事务,JPA审计之类的测试所必须依赖的注解
//@EnableTransactionManagement
//@EnableJpaAuditing
public class SpringbootTestApplication {
}
```

#### 模式3-不依赖任何自动配置,不论包目录结构,按需自行加载

```
TestSpringbootMain.java:

@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringbootTestApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class TestSpringbootMain{
}

SpringbootTestApplication.java:

@SpringBootConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
#按需手动添加XxxxAutoConfiguration.class
@Import({ServletWebServerFactoryAutoConfiguration.class})#web必须
public class SpringbootTestApplication {
}
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
        <RollingFile name="${project-name}-debug" fileName="${logfile-dir}${project-name}-debug.log" filePattern="${logfile-dir}${project-name}-debug.%d{yyyyMMdd}-%i.log.gz" bufferedIO="true" immediateFlush="false">
			<PatternLayout pattern="${logfile-pattern}" />
			<filters>
					<ThresholdFilter level="FATAL" onMatch="DENY" onMismatch="NEUTRAL"/>
					<ThresholdFilter level="ERROR" onMatch="DENY" onMismatch="NEUTRAL"/>
					<ThresholdFilter level="WARN" onMatch="DENY" onMismatch="NEUTRAL"/>
					<ThresholdFilter level="INFO" onMatch="DENY" onMismatch="NEUTRAL"/>
					<ThresholdFilter level="DEBUG" onMatch="ACCEPT" onMismatch="DENY"/>
	        </filters>
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
        <!--设置druid日志level为debug-->
	    <Logger name="druid" level="debug" additivity="false">
	        <AppenderRef ref="stdout"/>
	        <AppenderRef ref="${project-name}"/>
	    </Logger>
		<Root level="info">
			<AppenderRef ref="stdout" />
			<AppenderRef ref="${project-name}" />
			<AppenderRef ref="${project-name}-info" />
			<AppenderRef ref="${project-name}-error" />
		</Root>
	</Loggers>
</Configuration>
```

#### 通用nohup方式
```
部署目录结构：
deploy/xxx/spring-boot-project
deploy/xxx/run
deploy
    xxx
        run
            config
                application-prod.properties
            spring-boot-project-0.0.1.conf
            spring-boot-project-0.0.1.jar
            log4j2_test.xml
            log4j2_prod.xml
            start.sh
            stop.sh
        spring-boot-project
            src
> cd xxx/spring-boot-project && mvn clean package && mv target/spring-boot-project-0.0.1.jar run/

> vim run/spring-boot-project-0.0.1.conf
LOG_FOLDER=/data/logs
JAVA_HOME="/usr/local/java/jdk8202"
JAVA_OPTS="-Xms250m -Xmx500m -Xmn256m -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m -Dfile.encoding=UTF8 -Dsun.jnu.encoding=UTF8"
RUN_ARGS="--spring.profiles.active=test --server.port=8080"

> run/log4j2_test.xml
<property name="logfile-dir">/data/logs/xxx/</property>

> vim run/start.sh
#!/bin/bash
cd deploy/xxx/run
nohup ./spring-boot-project-0.0.1.jar >/dev/null 2>&1 &

tail -1f /data/logs/my-boot-prod/my-boot.log | sed '/Undertow started on port/Q'

exit 0

> vim run/stop.sh
#!/bin/bash
cd deploy/xxx/run
kill `pgrep -f spring-boot-project-0.0.1.jar` 2>/dev/null
ps -ef |grep spring-boot-project-0.0.1.jar | grep -v grep |awk '{print $2}'|xargs kill -9 1>/dev/null 2>&1 
exit 0

注意：如果是自定义config/application-prod.properties配置，则需要改变日志路径：
logging.config=classpath:log4j2.xml -> logging.config=log4j2.xml

> vim deploy/xxx/deploy.sh
#!/bin/sh

run_dir=deploy/xxx/run
project_dir=deploy/xxx/spring-boot-project
jar_name=spring-boot-project-0.0.1.jar
cd $project_dir

if [ $# -eq 0 ];then
echo "分支名不能为空！"
git branch -la
exit 0
fi

$run_dir/stop.sh

branch_name=test
git pull
git reset --hard origin/$branch_name
git checkout $branch_name
git pull

rm -rf target/*
rm -rf $run_dir/*.jar

mvn clean package -Dmaven.test.skip=true -e

error_code=$?
if [ $error_code -eq 0 ];then
    error_msg="jenkins-deploy-backend-success！"
else
    error_msg="jenkins-deploy-backend-fail！" 
fi

cp -rf $project_dir/target/$jar_name $run_dir/

$run_dir/start.sh

echo $error_msg

curl 'https://oapi.dingtalk.com/robot/send?access_token=xxx' \
   -H 'Content-Type: application/json' \
   -d '
  {"msgtype": "text", 
    "text": {
        "content":"'$error_msg'"
     }
  }'

exit 0

> vim deploy/xxx/frontend/deploy.sh
project_dir=deploy/xxx/frontend/my-frontend
branch_name=test

cd $project_dir
git pull
git reset --hard origin/$branch_name
git checkout $branch_name
git pull

rm -rf dist/*
npm install
npm run build:test

error_code=$?
if [ $error_code -eq 0 ];then
    error_msg="jenkins-deploy-frontend-success！"
else
    error_msg="jenkins-deploy-frontend-fail！" 
fi

echo $error_msg

curl 'https://oapi.dingtalk.com/robot/send?access_token=xxx' \
   -H 'Content-Type: application/json' \
   -d '
  {"msgtype": "text", 
    "text": {
        "content":"'$error_msg'"
     }
  }'

exit 0

```

#### CentOS6使用init.d方式
```
> vim /data/deploy/mytest/target/config/application-prod.properties

server.port=8080
spring.devtools.restart.enabled=false
logging.config=log4j2_prod.xml

> vim   /data/deploy/mytest/target/log4j2_prod.xml

> ln -s /data/deploy/mytest/target/xxx-mytest-0.01.jar /etc/init.d/mytest

> vim   /data/deploy/mytest/target/xxx-mytest-0.01.conf:

LOG_FOLDER=/data/logs
JAVA_HOME="/data/java/jdk1.8.0_181"
JAVA_OPTS="-Xms250m -Xmx500m -Xmn256m -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m -Dfile.encoding=UTF8 -Dsun.jnu.encoding=UTF8"
RUN_ARGS="--spring.profiles.active=prod --server.port=8080"

> chkconfig --del mytest
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

### Jetty相关
```
下载Jetty9

https://www.eclipse.org/jetty/download.html

解压jetty到jetty9，部署war文件到jetty9/webapps目录，没有ROOT可以手动创建(demo-base目录可以删除)

修改配置文件：

> vim ./start.ini
--module=ext
--module=server
--module=jsp
--module=resources
--module=deploy
--module=jstl
#--module=websocket(没有用到websocket就禁用)
--module=http
jetty.http.port=8080
jetty.http.connectTimeout=35000

启动/停止/重启/调试 jetty：

> ./bin/jetty.sh start
> ./bin/jetty.sh stop
> ./bin/jetty.sh restart
> ./bin/jetty.sh run (相当于tomcat catalina.sh run)

同一服务器跑多个jetty：

1. 修改配置文件：

> vim ./start.ini
jetty.http.port=8080(修改这里的端口)

2. 重命名启动脚本：

> mv ./bin/jetty.sh ./bin/jetty01.sh

以服务方式启动：

> ln -s /jetty9/bin/jetty01.sh /etc/init.d/myJettyService01
> vim /jetty9/bin/jetty01.sh
#!/usr/bin/env bash
JETTY_HOME=/jetty9

> service myJettyService01 check
(检查服务启动参数如果正常则会打印相关jetty启动参数)

> service myJettyService01 start

> chkconfig myJettyService01 on
```

### aliyun相关

#### oss
```
oss权限分离：新建用户子账户（不要添加策略权限），新建Bucket设置选择子账户读写相关权限即可
测试代码TestAliYunOSS.java：

import java.io.File;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import com.aliyun.oss.ClientConfiguration;
import com.aliyun.oss.OSSClient;
import com.aliyun.oss.common.auth.CredentialsProvider;
import com.aliyun.oss.common.auth.DefaultCredentialProvider;
import com.aliyun.oss.model.BucketReferer;
import com.aliyun.oss.model.PutObjectResult;

public class TestAliYunOSS {
	private String imgURL = "https://mybucket.oss-cn-shanghai.aliyuncs.com";
	private String endpoint = "http://oss-cn-shanghai.aliyuncs.com";
	private String accessKeyId = "xxxx";
	private String accessKeySecret = "xxxxx";
	private String bucketName = "mybucket";

	private OSSClient ossClient = null;

	// 设置防盗链白名单
	@Test
	public void testAddReferer() throws Exception {
		try {
			List<String> refererList = new ArrayList<String>();
			refererList.add("https://*.console.aliyun.com");
			refererList.add("http://*.mogkh.com");
			refererList.add("https://*.mogkh.com");
			BucketReferer br = new BucketReferer(true, refererList);
			ossClient.setBucketReferer(bucketName, br);
		} catch (Exception e) {
			e.printStackTrace();
			throw e;
		}
	}

	// 获取防盗链白名单
	@Test
	public void testGetReferer() throws Exception {
		try {
			BucketReferer br = ossClient.getBucketReferer(bucketName);
			List<String> refererList = br.getRefererList();
			for (String referer : refererList) {
				System.out.println(referer);
			}
		} catch (Exception e) {
			e.printStackTrace();
			throw e;
		}
	}

	@Before
	public void init() {
		try {
			System.out.println("init...");
			CredentialsProvider credsProvider = new DefaultCredentialProvider(accessKeyId, accessKeySecret);
			ClientConfiguration config = new ClientConfiguration();
			config.setConnectionTimeout(5 * 1000);// 5秒
			config.setConnectionRequestTimeout(5 * 1000);// 5秒
			config.setSocketTimeout(15 * 1000);// 15秒
			config.setRequestTimeout(1 * 60 * 1000);// 1分钟
			config.setRequestTimeoutEnabled(true);
			this.ossClient = new OSSClient(endpoint, credsProvider, config);
		} catch (Exception e) {
			e.printStackTrace();
			throw e;
		}
	}

	@After
	public void stop() {
		try {
			if (this.ossClient != null) {
				System.out.println("stop...");
				this.ossClient.shutdown();
			}
		} catch (Exception e) {
			e.printStackTrace();
			throw e;
		}
	}

	protected String putImageFile(String uId, String fileName, File localFile, InputStream inputStream) {
		String key = "userdir/" + uId + "/" + fileName;
		PutObjectResult putObjectResult = null;
		if (localFile != null) {
			putObjectResult = ossClient.putObject(bucketName, key, localFile);
		} else if (inputStream != null) {
			putObjectResult = ossClient.putObject(bucketName, key, inputStream);
		}
		if (putObjectResult == null) {
			return null;
		}
		String etag = putObjectResult.getETag();
		if (etag == null || etag.isEmpty()) {
			return null;
		}
		return imgURL + "/" + key + "?t=" + etag;
	}
}
```


### springboot2-docker-aliyunserverless

#### boot2-parent-pom/pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.5.RELEASE</version>
	</parent>

	<groupId>com.kehong</groupId>
	<artifactId>boot2-parent-pom</artifactId>
	<name>boot2-parent-pom</name>
	<packaging>pom</packaging>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<!-- 系统变量 -->
		<java.version>1.8</java.version>
		<java_source_version>1.8</java_source_version>
		<java_target_version>1.8</java_target_version>
		<file_encoding>UTF-8</file_encoding>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>

		<!-- 工程变量 -->
		<skipTests>true</skipTests>

		<!-- jar包变量 -->
		<mybatis-plus-boot-starter.version>3.1.1</mybatis-plus-boot-starter.version>
		<HikariCP.version>3.3.1</HikariCP.version>
		<fastjson.version>1.2.58</fastjson.version>
		<mysql.version>8.0.16</mysql.version>
		<ojdbc6.version>11.2.0.4.0</ojdbc6.version>
		<velocity-engine-core.version>2.0</velocity-engine-core.version>
		<hutool-all.version>4.5.10</hutool-all.version>
		<easypoi.version>3.0.1</easypoi.version>
		<swagger.version>2.8.0</swagger.version>

		<!-- 插件变量 -->
		<maven-compiler-plugin.version>3.8.1</maven-compiler-plugin.version>
		<!-- docker -->
		<dockerfile-maven-plugin.version>1.4.10</dockerfile-maven-plugin.version>
		<docker.image.prefix>mydocker</docker.image.prefix>
	</properties>

	<!-- 统一包定义 -->
	<dependencyManagement>
		<dependencies>
			<!-- spring-cloud -->
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Edgware.SR3</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>

			<!-- mybatis-plus -->
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
				<version>${mysql.version}</version>
			</dependency>
			<dependency>
				<groupId>com.oracle.jdbc</groupId>
				<artifactId>ojdbc6</artifactId>
				<version>${ojdbc6.version}</version>
			</dependency>
			<dependency>
				<groupId>com.zaxxer</groupId>
				<artifactId>HikariCP</artifactId>
				<version>${HikariCP.version}</version>
			</dependency>
			<dependency>
				<groupId>com.baomidou</groupId>
				<artifactId>mybatis-plus-boot-starter</artifactId>
				<version>${mybatis-plus-boot-starter.version}</version>
				<exclusions>
					<exclusion>
						<artifactId>tomcat-jdbc</artifactId>
						<groupId>org.apache.tomcat</groupId>
					</exclusion>
				</exclusions>
			</dependency>
			<dependency>
				<groupId>com.baomidou</groupId>
				<artifactId>mybatis-plus-generator</artifactId>
				<version>${mybatis-plus-boot-starter.version}</version>
				<scope>test</scope>
			</dependency>

			<!-- fastjson -->
			<dependency>
				<groupId>com.alibaba</groupId>
				<artifactId>fastjson</artifactId>
				<version>${fastjson.version}</version>
			</dependency>

			<!-- velocity -->
			<dependency>
				<groupId>org.apache.velocity</groupId>
				<artifactId>velocity-engine-core</artifactId>
				<version>${velocity-engine-core.version}</version>
			</dependency>

			<!-- hutool -->
			<dependency>
				<groupId>cn.hutool</groupId>
				<artifactId>hutool-all</artifactId>
				<version>${hutool-all.version}</version>
			</dependency>

			<!-- easypoi -->
			<dependency>
				<groupId>cn.afterturn</groupId>
				<artifactId>easypoi-base</artifactId>
				<version>${easypoi.version}</version>
			</dependency>
			<dependency>
				<groupId>cn.afterturn</groupId>
				<artifactId>easypoi-annotation</artifactId>
				<version>${easypoi.version}</version>
			</dependency>

			<!-- swagger -->
			<dependency>
				<groupId>io.springfox</groupId>
				<artifactId>springfox-swagger-ui</artifactId>
				<version>${swagger.version}</version>
			</dependency>
			<dependency>
				<groupId>io.springfox</groupId>
				<artifactId>springfox-swagger2</artifactId>
				<version>${swagger.version}</version>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<!-- 公用包配置 -->
	<dependencies>
		<!-- log4j2 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-log4j2</artifactId>
		</dependency>
	</dependencies>

	<build>
		<!-- 统一构建插件定义 -->
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-maven-plugin</artifactId>
					<configuration>
						<executable>true</executable>
					</configuration>
				</plugin>
				<plugin>
					<groupId>com.spotify</groupId>
					<artifactId>dockerfile-maven-plugin</artifactId>
					<version>${dockerfile-maven-plugin.version}</version>
					<executions>
						<execution>
							<id>default</id>
							<goals>
								<goal>build</goal>
								<goal>push</goal>
							</goals>
						</execution>
					</executions>
					<configuration>
						<dockerInfoDirectory>${project.basedir}/docker</dockerInfoDirectory>
						<repository>${docker.image.prefix}/${project.artifactId}</repository>
						<tag>${project.version}</tag>
						<buildArgs>
							<MY_JAR_FILE>${project.build.finalName}.jar</MY_JAR_FILE>
						</buildArgs>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>${maven-compiler-plugin.version}</version><!--$NO-MVN-MAN-VER$ -->
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
					<encoding>${project.build.sourceEncoding}</encoding>
				</configuration>
				<executions>
					<execution>
						<id>default-testCompile</id>
						<phase>test-compile</phase>
						<goals>
							<goal>testCompile</goal>
						</goals>
						<configuration>
							<skip>${skipTests}</skip>
						</configuration>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

#### Springboot2子项目根目录/Dockerfile

```sh
FROM openjdk:8-jdk-alpine

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' >/etc/timezone

ARG MY_JAR_FILE
ADD target/${MY_JAR_FILE} /usr/local/myapp/app.jar
ENTRYPOINT ["/usr/bin/java","-Djava.security.egd=file:/dev/./urandom","-Dspring.profiles.active=test","-jar","/usr/local/myapp/app.jar"]

```

#### 打包Springboot2子项目并发布到aliyun镜像

```
> mvn clean package dockerfile:build
本地运行
> docker run --name 容器名称 -p 8081:8080 -d 镜像:标签

> docker login --username=用户名 registry.cn-hangzhou.aliyuncs.com 
> docker tag [本地容器镜像ID] registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名:版本号
> docker push registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名:版本号
```

#### 阿里云Kubernetes集群使用Springboot镜像并运行

```
1.容器服务 -> 集群 -> 创建Serverless Kubernetes（公测）

2.从https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG.md下载最新的kubectl客户端

3.复制$HOME/.kube/config安装和设置kubectl客户端ln -s /usr/local/bin/kubectl指向下载kubectl文件

4.应用 - 无状态 - 使用镜像创建 - 选择私有镜像(打包上传的springboot)

5.路由服务（访问方式以供外网访问,内部访问方便容器地址使用内网） - 负载均衡公网访问 - 容器端口（springboot启动端口） - 服务端口(外网开放端口)

6.完成创建，外网访问即可

7. 复制更多查看yaml文件myapp.yaml，把多余自动创建的时间等状态信息去掉即可成为自动打包部署的yaml模板文件。
```

#### 自动打包部署到阿里云脚本(依赖于myapp.yaml)

```sh
deploy_dir=/data/dev/deploy/myapp
project_dir=$deploy_dir/myapp
prod_cfgdir=$deploy_dir/prod_cfg

cd $project_dir

docker rmi registry-vpc.cn-huhehaote.aliyuncs.com/mydocker/myapp:0.01
#docker rmi registry-vpc.cn-huhehaote.aliyuncs.com/mydocker/myapp:0.01-prod
docker rmi mydocker/myapp:0.01
docker images|sed "1 d"|grep "<none>" |awk '{print $3}' |xargs docker rmi
git remote prune origin

if [ $# -eq 0 ];then
echo "分支名不能为空!"
git branch -la
exit 0
fi

branch_name=test
#branch_name=master
git pull
git reset --hard origin/$branch_name
git checkout $branch_name
git pull

#if prod
#(
#sed -i "s/spring.profiles.active=dev/spring.profiles.active=prod/g" $project_dir/src/main/resources/application.properties
#cp -rf $prod_cfgdir/Dockerfile $project_dir/
#rm -rf $project_dir/src/main/resources/application-*.properties
#cp -rf $prod_cfgdir/application-prod.properties $project_dir/src/main/resources/
#)

mvn clean package dockerfile:build
docker tag mydocker/myapp:0.01 registry-vpc.cn-huhehaote.aliyuncs.com/mydocker/myapp:0.01
#docker tag mydocker/myapp:0.01 registry-vpc.cn-huhehaote.aliyuncs.com/mydocker/myapp:0.01-prod
docker push registry-vpc.cn-huhehaote.aliyuncs.com/mydocker/myapp:0.01
#docker push registry-vpc.cn-huhehaote.aliyuncs.com/mydocker/myapp:0.01-prod

cd $deploy_dir
kubectl delete deployments/myapp

echo "delete status..."
kubectl rollout status deployment/myapp

kubectl create -f myapp.yaml
#kubectl create -f $prod_cfgdir/myapp-prod.yaml

echo "create status..."
kubectl rollout status deployment/myapp

```




