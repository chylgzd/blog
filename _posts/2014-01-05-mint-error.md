---
layout: post
title: Mint各种问题
category: [Linux-Mint,Mint各种问题,乱码,ssh暴力破解]
comments: false
---

* content
{:toc}

#### su root 出现 Authentication failure
```
原因没有创建过root用户直接设置密码即可
> sudo passwd root
```

#### 系统没有声音的解决方法
```
pulseaudio --kill
```

#### 系统时间不正确的解决方法
```
查看ntpdate时间同步工具是否有安装，如果没有则：
sudo apt-get install ntpdate

设置系统时间与网络时间同步：
sudo ntpdate cn.pool.ntp.org

将系统时间写入硬件时间：
sudo hwclock --systohc

查看系统时间命令：
date

设置系统时间命令：
date –set "10/11/10 10:15"

查看硬件时间命令：
sudo hwclock

设置硬件时间命令：
sudo hwclock –set –date = （月/日/年 时：分：秒）
```

#### sudo apt-get update: Hash Sum mismatch
```
切换快一点的源或者下次网络好的时候重试或者参考以下：

sudo gedit /etc/apt/apt.conf.d/00aptitude
添加如下内容：

Aptitude::Get-Root-Command "sudo:/usr/bin/sudo";
Acquire::CompressionTypes::Order "gz";
Acquire::http::No-Cache "true";
Acquire::http::Max-Age "0";

Debug::Acquire::http false;
Debug::pkgAcquire::Auth false;
Debug::Hashs false;

删除缓存：
sudo rm –rf /var/lib/apt/lists/*
sudo apt-get clean

然后更新：
sudo apt-get update
或者
sudo apt-get update -o Acquire::No-Cache=True
```

#### dpkg: error: dpkg status database is locked by another process
```
执行
sudo dpkg -i xxx.deb
后如果出现此问题的解决办法：
sudo rm -rf /var/lib/dpkg/lock
```

#### NO_PUBKEY
```
apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 3B4FE6ACC0B21F32

如果是 NO_PUBKEY 6494C6D6997C215E Chrome fix：
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
sudo apt-get update
```

#### 硬件模块列表
```
rfkill list all
```

#### 强制禁用蓝牙
```
> sudo xed /etc/rc.local
rfkill block bluetooth
```

#### vim 乱码问题
```
vim ~/.vimrc （没有就新建）加入：

set fileencodings=utf-8,gb2312,gb18030,gbk,ucs-bom,cp936,latin1
set enc=utf8
set fencs=utf8,gbk,gb2312,gb18030

```

#### xed文本编辑器乱码问题
```
> sudo apt install dconf-editor
> dconf-editor

展开org->x->editor->preferences->encodings
设置auto-detected:
['GB18030', 'UTF-8', 'CURRENT', 'ISO-8859-15', 'UTF-16']

```

#### zsh: no matches found
```
去掉zsh自动匹配：
> vim ~/.zshrc
setopt no_nomatch

> source ~/.zshrc

```

#### 重新开启网络连接通知 re-enable network notification
```
> gsettings set org.gnome.nm-applet disable-connected-notifications false
> gsettings set org.gnome.nm-applet disable-disconnected-notifications false
> gsettings set org.gnome.nm-applet disable-vpn-notifications false

```

#### ssh远程登陆很慢的问题
```
首先调试连接，看问题慢在哪一行：
> ssh -v root@112.124.200.228

如果停留在debug1: Next authentication method: gssapi-with-mic很慢添加DNS
> vim /etc/resolv.conf
nameserver 223.5.5.5
nameserver 223.6.6.6

```

#### 乱码locale: Cannot set LC_CTYPE/LC_ALL to default locale: No such file or directory
```
> locale -a | grep zh 没有中文zh

ubuntu系统下远程登录命令控制台输入中文乱码的解决方法：
> vim /etc/environment 
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8

```

#### 缺少相应字体JAVA程序生成图片中文字乱码
```
添加相应字体到该目录，比如黑体simhei.ttf:
/usr/local/share/fonts/simhei.ttf

刷新字体缓存
sudo fc-cache -fv /usr/local/share/fonts

```


#### Last failed login:xx from [ip] on ssh:notty,There were xx failed login attempts...
```
查看登陆失败日志,可能被暴力破解登陆中：
> lastb

解决方法1：
记录下ip地址加入限制ip列表(lastb查询所有登录未成功的ip)：
> vim /etc/hosts.deny
sshd:192.168.0.123

sshd:192.168.0.123,sshd:192.168.0.124

sshd:192.168.*.*

sshd:all 或 sshd:all:deny
(sshd:ALL代表禁止所有IP登录，慎用)

(
 也可以使用/etc/hosts.allow里添加 sshd:192.168.0.123:allow或sshd:192.168.0.123:deny，
 注：当hosts.allow和 host.deny相冲突时，以hosts.allow设置为准
)

修改完毕重启xinetd:
> service xinetd restart

解决方法2：
> vim /etc/ssh/sshd_config
#禁用空密码
PermitEmptyPasswords no

#禁用密码认证方式
PasswordAuthentication no

#启用密钥验证（即免密登陆方式,一般自动支持可以忽略,如果不能免密登陆再尝试下配置）
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# 修改Linux服务端默认ssh端口22为其它端口(可用范围1024 - 65535,系统内端口范围1-1024)
Port 43861

#禁用root登陆（改用新建远程用户
				>useradd testusr
				>passwd testusr
				>visudo (使用visudo命令允许此用户能够以root用户身份运行命令)
				......
				root ALL=(ALL) ALL
				testusr ALL=(ALL) NOPASSWD:ALL
				再切换到root方式，一般上述几个有效后该项可以不用
		）
PermitRootLogin no

> systemctl restart sshd
or
> service sshd restart
or
> service ssh restart

本地客户端相应修改后ssh命令就不用带端口号了
> vim ~/.ssh/config
host git.mytest.com
port 10022

host 192.168.10.11
port 43861

```




