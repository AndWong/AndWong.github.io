---
title: 四大组件之Activity
date: 2016-10-15 19:48:09
tags: 四大组件之Activity
---

Activity乃Andoid四大组件中的一个，主要负责用户界面的展示和用户交互处理。

1.生命周期
正常的一次启动到结束 ：onCreate -> onStart -> onResume -> onPause -> onStop -> onDestory

切换到后台 : onPause -> onStop
从后台回来 : onRestart ->onStart -> onResume

从A切到B ： A-onPause -> B-onCreate -> B-onStart -> B-onResume -> A-onStop
从B返回A ： B-onPause -> A-onRestart -> A-onStart -> A-onResume -> B-onStop -> B-onDestroy

横竖屏切换 ： onSaveInstanceState -> onPause -> onStop -> onDestory -> onCreate -> onStart -> onRestoreInstanceState -> onResume
<!--more-->
2.启动模式
（在AndroidManifest.xml中的android:launchMode="singleTask"配置）
 (1)standard : 默认的标准模式，每次启动Activity都会创建Activity并放人Task栈中
 (2)singleTop ：当要启动的Activity刚好在Task栈的栈顶时使用这个，否则重新创建一个
 (3)singleTask ： 只要Task栈中存在就使用，并把该Activity以上的其他Actitity弹出栈中
 (4)singleInstance ： 只要在该Activity存在Task栈中，如何应用启动都可以直接调用

3.闪屏页的快速启动
  通常闪屏页是一张产品的Logo图，所以可以通过为闪屏页设置theme，
  让app启动时activity为展示时先显示theme中的背景图，
  以此达到闪屏页的“快速启动”，否则闪屏页显示的app-theme中背景色。

  1)在AndroidManifest中为闪屏页设置Theme：
  ```
  android:theme="@style/AppSplash"
  ```

  2)AppSplash如下：
  ```
  <style name="AppSplash">
    <item name="android:windowBackground">@drawable/layer_logo_page</item>
  </style>
  ```

  3)layer_logo_page如下：
  ```
  <?xml version="1.0" encoding="utf-8"?>
  <layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <color android:color="@color/C15_white" />
    </item>
    <item android:bottom="35dp">
        <bitmap
            android:gravity="center_horizontal|bottom"
            android:src="@drawable/icon_logo" />
    </item>
  </layer-list>
  ```

4.横竖屏
  1)如何禁止横竖屏切换时生命周期变换
  AndroidManifest中为Activity添加配置：
```
  android:configChanges="orientation|keyboardHidden|screenSize"
```
  2)如何禁止横屏：
  AndroidManifest中为Activity添加配置：
```
   android:screenOrientation="portrait"
```
  3)Java代码中设置横竖屏：
  // 如果是竖屏则转为横屏
```
  if (activity.getRequestedOrientation() == ActivityInfo.SCREEN_ORIENTATION_PORTRAIT) {
    activity.setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
  }
```

  4)横竖屏变换时通常会有回调：
```
  @Override
  public void onConfigurationChanged(Configuration newConfig) {
    super.onConfigurationChanged(newConfig);
  }
```

5.Activity设置为首页
  为Activity配置如下代码：
```
  <intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
  </intent-filter>
```
6.启动Actitiy的几种方式
