---
layout: post
title: eclipse相关
category: [eclipse,lombok]
comments: false
---

* content
{:toc}


### eclipse查找替换

```
1：替换所有 #para(xxx) 为 #{xxx}：

查询正则：#para\((\w+)\)
替换正则：#{$1}

2：替换#{_xxx} 为 #{vo.xxx}
查询正则：#\{_(\w+)\}
替换正则：#{vo.$1}

```


#### 错误相关

```
输入.号后不自动弹出提示：
windows ——preferences ——java ——editor —— content assist - 勾选Java Prolosals即可

```



### eclipse插件相关

#### lombok插件
```
1,下载jar,注意下载项目中依赖对应或接近版本,否则容易出兼容问题(如maven项目中依赖lombok为1.16.22,则应该选择lombok-1.16.18.jar)

https://projectlombok.org/all-versions
https://projectlombok.org/downloads/lombok-1.16.18.jar

2, > java -jar lombok-1.16.18.jar 按提示install后重启eclipse即可

```