---
layout: post
title: Weixin相关
category: [Linux-Mint,微信,Weixin,frp]
comments: false
---

* content
{:toc}

### 内网穿透

#### 安装[frp](https://github.com/fatedier/frp)
```
服务端frps.ini:
[common]
bind_port = 7000

vhost_http_port = 7000
subdomain_host = mytest.com
dashboard_port = 7500
dashboard_user = xxx
dashboard_pwd = xxx

客户端frpc.ini:
[common]
server_addr = 服务端所在公网IP
server_port = 7000

[web01]
type = http
local_ip = 127.0.0.1
local_port = 8080
use_encryption = false
use_compression = true
http_user = 
http_pwd = 
subdomain = dev

服务端nginx配置vim /etc/nginx/conf.d/dev.mytest.com.conf:
server {
    listen       80;
    server_name dev.mytest.com www.dev.mytest.com;
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    location / {
                index index.jsp index.html index.htm;
                proxy_pass http://127.0.0.1:7000;
                proxy_set_header   Host             $host;
                proxy_set_header   X-Real-IP        $remote_addr;
                proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }

}
nginx -s reload
启动服务端：./frps -c ./frps.ini
启动客户端：./frpc -c ./frpc.ini
启动本地eclipse或其它web工程并设置为8080端口
其它外网机器通过dev.mytest.com即可访问到内网机器进行远程调试

```

#### 使用免费服务[natapp](https://natapp.cn/login)
```
[下载natapp](https://natapp.cn/#download)
客户端选择[linux版本](http://download.natapp.cn/assets/downloads/clients/2_3_8/natapp_linux_amd64_2_3_8.zip),

~/bin/natapp -authtoken=登录后台（1842-c17）查看购买免费型隧道生成的authtoken	

```






