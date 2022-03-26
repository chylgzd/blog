---
layout: post
title: Docker,Kubernetes相关
category: [Docker,Kubectl]
comments: false
---

* content
{:toc}

具体参照：

[https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)

[http://www.widuu.com/chinese_docker/installation/ubuntu.html](http://www.widuu.com/chinese_docker/installation/ubuntu.html)

### 安装docker

#### HW云安装
```
清理旧的：
> sudo yum remove docker docker-common docker-selinux docker-engine
> sudo yum install -y yum-utils device-mapper-persistent-data lvm2

下载对应系统版本https://mirrors.huaweicloud.com/home (容器相关docker-ce)
centos:
> wget -O /etc/yum.repos.d/docker-ce.repo https://repo.huaweicloud.com/docker-ce/linux/centos/docker-ce.repo

> sudo sed -i 's+download.docker.com+repo.huaweicloud.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo

> sudo yum makecache fast
安装
> sudo yum install docker-ce
启动
> sudo systemctl start docker

```

#### 旧版安装
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
方式1使用云加速帐户：
登陆https://cr.console.aliyun.com
zfd打开镜像加速器会有提示：

> sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxx.mirror.aliyuncs.com"]
}
EOF

> sudo systemctl daemon-reload
> sudo systemctl restart docker

方式2使用通用加速(daemon.json不存在则新建)：
> vim /etc/docker/daemon.json
{
  "registry-mirrors": [
  	"https://docker.mirrors.ustc.edu.cn/",
  	"https://hub-mirror.c.163.com",
  	"https://registry.docker-cn.com",
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com",
    "https://registry.docker-cn.com"
  ]
}
> sudo systemctl daemon-reload
> sudo systemctl restart docker

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

#### 通用
```
修改mysql时区错误问题：
宿主 > docker exec -it mysql5 bash #进入容器
		> docker exec -u root -it superset bash #使用root进入容器
容器 > mkdir -p /usr/share/zoneinfo/Asia && rm -rf /etc/localtime
宿主 > docker cp /usr/share/zoneinfo/Asia/Shanghai 容器ID:/usr/share/zoneinfo/Asia
容器 > cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
宿主 > docker restart mysql5

容器内访问宿主机端口(一般172.17.0.1代表宿主机,或ifconfig查看与宿主机的网桥ip)如访问宿主mysql:
> telnet 172.17.0.1 3306

运行容器docker run 参数：
--name 容器别名
--link 外部容器名：内定别名（如gitlab容器配置--link redis:redisio，其中redisio是内部定的，如果没有规定可以随意，而redis是另外的容器）
-v 外部文件夹目录：内部目录（如tomcat容器配置-v /home/yourname/myapps:/usr/local/tomcat/webapps）
(
使用 -v /etc/localtime:/etc/localtime:ro 可以解决 docker容器时间不同步问题 
docker run --name nginx -p 80:80 -v /home/html:/usr/share/nginx/html -v /home/nginx.conf:/etc/nginx/nginx.conf -v /etc/localtime:/etc/localtime:ro -d daocloud.io/library/nginx:latest
)
-p 外部端口：内部端口（如redis容器配置 -p 6379:6379把内部端口映射出来）
-d 镜像名：镜像版本号

#临时运行并进入容器(运行完毕自动删除容器):
> docker run --rm -it 容器名:版本号 sh

启动/停止docker服务
sudo service docker start
sudo service docker stop
或
sudo systemctl restart docker

已启动的容器修改参数，如：是否开启自启动参数
docker container update --restart=always 容器ID(docker ps -a可查看到)
docker container update --restart=no 容器ID
restart相关参数值：
-no，默认策略，在容器退出时不重启容器
-on-failure，在容器非正常退出时（退出状态非0），才会重启容器
-on-failure:3，在容器非正常退出时重启容器，最多重启3次
-always，docker启动时自启动,在容器退出时总是重启容器
-unless-stopped，在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器

查看/删除镜像
docker images
docker rmi 镜像ID

删除所有空标签的镜像
docker images|sed "1 d"|grep "<none>" |awk '{print $3}' |xargs docker rmi

查看所有容器状态(容器ID:CONTAINER ID)：
docker ps -a

查看容器内存
docker stats
docker stats --no-stream 容器ID
docker top

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

下载复制docker容器内文件到宿主机(docker ps -a查看容器id)：
docker cp 容器ID:/etc/profile.d/env.sh /home/soft/download

执行某个容器内的命令：
docker exec -it jenkins-web cat /var/jenkins_home/secrets/initialAdminPassword

进入某个容器1：
docker exec -it gitlab bash
docker exec -it -u root jenkins bash (指定root用户)

进入某个容器2：
先查找出容器的进程PID，再用系统命令nsenter连接即可，连上后就如同操作linux：
使用
docker top redis-web
或者使用(.State.Pid)
docker inspect -f "{{  _State_Pid  }}" redis-web
sudo nsenter -t 进程PID -m -u -i -n -p

推送本地镜像到阿里云[从ECS内网服务器上推送不会耗公网流量开头使用registry-vpc]：

sudo docker login --username=用户名 registry.cn-xxx.aliyuncs.com
sudo docker tag [ImageId] registry.cn-xxx.aliyuncs.com/xxx/mysql:[镜像版本号]
sudo docker push registry.cn-xxx.aliyuncs.com/xxx/mysql:[镜像版本号]

```

