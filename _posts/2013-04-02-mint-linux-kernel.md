---
layout: post
title: Mint降级内核版本
category: [Linux-Mint,内核,Grub]
comments: false
---

* content
{:toc}

降级原因linux-image-4.4.0-21-generic发热高，而linux-image-3.19.0-32-generic发热低
更多参考[http://www.oldcai.com/archives/1026](http://www.oldcai.com/archives/1026)

### 查看内核版本列表
```
dpkg --get-selections|grep linux-image

结果示例：
linux-image-3.19.0-32-generic			install
linux-image-4.4.0-21-generic			install
linux-image-extra-3.19.0-32-generic		install
linux-image-extra-4.4.0-21-generic		install
```

### 删除发热高的内核版本
```bash
sudo apt-get purge linux-image-4.4.0-21-generic
```

### 重新修复grub启动菜单
```
查看grub启动菜单父选项：
grep menuentry /boot/grub/grub.cfg

查看grub启动菜单子选项：
grep submenu /boot/grub/grub.cfg

修改默认启动选项：
sudo gedit /etc/default/grub

(此步骤只是明确指定默认启动，非必须)将GRUB_DEFAULT=0 改为
GRUB_DEFAULT="Advanced options for Linux Mint 18 Xfce 64-bit>Linux Mint 18 Xfce 64-bit, with Linux 3.19.0-32-generic"

然后执行
sudo update-grub
此时如果有错误，则去掉GRUB_DEFAULT前面部分"Advanced options for Linux Mint 18 Xfce 64-bit>"后重新执行
没有什么错误提示，即可重启
重新连接上后，检查uname -a或者lsb_release -a，如果已经到目的版本，则说明成功了

```
