---
title: Android事件分发机制的简单解读
date: 2017-02-12 20:52:44
tags: Android
---
本篇文章只是简单的终结Android中事件的分发机制便于记忆

首先ViewGroup有onInterceptTouchEvent，dispatchTouchEvent和onTouchEvent方法
而View有dispatchTouchEvent和onTouchEvent方法。

每一个事件都是从ViewGroup开始，
1.父控件ViewGroup先执行onInterceptTouchEvent（拦截），
  onInterceptTouchEvent若返回true表示拦截则由该ViewGroup的onTouchEvent处理，
  （其中onTouchEvent若返回true表示自身消费该事件，若onTouchEvent返回false则执行dispatchTouchEvent（向上反馈）给ViewGroup的父控件处理该onTouchEvent）
  onInterceptTouchEvent若返回false表示不拦截则由该ViewGroup的dispatchTouchEvent（向下传递）给子控件（ViewGroup或View）处理。

2.当事件传递到View控件时，先执行onTouchEvent事件，
  其中onTouchEvent若返回true表示自身消费该事件，
  若onTouchEvent返回false则执行dispatchTouchEvent（向上反馈）给父控件处理该onTouchEvent。
