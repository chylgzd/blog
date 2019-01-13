---
layout: post
title: Mint各种问题
category: [Linux-Mint,Mint各种问题,乱码]
comments: false
---

* content
{:toc}

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

