---
title: Android仿ios锁屏的验证小滑块
date: 2019-10-12 17:21:27
tags: 
- Android
- UI
categories: Android
---

xml中
```
   <androidx.appcompat.widget.AppCompatSeekBar
            android:layout_marginLeft="40dp"
            android:layout_marginRight="40dp"
            android:id="@+id/seekbar"
            android:layout_width="match_parent"
            android:layout_height="50dp" />
```

![](https://user-gold-cdn.xitu.io/2019/10/12/16dbedd0b0c0599d?w=379&h=58&f=png&s=3572)
代码中
```kotlin
        val dp25 = SizeUtils.dp2px(25F)
        // 画背景，圆角为宽度一半，画出矩形
        seekbar.background = GradientDrawable().apply {
            shape = GradientDrawable.RECTANGLE
            cornerRadius = dp25.toFloat()
            color = ColorStateList.valueOf(resources.getColor(R.color.colorPrimary))
        }
        // 设置padding，值为滑块的半径。这样progress为0和100的时候，滑块会刚好顶住两边。
        // 否则滑块会凸出去，难看。
        seekbar.setPadding(dp25, 0, dp25, 0)
        // 偏移量，我这都自定义的drawable，如果有图片的话可能会用到
        seekbar.thumbOffset = 0
        // 同上，滑块显示透明啥的出问题可能会用上这个
        seekbar.splitTrack = true
        // 滑块的drawable
        seekbar.thumb = GradientDrawable().apply {
            shape = GradientDrawable.OVAL
            color = ColorStateList.valueOf(Color.GRAY)
            val dp50 = SizeUtils.dp2px(50f)
            setSize(dp50, dp50)
        }
        // 验证滑块嘛，要禁止直接点击，只允许拖动
        seekbar.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
            var progress = seekbar.progress // 默认的progress
            var started = false // 默认未开始
            override fun onProgressChanged(seekBar: SeekBar?, progressValue: Int, fromUser: Boolean) {
                if (!started) {
                    //求出滑块的大小所在的progress范围，在这范围内才响应
                    //seekbar.max.toFloat()/seekbar.width 是求出一个像素对应多少progress
                    //seekbar.thumb.intrinsicWidth/2  再乘以这个球的半径，就是阈值了
                    val threshold = seekbar.max.toFloat()/seekbar.width*seekbar.thumb.intrinsicWidth/2
                    // 原progress与当前progress的绝对值小于阈值，且是用户操作的，更改标记
                    if (abs(progressValue - progress) < threshold) {
                        if (fromUser) {
                            started = true
                        }
                    }else {
                        seekbar.progress = progress
                        return
                    }
                }
                if (started) {
                    progress = progressValue
                }
            }

            override fun onStartTrackingTouch(seekBar: SeekBar?) {
                LogUtils.d("on start touch ${seekBar?.progress}")
//                if(abs(seekbar?.progress?:0 - progress) >10){
//                    seekbar?.progress = progress
//                }
            }

            override fun onStopTrackingTouch(seekBar: SeekBar?) {
                LogUtils.d("on stop touch")
                started = false
            }

        })
        //  其他逻辑，比如什么滑块滑到头，滑块要变化图片，那自己设置`setThumb`就是了。
        //  滑到头，通常用禁用seekbar不允许再划回去。。之类的，自己加`setEnabled`

```