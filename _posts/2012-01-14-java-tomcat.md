---
layout: post
title: Tomcat 相关
category: [Java,Tomcat,乱码,启动慢,CentOS6]
comments: false
---

* content
{:toc}

具体参照：

[https://www.igray.cc/465.html](https://www.igray.cc/465.html)

[http://maven.apache.org/settings.html](http://maven.apache.org/settings.html)

### CentOS6下

#### 安装Tomcat,jsvc自启动

```

#查看系统自带的Java并对应删除
[root@localhost ~]# rpm -qa | grep java
java_cup-0.10k-5.el6.i686
tzdata-java-2017b-1.el6.noarch
java-1.5.0-gcj-1.5.0.0-29.1.el6.i686
[root@localhost ~]# rpm -e --nodeps java-1.5.0-gcj-1.5.0.0-29.1.el6.i686

#配置环境变量
打开vi /etc/profile，文件最后添加如下内容后source /etc/profile查看java -version
export JAVA_HOME=/home/xxx/jdk1.8.0_181
export JRE_HOME=$JAVA_HOME/jre
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib

#添加tomcat用户
sudo useradd -r -s /sbin/nologin tomcat

#NIO与URIEncoding /xxx/apache-tomcat-8.5.33/conf/server.xml
<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol"
           connectionTimeout="20000"
           redirectPort="8443" URIEncoding="UTF-8" />

#Tomcat开机自启动（自带jsvc工具让Tomcat以daemon模式运行服务）
[root@localhost ~]# cd /xxx/apache-tomcat-8.5.33/bin
[root@localhost ~]# tar -zxvf commons-daemon-native.tar.gz
[root@localhost ~]# cd commons-daemon-1.1.0-native-src/unix
[root@localhost ~]# ./configure --with-java=/home/xxx/jdk1.8.0_181
[root@localhost ~]# make
[root@localhost ~]# cp jsvc ../..
[root@localhost ~]# cd ../..
[root@localhost ~]# vim daemon.sh
#!/bin/sh
# chkconfig: - 80 20
JAVA_HOME=/home/soft/jdk1.8.0_181
CATALINA_HOME=/home/soft/apache-tomcat-8.5.33
TOMCAT_USER=tomcat
#防止乱码
CATALINA_OPTS="-Dfile.encoding=UTF8 -Dsun.jnu.encoding=UTF8"

# Licensed to the Apache Software Foundation (ASF) under one or more
[root@localhost ~]# ln -s /xxx/apache-tomcat-8.5.33/bin/daemon.sh /etc/init.d/tomcatd
[root@localhost ~]# chkconfig --add tomcatd
[root@localhost ~]# chkconfig --level 2345 tomcatd on
[root@localhost ~]# service tomcatd start
[root@localhost ~]# service tomcatd stop

```

#### Tomcat日志jsvc切分

```
#切分日志脚本，删除7天前的日志

> vim /xx/shell/test.sh

#!/bin/bash
thedate=`date --rfc-3339=date`
predate=`date +%Y-%m-%d --date="-7 day"`
logs_dir=/xxx/xxx/apache-tomcat-8.5.33/logs

rmfile="${logs_dir}/catalina-daemon.${predate}.out"
outfile="${logs_dir}/catalina-daemon.out"
if [ -f ${rmfile} ];then
   rm -f ${rmfile}
fi

if [ -f ${outfile} ];then
   cp ${outfile} ${logs_dir}/catalina-daemon.${thedate}.out
   echo "" > ${outfile}
fi

exit 0


#定时任务crontab脚本执行切分脚本
> crontab -e(centos6 vim /etc/crontab )

59 23 * * * root /xx/shell/splitlog.sh

> /etc/rc.d/init.d/crond restart (service crond restart)
> crontab -l

```

#### Tomcat 并发设置

```
默认配置
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="150" minSpareThreads="4"/>
并发参考
<Executor name="tomcatThreadPool"        # 配置TOMCAT共享线程池，NAME为名称　
          namePrefix="HTTP-8080-exec-"    # 线程的名字前缀，用于标记线程名称
          prestartminSpareThreads="true"  # executor启动时，是否开启最小的线程数
          maxThreads="5000"               # 允许的最大线程池里的线程数量，默认是200，大的并发应该设置的高一些，这里设置可以支持到5000并发
          maxQueueSize="100"              # 任务队列上限
          minSpareThreads="50"            # 最小的保持活跃的线程数量，默认是25.这个要根据负载情况自行调整了。太小了就影响反应速度，太大了白白占用资源
          maxIdleTime="10000"             # 超过最小活跃线程数量的线程，如果空闲时间超过这个设置后，会被关别。默认是1分钟。
 />

<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol"
   connectionTimeout="5000" redirectPort="443" proxyPort="443" executor="tomcatThreadPool"
   URIEncoding="UTF-8"/>


# 查看tomcat端口进程pid
> lsof -i:8080

# 查看进程号线程数
> ps -Lf 26543|wc -l
> ps -T -p 26543|wc -l

# 查看端口的并发数(tomcat-8080)
> netstat -an|grep 8080|awk '{count[$6]++} END{for (i in count) print(i,count[i])}'

```


### 自动拉取代码打包编译部署war

```
#!/bin/sh

mvn_bin=/data/maven/apache-maven-3.5.4/bin/mvn
project_dir=/data/deploy/yourproject/yourgitproject
war_name=yourproject-0.0.1.war
tomcat_dir=/home/xxx/apache-tomcat-8.5.33
webroot_dir=$tomcat_dir/webapps/ROOT
webroot_bkdir=/data/_bak/webapp
cd $project_dir

if [ $# -eq 0 ];then
echo "branch name is empty！"
git branch -la
exit 0
fi

#停止服务
service tomcatd stop

#备份ROOT
b_time=`date '+%Y%m%d%H%M%S'`
tar -zcPf $webroot_bkdir/ROOT$b_time.tar.gz -C $tomcat_dir/webapps ROOT --exclude=yourdir

#拉取代码
branch_name=master
git fetch origin $branch_name
git reset --hard origin/$branch_name
git pull
git checkout $branch_name

#删除本地历史target并打包编译
rm -rf target/*
$mvn_bin clean package -Dmaven.test.skip=true -DskipYuicompressor=false -e

error_code=$?
if [ $error_code -eq 0 ];then
    error_msg="ok！"
    echo $error_msg
else
    error_msg="fail！"
    echo $error_msg
fi

#删除线上运行webroot目录并解压刚打包的到该目录
rm -rf $webrootdir_dir/*
unzip -oq target/$war_name -d $webroot_dir

#做一些其它替换
rm -rf $webroot_dir/WEB-INF/classes/application-*.properties 
cp -rf ../application-prod.properties $webroot_dir/WEB-INF/classes/
cat ../application.properties > $webroot_dir/WEB-INF/classes/application.properties

#启动服务
service tomcatd start 

#通知部署结果
curl 'https://oapi.dingtalk.com/robot/send?access_token=xxxxxx' \
   -H 'Content-Type: application/json' \
   -d '
  {"msgtype": "text", 
    "text": {
        "content":"'$error_msg'"
     }
  }'

exit 0

```

### tomcat启动慢的问题

```
解决方法：
> vim ./bin/catalina.sh

#!/bin/sh

JAVA_OPTS="-DXmx=300M -Dfile.encoding=UTF8 -Dsun.jnu.encoding=UTF8 -Djava.security.egd=file:/dev/./urandom"

# Licensed to the Apache Software Foundation (ASF) under one or more

产生原因：
Tomcat 7/8都使用org.apache.catalina.util.SessionIdGeneratorBase.createSecureRandom类产生安全随机类SecureRandom的实例作为会话ID，耗时几分钟
SHA1PRNG算法是基于SHA-1算法实现且保密性较强的伪随机数生成器
在SHA1PRNG中，有一个种子产生器，它根据配置执行各种操作
如果java.security.egd属性或securerandom.source属性指定的是”file:/dev/random”或”file:/dev/urandom”，
那么JVM会使用本地种子产生器NativeSeedGenerator，它会调用super()方法，即调用SeedGenerator.URLSeedGenerator(/dev/random)方法进行初始化
如果java.security.egd属性或securerandom.source属性指定的是其它已存在的URL，那么会调用SeedGenerator.URLSeedGenerator(url)方法进行初始化
这就是为什么设置值”file:///dev/urandom”或者值为”file:/./dev/random”都会起作用的原因
在这个实现中，产生器会评估熵池（entropy pool）中的噪声数量。随机数是从熵池中进行创建的
当读操作时，/dev/random设备会只返回熵池中噪声的随机字节。/dev/random非常适合那些需要非常高质量随机性的场景，比如一次性的支付或生成密钥的场景
当熵池为空时，来自/dev/random的读操作将被阻塞，直到熵池收集到足够的环境噪声数据
这么做的目的是成为一个密码安全的伪随机数发生器，熵池要有尽可能大的输出
对于生成高质量的加密密钥或者是需要长期保护的场景，一定要这么做

那么什么是环境噪声？
随机数产生器会手机来自设备驱动器和其它源的环境噪声数据，并放入熵池中,产生器会评估熵池中的噪声数据的数量
当熵池为空时，这个噪声数据的收集是比较花时间的
这就意味着，Tomcat在生产环境中使用熵池时，会被阻塞较长的时间。

解决
有两种解决办法：

1.在Tomcat环境中解决，可以通过配置JRE使用非阻塞的Entropy Source。在catalina.sh中加入这么一行：-Djava.security.egd=file:/dev/./urandom;
2.在JVM环境中解决，打开$JAVA_HOME/jre/lib/security/java.security这个文件，
找到下面的内容：securerandom.source=file:/dev/urandom替换成securerandom.source=file:/dev/./urandom

```



