---
title: Android动态修改某个控件大小
date: 2016-06-10 20:05:15
tags: Android
---

1. 动态修改Margin 和 大小
```
ViewGroup.LayoutParams layoutParams = (ViewGroup.LayoutParams)view.getLayoutParams();
layoutParams.topMargin= top; //像素px
parlayoutParamsams.width = width;
layoutParams.height = height;
view.setLayoutParams(layoutParams);
```
2. 动态修改Padding
```
view.setPadding(left,top,right,bottom);
```
3. 动态修改TextView的drawable
```
Drawable icon = getResources().getDrawable(R.mipmap.img);
icon.setBounds(0,0,iconUp.getMinimumWidth(),iconUp.getMinimumHeight());
tv.setCompoundDrawables(icon, null, null, null);
```
