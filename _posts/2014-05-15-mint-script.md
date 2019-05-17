---
layout: post
title: Mint各种小脚本
category: [Linux-Mint,自动更新hosts,清理arp,防御arp,脚本,Rinetd,端口转发,杀死进程,钉钉机器人,ss-libev,crontab,虚拟机,virtualbox]
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

> sudo apt-get install software-properties-common -y
> sudo add-apt-repository ppa:max-c-lv/shadowsocks-libev -y
> sudo apt-get update
> sudo apt install shadowsocks-libev

2】https://github.com/shadowsocks/simple-obfs

> sudo apt-get install --no-install-recommends build-essential autoconf libtool libssl-dev libpcre3-dev libev-dev asciidoc xmlto automake
> git clone https://github.com/shadowsocks/simple-obfs.git
> cd simple-obfs
> git submodule update --init --recursive
> ./autogen.sh
> ./configure && make
> sudo make install

3】http://shadowsocks.org/en/config/quick-guide.html

> vim /etc/shadowsocks-libev/config_obfs.json(obfs方式)：
{
    "server":"xx.xxx.com",
    "server_port":15376,
    "local_port":8087,
    "password":"xxx",
    "timeout":8,
    "method":"aes-256-cfb",
    "plugin":"obfs-local",
    "plugin_opts":"obfs=http;obfs-host=bing.com"
}

> vim /etc/shadowsocks-libev/config.json（原始方式）：
{
    "server":"xx.xxx.com",
    "server_port":15376,
    "local_port":8087,
    "password":"xxx",
    "timeout":8,
    "method":"aes-256-cfb"
}

4】Usage 启动：

(默认ss-local命令会使用配置/etc/shadowsocks-libev/config.json)
> ss-local 

(配置文件已经有plugin和plugin_opts选项)
> ss-local -c /data/my-config.json 

(配置文件没有plugin和plugin_opts选项)
> ss-local -c /data/my-config.json --plugin obfs-local --plugin-opts "obfs=http;obfs-host=www.bing.com" 

(使用双配置脚本,加任意一个参数即可切换到原始方式)
> vim ~/bin/sds.sh
#!/bin/sh

cfg="/etc/shadowsocks-libev/config_obfs.json"
if [ $# -eq 1 ];then

echo "use config.json..."
cfg="/etc/shadowsocks-libev/config.json"

else

echo "use config_obfs.json..."

fi

ss-local -c $cfg

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

####  LinuxMint下安装虚拟机virtualbox

```
更多参考：https://www.virtualbox.org/wiki/Linux_Downloads
先清理其它vm版本以免冲突出错The VirtualBox kernel modules do not match this version of VirtualBox：
sudo apt-get autoremove open-vm-tools open-vm-tools-desktop open-vm-tools-dkms --purge

vim /etc/apt/sources.list
deb https://download.virtualbox.org/virtualbox/debian xenial contrib
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
sudo apt-get update
sudo apt-get install virtualbox-5.2

重启快捷键系统进入bois设置Secure Boot set disable
（没有出现grub界面的先设置等待选择时间
vim /etc/default/grub
GRUB_DEFAULT=0
#GRUB_HIDDEN_TIMEOUT=0
#GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=10
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""
）
按F2进入bois(有些直接在grub选择system setup)
function key behavior的multimedia key first,改成function key first
修改不了的时候先设置supervisor密码
BIOS set supervisor password save && 重启
重新设置Secure Boot set disable保存重启

进入界面下载添加xp.iso纯净版：
http://xxx.74.xdowns.com/WindowsXP-SP3-VOLMSDN.rar
MRX3F-47B9T-2487J-KWKMF-RPWBY

远程计算机需要网络级别身份验证错误：
1	自动升级系统到最新SP3补丁包
2	regedit -> HKEY_LOCAL_MACHINE＼SYSTEM＼CurrentControlSet＼Control＼Lsa”，编辑Security Packages列表增加tspkg字符　　 
4	regedit -> HKEY_LOCAL_MACHINE＼SYSTEM＼CurrentControlSet＼Control＼SecurityProviders”，编辑SecurityProviders在末端中添加, credssp.dll（注意空格）　 
5	重启系统后重新远程连接mstsc即可

```

#### 终端颜色

```
> vim ~/.bashrc
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi
export PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

export JAVA_HOME=/data/jdk1.8.0_181
```




