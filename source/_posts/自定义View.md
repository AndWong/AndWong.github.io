---
title: 自定义View
date: 2017-02-17 21:07:24
tags: Android
---
一.使用LayoutInflater加载视图
写法一:
```
LayoutInflater layoutInflater = LayoutInflater.from(context);
```
写法二:
```
LayoutInflater layoutInflater =(LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);  
```
加载布局:
```
layoutInflater.inflate(resourceId, root);
```
<!--more-->
补充:layout_width,layout_height其实是用于设置View在布局中的大小的,
如果控件没有在布局中,这个参数就会不起作用,所以叫layout_width而不是width.
setContentView,系统会自动在布局外层嵌套一层FrameLayout.

二.视图的绘制过程
1. onMeasure
2. onLayout
3. onDraw

三.自定义View
1.自定义属性,编写value/attrs.xml
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
 <declare-styleable name="test">
     <attr name="text" format="string" />
     <attr name="testAttr" format="integer" />
 </declare-styleable>
</resources>
```
2.在自定义类中使用
```
public class MyTextView extends View {
    public MyTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
        TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.test);
        String text = ta.getString(R.styleable.test_testAttr);
        int textAttr = ta.getInteger(R.styleable.test_text, -1);
        ta.recycle();
    }
}
```
3.在布局中使用
```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:demo="http://schemas.android.com/apk/res/com.example.test" //自定义命名空间
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <com.example.test.MyTextView
        android:layout_width="100dp"
        android:layout_height="200dp"
        demo:testAttr="520"
        demo:text="hello world" />

</RelativeLayout>
```
