---
layout: post
title: 安装Docker
category: [Docker]
comments: false
---

* content
{:toc}

具体参照：

[https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)

[http://www.widuu.com/chinese_docker/installation/ubuntu.html](http://www.widuu.com/chinese_docker/installation/ubuntu.html)

### 安装配置

```bash

sudo gedit /etc/apt/sources.list.d/docker.list

使用cat /proc/version查看系统Ubuntu版本

Ubuntu Precise 12.04 添加此源：
deb https://apt.dockerproject.org/repo ubuntu-precise main

Ubuntu Trusty 14.04或Linux Mint 17 添加此源：
deb https://apt.dockerproject.org/repo ubuntu-trusty main

Ubuntu Xenial 16.04或Linux Mint 18 添加此源：
deb https://apt.dockerproject.org/repo ubuntu-xenial main

保存文件并更新源：
sudo apt-get update
sudo apt-get purge lxc-docker
apt-cache policy docker-engine

安装前需：
sudo apt-get install linux-image-extra-$(uname -r)
sudo apt-get install apparmor

安装：
sudo apt-get install docker-engine
启动：
sudo service docker start
或
sudo start docker

验证安装是否正常：
sudo docker run hello-world

配置用户/组（需要注销重新登陆）：
sudo groupadd docker
sudo usermod -aG docker yourname

确认用户是否成功添加到docker：
docker run hello-world

查看版本及日志：
docker version
sudo more /var/log/upstart/docker.log

防火墙配置：
sudo ufw status
sudo gedit /etc/default/ufw
修改值为ACCEPT：
DEFAULT_FORWARD_POLICY="ACCEPT"
保存并重启防火墙：
sudo ufw reload
sudo ufw allow 2375/tcp

Docker DNS server配置：
sudo gedit /etc/default/docker
修改内容值：
DOCKER_OPTS="--dns 223.5.5.5 --dns 114.114.114.114 --dns 192.168.0.1"
保存重启Docker：
sudo restart docker

更新Docker：
sudo apt-get upgrade docker-engine

取消Docker服务自启动：
sudo systemctl disable docker
echo manual | sudo tee /etc/init/docker.override

开启Docker自启动：
sudo systemctl enable docker
sudo rm /etc/init/docker.override

卸载docker：
sudo apt-get autoremove --purge docker-engine
彻底删除images,containers, volumes等：
rm -rf /var/lib/docker
```

###  Docker命令使用：

```
docker run 参数：
--name 容器别名
--link 外部容器名：内定别名（如gitlab容器配置--link redis:redisio，其中redisio是内部定的，如果没有规定可以随意，而redis是另外的容器）
-v 外部文件夹目录：内部目录（如tomcat容器配置-v /home/yourname/myapps:/usr/local/tomcat/webapps）
-p 外部端口：内部端口（如redis容器配置 -p 6379:6379把内部端口映射出来）
-d 镜像名：镜像版本号

启动/停止docker服务
sudo service docker start
sudo service docker stop

查看/删除镜像
docker images
docker rmi 镜像ID

查看所有容器状态：
docker ps -a

查看某容器日志：
docker logs redis-web

根据NAMES启动或停止相关的容器：
docker start redis
docker stop redis
docker start redis-web
docker stop redis-web

根据docker ps -a列表中的CONTAINER ID或NAMES删除相关容器：
docker rm 0a522e3f3009
docker rm redis
docker rm redis-web

查看某个容器的全部环境变量：
docker exec jenkins-web env

执行某个容器内的命令：
docker exec -it jenkins-web cat /var/jenkins_home/secrets/initialAdminPassword

进入某个容器，先查找出容器的进程PID，再用系统命令nsenter连接即可，连上后就如同操作linux：
使用
docker top redis-web
或者使用(.State.Pid)
docker inspect -f "{{  _State_Pid  }}" redis-web
sudo nsenter -t 进程PID -m -u -i -n -p
```

### Dockerfile创建docker镜像

```
cd /tmp
gedit Dockerfile

写入以下内容：
FROM tomcat:latest
ADD mywebapp.war /usr/local/tomcat/webapps/
CMD ["catalina.sh", "run"]

运行build命令打包镜像：
docker build -t hello-test . 

运行docker images即可查看到镜像已创建，运行容器：
docker run  -p 8080:8080 hello-test -d hello-test:latest
```

### 用希云cSphere管理docker

```
参考：
https://csphere.cn/docs/installation.html

在宿主机器上执行以下2句命令：

curl -SsL -o /tmp/csphere-install.sh https://csphere.cn/static/csphere-install-v2.sh

sudo env ROLE=controller CSPHERE_VERSION=1.0.1 /bin/sh /tmp/csphere-install.sh

成功后访问 http://localhost:1016

进入设置 -> 基本设置 Controller（IP即csphere安装所在机器的IP地址） 修改
连接地址：192.168.0.33 （不要用localhost或者127.0.0.1）
端口：1016

添加主机（即安装有docker的客户机） -> NAT 局域网 -> 得到一串字符串命令

curl -SsL -o /tmp/csphere-install.sh https://csphere.cn/static/csphere-install-v2.sh
sudo env ROLE=agent CONTROLLER_IP=192.168.0.33 CONTROLLER_PORT=1016 CSPHERE_VERSION=1.0.1 AUTH_KEY=xxx SVRPOOLID=xxx /bin/sh /tmp/csphere-install.sh

在客户机执行即可
```



