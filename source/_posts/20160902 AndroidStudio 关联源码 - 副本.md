---
title: AndroidStudio 关联源码
date: 2016-09-02 10:33:27
tags: tips
---
首先，确保你已经下载完成了，
然后，在`C:\Users\xxx\.AndroidStudio2.1\config\options`这里找到`jdk.table.xml`
打开它
~~~java
<jdk version="2">
      <name value="Android API 24 Platform" /> //找到对应的版本
      <type value="Android SDK" />
      <version value="java version "1.8.0_91"" />
      <homePath value="D:\android-studio-sdk" />
      <roots>
        <annotationsPath>
          <root type="composite">
            <root type="simple" url="jar://$APPLICATION_HOME_DIR$/plugins/android/lib/androidAnnotations.jar!/" />
          </root>
        </annotationsPath>
        <classPath>
          <root type="composite">
            <root type="simple" url="jar://D:/android-studio-sdk/platforms/android-24/android.jar!/" />
            <root type="simple" url="file://D:/android-studio-sdk/platforms/android-24/data/res" />
          </root>
        </classPath>
        <javadocPath>
          <root type="composite">
            <root type="simple" url="file://D:/android-studio-sdk/docs/reference" />
          </root>
        </javadocPath>
        <sourcePath>   //源码在這里
          <root type="composite">
          //把路径填进去，重开AndroidStudio就ok了
            <root type="simple" url="file://D:/android-studio-sdk/sources/android-24" />
          </root>
        </sourcePath>
      </roots>
      <additional jdk="1.8" sdk="android-24" />
    </jdk>
~~~