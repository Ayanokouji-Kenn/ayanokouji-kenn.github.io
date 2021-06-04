---
title: 解决toggleButton关闭状态初始化背景无效
date: 2016-07-24 10:33:27
tags: problems
---
-      做安卓开发想必最头疼的是“与IOS一样”了，
询问IOS这个怎么做的，那个怎么做的，
答曰：系统默认的/系统提供了···

安卓也提供了toggleButton，不过项目开发中为了保持一致性，设计师基本需要用到开关的时候基本用的还是IOS得那种，于是我找到了
https://github.com/zcweng/ToggleButton                      这哥们写的
用的时候发现在初始化的时候，会出现**不绘制背景**的问题。。。

-   解决：
将onLayout中最后一句`offLineWidth = 0;`改成`calculateEffect(toggleOn?1:0);`