---
title: popupWindow的一个坑
date: 2016-12-05 10:33:27
tags: problems
---
- PopupWindow这东西可以说大量存在于各种app里，
  ` popupWindow = new PopupWindow(popupWindowView, DP2PX.dip2px(this, 149F), LinearLayout.LayoutParams.WRAP_CONTENT, true);` 

通常会这么写，最后一个true表示获取焦点，这样做有一个好处，当点击pop以外的区域，都会让pop消失，同时因为焦点在pop上，点击外面的按钮因为按钮没有焦点所以不会触发点击事件。这也是符合一般用户习惯的，如下图

![](http://ooo.0o0.ooo/2016/12/05/58453bc622280.gif)

西卡西！！！如果我要相应外部的点击事件呢？

如果你认为：那好办，调用另一个构造方法，不获取焦点不就行了。

曾经我也是这么的年轻。。。不获取焦点，确实能够让外部响应点击事件，然而

**弹出pop的那个按钮也响应了点击事件了啊喂！！**  用户习惯通常应该是点击一次，弹出popup，再点击一次，消失啊！

![](http://ooo.0o0.ooo/2016/12/05/58453e660cfec.gif)

这样爽么？这肯定不是我们想要的

那么如何让这个按钮不要响应点击事件或者响应了点击事件但是关闭pop而不是再次打开呢

你可能又会想到：监听pop的开闭状态，搞个变量记住，在点击事件里根据这个变量判断不就行了？

too young to simple，当pop外部被获取焦点的时候，pop会消失，这个时候你的标记就会被改成了关闭状态，然后才会触发点击事件，此时认为pop是关闭的，那么会再次打开，做了这么多事等于白干！

说了辣么多，怎么解决呢？

**在popupwindow.setOnDismissListener()里记录关闭的时间，在点击时间里用当前时间与这个时间对比，大于500ms，才打开，没错这是hardcode，你们有好的解决办法请告诉我**

