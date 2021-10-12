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
local_port = 80
use_encryption = false
use_compression = true
http_user = 
http_pwd = 
subdomain = dev

服务端nginx配置vim /etc/nginx/conf.d/dev.mytest.com.conf:
server {
    listen       80;
    server_name dev.mytest.com;
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

客户端nginx配置vim /etc/nginx/conf.d/dev.mytest.com.conf:
server {
    listen       80;
    server_name dev.mytest.com;
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    location / {
        root /data/www/dist;
        index index.html index.htm;
        access_log off;
        expires 1d;
    }
    location /back-end {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
    }
}
启动客户端：./frpc -c ./frpc.ini
启动本地eclipse或其它web工程并设置为8080端口（对应本地nginx的/back-end）
其它外网机器通过dev.mytest.com即可访问到内网机器进行远程调试

```

#### 使用免费服务[natapp](https://natapp.cn/login)
```
[下载natapp](https://natapp.cn/#download)
客户端选择[linux版本](http://download.natapp.cn/assets/downloads/clients/2_3_8/natapp_linux_amd64_2_3_8.zip),

~/bin/natapp -authtoken=登录后台（1842-c17）查看购买免费型隧道生成的authtoken	

```

### 微信开发配置相关

#### 微信公众号设置登录运营管理号

```
https://mp.weixin.qq.com/

设置 -> 人员设置 -> 运营者管理 -> 绑定运营者微信号

```

#### 微信公众平台配置开发者

```
https://mp.weixin.qq.com/
登录需要调试开发的公众号：

开发 -> 开发者工具 -> web开发者工具 -> 绑定开发者微信号

```

#### 微信开发者工具下载(需先绑定开发者微信号)
```
github-Linux版:
https://github.com/cytle/wechat_web_devtools/releases

gitee-Linux版：
https://gitee.com/chao-fan/wechat_web_devtools/releases

官方下载地址：
https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html

启动：
/wechat_web_devtools/bin/wxdt
```

#### 微信开放平台配置（多个公众号UnionId）

```
https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=1417751808&token=&lang=zh_CN
```

#### 微信反盗链设置

```
出现此图片来自微信公众平台未经允许不可引用

方法1，在public目录的index.html引入头:

#所有请求都不发送referrer
<meta name="referrer" content="never">
或
#只发送相同域的referrer
<meta name="referrer" content="same-origin">

方法2，使用代理:
location ^~ /wechat_image/ {
    add_header 'Access-Control-Allow-Origin' "$http_origin" always;
    add_header 'Access-Control-Allow-Credentials' 'true' always;
    add_header 'Access-Control-Allow-Methods' 'GET, OPTIONS' always;
    add_header 'Access-Control-Allow-Headers' 'Accept,Authorization,Cache-Control,Content-Type,DNT,If-Mod    ified-  Since,Keep-Alive,Origin,User-Agent,X-Requested-With' always;
    proxy_pass http://wx.qlogo.cn/;
 }
 
然后把请求微信图片地址替换成/wechat_image/xxx即可
```



