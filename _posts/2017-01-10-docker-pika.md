---
layout: post
title: 用Docker打包编译Pika并制成docker镜像
category: [Docker,Pika]
comments: false
---

* content
{:toc}

具体参照：

[pika安装维护相关](https://github.com/Qihoo360/pika/blob/master/README_CN.md)

### 在docker容器里面打包编译pika

```
docker下载一个centos6镜像：
docker pull hub.c.163.com/public/centos:6.5

启动镜像以推出就删除模式：
docker run --rm -it centos:6.8

进去后就是一个centos环境了，安装vim，修改更新源为aliyun(参考此文中CentOS6-Base-aliyun.repo)：
yum install -y vim
vim /etc/yum.repos.d/CentOS-Base.repo

修改之后根据pika安装维护相关里的说明编译安装，过程中需要注意：

编译前设置Makefile.global为（启动pika的时候会去这个路径找so文件）：
./lib

安装bz2：
yum install -y bzip2*

安装protobuf：
wget https://sourceforge.net/projects/protobuf/files/protobuf-2.6.0.tar.gz/download
mv download protobuf-2.6.0.tar.gz
tar zxvf protobuf-2.6.0.tar.gz
cd protobuf-2.6.0
./configure
make
make check
make install
安装结束。
验证：
查看是否安装成功：protoc --version
如果出现：libprotoc 2.6.0 则说明安装成功！

如果需要可以安装开发编译工具包：
yum groupinstall 'Development Tools'

编译成功后，打包output目录：
tar -zcf pikax.x.x_centos6_bin.tar.gz

上传到宿主机输入密码即可：
scp pikax.x.x_centos6_bin.tar.gz root@192.168.0.11:/tmp
```

### 根据已经编译的pika包做成docker镜像文件

四个文件：CentOS6-Base-aliyun.repo，Dockerfile，[pika2.1.4_centos6_bin.tar.gz](/blog/download/docker/pika/2.1.4_centos6/pika2.1.4_centos6_bin.tar.gz)，readme.md

按照readme.md执行即可

#### CentOS6-Base-aliyun.repo

```
# CentOS6-Base-aliyun.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
 
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-6
 
#released updates 
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-6
 
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-6
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-6
 
#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/contrib/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-6
```

#### CentOS7-Base-aliyun.repo

```
# CentOS7-Base-aliyun.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/contrib/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

```

#### CentOS7-Base-163.repo

```
# CentOS7-Base-163.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
[base]
name=CentOS-$releasever - Base - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
baseurl=http://mirrors.163.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
baseurl=http://mirrors.163.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

```

#### Dockerfile

```
FROM hub.c.163.com/public/centos:6.5
ADD CentOS6-Base-aliyun.repo /etc/yum.repos.d/CentOS-Base.repo
#RUN yum makecache fast && yum -y update glic && yum install -y wget vim tar 
RUN yum -y update && yum install -y wget vim
#RUN cd /tmp && wget https://github.com/Qihoo360/pika/releases/download/v2.1.4/pika2.1.4_centos6_bin.tar.gz
ENV PIKA_HOME /opt/pika2.1.4/
RUN mkdir -p $PIKA_HOME
ADD pika2.1.4_centos6_bin.tar.gz $PIKA_HOME
#RUN cd $PIKA_HOME && ls -l
EXPOSE 9221
#CMD ["/bin/bash"]
CMD cd $PIKA_HOME/output && ./bin/pika -c conf/pika.conf
```

#### readme.md

```
docker build -t pika360:2.1.4  .

docker run --name pika -p 9221:9221 -d pika360:2.1.4

redis client connect port 9221
```

### docker-tengine-php.Dockerfile

```
FROM hub.c.163.com/public/centos:7.2
MAINTAINER ngineered <yourname@mail.com>

ADD repo/CentOS7-Base-aliyun.repo /etc/yum.repos.d/CentOS-Base.repo
ADD repo/CentOS7-Base-163.repo /etc/yum.repos.d/CentOS-Base-163.repo
# env set ==============================================
RUN echo "环境变量设置..."
ENV WUSER=www \
    TENGINX=2.1.2 \
    PHP=7.0.17 \
    MCRYPT=2.5.7 \
    TMP_WK=/tmp/wk

# install tengine dependency ==============================================
RUN echo "依赖包安装..."
RUN yum install -y gcc gcc-c++ make automake cmake bison autoconf wget lrzsz \
    yum install -y libtool libtool-ltdl-devel  \
    yum install -y freetype-devel libjpeg.x86_64 libjpeg-devel libpng-devel gd-devel \
    yum install -y python-devel  patch  sudo  \
    yum install -y openssl* openssl openssl-devel ncurses-devel \
    yum install -y bzip* bzip2 unzip zlib-devel \
    yum install -y libevent* \
    yum install -y libxml* libxml2-devel \
    yum install -y libcurl* curl-devel  \
    yum install -y readline-devel \
    #yum install libmcrypt libmcrypt-devel mcrypt mhash \
    yum install -y zlib zlib-devel openssl openssl-devel pcre-devel

# install tengine ==============================================
RUN echo "安装tengine..."
ADD lib/tengine-$TENGINX.tar.gz $TMP_WK
RUN CONFIG_TENGINE="\
    --prefix=/usr/local/nginx \
    --sbin-path=/usr/sbin/nginx \
    --conf-path=/usr/local/nginx/config/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --pid-path=/var/run/nginx/nginx.pid \
    --user=$WUSER \
    --group=$WUSER \
    --with-http_ssl_module \
    --with-http_flv_module \
    --with-http_gzip_static_module \
    --http-log-path=/var/log/nginx/access.log \
    --http-client-body-temp-path=/var/tmp/nginx/client \
    --http-proxy-temp-path=/var/tmp/nginx/proxy \
    --http-fastcgi-temp-path=/var/tmp/nginx/fcgi \
    --with-http_stub_status_module \
  " \
  && cd $TMP_WK/tengine-$TENGINX/ \
  && ./configure $CONFIG_TENGINE \
  && make -j$(getconf _NPROCESSORS_ONLN) \
  && make install -j$(getconf _NPROCESSORS_ONLN) \
  && groupadd $WUSER \
  && useradd -g $WUSER $WUSER \
  && mkdir -p /usr/local/nginx/logs \
  && mkdir -p /var/tmp/nginx/client \
  && mkdir -p /var/tmp/nginx/proxy \
  && mkdir -p /var/tmp/nginx/fcgi

ADD conf/nginx/config/ /usr/local/nginx/config/
ADD wwwroot/ /var/www/html/

# install php ==============================================
RUN echo "安装php7..."
ADD lib/php-$PHP.tar.gz $TMP_WK
ADD lib/libmcrypt-$MCRYPT.tar.gz $TMP_WK
RUN CONFIG_PHP="\
    --prefix=/usr/local/php7 \
    --with-config-file-path=/usr/local/php7/etc \
    --with-config-file-scan-dir=/usr/local/php7/etc/php.d \
    --with-mcrypt=/usr/include \
    --enable-mysqlnd \
    --with-mysqli \
    --with-pdo-mysql \
    --enable-fpm \
    --with-fpm-user=$WUSER \
    --with-fpm-group=$WUSER \
    --with-gd \
    --with-iconv \
    --with-zlib \
    --enable-xml \
    --enable-shmop \
    --enable-sysvsem \
    --enable-inline-optimization \
    --enable-mbregex \
    --enable-mbstring \
    --enable-ftp \
    --enable-gd-native-ttf \
    --with-openssl \
    --enable-pcntl \
    --enable-sockets \
    --with-xmlrpc \
    --enable-zip \
    --enable-soap \
    --without-pear \
    --with-gettext \
    --enable-session \
    --with-curl \
    --with-jpeg-dir \
    --with-freetype-dir \
    --enable-opcache \
  " \
  && cd $TMP_WK/libmcrypt-$MCRYPT/ \
  && ./configure --prefix=/usr/local \
  && make -j$(getconf _NPROCESSORS_ONLN) \
  && make install -j$(getconf _NPROCESSORS_ONLN) \

  && cd $TMP_WK/php-$PHP/ \
  && ./configure $CONFIG_PHP \
  && make -j$(getconf _NPROCESSORS_ONLN) \
  && make install -j$(getconf _NPROCESSORS_ONLN)

ADD conf/php7/etc/ /usr/local/php7/etc/

# start script ==============================================
ADD scripts/start.sh /usr/local/main/start.sh
RUN chmod 700 /usr/local/main/start.sh

EXPOSE 7777 80
#STOPSIGNAL SIGTERM
CMD ["/usr/local/main/start.sh"]

```

