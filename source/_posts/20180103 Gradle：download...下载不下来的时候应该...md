---
title: Gradle：download...下载不下来的时候应该..
tags: 
- Android
- problems
date: 2018-01-03 17:23:27
categories: Android
---

有时候compile一些库，特别是谷歌自己的库的时候，下不动。
在项目里的`gradle.properties`下加上
`org.gradle.jvmargs=-DsocksProxyHost=127.0.0.1 -DsocksProxyPort=1080`试试。
此法对于gradle下载库有效，下载gradle本身应该是无效的。