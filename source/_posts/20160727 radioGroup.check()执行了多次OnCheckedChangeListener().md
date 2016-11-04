---
title: radioGroup.check()执行了多次OnCheckedChangeListener()
date: 2016-07-27 10:33:27
tags: problems
---
- 问题： 
我对某个radioButton里面写了个startActivity跳转到另一个界面，然后那个界面finish()之前需要调用radioGroup.check(),发现又跳转进这个界面了，纳闷之下，debug走起，发现OnCheckedChangeListener()走了好几次，所以页面又被启动了。

- 解决：
将radioGroup.check() 替换成radioButton.setChecked(true);