---
layout: post
title: Mint各种小脚本
category: [Linux-Mint,自动更新hosts,清理arp,防御arp,脚本,Rinetd,端口转发,杀死进程,钉钉机器人,ss-libev,crontab]
comments: false
---

* content
{:toc}

更多可参考 https://github.com/racaljk/hosts

#### 创建和编辑脚本内容：

gedit ~/bin/host_tool.sh

```bash
#!/bin/sh
#
# script_tool_for_linux
#
# Use command: `sudo sh script_tool_for_linux.sh` or
#                `su -c 'sh script_tool_for_linux.sh'`
# to update your hosts file.
#
# WARNING: the script CAN NOT replace others' hosts rules.
#           If you have hosts rules provided by others, you may get conflict.
#
if [ `id -u` -eq 0 ]; then
    curl -fLo /tmp/fetchedhosts 'https://raw.githubusercontent.com/racaljk/hosts/master/hosts'
    sed -i '/# Copyright (c) 2014/,/# Modified hosts end/d' /etc/hosts

    sed -i "s/localhost/`hostname`/g" /tmp/fetchedhosts

    cat /tmp/fetchedhosts >> /etc/hosts
    rm -f /tmp/fetchedhosts
	
	# Modified self hosts
	sed -i -e '/# Modified hosts start/a\127.0.0.1	localhost\n127.0.0.1 xxx.test.com' /etc/hosts

	#clean dns and restart	
	sudo /etc/init.d/dns-clean start
	sudo /etc/init.d/networking restart
    echo 'Success.'
else
    echo 'Permission denied, are you root?'
fi
```

#### 保存后执行即可完成手动更新：

```
sudo ~/bin/host_tool.sh
```


#### 清理ARP脚本：

```bash
#!/bin/sh
sudo ip link set arp off dev eth0
sudo ip link set arp on dev eth0
sudo arp -s 192.168.0.1 5C:63:BF:35:B6:CC
```

#### 防御ARP攻击：

```
arp 查看
192.168.0.1              ether   5c:63:bf:35:b6:cc   C                    eth0

如果是CM而不是C说明已经绑定IP与MAC地址成功，否则进行如下步骤：
sudo -s
gedit /etc/ip-mac
写入arp查看到的结果（MAC地址大写）

more /etc/ip-mac
192.168.0.1 5C:63:BF:35:B6:CC

执行网关IP与MAC地址绑定（在sudo -s下操作，否则有权限问题）：
arp -f /etc/ip-mac

再次arp 查看
192.168.0.1              ether   5c:63:bf:35:b6:cc   CM                    eth0

（这只能防住一些arp攻击，如果将网内所有ip mac导入ip-mac文件，能有效的防止arp攻击
扫描局域网内所有，并导入ip-mac文件
nmap -sP 192.168.1.0/24）
```

#### 停止笔记本摄像头：

```
停止
sudo modprobe -r uvcvideo
启用
sudo modprobe uvcvideo
永久禁用
echo 'blacklist uvcvideo' | sudo tee -a /etc/modprobe.d/blacklist.conf
```

#### Rinetd 端口转发工具
```
http://www.ubuntugeek.com/rinetd-redirects-tcp-connections-from-one-ip-address-and-port-to-another.html

sudo apt install rinetd
sudo gedit  /etc/rinetd.conf
0.0.0.0 8080 10.1.1.2 7777
sudo /etc/init.d/rinetd restart

```

