---
title: MaterialButtonToggleGroup中按钮样式
date: 2020-05-20 16:06:27
tags: 
- Android
- UI
categories: Android
cover: https://i.loli.net/2020/07/14/vKVjuy7O59FBNAa.png
---

我这边做因为数据是动态获取的，本来想用recyclerview来做，突然想起来某个版本加了这东西。看演示，别人的是这样的，

![aaq7i-muy3j.gif](https://i.loli.net/2020/07/14/cVT7HzqbjBiUZrS.gif)
我做完是这样，注意这不是全选中，而是全没选中的样子。。
![21.png](https://i.loli.net/2020/07/14/vKVjuy7O59FBNAa.png)
![22.png](https://i.loli.net/2020/07/14/Y7Fzi6NkrHGX25h.png)

然后查了下[官网]([https://developer.android.com/reference/com/google/android/material/button/MaterialButtonToggleGroup](https://developer.android.com/reference/com/google/android/material/button/MaterialButtonToggleGroup)
)，发现要对`MaterialButtonToggleGroup`内部的`MaterialButton`加上`
         style="?attr/materialButtonOutlinedStyle"`的样式。
因为我是动态添加的，所以没有在xml写了，代码是这样的
```kotlin 
    btnToggleGroup.addView(createBtnToggle( "-"))

    private fun createBtnToggle(content: String): Button {
        val btn = MaterialButton(
            requireContext(),
            null,
            R.attr.materialButtonOutlinedStyle
        )
        val layoutParam = ViewGroup.LayoutParams(
            ViewGroup.LayoutParams.WRAP_CONTENT,
            ViewGroup.LayoutParams.WRAP_CONTENT
        )
        btn.layoutParams = layoutParam
        btn.setPadding(16f.dp.toInt(), 8f.dp.toInt(), 16f.dp.toInt(), 8f.dp.toInt())
        btn.text = content
        btn.textSize = 20f.sp
        return btn
    }
```
如果在xml里用，那直接官网上这样就行
```
<com.google.android.material.button.MaterialButtonToggleGroup
     xmlns:android="http://schemas.android.com/apk/res/android"
     android:id="@+id/toggle_button_group"
     android:layout_width="wrap_content"
     android:layout_height="wrap_content">

     <com.google.android.material.button.MaterialButton
         style="?attr/materialButtonOutlinedStyle"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="@string/button_label_private"/>
     <com.google.android.material.button.MaterialButton
         style="?attr/materialButtonOutlinedStyle"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="@string/button_label_team"/>
     <com.google.android.material.button.MaterialButton
         style="?attr/materialButtonOutlinedStyle"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="@string/button_label_everyone"/>
     <com.google.android.material.button.MaterialButton
         style="?attr/materialButtonOutlinedStyle"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="@string/button_label_custom"/>

 </com.google.android.material.button.MaterialButtonToggleGroup>
 
```


 