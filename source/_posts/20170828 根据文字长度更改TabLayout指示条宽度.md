---
title: 根据文字长度更改TabLayout指示条宽度
date: 2017-08-28 10:33:27
tags: 
- Android
- UI
- tablayout
categories: Android
---

经常安卓开发要用苹果风格的东西，比如dialog

![dialog.png](http://upload-images.jianshu.io/upload_images/2524531-dfdb942a05052da6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

直接上代码
```java
public class SimpleDialog extends AlertDialog {
    private String title;
    private String left;
    private String right;
    private View.OnClickListener listener;

    public SimpleDialog(@NonNull Context context,String title,String left,String right, View.OnClickListener listener) {
        super(context);
        this.title = title;
        this.left = left;
        this.right = right;
        this.listener = listener;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.dialog_simple);
//这里是设置宽度，不设置的话是有一个margin值的match_parent效果。
        Window window = getWindow();
        WindowManager.LayoutParams lp = window.getAttributes();
        lp.width = SizeUtils.dp2px(250F);
        window.setAttributes(lp);
//如果你是圆角之类的话，这句设置背景透明要加上。
//否则有个白底在那儿，你的dialog也是白色的话是看不到圆角的
        window.setBackgroundDrawableResource(android.R.color.transparent);
//dialog是可以直接findViewById的，拿到控件后设置文字、点击
        TextView tvTitle = (TextView) findViewById(R.id.tv_title);
        TextView tvLeft = (TextView) findViewById(R.id.tv_left);
        TextView tvRight = (TextView) findViewById(R.id.tv_right);
        tvTitle.setText(title);
        tvLeft.setText(left);
        tvRight.setText(right);
        tvLeft.setOnClickListener(listener);
        tvRight.setOnClickListener(listener);
    }
}
```
在dialog的布局文件中，宽度最好用match_parent和weight=1，高度可以写固定值。然后在window里设置具体宽度，高度不设置。