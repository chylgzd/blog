---
layout: post
title: WIn7下通过EasyBCD安装Ubuntu双系统
category: [Win7,EasyBCD,双系统]
comments: false
---

* content
{:toc}


### 安装前准备[下载EasyBCD](/blog/download/win/EasyBCD2.3.zip)

```
1.  压缩出一个空的磁盘分区以便安装Ubuntu系统，至少20G

2.  打开已经下载的iso文件，解压(一般在casper目录里找到)vmlinuz和initrd.lz至磁盘某个分区的根目录，并把iso文件也复制或剪切到这里

3.  启动EasyBCD  -> 添加新条目 -> NeoGrub - 点击安装 -> 点击配置
```

### 配置NeoGrub引导文件

```
title Install install-ubuntu-14.4
root (hd0,1)
kernel /vmlinuz boot=casper iso-scan/filename=/ubuntu-14.4-amd64.iso ro quiet splash locale=zh_CN.UTF-8
initrd /initrd.lz

title reboot
reboot

title halt
halt
```

### 配置文件说明

```
root (hd0,1)：
设置iso，vmlinuz和initrd三个文件所在磁盘分区，hd0表示在第一个磁盘，1表示在第二个分区索引从0(具体要看不同系统情况)
一般通过磁盘管理可以看到，或者cmd -> diskpart -> list volume

该配置文件会保存在C:\NST\menu.lst，以后如果有修改或者安装其它系统编辑该文件即可
```

### 重启开始安装

```
重启并进入ubuntu安装系统后，先在终端里执行该命令以取消掉对光盘所在驱动器的挂载：
sudo umount -l /isodevice

然后找到安装图标进行安装
```

### 注意事项

```
安装前在选择Install install-ubuntu-14.4后如果出现file not found，
可使用grub命令从索引0开始依次查找是否存在那三个文件，再修改启动配置文件重新选择安装即可;
ls (hd0,0)/
ls (hd0,1)/
ls (hd1,1)/....

安装过程开始前会检测到是否有其它系统，然后注意慎重选择全部格式化当前整个系统或是与其它系统并存(双系统);

安装中需要选择安装到之前压缩出的空分区，系统会自动格式化那块的数据并安装ubuntu系统到该分区；
```
