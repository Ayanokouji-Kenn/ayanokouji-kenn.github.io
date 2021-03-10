---
title:  parameter must be a descendant of this view
date: 2017-10-12 16:33:27
tags: 
- Android
- problems
categories: Android
---

非法参数异常，遇到的问题是：
ScrollView里面有RecyclerView，RecyclerView里的item有EditText，我是在这个页面在后台的时候，因为数据发生了变化，让页面刷新了，比如`adapter.notifyDataSetChanged();`，因获取焦点产生的问题
```java
java.lang.IllegalArgumentException: parameter must be a descendant of this view
	at android.view.ViewGroup.offsetRectBetweenParentAndChild(ViewGroup.java:5103)
	at android.view.ViewGroup.offsetDescendantRectToMyCoords(ViewGroup.java:5040)
	at android.widget.ScrollView.isWithinDeltaOfScreen(ScrollView.java:1140)
	at android.widget.ScrollView.onSizeChanged(ScrollView.java:1543)
	at android.view.View.sizeChange(View.java:15843)
	at android.view.View.setFrame(View.java:15808)
	at android.view.View.layout(View.java:15724)
```

解决办法：
给这个页面的其他什么东西,比如顶层view设置上
```
android:focusable="true"
android:focusableInTouchMode="true"
```
这样item里的EditText不会获取焦点就ok了。


