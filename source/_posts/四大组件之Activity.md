---
title: 四大组件之Activity
date: 2017-02-15 19:48:09
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

2.启动模式
（在AndroidManifest.xml中的android:launchMode="singleTask"配置）
 (1)standard : 默认的标准模式，每次启动Activity都会创建Activity并放人Task栈中
 (2)singleTop ：当要启动的Activity刚好在Task栈的栈顶时使用这个，否则重新创建一个
 (3)singleTask ： 只要Task栈中存在就使用，并把该Activity以上的其他Actitity弹出栈中
 (4)singleInstance ： 只要在该Activity存在Task栈中，如何应用启动都可以直接调用

3.闪屏页的快速启动
4.如何禁止横竖屏切换生命周期变换
5.Activity设置为首页Activity
6.启动Actitiy的几种方式
