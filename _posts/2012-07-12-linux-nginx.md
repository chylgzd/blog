---
layout: post
title: Nginx配置文件备份
category: [Linux,Nginx]
comments: false
---

* content
{:toc}

```
nginx -s stop
nginx
```

#### /alidata/server/nginx/conf/proxy.conf
```conf
proxy_redirect          off;
proxy_set_header        Host $host;
proxy_set_header        X-Real-IP $remote_addr;
proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
client_max_body_size    10m;
client_body_buffer_size 128k;
proxy_connect_timeout   300;
proxy_send_timeout      300;
proxy_read_timeout      300;
proxy_buffer_size       4k;
proxy_buffers           4 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
```

#### /alidata/server/nginx/conf/nginx.conf
```conf
user  www www;
worker_processes  2;

error_log  /alidata/log/nginx/error.log crit;
pid        /alidata/server/nginx/logs/nginx.pid;

#Specifies the value for maximum file descriptors that can be opened by this process. 
worker_rlimit_nofile 65535;

events 
{
  use epoll;
  worker_connections 65535;
}


http {
	include       mime.types;
	default_type  application/octet-stream;

	#charset  gb2312;

	server_names_hash_bucket_size 128;
	client_header_buffer_size 32k;
	large_client_header_buffers 4 32k;
	client_max_body_size 8m;

	sendfile on;
	tcp_nopush     on;

	keepalive_timeout 60;

	tcp_nodelay on;

	fastcgi_connect_timeout 300;
	fastcgi_send_timeout 300;
	fastcgi_read_timeout 300;
	fastcgi_buffer_size 64k;
	fastcgi_buffers 4 64k;
	fastcgi_busy_buffers_size 128k;
	fastcgi_temp_file_write_size 128k;

	gzip on;
	gzip_min_length  1k;
	gzip_buffers     4 16k;
	gzip_http_version 1.0;
	gzip_comp_level 2;
	gzip_types       text/plain application/x-javascript text/css application/xml;
	gzip_vary on;
	#limit_zone  crawler  $binary_remote_addr  10m;
	log_format '$remote_addr - $remote_user [$time_local] "$request" '
	              '$status $body_bytes_sent "$http_referer" '
	              '"$http_user_agent" "$http_x_forwarded_for"';
	include /alidata/server/nginx/conf/vhosts/*.conf;
}
```

#### /alidata/server/nginx/conf/vhosts/hello.com.conf
```conf
server
{
	listen 80;
	server_name www.hello.com hello.com;
	index index.html index.php;
	rewrite ^(.*) http://www.hello.com$1 redirect;
}
server {
    listen       80;
    server_name hello.com;
 	location / {
		index index.jsp index.html index.htm;
		proxy_pass http://127.0.0.1:8080;
		include proxy.conf;
    }
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    location ~ \.(html|gif|jpg|jpeg|png|ico|rar|css|js|zip|txt|flv|swf|doc|ppt|xls|pdf)$ {
  		root /home/tomcat-hello/webapps/ROOT;
  		access_log off;
      	expires 1d;
    }
}
server {
    listen       443; 
    server_name  hello.com;  
    ssl                  on;  
    ssl_certificate      sslkey/hello.com_bundle.crt;
    ssl_certificate_key      sslkey/hello.com.key;
    ssl_session_timeout  5m;  
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;  
    ssl_ciphers ALL:!ADH:!EXPORT56:-RC4+RSA:+HIGH:+MEDIUM:!EXP; 
    ssl_prefer_server_ciphers   on;  
    location / {
		index index.jsp index.html index.htm;
		proxy_pass https://127.0.0.1:8443;
		include proxy.conf;
    }
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    location ~ \.(html|gif|jpg|jpeg|png|ico|rar|css|js|zip|txt|flv|swf|doc|ppt|xls|pdf)$ {
  		root /home/tomcat-hello/webapps/ROOT;
  		access_log off;
      	expires 1d;
    }
}
```

#### /alidata/server/nginx/conf/sslkey目录存放安全认证文件
```
hello.com.key
hello.com_bundle.crt
```
