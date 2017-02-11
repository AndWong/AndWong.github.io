---
title: Android自定义圆形进度条
date: 2017-02-10 20:08:02
tags: Android自定义圆形进度条
---
1.res目录下定义旋转动画 progress_circle：
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/icon_wifi_connect_gif"
    android:fromDegrees="0"
    android:pivotX="50%"
    android:pivotY="50%"
    android:toDegrees="360" >
</rotate>

2.定义style：
<style name="mProgress_circle">
    <item name="android:indeterminateDrawable">@drawable/progress_circle</item>
    <item name="android:minWidth">25dp</item>
    <item name="android:minHeight">25dp</item>
    <item name="android:maxWidth">60dp</item>
    <item name="android:maxHeight">60dp</item>
</style>

3.控件中设置style
<ProgressBar
    android:id="@+id/pb001"
    style="@style/mProgress_circle"
    android:layout_width="15dp"
    android:layout_height="15dp"
    android:indeterminateDuration="1200" />
