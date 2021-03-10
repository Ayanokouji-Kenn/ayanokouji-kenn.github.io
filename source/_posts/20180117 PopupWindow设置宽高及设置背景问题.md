---
title: PopupWindow设置宽高及设置背景问题
date: 2018-01-17 11:10:27
tags: 
- Android
- UI
- problems
- PopupWindows
categories: Android
---

 ```
    private val mPopTitle: PopupWindow by lazy {
        PopupWindow(initPopView()).apply {
//            有了contentView后要先测量一下才能拿到正确的高度
            contentView.measure(View.MeasureSpec.UNSPECIFIED,View.MeasureSpec.UNSPECIFIED)
            width = ScreenUtils.getScreenWidth()
            height = contentView.measuredHeight
            isOutsideTouchable = true
            isFocusable = true
            setBackgroundDrawable(ColorDrawable(resources.getColor(R.color.black_35)))
            setOnDismissListener {
                tvTitle.setCompoundDrawablesRelativeWithIntrinsicBounds(0, 0, R.drawable.arrow_down_white, 0)
                PopUtils.clearDim(mContext)
            }
        }
    }
```
在initPopView（）返回了contentView之后，要先测量一下再设置宽高。
contentView的xml里全是wrap_content。
因为需要黑色背景，直接加载popupwindow上就行了`setBackgroundDrawable(ColorDrawable(resources.getColor(R.color.black_35)))`








