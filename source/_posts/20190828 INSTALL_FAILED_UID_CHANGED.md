---
title: INSTALL_FAILED_UID_CHANGED
date: 2019-08-28 17:21:27
tags: 
- Android
categories: Android
---

错误：安装出现`INSTALL_FAILED_UID_CHANGED`

原因：设备已root，清单配置了系统应用，签名也是额外用系统签名打包的，在`AndroidManifest.xml`中设置了`android:sharedUserId="android.uid.system`，而打包时忘了添加上。导致的UID错误不同从而安装失败。