#### 让本地docker可运行多平台系统的容器
```
1. 修改/etc/docker/daemon.json配置experimental:true(开启Docker Daemon的实验功能)后重启Docker
2. 宿主> export DOCKER_CLI_EXPERIMENTAL=enabled(开启Docker Client的实验功能)
3. 宿主> docker version (查看实验功能是否开启,Experimental值是否为true)
4. 测试是否可运行arm64容器(本地为x64系统,未开启实验功能前无法运行不同平台系统容器)
	宿主> docker pull --platform arm64 alpine:3.10
	宿主> docker run --rm -it alpine:3.10 sh
```

#### docker buildx 打包成多平台系统容器
```
下载(mac系统选择darwin-amd64): https://github.com/docker/buildx/releases/tag/v0.6.3

# 重命名文件为docker-buildx并放置对应目录及赋权限:
>  mv ~/tmp/buildx-v0.6.3.darwin-amd64 ~/.docker/cli-plugins/docker-buildx
>  chmod 700 docker-buildx

后续就可以参考 https://github.com/klo2k/nexus3-docker 编译不同平台
或
https://segmentfault.com/a/1190000021529637

> docker buildx build --pull \
  --platform "linux/arm64" \
  --tag "klo2k/nexus3" \
  --output=type=docker \
  .

```

#### docker apt更新国内源

```
容器 > cat /etc/apt/sources.list 复制获取到的当前容器sources.list例如：
deb http://deb.debian.org/debian stretch main
deb http://security.debian.org/debian-security stretch/updates main
deb http://deb.debian.org/debian stretch-updates main

容器 > mv /etc/apt/sources.list /etc/apt/sources.list.bak #备份

本机 > vim aliyun.list # 替换原来deb.debian.org和security.debian.org为mirrors.aliyun.com例如：
deb http://mirrors.aliyun.com/debian stretch main
#deb http://security.debian.org/debian-security stretch/updates main
deb http://mirrors.aliyun.com/debian stretch-updates main

本机 > docker cp aliyun.list 容器ID:/etc/apt/sources.list.d/aliyun.list
容器 > apt-get clean
容器 > apt-get update

容器 > apt-get -y install vim #可以自由安装所需
```

### 使用Dockerfile创建docker镜像

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

### Kubernetes相关

#### kubectl命令(管理Kubernetes集群的工具https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG.md)
```
查看客户端和服务端kubectl的版本
> kubectl version

显示所有已创建的容器
> kubectl get pods

创建容器根据yaml：
> kubectl create -f /tmp/my-app.yaml

创建容器根据镜像（不能使用registry-vpc内网地址需使用公网地址）
> kubectl run my-app --image registry.cn-huhehaote.aliyuncs.com/mytest/my-app:0.01

查看是否创建容器成功(会一直阻塞直到状态成功)
> kubectl rollout status deployment/my-app

进入容器：
> kubectl exec -it my-app-5f9dx4fb --container my-app -- /bin/sh

创建网络服务（可以手动在阿里云创建，不用删除）
> kubectl expose deployment my-app --port=7070 --target-port=8080 --name=my-app-svc --type=LoadBalancer

删除my-app所有包括网络服务
> kubectl delete deployment my-app

只删除my-app容器，不包括服务
> kubectl delete deployments/my-app

删除my-app容器和网络访问服务
> kubectl delete deployments/my-app services/my-app-svc

```

