---
layout: post
title: Mint各种小脚本
category: [Linux-Mint,自动更新hosts,清理arp,防御arp,脚本,Rinetd,端口转发,杀死进程,钉钉机器人,ss-libev,crontab,虚拟机,virtualbox,远程终端颜色]
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

#### 远程终端颜色别名

```
> vim /etc/hostname
my-prod-web

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

#### 端口相关

##### 端口开放
```
# 查看已开放的端口
> /sbin/iptables -L -n
# 查看防火墙状态
> service iptables status
# 安装
> yum install iptables-services
# 启动
> service iptables start
# 重启
> service iptables restart
# 开放某个端口(比如80,需重启iptables)
> vim /etc/sysconfig/iptables
# sample configuration for iptables service
# you can edit this manually or use system-config-firewall
# please do not ask us to add additional ports/services to this default configuration
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT

```

##### 转发端口
```
1.(正向代理-从本地访问公网跳板机内非公开的局域网机器)将本地3307端口代理目标内网192.168.0.2的3306端口通过跳板机x.x.x.x
local > ssh -fCNgL 3307:192.168.0.2:3306 root-remote-jump-public@x.x.x.x
local > telnet localhost 3307#从内网访问远程非公开的机器(等于访问远程3306端口)
#nohup
local > nohup ssh -fCNgL 3307:192.168.0.2:3306 xx@x.x.x.x >/dev/null 2>&1 &
#autossh(-M参数指定监听端口可断线自动重连)
local > yum install autossh
local > nohup autossh -M 4010 -fCNgL 3307:192.168.0.2:3306 xx@x.x.x.x >/dev/null 2>&1 &

2.(反向代理-从公网机器穿墙访问本地内网机器)将本地3306端口转发到公网服务器的7788端口上
local > ssh -fCNR 7788:localhost:3306 root-remote-public@x.x.x.x
remote> lsof -i:7788 #查看是否绑定
remote> telnet localhost 7788 #从公网服务访问内网(等于访问内网3306端口)

ssh参数：
-f 后台执行ssh指令
-C 允许压缩数据
-N 不执行远程指令
-R 将远程主机(服务器)的某个端口转发到本地端指定机器的指定端口
-L 将本地机(客户机)的某个端口转发到远端指定机器的指定端口
-p 指定远程主机的端口

(停止所有ssh进程)
local> pgrep ssh | xargs kill

(从公网机器穿墙访问本地内网机器)将局域内网主机A的4000端口映射至公网主机B的80端口上
local-A>yum install autossh
local-A>autossh -M 4010 -NR 80:localhost:4000 root-B@x.x.x.x (-p 22)
'-M 4010':使用内网主机的4010端口监视SSH连接状态自动重连
'-N':不执行远程命令
'-R':将远程公网机的某个端口转发到本地指定机器的指定端口

#停止代理:
(停止所有ssh进程)
local> pgrep ssh | xargs kill
(只停止查询到的对应进程)
local> ps -ef |grep 3307:192.168.0.2:3306 | grep -v grep |awk '{print $2}'|xargs kill -9 1>/dev/null 2>&1 

#转发本地端口到远程
local> ssh -L local-port:target-host:target-port tunnel-host
#转发远程端口到本地
local> ssh -R remote-port:target-host:target-port -N remotehost

案例一：
1,从公网服务器穿透到本地机器:
将本地22端口转发到公网服务器 (remote@IP) 的 7788 端口上
local	> ssh -fNTR 7788:localhost:22 remote@IP
local	> ssh remote@IP
remote> ssh -p 7788 本地机器用户名@localhost
remote> vim ~/.ssh/config
Host localmac
  User 本地机器用户名
  Port 7788
  HostName localhost
remote> ssh localmac

2,从本地转发公网服务器
把本地8080端口转发到REMOTE-IP这台机器上的8888端口
local> ssh -CfNg -L 8080:REMOTE-IP:8888 REMOTE-IP


案例二：
本地配置跳板机，登录跳板机后面的内网机器
local > vim ~/.ssh/config
Host tiaoban
	HostName x.x.x.x (跳板机公网IP)
	Port 22
	User u_tiaoban
Host lan-ecs1
	HostName 192.168.0.11 (内网IP)
	Port 22
	User u_lan-ecs1
	ProxyCommand ssh u_tiaoban@tiaoban -W %h:%p
Host lan-ecs2
	HostName 192.168.0.12 (内网IP)
	Port 22
	User u_lan-ecs2
	ProxyCommand ssh u_tiaoban@tiaoban -W %h:%p
	
local > ssh lan-ecs1
local > ssh lan-ecs2



案例三：
借助远程vps让两台不能直接相通的机器相互能访问。
有主机vps和主机A、B。A、B无法直连，通过“中介”搭桥相连。（两台机器都能主动ssh到vps就能完成。）
A要ssh到B（B要ssh到A是同理）：
1、主机B用ssh远程转发自己的2222端口到vps的127.0.0.1:12222
B> ssh -NfR 12222:127.0.0.1:2222 user@vps -p2222