#### 杀死重启springboot进程，发送钉钉机器人消息，传入2个参数即可执行该脚本
```
function start_springboot(){
	/home/yourname/jdk1.8/bin/java -jar springboot-0.1.jar --spring.profiles.active=$1 --server.port=$2 --spring.config.location=file:./cfg/
}

function send_message(){
	#更多参考https://open-doc.dingtalk.com/docs/doc.htm?treeId=257&articleId=105735&docType=1
    curl -H "Content-Type: application/json;charset=UTF-8" -X POST -d '{"msgtype":"text","text":{"content":"'$1'"}}' https://oapi.dingtalk.com/robot/send?access_token=xxx 
}

function kill_existed_process(){
    rs=`ps -ax | awk '{ print $1 }' | grep -e "^$1$"`
    if [ -n "$rs" ]; then 
        echo "kill $2...$1"
        kill -9 $1
    fi 
    sleep 1
}

function kill_processes(){
    pid_list=`ps -ef|grep $1 | gawk '{print $2}'`
    for pid in $pid_list
    do 
        kill_existed_process $pid $1
    done 
}

kill_processes "springboot"

send_message "服务器被重启..."

start_springboot $1 $2

```

#### 更简单的杀死所有搜索到的进程方法
```
（grep -v grep排除自身查找的进程）

ps -ef |grep springboot | grep -v grep |awk '{print $2}'|xargs kill -9 1>/dev/null 2>&1

```

#### ss-libev + obfs-local：

```
1】https://github.com/shadowsocks/shadowsocks-libev#debian--ubuntu

sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:max-c-lv/shadowsocks-libev -y
sudo apt-get update
sudo apt install shadowsocks-libev

2】https://github.com/shadowsocks/simple-obfs

cd ~/programming/project/a.yitianjianss.com/
sudo apt-get install --no-install-recommends build-essential autoconf libtool libssl-dev libpcre3-dev libev-dev asciidoc xmlto automake
git clone https://github.com/shadowsocks/simple-obfs.git
cd simple-obfs
git submodule update --init --recursive
./autogen.sh
./configure && make
sudo make install

3】http://shadowsocks.org/en/config/quick-guide.html

~/programming/project/a.yitianjianss.com/yitianjianss-config.json：
{
    "server":"xx.xx.xx",
    "server_port":1234,
    "local_port":8087,
    "password":"abc123",
    "timeout":600,
    "method":"aes-256-cfb"
}

4】Usage

ss-local -c yitianjianss-config.json --plugin obfs-local --plugin-opts "obfs=http;obfs-host=xx.xx"
```

####  查看系统资源占用

```
大于100M的文件：
find / -size +100M -exec ls -lh {} \;

内存占用前10：
ps aux|head -1;ps aux|grep -v PID|sort -rn -k +4|head

CPU占用前10：
ps aux|head -1;ps aux|grep -v PID|sort -rn -k +3|head

```

####  sudo systemctl 免密码

```
sudo visudo

%wheel  ALL=(ALL)  ALL后面新增：
myuser  ALL=(ALL)  NOPASSWD: /usr/bin/systemctl

vim /etc/systemd/system/mytest.service：
[Unit]
Description=mytest
After=syslog.target

[Service]
User=myuser
Group=myuser
ExecStart=/data/jdk/bin/java -Dspring.profiles.active=prod  -jar /data/deploy/mytest-2.12.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target

sudo systemctl start/stop mytest.service

```

####  定时任务crontab脚本

```
vim /etc/crontab

59 23 * * * root /xx/shell/test.sh

/etc/rc.d/init.d/crond restart

crontab -l
crontab -e
```

####  切分日志文件脚本（配合crontab定时执行）

```
#!/bin/bash
thedate=`date --rfc-3339=date`
predate=`date +%Y-%m-%d --date="-7 day"`

rmfile="/xxx/xxx/apache-tomcat-8.5.33/logs/catalina-daemon.${predate}.out"
outfile="/xxx/xxx/apache-tomcat-8.5.33/logs/catalina-daemon.out"
if [ -f ${rmfile} ];then
   rm -f ${rmfile}
fi

if [ -f ${outfile} ];then
   cp ${outfile} /xxx/xxx/apache-tomcat-8.5.33/logs/catalina-daemon.${thedate}.out
   echo "" > ${outfile}
fi

exit 0
```




