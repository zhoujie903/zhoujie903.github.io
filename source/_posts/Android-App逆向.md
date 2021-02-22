---
title: Android App逆向
date: 2021-01-18 11:10:37
tags:
-- App逆向
---

[TOC]

## 环境
* Mac 
* Android SDK:

```
/Users/zhoujie/Library/Android/sdk
```

* JDK

```
/Applications/Android Studio.app/Contents/jre/jdk/Contents/Home
```

* 夜神模拟器

## 任务一：签名

* 下载 抖音极速版

```
从 小米应用商店 下载 抖音极速版
https://app.mi.com/details?id=com.ss.android.ugc.aweme.lite&ref=search

com.ss.android.ugc.aweme.lite.apk
```
* 在模拟器上安装 原版抖音极速版 并运行，确保能正常运行
* 删除原签名，重新打包

```
mkdir com.ss.self

unzip com.ss.android.ugc.aweme.lite.apk -d com.ss.self

// 删除 META-INF 目录
rm -rf com.ss.self/META-INF


// 打包成Apk - com.ss.self.apk 
cd com.ss.self
zip -r com.ss.self.apk *
```
* 签名

```
// 进入到 apksigner 所在目录
cd /Users/zhoujie/Library/Android/sdk/build-tools/29.0.2

// 签名
./apksigner sign --ks ~/.keystore ~/Downloads/com.ss.self/com.ss.self.apk 
```
* 在模拟器上安装 重新签名的Apk 并运行，确保能正常运行

```
adb connect 127.0.0.1:62001
adb install ~/Downloads/com.ss.self/com.ss.self.apk 
```

* 一步到位

```
./apksigner sign --ks ~/.keystore ~/Downloads/com.ss.android.ugc.aweme.lite.apk
```

## 任务二：信任用户证书

* 在清单文件`AndroidManifest.xml`中开启网络安全配置，代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application android:networkSecurityConfig="@xml/network_security_config"
                    ... >
        ...
    </application>
</manifest>
```

* 新建文件`res/xml/network_security_config.xml`来进行网络安全的配置，通过trust-anchors来设置信任的证书，代码如下：


```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="user" />
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
</network-security-config>
```

### 加固的App
映客直播极速版 - com.ingkee.lite - 爱加密

https://app.mi.com/details?id=com.sljh.zqxsp&ref=search
![](https://gitee.com/zhoujie903/image-repo/raw/master/img/WechatIMG558.jpeg)
