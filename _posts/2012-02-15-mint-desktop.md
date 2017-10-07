---
layout: post
title: Mint下为eclipse和其类似或其它sh运行程序添加桌面快捷方式
category: [Linux-Mint,Desktop,桌面快捷图标]
comments: false
---

* content
{:toc}

桌面快捷方式存放目录在

~/.local/share/applications/

或者

/usr/share/applications/

### 新建eclipse.desktop

gedit ~/.local/share/applications/eclipse.desktop
写入如下内容：

```
[Desktop Entry]
Encoding=UTF-8
Name=eclipse
GenericName=Eclipse IDE
Comment=Eclipse IDE
Exec=/home/yourname/_ain/eclipse/eclipse
Terminal=false
Icon=/home/yourname/_ain/eclipse/icon.xpm
Type=Application
Categories=Application;Eclipse;
Version=1.0
```

（如果图片是ico可以在线转化成png
https://iconverticons.com/online）

编辑完后在系统左下角菜单出现[其它]选项，右键里面的图标添加到桌面即可.

# 其它的一些例子：
gedit ~/.local/share/applications/AndroidStudio.desktop

```
[Desktop Entry]
Encoding=UTF-8
Name=AndroidStudio
GenericName=android-studio
Comment=android-studio
Exec=/home/yourname/_ain/android/android-studio/bin/studio.sh
Terminal=false
Icon=/home/yourname/_ain/android/android-studio/bin/studio.png
Type=Application
Categories=Application;AndroidStudio;
Version=1.0
```

gedit ~/.local/share/applications/heidisql.desktop

```
[Desktop Entry]
Encoding=UTF-8
Name=Heidisql
GenericName=Heidisql
Comment=Heidisql
Exec=/usr/db/tools/HeidiSQL/heidisql.exe
Terminal=false
Icon=/usr/db/tools/HeidiSQL/heidisql.png
Type=Application
Categories=Application;Heidisql;
Version=1.0
```

gedit ~/.local/share/applications/goagent.desktop

```
[Desktop Entry]
Encoding=UTF-8
Name=Goagent
GenericName=goagent
Comment=goagent
Exec=sh /home/yourname/soft/tool/goagent/gae.sh
Terminal=false
Icon=/home/yourname/soft/tool/goagent/goagent.png
Type=Application
Categories=Application;Goagent;
Version=1.0
```

（其中gae.sh，桌面添加成功后直接执行是没有命令窗口，可以右键桌面用终端模拟器打开）

```bash
#!/bin/sh
#
# goagent start script
#
#sudo /usr/bin/env python2.7 /home/xxx/soft/tool/goagent/local/proxy.py
python2 /home/yourname/soft/tool/goagent/local/proxy.py
#exit 0
```
