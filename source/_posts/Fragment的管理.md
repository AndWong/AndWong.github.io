---
title: Fragment的管理
date: 2016-06-17 21:09:27
tags: Android
---
一.生命周期
Fragment的生命周期方法主要有
onAttach()
onCreate()
onCreateView()
onActivityCreated()
onstart()
onResume()
onPause()
onStop()
onDestroyView()
onDestroy()
onDetach()等11个方法。
<!--more-->

切换到该Fragment，分别执行onAttach()、onCreate()、onCreateView()、onActivityCreated()、onstart()、onResume()方法。
锁屏，分别执行onPause()、onStop()方法。
亮屏，分别执行onstart()、onResume()方法。
覆盖，切换到其他Fragment，分别执行onPause()、onStop()、onDestroyView()方法。
从其他Fragment回到之前Fragment，分别执行onCreateView()、onActivityCreated()、onstart()、onResume()方法。

二.Fragment与activity如何传值和交互？
Fragment对象有一个getActivity的方法，通过该方法与activity交互.
使用framentmentManager.findFragmentByXX可以获取fragment对象，在activity中直接操作fragment对象
