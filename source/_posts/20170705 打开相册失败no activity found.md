---
title: 打开相册失败no activity found
date: 2017-07-05 10:33:27
tags: 
- Android
categories: Android
---

- 今天看到crash列表里出现了`ActivityNotFoundException`，信息就是标题上的那些，定位之后发现是从相册选择图片处隐式启动相册找不到Activity，可能是用户的设备上没有相册应用（黑人问号）。

- 之前的处理方式是这样
```java
Intent intent = new Intent(Intent.ACTION_PICK,
android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
startActivityForResult(intent, RC_ALBUM);
```
在`onActivityResult`里面是这样
```
else if (requestCode == RC_ALBUM && resultCode == RESULT_OK && data != null) {
  Uri selectedImage = data.getData();
  if (selectedImage != null) {
    String[] filePathColumn = {MediaStore.Images.Media.DATA};
    // 获取选择照片的数据视图
    Cursor cursor = getContentResolver()
    .query(selectedImage,filePathColumn, null, null, null);
    cursor.moveToFirst();
    // 从数据视图中获取已选择图片的路径
    int columnIndex = cursor.getColumnIndex(filePathColumn[0]);
    String picturePath = cursor.getString(columnIndex);
    cursor.close();
    // 将图片显示到界面上
    Bitmap bm = BitmapFactory.decodeFile(picturePath);

    findViewById(R.id.rl_table).setBackground(new BitmapDrawable(bm));
 }
```
- 解决办法有三种，自行选择：
1、简单粗暴的方法就是判断下隐式intent有没有匹配的Activity，有再去启动，没有就给他Toast什么的吧

```java
Intent intent = new Intent(Intent.ACTION_PICK,
android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
List<ResolveInfo> resolveInfos = getPackageManager()
  .queryIntentActivities(
    intent,PackageManager.MATCH_DEFAULT_ONLY);
if(resolveInfos.size()!=0){
    startActivityForResult(intent, RC_ALBUM);
}else {
      ...
}
```

2、打开内容管理器，也就是“文档”，设置查找的type为image，这玩意一般设备都有的，不过还是判断一下吧，现在看到隐式启动心里都虚的，你根本不知道你的用户是什么奇奇怪怪的设备。

```java
Intent intent = new Intent();
intent.setType("image/*");
intent.setAction(Intent.ACTION_GET_CONTENT);
startActivityForResult(Intent.createChooser(intent, "Select Picture"), RC_ALBUM);
List<ResolveInfo> resolveInfos = 
        getPackageManager()
        .queryIntentActivities(intent
        ,PackageManager.MATCH_DEFAULT_ONLY);
if(resolveInfos.size()!=0){
    startActivityForResult(intent, RC_ALBUM);
}else {
      ...
}
```

3、让用户选择用文档还是相册来打开
```java

Intent getIntent = new Intent(Intent.ACTION_GET_CONTENT);
getIntent.setType("image/*");
Intent pickIntent = new Intent(Intent.ACTION_PICK, android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
pickIntent.setType("image/*");
Intent chooserIntent = Intent.createChooser(getIntent, "Select Image");
chooserIntent.putExtra(Intent.EXTRA_INITIAL_INTENTS, new Intent[] {pickIntent});
 startActivityForResult(chooserIntent, RC_ALBUM);
```

- 最后需要注意的是如果使用`ACTION_GET_CONTENT`方式打开的话，在onActivityResult中获取文件的方式和用 `ACTION_PICK`是不一样的，而且在安卓4.4之前和之后也是不一样的，请注意。
有个台湾友人做了个工具类，可以看看他的[blog](https://magiclen.org/android-filechooser/)
***
EOF