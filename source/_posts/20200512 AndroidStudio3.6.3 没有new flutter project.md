---
title: AndroidStudio3.6.3 没有new flutter project
date: 2020-05-12 10:19:27
tags: 
- Android
- AndroidStudio
- Flutter
categories: Android
cover: https://i.loli.net/2020/07/27/Cbzq6x9seiZNcLW.png
---

很久之前，AS就提示更新了，因为有项目做，不敢贸然改变环境就放下来了。现在有空就升级了一波，然后就发生了标题上的事情。直接上解决办法。

![1.png](https://i.loli.net/2020/07/27/Cbzq6x9seiZNcLW.png)
点击Help- About，查看当前AS的版本，图上的是192.7142.xxxxxx。

去[dart]([https://plugins.jetbrains.com/plugin/6351-dart/versions](https://plugins.jetbrains.com/plugin/6351-dart/versions)
)和[flutter]([https://plugins.jetbrains.com/plugin/9212-flutter/versions](https://plugins.jetbrains.com/plugin/9212-flutter/versions)
)网站上查询与你AS匹配的插件版本。

注意这里先切换成AS
![2.png](https://i.loli.net/2020/07/27/rg874HbhFxWvCkn.png)

我AS3.6.3的版本是192.7142，所以用这个，将dart和flutter都下载下来。
![3.png](https://i.loli.net/2020/07/27/syNu9JIc8lvD6OU.png)

然后回到AS，File-Settings-Plugins
![4.png](https://i.loli.net/2020/07/27/fkFQ2JZLuBhRNCy.png)
点这个齿轮，Install Plugin from Disk,选择你刚刚下载好的插件压缩包安装，然后重启AS即可。




