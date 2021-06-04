---
title: radioGroup.clearCheck()的坑
date: 2016-11-09 10:33:27
tags: problems
---
##### 遇到的问题
说到radioGroup的时候，我们肯定会这么用

```java
radioGroup.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(RadioGroup group, int checkedId) {
      
        });
```

项目中一个地方，radioGroup和一个popupWindow里的选项是只能选一个的，所以我是在popupWindow里被选中的时候调用了`radioGroup.clearCheck()`然而发现pop里的点击事件无效。。反而是清除掉的radioButton的点击事件又被执行了一次。

##### 解决办法
在其他地方`radioGroup.clearCheck()` 之前，设个标记表示我要开始清理checked状态了，在`OnCheckedChangeListener()`里通过这个标记来过滤掉这种情况