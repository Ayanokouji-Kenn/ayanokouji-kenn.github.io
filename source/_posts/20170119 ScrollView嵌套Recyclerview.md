---
title: ScrollView嵌套Recyclerview
date: 2017-01-19 10:33:27
tags: tips
categories: Android
---

- 老问题了，以前用一些别的方式解决了比如linearlayout，比如listview的type，比如footview之类的

  但这次的需求不太一样

  ![](https://ooo.0o0.ooo/2017/01/19/588063134fec6.png)

  ​

  上面是一个recyclerview全部显示，滑到底了，下面一个“今日推荐”，再下面一个资讯类的recyclerview有分页功能，滑到底了加载下一页。

- 思路就是让recyclerview不能滑动，滑动交给外层的scrollview，将recyclerview替换成下面这个

  ```java

  public class NoScrollRecyclerView extends RecyclerView {
      public NoScrollRecyclerView(Context context) {
          super(context);
      }
      public NoScrollRecyclerView(Context context, @Nullable AttributeSet attrs) {
          super(context, attrs);
      }
      public NoScrollRecyclerView(Context context, @Nullable AttributeSet attrs, int defStyle) {
          super(context, attrs, defStyle);
      }
      @Override
      protected void onMeasure(int widthSpec, int heightSpec) {
          int IheightSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
          super.onMeasure(widthSpec, IheightSpec);
      }
  }
  ```

  至少数据就都能显示了

  然后会发现，自带的惯性滑动没有了，阻尼也有点别扭，那么把scrollview也给换了吧，加上惯性

  ```java
  public class MyScrollView extends ScrollView {
      private int downX;
      private int downY;
      private int mTouchSlop;

      public MyScrollView(Context context) {
          super(context);
          mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
      }

      public MyScrollView(Context context, AttributeSet attrs) {
          super(context, attrs);
          mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
      }

      public MyScrollView(Context context, AttributeSet attrs, int defStyleAttr) {
          super(context, attrs, defStyleAttr);
          mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
      }

      @Override
      public boolean onInterceptTouchEvent(MotionEvent e) {
          int action = e.getAction();
          switch (action) {
              case MotionEvent.ACTION_DOWN:
                  downX = (int) e.getRawX();
                  downY = (int) e.getRawY();
                  break;
              case MotionEvent.ACTION_MOVE:
                  int moveY = (int) e.getRawY();
                  if (Math.abs(moveY - downY) > mTouchSlop) {
                      return true;
                  }
          }
          return super.onInterceptTouchEvent(e);
      }
  }
  ```

  ps：其实还有一些别的问题，我的recyclerview用的是hongyang的那baseAdapt，他好久没维护了，有不少问题。。不过与标题无关，这篇文章暂且按下不表。
