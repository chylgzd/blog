---
layout: post
title: Mint系统中安装其它字体
category: [Linux-Mint,字体]
comments: false
---

* content
{:toc}

字体下载[fontsquirrel.com](https://www.fontsquirrel.com)

终端字体推荐：[SourceCodePro-Light，10](https://www.fontsquirrel.com/fonts/download/source-code-pro)

文本编辑器字体推荐：[OpenSans-Regular，11](https://www.fontsquirrel.com/fonts/download/open-sans)

### 安装Courier New字体
```bash
sudo apt-get install ttf-mscorefonts-installer
它的本质是安装 Courier New字体
安装的时候会出现一个协议 
按TAB键 ，可以选中<确定>按钮（有些会看不到确定按钮），按Enter 。
```

### 安装source code pro字体

点击[下载](https://github.com/adobe-fonts/source-code-pro/archive/2.010R-ro/1.030R-it.tar.gz)，

```
下载后可以解压文件到scp目录（没有则创建mkdir /usr/share/fonts/opentype/scp）
或者
解压到~/.fonts（没有则创建mkdir -p ~/.fonts）目录

sudo tar -zxvf /下载目录/source-code-pro.tar.gz/otf/*.otf -C ~/.fonts/
或者
sudo tar -zxvf /下载目录/source-code-pro.tar.gz -C /usr/share/fonts/opentype/scp/

解压成功后执行字体缓存更新：
sudo fc-cache -f -v
```
