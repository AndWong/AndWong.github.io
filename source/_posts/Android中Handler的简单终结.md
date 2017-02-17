---
title: Android中Handler的简单终结
date: 2016-05-12 21:37:59
tags: Android
---
1.Handler何用？
在实际开发中Handler是为了解决在子线程更新UI的问题。

2.Handler，Looper，MessageQueue的创建？
一个Handler通常和一个Looper和一个MessageQueue绑定在一起。
在线程中创建一个Handler时通常会同时创建一个Looper（通过Looper.prepare()为当前线程创建一个Looper,
并使用Looper.loop()来开启消息的读取）及MessageQueue（创建Looper的同时创建），
而平时我们在主线程中创建Handler却可以省略该步骤主要是由于主线程在开启时就已自动Looper，
可以通过getMainLooper()来获取主线程的Looper。

3.Looper,Handler,MessageQueue的引用关系?
Looper：好比一个泵，循环不断的在MessageQueue中查询消息
MessageQueue：消息池，用于存放消息

一个线程对应只有一个Looper和一个MessageQueue，但可以有多个Handler。

子线程创建一条MSG通过Handler发送消息到主线程的MessageQueue中，
主线程对应的Looper循环不断的在MessageQueue中查询MSG，
查询到则主线程通过Hanler处理消息。

4.Handler导致内存泄露问题?
一般我们写Handler:
```
Handler mHandler = new Handler() {
@Override
public void handleMessage(Message msg) {
  }
}
```
当使用内部类（包括匿名类）来创建Handler的时候，Handler对象会隐式地持有一个外部类对象（通常是一个Activity）的引用,而常常在Activity退出后，消息队列还有未被处理完的消息，此时activity依然被handler引用，导致内存无法回收而内存泄露。

解决方法：
在Handler中增加一个对Activity的弱引用（WeakReference）：
```
static class MyHandler extends Handler {
WeakReference mActivityReference;

MyHandler(Activity activity) {
mActivityReference= new WeakReference(activity);
}

@Override
public void handleMessage(Message msg) {
final Activity activity = mActivityReference.get();
if (activity != null) {
    }
  }
}
```
对于上面的代码，用户在关闭Activity之后，就算后台线程还没结束，但由于仅有一条来自Handler的弱引用指向Activity，所以GC仍然会在检查的时候把Activity回收掉。这样，内存泄露的问题就不会出现了。
