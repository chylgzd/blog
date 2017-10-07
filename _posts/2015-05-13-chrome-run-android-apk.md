---
layout: post
title: Chrome浏览器里运行Android程序
category: [Chrome,Android,ARC-Welder,App-Runtime-for-Chrome]
comments: false
---

* content
{:toc}

(目前还有bug，在显卡中WebGl的问题，mint下切换为开源显卡后正常)

#### 第一种方法：

 运行环境安装 App Runtime for Chrome (Beta)扩展插件

```
谷歌浏览器扩展里搜索mfaihdlpglflfgpfjcifdjdjcckigekc或者下面地址：
https://chrome.google.com/webstore/detail/app-runtime-for-chrome-be/mfaihdlpglflfgpfjcifdjdjcckigekc
```

用于安装APK文件到chrome的应用 ARC Welder（依赖App Runtime）

```
谷歌浏览器应用里搜索emfinbmielocnlhgmfkkmkngdoccbadn或者下面地址：

https://chrome.google.com/webstore/detail/arc-welder/emfinbmielocnlhgmfkkmkngdoccbadn

（第一次打开ARC Welder需要设置安装APK文件到哪个目录）

安装好之后在应用界面里打开ARC Welder点击添加APK文件

然后选择模拟器大屏/小屏以及横放/垂直

点击TEST会自动安装APK并生成文件（在第一次设置的目录下面）

（另外一个类似应用工具twerk，方法差不多，目前测试没有ARC Welder稳定：
https://chrome.google.com/webstore/detail/twerk/jhdnjmjhmfihbfjdgmnappnoaehnhiaf
）
```

#### 第二种方法，具体参考（目前测试比第一种方法不太稳定）：

```
https://archon-runtime.github.io/
https://github.com/vladikoff/chromeos-apk/blob/master/archon.md
```

