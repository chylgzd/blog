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

#### 国内镜像加速

```
登陆https://cr.console.aliyun.com
zfd打开镜像加速器会有提示：

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxx.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker

```

#### 根目录/var/lib/docker迁移到其它挂载数据盘

```
service docker stop

挂载数据磁盘到data目录：
mkdir -p /data
1. fdisk -l 找出所有磁盘及空闲磁盘比如/dev/vdb
2. 格式化该磁盘mkfs -t ext4 /dev/vdb
3. 创建mkdir -p /data数据文件夹
4. 挂载磁盘 mount /dev/vdb /data
5. 系统级重启挂载: 
vim /etc/fstab
/dev/vdb		/data		ext4		defaults 	0 0

迁移到/data/docker/root/目录下：
mkdir -p /data/docker/root 
mv  /var/lib/docker/*  /data/docker/root/
chown -R root:root  /data/docker/root
chmod 777 -R /data/docker/root

临时邦定文件夹，重启会失效：
mount --bind  /data/docker/root  /var/lib/docker

service docker start

系统级邦定文件夹，重启后不失效：
vim /etc/fstab
/data/docker/root /var/lib/docker                 none    bind        0 0


``` 

#### 查找docker数据目录默认文件夹(方便迁移复制数据目录)

```
> docker inspect postgresql
找到Mounts节点，Destination表示docker容器内目录，Source表示宿主机目录
"Mounts": [
    {
        "Type": "volume",
        "Name": "5a1a07a6cd8d997cb7f86cfe44",
        "Source": "/var/lib/docker/volumes/5a1a07a6cd8d997cb7f86cfe44/_data",
        "Destination": "/var/lib/postgresql/data",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
]

```


###  Docker命令使用：

```
修改时区错误问题：
> docker cp /etc/localtime 容器ID:/etc/localtime

docker run 参数：
--name 容器别名
--link 外部容器名：内定别名（如gitlab容器配置--link redis:redisio，其中redisio是内部定的，如果没有规定可以随意，而redis是另外的容器）
-v 外部文件夹目录：内部目录（如tomcat容器配置-v /home/yourname/myapps:/usr/local/tomcat/webapps）
(
使用 -v /etc/localtime:/etc/localtime:ro 可以解决 docker容器时间不同步问题 
docker run --name nginx -p 80:80 -v /home/html:/usr/share/nginx/html -v /home/nginx.conf:/etc/nginx/nginx.conf -v /etc/localtime:/etc/localtime:ro -d daocloud.io/library/nginx:latest
)
-p 外部端口：内部端口（如redis容器配置 -p 6379:6379把内部端口映射出来）
-d 镜像名：镜像版本号

启动/停止docker服务
sudo service docker start
sudo service docker stop

查看/删除镜像
docker images
docker rmi 镜像ID

删除所有空标签的镜像
docker images|sed "1 d"|grep "<none>" |awk '{print $3}' |xargs docker rmi

查看所有容器状态：
docker ps -a

查看某容器相关文件存储版本情况：
docker inspect gitlab

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

复制docker容器内文件到宿主机(docker ps -a查看容器id)：
docker cp 容器ID:/etc/profile.d/env.sh /home/soft/download

执行某个容器内的命令：
docker exec -it jenkins-web cat /var/jenkins_home/secrets/initialAdminPassword

进入某个容器1：
docker exec -it gitlab bash

进入某个容器2：
先查找出容器的进程PID，再用系统命令nsenter连接即可，连上后就如同操作linux：
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



