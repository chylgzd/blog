---
layout: post
title: 手动安装nodejs新版本及npm更新
category: [Nodejs,Npm,TaobaoMirrors]
comments: false
---

* content
{:toc}

```
ubuntu自动安装模式参考：
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs

更多参考：
https://nodejs.org/en/download/package-manager

本文讨论如何手动安装
```

###  下载linux安装包并解压  
查看页面版本 [latest-v6](https://nodejs.org/dist/latest-v6.x)   国内[taobao下载v6](https://npm.taobao.org/mirrors/node/latest-v6.x/),[其它镜像](https://npm.taobao.org/mirrors/node/)
找到.xz连接地址如：https://nodejs.org/dist/latest-v6.x/node-v6.2.1-linux-x64.tar.xz  
按照以下步骤将会安装到 ~/programming/tools/nodejs/node_home 文件夹  
以后版本更新的时候只需要执行这个步骤就行  

```
cd /tmp
wget -O nodejs.tar.xz https://nodejs.org/dist/latest-v6.x/node-v6.2.1-linux-x64.tar.xz
mkdir -p ~/programming/tools/nodejs/node_home && rm -rf ~/programming/tools/nodejs/node_home/* && tar -Jxvf ./nodejs.tar.xz  -C ~/programming/tools/nodejs/node_home --strip-components 1
```

###  手动添加link
```bash
配置完后注销运行npm --version如果出现： 
/usr/bin/env: node: No such file or directory

则需手动添加link node 到~/bin目录
ln -sf ~/programming/tools/nodejs/node_home/bin/node ~/bin/node
```

### npm config 配置 ([更多参考](https://docs.npmjs.com/files/npmrc))
(或 sudo gedit ~/.npmrc 里面牵涉到的文件夹需要自己先创建好) 

```
设置为国内镜像(默认是 https://registry.npmjs.org)
npm config set registry https://registry.npm.taobao.org

设置缓存文件夹
npm config set cache "~/programming/tools/nodejs/cache/"

设置全局文件夹
npm config set prefix "~/programming/tools/nodejs/npm_home/"

设置electron镜像，默认下载文件在~/.electron/目录(electron >=1.6.10 && electron-packager >= 8.7.0 && nodejs>=8.0.0 && npm>=5.0.2)
npm config set electron_mirror https://npm.taobao.org/mirrors/electron/
npm config set force_no_cache true
(也可以在环境变量里配置：
#export ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/"
#export SASS_BINARY_SITE="https://npm.taobao.org/mirrors/node-sass"
)
```

### 添加环境变量  
(nodejs目前对～配置有bug，会造成node命令和NODE_PATH无法生效)

```bash

sudo gedit /etc/profile.d/self.sh

export NODE_HOME=/home/yourname/programming/tools/nodejs/node_home
export NPM_GLOBAL_HOME=/home/yourname/programming/tools/nodejs/npm_home
export NODE_PATH=$NPM_GLOBAL_HOME/lib/node_modules
export PATH=$PATH:$NODE_HOME/bin:$NPM_GLOBAL_HOME/bin
```
关于环境变量：  
在 /etc/profile 中配置可以对任意用户登陆有效,但sudo执行时无效  
在 ~/.bash_profile 中配置只对当前用户有效  
在 /var/root/.bash_profile配置的，对su之后对root用户有效  

#### 各种配置完成注销成功生效后可运行下列帮助命令：  
```
查看npm的所有配置属性：
npm config ls -l
查看某一个具体配置：
npm config get xxx 比如 npm config get prefix
查看npm目录：
npm prefix -g

查看 运行npm i -g xxx命令 会安装到哪个目录：
npm root -g
```

#### npm link：
```
全局模式下安装nobin-debian-installer
npm install nobin-debian-installer -g

进入开发目录
cd ~/work/test

把全局模式的nobin-debian-installer模块链接到本地的node_modules下
npm link nobin-debian-installer

进入另外的一个开发目录
cd ../work/test2

把全局模式的nobin-debian-installer模块链接到本地
npm link nobin-debian-installer

更新全局模式的nobin-debian-installer，会更新所有link过去的项目
npm update nobin-debian-installer -g
```

#### npm常用命令(参考 [https://docs.npmjs.com](https://docs.npmjs.com))
```
npm install xxx 安装模块
npm install xxx@1.1.1   安装1.1.1版本的xxx
npm install xxx -g 将模块安装到全局环境中。
npm ls 查看安装的模块及依赖
npm ls -g 查看全局安装的模块及依赖
npm uninstall xxx  (-g) 卸载模块
npm cache clean 清理缓存
npm help xxx  查看帮助
npm view moudleName dependencies  查看包的依赖关系
npm view moduleNames  查看node模块的package.json文件夹
npm view moduleName labelName  查看package.json文件夹下某个标签的内容
npm view moduleName repository.url  查看包的源文件地址
npm view moduleName engines   查看包所依赖的Node的版本
npm help folders   查看npm使用的所有文件夹
npm rebuild moduleName    用于更改包内容后进行重建
npm outdated   检查包是否已经过时，此命令会列出所有已经过时的包，可以及时进行包的更新
npm update moduleName   更新node模块
```

#### 更新npm(查看最新版 [https://github.com/npm/npm/releases](https://github.com/npm/npm/releases))
```
cd /tmp
npm i npm
rm -rf ~/programming/tools/nodejs/node_home/lib/node_modules/npm
mv /tmp/node_modules/npm ~/programming/tools/nodejs/node_home/lib/node_modules/
```

### 关于NODE_PATH环境变量无效的bug解决

```javascript
var filename = "~/programming/tools/nodejs/npm_home/lib/node_modules";
var internalModuleStat = process.binding('fs').internalModuleStat;
var result = internalModuleStat(filename);
console.log(result);
```
以上代码返回-2，把~更换为/home/yourname即可

#### 关于NODE_PATH参考：
https://nodejs.org/api/modules.html#modules_loading_from_the_global_folders  
可以用node-inspector调试nodejs加载模块过程
