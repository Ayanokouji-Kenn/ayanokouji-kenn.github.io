---
title: AlertDialog不显示文字
date: 2017-06-27 10:33:27
tags: 
- Android
- UI
- Dialog
categories: Android
---

### 问题描述 
在自己小米5S安卓6.0上测试，发现弹框只有按钮没有文字，title和message都不显示，但是位置还是预留了的，很奇怪。



### 解决办法：

```java
AlertDialog.Builder builder;
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    builder = new AlertDialog.Builder(mContext,R.style.Theme_AppCompat_Light_Dialog_Alert);

}else {
    builder = new AlertDialog.Builder(mContext);
}
mDialog= builder.setTitle("需要开启一些权限")
        .setMessage("因为加入了语音识别，所以需要获取一些手机状态、定位信息等权限，麻烦您通过一下")
        .setPositiveButton(getString(R.string.confirm),this )
        .setNegativeButton(getString(R.string.cancel),this)
        .create();
```
5.0以上设置一个主题就行了.