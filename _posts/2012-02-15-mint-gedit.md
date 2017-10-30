---
layout: post
title: 安装gedit2插件及gedit3相关配置
category: [Linux-Mint,gedit2,gedit3]
comments: false
---

* content
{:toc}

### 如果重启后恢复原来的默认配置，可做下修正(gedit2)：
```bash
sudo chown -R youruser:youruser ~/.gconf/apps/gedit-2
```

### 安装插件集 gedit-plugins(gedit2)
```bash
sudo apt-get install gedit-plugins
```

#### 安装完插件后启用如下(gedit2)：

字体Noto Sans Armenian10

* 代码注释 (Ctrl+M)

* 单词补全

* 显示行号

* 括号补全

* 突出显示当前行

* 嵌入式终端（可以在查看-打开底面板）

* 模式行

* 片段

* 文档统计

* 文件浏览器面板（可以在查看-侧边栏）

#### 自定义样式设置(gedit2)( 下载 [myEclipseStyle.xml](/blog/download/gedit/styles/myEclipseStyle.xml)  )

```
~/.gnome2/gedit目录权限要更改为当前用户chown -R yourname:yourname ~/.gnome2/gedit

检查styles文件夹是否存在不存在则创建：

mkdir -p ~/.gnome2/gedit/styles

cp ~/下载/myEclipseStyle.xml ~/.gnome2/gedit/styles/

更多样式参考 https://github.com/mig/gedit-themes
```

#### 自定义外部工具(gedit2)( 下载 [electron.sh](/blog/download/gedit/tools/electron)  [node.sh](/blog/download/gedit/tools/node)  )

```

检查tools文件夹是否存在不存在则创建：

mkdir -p ~/.gnome2/gedit/tools

cp ~/下载/electron ~/下载/node ~/.gnome2/gedit/tools/

```

##### 如果需要更改快捷键可参考(gedit2)(实测并无作用)：

```
sudo chown yourname:yourname ~/.gnome2/accels/gedit
gsettings set org.gnome.desktop.interface can-change-accels true

gedit ~/.gnome2/accels/gedit
; (gtk_accel_path "<Actions>/CodeCommentActions/CodeUncomment" "<Primary><Shift>/")
; (gtk_accel_path "<Actions>/CodeCommentActions/CodeComment" "<Primary>/")
sudo chown root:root ~/.gnome2/accels/gedit
gsettings set org.gnome.desktop.interface can-change-accels false
```

#### 其中代码片断在目录/usr/share/gedit-2/plugins/snippets/目录下需要更改权限才能保存片断中的设置(gedit2)

```bash
sudo chown youruser:youruser /usr/share/gedit-2/plugins/snippets/*.xml
```

#### 以下是备份的一份html.xml代码片断设置(gedit2)：

[html.xml](/blog/download/gedit/snippets/html.xml)

[javascript.xml](/blog/download/gedit/snippets/javascript.xml)


#### 安装最新版本(gedit2)

```
1.sudo apt-add-repository ppa:suraia/ppa 

2.sudo apt-get update 

3.sudo apt-get remove gedit && sudo apt-get install gedit 

4.sudo apt-add-repository -r ppa:suraia/ppa then REBOOT 

5.gedit -V you should see the version of gedit.
```

#### 语法高亮设置(gedit2)

```
让模板引擎使用html语法高亮：

sudo gedit /usr/share/gtksourceview-2.0/language-specs/html.lang

修改globs中的值：
<property name="globs">*.html;*.htm;*.jsp;*.vue;*.ejs;*.vm;*.ftl;*.jade</property>
```


### gedit3相关(Linux Mint18)

```
代码片断和外部工具文件夹：
~/.config/gedit/snippets
~/.config/gedit/tools

语法高亮文件夹：
/usr/share/gtksourceview-3.0/language-specs
~/.local/share/gtksourceview-3.0/language-specs
```

#### 代码片断
```
version:3.18.3
打开gedit3，tools -> Manage Snippets可以修改和删除一些片断，更新后的文件在该目录下：
~/.config/gedit/snippets/
```

#####   ~/.config/gedit/snippets/html.xml备份(gedit3)

