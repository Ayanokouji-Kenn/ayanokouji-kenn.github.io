---
title: gravity=center_vertical与centerVertcal=true的区别
date: 2017-07-18 10:33:27
tags: 
- Android
categories: Android
---

这特么是一个坑。
一个ImageView放在RelativeLayout中，给RelativeLayout设置`android:gravity="center_vertical`发现并没有垂直居中，歪了一点点。而直接给ImageView设置`android:layout_centerVertical="true"`就正好。

用ImageView的getTop()和getBottom()探索过程不提了，直接说结果：
首先我ImageView的资源文件高度为28px；
当没有设置drawable-hdpi，图片资源只放在了drawable-xhdpi里，而设备是240dpi的时候，hdpi里找不到，就去xhdpi里找了，找到后显示在屏幕上会缩放0.75倍，变成21px；区别就在这儿了。

比如RelativeLayout的高度是80px；
`android:layout_centerVertical="true"`是设置给ImageView的，会测量好ImageView的实际高度得到21px，然后垂直居中就是距离顶部（80-21）/2=29。

而`android:gravity="center_vertical`是设置给RelativeLayout的，他还是按照源资源大小给你计算（80-28）/2 = 26；距离父控件的距离是26px。

就是这3个像素让我看出来“歪了”。

解决办法有2个：
1、现在mdpi的设备不多了。。但hdpi还是有一些的，该为他们弄一套图还是专门弄一套图吧。
2、LinearLayout的`android:gravity="center_vertical`是准的，RelativeLayout既然不准，那就还是用`android:layout_centerVertical="true"`靠谱。
