---
layout: post
title: Mint中eclipse界面调整
category: [Linux-Mint,Gtk,Xface,Eclipse]
comments: false
---

* content
{:toc}

### 修改/home/yourname/目录下的.gtkrc-2.0文件为如下代码

```
include ".gtkrc-xfce"

style "eclipse" {  
	font_name="Noto Sans 8"  
	GtkButton::default_border={1,1,1,1} 
	GtkButton::default_outside_border={0,0,0,0} 
	GtkButtonBox::child_min_width=0 
	GtkButtonBox::child_min_heigth=0   
	GtkButtonBox::child_internal_pad_x=0    
	GtkButtonBox::child_internal_pad_y=0 
	GtkMenu::vertical-padding=1  
	GtkMenuBar::internal_padding=1  
	GtkMenuItem::horizontal_padding=4  
	GtkToolbar::internal-padding=1  
	GtkToolbar::space-size=1  
	GtkOptionMenu::indicator_size=0 
	GtkOptionMenu::indicator_spacing=0  
	GtkPaned::handle_size=4 
	GtkRange::trough_border=0   
	GtkRange::stepper_spacing=0 
	GtkScale::value_spacing=0   
	GtkScrolledWindow::scrollbar_spacing=0  
	GtkExpander::expander_size=10   
	GtkExpander::expander_spacing=0 
	GtkTreeView::vertical-separator=0   
	GtkTreeView::horizontal-separator=0 
	GtkTreeView::expander-size=10   
	GtkTreeView::fixed-height-mode=TRUE 
	GtkWidget::focus_padding=0  
}  
class "GtkWidget" style "eclipse"  
  
style "eclipse2" {  
	xthickness=1  
	ythickness=2  
} 
class "GtkButton" style "eclipse2"

style "eclipse3" {  
	xthickness=0  
	ythickness=0  
}  
class "GtkToolbar" style "eclipse3"  
class "GtkPaned" style "eclipse3"
```

### eclipse安装目录的eclipse.ini文件中--launcher前添加--launcher.GTK_version 2：
```ini
-startup
plugins/org.eclipse.equinox.launcher_1.3.100.v20150511-1540.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.gtk.linux.x86_64_1.1.300.v20150602-1417
-product
org.eclipse.epp.package.jee.product
--launcher.defaultAction
openFile
-showsplash
org.eclipse.platform
--launcher.XXMaxPermSize
256m
--launcher.defaultAction
openFile
--launcher.GTK_version
2
--launcher.appendVmargs
-vmargs
-Dosgi.requiredJavaVersion=1.7
-XX:MaxPermSize=256m
-Xms256m
-Xmx1024m
```

### 重启eclipse.