2、主机A用ssh本地转发vps的127.0.0.1:12222到本地的127.0.0.1:12222
A> ssh -NfL 12222:127.0.0.1:12222 user@vps -p2222

3、主机A登录主机B
A> ssh user@localhost -p12222



案例四：
如果2222端口被封，如果绕过封死2222端口的防火墙直接ssh到内网机器。（就是说限某几个端口是有局限的）
1、登录最重要的机器把2222端口映射到12222端口：
  ssh -gfNL 12222:0.0.0.0:2222 localhost -p2222
2、使用该机器做隧道代理访问其他内网机器：
  ssh -NfD 10000 user@host -p12222
3、ssh绕道访问其他内网机器：
  ssh -o "ProxyCommand=nc -x localhost:10000 %h %p" user@host -p2222

```

#### 账户相关

```
#添加用户
> sudo useradd -m jenkins
> passwd jenkins
> sudo usermod -s /bin/bash demo (ubuntu)

# 添加只读用户
> useradd -s /bin/bash readonly
> passwd readonly
> mkdir /home/readonly/.bin
	> vim /home/readonly/.bash_profile
		修改PATH路径 PATH=$HOME/.bin
> mkdir /home/readonly/.ssh (只允许ssh登录时需要)
	> touch /home/readonly/.ssh/authorized_keys
	> chown -R readonly:readonly /home/readonly/.ssh
## root修改用户的shell配置文件
> chown root. /home/readonly/.bash_profile 
> chmod 755 /home/readonly/.bash_profile
> chattr -i /home/readonly/.bash_profile
## (可选,让只读用户执行命令)将允许执行的命令链接到$HOME/.bin目录
ln -s /usr/bin/wc  /home/readonly/.bin/wc
ln -s /usr/bin/tail  /home/readonly/.bin/tail
ln -s /bin/more  /home/readonly/.bin/more
ln -s /bin/cat  /home/readonly/.bin/cat
ln -s /bin/grep  /home/readonly/.bin/grep
ln -s /bin/find  /home/readonly/.bin/find
ln -s /bin/pwd  /home/readonly/.bin/pwd
ln -s /bin/ls  /home/readonly/.bin/ls
ln -s /usr/bin/less /home/readonly/.bin/less
ln -s /bin/tar  /home/readonly/.bin/tar
## (可选)切换到只读账号使环境变量生效
> su - readonly
> source /home/readonly/.bash_profile
或
> ssh readonly@IP

#添加非登录用户
sudo useradd -r -s /sbin/nologin tomcat

# 查看系统所有用户
> cut -d: -f1 /etc/passwd 
> cut -d: -f1 /etc/passwd | grep xxx
# 查看系统所有组
> cut -d: -f1 /etc/group 

# 查看系统当前已登录的用户列表
> w
或
> who
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    121.234.185.39  12:12    2.01m  0.01s  0.00s w
root     pts/1    101.100.22.102  11:14    2:14m  0.01s  0.01s -bash

# 踢出其它用户
> pkill -kill -t pts/1

# 只允许root登录
root> touch /etc/nologin 
root> echo "Systems are wealth maintenance, 1 hour after the landing." > /etc/nologin
```

##### 禁止账户操作
```
## 禁止用户访问某个目录或权限(设置ACL规则,r读x执行w写-则表示没有任何权限)
> setfacl -R -m u:readonly:- /bin
> setfacl -R -m u:readonly:r /home/readonly/.ssh
> cd /bin
> setfacl -m u:readonly:- mv
> setfacl -m u:readonly:- rm
> setfacl -m u:readonly:- vim
> setfacl -m u:readonly:- vi
> setfacl -m u:readonly:- ls
> setfacl -m u:readonly:- find
> setfacl -m u:readonly:- tail
> setfacl -m u:readonly:- more
> setfacl -m u:readonly:- less
> setfacl -m u:readonly:- cat
> setfacl -m u:readonly:- wget
> setfacl -m u:readonly:- curl
> setfacl -m u:readonly:- tar
> setfacl -m u:readonly:- unzip
> setfacl -m u:readonly:- zip

## 禁止用户访问某个目录下的部分文件权限(禁止查看.yml和.sh脚本等重要配置)
> find /usr/local -type f -regex ".*\.\(yml\|sh\|conf\|xml\|properties\)" | xargs setfacl -m u:readonly:-

## 允许用户访问某个目录或权限(r读x执行w写-则表示没有任何权限)
> setfacl -R -m u:readonly:rx /bin
> setfacl -m u:readonly:rx /bin/vim
> setfacl -R -m u:readonly:r /home/readonly/.ssh

## 查看某个目录或文件的ACL规则
> getfacl /bin
> getfacl /bin/vim

## 删除某个目录或文件的ACL规则
> setfacl -b /bin
> setfacl -b /bin/vim
```

