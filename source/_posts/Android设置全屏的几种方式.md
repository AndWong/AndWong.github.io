---
title: Android设置全屏的几种方式
date: 2017-02-10 19:39:20
tags: Android
---

1. 将状态栏导航栏透明化，API19以上有效
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    //透明状态栏
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
    //透明导航栏
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);
}

2. 设置全屏参数
getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);

3. 沉浸式
具体参照 ： http://blog.csdn.net/sinyu890807/article/details/51763825
View decorView = getWindow().getDecorView();
int option = View.SYSTEM_UI_FLAG_FULLSCREEN | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION;
decorView.setSystemUiVisibility(option);