```xml
<?xml version='1.0' encoding='utf-8'?>
<snippets language="html">
  <snippet override="html-form">
    <description>Form</description>
    <text><![CDATA[<form action="${1:/test.do}" method="${2:[get,post]}" enctype="${3:[application/x-www-form-urlencoded,multipart/form-data]}">
	<p><input id="name" value="test" /></p>
	<p><input id="age" value="17" /></p>
	<p><input type="submit" value="提交" /></p>
</form>]]></text>
    <tag>form</tag>
  </snippet>
  <snippet>
    <description>HTML5</description>
    <tag>html</tag>
    <text><![CDATA[<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8"/>
	<title>test</title>
	<meta name="keywords" content=""/>
	<meta name="description" content=""/>
	<meta name="viewport" content="width=device-width"/>
</head>
<body>
	$0
</body>
</html>]]></text>
  </snippet>
  <snippet override="html-link">
    <description>Link</description>
    <text><![CDATA[<link rel="${1:[stylesheet,shortcut icon]}" href="${2:/demo.css}" type="${3:[text/css,image/x-icon]}">
$0]]></text>
    <tag>link</tag>
  </snippet>
  <snippet override="html-meta">
    <description>Meta</description>
    <text><![CDATA[<meta name="${1:[viewport,description,keywords]}" content="${2:[width=device-width]}" />
$0]]></text>
    <tag>meta</tag>
  </snippet>
  <snippet override="html-scriptsrc">
    <description>Script With External Source</description>
    <text><![CDATA[<script src="${1:resource/js/main.js}" type="text/javascript"></script>]]></text>
    <tag>scripts</tag>
  </snippet>
  <snippet>
    <description>JS-Function</description>
    <tag>function</tag>
    <text><![CDATA[function ${1:fuc_name}(){
	$0
}]]></text>
  </snippet>
  <snippet>
    <description>JS-Log</description>
    <tag>log</tag>
    <text><![CDATA[console.log('${1:hello}');
$0]]></text>
  </snippet>
</snippets>
```

#####   ~/.config/gedit/snippets/js.xml备份(gedit3)

```xml
<?xml version='1.0' encoding='utf-8'?>
<snippets language="js">
  <snippet override="js-fun">
    <description>JS-Function</description>
    <tag>function</tag>
    <text><![CDATA[function ${1:fuc_name}(){
	$0
}]]></text>
  </snippet>
  <snippet>
    <description>JS-Log</description>
    <tag>log</tag>
    <text><![CDATA[console.log('${1:hello}');$0]]></text>
  </snippet>
  <snippet>
    <description>JS-Alert</description>
    <tag>alert</tag>
    <text><![CDATA[alert('${1:hello}');$0]]></text>
  </snippet>
</snippets>
```

#### gedit3扩展工具(gedit3)(更多参考：[ExternalTools](https://wiki.gnome.org/Apps/Gedit/Plugins/ExternalTools) )

```
version:3.18.3
打开gedit3，tools -> Manage External Tools可以修改删除添加一些外部工具，更新后的文件在该目录下：
~/.config/gedit/tools/
```

#####   ~/.config/gedit/tools/node.js备份(gedit3)

```bash
#!/bin/sh
# [Gedit Tool]
# Applicability=local
# Save-files=nothing
# Input=nothing
# Languages=js
# Name=node.js
# Shortcut=<Alt>n
# Output=output-panel

echo "exec => node ${GEDIT_CURRENT_DOCUMENT_PATH}"
node $GEDIT_CURRENT_DOCUMENT_PATH
```

#####   ~/.config/gedit/tools/golang备份(gedit3)

```bash
#!/bin/sh
# [Gedit Tool]
# Save-files=nothing
# Languages=go
# Name=golang
# Applicability=local
# Input=nothing
# Shortcut=<Alt>g
# Output=output-panel

echo "exec => go run ${GEDIT_CURRENT_DOCUMENT_PATH}"
go run $GEDIT_CURRENT_DOCUMENT_PATH
```

#####   ~/.config/gedit/tools/electron备份(gedit3)

```bash
#!/bin/sh
# [Gedit Tool]
# Languages=html
# Input=nothing
# Name=electron
# Output=nothing
# Shortcut=<Alt>e
# Save-files=nothing
# Applicability=local


cd $GEDIT_CURRENT_DOCUMENT_DIR
electron .
```

#### gedit3语法高亮

```
默认文件夹
/usr/share/gtksourceview-3.0/language-specs
用户自定义文件夹
~/.local/share/gtksourceview-3.0/language-specs

修改默认语法高亮关联后缀，比如：

html.lang：
cp /usr/share/gtksourceview-3.0/language-specs/html.lang ~/.local/share/gtksourceview-3.0/language-specs/
<property name="globs">*.html;*.htm;*.jsp;*.vue;*.ejs;*.vm;*.ftl;*.jade</property>

css.lang：
cp /usr/share/gtksourceview-3.0/language-specs/css.lang ~/.local/share/gtksourceview-3.0/language-specs/
<property name="globs">*.css;*.CSSL;*.scss;*.less</property>
```

#### gedit 乱码问题
```
gsettings set org.gnome.gedit.preferences.encodings auto-detected "['UTF-8', 'GB18030', 'GB2312', 'GBK', 'BIG5', 'CURRENT', 'UTF-16']"
gsettings set org.gnome.gedit.preferences.encodings shown-in-menu "['UTF-8', 'GB18030', 'GB2312', 'GBK', 'BIG5', 'CURRENT', 'UTF-16']"

如果提示No such key 'auto-detected' 则用dconf-editor打开路径是不是有一个candidate-encodings
gsettings set org.gnome.gedit.preferences.encodings candidate-encodings "['UTF-8', 'GB18030', 'GB2312', 'GBK', 'BIG5', 'CURRENT', 'UTF-16']"

```

