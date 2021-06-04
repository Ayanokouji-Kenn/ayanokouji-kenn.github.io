---
title: Could not find com.android.supportappcompat-v723.0.3
date: 2016-07-20 10:33:27
tags: problems
---
##### 遇到的问题
将整个AS项目拷到另一个电脑的时候，出现了这个问题，提示app:unspecified
请去下载android support Respository,打开后SDK Manager更新到最新了还是不行

##### 解决办法
将项目clean一下，AS会列出一堆目录，他在这些目录里面找appcompat-v7:23.0.3
找到其中的一个目录，比如C:\develop_software\android-studio-sdk\extras\android\m2repository\com\android\support\appcompat-v7
在这里面找23.0.3，发现并没有···那么看看有什么，有23.3.0，那么在build.gradle改成23.3.0，然后clean，rebuild即可