---
layout: post
title: 安卓相关
category: [Linux-Mint,安卓,Android,反编译]
comments: false
---

* content
{:toc}

### 下载安装

#### 下载安装Android-SDK
```
下载	sdk-tools-linux-4333796.zip
https://developer.android.google.cn/studio/#downloads
https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip

解压到android-sdk目录

> cd android-sdk/tools
> ./bin/sdkmanager --update
> ./bin/sdkmanager --list
选择列表下对应需要安装的版本和工具即可：
> ./bin/sdkmanager "platforms;android-25" "build-tools;25.0.2"
```

#### 下载安装Android-Studio
```
下载地址：
https://developer.android.google.cn/studio/
更多版本下载：
https://developer.android.google.cn/studio/archive
3.1.3版本：
https://dl.google.com/dl/android/studio/ide-zips/3.1.3.0/android-studio-ide-173.4819257-linux.zip
反编译apk：
Build -> Analyze APK..选择apk文件即可
```

### 启用开发者调试

#### Linux下方法1
```
参考
https://developer.android.google.cn/studio/run/device#developer-device-options

Linux下：

> vim /etc/udev/rules.d/51-android.rules

SUBSYSTEM=="usb", ATTR{idVendor}=="0bb4", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTR{idVendor}=="12d1", MODE="0666", GROUP="plugdev"

> chmod a+r /etc/udev/rules.d/51-android.rules

其中12d1就是要调试的手机USB供应商ID，替换需要调试的下列手机即可： 
Acer	0502
ASUS	0b05
Dell	413c
Foxconn	0489
Fujitsu	04c5
Fujitsu Toshiba	04c5
Garmin-Asus	091e
Google	18d1
Haier	201E
Hisense	109b
HP	03f0
HTC	0bb4
Huawei	12d1
Intel	8087
K-Touch	24e3
KT Tech	2116
Kyocera	0482
Lenovo	17ef
LG	1004
Motorola	22b8
MTK	0e8d
NEC	0409
Nook	2080
Nvidia	0955
OTGV	2257
Pantech	10a9
Pegatron	1d4d
Philips	0471
PMC-Sierra	04da
Qualcomm	05c6
SK Telesys	1f53
Samsung	04e8
Sharp	04dd
Sony	054c
Sony Ericsson	0fce
Sony Mobile Communications	0fce
Teleepoch	2340
Toshiba	0930
ZTE	19d2

```

#### Linux下方法2
```
# 插上手机前后的usb信息差就是手机USB供应商ID信息
> lsusb
Bus 001 Device 004: ID 17ef:6050 Lenovo
Bus 001 Device 003: ID 17ef:1003 Lenovo Integrated Smart Card Reader
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 003: ID 2717:0310
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

其中ID为2717即为手机USB供应商ID

> vim /etc/udev/rules.d/51-android.rules
SUBSYSTEM=="usb", ATTR{idVendor}=="2717", MODE="0666", GROUP="plugdev"

> android-sdk/platform-tools/adb devices
```

### 配置打包相关

#### 签名证书
```
> keytool -genkey -alias xxx_prod -keyalg RSA -validity 10000 -keystore xxx_prod.keystore
123456
654321
> keytool -list -v -keystore xxx_prod.keystore
证书指纹:MD5,去掉:后字母全变小写即为微信签名

```

#### 签名编译打包
```
# app>build.gradle
# ./gradlew assembleRelease 后在app/build/release目录找到相关apk

android {
    compileSdkVersion 27
    buildToolsVersion "27.0.3"

    defaultConfig {
        applicationId "com.xxx.test"
        minSdkVersion 15
        targetSdkVersion 27
        versionCode 1
        versionName "0.0.1"
    }
    signingConfigs{
        debug{
            storeFile file("xxx_debug.keystore")
            storePassword '123456'
            keyAlias 'xxx_debug'
            keyPassword '123567'
        }
        release{
            storeFile file("xxx_prod.keystore")
            storePassword '123456'
            keyAlias 'xxx_prod'
            keyPassword '654321'
        }
    }
//    productFlavors {
//        yingyongbao {
//            applicationId "com.xxx.test"
//            versionName ".wx-1.0.0"
//        }
//        wandoujia {
//            applicationId "com.xxx.test"
//            versionName ".wdj-1.0.0"
//        }
//    }
//    productFlavors.all {
//        flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
//    }

    buildTypes {
        debug {
            minifyEnabled false
            zipAlignEnabled false
            shrinkResources false
            applicationIdSuffix ".debug"
        }
        release {
            minifyEnabled false//是否启动混淆
            zipAlignEnabled false//是否启动zipAlign
            shrinkResources false//是否移除无用的resource文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
            applicationVariants.all { variant ->
                if (variant.buildType.name.equals('release')){
                    //variant.mergedFlavor.versionCode = gitVersionCode()
                    //variant.mergedFlavor.versionName = gitVersionTag()
                    variant.outputs.each { output ->
                        def outputFile = output.outputFile
                        if(outputFile!=null && outputFile.name.endsWith(".apk")){
                            def fileName = "xxx-v${defaultConfig.versionName}-${defaultConfig.versionCode}-${releaseTime()}.apk".toLowerCase()
                            def tempFile = file("build.gradle")
                            output.outputFile = new File(tempFile.parent + "/build/${variant.buildType.name}",fileName)
                        }
                        //output.outputFile = new File(
                        //        output.outputFile.parent + "/${variant.buildType.name}",
                        //        "xxx-${variant.buildType.name}-v${variant.versionName}-${variant.productFlavors[0].name}-${releaseTime()}.apk".toLowerCase())
                    }
                }
            }
        }
    }
    lintOptions {
        checkReleaseBuilds false
        abortOnError false
    }
    android {
        defaultConfig {
            ndk {
                // 设置支持的SO库架构
                abiFilters 'armeabi' , 'x86', 'armeabi-v7a', 'x86_64', 'arm64-v8a'
            }
        }
    }


}

dependencies {
    ...
}

def gitVersionCode() {
    def cmd = 'git rev-list HEAD --first-parent --count'
    cmd.execute().text.trim().toInteger()
}

def gitVersionTag() {
    def cmd = 'git describe --tags'
    def version = cmd.execute().text.trim()
    def pattern = "-(\\d+)-g"
    def matcher = version =~ pattern
    if (matcher) {
        version = version.substring(0, matcher.start()) + "." + matcher[0][1]
    } else {
        version = version + ".0"
    }
    return version
}

def releaseTime() {
    return new Date().format("yyyyMMdd-HHmmss", TimeZone.getTimeZone("Asia/Shanghai"))
}

```





