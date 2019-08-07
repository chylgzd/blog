---
layout: post
title: eclipse相关
category: [eclipse]
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
