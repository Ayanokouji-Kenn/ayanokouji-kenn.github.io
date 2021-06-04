---
title: 20210310 flutter web 页面空白 canvaskit
date: 2021-03-10 10:04:27
tags: 
- Flutter
- Web
categories: Flutter
 
---

------

- 问题：flutter web页面空白

- 原因：flutter会在不同平台自动选择渲染引擎，你当前如果使用的事canvaskit并且无法加载，就白屏了

- 解决：

  1. F12，检查是否有`Failed to load resource: net::ERR_CONNECTION_CLOSED ---> canvaskit.js`

  2. 如果有，就是你网页没下载成功（这玩意可能要科学上网，并且还挺大的 7M左右）。

     找到你flutter sdk的安装目录下的flutter\bin\cache\flutter_web_sdk\lib\_engine\engine\canvaskit\initialization.dart，

     定位到

     ```dart
     const String canvasKitBaseUrl = String.fromEnvironment(
       'FLUTTER_WEB_CANVASKIT_URL',
       defaultValue: 'https://unpkg.com/canvaskit-wasm@0.24.0/bin/',
     );
     ```

  3.  去上面那网址把文件全部下载到你本地，包括profiling文件夹、canvaskit.js、canvaskit.wasm，并将本地路径配置给defaultValue即可，注意要以`/`结尾。

